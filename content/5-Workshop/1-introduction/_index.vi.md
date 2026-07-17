---
title : "Giới thiệu & Kiến trúc"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 1. </b> "
---

#### Tổng quan

LingoRise là nền tảng học tiếng Anh và luyện thi IELTS: người học đăng ký, làm các bài thi thử tính giờ (Listening, Reading, Writing, Speaking), luyện tập theo ngân hàng câu hỏi, mua khoá học, và nhận chấm điểm cùng phản hồi từ AI. Backend là stack serverless Node.js; frontend là ứng dụng Next.js.

Trong workshop này, bạn sẽ triển khai toàn bộ nền tảng đó lên AWS tại region `ap-southeast-1` (Singapore), dùng stage `dev`. Khi hoàn thành, bạn sẽ có một endpoint API Gateway đang chạy, các hàm Lambda, một Cognito user pool, một cơ sở dữ liệu RDS PostgreSQL, một S3 asset bucket, và cấu hình được mã hoá trong SSM Parameter Store — tất cả được cấp phát qua một stack CloudFormation duy nhất tên `lingorise-dev`. Chúng ta đi từ dưới lên: hiểu kiến trúc trước, rồi chuẩn bị công cụ và thông tin đăng nhập, sau đó cấp phát từng thành phần, cuối cùng gắn frontend lên trên.

Chương này là tấm bản đồ. Nó cho thấy mọi dịch vụ AWS trong phạm vi, cách các thành phần trao đổi với nhau, và những gì bạn sẽ có khi hoàn tất.

#### Kiến trúc

LingoRise chạy theo mô hình serverless. Lưu lượng người dùng đổ vào hai bề mặt: **frontend** (Amplify Hosting, phục vụ ứng dụng Next.js) và **API** (API Gateway, đứng trước mọi backend handler). Lambda là tầng tính toán duy nhất — không có server chạy dài hạn nào phải vá lỗi. Lambda giao tiếp với RDS cho dữ liệu quan hệ, S3 cho asset, SSM cho secret, và gọi ra các đối tác bên ngoài để sinh nội dung AI và xử lý thanh toán. CloudWatch thu thập log từ mọi hàm.

![Kiến trúc tổng quan LingoRise](/images/Workshop-LingoRise/1-introduction/architecture.jpg)

Bảng dưới đây liệt kê mọi dịch vụ AWS trong phạm vi và vai trò của nó trong LingoRise.

| Dịch vụ | Mục đích |
|---|---|
| API Gateway (REST) | Endpoint HTTPS công khai cho mọi Lambda handler |
| AWS Lambda (Node.js 20) | Tính toán stateless cho toàn bộ backend handler |
| Amazon Cognito User Pool | Xác thực người dùng cuối; phát hành JWT được Lambda kiểm tra |
| Amazon RDS for PostgreSQL | Kho dữ liệu chính (users, courses, exams, ngân hàng câu hỏi, payments, OCR jobs, audit log) |
| Amazon S3 | Lưu trữ asset (ảnh câu hỏi, bản nháp DOCX, bài nộp phần Speaking) |
| Amazon CloudFront | CDN tuỳ chọn + URL có chữ ký cho asset riêng tư |
| AWS Systems Manager Parameter Store | Secret được mã hoá (DB URL, khoá thanh toán, khoá AI) |
| Amazon CloudWatch Logs | Nơi ghi log của Lambda |
| AWS Amplify Hosting | Hosting frontend Next.js (SSR + asset tĩnh) |
| Amazon SES | Email giao dịch (tuỳ chọn, tạm hoãn) |
| AWS IAM | Execution role của Lambda, policy cho deployer |

Một vài phụ thuộc bên ngoài (không phải AWS) vẫn nằm trong luồng dữ liệu và phải tiếp cận được từ Lambda:

- **Ollama / OpenRouter** — sinh nội dung AI và chat (chuyển sang OpenRouter để có endpoint truy cập được từ cloud)
- **PayOS** — redirect thanh toán và callback IPN
- **YouTube Data API, NewsAPI, GNews, Reddit** — các handler khám phá nội dung

{{% notice note %}}
Với stage `dev`, bạn có thể triển khai Lambda **bên ngoài** VPC và dùng một RDS instance truy cập công khai được nhưng bị khoá chặt bằng security group. Bố cục VPC với private subnet (Lambda ENI trải trên hai AZ, NAT cho lưu lượng ra, VPC endpoint cho S3 và SSM) là hướng gia cố cho production và là tuỳ chọn trong workshop này.
{{% /notice %}}

#### Các luồng xử lý

Ba luồng cho thấy các thành phần phối hợp ra sao. Hãy đọc chúng như danh sách các bước; cùng những luồng này được vẽ dưới dạng sequence diagram trong tài liệu kiến trúc gốc.

![Luồng đăng nhập](/images/Workshop-LingoRise/1-introduction/auth-flow.png)

