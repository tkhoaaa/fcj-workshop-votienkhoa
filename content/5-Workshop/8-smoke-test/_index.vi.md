---
title : "Kiểm thử & Vận hành"
date : 2024-01-01
weight : 8
chapter : false
pre : " <b> 8. </b> "
---

#### Tổng quan

Stack đã được triển khai và frontend đã được nối vào nó. Trước khi coi như đã triển khai xong, bạn muốn có bằng chứng rằng toàn bộ đường đi của request hoạt động từ đầu đến cuối: API Gateway định tuyến request, Lambda chạy, authorizer Cognito chấp nhận (hoặc từ chối đúng cách) token, và RDS trả về dữ liệu. Chương này hướng dẫn một bài smoke test trên API đang chạy, sau đó là một hành trình người dùng thực sự qua ứng dụng được host trên Amplify trong khi bạn theo dõi log. Chương khép lại với các phần vận hành day-two: ba script bảo trì, cách lên lịch cho chúng, và một bức tranh chi phí sơ bộ kèm cảnh báo AWS Budgets để không bao giờ bị bất ngờ vì hoá đơn tăng vọt.

Hãy chuẩn bị sẵn URL gốc của API và URL Amplify mà bạn đã ghi lại từ các chương trước. Trong các câu lệnh bên dưới, `$ApiUrl` là URL invoke của API Gateway cho `lingorise-dev` (ví dụ `https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/dev`).

#### Smoke test API

Bắt đầu bằng phép kiểm tra rẻ nhất có thể: gọi một route được bảo vệ mà không có token và xác nhận bạn nhận được `401`. Sau đó gọi một route công khai và xác nhận `200`. Chỉ riêng hai lời gọi này đã chứng minh rằng API Gateway, Lambda, và authorizer đều đã được nối đúng.

PowerShell:

```powershell
$ApiUrl = "https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/dev"

# Protected route, no token → expect 401
curl.exe -i "$ApiUrl/auth/me"

# Public route → expect 200
curl.exe -i "$ApiUrl/courses"
```

Bash (cùng các lời gọi):

```bash
ApiUrl="https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/dev"

# Protected route, no token → expect 401
curl -i "$ApiUrl/auth/me"

# Public route → expect 200
curl "$ApiUrl/courses"
```

![Kết quả smoke test curl hiển thị 401 và 200](/images/Workshop-LingoRise/8-smoke-test/smoke-test-curl.png)

{{% notice tip %}}
Một `401 Unauthorized` từ `/auth/me` chính là **phản hồi khoẻ mạnh được mong đợi**, không phải lỗi. Nó chứng minh toàn bộ chuỗi đang hoạt động: API Gateway đã nhận request, gọi Lambda, và authorizer Cognito đã từ chối đúng cách vì không có Bearer token hợp lệ. Nếu thay vào đó bạn nhận được `403`, `500`, hoặc lỗi kết nối, nghĩa là phần nối dây đang có vấn đề — hãy kiểm tra outputs của stack và log CloudWatch trước khi đi tiếp.
{{% /notice %}}

Bây giờ hãy chứng minh luồng xác thực hoạt động với thông tin đăng nhập thật. Đăng ký một người dùng, sau đó đăng nhập và xác nhận bạn nhận lại được một JWT.

PowerShell:

```powershell
# Register a new user
curl.exe -i -X POST "$ApiUrl/auth/register" `
  -H "Content-Type: application/json" `
  -d '{\"email\":\"smoke@example.com\",\"password\":\"<your-password>\",\"name\":\"Smoke Test\"}'

# Log in → expect 200 with a token in the body
curl.exe -s -X POST "$ApiUrl/auth/login" `
  -H "Content-Type: application/json" `
  -d '{\"email\":\"smoke@example.com\",\"password\":\"<your-password>\"}'
```

Bash:

```bash
# Register a new user
curl -i -X POST "$ApiUrl/auth/register" \
  -H "Content-Type: application/json" \
  -d '{"email":"smoke@example.com","password":"<your-password>","name":"Smoke Test"}'

# Log in → expect 200 with a token in the body
curl -s -X POST "$ApiUrl/auth/login" \
  -H "Content-Type: application/json" \
  -d '{"email":"smoke@example.com","password":"<your-password>"}'
```

Phần body của phản hồi đăng nhập chứa trường `token` — đó chính là JWT. Sao chép nó và xác nhận rằng route được bảo vệ giờ trả về `200` kèm bản ghi người dùng của bạn thay vì `401`:

```bash
TOKEN="<paste>"
curl -i "$ApiUrl/auth/me" -H "Authorization: Bearer $TOKEN"
```

#### Đi qua một hành trình người dùng

Curl chứng minh phần đường ống; một hành trình thật chứng minh sản phẩm. Mở URL Amplify trong trình duyệt và đi qua đúng con đường mà một học viên sẽ đi, giữ log CloudWatch mở ở một cửa sổ thứ hai để bạn có thể thấy từng handler chạy.

1. **Đăng nhập** bằng người dùng `smoke@example.com` bạn vừa tạo. Ứng dụng sẽ lưu token và đưa bạn tới dashboard.
2. **Duyệt khoá học** — danh sách khoá học được phục vụ bởi cùng route công khai `/courses` mà bạn đã smoke-test.
3. **Bắt đầu một bài thi** — chọn một bài thi thử và bắt đầu. Thao tác này chạy `POST /exams/start`, vốn dựng một session, chọn câu hỏi, và phân giải URL tài nguyên, nên nó chạm cả RDS và S3 trong một lời gọi.

Trong lúc bạn click qua các bước, hãy theo dõi (tail) các log group của Lambda. Mọi hàm đều ghi log dưới `/aws/lambda/lingorise-dev-*`:

