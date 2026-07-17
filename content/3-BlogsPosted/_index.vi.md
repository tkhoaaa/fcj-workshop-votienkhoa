---
title: "Các bài blogs đã đăng"
date: 2026-07-20
weight: 3
chapter: false
pre: " <b> 3. </b> "
---
Tại đây sẽ liệt kê, giới thiệu các blog về cách **LingoRise** — nền tảng luyện thi IELTS/TOEIC ứng dụng AI — được xây dựng và vận hành trên AWS.

###  [Blog 1 - Xây dựng LingoRise trên AWS: Kiến trúc Serverless cho luyện thi IELTS/TOEIC](3.1-Blog1/)
Blog này trình bày kiến trúc serverless đầu-cuối của LingoRise: Route53 + ACM + Amplify phục vụ frontend Next.js, API Gateway chuyển tiếp tới một backend Express-trên-Lambda duy nhất, Cognito cho xác thực người dùng, S3 để lưu trữ tài nguyên, và AWS WAF làm lớp phòng thủ L7. Blog giải thích lợi ích cụ thể mà từng dịch vụ AWS mang lại cho một nhóm nhỏ — tính phí theo lượt gọi, chi phí scale-to-zero, xác thực được quản lý sẵn, và phòng thủ nhiều lớp — cùng lý do serverless phù hợp với lưu lượng tăng đột biến trong ngày thi.

###  [Blog 2 - AI Serverless trên AWS: Cách LingoRise tạo và chấm đề thi](3.2-Blog2/)
Blog này nói về pipeline nội dung AI: cách Lambda điều phối các lệnh gọi AI với chiến lược pre-generate-và-cache, lý do function được cấu hình 1024 MB / 120 giây, cách Amazon Polly neural TTS tổng hợp audio phần nghe mà không cần server, và cách S3 signed URL cùng lifecycle rule phân phối và tự hết hạn các tài nguyên được tạo ra. Blog kết lại ở chi phí và khả năng chịu lỗi — cache để giảm chi phí AI, rate limiting, và fallback mượt mà.
