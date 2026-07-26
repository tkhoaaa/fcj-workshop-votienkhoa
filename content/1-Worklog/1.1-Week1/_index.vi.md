---
title: "Worklog Tuần 1"
date: 2026-04-20
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---
### Mục tiêu tuần 1:

* Hoàn tất onboarding cùng team First Cloud AI Journey (FCAJ) và thiết lập tài khoản AWS Free Tier an toàn cho toàn bộ kỳ thực tập.
* Tìm hiểu kiến thức nền tảng về **Amazon EC2** và **Amazon VPC**, hoàn thành các bài tập thực hành cơ bản.
* Tìm hiểu **AWS Elastic Disaster Recovery (DRS)** ở mức khái niệm, gồm cơ chế replication, RPO và RTO.
* Thử cấu hình deploy bằng **AWS SAM** kết hợp **AWS Systems Manager Parameter Store** để lưu khóa bảo mật, và tạo **AMI** phục vụ backup cùng repeated deployment.

### Các công việc cần triển khai trong tuần này:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Tham gia team FCAJ, tìm hiểu quy định và quy trình thực tập, tạo tài khoản AWS Free Tier, bật MFA cho Root account và tạo IAM administrator user. | 20/04/2026 | 20/04/2026 | [FCAJ Guide](https://workshop-sample.fcjuni.com/) |
| **2** | Cài đặt và cấu hình AWS CLI cho `ap-southeast-1`, thiết lập AWS Budget alert, tìm hiểu kiến thức cơ bản về Amazon EC2: instance types, AMI, key pair, Security Group, EBS. | 21/04/2026 | 21/04/2026 | [AWS CLI Guide](https://docs.aws.amazon.com/cli/) |
| **3** | Tìm hiểu kiến thức nền tảng về Amazon VPC và làm bài lab cơ bản: khởi tạo Amazon Linux 2023 instance trong VPC tự tạo, kiểm tra kết nối bằng SSH và EC2 Instance Connect. | 22/04/2026 | 22/04/2026 | [Amazon VPC](https://aws.amazon.com/vpc/) |
| **4** | Nghiên cứu AWS Elastic Disaster Recovery: replication agent, staging area subnet, point-in-time recovery và cách RPO, RTO quyết định thiết kế khôi phục. | 23/04/2026 | 23/04/2026 | [AWS DRS Docs](https://docs.aws.amazon.com/drs/) |
| **5** | Thử deploy một stack nhỏ gồm Lambda và API Gateway bằng AWS SAM, lưu secret vào SSM Parameter Store dạng SecureString và tạo AMI từ EC2 instance để backup. | 24/04/2026 | 24/04/2026 | [AWS SAM Docs](https://docs.aws.amazon.com/serverless-application-model/) |

### Cách thực hiện chi tiết:

#### 1. Onboarding và bảo mật tài khoản AWS Free Tier

Tuần đầu bắt đầu bằng phần onboarding của FCAJ: quy định thực tập, quy trình báo cáo hằng tuần và cách dự án capstone sẽ được đánh giá. Bước thực hành đầu tiên không phải là dựng server mà là bảo mật tài khoản. Tôi tạo tài khoản AWS Free Tier, bật **MFA** cho Root user, sau đó tạo một **IAM** administrator user để làm việc hằng ngày, còn Root user chỉ dùng cho các thao tác ở cấp tài khoản.

Tiếp theo tôi cấu hình **AWS CLI** trên máy và đặt `ap-southeast-1` làm region mặc định, để công việc trên console và trên dòng lệnh luôn trỏ về cùng một nơi.

```bash
aws configure --profile fcaj
aws sts get-caller-identity --profile fcaj
aws budgets describe-budgets --account-id <account-id>
```

Tôi cũng tạo một **AWS Budgets** alert với ngưỡng chi phí nhỏ theo tháng. Việc kiểm soát chi phí là cần thiết vì các tuần sau sẽ dùng thêm nhiều managed service chỉ được Free Tier bao một phần.

#### 2. Kiến thức nền tảng về Amazon EC2 và Amazon VPC

Khi làm các bài lab cơ bản, tôi khởi tạo một Amazon Linux 2023 instance và lần lượt xem lại từng thành phần mà nó phụ thuộc:

* **AMI** cung cấp image cho máy ảo
* **Key Pair** dùng để xác thực khi SSH
* **Security Group** lọc inbound và outbound traffic ở mức instance
* **Amazon EBS** cung cấp root volume gắn kèm
* **Region và Availability Zone** quyết định vị trí vật lý của instance

Sau đó tôi chuyển instance vào một **Amazon VPC** tự tạo thay vì dùng default VPC. Tôi tạo VPC với CIDR `/16`, một public subnet, một internet gateway và cấu hình **route table** đẩy `0.0.0.0/0` qua gateway đó. Việc kiểm tra kết nối bằng cả **EC2 Instance Connect** và SSH từ laptop cho thấy rất rõ chuỗi phụ thuộc: nếu route table thiếu một entry thì instance vẫn hiện trên console nhưng không thể truy cập qua mạng.

Bài học chính là EC2 không bao giờ đứng một mình. IAM quyết định ai được khởi tạo, VPC quyết định có kết nối được hay không, còn EBS giữ dữ liệu.

#### 3. Tìm hiểu AWS Elastic Disaster Recovery

Tôi tìm hiểu **AWS Elastic Disaster Recovery (DRS)** ở mức khái niệm chứ chưa triển khai trọn vẹn, vì một bài drill DRS đầy đủ cần source server và dung lượng replication vượt quá giới hạn Free Tier. Tôi ghi lại các thành phần chính:

* **Replication agent** cài trên source server, stream thay đổi ở mức block về AWS.
* **Staging area subnet** chứa các replication server và EBS volume chi phí thấp, không phải instance kích thước thật.
* **Point-in-time recovery** cho phép launch recovery instance từ một thời điểm được chọn, không chỉ trạng thái mới nhất.
* **RPO** cho biết mức mất dữ liệu có thể chấp nhận, **RTO** cho biết thời gian khôi phục cho phép. Hai con số này định hình toàn bộ thiết kế.

Điểm nối lại với các bài lab phía trước là recovery instance của DRS sẽ chạy trong VPC và subnet đã chuẩn bị sẵn, cùng với Security Group và IAM role có từ trước. Disaster recovery vì vậy là bài toán về networking và identity không kém gì về storage.

#### 4. Thử deploy bằng SAM, Parameter Store và AMI backup

Cuối tuần tôi thử một lượt deploy bằng **AWS SAM**: một template tối giản với một hàm **AWS Lambda** đứng sau **Amazon API Gateway**. Lệnh `sam deploy --guided` tạo ra CloudFormation stack, S3 artifact bucket và execution role, cho tôi thấy SAM chỉ là một lớp mỏng phía trên CloudFormation chứ không phải một hệ thống deploy riêng biệt.

Để không đặt secret trực tiếp trong template, tôi lưu nó vào **AWS Systems Manager Parameter Store** dạng SecureString và tham chiếu tại thời điểm deploy.

```bash
aws ssm put-parameter --name "/lingorise/dev/demo/api-key" \
  --value "<secret>" --type SecureString --region ap-southeast-1
sam deploy --guided --region ap-southeast-1
```

Template resolve giá trị bằng `{{resolve:ssm:/lingorise/${Stage}/demo/api-key}}`, nhờ đó secret không bao giờ xuất hiện trong source control. Lần thử này về sau trở thành đường deploy chính cho dự án capstone **LingoRise**.

Cuối cùng tôi tạo một **AMI** từ EC2 instance đã cấu hình xong. Image này giữ lại OS, các package đã cài và EBS snapshot, nên tôi có thể launch lại một instance y hệt thay vì làm lại toàn bộ thao tác setup. Đó vừa là bản backup vừa là baseline cho repeated deployment.

### Kết nối các dịch vụ AWS trong tuần này:

* **IAM + AWS CLI:** CLI xác thực bằng access key của IAM administrator user thay vì credential của Root.
* **EC2 + VPC + Security Group:** Khả năng truy cập instance đến từ route table của subnet và internet gateway, còn Security Group lọc traffic tại biên instance.
* **EC2 + EBS + AMI:** AMI đóng gói cấu hình instance cùng EBS snapshot để launch lại nhiều lần.
* **AWS DRS + VPC + EBS:** DRS replicate vào các staging EBS volume và launch recovery instance vào VPC, subnet đã chuẩn bị trước.
* **AWS SAM + CloudFormation + Lambda + API Gateway:** SAM mở rộng thành một CloudFormation stack tạo đồng thời function, REST API và execution role.
* **SSM Parameter Store + CloudFormation:** Stack resolve SecureString parameter tại thời điểm deploy nên không có secret nào nằm trong template.

### Kết quả đạt được tuần 1:

* Hoàn tất onboarding FCAJ và bảo mật tài khoản AWS Free Tier bằng **MFA**, IAM administrator user và Budgets alert.
* Cấu hình **AWS CLI** cho `ap-southeast-1` và xác minh danh tính bằng `aws sts get-caller-identity`.
* Hoàn thành các bài lab cơ bản: khởi tạo Amazon Linux 2023 instance trong **VPC** tự tạo và kiểm tra kết nối bằng SSH cùng EC2 Instance Connect.
* Ghi lại mô hình **AWS Elastic Disaster Recovery** gồm replication agent, staging subnet, point-in-time recovery và các đánh đổi RPO/RTO.
* Thử deploy thành công bằng **AWS SAM** với secret được lưu trong **SSM Parameter Store** dạng SecureString.
* Tạo **AMI** từ instance đang hoạt động để phục vụ backup và repeated deployment.

### Khó khăn gặp phải:

* Rất nhiều khái niệm xuất hiện cùng lúc. IAM, VPC routing, thuật ngữ của DRS và cách SAM đóng gói đều có hệ từ vựng riêng, tôi phải mất thời gian để thấy chúng ghép với nhau thế nào.
* DRS không thể triển khai đầy đủ trong giới hạn Free Tier, nên tôi ghi rõ phần này ở mức nghiên cứu và tài liệu hóa chứ chưa chạy một bài failover thật.
* Lần `sam deploy` đầu tiên fail vì thiếu quyền IAM, cho đến khi tôi hiểu SAM cần quyền tạo CloudFormation stack, role và artifact bucket, không chỉ tạo Lambda function.

### Bài học rút ra và định hướng tiếp theo:

* Bảo mật tài khoản và hàng rào chi phí phải đi trước hạ tầng, không phải sau. Mọi bài lab về sau đều thừa hưởng nền tảng này.
* Infrastructure as code kết hợp Parameter Store giúp cấu hình có thể tái lập và secret không nằm trong repository, đúng nhu cầu của dự án capstone.
* Sang tuần tiếp theo, tôi sẽ đi sâu vào các thành phần của **Amazon VPC** gồm subnet, route table, internet gateway, NAT Gateway và NACL, sau đó là các dịch vụ database của AWS với **Amazon RDS**, **Amazon DynamoDB** và tìm hiểu chi tiết hơn về **AWS IAM**.
