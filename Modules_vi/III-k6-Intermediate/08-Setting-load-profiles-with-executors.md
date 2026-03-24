# Thiết lập Load Profiles với Executors

Một trong những khía cạnh phức tạp hơn của việc kiểm thử là mô hình hóa hồ sơ tải (load profile) mong muốn của bạn. Các hồ sơ này xác định khối lượng (volume) và tốc độ (velocity) của các yêu cầu gửi đến hệ thống đang được kiểm thử. k6 cung cấp một bộ các [executors](https://k6.io/docs/using-k6/scenarios/executors/) để định hình các mô hình thực thi cho các bài kiểm tra của bạn.

> :point_up: Một điểm quan trọng cần lưu ý là mỗi executor kiểm soát số lượng VUs và/hoặc số lượng--hoặc thậm chí là tỷ lệ--của các lần lặp (iterations) trên mỗi chu kỳ kiểm thử.

Chúng tôi đã cung cấp các bài tập ví dụ để đi sâu vào chi tiết khi làm việc với từng executor có sẵn.

### Shared Iterations

_Shared Iterations_ là executor cơ bản nhất. Như có thể suy ra từ cái tên, trọng tâm chính sẽ là số lượng _iterations_ cho bài kiểm tra của bạn; đây là số lần hàm kiểm thử của bạn sẽ được chạy.

| Tùy chọn      | Mô tả                                                                    | Mặc định |
| ------------- | ------------------------------------------------------------------------ | -------- |
| `iterations`  | Bài kiểm tra của bạn nên thực thi bao nhiêu lần?                         | `1`      |
| `maxDuration` | Dừng cưỡng bức bài kiểm tra nếu không kết thúc trong khung thời gian này | `"10m"`  |
| `vus`         | Số lượng người dùng ảo chạy đồng thời (concurrency)                      | `1`      |

Như đã lưu ý trước đó, mục tiêu chính là thực hiện bài kiểm tra của bạn với số lần `iterations`, trong một khoảng thời gian không vượt quá `maxDuration`. Tại bất kỳ thời điểm nào trong kịch bản kiểm thử, nên có (các) lần lặp `vus` đang diễn ra trừ khi đã đạt đến tổng số lần lặp mong muốn. Trong kịch bản này, có khả năng một số VUs thực hiện nhiều công việc hơn những VUs khác.

[Hãy tự mình thử nghiệm với Shared Iterations!](./08-Setting-load-profiles-with-executors/Shared-Iterations-Exercises.md)

### Per VU Iterations

_Per VU Iterations_ là một bước phát triển nhẹ từ executor _Shared Iterations_. Với executor này, chúng ta vẫn tập trung vào số lượng _iterations_, tuy nhiên, lần này chúng ta muốn **mỗi người dùng ảo (virtual user)** thực thi **cùng một số lượng** lần lặp.

| Tùy chọn      | Mô tả                                                                    | Mặc định |
| ------------- | ------------------------------------------------------------------------ | -------- |
| `iterations`  | Đối với mỗi VU, bài kiểm tra của bạn nên thực thi bao nhiêu lần?         | `1`      |
| `maxDuration` | Dừng cưỡng bức bài kiểm tra nếu không kết thúc trong khung thời gian này | `"10m"`  |
| `vus`         | Số lượng người dùng ảo chạy đồng thời                                    | `1`      |

Trong trường hợp này, chúng ta mong muốn mỗi VU trong số `vus` thực hiện bài kiểm tra với số lần `iterations` trong một khoảng thời gian không vượt quá `maxDuration`. Điều này có nghĩa là các lần lặp kiểm thử được phân bổ _công bằng_; không có VU đơn lẻ nào thực hiện nhiều hơn VU khác.

[Hãy tự mình thử nghiệm với Per VU Iterations!](./08-Setting-load-profiles-with-executors/Per-VU-Iterations-Exercises.md)

### Constant VUs

_Constant VUs_ tập trung vào việc thực hiện liên tục bài kiểm tra của bạn trong một khoảng thời gian cụ thể. Điều này cho phép mỗi người dùng ảo thực hiện nhiều yêu cầu nhất có thể trong khung thời gian cho phép.

| Tùy chọn       | Mô tả                                   | Mặc định     |
| -------------- | --------------------------------------- | ------------ |
| **`duration`** | Tổng thời lượng của kịch bản (scenario) | - (bắt buộc) |
| `vus`          | Số lượng người dùng ảo chạy đồng thời   | `1`          |

Như đã lưu ý ở trên, mục tiêu chính là để mỗi VU trong số `vus` thực hiện nhiều lần lặp kiểm thử nhất có thể trong khoảng thời gian `duration` yêu cầu, ví dụ: `"30s"`, `"1h"`, v.v.

[Hãy tự mình thử nghiệm với Constant VUs!](./08-Setting-load-profiles-with-executors/Constant-VUs-Exercises.md)

### Ramping VUs

_Ramping VUs_ là một bước phát triển của executor _Constant VUs_ có giới thiệu các **_stages_**. Điều này cho phép k6 chuyển đổi số lượng VUs mong muốn từ giai đoạn này sang giai đoạn khác. Mỗi giai đoạn định nghĩa khung thời gian riêng mà tại đó tất cả VUs sẽ thực hiện liên tục bài kiểm tra của bạn.

| Tùy chọn           | Mô tả                                                                                   | Mặc định     |
| ------------------ | --------------------------------------------------------------------------------------- | ------------ |
| **`stages`**       | Bao gồm một khoảng thời gian `duration` và `target` cho số lượng VUs mong muốn          | - (bắt buộc) |
| `gracefulRampDown` | Thời gian ân hạn để một lần lặp kiểm thử kết thúc trước khi tắt một VU khi ramping down | `"30s"`      |
| `startVUs`         | Số lượng người dùng ảo tại thời điểm bắt đầu bài kiểm tra                               | `1`          |

Là một kịch bản dựa trên thời gian, tổng thời lượng bằng tổng của (các) khung thời gian `duration` từ mỗi `stage`. Giai đoạn đầu tiên sẽ bắt đầu với số lượng `startVUs` VU(s) và tăng (hoặc giảm) tuyến tính trong suốt `duration` đã cấu hình đến số lượng VUs `target` được chỉ định trong giai đoạn đó. Giai đoạn tiếp theo, nếu được cấu hình, sau đó sẽ tăng hoặc giảm từ điểm đó đến số lượng VUs `target` mong muốn trong khung thời gian `duration` đã chỉ định. Mô hình này tiếp tục cho mỗi giai đoạn còn lại. Giống như với _Constant VUs_, mỗi VU đang chạy sẽ thực hiện liên tục các lần lặp kiểm thử cho đến khi kịch bản kết thúc.

[Hãy tự mình thử nghiệm với Ramping VUs!](./08-Setting-load-profiles-with-executors/Ramping-VUs-Exercises.md)

### Constant Arrival Rate

Với _Constant Arrival Rate_, bây giờ chúng ta bắt đầu tập trung vào tốc độ mà các lần lặp kiểm thử của bạn được thực hiện trong một khoảng thời gian quy định. k6 sẽ tự động điều chỉnh số lượng VUs để đạt được tỷ lệ mong muốn.

| Tùy chọn              | Mô tả                                                               | Mặc định          |
| --------------------- | ------------------------------------------------------------------- | ----------------- |
| **`duration`**        | Tổng thời lượng của kịch bản                                        | - (bắt buộc)      |
| **`preAllocatedVUs`** | Số lượng người dùng ảo tại thời điểm bắt đầu bài kiểm tra           | - (bắt buộc)      |
| **`rate`**            | Tỷ lệ lần lặp mong muốn trên mỗi `timeUnit` cần đạt được và duy trì | - (bắt buộc)      |
| `maxVUs`              | Số lượng người dùng ảo tối đa được phép mở rộng                     | - (không mở rộng) |
| `timeUnit`            | Khoảng thời gian mà tỷ lệ `rate` mong muốn áp dụng                  | `"1s"`            |

Trọng tâm chính của chúng ta sẽ là đạt được và duy trì một _tỷ lệ lần lặp (iteration rate)_ là `rate` trên mỗi `timeUnit` trong suốt `duration` mong muốn. Số lượng VUs để đạt được tỷ lệ mong muốn sẽ được k6 quản lý, và sẽ nằm trong khoảng từ `preAllocatedVUs` đến `maxVUs`. Lưu ý rằng có chi phí về thời gian và tài nguyên liên quan đến việc tạo VU, vì vậy việc định nghĩa một `preAllocatedVUs` hợp lý sẽ cho phép có nhiều thời gian kiểm thử hơn ở tỷ lệ mong muốn.

[Hãy tự mình thử nghiệm với Constant Arrival Rate!](./08-Setting-load-profiles-with-executors/Constant-Arrival-Rate-Exercises.md)

### Ramping Arrival Rate

_Ramping Arrival Rate_ là một bước phát triển của executor _Constant Arrival Rate_ có giới thiệu các **_stages_**. Đây có lẽ là ứng cử viên tốt nhất để mô hình hóa các kịch bản kiểm thử trong thế giới thực, cho phép k6 chuyển đổi _iteration rate_ mong muốn từ giai đoạn này sang giai đoạn khác. Mỗi giai đoạn định nghĩa khung thời gian riêng để đạt được tỷ lệ mong muốn.

| Tùy chọn              | Mô tả                                                                                               | Mặc định          |
| --------------------- | --------------------------------------------------------------------------------------------------- | ----------------- |
| **`preAllocatedVUs`** | Số lượng người dùng ảo tại thời điểm bắt đầu bài kiểm tra                                           | - (bắt buộc)      |
| **`stages`**          | Bao gồm một khoảng thời gian `duration` và `target` cho tỷ lệ lần lặp mong muốn trên mỗi `timeUnit` | - (bắt buộc)      |
| `maxVUs`              | Số lượng người dùng ảo tối đa được phép mở rộng                                                     | - (không mở rộng) |
| `startRate`           | Tỷ lệ lần lặp mong muốn trên mỗi `timeUnit` cần đạt được và duy trì ban đầu                         | `0`               |
| `timeUnit`            | Khoảng thời gian mà tỷ lệ `rate` mong muốn áp dụng                                                  | `"1s"`            |

Tương tự như _Constant Arrival Rate_, trọng tâm chính là đạt được một _iteration rate_ mục tiêu. Sự khác biệt chính là tỷ lệ mong muốn đạt được trong mỗi `stage` được định nghĩa. Tổng thời lượng sẽ bằng tổng của (các) khung thời gian `duration` từ mỗi `stage`. Giai đoạn đầu tiên sẽ bắt đầu với một _iteration rate_ là `startRate` trên mỗi `timeUnit` được thực hiện bởi (các) người dùng ảo `preAllocatedVUs`. _Iteration rate_ sẽ tăng (hoặc giảm) tuyến tính trong suốt `duration` đã cấu hình đến tỷ lệ `target` được chỉ định cho giai đoạn đó. Giai đoạn tiếp theo, nếu được cấu hình, sau đó sẽ tăng hoặc giảm từ điểm đó đến tỷ lệ `target` mong muốn trong khung thời gian `duration` đã chỉ định. Mô hình này tiếp tục cho mỗi giai đoạn còn lại. Các hiện tượng _Spikes_ (đột biến), _valleys_ (thung lũng), và _plateaus_ (cao nguyên) có thể được mô phỏng với các giai đoạn này.

[Hãy tự mình thử nghiệm với Ramping Arrival Rate!](./08-Setting-load-profiles-with-executors/Ramping-Arrival-Rate-Exercises.md)

### Externally Controlled

_Externally Controlled_ là một executor hoàn toàn khác biệt ở chỗ nó không thay đổi số lượng người dùng ảo ngoài việc bắt đầu bài kiểm tra và thiết lập các giới hạn về VUs và thời lượng. Việc kiểm soát này được kỳ vọng sẽ được cung cấp bởi các tiến trình bên ngoài bằng cách sử dụng k6 REST API hoặc k6 CLI.

| Tùy chọn       | Mô tả                                           | Mặc định     |
| -------------- | ----------------------------------------------- | ------------ |
| **`duration`** | Tổng thời lượng của kịch bản                    | - (bắt buộc) |
| `maxVUs`       | Số lượng người dùng ảo tối đa được phép sử dụng | `0`          |
| `vus`          | Số lượng người dùng ảo chạy đồng thời           | `0`          |

Mục tiêu chính của executor này là xác định khung thời gian `duration` của bài kiểm tra. Nếu không có gì khác được cung cấp, k6 về cơ bản sẽ ở trạng thái chờ lệnh. Nếu không có lệnh nào được đưa ra, k6 sẽ đơn giản là thoát sau khi đạt đến thời lượng. Nếu `vus` được chỉ định với một giá trị khác không, executor sẽ chạy tương tự như _Constant VUs_ cho đến khi được tác động bởi một tác nhân bên ngoài. Thiết lập `maxVUs` sẽ đặt ra một giới hạn để thực thi khi nhận được các yêu cầu bên ngoài để mở rộng quy mô (scale up).

[Hãy tự mình thử nghiệm với Externally Controlled!](./08-Setting-load-profiles-with-executors/Externally-Controlled-Exercises.md)

## Kiểm tra kiến thức của bạn

Hãy xem bạn hiểu các executors đến mức nào với bài trắc nghiệm sau. Kiểm tra câu trả lời của bạn với [phần đáp án](#Answers) ở phía cuối trang.

### Câu hỏi 1

Một hệ thống chạy với khoảng 80 người dùng đồng thời và thỉnh thoảng gặp các đột biến thêm 50 người dùng. Điều này được mô hình hóa tốt nhất như thế nào?

A: Sử dụng `ramping-arrival-rate` với nhiều giai đoạn. Bắt đầu với 80 VUs với một giai đoạn để duy trì trong một khoảng thời gian ngắn, sau đó một giai đoạn khác để nhắm mục tiêu 130 người dùng trong một thời lượng ngắn, sau đó một giai đoạn khác để đưa về 80 người dùng.

B: Sử dụng `constant-vus` với 130 người dùng ảo. Cho chạy với thời lượng 5 phút.

C: Sử dụng `ramping-vus` với nhiều giai đoạn. Bắt đầu với 80 VUs với một giai đoạn để duy trì trong một khoảng thời gian ngắn, sau đó một giai đoạn khác để nhắm mục tiêu 130 người dùng trong một thời lượng ngắn, sau đó một giai đoạn khác để đưa về 80 người dùng.

### Câu hỏi 2

Chúng ta đang tìm cách thiết lập một số chỉ số cơ sở để định nghĩa một SLA cho một dịch vụ hiện có, cách nhanh nhất để đạt được điều này là gì?

A: Sử dụng executor `shared-iterations` để chạy qua 1.000 yêu cầu.

B: Sử dụng `constant-arrival-rate` để xem cần bao nhiêu người dùng ảo để duy trì mức 50 yêu cầu trên giây (RPS) không đổi.

C: Sử dụng executor `externally-controlled` để bắt đầu k6 ở chế độ máy chủ để kịch bản Bash _xịn xò_ của bạn tăng dần người dùng ảo.

### Câu hỏi 3

Đội ngũ SRE của bạn đã thấy các vấn đề với dịch vụ của bạn khi gặp phải tình trạng tạm dừng để thu gom rác (garbage collection pause) một khi một instance bắt đầu vượt quá 30 RPS. Bạn vẫn chưa sử dụng Kubernetes, vì vậy việc mở rộng quy mô không phải là một lựa chọn dễ dàng. Làm thế nào các nhà phát triển của bạn có thể mô phỏng tải tại địa phương để kiểm tra mã marshalling JSON của họ?

A: Sử dụng `constant-vus` để mô phỏng 30 người dùng ảo thực hiện các yêu cầu nhanh nhất có thể.

B: Sử dụng `constant-arrival-rate` để duy trì mức 30 yêu cầu trên giây (RPS) không đổi.

C: Sử dụng executor `per-vu-iterations` để 30 người dùng ảo chạy 1.000 yêu cầu mỗi người.

> :rocket: **Muốn tìm hiểu thêm?** Hãy xem buổi [k6 Office Hours](https://www.youtube.com/playlist?list=PLJdv3RhAQXNE1TFXn2pp9h_Ul1q_kJrEZ) nơi chúng tôi đã nói sâu hơn về _Executors trong k6_!

[![k6 Office Hours](../../images/office-hours-executors-k6.png)](https://www.youtube.com/playlist?list=PLJdv3RhAQXNE1TFXn2pp9h_Ul1q_kJrEZ)

#### Đáp án

1. C. `ramping-vus` cho phép bạn mô hình hóa các đột biến về số lượng _người dùng ảo_ đồng thời.
2. A. Với `shared-iterations`, chúng ta có thể dễ dàng chạy qua một số lượng cố định các lần lặp kiểm thử, có hoặc không có tính đồng thời. Chúng ta đang tìm kiếm các mức dịch vụ đơn giản để thiết lập một đường cơ sở (baseline), vì vậy có lẽ sẽ không cần bất cứ điều gì quá phức tạp.
3. B. Với executor `constant-arrival-rate`, bạn có thể yêu cầu kịch bản của mình đạt được và duy trì tốc độ yêu cầu mục tiêu để theo dõi bộ nhớ heap của mình.
