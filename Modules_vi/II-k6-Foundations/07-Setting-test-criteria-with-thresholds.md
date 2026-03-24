k6 khuyến khích các nhà phát triển và người kiểm thử xác định mục tiêu cho mỗi bài kiểm tra. Để làm điều này, bạn có thể thiết lập các [ngưỡng (thresholds)](https://k6.io/docs/using-k6/thresholds) để đánh giá xem bài kiểm tra có thực hiện đúng theo một tiêu chí nhất định hay không. Ví dụ, bạn có thể sử dụng thresholds để khẳng định rằng hệ thống hoạt động trong phạm vi các mục tiêu cấp độ dịch vụ (service-level objectives - SLOs) của bạn trong khi bài kiểm tra đang chạy.

Bạn cũng có thể sử dụng Thresholds để xác định xem một bài kiểm tra là đạt (pass) hay không đạt (fail). Việc thêm thresholds vào một kịch bản load-testing là rất hữu ích vì nó yêu cầu k6 cảnh báo cho bạn khi các ngưỡng đó bị vi phạm, hoặc thậm chí là dừng bài kiểm tra. Việc có thresholds như một phần của mã nguồn giúp các đồng nghiệp khác dễ dàng tiếp quản và chạy bài kiểm tra hơn.

Nếu bạn muốn chạy load tests trong một đường ống CI/CD, bạn cũng sẽ muốn k6 gửi các mã thoát (exit codes) khác không để các thất bại được ghi lại một cách rõ ràng.

Bạn có thể thêm thresholds vào một kịch bản k6 trong đối tượng Test Options:

```js
export let options = {
  stages: [
    { duration: '30m', target: 100 },
    { duration: '1h', target: 100 },
    { duration: '5m', target: 0 },
  ],
  thresholds: {
    http_req_failed: ['rate<=0.05'],
    http_req_duration: ['p(95)<=5000'],
  },
};
```

Thresholds luôn dựa trên các metrics. Bạn có thể xem danh sách đầy đủ các [metrics tích hợp sẵn tại đây](https://k6.io/docs/using-k6/metrics/#built-in-metrics).

Thresholds được thể hiện như một tuyên bố về những gì được kỳ vọng:

- Khi các tuyên bố threshold được đánh giá là `true`, ngưỡng đó được đáp ứng và bài kiểm tra vượt qua. Mã thoát mà k6 trả về là 0.
- Khi các tuyên bố threshold là `false`, ngưỡng đó không được đáp ứng, bài kiểm tra thất bại và k6 trả về một mã thoát khác không.

## Các loại thresholds

Dưới đây là các loại thresholds phổ biến nhất mà bạn có thể thiết lập. Bạn có thể thiết lập nhiều loại thresholds trong một bài kiểm tra duy nhất:

- Tỷ lệ lỗi (Error rate)
- Thời gian phản hồi (Response time)
- Phép kiểm tra (Checks)

**Thực hành kiểm thử tốt nhất**: Sử dụng thresholds cho error rate, response time và checks trong các bài kiểm tra của bạn nếu có thể.

### Tỷ lệ lỗi (Error rate)

```js
thresholds: {
    http_req_failed: ['rate<=0.05'],
},
```

Để thêm một threshold cho tỷ lệ lỗi, hãy sử dụng metric `http_req_failed` và nhập tỷ lệ lỗi mà bạn mong đợi bài kiểm tra sẽ nằm trong phạm vi đó. Theo mặc định, `http_req_failed` tính bất kỳ lỗi HTTP 4xx và HTTP 5xx nào là một thất bại. Bạn có thể thay đổi hành vi này bằng [`setResponseCallback()`](https://k6.io/docs/javascript-api/k6-http/setresponsecallback-callback/).

Threshold ở trên sẽ chỉ được đáp ứng nếu tỷ lệ lỗi trong quá trình kiểm thử nhỏ hơn hoặc bằng 5%.

#### Quy tắc chung: tỷ lệ lỗi

Tỷ lệ lỗi bao nhiêu là tốt? Điều này phụ thuộc vào bài kiểm tra, kịch bản, dữ liệu kiểm thử, ứng dụng và khả năng chịu lỗi của người dùng cuối:

- Đối với các ứng dụng quan trọng mang tính sống còn (mission-critical), hãy cân nhắc tỷ lệ lỗi thấp hơn như 1%.
- Đối với các ứng dụng phụ trợ hoặc không quan trọng, hãy cân nhắc tỷ lệ lỗi cao hơn là 5%.
- Đối với việc kiểm thử các tình huống phục hồi sau thảm họa, nơi bạn đang cố tình chấm dứt hoặc khởi động lại các nút máy chủ, tỷ lệ lỗi từ 10-15% có thể chấp nhận được.

### Thời gian phản hồi (Response time)

```js
thresholds: {
    http_req_duration: ['p(95)<=5000'],
},
```

Trong ví dụ trên, ngưỡng thời gian phản hồi được đặt thành thời gian phản hồi ở phân vị thứ 95 (95th percentile response time) là 5000 mili giây. Tuyên bố này sẽ đúng nếu 95% tất cả các yêu cầu HTTP có thời gian phản hồi từ 5 giây trở xuống.

Bạn có thể thêm một ngưỡng thời gian phản hồi bằng cách sử dụng bất kỳ chỉ số nào được hiển thị trong [tóm tắt kết thúc kiểm thử của k6](03-Understanding-k6-results.md):

```plain
http_req_duration..............: avg=151.06ms min=151.06ms med=151.06ms max=151.06ms p(90)=151.06ms p(95)=151.06ms
```

Bao gồm:

- trung bình (average): `http_req_duration: ['avg<=5000'],`
- tối thiểu (minimum): `http_req_duration: ['min<=1000'],`
- trung vị (median): `http_req_duration: ['med<=3000'],`
- tối đa (maximum): `http_req_duration: ['max<=6000'],`
- các phân vị (percentiles): `http_req_duration: ['p(95)<=5000'],`

**Thực hành kiểm thử tốt nhất**: Tất cả các chỉ số này đều có thể bị làm sai lệch bởi sự hiện diện của các giá trị ngoại lai (outliers) trong thời gian phản hồi ở cả hai thái cực, nhưng nếu bạn không biết nên chọn cái nào, chúng tôi khuyên bạn nên bắt đầu với thời gian phản hồi ở phân vị thứ 95.

#### Quy tắc chung: thời gian phản hồi

[![Google mobile-page-speed-new-industry-benchmarks](../../images/52qQi-marketing-strategies-app-and-mobile-page-load-time-statistics-downlo.jpg)](https://www.thinkwithgoogle.com/marketing-strategies/app-and-mobile/page-load-time-statistics/)

[Theo nghiên cứu của Google](https://www.thinkwithgoogle.com/marketing-strategies/app-and-mobile/mobile-page-speed-new-industry-benchmarks/), xác suất khách hàng tiềm năng rời khỏi trang web của bạn tăng 32% khi thời gian phản hồi tăng lên 3 giây.

Khi nghi ngờ, chúng tôi khuyên bạn nên sử dụng thời gian phản hồi ở phân vị thứ 95 là 2 giây làm điểm bắt đầu.

#### Sử dụng nhiều ngưỡng thời gian phản hồi

Bạn cũng có thể xâu chuỗi nhiều thresholds cho cùng một metric trong một mảng, và thời gian phản hồi là một ứng cử viên tốt cho việc này:

```js
thresholds: {
    http_req_duration: ['p(90) < 400', 'p(95) < 800', 'p(99.9) < 2000'],
},
```

Threshold ở trên tuyên bố rằng:

- 90% tất cả các yêu cầu HTTP nên có thời gian phản hồi thấp hơn 400 ms
- 95% tất cả các yêu cầu HTTP nên có thời gian phản hồi thấp hơn 800 ms
- 99.9% tất cả các yêu cầu HTTP nên có thời gian phản hồi thấp hơn 2000 ms

> :warning: Việc chỉ định nhiều thresholds cho cùng một metric. Nếu bạn muốn bao gồm nhiều hơn một threshold cho cùng một metric, chẳng hạn như `http_req_duration` ở trên, bạn **phải** khai báo chúng trong một mảng. Ví dụ dưới đây sẽ KHÔNG hoạt động, và sẽ dẫn đến việc chỉ dòng cuối cùng được sử dụng làm threshold:

```js
thresholds: {
  http_req_duration: ['p(90) < 400'],
  http_req_duration: ['p(95) < 800'],
  http_req_duration: ['p(99.9) < 2000'],
},
```

### Phép kiểm tra (Checks)

Như bạn đã học trong phần [Thêm checks vào kịch bản của bạn](04-Adding-checks-to-your-script.md), các checks thất bại được báo cáo, nhưng chúng không ảnh hưởng đến trạng thái của toàn bộ bài kiểm tra. Nếu bạn muốn làm cho bài kiểm tra thất bại khi đạt đến một tỷ lệ lỗi check nhất định, bạn có thể sử dụng kết hợp các checks và thresholds:

```js
thresholds: {
  checks: ['rate>=0.9'],
},
```

Threshold ở trên tuyên bố rằng 90% hoặc nhiều hơn tất cả các checks trong bài kiểm tra phải thành công. Nếu không, bài kiểm tra sẽ thất bại.

## Hủy bỏ bài kiểm tra khi thất bại (Aborting test on fail)

Có thể có những tình huống mà bạn muốn bài kiểm tra dừng chạy nếu các thresholds không được đáp ứng. Ví dụ, nếu một bài kiểm tra có tỷ lệ lỗi rất cao do một thành phần ứng dụng không phản hồi, tốt hơn là nên dừng bài kiểm tra và bắt đầu khắc phục sự cố.

Trong những tình huống này, bạn có thể mở rộng threshold liên quan để yêu cầu k6 hủy bỏ bài kiểm tra nếu ngưỡng không được đáp ứng. Thay vì:

```js
thresholds: {
    http_req_failed: ['rate<=0.05'],
},
```

hãy thử cách này:

```js
thresholds: {
    http_req_failed: [{
      threshold: 'rate<=0.05',
      abortOnFail: true,
    }],
},
```

Tham số bổ sung `abortOnFail: true` hướng dẫn k6 dừng bài kiểm tra (mà không có [graceful stop](https://k6.io/docs/misc/glossary/#graceful-stop)) ngay khi ngưỡng bị vượt qua. Trong trường hợp này, điều đó xảy ra khi tỷ lệ lỗi vượt quá 5%. k6 sẽ hiển thị báo cáo tóm tắt kết thúc kiểm thử với một lỗi trông như thế này:

```plain
ERRO[0012] some thresholds have failed
```

và bạn sẽ biết rằng bài kiểm tra đã bị hủy bỏ do vi phạm threshold.

## Tự chạy nó!

Kịch bản của bạn nên trông giống như thế này:

```js
import http from 'k6/http';
import { check, sleep } from 'k6';

export let options = {
  stages: [
    { duration: '30m', target: 100 },
    { duration: '1h', target: 100 },
    { duration: '5m', target: 0 },
  ],
  thresholds: {
    http_req_failed: [{
      threshold: 'rate<=0.05',
      abortOnFail: true,
    }],
    http_req_duration: ['p(95)<=100'],
    checks: ['rate>=0.99'],
  },
};

export default function() {
  let url = 'https://httpbin.test.k6.io/post';
  let response = http.post(url, 'Hello world!');
  check(response, {
      'Application says hello': (r) => r.body.includes('Hello world!')
  });

  sleep(Math.random() * 5);
}
```

Sao chép kịch bản, lưu lại và thực hiện lệnh `k6 run test.js` để chạy bài kiểm tra với thresholds.

## Kiểm tra kiến thức của bạn

### Câu hỏi 1

Đội ngũ của bạn đã quyết định hướng tới thời gian phản hồi ở phân vị thứ 95 là 2 giây. Cách tốt nhất để thể hiện điều này dưới dạng một threshold là gì?

A: `http_req_duration: ['p(95)>2000'],`

B: `http_response_time: ['avg<=2000'],`

C: `http_req_duration: ['p(95)<=2000'],`

### Câu hỏi 2

Kịch bản kiểm thử của bạn có threshold sau được định nghĩa trong các tùy chọn kiểm thử:

```js
thresholds: {
    checks: ['rate>=0.99'],
},
```

Bài kiểm tra chạy trong 100 lần lặp, mỗi lần lặp có 1 yêu cầu HTTP. Có 3 yêu cầu HTTP không vượt qua check. Bạn mong đợi kết quả nào sau đây?

A: Bài kiểm tra sẽ hủy bỏ khi thất bại ngưỡng check.

B: Bài kiểm tra sẽ chạy đến khi hoàn thành và được đánh dấu là thành công vì các thất bại check không làm hỏng bài kiểm tra.

C: Bài kiểm tra sẽ chạy đến khi hoàn thành, nhưng sẽ có thông báo lỗi rằng một số thresholds đã thất bại.

### Câu hỏi 3

Kịch bản kiểm thử của bạn có threshold sau được định nghĩa trong các tùy chọn kiểm thử:

```js
thresholds: {
	http_req_failure: ['rate<=0.05'],
	http_req_failure: ['rate<=0.03'],
	http_req_duration: ['p(90)<4000'],
}
```

Phát biểu nào sau đây là đúng?

A: Bài kiểm tra sẽ thất bại nếu tỷ lệ lỗi là 5% hoặc cao hơn.

B: Bài kiểm tra sẽ thất bại nếu tỷ lệ lỗi vượt quá 3%.

C: Bài kiểm tra sẽ vượt qua nếu thời gian phản hồi ở phân vị thứ 90 là 4 giây.

## Một ghi chú về thresholds

Thresholds hữu ích như những chỉ số ban đầu về sự thành công hay thất bại của một lần chạy thử. Tuy nhiên, chúng không nên được sử dụng như cách _duy nhất_ để đánh giá một bài kiểm tra.

Hạn chế chính của thresholds là chúng dựa trên các metrics, và các metrics có thể bị ảnh hưởng nặng nề bởi sự hiện diện của một vài phép đo ngoại lai cực thấp hoặc cực cao.

Các metrics như phân vị (percentiles) thực hiện công việc mô tả vị trí các phép đo rơi vào tốt hơn một chút, nhưng không có gì thay thế được việc vẽ đồ thị cho từng phép đo đó và tự mình nhìn thấy hình dạng phân phối của dữ liệu.

Vấn đề là k6 OSS không tạo ra các biểu đồ một cách nguyên bản. Trong phần tiếp theo, bạn sẽ tìm hiểu về cách xuất kết quả để sử dụng trong công cụ trực quan hóa kết quả mà bạn chọn.

### Đáp án

1. C. Thresholds được thể hiện theo những gì sẽ khiến chúng vượt qua, vì vậy C là câu trả lời đúng vì nó mô tả một bài kiểm tra có thời gian phản hồi ở phân vị thứ 95 nhỏ hơn (nhanh hơn) hoặc bằng 2000 ms, tức là 2 giây. Bất cứ điều gì vượt quá mức này sẽ khiến threshold thất bại.
2. C. Thresholds không hủy bỏ bài kiểm tra khi thất bại [trừ khi `abortOnFail` được đặt thành `true`](https://k6.io/docs/using-k6/thresholds/#aborting-a-test-when-a-threshold-is-crossed), vì vậy chỉ có C là đúng. Bài kiểm tra vẫn sẽ thực hiện hết quá trình chạy, nhưng một thông báo sẽ được hiển thị trong tóm tắt kết thúc kiểm thử để nói rằng một số thresholds đã thất bại.
3. B. A sai vì khi các thresholds mâu thuẫn được định nghĩa, cái cuối cùng sẽ được sử dụng, vì vậy ngưỡng trên `http_req_failure` sẽ được đặt thành 3% thay vì 5%. C sai vì ngưỡng được thiết lập để vượt qua nếu thời gian phản hồi _nhỏ hơn_ 4 giây, vì vậy thời gian phản hồi 4 giây sẽ khiến nó thất bại.
