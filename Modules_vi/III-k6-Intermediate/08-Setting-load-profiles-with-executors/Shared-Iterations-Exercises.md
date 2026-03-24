# Bài tập về Shared Iterations Executor

Như đã lưu ý trong [Thiết lập hồ sơ tải với executors](../08-Setting-load-profiles-with-executors.md#Shared-Iterations), _Shared Iterations_ là executor cơ bản nhất. Như có thể suy ra từ tên gọi, trọng tâm chính sẽ là số lượng _iterations_ (lần lặp) cho bài kiểm tra của bạn; đây là số lần hàm kiểm thử của bạn sẽ được chạy.

## Bài tập

Đối với các bài tập của chúng ta, chúng ta sẽ bắt đầu bằng cách sử dụng một kịch bản rất cơ bản, đơn giản là thực hiện một yêu cầu HTTP sau đó đợi một giây trước khi hoàn thành lần lặp kiểm thử. Chúng tôi cung cấp một số đầu ra console khi các thông số thay đổi.

### Tạo kịch bản của chúng ta

Hãy bắt đầu bằng cách triển khai kịch bản kiểm thử. Tạo một tệp tên là _test.js_ với nội dung sau:

```js
import http from "k6/http";
import { sleep } from "k6";

export const options = {
    scenarios: {
        k6_workshop: {
            executor: "shared-iterations",
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

Nhìn vào kết quả, bạn sẽ thấy xác nhận rằng chúng ta thực sự đã chạy một lần lặp kiểm thử duy nhất.

```bash
INFO[0000] [VU: 1, iteration: 0] Starting iteration...   source=console

running (00m02.3s), 0/1 VUs, 1 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 1 VUs  00m02.3s/10m0s  1/1 shared iters
```

### Thay đổi số lần lặp

Một bài kiểm tra chỉ với một yêu cầu thì không mang lại nhiều thông tin, vì vậy hãy sửa đổi điều đó. Hãy tăng số lượng `iterations` bài kiểm tra của chúng ta sẽ thực hiện bằng cách sửa đổi phần `options` cho bài kiểm tra.

```js
export const options = {
    scenarios: {
        k6_workshop: {
            executor: "shared-iterations",
            iterations: 200,
        },
    },
};
```

Một lần nữa, chúng ta sẽ thực thi kịch bản bằng k6:

```bash
k6 run test.js
```

Wow... việc này đang tốn khá nhiều thời gian. Có gì đó sai sao? Không. Hãy tiến hành ngắt bài kiểm tra này bằng cách sử dụng `Ctrl+C` nếu nó vẫn chưa kết thúc.

Chúng ta đang thêm thời gian vào các bài kiểm tra một cách nhân tạo, ví dụ: `sleep(1)`, nhằm mục đích minh họa. Bởi vì chúng ta không chỉ định số lượng `vus`, hoặc người dùng ảo (VUs), k6 đang chạy như một người dùng duy nhất thực hiện bài kiểm tra hết lần này đến lần khác. Với 200 lần lặp; cộng với sự trì hoãn nhân tạo, bài kiểm tra của chúng ta sẽ mất hơn 3 phút!

### Thêm giới hạn thời gian

Khi chạy một kịch bản cục bộ, thật dễ dàng để thấy liệu có điều gì đó đang tốn nhiều thời gian hơn dự kiến của bạn và sau đó chấm dứt tiến trình. Trong một đường ống tự động (automated pipeline), hệ thống có thể vui vẻ chờ đợi tiến trình hoàn thành mà không quan tâm đến sự cần thiết của việc chờ đợi hoặc chi phí mà nó có thể gây ra.

Vì lý do này, executor cung cấp tùy chọn `maxDuration` để thiết lập giới hạn thời gian cho bài kiểm tra. Nếu không được chỉ định, executor sẽ sử dụng thời gian chờ mặc định là 10 phút, con số này có thể là quá nhiều hoặc quá ít. Một thực hành tốt nhất là chỉ định `maxDuration` dựa trên phán đoán tốt nhất của bạn hoặc dựa trên các bài kiểm tra trước đó.

Chúng ta sẽ cập nhật kịch bản kiểm thử để bao gồm một `maxDuration` là 30 giây:

```js
export const options = {
    scenarios: {
        k6_workshop: {
            executor: "shared-iterations",
            iterations: 200,
            maxDuration: "30s",
        },
    },
};
```

> :point_up: Các thời lượng (durations) được cấu hình dưới dạng các giá trị chuỗi bao gồm một số nguyên dương và một hậu tố đại diện cho đơn vị thời gian. Ví dụ, "s" cho giây, "m" cho phút.

Như trước, chạy kịch bản với `k6 run test.js`.

Để kịch bản chạy, chúng ta thấy rằng kịch bản của mình đã không thể hoàn thành tất cả các lần lặp do đạt đến giới hạn thời gian.

```bash
INFO[0029] [VU: 1, iteration: 25] Starting iteration...  source=console
INFO[0030] [VU: 1, iteration: 26] Starting iteration...  source=console

running (0m30.9s), 0/1 VUs, 27 complete and 0 interrupted iterations
k6_workshop ✗ [====>---------------------------------] 1 VUs  30.9s/30s  027/200 shared iters
```

Kịch bản của chúng ta chỉ có thể hoàn thành 27 lần lặp trong khung thời gian 30 giây cho phép.

### Thay đổi tính đồng thời (concurrency)

Cho đến nay, bài kiểm tra của chúng ta chỉ sử dụng một người dùng ảo duy nhất, hoặc VU. Chúng ta có thể cập nhật tùy chọn `vus` để tăng số lượng yêu cầu được thực hiện đồng thời. Hãy thay đổi `vus` để mô phỏng 10 người dùng. Một lần nữa, chúng ta sẽ cập nhật phần `options` trong kịch bản kiểm thử:

```js
export const options = {
    scenarios: {
        k6_workshop: {
            executor: "shared-iterations",
            iterations: 200,
            maxDuration: "30s",
            vus: 10,
        },
    },
};
```

Chạy lại kịch bản một lần nữa:

```bash
k6 run test.js
```

Bài kiểm tra bây giờ sẽ thực hiện cùng một số lượng lần lặp, nhưng trong thời gian ngắn hơn nhiều nhờ vào số lượng các yêu cầu đồng thời.

```bash
INFO[0020] [VU: 7, iteration: 17] Starting iteration...  source=console
...
INFO[0020] [VU: 2, iteration: 17] Starting iteration...  source=console
...
INFO[0020] [VU: 6, iteration: 19] Starting iteration...  source=console
INFO[0020] [VU: 1, iteration: 18] Starting iteration...  source=console
INFO[0021] [VU: 3, iteration: 19] Starting iteration...  source=console
INFO[0021] [VU: 5, iteration: 20] Starting iteration...  source=console
INFO[0021] [VU: 4, iteration: 20] Starting iteration...  source=console
INFO[0021] [VU: 8, iteration: 20] Starting iteration...  source=console
INFO[0021] [VU: 9, iteration: 20] Starting iteration...  source=console
INFO[0021] [VU: 10, iteration: 20] Starting iteration...  source=console

running (0m22.6s), 00/10 VUs, 200 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 10 VUs  22.6s/30s  200/200 shared iters
```

Hãy xem kỹ hơn các kết quả này. Các kết quả trên (đã được cắt bớt cho gọn) hiển thị báo cáo cuối cùng cho mỗi VU đang chạy.

> :point_up: Bộ đếm lần lặp bắt đầu từ 0, nghĩa là số đếm 20 thực chất là 21 lần lặp.

Hành vi của executor `shared-iterations` có thể trở nên rõ ràng: _một số VUs thực hiện nhiều bài kiểm tra hơn những VUs khác_. Đây là khía cạnh "shared" (chia sẻ) trong tên của executor. Khi mỗi VU hoàn thành một lần lặp kiểm thử, chúng ngay lập tức đặt trước một lần lặp khác trong khi vẫn còn các lần lặp chưa thực hiện. Nếu một VU liên tục nhận được phản hồi nhanh từ dịch vụ đang được kiểm thử, có khả năng, như trong trường hợp này, VU đó sẽ nhận được nhiều hơn _phần công bằng_ của nó.

### Kết thúc

Với bài tập này, bạn đã thấy cách chạy một bài kiểm tra rất cơ bản và cách bạn có thể kiểm soát số lượng lần lặp, tính đồng thời, và thậm chí là thiết lập các giới hạn thời gian. Ngoài ra, bạn thấy rằng việc phân bổ các bài kiểm tra giữa các VUs có thể không _công bằng_.
