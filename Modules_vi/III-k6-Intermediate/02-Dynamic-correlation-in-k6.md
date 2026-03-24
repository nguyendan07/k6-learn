# Tương quan động (Dynamic correlation) trong k6

Trong phần trước, bạn đã học [các phương pháp khác nhau để gỡ lỗi một kịch bản trong k6](01-How-to-debug-k6-load-testing-scripts.md), và cuối cùng đã gặp phải một vấn đề mà mọi người kiểm thử tải đều phải đối mặt: tương quan (correlation).

## Tương quan (correlation) là gì?

Tương quan dữ liệu là quá trình trích xuất một giá trị từ một phản hồi HTTP trước đó và sử dụng giá trị đó trong yêu cầu HTTP tiếp theo.

Luồng yêu cầu của một người dùng truy cập một ứng dụng có thể trông như sau:

- Bước 1. Yêu cầu HTTP GET cho trang _My Messages_.
- Bước 2. Yêu cầu HTTP POST với tên người dùng và mật khẩu

Trong [phần trước](./01-How-to-debug-k6-load-testing-scripts.md), kịch bản đã cố gắng đăng nhập ngay lập tức (bỏ qua Bước 1 và đi thẳng đến Bước 2), nhưng sau khi áp dụng một số kỹ thuật gỡ lỗi, bạn đã phát hiện ra rằng mã phản hồi là HTTP 403 Forbidden, với `error: invalid csrf token` trong phần thân phản hồi.

Để khắc phục lỗi này và mô phỏng chính xác hành vi của người dùng, kịch bản phải:

