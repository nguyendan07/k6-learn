# Thêm dữ liệu kiểm thử (Adding test data)

Trong phần này, bạn sẽ học những cách tốt nhất để thêm dữ liệu kiểm thử vào kịch bản kiểm thử tải của mình, giúp nó trở nên động và thực tế hơn.

## Tại sao cần thêm dữ liệu kiểm thử?

Dữ liệu kiểm thử (Test data) là thông tin được thiết kế để sử dụng bởi một kịch bản kiểm thử trong quá trình thực thi. Thông tin này làm cho bài kiểm tra trở nên thực tế hơn bằng cách sử dụng các giá trị khác nhau cho các tham số nhất định. Việc gửi lặp đi lặp lại cùng một giá trị cho những thứ như tên người dùng và mật khẩu có thể khiến việc lưu bộ nhớ đệm (caching) xảy ra.

Một bộ nhớ đệm (cache) đại diện cho nội dung được lưu lại với mục tiêu tăng hiệu năng ứng dụng. Caching có thể xảy ra ở phía máy chủ (server side) hoặc phía máy khách (client side). Cache phía máy chủ lưu các tài nguyên thường xuyên được yêu cầu (chẳng hạn như các tài nguyên trên trang chủ) để khi các tài nguyên đó được yêu cầu, máy chủ có thể trả về ngay lập tức thay vì phải lấy chúng từ một máy chủ hạ nguồn.

Cache phía máy khách có thể lưu cùng các tài nguyên đó để trình duyệt của người dùng biết rằng nó không cần thực hiện yêu cầu cho các tài nguyên đó trừ khi chúng đã bị thay đổi. Việc triển khai này làm giảm số lượng yêu cầu mạng cần gửi và đặc biệt quan trọng đối với các ứng dụng chủ yếu được truy cập qua thiết bị di động.

Caching có thể ảnh hưởng đáng kể đến kết quả kiểm thử tải. Có những tình huống nên bật caching (chẳng hạn như khi cố gắng mô phỏng hành vi của khách hàng cũ), nhưng cũng có những tình huống nên tắt caching (chẳng hạn như khi mô phỏng những người dùng hoàn toàn mới). Dù bằng cách nào, bạn nên quyết định chiến lược caching của mình một cách có chủ đích và viết kịch bản tương ứng.

Thêm dữ liệu kiểm thử có thể giúp ngăn chặn việc lưu cache phía máy chủ. Dữ liệu kiểm thử phổ biến bao gồm:

- Tên người dùng và mật khẩu, để đăng nhập vào ứng dụng và thực hiện các hành động cần xác thực
- Tên, địa chỉ, email và các thông tin cá nhân khác để đăng ký tài khoản hoặc điền vào các biểu mẫu liên hệ
- Tên sản phẩm để truy xuất các trang sản phẩm
- Từ khóa để tìm kiếm trong danh mục
- Các tệp PDF để kiểm thử tính năng tải lên

## Mảng (Array)

Cách đơn giản nhất để thêm dữ liệu kiểm thử là sử dụng một mảng. Trong phần [Dynamic correlation in k6](02-Dynamic-correlation-in-k6.md), bạn đã định nghĩa một mảng như thế này:

```js
let usernameArr = ["admin", "test_user"];
let passwordArr = ["123", "1234"];
```

Sau khi định nghĩa các mảng, bạn có thể tạo một số ngẫu nhiên để chọn ngẫu nhiên một giá trị từ mảng:

```js
// Get random username and password from array
let rand = Math.floor(Math.random() * usernameArr.length);
let username = usernameArr[rand];
let password = passwordArr[rand];
console.log("username: " + username, " / password: " + password);
```

Mảng được sử dụng tốt nhất cho các danh sách văn bản rất ngắn, như trong ví dụ này, hoặc để gỡ lỗi một kịch bản kiểm thử.

## Các tệp CSV

Các tệp CSV là danh sách thông tin được tạo thành từ các _giá trị phân cách bằng dấu phẩy_ (comma-separated values) và được lưu riêng biệt với kịch bản.

Dưới đây là ví dụ về một tệp CSV tên là `users.csv` chứa tên người dùng và mật khẩu:

```plain
username,password
admin,123
test_user,1234
...
```

Trong k6, các tệp CSV sau đó có thể được thêm vào kịch bản như thế này:

```js
import papaparse from "https://jslib.k6.io/papaparse/5.1.1/index.js";

const csvData = papaparse.parse(open("users.csv"), { header: true }).data;

export default function () {
    let rand = Math.floor(Math.random() * csvData.length);
    console.log(
        "username: ",
        csvData[rand].username,
        " / password: ",
        csvData[rand].password,
    );
}
```

Mã nguồn trên nhập một thư viện tên là `papaparse` cho phép bạn tương tác với các tệp CSV. Các tệp CSV được sử dụng tốt nhất khi tạo tên người dùng và tài khoản mới hoặc khi bạn muốn có thể mở tệp dữ liệu kiểm thử trong một ứng dụng bảng tính.

