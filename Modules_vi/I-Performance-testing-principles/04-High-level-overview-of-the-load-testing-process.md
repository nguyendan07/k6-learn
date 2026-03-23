# Tổng quan cấp cao về quy trình kiểm thử tải

Trong phần trước, bạn đã tìm hiểu kiểm thử tải là gì, nó khác với kiểm thử hiệu năng như thế nào và các loại kịch bản kiểm thử của nó là gì. Trong phần này, bạn sẽ tìm hiểu về các giai đoạn khác nhau của quy trình kiểm thử tải:

- Lập kế hoạch cho kiểm thử tải
- Viết kịch bản cho một bài kiểm thử tải
- Thực thi các bài kiểm thử tải
- Phân tích kết quả kiểm thử tải

Để rõ ràng, các hoạt động này được hiển thị dưới dạng các giai đoạn riêng biệt ở đây; tuy nhiên, trong thực tế, chúng thường chồng chéo lên nhau. Giống như quy trình phát hành ứng dụng, quy trình kiểm thử nên diễn ra liên tục và theo mô hình Agile, với mỗi bước tăng trưởng nhỏ được xây dựng dựa trên công việc trước đó, ngày càng trở nên mạnh mẽ hơn theo thời gian.

![Continuous Testing Snowball](../../images/continuous-testing-snowball.png)

Giống như mã nguồn ứng dụng, một bài kiểm tra bắt đầu một cách chậm rãi. Tại đỉnh núi, bài kiểm tra ở dạng đơn giản nhất. Bạn có thể coi một bài kiểm tra ở giai đoạn đầu chủ yếu là **dựa trên rủi ro (risk-based)**: kiểm thử ở cấp độ này là một phản ứng đối với các thất bại có khả năng xảy ra. Các bài kiểm tra này được viết cụ thể để ngăn chặn thất bại trong các thành phần quan trọng nhất của ứng dụng, và có thể không có đủ thời gian cho bất cứ việc gì khác.

Khi việc kiểm thử trưởng thành hơn trong dự án, nó có thể bắt đầu bao gồm **kiểm thử hồi quy (regression)**. Giờ đây, khi tất cả các khu vực có rủi ro cao đã được bao phủ, các đội ngũ có thời gian để làm cho các bài kiểm tra có tính tương thích ngược và mang tính phòng ngừa cao hơn một chút, và họ viết các bài kiểm tra để xem liệu mã mới có làm hỏng các chức năng cũ hay không.

Quả cầu tuyết Kiểm thử Liên tục (Continuous Testing Snowball) lấy tốc độ khi đội ngũ phát triển cùng với bộ kiểm thử, tập trung vào việc **tự động hóa (automating)** ngày càng nhiều phần của quy trình kiểm thử sau mỗi lần lặp hoặc sprint. Một khung làm việc có thể lặp lại được thiết lập. Ở giai đoạn này, quả cầu tuyết đang ở tốc độ nhanh nhất.

Ở dưới chân dốc, kiểm thử liên tục đã phát triển việc kiểm thử đến mức đội ngũ có thể mở rộng trọng tâm từ việc giải quyết các lỗi cụ thể sang việc tăng cường **độ tin cậy và sự tự tin (reliability and confidence)** tổng thể vào khả năng chịu đựng các sự kiện bất ngờ của ứng dụng. Khung làm việc thậm chí còn được làm cho mạnh mẽ hơn với các loại kiểm thử mang tính khám phá cao hơn như Chaos Engineering.

Trọng tâm của Kiểm thử Liên tục là phát triển bộ kiểm thử một cách hữu cơ và lặp đi lặp lại song song với ứng dụng, bắt đầu từ một thứ gì đó nhỏ với mục tiêu thực hiện đủ các thay đổi tăng dần cho đến khi nó trở nên mạnh mẽ hơn.

## Lập kế hoạch cho kiểm thử tải

Lập kế hoạch cho một bài kiểm thử tải là phần đầu tiên của quy trình. Nó liên quan đến việc xác định lý do _tại sao_ cần kiểm thử, tìm ra _cái gì_ cần kiểm thử và phác thảo cách thức kiểm thử nói chung.

Trong giai đoạn này, chúng ta xây dựng các yêu cầu cho kiểm thử tải:

