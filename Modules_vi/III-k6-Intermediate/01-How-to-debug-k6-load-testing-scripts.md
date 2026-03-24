# Cách gỡ lỗi (debug) kịch bản kiểm thử tải k6

Trong khi các chỉ số (metrics) của k6 giúp bạn [hiểu điều gì đã xảy ra trong một bài kiểm tra k6](../II-k6-Foundations/03-Understanding-k6-results.md), bạn có thể cần thông tin chi tiết hơn khi đang trong quá trình viết kịch bản kiểm thử tải. Trong phần này, bạn sẽ học những gì có thể làm để tìm ra lý do tại sao kịch bản của mình bị lỗi.

[![Trong buổi k6 Office Hours này, Tom Miseur và Nicole van der Hoeven trình bày các cách khác nhau để gỡ lỗi một kịch bản kiểm thử tải k6](https://img.youtube.com/vi/Zln_TWOuoho/0.jpg)](https://www.youtube.com/embed/Zln_TWOuoho)

_Trong buổi k6 Office Hours này, Tom Miseur và Nicole van der Hoeven trình bày các cách khác nhau để gỡ lỗi một kịch bản kiểm thử tải k6._

Có một vài thứ bạn có thể thêm vào kịch bản của mình để giúp xác định điều gì đang xảy ra tại thời điểm nào trong kịch bản.

### Thêm các checks

Bạn đã học rằng mình có thể [thêm các checks vào kịch bản](../II-k6-Foundations/04-Adding-checks-to-your-script.md) để xác định các vấn đề trong quá trình thực thi, nhưng chúng cũng hữu ích trong khi gỡ lỗi.
Việc bao gồm các checks cho mọi hành động chính mà kịch bản thực hiện là một thực hành tốt, để có thể thấy rõ vấn đề nằm ở đâu.

Hãy xem xét kịch bản bên dưới.

```js
import http from "k6/http";

let usernameArr = ["admin", "test_user", "guest"];
let passwordArr = ["123", "1234", "12345"];

export default function () {
    // Get random username and password from array
    let rand = Math.floor(Math.random() * 3);
    let username = usernameArr[rand];
    let password = passwordArr[rand];

    http.post("http://test.k6.io/login.php", {
        login: username,
        password: password,
    });
}
```

Kịch bản này khai báo hai mảng đơn giản: một mảng chứa tên người dùng (usernames) và một mảng chứa mật khẩu (passwords).
Kịch bản chọn ngẫu nhiên một cặp từ các mảng đó, sau đó truyền chúng vào một yêu cầu HTTP POST.

Khi bạn chạy kịch bản này, bạn sẽ thấy không có lỗi nào được trả về:

```shell

          /\      |‾‾| /‾‾/   /‾‾/
     /\  /  \     |  |/  /   /  /
    /  \/    \    |     (   /   ‾‾\
   /          \   |  |\  \ |  (‾)  |
  / __________ \  |__| \__\ \_____/ .io

  execution: local
     script: testdata-simple.js
     output: -

  scenarios: (100.00%) 1 scenario, 1 max VUs, 10m30s max duration (incl. graceful stop):
           * default: 1 iterations for each of 1 VUs (maxDuration: 10m0s, gracefulStop: 30s)


running (00m01.1s), 0/1 VUs, 1 complete and 0 interrupted iterations
default ✓ [======================================] 1 VUs  00m01.1s/10m0s  1/1 iters, 1 per VU

     data_received..................: 6.3 kB 5.5 kB/s
     data_sent......................: 839 B  743 B/s
     http_req_blocked...............: avg=424.56ms min=248.87ms med=424.56ms max=600.26ms p(90)=565.12ms p(95)=582.69ms
     http_req_connecting............: avg=156.1ms  min=131.71ms med=156.1ms  max=180.49ms p(90)=175.61ms p(95)=178.05ms
     http_req_duration..............: avg=139.17ms min=130.49ms med=139.17ms max=147.84ms p(90)=146.1ms  p(95)=146.97ms
       { expected_response:true }...: avg=130.49ms min=130.49ms med=130.49ms max=130.49ms p(90)=130.49ms p(95)=130.49ms
     http_req_failed................: 50.00% ✓ 1        ✗ 1
     http_req_receiving.............: avg=84.99µs  min=74µs     med=84.99µs  max=96µs     p(90)=93.8µs   p(95)=94.9µs
     http_req_sending...............: avg=203µs    min=91µs     med=203µs    max=315µs    p(90)=292.59µs p(95)=303.79µs
     http_req_tls_handshaking.......: avg=209.81ms min=0s       med=209.81ms max=419.63ms p(90)=377.66ms p(95)=398.65ms
     http_req_waiting...............: avg=138.88ms min=130.08ms med=138.88ms max=147.67ms p(90)=145.91ms p(95)=146.79ms
     http_reqs......................: 2      1.771779/s
     iteration_duration.............: avg=1.12s    min=1.12s    med=1.12s    max=1.12s    p(90)=1.12s    p(95)=1.12s
     iterations.....................: 1      0.885889/s
     vus............................: 1      min=1      max=1
     vus_max........................: 1      min=1      max=1
```

Không có gì cho thấy có lỗi, nhưng thực tế là có.

Để tìm ra lỗi đó, hãy thêm một check để xem phản hồi trả về là gì.

```js
import http from "k6/http";
import { check } from "k6";

let usernameArr = ["admin", "test_user", "guest"];
let passwordArr = ["123", "1234", "12345"];

export default function () {
    // Get random username and password from array
    let rand = Math.floor(Math.random() * 3);
    let username = usernameArr[rand];
    let password = passwordArr[rand];

    let response = http.post("http://test.k6.io/login.php", {
        login: username,
        password: password,
    });
    check(response, {
        "is status 200": (r) => r.status === 200,
    });
}
```

Kịch bản trên bây giờ sẽ kiểm tra xem phản hồi trả về có phải là HTTP 200 hay không.

Chạy kịch bản đó, và bạn sẽ thấy một sự thay đổi đáng kể trong tóm tắt kết thúc kiểm thử:

```shell
     ✗ is status 200
      ↳  0% — ✓ 0 / ✗ 1
```

Hóa ra kịch bản không trả về HTTP 200 như mong đợi, nhưng điều đó rất khó phát hiện nếu không có check. Việc thêm check giúp xác định rằng đang có vấn đề xảy ra.

Nhưng chuyện gì đang diễn ra?

### Thêm nhật ký (logging)

Vì kịch bản sử dụng dữ liệu kiểm thử, rất có thể tên người dùng và mật khẩu không chính xác. Có thể sự kết hợp này là nguyên nhân gây ra lỗi xác thực (authentication). Trong trường hợp này, các mảng `username` và `password` chỉ có ba phần tử, vì vậy việc kiểm tra thủ công tất cả chúng không quá khó. Nhưng nếu bạn có hàng trăm phần tử thì sao?

Trong trường hợp đó, bạn có thể thử thêm nhật ký tại các phần cụ thể của kịch bản bằng cách sử dụng `console.log()`. Kịch bản bên dưới minh họa câu lệnh này đang hoạt động:

```js
import http from "k6/http";
import { check } from "k6";

let usernameArr = ["admin", "test_user", "guest"];
let passwordArr = ["123", "1234", "12345"];

export default function () {
    // Get random username and password from array
    let rand = Math.floor(Math.random() * 3);
    let username = usernameArr[rand];
    let password = passwordArr[rand];
    console.log("username: " + username, " / password: " + password);

    let response = http.post("http://test.k6.io/login.php", {
        login: username,
        password: password,
    });
    check(response, {
        "is status 200": (r) => r.status === 200,
    });
}
```

Câu lệnh `console.log()` in ra chính xác sự kết hợp được sử dụng, như thế này:

```shell
INFO[0000] username: guest  / password: 12345          source=console
```

Bằng cách đó, bạn biết chính xác sự kết hợp nào cần thử. Có thể tên người dùng hoặc mật khẩu không đúng, hoặc cả hai. Dù bằng cách nào, việc thêm nhật ký vào kịch bản sẽ giúp bạn hiểu giá trị của các biến chính mà kịch bản sử dụng.

> :warning: **Vô hiệu hóa logging trong quá trình kiểm thử tải**. `console.log()` có thể tiêu tốn rất nhiều tài nguyên và việc ghi nhật ký quá nhiều có thể ảnh hưởng đến kết quả kiểm thử. Hãy đóng ghi chú (comment out) phần logging nhiều nhất có thể để tránh việc sử dụng tài nguyên cao khi bài kiểm tra chạy.

Bây giờ, hãy tưởng tượng bạn thử đăng nhập vào ứng dụng bằng tên người dùng và mật khẩu do kịch bản chọn (`guest` và `12345`) và nó không hoạt động. Aha! Thông tin đăng nhập đã sai ngay từ đầu. Bạn xác minh rằng hai tài khoản còn lại hoạt động.

Loại bỏ thông tin đăng nhập `guest` thứ ba và sửa đổi số ngẫu nhiên để chỉ trả về `0` hoặc `1`:

```js
import http from "k6/http";
import { check } from "k6";

let usernameArr = ["admin", "test_user"];
let passwordArr = ["123", "1234"];

export default function () {
    // Get random username and password from array
    let rand = Math.floor(Math.random() * 2);
    let username = usernameArr[rand];
    let password = passwordArr[rand];
    console.log("username: " + username, " / password: " + password);

    let response = http.post("http://test.k6.io/login.php", {
        login: username,
        password: password,
    });
    check(response, {
        "is status 200": (r) => r.status === 200,
    });
}
```

Khi bạn chạy lại kịch bản đó, bạn có thể nhận thấy rằng mặc dù thông tin đăng nhập đã được sửa, nó vẫn thất bại. Đây là một điều tốt! Nó có nghĩa là bạn đã giải quyết được lỗi đầu tiên và có thể bắt đầu xử lý lỗi thứ hai.

Còn điều gì khác có thể sai? Phản hồi nào đang được trả về, nếu không phải là HTTP 200?

### Sử dụng cờ debug HTTP

Để tìm hiểu chính xác yêu cầu đang trả về điều gì, bạn có thể sử dụng cờ `--http-debug`. Chạy bài kiểm tra như sau:

```shell
k6 run test.js --http-debug
```

Lần này, đầu ra bao gồm một số thông tin mới:

```shell
          /\      |‾‾| /‾‾/   /‾‾/
     /\  /  \     |  |/  /   /  /
    /  \/    \    |     (   /   ‾‾\
   /          \   |  |\  \ |  (‾)  |
  / __________ \  |__| \__\ \_____/ .io

  execution: local
     script: testdata-simple.js
     output: -

  scenarios: (100.00%) 1 scenario, 1 max VUs, 10m30s max duration (incl. graceful stop):
           * default: 1 iterations for each of 1 VUs (maxDuration: 10m0s, gracefulStop: 30s)

INFO[0000] Request:
POST /login.php HTTP/1.1
Host: test.k6.io
User-Agent: k6/0.36.0 (https://k6.io/)
Content-Length: 26
Content-Type: application/x-www-form-urlencoded
Accept-Encoding: gzip

  group= iter=0 request_id=34a024e9-d51d-41e6-5695-a4a465b12978 scenario=default source=http-debug vu=1
INFO[0000] Response:
HTTP/1.1 308 Permanent Redirect
Content-Length: 164
Connection: keep-alive
Content-Type: text/html
Date: Tue, 01 Feb 2022 14:29:13 GMT
Location: https://test.k6.io/login.php

  group= iter=0 request_id=34a024e9-d51d-41e6-5695-a4a465b12978 scenario=default source=http-debug vu=1
INFO[0000] Request:
POST /login.php HTTP/1.1
Host: test.k6.io
User-Agent: k6/0.36.0 (https://k6.io/)
Content-Length: 26
Content-Type: application/x-www-form-urlencoded
Referer: http://test.k6.io/login.php
Accept-Encoding: gzip

  group= iter=0 request_id=d33e02d3-0ac5-46cd-4437-45650f2c052e scenario=default source=http-debug vu=1
INFO[0001] Response:
HTTP/1.1 403 Forbidden
Transfer-Encoding: chunked
Connection: keep-alive
Content-Type: text/plain;charset=UTF-8
Date: Tue, 01 Feb 2022 14:29:14 GMT
Set-Cookie: uid=bad; expires=Tue, 01-Feb-2022 15:29:14 GMT; Max-Age=3600; path=/; domain=test.k6.io; httponly
Set-Cookie: sid=bad; expires=Tue, 01-Feb-2022 15:29:14 GMT; Max-Age=3600; path=/; domain=test.k6.io; httponly
X-Powered-By: PHP/5.6.40

  group= iter=0 request_id=d33e02d3-0ac5-46cd-4437-45650f2c052e scenario=default source=http-debug vu=1
```

> :warning: **Chỉ sử dụng HTTP-debug khi đang gỡ lỗi** `http-debug` có thể tạo ra rất nhiều thông tin nhiễu, vì vậy hãy tránh sử dụng nó cho các bài kiểm tra dài hơn hoặc lớn hơn. Nó được sử dụng tốt nhất để khắc phục sự cố trong khi viết kịch bản.

Tóm tắt kết thúc kiểm thử bây giờ bao gồm thông tin về 2 yêu cầu và 2 phản hồi. Nhưng đợi một chút. Chẳng phải kịch bản chỉ có một yêu cầu sao?

Phân tích đầu ra kỹ hơn, bạn có thể thấy điều gì đã xảy ra:

- k6 đã gửi một yêu cầu POST ban đầu.
- Máy chủ phản hồi bằng một chuyển hướng (redirect) HTTP 308.
- k6 gửi một yêu cầu POST khác, do được nhắc bởi lệnh chuyển hướng.
- Máy chủ phản hồi bằng một mã HTTP 403 Forbidden.

Mặc dù kịch bản chỉ chứa một yêu cầu duy nhất, phản hồi của máy chủ đã khiến k6 thực hiện một yêu cầu chuyển hướng, và cuối cùng yêu cầu này đã thất bại. [HTTP 403 Forbidden](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/403) là một lỗi liên quan đến vấn đề xác thực. Lỗi này xác nhận giả thuyết về lỗi xác thực.

Nhưng _tại sao_ lại có lỗi xác thực?

Để có thêm thông tin, hãy chạy lại kịch bản, lần này yêu cầu debug HTTP đầy đủ:

```shell
k6 run test.js --http-debug=full
```

Lần này, bạn không chỉ thấy các tiêu đề (headers) phản hồi và yêu cầu mà còn thấy cả phần thân (bodies) của yêu cầu và phản hồi:

```shell
INFO[0000] Request:
POST /login.php HTTP/1.1
Host: test.k6.io
User-Agent: k6/0.36.0 (https://k6.io/)
Content-Length: 26
Content-Type: application/x-www-form-urlencoded
Accept-Encoding: gzip

login=user&password=userpw  group= iter=0 request_id=9b3c2dd9-8c2a-49bf-7a92-713c757d1a3a scenario=default source=http-debug vu=1
INFO[0000] Response:
HTTP/1.1 308 Permanent Redirect
Content-Length: 164
Connection: keep-alive
Content-Type: text/html
Date: Tue, 01 Feb 2022 14:35:34 GMT
Location: https://test.k6.io/login.php

<html>
<head><title>308 Permanent Redirect</title></head>
<body>
<center><h1>308 Permanent Redirect</h1></center>
<hr><center>nginx</center>
</body>
</html>
  group= iter=0 request_id=9b3c2dd9-8c2a-49bf-7a92-713c757d1a3a scenario=default source=http-debug vu=1
INFO[0000] Request:
POST /login.php HTTP/1.1
Host: test.k6.io
User-Agent: k6/0.36.0 (https://k6.io/)
Content-Length: 26
Content-Type: application/x-www-form-urlencoded
Referer: http://test.k6.io/login.php
Accept-Encoding: gzip

login=user&password=userpw  group= iter=0 request_id=3d0d9247-b93e-47f0-6dee-2f0a3420ca4e scenario=default source=http-debug vu=1
INFO[0001] Response:
HTTP/1.1 403 Forbidden
Transfer-Encoding: chunked
Connection: keep-alive
Content-Type: text/plain;charset=UTF-8
Date: Tue, 01 Feb 2022 14:35:35 GMT
Set-Cookie: uid=bad; expires=Tue, 01-Feb-2022 15:35:35 GMT; Max-Age=3600; path=/; domain=test.k6.io; httponly
Set-Cookie: sid=bad; expires=Tue, 01-Feb-2022 15:35:35 GMT; Max-Age=3600; path=/; domain=test.k6.io; httponly
X-Powered-By: PHP/5.6.40

34
error: invalid csrf token
token: <not set>
session:
0
```

Phần thân phản hồi cho HTTP 403 tiết lộ vấn đề: `error: invalid csrf token`.

Kịch bản gửi tên người dùng và mật khẩu, nhưng không có mã thông báo (token) CSRF! Đúng rồi.

CSRF ([Cross-Site Request Forgery](https://owasp.org/www-community/attacks/csrf)) tokens được sử dụng để cải thiện bảo mật bằng cách ngăn chặn kẻ tấn công chiếm đoạt phiên làm việc của người dùng. Có vẻ như ứng dụng yêu cầu một mã thông báo CSRF mà kịch bản vẫn chưa có. Để lấy mã thông báo CSRF, bạn có thể tạo một yêu cầu ban đầu để truy cập `https://test.k6.io/` và phân tích phản hồi để lấy mã thông báo đó. Chúng ta sẽ đề cập đến vấn đề này trong phần tiếp theo.

## Kiểm tra kiến thức của bạn

### Câu hỏi 1

Cách tốt nhất để xác minh xem một ứng dụng có trả về tệp PDF để phản hồi yêu cầu do kịch bản của bạn gửi hay không?

A: Thêm một check dựa trên kích thước phần thân phản hồi.

B: Sử dụng cờ `http-debug` để cố gắng đọc nội dung của tệp PDF.

C: Thêm một câu lệnh `console.log()` sau khi yêu cầu được thực hiện.

### Câu hỏi 2

Trong kịch bản được sử dụng trên trang này, một số ngẫu nhiên được sử dụng để chọn tài khoản người dùng từ các mảng. Làm thế nào bạn có thể tìm ra giá trị của số ngẫu nhiên đó mỗi khi chạy kịch bản?

A:

```js
check(response, {
    "random number selected": (r) => r.rand === 0,
});
```

B: `k6 run test.js --http-debug="rand"`

C: `console.log('rand', rand);`

### Câu hỏi 3

Tại sao bạn nên vô hiệu hóa logging nhiều nhất có thể trong một bài kiểm tra tải có tăng tải (ramped-up)?

A: Bạn không bao giờ nên thực hiện bất kỳ logging nào khi đang chạy một bài kiểm tra thực sự.

B: Việc logging không cần thiết sẽ tiêu tốn tài nguyên trên trình tạo tải (load generator).

C: Sử dụng `http-debug` là lựa chọn tốt hơn để sử dụng trong bài kiểm tra tải.

### Đáp án

1. A. Các phương pháp `http-debug` và `console.log` có thể quá rườm rà và gây gánh nặng cho tài nguyên trình tạo tải một cách không cần thiết. Nếu bạn biết rằng tệp PDF lớn hơn đáng kể so với kích thước phản hồi trả về khi không có PDF, bạn có thể sử dụng kích thước phần thân phản hồi để xác minh tệp PDF.
2. C. `console.log()` rất phù hợp cho các trường hợp sử dụng như thế này, đặc biệt là khi đang khắc phục sự cố hoặc gỡ lỗi kịch bản.
3. B. Việc ghi quá nhiều thông tin vào nhật ký có thể chính là một nút thắt cổ chai về hiệu năng bên trong trình tạo tải, vì vậy hãy thận trọng về thông tin bạn in ra và lưu lại.
