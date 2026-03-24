# Các hàm Setup và Teardown

Cho đến nay, bạn đã đặt hầu hết mã nguồn bên trong hàm `default`. Tất cả mã nguồn bên trong hàm default được gọi là **VU code**, vì đó là mã mà mọi người dùng ảo (virtual user) thực thi và lặp lại cho đến khi kết thúc bài kiểm tra.

Trừ khi bạn chỉ định số lượng người dùng ảo và thời lượng kiểm thử trong [các tùy chọn kiểm thử](../II-k6-Foundations/06-k6-Load-Test-Options.md), k6 sẽ chạy kịch bản của bạn một lần với một lần lặp (iteration) và một người dùng ảo. Nhưng điều gì sẽ xảy ra khi bạn thiết lập thời lượng kiểm thử là một phút?

```js
import { sleep } from "k6";

export const options = {
    duration: "1m",
};

export default function () {
    console.log("An iteration was executed");
    sleep(1);
}
```

Nếu bạn chạy bài kiểm tra trên, bạn sẽ nhận thấy các dòng sau trong đầu ra:

```plain
INFO[0000] An iteration was executed                     source=console
INFO[0001] An iteration was executed                     source=console
INFO[0002] An iteration was executed                     source=console
INFO[0003] An iteration was executed                     source=console
INFO[0004] An iteration was executed                     source=console
INFO[0005] An iteration was executed                     source=console
INFO[0006] An iteration was executed                     source=console
INFO[0007] An iteration was executed                     source=console
INFO[0008] An iteration was executed                     source=console
...
...
running (1m00.0s), 0/1 VUs, 60 complete and 0 interrupted iterations
```

Mỗi dòng này đại diện cho một lần lặp - một lần thực thi duy nhất của VU code.

Vì k6 thực thi lặp lại mọi thứ bên trong hàm mặc định, bạn có thể lập kế hoạch cho các kịch bản của mình một cách tương ứng. Ví dụ, không cần phải lặp lại mã nguồn bên trong hàm mặc định khi bạn đã biết rằng toàn bộ hàm sẽ được lặp lại trong suốt bài kiểm tra.

Tuy nhiên, việc đặt TẤT CẢ mã nguồn bên trong hàm mặc định cũng có thể gây bất lợi cho bài kiểm tra của bạn.

## Các vấn đề khi lặp lại tất cả mã nguồn

Việc lặp lại tất cả mã nguồn bên trong hàm mặc định có thể tiêu tốn tài nguyên một cách không cần thiết, đặc biệt nếu kịch bản của bạn bao gồm những việc sau:

- Tạo hoặc đọc từ các tệp dữ liệu lớn
- Chuẩn bị dữ liệu kiểm thử, chẳng hạn như tạo các tài khoản người dùng mà các bài kiểm tra sẽ sử dụng
- Khởi tạo các thư viện được nhập (imported libraries)

Một vấn đề khác có thể phát sinh từ việc đặt tất cả mã nguồn trong hàm mặc định có liên quan đến quy trình nghiệp vụ mà bạn đang cố gắng kiểm thử.

Nếu trọng tâm của kịch bản kiểm thử của bạn là một hành động yêu cầu đăng nhập, chẳng hạn như một trang cổng thông tin người dùng mới, bạn có thể không nhất thiết quan tâm đến việc lặp lại việc đăng nhập ở mỗi lần lặp. Trên thực tế, việc đăng nhập và đăng xuất lặp đi lặp lại có thể khiến các tài khoản người dùng kiểm thử bị khóa bởi các chính sách bảo mật của ứng dụng.

Bạn cũng có thể muốn thực thi một số mã nguồn chỉ sau khi toàn bộ bài kiểm tra kết thúc. Một trường hợp sử dụng phổ biến cho việc này là xóa mọi dữ liệu không cần thiết mà bài kiểm tra có thể đã tạo ra. Ví dụ: bạn có thể muốn xóa tất cả các mặt hàng còn lại trong giỏ hàng hoặc xóa một tài khoản. Việc quản lý dữ liệu kiểm thử không đầy đủ có thể làm suy giảm môi trường kiểm thử của bạn theo thời gian và dẫn đến kết quả kiểm thử tải sai lệch.

## Nên đặt mã nguồn nào ở đâu?

Trong k6, vị trí bạn đặt mã nguồn sẽ thay đổi thời điểm và tần suất mã nguồn đó được thực thi.

```js
// Đây là init code. Nó chạy một lần cho mỗi VU trước VU code, một lần trước setup và một lần trước teardown.

export function setup() {
    // Đây là setup code. Nó chạy một lần vào đầu bài kiểm tra, bất kể số lượng VUs.
}

export default function () {
    // Đây là VU code. Nó chạy lặp lại cho đến khi bài kiểm tra dừng lại.
}

export function teardown() {
    // Đây là teardown code. Nó chạy một lần vào cuối bài kiểm tra, bất kể số lượng VUs.
}
```

### Init