- Làm rõ phạm vi kiểm thử
- Xác định các SLOs
- Xác định các mô hình khối lượng công việc (workload models)
- Thiết lập môi trường để kiểm thử, bao gồm cả việc giám sát
- Thống nhất về tần suất và lịch trình kiểm thử
- Chuẩn bị dữ liệu kiểm thử, nếu có

Lập kế hoạch cho bất kỳ hoạt động kiểm thử nào đều là một hoạt động của đội ngũ, và kiểm thử tải cũng không ngoại lệ. Giai đoạn này là cơ hội để tất cả các bên liên quan cùng nhau hiểu việc kiểm thử sẽ như thế nào và điều gì sẽ định nghĩa một vòng kiểm thử thành công.

## Viết kịch bản cho một bài kiểm thử tải

Viết kịch bản cho bài kiểm thử tải liên quan đến việc chuyển đổi kế hoạch kiểm thử thành các bài kiểm tra có thể thực thi được. Trong giai đoạn này, bạn có thể thực hiện một số việc sau:

- Tạo các kịch bản kiểm thử bao quát đầy đủ các yêu cầu
- Viết kịch bản kiểm thử bằng các công cụ kiểm thử tải
- Làm cho các kịch bản trở nên thực tế
- Chạy các bài shakeout tests để xác minh rằng kịch bản hoạt động như mong đợi
- Chạy các bài kiểm tra đối với các môi trường thượng nguồn, thường là dev hoặc staging
- Chia sẻ kịch bản kiểm thử với đội ngũ
- Thiết lập một khung kiểm thử có thể phát triển cùng với bộ kiểm thử

Mặc dù các bài kiểm tra có thể được thực thi trong khi viết kịch bản, nhưng chúng thường nhằm mục đích gỡ lỗi (debugging) hoặc sàng lọc hơn là các bài kiểm thử tải đầy đủ. Giai đoạn viết kịch bản có thể lấn sang giai đoạn thực thi kiểm thử khi các thay đổi được thực hiện đối với các kịch bản hiện có hoặc các kịch bản mới được tạo ra để giải quyết các vấn đề tìm thấy trong quá trình thực thi kiểm thử.

## Thực thi các bài kiểm thử tải

Trong quá trình thực thi kiểm thử, các kịch bản kiểm thử tải chạy đối với các mục tiêu dự định của chúng, thường là môi trường kiểm thử hoặc production. Trong giai đoạn thực thi kiểm thử, bạn có thể thực hiện một số việc sau:

- Thiết lập hạ tầng đám mây hoặc tại chỗ (on-premise)
- Thiết lập các công cụ observability để giám sát sức khỏe của máy chủ ứng dụng và trình tạo tải (load-generator)
- Chạy các bài shakeout tests để xác minh rằng môi trường kiểm thử hoạt động như mong đợi
- Chạy các bài kiểm thử cơ sở (baseline tests) để hiểu hành vi hiện tại của hệ thống
- Thực hiện các thay đổi đối với mã hoặc môi trường và chạy các bài kiểm tra để so sánh với bài kiểm thử cơ sở
- Mở rộng phạm vi kiểm thử
- Thực hiện kiểm thử phân tán bằng cách tăng tải hoặc mở rộng quy mô bài kiểm thử tải

Thực thi kiểm thử tải không chỉ đơn thuần là chạy các bài kiểm tra, bởi vì bản thân các bài kiểm tra có thể không hữu ích nếu không có các công cụ observability thích hợp để giám sát sức khỏe của hệ thống và sức khỏe của chính bài kiểm tra đó.

## Phân tích kết quả kiểm thử tải

Trong giai đoạn phân tích, bạn có thể thực hiện một số việc sau:

- Đối chiếu dữ liệu từ các máy chủ ứng dụng cũng như từ các trình tạo tải
- Tổng hợp dữ liệu để có cái nhìn tổng quát về những gì đã xảy ra trong quá trình kiểm thử
- Sử dụng các công cụ trực quan hóa và phân tích dữ liệu để xác định cách hệ thống hành xử trong các điều kiện kiểm thử
- Báo cáo các phát hiện cho các bên liên quan
- Khắc phục các nút thắt cổ chai
- Chạy lại các bài kiểm tra để xác nhận các vấn đề hoặc các bản sửa lỗi

