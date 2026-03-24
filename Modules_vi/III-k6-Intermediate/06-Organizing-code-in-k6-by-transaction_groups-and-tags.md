# Tổ chức mã nguồn trong k6 theo transaction groups và tags

Nếu bạn đã từng viết kịch bản kiểm thử tải (load testing) trước đây, bạn có thể đã quen thuộc với thuật ngữ transaction (giao dịch). Một transaction trong kiểm thử tải là một yêu cầu hoặc một nhóm các yêu cầu tương ứng với một hành động duy nhất của người dùng.

Ví dụ, bạn có thể có một transaction tên là `01_Go_to_homepage` bao gồm việc người dùng truy cập vào trang chủ trang web của bạn lần đầu tiên. Transaction đó sau đó sẽ bao gồm các yêu cầu GET cho mã HTML chính, cùng với các tài nguyên nhúng như scripts, hình ảnh và phông chữ.

Các transaction có thể hữu ích trong việc định hình lại hiệu năng (performance) dưới dạng trải nghiệm người dùng. Trong ví dụ trên, người dùng cuối của ứng dụng có thể không nhất thiết quan tâm hoặc thậm chí không nhận thấy rằng `https://yourdomain.com/script.js` có thời gian phản hồi là 300ms. Các nhận xét của người dùng về một trang web thường được thể hiện dưới dạng các trang, vì vậy việc cân nhắc xem kịch bản của bạn có nên được cấu trúc tương tự hay không là điều đáng giá. Việc báo cáo thời gian phản hồi của một trang thay vì các yêu cầu riêng lẻ khi trao đổi với các bên liên quan cũng có thể dễ dàng hơn.

Các transaction cũng giúp tổ chức mã kiểm thử, làm cho nó dễ đọc hơn và dễ gỡ lỗi (debug) hơn. Có một số cách để bạn có thể tạo các transaction trong k6.

## Groups

