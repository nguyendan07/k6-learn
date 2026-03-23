# Kiểm thử tải (Load Testing)

Một sai lầm phổ biến trong ngành là sử dụng các thuật ngữ _performance testing_ (kiểm thử hiệu năng) và _load testing_ (kiểm thử tải) thay thế cho nhau. Chúng không thể thay thế cho nhau! Thay vào đó, load testing là một nhánh con của performance testing.

**Performance testing != Load testing**

[Performance testing](01-Introduction-to-Performance-Testing.md) xác minh hệ thống hoạt động tốt như thế nào về tổng thể, bao gồm các khía cạnh như khả năng mở rộng (scalability), tính linh hoạt (elasticity), tính sẵn sàng (availability), độ tin cậy (reliability), khả năng phục hồi (resiliency) và độ trễ (latency). Load testing chỉ là một loại của performance testing, và nó là một phương pháp có thể được sử dụng để kiểm tra nhiều khía cạnh của hiệu năng ứng dụng. Tuy nhiên, không phải tất cả các hoạt động performance testing đều liên quan đến load testing.

Load testing tập trung cụ thể vào việc xác minh và xác nhận hiệu năng của một ứng dụng khi nó có một khối lượng lớn các tác vụ cần xử lý. Load testing có thể được thực hiện thủ công, nhưng hầu hết các đội ngũ đều viết các kịch bản load testing tự động để mô phỏng một cách lập trình những người dùng thực truy cập vào ứng dụng.

## Các tham số kiểm thử (Test parameters)

Khi viết các kịch bản load test, bạn phải xem xét nhiều yếu tố ngoài lượng tải thuần túy cần tạo ra. Các yếu tố này, được gọi là _tham số kiểm thử_ (test parameters), bao gồm sự phân bổ, hình dạng và mô hình của tải.

Dưới đây là danh sách không đầy đủ của một số tham số kiểm thử phổ biến:

- **Virtual users (VUs).** Một VU là một luồng thực thi độc lập chạy đồng thời với các luồng VU khác. Thông thường, các kịch bản sẽ được thiết kế sao cho hoạt động của 1 VU đại diện cho hoạt động của 1 người dùng thực.
- **Lần lặp (Iterations)**. Tổng số lần lặp lại kịch bản sẽ được thực hiện bởi các VU.
- **Lưu lượng (Throughput)**. Thước đo lượng tải mà bài kiểm tra tạo ra theo thời gian, thường được định nghĩa bằng VUs trên giây, yêu cầu trên giây (requests per second), hoặc lần lặp trên giây (iterations per second).
- **Luồng người dùng (User flows)**. Các hành động mà kịch bản thực hiện và thứ tự chúng chạy. Nhìn chung, một luồng người dùng đại diện cho lộ trình của một người dùng thực qua ứng dụng.
- **Hồ sơ tải (Load profile)**. Hình dạng của lưu lượng truy cập được tạo ra bởi bài kiểm tra theo thời gian. Nó bao gồm số lượng khoảng hoãn (trong think time), các giai đoạn tăng tải (ramp-up) và giảm tải (ramp-down) khi bài kiểm tra tăng hoặc giảm dần số lượng VU theo thời gian, và các giai đoạn (stages) của bài kiểm tra.
- **Thời gian chạy (Duration)**. Thời gian cần thiết để chạy toàn bộ bài kiểm tra và các giai đoạn riêng lẻ của nó.

## Cách mô phỏng tải

Nhìn chung, bạn có thể mô phỏng tải theo một vài cách:

**Kiểm thử tải dựa trên giao thức (Protocol-based load testing)** mô phỏng các yêu cầu cơ bản gửi đến máy chủ ứng dụng. Các yêu cầu này được gửi qua lớp giao thức. Kiểm thử tải dựa trên giao thức thay đổi tùy theo phạm vi, bao gồm cả:

- kiểm thử API mục tiêu của một hoặc nhiều thành phần ứng dụng cụ thể, và
- kiểm thử đầu cuối (end-to-end) hoặc thực tế hơn nhằm mô phỏng lưu lượng chạy qua toàn bộ hệ thống, đi qua nhiều thành phần.

