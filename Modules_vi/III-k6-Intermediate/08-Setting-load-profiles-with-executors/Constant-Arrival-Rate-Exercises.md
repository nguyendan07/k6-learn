# Constant Arrival Rate Executor

Như đã lưu ý trong [Thiết lập Load Profiles với Executors](../08-Setting-load-profiles-with-executors.md#Constant-Arrival-Rate), executor _Constant Arrival Rate_ tập trung trọng tâm vào _iteration rate_ (tỷ lệ lần lặp) được áp dụng trong một khung thời gian cụ thể.

Đây là một kịch bản thường thấy khi [kiểm thử tải một API](https://k6.io/docs/testing-guides/api-load-testing/) khi có nhu cầu mô phỏng một tốc độ yêu cầu không đổi cho một API endpoint cụ thể.

## Bài tập

Các bài tập của chúng ta bắt đầu với một kịch bản cơ bản. Nó chạy một yêu cầu HTTP trên mỗi lần lặp kiểm thử và cố gắng đạt được 50 yêu cầu trên mỗi đơn vị thời gian (mặc định là 1 giây). Chúng tôi cung cấp một số đầu ra console khi mọi thứ thay đổi.

### Tạo kịch bản của chúng ta

Hãy bắt đầu bằng cách triển khai kịch bản kiểm thử của chúng ta với cấu hình tối thiểu được yêu cầu. Tạo một tệp tên là _test.js_ với nội dung sau:

```js
import http from "k6/http";

export const options = {
    scenarios: {
        k6_workshop: {
            executor: "constant-arrival-rate",
            rate: 50,
            duration: "30s",
            preAllocatedVUs: 2,
        },
    },
};

export default function () {
    console.log(`[VU: ${__VU}, iteration: ${__ITER}] Starting iteration...`);
    http.get("https://test.k6.io/contacts.php");
}
```

Chúng ta đang bắt đầu với mức tối thiểu để sử dụng executor. So với các executors trước đó, cái này yêu cầu cấu hình nhiều hơn một chút ngoài bản thân `executor` thông thường.

Như đã lưu ý trong [tài liệu constant-arrival-rate](https://k6.io/docs/using-k6/scenarios/executors/constant-arrival-rate/), `rate` và `duration` hiện là bắt buộc. Điều này có ý nghĩa vì trọng tâm của executor này là đạt được và duy trì _iteration rate_ đã chỉ định trong khung thời gian được cung cấp.

Hãy tạm hoãn cuộc thảo luận về `preAllocatedVUs` cho đến khi bài kiểm tra ban đầu của chúng ta kết thúc.

### Chạy bài kiểm tra ban đầu

Hãy tiến hành kích hoạt lần chạy đầu tiên của kịch bản:

```bash
k6 run test.js
```

Bài kiểm tra của bạn bây giờ sẽ bắt đầu in kết quả ra terminal...

```bash
  execution: local
     script: test.js
     output: -

  scenarios: (100.00%) 1 scenario, 2 max VUs, 1m0s max duration (incl. graceful stop):
           * k6_workshop: 50.00 iterations/s for 30s (maxVUs: 2, gracefulStop: 30s)

INFO[0000] [VU: 1, iteration: 0] Starting iteration...   source=console
INFO[0000] [VU: 2, iteration: 0] Starting iteration...   source=console
WARN[0000] Insufficient VUs, reached 2 active VUs and cannot initialize more  executor=constant-arrival-rate scenario=k6_workshop
INFO[0000] [VU: 1, iteration: 1] Starting iteration...   source=console
INFO[0000] [VU: 2, iteration: 1] Starting iteration...   source=console
...
INFO[0030] [VU: 2, iteration: 204] Starting iteration...  source=console
INFO[0030] [VU: 1, iteration: 204] Starting iteration...  source=console

running (0m30.1s), 0/2 VUs, 410 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 0/2 VUs  30s  50.00 iters/s

     data_received..................: 325 kB 11 kB/s
     data_sent......................: 47 kB  1.6 kB/s
     dropped_iterations.............: 1091   36.235918/s
     http_req_blocked...............: avg=3.48ms   min=1µs      med=6µs      max=256.93ms p(90)=10µs     p(95)=12µs
     http_req_connecting............: avg=1.73ms   min=0s       med=0s       max=136.79ms p(90)=0s       p(95)=0s
     http_req_duration..............: avg=132.35ms min=103.06ms med=115.38ms max=827.03ms p(90)=143.56ms p(95)=165.43ms
       { expected_response:true }...: avg=132.35ms min=103.06ms med=115.38ms max=827.03ms p(90)=143.56ms p(95)=165.43ms
     http_req_failed................: 0.00%  ✓ 0         ✗ 410
     http_req_receiving.............: avg=86.77µs  min=19µs     med=80µs     max=524µs    p(90)=142µs    p(95)=153.54µs
     http_req_sending...............: avg=26.88µs  min=7µs      med=24.5µs   max=194µs    p(90)=39.1µs   p(95)=43.54µs
     http_req_tls_handshaking.......: avg=1.74ms   min=0s       med=0s       max=126.36ms p(90)=0s       p(95)=0s
     http_req_waiting...............: avg=132.23ms min=102.98ms med=115.28ms max=826.93ms p(90)=143.43ms p(95)=165.28ms
     http_reqs......................: 410    13.617531/s
     iteration_duration.............: avg=136.06ms min=103.26ms med=115.77ms max=827.24ms p(90)=144.98ms p(95)=183ms
     iterations.....................: 410    13.617531/s
     vus............................: 2      min=2       max=2
     vus_max........................: 2      min=2       max=2
```

Bài kiểm tra của chúng ta đã chạy thành công. Tuy nhiên, việc kiểm tra kỹ hơn cho thấy kết quả của chúng ta không như ý muốn.

Nhìn vào đầu ra, chúng ta thấy dòng sau:

```bash
WARN[0000] Insufficient VUs, reached 2 active VUs and cannot initialize more  executor=constant-arrival-rate scenario=k6_workshop
```

Chuyện gì đã xảy ra ở đây?

Hãy nhớ thiết lập `preAllocatedVUs` mà chúng ta đã lướt qua trước đó? Với thiết lập này, chúng ta đã bảo k6 bắt đầu bài kiểm tra với 2 người dùng ảo. Với quá ít VUs được phân bổ như vậy, k6 **không thể** đạt được tỷ lệ lần lặp mong muốn (50 iterations/s).

Bạn có thể xác nhận sự thật này trong giá trị `iterations` trong tóm tắt kiểm thử; bài kiểm tra của chúng ta chỉ đạt được tỷ lệ 13,61 lần lặp trên giây trong khi chúng ta muốn 50.

### Điều chỉnh số lượng người dùng ảo được phân bổ trước

Để nhắc lại, _Constant Arrival Rate_ tập trung vào _iteration rate_. k6 hướng tới việc loại bỏ nhu cầu phải quá lo lắng về số lượng người dùng thực tế cần thiết để đạt được tỷ lệ đó. Tuy nhiên, chúng ta vẫn phải cung cấp cho kịch bản của mình đủ VUs để đạt được tỷ lệ đó.

Trong ví dụ trước, thời lượng yêu cầu hoặc độ trễ (latency) trung bình là 132,35ms, và phân vị thứ 95 là khoảng 165,43ms. Chỉ với 2 VUs (`preAllocatedVUs`), trong một kịch bản rất lạc quan, chúng ta không thể mong đợi nhiều hơn `2 VUs / 0.132 s = 15.15 iterations/s`.

Thử nghiệm một chút với số lượng `preAllocatedVUs`, chúng ta có thể cập nhật kịch bản của bạn lên một giá trị cao hơn, ví dụ: 25.

```js
export const options = {
    scenarios: {
        k6_workshop: {
            executor: "constant-arrival-rate",
            rate: 50,
            duration: "30s",
            preAllocatedVUs: 25,
        },
    },
};
```

Hãy chạy lại bài kiểm tra của chúng ta một lần nữa:

```bash
k6 run test.js
```

```bash
running (0m30.5s), 00/25 VUs, 1501 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 00/25 VUs  30s  50.00 iters/s

     data_received..................: 1.2 MB 40 kB/s
     data_sent......................: 174 kB 5.7 kB/s
     http_req_blocked...............: avg=4.26ms   min=1µs      med=4µs      max=332.88ms p(90)=8µs      p(95)=14µs
     http_req_connecting............: avg=2.05ms   min=0s       med=0s       max=167.98ms p(90)=0s       p(95)=0s
     http_req_duration..............: avg=136.87ms min=101.78ms med=114.56ms max=862.77ms p(90)=146.68ms p(95)=184.36ms
       { expected_response:true }...: avg=136.87ms min=101.78ms med=114.56ms max=862.77ms p(90)=146.68ms p(95)=184.36ms
     http_req_failed................: 0.00%  ✓ 0         ✗ 1501
     http_req_receiving.............: avg=46.57µs  min=11µs     med=39µs     max=503µs    p(90)=82µs     p(95)=96µs
     http_req_sending...............: avg=17.21µs  min=5µs      med=15µs     max=151µs    p(90)=28µs     p(95)=30µs
     http_req_tls_handshaking.......: avg=2.19ms   min=0s       med=0s       max=203.88ms p(90)=0s       p(95)=0s
     http_req_waiting...............: avg=136.81ms min=101.73ms med=114.5ms  max=862.74ms p(90)=146.57ms p(95)=184.31ms
     http_reqs......................: 1501   49.213783/s
     iteration_duration.............: avg=141.27ms min=101.88ms med=114.83ms max=862.9ms  p(90)=150.38ms p(95)=407.84ms
     iterations.....................: 1501   49.213783/s
     vus............................: 25     min=25      max=25
     vus_max........................: 25     min=25      max=25
```

Xem lại đầu ra bây giờ, chúng ta thấy rằng tỷ lệ mong muốn đã đạt được gần hơn ở mức 49,21 lần lặp trên giây (cho toàn bộ bài kiểm tra).

Bạn có thể bị cám dỗ để thiết lập một giá trị thấp hơn cho [`preAllocatedVUs` và sử dụng tùy chọn `maxVUs`](https://k6.io/docs/using-k6/scenarios/executors/constant-arrival-rate/) để tự động mở rộng (autoscale) bài kiểm tra. `maxVUs` là số lượng VUs tối đa cho phép trong suốt quá trình chạy thử, mặc định là `preAllocatedVUs` khi không được thiết lập. Tuy nhiên, tốt nhất thường là để giá trị mặc định cho `maxVUs` và tăng `preAllocatedVUs`. Bằng cách này, các VUs được phân bổ trước khi bài kiểm tra bắt đầu và sẽ được sử dụng khi (và chỉ khi) cần thiết.

Việc phân bổ VUs giữa chừng bài kiểm tra có thể tốn kém về tài nguyên CPU trong instance trình tạo tải, và có thể làm sai lệch các bài kiểm tra. Với việc phân bổ trước VUs, bài kiểm tra bắt đầu với tất cả các VUs có sẵn―không cần phải chờ để phân bổ thêm khi cần thiết ở giữa quá trình chạy thử.

### Các tùy chọn tỷ lệ (rate) khác

Từ các ví dụ của chúng ta cho đến nay, chúng ta đã căn cứ _iteration rate_ dựa trên số lần lặp mỗi giây. Chúng ta có thể thay đổi mẫu số của tỷ lệ bằng cách sử dụng tùy chọn `timeUnit`. Một lần nữa, theo mặc định, giá trị thiết lập là `1s`.

Ví dụ: giả sử tổng hợp các bản ghi (logs) của một dịch vụ cho thấy nó nhận được 10.000 yêu cầu mỗi giờ. Thay vì tự mình tính toán để chuyển đổi sang số lần lặp mỗi giây, chúng ta có thể để k6 làm việc đó:

```js
export const options = {
    scenarios: {
        k6_workshop: {
            executor: "constant-arrival-rate",
            rate: 10000,
            timeUnit: "1h",
            duration: "30s",
            preAllocatedVUs: 5,
        },
    },
};
```

Chạy kịch bản cho thấy tốc độ 2,78 lần lặp mỗi giây, và chúng ta có thể đạt được điều đó với 5 VUs.

```bash
  scenarios: (100.00%) 1 scenario, 5 max VUs, 1m0s max duration (incl. graceful stop):
           * k6_workshop: 2.78 iterations/s for 30s (maxVUs: 5, gracefulStop: 30s)
...
INFO[0029] [VU: 1, iteration: 16] Starting iteration...  source=console
INFO[0030] [VU: 5, iteration: 16] Starting iteration...  source=console

running (0m30.0s), 0/5 VUs, 84 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 0/5 VUs  30s  2.78 iters/s
...
     iterations.....................: 83    2.682393/s
     vus............................: 4     min=3      max=4
```

### Kết thúc

Với bài tập này, bạn đã thấy cách chạy một bài kiểm tra rất cơ bản và cách bạn có thể cho phép k6 tự động kiểm soát số lượng người dùng ảo để đạt được tỷ lệ lần lặp mong muốn.
