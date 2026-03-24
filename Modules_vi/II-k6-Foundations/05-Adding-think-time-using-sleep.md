# Thêm think time bằng cách sử dụng sleep

Trước khi bạn tăng tốc các bài kiểm thử tải (load tests) của mình, còn một điều nữa cần thêm vào: think time.

_Think time_ (thời gian suy nghĩ) là khoảng thời gian mà một kịch bản tạm dừng trong quá trình thực thi kiểm thử để mô phỏng sự chậm trễ mà người dùng thực tế gặp phải trong quá trình sử dụng ứng dụng.

### Khi nào bạn nên sử dụng think time?

Nhìn chung, việc sử dụng think time để mô phỏng chính xác hành vi của người dùng cuối giúp kịch bản kiểm thử tải trở nên thực tế hơn. Nếu sự thực tế giúp bạn đạt được các mục tiêu kiểm thử của mình, việc sử dụng think time có thể hỗ trợ điều đó.

Bạn nên xem xét việc thêm think time trong các tình huống sau:

- Bài kiểm tra của bạn tuân theo một luồng người dùng (user flow), chẳng hạn như truy cập các phần khác nhau của ứng dụng theo một thứ tự nhất định
- Bạn muốn mô phỏng các hành động tốn một khoảng thời gian để thực hiện, chẳng hạn như đọc văn bản trên một trang hoặc điền vào một biểu mẫu
- Trình tạo tải (load generator), hoặc máy mà bạn đang chạy k6, hiển thị mức sử dụng CPU cao (> 80%) trong quá trình thực thi kiểm thử.

