# Các tùy chọn kiểm thử tải của k6 (k6 Load Test Options)

Cho đến nay, bạn đã chạy cùng một kịch bản với một VU duy nhất và một lần lặp duy nhất. Trong phần này, bạn sẽ học cách mở rộng quy mô đó và chạy một bài kiểm thử tải đầy đủ đối với ứng dụng của mình.

Các tùy chọn kiểm thử (test options) là các giá trị cấu hình ảnh hưởng đến cách kịch bản kiểm thử của bạn được thực thi, chẳng hạn như số lượng VUs hoặc iterations, thời lượng của bài kiểm tra, và nhiều hơn nữa. Đôi khi chúng còn được gọi là "các tham số kiểm thử".

k6 đi kèm với một số tùy chọn kiểm thử mặc định, nhưng có bốn cách khác nhau để thay đổi các tham số kiểm thử cho một kịch bản:

1. Bạn có thể bao gồm các cờ dòng lệnh (command-line flags) khi chạy một kịch bản k6 (chẳng hạn như `k6 run --vus 10 --iterations 30`).
2. Bạn có thể xác định [các biến môi trường](https://k6.io/docs/using-k6/environment-variables/) trên dòng lệnh để truyền vào kịch bản.
3. Bạn có thể xác định chúng ngay bên trong chính kịch bản kiểm thử.
4. Bạn có thể bao gồm một tệp cấu hình.

Hiện tại, bạn sẽ học cách thực hiện tùy chọn thứ ba: xác định các tham số kiểm thử bên trong chính kịch bản. Các ưu điểm của cách tiếp cận này là:

- Sự đơn giản: không yêu cầu thêm tệp hoặc lệnh bổ sung.
- Khả năng lặp lại: Việc thêm các tham số này vào kịch bản giúp đồng nghiệp dễ dàng chạy các bài kiểm tra mà bạn đã viết.
- Khả năng quản lý phiên bản: Các thay đổi đối với các tham số kiểm thử có thể được theo dõi cùng với mã kiểm thử.

Để sử dụng các tùy chọn kiểm thử bên trong một kịch bản, hãy thêm các dòng sau vào kịch bản của bạn. Theo quy ước, tốt nhất là thêm nó sau các câu lệnh import và trước hàm mặc định, để các tùy chọn dễ dàng được đọc khi mở kịch bản:

```js
export let options = {
  vus: 10,
  iterations: 40,
};
```

Nếu bạn thiết lập nhiều tùy chọn, hãy đảm bảo bạn kết thúc mỗi tùy chọn bằng dấu `,`.

## VUs

```js
vus: 10,
```

Trong dòng này, bạn có thể thay đổi số lượng người dùng ảo (virtual users) mà k6 sẽ chạy.

Lưu ý rằng nếu bạn chỉ xác định VUs mà không có tùy chọn kiểm thử nào khác, bạn có thể nhận được lỗi sau:

```plain
          /\      |‾‾| /‾‾/   /‾‾/   
     /\  /  \     |  |/  /   /  /    
    /  \/    \    |     (   /   ‾‾\  
   /          \   |  |\  \ |  (‾)  | 
  / __________ \  |__| \__\ \_____/ .io

WARN[0000] the `vus=10` option will be ignored, it only works in conjunction with `iterations`, `duration`, or `stages`
  execution: local
     script: test.js
     output: -
```

Nếu bạn thiết lập số lượng VUs, bạn cần xác định thêm thời gian những người dùng đó sẽ được thực thi, bằng cách sử dụng một trong các tùy chọn sau:

- iterations
- durations
- stages

## Iterations

```js
  vus: 10,
  iterations: 40,
```

Thiết lập số lần lặp (iterations) trong các tùy chọn kiểm thử sẽ xác định nó cho _tất cả_ người dùng. Trong ví dụ trên, bài kiểm tra sẽ chạy tổng cộng 40 lần lặp, với mỗi người dùng trong số 10 người dùng thực thi kịch bản đúng 4 lần.

## Duration

```js
  vus: 10,
  duration: '2m'
```

Thiết lập thời lượng (duration) hướng dẫn k6 lặp lại kịch bản cho mỗi số lượng người dùng đã chỉ định cho đến khi đạt đến thời lượng đó.

Thời lượng có thể được thiết lập bằng cách sử dụng `h` cho giờ, `m` cho phút và `s` cho giây, như các ví dụ sau:

- `duration: '1h30m'`
- `duration: '30s'`
- `duration: '5m30s'`

Nếu bạn thiết lập thời lượng nhưng không chỉ định số lượng VUs, k6 sẽ sử dụng số lượng VU mặc định là 1.

Nếu bạn thiết lập thời lượng kết hợp với việc thiết lập số lần lặp, giá trị nào kết thúc sớm hơn sẽ được sử dụng. Ví dụ, với các tùy chọn sau:

```js
  vus: 10,
  duration: '5m',
  iterations: 40,
```

k6 sẽ thực hiện bài kiểm tra trong 40 lần lặp hoặc 5 phút, _tùy theo điều kiện nào kết thúc trước_. Nếu mất 1 phút để hoàn thành tổng cộng 40 lần lặp, bài kiểm tra sẽ kết thúc sau 1 phút. Nếu mất 10 phút để hoàn thành tổng cộng 40 lần lặp, bài kiểm tra sẽ kết thúc sau 5 phút.

### Stages

Xác định iterations và durations đều khiến k6 thực thi kịch bản kiểm thử của bạn bằng một [hồ sơ tải đơn giản (simple load profile)](../XX-Future-Ideas/Parameters-of-a-load-test.md#Simple-load-profile): VUs được khởi động, duy trì trong một thời gian hoặc số lần lặp nhất định, và sau đó kết thúc.

![A simple load profile](../../images/load_profile-no_ramp-up_or_ramp-down.png)

_Hồ sơ tải đơn giản_

Điều gì sẽ xảy ra nếu bạn muốn thêm một giai đoạn [ramp-up hoặc ramp-down](../XX-Future-Ideas/Parameters-of-a-load-test.md#ramp-up-and-ramp-down-periods), để hồ sơ trông giống như thế này hơn?

![Constant load profile, with ramps](../../images/load_profile-constant.png.png)

_Hồ sơ tải không đổi, có ramp_

Trong trường hợp đó, bạn có thể muốn sử dụng [stages](https://k6.io/docs/using-k6/options/#stages).

```js
export let options = {
  stages: [
    { duration: '30m', target: 100 },
    { duration: '1h', target: 100 },
    { duration: '5m', target: 0 },
  ],
};
```

Tùy chọn stages cho phép bạn xác định các bước hoặc giai đoạn khác nhau cho bài kiểm thử tải của mình, mỗi giai đoạn có thể được cấu hình với số lượng VUs và thời lượng. Ví dụ trên bao gồm ba bước (nhưng bạn có thể thêm nhiều hơn nếu muốn).

1. Bước đầu tiên là tăng dần (ramp-up) từ 0 VUs lên 100 VUs.
2. Bước thứ hai xác định [trạng thái ổn định (steady state)](../XX-Future-Ideas/Parameters-of-a-load-test.md#Steady-state). Tải được giữ không đổi ở mức 100 VUs trong 1 giờ.
3. Sau đó, bước thứ ba là giảm dần (ramp-down) từ 100 VUs trở về 0, tại thời điểm đó bài kiểm tra kết thúc.

Stages là cách linh hoạt nhất để xác định các tham số kiểm thử cho một kịch bản duy nhất. Chúng mang lại cho bạn sự linh hoạt trong việc định hình tải của bài kiểm tra để phù hợp với tình huống trong môi trường production mà bạn đang cố gắng mô phỏng.

## Kịch bản đầy đủ cho đến nay

Nếu bạn đang sử dụng stages, đây là giao diện kịch bản của bạn cho đến nay:

```js
import http from 'k6/http';
import { check, sleep } from 'k6';

export let options = {
  stages: [
    { duration: '30m', target: 100 },
    { duration: '1h', target: 100 },
    { duration: '5m', target: 0 },
  ],
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

## Kiểm tra kiến thức của bạn

### Câu hỏi 1

Bạn đã được hướng dẫn tạo một kịch bản gửi cùng một yêu cầu HTTP đúng 100 lần. Tùy chọn kiểm thử nào sau đây là cách tốt nhất để hoàn thành nhiệm vụ này?

A: Iterations

B: Stages

C: Duration

### Câu hỏi 2

Với các tùy chọn kiểm thử như được chỉ định bên dưới, bài kiểm tra sẽ được thực thi trong bao lâu?

```js
export let options = {
  vus: 10,
  iterations: 3,
  duration: '1h',
};
```

A: 10 giờ

B: Cho đến khi hoàn thành 3 lần lặp hoặc 1 giờ, tùy theo điều kiện nào ngắn hơn

C: 1 giờ cộng với thời gian cần thiết để hoàn thành 3 lần lặp

### Câu hỏi 3

Tùy chọn kiểm thử nào sau đây sẽ tạo ra một mô hình tải theo bậc thang để thêm 100 người dùng trong vòng 10 phút, giữ mức tải đó ổn định trong 30 phút, và sau đó tiếp tục mô hình đó cho đến khi có 300 VUs chạy trong 30 phút?

A:

```js
export let options = {
  stages: [
    { duration: '10m', target: 100 },
    { duration: '30m', target: 100 },
    { duration: '10m', target: 200 },
    { duration: '30m', target: 200 },
    { duration: '10m', target: 300 },
    { duration: '30m', target: 300 },	  
  ],
};
```

B:

```js
export let options = {
  stages: [
    { duration: '30m', target: 300 },
};
```

C:

```js
export let options = {
  vus: 300,
  duration: '30m',
};
```

### Đáp án

1. A. Thiết lập số lần lặp thành 100 sẽ là cách tốt nhất để hoàn thành nhiệm vụ này. Các stages cho [một số executors](https://k6.io/docs/using-k6/scenarios/executors/ramping-arrival-rate) cũng cho phép bạn làm điều này, nhưng không phải cho tất cả chúng. Duration chỉ thay đổi thời gian bài kiểm tra sẽ chạy trong bao lâu, không cụ thể là nó lặp lại bao nhiêu lần.
2. B. Trong trường hợp các tham số mâu thuẫn nhau, k6 sẽ chạy trong khoảng thời gian ngắn hơn. Trong trường hợp này, 3 lần lặp có thể sẽ mất ít thời gian hơn 1 giờ, vì vậy bài kiểm tra sẽ kết thúc trong thời gian ít hơn một giờ.
3. A. B và C sai vì cả hai đều mô tả một bài kiểm tra sẽ tăng dần và đều đặn người dùng ảo cho đến khi có 300 người dùng đang chạy. Nghĩa là, tốc độ tăng là ổn định. A là câu trả lời đúng duy nhất vì nó là câu duy nhất bao gồm một trạng thái ổn định (một khoảng thời gian không tăng VU) giữa các giai đoạn ramp-up.
