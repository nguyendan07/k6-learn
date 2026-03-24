# Per VU Iterations Executor

Như đã lưu ý trong [Thiết lập Load Profiles với Executors](../08-Setting-load-profiles-with-executors.md#Per-VU-Iterations), _Per VU Iterations_ là một executor tập trung vào các _lần lặp (iterations)_ được thực hiện bởi một _người dùng ảo (VU)_.

## Bài tập

Đối với các bài tập của chúng ta, chúng ta sẽ bắt đầu bằng cách sử dụng một kịch bản rất cơ bản, đơn giản là thực hiện một yêu cầu HTTP và sau đó đợi một giây trước khi hoàn thành lần lặp kiểm thử. Chúng tôi sẽ cung cấp một số đầu ra console khi mọi thứ thay đổi.

### Tạo kịch bản của chúng ta

Hãy bắt đầu bằng cách triển khai kịch bản kiểm thử của chúng ta. Tạo một tệp tên là _test.js_ với nội dung sau:

```js
import http from "k6/http";
import { sleep } from "k6";

export const options = {
    scenarios: {
        k6_workshop: {
            executor: "per-vu-iterations",
        },
    },
};

export default function () {
    console.log(`[VU: ${__VU}, iteration: ${__ITER}] Starting iteration...`);
    http.get("https://test.k6.io/contacts.php");
    sleep(1);
}
```

### Chạy bài kiểm tra ban đầu

Chúng ta đang bắt đầu với mức tối thiểu để sử dụng executor, chỉ yêu cầu chúng ta chỉ định chính `executor`. Bây giờ chúng ta đã định nghĩa kịch bản cơ bản của mình, chúng ta sẽ tiến hành chạy k6:

```bash
k6 run test.js
```

Nhìn vào kết quả của chúng ta, bạn sẽ thấy xác nhận rằng chúng ta đã chạy một lần lặp kiểm thử duy nhất từ một người dùng ảo duy nhất.

```bash
INFO[0000] [VU: 1, iteration: 0] Starting iteration...   source=console

running (00m01.3s), 0/1 VUs, 1 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 1 VUs  00m01.3s/10m0s  1/1 iters, 1 per VU
```

### Tăng số lượng người dùng ảo

Bằng cách không chỉ định số lượng người dùng ảo cho bài kiểm tra của chúng ta, giá trị mặc định sẽ là một người dùng duy nhất. Như đã ngụ ý trong cái tên, số lượng VUs là một khía cạnh quan trọng của executor này. Hãy tăng số lượng người dùng ảo bằng tùy chọn `vus` để chúng ta mô phỏng 10 người dùng. Chúng ta sẽ cập nhật phần `options` trong kịch bản kiểm thử của mình:

```js
export const options = {
    scenarios: {
        k6_workshop: {
            executor: "per-vu-iterations",
            vus: 10,
        },
    },
};
```

Chạy lại kịch bản:

```bash
k6 run test.js
```

Nhìn vào đầu ra, bây giờ bạn sẽ thấy rằng bài kiểm tra của chúng ta đã chạy 10 lần lặp. Nói cách khác, _1 lần lặp trên mỗi VU_.

```bash
INFO[0000] [VU: 7, iteration: 0] Starting iteration...   source=console
INFO[0000] [VU: 2, iteration: 0] Starting iteration...   source=console
INFO[0000] [VU: 1, iteration: 0] Starting iteration...   source=console
INFO[0000] [VU: 5, iteration: 0] Starting iteration...   source=console
INFO[0000] [VU: 3, iteration: 0] Starting iteration...   source=console
INFO[0000] [VU: 4, iteration: 0] Starting iteration...   source=console
INFO[0000] [VU: 8, iteration: 0] Starting iteration...   source=console
INFO[0000] [VU: 9, iteration: 0] Starting iteration...   source=console
INFO[0000] [VU: 6, iteration: 0] Starting iteration...   source=console
INFO[0000] [VU: 10, iteration: 0] Starting iteration...  source=console

running (00m01.7s), 00/10 VUs, 10 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 10 VUs  00m01.7s/10m0s  10/10 iters, 1 per VU
```

### Thay đổi số lần lặp

Trong ví dụ trước, chúng ta đã không chỉ định mình mong muốn bao nhiêu lần lặp, vì vậy k6 đã sử dụng giá trị mặc định là 1 cho mỗi VU trong số 10 VUs, do đó dẫn đến tổng cộng 10 lần lặp. Mở rộng điều này, hãy tăng tùy chọn `iterations` lên 20. Bởi vì các lần lặp là _trên mỗi VU_, chúng ta sẽ mong đợi tổng cộng 200 (`10 VUs * 20 iterations`) lần lặp cho quá trình chạy thử.

```js
export const options = {
    scenarios: {
        k6_workshop: {
            executor: "per-vu-iterations",
            vus: 10,
            iterations: 20,
        },
    },
};
```

Một lần nữa, chúng ta sẽ thực thi kịch bản với k6:

```bash
k6 run test.js
```

Đúng như dự đoán, bài kiểm tra của chúng ta đã chạy tổng cộng 200 lần lặp. Hãy nhìn kỹ hơn một chút vào các kết quả này:

```bash
INFO[0022] [VU: 1, iteration: 19] Starting iteration...  source=console
INFO[0022] [VU: 8, iteration: 19] Starting iteration...  source=console
INFO[0022] [VU: 9, iteration: 18] Starting iteration...  source=console
INFO[0022] [VU: 2, iteration: 18] Starting iteration...  source=console
INFO[0022] [VU: 3, iteration: 19] Starting iteration...  source=console
INFO[0022] [VU: 5, iteration: 19] Starting iteration...  source=console
INFO[0022] [VU: 6, iteration: 19] Starting iteration...  source=console
INFO[0022] [VU: 10, iteration: 19] Starting iteration...  source=console
INFO[0022] [VU: 4, iteration: 19] Starting iteration...  source=console
INFO[0022] [VU: 7, iteration: 18] Starting iteration...  source=console
INFO[0023] [VU: 9, iteration: 19] Starting iteration...  source=console
INFO[0023] [VU: 2, iteration: 19] Starting iteration...  source=console
INFO[0024] [VU: 7, iteration: 19] Starting iteration...  source=console

running (00m25.1s), 00/10 VUs, 200 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 10 VUs  00m25.1s/10m0s  200/200 iters, 20 per VU

```

> :point_up: Bộ đếm lần lặp bắt đầu từ 0, nghĩa là số đếm 19 thực chất là 20 lần lặp.

Từ đầu ra, bạn nên lưu ý rằng _VU #1_ đã hoàn thành sớm khi so sánh với _VU #7_. Trên thực tế, _VU #7_ vẫn phải hoàn thành 2 lần lặp **sau khi** _VU #1_ đã kết thúc. Điều này có thể được coi là _lập lịch chia sẻ công bằng (fair-share scheduling)_; mỗi VU thực hiện cùng một lượng công việc.

Tương tự như những ngày đầu đi học, một khi bạn hoàn thành bài kiểm tra của mình, bạn phải ngồi chờ rảnh rỗi cho phần còn lại của lớp học kết thúc hoặc cho đến khi tiếng chuông trường vang lên. Nói về việc rung chuông trường...

### Thiết lập giới hạn thời gian

Cho đến nay, với ví dụ của chúng ta, chúng ta đã có dư dả thời gian để kịch bản của mình hoàn thành các lần lặp mong muốn. Bởi vì chúng ta đã không chỉ định `maxDuration`, k6 sử dụng giá trị mặc định là 10 phút.

Chúng ta sẽ cập nhật kịch bản kiểm thử của mình để bao gồm một `maxDuration` là 10 giây:

```js
export const options = {
    scenarios: {
        k6_workshop: {
            executor: "per-vu-iterations",
            vus: 10,
            iterations: 20,
            maxDuration: "10s",
        },
    },
};
```

> :point_up: Các thời lượng (durations) được cấu hình dưới dạng các giá trị chuỗi bao gồm một số nguyên dương và một hậu tố đại diện cho đơn vị thời gian. Ví dụ, "s" cho giây, "m" cho phút.

Như trước, chạy kịch bản với `k6 run test.js`.

Để kịch bản chạy, chúng ta thấy rằng kịch bản của mình đã không thể hoàn thành tất cả các lần lặp do đạt đến giới hạn thời gian. _Cả lớp dừng bút!_

```bash
INFO[0010] [VU: 9, iteration: 9] Starting iteration...   source=console
INFO[0010] [VU: 6, iteration: 9] Starting iteration...   source=console
INFO[0010] [VU: 10, iteration: 9] Starting iteration...  source=console

running (11.0s), 00/10 VUs, 95 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 10 VUs  10s  095/200 iters, 20 per VU
```

Kịch bản của chúng ta chỉ có thể hoàn thành 95 lần lặp trong khung thời gian 10 giây cho phép. Bằng chứng trước đó cho thấy đầy đủ 200 lần lặp thường hoàn thành trong 25 giây.

Chúng ta có thể sử dụng thông tin đó làm đường cơ sở (baseline) để thiết lập một _Thỏa thuận mức dịch vụ (SLA)_ cho dịch vụ đang được kiểm thử; chúng ta sẽ tính đến một số biến động bằng cách thiết lập `maxDuration` thành 30 giây. Nếu các bài kiểm tra không thể hoàn thành trong khung thời gian đó, có khả năng hiệu năng dịch vụ đã bị giảm sút đến mức cần phải điều tra.

### Kết thúc

Với bài tập này, bạn đã thấy cách chạy một bài kiểm tra rất cơ bản và cách bạn có thể kiểm soát số lượng lần lặp, người dùng ảo, và thậm chí là thiết lập các giới hạn thời gian. Ngoài ra, bạn thấy rằng việc phân bổ các bài kiểm tra giữa các VUs là _được lập lịch công bằng_ khi sử dụng _Per VU Iterations_.
