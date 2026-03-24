# Các yêu cầu song song (Parallel requests) trong k6

Các yêu cầu song song là các yêu cầu được gửi cùng một lúc. Các yêu cầu song song đôi khi được gọi là các yêu cầu "đồng thời" (concurrent).

Ảnh chụp màn hình này hiển thị bảng Network trong DevTools của Chrome khi một trang web được tải. Mỗi thanh hiển thị thời điểm các tài nguyên nhúng trong trang được yêu cầu trong suốt 4000 ms đầu tiên.

![](../../images/parallel-requests.png)

Mũi tên màu cam chỉ vào ba yêu cầu song song. Điều này cho thấy máy khách (trong trường hợp này là trình duyệt) đã gửi ba yêu cầu HTTP khác nhau cùng một lúc.

Các yêu cầu song song ảnh hưởng đến kết quả kiểm thử tải vì chúng làm tăng lưu lượng (throughput) kiểm thử. Việc tăng throughput kiểm thử ảnh hưởng đến kiểm thử tải theo hai cách:

- Thứ nhất, việc gửi các yêu cầu song song tốn ít thời gian hơn so với gửi chúng tuần tự, vì vậy trình tạo tải có thể trải qua mức sử dụng tài nguyên cao hơn trong một khoảng thời gian ngắn hơn.
- Thứ hai, các yêu cầu song song cũng đến máy chủ ứng dụng nhanh hơn, điều này cũng có thể làm tăng mức sử dụng tài nguyên ở phía máy chủ.

Nếu các yêu cầu song song có khả năng làm tăng mức sử dụng trên trình tạo tải đang chạy kịch bản _và_ máy chủ ứng dụng, tại sao chúng ta lại sử dụng chúng?

## Khi nào bạn nên sử dụng các yêu cầu song song?

Các yêu cầu song song nên được sử dụng khi chúng cũng được sử dụng trong môi trường thực tế (production). Trong những tình huống này, các yêu cầu song song làm cho bài kiểm tra thực tế hơn. Các trình duyệt hiện đại có một mức độ song song hóa theo mặc định, vì vậy một bài kiểm thử tải cho một trang web hoặc ứng dụng web thường nên tính đến các yêu cầu song song.

Ngược lại, các cuộc gọi API thường có thể _không_ được kích hoạt đồng thời, vì vậy việc gửi các yêu cầu một cách tuần tự có thể sẽ thực tế hơn.

Khi nghi ngờ về việc liệu bạn có nên sử dụng các yêu cầu song song hay không, hãy sử dụng DevTools của trình duyệt hoặc một công cụ bắt gói tin (proxy sniffer) trong khi bạn truy cập ứng dụng của mình để xem các yêu cầu được gửi như thế nào trong môi trường thực tế.

## Gộp nhóm (Batching) trong k6

Theo mặc định, mỗi người dùng k6 gửi từng yêu cầu trong kịch bản một cách tuần tự. Để thay đổi hành vi này, bạn có thể sử dụng tính năng batching.

Batching báo hiệu cho k6 rằng các yêu cầu trong một nhóm (batch) phải được gửi đồng thời, và bạn có thể sử dụng nó như thế này:

```js
import { check } from "k6";
import http from "k6/http";

const domain = "https://test.k6.io";

export default function () {
    let responses = http.batch([
        ["GET", domain + "/"],
        ["GET", domain + "/static/css/site.css"],
        ["GET", domain + "/static/js/prisms.js"],
        ["GET", domain + "/static/favicon.ico"],
    ]);
    check(responses[0], {
        "Homepage successfully loaded": (r) =>
            r.body.includes(
                "Collection of simple web-pages suitable for load testing",
            ),
    });
}
```

Kịch bản trên gửi bốn yêu cầu song song, tất cả đều được nhúng vào trang chủ. Mỗi phản hồi được lưu dưới dạng một mảng trong biến `responses`, và lệnh check xác minh rằng phần tử đầu tiên của mảng đó, `responses[0]`, chính là trang chủ, chứa văn bản chứng minh rằng kịch bản đã truy xuất thành công phần thân HTML.

Bạn có thể batch các yêu cầu ngay cả khi chúng không phải tất cả đều là yêu cầu HTTP GET. Ví dụ, bạn có thể sử dụng các yêu cầu HTTP GET và HTTP POST trong cùng một batch. Bạn có thể tìm thấy [thêm thông tin tại đây](https://k6.io/docs/javascript-api/k6-http/batch-requests/).

## Kiểm tra kiến thức của bạn

### Câu hỏi 1

Phát biểu nào sau đây là đúng?

A: Việc có sử dụng các yêu cầu song song hay không phụ thuộc vào kịch bản kiểm thử của bạn.

B: Sử dụng các yêu cầu song song luôn được khuyến nghị như một thực hành tốt nhất trong kiểm thử hiệu năng.

C: Các yêu cầu song song nên luôn được sử dụng để kiểm thử các trang web.

### Câu hỏi 2

Điều nào sau đây có thể là một tác dụng phụ của việc bạn đưa các yêu cầu song song vào kịch bản của mình?

A: Trình tạo tải của bạn gửi ít yêu cầu hơn khi một số được gộp nhóm (batched).

B: Số lượng yêu cầu trên giây (rps) được thực thi của bài kiểm thử tải sẽ tăng lên.

C: Máy chủ ứng dụng lưu cache các yêu cầu đã gộp nhóm và hiệu năng được cải thiện.

### Câu hỏi 3

Khi nào bạn có thể _không_ muốn sử dụng các yêu cầu song song?

A: Khi bạn muốn tăng số lượng yêu cầu mà bài kiểm tra của bạn đang gửi

B: Khi bạn có các yêu cầu sử dụng nhiều loại phương thức HTTP khác nhau

C: Khi bạn đang kiểm thử các endpoint của API

### Đáp án

1. A. Các yêu cầu song song đôi khi hữu ích, nhưng không phải luôn luôn. Ví dụ, chúng hữu ích nếu bạn đang cố gắng mô phỏng một người dùng truy cập một ứng dụng web trên trình duyệt, nhưng không quá hữu ích nếu bạn đang cố gắng bắt chước các yêu cầu tuần tự đến một API endpoint.
2. B. Nếu các yếu tố khác không đổi, việc tăng tính song song của các yêu cầu sẽ làm tăng lưu lượng (rps) cho bài kiểm tra của bạn.
3. A. Điểm then chốt ở đây là mục tiêu của bạn, không phải là bạn đang kiểm thử _cái gì_. Nếu bạn muốn tăng lưu lượng kiểm thử (số lượng yêu cầu được gửi bởi k6), các yêu cầu song song _sẽ_ là một cách hợp lệ để làm điều đó. Việc bạn đang sử dụng nhiều phương thức HTTP hay kiểm thử các API endpoint là không quan trọng.
