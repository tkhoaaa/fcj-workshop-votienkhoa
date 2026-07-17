---
title: "Bản đề xuất"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---
# LingoRise — Nền tảng luyện thi IELTS & TOEIC ưu tiên tiếng Việt
## Giải pháp AWS Serverless cho luyện thi có hỗ trợ AI và vận hành nội dung phía quản trị

### 1. Tóm tắt điều hành
LingoRise là nền tảng web ưu tiên tiếng Việt, giúp người học chuẩn bị cho IELTS Academic, IELTS General Training và TOEIC. Nền tảng kết hợp lớp cá nhân hóa có hỗ trợ AI với quy trình xử lý nội dung phía quản trị, biến các kho tài liệu DOCX, PDF và PDF scan sẵn có thành một ngân hàng câu hỏi có cấu trúc và chấm điểm được. Toàn bộ hệ thống được xây dựng trên các dịch vụ AWS Serverless — API Gateway, AWS Lambda (Node.js 20), Amazon Cognito, Amazon RDS for PostgreSQL, Amazon S3 và AWS Amplify Hosting — nên có khả năng co giãn theo nhu cầu mà không cần quản lý máy chủ, đồng thời giữ chi phí vận hành tương ứng với mức sử dụng. Người học nhận được gợi ý học tập hằng ngày mang tính thích ứng, phân tích mẫu lỗi sai, chấm điểm band theo Cambridge và xem lại từng câu hỏi; người quản lý nội dung có quy trình import + OCR kèm hàng đợi kiểm duyệt và nhật ký audit; quản trị viên quản lý gói đăng ký, thanh toán và người dùng từ một nguồn dữ liệu duy nhất.

### 2. Tuyên bố vấn đề
*Vấn đề hiện tại*
Người học tiếng Việt luyện thi IELTS và TOEIC gặp ba khó khăn thường trực. Thứ nhất, **tài liệu luyện tập bị phân tán** trong các file PDF, DOCX và các website rời rạc, và việc đưa chúng vào một hệ thống thi có cấu trúc là thủ công và dễ sai sót. Thứ hai, **phản hồi quá chung chung** — hầu hết nền tảng chỉ đưa ra điểm band nhưng không cho người học biết họ *sai theo mẫu nào* hay *cần học gì tiếp theo*. Thứ ba, **các gói premium thiếu minh bạch** — đăng ký được gắn vào sản phẩm mà không có mô hình quyền lợi rõ ràng, khiến quản trị viên khó theo dõi ai đã trả tiền cho gì.

*Giải pháp*
LingoRise giải quyết cả ba. Quy trình xử lý nội dung phía quản trị chuyển đổi DOCX (văn bản + ảnh nội dòng), PDF (trích xuất luồng ảnh JPEG/JP2) và PDF scan (rasterize trang → OCR → câu hỏi khung) thành các câu hỏi đã duyệt, kèm phát hiện trùng lặp và hàng đợi kiểm duyệt. Trải nghiệm người học khai thác các câu trả lời trong bài thi thành các mẫu lỗi theo kỹ năng và theo section, sau đó cung cấp cho bộ gợi ý ("Hôm nay nên học gì?", thẻ kỹ năng yếu, sổ tay từ vựng). Mô hình đăng ký và thanh toán xác định quyền lợi ngay tại thời điểm request từ bảng `user_subscriptions`, mang lại cho quản trị viên một nguồn sự thật duy nhất cho quyền truy cập premium. Bộ máy thi hỗ trợ tiếp tục làm bài (resume), chấm điểm band Cambridge cho IELTS, band quy đổi cho TOEIC, và xem lại từng câu kèm giải thích và transcript phần nghe.

*Lợi ích và hoàn vốn đầu tư (ROI)*
Nền tảng loại bỏ hàng giờ nhập câu hỏi thủ công cho mỗi bộ đề thông qua quy trình import DOCX/PDF, và thay thế điểm số chung chung bằng phản hồi theo từng kỹ năng có thể hành động được, giúp cải thiện tỷ lệ giữ chân người học. Vì chạy trên AWS Serverless nên không có chi phí máy chủ nhàn rỗi: Lambda, API Gateway và S3 tính phí theo request, và thành phần luôn bật duy nhất là một instance RDS `db.t4g.micro` nhỏ. Chi phí hạ tầng ước tính ở quy mô dev/đầu production khoảng **30–40 USD/tháng** (xem Mục 6), chủ yếu đến từ RDS và WAF; compute và lưu trữ vẫn gần mức free tier ở lưu lượng khởi đầu. Bài toán hoàn vốn dựa trên thời gian tiết kiệm được trong vận hành nội dung và tỷ lệ chuyển đổi đăng ký premium nhờ lớp cá nhân hóa.

