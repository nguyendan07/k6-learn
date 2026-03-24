Bạn đã học cách [viết kịch bản k6 đầu tiên](01-Getting-started-with-k6-OSS.md). Tại sao bạn lại muốn _ghi lại_ (record) một kịch bản?

Tự mình viết kịch bản k6 là cách tốt nhất để tìm hiểu cách k6 hoạt động, và những kỹ năng bạn đã học trong các phần trước sẽ giúp bạn trở thành một người kiểm thử giỏi hơn vì ngay cả các kịch bản được ghi lại cũng cần được sửa đổi. Viết kịch bản k6 thủ công vẫn có thể là lựa chọn tốt hơn trong các tình huống sau:

- Bạn muốn kịch bản của mình thực hiện một chuỗi các cuộc gọi API.
- Bài kiểm tra của bạn nhắm vào một thành phần hoặc nhóm thành phần cụ thể, thay vì tạo tải từ phía máy khách (client side).
- Các endpoint của API bạn muốn kiểm thử đã được tài liệu hóa tốt, hoặc bạn đã có kiến thức trước đó về chúng.
- Luồng (flow) bạn đang mô phỏng không thể dễ dàng mô phỏng từ trình duyệt.

Tuy nhiên, cũng có những tình huống mà việc ghi lại một kịch bản có thể tiết kiệm thời gian:

- Bạn đang kiểm thử một trang web và muốn bao gồm các yêu cầu cho các tài nguyên nhúng trên trang (scripts và hình ảnh), thay vì chỉ có mã HTML.
- Ứng dụng liên quan đến rất nhiều [tương quan động (dynamic correlation)](../III-k6-Intermediate/02-Dynamic-correlation-in-k6.md), và bạn muốn hầu hết công việc đó được thực hiện tự động cho mình.
- Bạn không chắc chắn các yêu cầu HTTP bên dưới là gì, hoặc API không được tài liệu hóa tốt.

Trong phần này, bạn sẽ học cách ghi lại một kịch bản k6.

## Tạo tài khoản k6 Cloud

Bạn có biết rằng bạn có thể sử dụng các tính năng ghi kịch bản của k6 Cloud miễn phí không?

