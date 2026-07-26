---
title: "Worklog"
date: 2026-05-12
weight: 1
chapter: true
pre: "<b>1. </b>"
---

{{% notice info %}}
**Giới thiệu:** Trang này tóm tắt toàn bộ quá trình thực tập 12 tuần của tôi tại chương trình **Workforce Bootcamp - First Cloud AI Journey (FCAJ)**. Lộ trình đi từ nền tảng AWS — compute, networking, database, security, DevOps và monitoring — sau đó chuyển sang xây dựng và triển khai nền tảng serverless **LingoRise** trên AWS, và kết thúc bằng kiểm thử, thu thập phản hồi người dùng, tài liệu hóa và báo cáo thực tập cuối kỳ.
{{% /notice %}}

### Tổng quan lộ trình 12 tuần

Kỳ thực tập được chia thành hai nửa rõ rệt. Bốn tuần đầu tôi tìm hiểu từng dịch vụ AWS và thực hành qua các bài lab. Từ tuần 5 trở đi, tôi áp dụng nền tảng đó vào một sản phẩm thực tế — **LingoRise**, nền tảng luyện thi IELTS/TOEIC — trải qua các bước proposal, thiết kế kiến trúc, phát triển, tích hợp, kiểm thử, thu phản hồi người dùng và bàn giao.

| Tuần | Chủ đề chính | Tóm tắt công việc |
| :--- | :--- | :--- |
| **Tuần 1** | [Nền tảng AWS: EC2, VPC và DRS](1.1-week1/) | Tìm hiểu EC2, VPC và Elastic Disaster Recovery, làm các bài tập thực hành cơ bản, thử deploy SAM và lưu khóa bảo mật trong Systems Manager Parameter Store. |
| **Tuần 2** | [Đi sâu VPC, các dịch vụ cơ sở dữ liệu và IAM](1.2-week2/) | Làm việc với Subnets, Route Tables, IGW, NAT Gateway và NACLs; so sánh Amazon RDS với DynamoDB; quản lý User, Group, Role và Policy trong IAM. |
| **Tuần 3** | [Workload trên EC2, Nginx, S3 Events và AMI](1.3-week3/) | Tái sử dụng VPC để chạy workload, cài Nginx trên Amazon Linux, dùng S3 Event Notifications để kích hoạt Lambda, tạo AMI phục vụ backup và repeated deployment. |
| **Tuần 4** | [DevOps trên AWS: CI/CD và CloudWatch](1.4-week4/) | Tìm hiểu CodeCommit, CodeBuild và CodePipeline, thiết lập CI/CD pipeline, tạo CloudWatch dashboard và alarm, triển khai tự động một ứng dụng web hoàn chỉnh. |
| **Tuần 5** | [Project Proposal và thiết kế kiến trúc](1.5-week5/) | Họp nhóm phân chia công việc, viết Project Proposal cho LingoRise, thiết kế sơ đồ kiến trúc hệ thống, khởi tạo source code và hạ tầng dạng code (IaC). |
| **Tuần 6** | [Tính năng cốt lõi, xác thực và lớp API](1.6-week6/) | Code phần quản lý người dùng và phân quyền với Cognito, phát triển các service xử lý dữ liệu, xây dựng API giữa Frontend và Backend với CORS và API Gateway. |
| **Tuần 7** | [Dịch vụ Backend và nền tảng Frontend](1.7-week7/) | Xây dựng core services, entities, repositories và logic nghiệp vụ ở Backend; thiết lập cấu trúc dự án, routing, state management và UI ở Frontend. |
| **Tuần 8** | [Tích hợp Frontend-Backend và bảo mật](1.8-week8/) | Kết nối Frontend với Backend qua RESTful API, hiển thị dữ liệu thật lên giao diện, hiện thực xác thực và phân quyền bằng JWT và Amazon Cognito. |
| **Tuần 9** | [Kiểm thử, sửa lỗi và tối ưu AI](1.9-week9/) | Kiểm thử chức năng và kiểm thử tích hợp giữa Frontend, Backend và các dịch vụ AWS, sửa các lỗi tồn đọng, tinh chỉnh luồng xử lý AI về tốc độ và độ chính xác. |
| **Tuần 10** | [Bàn giao, tối ưu chi phí và báo cáo tổng kết](1.10-week10/) | Đóng gói toàn bộ source code trên GitHub, dọn dẹp tài nguyên AWS không dùng và đặt hạn mức chi phí, hoàn thiện Final Internship Report bằng tiếng Anh. |
| **Tuần 11** | [Phân tích phản hồi người dùng và cải tiến sản phẩm](1.11-week11/) | Tổng hợp và phân tích chỉ số trải nghiệm người dùng, tỷ lệ lỗi trong thử nghiệm thực tế, sau đó tinh chỉnh giao diện React và sửa các lỗi phát sinh. |
| **Tuần 12** | [Sửa lỗi cuối, báo cáo Workshop và đóng gói](1.12-week12/) | Sắp xếp ưu tiên và xử lý triệt để các bug còn lại, viết Final Workshop Report kèm số liệu đánh giá của người dùng, đóng gói toàn bộ hồ sơ thực tập. |

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
   * một request đi từ browser tới database như thế nào
   * IAM permission và Cognito token ảnh hưởng tới quyền truy cập ra sao
   * cấu hình nào — network, policy hay environment variable — quyết định kết quả cuối cùng

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

* **Identity and Security:** IAM Users, Roles, Policies, MFA, Amazon Cognito
* **Networking:** VPC, Subnets, Route Tables, Internet Gateway, NAT Gateway, Security Groups, NACLs
* **Compute:** Amazon EC2, AWS Lambda (Node.js 20)
* **Application delivery:** Amazon API Gateway, AWS SAM, AWS Amplify Hosting
* **Database:** Amazon RDS for PostgreSQL, Amazon DynamoDB
* **Storage and Content Delivery:** Amazon S3, Amazon EBS, Amazon CloudFront
* **Observability:** Amazon CloudWatch, AWS CloudTrail
* **Configuration and Security:** AWS Systems Manager Parameter Store, AWS WAF
* **DevOps:** AWS CodeCommit, CodeBuild, CodePipeline
* **Governance and Cost Awareness:** AWS Budgets, Billing, Cost Explorer

Những dịch vụ này không được học tách rời, mà được kết nối dần thành một chuỗi hoàn chỉnh phục vụ một mục tiêu thực tế: xây dựng và triển khai **LingoRise**, nền tảng luyện thi IELTS/TOEIC theo kiến trúc serverless, trên AWS.

---

### Mục tiêu cuối cùng của phần Worklog

Phần worklog này không chỉ dùng để liệt kê công việc tôi đã hoàn thành trong thời gian thực tập, mà còn nhằm thể hiện:

* Quá trình tôi học và áp dụng AWS theo từng bước
* Cách tôi kết nối các dịch vụ AWS thành kiến trúc của một sản phẩm thực tế
* Tư duy troubleshooting, security và cost awareness trong quá trình làm lab và làm dự án
* Sự phát triển của tôi về kỹ năng kỹ thuật, teamwork, khả năng tự học và kỹ năng trình bày

Thông qua 12 tuần worklog, tôi mong muốn thể hiện rõ hành trình từ một người mới làm quen với AWS đến khi có thể hiểu, triển khai, giải thích và bàn giao một sản phẩm serverless hoàn chỉnh được xây dựng từ nhiều thành phần AWS liên kết với nhau.