### 3. Kiến trúc giải pháp
LingoRise sử dụng kiến trúc AWS Serverless hoàn toàn tại **ap-southeast-1**. Lưu lượng người dùng đi vào hai bề mặt: frontend Next.js trên Amplify Hosting, và REST API trên API Gateway. AWS Lambda là tầng compute duy nhất — mỗi handler backend là một hàm Node.js 20 không trạng thái. Lambda xác thực JWT do Cognito phát hành ngay tại chỗ, đọc/ghi dữ liệu quan hệ trong RDS PostgreSQL, lưu tài sản (ảnh câu hỏi, bài nói, ảnh DOCX nháp) trong S3, và lấy secret mã hóa từ SSM Parameter Store. Một phân phối CloudFront tùy chọn phục vụ tài sản S3 riêng tư qua signed URL, và AWS WAF bảo vệ bề mặt API. Một worker OCR chạy theo lịch riêng sẽ rút hàng đợi công việc OCR dựa trên RDS và S3.

![LingoRise System Architecture](/images/2-Proposal/architecture.jpg)

*Dịch vụ AWS sử dụng*
- *Amazon API Gateway (REST)*: Endpoint HTTPS công khai đứng trước mọi Lambda handler.
- *AWS Lambda (Node.js 20)*: Compute không trạng thái cho toàn bộ handler backend (auth, exams, courses, admin, payments, health).
- *Amazon Cognito*: Xác thực người dùng cuối; phát hành JWT được Lambda xác minh với Cognito JWKS.
- *Amazon RDS for PostgreSQL*: Kho dữ liệu chính — người dùng, khóa học, bài thi, ngân hàng câu hỏi, thanh toán, job OCR, audit log.
- *Amazon S3*: Lưu trữ tài sản gồm ảnh câu hỏi, ảnh DOCX nháp và bài nói.
- *Amazon CloudFront*: CDN tùy chọn + signed URL cho tài sản riêng tư (`PriceClass_200`).
- *AWS Systems Manager Parameter Store*: Secret mã hóa (DB URL, khóa thanh toán, khóa AI).
- *Amazon CloudWatch Logs*: Đích ghi log của Lambda và access log có cấu trúc.
- *AWS Amplify Hosting*: Lưu trữ frontend Next.js (SSR + static assets).
- *AWS WAF*: Web ACL bảo vệ stage của API Gateway.
- *AWS IAM*: Execution role của Lambda và policy deployer theo nguyên tắc đặc quyền tối thiểu.

*Thiết kế thành phần*
- *Frontend*: Next.js (App Router) trên Amplify Hosting, giao diện ưu tiên tiếng Việt, React Query, gọi REST API kèm Bearer JWT.
- *API + Auth*: API Gateway định tuyến tới Lambda; Lambda xác minh JWT Cognito và tra bản ghi `users` cục bộ để lấy role/cờ hoạt động.
- *Compute*: Mỗi nhóm handler một Lambda; worker OCR theo lịch rút bảng `question_asset_ocr_jobs` bằng `FOR UPDATE SKIP LOCKED`.
- *Data*: RDS PostgreSQL (khóa chính UUID, metadata JSONB, migration idempotent); S3 asset bucket; SSM cho secret.
- *Bộ máy thi*: `POST /exams/start` dựng session từ một bộ đề cố định (fixed test set) hoặc lấy ngẫu nhiên từ ngân hàng câu hỏi, sau đó resolve URL tài sản (URL công khai hoặc presigned S3 URL 15 phút).
- *Thanh toán*: `POST /payments/create` ghi giao dịch pending; webhook IPN của nhà cung cấp xác minh HMAC trước khi chuyển trạng thái sang paid và kích hoạt gói đăng ký.

### 4. Triển khai kỹ thuật
*Các giai đoạn triển khai*
Nền tảng được triển khai qua bốn giai đoạn:
1. *Thiết kế & nền móng*: Mô hình dữ liệu (35 migration SQL idempotent), template AWS SAM, Cognito user pool, RDS, S3, IAM role.
2. *Bộ máy học + thi cốt lõi*: Danh mục khóa học, danh mục bài thi kèm trạng thái khả dụng, bắt đầu/tiếp tục session, chấm band Cambridge, xem lại từng câu.
3. *Quy trình nội dung phía quản trị*: Import DOCX/PDF/PDF scan, phát hiện trùng lặp, hàng đợi kiểm duyệt, CRUD ngân hàng câu hỏi, hàng đợi OCR + worker, audit log.
4. *Kiếm tiền & vận hành*: Gói đăng ký, resolve quyền lợi, bảng thanh toán (mock checkout + IPN nhà cung cấp), health endpoint, log có cấu trúc, security header, WAF, tài sản CloudFront signed.

*Yêu cầu kỹ thuật*
- *Backend*: Handler Node.js 20 đóng gói bằng esbuild thành artifact sẵn sàng cho Lambda, deploy qua AWS SAM tới stack CloudFormation `lingorise-dev`. Secret resolve từ SSM lúc deploy (`{{resolve:ssm:/lingorise/${Stage}/...}}`).
- *Cơ sở dữ liệu*: PostgreSQL 14+ trên RDS (`db.t4g.micro`, 20 GB), truy cập qua `pg.Pool` singleton; migration được theo dõi trong bảng `_migrations`.
- *Xác thực*: Xác minh JWT Cognito ở production (`aws-jwt-verify`), kèm dev-token fallback cho phát triển cục bộ.
- *OCR*: Worker nền Tesseract.js; job được đưa vào hàng đợi bởi hành động của quản trị viên và rút tuần tự.
- *Frontend*: Next.js App Router trên Amplify Hosting; CI/CD tự build khi push lên nhánh đã kết nối.

