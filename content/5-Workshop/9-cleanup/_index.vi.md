---
title : "Dọn dẹp tài nguyên"
date : 2024-01-01
weight : 9
chapter : false
pre : " <b> 9. </b> "
---

{{% notice warning %}}
Chương này sẽ **xóa vĩnh viễn mọi tài nguyên và toàn bộ dữ liệu** bạn đã tạo trong workshop — cơ sở dữ liệu RDS, bucket tài nguyên S3 cùng các object bên trong, các tham số SSM, và toàn bộ stack `lingorise-dev`. Thao tác này không thể hoàn tác. Trước khi chạy bất kỳ câu lệnh nào bên dưới, hãy chắc chắn bạn đã sao lưu những gì cần giữ lại (một snapshot RDS cuối cùng, tải các tài nguyên về, xuất giá trị các tham số).
{{% /notice %}}

#### Tổng quan

Sau khi đã triển khai, kiểm thử và vận hành LingoRise, bước cuối cùng là dỡ bỏ toàn bộ để bạn ngừng phát sinh chi phí. AWS tính phí cho instance RDS, dung lượng S3, và các tài nguyên khác chừng nào chúng còn tồn tại, bất kể có ai đang dùng hay không — nên một lần dọn dẹp sạch sẽ chính là ranh giới giữa một tài khoản dev yên ắng và một hóa đơn bất ngờ.

Thứ tự rất quan trọng. CloudFormation từ chối xóa một stack đang sở hữu các tài nguyên phụ thuộc mà nó không thể gỡ sạch — ví dụ một bucket S3 vẫn còn object bên trong, hay một instance RDS đang bật deletion protection. Vì vậy chúng ta làm từ ngoài vào trong: dọn rỗng bucket trước, sau đó xóa stack, rồi gỡ các tài nguyên độc lập bạn đã tạo thủ công ở các chương trước (instance RDS, các tham số SSM, và security group). Làm theo năm bước sau đúng thứ tự thì mọi thứ sẽ được dỡ bỏ mà không gặp lỗi phụ thuộc.

#### Các bước dỡ bỏ

**1. Dọn rỗng bucket tài nguyên**

CloudFormation (và `sam delete`) không thể xóa một bucket S3 chưa rỗng, nên hãy dọn sạch nó trước. Câu lệnh này xóa mọi object nằm dưới bucket tài nguyên của LingoRise.

```powershell
aws s3 rm s3://lingorise-assets-dev-<accountId> --recursive `
  --region ap-southeast-1 --profile lingorise-dev
```

```bash
aws s3 rm s3://lingorise-assets-dev-<accountId> --recursive \
  --region ap-southeast-1 --profile lingorise-dev
```

**2. Xóa stack SAM/CloudFormation**

Khi bucket đã rỗng, hãy dỡ bỏ toàn bộ stack backend — các hàm Lambda, API Gateway, Cognito, các IAM role, và cả chính bucket — chỉ bằng một câu lệnh.

```powershell
sam delete --stack-name lingorise-dev `
  --region ap-southeast-1 --profile lingorise-dev
```

```bash
sam delete --stack-name lingorise-dev \
  --region ap-southeast-1 --profile lingorise-dev
```

SAM sẽ yêu cầu bạn xác nhận trước khi xóa stack và các artifact triển khai. Trả lời `y` cho cả hai câu hỏi.

![Hộp thoại xác nhận sam delete](/images/Workshop-LingoRise/9-cleanup/sam-delete-confirm.png)

**3. Xóa instance RDS**

Instance RDS (`lingorise-dev-db`) được tạo thủ công ở Chương 4, nên nó không thuộc stack và phải được xóa riêng.

{{% notice warning %}}
Ở **Chương 4** bạn đã tạo cơ sở dữ liệu với `--deletion-protection` được bật để không ai xóa nhầm. Lớp bảo vệ này chặn `delete-db-instance`, nên bạn phải **tắt nó trước**. Chạy câu lệnh `modify-db-instance --no-deletion-protection` bên dưới và chờ nó áp dụng xong, rồi mới chạy lệnh xóa. Bỏ qua bước này sẽ khiến lệnh xóa thất bại với lỗi `InvalidParameterCombination`.
{{% /notice %}}

Trước tiên, tắt deletion protection:

```powershell
aws rds modify-db-instance `
  --db-instance-identifier lingorise-dev-db `
  --no-deletion-protection `
  --apply-immediately `
  --region ap-southeast-1 --profile lingorise-dev
```

```bash
aws rds modify-db-instance \
  --db-instance-identifier lingorise-dev-db \
  --no-deletion-protection \
  --apply-immediately \
  --region ap-southeast-1 --profile lingorise-dev
```

Sau đó xóa instance. `--skip-final-snapshot` xóa nó ngay lập tức mà không tạo bản sao lưu cuối cùng — phù hợp với một cơ sở dữ liệu `dev` dùng một lần, nhưng hãy bỏ cờ này (và truyền `--final-db-snapshot-identifier`) nếu bạn muốn giữ lại một snapshot.