**Kiểm thử tải dựa trên trình duyệt (Browser-based load testing)**, mặt khác, mô phỏng cách người dùng tương tác với giao diện người dùng của ứng dụng. Thay vì mô phỏng các yêu cầu ở cấp độ giao thức, kiểm thử dựa trên trình duyệt tự động hóa những việc như nhấp vào các phần tử của ứng dụng web, nhập vào biểu mẫu và các hành động khác mà người dùng thực có thể thực hiện. Kiểm thử tải dựa trên trình duyệt thường liên quan đến kiểm thử đầu cuối hơn.

**Kiểm thử tải hỗn hợp (Hybrid load testing)** là sự kết hợp giữa kiểm thử dựa trên giao thức và dựa trên trình duyệt. Nó thường được sử dụng để kiểm tra các khía cạnh khác nhau của hiệu năng. Cách tiếp cận kinh tế nhất bao gồm sử dụng kiểm thử tải dựa trên giao thức để tạo ra phần lớn tải kết hợp với một số lượng nhỏ người dùng kiểm thử tải dựa trên trình duyệt.

## Các kịch bản kiểm thử tải (Load test scenarios)

Một kịch bản load test kết hợp các giá trị cụ thể của các tham số kiểm thử. Mỗi kịch bản tái tạo một tình huống hoặc một tập hợp các điều kiện nhất định mà ứng dụng sẽ gặp phải.

Các kịch bản load test thường được gọi là _loại kiểm thử tải_ (load test types). Một số kịch bản phổ biến nhất được liệt kê ở đây.

### Kiểm thử sàng lọc (Shakeout test)

Một bài kiểm thử sàng lọc (shakeout test), đôi khi được gọi là smoke test, là một bài kiểm tra nhỏ nhằm kiểm tra các vấn đề lớn trước khi dành nhiều thời gian và tài nguyên hơn. Một bài shakeout test thường sử dụng một hoặc một vài VU chạy trong một khoảng thời gian ngắn và kiểm tra các vấn đề lớn như:

- các vấn đề liên quan đến kịch bản có thể ảnh hưởng đáng kể đến độ chính xác của kết quả kiểm thử
- cấu hình môi trường không mong muốn
- các nút thắt cổ chai hiệu năng nghiêm trọng của ứng dụng phát sinh ngay cả ở mức tải thấp

Nếu bài shakeout test thất bại, bất kỳ vấn đề nào được phát hiện phải được giải quyết trước khi tiếp tục tăng throughput của bài kiểm tra.

_Ví dụ:_
![](../../images/scenarios-shakeout.png)

```
Name: Shakeout Test
Total VUs: 5
Ramp-up: 0 seconds
Duration: 10 minutes
Ramp-Down: 0 seconds
```

Một bài shakeout test cũng có thể bao gồm [nhiều kịch bản (multiple scenarios)](../III-k6-Intermediate/09-Workload-modeling-with-scenarios.md).

### Kiểm thử tải trung bình (Average Load Test)

Kịch bản này mô phỏng khối lượng công việc của người dùng trong một giờ điển hình trong môi trường production. Kịch bản bao gồm các yêu cầu hoặc chức năng được thực thi thường xuyên nhất trong giờ đó.

Kịch bản kiểm thử này thường bao gồm các giai đoạn ramp-up và ramp-down để mô phỏng việc người dùng đăng nhập và tương tác với hệ thống một cách dần dần. Bài kiểm thử tải tăng dần số lượng VU cho đến khi đạt được mức tải mong muốn để bắt chước hành vi tải trung bình trong production.

Giữa các giai đoạn ramp-up và ramp-down là trạng thái ổn định (steady state), một khoảng thời gian mà số lượng người dùng ảo là không đổi.
Trong một bài kiểm thử tải trung bình, bài kiểm tra duy trì mô phỏng tải ở trạng thái ổn định trong khoảng một giờ hoặc lâu hơn.

_Ví dụ_:

![](../../images/test-scenario-average.png)

```
Name: Average Load Test
Total VUs: 100
Ramp-up: 30 minutes
Steady state: 60 minutes
Ramp-down: 10 minutes
Total duration: 100 minutes
```

Trong ví dụ trên, bài kiểm thử tải trung bình được định nghĩa theo số lượng VU, nhưng nó cũng có thể được định nghĩa theo số lượng iterations hoặc requests per second mà bài kiểm tra tạo ra. Để biết thêm thông tin, hãy xem [Thiết lập hồ sơ tải với executors](../III-k6-Intermediate/08-Setting-load-profiles-with-executors/Setting-load-profiles-with-executors.md).