> k6 không cho phép bạn đặt mã nguồn đọc từ hệ thống tệp cục bộ bên trong hàm mặc định (default function). Hạn chế này thực thi quy tắc tốt nhất là đặt các lệnh đọc tệp trong ngữ cảnh khởi tạo (init context - bên ngoài hàm mặc định). Bạn có thể đọc thêm về điều đó trong [Vòng đời kiểm thử (Test life cycle)](https://k6.io/docs/using-k6/test-life-cycle/).

## Các tệp JSON

Bạn cũng có thể lưu trữ dữ liệu kiểm thử của mình trong các tệp JSON.

Dưới đây là một tệp JSON tên là `users.json`:

```plain
{
  "users": [
    { "username": "admin", "password": "123" },
    { "username": "test_user", "password": "1234" }
  ]
}
```

Sau đó, bạn có thể sử dụng nó trong kịch bản k6 của mình như thế này:

```js
const jsonData = JSON.parse(open("./users.json")).users;

export default function () {
    let rand = Math.floor(Math.random() * jsonData.length);
    console.log(
        "username: ",
        jsonData[rand].username,
        " / password: ",
        jsonData[rand].password,
    );
}
```

Các tệp JSON được sử dụng tốt nhất khi thông tin được sử dụng cho bài kiểm tra đã được xuất dưới định dạng JSON, hoặc khi dữ liệu có tính phân cấp cao hơn mức mà một tệp CSV có thể xử lý.

## Mảng dùng chung (Shared Array)

Một Shared Array kết hợp một số yếu tố của ba phương pháp trước đó (mảng đơn giản, tệp CSV và tệp JSON) trong khi giải quyết một vấn đề phổ biến với dữ liệu kiểm thử trong quá trình thực thi: sử dụng tài nguyên cao.

Trong các phương pháp trước, khi một tệp được sử dụng trong một bài kiểm tra, nhiều bản sao của tệp đó được tạo ra và gửi đến trình tạo tải. Khi tệp rất lớn, điều này có thể tiêu tốn tài nguyên trên trình tạo tải một cách không cần thiết, làm cho kết quả kiểm thử kém chính xác hơn.

Để ngăn chặn điều này, hãy sử dụng một `SharedArray`:

```js
import papaparse from "https://jslib.k6.io/papaparse/5.1.1/index.js";
import { SharedArray } from "k6/data";

const sharedData = new SharedArray("Shared Logins", function () {
    let data = papaparse.parse(open("users.csv"), { header: true }).data;
    return data;
});

export default function () {
    let rand = Math.floor(Math.random() * sharedData.length);
    console.log(
        "username: ",
        sharedData[rand].username,
        " / password: ",
        sharedData[rand].password,
    );
}
```

Lưu ý rằng SharedArray phải được kết hợp với các phương pháp khác—trong trường hợp này là một tệp CSV.

Sử dụng SharedArray là cách hiệu quả nhất để thêm một danh sách dữ liệu bên trong một bài kiểm tra k6.

## Các dữ liệu kiểm thử khác

Dữ liệu kiểm thử cũng có thể đề cập đến các tệp cần được tải lên như một phần của kịch bản đang được kiểm thử. Ví dụ, một hình ảnh có thể cần được tải lên để mô phỏng một người dùng tải lên ảnh hồ sơ của họ. Để biết thêm thông tin về việc viết kịch bản cho các loại tải lên dữ liệu này, [hãy kiểm tra tài liệu tại đây.](https://k6.io/docs/examples/data-uploads/)

## Kiểm tra kiến thức của bạn

### Câu hỏi 1

Trong tình huống nào sau đây thì việc thêm dữ liệu kiểm thử vào kịch bản kiểm thử tải của bạn là điều nên làm?

A: Ứng dụng của bạn khóa tài khoản người dùng sau ba lần đăng nhập trong một khoảng thời gian ngắn.

B: Bạn muốn xem ứng dụng hành xử thế nào khi cùng một người dùng tải lại một trang liên tục.

C: Cả A và B.

### Câu hỏi 2

Bạn có một tệp CSV với 100 MB thông tin cá nhân mà bạn muốn sử dụng làm dữ liệu kiểm thử. Phương pháp nào sau đây là tốt nhất để sử dụng?

A: SharedArray

B: Mảng đơn giản (Simple array)

C: Tệp JSON vì CSV được chuyển đổi sang JSON tốt hơn

### Câu hỏi 3

Tất cả các ví dụ trên trang này đều bao gồm hàm `Math.random()` để chọn ngẫu nhiên một phần tử từ tệp dữ liệu. Trong tình huống nào bạn có thể muốn loại bỏ sự ngẫu nhiên này?

A: Khi bạn muốn ngăn chặn việc lưu bộ nhớ đệm phía máy chủ.

B: Khi bạn muốn đảm bảo rằng mỗi phần tử của dữ liệu kiểm thử đã được kịch bản sử dụng một cách tuần tự.

C: Khi bạn muốn làm cho các bài kiểm tra của mình thực tế nhất có thể.

### Đáp án

1. A. Bạn có thể giải quyết tình huống mô tả trong A bằng cách thêm dữ liệu kiểm thử của các thông tin đăng nhập khác nhau mà kịch bản có thể sử dụng. B sai, vì thực tế đó là một ví dụ điển hình về trường hợp sử dụng mà việc _không_ bao gồm dữ liệu kiểm thử có thể là lựa chọn tốt hơn.
2. A. Các tệp dữ liệu rất lớn có thể gây ảnh hưởng đến kết quả kiểm thử tải khi chúng được sao chép và chuyển giao lặp đi lặp lại cho mọi trình tạo tải. SharedArray là cách tốt hơn để xử lý những trường hợp này, mặc dù cũng có thể đáng cân nhắc việc lưu trữ dữ liệu kiểm thử trong một cơ sở dữ liệu.
3. B. Việc chọn ngẫu nhiên dữ liệu kiểm thử có thể ngăn chặn việc lưu bộ nhớ đệm và làm cho bài kiểm tra thực tế hơn. Tuy nhiên, việc chọn ngẫu nhiên cũng có thể khiến việc xác định dữ liệu kiểm thử nào đã được bài kiểm tra sử dụng trở nên khó khăn hơn vì tệp dữ liệu không được phân tích tuần tự.
