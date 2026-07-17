---
title: "Worklog"
date: 2026-05-12
weight: 1
chapter: true
pre: "<b>1. </b>"
---

{{% notice info %}}
**Giới thiệu:** Trang này tóm tắt toàn bộ quá trình thực tập 12 tuần của tôi tại chương trình **Workforce Bootcamp - First Cloud AI Journey (FCAJ)**. Lộ trình được xây dựng theo hướng đi từ nền tảng AWS, networking, compute, storage, security, monitoring đến workshop chuyên sâu về **private/hybrid access to Amazon S3**, sau đó hoàn thiện bằng documentation, knowledge sharing và internship report.
{{% /notice %}}

### Tổng quan lộ trình 12 tuần

Trong suốt thời gian thực tập, tôi không chỉ học từng dịch vụ AWS riêng lẻ mà còn từng bước kết nối chúng thành những kiến trúc hoàn chỉnh. Lộ trình này giúp tôi hiểu rõ hơn cách các thành phần như **IAM**, **VPC**, **EC2**, **EBS**, **S3**, **EFS**, **CloudWatch**, **CloudTrail**, **Route 53** và **VPC Endpoints** phối hợp với nhau trong thực tế.

| Tuần | Chủ đề chính | Tóm tắt công việc |
| :--- | :--- | :--- |
| **Tuần 1** | [Khởi động, bảo mật tài khoản và thiết lập môi trường](1.1-week1/) | Làm quen với AWS, bảo mật tài khoản bằng MFA và IAM, cài AWS CLI, khởi tạo EC2 đầu tiên. |
| **Tuần 2** | [Networking với VPC, Subnet và Route Table](1.2-week2/) | Thiết kế custom VPC, chia public/private subnet, cấu hình IGW, NAT Gateway, Security Group và NACL. |
| **Tuần 3** | [Compute với EC2, Linux và EBS](1.3-week3/) | Cấu hình EC2 instance, cài Nginx, attach và mở rộng EBS volume, tạo AMI để tái sử dụng. |
| **Tuần 4** | [Storage với S3 và EFS](1.4-week4/) | Tạo S3 bucket, bật versioning và lifecycle rules, host static website và mount EFS vào EC2. |
| **Tuần 5** | [IAM, Roles và Security Governance](1.5-week5/) | Thiết kế least-privilege access, gắn IAM Role vào EC2, test truy cập S3 không dùng static credentials. |
| **Tuần 6** | [CloudWatch, CloudTrail và khả năng quan sát hệ thống](1.6-week6/) | Thiết lập metrics, alarms, logs và audit trail để theo dõi tình trạng hạ tầng đã triển khai. |
| **Tuần 7** | [Gateway VPC Endpoint cho Amazon S3](1.7-week7/) | Thực hành workshop private S3 access từ workload trong VPC thông qua Gateway Endpoint và route tables. |
| **Tuần 8** | [Interface Endpoint, Route 53 và Hybrid Connectivity](1.8-week8/) | Mô phỏng truy cập S3 từ on-premises environment, cấu hình Interface Endpoint và DNS resolution với Route 53. |
| **Tuần 9** | [VPC Endpoint Policies và bảo mật nhiều lớp](1.9-week9/) | Giới hạn truy cập S3 bằng endpoint policy, bucket policy và IAM policy, đồng thời xây dựng checklist troubleshooting. |
| **Tuần 10** | [Automation, documentation và cost awareness](1.10-week10/) | Tổng hợp AWS CLI commands, tạo helper scripts, rà soát chi phí và chuẩn hóa tài liệu workshop. |
| **Tuần 11** | [Knowledge sharing và report drafting](1.11-week11/) | Chuyển phần kỹ thuật thành nội dung dễ hiểu, viết nháp worklog/report và chuẩn bị tài liệu chia sẻ. |
| **Tuần 12** | [Final review, self-evaluation và internship wrap-up](1.12-week12/) | Rà soát toàn bộ 12 tuần, hoàn thiện báo cáo, dọn tài nguyên lab và xác định định hướng học tập tiếp theo. |

---

### Cách tôi thực hiện worklog trong suốt internship

Tôi theo đuổi phương pháp **Learning by Doing** và duy trì cấu trúc worklog theo hướng vừa học lý thuyết, vừa thực hành, vừa ghi nhận bài học kỹ thuật thực tế:

1. **Nghiên cứu lý thuyết**
   Tôi đọc tài liệu chính thống từ [AWS Documentation](https://docs.aws.amazon.com/) và workshop materials để hiểu đúng khái niệm trước khi thao tác.

2. **Triển khai thực hành**
   Tôi tạo tài nguyên trực tiếp trên AWS bằng cả **AWS Management Console** và **AWS CLI** để hiểu đồng thời hai cách vận hành.

3. **Quan sát kết nối giữa các dịch vụ**
   Mỗi tuần tôi không chỉ ghi lại “đã tạo gì”, mà còn chú ý tới việc:
   * dịch vụ nào kết nối với dịch vụ nào
   * network path đi như thế nào
   * IAM permission ảnh hưởng ra sao
   * DNS, route table hay policy nào quyết định kết quả cuối cùng

4. **Ghi chép và phản tư**
   Tôi cập nhật worklog theo tuần để ghi lại:
   * mục tiêu
   * công việc đã làm
   * cách triển khai chi tiết
   * kết quả đạt được
   * khó khăn gặp phải
   * bài học rút ra và định hướng tiếp theo

---

### Những nhóm dịch vụ AWS nổi bật trong lộ trình này

Trong 12 tuần, các nhóm dịch vụ tôi sử dụng và kết nối nhiều nhất gồm:

* **Identity and Security:** IAM Users, Roles, Policies, MFA
* **Networking:** VPC, Subnets, Route Tables, Internet Gateway, NAT Gateway, Security Groups, NACLs
* **Compute:** Amazon EC2
* **Storage:** Amazon EBS, Amazon S3, Amazon EFS
* **Observability:** Amazon CloudWatch, AWS CloudTrail
* **Hybrid/Private Access:** Gateway VPC Endpoint, Interface VPC Endpoint, Route 53
* **Governance and Cost Awareness:** AWS Budgets, Billing, Cost Explorer

Những dịch vụ này không được học tách rời, mà được kết nối dần thành một chuỗi hoàn chỉnh phục vụ bài toán thực tế về **private access to Amazon S3** trong môi trường cloud và hybrid.

---

### Mục tiêu cuối cùng của phần Worklog

Phần worklog này không chỉ dùng để liệt kê công việc tôi đã hoàn thành trong thời gian thực tập, mà còn nhằm thể hiện:

* Quá trình tôi học và áp dụng AWS theo từng bước
* Cách tôi kết nối các dịch vụ AWS thành kiến trúc thực tế
* Tư duy troubleshooting, security và cost awareness trong quá trình làm lab
* Sự phát triển của tôi về kỹ năng kỹ thuật, khả năng tự học và kỹ năng trình bày

Thông qua 12 tuần worklog, tôi mong muốn thể hiện rõ hành trình từ một người mới làm quen với AWS đến khi có thể hiểu, triển khai, giải thích và tổng kết một kiến trúc cloud có nhiều thành phần liên quan với nhau.
