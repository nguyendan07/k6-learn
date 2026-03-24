# Mô hình hóa khối lượng công việc với các kịch bản (scenarios)

Trong phần [Mô hình hóa khối lượng công việc (Workload modeling)](03-Workload-modeling.md), chúng ta đã đề cập rằng một trong những thách thức là các luồng người dùng (user flows) có thể rất phức tạp. Một bài kiểm tra đơn lẻ có thể cần bao quát nhiều luồng đầu-cuối (end-to-end flows) cùng một lúc. Ví dụ, trong một ứng dụng cửa hàng trực tuyến, chúng ta có thể có một số thành viên đang duyệt sản phẩm trong khi nhiều người khác (hy vọng thế!) đang thêm hàng vào giỏ. Để giúp tổ chức các luồng đa dạng này, đội ngũ k6 đã tạo ra các [kịch bản (scenarios)](https://k6.io/docs/using-k6/scenarios/).

Với _scenarios_, tác giả kịch bản có thể tạo ra các luồng riêng biệt có các mô hình thực thi và thiết lập môi trường khác nhau. Các scenarios có thể được thực thi song song hoặc so le bằng cách sử dụng khoảng thời gian bù với tùy chọn `startTime`.

Bằng cách chia nhỏ từng luồng của bạn thành một scenario khác nhau, kịch bản tổng thể của bạn có thể tinh gọn hơn với logic dễ theo dõi hơn. Từ ví dụ cửa hàng trực tuyến, một scenario sẽ là người dùng thêm hàng vào giỏ và đặt hàng. Một scenario khác sẽ đại diện cho những khách hàng khác có thể đang tìm kiếm sự sẵn có của một số sản phẩm nhất định. Mỗi quy trình công việc có thể phức tạp; việc giữ chúng tách biệt giúp bảo trì kịch bản dễ dàng hơn.

## Định nghĩa các scenarios

Mỗi _scenario_ bao gồm các [yếu tố của một mô hình khối lượng công việc](03-Workload-modeling.md#Elements-of-a-workload-model) tương tự, ngoài ra còn có `executor`, `startTime`, và tùy chọn là các biến `env` cùng các `tags` dành riêng cho _scenario_ đó. Kiểm tra danh sách đầy đủ các tùy chọn scenario trong [tài liệu](https://k6.io/docs/using-k6/scenarios/).

Lưu kịch bản sau đây dưới tên `playground.js`.

```javascript
import { sleep } from "k6";

export const options = {
    scenarios: {
        // Scenario 1: Users browsing through our product catalog
        users_browsing_products: {
            executor: "shared-iterations", // name of the executor to use
            env: { WORKFLOW: "browsing" }, // environment variable for the workflow

            // executor-specific configuration
            vus: 3,
            iterations: 10,
            maxDuration: "10s",
        },
        // Scenario 2: Users adding items to their cart and checking out
        users_buying_products: {
            executor: "per-vu-iterations", // name of the executor to use
            exec: "usersBuyingProducts", // js function for the purchasing workflow
            env: { WORKFLOW: "buying" }, // environment variable for the workflow
            startTime: "2s", // delay start of purchasing workflow

            // executor-specific configuration
            vus: 3,
            iterations: 1,
            maxDuration: "10s",
        },
    },
};

export default function () {
    // Model your workflow for searching through products
    console.log(
        `[VU: ${__VU}, iteration: ${__ITER}, workflow: ${__ENV.WORKFLOW}] Just looking!!!`,
    );
    sleep(1);
}

export function usersBuyingProducts() {
    // Model your workflow for adding items to cart and checking out
    console.log(
        `[VU: ${__VU}, iteration: ${__ITER}, workflow: ${__ENV.WORKFLOW}] CHA-CHING!!!`,
    );
    sleep(1);
}
```

Chạy kịch bản bằng lệnh `k6 run playground.js` sẽ tạo ra kết quả tương tự như sau:

```plain
$ k6 run playground.js

          /\      |‾‾| /‾‾/   /‾‾/
     /\  /  \     |  |/  /   /  /
    /  \/    \    |     (   /   ‾‾\
   /          \   |  |\  \ |  (‾)  |
  / __________ \  |__| \__\ \_____/ .io

  execution: local
     script: playground.js
     output: -

  scenarios: (100.00%) 2 scenarios, 6 max VUs, 42s max duration (incl. graceful stop):
           * users_browsing_products: 10 iterations shared among 3 VUs (maxDuration: 10s, gracefulStop: 30s)
           * users_buying_products: 1 iterations for each of 3 VUs (maxDuration: 10s, exec: usersBuyingProducts, startTime: 2s, gracefulStop: 30s)

INFO[0000] [VU: 4, iteration: 0, workflow: browsing] Just looking!!!  source=console
INFO[0000] [VU: 1, iteration: 0, workflow: browsing] Just looking!!!  source=console
...
INFO[0002] [VU: 1, iteration: 2, workflow: browsing] Just looking!!!  source=console
INFO[0002] [VU: 5, iteration: 0, workflow: buying] CHA-CHING!!!  source=console
INFO[0003] [VU: 4, iteration: 3, workflow: browsing] Just looking!!!  source=console

running (04.0s), 0/6 VUs, 13 complete and 0 interrupted iterations
users_browsing_products ✓ [======================================] 3 VUs  04.0s/10s  10/10 shared iters
users_buying_products   ✓ [======================================] 3 VUs  01.0s/10s  3/3 iters, 1 per VU
```

Xem kết quả, bạn sẽ nhận thấy rằng người dùng đang duyệt sản phẩm, sau đó sau một khoảng trì hoãn ban đầu, chúng ta có một người dùng khác đang thực hiện mua hàng.

## Các tùy chọn cấu hình

Ví dụ của chúng ta đã sử dụng một số lượng tối thiểu các tùy chọn cấu hình cho các _scenarios_, nhưng các tùy chọn có sẵn bao gồm những mục sau:

> Hãy nhớ rằng các tùy chọn bổ sung có thể được yêu cầu tùy thuộc vào `executor` được chỉ định. Xem [thiết lập hồ sơ tải với executors](08-Setting-load-profiles-with-executors.md) để biết thêm về _executors_.

| Tùy chọn       | Mô tả                                                                                                    | Mặc định     |
| -------------- | -------------------------------------------------------------------------------------------------------- | ------------ |
| **`executor`** | [executor](https://k6.io/docs/using-k6/scenarios/executors/) cụ thể để "định hình" các mô hình lưu lượng | - (bắt buộc) |
| `env`          | Các biến môi trường dành riêng cho scenario                                                              | `{}`         |
| `exec`         | Tên của hàm JS được xuất để thực thi cho scenario                                                        | `"default"`  |
| `gracefulStop` | Khoảng thời gian cho phép một lần lặp hoàn thành trước khi bị buộc chấm dứt                              | `"30s"`      |
| `startTime`    | Khoảng thời gian bù trong bài kiểm tra để bắt đầu scenario                                               | `"0s"`       |
| `tags`         | Các thẻ dành riêng cho scenario                                                                          | `{}`         |

Xem xét kỹ hơn ví dụ của chúng ta, chúng ta đã tạo ra hai scenarios: `users_browsing_products` và `users_buying_products`.

Ý tưởng cho scenario `users_browsing_products` là tạo ra một số _hoạt động nền_ cho bài kiểm tra của chúng ta. Chúng ta sử dụng executor `shared-iterations` để mô phỏng ba người dùng đang duyệt danh mục sản phẩm. Chúng ta đang giới hạn hoạt động này ở 10 lần lặp và đảm bảo kịch bản của chúng ta kéo dài không quá 10 giây. Tùy chọn `env` thiết lập biến môi trường `WORKFLOW` thành `browsing`, biến này có thể được sử dụng trong mã JavaScript để cung cấp một số ngữ cảnh thực thi. Bởi vì chúng ta _không_ chỉ định tùy chọn `exec`, scenario của chúng ta sẽ thực thi bất kỳ hàm JavaScript nào là `default`.

`users_buying_products` là scenario thứ hai của chúng ta để mô phỏng ba người dùng khác nhau đang thực hiện mua hàng. Scenario này sử dụng `per-vu-iterations` để kiểm soát hoạt động. Scenario chỉ định `exec: 'usersBuyingProducts'` để gọi logic mua hàng của chúng ta cho mỗi lần lặp scenario. Giống như scenario trước, chúng ta đưa vào biến môi trường `WORKFLOW` để cung cấp cho kịch bản một số ngữ cảnh cho scenario. Tuy nhiên, lần này chúng ta chỉ định `startTime: '2s'` để xác định một khoảng trì hoãn cho scenario bắt đầu; điều này giúp `users_browsing_products` có một sự "khởi đầu trước" để tạo ra hoạt động nền trước khi chúng ta bắt đầu có người dùng thực hiện mua hàng.

## Kiểm tra kiến thức của bạn

### Câu hỏi 1

_Đúng_ hay _Sai_, bạn có thể coi các scenarios như các lớp hoạt động trong một bài kiểm tra mở rộng hơn?

### Câu hỏi 2

_Đúng_ hay _Sai_, bạn phải chỉ định tùy chọn `exec` cho mỗi scenario?

### Câu hỏi 3

_Đúng_ hay _Sai_, tùy chọn `exec` quyết định _hình dạng_ lưu lượng (traffic shape) cho một scenario?

### Đáp án

1. Đúng. Mỗi scenario hoạt động độc lập nhưng có thể bị ảnh hưởng bởi các scenarios khác. Từ ví dụ của chúng ta, chúng ta đã định nghĩa hoạt động duyệt web của người dùng ở _nền_, sau đó có người dùng thực hiện mua hàng ở _phía trên_.
2. Sai. Mặc dù việc cấu hình rõ ràng tùy chọn `exec` là điều nên làm, nhưng nó _không_ bắt buộc. Nhiều scenarios được cấu hình mà không có tùy chọn `exec` sẽ dẫn đến việc mỗi cái đều thực thi cùng một hàm `default`, điều này có thể gây nhầm lẫn.
3. Sai. `executor` cuối cùng mới là thứ định nghĩa mô hình của hồ sơ tải (load profile) cho scenario của bạn. `exec` biểu thị hàm JavaScript nào sẽ thực thi cho mỗi lần lặp của scenario.
