---
title: "Worklog Tuần 12"
date: 2026-07-06
weight: 12
chapter: false
pre: " <b> 1.12. </b> "
---
### Mục tiêu tuần 12:

* Hoàn thiện internship report và nhìn lại toàn bộ hành trình học tập trong 12 tuần.
* Tự đánh giá sự phát triển về kỹ năng kỹ thuật, kỹ năng mềm và mức độ sẵn sàng nghề nghiệp.
* Dọn dẹp tài nguyên lab còn lại và chuẩn bị cho phần trình bày hoặc nộp báo cáo cuối kỳ.

### Các công việc cần triển khai trong tuần này:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Rà soát toàn bộ worklog theo tuần để bảo đảm tính nhất quán về ngày tháng, chủ đề và tiến trình học tập. | 06/07/2026 | 06/07/2026 | Hồ sơ worklog |
| **2** | Hoàn thiện phần self-evaluation, tập trung vào technical skills, communication, teamwork và problem-solving. | 07/07/2026 | 07/07/2026 | Ghi chú tự đánh giá |
| **3** | Hoàn thành phần kết luận của internship report và tóm tắt các bài học quan trọng nhất rút ra được. | 08/07/2026 | 08/07/2026 | Bản nháp báo cáo |
| **4** | Kiểm tra định dạng, screenshots, links và chất lượng nội dung trên toàn bộ report site. | 09/07/2026 | 09/07/2026 | Hugo site preview |
| **5** | Dọn dẹp hoặc xác minh các tài nguyên lab còn lại để tránh phát sinh sử dụng và chi phí không cần thiết. | 10/07/2026 | 10/07/2026 | AWS Console / CLI |
| **6** | Chuẩn bị phần tổng kết cuối cùng cho buổi trình bày hoặc nộp báo cáo, bao gồm cả định hướng học tập tiếp theo. | 11/07/2026 | 11/07/2026 | Ghi chú presentation cuối |

### Cách thực hiện chi tiết:

#### 1. Xem lại internship như một hành trình kiến trúc có liên kết

Thay vì coi từng tuần là nội dung riêng lẻ, tôi xem toàn bộ internship như một tiến trình kỹ thuật nối tiếp:

* thiết lập tài khoản an toàn
* thiết kế networking
* triển khai compute
* sử dụng storage
* IAM và observability
* private và hybrid S3 connectivity
* policy-based access control
* documentation và reporting

Nhờ vậy báo cáo cuối kỳ trở nên đầy đủ hơn và phản ánh đúng bản chất của một engineering learning journey.

#### 2. Kiểm tra tính nhất quán kỹ thuật giữa các phần

Tôi rà soát xem những mối quan hệ giữa các dịch vụ AWS được mô tả trong báo cáo có nhất quán hay không, ví dụ:

* EC2 đặt trong VPC như thế nào
* IAM Role được dùng thay cho static credentials ra sao
* S3 access qua Gateway hoặc Interface Endpoint thế nào
* Route 53 tham gia private DNS resolution ra sao
* endpoint policy giới hạn quyền truy cập như thế nào

Bước này cần thiết vì khi mỗi phần được viết ở thời điểm khác nhau, báo cáo rất dễ lệch nếu không có final pass.

#### 3. Dọn tài nguyên như một phần của responsible cloud practice

Tôi xem lại các tài nguyên có thể tiếp tục phát sinh phí hoặc gây rối môi trường, chẳng hạn:

* EC2 instances
* NAT Gateway
* Interface Endpoints
* S3 resources dùng để test

Nhờ đó phần cleanup trở thành một phần của kết quả học tập chứ không chỉ là thao tác phụ cuối cùng.

#### 4. Xác định định hướng học tiếp theo

Sau khi nhìn lại toàn bộ 12 tuần, tôi xác định các mảng kỹ thuật muốn học sâu hơn:

* cloud networking
* cloud security
* hybrid connectivity
* automation và Infrastructure as Code

Phần reflection này giúp kết nối kết quả internship với kế hoạch phát triển bản thân sau đó, thay vì kết thúc chỉ bằng một báo cáo hoàn chỉnh.

### Kết nối các dịch vụ AWS trong tuần này:

* **Full AWS Learning Stack:** Báo cáo cuối kết nối identity, networking, compute, storage, monitoring và endpoint-based access control thành một mạch thống nhất.
* **Documentation + Infrastructure:** Báo cáo được dùng như công cụ để kiểm tra xem các tích hợp kỹ thuật đã được giải thích nhất quán chưa.
* **AWS Console/CLI + Resource Cleanup:** Các thao tác cleanup được thực hiện dựa trên kiến trúc đã dựng từ các tuần trước.
* **Reflection + Technical Practice:** Bài học rút ra luôn gắn với các tương tác AWS service thực tế chứ không chỉ là tổng kết trừu tượng.

### Kết quả đạt được tuần 12:

* Hoàn thiện internship report với technical narrative rõ hơn và quan hệ dịch vụ mạch lạc hơn.
* Nhìn lại toàn bộ chương trình 12 tuần như một AWS learning path có liên kết.
* Nâng cao khả năng tự đánh giá cả về kỹ thuật lẫn giao tiếp chuyên nghiệp.
* Hoàn tất việc rà soát tài nguyên và cleanup như một phần của responsible cloud operations.
* Xây dựng được định hướng học tiếp rõ ràng hơn về AWS solution design, security và automation.

### Khó khăn gặp phải:

* Việc tóm tắt một hành trình dài mà vẫn giữ đủ chiều sâu kỹ thuật là một thử thách lớn.
* Quá trình final review đòi hỏi kiểm tra kỹ tính nhất quán giữa nhiều phần và nhiều dịch vụ AWS khác nhau.

### Bài học rút ra và định hướng tiếp theo:

* Một internship report tốt không chỉ cho thấy đã làm gì mà còn phải thể hiện được hệ thống được kết nối như thế nào, vì sao chọn thiết kế đó và bản thân đã học được gì.
* Sau internship này, tôi sẽ tiếp tục đào sâu AWS theo hướng private networking, cloud security và automation tooling.
