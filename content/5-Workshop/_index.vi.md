---
title : "Workshop"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5. </b> "
---


# Triển khai LingoRise — Nền tảng học tiếng Anh Serverless trên AWS

#### Tổng quan

LingoRise là một ứng dụng web học tiếng Anh/luyện thi IELTS, giúp người học luyện các kỹ năng đọc, viết, nghe và nói với chấm điểm hỗ trợ bởi AI cùng mô phỏng đề thi. Trong workshop này, bạn sẽ triển khai toàn bộ nền tảng lên AWS theo kiến trúc ưu tiên serverless.

Backend chạy trên **AWS Lambda (Node.js 20)** phía sau **Amazon API Gateway (REST)**, với việc xác thực do **Amazon Cognito** đảm nhiệm. Dữ liệu ứng dụng nằm trong **Amazon RDS for PostgreSQL 16**, trong khi tài nguyên tĩnh được lưu ở **Amazon S3** và phân phối qua **Amazon CloudFront**. Frontend được host trên **AWS Amplify Hosting**, các bí mật của ứng dụng được lưu trong **AWS Systems Manager (SSM) Parameter Store**, và khả năng giám sát được cung cấp bởi **Amazon CloudWatch**. Mọi thứ được đóng gói và triển khai bằng **AWS SAM** (Serverless Application Model).

Kết thúc workshop, bạn sẽ có một bản triển khai LingoRise hoạt động, có thể truy cập công khai, chạy tại region **ap-southeast-1** ở stage **dev**, với SAM stack tên là **lingorise-dev**. Bạn sẽ cấp phát cơ sở dữ liệu, nạp các bí mật, triển khai API, xuất bản frontend, kết nối phân phối tài nguyên, chạy kiểm thử nhanh, và cuối cùng dọn dẹp toàn bộ.

Vì kiến trúc là serverless, bạn nhận được những lợi ích thực sự so với cách dựng server truyền thống:

- **Trả theo mức dùng (pay-per-use)** — bạn chỉ bị tính phí cho số lượng request và thời gian tính toán thực sự sử dụng, không phải cho những server để không.
- **Không phải quản lý server** — không vá lỗi, không hoạch định dung lượng cho tầng tính toán; AWS vận hành runtime cho bạn.
- **Co giãn về không và mở rộng linh hoạt** — Lambda thu về không khi rảnh và tự động mở rộng khi có tải.

![Kiến trúc serverless của LingoRise trên AWS](/images/Workshop-LingoRise/1-introduction/architecture.jpg)

#### Kiến trúc

Bản triển khai sử dụng các dịch vụ AWS sau:

- **AWS Lambda (Node.js 20)** — chạy logic nghiệp vụ của API LingoRise.
- **Amazon API Gateway (REST)** — điểm vào HTTPS công khai, định tuyến request tới Lambda.
- **Amazon Cognito** — đăng ký, đăng nhập người dùng và ủy quyền dựa trên JWT.
- **Amazon RDS for PostgreSQL 16** — cơ sở dữ liệu quan hệ (`lingorise-dev-db`, tên DB `lingorise`).
- **Amazon S3** — lưu trữ tài nguyên (bucket `lingorise-assets-dev-<accountId>`).
- **Amazon CloudFront** — CDN đặt trước S3 để phân phối tài nguyên nhanh, có cache.
- **AWS Amplify Hosting** — build và phục vụ ứng dụng web frontend.
- **AWS Systems Manager (SSM) Parameter Store** — cấu hình ứng dụng và bí mật dưới `/lingorise/dev/`.
- **Amazon CloudWatch** — log và metric cho Lambda cùng phần còn lại của stack.
- **AWS SAM** — hạ tầng dưới dạng mã (infrastructure-as-code) để đóng gói và triển khai backend.

#### Nội dung

1. [Giới thiệu & Kiến trúc](1-introduction/)
2. [Chuẩn bị](2-prerequisites/)
3. [Bí mật & SSM Parameter Store](3-secrets-ssm/)
4. [Cơ sở dữ liệu — RDS PostgreSQL](4-database-rds/)
5. [Backend — Triển khai AWS SAM](5-backend-sam/)
6. [Frontend — AWS Amplify Hosting](6-frontend-amplify/)
7. [Lưu trữ & CloudFront (làm thêm)](7-storage-cloudfront/)
8. [Kiểm thử & Vận hành](8-smoke-test/)
9. [Dọn dẹp tài nguyên](9-cleanup/)
