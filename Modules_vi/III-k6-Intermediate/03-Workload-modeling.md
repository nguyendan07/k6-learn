# Mô hình hóa khối lượng công việc (Workload modeling)

Biết cần kiểm thử cái gì (trang nào hoặc endpoint nào) là chưa đủ—bạn cũng nên nghĩ về cách thức kiểm thử. Bạn nên mô phỏng bao nhiêu người dùng ảo (virtual users)? Những người dùng đó có tạm dừng thực thi để mô phỏng "thời gian suy nghĩ" (think times) của người dùng thực hay không? Người dùng là người mới hay người cũ? Câu trả lời cho những câu hỏi này có thể ảnh hưởng đến kết quả kiểm thử của bạn.

Quá trình **mô hình hóa khối lượng công việc** (workload modeling) bao gồm việc xác định _cách thức_ áp tải (load) lên hệ thống, và nó rất thiết yếu cho một bài kiểm thử tải (load test) thành công.

Mô hình khối lượng công việc của bạn bị ảnh hưởng nặng nề bởi các tình huống hoặc kịch bản (scenarios) mà bạn muốn kiểm thử. Bài kiểm thử tải của bạn càng mô phỏng gần với những hoàn cảnh đó, thì nó càng mang tính _thực tế_ (realistic). Tính thực tế có thể là mô phỏng lưu lượng truy cập cao điểm trong môi trường production, nhưng nó cũng có thể là mô phỏng lưu lượng nhỏ hơn và có mục tiêu cụ thể đối với một thành phần riêng lẻ trong hệ thống của bạn. Tất cả phụ thuộc vào tình huống mà bạn đang cố gắng tái tạo và kiểm thử.

Nếu kịch bản kiểm thử tải của bạn không đủ thực tế, bạn có thể không đạt được lưu lượng (throughput) kiểm thử dự kiến, hoặc bạn có thể không kiểm thử cùng các thành phần của hệ thống mà người dùng thực truy cập trong môi trường production. Các kịch bản và kịch bản kiểm thử không thực tế có thể dẫn đến kết quả không nhất quán và không chính xác. Nguy hiểm hơn, chúng có thể tạo ra cảm giác tự tin giả tạo về khả năng chịu đựng của hệ thống.

## Những thách thức trong mô hình hóa khối lượng công việc

Việc làm cho các kịch bản và tình huống trở nên thực tế sẽ làm tăng giá trị mà một bài kiểm thử tải có thể mang lại. Tuy nhiên, tính thực tế trong kiểm thử không phải là một nhiệm vụ dễ dàng. Việc tăng tính thực tế của một bài kiểm thử tải thường có thể làm tăng lượng thời gian và công sức cần thiết để tạo và bảo trì bộ kiểm thử của bạn. Cũng có nhiều yếu tố khiến hành vi của con người khó mô phỏng:

- **Máy tính nhanh hơn con người**. Các mô phỏng tự động có thể được thực thi với tốc độ phi nhân tính. Một cỗ máy không cần phải dừng lại để suy nghĩ xem nên làm gì tiếp theo.
- **Hành vi của con người là không thể đoán trước**. Đôi khi, con người không làm những việc logic hoặc hợp lý nhất. Dữ liệu lịch sử có thể giúp xác định chính xác cách người dùng cuối của bạn hành xử và cung cấp thông tin cho hành vi kịch bản kiểm thử tải của bạn.
- **Luồng người dùng (user flows) có thể phức tạp**. Khi các hệ thống phát triển về quy mô, số lượng luồng người dùng mà một bài kiểm thử tải cần mô phỏng để đảm bảo tính thực tế cũng tăng lên. Một bài kiểm thử tải có thể cần bao quát nhiều luồng đầu-cuối (end-to-end flows), mỗi luồng trong đó có thể yêu cầu các tham số kiểm thử khác nhau.
- **Các hệ thống phân tán (distributed systems) đi kèm với nhiều điểm lỗi**. Phần mềm với kiến trúc hướng sự kiện (event-driven) hoặc dựa trên microservices có nhiều thành phần mô-đun, mỗi thành phần trong đó có thể cần được kiểm thử và giám sát.
- **Nhiều hệ thống có nhiều nguồn lưu lượng truy cập.** Vị trí địa lý và tốc độ internet của người dùng ảnh hưởng đến tải (load) mà họ áp lên hệ thống.

