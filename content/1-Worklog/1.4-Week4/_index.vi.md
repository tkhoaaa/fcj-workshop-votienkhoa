---
title: "Worklog Tuần 4"
date: 2026-05-11
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---
### Mục tiêu tuần 4:

* Hiểu quy trình phát triển phần mềm hiện đại (DevOps) trên AWS qua CodeCommit, CodeBuild và CodePipeline.
* Xây dựng CI/CD pipeline tự động hóa việc kiểm thử và triển khai mã nguồn lên môi trường thực thi.
* Dùng Amazon CloudWatch để tạo dashboard giám sát và cấu hình alarms cho workload.
* Triển khai hoàn chỉnh một ứng dụng web (frontend và backend) qua pipeline tự động, hướng tới tính sẵn sàng cao.

### Các công việc cần triển khai trong tuần này:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Tìm hiểu mô hình DevOps trên AWS và các stage source, build, test, deploy; tạo repository AWS CodeCommit cho ứng dụng lab. | 11/05/2026 | 11/05/2026 | [AWS CodeCommit Docs](https://docs.aws.amazon.com/codecommit/) |
| **2** | Tạo project AWS CodeBuild, viết `buildspec.yml` và lưu build artifact vào artifact bucket trên Amazon S3. | 12/05/2026 | 12/05/2026 | [AWS CodeBuild Docs](https://docs.aws.amazon.com/codebuild/) |
| **3** | Ghép AWS CodePipeline với các stage source, build, deploy và rà soát IAM service role mà từng stage assume. | 13/05/2026 | 13/05/2026 | [AWS CodePipeline Docs](https://docs.aws.amazon.com/codepipeline/) |
| **4** | Xây dựng CloudWatch dashboard cho workload và cấu hình alarms trên CPU cùng error metric, gửi thông báo qua Amazon SNS. | 14/05/2026 | 14/05/2026 | [Amazon CloudWatch Docs](https://docs.aws.amazon.com/cloudwatch/) |
| **5** | Triển khai trọn vẹn một ứng dụng web frontend và backend qua pipeline, kiểm tra tính sẵn sàng sau một lần release tự động. | 15/05/2026 | 15/05/2026 | [FCAJ Workshop](https://awsstudygroup.com/) |

### Cách thực hiện chi tiết:

#### 1. Ánh xạ mô hình DevOps sang các dịch vụ AWS

Trước khi mở console, tôi viết ra các stage của quy trình delivery và gán mỗi stage cho một dịch vụ. **AWS CodeCommit** giữ mã nguồn Git, **AWS CodeBuild** build và chạy test, còn **AWS CodePipeline** điều phối thứ tự chạy của các stage đó.

Mô hình tôi rút ra là:

* **Source** -> một commit vào branch trên CodeCommit kích hoạt pipeline
* **Build** -> CodeBuild chạy install, test và package trong container được quản lý
* **Deploy** -> artifact đã đóng gói được phát hành lên môi trường đích

Điểm quan trọng là bản thân pipeline không lưu gì cả. Mỗi stage chuyển artifact cho stage sau thông qua **Amazon S3**, và từng stage hoạt động bằng một **IAM** service role chứ không dùng credential cá nhân của tôi.

#### 2. Định nghĩa build bằng buildspec.yml và artifact bucket

Tôi tạo project CodeBuild trỏ tới repository CodeCommit và mô tả quá trình build trong `buildspec.yml` được commit cùng mã nguồn. Đặt định nghĩa build trong repository giúp build thay đổi đồng bộ với ứng dụng.

```yaml
version: 0.2
phases:
  install:
    runtime-versions:
      nodejs: 20
    commands: [npm ci]
  build:
    commands: [npm test, npm run build]
artifacts:
  files: ['**/*']
  base-directory: dist
```

CodeBuild ghi output vào artifact bucket S3 do CodePipeline tạo ra, đồng thời stream từng phase vào một log group của **CloudWatch Logs**. Khi phase test thất bại, pipeline dừng ngay ở build stage và deploy stage không chạy, đúng với tính chất an toàn mà CI/CD cần có.

#### 3. Ghép pipeline và rà soát các IAM service role

Trong CodePipeline tôi nối ba stage lại và kiểm tra các trust relationship. Pipeline role được đọc source và khởi động build; CodeBuild role được ghi log, đọc ghi artifact bucket và truy cập môi trường triển khai. Hai role tách biệt nên lỗi quyền ở build stage không ảnh hưởng tới source stage.

Sau đó tôi push một commit và theo dõi toàn bộ lần chạy mà không cần mở terminal thao tác tay:

```bash
git push origin main
aws codepipeline get-pipeline-state --name lingorise-lab-pipeline
```

Đây là tuần đầu tiên một thay đổi mã nguồn đi tới môi trường thực thi mà tôi không phải deploy thủ công.

#### 4. Giám sát workload bằng CloudWatch dashboard và alarms

Khi việc release đã tự động, câu hỏi tiếp theo là ứng dụng sau khi triển khai có khỏe hay không. Tôi xây dựng một dashboard **Amazon CloudWatch** gom CPU utilization của EC2, số lượng request và error metric trên cùng một màn hình, rồi tạo alarm cho CPU utilization và cho error metric. Cả hai alarm publish vào một topic **Amazon SNS** đã subscribe email, nhờ vậy một lần vượt ngưỡng trở thành thông báo thay vì điều tôi phát hiện muộn.

Phần thực hành cuối tuần, tôi triển khai hoàn chỉnh một ứng dụng web gồm frontend và backend qua pipeline và xác nhận ứng dụng vẫn truy cập được xuyên suốt một lần release tự động. Đây chính là bài tập dượt cho delivery pipeline của LingoRise: **AWS SAM** đóng gói backend Lambda Node.js 20 vào stack `lingorise-dev`, còn **AWS Amplify Hosting** chạy CI/CD riêng mỗi khi push frontend Next.js. Tên dịch vụ khác nhau, nhưng các stage, cách chuyển artifact và alarm trên CloudWatch đều cùng một hình dạng.

### Kết nối các dịch vụ AWS trong tuần này:

* **CodeCommit + CodePipeline:** Một commit vào branch được theo dõi trở thành trigger tự động cho cả lần release.
* **CodeBuild + S3:** Kết quả build được publish thành artifact để các stage sau tiêu thụ lại.
* **CodeBuild + CloudWatch Logs:** Mọi build phase được stream vào log group, giúp đọc lại nguyên nhân test fail sau đó.
* **CodePipeline + IAM:** Mỗi stage assume một service role giới hạn quyền thay vì chạy bằng danh tính của developer.
* **CloudWatch Alarms + SNS:** Việc vượt ngưỡng CPU và error metric được gửi tới email dưới dạng notification.
* **Pipeline + Ứng dụng đã triển khai:** Frontend và backend đang chạy chỉ được cập nhật qua đường tự động, không bao giờ bằng tay.

### Kết quả đạt được tuần 4:

* Xây dựng được CI/CD pipeline hoạt động thật, phủ source, build, test và deploy trên **CodeCommit**, **CodeBuild** và **CodePipeline**.
* Đưa định nghĩa build vào repository dưới dạng `buildspec.yml` để quá trình build tái lập được và review được.
* Hiểu cách artifact chảy giữa các stage của pipeline thông qua bucket **S3** thay vì build lại từ đầu.
* Tạo dashboard **CloudWatch** riêng cùng alarm cho CPU và error, kết nối tới topic **SNS**.
* Triển khai hoàn chỉnh ứng dụng frontend và backend qua pipeline và xác nhận ứng dụng trụ được qua một lần release tự động.
* Liên kết pipeline trong lab với đường delivery dự kiến của LingoRise dùng **AWS SAM** và **Amplify Hosting**.

### Khó khăn gặp phải:

* Những lần chạy pipeline đầu tiên fail vì quyền chứ không vì code, do CodeBuild service role không ghi được vào artifact bucket. Đọc log trên CloudWatch Logs nhanh hơn nhiều so với đoán policy.
* Chọn ngưỡng cho alarm khó hơn việc tạo alarm. Ngưỡng quá chặt thì mỗi lần deploy đều báo động, quá lỏng thì sự cố thật lại bị bỏ qua.

### Bài học rút ra và định hướng tiếp theo:

* Tự động hóa chỉ hữu ích khi nó báo lỗi rõ ràng. Pipeline dừng lại ở một test fail và alarm đến được tay người chịu trách nhiệm là hai nửa của cùng một ý tưởng.
* Giữ định nghĩa build và cấu hình triển khai trong repository giúp quy trình delivery được review như mọi thay đổi mã nguồn khác.
* Ở tuần tiếp theo, tôi sẽ chuyển từ các lab riêng lẻ sang **khởi động dự án**: phân chia công việc trong nhóm, viết project proposal, vẽ sơ đồ kiến trúc hệ thống và dựng khung mã nguồn cùng infrastructure as code.
