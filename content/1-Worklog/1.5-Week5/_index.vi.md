---
title: "Worklog Tuần 5"
date: 2026-05-18
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---
### Mục tiêu tuần 5:

* Củng cố kiến thức về IAM, Roles và các thực hành bảo mật tài khoản AWS.
* Hiểu cách áp dụng nguyên tắc **least privilege** cho user, service và EC2 instance.
* Thực hành rà soát quyền truy cập bằng cả AWS Console và AWS CLI.
* Chuẩn bị mô hình access control có thể tái sử dụng cho workshop về S3.

### Các công việc cần triển khai trong tuần này:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Ôn lại các khái niệm IAM gồm Users, Groups, Roles và Policies. So sánh managed policy và inline policy. | 18/05/2026 | 18/05/2026 | [AWS IAM Docs](https://docs.aws.amazon.com/iam/) |
| **2** | Kiểm tra thêm các hạng mục bảo mật tài khoản như MFA, password policy và access key rotation awareness. | 19/05/2026 | 19/05/2026 | [AWS Security Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) |
| **3** | Tạo IAM Role cho EC2 và gắn policy truy cập S3 ở phạm vi giới hạn. Kiểm tra role từ EC2 bằng AWS CLI. | 20/05/2026 | 20/05/2026 | [IAM Roles for Amazon EC2](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html) |
| **4** | Viết và kiểm tra một policy chỉ cho phép truy cập vào một S3 bucket cụ thể và một số action cần thiết. | 21/05/2026 | 21/05/2026 | [IAM JSON Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements.html) |
| **5** | Tìm hiểu thêm về governance trên AWS như AWS Organizations và Service Control Policies ở mức khái niệm. | 22/05/2026 | 22/05/2026 | [AWS Organizations](https://docs.aws.amazon.com/organizations/) |
| **6** | Tổng hợp bài học bảo mật trong tuần và ghi chú các lỗi thường gặp dẫn tới cấp quyền quá rộng. | 23/05/2026 | 23/05/2026 | Ghi chú cá nhân |

### Cách thực hiện chi tiết:

#### 1. Ôn lại mô hình cấp quyền trước khi gán permissions

Tôi xem lại cách AWS cấp quyền bằng cách so sánh:

* **IAM Users** cho con người sử dụng
* **IAM Groups** để gom nhóm quyền
* **IAM Roles** cho temporary delegated access
* **Policies** là ngôn ngữ mô tả quyền hạn

Điều này rất quan trọng vì các tuần tiếp theo không chỉ có user thao tác trên Console, mà còn có dịch vụ cần truy cập sang dịch vụ khác.

#### 2. Kết nối EC2 với AWS services mà không dùng hardcoded credentials

Để thực hành mô hình service-to-service access an toàn, tôi tạo **IAM Role** cho EC2 và gắn policy chỉ cho phép một số **S3 actions** trên đúng một bucket. Sau đó tôi attach role vào EC2 instance và kiểm tra truy cập từ chính EC2 bằng AWS CLI.

Luồng tích hợp an toàn ở đây là:

* **EC2 instance** -> assume **IAM Role**
* **IAM Role** -> có **S3 permissions** ở phạm vi hẹp
* **AWS CLI trên EC2** -> truy cập **S3** bằng temporary credentials

Mô hình này an toàn hơn nhiều so với việc lưu access key dài hạn trong hệ điều hành.

#### 3. Viết policy gắn với tài nguyên thật

Thay vì chỉ đọc IAM policy ở mức lý thuyết, tôi viết policy với:

* **Action** cụ thể
* **Resource ARN** cụ thể
* phạm vi quyền truy cập giới hạn

Sau đó tôi kiểm tra xem instance có thể list, read hoặc upload object tùy theo quyền được cấp. Bài thực hành này giúp tôi thấy rõ IAM gắn trực tiếp với cách bảo vệ storage.

#### 4. Liên hệ IAM với các use case workshop sau này

Chủ đề bảo mật của tuần này kết nối rất chặt với workshop về sau vì private S3 access sẽ không có ý nghĩa nếu permissions không đúng. Tôi ghi chú rằng dù:

* network path hoạt động đúng
* endpoint được cấu hình chính xác

request vẫn sẽ fail nếu **IAM Role** hoặc **S3 permissions** chưa đúng.

### Kết nối các dịch vụ AWS trong tuần này:

* **IAM User + Console/CLI:** Các thao tác quản trị được thực hiện qua user có kiểm soát.
* **EC2 + IAM Role:** Instance dùng role thay vì static access keys.
* **IAM Policy + S3:** Quyền được giới hạn theo bucket và action cụ thể.
* **Governance Concepts + Account Security:** Tư duy policy được mở rộng từ một user hay một service sang cấp account.

### Kết quả đạt được tuần 5:

* Hiểu rõ sự khác nhau thực tế giữa **Users**, **Groups**, **Roles** và **Policies**.
* Gắn thành công **IAM Role** vào EC2 và kiểm tra truy cập AWS service mà không dùng static credentials.
* Cải thiện khả năng viết policy khi gắn quyền với một S3 use case cụ thể.
* Xây dựng tư duy least-privilege rõ ràng hơn cho các thiết kế access control.
* Chuẩn bị được mô hình quyền cần thiết cho workshop về VPC Endpoint và S3 ở các tuần sau.

### Khó khăn gặp phải:

* Cú pháp IAM policy vẫn khá chi tiết và rất dễ sai nếu action hoặc ARN không khớp chính xác.
* Tôi cần test cẩn thận để phân biệt lỗi do IAM với lỗi do các yếu tố khác.

### Bài học rút ra và định hướng tiếp theo:

* Tích hợp dịch vụ an toàn trên AWS phụ thuộc vào cả thiết kế identity lẫn resource permissions.
* Sang tuần tiếp theo, tôi sẽ mở rộng phần này bằng cách tập trung vào **CloudWatch** và **CloudTrail** để quan sát hệ thống và phát hiện các vấn đề truy cập hoặc hiệu năng tốt hơn.
