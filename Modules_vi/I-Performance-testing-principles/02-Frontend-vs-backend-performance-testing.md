Để có một cái nhìn toàn diện về hiệu năng, những người kiểm thử cần kiểm thử cả frontend và backend của một ứng dụng. Mặc dù có một số điểm tương đồng về công cụ và kỹ thuật, nhưng cách tiếp cận và trọng tâm sẽ khác nhau khi kiểm thử các phần khác nhau của ứng dụng.

## Kiểm thử hiệu năng Frontend

Kiểm thử hiệu năng frontend xác minh hiệu năng ứng dụng ở cấp độ giao diện, đo lường các chỉ số vòng lặp (round-trip metrics) xem xét cách thức và thời điểm các thành phần trang xuất hiện trên màn hình. Nó quan tâm đến trải nghiệm người dùng cuối của một ứng dụng, thường liên quan đến trình duyệt.

Kiểm thử hiệu năng frontend xuất sắc trong việc xác định các vấn đề ở cấp độ vi mô nhưng không bộc lộ các vấn đề trong kiến trúc nền tảng của một hệ thống.

Bởi vì nó chủ yếu đo lường trải nghiệm của một người dùng duy nhất đối với hệ thống, kiểm thử hiệu năng frontend có xu hướng dễ thực hiện hơn ở quy mô nhỏ.
Kiểm thử hiệu năng frontend có các chỉ số khác biệt với kiểm thử hiệu năng backend. Kiểm thử hiệu năng frontend kiểm tra những thứ như:

- Liệu các trang của ứng dụng có được tối ưu hóa để hiển thị nhanh chóng trên màn hình của người dùng hay không
- Mất bao lâu để người dùng tương tác với các thành phần UI của ứng dụng.

Một số mối quan tâm khi thực hiện loại kiểm thử hiệu năng này là sự phụ thuộc của nó vào các môi trường được tích hợp đầy đủ và chi phí mở rộng. Bạn chỉ có thể kiểm thử hiệu năng frontend sau khi mã ứng dụng và hạ tầng đã được tích hợp với giao diện người dùng. Các công cụ để tự động hóa kiểm thử frontend cũng vốn dĩ tốn nhiều tài nguyên hơn, vì vậy chúng có thể tốn kém khi chạy ở quy mô lớn và không phù hợp cho các bài kiểm tra tải cao.

### Tại sao kiểm thử hiệu năng frontend là chưa đủ?

Vì kiểm thử hiệu năng frontend đã đo lường trải nghiệm người dùng cuối, tại sao chúng ta vẫn cần kiểm thử hiệu năng backend?

Các công cụ kiểm thử frontend được thực thi ở phía máy khách (client side) và bị giới hạn về phạm vi. Chúng không cung cấp đủ thông tin về các thành phần backend để tinh chỉnh ngoài giao diện người dùng.

Sự hạn chế này có thể dẫn đến sự tự tin giả tạo vào hiệu năng tổng thể của ứng dụng khi lượng lưu lượng truy cập vào ứng dụng tăng lên. Trong khi thành phần frontend của thời gian phản hồi vẫn giữ nguyên ít nhiều, thành phần backend của thời gian phản hồi tăng theo cấp số nhân với số lượng người dùng đồng thời (concurrency):

![Một biểu đồ tổng hợp thời gian phản hồi frontend và backend. Khi concurrency tăng lên, thời gian phản hồi backend trở nên dài hơn nhiều.](../../images/frontend-backend.png)

Việc chỉ kiểm thử hiệu năng frontend sẽ bỏ qua một phần lớn của ứng dụng, phần dễ bị lỗi và gặp nút thắt cổ chai về hiệu năng hơn ở các mức tải cao hơn.

## Kiểm thử hiệu năng Backend

Kiểm thử hiệu năng backend nhắm vào các máy chủ ứng dụng nền tảng để xác minh tính khả năng mở rộng (scalability), tính linh hoạt (elasticity), tính sẵn sàng (availability), độ tin cậy (reliability), và khả năng phản hồi (responsiveness) của toàn bộ hệ thống.

