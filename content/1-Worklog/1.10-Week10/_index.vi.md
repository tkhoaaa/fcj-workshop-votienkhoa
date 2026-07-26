---
title: "Worklog Tuần 10"
date: 2026-06-22
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Mục tiêu tuần 10:

* Đóng gói toàn bộ source tree của LingoRise trên GitHub để một engineer khác có thể redeploy stack mà không cần hỏi tôi bất cứ điều gì.
* Dọn dẹp các tài nguyên AWS còn sót lại từ những tuần trước và đặt hạn mức chi tiêu cho phần còn giữ lại.
* Xác định đúng các thành phần tốn chi phí nhất của môi trường đang chạy bằng **AWS Cost Explorer** và hóa đơn theo từng service.
* Hoàn thiện Final Internship Report bằng Tiếng Anh, bao gồm khảo sát dịch vụ AWS, thiết kế kiến trúc, triển khai, kết quả kiểm thử và bài học kinh nghiệm.

### Các công việc cần triển khai trong tuần này:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Dọn dẹp repository để bàn giao: README kèm deploy steps, `.env.example`, tài liệu migration và rà soát để chắc chắn không commit secret nào. | 22/06/2026 | 22/06/2026 | Repository dự án |
| **2** | Viết runbook redeploy cho SAM stack `lingorise-dev` và tag một release trên GitHub. | 23/06/2026 | 23/06/2026 | [AWS SAM Docs](https://docs.aws.amazon.com/serverless-application-model/) |
| **3** | Rà soát Cost Explorer và hóa đơn theo service, sau đó tắt hoặc xóa các tài nguyên lab không còn dùng từ những tuần trước. | 24/06/2026 | 24/06/2026 | [AWS Cost Explorer](https://docs.aws.amazon.com/cost-management/latest/userguide/ce-what-is.html) |
| **4** | Cấu hình AWS Budgets alerts cho account và xác nhận RDS cùng WAF là phần chi phí cố định lớn nhất. | 25/06/2026 | 25/06/2026 | [AWS Budgets Docs](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html) |
| **5** | Viết Final Internship Report bằng Tiếng Anh và chuẩn hóa nội dung site báo cáo Hugo cho khớp với báo cáo. | 26/06/2026 | 26/06/2026 | [FCJ Workshop Sample](https://workshop-sample.fcjuni.com/) |

### Cách thực hiện chi tiết:

#### 1. Đóng gói repository để bàn giao

Tôi coi repository chính là sản phẩm bàn giao, không chỉ là nơi chứa code. README giờ mô tả rõ cấu trúc của frontend Next.js, các handler **AWS Lambda** và template **AWS SAM**, kèm đúng chuỗi lệnh để build và deploy vào một account mới.

Tôi thêm `.env.example` liệt kê mọi biến mà ứng dụng cần, với giá trị thay bằng placeholder. Giá trị thật nằm trong **AWS Systems Manager Parameter Store** dưới dạng SecureString và được resolve lúc deploy qua `{{resolve:ssm:/lingorise/${Stage}/...}}`, nên không có gì nhạy cảm phải nằm trong repository. Sau đó tôi rà lại history theo các pattern quen thuộc và xác nhận chưa từng commit access key, database password hay webhook secret nào.

Phần migration cũng cần được ghi lại. Migration là các file SQL idempotent được theo dõi trong bảng `_migrations`, nên tôi ghi rõ quy tắc thứ tự và việc chạy lại toàn bộ trên một database **Amazon RDS for PostgreSQL** đang có dữ liệu là an toàn.

#### 2. Viết runbook redeploy và tag release

Runbook đi từ một account trống đến môi trường chạy được: tạo các entry trong Parameter Store, build bundle Lambda bằng esbuild, deploy stack, rồi chạy migration lên database.

```bash
sam build && sam deploy --stack-name lingorise-dev \
  --parameter-overrides Stage=dev --capabilities CAPABILITY_IAM
aws cloudformation describe-stacks --stack-name lingorise-dev \
  --query "Stacks[0].Outputs" --region ap-southeast-1
```

Stack outputs trả về invoke URL của **Amazon API Gateway** và user pool ID của **Amazon Cognito**, đây đúng là hai giá trị mà bản build frontend trên **AWS Amplify Hosting** cần. Điều tôi muốn ghi lại chính là chuỗi phụ thuộc này: không thể cấu hình frontend trước khi CloudFormation hoàn tất, nên runbook phải sắp thứ tự theo đúng logic đó. Tôi tag commit thành một release để báo cáo tham chiếu tới một trạng thái code cố định.

#### 3. Dọn dẹp tài nguyên AWS và kiểm soát chi phí

Tôi mở **AWS Cost Explorer** group theo service cho ba tháng gần nhất rồi rà từ trên xuống. Khá nhiều tài nguyên từ các tuần networking và compute vẫn còn sống: các instance **Amazon EC2** dư, các volume **Amazon EBS** không còn attach, những AMI cũ cùng snapshot phía sau, và một **NAT Gateway** vẫn tính tiền theo giờ rất lâu sau khi bài lab cần nó đã kết thúc. Tôi terminate các instance, xóa volume và snapshot, deregister các AMI cũ và xóa NAT Gateway.

Phần còn lại cho thấy hình dạng thật của hóa đơn. **Amazon RDS** với `db.t4g.micro` và **AWS WAF** trên API Gateway stage là chi phí cố định lớn nhất, vì cả hai tính tiền theo thời gian và theo rule chứ không theo lưu lượng. Các thành phần serverless như Lambda, API Gateway, **Amazon S3** gần như không đáng kể ở mức traffic phát triển. Sự đối lập đó đáng đưa vào báo cáo: chọn serverless compute đã dồn gần như toàn bộ chi phí vào hai thành phần luôn bật.

Sau đó tôi đặt **AWS Budgets** alerts ở các mốc 50, 80 và 100 phần trăm của mức chi tiêu tháng, gửi thông báo qua email để mọi tăng trưởng bất thường lộ ra sớm thay vì đến cuối kỳ billing mới biết.

#### 4. Hoàn thiện Final Internship Report bằng Tiếng Anh

Tôi viết báo cáo theo đúng thứ tự công việc đã diễn ra: khảo sát dịch vụ AWS, thiết kế kiến trúc, triển khai, kết quả kiểm thử, rồi bài học kinh nghiệm. Phần kiến trúc mô tả LingoRise như một đường đi của request chứ không phải một danh sách dịch vụ — client Next.js ưu tiên tiếng Việt trên Amplify Hosting gọi API Gateway, WAF lọc request, Lambda verify Cognito JWT bằng `aws-jwt-verify`, resolve entitlement từ `user_subscriptions`, rồi đọc ghi RDS qua `pg.Pool` singleton, còn S3 giữ ảnh câu hỏi và bài speaking submission.

Phần kết quả kiểm thử dùng lại bằng chứng từ tuần 9: hành vi resume session của exam engine, Cambridge band scoring cho IELTS và scaled bands cho TOEIC, cùng OCR job queue được worker theo lịch xử lý bằng `FOR UPDATE SKIP LOCKED`. Tôi cũng chuẩn hóa nội dung site Hugo để các trang worklog và bản báo cáo in kể cùng một câu chuyện với cùng thuật ngữ ở cả tiếng Anh và tiếng Việt.

### Kết nối các dịch vụ AWS trong tuần này:

* **AWS SAM + CloudFormation:** Stack `lingorise-dev` trở thành đơn vị deploy duy nhất, tái hiện được, và là trung tâm của runbook.
* **CloudFormation Outputs + AWS Amplify Hosting:** URL API Gateway và Cognito pool ID đi từ stack outputs vào cấu hình build của frontend.
* **Parameter Store + AWS SAM:** Secret nằm ngoài repository và được resolve lúc deploy, đó là điều khiến việc bàn giao công khai trở nên an toàn.
* **Cost Explorer + Amazon RDS/AWS WAF:** Phân tích chi phí chỉ ra hai dịch vụ luôn bật là phần chi tiêu cố định lớn nhất của một thiết kế vốn serverless.
* **AWS Budgets + Billing:** Các ngưỡng cảnh báo giờ theo dõi account để tài nguyên bỏ quên không âm thầm phát sinh chi phí.
* **Amazon EC2 + Amazon EBS + NAT Gateway:** Xóa compute còn phải xóa cả storage đã attach, snapshot và đường mạng đi kèm mới thật sự dừng được hóa đơn.

### Kết quả đạt được tuần 10:

* Bàn giao một repository đã đóng gói với deploy steps, `.env.example`, tài liệu migration và một release được tag.
* Hoàn thành runbook redeploy đưa một account trống tới stack `lingorise-dev` chạy được.
* Xóa các EC2 instance không dùng, EBS volume không attach, AMI và snapshot cũ, cùng NAT Gateway.
* Xác nhận RDS và WAF là chi phí cố định lớn nhất và ghi lại lý do các thành phần serverless đóng góp rất ít.
* Cấu hình AWS Budgets alerts ở ba mốc ngưỡng cho account.
* Hoàn thiện Final Internship Report bằng Tiếng Anh và chuẩn hóa nội dung site Hugo cho khớp.

### Khó khăn gặp phải:

* Xóa tài nguyên an toàn cần cẩn thận, vì terminate một instance không xóa EBS volume của nó hay snapshot phía sau AMI, và những thứ đó vẫn âm thầm tính tiền.
* Viết runbook làm lộ ra những bước tôi chỉ từng làm thủ công, nên phải tìm lại và ghi rõ vài giả định về thứ tự trước khi chuỗi lệnh chạy được từ đầu.
* Nén mười tuần công việc vào một báo cáo tiếng Anh buộc tôi phải quyết định bỏ bớt gì mà không làm mất mạch kiến trúc.

### Bài học rút ra và định hướng tiếp theo:

* Một cloud project chỉ được coi là bàn giao khi người khác redeploy được chỉ từ repository; những gì còn nằm trong đầu tôi đều là dependency chưa được tài liệu hóa.
* Kỷ luật chi phí là một vấn đề của kiến trúc. Chuyển sang serverless không tự động giảm chi phí khi một managed database và một WAF vẫn bật suốt ngày.
* Sang tuần tiếp theo, tôi sẽ phân tích các chỉ số trải nghiệm người dùng và tỉ lệ lỗi từ những lượt dùng thử thật, rồi tinh chỉnh frontend **Next.js**/React và sửa các vấn đề phát sinh ngoài dự kiến dựa trên feedback.
