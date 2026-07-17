---
title: "Worklog Tuần 1"
date: 2026-04-20
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---
### Mục tiêu tuần 1:

* Làm quen với môi trường thực tập và team First Cloud AI Journey (FCAJ).
* Hiểu các khái niệm nền tảng về Cloud Computing và AWS Global Infrastructure.
* Thiết lập môi trường AWS Free Tier an toàn để phục vụ các bài lab thực hành.
* Xây dựng quy trình làm việc cơ bản sẽ sử dụng trong suốt 12 tuần thực tập.

### Các công việc cần triển khai trong tuần này:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Tham gia team, tìm hiểu quy định thực tập và nghiên cứu các mô hình Cloud Computing như IaaS, PaaS và SaaS. | 20/04/2026 | 20/04/2026 | [FCAJ Guide](https://workshop-sample.fcjuni.com/) |
| **2** | Tạo tài khoản AWS Free Tier và áp dụng các thực hành bảo mật như bật MFA cho Root account, tạo IAM administrator user. | 21/04/2026 | 21/04/2026 | [AWS IAM Docs](https://docs.aws.amazon.com/iam/) |
| **3** | Tìm hiểu về AWS Regions và Availability Zones, sau đó cấu hình AWS Budget Alerts để kiểm soát chi phí. | 22/04/2026 | 22/04/2026 | [AWS Billing](https://aws.amazon.com/billing/) |
| **4** | Cài đặt và cấu hình AWS CLI trên máy tính cá nhân, tạo access key và thiết lập profile mặc định cho `ap-southeast-1`. | 23/04/2026 | 23/04/2026 | [AWS CLI Guide](https://docs.aws.amazon.com/cli/) |
| **5** | Tìm hiểu kiến thức cơ bản về Amazon EC2 như instance types, AMIs, key pairs và Shared Responsibility Model. | 24/04/2026 | 25/04/2026 | [Amazon EC2](https://aws.amazon.com/ec2/) |
| **6** | Khởi tạo EC2 đầu tiên với Amazon Linux 2023 và kiểm tra kết nối bằng SSH cũng như AWS Instance Connect. | 26/04/2026 | 26/04/2026 | [FCAJ Workshop](https://awsstudygroup.com/) |

### Cách thực hiện chi tiết:

#### 1. Bảo mật tài khoản AWS trước khi tạo tài nguyên

Bước thực hành đầu tiên của tôi không phải là tạo EC2 ngay mà là bảo mật tài khoản AWS đúng cách. Tôi đăng nhập bằng Root account, bật **MFA**, sau đó tạo riêng một **IAM administrator user** để sử dụng hằng ngày. Đây là best practice của AWS vì Root account chỉ nên dùng cho các thao tác nhạy cảm ở cấp tài khoản.

Tôi cũng rà lại vai trò của:

* **Root account** dùng để sở hữu tài khoản
* **IAM user** dùng cho quản trị hằng ngày
* **IAM policy** dùng để kiểm soát quyền hạn

Thiết lập này trở thành nền tảng cho toàn bộ các bài lab sau, vì mọi dịch vụ AWS ở các tuần tiếp theo đều được tạo trong mô hình truy cập đã được kiểm soát.

#### 2. Cấu hình AWS CLI để kết nối máy cá nhân với AWS

Sau khi tạo IAM user, tôi tạo access key và cấu hình AWS CLI trên máy bằng:

```bash
aws configure
aws sts get-caller-identity
```

Bước này giúp kết nối **máy local** với **AWS account**, từ đó tôi có thể quản lý dịch vụ bằng cả giao diện web lẫn dòng lệnh. Cách làm song song này rất hữu ích về sau khi test networking, truy cập S3 và kiểm tra hành vi của các endpoint.

#### 3. Hiểu EC2 như một phần của môi trường AWS

Trước khi khởi tạo instance, tôi xem lại cách EC2 phụ thuộc vào nhiều thành phần AWS khác:

* **AMI** cung cấp image cho máy ảo
* **Key Pair** dùng để SSH an toàn
* **Security Group** kiểm soát inbound và outbound traffic
* **Region/AZ** quyết định vị trí tài nguyên
* **EBS** đóng vai trò là block storage gắn kèm

Qua đó tôi hiểu rằng EC2 không phải là một dịch vụ đứng riêng, mà luôn đi cùng với **IAM**, **network security** và **storage**.

#### 4. Tạo và kiểm tra EC2 đầu tiên

Tôi khởi tạo một Amazon Linux 2023 instance từ AWS Console, chọn instance type phù hợp với Free Tier, gắn system EBS volume và tạo hoặc chọn Key Pair. Sau đó tôi mở rule inbound tối thiểu trong Security Group để cho phép SSH.

Khi instance đã sẵn sàng, tôi kiểm tra kết nối bằng:

* **AWS Instance Connect** ngay trên trình duyệt
* **SSH** từ máy local bằng private key đã tải xuống

Bài lab này thể hiện mối liên kết đầu tiên giữa:

* **IAM user permissions**
* **EC2**
* **Security Group**
* **Key Pair**
* **EBS volume**

### Kết nối các dịch vụ AWS trong tuần này:

* **IAM + AWS Account Security:** IAM user được dùng thay cho Root account trong quản trị hằng ngày.
* **AWS CLI + IAM Credentials:** CLI xác thực bằng access key của IAM user.
* **EC2 + Security Group:** Truy cập tới instance được kiểm soát bằng các rule mạng.
* **EC2 + EBS:** Instance sử dụng block storage gắn kèm làm system disk.
* **Region + Availability Zone + EC2:** Compute resource được khởi tạo ở vị trí AWS xác định để đồng nhất cho các bài lab tiếp theo.

### Kết quả đạt được tuần 1:

* Nắm được các lợi ích chính của Cloud Computing và ý nghĩa của AWS Global Infrastructure.
* Bảo mật tài khoản AWS thành công bằng **MFA** và mô hình **least privilege**.
* Cài đặt và cấu hình **AWS CLI**, xác minh danh tính bằng `aws sts get-caller-identity`.
* Tạo và kết nối thành công **EC2 instance** bằng cả trình duyệt và terminal.
* Xây dựng nền tảng thực hành ban đầu để phục vụ việc kết nối các dịch vụ ở những tuần sau.

### Khó khăn gặp phải:

* Ở giai đoạn đầu, có khá nhiều khái niệm mới cùng xuất hiện như IAM, CLI, Regions và các phụ thuộc của EC2.
* Việc hiểu vì sao cần bảo mật tài khoản trước khi dựng hạ tầng đòi hỏi tôi chuyển tư duy từ “chạy được” sang “chạy an toàn”.

### Bài học rút ra và định hướng tiếp theo:

* Mọi bài lab trên AWS đều phụ thuộc vào một nền tảng tài khoản an toàn và được cấu hình đúng ngay từ đầu.
* Sang tuần tiếp theo, tôi sẽ chuyển sang **VPC networking** để đặt EC2 vào môi trường mạng tùy biến thay vì chỉ dùng mặc định của AWS.
