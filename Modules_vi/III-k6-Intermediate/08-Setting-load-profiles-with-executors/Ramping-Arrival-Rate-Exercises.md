# Bài tập về Ramping Arrival Rate Executor

Như đã lưu ý trong [Thiết lập hồ sơ tải với executors](Setting-load-profiles-with-executors.md#Ramping-Arrival-Rate), executor _Ramping Arrival Rate_ tập trung chủ yếu vào _iteration rate_ (tỷ lệ lần lặp) được áp dụng trong một thời lượng cụ thể thông qua các _stages_ (giai đoạn).

Đây là kịch bản thường thấy trong [kiểm thử căng thẳng (stress testing) hoặc kiểm thử đột biến (spike testing)](https://k6.io/docs/test-types/stress-testing/) khi có nhu cầu đẩy một API vượt quá điểm gãy của nó một cách dần dần hoặc để mô phỏng các đột biến tải cực lớn trong một khoảng thời gian rất ngắn.

## Bài tập

Đối với các bài tập của chúng ta, hãy bắt đầu với một kịch bản cơ bản thực hiện một yêu cầu HTTP và cố gắng đạt được 30 yêu cầu trên mỗi đơn vị thời gian (mặc định là 1 giây) trong _stage_ duy nhất mà chúng ta định nghĩa. Chúng tôi cung cấp một số đầu ra console khi các thông số thay đổi.

### Tạo kịch bản của chúng ta

Hãy bắt đầu bằng cách triển khai kịch bản kiểm thử. Tạo một tệp tên là _test.js_ với nội dung sau:

```js
import http from "k6/http";

export const options = {
    scenarios: {
        k6_workshop: {
            executor: "ramping-arrival-rate",
            stages: [{ target: 30, duration: "30s" }],
            preAllocatedVUs: 2,
        },
    },
};

export default function () {
    console.log(`[VU: ${__VU}, iteration: ${__ITER}] Starting iteration...`);
    http.get("https://test.k6.io/contacts.php");
}
```

Kịch bản "cơ bản" này phức tạp hơn một chút so với hầu hết các ví dụ trước đây. Điều này là do nhu cầu định nghĩa `stages` bên cạnh chính `executor`. Cần ít nhất một _stage_, mỗi giai đoạn phải bao gồm `target` và `duration`. Trong trường hợp này, `target` đại diện cho _iteration rate_ mong muốn đạt được vào cuối khoảng thời gian `duration` trong `stage`. Hãy tạm hoãn việc thảo luận về tham số `preAllocatedVUs` bắt buộc cho đến khi chúng ta hoàn thành lần thực thi kiểm thử ban đầu.

### Chạy bài kiểm tra ban đầu

Hãy tiến hành chạy kịch bản bằng k6 CLI:

```bash
k6 run test.js
```

Bài kiểm tra của bạn bây giờ sẽ bắt đầu in kết quả ra terminal...

```bash
  execution: local
     script: test.js
     output: -

  scenarios: (100.00%) 1 scenario, 2 max VUs, 1m0s max duration (incl. graceful stop):
           * k6_workshop: Up to 30.00 iterations/s for 30s over 1 stages (maxVUs: 2, gracefulStop: 30s)

INFO[0001] [VU: 1, iteration: 0] Starting iteration...   source=console
INFO[0002] [VU: 2, iteration: 0] Starting iteration...   source=console
INFO[0002] [VU: 1, iteration: 1] Starting iteration...   source=console
...
INFO[0015] [VU: 1, iteration: 55] Starting iteration...  source=console
INFO[0015] [VU: 2, iteration: 56] Starting iteration...  source=console
WARN[0015] Insufficient VUs, reached 2 active VUs and cannot initialize more  executor=ramping-arrival-rate scenario=k6_workshop
INFO[0015] [VU: 1, iteration: 56] Starting iteration...  source=console
INFO[0015] [VU: 2, iteration: 57] Starting iteration...  source=console
...
INFO[0030] [VU: 2, iteration: 157] Starting iteration...  source=console
INFO[0030] [VU: 1, iteration: 156] Starting iteration...  source=console

running (0m30.1s), 0/2 VUs, 315 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 0/2 VUs  30s  29.95 iters/s

     data_received..................: 246 kB 8.2 kB/s
     data_sent......................: 36 kB  1.2 kB/s
     dropped_iterations.............: 134    4.455953/s
     http_req_blocked...............: avg=2.96ms   min=2µs      med=8µs      max=280.3ms  p(90)=12µs     p(95)=16µs
     http_req_connecting............: avg=1.37ms   min=0s       med=0s       max=121.04ms p(90)=0s       p(95)=0s
     http_req_duration..............: avg=116.57ms min=100.21ms med=112.53ms max=423.4ms  p(90)=133.78ms p(95)=151.35ms
       { expected_response:true }...: avg=116.57ms min=100.21ms med=112.53ms max=423.4ms  p(90)=133.78ms p(95)=151.35ms
     http_req_failed................: 0.00%  ✓ 0         ✗ 315
     http_req_receiving.............: avg=93.73µs  min=25µs     med=94µs     max=244µs    p(90)=146µs    p(95)=158.29µs
     http_req_sending...............: avg=31.07µs  min=8µs      med=30µs     max=470µs    p(90)=42.6µs   p(95)=47µs
     http_req_tls_handshaking.......: avg=1.49ms   min=0s       med=0s       max=133.12ms p(90)=0s       p(95)=0s
     http_req_waiting...............: avg=116.45ms min=100.05ms med=112.39ms max=423.24ms p(90)=133.68ms p(95)=151.19ms
     http_reqs......................: 315    10.474815/s
     iteration_duration.............: avg=119.81ms min=100.54ms med=112.83ms max=423.76ms p(90)=140.04ms p(95)=154.62ms
     iterations.....................: 315    10.474815/s
     vus............................: 2      min=2       max=2
     vus_max........................: 2      min=2       max=2
```

Mặc dù đã chạy thành công, nhưng việc kiểm tra kỹ hơn các kết quả cho thấy hành vi bài kiểm tra không như mong muốn.

Nhìn vào đầu ra, chúng ta thấy thông báo sau:

```bash
WARN[0015] Insufficient VUs, reached 2 active VUs and cannot initialize more  executor=ramping-arrival-rate scenario=k6_workshop
```

Chuyện gì đã xảy ra?

Với thiết lập `preAllocatedVUs` mà chúng ta đã lướt qua trước đó, chúng ta đã yêu cầu k6 bắt đầu bài kiểm tra với 2 người dùng ảo. Với việc phân bổ trước này, k6 **không thể** đạt được tỷ lệ lần lặp mong muốn trong giai đoạn này.

Sự thật này hiển nhiên khi nhìn vào giá trị `iterations` trong tóm tắt kiểm thử; bài kiểm tra của chúng ta chỉ có thể đạt được tỷ lệ 10,47 lần lặp trên giây trong khi chúng ta muốn 30.

### Điều chỉnh số lượng người dùng ảo được phân bổ trước

Nhắc lại một lần nữa, executor _Ramping Arrival Rate_ tập trung vào _iteration rate_ trong một khung thời gian thuộc mỗi giai đoạn. k6 hướng tới việc loại bỏ nhu cầu phải quá quan tâm đến số lượng người dùng thực tế cần thiết để đạt được tỷ lệ đó. Tuy nhiên, chúng ta vẫn cần cung cấp cho kịch bản đủ VUs để đạt được tỷ lệ đó.

Từ ví dụ trên, chúng ta thấy thời lượng yêu cầu -- hoặc latency (độ trễ) -- trung bình là 116,57ms, và phân vị 95 là khoảng 151,35ms. Với chỉ 2 VUs (`preAllocatedVUs`), trong một kịch bản rất lạc quan, chúng ta không thể mong đợi nhiều hơn `2 VUs / 0,116 s = 17,24 iterations/s`. Ngay cả khi đây là yếu tố duy nhất tác động, điều mà chúng ta sẽ thấy là không phải vậy trong [phần tiếp theo](#hieu-ung-ramping).

Thử nghiệm một chút với số lượng `preAllocatedVUs`, chúng ta có thể cập nhật kịch bản lên một giá trị cao hơn, ví dụ: 10.

```js
export const options = {
    scenarios: {
        k6_workshop: {
            executor: "ramping-arrival-rate",
            stages: [{ target: 30, duration: "30s" }],
            preAllocatedVUs: 10,
        },
    },
};
```

Hãy chạy lại bài kiểm tra một lần nữa:

```bash
k6 run test.js
```

```bash
running (0m30.1s), 00/10 VUs, 449 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 00/10 VUs  30s  29.95 iters/s

     data_received..................: 374 kB 12 kB/s
     data_sent......................: 53 kB  1.8 kB/s
     http_req_blocked...............: avg=5.19ms   min=2µs      med=8µs      max=254.28ms p(90)=12µs     p(95)=13µs
     http_req_connecting............: avg=2.46ms   min=0s       med=0s       max=122.56ms p(90)=0s       p(95)=0s
     http_req_duration..............: avg=115.77ms min=101.27ms med=110.42ms max=182.21ms p(90)=134.9ms  p(95)=143.31ms
       { expected_response:true }...: avg=115.77ms min=101.27ms med=110.42ms max=182.21ms p(90)=134.9ms  p(95)=143.31ms
     http_req_failed................: 0.00%  ✓ 0         ✗ 449
     http_req_receiving.............: avg=87.06µs  min=23µs     med=85µs     max=942µs    p(90)=130µs    p(95)=150.19µs
     http_req_sending...............: avg=31.32µs  min=9µs      med=29µs     max=386µs    p(90)=43µs     p(95)=49.19µs
     http_req_tls_handshaking.......: avg=2.64ms   min=0s       med=0s       max=131.24ms p(90)=0s       p(95)=0s
     http_req_waiting...............: avg=115.65ms min=101.19ms med=110.29ms max=182.12ms p(90)=134.75ms p(95)=143.19ms
     http_reqs......................: 449    14.929648/s
     iteration_duration.............: avg=121.23ms min=101.49ms med=110.94ms max=367.86ms p(90)=138.57ms p(95)=150.07ms
     iterations.....................: 449    14.929648/s
     vus............................: 10     min=10      max=10
     vus_max........................: 10     min=10      max=10
```

Và chúng ta có thể thấy iterations/s đã tăng lên. Tuy nhiên, vẫn chưa đủ.

### Hiệu ứng ramping

Nhìn vào bảng tóm tắt trước đó, có vẻ như _iteration rate_ của chúng ta kết thúc ở mức 14,92 lần lặp trên giây, và chúng ta muốn 30! Số lượng VUs không phải là yếu tố duy nhất tác động khi kiểm soát _iteration rate_.

Điều này gợi lên khía cạnh _ramping_ (tăng/giảm dần) của executor này: k6 sẽ mở rộng hoặc thu hẹp tỷ lệ lần lặp theo đường thẳng để đạt được tỷ lệ `target` trong giai đoạn đó. `duration` sẽ xác định việc tăng/giảm dần sẽ diễn ra trong bao lâu.

Vì chúng ta không chỉ định tùy chọn `startRate`, k6 đã sử dụng giá trị mặc định là 0 lần lặp trên giây. Do đó, bài kiểm tra của chúng ta bắt đầu từ 0, sau đó _tăng dần_ (ramp up) lên 30 lần lặp trên giây trong khoảng thời gian 30 giây.

```text
Rate (Tỷ lệ)

30/s |                                .......
     |                        ......./
     |                ......./
     |        ......./
     |......./
 0/s +---------------------------------------+ 30s
                 S T A G E  # 1
```

Bởi vì chúng ta có sự gia tăng tuyến tính này, việc tỷ lệ tổng thể là 14,92 iterations/s là hợp lý vì con số này xấp xỉ bằng điểm giữa của khoảng từ 0 - 30.

### Thay đổi độ dốc

Với _ramping_, mỗi giai đoạn định nghĩa `target` cần đạt được vào cuối khoảng thời gian `duration`. Đối với giai đoạn đầu tiên, tỷ lệ bắt đầu được định nghĩa bởi tùy chọn `startRate`. Như đã đề cập, nếu bỏ qua, tỷ lệ bắt đầu mặc định sẽ là 0. Hãy làm "phẳng" độ dốc của chúng ta trong ví dụ tiếp theo bằng cách cung cấp một `startRate`:

```js
export const options = {
    scenarios: {
        k6_workshop: {
            executor: "ramping-arrival-rate",
            startRate: 30,
            stages: [{ target: 30, duration: "30s" }],
            preAllocatedVUs: 10,
        },
    },
};
```

Chạy lại bài kiểm tra một lần nữa bằng lệnh `k6 run test.js` sẽ tạo ra kết quả sau:

```bash
running (0m30.8s), 00/10 VUs, 897 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 00/10 VUs  30s  30.00 iters/s

...
     iteration_duration.............: avg=130.68ms min=101.77ms med=112.27ms max=1.54s    p(90)=145.2ms  p(95)=165.15ms
     iterations.....................: 897    29.089145/s
     vus............................: 10     min=10      max=10
     vus_max........................: 10     min=10      max=10
```

Lần này chúng ta yêu cầu bài kiểm tra bắt đầu với tỷ lệ lần lặp là 30, sử dụng 10 VUs, và điều này là đủ để đưa chúng ta đến gần tỷ lệ đó.

```text
Rate (Tỷ lệ)

30/s |........................................
     |
     |
 0/s +---------------------------------------+ 30s
                 S T A G E  # 1
```

Bạn có thể bị cám dỗ để thiết lập một giá trị thấp hơn cho [`preAllocatedVUs` và sử dụng tùy chọn `maxVUs`](https://k6.io/docs/using-k6/scenarios/executors/ramping-arrival-rate/) để tự động mở rộng bài kiểm tra. `maxVUs` là số lượng VUs tối đa cho phép trong quá trình chạy thử, mặc định là `preAllocatedVUs` khi không được thiết lập. Tuy nhiên, tốt nhất thường là để giá trị mặc định cho `maxVUs` và tăng `preAllocatedVUs`. Bằng cách này, các VUs được phân bổ trước khi bài kiểm tra bắt đầu và sẽ được sử dụng khi (và chỉ khi) cần thiết.

Việc phân bổ VUs ở giữa bài kiểm tra có thể tốn kém về tài nguyên CPU trong instance trình tạo tải, và có thể làm sai lệch kết quả kiểm thử. Việc phân bổ trước VUs sẽ cho phép bài kiểm tra bắt đầu với tất cả các VUs có sẵn mà không cần chờ đợi để phân bổ thêm khi cần thiết ở giữa quá trình chạy thử.

Nếu chúng ta muốn một giai đoạn duy nhất với tỷ lệ phẳng---hoặc không đổi (constant)---chúng ta nên sử dụng executor _Constant Arrival Rate_ thay thế! Executor này có thể hữu ích cho [kiểm thử căng thẳng hoặc đột biến](https://k6.io/docs/test-types/stress-testing/) như chúng ta sẽ thấy trong phần tiếp theo.

### Thêm các stages để mô phỏng đột biến

Sức mạnh thực sự của _ramping_ là khả năng mô phỏng các đột biến hoạt động. Điều này có thể được thực hiện bằng cách định nghĩa nhiều giai đoạn (stages).

Giả sử chúng ta có một đợt giảm giá lớn mà chúng ta dự đoán sẽ có một đợt đột biến hoạt động ngay khi mở cửa, duy trì ở đó trong một thời gian ngắn, sau đó dịu xuống ở một tốc độ ổn định nhưng cao hơn bình thường:

```text
Rate (Tỷ lệ)
                  * Tiết kiệm cực khủng khi mở cửa!!
50/s |               .....
     |              /     \.....
30/s |             /            \............
     |             |
10/s |............/
     +------------+--+---+-------+-----------+ 30s
           #1      #2  #3   #4         #5
```

Chúng ta có thể mô phỏng điều này bằng cách cập nhật phần `options` trong kịch bản như sau:

```js
export const options = {
    scenarios: {
        k6_workshop: {
            executor: "ramping-arrival-rate",
            startRate: 10,
            stages: [
                // Duy trì ở mức 10 iters/s trong 6 giây
                { target: 10, duration: "6s" },
                // Đột biến từ 10 iters/s lên 50 iters/s trong 3 giây!
                { target: 50, duration: "3s" },
                // Duy trì ở mức 50 iters/s trong 5 giây
                { target: 50, duration: "5s" },
                // Giảm dần từ 50 iters/s xuống 30 iters/s trong 8 giây
                { target: 30, duration: "8s" },
                // Duy trì ổn định ở mức 30 iters/s cho phần còn lại
                { target: 30, duration: "8s" },
            ],
            preAllocatedVUs: 50,
        },
    },
};
```

### Các tùy chọn tỷ lệ khác

Từ các ví dụ cho đến nay, chúng ta đã căn cứ _iteration rate_ dựa trên số lần lặp mỗi giây. Chúng ta có thể thay đổi mẫu số của tỷ lệ bằng cách sử dụng tùy chọn `timeUnit`. Một lần nữa, theo mặc định, giá trị thiết lập là `1s`.

> :point_up: `timeUnit` áp dụng cho các tỷ lệ `target` trong tất cả các giai đoạn cũng như `startRate`.

Hãy làm chậm ví dụ của chúng ta bằng cách thay đổi tỷ lệ từ số lần lặp mỗi giây sang số lần lặp mỗi phút.

```js
export const options = {
    scenarios: {
        k6_workshop: {
            executor: "ramping-arrival-rate",
            startRate: 10,
            timeUnit: "1m",
            stages: [
                // Duy trì ở mức 10 iters/phút trong 6 giây
                { target: 10, duration: "6s" },
                // Đột biến từ 10 iters/phút lên 50 iters/phút trong 3 giây!
                { target: 50, duration: "3s" },
                // Duy trì ở mức 50 iters/phút trong 5 giây
                { target: 50, duration: "5s" },
                // Giảm dần từ 50 iters/phút xuống 30 iters/phút trong 8 giây
                { target: 30, duration: "8s" },
                // Duy trì ổn định ở mức 30 iters/phút cho phần còn lại
                { target: 30, duration: "8s" },
            ],
            preAllocatedVUs: 50,
        },
    },
};
```

### Kết thúc

Với bài tập này, bạn đã có thể thấy sức mạnh của việc có thể tăng và giảm tỷ lệ lần lặp để mô hình hóa hoạt động kiểm thử của mình một cách thực tế hơn.
