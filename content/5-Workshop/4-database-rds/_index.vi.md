---
title : "Cơ sở dữ liệu — RDS PostgreSQL"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 4. </b> "
---

#### Tổng quan

LingoRise lưu mọi hồ sơ học viên, phiên thi, mục ngân hàng câu hỏi và bản ghi thanh toán trong **PostgreSQL**. Trong chương này, bạn sẽ khởi tạo một instance **Amazon RDS for PostgreSQL 16.4** được quản lý (`lingorise-dev-db`), giới hạn truy cập mạng chỉ tới máy trạm của chính bạn, đưa chuỗi kết nối vào **SSM Parameter Store**, và cuối cùng chạy bộ migration idempotent để dựng lên ~20+ bảng mà backend cần.

Toàn bộ chạy trong **ap-southeast-1** với profile CLI `lingorise-dev`. Các lệnh PowerShell dùng ký tự nối dòng backtick (`` ` ``); nếu bạn dùng bash/zsh, hãy thay backtick bằng backslash (`\`).

![Đang tạo RDS instance trên console](/images/Workshop-LingoRise/4-database-rds/rds-creating.png)

#### Tạo security group

RDS sẽ nằm trong **VPC mặc định** của tài khoản. Trước tiên hãy tra id của VPC đó, rồi tạo một security group riêng cho cơ sở dữ liệu.

PowerShell:

```powershell
$VpcId = aws ec2 describe-vpcs `
  --filters "Name=isDefault,Values=true" `
  --query "Vpcs[0].VpcId" `
  --output text `
  --profile lingorise-dev `
  --region ap-southeast-1

$SgId = aws ec2 create-security-group `
  --group-name lingorise-dev-db `
  --description "LingoRise dev database access" `
  --vpc-id $VpcId `
  --query "GroupId" `
  --output text `
  --profile lingorise-dev `
  --region ap-southeast-1
```

bash:

```bash
VpcId=$(aws ec2 describe-vpcs \
  --filters "Name=isDefault,Values=true" \
  --query "Vpcs[0].VpcId" \
  --output text \
  --profile lingorise-dev \
  --region ap-southeast-1)

SgId=$(aws ec2 create-security-group \
  --group-name lingorise-dev-db \
  --description "LingoRise dev database access" \
  --vpc-id "$VpcId" \
  --query "GroupId" \
  --output text \
  --profile lingorise-dev \
  --region ap-southeast-1)
```

Bây giờ mở cổng TCP **5432** chỉ cho máy trạm của bạn. Tra địa chỉ IP công cộng hiện tại và cấp quyền đúng `/32` đó.

PowerShell:

```powershell
$MyIp = (Invoke-RestMethod -Uri "https://checkip.amazonaws.com").Trim()

aws ec2 authorize-security-group-ingress `
  --group-id $SgId `
  --protocol tcp `
  --port 5432 `
  --cidr "$MyIp/32" `
  --profile lingorise-dev `
  --region ap-southeast-1
```

bash:

```bash
MyIp=$(curl -s https://checkip.amazonaws.com)

aws ec2 authorize-security-group-ingress \
  --group-id "$SgId" \
  --protocol tcp \
  --port 5432 \
  --cidr "$MyIp/32" \
  --profile lingorise-dev \
  --region ap-southeast-1
```

{{% notice note %}}
Địa chỉ IP tại nhà và văn phòng hay thay đổi. Nếu về sau migration không kết nối được, hãy chạy lại lệnh `authorize-security-group-ingress` với IP công cộng mới — quy tắc `/32` cũ có thể không còn khớp.
{{% /notice %}}

#### Tạo DB instance

Khởi tạo một instance `db.t4g.micro` nhỏ, tiết kiệm chi phí, chạy **PostgreSQL 16.4**, với master user `lingorise`. Chọn một mật khẩu master mạnh và giữ lại để dùng cho chuỗi kết nối ngay sau đây.

PowerShell:

```powershell
aws rds create-db-instance `
  --db-instance-identifier lingorise-dev-db `
  --db-instance-class db.t4g.micro `
  --engine postgres `
  --engine-version 16.4 `
  --master-username lingorise `
  --master-user-password "<your-strong-password>" `
  --allocated-storage 20 `
  --db-name lingorise `
  --vpc-security-group-ids $SgId `
  --publicly-accessible `
  --backup-retention-period 7 `
  --deletion-protection `
  --profile lingorise-dev `
  --region ap-southeast-1
```

bash:

```bash
aws rds create-db-instance \
  --db-instance-identifier lingorise-dev-db \
  --db-instance-class db.t4g.micro \
  --engine postgres \
  --engine-version 16.4 \
  --master-username lingorise \
  --master-user-password "<your-strong-password>" \
  --allocated-storage 20 \
  --db-name lingorise \
  --vpc-security-group-ids "$SgId" \
  --publicly-accessible \
  --backup-retention-period 7 \
  --deletion-protection \
  --profile lingorise-dev \
  --region ap-southeast-1
```

{{% notice note %}}
`--deletion-protection` được bật có chủ đích để không ai vô tình xóa mất cơ sở dữ liệu. Đánh đổi là khi dọn dẹp bạn cần thêm một bước: phải tắt deletion protection trước khi có thể xóa instance. Chúng ta sẽ làm đúng điều đó trong **Chương 9 (Dọn dẹp)**.
{{% /notice %}}

#### Lấy endpoint

Quá trình khởi tạo mất vài phút. Hãy chờ tới khi instance báo trạng thái **available**, rồi đọc lại endpoint kết nối của nó.

PowerShell:

```powershell
aws rds wait db-instance-available `
  --db-instance-identifier lingorise-dev-db `
  --profile lingorise-dev `
  --region ap-southeast-1

$DbHost = aws rds describe-db-instances `
  --db-instance-identifier lingorise-dev-db `
  --query "DBInstances[0].Endpoint.Address" `
  --output text `
  --profile lingorise-dev `
  --region ap-southeast-1
```

bash:

```bash
aws rds wait db-instance-available \
  --db-instance-identifier lingorise-dev-db \
  --profile lingorise-dev \
  --region ap-southeast-1

DbHost=$(aws rds describe-db-instances \
  --db-instance-identifier lingorise-dev-db \
  --query "DBInstances[0].Endpoint.Address" \
  --output text \
  --profile lingorise-dev \
  --region ap-southeast-1)
```

![RDS instance ở trạng thái available cùng endpoint của nó](/images/Workshop-LingoRise/4-database-rds/rds-available.png)

#### Lưu DATABASE_URL vào SSM

Ghép chuỗi kết nối PostgreSQL đầy đủ và đưa vào **SSM Parameter Store** dưới dạng `SecureString` tại gốc `/lingorise/dev/`. Cả backend lẫn migration runner đều đọc `DATABASE_URL` từ đây.

PowerShell:

```powershell
$DbUrl = "postgresql://lingorise:<your-strong-password>@$($DbHost):5432/lingorise?sslmode=require"

aws ssm put-parameter `
  --name "/lingorise/dev/DATABASE_URL" `
  --type SecureString `
  --value $DbUrl `
  --overwrite `
  --profile lingorise-dev `
  --region ap-southeast-1
```

bash:

```bash
DbUrl="postgresql://lingorise:<your-strong-password>@${DbHost}:5432/lingorise?sslmode=require"

aws ssm put-parameter \
  --name "/lingorise/dev/DATABASE_URL" \
  --type SecureString \
  --value "$DbUrl" \
  --overwrite \
  --profile lingorise-dev \
  --region ap-southeast-1
```

{{% notice warning %}}
`sslmode=require` là **bắt buộc**. RDS PostgreSQL buộc mã hóa TLS trên đường truyền, và backend LingoRise từ chối mở kết nối không mã hóa. Bỏ hậu tố `?sslmode=require` thì cả migration lẫn ứng dụng đang chạy đều sẽ không kết nối được.
{{% /notice %}}

#### Chạy migration

Migration runner là idempotent — nó theo dõi những migration đã áp dụng trong bảng `_migrations`, nên chạy lại là an toàn và chỉ thực thi phần mới. Vào thư mục backend, export cùng chuỗi kết nối ở máy cục bộ, cài dependency, và chạy bộ migration.

PowerShell:

```powershell
cd lingorise-backend

$env:DATABASE_URL = $DbUrl

npm install
npm run migrate
```

bash:

```bash
cd lingorise-backend

export DATABASE_URL="$DbUrl"

npm install
npm run migrate
```

![Kết quả migration runner liệt kê các migration đã áp dụng](/images/Workshop-LingoRise/4-database-rds/migrations-output.png)

#### Kiểm tra

Kết nối bằng `psql` và liệt kê các bảng. Một schema khỏe mạnh sẽ hiện **~20+ bảng** — users, exam sessions, question banks, payments, testimonials, và nhiều hơn nữa.

```bash
psql "$DbUrl" -c "\dt"
```

Bạn sẽ thấy danh sách bảng tương tự:

```text
             List of relations
 Schema |       Name        | Type  |   Owner
--------+-------------------+-------+-----------
 public | _migrations       | table | lingorise
 public | exam_sessions     | table | lingorise
 public | payments          | table | lingorise
 public | question_bank     | table | lingorise
 public | testimonials      | table | lingorise
 public | users             | table | lingorise
 ...
(20+ rows)
```

Nếu bảng `_migrations` xuất hiện và số dòng từ 20 trở lên, tầng cơ sở dữ liệu của bạn đã sẵn sàng. Tiếp theo, trong **Chương 5**, chúng ta sẽ triển khai backend giao tiếp với nó.