### Kiểm thử căng thẳng (Stress test)

Một bài kiểm thử căng thẳng (stress test), còn được gọi là peak load test, mô phỏng lưu lượng truy cập mà ứng dụng dự kiến sẽ trải qua tại thời điểm _cao nhất_ trong ngày hoặc trong mùa. Trong khi bài kiểm thử tải trung bình mô phỏng lưu lượng truy cập vào một ngày điển hình, được tổng hợp qua một tuần, một tháng hoặc lâu hơn, thì stress test tập trung vào lượng lưu lượng truy cập cao nhất mà ứng dụng trải qua.

Mặc dù stress test có hình dạng tương tự như bài kiểm thử tải trung bình, nhưng nó thường tạo ra throughput cao hơn nhiều. Hãy xem xét một ứng dụng thường có 100 người dùng truy cập trong các giờ "bình thường", nhưng có 300 người dùng trong giờ nghỉ trưa. Ứng dụng này có thể hưởng lợi từ việc được kiểm thử với một bài stress test ở mức 300 VU.

Stress tests là một kịch bản kiểm thử tốt khi kiểm tra các giờ cao điểm hoặc các giai đoạn bán hàng mà ứng dụng phải đối mặt với tải nặng bất thường.

### Kiểm thử ngâm hoặc Kiểm thử độ bền (Soak or Endurance Test)

Soak tests, còn được gọi là endurance tests, là các bài kiểm tra có thời gian chạy dài hơn so với các bài kiểm tra trung bình hoặc cao điểm. Một số nút thắt cổ chai hiệu năng, chẳng hạn như những nút thắt do lỗi quản lý bộ nhớ gây ra, chỉ xuất hiện trong thời gian dài hơn. Soak tests xác minh xem hiệu năng có bị giảm sút theo thời gian hay không.

Kịch bản này có xu hướng có mức throughput kiểm thử tương tự như bài kiểm thử tải trung bình, nhưng nó được kéo dài thời gian chạy lên vài giờ hoặc thậm chí vài ngày, tùy thuộc vào ứng dụng.

_Ví dụ_:

![](../../images/test-scenario-soak.png)

```
Name: Soak Test
Total VUs: 50
Ramp-up: 30 minutes
Steady state: 480 minutes
Ramp-down: 10 minutes
Total duration: 520 minutes (8 hours and 40 minutes)
```

### Kiểm thử đột biến (Spike Test)

Các loại kiểm thử trước đây đều tái tạo tình huống tải được đưa vào dần dần (như trong trường hợp kiểm thử trung bình, căng thẳng và ngâm) hoặc nơi thực hiện một lượng tải nhỏ (sàng lọc). Ngược lại, một bài kiểm thử đột biến (spike test) tái tạo tình huống ứng dụng trải qua một sự gia tăng lưu lượng truy cập _đột ngột_ và khổng lồ.

Spike tests có thể được sử dụng để xác minh hiệu năng ứng dụng trong những thời điểm lưu lượng truy cập đi từ thấp đến cực cao trong một khoảng thời gian ngắn. Spike tests rất tốt để mô phỏng các sự kiện có thời gian cụ thể như:

- các thông báo sản phẩm nổi bật (như trong một quảng cáo Super Bowl)
- ra mắt sản phẩm hoặc bán vé buổi hòa nhạc
- thời hạn cuối (những ngày cuối cùng nộp hồ sơ thuế)
- mở đầu các mùa giảm giá (Black Friday hoặc Cyber Monday)

Spike tests có throughput cao và thời gian trạng thái ổn định ngắn. Chúng thường có giai đoạn ramp-up và ramp-down không đáng kể.

Một điểm khác biệt nữa so với các kịch bản trước đó là việc lựa chọn các luồng người dùng để kiểm thử. Thay vì một loạt các quy trình thông thường hàng ngày, spike tests thường ưu tiên một luồng người dùng duy nhất.
Ví dụ, trong một đợt bán vé sự kiện, người dùng có thể tập trung vào việc mua vé hơn là duyệt các trang khác.

_Ví dụ_:

![](../../images/test-scenario-spike-test.png)

