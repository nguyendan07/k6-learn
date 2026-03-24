# Tạo và sử dụng các chỉ số (metrics) tùy chỉnh

Trong phần [Hiểu kết quả k6](../II-k6-Foundations/03-Understanding-k6-results.md), bạn đã học cách diễn giải các chỉ số (metrics) mà k6 báo cáo theo mặc định. Nhưng nếu bạn muốn đo lường một thứ gì đó khác mà k6 không theo dõi thì sao?

Bạn có thể tạo các chỉ số tùy chỉnh (custom metrics) của riêng mình trong kịch bản. Một khi bạn đã định nghĩa những gì bạn muốn đo lường, k6 sẽ thu thập các phép đo trong quá trình kiểm thử và báo cáo các chỉ số tùy chỉnh của bạn cùng với các chỉ số tích hợp sẵn trong báo cáo tóm tắt kết thúc kiểm thử.

## Các loại chỉ số (metrics)

Có [bốn loại](https://k6.io/docs/using-k6/metrics/#metric-types) [chỉ số tùy chỉnh](https://k6.io/docs/using-k6/metrics/#custom-metrics) mà bạn có thể tạo, và mỗi loại đo lường một kiểu dữ liệu khác nhau.

### Counter (Bộ đếm)

Một counter là một số nguyên có thể được tăng lên bất cứ khi nào một sự kiện nhất định xảy ra. Counters rất tốt để theo dõi các sự kiện như:

- số lần một lỗi cụ thể xảy ra
- các tài khoản người dùng có quyền truy cập vào các chức năng Quản trị (Admin)
- tần suất một yêu cầu bị chuyển hướng
- các lượt truy cập của một trang cụ thể, trong trường hợp kịch bản của bạn truy cập các trang được xác định ngẫu nhiên

### Gauge (Thước đo)

Một chỉ số gauge lưu trữ ba con số: giá trị nhỏ nhất, giá trị lớn nhất và giá trị _cuối cùng_. Gauges hữu ích để theo dõi trong thời gian thực các thứ chỉ có liên quan trong khi bài kiểm tra đang chạy. Bạn có thể sử dụng gauges để đo lường:

- kích thước của các thân phản hồi (response bodies) được trả về
- một phiên bản tùy chỉnh của thời gian phản hồi (ví dụ: thời gian phản hồi của ba URL không được nhóm)
- số lượng các đơn đăng ký chưa được xử lý, trong một bài kiểm tra liên quan đến các scenarios tạo đơn đăng ký và xử lý chúng đồng thời

### Rate (Tỷ lệ)

Một chỉ số rate theo dõi một giá trị boolean trên tổng số các phép đo đã thực hiện. Vào cuối bài kiểm tra, rate thể hiện tỷ lệ phần trăm của tất cả các phép đo là `true` và tỷ lệ phần trăm của chúng là `false`. Các chỉ số rate có thể được sử dụng để đo lường:

- bao nhiêu phần trăm VUs đã thực hiện quy trình thanh toán cho một sản phẩm
- tỷ lệ các trang có quảng cáo xuất hiện
- tỷ lệ phần trăm các tài khoản người dùng không đủ quyền hạn

### Trend (Xu hướng)

Một chỉ số trend đo lường các số liệu thống kê tối thiểu, tối đa, trung bình và phân vị. Không giống như gauges, trends giữ _tất cả_ các giá trị và báo cáo sự phân phối của tất cả các giá trị vào cuối bài kiểm tra. Trends tương tự như cách `http_req_duration` (thời gian phản hồi) được đo lường trong k6, và chúng có thể được sử dụng để theo dõi:

- các bộ đếm thời gian tùy chỉnh, chẳng hạn như bộ đếm mà thời gian suy nghĩ (think time) hoặc sleep được cộng vào thời gian phản hồi
- số lần thử lại (retries) được thực hiện trước khi nhận được phản hồi chính xác (chẳng hạn như khi mô phỏng hành vi thăm dò - polling)
- số lượng tài khoản người dùng đang hoạt động được báo cáo cho một tài khoản người dùng quản trị

## Ví dụ: Bộ đếm thời gian tùy chỉnh (Custom timers)

Một lý do phổ biến để sử dụng các chỉ số tùy chỉnh là thiết lập một bộ đếm thời gian cho các hành động cụ thể mà bạn chỉ định. Ví dụ, k6 tự động loại bỏ thời gian sleep trong tính toán thời gian phản hồi, nhưng đôi khi có thể cần phải theo dõi tổng thời gian mà một VU dành cho một phần của kịch bản, bao gồm cả thời gian sleep.

Dưới đây là một kịch bản ngắn bạn có thể sử dụng để tạo một bộ đếm thời gian tùy chỉnh:

```js
import http from "k6/http";
import { sleep, check } from "k6";
import { Trend } from "k6/metrics";

const transactionDuration = new Trend("transaction_duration");

export default function () {
    // This is a transaction
    let timeStart = Date.now();
    let res = http.get("https://test.k6.io", { tags: { name: "01_Home" } });
    check(res, {
        "is status 200": (r) => r.status === 200,
    });
    sleep(Math.random() * 5);
    let timeEnd = Date.now();
    transactionDuration.add(timeEnd - timeStart);
    console.log("The transaction took", timeEnd - timeStart, "ms to complete.");

    // This is another transaction
    res = http.get("https://test.k6.io/about", { tags: { name: "02_About" } });
}
```

Đầu tiên, nhập loại chỉ số yêu cầu.

```js
import { Trend } from "k6/metrics";
```

Trong trường hợp này, chỉ số Trend có vẻ phù hợp nhất, để k6 sẽ báo cáo đầy đủ các số liệu thống kê về bộ đếm thời gian mới của bạn.

Thứ hai, khai báo bộ đếm thời gian của bạn.

```js
const transactionDuration = new Trend("transaction_duration");
```

Thứ ba, xác định nơi bắt đầu và dừng bộ đếm thời gian.

```js
let timeStart = Date.now();
let timeEnd = Date.now();
```

Đặt các dòng này một cách chiến lược trong kịch bản của bạn sao cho việc thực thi tất cả mã nguồn nằm giữa `timeStart` và `timeEnd` được bao gồm trong khoảng thời gian trôi qua mà bạn muốn đo lường.

Cuối cùng, thêm thời gian trôi qua vào bộ đếm thời gian của bạn.

```js
transactionDuration.add(timeEnd - timeStart);
```

Khi bạn chạy kịch bản, bạn sẽ thấy một cái gì đó như thế này:

```plain
checks.........................: 100.00% ✓ 1        ✗ 0
     data_received..................: 17 kB   5.3 kB/s
     data_sent......................: 621 B   194 B/s
     http_req_blocked...............: avg=320.48ms min=15µs     med=320.48ms max=640.95ms p(90)=576.86ms p(95)=608.9ms
     http_req_connecting............: avg=59.06ms  min=0s       med=59.06ms  max=118.12ms p(90)=106.31ms p(95)=112.21ms
     http_req_duration..............: avg=221.24ms min=128.58ms med=221.24ms max=313.89ms p(90)=295.36ms p(95)=304.63ms
       { expected_response:true }...: avg=128.58ms min=128.58ms med=128.58ms max=128.58ms p(90)=128.58ms p(95)=128.58ms
     http_req_failed................: 50.00%  ✓ 1        ✗ 1
     http_req_receiving.............: avg=71µs     min=55µs     med=71µs     max=87µs     p(90)=83.8µs   p(95)=85.4µs
     http_req_sending...............: avg=546µs    min=12µs     med=546µs    max=1.08ms   p(90)=973.2µs  p(95)=1.02ms
     http_req_tls_handshaking.......: avg=258.3ms  min=0s       med=258.3ms  max=516.6ms  p(90)=464.94ms p(95)=490.77ms
     http_req_waiting...............: avg=220.62ms min=127.42ms med=220.62ms max=313.83ms p(90)=295.19ms p(95)=304.51ms
     http_reqs......................: 2       0.623378/s
     iteration_duration.............: avg=3.2s     min=3.2s     med=3.2s     max=3.2s     p(90)=3.2s     p(95)=3.2s
     iterations.....................: 1       0.311689/s
     transaction_duration...........: avg=2893     min=2893     med=2893     max=2893     p(90)=2893     p(95)=2893
     vus............................: 1       min=1      max=1
     vus_max........................: 1       min=1      max=1
```

Chỉ số tùy chỉnh của bạn, `transaction_duration`, được báo cáo cùng với các chỉ số k6 tích hợp sẵn khác.

## Kiểm tra kiến thức của bạn

### Câu hỏi 1

Trong tình huống nào sau đây thì việc sử dụng một chỉ số tùy chỉnh trong kịch bản k6 của bạn có thể là phù hợp?

- [ ] A: Khi bạn muốn biết thời gian phản hồi của một URL nhất định

- [ ] B: Khi bạn được yêu cầu báo cáo tổng cộng có bao nhiêu yêu cầu đã được thực thi trong một bài kiểm tra

- [ ] C: Khi bạn muốn đo lường một thứ gì đó mà các chỉ số tích hợp sẵn không bao quát

### Câu hỏi 2

Loại chỉ số tùy chỉnh nào sẽ tốt nhất để theo dõi có bao nhiêu biểu mẫu đã mở có một hộp kiểm (checkbox) nhất định được đánh dấu?

- [ ] A: Trend

- [ ] B: Counter

- [ ] C: Gauge

### Câu hỏi 3

Làm thế nào bạn có thể nhận được báo cáo về các chỉ số tùy chỉnh sau một bài kiểm tra?

- [ ] A: Tất cả các chỉ số tùy chỉnh được hiển thị trong báo cáo tóm tắt kết thúc kiểm thử.

- [ ] B: Bạn phải đăng nhập vào k6 Cloud để xem kết quả.

- [ ] C: Chỉ các kết quả chỉ số rate và trend được hiển thị trong báo cáo tóm tắt kết thúc kiểm thử.

### Đáp án

1. **C.** Cả A và B đều mô tả các chỉ số đã tồn tại theo mặc định. Các chỉ số tùy chỉnh dùng để tạo và đo lường những thứ nằm ngoài những gì k6 cung cấp sẵn.

2. **B.** Một counter sẽ là tốt nhất để theo dõi số lần xuất hiện của một sự kiện trong quá trình kiểm thử.

3. **A.** Các chỉ số tùy chỉnh được hiển thị tại báo cáo tóm tắt kết thúc kiểm thử, mặc dù chúng _cũng_ có sẵn trong k6 Cloud nếu bạn sử dụng nó. C là sai.