Mối nguy hiểm chính khi loại bỏ hoặc giảm think time là nó làm tăng tốc độ gửi các yêu cầu, điều này có thể làm tăng mức sử dụng CPU. Khi mức sử dụng CPU quá cao, chính trình tạo tải sẽ gặp khó khăn trong việc _gửi_ các yêu cầu, điều này có thể dẫn đến kết quả không chính xác chẳng hạn như kết quả âm tính giả (false negatives). Thêm think time là một cách để [giảm mức sử dụng CPU cao của trình tạo tải](https://k6.io/docs/cloud/analyzing-results/performance-insights/#high-load-generator-cpu-usage).

### Khi nào bạn không nên sử dụng think time?

Sử dụng think time làm giảm tốc độ yêu cầu tối đa trên mỗi VU mà bạn có thể đạt được trong bài kiểm tra của mình. Nó làm chậm tốc độ gửi các yêu cầu.

Think time là không cần thiết trong các tình huống sau:

- Bạn muốn thực hiện một bài [stress test](https://k6.io/docs/test-types/stress-testing/) để tìm hiểu xem ứng dụng của bạn có thể xử lý bao nhiêu yêu cầu trên giây
- Endpoint của API mà bạn đang kiểm thử trải qua một lượng lớn yêu cầu trên giây trong thực tế mà không có sự chậm trễ
- Trình tạo tải của bạn có thể chạy kịch bản kiểm thử mà không vượt quá mốc sử dụng CPU 80%.

Điểm cuối cùng là một yêu cầu để đảm bảo rằng think time không làm tăng throughput của bài kiểm tra đến mức ảnh hưởng đến sức khỏe của trình tạo tải, như đã thảo luận trong phần trước.

Như bạn có thể thấy, câu hỏi về việc có nên sử dụng think time hay không phụ thuộc vào mục tiêu kiểm thử của bạn. Khi nghi ngờ, hãy sử dụng think time.

## Sleep

Trong k6, bạn có thể thêm think time bằng cách sử dụng [`sleep()`](https://k6.io/docs/javascript-api/k6/sleep-t/). Để sử dụng nó, bạn sẽ cần nhập sleep và sau đó thêm sleep vào phần kịch bản bên trong hàm mặc định mà bạn muốn việc thực thi kiểm thử tạm dừng:

```js
import http from 'k6/http';
import { check, sleep } from 'k6';

export default function() {
  let url = 'https://httpbin.test.k6.io/post';
  let response = http.post(url, 'Hello world!');
  check(response, {
      'Application says hello': (r) => r.body.includes('Hello world!')
  });

  sleep(1);
}
```

`sleep(1);` có nghĩa là kịch bản sẽ tạm dừng trong 1 giây khi nó được thực thi.

Việc bao gồm sleep không ảnh hưởng đến thời gian phản hồi (`http_req_duration`); thời gian phản hồi luôn được báo cáo sau khi đã loại bỏ sleep. Tuy nhiên, sleep _được_ bao gồm trong [thời lượng lần lặp (iteration duration)](03-Understanding-k6-results.md#Iteration-duration).

### Think time động (Dynamic think time)

Vấn đề của việc mã hóa cứng (hard-coding) một khoảng thời gian trì hoãn vào kịch bản của bạn là nó đưa vào một mô hình nhân tạo cho bài kiểm tra, điều này sau đó có thể khiến tải lên ứng dụng của bạn dễ dự đoán hơn so với thực tế trong môi trường production.

**Thực hành kiểm thử tốt nhất:** Sử dụng think time động.

Think time động thực tế hơn và mô phỏng người dùng thực chính xác hơn, từ đó cải thiện độ chính xác và độ tin cậy của kết quả kiểm thử.

#### Sleep ngẫu nhiên (Random sleep)

Một cách để triển khai think time động là sử dụng hàm `Math.random()` của JavaScript:

```js
sleep(Math.random() * 5);
```

Dòng trên hướng dẫn k6 sleep trong một khoảng thời gian ngẫu nhiên từ 0 đến 5, bao gồm cả 0 nhưng không bao gồm 5. Giá trị được chọn không nhất thiết phải là một số nguyên.

#### Sleep ngẫu nhiên trong khoảng (Random sleep between)

Nếu bạn muốn xác định think time bằng các số nguyên, hãy thử hàm `randomIntBetween` từ thư viện các hàm hữu ích của k6, được gọi là [jslib](https://jslib.k6.io/).

Đầu tiên, hãy nhập hàm liên quan:

```js
import { randomIntBetween } from "https://jslib.k6.io/k6-utils/1.0.0/index.js";
```

Sau đó, thêm dòng này vào bên trong hàm mặc định của bạn:

```js
sleep(randomIntBetween(1, 5));
```

Kịch bản sẽ tạm dừng trong một số giây từ 1 đến 5, bao gồm cả 1 và 5.

## Bạn nên thêm bao nhiêu think time?

Câu trả lời thực sự là: nó tùy thuộc.

Một số yếu tố ảnh hưởng đến thời lượng think time bạn thêm vào là mục tiêu kiểm thử, lưu lượng truy cập thực tế trông như thế nào và tài nguyên máy tính mà bạn có. Tốt nhất là mô hình hóa các kịch bản kiểm thử càng giống với những gì xảy ra trong môi trường thực tế càng tốt.

Tuy nhiên, trong trường hợp không có bất kỳ dữ liệu nào về lưu lượng truy cập thực tế, bạn có thể tính thời gian bạn thực hiện qua một luồng người dùng và sử dụng đó làm điểm bắt đầu.

## Kiểm tra kiến thức của bạn

### Câu hỏi 1

Bạn đang kiểm thử một trang "Mở vé hỗ trợ" mới, nơi người dùng được yêu cầu nhập tên, địa chỉ email và mô tả vấn đề của họ, và các phản hồi của họ được gửi đến đội ngũ ứng dụng. Bạn có nên sử dụng think time không?

A: Có, vì sẽ mất thời gian để đội ngũ ứng dụng phản hồi vé hỗ trợ.

B: Có, vì người dùng mất thời gian để nhập vấn đề của họ.

C: Không, vì mức sử dụng CPU của trình tạo tải quá cao.

### Câu hỏi 2

Trong dòng sau, con số 3 đại diện cho điều gì?

`sleep(3)`

A: Think time là 3 mili giây

B: Số lần lặp sẽ có think time

C: Think time là 3 giây

### Câu hỏi 3

Một kịch bản không có think time chạy với một lần lặp duy nhất, và thời lượng lần lặp là 5 giây. Thời lượng lần lặp sẽ là bao nhiêu nếu kịch bản bao gồm một lệnh sleep 1 giây?

A: 5 giây

B: 6 giây

C: 4 giây

### Đáp án

1. B. A sai vì think time mô phỏng thời gian người dùng tương tác với ứng dụng, không phải thời gian đội ngũ ứng dụng phản hồi. C sai vì thêm think time không làm tăng mức sử dụng CPU; thực tế, điều ngược lại là đúng vì nó làm giảm mức sử dụng CPU bằng cách giãn cách các yêu cầu.
2. C. Tham số của `sleep()` nhận một giá trị tính bằng giây, vì vậy `sleep(3)` sẽ tạm dừng việc thực thi kịch bản trong 3 giây.
3. B. Nếu một kịch bản mất 5 giây để thực thi mà không có sleep, và một lệnh sleep 1 giây được thêm vào, thì kịch bản sẽ mất 6 giây để thực thi.
