---
title: "Worklog Tuần 9"
date: 2026-06-15
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Mục tiêu tuần 9:

* Kiểm thử chức năng toàn bộ luồng người học của LingoRise và các luồng quản trị nội dung.
* Kiểm thử tích hợp tại các ranh giới dịch vụ thực tế: **API Gateway**, **AWS Lambda**, **Amazon RDS**, **Amazon S3**, **AWS Systems Manager** và **Amazon Cognito**.
* Sửa các lỗi tồn đọng phát hiện trong quá trình kiểm thử, không chỉ ghi nhận lại.
* Tối ưu hóa luồng xử lý AI và OCR về tốc độ trích xuất, độ chính xác và khả năng chịu lỗi.

### Các công việc cần triển khai trong tuần này:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Viết test plan chức năng cho luồng người học (sign-in, catalog, start exam, answer, submit, score, review) và chạy trên API stage `lingorise-dev`. | 15/06/2026 | 15/06/2026 | Repository dự án |
| **2** | Kiểm thử chức năng các luồng quản trị nội dung: import DOCX và PDF, duplicate detection, review queue, publish và audit log. | 16/06/2026 | 16/06/2026 | Ghi chú tự học |
| **3** | Kiểm thử tích hợp end-to-end tại các ranh giới dịch vụ và dùng **Amazon CloudWatch Logs** để truy vết lỗi xảy ra bên trong Lambda. | 17/06/2026 | 17/06/2026 | [Amazon CloudWatch Logs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html) |
| **4** | Sửa các lỗi phát hiện trong ba ngày trước và chạy lại những test case liên quan sau mỗi lần `sam deploy`. | 18/06/2026 | 18/06/2026 | [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/using-sam-cli.html) |
| **5** | Tinh chỉnh OCR job queue và độ chính xác của Tesseract.js, sau đó kiểm tra luồng fallback từ AI provider nội bộ sang OpenRouter. | 19/06/2026 | 19/06/2026 | [AWS Lambda Docs](https://docs.aws.amazon.com/lambda/) |

### Cách thực hiện chi tiết:

#### 1. Kiểm thử chức năng luồng người học và luồng quản trị

Tôi kiểm thử luồng người học đúng theo cách một người dùng thật sẽ đi: đăng nhập qua **Amazon Cognito**, xem catalog đề thi, bắt đầu một session IELTS, trả lời câu hỏi, submit, nhận band score và mở phần review từng câu kèm giải thích và transcript phần listening. Trường hợp tôi kiểm tra lại nhiều nhất là session resume, vì người học đóng tab rồi quay lại phải nhận đúng bản ghi `exam_sessions` cũ, không phải một lượt thi mới.

Ở phía quản trị, tôi import một file DOCX thật và một scanned PDF, rồi kiểm tra duplicate detection có phát hiện câu hỏi tôi cố tình import hai lần, review queue có giữ draft lại thay vì publish trực tiếp, và mọi hành động publish đều được ghi vào bảng audit log.

Thói quen quan trọng ở đây là viết kết quả mong đợi trước khi chạy test. Nếu không, một band score sai vẫn trông hợp lý và sẽ đi qua mà không ai để ý.

#### 2. Kiểm thử tích hợp tại các ranh giới dịch vụ thực tế

Test local chỉ chứng minh được phần logic của handler. Các ranh giới phải được kiểm thử trên hạ tầng đã deploy, nên tôi chạy các case trực tiếp trên stack `lingorise-dev`:

* **API Gateway -> Lambda:** request mapping, kích thước payload và CORS preflight cho origin trên Amplify.
* **Lambda -> Amazon RDS:** việc tái sử dụng `pg.Pool` giữa các warm invocation và hành vi khi cold start tạo pool mới.
* **Lambda -> Amazon S3:** sinh presigned URL, thời gian hết hạn và upload trực tiếp từ browser lên bucket.
* **Lambda -> AWS Systems Manager:** các parameter SecureString được resolve lúc deploy qua `{{resolve:ssm:/lingorise/dev/...}}`.
* **Amazon Cognito -> Lambda:** xác thực JWT bằng `aws-jwt-verify`, gồm cả token đã hết hạn và token từ user pool khác.

Có hai thứ không thể tái hiện được ở local: chữ ký presigned URL dưới IAM role đã deploy, và việc verify JWT với JWKS endpoint thật. Cả hai chỉ lỗi trên AWS và chỉ chẩn đoán được từ **Amazon CloudWatch Logs**. Tôi bỏ hẳn thói quen đoán nguyên nhân khi Lambda lỗi và chuyển sang đọc log stream trước.

#### 3. Các lỗi phát hiện và đã sửa

Ba lỗi tiêu biểu cho cả tuần:

* Submit lại một session đã submit tạo thêm một dòng score thứ hai. Cách sửa là dùng conditional update theo trạng thái session để chỉ ghi khi session còn đang mở.
* Việc quy đổi scaled band của TOEIC làm tròn sai ở biên, nên một raw score đúng ngay điểm biên bị tụt một band. Tôi sửa ở phần tra bảng quy đổi và thêm các giá trị biên thành test case tường minh.
* Một số lần cold start lỗi kết nối vì RDS pool được tạo trước khi credentials từ SSM được đọc. Sắp lại thứ tự khởi tạo là hết.

```bash
sam build && sam deploy --stack-name lingorise-dev --no-confirm-changeset
aws logs tail /aws/lambda/lingorise-dev-api --since 10m --follow
```

#### 4. Tối ưu hóa luồng xử lý AI và OCR

Luồng OCR là một job queue theo từng asset trong PostgreSQL, được một worker theo lịch drain bằng cách claim dòng với `FOR UPDATE SKIP LOCKED`. Kiểm thử cho thấy worker chỉ claim một job mỗi lần chạy, nên một scanned PDF 30 trang mất quá nhiều thời gian. Tôi tăng batch size khi claim và để worker tiếp tục drain khi vẫn còn thời gian trong lần invocation.

```sql
SELECT id, asset_key FROM ocr_jobs
WHERE status = 'pending'
ORDER BY created_at
FOR UPDATE SKIP LOCKED
LIMIT 5;
```

Về độ chính xác, tôi rasterize trang ở DPI cao hơn trước khi đưa vào **Tesseract.js**, load đồng thời language data tiếng Việt và tiếng Anh, đồng thời thêm bước preprocessing grayscale và tăng contrast. Nhờ vậy lượng chỉnh sửa tay trong review queue giảm rõ rệt với tài liệu scan.

Về khả năng chịu lỗi, tôi kiểm tra luồng fallback của AI provider: khi provider nội bộ không truy cập được, request sẽ chuyển sang OpenRouter thay vì trả lỗi. Tôi cố tình trỏ provider vào một endpoint đã chết và xác nhận request vẫn hoàn tất, với lần fallback được ghi vào log để có thể nhìn lại sau.

### Kết nối các dịch vụ AWS trong tuần này:

* **API Gateway + AWS Lambda:** Mọi test case chức năng đều đi qua REST stage nên request mapping và CORS được kiểm thử thật, không bị bỏ qua.
* **AWS Lambda + Amazon RDS:** Kiểm thử tích hợp xác nhận `pg.Pool` singleton tồn tại qua các warm invocation và khởi tạo đúng khi cold start.
* **AWS Lambda + Amazon S3:** Upload và download qua presigned URL được kiểm thử với execution role đã deploy, nơi duy nhất chữ ký là thật.
* **Amazon Cognito + AWS Lambda:** Việc verify JWT được kiểm thử với token hợp lệ, token hết hạn và token sai user pool để chắc chắn authorizer từ chối thay vì mặc định cho qua.
* **AWS Lambda + AWS Systems Manager:** SecureString trong Parameter Store được resolve lúc deploy, và chính điều đó làm lộ ra lỗi thứ tự khởi tạo gây lỗi cold start.
* **Amazon CloudWatch Logs + AWS SAM:** Sau mỗi `sam deploy` tôi tail log ngay, biến một lỗi HTTP 500 mơ hồ thành stack trace cụ thể.

### Kết quả đạt được tuần 9:

* Thực thi một test plan chức năng đã viết trước cho luồng người học và luồng nội dung quản trị, gồm cả exam session resume.
* Hoàn thành kiểm thử tích hợp trên stack `lingorise-dev` đã deploy ở cả năm ranh giới dịch vụ.
* Sửa lỗi trùng dòng score, lỗi làm tròn biên band TOEIC và lỗi khởi tạo pool khi cold start, rồi kiểm chứng lại bằng chính case đã fail.
* Tăng throughput OCR bằng cách claim job theo batch trong worker `FOR UPDATE SKIP LOCKED` thay vì xử lý một asset mỗi lần chạy.
* Cải thiện độ chính xác OCR nhờ tăng chất lượng rasterize trang, dùng đồng thời language data tiếng Việt và tiếng Anh, và preprocessing ảnh.
* Kiểm chứng luồng fallback của AI provider để khi provider gặp sự cố thì chất lượng phản hồi giảm chứ không làm chết request.

### Khó khăn gặp phải:

* Nhiều lỗi chỉ xuất hiện trên AWS, nên bộ test local tạo cảm giác an toàn sai và CloudWatch Logs mới là nơi debug thật sự.
* Tinh chỉnh độ chính xác OCR không có một cấu hình đúng duy nhất; DPI rasterize cao hơn giúp nhận diện tốt hơn nhưng tốn thời gian xử lý, nên batch size phải cân với timeout của worker.
* Tái hiện chủ động một Cognito token hết hạn và một sự cố provider đòi hỏi phải cố tình làm sai cấu hình, việc này mất thời gian hơn tưởng.

### Bài học rút ra và định hướng tiếp theo:

* Một test chỉ hữu ích khi kết quả mong đợi được viết ra trước khi chạy. Output sai nhưng trông hợp lý là kiểu lỗi sống sót qua kiểm thử.
* Ranh giới giữa các dịch vụ là nơi lỗi thật nằm. Unit test của handler vẫn pass trong khi presigned URL và verify JWT đều đang lỗi.
* Sang tuần tiếp theo tôi chuyển sang bàn giao và đóng gói: đưa source code lên GitHub, dọn dẹp cùng giới hạn tài nguyên AWS để tối ưu chi phí, và viết báo cáo thực tập cuối kỳ bằng tiếng Anh.
