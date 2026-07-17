---
title: "Blog 1 - Kiến trúc Serverless LingoRise"
date: 2026-07-20
weight: 1
chapter: false
pre: " <b> 1. </b> "
---
# XÂY DỰNG LINGORISE TRÊN AWS — KIẾN TRÚC SERVERLESS CHO NỀN TẢNG LUYỆN THI IELTS/TOEIC

LingoRise là nền tảng luyện thi IELTS và TOEIC ứng dụng AI: tự sinh đề luyện tập, chấm Writing và Speaking, tổng hợp audio phần Listening và xử lý thanh toán — tất cả mà không cần một máy chủ nào để chúng tôi phải vá lỗi, scale hay thức dậy lúc 2 giờ sáng để xử lý. Toàn bộ backend chạy trên AWS như một stack serverless duy nhất, khai báo trong một file AWS SAM template. Bài viết này đi qua kiến trúc đó và, với từng dịch vụ AWS, chỉ ra lợi ích cụ thể mà nó mang lại cho một đội nhỏ đang xây dựng sản phẩm thật.

## Vì sao chọn serverless, và vì sao chọn AWS

Trước khi viết một dòng hạ tầng nào, chúng tôi đưa ra một quyết định định hình mọi thứ về sau: **không quản lý máy chủ**. Một nền tảng luyện thi có lưu lượng truy cập thất thường và đột biến — ban đêm yên ắng, rồi tăng vọt khi cả một lớp cùng làm bài thi thử vào một buổi tối. Provisioning EC2 cho mức đỉnh nghĩa là trả tiền cho công suất nhàn rỗi 90% thời gian; provisioning cho mức trung bình nghĩa là sập vào ngày thi.

Các khối serverless của AWS giải quyết trực tiếp bài toán này:

* **Kinh tế scale-to-zero** — trả tiền theo từng request, không theo giờ. Một đêm nhàn rỗi gần như không tốn gì.
* **Co giãn mặc định** — Lambda tự động fan-out ra hàng trăm execution đồng thời khi có đột biến, không cần pre-warm hay lập kế hoạch công suất.
* **Mọi thứ đều managed** — Cognito lo authentication, S3 lo lưu trữ, Polly lo giọng nói. Chúng tôi viết logic nghiệp vụ, không viết hạ tầng không tạo khác biệt.
* **Một template, một lệnh deploy** — toàn bộ stack (API, auth, storage, WAF, IAM) khai báo trong `template.yaml` và triển khai chỉ với một lệnh `sam deploy`.

## Luồng request, từ đầu đến cuối

Một request từ trình duyệt của học viên đi qua một chuỗi dịch vụ AWS, mỗi dịch vụ làm tốt một việc:

```
Trình duyệt
  → Route 53 (DNS)  → ACM + Amplify (frontend Next.js, TLS)
  → AWS WAF (lá chắn tầng 7)
  → API Gateway (REST, stage "Prod", có throttling)
  → AWS Lambda (một app Express duy nhất qua serverless-http)
  → PostgreSQL  •  Cognito  •  S3  •  Polly
  → CloudWatch (log JSON có cấu trúc)
```

Điểm thiết kế đáng nói: thay vì mỗi route một Lambda, LingoRise chạy **một Lambda duy nhất** host một app Express và để framework tự định tuyến bên trong. API Gateway chỉ đơn giản forward mọi path (`/` và `/{proxy+}`) đến nó. Cách này giúp cold start hiếm khi xảy ra, deploy cực đơn giản, và môi trường dev giống hệt production — cùng một Express server chạy trên `localhost:4000` và bên trong Lambda.

## Các dịch vụ AWS chính và lợi ích của từng cái

