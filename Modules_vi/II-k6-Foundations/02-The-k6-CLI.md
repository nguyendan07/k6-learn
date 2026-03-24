# k6 CLI

Khi bạn đã có một kịch bản k6, bạn có thể sử dụng giao diện dòng lệnh k6 (k6 Command Line Interface - CLI) để tương tác với nó từ terminal. k6 CLI có thể thực thi các kịch bản kiểm thử k6 và cấu hình các thiết lập thực thi thông qua các lệnh con (sub-commands) và các cờ (flags).

Ba lệnh phổ biến nhất là:

| Lệnh      | Mô tả                            | Cách dùng        |
| --------- | -------------------------------- | ---------------- |
| `help`    | Hiển thị tất cả các lệnh khả thi | `k6 help`        |
| `run`     | Thực thi một kịch bản k6         | `k6 run test.js` |
| `version` | Hiển thị phiên bản k6 đã cài đặt | `k6 version`     |

_Flags_ (cờ) là các thiết lập được thêm vào lệnh để thay đổi một phần cấu hình. Dưới đây là tổng quan về các cờ phổ biến cho lệnh `run`:

| Cờ                       | Mô tả                                                  | Cách dùng                                |
| ------------------------ | ------------------------------------------------------ | ---------------------------------------- |
| `--help`                 | Hiển thị các cờ khả thi cho lệnh đã cho                | `k6 run --help`                          |
| `--vus` hoặc `-u`        | Thiết lập số lượng người dùng ảo (virtual users)       | `k6 run test.js --vus 10 --duration 30s` |
| `--duration`             | Thiết lập thời lượng của bài kiểm tra                  | `k6 run test.js --duration 10m`          |
| `--iterations` hoặc `-i` | Hướng dẫn k6 lặp lại hàm mặc định một số lần nhất định | `k6 run test.js -i 3`                    |
| `-e`                     | Thiết lập một biến môi trường để truyền vào kịch bản   | `k6 run test.js -e DOMAIN=test.k6.io`    |

Phần này sẽ xem xét các lệnh và cờ phổ biến này.

## Help

Thực thi `k6 help` sẽ hiển thị danh sách tất cả các lệnh có sẵn.

```shell
          /\      |‾‾| /‾‾/   /‾‾/
     /\  /  \     |  |/  /   /  /
    /  \/    \    |     (   /   ‾‾\
   /          \   |  |\  \ |  (‾)  |
  / __________ \  |__| \__\ \_____/ .io

Usage:
  k6 [command]

Available Commands:
  archive     Create an archive
  cloud       Run a test on the cloud
  convert     Convert a HAR file to a k6 script
  help        Help about any command
  inspect     Inspect a script or archive
  login       Authenticate with a service
  pause       Pause a running test
  resume      Resume a paused test
  run         Start a load test
  scale       Scale a running test
  stats       Show test metrics
  status      Show test status
  version     Show application version

Flags:
  -a, --address string      address for the api server (default "localhost:6565")
  -c, --config string       JSON config file (default "/Users/nic/Library/Application Support/loadimpact/k6/config.json")
  -h, --help                help for k6
      --log-output string   change the output for k6 logs, possible values are stderr,stdout,none,loki[=host:port] (default "stderr")
      --log-format string   log output format
      --no-color            disable colored output
  -q, --quiet               disable progress updates
  -v, --verbose             enable verbose logging

Use "k6 [command] --help" for more information about a command.
```

Để lấy thông tin về một lệnh cụ thể, hãy thêm cờ `--help`:

```shell
k6 run --help
```

## Thực thi và các tùy chọn thực thi: `run`

Một lệnh k6 phổ biến khác là `k6 run [filename].js`. Trong phần trước, bạn đã học cách sử dụng `k6 run` để thực thi một kịch bản k6 hiện có bằng JavaScript.

