---
title : "Bí mật & SSM Parameter Store"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 3. </b> "
---

#### Tổng quan

LingoRise dựa vào một loạt dịch vụ bên thứ ba — VNPay và MoMo cho thanh toán, OpenRouter cho việc chấm điểm bằng AI, cùng vài API nội dung (YouTube, NewsAPI, GNews, Reddit) — và mỗi dịch vụ đều đi kèm một API key hoặc một chuỗi bí mật để ký. Thay vì nhúng các giá trị đó vào code hay file môi trường, chúng ta lưu chúng trong **AWS Systems Manager (SSM) Parameter Store** dưới dạng tham số `SecureString`.

`SecureString` mang lại hai lợi ích miễn phí. Thứ nhất, giá trị được **mã hóa khi lưu trữ bằng khóa KMS** (ở đây là khóa mặc định do AWS quản lý), nên không ai duyệt console mà nhìn thấy được nội dung gốc. Thứ hai, việc truy xuất được ghi log và kiểm soát — bạn phải yêu cầu giải mã một cách rõ ràng bằng `--with-decryption` mới đọc lại được giá trị.

Có một lý do về thứ tự bắt buộc phải làm việc này *ngay bây giờ*, trước khi triển khai bất cứ thứ gì. SAM template tham chiếu các tham số này bằng biểu thức `{{resolve:ssm:...}}`, và CloudFormation phân giải chúng **tại thời điểm triển khai**. Nếu bất kỳ tham số được tham chiếu nào bị thiếu, stack sẽ lỗi ngay lập tức. Vì vậy chúng ta gieo trước dữ liệu vào Parameter Store, rồi xây dựng tiếp trên nền đó ở các chương sau.

Mọi thứ chạy tại **ap-southeast-1** với CLI profile `lingorise-dev`. Các ví dụ dùng PowerShell (Windows) kèm bản tương đương bash; các lệnh `aws ssm put-parameter` bên dưới đều nằm trên một dòng, nên chúng đọc y hệt nhau ở cả hai shell.

![SSM Parameter Store list showing the /lingorise/dev/ parameters](/images/Workshop-LingoRise/3-secrets-ssm/ssm-parameters-list.png)

#### Tạo các tham số

Gieo mọi bí mật ngoại trừ `DATABASE_URL` dưới root `/lingorise/dev/`. Dùng `--type SecureString` để mỗi giá trị được mã hóa khi lưu trữ. Thay các placeholder `<your-...>` bằng thông tin sandbox thật khi bạn dán vào.

{{% notice info %}}
`DATABASE_URL` cố ý **không** có ở đây. Bạn chưa có endpoint RDS — chúng ta sẽ lắp ráp và lưu chuỗi kết nối đầy đủ ở **Chương 4 (Database — RDS PostgreSQL)** sau khi instance chạy xong.
{{% /notice %}}

PowerShell:

```powershell
# VNPay sandbox
aws ssm put-parameter --name "/lingorise/dev/VNPAY_HASH_SECRET" --type SecureString --value "<your-vnpay-hash-secret>" --profile lingorise-dev --region ap-southeast-1

# MoMo sandbox
aws ssm put-parameter --name "/lingorise/dev/MOMO_ACCESS_KEY" --type SecureString --value "<your-momo-access-key>" --profile lingorise-dev --region ap-southeast-1
aws ssm put-parameter --name "/lingorise/dev/MOMO_SECRET_KEY" --type SecureString --value "<your-momo-secret-key>" --profile lingorise-dev --region ap-southeast-1

# AI provider keys
aws ssm put-parameter --name "/lingorise/dev/OPENROUTER_API_KEY" --type SecureString --value "<your-openrouter-key>" --profile lingorise-dev --region ap-southeast-1

# Content APIs
aws ssm put-parameter --name "/lingorise/dev/YOUTUBE_API_KEY" --type SecureString --value "<your-youtube-key>" --profile lingorise-dev --region ap-southeast-1
aws ssm put-parameter --name "/lingorise/dev/NEWS_API_KEY" --type SecureString --value "<your-newsapi-key>" --profile lingorise-dev --region ap-southeast-1
aws ssm put-parameter --name "/lingorise/dev/GNEWS_API_KEY" --type SecureString --value "<your-gnews-key>" --profile lingorise-dev --region ap-southeast-1
aws ssm put-parameter --name "/lingorise/dev/REDDIT_CLIENT_ID" --type SecureString --value "<your-reddit-client-id>" --profile lingorise-dev --region ap-southeast-1
aws ssm put-parameter --name "/lingorise/dev/REDDIT_CLIENT_SECRET" --type SecureString --value "<your-reddit-client-secret>" --profile lingorise-dev --region ap-southeast-1

# Database password (dùng ở Chương 4 bởi RDS)
aws ssm put-parameter --name "/lingorise/dev/DB_PASSWORD" --type SecureString --value "<your-strong-password>" --profile lingorise-dev --region ap-southeast-1
```

