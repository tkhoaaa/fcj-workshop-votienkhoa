---
title: "Worklog Tuần 10"
date: 2026-06-22
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---
### Mục tiêu tuần 10:

* Tăng hiệu quả làm việc bằng cách tài liệu hóa command và tự động hóa các thao tác lặp lại.
* Rà soát mức sử dụng tài nguyên và xây dựng ý thức kiểm soát chi phí khi làm lab.
* Chuẩn bị đầy đủ tư liệu kỹ thuật cho báo cáo internship cuối kỳ.

### Các công việc cần triển khai trong tuần này:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Tổng hợp và phân loại các AWS CLI commands đã sử dụng trong suốt workshop. | 22/06/2026 | 22/06/2026 | Ghi chú CLI cá nhân |
| **2** | Tạo một helper script đơn giản hoặc checklist command để hỗ trợ kiểm tra môi trường hay cleanup. | 23/06/2026 | 23/06/2026 | AWS CLI Docs |
| **3** | Xem lại AWS Billing, Budgets và Cost Explorer để hiểu resource usage trong môi trường lab. | 24/06/2026 | 24/06/2026 | [AWS Billing Docs](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/) |
| **4** | Sắp xếp lại screenshots, diagrams và ghi chú từng bước để phần workshop có câu chuyện rõ ràng hơn. | 25/06/2026 | 25/06/2026 | Ghi chú chuẩn bị báo cáo |
| **5** | Viết outline có cấu trúc cho các phần của report: objective, architecture, implementation, result và lessons learned. | 26/06/2026 | 26/06/2026 | Outline cá nhân |
| **6** | Tự rà soát tài liệu để bổ sung những phần còn thiếu về screenshot, command output hoặc diễn giải kỹ thuật. | 27/06/2026 | 27/06/2026 | Self-review |

### Cách thực hiện chi tiết:

#### 1. Biến các bước thủ công lặp lại thành tài sản vận hành có thể tái sử dụng

Ở thời điểm này của internship, có rất nhiều thao tác AWS đã được lặp lại nhiều lần. Thay vì chỉ nhớ bằng trí nhớ, tôi tổng hợp các **AWS CLI commands** quan trọng thành ghi chú và helper script có thể tái sử dụng.

Các lệnh này bao gồm:

* kiểm tra S3 bucket
* xác minh EC2
* kiểm tra IAM identity
* test endpoint
* chuẩn bị cleanup

Việc này kết nối phần thực thi kỹ thuật với tính nhất quán trong vận hành.

#### 2. Dùng CLI-based verification để hỗ trợ tính lặp lại của workshop

Tôi tạo một verification flow nhỏ để sau mỗi lần đổi cấu hình có thể nhanh chóng kiểm tra:

* mình đang xác thực bằng tài khoản nào
* EC2 hoặc endpoint mong muốn có tồn tại không
* S3 access có đúng như kỳ vọng không

Điều này kết nối:

* **AWS CLI**
* **IAM identity**
* **EC2**
* **S3**
* **endpoint-based architecture**

và giúp rút ngắn thời gian re-test.

#### 3. Rà soát chi phí của các tài nguyên lab

Tôi xem lại **AWS Budgets**, **Billing** và **Cost Explorer** để xác định những tài nguyên nào có thể tiếp tục phát sinh chi phí nếu để quên, đặc biệt là:

* NAT Gateway
* Interface Endpoints
* EC2 instances
* storage resources

Qua đó tôi nhận ra rằng thiết kế cloud không chỉ cần đúng về mặt kỹ thuật mà còn phải có kỷ luật về chi phí.

#### 4. Tổ chức evidence để phục vụ báo cáo

Tôi gom diagrams, screenshots, policy snippets và CLI outputs theo từng chủ đề để có thể dùng trực tiếp cho báo cáo kỹ thuật. Việc này biến kết quả lab thành report-ready material và giúp giải thích mối quan hệ giữa các dịch vụ dễ hơn.

### Kết nối các dịch vụ AWS trong tuần này:

* **AWS CLI + IAM:** Xác minh identity giúp chạy lệnh an toàn và đúng ngữ cảnh.
* **AWS CLI + S3/EC2/VPC Endpoints:** Trạng thái dịch vụ và hành vi truy cập được kiểm tra bằng lập trình.
* **Billing + Deployed Resources:** Phân tích chi phí gắn trực tiếp với các quyết định triển khai hạ tầng.
* **Documentation + Workshop Architecture:** Bằng chứng kỹ thuật được tổ chức xoay quanh các tích hợp dịch vụ thật chứ không phải screenshot rời rạc.

### Kết quả đạt được tuần 10:

* Xây dựng được bộ CLI reference và verification steps có thể tái sử dụng cho workshop.
* Cải thiện tính lặp lại và giảm thời gian test nhờ documentation và một số bước automation đơn giản.
* Tăng ý thức về chi phí đối với các tài nguyên cloud tạm thời.
* Tổ chức lại tài liệu kỹ thuật thành cấu trúc phù hợp hơn cho báo cáo chính thức.
* Chuẩn bị tốt hơn cho giai đoạn viết báo cáo và chia sẻ kết quả.

### Khó khăn gặp phải:

* Công việc tài liệu hóa cần nhiều kỷ luật vì mỗi screenshot và command đều phải có ngữ cảnh đủ rõ.
* Việc kiểm soát chi phí đòi hỏi hiểu tài nguyên nào có thể phát sinh charge kể cả khi tưởng như “để yên”.

### Bài học rút ra và định hướng tiếp theo:

* Một cloud project tốt không chỉ chạy được mà còn phải tái hiện được, tài liệu hóa tốt và có ý thức tối ưu chi phí.
* Sang tuần tiếp theo, tôi sẽ tập trung vào **knowledge sharing** và **report drafting**, dùng chính các tài liệu này để trình bày công việc rõ ràng hơn.