```powershell
aws rds delete-db-instance `
  --db-instance-identifier lingorise-dev-db `
  --skip-final-snapshot `
  --region ap-southeast-1 --profile lingorise-dev
```

```bash
aws rds delete-db-instance \
  --db-instance-identifier lingorise-dev-db \
  --skip-final-snapshot \
  --region ap-southeast-1 --profile lingorise-dev
```

**4. Xóa các tham số SSM**

Gỡ mọi tham số nằm dưới đường dẫn `/lingorise/dev/`. Vòng lặp này liệt kê tên các tham số trước, rồi xóa lần lượt từng cái.

```powershell
$names = aws ssm get-parameters-by-path `
  --path "/lingorise/dev/" --recursive `
  --query "Parameters[].Name" --output text `
  --region ap-southeast-1 --profile lingorise-dev
foreach ($n in ($names -split "\s+")) {
  if ($n) {
    aws ssm delete-parameter --name $n `
      --region ap-southeast-1 --profile lingorise-dev
  }
}
```

```bash
for n in $(aws ssm get-parameters-by-path \
  --path "/lingorise/dev/" --recursive \
  --query "Parameters[].Name" --output text \
  --region ap-southeast-1 --profile lingorise-dev); do
  aws ssm delete-parameter --name "$n" \
    --region ap-southeast-1 --profile lingorise-dev
done
```

**5. Xóa security group**

Security group `lingorise-dev-db` từ Chương 4 là tài nguyên độc lập cuối cùng. Nó chỉ có thể bị xóa khi instance RDS từng dùng nó đã biến mất hoàn toàn, nên hãy chờ lệnh xóa ở Bước 3 hoàn tất rồi mới chạy lệnh này.

```powershell
$SgId = aws ec2 describe-security-groups `
  --filters "Name=group-name,Values=lingorise-dev-db" `
  --query "SecurityGroups[0].GroupId" --output text `
  --region ap-southeast-1 --profile lingorise-dev
aws ec2 delete-security-group --group-id $SgId `
  --region ap-southeast-1 --profile lingorise-dev
```

```bash
SgId=$(aws ec2 describe-security-groups \
  --filters "Name=group-name,Values=lingorise-dev-db" \
  --query "SecurityGroups[0].GroupId" --output text \
  --region ap-southeast-1 --profile lingorise-dev)
aws ec2 delete-security-group --group-id "$SgId" \
  --region ap-southeast-1 --profile lingorise-dev
```

{{% notice note %}}
Nếu `delete-security-group` trả về lỗi `DependencyViolation`, nghĩa là instance RDS vẫn đang tách khỏi security group. Hãy chờ thêm một phút, xác nhận instance đã biến mất (xem phần Kiểm tra bên dưới), rồi chạy lại câu lệnh.
{{% /notice %}}

#### Kiểm tra mọi thứ đã sạch

Chạy ba phép kiểm tra sau. Mỗi câu lệnh nên trả về rỗng (hoặc không còn tài nguyên LingoRise), xác nhận không còn gì để bị tính phí nữa.

Stack không còn xuất hiện dưới trạng thái đang hoạt động:

```powershell
aws cloudformation list-stacks `
  --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE `
  --query "StackSummaries[?StackName=='lingorise-dev']" `
  --region ap-southeast-1 --profile lingorise-dev
```

```bash
aws cloudformation list-stacks \
  --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE \
  --query "StackSummaries[?StackName=='lingorise-dev']" \
  --region ap-southeast-1 --profile lingorise-dev
```

Instance RDS đã biến mất (câu lệnh này trả về lỗi `DBInstanceNotFound` khi việc xóa hoàn tất):

```powershell
aws rds describe-db-instances `
  --db-instance-identifier lingorise-dev-db `
  --region ap-southeast-1 --profile lingorise-dev
```

```bash
aws rds describe-db-instances \
  --db-instance-identifier lingorise-dev-db \
  --region ap-southeast-1 --profile lingorise-dev
```

Bucket tài nguyên không còn liệt kê được nữa (lệnh xóa stack đã gỡ nó sau khi Bước 1 dọn rỗng):

```powershell
aws s3 ls s3://lingorise-assets-dev-<accountId> `
  --region ap-southeast-1 --profile lingorise-dev
```

```bash
aws s3 ls s3://lingorise-assets-dev-<accountId> \
  --region ap-southeast-1 --profile lingorise-dev
```

![Tài khoản đã sạch sau khi dỡ bỏ](/images/Workshop-LingoRise/9-cleanup/empty-account.png)

{{% notice tip %}}
Khi cả ba phép kiểm tra đều trả về rỗng, tài khoản `dev` của bạn đã sạch và chi phí ngừng phát sinh. Đó là điểm kết thúc của workshop — chúc mừng bạn đã triển khai, vận hành, và dỡ bỏ LingoRise trọn vẹn từ đầu đến cuối trên AWS.
{{% /notice %}}
