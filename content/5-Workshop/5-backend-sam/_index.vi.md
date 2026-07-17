---
title : "Backend — Triển khai AWS SAM"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

#### Tổng quan

AWS SAM (Serverless Application Model) là phần mở rộng của CloudFormation được thiết kế riêng cho ứng dụng serverless. Bạn mô tả toàn bộ backend trong một file `template.yaml` duy nhất — các hàm Lambda, một API Gateway HTTP API, một Cognito user pool, các IAM policy và một S3 bucket — rồi SAM biến khai báo đó thành một CloudFormation stack thực sự.

Với LingoRise, `template.yaml` ánh xạ mỗi route backend thành một `AWS::Serverless::Function` (mã handler được biên dịch từ `src/handlers/**`), đặt chúng phía sau một API Gateway dùng chung, và xử lý xác thực thông qua `CognitoUserPool` + `CognitoUserPoolClient`. Các giá trị nhạy cảm được lấy tại thời điểm triển khai từ SSM Parameter Store bằng `{{resolve:ssm:/lingorise/${Stage}/...}}`, nên không có secret nào nằm trong template.

Trước khi triển khai, template cần khắc phục năm gap về mức độ sẵn sàng. Chúng ta áp dụng chúng theo một thứ tự có chủ đích, kiểm tra tính hợp lệ, build, rồi chạy triển khai có hướng dẫn.

{{% notice info %}}
Mọi thao tác trong chương này chạy từ thư mục `lingorise-backend/` với profile CLI `lingorise-dev` trên region `ap-southeast-1`. Stage là `dev`; stack được đặt tên `lingorise-dev`.
{{% /notice %}}

#### Áp dụng các gap sẵn sàng

Phần khung của template (Lambda, API Gateway, Cognito, CORS, phân giải SSM) đã ở trạng thái tốt. Các gap dưới đây là những phần bắt buộc phải hoàn thiện trước khi các route asset, OCR và thanh toán hoạt động tại runtime. Hãy áp dụng theo thứ tự sau.

**Gap 2 — sửa build script trước.** Câu lệnh esbuild hiện tại đang externalize các thư viện thuần JS (`mammoth`, `pdf-parse`, `tesseract.js`, `@anthropic-ai/sdk`) mà SAM sẽ không đóng gói, khiến `/admin/import/parse` và OCR worker ném lỗi `MODULE_NOT_FOUND` tại runtime. Bỏ các external đó và chỉ giữ lại các binary native. Sửa `lingorise-backend/package.json`:

```json
"build": "esbuild src/handlers/**/*.ts --bundle --platform=node --target=node20 --outdir=dist --external:pg-native --external:@napi-rs/canvas --external:@napi-rs/canvas-*"
```

Câu lệnh này tạo ra các bundle độc lập cho từng handler. `pg-native` và `@napi-rs/canvas` vẫn để external vì chúng là binary native không được phép bundle.

**Gap 3 — khai báo S3 asset bucket.** Mã đọc `ASSET_S3_BUCKET`, nhưng chưa có resource bucket nào. Thêm khối này vào `Resources:`:

```yaml
  AssetBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub "lingorise-assets-${Stage}-${AWS::AccountId}"
      OwnershipControls:
        Rules:
          - ObjectOwnership: BucketOwnerEnforced
      PublicAccessBlockConfiguration:
        BlockPublicAcls: true
        BlockPublicPolicy: true
        IgnorePublicAcls: true
        RestrictPublicBuckets: true
      CorsConfiguration:
        CorsRules:
          - AllowedMethods: [GET, HEAD]
            AllowedOrigins:
              - !Ref FrontendUrl
            AllowedHeaders: ["*"]
            MaxAge: 3000
      LifecycleConfiguration:
        Rules:
          - Id: ExpireImportDrafts
            Status: Enabled
            Prefix: import-drafts/
            ExpirationInDays: 7
          - Id: AbortIncompleteMultipart
            Status: Enabled
            AbortIncompleteMultipartUpload:
              DaysAfterInitiation: 1
```

…và đưa tên bucket ra `Outputs:`:

```yaml
  AssetBucketName:
    Description: S3 bucket holding question assets and import drafts
    Value: !Ref AssetBucket
```

Bucket mặc định ở chế độ riêng tư (`BlockPublicAcls=true`), phù hợp với presigned URL do backend cấp.

