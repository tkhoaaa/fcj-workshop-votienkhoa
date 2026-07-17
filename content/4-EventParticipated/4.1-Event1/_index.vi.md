---
title: "Sự kiện 1: AWS First Cloud AI Journey — Community Day"
date: 2026-07-23
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

# AWS First Cloud AI Journey — Community Day

Tôi dành nguyên một buổi sáng thứ Bảy trên tầng 26 của Bitexco để tham dự AWS First Cloud AI Journey Community Day, và thật lòng thì đây là một trong những buổi hữu ích nhất mà tôi làm được bên ngoài mấy task internship hằng ngày. Sự kiện do AWS Study Group tổ chức, trong phòng là một mớ trộn lẫn giữa các cloud architect, GenAI engineer, dân DevOps, và những bạn sinh viên như tôi, chủ yếu tới để hóng và học lỏm. Tôi đi với vai trò người tham dự, không kỳ vọng gì nhiều ngoài mấy màn giới thiệu kiểu "đây là service AWS hay ho". Nhưng thực tế không phải vậy, và đó là lý do tôi chịu khó ngồi ghi lại.


## Những phiên thật sự đọng lại

Có 6 bài nói trải dài cả buổi sáng, từ 8:30 tới trưa. Có bài "trúng" hơn hẳn bài khác, nên mình sẽ thành thật là bài nào tác động tới mình.

**"Context Is Everything" của anh Tinh Truong (GoTymeX).** Đây là bài tôi cứ nghĩ đi nghĩ lại mãi. Luận điểm của anh là khi AI trả về rác, thường không phải lỗi model, mà là do mình không cho nó đủ thông tin để hiểu bài toán thật sự. Anh chia context thành mục tiêu, tình huống, ràng buộc và bằng chứng liên quan, và câu đọng lại với tôi là "context biến một yêu cầu mơ hồ thành một bài toán giải được". Điều tôi thích là anh cũng cảnh báo cả cái sai ngược lại, cái anh gọi là vấn đề "internet puller", khi người ta dán nguyên cả PDF và năm chục cái screenshot vào một ô chat rồi thắc mắc sao câu trả lời ngày càng tệ. Nhiều context không có nghĩa là context tốt hơn. Là người mắc đúng cái tật này, tôi thấy hơi bị "réo tên", nhưng cần thiết.

**"GenAI-Powered Auto Audit for AWS Workload" của chị Pham Nguyen Hai Anh (G-AsiaPacific).** Bài này nghiêng về hướng doanh nghiệp hơn cái tôi thường quan tâm, nhưng cách chị mô tả nỗi đau của người dùng nghiệp vụ thì rất sắc. Chị đi qua Amazon Q Business, phần hỏi đáp bằng ngôn ngữ tự nhiên, hơn 40 cổng kết nối dữ liệu, các guardrail, và một demo trợ lý PM tự ghi biên bản họp rồi tự lên lịch họp tiếp. Tôi thú thật là mấy chi tiết sâu về audit tự động hơi vượt tầm hiểu của tôi, nhưng ý tưởng chung là chĩa một con LLM vào chính hạ tầng của mình để dò các lỗ hổng tuân thủ thì rất thực tế.

**"From Edge to Origin: CloudFront as Your Foundation" của anh Nguyen Tuan Thinh.** Tôi vào phòng với suy nghĩ CloudFront chỉ là "cái CDN đó thôi", và bước ra hơi sợ, theo nghĩa tốt. Điểm về nghịch lý pay-as-you-go làm tôi nhớ mãi: gói free tier 1TB rất hào phóng, nhưng một cú viral hay một đợt tấn công DDoS có thể biến thành hóa đơn năm con số chỉ sau một đêm. Mấy con số kinh dị anh nêu ra (hơn $100K trong trường hợp cực đoan) là kiểu khiến người ta phải đi bật billing alert ngay. Mà tôi làm thật, sẽ nói thêm ở dưới.

