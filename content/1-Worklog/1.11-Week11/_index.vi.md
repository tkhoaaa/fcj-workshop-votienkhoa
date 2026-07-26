---
title: "Worklog Tuần 11"
date: 2026-06-29
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---
### Mục tiêu tuần 11:

* Tổng hợp các chỉ số trải nghiệm người dùng và tỷ lệ lỗi ghi nhận trong quá trình thử nghiệm thực tế LingoRise.
* Tách phần vướng mắc về sản phẩm ra khỏi phần lỗi kỹ thuật để mỗi vấn đề có hướng xử lý rõ ràng.
* Tinh chỉnh lại frontend **Next.js** dựa trên phản hồi của người thử nghiệm, tập trung vào điều hướng bài thi, loading state và cách diễn đạt tiếng Việt.
* Sửa các lỗi phát sinh ngoài dự kiến trong quá trình thử nghiệm và kiểm chứng lại từng bản sửa bằng chính chỉ số đã đo.

### Các công việc cần triển khai trong tuần này:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Thu thập dữ liệu thử nghiệm từ nhóm tester: tỷ lệ hoàn thành việc bắt đầu và kết thúc một bài thi thử, thời gian tới câu hỏi đầu tiên và các điểm bỏ dở. | 29/06/2026 | 29/06/2026 | Ghi chú tự học |
| **2** | Lấy góc nhìn kỹ thuật từ **Amazon CloudWatch**: số lượng 4xx/5xx của API Gateway, p95 duration của Lambda và các dòng log lỗi trong log group. | 30/06/2026 | 30/06/2026 | [Amazon CloudWatch Metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/working_with_metrics.html) |
| **3** | Phân tích toàn bộ dữ liệu, chia phát hiện thành nhóm vướng mắc sản phẩm và nhóm lỗi kỹ thuật, rồi xếp hạng theo số tester bị ảnh hưởng. | 01/07/2026 | 01/07/2026 | Repository dự án |
| **4** | Tinh chỉnh UI React: điều hướng bài thi và answer sheet rõ hơn, loading và error state thật, layout mobile và viết lại phần copy tiếng Việt. | 02/07/2026 | 02/07/2026 | [Next.js App Router](https://nextjs.org/docs/app) |
| **5** | Sửa các lỗi phát sinh ngoài dự kiến và đo lại tỷ lệ lỗi sau khi deploy lên stack `lingorise-dev`. | 03/07/2026 | 03/07/2026 | [AWS SAM Docs](https://docs.aws.amazon.com/serverless-application-model/) |

### Cách thực hiện chi tiết:

#### 1. Thu thập chỉ số trải nghiệm từ đợt thử nghiệm thực tế

Nhóm tester làm trọn bài thi thử trên thiết bị của họ, nên dữ liệu sát với cách dùng thật hơn nhiều so với lúc tôi tự kiểm tra. Với từng tester, tôi ghi lại việc họ có bắt đầu được bài thi không, có làm xong không, ứng dụng mất bao lâu để hiện câu hỏi đầu tiên và họ dừng ở đâu nếu không hoàn thành.

Những chỉ số quan trọng nhất:

* tỷ lệ hoàn thành của luồng bắt đầu bài thi và luồng nộp bài
* thời gian tới câu hỏi đầu tiên, tính từ lúc bấm "Bắt đầu" đến khi câu hỏi hiển thị
* thời điểm trong phiên thi mà tester bỏ dở
* phản hồi dạng văn bản tiếng Việt, giữ nguyên câu chữ để tôi không tự làm nhẹ đi vấn đề

Đó là phần góc nhìn sản phẩm. Góc nhìn kỹ thuật phải lấy từ AWS.

#### 2. Đọc tỷ lệ lỗi từ CloudWatch

**Amazon CloudWatch** đã có sẵn nửa còn lại của câu chuyện. Tôi xem metric `4XXError` và `5XXError` trên stage của **Amazon API Gateway**, p95 `Duration` của từng function **AWS Lambda**, và các dòng lỗi thô trong log group. Một truy vấn Logs Insights trên function bài thi là cách nhanh nhất để biết route nào thực sự đang lỗi:

```sql
fields @timestamp, level, route, err.name, err.message
| filter level = "error"
| stats count(*) as errors by route, err.name
| sort errors desc
| limit 20
```

Tỷ lệ lỗi API đo được thấp về giá trị tuyệt đối, nhưng không phân bố đều. Gần như toàn bộ nằm ở hai route: start phiên thi và fetch asset cho audio phần listening. P95 của Lambda cho thấy đúng điều đó, function start phiên thi cao hơn hẳn phần còn lại, một phần do cold start và một phần do query chậm.

#### 3. Tách vướng mắc sản phẩm khỏi lỗi kỹ thuật

Khi ghép hai nguồn dữ liệu, phần phân tích trở nên rõ ràng. Vướng mắc sản phẩm là những chỗ tester chần chừ hoặc đi sai đường mà log không ghi lỗi nào. Lỗi kỹ thuật là những chỗ có log entry hoặc latency tăng vọt trùng đúng thời điểm tester bỏ dở.

Danh sách sau khi xếp hạng:

* **Vướng mắc sản phẩm:** answer sheet không cho thấy rõ câu nào còn chưa trả lời; chữ "Nộp bài" đứng cạnh "Lưu tạm" gây nhập nhằng; trên mobile, đoạn reading và phần câu hỏi tranh nhau không gian; loading state chỉ là một spinner trơ không kèm ngữ cảnh.
* **Lỗi kỹ thuật:** query start phiên thi quét `exam_questions` mà không có index phù hợp; một cache key của **React Query** thiếu session id nên khi resume bài thi đôi lúc render lại lần làm trước; presigned URL cho audio listening hết hạn giữa bài với các section dài; phần kiểm tra entitlement trên `user_subscriptions` chạy lại ở mọi request trong một trang thay vì một lần.

Hai tester báo cùng một triệu chứng "audio tự dừng giữa bài" mà tôi chưa từng tái hiện được, và presigned URL hết hạn chính là nguyên nhân.

#### 4. Tinh chỉnh frontend và sửa lỗi

Về UI, tôi làm lại answer sheet để câu chưa trả lời khác biệt hẳn về thị giác và chạm một lần là tới, đổi tên hai hành động lưu sang tiếng Việt không gây nhập nhằng, thêm skeleton state cho khung bài thi để người dùng biết đang tải gì, thêm nút thử lại vào error state thay cho một thông báo bế tắc, và xếp đoạn reading lên trên danh sách câu hỏi ở breakpoint mobile. Tất cả deploy qua **AWS Amplify Hosting** ngay khi push, nên mỗi vòng phản hồi lên production chỉ trong vài phút.

Về kỹ thuật, mỗi lỗi có một bản sửa cụ thể:

* thêm composite index để query start phiên thi không còn quét bảng, áp dụng dưới dạng migration idempotent được ghi nhận trong `_migrations`:

```sql
CREATE INDEX IF NOT EXISTS idx_exam_questions_exam_order
  ON exam_questions (exam_id, question_order);
```

* sửa cache key của React Query để chứa session id, xử lý được việc render lại lần làm bài cũ
* nâng TTL của presigned URL trên **Amazon S3** cho asset listening để URL sống lâu hơn section dài nhất
* memoize phần resolve entitlement theo request, để một lần tải trang chỉ truy vấn `user_subscriptions` một lần

Sau khi deploy lại backend bằng **AWS SAM** lên stack `lingorise-dev`, tôi đo lại đúng những chỉ số cũ. Tỷ lệ lỗi trên cả hai route có vấn đề về 0 trong suốt đợt test lại, và p95 của start phiên thi giảm đủ để khiếu nại về thời gian tới câu hỏi đầu tiên không còn xuất hiện trong phản hồi.

### Kết nối các dịch vụ AWS trong tuần này:

* **CloudWatch Logs + Lambda:** log lỗi có cấu trúc từ từng function được truy vấn bằng Logs Insights để quy lỗi về đúng route.
* **CloudWatch Metrics + API Gateway:** số 4xx/5xx trên stage cho ra tỷ lệ lỗi API để đối chiếu với báo cáo của tester.
* **RDS for PostgreSQL + Lambda:** migration thêm index giảm thời gian query start phiên thi, thể hiện trực tiếp qua p95 của Lambda thấp hơn.
* **S3 + CloudFront:** TTL của presigned URL cho audio listening được nâng lên để việc truy cập asset sống qua trọn một section.
* **Amplify Hosting + Next.js:** mỗi lần tinh chỉnh UI được build và deploy tự động, giữ vòng phản hồi với tester ngắn.
* **SAM + CloudFormation:** các bản sửa backend đi ra trong một lần deploy lên `lingorise-dev` để số liệu trước và sau vẫn so sánh được.

### Kết quả đạt được tuần 11:

* Tạo được một bức tranh đo lường thống nhất cho đợt thử nghiệm, ghép tỷ lệ hoàn thành và phản hồi tester với tỷ lệ lỗi API và p95 latency.
* Quy được từng khiếu nại về đúng nhóm vướng mắc sản phẩm hoặc lỗi kỹ thuật, thay vì coi tất cả đều là bug.
* Hoàn thiện giao diện bài thi rõ ràng hơn: câu chưa trả lời nổi bật, hành động tiếng Việt không nhập nhằng, loading và error state thật, layout mobile dùng được.
* Sửa bốn lỗi kỹ thuật cụ thể, trong đó có lỗi audio gián đoạn mà chỉ metric mới giải thích được.
* Kiểm chứng các bản sửa bằng cách đo lại chính chỉ số cũ, không chỉ giả định là đã xong.

### Khó khăn gặp phải:

* Lỗi gây thiệt hại nhất lại là lỗi gián đoạn và không tái hiện được trên máy tôi, chỉ lộ ra khi xếp báo cáo của tester cạnh timestamp trong CloudWatch.
* Phản hồi đến dưới dạng cảm nhận về giao diện, nên việc chuyển nó thành danh sách xếp hạng kèm nguyên nhân kỹ thuật tốn thời gian hơn cả lúc viết code sửa.

### Bài học rút ra và định hướng tiếp theo:

* Metric và phản hồi người dùng không thay thế được cho nhau. Log cho biết route nào lỗi; chỉ tester mới cho biết lỗi nào thực sự làm ai đó mất một bài thi.
* Ghi rõ nguyên nhân của từng bản sửa, một index, một cache key, một URL TTL, giúp changelog trung thực và dễ phát hiện regression về sau.
* Tuần tới, tôi sẽ xử lý nốt các lỗi còn lại theo thứ tự ưu tiên, viết báo cáo workshop tổng kết kèm dữ liệu đánh giá của người dùng, và đóng gói toàn bộ dự án cùng hồ sơ internship để nghiệm thu.
