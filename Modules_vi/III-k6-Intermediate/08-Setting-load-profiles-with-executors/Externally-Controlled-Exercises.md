# Externally Controlled Executor

Như đã lưu ý trong [Thiết lập Load Profiles với Executors](../08-Setting-load-profiles-with-executors.md#Externally-Controlled), executor cụ thể này ủy thác quyền kiểm soát VUs và trạng thái chạy của các bài kiểm tra cho các tiến trình bên ngoài. Bạn có thể thoải mái sử dụng Bash, Python, hoặc một số thành phần tự động hóa; nguồn gốc của các tiến trình này không ảnh hưởng đến executor.

Trọng tâm của executor sẽ là thiết lập kịch bản kiểm thử và đưa ra các ràng buộc về tổng thời lượng và số lượng người dùng ảo cho phép. Từ thời điểm này, một bài kiểm tra đang chạy sẽ ở trạng thái gần như là _chế độ chờ_ để đợi các hướng dẫn tiếp theo.

Tương tác với bài kiểm tra đang chạy sử dụng [REST APIs](https://k6.io/docs/misc/k6-rest-api/) được phơi bày bởi tiến trình k6, hoặc bằng cách sử dụng giao diện dòng lệnh k6 ([CLI](https://k6.io/blog/how-to-control-a-live-k6-test/)).

## Bài tập

Đối với các bài tập của chúng ta, chúng ta sẽ bắt đầu bằng cách sử dụng một kịch bản rất cơ bản, đơn giản là thực hiện một yêu cầu HTTP và sau đó đợi ba giây trước khi hoàn thành lần lặp kiểm thử. Chúng tôi sẽ cung cấp một số đầu ra console khi mọi thứ thay đổi.

Phần `options` được cấu hình sẽ là mức tối thiểu tuyệt đối cần thiết cho executor `externally-controlled`.

### Tạo kịch bản kiểm thử của chúng ta

Tạo một tệp tên là _test.js_ với nội dung sau:

```js
import http from "k6/http";
import { sleep } from "k6";

export const options = {
    scenarios: {
        k6_workshop: {
            executor: "externally-controlled",
            duration: "5m",
        },
    },
};

export default function () {
    console.log(`[VU: ${__VU}, iteration: ${__ITER}] Starting iteration...`);
    http.get("https://test.k6.io/contacts.php");
    sleep(3);
}
```

### Chạy bài kiểm tra ban đầu

Mở một cửa sổ _terminal_ trong cùng thư mục với kịch bản `test.js`. Bây giờ chúng ta sẽ thực thi kịch bản của mình bằng tệp thực thi k6:

```bash
k6 run test.js
```

k6 bây giờ sẽ bắt đầu. Bạn sẽ thấy một bộ đếm thời gian đang chạy, nhưng không có nhiều việc xảy ra. Chúng ta không thấy thông báo console mong đợi có trong kịch bản của mình! Chúng ta sẽ ở trong trạng thái chờ này cho đến khi bộ đếm thời gian đạt đến `duration` đã cấu hình từ các tùy chọn kịch bản.

### Mở rộng quy mô (Scaling up) VUs

Thực tế không có bài kiểm tra nào được thực hiện do có `0` người dùng ảo (VUs) được bắt đầu bởi kịch bản. k6 đợi tiến trình bên ngoài _scale up_ VUs. Để scale up, chúng ta sẽ sử dụng dòng lệnh 'k6'.

Hãy mở một cửa sổ _terminal_ khác. Tuy nhiên, lần này, thư mục không quan trọng. Nếu kịch bản của bạn đã hết thời gian chờ (timed out) trong lúc này, hãy khởi động lại nó bằng `k6 run test.js`.

```bash
k6 scale --vus 2 --max 10
```

> :point_up: Nếu bạn không chỉ định `maxVUs` trong các tùy chọn kịch bản của mình, bất kỳ yêu cầu scale up nào cũng sẽ thất bại trừ khi bạn cung cấp một `--max` cùng với yêu cầu scale!

Bây giờ VUs đã được scale up, bạn sẽ thấy đầu ra console cho thấy mỗi người dùng ảo hiện đang thực thi mã kiểm thử.

### Kiểm soát công cụ thực thi

Tiếp tục với bài kiểm tra hiện đang chạy từ bước trước, hãy thử nghiệm với các tùy chọn khác có sẵn để kiểm soát toàn bộ việc thực thi kiểm thử.

Sử dụng một cửa sổ _terminal_ đang rảnh (cửa sổ không chạy bài kiểm tra k6), chúng ta sẽ đưa ra các lệnh sau:

```bash
# Tạm dừng bài kiểm tra đang chạy; đầu ra console sẽ dừng lại
k6 pause

# Tiến hành scale up số lượng VUs mong muốn
k6 scale --vus 5

# Tiếp tục bài kiểm tra; đầu ra sẽ xác nhận bài kiểm tra đang chạy với các VUs bổ sung
k6 resume
```

### Kiểm tra trạng thái

Tại bất kỳ thời điểm nào, bạn có thể sử dụng lệnh `k6` để hỏi về trạng thái của một instance đang chạy:

```bash
$ k6 status
status: 7
paused: "false"
vus: "5"
vus-max: "10"
stopped: false
running: true
tainted: false
```

### Thu thập ảnh chụp nhanh các chỉ số (metrics snapshot)

Trong khi kịch bản của bạn đang chạy, dù đang tạm dừng hay đang hoạt động, bạn có thể thăm dò các chỉ số hiện tại từ kịch bản đang chạy. Điều này sẽ đổ các chỉ số hiện tại ở định dạng YAML ra console của bạn, và có thể được _điều hướng (piped)_ đến một tệp đầu ra.

```bash
$ k6 stats
...
- name: http_req_duration
  type:
      type: trend
      valid: true
  contains:
      type: time
      valid: true
  tainted: ""
  sample:
      avg: 60.06574999999998
      max: 71.202
      med: 59.515
      min: 55.203
      p(90): 63.559000000000005
      p(95): 65.21095
...
```

### Kết thúc bài kiểm tra của bạn

Nếu bạn muốn hoàn thành một bài kiểm tra trước khi đạt đến khung thời gian `duration`, bạn sẽ phải sử dụng lệnh bàn phím `Ctrl+C` trong _terminal_ đang chạy bài kiểm tra của bạn hoặc sử dụng REST API:

```bash
curl -X PATCH \
  http://localhost:6565/v1/status \
  -H 'Content-Type: application/json' \
  -d '{
    "data": {
        "attributes": {
            "stopped": true
        },
        "id": "default",
        "type": "status"
    }
}'
```

> CLI `k6` không cung cấp một lệnh `k6 stop` tương đương. Đơn giản chỉ cần sử dụng `Ctrl+C` để kết thúc một bài kiểm tra.

### Các tùy chọn kịch bản

Kịch bản ban đầu của chúng ta cung cấp mức tối thiểu để bắt đầu bài kiểm tra của bạn.

Hãy cân nhắc việc chỉ định `maxVUs` của bạn. Như đã lưu ý trước đó, `maxVUs` không bắt buộc. Tuy nhiên, bất kỳ nỗ lực scale nào cũng sẽ gặp lỗi trừ khi một `--max` được chỉ định cùng với yêu cầu scale ban đầu. Giá trị `maxVUs` có thể được ghi đè bằng cách sử dụng đối số `--max` nếu thấy cần thiết.

Tùy chọn kịch bản cuối cùng là thiết lập `vus`. Khi được cung cấp, số lượng VUs này sẽ bắt đầu xử lý ngay khi kịch bản được khởi động, do đó loại bỏ nhu cầu scale-up ban đầu.

Hãy cập nhật khai báo _options_ trong kịch bản _test.js_ của chúng ta để bao gồm các thiết lập bổ sung này:

```js
export const options = {
    scenarios: {
        k6_workshop: {
            executor: "externally-controlled",
            duration: "5m",
            maxVUs: 10,
            vus: 2,
        },
    },
};
```

Chạy kịch bản của chúng ta bây giờ sẽ ngay lập tức cho thấy rằng các bài kiểm tra của chúng ta đang được thực thi bởi 2 người dùng ảo:

```bash
$ k6 run test.js

          /\      |‾‾| /‾‾/   /‾‾/
     /\  /  \     |  |/  /   /  /
    /  \/    \    |     (   /   ‾‾\
   /          \   |  |\  \ |  (‾)  |
  / __________ \  |__| \__\ \_____/ .io

  execution: local
     script: scripts/test.js
     output: -

  scenarios: (100.00%) 1 scenario, 10 max VUs, 5m0s max duration (incl. graceful stop):
           * k6_workshop: Externally controlled execution with 2 VUs, 10 max VUs, 5m0s duration

INFO[0000] [VU: 4, iteration: 0] Starting iteration...   source=console
INFO[0000] [VU: 1, iteration: 0] Starting iteration...   source=console
INFO[0003] [VU: 1, iteration: 1] Starting iteration...   source=console
INFO[0003] [VU: 4, iteration: 1] Starting iteration...   source=console
INFO[0006] [VU: 1, iteration: 2] Starting iteration...   source=console
INFO[0006] [VU: 4, iteration: 2] Starting iteration...   source=console
```

### Kết thúc

Vậy là xong! Hy vọng rằng bạn thấy được sức mạnh bổ sung trong việc có thể kiểm soát bài kiểm thử tải k6 của mình từ một nguồn bên ngoài!
