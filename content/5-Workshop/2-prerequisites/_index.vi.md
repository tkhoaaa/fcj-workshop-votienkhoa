---
title : "Chuẩn bị"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

#### Tổng quan

Trước khi khởi tạo bất cứ thứ gì trên AWS, hãy chuẩn bị sẵn sàng bộ công cụ trên máy của bạn và quyền truy cập AWS. Chương này cài đặt bốn công cụ mà workshop sử dụng, tạo một danh tính IAM chuyên dụng để triển khai (deployer) nhằm tránh làm việc bằng tài khoản root, và thiết lập một CLI profile có tên (`lingorise-dev`) mà mọi lệnh về sau đều dùng đến. Khi `aws sts get-caller-identity` trả về đúng tài khoản bạn mong đợi, bạn đã sẵn sàng để bắt đầu xây dựng.

Trong quá trình này, chúng ta sẽ lưu lại hai giá trị — **AWS account ID** (`$AccountId`) và **stage** (`$Stage = "dev"`). Chúng được dùng cho tên tài nguyên ở các bước sau, đáng chú ý nhất là S3 asset bucket `lingorise-assets-dev-<accountId>`.

#### Công cụ cần cài đặt

Cài đặt các công cụ sau, rồi xác nhận từng công cụ in ra được phiên bản. Cả bốn đều miễn phí và chạy đa nền tảng.

- **AWS CLI v2** — giao diện dòng lệnh cho AWS. [Hướng dẫn cài đặt](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
- **AWS SAM CLI** — đóng gói và triển khai stack backend serverless. [Hướng dẫn cài đặt](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html)
- **Node.js 20** — runtime của Lambda; đồng thời chạy công cụ build và các migration cơ sở dữ liệu. [Tải về](https://nodejs.org/en/download)
- **PostgreSQL client 16 (`psql`)** — kết nối tới RDS để chạy migration và kiểm tra. [Tải về](https://www.postgresql.org/download/)

Xác nhận từng bản cài đặt:

```powershell
aws --version
sam --version
node --version
psql --version
```

```bash
aws --version
sam --version
node --version
psql --version
```

{{% notice tip %}}
Bạn cần **AWS CLI v2** (không phải v1) và **Node 20.x**. Nếu `aws --version` báo bản `1.x`, hãy gỡ bỏ và cài lại v2 từ liên kết ở trên — một vài lệnh trong workshop này hoạt động khác đi trên v1.
{{% /notice %}}

#### Quyền IAM

Thay vì triển khai bằng tài khoản root, hãy tạo một IAM user chuyên dụng tên **`lingorise-deployer`** và cấp cho nó quyền truy cập lập trình (một access key). User này chính là danh tính đứng sau CLI profile `lingorise-dev`.

Với môi trường **dev**, hãy gắn policy do AWS quản lý **`PowerUserAccess`** và thêm một inline policy nhỏ cấp quyền **`iam:*`** — SAM cần tạo các execution role và policy của Lambda trong quá trình triển khai `lingorise-dev`, trong khi riêng `PowerUserAccess` không bao gồm các quyền ghi (write) đối với IAM.

Inline policy để gắn vào `lingorise-deployer`:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "LingoRiseIamForSam",
            "Effect": "Allow",
            "Action": "iam:*",
            "Resource": "*"
        }
    ]
}
```

![Tạo IAM user lingorise-deployer](/images/Workshop-LingoRise/2-prerequisites/iam-create-user.png)

{{% notice warning %}}
**Không** dùng thông tin đăng nhập của tài khoản **root** AWS cho workshop này. Root có quyền truy cập không giới hạn vào mọi thứ trong tài khoản, bao gồm cả thanh toán và đóng tài khoản, và khóa (key) của nó không thể thu hẹp phạm vi. Luôn triển khai thông qua một IAM user chuyên dụng như `lingorise-deployer`.
{{% /notice %}}

{{% notice note %}}
`PowerUserAccess` + inline `iam:*` là tiện lợi cho tài khoản **dev** khi bạn là người dùng duy nhất. Với **production**, hãy thu hẹp phạm vi này chỉ còn các dịch vụ mà workshop chạm tới (CloudFormation, Lambda, API Gateway, RDS, S3, Cognito, CloudFront, SSM, CloudWatch, Amplify) và giới hạn theo ARN tài nguyên cụ thể khi có thể.
{{% /notice %}}

#### Cấu hình CLI profile

Với access key từ bước trước, hãy tạo một profile có tên **`lingorise-dev`**. Chạy `aws configure` và dán key, secret, region cùng định dạng đầu ra khi được hỏi:

```powershell
aws configure --profile lingorise-dev
# AWS Access Key ID [None]: <paste>
# AWS Secret Access Key [None]: <paste>
# Default region name [None]: ap-southeast-1
# Default output format [None]: json
```

```bash
aws configure --profile lingorise-dev
# AWS Access Key ID [None]: <paste>
# AWS Secret Access Key [None]: <paste>
# Default region name [None]: ap-southeast-1
# Default output format [None]: json
```

![Chạy aws configure cho profile lingorise-dev](/images/Workshop-LingoRise/2-prerequisites/cli-configure.png)

Tiếp theo, xuất profile và region vào shell của bạn để không phải truyền `--profile` trong từng lệnh. Thiết lập chúng cho phiên làm việc hiện tại:

```powershell
$env:AWS_PROFILE = "lingorise-dev"
$env:AWS_DEFAULT_REGION = "ap-southeast-1"
```

```bash
export AWS_PROFILE=lingorise-dev
export AWS_DEFAULT_REGION=ap-southeast-1
```

Bây giờ hãy xác nhận CLI đang giao tiếp với đúng tài khoản và lưu account ID cùng stage vào các biến để tái sử dụng:

```powershell
aws sts get-caller-identity
$AccountId = (aws sts get-caller-identity --query Account --output text)
$Stage = "dev"
Write-Host "AccountId=$AccountId  Stage=$Stage"
```

```bash
aws sts get-caller-identity
AccountId=$(aws sts get-caller-identity --query Account --output text)
Stage=dev
echo "AccountId=$AccountId  Stage=$Stage"
```

Một lệnh `get-caller-identity` thành công trông như thế này — lưu ý `Arn` kết thúc bằng `user/lingorise-deployer`:

```json
{
    "UserId": "AIDA...EXAMPLE",
    "Account": "<accountId>",
    "Arn": "arn:aws:iam::<accountId>:user/lingorise-deployer"
}
```

![Kết quả aws sts get-caller-identity](/images/Workshop-LingoRise/2-prerequisites/caller-identity.png)

#### Kiểm tra

Bạn đã sẵn sàng tiếp tục khi tất cả những điều sau đều đúng:

- `aws --version`, `sam --version`, `node --version`, và `psql --version` đều in ra được phiên bản (AWS CLI v2, Node 20).
- IAM user `lingorise-deployer` đã tồn tại với `PowerUserAccess` + inline `iam:*`.
- `aws sts get-caller-identity` trả về đúng tài khoản của bạn và ARN `user/lingorise-deployer`.
- `$AccountId` và `$Stage` đã được thiết lập trong shell.

{{% notice note %}}
Mọi lệnh trong các chương tiếp theo đều giả định profile **`lingorise-dev`** đang hoạt động — thông qua `$env:AWS_PROFILE` / `export AWS_PROFILE=lingorise-dev`, hoặc bằng cách thêm `--profile lingorise-dev`. Nếu một lệnh báo lỗi xác thực (authentication) hoặc region, hãy kiểm tra lại hai biến môi trường này trước tiên.
{{% /notice %}}