Với [groups](https://k6.io/docs/using-k6/tags-and-groups/#groups), bạn có thể xác định nhiều yêu cầu thuộc về cùng một transaction. k6 sẽ hiển thị thời gian phản hồi của các yêu cầu được nhóm riêng biệt và dưới dạng một nhóm. Groups được sử dụng tốt nhất cho các yêu cầu được kích hoạt bởi cùng một hành động. Dưới đây là một ví dụ về cách bạn có thể sử dụng các nhóm để cấu trúc kịch bản của mình:

```js
import { group } from "k6";

export default function () {
    group("01_VisitHomepage", function () {
        // ...
    });
    group("02_ClickOnProduct", function () {
        // ...
    });
    group("03_AddProductToCart", function () {
        // ...
    });
    group("04_ViewCart", function () {
        // ...
    });
    group("05_ProceedToCheckout", function () {
        // ...
    });
}
```

Khi bạn tạo một kịch bản [sử dụng k6 browser recorder](../II-k6-Foundations/09-Recording-a-k6-script.md), bạn có thể nhận thấy rằng các groups này được tự động tạo cho bạn bất cứ khi nào k6 phát hiện nhiều tài nguyên nhúng trên một trang.

Khi bạn sử dụng groups, k6 vẫn báo cáo và lưu các chỉ số cho từng yêu cầu riêng lẻ, nhưng mỗi yêu cầu cũng bao gồm tên nhóm của nó. Nếu bạn [lưu kết quả đầu ra vào tệp CSV](../II-k6-Foundations/08-k6-results-output-options.md#CSV) bằng cách sử dụng `k6 run test.js -o csv=results.csv`, bạn sẽ thấy rằng nhóm của mỗi yêu cầu cũng được báo cáo:

```plain
metric_name,timestamp,metric_value,check,error,error_code,expected_response,group,method,name,proto,scenario,service,status,subproto,tls_version,url,extra_tags
vus,1645006049,1.000000,,,,,,,,,,,,,,,
vus_max,1645006049,1.000000,,,,,,,,,,,,,,,
http_reqs,1645006050,1.000000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,
http_req_duration,1645006050,1530.111000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,
http_req_blocked,1645006050,134.352000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,
http_req_connecting,1645006050,132.192000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,
http_req_tls_handshaking,1645006050,0.000000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,
http_req_sending,1645006050,0.141000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,
http_req_waiting,1645006050,1120.729000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,
http_req_receiving,1645006050,409.241000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,
http_req_failed,1645006050,0.000000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,
group_duration,1645006069,20792.450912,,,,,::01_Homepage,,,,default,,,,,,
```

Giá trị `::01_Homepage` trong tệp CSV ở trên đề cập đến tên nhóm của yêu cầu, với dấu `::` biểu thị rằng đó là một nhóm thay vì một yêu cầu.

Dòng cuối cùng cũng cho thấy k6 đã lưu một chỉ số `http_req_duration` tổng hợp cho cả nhóm, được gọi là `group_duration`. Mỗi nhóm được chỉ định sẽ có `group_duration` riêng của nó.

Vì các nhóm _thêm_ các chỉ số thay vì thay thế chúng, chúng có thể là một cách tuyệt vời để tổ chức mã nguồn của bạn thành các transaction.

## Tags

Một [tag](https://k6.io/docs/using-k6/tags-and-groups/#tags) là một cách để gắn nhãn các phần tử trong k6, và các tag có thể được sử dụng kết hợp với các groups. Không giống như groups, các tag sẽ không khiến k6 báo cáo các chỉ số theo từng tag.

Dưới đây là cách để gắn tag cho một yêu cầu:

```js
import { sleep, group } from "k6";
import http from "k6/http";

export default function () {
    let response;

    group("01_Homepage", function () {
        response = http.get("http://ecommerce.k6.io/", {
            tags: {
                page: "Homepage",
                type: "HTML",
            },
        });

        // ... (các yêu cầu khác trong nhóm)

        // ... (các yêu cầu khác trong nhóm)
    });
}
```

Phần `tags`, được thêm vào yêu cầu, định nghĩa hai tag khác nhau:

- một tag tên là `page`, có giá trị cho yêu cầu này là `Homepage`, và
- một tag tên là `type`, có giá trị cho yêu cầu này là `HTML`

Bạn có thể thay đổi cả tên và giá trị của tag theo cách bạn thấy phù hợp, và bạn có thể sử dụng nhiều tag.

Theo mặc định, k6 thêm một tag `name` cho mỗi yêu cầu, với giá trị bằng URL. Bạn có thể ghi đè hành vi này nếu bạn muốn đặt tên cho yêu cầu một cái gì đó dễ đọc hơn.

Dưới đây là giao diện của các tag trong kết quả CSV của k6:

```plain
metric_name,timestamp,metric_value,check,error,error_code,expected_response,group,method,name,proto,scenario,service,status,subproto,tls_version,url,extra_tags
vus,1645007331,1.000000,,,,,,,,,,,,,,,
vus_max,1645007331,1.000000,,,,,,,,,,,,,,,
http_reqs,1645007331,1.000000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,page=Homepage&type=HTML
http_req_duration,1645007331,1405.515000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,page=Homepage&type=HTML
http_req_blocked,1645007331,139.488000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,page=Homepage&type=HTML
http_req_connecting,1645007331,137.223000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,page=Homepage&type=HTML
http_req_tls_handshaking,1645007331,0.000000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,type=HTML&page=Homepage
http_req_sending,1645007331,0.137000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,type=HTML&page=Homepage
http_req_waiting,1645007331,1171.681000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,page=Homepage&type=HTML
http_req_receiving,1645007331,233.697000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,page=Homepage&type=HTML
http_req_failed,1645007331,0.000000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,page=Homepage&type=HTML
```

Trường cuối cùng của mọi chỉ số cho yêu cầu này bây giờ có `page=Homepage&type=HTML`, tương ứng với các tag đã được thiết lập.

Vì k6 không báo cáo một chỉ số tổng hợp cho tất cả các yêu cầu được gắn tag, giống như cách nó thực hiện với các yêu cầu được nhóm, nên các groups tốt hơn để sử dụng làm các transaction. Tuy nhiên, các tag có thể hữu ích trong các trường hợp sau:

- Bạn muốn thêm siêu dữ liệu (metadata) vào các yêu cầu để gỡ lỗi dễ dàng hơn (chẳng hạn như bằng cách thêm loại kịch bản hoặc dữ liệu kiểm thử được sử dụng bởi yêu cầu đó).
- Bạn muốn khám phá các mô hình trải dài trên nhiều trang, chẳng hạn như tổng thời gian phản hồi mà tất cả các hình ảnh đóng góp vào tổng số là bao nhiêu.
- Bạn muốn gắn nhãn nhiều thứ hơn là chỉ các yêu cầu.

Với các tag, bạn có thể gắn nhãn cho:

- các yêu cầu (requests)
- các phép kiểm tra (checks)
- các ngưỡng (thresholds)
- các chỉ số tùy chỉnh (custom metrics)
- tất cả các chỉ số trong một bài kiểm tra

Để biết thông tin chi tiết hơn về các tag, [hãy nhấp vào đây](https://k6.io/docs/using-k6/tags-and-groups/#tags).

## URL grouping

Như đã đề cập trước đó, k6 thêm một tag `name` cho tất cả các yêu cầu theo mặc định và đặt giá trị là URL yêu cầu. Điều này hữu ích khi mỗi URL đại diện cho một trang khác nhau. Tuy nhiên, đôi khi bạn có thể quyết định rằng kịch bản kiểm thử của mình cần bao gồm các trang tương tự nhau để được phân tích như một. Ví dụ: nếu bạn đang kiểm thử một trang web thương mại điện tử, có thể sẽ hữu ích hơn nếu bạn báo cáo tất cả các trang sản phẩm dưới dạng một, thay vì các sản phẩm riêng biệt.

[URL grouping](https://k6.io/docs/using-k6/http-requests/#url-grouping) là một cách triển khai các tag đặc biệt hữu ích khi bạn đang tạo các URL động, như thế này:

```js
import http from "k6/http";

export default function () {
    let product = ["album", "beanie", "beanie-with-logo"];
    let rand = Math.floor(Math.random() * product.length);
    let productSelected = product[rand];
    let response = http.get(
        "http://ecommerce.test.k6.io/product/" + productSelected,
        {
            tags: { name: "ProductPage" },
        },
    );
}
```

Trong ví dụ trên, kịch bản truy cập ngẫu nhiên một trong ba trang sản phẩm này:

- `http://ecommerce.test.k6.io/product/album`
- `http://ecommerce.test.k6.io/product/beanie`,
- `http://ecommerce.test.k6.io/product/beanie-with-logo`

Nếu không có URL grouping, mỗi trang này sẽ được gắn tag với một tên khác nhau (bằng với URL của chúng). _Với_ URL grouping, các trang này được gắn tag cùng một tên, `ProductPage`. Dưới đây là một dòng từ kết quả CSV thu được bằng cách chạy kịch bản:

```plain
http_req_duration,1645010817,1203.076000,,,,true,,GET,ProductPage,HTTP/1.1,default,,301,,,http://ecommerce.test.k6.io/product/beanie,
```

## Cải thiện khả năng đọc của kịch bản

Sử dụng các groups và tags (bao gồm cả URL grouping) là tốt để tổ chức kết quả, nhưng chúng chỉ làm tăng thêm độ phức tạp cho kịch bản của bạn. Dưới đây là một số điều bạn có thể làm để cải thiện khả năng đọc của kịch bản:

- Sử dụng các bình luận (comments)
- Sử dụng các hàm (functions)
- Mô-đun hóa (modularize) kịch bản của bạn

### Comments

Việc để lại bình luận cho mọi hành động trong kịch bản của bạn có lợi thế kép là phân biệt mã nguồn về mặt thị giác và cũng giải thích (các) hành động nào tương ứng với mã nguồn đó. Dưới đây là một ví dụ:

```js
import http from "k6/http";

export default function () {
    /************
       BƯỚC 1: Truy cập trang chủ.
    ************/
    // Mã nguồn cho Bước 1.
    /************
       BƯỚC 2: Nhấp vào sản phẩm để đi đến trang sản phẩm.
    ************/
    // Mã nguồn cho Bước 2.
    /************
       BƯỚC 3: Nhấp vào nút Thêm vào giỏ hàng trên trang sản phẩm.
    ************/
    // Mã nguồn cho Bước 3.
}
```

### Functions

Một cách khác để làm cho kịch bản của bạn dễ đọc là tạo một hàm cho mỗi transaction, sau đó gọi từng hàm trong hàm mặc định, như thế này:

```js
import http from "k6/http";

export default function () {
    Homepage();
    ProductPage();
    CartPage();
    CheckoutPage();
}

export function Homepage() {
    // ...
}

export function ProductPage() {
    // ...
}

export function CartPage() {
    // ...
}

export function CheckoutPage() {
    // ...
}
```

Sử dụng các hàm theo cách này mang lại cho bạn lợi ích bổ sung là có thể xác định nhiều luồng người dùng (user flows) trong cùng một kịch bản. Ví dụ: bạn có thể xác định một luồng người dùng chỉ truy cập trang chủ, một luồng khác truy cập trang chủ và một vài trang sản phẩm, và luồng thứ ba đi qua tất cả các trang và thanh toán. Bạn sẽ tìm hiểu thêm về các kịch bản trong phần [Mô hình hóa khối lượng công việc với các kịch bản](09-Workload-modeling-with-scenarios.md).

### Mô-đun hóa kịch bản của bạn sâu hơn

Điều gì sẽ xảy ra nếu bạn đang sử dụng tất cả các groups, tags (bao gồm cả URL grouping), comments, functions mà kịch bản của bạn _vẫn_ quá dài và khó đọc? [Modular scripting](../XX-Future-Ideas/Modular-scripting.md) có thể là câu trả lời cho vấn đề của bạn.

Modular scripting bao gồm việc chia nhỏ kịch bản kiểm thử k6 của bạn thành nhiều kịch bản, sau đó gọi từng kịch bản vào một kịch bản "runner" duy nhất để điều phối tất cả. Bạn sẽ tìm hiểu thêm về việc xây dựng một khung làm việc theo mô-đun để chạy các bài kiểm tra k6 trong phần [Modular scripting](../XX-Future-Ideas/Modular-scripting.md).

## Kiểm tra kiến thức của bạn

### Câu hỏi 1

Kỹ thuật nào là tốt nhất để sử dụng như một transaction trong kiểm thử tải?

A: Groups

B: Tags

C: URL grouping

### Câu hỏi 2

Hãy xem xét đoạn mã sau:

```js
import { sleep, group } from "k6";
import http from "k6/http";

export default function () {
    let response;

    group("01_Homepage", function () {
        response = http.get("http://ecommerce.k6.io/", {
            tags: {
                page: "Homepage",
                type: "HTML",
            },
        });

        // ... (các yêu cầu khác trong nhóm)

        // ... (các yêu cầu khác trong nhóm)
    });
}
```

Giá trị của tag `name` cho yêu cầu HTTP là gì?

A: `Homepage`

B: `HTML`

C: `http://ecommerce.k6.io/`

### Câu hỏi 3

Điều nào sau đây là một lợi thế của việc sử dụng các hàm để tổ chức kịch bản kiểm thử k6 của bạn?

A: Các hàm tổng hợp tất cả các chỉ số trong một hàm và báo cáo chúng dưới một tên duy nhất.

B: Các hàm mang lại cho bạn sự tự do để tạo ra các kịch bản kiểm thử tương ứng với hành trình của người dùng (user journeys) cho ứng dụng của bạn.

C: Các hàm gắn tag cho tất cả các yêu cầu bằng tên hàm để bạn có thể lọc theo hàm trong kết quả kiểm thử.

### Đáp án

1. A. Việc nhóm các yêu cầu sẽ lưu các chỉ số cho mỗi yêu cầu _và_ cho mỗi nhóm (không giống như tags và URL groups).
2. C. Nếu không có một `name` cụ thể được chỉ định, k6 sẽ mặc định sử dụng URL làm tên, để bạn có thể xác định yêu cầu dễ dàng hơn.
3. B. Các hàm không tổng hợp các chỉ số (groups làm việc đó tốt hơn) hoặc gắn tag cho các yêu cầu bên trong chúng (bạn cần thêm các tag để làm việc đó). Thay vào đó, chúng được sử dụng tốt hơn để tổ chức các hành động của người dùng hoặc luồng người dùng trong một kịch bản.
