---
title: "Worklog Tuần 4"
date: 2026-05-11
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---
### Mục tiêu tuần 4:

* Tìm hiểu các dịch vụ lưu trữ của AWS như S3 và EFS.
* Hiểu sự khác nhau giữa object storage và file storage.
* Kết nối các dịch vụ lưu trữ với compute workload theo cách thực tế.

### Các công việc cần triển khai trong tuần này:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Tạo Amazon S3 buckets và tìm hiểu bucket naming, object structure cùng access control cơ bản. | 11/05/2026 | 11/05/2026 | [Amazon S3 Docs](https://docs.aws.amazon.com/s3/) |
| **2** | Bật Versioning và cấu hình Lifecycle Rules để mô phỏng hành vi lưu trữ lâu dài. | 12/05/2026 | 12/05/2026 | [S3 Versioning](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Versioning.html) |
| **3** | Cấu hình bucket access settings và tìm hiểu Bucket Policies để bảo vệ object access. | 13/05/2026 | 13/05/2026 | [S3 Bucket Policies](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-policy-language-overview.html) |
| **4** | Dùng Amazon S3 để host static website và kiểm tra khả năng public object delivery. | 14/05/2026 | 14/05/2026 | [Static Website Hosting](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html) |
| **5** | Tạo và mount Amazon EFS vào EC2 instances để test shared file storage. | 15/05/2026 | 15/05/2026 | [Amazon EFS Docs](https://docs.aws.amazon.com/efs/) |

### Cách thực hiện chi tiết:

#### 1. Lưu trữ file và asset bằng Amazon S3

Tôi tạo một hoặc nhiều **S3 bucket** để hiểu cách object storage hoạt động trên AWS. Thay vì gắn storage trực tiếp vào server, S3 lưu object độc lập và cho phép truy cập thông qua API hoặc URL.

Tôi thực hành:

* Upload file
* Tổ chức object theo key path
* Kiểm tra quyền truy cập ở cấp bucket và object

Qua đó tôi hiểu S3 phù hợp hơn với:

* Static assets
* Backups
* Public website files
* Durable object storage

#### 2. Áp dụng versioning và lifecycle cho object storage

Để mô hình quản lý storage thực tế hơn, tôi bật:

* **Versioning** để giữ nhiều version của cùng một object
* **Lifecycle Rules** để tự động chuyển object cũ sang storage class phù hợp hơn

Điều này cho thấy storage không chỉ là nơi lưu file mà còn gắn với governance và cost optimization.

#### 3. Dùng S3 làm static website hosting

Tôi cấu hình **static website hosting** cho bucket S3 và upload nội dung web đơn giản. Khi đó bucket trở thành một nền tảng nhẹ để host HTML và static assets.

Luồng tích hợp dịch vụ của tuần này là:

* **Local files** -> **S3 bucket**
* **Bucket policy/public access settings** -> quyết định khả năng truy cập từ browser
* **Web browser** -> lấy object trực tiếp từ S3

Đây là ví dụ rõ ràng cho việc một storage service có thể đóng vai trò phân phối nội dung web đơn giản.

#### 4. Kết nối EC2 với shared file storage bằng EFS

Khác với S3, **Amazon EFS** cung cấp shared file storage mà nhiều EC2 instances có thể mount cùng lúc. Tôi tạo hoặc rà soát EFS file system và mount vào EC2 để hiểu pattern lưu trữ dùng chung cho workload.

Từ đó tôi phân biệt rõ:

* **EBS** là block storage gắn cho một instance
* **EFS** là shared file storage dùng cho nhiều instance
* **S3** là object storage truy cập qua API

### Kết nối các dịch vụ AWS trong tuần này:

* **S3 + Bucket Policy:** Quyền truy cập object được kiểm soát bằng storage-level policies.
* **S3 + Static Website Hosting:** Storage được dùng trực tiếp như một lớp cung cấp nội dung web tĩnh.
* **EC2 + EFS:** Compute instances mount cùng một network file system.
* **S3 + Lifecycle/Versioning:** Quản lý lưu trữ được kết hợp với bảo vệ dữ liệu và tự động lưu trữ lâu dài.
* **Storage + Cost Awareness:** Storage class và lifecycle behavior ảnh hưởng đến hiệu quả chi phí.

### Kết quả đạt được tuần 4:

* Hiểu sâu hơn sự khác nhau giữa **object**, **block** và **file** storage trên AWS.
* Sử dụng thành công **S3** để lưu trữ object và host static website.
* Thực hành bảo vệ dữ liệu bằng **Versioning** và **Lifecycle Rules**.
* Mount thành công **EFS** vào EC2 và hiểu cách shared storage phục vụ nhiều workload.
* Chuẩn bị nền tảng storage quan trọng cho chủ đề private S3 access ở giai đoạn workshop sau này.

### Khó khăn gặp phải:

* Hành vi truy cập của S3 ban đầu khá khó hiểu vì public access settings, bucket policy và object permissions cùng ảnh hưởng đến kết quả cuối cùng.
* EFS đòi hỏi hiểu cả cấu hình phía AWS lẫn thao tác mount ở phía hệ điều hành trên EC2.

### Bài học rút ra và định hướng tiếp theo:

* Mỗi loại storage service giải quyết một bài toán khác nhau, và việc chọn đúng dịch vụ phụ thuộc vào access pattern của ứng dụng.
* Ở tuần tiếp theo, tôi sẽ quay lại chủ đề **security và access control**, đặc biệt là IAM và thiết kế quyền truy cập giữa các dịch vụ AWS.