Vậy, làm thế nào chúng ta có thể làm cho các bài kiểm thử tự động trở nên thực tế bất chấp những trở ngại này?

## Các yếu tố của một mô hình khối lượng công việc

Khi tạo một mô hình khối lượng công việc, hãy xem xét các biến số này.

### Các tham số kiểm thử (Test parameters)

Các tham số kiểm thử là các giá trị ảnh hưởng đến cách kịch bản kiểm thử tải của bạn được thực thi. Các tham số bao gồm:

- Thời lượng (Duration)
- Số lượng VUs
- Số lượng lần lặp (iterations)
- Hồ sơ tải (Load profile), bao gồm các giai đoạn (stages) trong quá trình kiểm thử

Mỗi tham số này ảnh hưởng đến lượng và loại tải được mô phỏng.

Hãy xem [k6 Load Test Options](../II-k6-Foundations/06-k6-Load-Test-Options.md) để biết thêm thông tin về các tham số này, hoặc [Setting load profiles with executors](08-Setting-load-profiles-with-executors.md) để biết hướng dẫn về cách triển khai chúng trong k6.

### Thời gian suy nghĩ (Think time) và điều phối nhịp độ (pacing)

Think time và pacing đều là các loại trì hoãn trong kịch bản nhằm mô phỏng các khoảng tạm dừng mà một người dùng thực có khả năng thực hiện khi truy cập ứng dụng. Think time, thường được gọi là sleep, có thể được thêm vào trước hoặc sau các hành động trong kịch bản, trong khi pacing thường được thêm vào giữa các lần lặp (iterations).

Các khoảng trì hoãn dài hơn đồng nghĩa với ít yêu cầu hơn trong một thời lượng cố định, và trì hoãn ngắn hơn đồng nghĩa với nhiều yêu cầu hơn. Số lượng yêu cầu và tốc độ gửi yêu cầu có thể thay đổi cách hệ thống của bạn phản hồi. Chúng tôi cũng khuyên bạn nên sử dụng các khoảng trì hoãn động nếu bạn muốn làm cho tải được tạo ra trông bớt đều đặn và đồng nhất hơn.

Xem [Adding think time using sleep](../II-k6-Foundations/05-Adding-think-time-using-sleep.md) để biết cách triển khai các khoảng trì hoãn trong k6.

### Thêm các tài nguyên tĩnh (static resources)

Trong các ứng dụng web, tài nguyên tĩnh đề cập đến hình ảnh, kịch bản phía máy khách (client-side scripts), phông chữ và các tệp khác được nhúng vào trang. Nếu bạn muốn kịch bản của mình truy cập trang đó, bạn phải quyết định xem bạn có muốn kịch bản cũng tải xuống các tài nguyên đó hay không.

Việc tải xuống các tài nguyên tĩnh làm cho kịch bản thực tế hơn nếu bạn muốn mô phỏng hành vi của người dùng cuối, vì các trình duyệt web tự động tải xuống chúng. Tuy nhiên, nếu bạn chỉ muốn tải xuống mã HTML của trang (ví dụ, có thể vì hình ảnh được cung cấp bởi một CDN (Content Delivery Network) mà bạn không muốn kiểm thử), thì việc _không_ tải xuống các tài nguyên tĩnh có thể sẽ thận trọng hơn.

### Các yêu cầu song song (Parallel requests)

Các yêu cầu song song là các yêu cầu được gửi đồng thời. Các trình duyệt hiện đại yêu cầu một số lượng tài nguyên tĩnh nhất định cùng một lúc, vì vậy nếu kịch bản của bạn yêu cầu chúng một cách tuần tự, điều đó có thể thay đổi tải áp lên hệ thống của bạn.

Tham khảo [Parallel requests in k6](05-Parallel-requests-in-k6.md) để biết hướng dẫn về việc sử dụng batching để triển khai các yêu cầu song song.

### Hành vi của bộ nhớ đệm (cache) và cookie

