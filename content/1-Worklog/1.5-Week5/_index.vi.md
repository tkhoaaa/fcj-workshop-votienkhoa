---
title: "Worklog Tuần 5"
date: 2026-05-18
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5:

* Khởi động dự án capstone cùng nhóm: phân chia công việc, thống nhất ý tưởng và viết Project Proposal.
* Thiết kế **System Architecture Diagram** của LingoRise trên AWS theo hướng phân rã kiểu microservices.
* Khởi tạo cấu trúc source code cho cả frontend Next.js và backend Lambda Node.js.
* Mô tả toàn bộ hạ tầng dưới dạng code bằng AWS SAM để mọi môi trường đều dựng lại được từ một template.

### Các công việc cần triển khai trong tuần này:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Họp kickoff nhóm, thống nhất ý tưởng LingoRise và phân chia công việc giữa frontend, backend và hạ tầng. | 18/05/2026 | 18/05/2026 | Biên bản họp nhóm |
| **2** | Viết Project Proposal gồm bài toán, giải pháp serverless, danh sách dịch vụ AWS, ngân sách hàng tháng và các rủi ro chính. | 19/05/2026 | 19/05/2026 | [AWS Pricing Calculator](https://calculator.aws/) |
| **3** | Vẽ sơ đồ kiến trúc hệ thống trên Draw.io và Figma, từ Amplify Hosting qua API Gateway và Lambda xuống RDS, S3, SSM. | 20/05/2026 | 20/05/2026 | [AWS Architecture Icons](https://aws.amazon.com/architecture/icons/) |
| **4** | Khởi tạo repository: frontend Next.js App Router, các Lambda handler Node.js 20 bundle bằng esbuild, module database dùng chung. | 21/05/2026 | 21/05/2026 | Repository dự án |
| **5** | Viết `template.yaml` đầu tiên cho AWS SAM, định nghĩa quy ước đặt tên SSM parameter và tạo các file SQL migration idempotent. | 22/05/2026 | 22/05/2026 | [AWS SAM Developer Guide](https://docs.aws.amazon.com/serverless-application-model/) |

### Cách thực hiện chi tiết:

#### 1. Họp kickoff và viết Project Proposal cho LingoRise

Ngày đầu tuần nhóm họp để thống nhất ý tưởng capstone và phân chia trách nhiệm. Chúng tôi chọn **LingoRise**, một nền tảng luyện thi IELTS và TOEIC ưu tiên người học Việt Nam, vì bài toán rất dễ phát biểu và dễ kiểm chứng: tài liệu luyện tập nằm rải rác ở nhiều nguồn, phản hồi sau mỗi bài thi còn chung chung, và các gói premium không nói rõ người học thực sự nhận được gì.

Tôi nhận phần hạ tầng và scaffolding backend, một bạn nhận frontend Next.js, một bạn nhận phần nội dung và mô hình dữ liệu đề thi. Project Proposal sau đó ghi lại đúng những phần mà người review sẽ hỏi:

* bài toán và đối tượng người học mục tiêu
* giải pháp serverless và danh sách dịch vụ AWS theo từng tầng
* ngân sách dự kiến khoảng 30 đến 40 USD mỗi tháng, phần lớn đến từ **Amazon RDS** và **AWS WAF**
* các rủi ro như chất lượng nội dung, độ chính xác của scoring và việc chi phí trôi dần trên database chạy liên tục

Việc tính ngân sách trước khi viết code đã làm thay đổi vài quyết định thiết kế. Chọn `db.t4g.micro` với 20 GB thay vì instance lớn hơn, và chỉ bật WAF trên một API stage, đều là quyết định ở mức proposal chứ không phải phần dọn dẹp về sau.

#### 2. Sơ đồ kiến trúc hệ thống trên AWS

Tôi vẽ kiến trúc trên Draw.io rồi chỉnh lại layout trên Figma để đưa thẳng vào proposal. Đường đi của một request là một mạch thẳng, dễ trình bày:

**Amplify Hosting** phục vụ frontend Next.js App Router, browser gọi **Amazon API Gateway** (REST), API Gateway invoke **AWS Lambda**, và Lambda đọc ghi **Amazon RDS for PostgreSQL**, **Amazon S3** và **AWS Systems Manager Parameter Store**. **Amazon Cognito** phát hành JWT để Lambda verify, còn **Amazon CloudFront** đứng trước các private asset bằng signed URL.

Phần microservices ở đây là cách phân rã bên trong Lambda, không phải một cụm container. Tôi gom handler theo domain: auth, exams, courses, admin, payments và health. Mỗi nhóm có function riêng và IAM role riêng, nên một lỗi ở payments không thể đọc speaking submission trong S3, và handler exams không chạm được vào các bảng subscription không thuộc phạm vi của nó. Toàn bộ chạy ở **ap-southeast-1** để giữ latency thấp cho người học tại Việt Nam.

#### 3. Khởi tạo cấu trúc source code cho frontend và backend

Tôi khởi tạo repository với ranh giới rõ giữa frontend Next.js App Router và các handler backend. Code backend là Node.js 20 bundle bằng esbuild, giúp package deploy nhỏ và cold start ngắn. Một module dùng chung giữ `pg.Pool` singleton để container Lambda đang warm tái sử dụng connection thay vì mở connection mới cho mỗi request.

Tôi cũng chốt luôn quy ước migration đầu tiên. Migration là các file SQL thuần, viết sao cho chạy lại lần hai vẫn an toàn, và những file đã áp dụng được ghi lại trong bảng `_migrations`:

```sql
CREATE TABLE IF NOT EXISTS _migrations (
  id          SERIAL PRIMARY KEY,
  filename    TEXT NOT NULL UNIQUE,
  applied_at  TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Bảng này nhỏ nhưng nó quyết định cách cả dự án triển khai thay đổi database về sau, nên tôi làm trước mọi bảng nghiệp vụ.

#### 4. Hạ tầng dưới dạng code với AWS SAM

Ngày cuối tuần dành cho `template.yaml`. **AWS SAM** mô tả API, các Lambda function, S3 bucket và RDS instance, rồi deploy tất cả thành một CloudFormation stack duy nhất là `lingorise-dev`. Không có tài nguyên nào được tạo tay trên Console, nên stack có thể xóa và dựng lại mà không mất cấu hình.

Secret không bao giờ xuất hiện trong template. Tôi thống nhất quy ước đặt tên `/lingorise/<stage>/...` trong Parameter Store dưới dạng SecureString, và template resolve chúng ở thời điểm deploy:

```yaml
Environment:
  Variables:
    DB_HOST: '{{resolve:ssm:/lingorise/${Stage}/db/host}}'
    DB_PASSWORD: '{{resolve:ssm:/lingorise/${Stage}/db/password}}'
    COGNITO_USER_POOL_ID: '{{resolve:ssm:/lingorise/${Stage}/cognito/user-pool-id}}'
```

Tham số `Stage` cho phép cùng một template sinh ra stack dev, và sau này là stack production, từ đúng một nguồn cấu hình.

### Kết nối các dịch vụ AWS trong tuần này:

* **Amplify Hosting + API Gateway:** frontend Next.js gọi REST API qua HTTPS thay vì truy cập trực tiếp vào bất kỳ tài nguyên backend nào.
* **API Gateway + Lambda:** mỗi nhóm route theo domain map vào một function Node.js 20 riêng, đây chính là ranh giới microservice.
* **Lambda + RDS for PostgreSQL:** các handler dùng chung `pg.Pool` singleton để container warm tái sử dụng connection tới `db.t4g.micro`.
* **Lambda + Systems Manager Parameter Store:** cấu hình database và Cognito được resolve từ SecureString parameter lúc deploy, không commit vào repo.
* **Cognito + Lambda:** user pool phát hành JWT và handler verify token trước khi chạy bất kỳ query nào.
* **AWS SAM + CloudFormation:** toàn bộ sơ đồ kiến trúc được viết thành một template dựng nên stack `lingorise-dev`.

### Kết quả đạt được tuần 5:

* Thống nhất ý tưởng LingoRise với nhóm và hoàn thành Project Proposal có scope, danh sách dịch vụ AWS, ngân sách và rủi ro.
* Hoàn thành System Architecture Diagram trên Draw.io và Figma, với cách phân rã Lambda theo domain và một IAM role cho mỗi domain.
* Khởi tạo xong repository: frontend Next.js App Router, handler Node.js 20 bundle bằng esbuild và module database dùng chung.
* Viết `template.yaml` đầu tiên cho AWS SAM và deploy thành công stack `lingorise-dev` ở `ap-southeast-1`.
* Thiết lập bảng `_migrations` và quy trình SQL migration idempotent.
* Chốt quy ước đặt tên SSM `/lingorise/<stage>/...` để không có secret nào nằm trong repository.

### Khó khăn gặp phải:

* Việc quyết định chia Lambda function đến mức nào cần bàn khá lâu. Quá nhiều function thì trùng lặp bundling và tăng cold start, quá ít function thì IAM role rộng hơn phạm vi domain mà nó phục vụ.
* Ước lượng ngân sách khó chốt vì RDS chạy liên tục, còn chi phí Lambda và API Gateway phụ thuộc vào lượng traffic mà lúc đó nhóm chưa dự đoán được.
* Để SAM deploy sạch ngay từ đầu, tôi phải sửa template vài lần, chủ yếu ở phần resource dependency và thứ tự resolve parameter.

### Bài học rút ra và định hướng tiếp theo:

* Viết proposal và vẽ kiến trúc trước khi code giúp giảm rất nhiều việc phải làm lại, vì các quyết định về sizing và chi phí đã được chốt sẵn.
* Bỏ thêm một ngày cho infrastructure as code là đáng. Khi đã có `template.yaml`, mọi thay đổi sau đó là một diff có thể review thay vì một cú click trên Console mà không ai còn nhớ.
* Sang tuần tiếp theo, tôi sẽ bắt đầu code các tính năng cốt lõi trên nền scaffolding này: quản lý người dùng, authentication và authorization bằng **Cognito**, các service xử lý dữ liệu, và những API đầu tiên expose qua **API Gateway** với CORS cấu hình cho frontend Amplify.