bash:

```bash
# VNPay sandbox
aws ssm put-parameter --name "/lingorise/dev/VNPAY_HASH_SECRET" --type SecureString --value "<your-vnpay-hash-secret>" --profile lingorise-dev --region ap-southeast-1

# MoMo sandbox
aws ssm put-parameter --name "/lingorise/dev/MOMO_ACCESS_KEY" --type SecureString --value "<your-momo-access-key>" --profile lingorise-dev --region ap-southeast-1
aws ssm put-parameter --name "/lingorise/dev/MOMO_SECRET_KEY" --type SecureString --value "<your-momo-secret-key>" --profile lingorise-dev --region ap-southeast-1

# AI provider keys
aws ssm put-parameter --name "/lingorise/dev/OPENROUTER_API_KEY" --type SecureString --value "<your-openrouter-key>" --profile lingorise-dev --region ap-southeast-1

# Content APIs
aws ssm put-parameter --name "/lingorise/dev/YOUTUBE_API_KEY" --type SecureString --value "<your-youtube-key>" --profile lingorise-dev --region ap-southeast-1
aws ssm put-parameter --name "/lingorise/dev/NEWS_API_KEY" --type SecureString --value "<your-newsapi-key>" --profile lingorise-dev --region ap-southeast-1
aws ssm put-parameter --name "/lingorise/dev/GNEWS_API_KEY" --type SecureString --value "<your-gnews-key>" --profile lingorise-dev --region ap-southeast-1
aws ssm put-parameter --name "/lingorise/dev/REDDIT_CLIENT_ID" --type SecureString --value "<your-reddit-client-id>" --profile lingorise-dev --region ap-southeast-1
aws ssm put-parameter --name "/lingorise/dev/REDDIT_CLIENT_SECRET" --type SecureString --value "<your-reddit-client-secret>" --profile lingorise-dev --region ap-southeast-1

# Database password (dùng ở Chương 4 bởi RDS)
aws ssm put-parameter --name "/lingorise/dev/DB_PASSWORD" --type SecureString --value "<your-strong-password>" --profile lingorise-dev --region ap-southeast-1
```

{{% notice tip %}}
Thêm `--overwrite` cho phép chạy lại bất kỳ lệnh nào để cập nhật giá trị tại chỗ. Lần đầu hãy để trống nó để bạn nhận được lỗi nếu tên tham số đã tồn tại — một cách phòng vệ rẻ tiền tránh vô tình ghi đè thứ gì đó.
{{% /notice %}}

#### Tạo mật khẩu DB

Đừng tự gõ mật khẩu master của cơ sở dữ liệu. Hãy tạo một chuỗi ngẫu nhiên mạnh ở máy cục bộ rồi dán vào tham số `DB_PASSWORD` bên trên.

bash:

```bash
openssl rand -base64 32
```

PowerShell:

```powershell
[Convert]::ToBase64String((1..32 | ForEach-Object { Get-Random -Max 256 } | ForEach-Object {[byte]$_}))
```

{{% notice warning %}}
**Lưu mật khẩu vừa tạo vào trình quản lý mật khẩu trước khi dán vào SSM.** Một tham số `SecureString` không thể xem lại nội dung gốc từ console — muốn đọc lại phải dùng `aws ssm get-parameter ... --with-decryption`, và ngay cả khi đó cũng chỉ được nếu danh tính IAM của bạn được phép giải mã. Nếu làm mất nó mà không có quyền `--with-decryption`, bạn sẽ phải đặt lại mật khẩu master của RDS để khôi phục. Chương 4 tái sử dụng đúng giá trị này khi dựng `DATABASE_URL`.
{{% /notice %}}

#### Kiểm tra

Liệt kê mọi tham số dưới root `/lingorise/dev/` và xác nhận đủ cả mười tham số. `describe-parameters` chỉ trả về metadata — không bao giờ trả về giá trị — nên an toàn để chạy và dán vào ticket.

```powershell
aws ssm describe-parameters --query "Parameters[?starts_with(Name,'/lingorise/dev/')].Name" --profile lingorise-dev --region ap-southeast-1
```

Bạn sẽ thấy chín bí mật cộng với `DB_PASSWORD` — tổng cộng mười tên. `DATABASE_URL` chỉ xuất hiện ở đây sau Chương 4. Với Parameter Store đã được gieo đầy đủ, bạn đã sẵn sàng dựng cơ sở dữ liệu mà `DATABASE_URL` sẽ trỏ tới.