Mặc dù việc ghi lại một kịch bản k6 yêu cầu tài khoản k6 Cloud, nhưng nó không yêu cầu đăng ký trả phí và bạn không cần nhập chi tiết thẻ tín dụng. Để bắt đầu, hãy [đăng ký một tài khoản](https://app.k6.io/). Hãy đảm bảo bạn đã đăng nhập trước khi tiếp tục.

Tiếp theo, hãy tải xuống và cài đặt tiện ích mở rộng ghi trình duyệt k6 cho trình duyệt của bạn-- [Chrome](https://chrome.google.com/webstore/detail/k6-browser-recorder/phjdhndljphphehjpgbmpocddnnmdbda?hl=en) và [Firefox](https://addons.mozilla.org/en-US/firefox/addon/k6-browser-recorder/) đều được hỗ trợ. Khi bạn nhìn thấy biểu tượng k6 trên thanh tiện ích mở rộng, bạn đã sẵn sàng để bắt đầu ghi.

![](../../images/browser-extension-icon.png)

Sau đó, hãy mở một tab mới và nhấp vào biểu tượng tiện ích mở rộng trình duyệt k6. Bạn sẽ thấy một hộp thoại với một số tùy chọn ban đầu:

![k6-browser-recorder-01](../../images/k6-browser-recorder-01.png)
Dưới đây là ý nghĩa của các tùy chọn đó:

- **Download HAR file**: HAR là viết tắt của HTTP ARchive, đây là một tệp định dạng JSON lưu thông tin mạng như các yêu cầu HTTP và thời gian. **Bật tùy chọn này nếu bạn muốn lưu bản ghi ở một nơi khác ngoài k6**.
- **Clear cache (last 7 days):** Bật tùy chọn này sẽ xóa bộ nhớ đệm trình duyệt của bạn trong tuần qua. Điều này hữu ích nếu bạn muốn mô phỏng một người dùng mới đối với ứng dụng của mình. **Tắt tùy chọn này nếu bạn muốn mô phỏng một người dùng hiện tại.**
- Đọc thêm về [các tùy chọn lưu bộ nhớ đệm (caching options)](../XX-Future-Ideas/Caching-options.md).
- **Correlate request/response data:** Trình ghi k6 có thể tự động phát hiện khi các giá trị động đang được truyền đến máy chủ ứng dụng và cố gắng tương quan chúng cho bạn. Điều này không phải lúc nào cũng hoạt động đối với các ứng dụng phức tạp, nhưng nó có thể là một điểm bắt đầu tốt để viết kịch bản. **Bật tùy chọn này trừ khi bạn muốn có một bản ghi thô (raw recording)**.

Tiếp theo, nhấp vào nút _Start recording_. Có thể mất vài giây để bộ nhớ đệm trình duyệt của tuần trước bị xóa. Hãy đợi cho đến khi bạn thấy màn hình sau:

![k6-browser-recorder-02](../../images/k6-browser-recorder-02.png)

Bây giờ bạn đã sẵn sàng để bắt đầu ghi! Hãy điều hướng đến ứng dụng web của bạn và tương tác với nó theo cách mà một trong những người dùng của bạn sẽ làm.

### Mẹo khi ghi hình (recording)

Để tận dụng tốt nhất bản ghi của bạn, hãy ghi nhớ các mẹo sau:

- Nếu bạn muốn biết thời gian [sleep](05-Adding-think-time-using-sleep.md) nên là bao lâu, hãy thử điền vào các biểu mẫu và đọc văn bản theo cách mà một người dùng mới sẽ làm. Thời gian sleep được ghi lại bởi tiện ích mở rộng.
- Chờ tất cả các tài nguyên của trang tải xong trước khi bạn tiếp tục sang trang hoặc hành động tiếp theo. Nếu bạn muốn chắc chắn, hãy thử mở DevTools của trình duyệt và đợi cho đến khi không còn hoạt động nào trong tab Network.
- Ghi chú riêng về các hành động bạn đang làm, để giúp bạn hiểu kịch bản được tạo ra sau này.
- Ghi lại từng luồng nghiệp vụ (business flow) một. Nếu bạn ghi lại nhiều luồng dài hoặc phức tạp, kịch bản của bạn có thể trở nên quá lớn và khó quản lý. Việc kết hợp nhiều kịch bản sẽ dễ dàng hơn là chia nhỏ chúng sau này.

## Lưu kịch bản đã ghi vào k6 Cloud

Khi bạn hoàn tất, hãy nhấp vào _Stop_ trên trình ghi trình duyệt. Bạn sẽ tự động được chuyển hướng đến tài khoản k6 Cloud của mình:

![](../../images/k6-browser-recorder-3.png)

![](../../images/k6-browser-recorder-4.png)

Hãy cùng xem qua các tùy chọn!

- **Project:** Trong menu thả xuống này, bạn có thể chọn dự án k6 Cloud nào bạn muốn lưu bản ghi này vào. Nếu bạn chưa tạo bất kỳ dự án nào, bạn sẽ thấy dự án _Default_ được chọn.
- **Title:** Bên cạnh menu thả xuống chọn dự án, hãy đặt cho bản ghi của bạn một cái tên mô tả để giúp bạn nhớ bản ghi này là gì sau này.
- **Test Builder hoặc Script Editor:** Quyết định xem bạn muốn xem luồng đã ghi của mình trong Test Builder (giao diện đồ họa) hay Script Editor (JavaScript thuần túy). Bạn sẽ tìm hiểu thêm về các tùy chọn này sau. Hiện tại, hãy chọn Test Builder nếu bạn cảm thấy thoải mái hơn khi tương tác với giao diện, và Script Editor nếu bạn muốn xem mã nguồn. Nếu bạn không chắc chắn, hãy chọn Test Builder và bạn sẽ có thể chuyển sang Script Editor sau nếu bạn đổi ý.
- **Correlate request and response data:** Đánh dấu tùy chọn này nếu bạn muốn k6 phát hiện và tương quan dữ liệu động cho bạn.
- **Include static assets:** Đánh dấu tùy chọn này nếu bạn muốn kịch bản kiểm thử của mình bao gồm các yêu cầu cho các tài nguyên nhúng như scripts, fonts, và hình ảnh. Đánh dấu tùy chọn này giúp kịch bản trở nên [thực tế hơn](../III-k6-Intermediate/03-Workload-modeling.md) nhưng phải đánh đổi bằng việc có nhiều yêu cầu hơn.
- **Generate sleep**: Đánh dấu tùy chọn này nếu bạn muốn k6 [thêm think time](05-Adding-think-time-using-sleep.md) dựa trên sự chậm trễ thực tế của bạn trong khi ghi.
- **Third-party domains filtering:** Theo mặc định, k6 bỏ qua các yêu cầu được gửi đến các tên miền khác với tên miền bạn điều hướng ban đầu. Làm như vậy sẽ ngăn bạn vô tình kiểm thử tải các máy chủ không thuộc sở hữu của bạn. Hãy xem qua các tên miền được liệt kê và đánh dấu bất kỳ tên miền nào bạn muốn bao gồm trong bài kiểm tra. _Lưu ý: Việc kiểm thử các máy chủ bạn không sở hữu có thể dẫn đến các rắc rối về pháp lý hoặc tài chính, vì vậy chúng tôi khuyên bạn chỉ nên kiểm thử các máy chủ bạn sở hữu._

Sau đó, nhấp vào Save!

Nếu bạn chuyển sang Script Editor từ Test Builder, kịch bản của bạn sẽ được hiển thị ở chế độ _chỉ đọc_ (read only):

![](../../images/k6-cloud-script-editor-from-recording.png)

Nhấp vào _COPY SCRIPT_, như được đánh dấu màu cam trong ảnh chụp màn hình, để chọn và sao chép toàn bộ kịch bản. Sau đó, bạn có thể dán kịch bản vào IDE của mình và [chạy kịch bản cục bộ](01-Getting-started-with-k6-OSS.md#Hello-World-running-your-k6-script).

Nếu thay vào đó, bạn đã chọn tùy chọn _Script Editor_ trong hộp thoại ghi trước đó, bạn sẽ thấy kịch bản kiểm thử của mình được hiển thị trong k6 Cloud ở chế độ có thể chỉnh sửa.

![](../../images/k6-cloud-script-editor.png)

Bạn có thể sửa đổi kịch bản tại đây, hoặc nhấp vào nút _COPY SCRIPT_ để chọn toàn bộ văn bản trong kịch bản của bạn. Sau đó, bạn có thể dán kịch bản vào IDE bạn chọn và tiếp tục làm việc với nó ở đó.

<details>
<summary> Ví dụ kịch bản được ghi lại </summary>

```js
// Scenario: Scenario_1 (executor: ramping-vus)

import { sleep, group } from 'k6'
import http from 'k6/http'

export const options = {
  ext: {
    loadimpact: {
      distribution: { 'amazon:us:ashburn': { loadZone: 'amazon:us:ashburn', percent: 100 } },
      apm: [],
    },
  },
  thresholds: {},
  scenarios: {
    Scenario_1: {
      executor: 'ramping-vus',
      gracefulStop: '30s',
      stages: [
        { target: 20, duration: '1m' },
        { target: 20, duration: '3m30s' },
        { target: 0, duration: '1m' },
      ],
      gracefulRampDown: '30s',
      exec: 'scenario_1',
    },
  },
}

export function scenario_1() {
  let response

  group('page_1 - http://ecommerce.k6.io/', function () {
    response = http.get('http://ecommerce.k6.io/', {
      headers: {
        'upgrade-insecure-requests': '1',
        accept:
          'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9',
        'accept-encoding': 'gzip, deflate',
        'accept-language':
          'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
      },
    })
    sleep(1.2)

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-includes/css/dist/block-library/style.min.css?ver=5.9',
      {
        headers: {
          accept: 'text/css,*/*;q=0.1',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/plugins/woocommerce/packages/woocommerce-blocks/build/vendors-style.css?ver=4.0.0',
      {
        headers: {
          accept: 'text/css,*/*;q=0.1',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/plugins/woocommerce/packages/woocommerce-blocks/build/style.css?ver=4.0.0',
      {
        headers: {
          accept: 'text/css,*/*;q=0.1',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/themes/storefront/assets/css/base/gutenberg-blocks.css?ver=3.5.0',
      {
        headers: {
          accept: 'text/css,*/*;q=0.1',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/themes/storefront/style.css?ver=3.5.0',
      {
        headers: {
          accept: 'text/css,*/*;q=0.1',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/themes/storefront/assets/css/base/icons.css?ver=3.5.0',
      {
        headers: {
          accept: 'text/css,*/*;q=0.1',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/themes/storefront/assets/css/woocommerce/woocommerce.css?ver=3.5.0',
      {
        headers: {
          accept: 'text/css,*/*;q=0.1',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-includes/js/jquery/jquery.min.js?ver=3.6.0',
      {
        headers: {
          accept: '*/*',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-includes/js/jquery/jquery-migrate.min.js?ver=3.3.2',
      {
        headers: {
          accept: '*/*',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )
    sleep(0.7)

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/plugins/woocommerce/assets/js/jquery-blockui/jquery.blockUI.min.js?ver=2.70',
      {
        headers: {
          accept: '*/*',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/plugins/woocommerce/assets/js/frontend/add-to-cart.min.js?ver=5.0.1',
      {
        headers: {
          accept: '*/*',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/plugins/woocommerce/assets/js/js-cookie/js.cookie.min.js?ver=2.1.4',
      {
        headers: {
          accept: '*/*',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/plugins/woocommerce/assets/js/frontend/woocommerce.min.js?ver=5.0.1',
      {
        headers: {
          accept: '*/*',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/plugins/woocommerce/assets/js/frontend/cart-fragments.min.js?ver=5.0.1',
      {
        headers: {
          accept: '*/*',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/themes/storefront/assets/js/navigation.min.js?ver=3.5.0',
      {
        headers: {
          accept: '*/*',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/themes/storefront/assets/js/skip-link-focus-fix.min.js?ver=20130115',
      {
        headers: {
          accept: '*/*',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/themes/storefront/assets/js/vendor/pep.min.js?ver=0.4.3',
      {
        headers: {
          accept: '*/*',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/themes/storefront/assets/js/woocommerce/header-cart.min.js?ver=3.5.0',
      {
        headers: {
          accept: '*/*',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/themes/storefront/assets/js/footer.min.js?ver=3.5.0',
      {
        headers: {
          accept: '*/*',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-includes/js/wp-emoji-release.min.js?ver=5.9',
      {
        headers: {
          accept: '*/*',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/uploads/2021/02/K6-logo.png',
      {
        headers: {
          accept: 'image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/uploads/2021/02/album-1.jpg',
      {
        headers: {
          accept: 'image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/uploads/2021/02/beanie-2.jpg',
      {
        headers: {
          accept: 'image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/uploads/2021/02/beanie-with-logo-1.jpg',
      {
        headers: {
          accept: 'image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/uploads/2021/02/belt-2.jpg',
      {
        headers: {
          accept: 'image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/uploads/2021/02/cap-2.jpg',
      {
        headers: {
          accept: 'image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/uploads/2021/02/hoodie-2.jpg',
      {
        headers: {
          accept: 'image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/uploads/2021/02/hoodie-with-logo-2.jpg',
      {
        headers: {
          accept: 'image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/uploads/2021/02/hoodie-with-zipper-2.jpg',
      {
        headers: {
          accept: 'image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/uploads/2021/02/logo-1.jpg',
      {
        headers: {
          accept: 'image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/uploads/2021/02/long-sleeve-tee-2.jpg',
      {
        headers: {
          accept: 'image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/uploads/2021/02/polo-2.jpg',
      {
        headers: {
          accept: 'image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/uploads/2021/02/single-1.jpg',
      {
        headers: {
          accept: 'image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.post(
      'http://ecommerce.k6.io/?wc-ajax=get_refreshed_fragments',
      {
        time: '1645005395795',
      },
      {
        headers: {
          accept: '*/*',
          'x-requested-with': 'XMLHttpRequest',
          'content-type': 'application/x-www-form-urlencoded; charset=UTF-8',
          origin: 'http://ecommerce.k6.io',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )
    sleep(0.6)

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/uploads/2021/02/K6-logo.png',
      {
        headers: {
          accept: 'image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/uploads/2021/02/K6-logo.png',
      {
        headers: {
          accept: 'image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )
    sleep(6.7)

    response = http.get('http://ecommerce.k6.io/', {
      headers: {
        accept: '*/*',
        'accept-encoding': 'gzip, deflate',
        'accept-language':
          'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
      },
    })

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-includes/css/dist/block-library/style.min.css?ver=5.9',
      {
        headers: {
          accept: 'text/css,*/*;q=0.1',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/plugins/woocommerce/packages/woocommerce-blocks/build/vendors-style.css?ver=4.0.0',
      {
        headers: {
          accept: 'text/css,*/*;q=0.1',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/plugins/woocommerce/packages/woocommerce-blocks/build/style.css?ver=4.0.0',
      {
        headers: {
          accept: 'text/css,*/*;q=0.1',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/themes/storefront/assets/css/base/gutenberg-blocks.css?ver=3.5.0',
      {
        headers: {
          accept: 'text/css,*/*;q=0.1',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/themes/storefront/style.css?ver=3.5.0',
      {
        headers: {
          accept: 'text/css,*/*;q=0.1',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/themes/storefront/assets/css/base/icons.css?ver=3.5.0',
      {
        headers: {
          accept: 'text/css,*/*;q=0.1',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/themes/storefront/assets/css/woocommerce/woocommerce.css?ver=3.5.0',
      {
        headers: {
          accept: 'text/css,*/*;q=0.1',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )
  })
}
```

</details>

### Test Builder

Nếu bạn chọn tùy chọn Test Builder ở bước trước, bạn sẽ thấy một màn hình như thế này:

![](../../images/k6-browser-recorder-5.png)

Từ đây, bạn có thể tiếp tục khám phá, xây dựng kịch bản của mình và tăng tốc nó để chạy như một bài kiểm tra sàng lọc (shakeout) hoặc kiểm thử tải. Bạn có thể tìm thêm thông tin về [cách sử dụng Test Builder tại đây](../XX-Future-Ideas/Creating-a-script-using-the-Test-Builder.md). Tuy nhiên, hãy nhớ rằng việc chạy bài kiểm tra trên k6 Cloud sẽ tiêu tốn một trong những lượt chạy thử miễn phí của bạn. Nếu bạn muốn chạy bài kiểm tra cục bộ, hãy nhấp vào nút gạt để chuyển từ _Builder_ sang _Script_.

## Hạn chế của việc ghi lại một kịch bản

Trong phần này, bạn đã học cách sử dụng phiên bản miễn phí của k6 Cloud để ghi lại một luồng nghiệp vụ trong trình duyệt của mình và tạo kịch bản k6 từ bản ghi đó. Việc tạo kịch bản theo cách này có thể giúp giảm đáng kể thời gian chuẩn bị khi bạn bắt đầu quá trình viết kịch bản.

Tuy nhiên, việc ghi lại kịch bản cũng có những hạn chế sau:

- Bạn chỉ có thể ghi lại các luồng người dùng liên quan đến các hành động bạn có thể thực hiện trong trình duyệt web.
- Việc tương quan yêu cầu và phản hồi tự động chỉ hoạt động đối với các giá trị động thường dùng.
- Trình ghi lưu lại tất cả thông tin mà trình duyệt của bạn gửi và nhận, điều này có thể dẫn đến việc tăng "nhiễu" trong kịch bản.
- Thời gian sleep được mã hóa cứng dưới dạng các giá trị hằng số dựa trên những gì đã được ghi lại.

Vì những lý do này, bạn có thể kỳ vọng rằng việc ghi lại kịch bản sẽ thực hiện được rất nhiều công việc ban đầu cho bạn, nhưng việc sửa đổi kịch bản, [gỡ lỗi (debugging)](../III-k6-Intermediate/01-How-to-debug-k6-load-testing-scripts.md) để nó hoạt động theo cách bạn muốn và [làm cho nó thực tế hơn](../XX-Future-Ideas/Best-practices-for-designing-realistic-k6-scripts.md) vẫn là một thực hành tốt nhất.

Dưới đây là một số điều bạn nên cân nhắc thực hiện sau khi ghi lại một kịch bản:

- Đổi tên hoặc sửa đổi các [groups](https://k6.io/docs/using-k6/tags-and-groups/#groups) đã được ghi lại
- [Thêm checks vào kịch bản của bạn](04-Adding-checks-to-your-script.md)
- [Làm cho think time trở nên động](05-Adding-think-time-using-sleep.md#Dynamic-think-time)
- Thêm hoặc sửa đổi các [tùy chọn kiểm thử (test options)](06-k6-Load-Test-Options.md)
- Thêm các cách để [tổ chức mã nguồn](https://k6.io/docs/using-k6/tags-and-groups/) và giúp đồng nghiệp hiểu kịch bản làm gì
- Sàng lọc kịch bản bằng cách chạy nó vài lần và [gỡ lỗi nó](../III-k6-Intermediate/01-How-to-debug-k6-load-testing-scripts.md)

## Kiểm tra kiến thức của bạn

### Câu hỏi 1

Trong tình huống nào sau đây, việc ghi lại một kịch bản có thể phù hợp hơn là viết từ đầu?

A: Bạn muốn kiểm thử một vài API endpoint và frontend cho chúng vẫn chưa được xây dựng.

B: Trang bạn đang kiểm thử bao gồm rất nhiều hình ảnh và bạn muốn bao gồm thời gian cần thiết để truy xuất những hình ảnh đó trong kịch bản kiểm thử của mình.

C: Bạn muốn sửa đổi các yêu cầu HTTP để đi đến một dịch vụ giả lập (mocking service).

### Câu hỏi 2

Chi phí để sử dụng trình ghi trình duyệt k6 là bao nhiêu?

A: Không tốn gì cả.

B: Miễn phí trong tháng đầu tiên.

C: Bạn phải đăng ký một gói k6 Cloud.

### Câu hỏi 3

Phát biểu nào sau đây về việc ghi trình duyệt là đúng?

A: Bạn có thể ghi lại một kịch bản và phát lại nó ngay lập tức mà không cần sửa đổi gì.

B: Sau khi ghi, bạn nên tăng tải và chạy nó với 100 người dùng cho bài kiểm tra đầu tiên của mình.

C: Việc sử dụng trình ghi trình duyệt không có nghĩa là bạn sẽ không phải sửa đổi mã kiểm thử để kịch bản hoạt động.

### Đáp án

1. B. Ghi lại một kịch bản đặc biệt hữu ích cho các ứng dụng web nơi một hành động duy nhất (chẳng hạn như nhấp vào một liên kết) yêu cầu tải xuống nhiều tài nguyên nhúng khác, chẳng hạn như hình ảnh hoặc scripts.
2. A. Trình ghi trình duyệt k6 miễn phí và không yêu cầu đăng ký k6 Cloud.
3. C. Sau khi bạn ghi lại một kịch bản, bạn có thể cũng sẽ phải tương quan dữ liệu động, thêm dữ liệu kiểm thử hoặc sửa đổi nó theo các cách khác trước khi bạn có thể thực thi nó một cách nhất quán và lặp lại trong một bài kiểm thử tải.
