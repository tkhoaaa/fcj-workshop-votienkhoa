---
title: "Worklog Tuần 6"
date: 2026-05-25
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---
### Mục tiêu tuần 6:

* Nâng cao khả năng quan sát hệ thống bằng Amazon CloudWatch và AWS CloudTrail.
* Biết cách theo dõi EC2, tạo alarm và xem lại lịch sử API trên AWS.
* Rèn tư duy vận hành để phát hiện và xử lý sự cố sớm hơn.

### Các công việc cần triển khai trong tuần này:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Tìm hiểu các thành phần chính của Amazon CloudWatch: Metrics, Logs, Dashboards và Alarms. | 25/05/2026 | 25/05/2026 | [Amazon CloudWatch Docs](https://docs.aws.amazon.com/cloudwatch/) |
| **2** | Tạo CloudWatch Alarm cho EC2 như CPU utilization và instance status checks. | 26/05/2026 | 26/05/2026 | [Using Amazon CloudWatch Alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html) |
| **3** | Cài đặt hoặc rà soát cấu hình CloudWatch Agent để đẩy thêm logs và metrics từ EC2. | 27/05/2026 | 27/05/2026 | [CloudWatch Agent](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html) |
| **4** | Tìm hiểu AWS CloudTrail và xem event history của các thao tác đã thực hiện trong những bài lab trước đó. | 28/05/2026 | 28/05/2026 | [AWS CloudTrail Docs](https://docs.aws.amazon.com/cloudtrail/) |
| **5** | Xây dựng một checklist theo dõi cho workload EC2 gồm health, access và performance signals. | 29/05/2026 | 29/05/2026 | Ghi chú cá nhân |
| **6** | Viết một ghi chú ngắn về cách logs và alarms giúp rút ngắn thời gian troubleshooting. | 30/05/2026 | 30/05/2026 | Ghi chú cá nhân |

### Cách thực hiện chi tiết:

#### 1. Thêm monitoring vào workload EC2 từ các tuần trước

Tôi tái sử dụng môi trường EC2 đã tạo từ các tuần trước và kết nối nó với **CloudWatch**. Thay vì coi server như một “hộp đen”, tôi theo dõi:

* CPU utilization
* status checks
* các tín hiệu health cơ bản

Đây là bước đầu tiên để biến một EC2 đơn giản thành một workload có thể quản trị được.

#### 2. Kết nối logs và metrics từ EC2 vào CloudWatch

Thông qua việc cài hoặc rà soát **CloudWatch Agent**, tôi hiểu cách dữ liệu ở lớp hệ điều hành có thể được đẩy từ **EC2** lên **CloudWatch Logs** và **CloudWatch Metrics**. Luồng vận hành hữu ích khi đó là:

* **EC2 operating system**
* **CloudWatch Agent**
* **CloudWatch Logs / Metrics**
* **Alarms và dashboards**

Tích hợp này rất quan trọng vì các bài lab networking sau sẽ dễ troubleshooting hơn rất nhiều nếu có logs và health data được tập trung.

#### 3. Tạo alarms để tăng operational awareness

Tôi tạo **CloudWatch Alarms** dựa trên một số EC2 metrics để hệ thống có thể cảnh báo hoặc ít nhất phản ánh khi tài nguyên có hành vi bất thường. Dù là môi trường lab, điều này giúp tôi hình thành tư duy rằng hạ tầng cloud nên được quan sát chủ động chứ không chỉ kiểm tra thủ công.

#### 4. Dùng CloudTrail để truy vết hành động quản trị

Song song với CloudWatch, tôi xem **CloudTrail** event history để biết những AWS API calls nào đã được thực hiện trong quá trình tạo và test lab. Việc này giúp tôi phân biệt rõ:

* **CloudWatch** dùng cho performance và vận hành
* **CloudTrail** dùng cho audit và lịch sử thao tác

Khi kết hợp lại, hai dịch vụ này tạo nên một lớp visibility khá mạnh cho toàn bộ hạ tầng đã triển khai.

### Kết nối các dịch vụ AWS trong tuần này:

* **EC2 + CloudWatch Metrics:** Hiệu năng của instance trở nên đo lường được.
* **EC2 + CloudWatch Agent + CloudWatch Logs:** Dữ liệu ở mức OS được tập trung lại.
* **CloudWatch + Alarms:** Dữ liệu monitoring được dùng cho phát hiện sớm vấn đề.
* **AWS Account Activity + CloudTrail:** Các hành động quản trị và API calls trở nên có thể truy vết.
* **Operations + Existing Infrastructure:** Quan sát hệ thống được thêm lên trên nền hạ tầng đã có từ các tuần trước.

### Kết quả đạt được tuần 6:

* Có hiểu biết thực tế hơn về **CloudWatch** và **CloudTrail** trong bối cảnh vận hành.
* Tăng khả năng quan sát tình trạng instance, hiệu năng và lịch sử hoạt động.
* Hiểu cách logs tập trung và alarms giúp rút ngắn thời gian troubleshooting.
* Kết nối phần triển khai hạ tầng với các thực hành observability cơ bản.
* Hình thành thói quen monitoring có giá trị cho các bài lab hybrid connectivity ở các tuần sau.

### Khó khăn gặp phải:

* Tôi cần thời gian để xác định metric nào thực sự có ý nghĩa thay vì chỉ dùng những gì có sẵn mặc định.
* Việc phân biệt vai trò của CloudWatch và CloudTrail rõ ràng hơn chỉ khi tôi thực hành lặp lại nhiều lần.

### Bài học rút ra và định hướng tiếp theo:

* Vận hành cloud hiệu quả phụ thuộc vào khả năng quan sát, truy vết và giải thích hành vi hệ thống.
* Sang tuần tiếp theo, tôi sẽ áp dụng tư duy visibility này vào workshop **Gateway VPC Endpoint** cho Amazon S3.
