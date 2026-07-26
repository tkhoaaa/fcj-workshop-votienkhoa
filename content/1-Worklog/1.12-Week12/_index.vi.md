---
title: "Worklog Tuần 12"
date: 2026-07-06
weight: 12
chapter: false
pre: " <b> 1.12. </b> "
---

### Mục tiêu tuần 12:

* Sắp xếp mức độ ưu tiên cho toàn bộ lỗi phát hiện trong quá trình người dùng trải nghiệm thực tế theo severity và mức ảnh hưởng, sau đó xử lý triệt để các blocker.
* Kiểm chứng từng bản sửa trên luồng exam và luồng payment trước khi chốt phiên bản LingoRise cuối cùng.
* Viết báo cáo tổng kết buổi workshop cuối khóa và tổng hợp số liệu đánh giá của người tham dự cùng người dùng.
* Đóng gói dự án: tài liệu kỹ thuật hoàn thiện, source code sạch trên GitHub và toàn bộ hồ sơ thực tập để chuẩn bị nghiệm thu.

### Các công việc cần triển khai trong tuần này:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Tập hợp toàn bộ lỗi còn mở từ phản hồi người dùng và từ truy vấn **CloudWatch Logs**, rồi phân loại thành blocker, major, minor theo severity và mức ảnh hưởng nghiệp vụ. | 06/07/2026 | 06/07/2026 | [CloudWatch Logs Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AnalyzingLogData.html) |
| **2** | Sửa các blocker trên luồng exam session và payment webhook, deploy lại backend bằng **AWS SAM** và kiểm chứng từng bản sửa end to end. | 07/07/2026 | 07/07/2026 | [AWS SAM Docs](https://docs.aws.amazon.com/serverless-application-model/) |
| **3** | Viết báo cáo tổng kết cho buổi workshop cuối khóa và tổng hợp số liệu đánh giá thu được từ người tham dự và pilot users. | 08/07/2026 | 08/07/2026 | Ghi chú workshop |
| **4** | Hoàn thiện tài liệu kỹ thuật, đẩy source code sạch lên GitHub và đóng gói toàn bộ hồ sơ thực tập để nghiệm thu. | 09/07/2026 | 09/07/2026 | Repository dự án |

### Cách thực hiện chi tiết:

#### 1. Phân loại backlog lỗi còn lại theo severity và mức ảnh hưởng

Tôi bắt đầu từ một danh sách duy nhất thay vì sửa theo cái nào bị phàn nàn nhiều nhất. Danh sách này ghép phản hồi người dùng của tuần 11 với các bản ghi lỗi lấy từ **Amazon CloudWatch Logs** của các Lambda function trong stack `lingorise-dev`, nên mỗi mục đều có bước tái hiện hoặc một request ID đi kèm.

Sau đó tôi gán nhãn cho từng mục:

* **blocker** — người dùng không thể hoàn thành một exam session hoặc không thanh toán được
* **major** — chức năng vẫn chạy nhưng kết quả sai hoặc gây nhầm lẫn (làm tròn band score, thứ tự transcript ở trang review)
* **minor** — lỗi hiển thị hoặc ít lưu lượng (label tiếng Việt dài bị xuống dòng, độ rộng cột trong bảng admin)

Việc nhóm lỗi theo function và theo message làm thứ tự ưu tiên trở nên rõ ràng. Hai blocker và năm major chiếm gần như toàn bộ số request thất bại, còn danh sách minor thì dài nhưng vô hại.

#### 2. Sửa và kiểm chứng các blocker trên luồng exam và payment

Blocker thứ nhất là lỗi resume session: khi thí sinh reload trang giữa bài thi, handler đọc session row trước khi lần ghi answer cuối cùng kịp commit, khiến session được resume bị mất một câu trả lời. Tôi đưa việc ghi answer và cập nhật session cursor vào cùng một transaction trên **Amazon RDS for PostgreSQL** để hai thao tác cùng thành công hoặc cùng thất bại.

Blocker thứ hai nằm ở **IPN webhook** của payment. Một lần retry từ phía provider tạo ra bản ghi trùng trong `user_subscriptions`, khiến việc resolve entitlement bị sai. Tôi làm webhook idempotent theo provider transaction id sau khi đã verify HMAC.

```sql
ALTER TABLE user_subscriptions
  ADD CONSTRAINT user_subscriptions_provider_txn_key UNIQUE (provider, provider_txn_id);
```

Migration này idempotent và được theo dõi trong bảng `_migrations` giống mọi thay đổi trước đó. Sau khi deploy các handler đã sửa bằng **AWS SAM**, tôi chạy lại cả hai tình huống trên API Gateway stage và xác nhận trong CloudWatch rằng các error signature cũ không còn xuất hiện. Các mục minor được giữ lại có chủ đích, ghi rõ trong issue list kèm lý do tạm hoãn.

#### 3. Viết báo cáo workshop cuối khóa và tổng hợp số liệu đánh giá

Tôi viết báo cáo tổng kết cho buổi workshop cuối khóa của FCJ, "Deploying LingoRise — A Serverless English-Learning Platform on AWS". Báo cáo giữ đúng trình tự của phần demo trực tiếp:

* lưu secrets dưới dạng SecureString parameter trong **AWS Systems Manager Parameter Store**
* tạo instance **Amazon RDS for PostgreSQL** và chạy migrations
* build và deploy backend **AWS Lambda** cùng **Amazon API Gateway** bằng **AWS SAM**
* kết nối frontend **Next.js** trên **AWS Amplify Hosting**
* upload ảnh câu hỏi và audio lên **Amazon S3** rồi phân phối qua **Amazon CloudFront**
* chạy smoke test, sau đó làm theo các bước cleanup để không phát sinh chi phí tồn đọng

Song song với phần walkthrough, tôi tổng hợp số liệu đánh giá: phản hồi của người tham dự về buổi workshop và điểm usability thu được từ pilot users trong tuần 11. Báo cáo cả hai cùng lúc hữu ích hơn, vì cùng những điểm gây khó khăn đó xuất hiện cả trong câu hỏi tại workshop lẫn trong nhận xét của người dùng.

#### 4. Đóng gói dự án và hồ sơ thực tập để nghiệm thu

Bước cuối là đóng gói. Tôi hoàn thiện tài liệu kỹ thuật — tổng quan architecture, danh sách environment variables kèm đường dẫn SSM parameter tương ứng, quy trình migration, runbook deploy và rollback — rồi dọn repository trước lần push cuối: không commit `.env`, không còn build artifacts, README cập nhật đúng trình tự deploy thực tế.

```bash
sam build && sam deploy --stack-name lingorise-dev --no-confirm-changeset
aws cloudformation describe-stacks --stack-name lingorise-dev \
  --query "Stacks[0].StackStatus" --region ap-southeast-1
```

Sau đó tôi tập hợp chính hồ sơ thực tập: worklog mười hai tuần, proposal LingoRise, các bài blog kỹ thuật, phần ghi nhận event, báo cáo workshop và bản self-evaluation, tất cả đều được publish trên Hugo site này để người review đọc trọn quá trình thực tập ở một nơi.

### Kết nối các dịch vụ AWS trong tuần này:

* **CloudWatch Logs + Lambda:** Các truy vấn log biến những phàn nàn rời rạc của người dùng thành một danh sách lỗi có thứ tự ưu tiên và kèm request ID.
* **Lambda + RDS for PostgreSQL:** Bản sửa resume session đưa việc ghi answer và cập nhật cursor vào một transaction nên reload không còn làm mất câu trả lời.
* **API Gateway + Lambda + RDS:** Payment webhook idempotent giúp các lần retry từ provider trở nên an toàn, giữ `user_subscriptions` sạch cho việc kiểm tra entitlement.
* **AWS SAM + CloudFormation:** Mọi bản sửa đều vào stack `lingorise-dev` qua cùng một đường build và deploy, nên bản cuối có thể tái lập được.
* **SSM Parameter Store + SAM:** Việc ghi rõ các parameter path cho phép runbook deploy không cần chứa bất kỳ giá trị secret nào trong repository.
* **Amplify Hosting + S3 + CloudFront:** Frontend và các media asset được kiểm tra cùng nhau trong lần smoke test cuối.

### Kết quả đạt được tuần 12:

* Có được danh sách lỗi đã phân loại, trong đó severity gắn với mức ảnh hưởng thực tế lên người dùng chứ không theo thứ tự báo lỗi.
* Sửa và kiểm chứng xong cả hai blocker trên luồng exam và payment, thay đổi schema được quản lý như một migration idempotent.
* Chủ động hoãn các mục minor và ghi rõ lý do thay vì để chúng không được ghi nhận ở đâu.
* Hoàn thành báo cáo workshop cuối khóa với đầy đủ phần deployment walkthrough và số liệu đánh giá đã tổng hợp.
* Hoàn thiện tài liệu kỹ thuật và source code sạch trên GitHub.
* Tập hợp trọn bộ hồ sơ thực tập — worklog, proposal, blogs, events, workshop, self-evaluation — sẵn sàng nghiệm thu.

### Khó khăn gặp phải:

* Quyết định lỗi nào không sửa còn khó hơn việc sửa. Chấp nhận các lỗi minor đã biết trong bản cuối đòi hỏi một lý giải viết ra rõ ràng, không phải cảm tính.
* Lỗi subscription trùng chỉ xuất hiện khi provider retry, nên muốn tái hiện phải replay webhook payload thay vì click qua UI.
* Viết tài liệu để người khác có thể deploy theo buộc tôi phải tìm ra những bước chưa từng ghi lại mà vẫn đang nhớ trong đầu.

### Bài học rút ra và định hướng tiếp theo:

* Suốt mười hai tuần, lộ trình là liên tục: AWS fundamentals và bảo mật tài khoản, rồi networking, compute và storage, sau đó DevOps và IaC, cuối cùng là thiết kế, xây dựng và ship LingoRise theo kiến trúc serverless. Mỗi giai đoạn chỉ có ý nghĩa nhờ giai đoạn trước đó.
* Ship sản phẩm là một kỹ năng riêng so với việc xây sản phẩm. Triage, kiểm chứng, viết tài liệu và đóng gói chiếm trọn một tuần và chính là phần biến code chạy được thành một sản phẩm có thể giao.
* Định hướng học tiếp của tôi khá cụ thể: ôn thi chứng chỉ AWS Solutions Architect, đào sâu hơn về Infrastructure as Code, và hardening LingoRise cho production với Multi-AZ RDS, một read replica cho các truy vấn báo cáo, cùng bộ CloudWatch alarm chặt hơn.
