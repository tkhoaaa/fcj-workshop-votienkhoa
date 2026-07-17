---
title: "Worklog Tuần 3"
date: 2026-05-04
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---
### Mục tiêu tuần 3:

* Đào sâu vào quản trị EC2 instance và Linux administration.
* Hiểu cách compute và storage phối hợp với nhau trên AWS.
* Thực hành chuẩn bị một EC2 instance cho web hosting và tái sử dụng về sau.

### Các công việc cần triển khai trong tuần này:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Tìm hiểu các lựa chọn thanh toán của EC2 như On-Demand và Spot, đồng thời xem lại cách sizing instance. | 04/05/2026 | 04/05/2026 | [EC2 Pricing Models](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-on-demand-instances.html) |
| **2** | Tạo mới hoặc tái sử dụng EC2 instance trong custom VPC để thực hành Linux administration. | 05/05/2026 | 05/05/2026 | EC2 Console |
| **3** | Cài đặt và cấu hình **Nginx** trên Amazon Linux, sau đó kiểm tra truy cập bằng public IP hoặc browser test. | 06/05/2026 | 06/05/2026 | [Nginx](https://nginx.org/) |
| **4** | Tạo, attach, mount và mở rộng **EBS volume** để hiểu cách persistent block storage hoạt động. | 07/05/2026 | 07/05/2026 | [Amazon EBS Docs](https://docs.aws.amazon.com/ebs/) |
| **5** | Tạo **AMI** và xem lại cách dùng image này cho backup hoặc repeated deployment. | 08/05/2026 | 08/05/2026 | [Create Amazon Machine Image](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/creating-an-ami-ebs.html) |

### Cách thực hiện chi tiết:

#### 1. Tái sử dụng VPC để chạy workload thực tế

Tuần này nối tiếp kiến thức networking của tuần 2 bằng cách đặt workload thật vào trong custom VPC. Tôi tạo mới hoặc tái sử dụng một **EC2 instance** trong VPC và bảo đảm **Security Group** mở đúng port cho HTTP và SSH khi cần.

Qua đó tôi thấy rõ quan hệ trực tiếp:

* **EC2** phụ thuộc vào **subnet placement**
* **Security Group** quyết định khả năng truy cập ứng dụng
* **Route table + IGW** quyết định người dùng có thể truy cập web server hay không

#### 2. Cài Nginx và public web server

Sau khi SSH vào EC2, tôi cập nhật package và cài **Nginx**. Tiếp theo tôi enable và start service để instance có thể phục vụ nội dung web.

Luồng kiểm tra gồm:

* Kết nối SSH vào EC2
* Cài và khởi động Nginx
* Mở port `80` trong Security Group
* Truy cập qua trình duyệt bằng public IP hoặc public DNS

Bài thực hành này kết nối:

* **EC2 compute**
* **Linux administration**
* **Security Group firewall rules**
* **Browser/client access**

#### 3. Bổ sung persistent storage bằng EBS

Tôi tạo một **EBS volume** riêng, attach vào EC2 instance và mount trong hệ điều hành. Việc này giúp tôi phân biệt rõ:

* Instance là **compute**
* Volume gắn kèm là **persistent block storage**

Tôi cũng thực hành mở rộng EBS volume và xem cách thay đổi storage ở phía AWS cần đi kèm với việc mở rộng filesystem ở phía OS.

#### 4. Chụp cấu hình máy thành AMI

Sau khi EC2 đã được cấu hình hoàn chỉnh, tôi tạo **Amazon Machine Image (AMI)** từ instance đó. Điều này cho phép tái sử dụng cùng cấu hình web server sau này mà không cần lặp lại toàn bộ các bước cài đặt thủ công.

Bước này thể hiện mối liên kết giữa:

* **EC2** là máy nguồn
* **EBS snapshots** là cơ chế lưu phía sau AMI
* **AMI** là deployment template có thể tái sử dụng

### Kết nối các dịch vụ AWS trong tuần này:

* **EC2 + VPC/Subnet:** Instance chạy trong custom network của tuần 2.
* **EC2 + Security Group:** Truy cập web và SSH được kiểm soát qua inbound rules.
* **EC2 + EBS:** Persistent storage được attach, mount và mở rộng.
* **EC2 + AMI:** Máy đã cấu hình được chuyển thành reusable image.
* **Linux + AWS Infrastructure:** Các lệnh trên OS giúp tài nguyên cloud hoạt động đúng mục đích.

### Kết quả đạt được tuần 3:

* Cải thiện kỹ năng Linux administration trong môi trường AWS compute thực tế.
* Triển khai thành công web server đơn giản dựa trên Nginx trên EC2.
* Hiểu vòng đời của **EBS volumes** và cách thay đổi storage trên cloud tương tác với hệ điều hành.
* Biết AMI giúp hỗ trợ repeatable deployment và backup strategy như thế nào.
* Tạo cầu nối thực tế giữa networking, compute và storage.

### Khó khăn gặp phải:

* Một số thao tác đòi hỏi hiểu đồng thời cả lớp AWS Console lẫn lớp hệ điều hành Linux.
* Việc mở rộng storage không chỉ là tăng dung lượng volume ở AWS mà còn cần chỉnh lại filesystem trong instance.

### Bài học rút ra và định hướng tiếp theo:

* Compute service trên AWS chỉ thực sự hữu ích khi được kết hợp đúng với storage, security và cấu hình hệ điều hành.
* Sang tuần tiếp theo, tôi sẽ tập trung nhiều hơn vào **storage services** như **S3** và **EFS**, cũng như cách chúng kết nối với workload chạy trên EC2.
