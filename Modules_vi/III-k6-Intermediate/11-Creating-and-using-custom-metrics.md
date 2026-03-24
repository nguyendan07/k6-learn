# Sử dụng các biến ngữ cảnh thực thi (execution context variables)

Trong các phần trước, bạn đã học cách làm cho các kịch bản k6 của mình [trở nên thực tế hơn](03-Workload-modeling.md) bằng cách sử dụng các executors và scenarios để mô phỏng tốt hơn lưu lượng truy cập từ những người dùng thực của ứng dụng. Việc đưa kịch bản của bạn đến gần hơn với lưu lượng thực tế (production traffic) giúp kết quả kiểm thử chính xác hơn.

Tuy nhiên, bạn có thể nhanh chóng nhận thấy rằng việc tăng tốc kịch bản của mình trên nhiều người dùng ảo, scenarios và các instances cũng có thể dẫn đến việc khó khăn hơn trong việc hiểu kết quả. Ví dụ, khi kịch bản báo cáo một lỗi, làm thế nào để bạn truy vết nó đến đúng VU đã gặp phải lỗi đó?

Một biến ngữ cảnh thực thi (execution context variable) lưu trữ thông tin về cách bài kiểm tra đang chạy. Nó hoạt động hơi giống một biến môi trường, nhưng là một biến được k6 xác định trước và dành riêng cho trạng thái thực thi hiện tại. Trong k6, các biến này được bao gồm trong module `k6/execution`.

## Các loại biến ngữ cảnh thực thi

Module `k6/execution` chứa nhiều thuộc tính mà bạn có thể gọi trong các bài kiểm tra của mình. Chúng có thể được phân loại tùy theo việc chúng cung cấp thông tin về mục nào sau đây:

- Instance
- Scenario
- Test
- VU

Mỗi loại trong bốn loại trên là một đối tượng trong module `k6/execution`.

Đối tượng **instance** lưu trữ thông tin về mỗi trình tạo tải (load generator), hoặc máy mà tiến trình k6 đang chạy trên đó.

Đối tượng **scenario** nắm giữ thông tin về hồ sơ tải (load profile), chẳng hạn như scenario và executor nào đang được thực thi.

Đối tượng **test** cho phép bạn hủy bỏ việc thực thi bài kiểm tra với một thông báo lỗi tùy chọn.

Đối tượng **VU** gán một con số duy nhất cho mọi lần lặp và mọi VU trong trình tạo tải.

