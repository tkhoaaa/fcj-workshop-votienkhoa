---
title: "Worklog Tuần 7"
date: 2026-06-01
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Mục tiêu tuần 7:

* Thiết kế các entity cốt lõi của LingoRise cho courses, exams, questions, exam sessions và answers trên **Amazon RDS for PostgreSQL**.
* Xây dựng repository layer trên `pg.Pool` singleton để các handler **AWS Lambda** không còn viết SQL trực tiếp.
* Triển khai business logic cho exam engine, gồm tạo session và tính band score cho IELTS và TOEIC.
* Dựng nền tảng frontend trên **Next.js (App Router)**: cấu trúc thư mục, routing, state management và UI Vietnamese-first.

### Các công việc cần triển khai trong tuần này:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Thiết kế và migrate schema cốt lõi trên Amazon RDS for PostgreSQL: `courses`, `exams`, `exam_sections`, `questions`, `question_options` với UUID primary key và JSONB metadata. | 01/06/2026 | 01/06/2026 | [Amazon RDS for PostgreSQL](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_PostgreSQL.html) |
| **2** | Thêm các bảng session `exam_sessions` và `session_answers`, sau đó dựng repository layer trên `pg.Pool` singleton dùng chung cho mọi Lambda handler. | 02/06/2026 | 02/06/2026 | Repository dự án |
| **3** | Triển khai service cho exam engine: `POST /exams/start` tạo session từ bộ đề fixed đã tuyển chọn hoặc random pull từ question bank. | 03/06/2026 | 03/06/2026 | [AWS Lambda Node.js](https://docs.aws.amazon.com/lambda/latest/dg/lambda-nodejs.html) |
| **4** | Resolve asset URL của câu hỏi ngay tại service layer, chọn giữa public storage URL của **Amazon S3** và presigned URL hiệu lực 15 phút. | 04/06/2026 | 04/06/2026 | [S3 Presigned URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html) |
| **5** | Thiết lập cấu trúc Next.js App Router với route group cho learner và admin, React Query cho server state và các màn hình UI đầu tiên theo bản thiết kế. | 05/06/2026 | 05/06/2026 | [AWS Amplify Hosting](https://docs.aws.amazon.com/amplify/latest/userguide/welcome.html) |

### Cách thực hiện chi tiết:

#### 1. Thiết kế các entity cốt lõi trên Amazon RDS for PostgreSQL

Tôi bắt đầu từ data model vì toàn bộ service trong tuần đều dựa vào nó. Trên **Amazon RDS for PostgreSQL**, tôi thêm các file migration idempotent cho `courses`, `exams`, `exam_sections`, `questions`, `question_options`, rồi tới phần session gồm `exam_sessions` và `session_answers`. Mỗi migration được ghi vào bảng `_migrations` nên chạy lại runner trên database `lingorise-dev` vẫn an toàn.

Có hai quyết định định hình cả tuần:

* **UUID primary key** để có thể trả session id về browser mà không lộ số lượng bản ghi.
* Cột **JSONB metadata** trên `questions` và `exam_sessions`, nhờ đó đề Writing của IELTS và audio reference của TOEIC Part 3 mang cấu trúc khác nhau mà không cần đổi schema cho từng skill.

```sql
CREATE TABLE IF NOT EXISTS exam_sessions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL,
  exam_id UUID NOT NULL REFERENCES exams(id),
  status TEXT NOT NULL DEFAULT 'in_progress',
  metadata JSONB NOT NULL DEFAULT '{}'::jsonb,
  started_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

#### 2. Dựng repository layer trên pg.Pool singleton

Tuần trước, các API đầu tiên sau **Amazon API Gateway** truy vấn database ngay trong từng handler. Cách đó không mở rộng được, nên tôi tách ra repository layer: `courseRepository`, `examRepository`, `questionRepository` và `sessionRepository`. Tất cả repository dùng chung `pg.Pool` singleton được khởi tạo bên ngoài handler **AWS Lambda** và tái sử dụng qua các lần warm invocation.

Chỉ repository biết tới SQL. Handler giờ chỉ còn phần plumbing: parse event, verify claims từ Cognito JWT, gọi service, định dạng response. Vì esbuild bundle từng function riêng, việc giữ pool trong một module dùng chung cũng giúp số connection tới instance `db.t4g.micro` luôn dự đoán được.

#### 3. Triển khai business logic cho exam engine

Service layer là nơi chứa phần lõi sản phẩm. `POST /exams/start` xác định người gọi từ token đã verify, kiểm tra xem có session in-progress để resume hay không, nếu không thì tạo session mới. Một exam có thể gắn với bộ đề fixed đã tuyển chọn, khi đó câu hỏi giữ đúng thứ tự định trước, hoặc được sinh ra bằng random pull từ question bank có filter theo skill, level và section.

Khi danh sách câu hỏi đã có, service resolve từng asset reference. Ảnh minh họa công khai dùng storage URL thẳng của **Amazon S3**; những asset cần bảo vệ như audio listening thì nhận presigned URL hiệu lực 15 phút, đủ dài cho một section nhưng đủ ngắn để link bị copy ra ngoài không còn giá trị.

Phần scoring nằm cùng layer này: số câu đúng được map sang **Cambridge band** cho IELTS và scaled band cho TOEIC, nên frontend không bao giờ phải tự tính điểm.

#### 4. Dựng nền tảng Next.js App Router

Ở frontend, tôi thiết lập cấu trúc **Next.js** App Router mà **AWS Amplify Hosting** sẽ build. Tôi chia cây thư mục thành các route group: `(learner)` cho dashboard, trang course và exam runner, còn `(admin)` cho các màn hình content pipeline sẽ làm sau.

State management được chia rõ làm hai:

* **React Query** giữ server state, nhờ đó danh sách course và metadata của exam được cache và revalidate thay vì fetch lại mỗi lần render.
* **Local React state** giữ exam timer và answer sheet, vì chúng thay đổi từng giây và không được sinh network traffic.

Sau đó tôi dựng các component UI Vietnamese-first đầu tiên theo bản thiết kế: course card, section navigator và answer sheet grid. Nhãn hiển thị bằng tiếng Việt, còn identifier kỹ thuật giữ tiếng Anh, giúp phần copy tự nhiên với người học mà không phải đổi tên field của API.

### Kết nối các dịch vụ AWS trong tuần này:

* **Lambda + RDS for PostgreSQL:** `pg.Pool` singleton nằm ngoài handler nên warm invocation tái sử dụng connection thay vì mở mới.
* **Lambda + Amazon S3:** Exam service phát presigned URL 15 phút cho các question asset cần bảo vệ ngay khi start session.
* **API Gateway + Lambda:** Các route của exam engine được expose qua REST API dựng từ tuần trước, handler chỉ còn là adapter mỏng trên service layer.
* **Amazon Cognito + Lambda:** Claims trong JWT đã verify xác định learner nào đang được ghi session và answers.
* **Amplify Hosting + API Gateway:** App Next.js được tổ chức để mọi query server state trỏ về một API base URL lấy từ environment của Amplify.
* **CloudWatch Logs + Lambda:** Việc tạo session và scoring ghi một dòng log có cấu trúc cho mỗi request, giúp truy vết một band score sai rất nhanh.

### Kết quả đạt được tuần 7:

* Migrate xong toàn bộ schema cốt lõi cho courses, exams, questions, sessions và answers vào database PostgreSQL `lingorise-dev`.
* Thay SQL viết trực tiếp trong handler bằng repository layer dùng chung một `pg.Pool`.
* Triển khai `POST /exams/start` với khả năng resume, chọn câu hỏi theo bộ fixed hoặc random, và resolve asset URL.
* Triển khai Cambridge band scoring cho IELTS và scaled band scoring cho TOEIC tại một chỗ duy nhất.
* Thiết lập layout Next.js App Router, route group và cấu hình React Query cho frontend.
* Dựng các component UI Vietnamese-first đầu tiên trực tiếp từ bản thiết kế.

### Khó khăn gặp phải:

* Random question selection ban đầu trả về câu trùng giữa các section; tôi phải đưa logic loại trừ vào chính SQL query thay vì filter ở JavaScript.
* Việc quyết định asset nào cần presigned URL đòi hỏi cân nhắc, vì sign tất cả sẽ làm trang tải chậm hơn mà không tăng thêm mức bảo vệ.
* Giữ exam timer ở local state trong khi React Query refetch nền gây một bug re-render sớm làm reset đồng hồ đếm ngược.

### Bài học rút ra và định hướng tiếp theo:

* Tách repository và service đáng giá dù thêm file: khi business logic rời khỏi handler, việc test một quy tắc scoring không còn cần gọi API.
* Lưu phần khác biệt theo skill trong JSONB giúp schema ổn định trong lúc yêu cầu của IELTS và TOEIC còn thay đổi.
* Tuần tới tôi sẽ nối hai nửa lại với nhau: kết nối frontend Next.js với backend REST API và triển khai authentication cùng authorization đầu cuối bằng **JWT** và **Amazon Cognito**.