```
Name: Spike Test
Total VUs: 300
Ramp-up: 1 minute
Steady state: 20 minutes
Ramp-down: 5 minutes
Total duration: 26 minutes
```

### Kiểm thử điểm gãy (Breakpoint Test)

Trong khi các loại kiểm thử tải trước đó mô phỏng tải thực tế và dự kiến trong production, các bài kiểm thử điểm gãy (breakpoint tests) cố gắng tiến thêm một bước nữa. Một bài breakpoint test cho ứng dụng tiếp xúc với các mức tải tăng dần nhằm cố gắng xác định mức lưu lượng truy cập mà tại đó hiệu năng bắt đầu giảm sút.

Breakpoint tests xây dựng sự tin tưởng vào những gì một hệ thống có thể xử lý. Kết quả từ breakpoint tests cung cấp các thông tin đầu vào có giá trị cho việc lập kế hoạch năng lực (capacity planning).

Kịch bản breakpoint test tập trung vào giai đoạn ramp-up nhiều hơn các kịch bản khác. Nó có thể bao gồm hoàn toàn một giai đoạn ramp-up dần dần, hoặc có thể bao gồm các giai đoạn ramp-up xen kẽ với các giai đoạn trạng thái ổn định. Mô hình hồ sơ tải theo bậc thang trong ví dụ sau có thể giúp liên hệ sự giảm sút hiệu năng với các mức tải cụ thể.

_Ví dụ_:

![](../../images/test-scenarios-breakpoint.png)

```
Name: Breakpoint Test
Total max VUs: unknown
Ramp-up: 10 minutes before each stage
Steady state: 30 minutes
Ramp-down: 0 minutes
Total duration: unknown
```

Vì breakpoint tests mang tính chất khám phá hơn, những người kiểm thử không thể biết trước tối đa bao nhiêu VU sẽ được thực thi hoặc bài kiểm tra sẽ kéo dài bao lâu. Các đội ngũ thường theo dõi sát sao ứng dụng trong khi bài breakpoint test đang chạy và dừng bài kiểm tra thủ công hoặc lập trình để nó dừng lại khi vượt quá các [ngưỡng (thresholds)](../II-k6-Foundations/07-Setting-test-criteria-with-thresholds.md) nhất định.

## Kiểm tra kiến thức của bạn

### Câu hỏi 1

Phát biểu nào sau đây về các thực hành tốt nhất cho hiệu năng là đúng?

A: Kịch bản kiểm thử tải được chọn có thể ảnh hưởng đáng kể đến độ chính xác của kết quả kiểm thử.

B: Cách tốt nhất để thực hiện kiểm thử hiệu năng là mô phỏng tải cao điểm của những người dùng sẽ truy cập ứng dụng trong production.

C: Số lượng người dùng ảo trong một bài kiểm tra là tham số kiểm thử quan trọng nhất trong một kịch bản.

### Câu hỏi 2

Kịch bản kiểm thử nào sau đây bạn nên sử dụng cho lần thực thi kiểm thử đầu tiên trên một ứng dụng và môi trường mới?

A: Soak Test

B: Shakeout Test

C: Regression Test

### Câu hỏi 3

Kịch bản kiểm thử nào sau đây sẽ phù hợp nhất để kiểm tra rò rỉ bộ nhớ (memory leak)?

A: Stress Test

B: Soak Test

C: Average Load Test

### Đáp án

1. A. B mô tả một loại kiểm thử tải, peak load tests, và C chỉ đề cập đến một trong nhiều tham số kiểm thử có thể có tác động đến kịch bản kiểm thử được chọn và kết quả kiểm thử.
2. B. Một bài soak test được thực thi trong một khoảng thời gian dài hơn, và có thể kết thúc với quá nhiều lỗi để có thể hữu ích nếu môi trường chưa trải qua shakeout testing. Tương tự, các bài kiểm thử hồi quy đầy đủ (full regression tests) có thể không quá hữu ích nếu không xác minh trước rằng môi trường và (các) kịch bản hoạt động bình thường.
3. B. Mặc dù rò rỉ bộ nhớ có thể được phát hiện trong các bài stress test và average load test, nhưng thời gian chạy dài hơn của soak tests khiến chúng trở nên phù hợp duy nhất để phát hiện các xu hướng dần dần như rò rỉ bộ nhớ.
