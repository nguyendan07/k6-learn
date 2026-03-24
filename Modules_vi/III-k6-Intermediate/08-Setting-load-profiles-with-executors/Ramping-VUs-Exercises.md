# Bài tập về Ramping VUs Executor

Như đã lưu ý trong [Thiết lập hồ sơ tải với executors](../08-Setting-load-profiles-with-executors.md#Ramping-VUs), executor _Ramping VUs_ chủ yếu tập trung vào số lượng _người dùng ảo (VUs)_ trong các _stages_ (giai đoạn).

## Bài tập

Đối với các bài tập của chúng ta, chúng ta sẽ bắt đầu bằng cách sử dụng một kịch bản rất cơ bản, đơn giản là thực hiện một yêu cầu HTTP và sau đó đợi một giây trước khi hoàn thành lần lặp kiểm thử. Chúng tôi cung cấp một số đầu ra console khi các thông số thay đổi.

### Tạo kịch bản của chúng ta

Hãy bắt đầu bằng cách triển khai kịch bản kiểm thử. Tạo một tệp tên là _test.js_ với nội dung sau:

```js
import http from "k6/http";
import { sleep } from "k6";

export const options = {
    scenarios: {
        k6_workshop: {
            executor: "ramping-vus",
            stages: [{ target: 10, duration: "30s" }],
        },
    },
};

export default function () {
    console.log(`[VU: ${__VU}, iteration: ${__ITER}] Starting iteration...`);
    http.get("https://test.k6.io/contacts.php");
    sleep(1);
}
```

Kịch bản "cơ bản" của chúng ta phức tạp hơn một chút so với một số ví dụ trước đây. Điều này là do nhu cầu định nghĩa `stages` bên cạnh chính `executor`. Cần ít nhất một _stage_, mỗi giai đoạn bao gồm tùy chọn `target` và `duration`, cả hai đều là bắt buộc.

### Chạy bài kiểm tra ban đầu

Hãy tiến hành chạy kịch bản bằng k6 CLI:

```bash
k6 run test.js
```

Nhìn vào đầu ra từ kịch bản, bạn sẽ thấy rằng một vài người dùng ảo (VUs) đã thực hiện các lần lặp kiểm thử trong khoảng thời gian 30 giây.

```bash
INFO[0029] [VU: 2, iteration: 18] Starting iteration...  source=console
INFO[0029] [VU: 7, iteration: 2] Starting iteration...   source=console
INFO[0029] [VU: 4, iteration: 21] Starting iteration...  source=console
INFO[0029] [VU: 6, iteration: 15] Starting iteration...  source=console
INFO[0030] [VU: 5, iteration: 12] Starting iteration...  source=console

running (0m31.0s), 00/10 VUs, 141 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 00/10 VUs  30s
```

> :point_up: Bộ đếm lần lặp bắt đầu từ 0, nghĩa là số đếm 12 thực chất là 13 lần lặp.

Việc kiểm tra kỹ hơn các số lượng lần lặp có thể thấy lạ. Các con số dường như rải rác khắp nơi: _VU #7_ chỉ thực hiện 3 lần lặp, trong khi _VU #4_ thực hiện 22. Giống như executor [Constant VUs](../08-Setting-load-profiles-with-executors.md#Constant-VUs), mỗi người dùng ảo thực thi khối `default function ()` liên tục, vậy tại sao lại có sự chênh lệch như vậy?

Sự chênh lệch về số lượng lần lặp này là do khía cạnh _ramping_ (tăng/giảm dần) của executor. k6 sẽ mở rộng hoặc thu hẹp số lượng VUs đang chạy theo đường thẳng để đạt được số lượng VUs `target` được định nghĩa trong giai đoạn. `duration` sẽ xác định việc thay đổi quy mô diễn ra trong bao lâu.

Vì chúng ta không chỉ định tùy chọn `startVUs`, k6 đã sử dụng giá trị mặc định là 1. Do đó, bài kiểm tra của chúng ta bắt đầu với một VU duy nhất, sau đó _tăng dần_ lên 10 VUs trong khoảng thời gian 30 giây. Vì lý do này, chúng ta có thể suy ra rằng _VU #7_ đã hoạt động muộn hơn nhiều trong giai đoạn so với _VU #4_.

```text
VUs

10 |                                .......
   |                        ......./
 5 |                ......./
   |        ......./
 1 |......./
   +---------------------------------------+ 30s
                 S T A G E  # 1
```

### Đảo ngược độ dốc

Hãy đảo ngược độ dốc của bài kiểm tra bằng cách thay đổi `startVUs` và `target` cho giai đoạn. Chúng ta sẽ cập nhật phần `options` trong kịch bản kiểm thử như sau:

```js
export const options = {
    scenarios: {
        k6_workshop: {
            executor: "ramping-vus",
            stages: [{ target: 1, duration: "30s" }],
            startVUs: 10,
        },
    },
};
```

```bash
INFO[0026] [VU: 3, iteration: 24] Starting iteration...  source=console
INFO[0027] [VU: 6, iteration: 25] Starting iteration...  source=console
INFO[0027] [VU: 7, iteration: 25] Starting iteration...  source=console
INFO[0028] [VU: 6, iteration: 26] Starting iteration...  source=console
INFO[0028] [VU: 7, iteration: 26] Starting iteration...  source=console
INFO[0029] [VU: 6, iteration: 27] Starting iteration...  source=console
INFO[0029] [VU: 7, iteration: 27] Starting iteration...  source=console

running (0m30.5s), 00/10 VUs, 168 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 00/10 VUs  30s
```

Lần này, bạn có thể thấy số lượng VUs thực hiện số lần lặp cao hơn giảm dần theo thời lượng.

```text
VUs

10 |.......
   |       \.......
 5 |               \.......
   |                       \.......
 1 |                               \.......
   +---------------------------------------+ 30s
                 S T A G E  # 1
```

Khi _giảm dần_ (ramping down) số lượng VUs như chúng ta đã làm trong bài tập cuối này, k6 cung cấp một số linh hoạt khi thu hồi tài nguyên. Theo mặc định, các VUs bị nhắm mục tiêu loại bỏ được cung cấp một thời gian ân hạn 30 giây để hoàn thành lần lặp hiện tại mà nó đang thực hiện. Thời gian ân hạn này có thể được điều chỉnh bằng tùy chọn `gracefulRampDown` để tinh chỉnh giá trị nếu cần thiết.

### Mô phỏng các đột biến về số lượng người dùng đang hoạt động

Sức mạnh thực sự của _ramping_ là khả năng mô phỏng các đột biến hoạt động. Điều này có thể được thực hiện bằng cách định nghĩa nhiều giai đoạn (stages).

```text
VUs

10 |                 .
   |                / \
 5 |               /   \
   |              /     \............
 1 |............./
   +------------+---+---+------------+ 30s
        #1        #2  #3       #4
```

Từ biểu đồ _nghệ thuật ASCII_ đáng yêu của chúng ta, chúng ta sẽ sửa đổi kịch bản để chạy 4 giai đoạn nhằm mô phỏng một đợt đột biến người dùng ảo. Hãy thay đổi `startVUs`, giai đoạn đầu tiên, và thêm các giai đoạn mới vào mô phỏng của chúng ta:

```js
export const options = {
    scenarios: {
        k6_workshop: {
            executor: "ramping-vus",
            stages: [
                { target: 1, duration: "12s" },
                { target: 10, duration: "3s" },
                { target: 3, duration: "3s" },
                { target: 3, duration: "12s" },
            ],
            startVUs: 1,
        },
    },
};
```

Chạy lại kịch bản một lần nữa:

```bash
k6 run test.js
```

Bây giờ, bạn sẽ nhận thấy rằng bài kiểm tra bắt đầu chậm, sau đó tăng nhanh khi đợt đột biến xảy ra, sau đó giảm xuống tốc độ vừa phải cho đến khi kết thúc bài kiểm tra.

```bash
INFO[0028] [VU: 3, iteration: 26] Starting iteration...  source=console
INFO[0028] [VU: 2, iteration: 14] Starting iteration...  source=console
INFO[0028] [VU: 7, iteration: 15] Starting iteration...  source=console
INFO[0029] [VU: 3, iteration: 27] Starting iteration...  source=console
INFO[0029] [VU: 2, iteration: 15] Starting iteration...  source=console
INFO[0029] [VU: 7, iteration: 16] Starting iteration...  source=console

running (0m30.7s), 00/10 VUs, 80 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 00/10 VUs  30s
```

### Kết thúc

Với bài tập này, bạn đã có thể thấy sức mạnh của việc có thể tăng và giảm số lượng VUs để mô hình hóa hoạt động kiểm thử của mình.
