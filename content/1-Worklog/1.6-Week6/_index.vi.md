---
title: "Worklog Tuần 6"
date: 2026-05-25
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Mục tiêu tuần 6:

* Bắt đầu code các tính năng cốt lõi của LingoRise trên phần scaffolding SAM đã dựng tuần trước.
* Xây dựng phần quản lý người dùng với **Amazon Cognito** user pool và bảng `users` tương ứng trong **Amazon RDS**.
* Triển khai phân quyền theo role trong handler và tách **IAM** execution role least-privilege cho từng Lambda.
* Đưa các API đầu tiên ra ngoài qua **Amazon API Gateway** với cấu hình CORS chạy được cho frontend trên Amplify.

### Các công việc cần triển khai trong tuần này:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Tạo Cognito user pool và app client cho `lingorise-dev`, sau đó test luồng sign-up, confirm và sign-in. | 25/05/2026 | 25/05/2026 | [Amazon Cognito User Pools](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-identity-pools.html) |
| **2** | Thêm migration bảng `users` trong RDS ánh xạ theo Cognito subject và viết handler tự tạo user row local ở request có xác thực đầu tiên. | 26/05/2026 | 26/05/2026 | [Amazon RDS for PostgreSQL](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_PostgreSQL.html) |
| **3** | Triển khai kiểm tra role learner, content manager, admin và siết từng IAM execution role của Lambda về đúng tài nguyên nó dùng. | 27/05/2026 | 27/05/2026 | [IAM Least Privilege](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) |
| **4** | Viết các handler xử lý dữ liệu đầu tiên để đọc và ghi dữ liệu course cùng question bank qua `pg.Pool` singleton. | 28/05/2026 | 28/05/2026 | Repository dự án |
| **5** | Map REST resources và methods sang Lambda trong API Gateway, deploy stage `dev` và cấu hình CORS cho domain Amplify. | 29/05/2026 | 29/05/2026 | [API Gateway CORS](https://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-cors.html) |

### Cách thực hiện chi tiết:

#### 1. Quản lý người dùng tách giữa Cognito và database

Tôi tạo **Amazon Cognito** user pool cho `lingorise-dev` với email làm sign-in alias, sau đó tạo app client không dùng secret để frontend **Next.js** gọi trực tiếp từ browser. Tôi chạy tay toàn bộ luồng trước: sign-up, nhận confirmation code qua email, sign-in, rồi mở ID token ra xem nội dung.

Cognito cho tôi một subject (`sub`) ổn định nhưng đó không phải chỗ tốt để lưu application state. Vì vậy tôi thêm bảng `users` trong **Amazon RDS for PostgreSQL** ánh xạ theo subject đó và giữ những gì sản phẩm cần:

```sql
CREATE TABLE IF NOT EXISTS users (
  id            BIGSERIAL PRIMARY KEY,
  cognito_sub   TEXT UNIQUE NOT NULL,
  email         TEXT NOT NULL,
  display_name  TEXT,
  role          TEXT NOT NULL DEFAULT 'learner',
  is_active     BOOLEAN NOT NULL DEFAULT TRUE,
  created_at    TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Điểm quan trọng là cách chia trách nhiệm: Cognito quản lý identity và credentials, còn database quản lý role, active flag, profile và sau này là liên kết subscription. Một helper `ensureUser` dùng chung sẽ chạy ở request có xác thực đầu tiên và insert row theo kiểu idempotent, nên mỗi Cognito user luôn có đúng một bản ghi local tương ứng.

#### 2. Phân quyền: kiểm tra role trong code, least privilege trong IAM

Phân quyền tuần này diễn ra ở hai lớp. Trong handler, sau khi `aws-jwt-verify` xác thực token, tôi đọc user row local và quy `role` về một trong ba giá trị: `learner`, `content_manager` hoặc `admin`. Một wrapper nhỏ `requireRole()` trả 403 trước khi bất kỳ query nào chạy, nên một account đã bị vô hiệu hóa hoặc hạ quyền vẫn không chạm được vào question bank dù token còn hiệu lực.

Ở lớp hạ tầng, tôi bỏ cách dùng chung một execution role cho mọi function. Mỗi **AWS Lambda** giờ có **IAM** role riêng trong SAM template với đúng những gì nó cần: VPC networking cho các function nói chuyện với RDS, `s3:GetObject` giới hạn trong một prefix cho các function đọc asset, và `ssm:GetParameter` chỉ trong `/lingorise/dev/*`. Việc tự viết các policy này làm luồng dữ liệu hiện ra rõ ràng, vì thiếu quyền nào là lập tức thấy `AccessDenied` trong **CloudWatch Logs**.

#### 3. Các service xử lý dữ liệu đầu tiên trên question bank

Khi identity và permission đã xong, tôi viết những handler thật đầu tiên: list courses, đọc một course kèm các section, list questions có pagination, và tạo hoặc cập nhật question. Tất cả đều đi qua `pg.Pool` singleton khởi tạo bên ngoài thân handler, nên một Lambda đang warm sẽ tái sử dụng connection thay vì mở mới mỗi lần invoke. Chi tiết này đáng chú ý trên `db.t4g.micro`, nơi connection limit khá thấp.

Các đường ghi đều chạy trong transaction và lưu lại người thực hiện thay đổi bằng user id local, đây là bước đầu của audit log mà content pipeline sẽ cần về sau. Ảnh của question vẫn nằm trên **Amazon S3**; database chỉ lưu object key và handler trả về key đó để frontend tự resolve.

#### 4. API Gateway resources, deploy stage và CORS

Tôi khai báo các REST resource trong SAM template để **Amazon API Gateway** nằm trong source control: `/courses`, `/courses/{courseId}`, `/questions` và `/me`. Mỗi method map tới Lambda qua proxy integration, và toàn bộ API deploy vào stage `dev` ở `ap-southeast-1`.

CORS là phần mất thời gian nhất. Domain Amplify là origin khác với API, nên browser luôn gửi preflight `OPTIONS` trước mỗi request có header `Authorization`. Tôi khai báo allowed origin cụ thể thay vì `*`, cho phép header `Authorization` và `Content-Type`, và bảo đảm `OPTIONS` không bị authorizer chặn. Kiểm tra từ terminal nhanh hơn nhiều so với đoán trong browser:

```bash
curl -i -X OPTIONS "$API_URL/questions" \
  -H "Origin: https://dev.lingorise.app" \
  -H "Access-Control-Request-Method: POST" \
  -H "Access-Control-Request-Headers: authorization,content-type"
```

### Kết nối các dịch vụ AWS trong tuần này:

* **Cognito + Lambda:** ID token do user pool phát hành được verify trong handler bằng `aws-jwt-verify` trước khi chạy business logic.
* **Cognito + RDS:** Cognito `sub` liên kết mỗi identity với một row trong bảng `users` local đang giữ role và active state.
* **Lambda + IAM:** Mỗi function nhận một execution role riêng, giới hạn đúng các tài nguyên RDS, S3 và SSM mà nó dùng.
* **API Gateway + Lambda:** REST resources và methods được map qua proxy integration rồi release thành stage `dev`.
* **API Gateway + Amplify Hosting:** CORS được cấu hình cho origin Amplify để app Next.js gọi API kèm header `Authorization`.
* **Lambda + S3 + RDS:** Bản ghi question nằm trong PostgreSQL còn ảnh nằm trên S3, nối với nhau bằng object key được lưu lại.

### Kết quả đạt được tuần 6:

* Luồng sign-up, confirm và sign-in chạy được với Cognito user pool `lingorise-dev`.
* Bảng `users` cùng helper provisioning idempotent giữ đồng bộ giữa Cognito identity và application role.
* Phân quyền theo role được thực thi trong handler cho learner, content manager và admin.
* IAM execution role riêng cho từng function, thay cho role dùng chung trước đó.
* Các endpoint xử lý dữ liệu đầu tiên cho course và question bank, chạy trên connection RDS được pool lại.
* Stage `dev` của API Gateway đã deploy và frontend gọi được qua khác origin.

### Khó khăn gặp phải:

* Lỗi 401 từ authorizer trông giống hệt lỗi CORS trên browser console, vì response lỗi không mang theo CORS header. Tôi mất thời gian sửa sai vấn đề đến khi kiểm tra raw response bằng `curl`.
* Tách một execution role thành nhiều role làm hai function fail lúc deploy; đọc các dòng `AccessDenied` trong CloudWatch Logs là cách duy nhất đáng tin để biết mỗi handler thật sự cần gì.
* Việc quyết định cái gì nên nằm trong Cognito attributes và cái gì nên nằm trong database khiến tôi phải rà lại schema một lượt nữa.

### Bài học rút ra và định hướng tiếp theo:

* Identity và authorization là hai bài toán khác nhau. Cognito trả lời caller là ai, còn database và code trả lời caller được phép làm gì.
* Viết IAM least-privilege ngay khi tính năng còn mới thì dễ hơn nhiều so với sửa lại về sau, và nó tự mô tả luôn luồng dữ liệu.
* Tuần tiếp theo tôi sẽ đi sâu hơn ở cả hai phía: backend với core services, entities, repositories và business logic; frontend với cấu trúc project, routing, state management và UI.
