Trong phần trước, bạn đã chạy bài kiểm tra k6 đầu tiên của mình—nhưng làm thế nào để bạn biết điều gì đã xảy ra trong bài kiểm tra đó?

Trong phần này, bạn sẽ học cách hiểu đầu ra mặc định của k6 và xác định xem kịch bản của bạn có hoạt động như mong muốn hay không.

## Báo cáo tóm tắt kết thúc kiểm thử (End-of-test summary report)

Dưới đây là kết quả đầu ra đó một lần nữa:

```plain
$ k6 run test.js

          /\      |‾‾| /‾‾/   /‾‾/
     /\  /  \     |  |/  /   /  /
    /  \/    \    |     (   /   ‾‾\
   /          \   |  |\  \ |  (‾)  |
  / __________ \  |__| \__\ \_____/ .io

  execution: local
     script: test.js
     output: -

  scenarios: (100.00%) 1 scenario, 1 max VUs, 10m30s max duration (incl. graceful stop):
           * default: 1 iterations for each of 1 VUs (maxDuration: 10m0s, gracefulStop: 30s)

INFO[0001] Hello world!                                  source=console

running (00m00.7s), 0/1 VUs, 1 complete and 0 interrupted iterations
default ✓ [======================================] 1 VUs  00m00.7s/10m0s  1/1 iters, 1 per VU

     data_received..................: 5.9 kB 9.0 kB/s
     data_sent......................: 564 B  860 B/s
     http_req_blocked...............: avg=524.18ms min=524.18ms med=524.18ms max=524.18ms p(90)=524.18ms p(95)=524.18ms
     http_req_connecting............: avg=123.28ms min=123.28ms med=123.28ms max=123.28ms p(90)=123.28ms p(95)=123.28ms
     http_req_duration..............: avg=130.19ms min=130.19ms med=130.19ms max=130.19ms p(90)=130.19ms p(95)=130.19ms
       { expected_response:true }...: avg=130.19ms min=130.19ms med=130.19ms max=130.19ms p(90)=130.19ms p(95)=130.19ms
     http_req_failed................: 0.00%  ✓ 0        ✗ 1
     http_req_receiving.............: avg=165µs    min=165µs    med=165µs    max=165µs    p(90)=165µs    p(95)=165µs
     http_req_sending...............: avg=80µs     min=80µs     med=80µs     max=80µs     p(90)=80µs     p(95)=80µs
     http_req_tls_handshaking.......: avg=399.48ms min=399.48ms med=399.48ms max=399.48ms p(90)=399.48ms p(95)=399.48ms
     http_req_waiting...............: avg=129.94ms min=129.94ms med=129.94ms max=129.94ms p(90)=129.94ms p(95)=129.94ms
     http_reqs......................: 1      1.525116/s
     iteration_duration.............: avg=654.72ms min=654.72ms med=654.72ms max=654.72ms p(90)=654.72ms p(95)=654.72ms
     iterations.....................: 1      1.525116/s

```

Đây là báo cáo tóm tắt kết thúc kiểm thử. Đó là cách mặc định mà k6 hiển thị kết quả kiểm thử.

Hãy cùng xem xét từng dòng một.

### Các tham số thực thi (Execution parameters)

```plain
execution: local
```

Bạn có thể sử dụng k6 OSS để chạy các kịch bản kiểm thử cục bộ (`local`) hoặc trên k6 Cloud (`cloud`).
Trong bài kiểm tra này, kịch bản kiểm thử đã được thực thi trên máy cục bộ của bạn.

```plain
script: test.js`
```

Đây là tên tệp của kịch bản đã được thực thi.

```plain
output: -`
```

Điều này cho thấy hành vi mặc định: k6 đã in kết quả kiểm thử của bạn ra đầu ra tiêu chuẩn (standard output).

