Checks là một loại tiêu chí kiểm thử có thể được sử dụng để xác minh phản hồi được trả về bởi một yêu cầu (request).

Cho đến bây giờ, bạn đã sử dụng `console.log()` để in thân phản hồi (response body) ra terminal. Cách tiếp cận này hữu ích cho việc gỡ lỗi (debugging), nhưng nó có thể nhanh chóng trở nên mất kiểm soát nếu thân phản hồi lớn hoặc nếu bạn quyết định chạy bài kiểm tra với nhiều hơn một VU. Thay vào đó, trong phần này, bạn sẽ học cách sử dụng các checks.

## Cách thêm các checks vào kịch bản của bạn

Quay lại với kịch bản!

Bạn đã biết phản hồi dự kiến của máy chủ mục tiêu đối với yêu cầu mà kịch bản đang gửi: nó sẽ gửi lại bất cứ thứ gì kịch bản gửi cho nó. Trong trường hợp này, đó là `Hello world!`

Vì vậy, hãy xóa câu lệnh `console.log()` và thêm một check bằng cách sao chép đoạn mã này:

```js
import http from 'k6/http';
import { check } from 'k6';

export default function() {
  let url = 'https://httpbin.test.k6.io/post';
  let response = http.post(url, 'Hello world!');
  check(response, {
      'Application says hello': (r) => r.body.includes('Hello world!')
  });
}
```

Lưu ý rằng bạn cần nhập `check` từ thư viện k6:

```js
import { check } from 'k6';
```

Và bạn cần đặt check thực tế vào trong hàm mặc định:

```js
check(response, {
      'Application says hello': (r) => r.body.includes('Hello world!')
  });
}
```

Check mà bạn vừa thêm sẽ tìm kiếm chuỗi `Hello world!` trong thân của phản hồi của _mọi_ yêu cầu.

### Chạy kịch bản

Nó trông như thế nào trong tóm tắt kết thúc kiểm thử? Lưu kịch bản của bạn và chạy:

```plain
k6 run test.js
```

và bạn sẽ thấy đầu ra tương tự như thế này:

```plain
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


running (00m00.7s), 0/1 VUs, 1 complete and 0 interrupted iterations
default ✓ [======================================] 1 VUs  00m00.7s/10m0s  1/1 iters, 1 per VU

     ✓ Application says hello

     checks.........................: 100.00% ✓ 1        ✗ 0
     data_received..................: 5.9 kB  8.4 kB/s
     data_sent......................: 564 B   801 B/s
     http_req_blocked...............: avg=582.6ms  min=582.6ms  med=582.6ms  max=582.6ms  p(90)=582.6ms  p(95)=582.6ms
     http_req_connecting............: avg=121.14ms min=121.14ms med=121.14ms max=121.14ms p(90)=121.14ms p(95)=121.14ms
     http_req_duration..............: avg=120.62ms min=120.62ms med=120.62ms max=120.62ms p(90)=120.62ms p(95)=120.62ms
       { expected_response:true }...: avg=120.62ms min=120.62ms med=120.62ms max=120.62ms p(90)=120.62ms p(95)=120.62ms
     http_req_failed................: 0.00%   ✓ 0        ✗ 1
     http_req_receiving.............: avg=72µs     min=72µs     med=72µs     max=72µs     p(90)=72µs     p(95)=72µs
     http_req_sending...............: avg=292µs    min=292µs    med=292µs    max=292µs    p(90)=292µs    p(95)=292µs
     http_req_tls_handshaking.......: avg=405.16ms min=405.16ms med=405.16ms max=405.16ms p(90)=405.16ms p(95)=405.16ms
     http_req_waiting...............: avg=120.26ms min=120.26ms med=120.26ms max=120.26ms p(90)=120.26ms p(95)=120.26ms
     http_reqs......................: 1       1.419825/s
     iteration_duration.............: avg=703.46ms min=703.46ms med=703.46ms max=703.46ms p(90)=703.46ms p(95)=703.46ms
     iterations.....................: 1       1.419825/s
```

Check mới được hiển thị trong các dòng:

```plain
	✓ Application says hello

     checks.........................: 100.00% ✓ 1        ✗ 0
```

