---
title : "Frontend — AWS Amplify Hosting"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 6. </b> "
---

#### Tổng quan

Khi backend đã chạy sau API Gateway và Cognito đã được đấu nối, đã đến lúc dựng ứng dụng web ở phía trước. Trong chương này bạn sẽ host frontend của LingoRise trên **AWS Amplify Hosting**. Amplify kết nối trực tiếp tới kho GitHub của bạn, tự động nhận diện ứng dụng **Next.js** bên trong monorepo, build lại mỗi lần bạn push, và phục vụ qua HTTPS từ một CDN toàn cầu — không cần quản lý máy chủ build nào cả.

Bạn sẽ kết nối kho mã, cung cấp cho Amplify các biến môi trường trỏ ứng dụng web tới những tài nguyên bạn đã tạo ở Chương 5, chạy lần build đầu tiên để lấy URL đã host, và cuối cùng siết chặt CORS ở backend để chỉ tên miền Amplify của bạn mới được phép gọi API.

{{% notice info %}}
Mọi thao tác ở đây đều nhắm tới stage **`dev`** trong vùng **`ap-southeast-1`**, dùng CloudFormation stack **`lingorise-dev`**. Frontend nằm trong thư mục **`lingorise-app`** của monorepo và build từ nhánh **`votienkhoa-fullstack`**.
{{% /notice %}}

#### Kết nối kho mã

Mở **AWS Amplify Console** ở vùng `ap-southeast-1`, sau đó chọn **New app → Host web app**. Chọn **GitHub** làm nguồn và cấp quyền cho Amplify truy cập tài khoản của bạn.

1. Chọn kho **LingoRise**.
2. Chọn nhánh **`votienkhoa-fullstack`**.
3. Đặt tên app là **`lingorise-dev`**.
4. Vì đây là monorepo, hãy đặt **app root (gốc monorepo)** là **`lingorise-app`**.
5. Amplify tự động nhận diện **Next.js** và điền sẵn cấu hình build — giữ nguyên tệp `amplify.yml` được sinh ra.

![Kết nối kho GitHub LingoRise trong Amplify Console](/images/Workshop-LingoRise/6-frontend-amplify/amplify-connect-repo.png)

{{% notice note %}}
Nếu Amplify không tự nhận diện framework, hãy kiểm tra rằng app root được đặt là `lingorise-app` — trỏ vào gốc kho sẽ khiến dự án Next.js bị ẩn khỏi quá trình build.
{{% /notice %}}

#### Cấu hình biến môi trường

Frontend đọc một số biến `NEXT_PUBLIC_*` tại thời điểm build để biết API nằm ở đâu và làm sao giao tiếp với Cognito. Trong mục **Environment variables** của phần cài đặt app, thêm các biến sau:

| Biến | Giá trị | Nguồn |
|---|---|---|
| `NEXT_PUBLIC_API_URL` | `<ApiUrl output>` | `ApiUrl` từ stack `lingorise-dev` |
| `NEXT_PUBLIC_COGNITO_USER_POOL_ID` | `<UserPoolId output>` | `UserPoolId` từ stack `lingorise-dev` |
| `NEXT_PUBLIC_COGNITO_CLIENT_ID` | `<UserPoolClientId output>` | `UserPoolClientId` từ stack `lingorise-dev` |
| `NEXT_PUBLIC_COGNITO_REGION` | `ap-southeast-1` | Cố định — vùng của workshop |
| `NEXT_PUBLIC_USE_MOCK` | `false` | Cố định — gọi API thật, không dùng mock |

![Thiết lập các biến môi trường NEXT_PUBLIC trong Amplify](/images/Workshop-LingoRise/6-frontend-amplify/amplify-env-vars.png)

{{% notice tip %}}
Giá trị của `NEXT_PUBLIC_API_URL`, `NEXT_PUBLIC_COGNITO_USER_POOL_ID` và `NEXT_PUBLIC_COGNITO_CLIENT_ID` lấy **trực tiếp từ các output của stack ở Chương 5**. Nếu bạn đã đóng cửa sổ terminal đó, hãy in lại chúng:

