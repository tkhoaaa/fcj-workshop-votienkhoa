---
title: "Worklog Tuần 8"
date: 2026-06-08
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---
### Mục tiêu tuần 8:

* Mở rộng workshop sang mô hình hybrid bằng cách sử dụng **Interface VPC Endpoint**.
* Hiểu vai trò của DNS trong private connectivity tới Amazon S3.
* Thực hành troubleshooting private access từ môi trường on-premises mô phỏng.

### Các công việc cần triển khai trong tuần này:

| Ngày| Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :--- | :--- | :--- | :--- | :--- |
| **1** | So sánh **Gateway Endpoints** và **Interface Endpoints** về kiến trúc, use case và cost considerations. | 08/06/2026 | 08/06/2026 | [AWS PrivateLink Docs](https://docs.aws.amazon.com/vpc/latest/privatelink/) |
| **2** | Tạo **S3 Interface Endpoint** và rà soát subnet placement cùng security group rules. | 09/06/2026 | 09/06/2026 | [Create Interface Endpoint](https://workshop-sample.fcjuni.com/5-workshop/5.4-cloudfront-amazon-quick/5.4.2-cloudfront-foundation-benefits/) |
| **3** | Kiểm tra private connectivity từ EC2 mô phỏng on-premises tới Amazon S3. | 10/06/2026 | 10/06/2026 | [Test Interface Endpoint](https://workshop-sample.fcjuni.com/5-workshop/5.4-cloudfront-amazon-quick/5.4.3-amazon-quick-ai-assistant/) |
| **4** | Cấu hình và xác nhận DNS simulation với Route 53 private hosted zones và custom records. | 11/06/2026 | 11/06/2026 | [DNS Simulation Lab](https://workshop-sample.fcjuni.com/5-workshop/5.4-cloudfront-amazon-quick/5.4.4-practical-reflection/) |
| **5** | Rà soát các lỗi thường gặp liên quan đến DNS, security groups và endpoint configuration. | 12/06/2026 | 12/06/2026 | Ghi chú troubleshooting |
| **6** | Ghi lại các quan sát quan trọng về hybrid connectivity và chuẩn bị screenshot cho workshop report. | 13/06/2026 | 13/06/2026 | Ghi chú cá nhân |

### Cách thực hiện chi tiết:

#### 1. Chuyển từ VPC-only access sang hybrid access design

Ở tuần 7, đường truy cập S3 được thiết kế cho workload đã nằm trong VPC. Sang tuần này, tôi mở rộng kiến trúc để hỗ trợ **on-premises simulation** bằng cách dùng **Interface Endpoint**.

Mô hình tích hợp thay đổi từ:

* **EC2 trong VPC** -> **Gateway Endpoint** -> **S3**

thành một mô hình thiên về hybrid hơn:

* **Workload mô phỏng on-premises**
* **VPC network path**
* **Interface Endpoint**
* **private DNS resolution**
* **Amazon S3**

#### 2. Tạo Interface Endpoint với network placement đúng

Tôi tạo **S3 Interface Endpoint**, chọn các subnets phù hợp và gắn **Security Group** tương ứng. Khác với Gateway Endpoint, Interface Endpoint dựa trên elastic network interfaces và subnet placement, nên việc cấu hình ở lớp mạng trở nên chi tiết hơn.

Qua đó tôi thấy rõ mối liên kết giữa:

* **Interface Endpoint**
* **Subnets**
* **Security Groups**
* **Private IP-based connectivity**

#### 3. Dùng DNS để điều hướng traffic tới private endpoint

Bài học quan trọng nhất của tuần này là private connectivity không chỉ phụ thuộc vào network resource mà còn phụ thuộc vào **name resolution**. Tôi dùng bài DNS simulation trong workshop để hiểu cách **Route 53** và private DNS records điều hướng request tới S3 vào endpoint thay vì public address path.

Mô hình tích hợp rất đáng chú ý ở đây là:

* **Application request**
* **DNS resolution**
* **Private endpoint**
* **AWS service**

#### 4. Troubleshoot theo từng lớp

Khi kiểm tra cấu hình, tôi lần lượt rà:

* trạng thái endpoint
* security group rules
* subnet và route alignment
* kết quả phân giải DNS
* IAM access

Quy trình troubleshooting theo lớp như vậy giúp bài lab hybrid trở nên dễ hiểu hơn và cũng phản ánh đúng cách debug hệ thống cloud ngoài thực tế.

### Kết nối các dịch vụ AWS trong tuần này:

* **Interface Endpoint + Subnets:** Private connectivity được đặt trong các subnet đã chọn của VPC.
* **Interface Endpoint + Security Group:** Traffic tới endpoint network interfaces được kiểm soát bằng rule mạng.
* **Route 53 + DNS Simulation:** Name resolution chuyển request ứng dụng sang private path.
* **EC2 On-Prem Simulation + S3:** Workload mô phỏng bên ngoài truy cập S3 thông qua AWS private networking.
* **IAM + Endpoint Path + DNS:** Truy cập thành công đòi hỏi quyền, kết nối và phân giải tên cùng đúng.

### Kết quả đạt được tuần 8:

* Hiểu rõ sự khác nhau thực tế giữa **Gateway** và **Interface** endpoints.
* Nắm được vai trò quan trọng của **DNS** trong private service connectivity.
* Thực hành thành công một workflow hybrid-style connectivity trong môi trường workshop.
* Cải thiện kỹ năng troubleshooting bằng cách kiểm tra từng lớp một.
* Thu thập được nhiều tài liệu kỹ thuật tốt để giải thích hybrid AWS architecture trong báo cáo.

### Khó khăn gặp phải:

* Phần DNS phức tạp hơn dự kiến vì chỉ một lỗi nhỏ cũng có thể khiến traffic bị điều hướng sai.
* Hybrid scenario có nhiều thành phần hơn hẳn so với VPC-only scenario của tuần trước.

### Bài học rút ra và định hướng tiếp theo:

* Private service access trên AWS thường phụ thuộc vào tổ hợp networking, DNS và identity chứ không phải chỉ một lớp duy nhất.
* Sang tuần tiếp theo, tôi sẽ củng cố khía cạnh bảo mật của kiến trúc này bằng **VPC Endpoint Policies** và các bài test restricted access behavior.
