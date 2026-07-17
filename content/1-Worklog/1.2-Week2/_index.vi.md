---
title: "Worklog Tuần 2"
date: 2026-04-27
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---
### Mục tiêu tuần 2:

* Nắm vững các thành phần VPC và networking trên AWS.
* Xây dựng một network topology tùy biến cho workload cloud.
* Hiểu cách tài nguyên compute kết nối Internet và private subnet.

### Các công việc cần triển khai trong tuần này:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Tìm hiểu các thành phần VPC như Subnets, Route Tables, Internet Gateway, NAT Gateway và Network ACLs. | 27/04/2026 | 27/04/2026 | [Amazon VPC Docs](https://docs.aws.amazon.com/vpc/) |
| **2** | Tạo custom VPC với CIDR `10.0.0.0/16`, sau đó chia public subnet và private subnet trên các Availability Zones. | 28/04/2026 | 28/04/2026 | [VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html) |
| **3** | Cấu hình Internet Gateway và NAT Gateway để điều khiển outbound connectivity cho từng loại subnet. | 29/04/2026 | 30/04/2026 | [NAT Gateway Docs](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html) |
| **4** | Tạo Security Groups và Network ACLs để so sánh stateful security và stateless security. | 01/05/2026 | 02/05/2026 | [Security in VPC](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Security.html) |
| **5** | Khởi tạo EC2 test trong public subnet và private subnet để kiểm tra kết nối. | 03/05/2026 | 03/05/2026 | EC2 and VPC Console |

### Cách thực hiện chi tiết:

#### 1. Thiết kế VPC trước khi chạy workload

Thay vì dùng default VPC, tôi tạo một **VPC** riêng với CIDR `10.0.0.0/16`. Sau đó tôi chia thành:

* **Public subnet** cho tài nguyên cần truy cập Internet
* **Private subnet** cho tài nguyên không nên public trực tiếp

Đây là lần đầu tôi chủ động thiết kế tầng mạng thay vì chấp nhận cấu hình mặc định của AWS.

#### 2. Thiết lập đường Internet bằng Internet Gateway và NAT Gateway

Để public subnet có thể truy cập Internet, tôi gắn **Internet Gateway (IGW)** vào VPC và cập nhật public route table với route mặc định `0.0.0.0/0` trỏ đến IGW.

Đối với private subnet, tôi tạo **NAT Gateway** trong public subnet. Sau đó tôi cập nhật private route table để private instances có thể ra Internet cho việc update package hoặc cài phần mềm mà vẫn không bị public inbound.

Luồng dịch vụ thực tế khi đó là:

* **Private EC2** -> **Private Route Table** -> **NAT Gateway** -> **Internet Gateway** -> Internet

#### 3. Kết nối EC2 với network và security controls

Tôi tạo EC2 test trong cả hai loại subnet:

* Một instance trong **public subnet**
* Một instance trong **private subnet**

Public instance có thể SSH trực tiếp, trong khi private instance chỉ có outbound path được kiểm soát. Qua đó tôi hiểu cách **EC2**, **subnets**, **route tables** và **gateways** phối hợp với nhau trong môi trường thực tế.

#### 4. Áp dụng nhiều lớp network security

Tôi cấu hình:

* **Security Groups** ở cấp instance
* **Network ACLs** ở cấp subnet

Qua đó tôi so sánh được:

* **Stateful filtering** với Security Groups
* **Stateless filtering** với NACLs

Việc test traffic được phép và bị chặn giúp tôi hiểu rõ lớp nào phù hợp cho kiểm soát fine-grained ở cấp instance và lớp nào phù hợp cho kiểm soát rộng hơn ở cấp subnet.

### Kết nối các dịch vụ AWS trong tuần này:

* **VPC + Subnets:** VPC được chia thành các vùng mạng public và private.
* **Subnets + Route Tables:** Hành vi traffic được quyết định qua route association.
* **VPC + Internet Gateway:** Tài nguyên trong public subnet có thể ra Internet.
* **Private Subnet + NAT Gateway:** Private EC2 có outbound-only Internet access.
* **EC2 + Security Groups + NACLs:** Quyền truy cập được kiểm soát ở cả cấp instance và subnet.

### Kết quả đạt được tuần 2:

* Xây dựng thành công custom AWS network topology thay vì dùng default VPC.
* Hiểu rõ sự khác nhau về mục đích sử dụng giữa **IGW** và **NAT Gateway**.
* Biết đặt workload vào đúng subnet tùy theo mức độ public/private cần thiết.
* Nắm được sự khác biệt giữa **stateful** và **stateless** network security controls.
* Tạo nền tảng networking vững chắc cho các dịch vụ về sau như load balancing, storage access và private endpoints.

### Khó khăn gặp phải:

* Hành vi của route table ban đầu khá khó hình dung vì mỗi subnet có thể hoạt động khác nhau tùy route association.
* Tôi cần thời gian để hiểu rằng private subnet vẫn có thể ra Internet gián tiếp qua NAT Gateway mà không trở thành public.

### Bài học rút ra và định hướng tiếp theo:

* AWS networking là phần xương sống của cloud architecture, và mọi dịch vụ về sau đều phụ thuộc vào thiết kế subnet và routing đúng.
* Sang tuần tiếp theo, tôi sẽ đào sâu vào **EC2**, **Linux administration** và **EBS storage management** ngay trong môi trường mạng đã thiết kế.