```powershell
aws cloudformation describe-stacks `
  --stack-name lingorise-dev `
  --region ap-southeast-1 `
  --profile lingorise-dev `
  --query "Stacks[0].Outputs"
```

```bash
aws cloudformation describe-stacks \
  --stack-name lingorise-dev \
  --region ap-southeast-1 \
  --profile lingorise-dev \
  --query "Stacks[0].Outputs"
```
{{% /notice %}}

#### Chạy lần build đầu tiên

Lưu các biến môi trường và chọn **Save and deploy**. Amplify clone nhánh, cài dependency, chạy build Next.js, và triển khai kết quả lên CDN được quản lý của nó. Bạn có thể theo dõi trực tiếp các giai đoạn **Provision → Build → Deploy → Verify** trong console.

Khi pipeline chuyển sang màu xanh, Amplify hiển thị URL đã host — đại loại như `https://votienkhoa-fullstack.<app-id>.amplifyapp.com`. Mở URL đó và xác nhận trang chủ LingoRise hiển thị đúng.

![Amplify build thành công cùng URL đã host](/images/Workshop-LingoRise/6-frontend-amplify/amplify-build-success.png)

Sao chép URL đã host đó — bạn cần nó ở bước tiếp theo để siết chặt CORS.

{{% notice note %}}
Ở lần tải đầu tiên này, ứng dụng có thể gọi được API, nhưng backend vẫn cho phép một origin CORS khá lỏng từ các lần deploy trước. Việc đăng nhập và các lời gọi API phụ thuộc vào origin nghiêm ngặt có thể hoạt động sai cho tới khi bạn hoàn thành bước tiếp theo.
{{% /notice %}}

#### Siết chặt CORS

Hiện tại backend đã được triển khai mà chưa biết địa chỉ thật của frontend. Hãy deploy lại SAM stack, truyền vào URL Amplify để API Gateway và S3 chỉ chấp nhận yêu cầu từ ứng dụng web của bạn:

```powershell
sam deploy `
  --stack-name lingorise-dev `
  --region ap-southeast-1 `
  --profile lingorise-dev `
  --parameter-overrides FrontendUrl=<amplify-url>
```

```bash
sam deploy \
  --stack-name lingorise-dev \
  --region ap-southeast-1 \
  --profile lingorise-dev \
  --parameter-overrides FrontendUrl=<amplify-url>
```

Thay `<amplify-url>` bằng URL đã host ở bước trước (ví dụ `https://votienkhoa-fullstack.<app-id>.amplifyapp.com`). Chỉ một tham số override này cập nhật đồng thời hai thứ:

- **CORS của API Gateway** — `Access-Control-Allow-Origin` của REST API được đặt về tên miền Amplify của bạn.
- **CORS của S3** — bucket tài nguyên `lingorise-assets-dev-<accountId>` chỉ chấp nhận upload và đọc từ trình duyệt đến từ origin frontend của bạn.

{{% notice warning %}}
Dùng đúng scheme và host (`https://…`, không có dấu `/` ở cuối) mà trình duyệt gửi lên. Origin không khớp là nguyên nhân phổ biến nhất khiến yêu cầu bị chặn `CORS` sau bước này.
{{% /notice %}}

#### Kiểm tra

Tải lại URL Amplify, mở tab **Network** trong dev tools của trình duyệt, và thử đăng ký hoặc đăng nhập. Các lời gọi API tới `NEXT_PUBLIC_API_URL` phải trả về `200`/`401` (không phải lỗi CORS), và phần phản hồi phải kèm header `Access-Control-Allow-Origin` khớp với tên miền Amplify của bạn. Nếu đúng như vậy, frontend và backend đã được gắn kết và khóa chặt chính xác.
