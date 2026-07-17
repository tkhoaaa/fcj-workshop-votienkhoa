---
title: "Worklog Tuần 9"
date: 2026-06-15
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---
### Mục tiêu tuần 9:

* Tìm hiểu cách giới hạn truy cập dịch vụ bằng **VPC Endpoint Policies**.
* Củng cố kỹ năng troubleshooting cho mô hình private Amazon S3 connectivity.
* Tổng hợp phần kiến thức kỹ thuật cốt lõi của workshop.

### Các công việc cần triển khai trong tuần này:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Ôn lại cách IAM policies, S3 bucket policies và VPC Endpoint Policies kết hợp với nhau. | 15/06/2026 | 15/06/2026 | [Endpoint Policy Docs](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-access.html) |
| **2** | Cấu hình S3 VPC Endpoint Policy để giới hạn truy cập vào bucket hoặc phạm vi action cụ thể. | 16/06/2026 | 16/06/2026 | [Workshop Policy Section](https://workshop-sample.fcjuni.com/5-workshop/5.5-policy/) |
| **3** | Kiểm tra cả trường hợp truy cập thành công và bị từ chối bằng AWS CLI từ lab EC2 instance. | 17/06/2026 | 17/06/2026 | AWS CLI |
| **4** | Tạo checklist troubleshooting bao gồm route tables, security groups, DNS, IAM, bucket policy và endpoint policy. | 18/06/2026 | 18/06/2026 | Ghi chú cá nhân |
| **5** | Viết nháp một technical note hoặc blog outline tóm tắt workshop và các bài học bảo mật. | 19/06/2026 | 19/06/2026 | Bản nháp cá nhân |
| **6** | Rà soát và sắp xếp screenshot, policy snippets và command outputs để phục vụ báo cáo. | 20/06/2026 | 20/06/2026 | Ghi chú chuẩn bị báo cáo |

### Cách thực hiện chi tiết:

#### 1. Hiểu cơ chế kiểm soát quyền ở nhiều lớp

Tuần này tập trung vào việc ngay cả khi kết nối mạng đã đúng, truy cập dịch vụ vẫn có thể bị giới hạn có chủ đích. Tôi xem lại cách:

* **IAM policies**
* **S3 bucket policies**
* **VPC Endpoint Policies**

cùng ảnh hưởng đến một request nhưng theo các vai trò khác nhau.

Thay vì xem access control là một luật đơn giản, tôi tiếp cận nó như một mô hình đánh giá nhiều lớp.

#### 2. Giới hạn truy cập ngay tại endpoint

Tôi cấu hình **VPC Endpoint Policy** để chỉ cho phép truy cập đến một S3 bucket cụ thể hoặc một số action nhất định. Điều này có nghĩa là dù instance có quyền rộng hơn ở nơi khác, chính endpoint path vẫn có thể đóng vai trò như một lớp lọc bổ sung.

Luồng tích hợp nâng cao lúc này là:

* **EC2 với IAM Role**
* **private network path qua endpoint**
* **endpoint policy check**
* **S3 bucket policy check**
* **quyết định truy cập cuối cùng**

#### 3. Kiểm tra allowed và denied bằng AWS CLI

Tôi dùng AWS CLI từ EC2 instance để kiểm tra:

* các lệnh được phép
* các lệnh bị từ chối
* các failure case đúng như kỳ vọng

Điều này rất có ích vì nó biến kiến thức về policy từ lý thuyết thành hành vi có thể quan sát được.

#### 4. Xây dựng quy trình troubleshooting có thể lặp lại

Đến cuối tuần, tôi đã có một checklist bắt đầu từ:

* network path
* DNS behavior
* endpoint status
* IAM role permissions
* S3 bucket policy
* endpoint policy

Nhờ vậy việc troubleshooting trở nên có phương pháp hơn và giảm bớt sự rối khi request thất bại.

### Kết nối các dịch vụ AWS trong tuần này:

* **IAM Role + EC2:** Danh tính của workload quyết định instance được phép yêu cầu gì.
* **EC2 + VPC Endpoint:** Mọi lưu lượng tới S3 đi theo private path có kiểm soát.
* **VPC Endpoint Policy + S3 Bucket Policy:** Truy cập được lọc ở cả network-entry layer và storage-resource layer.
* **AWS CLI + Policy Testing:** Dòng lệnh được dùng để kiểm tra end-to-end security behavior.
* **Workshop Architecture + Governance:** Kiến trúc đang hoạt động được bổ sung thêm lớp security control.

### Kết quả đạt được tuần 9:

* Hiểu cách **VPC Endpoint Policies** thêm một lớp kiểm soát cho private service access.
* Có trải nghiệm thực tế với cả trường hợp thành công lẫn bị từ chối khi thao tác với S3.
* Xây dựng mô hình tư duy rõ ràng hơn về policy evaluation giữa nhiều dịch vụ AWS.
* Tạo được phương pháp troubleshooting có thể tái sử dụng ở các bài lab khác.
* Làm sâu thêm phần bảo mật của workshop thay vì chỉ dừng ở connectivity.

### Khó khăn gặp phải:

* Đôi khi khá khó xác định lớp policy nào là nguyên nhân gây lỗi.
* Việc khớp chính xác ARN và action đòi hỏi cẩn thận để tránh ra kết quả ngoài ý muốn.

### Bài học rút ra và định hướng tiếp theo:

* Một thiết kế AWS an toàn không chỉ là cho phép kết nối mà còn là giới hạn truy cập ở đúng control points.
* Sang tuần tiếp theo, tôi sẽ tập trung vào **documentation**, **automation** và **cost awareness** để workshop dễ bảo trì và trình bày chuyên nghiệp hơn.