[Tìm hiểu thêm về module `k6/execution`](https://k6.io/docs/javascript-api/k6-execution/).

## Tạo dữ liệu duy nhất

Hãy tưởng tượng rằng bạn muốn viết một kịch bản k6 để đăng nhập vào một ứng dụng web. Tuy nhiên, ứng dụng có các biện pháp bảo mật không cho phép nhiều lần đăng nhập đồng thời cho cùng một tài khoản. Tính năng này gây ra các lỗi HTTP 403 Forbidden khi VU k6 thứ hai đăng nhập trong khi VU đầu tiên vẫn đang đăng nhập. Một hoặc cả hai VUs có thể bị đăng xuất.

Bạn có thể làm gì để ngăn chặn điều này? Bạn có thể đảm bảo gán một thông tin đăng nhập duy nhất cho mỗi VU, sao cho chỉ VU đó mới thực hiện đăng nhập vào tài khoản người dùng đó. Để tạo một thông tin đăng nhập duy nhất, bạn có thể tận dụng một mã định danh mà bạn có thể đảm bảo sẽ là duy nhất cho mỗi VU trong toàn bộ bài kiểm tra: `vu.idInTest`, một biến ngữ cảnh thực thi.

Đầu tiên, tạo một tệp CSV chứa tên người dùng và mật khẩu theo định dạng sau:

```plain
username,password
user01,123
user02,234
user03,345
user04,456
user05,567
user06,678
user07,789
user08,890
user09,901
user10,012
```

Lưu tệp dưới tên `/data/users-unique.csv`. Hãy đảm bảo rằng tất cả các thông tin này đều là thông tin đăng nhập hợp lệ cho ứng dụng đang được kiểm thử của bạn.

Thứ hai, viết một kịch bản k6 sử dụng `users-unique.csv`. Kịch bản ví dụ bên dưới chứa một câu lệnh log thay vì một lần đăng nhập thực tế, để giúp bạn dễ hiểu hơn điều gì đang xảy ra.

```js
import papaparse from "https://jslib.k6.io/papaparse/5.1.1/index.js";
import { SharedArray } from "k6/data";
import { vu } from "k6/execution";
import { sleep } from "k6";

const users = new SharedArray("Logins", function () {
    let data = papaparse.parse(open("./data/users-unique.csv"), {
        header: true,
    }).data;
    return data;
});

export const options = {
    scenarios: {
        login: {
            executor: "per-vu-iterations",
            vus: users.length,
            iterations: 1,
        },
    },
};

export default function () {
    console.log(
        "VU: " + vu.idInTest + " / username: ",
        users[vu.idInTest - 1].username,
        " / password: ",
        users[vu.idInTest - 1].password,
    );
    sleep(1);
}
```

Hãy cùng xem kịch bản đang làm gì theo từng bước.

Có ba đối tượng được kịch bản nhập vào:

- `papaparse` cho phép sử dụng một [tệp CSV làm dữ liệu kiểm thử](04-Adding-test-data.md#CSV-Files)
- `SharedArray` phân phối các dòng của tệp CSV chỉ khi cần thiết cho mỗi VU để [tiết kiệm tài nguyên](04-Adding-test-data.md#Shared-Array)
- `vu` cho phép bạn truy cập vào mã định danh duy nhất cho mỗi VU
- `sleep` cho phép bạn sử dụng các khoảng trì hoãn trong kịch bản

Các dòng này thiết lập mảng dùng chung (shared array) để sử dụng một tệp CSV - tệp `users-unique.csv` mà bạn đã tạo trước đó:

```js
const users = new SharedArray("Logins", function () {
    let data = papaparse.parse(open("./data/users-unique.csv"), {
        header: true,
    }).data;
    return data;
});
```

> :warning: **Thiết lập header thành true**. Lưu ý rằng tùy chọn `header: true` là cần thiết trong trường hợp này, vì dòng đầu tiên trong tệp CSV bạn tạo đã định nghĩa tên của các cột trong tệp dữ liệu (`username,password`).

Vài dòng tiếp theo định nghĩa [các tùy chọn và tham số](../II-k6-Foundations/06-k6-Load-Test-Options.md) cho bài kiểm tra:

```js
export const options = {
    scenarios: {
        login: {
            executor: "per-vu-iterations",
            vus: users.length,
            iterations: 1,
        },
    },
};
```

Số lượng VUs được đặt thành `users.length` để đảm bảo rằng sẽ có một VU cho mỗi dòng trong tệp CSV. Thiết lập này tự động bỏ qua dòng tiêu đề (header). Vì có 11 dòng trong tệp CSV và dòng đầu tiên là tiêu đề, bạn có thể mong đợi 10 VUs được thực thi, mỗi cái một lần.

Bây giờ đến phần chính của bài kiểm tra:

```js
export default function () {
    console.log(
        "VU: " + vu.idInTest + " / username: ",
        users[vu.idInTest - 1].username,
        " / password: ",
        users[vu.idInTest - 1].password,
    );
    sleep(1);
}
```

Hàm này bao gồm một câu lệnh log và một lệnh sleep. Câu lệnh log là vật thế chỗ cho một hàm đăng nhập; thay vì đăng nhập với một người dùng, kịch bản hướng dẫn k6 in ra `idInTest` của VU hiện tại, vốn là một mã định danh duy nhất trên toàn cầu. k6 sau đó in ra tên người dùng và mật khẩu được chọn cho VU đó.

> :point_up: **Tại sao lại là `idInTest - 1`?**
> Bạn có thể nhận thấy rằng trong khi `vu.idInTest` được sử dụng khi in mã định danh, thì `vu.idInTest - 1` được sử dụng để chọn tên người dùng và mật khẩu từ tệp CSV.
> Sự khác biệt này là do thực tế mảng bắt đầu từ 0 trong khi `idInTest` bắt đầu từ 1. Trong ví dụ này, các phần tử mảng tương ứng với các hàng trong CSV là `0,1,2,3,4,5,6,7,8,9` trong khi các `idInTest` cho mỗi VU là `1,2,3,4,5,6,7,8,9,10`.

Cuối cùng, lưu kịch bản dưới tên `script.js` và chạy nó: `k6 run script.js`.

```plain
INFO[0000] VU: 10 / username:  user10  / password:  012  source=console
INFO[0000] VU: 5 / username:  user05  / password:  567   source=console
INFO[0000] VU: 6 / username:  user06  / password:  678   source=console
INFO[0000] VU: 9 / username:  user09  / password:  901   source=console
INFO[0000] VU: 4 / username:  user04  / password:  456   source=console
INFO[0000] VU: 1 / username:  user01  / password:  123   source=console
INFO[0000] VU: 8 / username:  user08  / password:  890   source=console
INFO[0000] VU: 2 / username:  user02  / password:  234   source=console
INFO[0000] VU: 3 / username:  user03  / password:  345   source=console
INFO[0000] VU: 7 / username:  user07  / password:  789   source=console
```

Dựa trên đầu ra của kịch bản ở trên, bạn có thể xác minh rằng:

- 10 VUs được thực thi mỗi cái một lần, theo đúng số lượng các dòng không phải tiêu đề trong tệp CSV
- Mỗi VU chỉ sử dụng tên người dùng và mật khẩu được gán cho nó
- `idInTest` cho mỗi VU là duy nhất trên toàn cầu.

Xung đột dữ liệu (data collision) có thể là một vấn đề lớn khi bạn tăng tốc bài kiểm tra k6 của mình để mở rộng trên nhiều VUs, scenarios và instances. Việc sử dụng các biến ngữ cảnh thực thi như được minh họa ở đây giúp ngăn ngừa các lỗi do viết kịch bản có thể cản trở bài kiểm tra của bạn đạt được mức tải mong muốn.

## Các trường hợp sử dụng khác cho các biến ngữ cảnh thực thi

Bạn cũng có thể sử dụng các biến ngữ cảnh thực thi để thực hiện những việc sau:

- [Thêm logic có điều kiện cho các scenarios](https://k6.io/docs/javascript-api/k6-execution/#script-naming)
- [Xác định khi nào cần hủy bỏ một bài kiểm tra](https://k6.io/docs/javascript-api/k6-execution/#test-abort)
- [Cải thiện việc ghi nhật ký khi có lỗi](https://k6.io/docs/javascript-api/k6-execution/#timing-operations)
- [Thêm các thẻ (tags) vào VUs và truy xuất chúng](https://k6.io/docs/javascript-api/k6-execution/#tags)

## Kiểm tra kiến thức của bạn

### Câu hỏi 1

Trong tình huống nào thì việc sử dụng một biến ngữ cảnh thực thi có thể hữu ích?

A: Bạn có một tệp dữ liệu lớn, nhưng muốn đảm bảo rằng không có dữ liệu trùng lặp nào được chọn giữa các VUs.

B: Bạn muốn chạy một kịch bản k6 nhưng thiết lập số lượng VUs từ dòng lệnh.

C: Bạn muốn chỉ định executor mà một scenario sử dụng.

### Câu hỏi 2

Một kịch bản kiểm thử định nghĩa 10 VUs, mỗi cái thực hiện 2 lần lặp. Bạn mong đợi có bao nhiêu giá trị duy nhất của `vu.idInTest` tồn tại cho bài kiểm tra này?

A: 20.

B: 10.

C: Nó phụ thuộc vào việc bài kiểm tra được thực thi trên bao nhiêu instances.

### Câu hỏi 3

Bạn sẽ sử dụng đối tượng ngữ cảnh thực thi nào sau đây để truy xuất thông tin về executor hiện đang chạy?

A: `scenario`

B: `vu`

C: `test`

### Đáp án

1. A. Bạn có thể ngăn chặn xung đột dữ liệu bằng cách sử dụng một mã định danh duy nhất như `vu.idInTest`. B sai vì bạn thiết lập số lượng VUs từ dòng lệnh bằng cách sử dụng [các cờ CLI](../II-k6-Foundations/02-The-k6-CLI.md), và C sai vì cách chọn một executor là [từ trong kịch bản](08-Setting-load-profiles-with-executors.md).
2. B. `vu.idInTest` được gán cho mỗi VU, không phải cho mỗi lần lặp.
3. A. `scenario` nắm giữ thông tin về executor đang được sử dụng.
