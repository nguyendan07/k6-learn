# Constant VUs Executor

Như đã lưu ý trong [Thiết lập Load Profiles với Executors](../08-Setting-load-profiles-with-executors.md#Constant-VUs), executor _Constant VUs_ tập trung trọng tâm vào số lượng _người dùng ảo (VUs)_ chạy trong một khung thời gian cụ thể.

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
            executor: "constant-vus",
            duration: "30s",
        },
    },
};

export default function () {
    console.log(`[VU: ${__VU}, iteration: ${__ITER}] Starting iteration...`);
    http.get("https://test.k6.io/contacts.php");
    sleep(1);
}
```

> :point_up: Nếu bạn đã thực hiện các bài tập hội thảo này theo thứ tự, bạn có thể nhận thấy rằng lần này, kịch bản ban đầu của chúng ta bao gồm một `duration`. Tùy chọn này là **bắt buộc** đối với executor này. Việc cố gắng chạy kịch bản mà không có tùy chọn này sẽ dẫn đến lỗi cấu hình.

### Chạy bài kiểm tra ban đầu

Chúng ta đang bắt đầu với mức tối thiểu để sử dụng executor, yêu cầu cả `executor` cũng như `duration` cho bài kiểm tra. Bây giờ chúng ta đã định nghĩa kịch bản cơ bản của mình, chúng ta sẽ tiến hành chạy k6:

```bash
k6 run test.js
```

Nhìn vào đầu ra từ kịch bản đang chạy, bạn sẽ thấy một người dùng ảo (VU) duy nhất đang thực hiện liên tục các lần lặp kiểm thử. Bài kiểm tra sẽ hoàn thành khi đạt đến thời lượng 30 giây đã cấu hình. Số lượng lần lặp được thực hiện hoàn toàn phụ thuộc vào mã nguồn đang được thực hiện trong khối mã `default function ()`.

```bash
INFO[0026] [VU: 1, iteration: 24] Starting iteration...  source=console
INFO[0027] [VU: 1, iteration: 25] Starting iteration...  source=console
INFO[0028] [VU: 1, iteration: 26] Starting iteration...  source=console
INFO[0029] [VU: 1, iteration: 27] Starting iteration...  source=console

running (0m30.5s), 0/1 VUs, 28 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 1 VUs  30s
```

### Thay đổi tính đồng thời (concurrency)

Một lần nữa, bài kiểm tra của chúng ta chỉ chạy một người dùng ảo hoặc VU duy nhất. Chúng ta có thể cập nhật tùy chọn `vus` để tăng số lượng yêu cầu được thực hiện đồng thời. Hãy thay đổi `vus` để mô phỏng 10 người dùng bằng cách cập nhật phần `options` trong kịch bản kiểm thử của chúng ta:

```js
export const options = {
    scenarios: {
        k6_workshop: {
            executor: "constant-vus",
            duration: "30s",
            vus: 10,
        },
    },
};
```

Chạy lại kịch bản một lần nữa:

```bash
k6 run test.js
```

Bây giờ, bạn sẽ thấy hoạt động nhiều hơn hẳn vì hiện tại chúng ta có 10 VUs thực hiện các lần lặp đi lặp lại cho đến khi đạt đến thời lượng đã cấu hình.

```bash
INFO[0029] [VU: 4, iteration: 27] Starting iteration...  source=console
INFO[0029] [VU: 8, iteration: 27] Starting iteration...  source=console
INFO[0029] [VU: 3, iteration: 27] Starting iteration...  source=console
INFO[0029] [VU: 9, iteration: 27] Starting iteration...  source=console
INFO[0029] [VU: 6, iteration: 27] Starting iteration...  source=console

running (0m30.5s), 00/10 VUs, 280 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 10 VUs  30s
```

> :point_up: Bộ đếm lần lặp (iteration counter) bắt đầu từ 0, nghĩa là số đếm 27 thực chất là 28 lần lặp.

Từ kết quả, bạn có thể thấy một số yêu cầu đan xen nhau cũng như một số VUs có khả năng thực hiện nhiều lần lặp hơn những VUs khác. Điều này sẽ phụ thuộc vào thời gian phản hồi của từng yêu cầu riêng lẻ.

### Kết thúc

Với bài tập này, bạn đã thấy cách chạy một bài kiểm tra rất cơ bản và cách bạn có thể kiểm soát mức độ đồng thời trong một thời lượng cụ thể. Ngoài ra, bạn đã học được rằng số lượng lần lặp được thực hiện hoàn toàn phụ thuộc vào thời gian cần thiết để hoàn thành mỗi lần lặp của khối `default function ()`.