**Gap 4 — truyền các biến môi trường asset & OCR.** Không biến `ASSET_*` / `OCR_*` nào mà mã cần đang có trong `Globals`. Thêm chúng vào khối `Globals.Function.Environment.Variables` sẵn có:

```yaml
        ASSET_STORAGE_DRIVER: "s3"
        ASSET_S3_BUCKET: !Ref AssetBucket
        ASSET_S3_PREFIX: ""
        ASSET_PUBLIC_BASE_URL: ""
        ASSET_S3_FORCE_PATH_STYLE: "false"
        ASSET_URL_MODE: "signed"
        ASSET_SIGNED_URL_TTL_SECONDS: "900"
        OCR_ENABLED: "false"
        OCR_PROVIDER: "disabled"
        OCR_RUN_MODE: "queue"
        OCR_LANG: "eng"
        OCR_TIMEOUT_MS: "60000"
```

Bucket riêng tư kết hợp presigned URL (`ASSET_URL_MODE: signed`) là trạng thái khởi đầu an toàn nhất cho dev. OCR mặc định tắt; bật theo từng stage về sau bằng cách chuyển `OCR_ENABLED=true` và `OCR_PROVIDER=tesseract`.

**Gap 5 — cấp IAM cho Lambda → S3 và Lambda → Cognito.** SAM không tự cấp quyền cho `{{resolve:ssm:...}}`, và mã asset/auth cần quyền trên bucket và user pool. Thêm khối `Policies:` vào `Globals.Function` để mọi hàm thừa hưởng quyền truy cập theo nguyên tắc tối thiểu:

```yaml
Globals:
  Function:
    # ...existing entries unchanged
    Policies:
      - S3CrudPolicy:
          BucketName: !Ref AssetBucket
      - Statement:
          - Effect: Allow
            Action:
              - cognito-idp:AdminInitiateAuth
              - cognito-idp:AdminGetUser
              - cognito-idp:AdminCreateUser
              - cognito-idp:AdminSetUserPassword
              - cognito-idp:ListUsers
            Resource: !GetAtt CognitoUserPool.Arn
```

`S3CrudPolicy` là policy template do SAM quản lý, giới hạn phạm vi trong bucket đã nêu; nó cũng bao gồm `s3:ListBucket` cần cho việc dọn dẹp import-draft.

**Gap 1 — rà soát độ phủ route sau cùng.** Template chỉ khai báo một tập con các route đã đăng ký trong `server.ts`. Trước khi triển khai, đối chiếu `template.yaml` với `server.ts` và thêm một entry `AWS::Serverless::Function` cho mỗi route bạn định expose. Handler tuân theo mẫu sẵn có: `dist/handlers/{group}/{file}.{export}`. Các route thường bị thiếu gồm các endpoint OCR/import của admin (`/admin/ocr/jobs`, `/admin/import/jobs`), các route asset của question-bank, `/admin/audit-logs`, và các endpoint thanh toán VNPay v2 / SePay (`/payments/vnpay/**`, `/payments/sepay/**`).

{{% notice tip %}}
Chạy gap 2 đến 5 trước vì chúng thay đổi trực tiếp template và build script. Để dành phần rà soát route (gap 1) sau cùng để bạn duyệt danh sách resource cuối cùng so với `server.ts` ngay trước khi kiểm tra tính hợp lệ.
{{% /notice %}}

#### Kiểm tra tính hợp lệ của template

Sau khi áp dụng các gap, hãy lint template trước khi build. Bước này bắt sớm các lỗi gõ nhầm, thụt lề sai và tham chiếu chưa phân giải được.

```powershell
sam validate --lint
```

```bash
sam validate --lint
```

Một lần chạy sạch sẽ báo template hợp lệ. Hãy sửa mọi lỗi được báo trước khi tiếp tục.

#### Build

Trước tiên biên dịch các handler TypeScript bằng esbuild (đây chính là phần chịu ảnh hưởng của thay đổi ở gap 2), rồi để SAM đóng gói mọi thứ cho Lambda.

```powershell
npm run build
sam build
```

```bash
npm run build
sam build
```

`npm run build` phải hoàn tất mà không có cảnh báo nào về module bị thiếu — điều đó xác nhận bản sửa gap 2 đã bundle được các dependency thuần JS. `sam build` sau đó lắp ráp các artifact triển khai vào `.aws-sam/`.