### 5. Lộ trình & Mốc triển khai
*Lộ trình dự án*
- *Giai đoạn 1 — Thiết kế & nền móng* (Tuần 1–2): Migration, template SAM, provisioning Cognito/RDS/S3.
- *Giai đoạn 2 — Bộ máy học + thi* (Tuần 3–5): Danh mục, session, chấm điểm, xem lại, resume.
- *Giai đoạn 3 — Quy trình nội dung phía quản trị* (Tuần 6–8): Import, hàng đợi OCR + worker, hàng đợi kiểm duyệt, audit log.
- *Giai đoạn 4 — Kiếm tiền & vận hành* (Tuần 9–10): Gói đăng ký, thanh toán, health, gia cố bảo mật (WAF, CloudFront signed URL).
- *Sau triển khai*: Bắc cầu IPN VNPay/MoMo thật lên bảng giao dịch, phiên luyện tập thích ứng, ứng dụng di động trên cùng bề mặt API.

### 6. Ước tính ngân sách
Nền tảng chạy trên AWS Serverless, nên chi phí chủ yếu đến từ instance RDS luôn bật và WAF; Lambda, API Gateway và S3 giữ gần mức free tier ở lưu lượng khởi đầu tại **ap-southeast-1**.

*Chi phí hạ tầng (ước tính, quy mô dev / đầu production)*
- *Amazon RDS (`db.t4g.micro`, 20 GB gp2)*: ~15 USD/tháng.
- *AWS WAF (Web ACL + rules)*: ~6–7 USD/tháng.
- *AWS Amplify Hosting*: ~1–5 USD/tháng (build minutes + request phục vụ).
- *Amazon API Gateway (REST)*: ~1–3 USD/tháng.
- *AWS Lambda*: ~0–1 USD/tháng (gần free tier ở lưu lượng khởi đầu).
- *Amazon S3 + CloudFront (`PriceClass_200`)*: ~1–2 USD/tháng.
- *Amazon Cognito*: 0 USD trong ngưỡng MAU miễn phí.
- *SSM Parameter Store (standard) + CloudWatch Logs*: ~0,5 USD/tháng.

*Tổng: khoảng 30–40 USD/tháng* ở quy mô dev/đầu production, co giãn theo class RDS và lưu lượng. Chi phí có thể ước tính chính xác bằng [AWS Pricing Calculator](https://calculator.aws/).

### 7. Đánh giá rủi ro
*Ma trận rủi ro*
- *RDS là điểm nghẽn đơn lẻ*: Ảnh hưởng cao, xác suất thấp.
- *Nhà cung cấp AI không khả dụng (Ollama/OpenRouter)*: Ảnh hưởng trung bình, xác suất trung bình.
- *Lệch tích hợp cổng thanh toán*: Ảnh hưởng trung bình, xác suất trung bình.
- *Vượt chi phí do đột biến lưu lượng*: Ảnh hưởng trung bình, xác suất thấp.

*Chiến lược giảm thiểu*
- *RDS*: Connection pooling qua `pg.Pool`; Multi-AZ standby và read replica có sẵn trên lộ trình gia cố production.
- *AI*: Fallback đám mây (OpenRouter) khi provider cục bộ không truy cập được; chiến lược pre-generation + cache giảm phụ thuộc trên request-path.
- *Thanh toán*: Thay đổi trạng thái chính thức diễn ra qua webhook IPN có xác minh HMAC, không phải qua browser return; mock checkout + admin mark-paid hỗ trợ luồng demo mà không chạm vào đường tiền thật.
- *Chi phí*: Cảnh báo AWS Budgets; billing serverless nghĩa là chi phí nhàn rỗi gần bằng không.

*Kế hoạch dự phòng*
- Quay về chế độ tài sản S3 công khai nếu cấu hình ký CloudFront gặp lỗi.
- Rollback bản deploy lỗi qua cập nhật stack CloudFormation; migration idempotent nên chạy lại an toàn.

### 8. Kết quả kỳ vọng
*Cải tiến kỹ thuật*: Việc nhập câu hỏi thủ công được thay bằng quy trình import DOCX/PDF/PDF scan kèm OCR và phát hiện trùng lặp. Điểm band chung chung được thay bằng phân tích mẫu lỗi theo từng kỹ năng và gợi ý thích ứng. Toàn bộ stack co giãn serverless theo nhu cầu.
*Giá trị dài hạn*: Mô hình quyền lợi một nguồn sự thật cho quyền truy cập premium, một bề mặt REST API tái sử dụng được cho ứng dụng di động tương lai, và nền móng cho các phiên luyện tập thích ứng cùng chấm điểm nói bằng AI thời gian thực.
