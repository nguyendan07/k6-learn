# Bắt đầu với k6 OSS

Có nhiều cách để bắt đầu viết kịch bản (scripting) với k6, nhưng chúng ta sẽ bắt đầu với [k6 OSS](https://github.com/grafana/k6) vì một vài lý do sau:

- Đây là một công cụ kiểm thử tải (load testing) hoàn chỉnh, và nó không yêu cầu đăng ký trả phí để sử dụng.
- k6 Cloud, nền tảng SaaS, cũng sử dụng k6 OSS, vì vậy những kỹ năng bạn học được trong phần này sẽ vẫn áp dụng được ngay cả khi bạn quyết định sử dụng k6 Cloud sau này.
- Bạn có thể thêm các kịch bản (scenarios) và tính năng nâng cao vào kịch bản k6 OSS của mình. Các phương pháp tạo kịch bản khác mà chúng ta sẽ thảo luận sau này thường bị hạn chế về chức năng.

Hãy bắt đầu nào!

## Cài đặt

Đầu tiên, hãy cài đặt k6 bằng cách [làm theo các hướng dẫn tại đây](https://k6.io/docs/getting-started/installation/) cho hệ điều hành của bạn.

Tiếp theo, hãy chọn IDE hoặc trình soạn thảo văn bản yêu thích của bạn. Nhiều người trong chúng tôi sử dụng và khuyên dùng [VS Code](https://code.visualstudio.com/), nhưng bạn cũng có thể sử dụng [Sublime Text](https://www.sublimetext.com/), [Atom](https://atom.io/), hoặc bất cứ thứ gì bạn đang sử dụng có thể tạo tệp văn bản.

## Viết kịch bản k6 đầu tiên của bạn

Đã đến lúc viết kịch bản!

k6 hỗ trợ nhiều giao thức, nhưng hiện tại, hãy tập trung vào HTTP. Kịch bản đầu tiên của bạn sẽ thực hiện một yêu cầu HTTP POST cơ bản tới một API thử nghiệm, API này sẽ phản hồi lại bất cứ thứ gì bạn gửi cho nó.

Cách nhanh nhất để tạo một bài kiểm tra k6 là sử dụng lệnh `k6 new [filename]` được giới thiệu trong k6 phiên bản 0.48.0. Lệnh này sẽ tự động tạo một tệp với mã mẫu (boilerplate) cơ bản bạn cần để bắt đầu nhanh chóng.

Nhưng, như một phần của việc học k6, chúng tôi cũng sẽ dạy bạn cách tạo một bài kiểm tra theo cách thủ công.

Tạo một tệp mới tên là `test.js`, và mở nó trong IDE yêu thích của bạn. Tệp này là kịch bản kiểm thử của chúng ta. Các kịch bản k6 luôn được viết bằng JavaScript, mặc dù bản thân k6 được viết bằng Go. Chúng ta sẽ cùng nhau tạo kịch bản này theo từng bước. Hãy sao chép và dán các đoạn mã khi cần thiết, để kịch bản của bạn trông giống như kịch bản ở đây.

Nhập HTTP Client từ module có sẵn `k6/http`:

```js
import http from 'k6/http';
```

Bây giờ, hãy tạo và xuất một hàm mặc định (default function):

```js
export default function() {
}
```

Bất kỳ mã nào trong hàm `default` sẽ được thực thi bởi mỗi người dùng ảo (virtual user - VU) của k6 khi bài kiểm tra chạy.

Thêm logic để thực hiện cuộc gọi HTTP thực tế:

```js
import http from 'k6/http';

export default function() {
  let url = 'https://httpbin.test.k6.io/post';
  let response = http.post(url, 'Hello world!');
}
```

Ở đây, bạn đang hướng dẫn k6 gửi một yêu cầu HTTP POST tới endpoint của API `https://httpbin.test.k6.io/post` với phần thân (body) là `Hello world!`

Thực tế bạn đã có thể chạy kịch bản này rồi, và k6 sẽ thực hiện yêu cầu HTTP POST, nhưng làm thế nào bạn biết nó có hoạt động hay không? Đây là cách để ghi lại phản hồi (response) ra console:

```js
import http from 'k6/http';

export default function() {
  let url = 'https://httpbin.test.k6.io/post';
  let response = http.post(url, 'Hello world!');

  console.log(response.json().data);
}
```

Bạn sẽ học thêm các cách khác để xác minh kết quả kiểm thử sau này, nhưng bây giờ, hãy tiến hành chạy bài kiểm tra đầu tiên của bạn!

## Hello World: chạy kịch bản k6 của bạn

Lưu kịch bản trong trình soạn thảo của bạn. Sau đó, mở terminal và đi đến thư mục nơi bạn đã lưu kịch bản k6. Bây giờ, hãy chạy bài kiểm tra:

```js
k6 run test.js
```

Bạn sẽ nhận được kết quả tương tự như thế này:

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

Đó là rất nhiều chỉ số (metrics)! Trong phần tiếp theo, chúng ta sẽ xem xét ý nghĩa của từng dòng này.

## Các tài nguyên khác

[![Week of Load testing day 3: Installing k6 and running load test](../../images/week-of-testing-youtube.png)](https://www.youtube.com/embed/y5tteMKZUqk)

## Kiểm tra kiến thức của bạn

### Câu hỏi 1

Cách tốt nhất để truy cập JSON body của một phản hồi HTTP là gì?

A: `response.json()`

B: `response.body()`

C: `response.content`

### Câu hỏi 2

Tên của module có sẵn chứa HTTP client là gì?

A: `http`

B: `k6/http-client`

C: `k6/http`

### Câu hỏi 3

Trong một kịch bản kiểm thử, chúng ta nên đặt các cuộc gọi HTTP ở đâu để chúng được thực thi bởi một người dùng ảo?

A: Trong phạm vi toàn cục (global scope)

B: Trong một hàm mặc định được xuất (exported default function)

C: Trong một hàm có tên là `exec()`

### Đáp án

1. A. Sử dụng `response.json()` không chỉ lấy được phần thân phản hồi mà còn phân tích cú pháp (parse) JSON đó.
2. C. `k6/http` là module cần được nhập vào kịch bản k6 nếu bạn muốn sử dụng HTTP. Hai tùy chọn còn lại không tồn tại.
3. B. Chỉ mã được đặt trong một hàm được xuất (có thể là hàm mặc định hoặc hàm được đặt tên trong tùy chọn `exec` [của một scenario](https://k6.io/docs/using-k6/scenarios/#common-options)) mới được thực thi bởi một người dùng ảo.