* **API Gateway** — cửa ngõ. Ngoài định tuyến, nó áp throttling ở mức stage (ổn định 50 req/s, burst 100). Lưu lượng vượt ngưỡng bị loại bỏ với mã `429` *trước khi* chạm tới Lambda, nên một đợt burst không thể fan-out thành hàng trăm invocation đồng thời và làm cạn connection pool của database.
* **AWS Lambda** — compute trả theo lần gọi, không tốn công vận hành server. Hàm API được cấu hình 1024 MB / 120 s để xử lý thoải mái việc sinh nội dung bằng AI và import file, trong khi mặc định gọn nhẹ 256 MB cho các tác vụ nhẹ hơn.
* **Amazon Cognito** — authentication được quản lý hoàn toàn. Một User Pool lo đăng ký, gửi mã xác thực qua email, chính sách mật khẩu, và phát hành JWT để Lambda xác minh trên mỗi route được bảo vệ. Chúng tôi không bao giờ tự lưu hay hash mật khẩu.
* **Amazon S3** — lưu trữ asset bền vững cho ảnh câu hỏi và audio được sinh ra. Bucket chặn toàn bộ truy cập public và phục vụ nội dung qua **signed URL** (hết hạn sau 15 phút), nên asset vẫn riêng tư mà trình duyệt vẫn tải trực tiếp được. Lifecycle rule tự xóa các bản nháp import sau 7 ngày.
* **Amazon CloudWatch** — log JSON có cấu trúc từ mỗi lần invoke, cho khả năng quan sát có thể tìm kiếm mà không cần tự dựng hệ thống logging.

## Bảo mật nhiều lớp

Bảo mật ở đây không phải một tính năng đơn lẻ; nó được xếp lớp để lỗi ở một lớp sẽ được lớp sau chặn lại:

* **AWS WAF** đứng trước API Gateway với bốn rule đánh giá theo thứ tự ưu tiên: rule rate-based theo IP (chặn bất kỳ IP nào vượt 2000 request / 5 phút), Common Rule Set do AWS quản lý (bảo vệ kiểu OWASP), IP Reputation List (bot xấu đã biết), và bộ Known Bad Inputs (Log4j/JNDI, path traversal, host-header injection). Hai sub-rule trong Common Rule Set được cố ý đặt thành *Count* thay vì *Block* vì nếu không sẽ chặn nhầm các lần import JSON lớn hợp lệ và upload file nhị phân.
* **Throttling ở mức stage** trên API Gateway là lớp phòng thủ thứ hai chống lại flood.
* **Xác minh JWT của Cognito** canh giữ mọi endpoint yêu cầu đăng nhập.
* **IAM đặc quyền tối thiểu** — execution role của Lambda chỉ được cấp đúng các action cần thiết: CRUD trên chính S3 bucket của nó, một danh sách ngắn các lệnh admin Cognito giới hạn theo ARN của User Pool cụ thể, và `polly:SynthesizeSpeech`. Không hơn.

## Vì sao cách này thắng với một đội nhỏ

Giá trị của cách tiếp cận serverless-trên-AWS được đo bằng những vấn đề chúng tôi *không* gặp phải:

* **Không lập kế hoạch công suất** — lưu lượng tăng gấp đôi vào tối thi; Lambda và API Gateway tự hấp thụ.
* **Không tốn chi phí nhàn rỗi** — một tuần yên ắng chỉ tốn phần lẻ, vì compute chỉ tính tiền khi có request đến.
* **Không chạy theo vòng vá lỗi** — không có OS, không có runtime host, không có load balancer để phải vá.
* **Deploy một lệnh** — toàn bộ stack nằm trong `template.yaml`; một lệnh deploy ship cả hạ tầng lẫn code cùng nhau, tái lập được.

Với một đội tinh gọn xây dựng sản phẩm AI, điều đó nghĩa là thời gian kỹ thuật đổ vào chất lượng đề thi và độ chính xác khi chấm — những thứ học viên thực sự cảm nhận — thay vì đổ vào việc giữ cho máy chủ sống.

...Hình ảnh...

...Link...

...Hướng dẫn...