và dấu `✓` có nghĩa là tất cả các yêu cầu đã thực thi (chỉ một yêu cầu trong lần chạy thử này) đã vượt qua check. Lần chạy thử này có tỷ lệ vượt qua check là 100%.

### Các checks bị thất bại

Nó trông như thế nào khi check thất bại?

Sửa đổi kịch bản để tìm kiếm văn bản không có trong phản hồi, như sau:

```js
import http from 'k6/http';
import { check } from 'k6';

export default function() {
  let url = 'https://httpbin.test.k6.io/post';
  let response = http.post(url, 'Hello world!');
  check(response, {
      'Application says hello': (r) => r.body.includes('Bonjour!')
  });
}
```

Chạy kịch bản đó, và bạn sẽ nhận được kết quả sau:

```plain
     ✗ Application says hello
      ↳  0% — ✓ 0 / ✗ 1

     checks.........................: 0.00%  ✓ 0        ✗ 1
```

Lần này, dấu `✗ 1` cho biết một check đã thất bại.

### Các checks thất bại không phải là lỗi

Bạn có thể nhận thấy rằng trong ví dụ cuối cùng, `http_req_failed`, hoặc tỷ lệ lỗi HTTP, không bị ảnh hưởng bởi check bị thất bại. Điều này là do các checks không ngăn một kịch bản thực thi thành công và chúng không trả về một trạng thái thoát (exit status) thất bại.

> :bulb: Để làm cho các checks thất bại dừng bài kiểm tra của bạn, bạn có thể [kết hợp chúng với các ngưỡng (thresholds)](https://k6.io/docs/using-k6/thresholds/#failing-a-load-test-using-checks).

## Các loại checks khác

Trang [checks](https://k6.io/docs/using-k6/checks/) chứa các loại checks khác mà bạn có thể thực hiện, bao gồm mã phản hồi HTTP và kích thước thân phản hồi. Bạn cũng có thể bao gồm nhiều checks cho một phản hồi duy nhất.

## Tiếp theo

Bạn đã gần sẵn sàng để mở rộng quy trình kiểm thử của mình cho nhiều người dùng! Trước khi thực hiện, phần tiếp theo sẽ thảo luận về cách làm cho bài kiểm tra của bạn thực tế hơn với think time.

## Kiểm tra kiến thức của bạn

### Câu hỏi 1

Bạn có thể sử dụng check để xác minh điều nào sau đây?

A: Liệu thời gian phản hồi ở phần trăm thứ 95 của yêu cầu có lớn hơn 1 giây hay không

B: Kích thước của thân phản hồi được trả về

C: Tỷ lệ lỗi của bài kiểm tra

### Câu hỏi 2

Các checks đánh giá phần nào của bài kiểm tra?

A: Thời gian phản hồi của một yêu cầu

B: Cú pháp của yêu cầu được gửi đến ứng dụng

C: Phản hồi của máy chủ ứng dụng

### Câu hỏi 3

Trong đoạn trích sau từ tóm tắt kết thúc kiểm thử, có bao nhiêu checks đã thất bại?

```plain
     ✗ Application says hello
      ↳  51% — ✓ 1215 / ✗ 1144

     checks.........................: 51.50% ✓ 1215      ✗ 1144
```

A: 51.50%

B: 1215

C: 1144

### Đáp án

1. B. Phần trăm thứ 95 của thời gian phản hồi là một chỉ số tổng hợp: nó dựa trên các phép đo từ tất cả các yêu cầu cho đến thời điểm đó. Đây không phải là thứ bạn có thể tạo check, mặc dù bạn chắc chắn có thể đưa nó vào như một [ngưỡng (threshold)](https://k6.io/docs/using-k6/thresholds/) trong kịch bản của mình. Tỷ lệ lỗi của một bài kiểm tra cũng được tổng hợp tương tự. B, kích thước của thân phản hồi, là câu trả lời đúng.
2. C. Các checks phân tích các phản hồi từ máy chủ ứng dụng, không phải yêu cầu được gửi bởi k6.
3. C. Số lượng các checks thất bại được hiển thị ở `✗ 1144`. Các checks của yêu cầu này đã vượt qua 51.50% số lần.