- _Scalability_: Hệ thống có thể điều chỉnh theo mức độ nhu cầu tăng dần đều không?
- _Elasticity_: Hệ thống có thể tiết kiệm tài nguyên trong các giai đoạn nhu cầu thấp hơn không?
- _Availability_: Thời gian hoạt động (uptime) của từng thành phần trong hệ thống là bao nhiêu?
- _Reliability_: Hệ thống có phản hồi nhất quán trong các điều kiện môi trường khác nhau không?
- _Resiliency_: Hệ thống có thể chịu đựng các sự kiện bất ngờ một cách nhẹ nhàng không?
- _Latency_: Hệ thống xử lý và phản hồi các yêu cầu nhanh như thế nào?

Kiểm thử backend có phạm vi rộng hơn kiểm thử hiệu năng frontend. Kiểm thử API có thể được sử dụng để nhắm mục tiêu vào các thành phần cụ thể hoặc các thành phần đã tích hợp, nghĩa là các đội ngũ ứng dụng có linh hoạt hơn và có cơ hội cao hơn để tìm thấy các vấn đề hiệu năng sớm hơn. Kiểm thử backend ít tốn tài nguyên hơn so với kiểm thử hiệu năng frontend và do đó phù hợp hơn để tạo tải cao.

Một số mối quan tâm khi thực hiện loại kiểm thử này là khả năng không thể kiểm thử "dặm đầu tiên" (the first mile) của trải nghiệm người dùng và phạm vi rộng. Kiểm thử backend liên quan đến việc truyền tin ở cấp độ giao thức thay vì tương tác với các thành phần trang. Nó xác minh nền tảng của một ứng dụng thay vì lớp cao nhất của nó mà người dùng cuối cùng nhìn thấy. Tùy thuộc vào độ phức tạp của kiến trúc ứng dụng, kiểm thử backend cũng có thể có phạm vi mở rộng hơn.

## Kiểm tra kiến thức của bạn

### Câu hỏi 1

k6 xuất sắc trong loại kiểm thử nào?

A: Backend testing

B: Manual testing

C: Accessibility testing

### Câu hỏi 2

Điều nào sau đây là một lợi thế của kiểm thử hiệu năng backend?

A: Nó cung cấp các chỉ số như Time To Interactive (TTI) để đo lường thời điểm người dùng có thể tương tác với ứng dụng lần đầu tiên.

B: Nó mô phỏng người dùng bằng cách điều khiển các trình duyệt thực để kiểm thử ứng dụng.

C: Nó có thể nhắm mục tiêu vào các thành phần ứng dụng trước khi chúng được tích hợp.

D: A và B.

### Câu hỏi 3

Phát biểu nào sau đây là đúng?

A: Kết quả kiểm thử hiệu năng frontend không bị ảnh hưởng bởi các nút thắt cổ chai ở backend của ứng dụng.

B: Kiểm thử hiệu năng frontend luôn được thực hiện với một người dùng tự động duy nhất.

C: Kiểm thử hiệu năng frontend có thể xác minh trải nghiệm người dùng theo những cách mà kiểm thử hiệu năng backend không thể.

### Đáp án

1. A. k6 không thể thực hiện kiểm thử thủ công (manual testing) hoặc kiểm thử khả năng truy cập (accessibility testing) tại thời điểm này.
2. C. A và B sai vì chúng đang đề cập đến kiểm thử hiệu năng frontend. Kiểm thử hiệu năng backend không cung cấp các chỉ số frontend như TTI và nó không tương tác với ứng dụng thông qua trình duyệt.
3. C. A sai vì các nút thắt cổ chai ở hiệu năng backend cũng có thể ảnh hưởng đến hiệu năng frontend. B sai vì các kịch bản kiểm thử frontend có thể được thực thi như một phần của bài kiểm tra tải. C đúng vì kiểm thử hiệu năng frontend đo lường trải nghiệm của người dùng thực từ trình duyệt, điều mà kiểm thử backend không thể đo lường được.
