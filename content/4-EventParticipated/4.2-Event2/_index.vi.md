---
title: "Event 2: Cloud Architect - Game Show"
date: 2026-06-20
weight: 3
chapter: false
pre: " <b> 4.2. </b> "
---

# Bài thu hoạch “Event 7: Cloud Architect - Game Show”

### Mục Đích Của Sự Kiện

- Củng cố kiến thức nền tảng về các dịch vụ đám mây (AWS) và các mẫu thiết kế kiến trúc hệ thống.
- Rèn luyện tư duy phân tích tình huống, bóc tách yêu cầu nghiệp vụ để lựa chọn giải pháp công nghệ tối ưu dưới áp lực thời gian.
- Tăng cường kỹ năng làm việc nhóm, giao tiếp chiến lược và khả năng quản trị rủi ro trong môi trường thi đấu đối kháng.

### Ban Tổ Chức & Thành Phần Tham Dự

- **Ban Tổ Chức** - AWS Study Group
- **Người chơi** - 8 đội thi đấu (mỗi đội gồm 5 thành viên được tuyển chọn ngẫu nhiên sau vòng đăng ký)
- **Vai trò** - Thí sinh tham gia thi đấu

### Nội Dung Nổi Bật

#### Thể thức thi đấu đối kháng (Head-to-Head)
- 8 đội được chia cặp thi đấu loại trực tiếp. Các đội luân phiên trả lời các bộ câu hỏi tình huống (case study) với mức độ khó tăng dần.
- Đội tích lũy được nhiều điểm số hơn sẽ giành quyền bước vào vòng trong. 
- **Quy tắc Tie-breaker**: Nếu kết thúc bộ đề mà hai đội hòa điểm, "Câu hỏi số 11" (Sudden Death) sẽ được tung ra. Đội nào bấm chuông và đưa ra đáp án chính xác nhanh hơn sẽ lập tức giành chiến thắng.

#### Trọng tâm chuyên môn: AWS Service Selection
- Các câu hỏi không nặng về lý thuyết suông mà đặt người chơi vào bối cảnh thực tế. Ví dụ: "Hệ thống đang gặp tình trạng nghẽn cổ chai khi xử lý đơn hàng ngày Black Friday, yêu cầu decouple các microservices mà không làm mất data, nên sử dụng AWS SQS hay Amazon Kinesis?".
- Đòi hỏi thí sinh phải nắm rõ các giới hạn (limits), đặc tính và phương thức tính phí của từng dịch vụ AWS để đưa ra quyết định kiến trúc chuẩn xác nhất.

#### Chiến thuật quản trị rủi ro qua bộ Kỹ năng
- **Rủi ro tối thiểu (1 lượt/trận)**: Kỹ năng phòng thủ. Sử dụng cho các câu hỏi mà nhóm còn mơ hồ hoặc có nhiều giả định chưa chắc chắn. Nếu trả lời sai, hệ thống bảo toàn điểm (không bị trừ). Nếu đúng, nhóm nhận được 1/2 số điểm của câu hỏi đó.
- **Ngôi sao hi vọng (1 lượt/trận)**: Kỹ năng tấn công (High risk, High reward). Kích hoạt khi nhóm đã phân tích triệt để bài toán và tự tin 100% vào đáp án. Trả lời đúng nhận điểm nhân đôi (x2), trả lời sai bị trừ số điểm tương ứng (x2).

### Những Gì Học Được

#### Tư Duy Thiết Kế Hệ Thống
- **Không đoán mò, không giấu sự mơ hồ**: Bối cảnh game show cho thấy việc vội vàng chọn một dịch vụ AWS chỉ vì "nghe có vẻ quen" thường dẫn đến mất điểm. Tư duy kiến trúc yêu cầu phải công khai mọi sự đánh đổi (trade-off) và bám sát vào yêu cầu gốc của đề bài.
- **Tối giản và đúng mục tiêu**: Việc thiết kế kiến trúc cũng giống như viết code: giải pháp tốt nhất là giải pháp đơn giản nhất đáp ứng đúng nhu cầu. Không tự "vẽ" thêm các dịch vụ phức tạp nếu bài toán chỉ yêu cầu lưu trữ tĩnh (S3) thay vì một hệ thống file system đắt đỏ (EFS).