PowerShell:

```powershell
aws logs tail /aws/lambda/lingorise-dev-auth-login --follow `
  --region ap-southeast-1 --profile lingorise-dev
```

Bash:

```bash
aws logs tail /aws/lambda/lingorise-dev-exams-start --follow \
  --region ap-southeast-1 --profile lingorise-dev
```

![Theo dõi log CloudWatch trong hành trình người dùng](/images/Workshop-LingoRise/8-smoke-test/cloudwatch-logs.png)

{{% notice note %}}
Nếu một trang bị treo hoặc trả về lỗi, dòng log CloudWatch của handler đó gần như luôn chỉ ra nguyên nhân — một tham số SSM bị thiếu, kết nối cơ sở dữ liệu bị timeout (thường là do quy tắc security group), hoặc một exception không được xử lý kèm stack trace. Hãy sửa nguyên nhân gốc thay vì thử lại một cách mù quáng.
{{% /notice %}}

#### Script vận hành

LingoRise đi kèm ba script bảo trì giúp giữ dữ liệu gọn gàng theo thời gian. Chạy chúng từ thư mục dự án backend với profile `lingorise-dev` đang được kích hoạt. Mỗi script đều an toàn để chạy theo yêu cầu, và mỗi script đều là ứng viên tốt cho một job được lên lịch khi bạn đã vào trạng thái ổn định.

```bash
# Process pending OCR jobs (question-image extraction)
npm run worker:ocr

# Remove stale import drafts left behind by incomplete uploads
npm run cleanup:import-drafts

# Expire subscriptions whose paid period has ended
npm run subscriptions:expire
```

{{% notice info %}}
Chạy theo yêu cầu là ổn cho một workshop, nhưng trong môi trường `dev` hay production thực tế bạn sẽ lên lịch cho chúng. Cách làm thuần serverless là một quy tắc cron của **Amazon EventBridge** để gọi một Lambda bọc lấy từng script — ví dụ, chạy `subscriptions:expire` mỗi ngày một lần và `worker:ocr` vài phút một lần. Biểu thức cron của EventBridge trông như `cron(0 17 * * ? *)` (hằng ngày lúc 17:00 UTC) hoặc `rate(5 minutes)`. Việc nối các lịch chạy là tuỳ chọn cho workshop này; điểm mấu chốt là biết ba job này tồn tại.
{{% /notice %}}

#### Chi phí & giám sát

Vì stack là serverless và co giãn về 0 khi rảnh, một môi trường `dev` dùng nhẹ sẽ rất rẻ. Bức tranh chi phí hàng tháng sơ bộ:

| Dịch vụ | Chi phí ước tính mỗi tháng (dev) |
|---|---|
| Lambda + API Gateway | ~$0–2 (co giãn về 0 khi rảnh) |
| RDS PostgreSQL (`db.t4g.micro`) | ~$13–15 (chạy 24/7) |
| S3 + CloudFront | ~$1–3 |
| Cognito | ~$0 (nằm gọn trong free tier) |
| CloudWatch Logs | ~$1–2 |
| **Tổng** | **~$18–25 / tháng** |

Khoản lớn nhất là RDS, vì cơ sở dữ liệu chạy suốt ngày đêm trong khi mọi thứ khác rảnh ở mức 0. Nếu bạn để stack chạy giữa các buổi làm, đó chính là chi phí sẽ tích luỹ.

Hãy đặt một hàng rào bảo vệ để hoá đơn không bao giờ khiến bạn bất ngờ. Tạo một cảnh báo AWS Budgets gửi email cho bạn khi chi tiêu vượt ngưỡng:

PowerShell:

```powershell
aws budgets create-budget `
  --account-id <accountId> `
  --budget '{\"BudgetName\":\"lingorise-dev-monthly\",\"BudgetLimit\":{\"Amount\":\"30\",\"Unit\":\"USD\"},\"TimeUnit\":\"MONTHLY\",\"BudgetType\":\"COST\"}' `
  --notifications-with-subscribers '[{\"Notification\":{\"NotificationType\":\"ACTUAL\",\"ComparisonOperator\":\"GREATER_THAN\",\"Threshold\":80},\"Subscribers\":[{\"SubscriptionType\":\"EMAIL\",\"Address\":\"<your-email>\"}]}]' `
  --profile lingorise-dev
```

Bash:

```bash
aws budgets create-budget \
  --account-id <accountId> \
  --budget '{"BudgetName":"lingorise-dev-monthly","BudgetLimit":{"Amount":"30","Unit":"USD"},"TimeUnit":"MONTHLY","BudgetType":"COST"}' \
  --notifications-with-subscribers '[{"Notification":{"NotificationType":"ACTUAL","ComparisonOperator":"GREATER_THAN","Threshold":80},"Subscribers":[{"SubscriptionType":"EMAIL","Address":"<your-email>"}]}]' \
  --profile lingorise-dev
```

![Cấu hình cảnh báo AWS Budgets](/images/Workshop-LingoRise/8-smoke-test/budget-alarm.png)

#### Kiểm tra

Bạn hoàn tất chương này khi: `GET /auth/me` trả về `401` khi không có token và `200` khi có token, `GET /courses` trả về `200`, bạn đã đi qua đăng nhập → khoá học → bắt đầu bài thi trong ứng dụng Amplify và thấy các handler tương ứng chạy trong `/aws/lambda/lingorise-dev-*`, và cảnh báo AWS Budgets của bạn đã được tạo.

{{% notice info %}}
Tiếp theo: **[9. Dọn dẹp]** — gỡ bỏ mọi thứ bạn đã cấp phát để đồng hồ đo RDS ngừng chạy và không còn tài nguyên sót lại tiếp tục tính tiền sau workshop.
{{% /notice %}}
