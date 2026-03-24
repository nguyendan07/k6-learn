k6 không có cách nguyên bản để vẽ biểu đồ kết quả load-testing. Tuy nhiên, nó có rất nhiều tùy chọn để lưu đầu ra ở các định dạng khác nhau. Bài [blog](https://k6.io/blog/ways-to-visualize-k6-results/) này là một điểm bắt đầu tốt. Và bạn có thể tìm thấy danh sách đầy đủ các tích hợp hoặc hướng dẫn trực quan hóa kết quả [tại đây](https://k6.io/docs/integrations/#result-store-and-visualization).

Trong phần này, chúng ta sẽ thảo luận về hai định dạng kết quả kiểm thử phổ biến: CSV và JSON.

Lưu ý: Đối với cả định dạng CSV và JSON, bạn sẽ cần công cụ trực quan hóa của riêng mình. Đây có thể là bất cứ thứ gì từ [Google Sheets](https://sheets.google.com), đến [Grafana](https://grafana.com), hay [Tableau](https://tableau.com).

## Sự khác biệt giữa kết quả kết thúc kiểm thử (end-of-test results) và kết quả chuỗi thời gian (time-series results) là gì?

## CSV

### Lưu kết quả k6 dưới dạng CSV

Lưu kết quả k6 vào tệp CSV rất tốt cho việc phân tích sâu hơn trong phần mềm trực quan hóa dữ liệu mà bạn chọn. Tệp CSV có thể được mở dưới dạng bảng tính hoặc được sử dụng để tạo biểu đồ và bảng tóm tắt.

Để xuất kết quả kiểm thử k6 ra tệp CSV, hãy sử dụng lệnh này khi chạy bài kiểm tra:

```plain
k6 run test.js -o csv=results.csv
```

Bạn cũng có thể sử dụng `--out` thay vì `-o`.

### Định dạng đầu ra kết quả CSV

Lệnh trên sẽ lưu kết quả k6 dưới dạng CSV theo định dạng sau:

```csv
metric_name,timestamp,metric_value,check,error,error_code,expected_response,group,method,name,proto,scenario,service,status,subproto,tls_version,url,extra_tags
http_reqs,1641298536,1.000000,,,,true,,POST,https://httpbin.test.k6.io/post,HTTP/1.1,default,,200,,tls1.2,https://httpbin.test.k6.io/post,
http_req_duration,1641298536,114.365000,,,,true,,POST,https://httpbin.test.k6.io/post,HTTP/1.1,default,,200,,tls1.2,https://httpbin.test.k6.io/post,
http_req_blocked,1641298536,589.667000,,,,true,,POST,https://httpbin.test.k6.io/post,HTTP/1.1,default,,200,,tls1.2,https://httpbin.test.k6.io/post,
http_req_connecting,1641298536,117.517000,,,,true,,POST,https://httpbin.test.k6.io/post,HTTP/1.1,default,,200,,tls1.2,https://httpbin.test.k6.io/post,
http_req_tls_handshaking,1641298536,415.043000,,,,true,,POST,https://httpbin.test.k6.io/post,HTTP/1.1,default,,200,,tls1.2,https://httpbin.test.k6.io/post,
http_req_sending,1641298536,0.251000,,,,true,,POST,https://httpbin.test.k6.io/post,HTTP/1.1,default,,200,,tls1.2,https://httpbin.test.k6.io/post,
http_req_waiting,1641298536,114.010000,,,,true,,POST,https://httpbin.test.k6.io/post,HTTP/1.1,default,,200,,tls1.2,https://httpbin.test.k6.io/post,
http_req_receiving,1641298536,0.104000,,,,true,,POST,https://httpbin.test.k6.io/post,HTTP/1.1,default,,200,,tls1.2,https://httpbin.test.k6.io/post,
http_req_failed,1641298536,0.000000,,,,true,,POST,https://httpbin.test.k6.io/post,HTTP/1.1,default,,200,,tls1.2,https://httpbin.test.k6.io/post,
checks,1641298536,1.000000,Application says hello,,,,,,,,default,,,,,,
vus,1641298536,2.000000,,,,,,,,,,,,,,,
vus_max,1641298536,100.000000,,,,,,,,,,,,,,,
```

Mỗi dòng trong tệp là một phép đo duy nhất được thực hiện trong quá trình thực thi kiểm thử.

Tệp kết quả sử dụng các cột sau:

- **`metric_name`**: Tên của chỉ số (metric) mà giá trị được ghi lại. Theo mặc định, k6 đi kèm với [các chỉ số tích hợp sẵn này](https://k6.io/docs/using-k6/metrics/#built-in-metrics). Tất cả giá trị cho tất cả các metrics được đưa vào cùng một tệp kết quả để phân tích dễ dàng hơn.
- **`timestamp`**: Ngày và giờ địa phương mà mỗi phép đo được thực hiện, [tính theo Epoch time](https://www.epochconverter.com/).
- **`metric_value`**: Giá trị đọc được cho chỉ số đã cho tại dấu thời gian được cung cấp. Đơn vị đo lường cho giá trị này khác nhau tùy thuộc vào chỉ số. Ví dụ, các giá trị `http_req_duration` tính bằng mili giây.
- **`check`**: Tên duy nhất được đặt cho check đang được xác minh. Trong ví dụ check bên dưới, tên check là `Application says hello`:

  ```js
  check(response, {
    "Application says hello": (r) => r.body.includes("Hello world!"),
  });
  ```

- **`error`**: Văn bản của bất kỳ lỗi nào không phải HTTP gặp phải, chẳng hạn như lỗi mạng hoặc DNS. Giá trị này để trống nếu không có lỗi.
- **`error_code`**: Một mã lỗi k6. Giá trị này để trống nếu không có lỗi. [Đây là danh sách đầy đủ](https://k6.io/docs/javascript-api/error-codes) các mã lỗi có thể có.
- **`expected_response`**: Một giá trị boolean (`true` hoặc `false`) cho biết phản hồi trả về có như mong đợi hay không (mặc định là mã HTTP nhỏ hơn 400).
- **`group`**: Tên của một [nhóm yêu cầu (request group)](https://k6.io/docs/using-k6/tags-and-groups/#groups) mà chỉ số đó thuộc về.
- **`method`**: Tên của phương thức HTTP được sử dụng, chẳng hạn như `GET` hoặc `POST`, hoặc tên phương thức RPC cho gRPC.
- **`name`**: Tên của yêu cầu đã gửi. Mặc định là URL, nhưng có thể [thay đổi bằng cách sử dụng tags](https://k6.io/docs/using-k6/http-requests#url-grouping).
- **`proto`**: Tên của giao thức đang được sử dụng, chẳng hạn như `HTTP/1.1`.
- **`scenario`**: Tên của kịch bản kiểm thử (test scenario) mà phép đo được thực hiện. Kịch bản tiêu chuẩn là `default`.
- **`service`**: Đối với gRPC, tên dịch vụ RPC.
- **`subproto`**: Đối với websockets, tên subprotocol.
- **`tls_version`**: Loại bảo mật lớp truyền tải (Transport Layer Security - TLS) được sử dụng để mã hóa kết nối.
- **`url`**: URL của yêu cầu đã gửi. Trừ khi tên được thay đổi, giá trị này giống như `name`.
- **`extra_tags`**: Các thẻ (tags) áp dụng cho phép đo này. Để trống trừ khi [tags](https://k6.io/docs/using-k6/tags-and-groups/) được sử dụng trong kịch bản.

## JSON

### Lưu kết quả k6 dưới dạng JSON

Để xuất kết quả k6 ra tệp JSON, hãy chạy bài kiểm tra với lệnh này:

```plain
k6 run test.js -o json=results.json
```

### Định dạng đầu ra kết quả JSON

Tệp JSON sẽ trông giống như thế này:

```plain
{"type":"Metric","data":{"name":"http_reqs","type":"counter","contains":"default","tainted":null,"thresholds":[],"submetrics":null,"sub":{"name":"","parent":"","suffix":"","tags":null}},"metric":"http_reqs"}
{"type":"Point","data":{"time":"2022-01-05T12:46:23.603474+01:00","value":1,"tags":{"expected_response":"true","group":"","method":"POST","name":"https://httpbin.test.k6.io/post","proto":"HTTP/1.1","scenario":"default","status":"200","tls_version":"tls1.2","url":"https://httpbin.test.k6.io/post"}},"metric":"http_reqs"}
{"type":"Metric","data":{"name":"http_req_duration","type":"trend","contains":"time","tainted":null,"thresholds":["p(95)<=100"],"submetrics":null,"sub":{"name":"","parent":"","suffix":"","tags":null}},"metric":"http_req_duration"}
{"type":"Point","data":{"time":"2022-01-05T12:46:23.603474+01:00","value":118.96,"tags":{"expected_response":"true","group":"","method":"POST","name":"https://httpbin.test.k6.io/post","proto":"HTTP/1.1","scenario":"default","status":"200","tls_version":"tls1.2","url":"https://httpbin.test.k6.io/post"}},"metric":"http_req_duration"}
{"type":"Metric","data":{"name":"http_req_blocked","type":"trend","contains":"time","tainted":null,"thresholds":[],"submetrics":null,"sub":{"name":"","parent":"","suffix":"","tags":null}},"metric":"http_req_blocked"}
```

Mỗi dòng là một metric hoặc một point. Một **metric** định nghĩa các [chỉ số tích hợp sẵn](https://k6.io/docs/using-k6/metrics/#built-in-metrics) hoặc các chỉ số tùy chỉnh theo loại của chúng, các thresholds liên quan đến chúng, hoặc liệu chúng có gây ra bất kỳ thresholds nào thất bại hay không. Một **point** là một phép đo cho một metric, và chứa giá trị thực tế của chỉ số đó tại một dấu thời gian nhất định.

## Kiểm tra kiến thức của bạn

### Câu hỏi 1

Phát biểu nào sau đây về định dạng kết quả CSV mặc định của k6 là đúng?

A: Để vẽ biểu đồ thời gian phản hồi, bạn phải lấy con số trong cột `metric_value` từ mọi dòng trong tệp CSV.

B: Mỗi dòng chứa một chỉ số và một giá trị cho chỉ số đó tại một dấu thời gian cụ thể.

C: Cột `metric_name` trong tệp CSV đề cập đến URL của yêu cầu HTTP đã được gửi.

### Câu hỏi 2

Dưới đây là một dòng từ tệp JSON chứa kết quả kiểm thử k6:

```plain
{"type":"Point","data":{"time":"2022-01-05T12:46:26.893633+01:00","value":100,"tags":null},"metric":"vus_max"}
```

Chúng ta có thể xác định được điều gì từ dòng này?

A: Số lượng VUs tối đa tại thời điểm đó là 100.

B: Phép đo này dành cho chỉ số `http_req_duration`.

C: Bài kiểm tra đã đạt đến số lượng VUs tối đa vào lúc 12:46 ngày 5 tháng 1 năm 2022.

### Câu hỏi 3

Lệnh nào sau đây là lệnh đúng để xuất kết quả k6 ra các định dạng khác nhau?

A: `k6 run test.js --out json=myresults.json`

B: `k6 run test -o csv=results.csv`

C: `k6 output csv=results.csv`

## k6 Cloud

Hai tùy chọn trước đó cho phép bạn xuất kết quả k6 ở các định dạng khác nhau, nhưng chúng vẫn yêu cầu một số thiết lập khi bạn thực hiện phân tích trong một phần mềm trực quan hóa kết quả riêng biệt.

Một giải pháp thay thế cho việc này là sử dụng k6 Cloud. k6 Cloud là một nền tảng SaaS trả phí được xây dựng xung quanh k6 OSS. Bạn có thể sử dụng k6 mà không cần sử dụng k6 Cloud, nhưng k6 Cloud cung cấp một số chức năng bổ sung khá hữu ích, và một trong số đó là trực quan hóa kết quả.

Phần tiếp theo sẽ nói về k6 Cloud, cách sử dụng nó kết hợp với k6 OSS và cách nó có thể giúp bạn phân tích kết quả.

### Đáp án

1. B. A sai vì đầu ra CSV bao gồm nhiều metrics, không chỉ mỗi `http_req_duration`. C sai vì `url` mới là cột chứa URL cho yêu cầu.
2. A. Chỉ số là `vus_max`, chính là số lượng người dùng ảo đang chạy tại thời điểm đó (trong trường hợp này là 100). Tuy nhiên, điều này không nhất thiết tương ứng với số lượng VUs tối đa cho bài kiểm tra, vì bài kiểm tra có thể đã tăng tải xa hơn điểm này.
3. A. Chỉ có A là có cú pháp đúng để xuất kết quả.