k6 cũng có thể [xuất kết quả ở các định dạng khác](https://grafana.com/docs/k6/latest/get-started/results-output/#time-series-and-external-outputs). Các tùy chọn này, khi được sử dụng, sẽ được hiển thị trong `output`.

```plain
scenarios: (100.00%) 1 scenario, 1 max VUs, 10m30s max duration (incl. graceful stop):
```

Một kịch bản thực thi ([scenario](https://k6.io/docs/misc/glossary/#scenario)) là một tập hợp các hướng dẫn về việc chạy một bài kiểm tra: mã nào nên chạy, khi nào và tần suất chạy như thế nào, và các tham số có thể cấu hình khác. Trong trường hợp này, bài kiểm tra đầu tiên của bạn đã được thực thi bằng các tham số mặc định: một scenario, một [người dùng ảo (virtual user - VU)](https://k6.io/docs/misc/glossary/#virtual-user), và thời lượng tối đa là 10 phút 30 giây.

Thời lượng tối đa (max duration) là giới hạn thời gian thực thi; đó là thời gian mà sau đó bài kiểm tra sẽ bị dừng cưỡng bức. Trong trường hợp này, k6 đã nhận được phản hồi cho yêu cầu trong kịch bản từ lâu trước khi khoảng thời gian này trôi qua.

[Graceful stop](https://k6.io/docs/misc/glossary/#graceful-stop) (dừng nhẹ nhàng) là một khoảng thời gian ở cuối bài kiểm tra khi k6 hoàn thành bất kỳ lần lặp ([iterations](https://k6.io/docs/misc/glossary/#iteration)) nào đang chạy, nếu có thể. Theo mặc định, k6 bao gồm một khoảng graceful stop là 30 giây trong tổng thời lượng tối đa là 10 phút 30 giây.

```plain
* default: 1 iterations for each of 1 VUs (maxDuration: 10m0s, gracefulStop: 30s)
```

`default` ở đây đề cập đến tên của scenario. Vì kịch bản kiểm thử không thiết lập tên cụ thể nào, k6 đã sử dụng tên mặc định.

Một lần lặp (iteration) là một vòng lặp thực thi duy nhất của bài kiểm tra. Các bài kiểm thử tải thường bao gồm các vòng lặp thực thi lặp đi lặp lại trong một khoảng thời gian nhất định để các yêu cầu được thực hiện liên tục. Trừ khi có chỉ định khác, k6 sẽ chạy qua hàm mặc định một lần.

Hãy coi một người dùng ảo (virtual user) như một luồng (thread) hoặc phiên bản duy nhất cố gắng mô phỏng một người dùng cuối thực sự của ứng dụng. Trong trường hợp này, k6 đã khởi chạy một người dùng ảo để chạy bài kiểm tra.

### Đầu ra console (Console output)

Phần này của tóm tắt kết thúc kiểm thử thường để trống, nhưng kịch bản kiểm thử của bạn có bao gồm một dòng để lưu một phần của thân phản hồi ra console (`console.log(response.json().data);`). Đây là giao diện của nó trong báo cáo:

```plain
INFO[0001] Hello world!                                  source=console`
```

Endpoint mục tiêu của kịch bản kiểm thử, `https://httpbin.test.k6.io/post`, trả về bất cứ thứ gì đã được gửi trong phần thân POST, vì vậy đây là một dấu hiệu tốt! Endpoint mục tiêu đã nhận được chuỗi `Hello world!` mà bạn đã gửi trong kịch bản và gửi lại chính phần thân đó.

Nếu bạn đã sử dụng nhiều câu lệnh `console.log()` trong kịch bản kiểm thử, tất cả chúng sẽ xuất hiện trong phần này.

### Tóm tắt thực thi (Execution summary)

Tóm tắt thực thi hiển thị cái nhìn tổng quan về những gì đã xảy ra trong quá trình chạy kiểm thử.

```plain
running (00m00.7s), 0/1 VUs, 1 complete and 0 interrupted iterations
default ✓ [======================================] 1 VUs  00m00.7s/10m0s  1/1 iters, 1 per VU
```

Trong trường hợp này, bài kiểm tra đã chạy trong 0,7 giây với 1 VU. Một lần lặp duy nhất đã được thực thi và hoàn thành đầy đủ (nghĩa là nó không bị gián đoạn). 1 lần lặp trên mỗi VU đã được thực thi (tổng cộng là 1).

### Các chỉ số tích hợp của k6 (k6 built-in metrics)

Bây giờ là các chỉ số ([metrics](https://k6.io/docs/misc/glossary/#metric))! k6 đi kèm với [nhiều chỉ số tích hợp sẵn](https://k6.io/docs/using-k6/metrics/#built-in-metrics).

```plain
     data_received..................: 5.9 kB 9.0 kB/s
     data_sent......................: 564 B  860 B/s
     http_req_blocked...............: avg=524.18ms min=524.18ms med=524.18ms max=524.18ms p(90)=524.18ms p(95)=524.18ms
     http_req_connecting............: avg=123.28ms min=123.28ms med=123.28ms max=123.28ms p(90)=123.28ms p(95)=123.28ms
     http_req_duration..............: avg=130.19ms min=130.19ms med=130.19ms max=130.19ms p(90)=130.19ms p(95)=130.19ms
       { expected_response:true }...: avg=130.19ms min=130.19ms med=130.19ms max=130.19ms p(90)=130.19ms p(95)=130.19ms
     http_req_failed................: 0.00%  ✓ 0        ✗ 1
     http_req_receiving.............: avg=165µs    min=165µs    med=165µs    max=165µs    p(90)=165µs    p(95)=165µs
     http_req_sending...............: avg=80µs     min=80µs     med=80µs     max=80µs     p(90)=80µs     p(95)=80µs
     http_req_tls_handshaking.......: avg=399.48ms min=399.48ms med=399.48ms max=399.48ms p(90)=399.48ms p(95)=399.48ms
     http_req_waiting...............: avg=129.94ms min=129.94ms med=129.94ms max=129.94ms p(90)=129.94ms p(95)=129.94ms
     http_reqs......................: 1      1.525116/s
     iteration_duration.............: avg=654.72ms min=654.72ms med=654.72ms max=654.72ms p(90)=654.72ms p(95)=654.72ms
     iterations.....................: 1      1.525116/s
```

Các chỉ số sau đây thường là quan trọng nhất để phân tích kiểm thử.

#### Thời gian phản hồi (Response time)

```plain
http_req_duration..............: avg=130.19ms min=130.19ms med=130.19ms max=130.19ms p(90)=130.19ms p(95)=130.19ms
```

"Thời gian phản hồi" có thể mơ hồ, vì nó có thể được chia thành nhiều thành phần. Tuy nhiên, trong hầu hết các trường hợp, `http_req_duration` là chỉ số bạn đang tìm kiếm. Nó bao gồm:

- `http_req_sending` (thời gian gửi dữ liệu đến máy chủ mục tiêu)
- `http_req_waiting` (Thời gian đến byte đầu tiên hoặc "TTFB"; thời gian chờ trước khi máy chủ mục tiêu bắt đầu phản hồi yêu cầu)
- `http_req_receiving` (thời gian để máy chủ mục tiêu xử lý và phản hồi đầy đủ cho k6)

Thời gian phản hồi trong dòng này được báo cáo dưới dạng trung bình (avg), tối thiểu (min), trung vị (med), tối đa (max), phần trăm thứ 90 (p90) và phần trăm thứ 95 (p95), tính bằng mili giây (ms). Nếu bạn không chắc chắn nên sử dụng cái nào, hãy lấy con số phần trăm thứ 95 (95th percentile).

Thời gian phản hồi ở phần trăm thứ 95 là 130,19 ms có nghĩa là 95% các yêu cầu có thời gian phản hồi từ 130,19 ms trở xuống. Tuy nhiên, trong tình huống cụ thể này, kịch bản kiểm thử của bạn chỉ thực hiện một yêu cầu duy nhất, vì vậy tất cả các chỉ số trong dòng này đều báo cáo cùng một giá trị: 130,19 ms. Khi bạn chạy các bài kiểm tra với nhiều yêu cầu, bạn sẽ thấy sự thay đổi trong các giá trị này.

Một điều cần lưu ý ở đây là `http_req_duration` là giá trị cho _tất cả_ các yêu cầu, bất kể chúng thành công hay thất bại. Hành vi này có thể gây ra hiểu lầm khi diễn giải thời gian phản hồi, vì các yêu cầu thất bại thường có thời gian phản hồi ngắn hơn hoặc dài hơn so với các yêu cầu thành công.

Dòng dưới đây báo cáo thời gian phản hồi chỉ cho các yêu cầu thành công.

```plain
  { expected_response:true }...: avg=130.19ms min=130.19ms med=130.19ms max=130.19ms p(90)=130.19ms p(95)=130.19ms
```

Để tăng độ chính xác và ngăn các yêu cầu thất bại làm sai lệch kết quả, hãy sử dụng giá trị phần trăm thứ 95 của các yêu cầu thành công làm thời gian phản hồi.

#### Tỷ lệ lỗi (Error rate)

Chỉ số `http_req_failed` mô tả tỷ lệ lỗi của bài kiểm tra. Tỷ lệ lỗi là số lượng yêu cầu thất bại trong bài kiểm tra tính theo tỷ lệ phần trăm của tổng số yêu cầu.

```plain
http_req_failed................: 0.00%  ✓ 0        ✗ 1
```

`http_req_failed` tự động đánh dấu các mã phản hồi HTTP từ 200 đến 399 là thành công. Điều này có nghĩa là các mã phản hồi HTTP 4xx và HTTP 5xx được k6 coi là lỗi theo mặc định. (Lưu ý: Hành vi này có thể được thay đổi bằng cách sử dụng [`setResponseCallback`](https://k6.io/docs/javascript-api/k6-http/setresponsecallback-callback).)

Lần chạy thử này có tỷ lệ lỗi là `0.00%`, vì yêu cầu duy nhất mà nó chạy đã thành công. Nghe có vẻ ngược với trực giác, nhưng dấu `✓` trên dòng này thực tế có nghĩa là không có yêu cầu nào có `http_req_failed = true`, nghĩa là không có lỗi nào. Ngược lại, dấu `✗` có nghĩa là có 1 yêu cầu có `http_req_failed = false`, nghĩa là nó đã thành công.

#### Số lượng yêu cầu (Number of requests)

Tổng số yêu cầu được gửi bởi tất cả các VU trong quá trình kiểm thử được mô tả trong dòng dưới đây.

```plain
http_reqs......................: 1      1.525116/s
```

Ngoài ra, con số `1.525116/s` là số lượng **yêu cầu trên giây (requests per second - rps)** mà bài kiểm tra đã thực hiện trong suốt quá trình. Trong một số công cụ, điều này được mô tả là "throughput của bài kiểm tra". Điều này giúp bạn định lượng thêm mức tải mà ứng dụng của bạn đã trải qua trong quá trình kiểm thử.

#### Thời lượng lần lặp (Iteration duration)

`http_req_duration` đo thời gian cần thiết để một yêu cầu HTTP trong kịch bản nhận được phản hồi từ máy chủ. Nhưng nếu bạn có nhiều yêu cầu HTTP được xâu chuỗi với nhau trong một luồng người dùng (user flow) và bạn muốn biết toàn bộ luồng đó sẽ mất bao lâu đối với một người dùng?

Trong trường hợp đó, thời lượng lần lặp (iteration duration) là chỉ số cần xem xét.

```plain
iteration_duration.............: avg=654.72ms min=654.72ms med=654.72ms max=654.72ms p(90)=654.72ms p(95)=654.72ms
```

Thời lượng lần lặp là khoảng thời gian k6 thực hiện một vòng lặp duy nhất mã VU của bạn. Nếu kịch bản của bạn bao gồm các bước như đăng nhập, duyệt trang sản phẩm, thêm vào giỏ hàng và nhập thông tin thanh toán, thì thời lượng lần lặp sẽ cho bạn biết một người dùng ứng dụng của bạn có thể mất bao lâu để mua một sản phẩm.

Chỉ số này có thể hữu ích khi bạn đang cố gắng quyết định thời gian phản hồi chấp nhận được cho mỗi yêu cầu HTTP là bao nhiêu. Ví dụ, có thể yêu cầu thanh toán mất 2 giây, nhưng nếu tổng thời lượng lần lặp vẫn chỉ là 3 giây, bạn có thể quyết định rằng điều đó vẫn có thể chấp nhận được.

Giống như các chỉ số khác, thời lượng lần lặp được thể hiện dưới dạng các mốc thời gian trung bình, tối thiểu, trung vị, tối đa, phần trăm thứ 90 và phần trăm thứ 95, tính bằng mili giây.

#### Số lần lặp (Number of iterations)

Số lần lặp mô tả tổng số lần k6 đã lặp qua kịch bản của bạn, bao gồm các lần lặp của tất cả các VU. Chỉ số này có thể hữu ích khi bạn muốn xác minh một số đầu ra liên quan đến mỗi lần lặp, chẳng hạn như việc đăng ký tài khoản.

```plain
iterations.....................: 1      1.525116/s
```

Con số `1.525116/s` trên cùng dòng là **số lần lặp trên giây**. Nó mô tả tốc độ mà k6 thực hiện các lần lặp đầy đủ qua kịch bản. Chỉ số này, giống như [yêu cầu trên giây](03-Understanding-k6-results.md#Number-of-requests), là thước đo tốc độ mà k6 gửi thông điệp đến máy chủ ứng dụng.

## Tiếp theo

Việc ghi lại thân phản hồi của các yêu cầu ra console có thể hữu ích khi khắc phục sự cố, nhưng nếu bạn chỉ muốn xác minh phản hồi một cách tự động mà không cần phải kiểm tra nhật ký thì sao? Trong phần tiếp theo, bạn sẽ tìm hiểu về các phép kiểm tra (checks).

## Kiểm tra kiến thức của bạn

Để trả lời các câu hỏi sau, hãy tham khảo báo cáo tóm tắt kết thúc kiểm thử mẫu bên dưới.

```plain
          /\      |‾‾| /‾‾/   /‾‾/
     /\  /  \     |  |/  /   /  /
    /  \/    \    |     (   /   ‾‾\
   /          \   |  |\  \ |  (‾)  |
  / __________ \  |__| \__\ \_____/ .io

  execution: local
     script: test.js
     output: -

  scenarios: (100.00%) 1 scenario, 10 max VUs, 2m30s max duration (incl. graceful stop):
           * default: 10 looping VUs for 2m0s (gracefulStop: 30s)


running (2m00.1s), 00/10 VUs, 9463 complete and 0 interrupted iterations
default ✓ [======================================] 10 VUs  2m0s

     data_received..................: 6.3 MB 52 kB/s
     data_sent......................: 1.5 MB 12 kB/s
     http_req_blocked...............: avg=4.46ms   min=1µs      med=5µs      max=647.67ms p(90)=9µs      p(95)=11µs
     http_req_connecting............: avg=1.4ms    min=0s       med=0s       max=255.11ms p(90)=0s       p(95)=0s
     http_req_duration..............: avg=122.22ms min=111.57ms med=120.95ms max=282.39ms p(90)=127.78ms p(95)=131.04ms
       { expected_response:true }...: avg=122.22ms min=111.57ms med=120.95ms max=282.39ms p(90)=127.78ms p(95)=131.04ms
     http_req_failed................: 0.00%  ✓ 0         ✗ 9463
     http_req_receiving.............: avg=107.08µs min=15µs     med=85µs     max=13.35ms  p(90)=163µs    p(95)=197µs
     http_req_sending...............: avg=35.86µs  min=5µs      med=30µs     max=2.7ms    p(90)=59µs     p(95)=72µs
     http_req_tls_handshaking.......: avg=2.99ms   min=0s       med=0s       max=501.24ms p(90)=0s       p(95)=0s
     http_req_waiting...............: avg=122.08ms min=111.47ms med=120.81ms max=282.16ms p(90)=127.64ms p(95)=130.87ms
     http_reqs......................: 9463   78.808231/s
     iteration_duration.............: avg=126.85ms min=111.75ms med=121.14ms max=803.13ms p(90)=128.46ms p(95)=132.55ms
     iterations.....................: 9463   78.808231/s
     vus............................: 10     min=10      max=10
     vus_max........................: 10     min=10      max=10
```

### Câu hỏi 1

Giá trị nào sau đây là tốt nhất để sử dụng làm thời gian phản hồi cho tất cả các yêu cầu HTTP?

A: 131.04 ms

B: 122.08 ms

C: 4.46 ms

### Câu hỏi 2

Bài kiểm tra này đã thực thi bao nhiêu người dùng ảo (virtual users)?

A: 9463

B: 1

C: 10

### Câu hỏi 3

Có bao nhiêu yêu cầu đã thất bại?

A: 0

B: 9463

C: 10

### Đáp án

1. A. B đề cập đến `http_req_waiting`, vốn chỉ là một thành phần của thời gian phản hồi. C đề cập đến `http_req_blocked`, tức là thời gian chờ để thiết lập kết nối TCP trước khi gửi yêu cầu. A là giá trị phần trăm thứ 95 cho `http_req_duration`, đây là chỉ số tốt nhất để sử dụng cho thời gian phản hồi.
2. C. Số lượng VU được liệt kê ở hàng thứ hai từ dưới lên trong kết quả, và trong trường hợp này là 10.
3. A. Trong dòng có chỉ số `http_req_failed`, phần `0.00% ✓ 0` đề cập đến số lượng phản hồi có lỗi. Nghĩa là, có bao nhiêu yêu cầu có `http_req_failed` được đặt thành `true`. 9463 là số lượng yêu cầu đã vượt qua, hoặc có `http_req_failed` được đặt thành `false`. Đáp án đúng là A, 0. Không có yêu cầu nào thất bại và bài kiểm tra có tỷ lệ lỗi là 0%.