**"36 hrs with LotusHacks: Building UTMorpho" của Team VIB.** Đây là bài vui nhất, và chắc cũng là bài cả phòng khoái nhất. Nhóm kể lại hành trình 36 giờ chạy hackathon từ "giờ số 0 với cái đầu trống rỗng" cho tới lúc có demo chạy được, và họ thành thật một cách dễ chịu về mấy con bug và cơn hoảng loạn ở đoạn giữa. Cái ý tập trung giải một vấn đề duy nhất thay vì chất chồng tính năng là thứ tôi cần nghe đi nghe lại, vì tôi lúc nào cũng muốn thêm cái này cái kia.

**"Non-Determinism of 'Deterministic' LLM Settings" của anh Duc Dao (Cloud Kinetics).** Bài này lặng lẽ gỡ được một nút thắt cho tôi. Anh giải thích vì sao đặt temperature bằng 0 vẫn không cho ra output giống hệt, về cơ bản là do phép cộng dấu phẩy động chạy song song trên các lõi GPU không có tính kết hợp, nên thứ tự cộng dồn đổi một chút là đủ làm lệch xác suất và đổi luôn token được chọn. Cộng thêm cân bằng tải server và độ lệch phần cứng ở lớp API. Tôi ngồi đó và nhận ra đây chính xác là lý do bộ tạo đề thi của LingoRise thỉnh thoảng trả JSON hơi lỗi dù tôi tưởng đã khóa hết mọi thứ.

**"Enterprise-Grade Multi-Agent System" của chị Vy Lam (VPBank).** Đây là bài có cấu trúc chặt nhất, nói về chấm điểm tín dụng cho startup. Vấn đề cốt lõi là ngân hàng đòi báo cáo tài chính ba năm trở lên còn startup thì không có, nên bị loại dù rất tiềm năng. Lời giải của chị là một "hội đồng tín dụng ảo" gồm các agent chuyên biệt, một người phân tích tài chính, một người thẩm định IP, một người kiểm toán rủi ro, phối hợp dưới một layer điều phối trung tâm, thay vì một agent đơn ôm hết. Chị cũng trình bày mô hình bảo mật năm lớp, perimeter, mạng, danh tính, ứng dụng và dữ liệu, hơi nhiều thứ để thấm nhưng là một checklist hay.

## Những gì tôi thật sự mang về

Điểm nối gần như mọi bài nói là: model hiếm khi là phần khó. Context, trí nhớ, độ tin cậy và guardrail mới là nơi công việc thật sự nằm ở đó. Điều này khiến tôi nhìn lại chính dự án của mình khác đi.

Vài thứ khớp thẳng với LingoRise:

- Bài multi-agent xác nhận một lựa chọn mà tôi đã vô tình làm đúng một nửa. Tôi tách logic AI của LingoRise thành một luồng tạo đề thi riêng và một luồng chấm bài Writing riêng, và hóa ra đó chính là cái bản năng "chia thành các chuyên gia" mà chị Vy Lam đang lập luận, chỉ là ở quy mô nhỏ hơn.
- Bài non-determinism của anh Đức cuối cùng đã giải thích vì sao tôi xây bộ parser fallback bằng regex `extractJsonObject()` hồi Tuần 5. Lúc đó tôi thấy nó như một miếng vá chắp vá. Giờ tôi hiểu nó là một cơ chế phòng thủ chính đáng trước sự dao động của output LLM, và tôi thấy yên tâm hơn khi giữ lại nó.
- Phiên bảo mật khớp gọn gàng với phần hardening tôi làm ở Tuần 10, rate limiting, input validation, chống prompt injection. Nghe dân enterprise mô tả đúng những lớp mà tôi đang mò mẫm tự làm khiến tôi khá an tâm.
- Và sau bài CloudFront, tôi về nhà bật billing alert cho đàng hoàng và tìm hiểu Origin Access Control cho mấy tài nguyên S3 của mình. Đó là một kết quả rất khô khan, rất hữu ích, và đúng là kiểu việc mà tự tôi sẽ không ưu tiên nếu không đi nghe.

#### A few photos from the event
![Ảnh chụp tại sự kiện](/images/4-EventParticipated/event1.jpg)
