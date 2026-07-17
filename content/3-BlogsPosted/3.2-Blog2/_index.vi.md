---
title: "Blog 2 - AI Serverless trên AWS"
date: 2026-07-20
weight: 2
chapter: false
pre: " <b> 2. </b> "
---
# AI SERVERLESS TRÊN AWS — CÁCH LINGORISE TẠO VÀ CHẤM ĐỀ THI

Phần khó của một nền tảng luyện thi dùng AI không phải là website, mà là việc tạo ra nội dung IELTS/TOEIC và chấm điểm ở quy mô lớn mà không cần sở hữu một cụm GPU. LingoRise giải quyết trọn vẹn bài toán này bằng các khối serverless của AWS: AWS Lambda điều phối tác vụ AI, Amazon Polly tổng hợp audio phần nghe, và Amazon S3 lưu trữ cùng phân phối mọi thứ. Bài viết này đi qua pipeline AI và nội dung đó, cùng lý do đằng sau cách nó được sizing và bảo mật.

## Bài toán: AI ở quy mô lớn mà không cần một server để trông coi

Việc tạo một đoạn văn đọc kèm câu hỏi, hoặc chấm một bài viết Writing tự luận, đồng nghĩa với việc gọi một large language model và xử lý hậu kỳ không hề đơn giản. Những lời gọi này bùng nổ theo đợt và đôi khi chậm. Chạy chúng trên một server luôn bật nghĩa là trả tiền cho GPU nhàn rỗi và phải hoạch định năng lực cho những đợt cao điểm buổi tối thi thử. Cách tiếp cận serverless đảo ngược điều này: compute chỉ tồn tại trong thời gian của request, và tự động scale-out khi cả một lớp cùng bấm "bắt đầu".

## Pipeline AI trên Lambda

Toàn bộ việc điều phối AI nằm bên trong function API Express-trên-Lambda duy nhất. Khi cần nội dung đề thi, handler gọi tới một endpoint chat tương thích OpenRouter, sau đó kiểm tra và chuẩn hóa output của model trước khi nó đến tay học viên.

Hai lựa chọn thiết kế giúp điều này hoạt động trong giới hạn serverless:

* **Pre-generate và cache.** Nội dung được tạo trước và cache lại thay vì tạo trực tiếp trên đường đi quan trọng (critical path). Một học viên bắt đầu bài thi sẽ đọc những câu hỏi đã được dựng sẵn và kiểm định — lời gọi AI chậm và tốn kém đã xảy ra từ trước, nên request nhanh và rẻ. Cache cũng cắt giảm trực tiếp hóa đơn AI, vì cùng một nội dung không bao giờ bị trả tiền hai lần.
* **Compute được sizing đúng.** Function API chạy ở **1024 MB với timeout 120 giây**, cố ý lớn hơn mức mặc định nhẹ 256 MB, chính vì tạo nội dung AI và import file hàng loạt là những tác vụ nặng nhất. Nhiều bộ nhớ hơn trên Lambda cũng đồng nghĩa với nhiều CPU hơn, giúp rút ngắn mỗi lần generate.

Với đường chấm điểm, output của model được parse và chấm dựa trên một answer key đã kiểm định, kèm xử lý null-safe để một phản hồi AI lỗi sẽ suy giảm nhẹ nhàng thay vì làm crash kết quả của học viên.

## Audio phần nghe mà không cần server: Amazon Polly

Phần Listening cần audio giọng nói. Ở môi trường local chúng tôi dùng `edge-tts`, nhưng nó spawn một tiến trình Python và **không thể chạy trên Lambda**. Amazon Polly giải quyết gọn gàng: nó là một lời gọi SDK thuần túy, không subprocess, không binary phải đóng gói.

* Polly chạy ở chế độ engine **neural** với giọng `Joanna`, tạo ra giọng nói tự nhiên cho các nút "Nghe thử" và "Xác nhận nhập".
* Vì chỉ là một lời gọi API, cùng một code path hoạt động y hệt trên production mà không cần phụ thuộc CLI local.
* IAM role của Lambda chỉ cấp đúng một action Polly — `polly:SynthesizeSpeech` — và không gì khác.

## Lưu trữ và phân phối: Amazon S3

Mọi thứ pipeline tạo ra — audio sinh ra, hình ảnh câu hỏi, bản nháp import — đều nằm trong một S3 bucket riêng tư:

* Bucket **chặn toàn bộ truy cập public** và dùng ownership `BucketOwnerEnforced`, nên không gì vô tình bị đọc công khai.
* Nội dung được phục vụ qua **signed URL với TTL 15 phút**, cho phép trình duyệt tải asset trực tiếp từ S3 mà không cần Lambda làm trung gian truyền byte, đồng thời giữ object ở chế độ riêng tư.
* Một **lifecycle rule** tự động hết hạn `import-drafts/` sau 7 ngày và hủy các multipart upload dang dở sau 1 ngày, nên các file làm việc tạm thời tự dọn dẹp.

## Chi phí và khả năng chống chịu

Pipeline được thiết kế sao cho phần đắt đỏ hiếm khi xảy ra và phần rẻ thì xảy ra thường xuyên:

* **Cache** đảm bảo model AI được trả tiền một lần cho mỗi nội dung độc nhất, không phải một lần cho mỗi học viên.
* **Rate limiting** — mặc định là bộ giới hạn in-memory theo từng Lambda, có thể nâng cấp lên một cửa sổ dùng chung dựa trên serverless Redis (ví dụ Upstash qua TLS) mà không cần đổi code — ngăn traffic lạm dụng biến thành một hóa đơn AI mất kiểm soát.
* **Graceful fallback** — chấm điểm null-safe và generate đã kiểm định nghĩa là một phản hồi model tệ không bao giờ đánh sập phiên làm bài của học viên.

Kết quả là một nền tảng AI nơi phần việc nặng — tạo nội dung, chấm điểm, tổng hợp audio — chạy theo nhu cầu, chỉ tốn tiền khi được dùng, và tự scale. Không cụm GPU, không server audio, không bản kế hoạch năng lực. Chỉ có Lambda, Polly và S3, mỗi thứ làm đúng một việc.

...Hình ảnh...

...Link...

...Hướng dẫn...
