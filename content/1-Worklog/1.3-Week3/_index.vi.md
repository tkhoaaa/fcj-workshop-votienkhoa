---
title: "Worklog Tuần 3"
date: 2026-05-04
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu tuần 3:

* Tái sử dụng custom VPC đã xây ở tuần 2 để chạy workload thật thay vì tạo network resources mới.
* Cài đặt và cấu hình **Nginx** trên Amazon Linux 2023 làm web server public.
* Xây luồng event-driven đầu tiên trên AWS bằng cách nối **Amazon S3** Event Notifications với **AWS Lambda**.
* Tạo **Amazon Machine Image (AMI)** phục vụ backup và repeated deployment.

### Các công việc cần triển khai trong tuần này:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Tái sử dụng custom VPC của tuần 2, launch một Amazon Linux 2023 instance vào public subnet và xem lại route table cùng Security Group đã có. | 04/05/2026 | 04/05/2026 | [Amazon VPC Docs](https://docs.aws.amazon.com/vpc/) |
| **2** | Cài **Nginx** bằng `dnf`, enable service qua `systemctl`, publish trang test và mở inbound rule HTTP trong Security Group. | 05/05/2026 | 05/05/2026 | [Nginx on Amazon Linux](https://docs.aws.amazon.com/linux/al2023/ug/what-is-amazon-linux.html) |
| **3** | Tạo S3 bucket cho asset upload và một Lambda function, sau đó cấu hình Event Notification `s3:ObjectCreated:*` để kích hoạt function. | 06/05/2026 | 06/05/2026 | [S3 Event Notifications](https://docs.aws.amazon.com/AmazonS3/latest/userguide/NotificationHowTo.html) |
| **4** | Kiểm tra luồng event-driven end to end bằng cách upload file và đọc record invocation trong **Amazon CloudWatch Logs**. | 07/05/2026 | 07/05/2026 | [Using AWS Lambda with S3](https://docs.aws.amazon.com/lambda/latest/dg/with-s3.html) |
| **5** | Tạo **AMI** từ web server đã cấu hình, rồi launch một instance mới từ image đó để chứng minh repeatable deployment. | 08/05/2026 | 08/05/2026 | [Create an AMI from an EC2 instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/creating-an-ami-ebs.html) |

### Cách thực hiện chi tiết:

#### 1. Tái sử dụng VPC của tuần 2 thay vì dựng network mới

Quyết định đầu tiên của tuần này là không tạo VPC mới. Tôi launch **Amazon EC2** instance vào public subnet của custom **Amazon VPC** từ tuần 2, nơi đã có Internet Gateway, public route table và private subnet dành riêng cho tài nguyên dữ liệu.

Đây chính là lý do tuần trước phải thiết kế network cẩn thận. Vì subnet layout và routing đã đúng, phần việc network còn lại trong tuần này chỉ là một Security Group rule. Kiến trúc LingoRise cũng theo nguyên tắc đó: RDS for PostgreSQL nằm trong private subnets, còn lớp compute mới đặt ở nơi traffic có thể tới được.

Tôi xác nhận lại vị trí instance trước khi cài bất cứ thứ gì:

* Instance nằm trong **public subnet** và có auto-assign public IPv4
* Route `0.0.0.0/0` trỏ về **Internet Gateway**
* Security Group giới hạn SSH chỉ từ IP của tôi

#### 2. Cài và cấu hình Nginx trên Amazon Linux 2023

Tôi SSH vào instance và cài **Nginx** từ repository của Amazon Linux 2023, sau đó enable service để nó tự chạy lại sau khi reboot.

```bash
sudo dnf install -y nginx
sudo systemctl enable --now nginx
echo "LingoRise web node - week 3" | sudo tee /usr/share/nginx/html/index.html
curl -I http://localhost
```

`curl` từ trong instance trả về `200 OK` nhưng trình duyệt vẫn timeout, điều này cho thấy rất rõ sự tách biệt giữa cấu hình ở lớp OS và lớp network. Trang chỉ truy cập được sau khi tôi thêm inbound rule TCP `80` từ `0.0.0.0/0` trong **Security Group**. Sau đó tôi kiểm tra lại từ laptop bằng `curl -I http://<public-ip>` để kết quả không bị ảnh hưởng bởi cache của browser.

#### 3. Nối S3 Event Notifications với Lambda

Nửa sau của tuần chuyển từ một server tự quản sang mô hình event-driven không cần server. Tôi tạo một S3 bucket cho asset upload và một function **AWS Lambda**, rồi thêm Event Notification trên bucket với sự kiện `s3:ObjectCreated:*` trỏ tới function đó.

Có hai chi tiết quan trọng:

* S3 không dùng execution role để gọi Lambda. Console tự thêm một **resource-based policy** trên function cho phép `s3.amazonaws.com` invoke, giới hạn theo source account và bucket ARN.
* Payload của event là một danh sách. Handler phải duyệt `event.Records` và đọc `record.s3.object.key`, vì một notification có thể chứa nhiều record.

Đây đúng là hình dạng của asset pipeline trong LingoRise. Khi admin upload ảnh câu hỏi hoặc file DOCX scan lên S3, chính hành động upload nên đẩy job OCR xuống downstream thay vì cần thêm một lời gọi API riêng. Tuần này là bản chạy được tối giản của ý tưởng đó.

#### 4. Xác nhận invocation và chụp server thành AMI

Tôi upload vài file bằng `aws s3 cp` và kiểm tra **Amazon CloudWatch Logs** ở log group của function. Mỗi lần upload đều tạo log entry riêng kèm bucket name và object key, xác nhận trigger chạy theo từng object chứ không gộp theo lô upload.

Cuối cùng tôi tạo **AMI** từ EC2 instance đã cấu hình. Image không phải một file đơn lẻ: AWS chụp **Amazon EBS** snapshots của các volume đang attach và đăng ký chúng phía sau AMI ID. Để chứng minh image dùng được thật, tôi launch một instance thứ hai từ image đó vào cùng subnet và thấy Nginx đã chạy sẵn, phục vụ đúng trang test mà không cần cài lại gì. Việc này khép lại vòng từ một server cấu hình tay thành một artifact deployment có thể lặp lại.

### Kết nối các dịch vụ AWS trong tuần này:

* **EC2 + VPC/Subnet:** Web server được launch vào public subnet của tuần 2, tái sử dụng routing có sẵn thay vì tạo network resources mới.
* **EC2 + Security Group:** Khả năng truy cập HTTP phụ thuộc vào inbound rule port 80, tách biệt với trạng thái service Nginx bên trong OS.
* **S3 + Lambda:** Event Notification `s3:ObjectCreated:*` tự động invoke function mỗi lần có upload.
* **S3 + Lambda resource policy:** S3 được cấp quyền `lambda:InvokeFunction` qua resource-based policy giới hạn theo bucket ARN và source account.
* **Lambda + CloudWatch Logs:** Mỗi invocation ghi object key vào log stream, đây là cách thực tế nhất để xác nhận trigger hoạt động.
* **EC2 + EBS + AMI:** AMI chụp EBS snapshots của các volume và trở thành template cho một instance thứ hai giống hệt.

### Kết quả đạt được tuần 3:

* Chạy được workload thật trong custom VPC đã dựng trước đó mà không tạo thêm network component nào.
* Triển khai Nginx web server hoạt động trên Amazon Linux 2023 và kiểm tra từ bên ngoài bằng `curl`.
* Xây thành công pipeline event-driven, trong đó upload lên S3 tự động kích hoạt Lambda function.
* Hiểu rằng quyền từ S3 sang Lambda đi qua resource-based policy trên function, không phải execution role.
* Tạo AMI và launch instance mới từ image đó, xác nhận repeatable deployment.
* Ánh xạ mô hình S3 sang Lambda vào thiết kế asset và OCR job queue của LingoRise.

### Khó khăn gặp phải:

* Trang web không truy cập được từ bên ngoài dù `localhost` vẫn trả về đúng, vấn đề nằm ở Security Group chứ không phải Nginx.
* Những lần invoke Lambda đầu tiên thất bại âm thầm vì handler giả định chỉ có một object thay vì duyệt `event.Records`.
* Bước tạo AMI mất nhiều thời gian hơn dự kiến vì phải chờ EBS snapshots của mọi volume đang attach hoàn tất.

### Bài học rút ra và định hướng tiếp theo:

* Đầu tư thiết kế network sớm sẽ có lợi về sau. Tái sử dụng VPC của tuần 2 biến cả phần setup networking thành một thay đổi duy nhất trên một rule.
* Tích hợp theo hướng event-driven giúp bỏ được glue code. Chính hành động upload trở thành trigger, đáng tin hơn việc yêu cầu client gọi thêm một endpoint.
* Sang tuần tiếp theo, tôi sẽ chuyển từ provisioning thủ công sang **DevOps on AWS**, làm việc với **AWS CodeCommit**, **AWS CodeBuild** và **AWS CodePipeline** để dựng một CI/CD pipeline, cùng với dashboard và alarm trên **Amazon CloudWatch** để theo dõi.
