---
title: "Worklog Tuần 7"
date: 2026-06-01
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---
### Mục tiêu tuần 7:

* Chuẩn bị môi trường workshop cho bài toán private access tới Amazon S3.
* Hiểu kiến trúc và use case của **Gateway VPC Endpoint**.
* Thực hành cấu hình network routing để truy cập S3 mà không đi qua Public Internet.

### Các công việc cần triển khai trong tuần này:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Đọc workshop overview và vẽ lại kiến trúc mục tiêu cho private S3 connectivity. | 01/06/2026 | 01/06/2026 | [Workshop Overview](https://workshop-sample.fcjuni.com/5-workshop/5.1-workshop-overview/) |
| **2** | Rà soát môi trường lab gồm VPC, subnets, route tables, EC2 instances và các tài nguyên hỗ trợ. | 02/06/2026 | 02/06/2026 | Ghi chú workshop |
| **3** | Tạo **Gateway VPC Endpoint** cho Amazon S3 trong workshop VPC. | 03/06/2026 | 03/06/2026 | [Gateway Endpoint Lab](https://workshop-sample.fcjuni.com/5-workshop/5.3-lotushacks-utmorpho/5.3.1-from-zero-to-idea/) |
| **4** | Kiểm tra cập nhật route table và xác nhận traffic S3 từ VPC đi qua endpoint. | 04/06/2026 | 04/06/2026 | [VPC Endpoints Docs](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-s3.html) |
| **5** | Test truy cập bucket từ EC2 instance và lưu lại kết quả bằng screenshot cũng như output CLI. | 05/06/2026 | 05/06/2026 | [Test Gateway Endpoint](https://workshop-sample.fcjuni.com/5-workshop/5.3-lotushacks-utmorpho/5.3.2-building-under-pressure/) |
| **6** | Tổng hợp khi nào nên dùng Gateway Endpoint và những giới hạn của nó so với Interface Endpoint. | 06/06/2026 | 06/06/2026 | Ghi chú cá nhân |

### Cách thực hiện chi tiết:

#### 1. Bắt đầu từ kiến trúc workshop thay vì từng tài nguyên rời rạc

Tuần này tôi chuyển từ việc học từng dịch vụ riêng lẻ sang hiểu một kiến trúc có liên kết hoàn chỉnh. Mục tiêu không chỉ là “tạo một endpoint”, mà là cho phép **private S3 access từ workload trong VPC**.

Tôi rà lại toàn bộ đường đi:

* **EC2 instance trong VPC**
* **Route table gắn với subnet**
* **Gateway VPC Endpoint**
* **Amazon S3**

Nhờ đó tôi hiểu rằng cấu hình endpoint thực chất là một bài toán tích hợp networking.

#### 2. Tạo Gateway VPC Endpoint cho S3

Tôi mở VPC console và tạo **Gateway Endpoint** cho Amazon S3. Trong quá trình này, tôi chọn:

* đúng **VPC**
* đúng **route table**
* endpoint policy mặc định để test trước

Bước này thay đổi logic routing sao cho các request tới S3 từ subnet liên quan đi qua endpoint thay vì public Internet path.

#### 3. Test truy cập S3 từ EC2 trong VPC

Từ EC2 instance trong VPC, tôi dùng AWS CLI để truy cập thử vào S3 bucket. Mục tiêu là xác minh đồng thời:

* hành vi network path
* hành vi IAM permissions

Điều này cho thấy mối liên kết rất rõ giữa các tuần:

* **Tuần 2:** VPC và route tables
* **Tuần 5:** IAM roles và permissions
* **Tuần 7:** S3 private access qua Gateway Endpoint

#### 4. Quan sát rằng networking và permissions phải cùng đúng

Bài test chỉ thành công khi nhiều lớp cùng thỏa:

* EC2 phải có IAM access hợp lệ
* subnet phải gắn đúng route table
* Gateway Endpoint phải attach đúng
* quyền truy cập tới S3 target phải được cho phép

Đây là bài học quan trọng vì rất nhiều tích hợp AWS luôn là bài toán nhiều lớp chứ không phải chỉ một service.

### Kết nối các dịch vụ AWS trong tuần này:

* **EC2 + IAM Role:** Instance xác thực tới AWS services bằng temporary credentials.
* **EC2 + VPC/Subnet/Route Table:** Network routing quyết định traffic rời instance như thế nào.
* **Route Table + Gateway VPC Endpoint:** Traffic tới S3 được chuyển sang private AWS-managed path.
* **Gateway Endpoint + S3:** Object storage có thể truy cập mà không cần public Internet routing.
* **Workshop Architecture + Earlier Weeks:** Networking, identity và storage được kết nối lại trong cùng một bài lab.

### Kết quả đạt được tuần 7:

* Hiểu rõ use case và lợi ích của **Gateway VPC Endpoints** cho Amazon S3.
* Cấu hình thành công private S3 access từ workload trong VPC.
* Kết nối được kiến thức networking với storage và IAM behavior trong một kiến trúc thực tế.
* Tự tin hơn khi đọc workshop diagram và chuyển nó thành cấu hình AWS cụ thể.
* Chuẩn bị nền tảng vững chắc cho hybrid access scenario ở tuần kế tiếp.

### Khó khăn gặp phải:

* Việc chọn đúng route table cho lab đòi hỏi kiểm tra kỹ.
* Không chỉ cần tạo endpoint, tôi còn phải xác minh rằng traffic path thực sự đã thay đổi như mong đợi.

### Bài học rút ra và định hướng tiếp theo:

* Private connectivity trong AWS hầu như luôn là kết quả của việc tích hợp đúng giữa network design và access control.
* Sang tuần tiếp theo, tôi sẽ mở rộng mô hình này sang **Interface Endpoints**, **DNS** và **on-premises simulation** để thực hành hybrid architecture.