- Lưu phản hồi cho yêu cầu ở Bước 1.
- Trích xuất mã thông báo [Cross-Site Request Forgery (CSFR)](https://owasp.org/www-community/attacks/csrf) từ Bước 1.
- Truyền mã thông báo CSFR khi yêu cầu Bước 2.

Vì kịch bản bắt đầu bằng Bước 2, bạn cần thêm một yêu cầu khác cho Bước 1.

## Viết kịch bản cho yêu cầu trước đó

Tạm thời đóng ghi chú (comment out) mọi thứ trong hàm `default` và dán mã này vào:

```js
let response = http.get("https://test.k6.io/my_messages.php");
check(response, {
    "is Unauthorized": (r) => r.body.includes("Unauthorized"),
});
```

Đoạn mã này yêu cầu trang _My Messages_ và xác minh xem nó có chứa từ "Unauthorized" trong phần thân hay không. Để xem điều đó hoạt động như thế nào, hãy chạy kịch bản với chế độ debug HTTP đầy đủ:

```shell
k6 run test.js --http-debug=full
```

Đầu ra của bạn sẽ trông giống như thế này:

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
GET /my_messages.php HTTP/1.1
Host: test.k6.io
User-Agent: k6/0.36.0 (https://k6.io/)
Accept-Encoding: gzip

  group= iter=0 request_id=aae39c79-6ad4-47de-631a-6213e3b218be scenario=default source=http-debug vu=1
INFO[0001] Response:
HTTP/1.1 200 OK
Transfer-Encoding: chunked
Connection: keep-alive
Content-Type: text/html; charset=UTF-8
Date: Thu, 03 Feb 2022 14:01:13 GMT
Set-Cookie: csrf=NjI1NjkwOTkw; expires=Thu, 03-Feb-2022 15:01:13 GMT; Max-Age=3600; path=/; domain=test.k6.io; httponly
X-Powered-By: PHP/5.6.40

3bb


<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
<head>
<title>My messages</title>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<link rel="icon" href="/static/favicon.ico" sizes="32x32">
</head>
<body>

<p><a href="/">&lt; Back</a></p>


<h2>Unauthorized</h2>
<p><small>Hint: try <code>admin</code>/<code>123</code> or
<code>test_user</code>/<code>1234</code></small></p>

<form method="POST" action="/login.php">
<input type="hidden" name="redir" value="1">
<input type="hidden" name="csrftoken" value="NjI1NjkwOTkw">
<table cellpadding="5" cellspacing="0" border="0" width="1%">
<tr>
  <td>Login:</td><td><input type="text" name="login" autocomplete="off"></td>
</tr>
<tr>
  <td>Password:</td><td><input type="password" name="password" autocomplete="off"></td>
</tr>
</table>
<input type="submit" value="Go!">
</form>


<p><small>Imitation page. Copyright &copy; 2022, k6.io</small></p>

</body>
</html>

0

  group= iter=0 request_id=aae39c79-6ad4-47de-631a-6213e3b218be scenario=default source=http-debug vu=1

running (00m00.6s), 0/1 VUs, 1 complete and 0 interrupted iterations
default ✓ [======================================] 1 VUs  00m00.6s/10m0s  1/1 iters, 1 per VU

     ✓ is Unauthorized

     checks.........................: 100.00% ✓ 1        ✗ 0
     data_received..................: 6.7 kB  11 kB/s
     data_sent......................: 526 B   866 B/s
     http_req_blocked...............: avg=493.46ms min=493.46ms med=493.46ms max=493.46ms p(90)=493.46ms p(95)=493.46ms
     http_req_connecting............: avg=112.34ms min=112.34ms med=112.34ms max=112.34ms p(90)=112.34ms p(95)=112.34ms
     http_req_duration..............: avg=112.26ms min=112.26ms med=112.26ms max=112.26ms p(90)=112.26ms p(95)=112.26ms
       { expected_response:true }...: avg=112.26ms min=112.26ms med=112.26ms max=112.26ms p(90)=112.26ms p(95)=112.26ms
     http_req_failed................: 0.00%   ✓ 0        ✗ 1
     http_req_receiving.............: avg=180µs    min=180µs    med=180µs    max=180µs    p(90)=180µs    p(95)=180µs
     http_req_sending...............: avg=66µs     min=66µs     med=66µs     max=66µs     p(90)=66µs     p(95)=66µs
     http_req_tls_handshaking.......: avg=378.53ms min=378.53ms med=378.53ms max=378.53ms p(90)=378.53ms p(95)=378.53ms
     http_req_waiting...............: avg=112.02ms min=112.02ms med=112.02ms max=112.02ms p(90)=112.02ms p(95)=112.02ms
     http_reqs......................: 1       1.646844/s
     iteration_duration.............: avg=606.35ms min=606.35ms med=606.35ms max=606.35ms p(90)=606.35ms p(95)=606.35ms
     iterations.....................: 1       1.646844/s
```

Bây giờ chúng ta đang dần gỡ được nút thắt! Toàn bộ phần thân HTML của phản hồi được lưu trong biến [`response`](https://k6.io/docs/javascript-api/k6-http/response/).

## Trích xuất giá trị từ phản hồi

Mã HTML cho biểu mẫu đăng nhập từ đầu ra trước đó bao gồm mã thông báo CSRF:

```html
<input type="hidden" name="csrftoken" value="NjI1NjkwOTkw" />
```

Giá trị này là động, vì vậy kịch bản cần trích xuất nó dựa trên định dạng mà nó được trình bày.

Thêm các dòng sau vào kịch bản của bạn:

```js
let csrfToken = response.html().find("input[name=csrftoken]").attr("value");
console.log(csrfToken);
```

Dòng đầu tiên phân tích cú pháp phần thân phản hồi HTML và tìm kiếm giá trị của một phần tử `input` có `name` bằng `csrftoken`. Chạy kịch bản để xem giá trị của `csrfToken` có đúng không. Bạn sẽ nhận được kết quả tương tự như sau:

```js
INFO[0001] NDExODkxOTcz                                  source=console
```

Có vẻ như nó đang hoạt động! Hãy thử làm cho kịch bản của bạn hoạt động:

- Bỏ đóng ghi chú (uncomment) yêu cầu đăng nhập và sửa đổi nó để mã thông báo CSRF vừa trích xuất được truyền kèm theo yêu cầu.
- Thêm một check cho cụm từ "successfully authorized" sau yêu cầu đăng nhập, để xác minh rằng người dùng trong kịch bản đã đăng nhập thành công.

Nếu bạn gặp khó khăn, hãy so sánh kịch bản của mình với kịch bản ở cuối phần này.

## Kiểm tra kiến thức của bạn

### Câu hỏi 1

Bạn nghi ngờ có thể có một giá trị động mà bạn cần tương quan từ một phản hồi trước đó và gửi kèm theo yêu cầu của mình. Cách tốt nhất để xác nhận nghi ngờ này là gì?

A: Sử dụng DevTools để xem lưu lượng mạng khi bạn thực hiện hành động, sau đó xem các tham số được truyền cùng với yêu cầu thành công.

B: Ghi lại kịch bản và chạy nó bằng k6.

C: Thêm một check cho từ `csrftoken` để xem có mã thông báo nào được trả về trong phản hồi hay không.

### Câu hỏi 2

Điều nào sau đây có thể giúp bạn xác định khi nào cần thực hiện tương quan (correlation)?

A:

```js
check(response, {
    "is Unauthorized": (r) => r.body.includes("Unauthorized"),
});
```

B: `let rand = Math.floor(Math.random() * 2);`

C: `--http-debug=full`

### Câu hỏi 3

Tại sao việc phát lại một kịch bản đã ghi lại có thể gây ra lỗi?

A: Một số bước có thể yêu cầu các giá trị động được trích xuất từ các phản hồi trước đó.

B: Không có các checks nào được định nghĩa trong kịch bản.

C: Chạy với nhiều người dùng có thể gây ra tải không cần thiết lên máy chủ ứng dụng.

## Kịch bản

```js
import http from "k6/http";
import { check } from "k6";

let usernameArr = ["admin", "test_user"];
let passwordArr = ["123", "1234"];

export default function () {
    let response = http.get("https://test.k6.io/my_messages.php");
    check(response, {
        "is Unauthorized": (r) => r.body.includes("Unauthorized"),
    });

    let csrfToken = response.html().find("input[name=csrftoken]").attr("value");

    // Get random username and password from array
    let rand = Math.floor(Math.random() * 2);
    let username = usernameArr[rand];
    let password = passwordArr[rand];
    console.log("username: " + username, " / password: " + password);

    response = http.post("http://test.k6.io/login.php", {
        login: username,
        password: password,
        csrftoken: csrfToken,
    });
    check(response, {
        "is status 200": (r) => r.status === 200,
        "Successful login": (r) => r.body.includes("successfully authorized"),
    });
}
```

### Đáp án

1. A. DevTools là một cách tuyệt vời để chạy qua các cặp yêu cầu và phản hồi của một ứng dụng web, từng bước một. B có lẽ sẽ không cung cấp nhiều thông tin hữu ích ngoài các lỗi, và C thì quá cụ thể - có những tham số động khác ngoài mã thông báo có thể gây ra lỗi nếu không được xử lý đúng cách.
2. C. Sử dụng cờ `http-debug` có thể hữu ích vì nó in ra tất cả thông tin phản hồi và yêu cầu. Nếu có quá nhiều thông tin không cần thiết, hãy cân nhắc sử dụng DevTools hoặc [một proxy](https://k6.io/blog/k6-load-testing-debugging-using-a-web-proxy/) thay thế.
3. A.