Sử dụng dữ liệu thu thập được trong quá trình kiểm thử để hiểu cách hệ thống hành xử trong các điều kiện kiểm thử khác nhau là một phần quan trọng tạo nên giá trị của kiểm thử tải. Điều quan trọng là phải xem xét dữ liệu một cách khách quan và truyền đạt kết quả cho đội ngũ một cách dễ hiểu.

## Kiểm thử tải liên tục

Kiểm thử tải liên tục là một thực hành trải dài trên tất cả các giai đoạn kiểm thử. Trong kiểm thử liên tục, bạn có thể:

- Thêm các kịch bản kiểm thử tải vào một kho lưu trữ được quản lý phiên bản (version-controlled repository)
- Tích hợp các bài kiểm thử tải vào đường ống CI/CD
- Tự động hóa việc báo cáo
- Thiết lập thông báo cho các bài kiểm tra thất bại
- Thiết lập một khung làm việc có thể lặp lại để chạy các bài kiểm thử tải, một khung gắn liền với các thay đổi mã hoặc chu kỳ phát hành
- Lưu lại kết quả kiểm thử vào cơ sở dữ liệu để xem các xu hướng lịch sử

Nếu không đảm bảo rằng việc kiểm thử tải được thực hiện liên tục, kiểm thử tải có thể trở thành một quy trình diễn ra một lần duy nhất. Việc tích hợp nó vào các đường ống CI/CD hiện có giúp mọi người liên quan luôn lưu tâm đến hiệu năng.

Khi bộ kiểm thử phát triển về độ chín và phạm vi, các đội ngũ nên tạo ra các khung làm việc hiệu quả hơn một cách tự nhiên để chạy các bài kiểm tra, quản lý thông báo, phân tích kết quả và báo cáo. Bằng cách này, việc kiểm thử có thể bắt đầu rất đơn giản, thường xoay quanh các chức năng quan trọng nhất hoặc có rủi ro cao nhất của ứng dụng, sau đó phát triển và cải thiện một cách hữu cơ cùng với ứng dụng.

## Kiểm tra kiến thức của bạn

### Câu hỏi 1

Phát biểu nào sau đây là đúng?

A: Thực hiện các hoạt động quan trọng hơn việc có một sự phân biệt rõ ràng giữa các giai đoạn của quy trình kiểm thử tải.

B: Các giai đoạn của quy trình kiểm thử tải luôn phải được thực hiện theo trình tự, để không bỏ sót bất kỳ hoạt động quan trọng nào.

C: Lập kế hoạch cho một bài kiểm thử tải tốt nhất nên được thực hiện bởi các quản lý kiểm thử, những người ở vị trí tốt nhất để hiểu những gì được yêu cầu từ kiểm thử tải.

### Câu hỏi 2

Thời điểm tốt nhất để bắt đầu nghĩ về các công cụ kiểm thử tải và observability là khi nào?

A: Lập kế hoạch (Planning)

B: Viết kịch bản (Scripting)

C: Thực thi (Execution)

### Câu hỏi 3

Tại sao kiểm thử tải liên tục lại quan trọng?

A: Tự động hóa kiểm thử tải giúp xác minh hành vi của hệ thống theo thời gian.

B: Việc tích hợp kiểm thử tải vào các đường ống CI/CD đồng nghĩa với việc không cần thuê những người kiểm thử tải chuyên dụng.

C: Chạy các bài kiểm thử tải tự động sẽ ngăn chặn các lập trình viên đưa vào những đoạn mã không đạt hiệu năng.

### Đáp án

1. A. Mặc dù việc nghĩ về các giai đoạn riêng biệt trong kiểm thử có thể hữu ích để hiểu quy trình, nhưng trong thực tế, kiểm thử nên được tích hợp chặt chẽ với các hoạt động khác trong vòng đời phát triển phần mềm.
2. A. Ngăn xếp kiểm thử và observability được chọn có thể có tác động đến loại hình kiểm thử bạn thực hiện và kết quả kiểm thử của bạn, vì vậy chúng tôi khuyên bạn nên xem xét các lựa chọn của mình càng sớm càng tốt.
3. A. Việc tích hợp kiểm thử tải vào các đường ống tích hợp liên tục giúp bạn thấy được các xu hướng về hiệu năng theo thời gian.