Nếu không có bất kỳ sửa đổi nào, `k6 run` sẽ hướng dẫn k6 chạy kịch bản của bạn như hiện tại (xem [Thay đổi các thiết lập trong k6](02-The-k6-CLI.md#Changing-settings-in-k6) để hiểu cách k6 xác định thiết lập cấu hình nào sẽ được sử dụng). Tuy nhiên, bạn cũng có thể sử dụng các cờ với lệnh `run` để ghi đè các thiết lập bên trong kịch bản (chẳng hạn như [k6 Load Test Options](06-k6-Load-Test-Options.md)) từ dòng lệnh.

### Cờ duration

Thời lượng (duration) chỉ định thời gian bài kiểm tra thực thi. Bạn có thể thiết lập điều này trên dòng lệnh với cờ `--duration`:

```shell
k6 run test.js --duration 30s
```

Bạn có thể sử dụng `s`, `h`, và `m` để xác định thời lượng. Sau đây là các đối số hợp lệ và tương đương cho cờ này:

- 1h30m10s
- 5410s
- 90m10s

### Cờ iterations

Bạn có thể thiết lập số lần lặp với cờ `--iterations` hoặc `-i`, như sau:

```shell
k6 run test.js --iterations 100
k6 run test.js -i 100
```

Trong cả hai dòng trên, k6 sẽ chạy 100 lần lặp của kịch bản.

### Các cờ virtual user

Bạn có thể điều chỉnh số lượng người dùng ảo với cờ `-u` hoặc `--vus` khi chạy bài kiểm tra:

```shell
k6 run test.js --vus 10 --duration 1m
k6 run test.js -u 10 --iterations 100
```

Hai dòng trên là tương đương, và cả hai đều hướng dẫn k6 thực thi tệp `test.js` với 10 người dùng ảo. Mỗi dòng cũng thiết lập một thời lượng kiểm thử và một số lần lặp.

### Các biến môi trường

Cho đến nay, bạn đã học cách thiết lập các tùy chọn thực thi trên dòng lệnh, thay đổi các tham số kiểm thử như người dùng ảo, thời lượng kiểm thử, số lần lặp và các giai đoạn (stages) trong một bài kiểm tra. Điều gì sẽ xảy ra nếu bạn muốn thiết lập các biến _khác_ trên dòng lệnh?

Trong trường hợp đó, bạn có thể sử dụng các biến môi trường (environment variables), các biến mà giá trị của chúng bạn có thể thiết lập bên ngoài kịch bản k6.

Ví dụ, bạn có thể sử dụng dòng lệnh để thiết lập một biến môi trường nhằm thay đổi tên miền (domain) mà kịch bản kiểm thử của bạn sử dụng. Điều này hữu ích khi bạn thường xuyên kiểm thử nhiều môi trường, chẳng hạn như staging và test.

Để sử dụng một biến môi trường, hãy định nghĩa biến đó trong kịch bản của bạn:

```js
import http from 'k6/http';

const hostname = `http://${__ENV.DOMAIN}`;

export default function () {
    let res = http.get(hostname + '/my_messages.php');
}
```

Trong kịch bản trên, `${__ENV.DOMAIN}` là một biến môi trường, nhưng nó không được định nghĩa ở bất kỳ đâu trong kịch bản. Đây là cách để định nghĩa nó trong khi chạy (runtime):

```shell
k6 run test.js -e DOMAIN=test.k6.io
```

Khi bài kiểm tra được thực thi, nó sẽ gửi một yêu cầu HTTP GET tới `http://test.k6.io/my_messages.php`. Bằng cách sử dụng các biến môi trường, bạn có thể thay đổi tên miền mà bài kiểm tra nhắm tới mà không cần thay đổi chính kịch bản đó.

Mặc dù có tên như vậy, các biến này có thể chứa nhiều loại thông tin, không chỉ thông tin về môi trường. Dưới đây là một số thứ khác bạn có thể sử dụng biến môi trường cho:

- think time (thời gian suy nghĩ)
- các tag bạn muốn gắn vào các yêu cầu cho lần chạy này
- tệp dữ liệu kiểm thử được sử dụng
- các trang cần loại trừ hoặc bao gồm
- kịch bản (scenario)

## Thay đổi các thiết lập trong k6

Trong phần này, bạn đã học cách sử dụng các cờ dòng lệnh và các biến môi trường để thay đổi cách kịch bản của bạn thực thi. Bạn cũng đã học cách thiết lập một số tùy chọn này bên trong chính kịch bản. Hãy kiểm tra tài liệu để biết [danh sách đầy đủ các tùy chọn](https://k6.io/docs/using-k6/k6-options/reference/).

Điều gì xảy ra nếu có sự xung đột giữa hai cách thay đổi thiết lập k6 này?

k6 luôn [ưu tiên các thiết lập](https://k6.io/docs/using-k6/k6-options/how-to/#order-of-precedence) theo thứ tự này:

1. Cờ dòng lệnh (Command-line flags)
2. Biến môi trường (Environment variables)
3. Các tùy chọn kịch bản k6 được xuất (Exported k6 script options)
4. Tệp cấu hình (Config file)
5. Mặc định (Defaults)

Các cờ dòng lệnh được ưu tiên cao nhất và luôn ghi đè mọi thứ khác. Hãy lập kế hoạch thực thi kịch bản của bạn cho phù hợp.

## Kiểm tra kiến thức của bạn

### Câu hỏi 1

Lệnh nào sau đây chạy một kịch bản k6?

A: `k6 --vus 1 test.js`

B: `k6 run test.js`

C: `run test.js -vus 1`

### Câu hỏi 2

Lệnh nào sau đây sẽ tạo ra cảnh báo khi thực thi?

A: `k6 run test.js --duration 10m`

B: `k6 run test.js -i 3`

C: `k6 run test.js --users 3`

### Câu hỏi 3

Phát biểu nào sau đây về việc sử dụng biến môi trường là đúng?

A: Để sử dụng một biến môi trường, kịch bản cần được cập nhật _và_ biến môi trường phải được thiết lập trong dòng lệnh.

B: Để sử dụng một biến môi trường, chỉ cần cập nhật kịch bản để định nghĩa giá trị của biến.

C: Để sử dụng một biến môi trường, chỉ cần sử dụng cờ dòng lệnh, và giá trị sẽ tự động được truyền vào kịch bản.

### Đáp án

1. B. Tùy chọn đầu tiên thiếu từ khóa `run`, và tùy chọn thứ ba thiếu `k6`. B là tùy chọn duy nhất sẽ thực sự chạy kịch bản.
2. C. `--users` không phải là một tùy chọn hợp lệ và sẽ tạo ra lỗi `invalid argument` trên CLI.
3. A. Giá trị của một biến môi trường sẽ không được kịch bản k6 tiếp nhận trừ khi kịch bản được cập nhật để chấp nhận nó bên cạnh việc giá trị đó được truyền qua dòng lệnh.