**1. Đăng nhập và request đã xác thực (Cognito)**

Cognito phát hành JWT; sau đó mỗi Lambda được bảo vệ sẽ kiểm tra token đó cục bộ dựa trên Cognito JWKS và tra dòng `users` tương ứng để gắn role cùng cờ active của người gọi.

1. Người dùng gửi form đăng nhập trong ứng dụng Next.js.
2. Frontend gọi `POST /auth/login` qua API Gateway, kích hoạt Lambda `auth/login`.
3. Lambda đó gọi Cognito `AdminInitiateAuth` (USER_PASSWORD_AUTH) và nhận về idToken, accessToken, refreshToken.
4. Nó tra người dùng cục bộ bằng `SELECT users WHERE email = $1`, kiểm tra `is_active`, rồi gắn role.
5. Nó trả về `200 { token, user }`; frontend lưu token.
6. Ở bất kỳ lời gọi đã xác thực sau đó (ví dụ `GET /auth/me` với Bearer token), Lambda kiểm tra JWT dựa trên Cognito JWKS, đọc lại dòng người dùng, và phản hồi.

**2. Bắt đầu bài thi (fixed-set vs random + URL asset)**

`POST /exams/start` dựng một session, chọn câu hỏi theo một trong hai chế độ, rồi giải URL asset dựa trên `ASSET_URL_MODE`.

1. Người dùng gọi `POST /exams/start { templateId, mode }`.
2. Lambda mở một transaction và select template cùng các section của nó.
3. Ở chế độ **fixed-set** nó lấy câu hỏi từ `test_set_questions`; ở chế độ **random** nó chọn từ `question_bank` với bộ lọc và `ORDER BY RANDOM()`.
4. Nó nạp các `question_assets` tương ứng, chèn một dòng vào `exam_sessions` với `status='in_progress'`, rồi commit.
5. Với mỗi asset: ở chế độ **public** nó dùng `storage_url` đã lưu; ở chế độ **signed** nó tạo URL presigned S3 `GetObject` có hạn 15 phút.
6. Nó trả về `200 { sessionId, sections, questions, urls }`.

**3. Callback thanh toán (PayOS)**

Redirect trên trình duyệt (return URL) chỉ để phục vụ trải nghiệm người dùng. Thay đổi trạng thái có thẩm quyền diễn ra qua webhook IPN server-to-server, được Lambda kiểm tra bằng HMAC trước khi chạm vào dòng `payments`.

1. Người dùng bấm "Mua khoá học"; frontend gọi `POST /payments/create`.
2. Lambda `payments/create` chèn một dòng `payments` với `status='pending'`, dựng payload có chữ ký HMAC-SHA512, và trả về `redirectUrl`.
3. Frontend redirect trình duyệt tới cổng thanh toán; người dùng hoàn tất thanh toán.
4. **Nguồn sự thật (IPN):** cổng thanh toán gọi `POST /payments/{provider}-callback`. Lambda kiểm tra chữ ký HMAC, cập nhật payment thành `status='paid'`, và kích hoạt enrollment/subscription.
5. **Chỉ cho UX (return):** cổng thanh toán redirect trình duyệt về `FRONTEND_URL/return`; một lời gọi `GET /payments/return` đọc trạng thái payment hiện tại và báo `paid: true | false`.

#### Những gì bạn sẽ triển khai

Khi kết thúc workshop, thông qua stack CloudFormation `lingorise-dev` tại `ap-southeast-1`, bạn sẽ cấp phát:

- Một REST API **API Gateway** phơi mọi route backend qua HTTPS.
- Một tập **các hàm Lambda** (Node.js 20) cho auth, exams, ngân hàng câu hỏi, payments, OCR, và các handler nội dung.
- Một **Cognito User Pool** phát hành và xác thực JWT cho người dùng cuối.
- Một instance **RDS PostgreSQL** (`lingorise-dev-db`, database `lingorise`) với schema đã migrate.
- Một **S3 asset bucket** (`lingorise-assets-dev-<accountId>`) cho ảnh câu hỏi và bài nộp.
- Các mục **SSM Parameter Store** dưới `/lingorise/dev/` chứa DB URL, khoá thanh toán, và khoá AI.
- Các nhóm **CloudWatch Logs** cho mọi hàm, cùng các **IAM** execution role gắn kết mọi thứ lại.
- **Frontend Next.js** trên **Amplify Hosting**, được nối với endpoint API Gateway.

{{% notice info %}}
Tiếp theo: **[2. Điều kiện tiên quyết]** — cài AWS CLI, SAM CLI, và Node.js 20, tạo CLI profile `lingorise-dev`, và xác nhận tài khoản của bạn có thể tạo tài nguyên tại `ap-southeast-1` trước khi chạy `sam deploy` đầu tiên.
{{% /notice %}}