#### Kiến Trúc Kỹ Thuật
- Củng cố mạnh mẽ khả năng phân biệt use-case của các dịch vụ tương đồng trên AWS (ví dụ: khi nào dùng DynamoDB vs RDS, ALB vs NLB, CloudFront vs Global Accelerator).
- Nhận thức rõ ràng về các "anti-patterns" trong thiết kế Cloud, đặc biệt là các lỗi thiết kế gây thâm hụt ngân sách hoặc tạo ra "single point of failure".

#### Chiến Lược & Teamwork
- **Giao tiếp phẫu thuật**: Thời gian bàn luận ngắn đòi hỏi các thành viên phải giao tiếp trực diện, bỏ qua các ý kiến chung chung và đi thẳng vào việc xác minh tính khả thi của từng dịch vụ AWS.
- **Quản lý rủi ro**: Học cách đánh giá độ tin cậy của luồng suy luận trước khi quyết định "xuống tiền" bằng thẻ *Ngôi sao hi vọng* hay lùi lại an toàn với thẻ *Rủi ro tối thiểu*.

### Ứng Dụng Vào Công Việc
- **Xây dựng test-case cho kiến trúc**: Áp dụng tư duy thi đấu vào công việc thực tế: trước khi quyết định tích hợp bất kỳ dịch vụ AWS nào vào dự án, cần phải liệt kê rõ các tiêu chí kiểm chứng (yêu cầu tải, chi phí, độ trễ) giống như việc giải quyết các case study trong game.
- **Lập kế hoạch giải quyết vấn đề**: Khi đối mặt với các task phức tạp hoặc bug hệ thống, luôn tuân thủ nguyên tắc phân rã tình huống. Chia nhỏ vấn đề thành các bước kiểm tra (verify) rõ ràng thay vì đưa ra giải pháp một cách cảm tính.

### Trải nghiệm trong event

Tham gia **Cloud Architect Game Show** mang lại một lăng kính hoàn toàn khác biệt so với các buổi hội thảo một chiều. Đây là nơi tư duy thiết kế kiến trúc bị đưa vào vòng thử thách thực chiến dưới áp lực thời gian. 

#### Sự tương đồng với triết lý lập trình
Điểm thú vị nhất của sự kiện là nó phản ánh chính xác triết lý làm việc mà tôi luôn theo đuổi: "Thận trọng luôn quan trọng hơn Tốc độ". Đứng trước một đề bài kiến trúc hóc búa, sự thôi thúc phải bấm chuông trả lời nhanh rất lớn. Tuy nhiên, nhóm chúng tôi đã thống nhất nguyên tắc: tuyệt đối không đưa ra đáp án dựa trên phán đoán mơ hồ. Mọi quyết định lựa chọn (ví dụ chọn Lambda thay vì EC2) đều phải dựa trên những giả định rõ ràng được bóc tách từ chính "bẫy" ngôn từ của đề bài. 

#### Phối hợp nhóm và ra quyết định
Làm việc cùng 4 thành viên khác trong nhóm là một trải nghiệm giao tiếp tuyệt vời. Việc tranh luận để loại trừ các đáp án sai diễn ra rất căng thẳng nhưng vô cùng logic. Khoảnh khắc đáng nhớ nhất là khi cả nhóm quyết định kích hoạt *Rủi ro tối thiểu* cho một câu hỏi phân bổ tải mạng phức tạp, và sử dụng *Ngôi sao hi vọng* đúng thời điểm cho một tình huống thiết kế decoupling quen thuộc, từ đó lật ngược thế cờ.

#### Bài học rút ra
- Việc hiểu rõ ngọn ngành một dịch vụ (bản chất, điểm mạnh, điểm yếu) quan trọng hơn việc biết tên của hàng chục dịch vụ khác nhau. 
- Trong kiến trúc hệ thống cũng như trong lập trình, mọi quyết định thay đổi hoặc tích hợp công nghệ đều phải có khả năng kiểm chứng rõ ràng. Không tự ý áp dụng các mẫu kiến trúc phức tạp nếu một giải pháp tối thiểu đã có thể giải quyết trọn vẹn bài toán.

#### Một số hình ảnh khi tham gia sự kiện
![Ảnh chụp tại sự kiện](/images/4-EventParticipated/event2.jpg)

> Tổng thể, sự kiện không chỉ là một sân chơi giải trí mà còn là một bài kiểm tra thực tế sắc bén. Nó củng cố niềm tin của tôi vào nguyên tắc làm việc hướng mục tiêu, đặt tính chính xác và tư duy phân tích thận trọng lên hàng đầu khi đối mặt với bất kỳ hệ thống đám mây nào.