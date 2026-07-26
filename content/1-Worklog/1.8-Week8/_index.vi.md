---
title: "Worklog Tuần 8"
date: 2026-06-08
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Mục tiêu tuần 8:

* Kết nối frontend **Next.js** của LingoRise với các endpoint REST trên **API Gateway** qua một API client có kiểu dữ liệu rõ ràng.
* Hiển thị dữ liệu thật từ **Amazon RDS for PostgreSQL** trên các màn hình course catalog, exam catalog và exam session.
* Hiện thực xác thực và phân quyền bằng JWT của **Amazon Cognito**, verify ngay trong **AWS Lambda**.
* Bảo đảm luồng resume exam session không mất câu trả lời khi người dùng reload trang.

### Các công việc cần triển khai trong tuần này:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Xây dựng typed API client trong Next.js, tập trung base URL theo từng stage và gắn `Authorization: Bearer <jwt>` cho mọi request. | 08/06/2026 | 08/06/2026 | [Amazon API Gateway](https://aws.amazon.com/api-gateway/) |
| **2** | Nối các React Query hook cho course catalog và exam catalog kèm trạng thái availability, cùng các trạng thái loading, error và empty. | 09/06/2026 | 09/06/2026 | Repository dự án |
| **3** | Verify JWT của Cognito trong Lambda bằng `aws-jwt-verify` với JWKS của user pool, giữ dev-token fallback cho môi trường local. | 10/06/2026 | 10/06/2026 | [Amazon Cognito Docs](https://docs.aws.amazon.com/cognito/latest/developerguide/amazon-cognito-user-pools-using-tokens-verifying-a-jwt.html) |
| **4** | Ánh xạ subject trong token đã verify sang bản ghi trong bảng `users` và kiểm tra role cùng trạng thái active trong một authorizer helper dùng chung. | 11/06/2026 | 11/06/2026 | [IAM and Authorization](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-control-access-to-api.html) |
| **5** | Tích hợp luồng start/resume exam session và màn hình review từng câu, sau đó debug lỗi 401 do token expiry và clock skew. | 12/06/2026 | 12/06/2026 | Ghi chú tự học |

### Cách thực hiện chi tiết:

#### 1. Một typed API client giữa Next.js và API Gateway

Trước tuần này frontend vẫn dùng fixture cục bộ. Tôi thay toàn bộ bằng một wrapper `apiFetch` đọc invoke URL của **API Gateway** từ `NEXT_PUBLIC_API_BASE_URL`, tự chèn `Authorization: Bearer <jwt>` lấy từ session **Amazon Cognito** hiện tại, và chuẩn hóa lại error envelope mà các Lambda handler đã trả về.

Việc tập trung client giúp mọi màn hình dùng chung một cách xử lý stage, timeout và error mapping. Khi API chuyển từ port SAM local sang URL của stage `lingorise-dev`, tôi chỉ cần đổi một biến môi trường trên **AWS Amplify Hosting**.

```ts
const res = await fetch(`${BASE}${path}`, {
  ...init,
  headers: { "Content-Type": "application/json", Authorization: `Bearer ${token}`, ...init?.headers },
});
if (!res.ok) throw new ApiError(res.status, (await res.json()).message);
return res.json() as Promise<T>;
```

#### 2. React Query hook cho catalog, session và review

Trên nền client đó tôi thêm các hook React Query: `useCourses`, `useExams`, `useSession` và `useSessionReview`. Exam catalog không chỉ là một danh sách, vì mỗi đề thi có trạng thái availability phụ thuộc vào entitlement được backend resolve từ bảng `user_subscriptions`. Tôi map các trạng thái đó thành ba cách hiển thị: available, locked kèm gợi ý nâng cấp gói, và chưa publish.

Mỗi hook giờ đều có nhánh loading, error và empty rõ ràng. Trạng thái empty khá quan trọng với một UI Vietnamese-first, vì trước đây danh sách đề trống trông giống như trang bị lỗi hơn là một khóa học chưa có đề nào được publish.

#### 3. Verify JWT của Cognito trong Lambda

Phân quyền được xử lý trong handler **AWS Lambda** thay vì ở lớp edge, bởi mỗi request còn cần tới bản ghi user trong database. Tôi dùng `aws-jwt-verify` để verify access token với JWKS của Cognito user pool, kiểm tra issuer, audience, `token_use` và thời hạn. Pool id cùng client id được lấy từ **AWS Systems Manager Parameter Store** ở thời điểm deploy, nên không có identifier nào bị hardcode trong SAM template.

Sau khi verify, authorizer helper dùng chung ánh xạ `sub` sang bản ghi tương ứng trong bảng `users` và đọc `role` cùng `is_active`. Nhờ vậy ranh giới rất rõ: token thiếu hoặc không hợp lệ trả về 401, còn token hợp lệ nhưng user không có role admin hoặc tài khoản đã bị vô hiệu thì trả về 403. Môi trường local vẫn giữ dev-token fallback, và fallback này chỉ bật khi stage không phải production.

#### 4. Start, resume exam session và review từng câu

Luồng session là phần khó tích hợp nhất. `POST /sessions` vừa tạo lượt làm mới vừa trả về lượt đang in-progress, nên frontend xử lý start và resume như cùng một lời gọi. Câu trả lời được lưu theo từng câu, và khi mount lại client rehydrate từ response của server thay vì từ state cục bộ. Reload trang giữa lúc đang thi giờ khôi phục đúng chỉ số câu hỏi, các đáp án đã lưu và thời gian còn lại tính từ `started_at` phía server.

Màn hình review đọc payload từng câu gồm đáp án đúng, phần giải thích và transcript listening, tất cả lấy từ **Amazon RDS for PostgreSQL** cùng hình ảnh được sign từ **Amazon S3**. Dữ liệu review chỉ trả về sau khi lượt thi đã submit, và điều kiện này do handler kiểm tra chứ không phải do UI.

### Kết nối các dịch vụ AWS trong tuần này:

* **Next.js trên Amplify Hosting + API Gateway:** Frontend gọi URL stage REST qua một typed client cấu hình theo từng môi trường.
* **Amazon Cognito + AWS Lambda:** Access token được verify ngay trong handler bằng `aws-jwt-verify` với JWKS của user pool.
* **Cognito + Amazon RDS for PostgreSQL:** Subject trong token đã verify được ánh xạ sang bản ghi `users` để kiểm tra role và trạng thái active.
* **API Gateway + Lambda + RDS:** Các request catalog, session và review trả về dữ liệu thật trong database thay cho fixture frontend.
* **Lambda + Amazon S3:** Hình ảnh câu hỏi và file listening được cấp cho màn hình review qua signed URL.
* **Parameter Store + AWS SAM:** Pool id và client id của Cognito được resolve ở thời điểm deploy vào stack `lingorise-dev`.

### Kết quả đạt được tuần 8:

* Thay toàn bộ fixture frontend bằng dữ liệu thật đi qua chuỗi API Gateway - Lambda - RDS.
* Hoàn thiện xác thực end-to-end với token Cognito verify phía server và protected route ở phía client.
* Chuẩn hóa ngữ nghĩa 401 và 403 nhất quán trên mọi handler nhờ một authorizer helper dùng chung.
* Luồng resume exam session hoạt động ổn định qua các lần reload trang, gồm cả thời gian còn lại.
* Bổ sung trạng thái loading, error và empty cho từng React Query hook nên không màn hình nào còn render trắng.

### Khó khăn gặp phải:

* Một loạt request bị 401 nhưng chỉ trên stage đã deploy. Token vẫn hợp lệ, song đồng hồ máy local của tôi lệch nhanh khoảng hai phút, khiến token vừa phát hành bị coi như chưa có hiệu lực. Tôi chỉnh lại đồng hồ, đặt thêm một mức tolerance nhỏ trong verifier, rồi bổ sung luồng refresh-and-retry cho các token hết hạn thật.
* Preflight của browser bắt đầu fail sau khi tôi thêm một custom header cho client version, vì cấu hình CORS trên API Gateway mới chỉ liệt kê `Authorization` trong `Access-Control-Allow-Headers`.
* Quyết định đặt phân quyền ở đâu phải làm lại vài lần, vì dùng Cognito authorizer ngay tại gateway thì vẫn phải truy vấn database để lấy role và entitlement.

### Bài học rút ra và định hướng tiếp theo:

* Verify token ngay cạnh truy vấn dữ liệu giúp mọi quyết định phân quyền nằm chung một chỗ, đổi lại thêm vài millisecond cho mỗi request.
* Phân biệt rõ 401 và 403 tiết kiệm nhiều thời gian cho frontend, vì client tự biết nên refresh session hay hiển thị gợi ý nâng cấp gói.
* Sang tuần tiếp theo, tôi sẽ chuyển sang phần testing: kiểm thử chức năng và kiểm thử tích hợp trên frontend, backend Lambda và các dịch vụ AWS phía sau, sửa các bug mà bộ test phát hiện và tối ưu pipeline xử lý AI.