Khi người dùng truy cập một trang web, một số tài nguyên có thể được lưu trong bộ nhớ đệm (cache) để các yêu cầu tiếp theo không yêu cầu tải lại các tài nguyên đó. Cookie là những mẩu thông tin nhỏ về các hoạt động trước đó của người dùng (chẳng hạn như lần cuối họ truy cập trang web) được lưu lại cho các mục đích chức năng, phân tích hoặc tiếp thị.

Cả cache và cookie đều góp phần vào tổng tải mà một kịch bản tạo ra và nên được điều chỉnh cho phù hợp với mục tiêu kiểm thử. Những người lần đầu truy cập trang web sẽ không có tài nguyên được lưu cache cục bộ, nhưng những người truy cập lặp lại có thể đang lấy tài nguyên từ cache.

Trong k6, bạn có thể thiết lập các tùy chọn cache [bằng cách sử dụng headers](https://k6.io/docs/using-k6/http-requests/#making-http-requests) và [quản lý cookie theo một vài cách](https://k6.io/docs/examples/cookies-example/).

### Dữ liệu kiểm thử (Test data)

Việc yêu cầu cùng một tài nguyên lặp đi lặp lại trong kịch bản của bạn có thể dẫn đến một số vấn đề. Nó có thể kích hoạt việc lưu cache ở phía máy chủ, gây ra các lỗi bảo mật nếu kịch bản của bạn đăng nhập lặp lại với cùng một người dùng, và giới hạn phạm vi kiểm thử của bạn vì các tài nguyên khác không được yêu cầu.

## Kiểm tra kiến thức của bạn

### Câu hỏi 1

Vấn đề nào sau đây có thể gây ra bởi việc mô hình hóa khối lượng công việc không đầy đủ hoặc không chính xác?

A: Công cụ kiểm thử tải được chọn không thể tạo ra tải yêu cầu và các kịch bản phải được viết lại bằng một công cụ khác.

B: Các sự cố liên quan đến hiệu năng trong môi trường production xảy ra bất chấp các bài kiểm thử tải thành công trước đó.

C: Công sức phát triển bị lãng phí vào các tính năng mà người dùng không muốn.

### Câu hỏi 2

Sự thật nào sau đây về một ứng dụng giả định là không liên quan khi xây dựng mô hình khối lượng công việc?

A: Hầu hết người dùng cuối sống ở Úc.

B: Hầu hết người dùng cuối là những bà mẹ đơn thân.

C: Hầu hết người dùng cuối truy cập ứng dụng trên điện thoại di động của họ, sử dụng mạng 3G/4G.

### Câu hỏi 3

Thay đổi nào sau đây sẽ làm tăng tải (load) lên một máy chủ ứng dụng?

A: Giảm thời gian suy nghĩ (think time)

B: Bật tính năng lưu bộ nhớ đệm (caching)

C: Tắt tính năng tải xuống các tài nguyên tĩnh

### Đáp án

1. B. Các sự cố trong môi trường production bất chấp việc đã kiểm thử hiệu năng là một triệu chứng kinh điển cho thấy tình huống trong quá trình kiểm thử tải không khớp với tình huống trong thực tế. Điều này có thể do nhiều yếu tố, một trong số đó là mô hình hóa khối lượng công việc.
2. B. Vị trí địa lý của người dùng có liên quan vì các Mạng phân phối nội dung (CDNs) không có máy chủ ở mọi quốc gia, và sự vắng mặt của chúng có thể ảnh hưởng đến tổng thời gian phản hồi của người dùng. Tốc độ mạng của người dùng cũng có thể ảnh hưởng tương tự đến thời gian phản hồi. B là câu trả lời đúng vì việc người dùng có phải là mẹ đơn thân hay không có liên quan đến nhân khẩu học kinh doanh, nhưng không liên quan đến thời gian phản hồi.
3. A. Giảm think time sẽ khiến kịch bản được thực thi nhanh hơn, và trong một số trường hợp có thể góp phần làm tăng mức sử dụng tài nguyên trên cả trình tạo tải và máy chủ ứng dụng. Việc bật caching và tắt tài nguyên tĩnh có tác dụng ngược lại.
