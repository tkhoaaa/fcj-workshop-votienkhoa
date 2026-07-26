---
title: "Worklog Tuần 2"
date: 2026-04-27
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu tuần 2:

* Hiểu sâu từng thành phần của VPC: subnets, route tables, Internet Gateway, NAT Gateway và Network ACLs.
* Tự tay xây một custom VPC thay vì dùng default VPC, với public subnet và private subnet trên hai Availability Zones.
* So sánh Amazon RDS với Amazon DynamoDB và chọn dịch vụ phù hợp cho data model của LingoRise.
* Học AWS IAM qua thực hành: users, groups, roles và policy document theo nguyên tắc least privilege.

### Các công việc cần triển khai trong tuần này:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Tìm hiểu sâu các thành phần VPC: cách hoạch định CIDR, subnets, route tables, Internet Gateway, NAT Gateway và Network ACLs. | 27/04/2026 | 27/04/2026 | [Amazon VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html) |
| **2** | Xây custom VPC `10.0.0.0/16` tại `ap-southeast-1` với hai public subnet và hai private subnet, gắn Internet Gateway và NAT Gateway rồi chỉnh lại route table association. | 28/04/2026 | 28/04/2026 | [NAT Gateway Docs](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html) |
| **3** | So sánh Security Groups và Network ACLs bằng cách test traffic được phép và bị chặn để xác nhận hành vi stateful so với stateless. | 29/04/2026 | 29/04/2026 | [Security in Amazon VPC](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Security.html) |
| **4** | Nghiên cứu Amazon RDS for PostgreSQL: DB subnet group, parameter group, automated backup, sau đó đối chiếu với partition key, sort key và on-demand capacity của Amazon DynamoDB. | 30/04/2026 | 30/04/2026 | [Amazon RDS](https://aws.amazon.com/rds/) |
| **5** | Tìm hiểu AWS IAM users, groups, roles và policies; đọc từng dòng một policy JSON và áp dụng least privilege cho account lab. | 01/05/2026 | 01/05/2026 | [AWS IAM Docs](https://docs.aws.amazon.com/iam/) |

### Cách thực hiện chi tiết:

#### 1. Hoạch định dải CIDR trước khi tạo subnet đầu tiên

Tôi bắt đầu từ bảng địa chỉ IP chứ không mở console trước. **VPC** dùng `10.0.0.0/16`, sau đó tôi chia thành bốn subnet `/24` để layout dễ đọc:

* `10.0.1.0/24` public tại `ap-southeast-1a`, `10.0.2.0/24` public tại `ap-southeast-1b`
* `10.0.11.0/24` private tại `ap-southeast-1a`, `10.0.12.0/24` private tại `ap-southeast-1b`

Việc dùng hai Availability Zones không phải tùy chọn. **Amazon RDS** yêu cầu DB subnet group có subnet ở tối thiểu hai AZ, nên tầng mạng phải multi-AZ trước khi database tồn tại. Việc chừa riêng dải `10.0.1x.0/24` cho private subnet cũng để dành chỗ mở rộng mà không phải đánh số lại.

#### 2. Đi dây routing với Internet Gateway, NAT Gateway và route table riêng

Tôi gắn **Internet Gateway** vào VPC và thêm route `0.0.0.0/0` trỏ tới IGW trong public route table. Một subnet chỉ trở thành public nhờ route đó, không phải nhờ cái tên, và đây chính là điểm trước giờ tôi hiểu sai.

Với private subnet, tôi đặt **NAT Gateway** trong public subnet kèm một Elastic IP, sau đó trỏ default route của private route table vào NAT Gateway. Outbound cài package chạy được, còn inbound thì không. Đường đi của traffic là:

* **Private subnet** -> private route table -> **NAT Gateway** -> **Internet Gateway** -> Internet

Tiếp đó tôi so sánh **Security Groups** với **Network ACLs** bằng cách lần lượt chặn port 22 ở từng lớp. Security Group là stateful, chỉ cần allow inbound SSH là traffic phản hồi tự động đi ra được. Network ACL là stateless, nên khi tôi deny dải ephemeral port `1024-65535` ở outbound rule thì session SSH đứt ngay dù inbound vẫn allow. Bài test đó giúp tôi nhớ sự khác biệt tốt hơn mọi sơ đồ.

#### 3. Chọn giữa Amazon RDS và Amazon DynamoDB cho LingoRise

Tôi đọc cả hai dịch vụ với question bank của LingoRise trong đầu. **Amazon DynamoDB** cho độ trễ đọc một chữ số millisecond, on-demand capacity không cần sizing instance, và data model phụ thuộc hoàn toàn vào partition key cùng sort key. **Amazon RDS** cho một relational engine được quản lý, kèm DB subnet group, parameter group, automated backup và maintenance window.

Các truy vấn của LingoRise không khớp với access pattern kiểu key-value. Exam session join sang questions, questions join sang passages và assets, còn review queue của admin thì filter theo nhiều cột cùng lúc. Metadata câu hỏi là dữ liệu bán cấu trúc, PostgreSQL xử lý được bằng `JSONB` mà vẫn cho phép join bằng SQL. Thay đổi schema được ship dưới dạng idempotent migration theo dõi trong bảng `_migrations`, và điều đó chỉ hợp lý với một relational engine.

Vì vậy dự án chốt dùng **Amazon RDS for PostgreSQL** trên `db.t4g.micro` với 20 GB storage, đặt trong private subnet ở phần việc thứ hai và không bao giờ mở public endpoint. DynamoDB vẫn là lựa chọn tốt cho tra cứu theo key thuần, nhưng LingoRise hiện chưa cần.

#### 4. Đọc IAM như một tập policy document, không phải các ô tick trên console

Tôi tạo group `fcj-developers`, gắn permission ở đó rồi thêm IAM user của mình vào group để không permission nào bị gán trực tiếp cho một user. Sau đó tôi tạo một **role** và thấy rõ khác biệt trong thực tế: user có credential dài hạn, còn role được assume và trả về temporary credentials, đúng thứ mà một Lambda function sẽ dùng về sau thay cho access key.

Phần quan trọng là đọc policy JSON chứ không tin bản tóm tắt của console:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["ssm:GetParameter", "ssm:GetParameters"],
    "Resource": "arn:aws:ssm:ap-southeast-1:*:parameter/lingorise/dev/*"
  }]
}
```

Mỗi thành phần đều có nhiệm vụ: `Action` là API call, `Resource` giới hạn phạm vi về đúng một parameter path, và dấu wildcard nằm ở cuối path chứ không phủ toàn bộ dịch vụ. Tôi cũng so sánh identity-based policy, loại gắn vào user, group hoặc role, với resource-based policy như S3 bucket policy, loại khai báo `Principal` và nằm ngay trên resource. Sau đó tôi kiểm tra lại permission thực tế bằng:

```bash
aws iam list-attached-group-policies --group-name fcj-developers
aws sts get-caller-identity
```

### Kết nối các dịch vụ AWS trong tuần này:

* **VPC + Subnets + Availability Zones:** Dải địa chỉ được chia thành public và private subnet trên hai AZ để các dịch vụ sau này được đặt theo mức độ public/private.
* **Route Tables + Internet Gateway:** Chính route `0.0.0.0/0` trỏ tới IGW mới làm một subnet trở thành public.
* **Private Subnets + NAT Gateway:** Workload private có outbound-only Internet access để cài package mà không hề có đường inbound.
* **Security Groups + Network ACLs:** Lớp lọc stateful ở cấp instance được xếp dưới lớp lọc stateless ở cấp subnet.
* **Amazon RDS + VPC:** DB subnet group ràng database vào private subnet, đó là lý do phải thiết kế network trước.
* **IAM + AWS CLI + SSM Parameter Store:** Một group policy giới hạn ở `/lingorise/dev/*` cho phép CLI chỉ đọc đúng những parameter mà stage đó cần.

### Kết quả đạt được tuần 2:

* Xây trọn một custom **VPC** từ bảng CIDR viết sẵn, gồm bốn subnet trên hai Availability Zones.
* Chứng minh bằng thực nghiệm rằng một subnet là public chỉ nhờ route table entry, không nhờ tên hay tag.
* Minh họa được stateful so với stateless bằng cách làm đứt session SSH qua outbound rule của **Network ACL** trong khi **Security Group** vẫn allow.
* Lý giải được vì sao LingoRise chọn **Amazon RDS for PostgreSQL** thay vì **Amazon DynamoDB** bằng lý do cụ thể: relational join, metadata `JSONB` và SQL migration.
* Áp dụng least privilege trong **IAM** qua một group policy giới hạn về đúng một Parameter Store path, và đọc policy JSON thay vì đọc bản tóm tắt console.

### Khó khăn gặp phải:

* NAT Gateway phải nằm trong public subnet nhưng lại phục vụ private subnet, điều này khá ngược cho tới khi tôi lần theo đường đi của packet qua cả hai route table.
* Rule của Network ACL được duyệt theo thứ tự số và cần entry outbound tương ứng cho return traffic, nên rule deny đầu tiên của tôi chặn nhiều hơn dự tính.
* Việc chọn giữa RDS và DynamoDB không phải câu hỏi về performance mà về data model, và ban đầu tôi so sánh chúng theo tiêu chí sai.

### Bài học rút ra và định hướng tiếp theo:

* Networking và identity là điều kiện tiên quyết, không phải bước làm sau. DB subnet group và IAM role đều phụ thuộc vào quyết định đưa ra trước khi có workload nào.
* Một policy document là thứ có thể đọc được. Khi đã hiểu `Effect`, `Action` và `Resource`, lỗi permission không còn là chuyện đoán mò.
* Sang tuần tiếp theo, tôi sẽ đưa VPC này vào việc thật: dựng web server **Nginx** trên **Amazon Linux** trong public subnet, dùng **S3 Event Notifications** trigger một **Lambda** function, và tạo **AMI** từ instance đã cấu hình.