Mã nguồn trong phạm vi toàn cục (global scope) được gọi là init code, và nó được thực thi:

- Một lần khi mỗi VU được khởi tạo, trước bất kỳ thứ gì trong hàm mặc định
- Một lần trước hàm `setup()`
- Một lần trước hàm `teardown()`

Bạn có thể coi phần này là nơi thực hiện bất kỳ sự chuẩn bị nào bạn cần trước khi chạy lần lặp đầu tiên cho mỗi người dùng.

Dưới đây là một số thứ mà bạn có thể muốn đặt trong init code:

- Khai báo các biến mà bạn muốn sử dụng lại giữa các lần lặp của một VU duy nhất hoặc giữa các VUs (chẳng hạn như một [Shared Array](04-Adding-test-data.md#Shared-Array))
- Một đối tượng [k6 Load Test Options](../II-k6-Foundations/06-k6-Load-Test-Options.md) nơi bạn có thể thiết lập các tham số của bài kiểm tra

Không phải tất cả các tính năng của k6 đều có sẵn để sử dụng trong phần init; ví dụ: bạn không thể thực hiện các yêu cầu HTTP bên trong mã init.

### Setup

```js
export function setup() {
    // Đây là setup code. Nó chạy một lần vào đầu bài kiểm tra, bất kể số lượng VUs.
}
```

Hàm setup là một hàm hỗ trợ nơi bạn có thể đặt mã nguồn chỉ cần được thực thi một lần duy nhất trên tất cả các VUs, trước bất kỳ VU code nào. Nó được thực thi ngay sau mã init.

Dưới đây là một số điều mà bạn có thể muốn thực hiện trong quá trình setup:

- Kiểm tra xem môi trường kiểm thử có ở trạng thái mong đợi hay không
- Tạo dữ liệu kiểm thử mà nhiều VUs sẽ sử dụng (dữ liệu này sau đó có thể được chuyển sang các hàm khác)
- Thiết lập các mã phản hồi HTTP mong đợi cho phần còn lại của bài kiểm tra thông qua [`setResponseCallback()`](https://k6.io/docs/javascript-api/k6-http/setresponsecallback-callback/)

Bất cứ thứ gì được trả về trong hàm setup sẽ được lưu lại và có thể được chuyển sang các hàm default và teardown như sau:

Toàn bộ k6 API đều có sẵn trong quá trình setup, bao gồm cả việc thực hiện các yêu cầu HTTP.

### Teardown

```js
export function teardown() {
    // Đây là teardown code. Nó chạy một lần vào cuối bài kiểm tra, bất kể số lượng VUs.
}
```

Hàm teardown tương tự như hàm setup ở chỗ nó chỉ chạy một lần cho mỗi bài kiểm tra (không phải cho mỗi VU), nhưng nó chạy vào _cuối_ bài kiểm tra. Bất kỳ mã teardown nào cũng sẽ là mã cuối cùng được k6 thực thi trước khi kết quả kiểm thử được tạo ra.

Nếu hàm `setup()` kết thúc một cách bất thường, hàm `teardown()` sẽ không được gọi. Hãy cân nhắc thêm logic vào setup để dọn dẹp đúng cách nếu cần thiết.

Dưới đây là một số điều bạn có thể muốn thực hiện trong quá trình teardown:

- Đăng xuất bất kỳ người dùng nào đang hoạt động
- Xóa dữ liệu kiểm thử được tạo ra trong quá trình thực thi kiểm thử
- Đưa môi trường kiểm thử trở lại trạng thái như trước khi kiểm thử

Toàn bộ k6 API đều có sẵn trong quá trình teardown, bao gồm cả việc thực hiện các yêu cầu HTTP.

```js
export function setup() {
    return { v: 1 };
}

export default function (data) {
    console.log(JSON.stringify(data));
}

export function teardown(data) {
    if (data.v != 1) {
        throw new Error("incorrect data: " + JSON.stringify(data));
    }
}
```

Kiểm tra [tài liệu về vòng đời kiểm thử (test lifecycle)](https://k6.io/docs/using-k6/test-lifecycle/) để biết thêm chi tiết.

## Setup và teardown trong khi gỡ lỗi

Bạn có thể muốn bỏ qua setup và teardown trong một số tình huống, chẳng hạn như khi bạn đang gỡ lỗi kịch bản của mình hoặc chạy một bài kiểm tra sàng lọc nhỏ không yêu cầu tạo dữ liệu.

Bạn có thể bỏ qua các hàm này bằng CLI như sau:

```shell
k6 run test.js --no-setup --no-teardown
```

### Setup/teardown cho mỗi VU

Các hàm hỗ trợ setup và teardown chạy một lần cho mỗi BÀI KIỂM TRA (TEST), không phải cho mỗi VU. Điều gì sẽ xảy ra nếu bạn muốn chạy một mã nguồn nhất định vào đầu hoặc cuối mỗi VU?

#### Trước mỗi VU

Bạn có thể khai báo một biến trong mã init mà bạn có thể sử dụng để chỉ chạy mã nguồn ở lần lặp đầu tiên của một VU như sau:

```js
// Đây là init code. Nó chạy một lần cho mỗi VU trước VU code, một lần trước setup và một lần trước teardown.

let isFirstIteration = true;

export function setup() {
    // Đây là setup code. Nó chạy một lần vào đầu bài kiểm tra, bất kể số lượng VUs.
}

export default function () {
    if (isFirstIteration) {
        // Mã này chỉ chạy một lần cho mỗi VU, sau mã setup nhưng trước phần còn lại của VU code.
        isFirstIteration = false;
    }

    // Đây là VU code. Nó chạy lặp lại cho đến khi bài kiểm tra dừng lại.
}
```

Bạn có thể sử dụng cách tiếp cận này để chỉ đăng nhập bằng tài khoản người dùng một lần và sau đó lặp lại các hành động sau khi xác thực.

#### Sau mỗi VU

Nếu bạn biết chính xác số lần lặp mà bạn muốn bài kiểm tra của mình chạy, bạn có thể áp dụng cùng một cách tiếp cận liên quan đến biến trong mã init để chạy mã cho mỗi VU, ngay trước khi đi đến hàm teardown. Bạn cũng có thể bao gồm VU code có điều kiện dựa trên thời gian tương đối mà bài kiểm tra đã chạy.

Tuy nhiên, bạn không phải lúc nào cũng biết bài kiểm tra của mình sẽ chạy bao nhiêu lần lặp hoặc nó sẽ chạy trong bao lâu. Đối với những tình huống đó, lựa chọn tốt nhất là sử dụng hàm teardown thay thế.

Dưới đây là một ví dụ về cách điều này có thể hoạt động trong tình huống bạn muốn chọn một tài khoản người dùng ngẫu nhiên, đăng nhập một lần cho mỗi VU và sau đó đăng xuất tất cả các tài khoản người dùng:

```js
import { sleep } from "k6";
// Đây là init code. Nó chạy một lần cho mỗi VU trước VU code, một lần trước setup và một lần trước teardown.

let counter = 0;
let usernames = [
    "user1",
    "user2",
    "user3",
    "user4",
    "user5",
    "user6",
    "user7",
    "user8",
    "user9",
    "user10",
];
let passwords = [
    "password1",
    "password2",
    "password3",
    "password4",
    "password5",
    "password6",
    "password7",
    "password8",
    "password9",
    "password10",
];
let rand = Math.floor(Math.random() * usernames.length);
let username = usernames[rand];
let password = passwords[rand];
console.log(`VU#${__VU} - username: ${username}, password: ${password}`);

export function setup() {
    // Đây là setup code. Nó chạy một lần vào đầu bài kiểm tra, bất kể số lượng VUs.
}

export default function () {
    if (counter === 0) {
        // Mã này chỉ chạy một lần cho mỗi VU, sau mã setup nhưng trước phần còn lại của VU code.
        console.log(
            `This is the first iteration of VU#${__VU} using username ${username} and password ${password}`,
        );
    }
    counter++;

    // Đây là VU code. Nó chạy lặp lại cho đến khi bài kiểm tra dừng lại.
    console.log(
        `This is iteration #${counter} of VU#${__VU} using username ${username} and password ${password}`,
    );

    sleep(1);
}

export function teardown() {
    // Đây là teardown code. Nó chạy một lần vào cuối bài kiểm tra, bất kể số lượng VUs.

    for (let i = 0; i < usernames.length; i++) {
        console.log(
            `Check if logged in, and if so, log out username ${usernames[i]} and password ${passwords[i]}`,
        );
    }
}
```

## Kiểm tra kiến thức của bạn

### Câu hỏi 1

Hàm nào thường được thực thi nhiều lần nhất trong một bài kiểm thử tải đầy đủ?

A: Hàm mặc định (The default function)

B: Hàm setup (The setup function)

C: Hàm teardown (The teardown function)

### Câu hỏi 2

Một bài kiểm tra k6 chạy 100 VUs trong 30 phút. Giả sử kịch bản kiểm thử chứa một hàm setup, bạn mong đợi hàm setup được thực thi bao nhiêu lần?

A: 100

B: 1

C: 30

### Câu hỏi 3

Một kịch bản kiểm thử có các hàm setup, default và teardown và chạy với 1 VU cho 1 lần lặp. Bạn mong đợi mã init được thực thi bao nhiêu lần?

A: 1

B: 3

C: 5

### Đáp án

1. A. Các hàm setup và teardown được thực thi một cách có chủ đích chỉ một lần duy nhất cho mỗi bài kiểm tra (tương ứng vào đầu và cuối). Trong khi đó, hàm mặc định là hàm được thực thi lặp đi lặp lại.
2. B. Hàm setup chỉ được thực thi một lần duy nhất, vào đầu bài kiểm tra.
3. B. Mã init (mã trong phạm vi toàn cục) được thực thi một lần cho mỗi VU, một lần trước setup và một lần trước teardown.