#### Triển khai

Chạy triển khai có hướng dẫn. SAM sẽ dẫn bạn qua từng tham số một cách tương tác và có thể lưu câu trả lời vào `samconfig.toml` cho các lần triển khai sau.

```powershell
sam deploy --guided --stack-name lingorise-dev --region ap-southeast-1
```

```bash
sam deploy --guided --stack-name lingorise-dev --region ap-southeast-1
```

Trả lời các prompt của wizard như sau:

| Tham số | Giá trị |
|---|---|
| Stack Name | `lingorise-dev` |
| AWS Region | `ap-southeast-1` |
| Stage | `dev` |
| FrontendUrl | `<your-frontend-url>` |
| VnpayTmnCode | `<paste>` |
| MomoPartnerCode | `<paste>` |
| AiProvider | `openrouter` |
| OllamaBaseUrl | (để mặc định / để trống) |
| OpenRouterModel | `<your-openrouter-model>` |
| LogRetentionDays | `30` |
| Confirm changes before deploy | `Y` |
| Allow SAM CLI IAM role creation | `Y` |
| Save arguments to samconfig.toml | `Y` |

{{% notice warning %}}
`AiProvider` bắt buộc phải là `openrouter`. Các hàm Lambda không thể truy cập `localhost:11434`, nên provider Ollama sẽ không bao giờ hoạt động trong môi trường đã triển khai — nó chỉ tồn tại cho phát triển cục bộ. Hãy đặt là `openrouter` (kèm một `OpenRouterModel` hợp lệ), nếu không các tính năng dùng AI sẽ lỗi tại runtime.
{{% /notice %}}

SAM hiển thị changeset CloudFormation — chính xác những resource nó sẽ tạo hoặc thay đổi. Hãy xem lại rồi xác nhận để thực thi.

![SAM deploy changeset review](/images/Workshop-LingoRise/5-backend-sam/sam-deploy-changeset.png)

CloudFormation cấp phát stack: các hàm Lambda, API Gateway, Cognito user pool, các IAM role và S3 asset bucket. Chờ đến trạng thái `CREATE_COMPLETE`.

![CloudFormation stack CREATE_COMPLETE](/images/Workshop-LingoRise/5-backend-sam/cfn-stack-complete.png)

{{% notice note %}}
Vì bạn đã trả lời `Y` cho việc lưu tham số, một file `samconfig.toml` được ghi vào `lingorise-backend/`. Các lần triển khai sau chỉ cần `sam build && sam deploy` — không cần `--guided` trừ khi có tham số thay đổi.
{{% /notice %}}

#### Ghi lại các output

Khi triển khai xong, SAM in ra các output của stack. Bạn cũng có thể lấy chúng bất kỳ lúc nào bằng CLI:

```powershell
aws cloudformation describe-stacks --stack-name lingorise-dev --region ap-southeast-1 --profile lingorise-dev --query "Stacks[0].Outputs" --output table
```

```bash
aws cloudformation describe-stacks --stack-name lingorise-dev --region ap-southeast-1 --profile lingorise-dev --query "Stacks[0].Outputs" --output table
```

Ghi lại bốn giá trị này — frontend và các chương sau đều phụ thuộc vào chúng:

- **ApiUrl** — URL gốc của API Gateway mà frontend sẽ gọi.
- **UserPoolId** — ID của Cognito user pool dùng cho xác thực.
- **UserPoolClientId** — ID của app client Cognito mà frontend dùng để đăng nhập.
- **AssetBucketName** — S3 bucket chứa asset câu hỏi và import draft (từ gap 3).

![CloudFormation stack outputs](/images/Workshop-LingoRise/5-backend-sam/stack-outputs.png)

#### Kiểm tra

Backend đã hoạt động khi stack ở trạng thái `CREATE_COMPLETE` và cả bốn output đều có giá trị. Một bài kiểm tra nhanh: gọi một endpoint công khai trên `ApiUrl` (ví dụ một route health) và xác nhận bạn nhận được phản hồi qua API Gateway. Hãy giữ sẵn `ApiUrl`, `UserPoolId` và `UserPoolClientId` — bạn sẽ nối chúng vào frontend ở chương tiếp theo.
