# Kiểm thử hiệu năng (performance testing) là gì?

Mối quan tâm hàng đầu của kiểm thử hiệu năng (performance testing) là hệ thống hoạt động _tốt như thế nào_. Khác với kiểm thử chức năng (functional testing), vốn dùng để kiểm tra xem hệ thống _có_ hoạt động hay không, kiểm thử hiệu năng tìm cách đo lường các khía cạnh định tính về trải nghiệm của người dùng đối với hệ thống, chẳng hạn như khả năng phản hồi (responsiveness) và độ tin cậy (reliability).

## Tại sao chúng ta nên thực hiện kiểm thử hiệu năng?

Với kiểm thử hiệu năng, đội ngũ của bạn có thể:

- **Cải thiện trải nghiệm người dùng.** Xác định các nút thắt cổ chai (bottlenecks) tiềm ẩn và các vấn đề sớm trong quá trình phát triển. Kiểm thử hiệu năng cung cấp một bức tranh hoàn chỉnh về trải nghiệm của người dùng khi truy cập ứng dụng của bạn, vượt xa hơn cả các chức năng của ứng dụng.
- **Chuẩn bị cho nhu cầu bất ngờ.** Kiểm thử vượt qua mức tải (load) dự kiến để tìm ra các điểm gãy (breaking points) của ứng dụng và xây dựng các quy trình tốt hơn để ứng phó và tận dụng những thành công chưa từng có.
- **Tăng sự tự tin vào ứng dụng.** Giảm thiểu rủi ro thất bại tổng thể bằng việc kiểm thử hiệu năng một cách có hệ thống. Việc giảm thiểu rủi ro này cũng giúp xây dựng sự tự tin cho đội ngũ. Các đội ngũ có thể làm việc tốt hơn khi biết rằng ứng dụng của họ có thể chịu đựng được các điều kiện bất ngờ trong môi trường thực tế (production).
- **Đánh giá và tối ưu hóa hạ tầng.** Giảm chi phí hạ tầng không cần thiết mà không làm ảnh hưởng đến hiệu năng. Mô phỏng các kịch bản để quan sát việc mở rộng theo chiều ngang (horizontal scaling) và chiều dọc (vertical scaling), đồng thời thực hiện các thử nghiệm để xác minh các tài nguyên mà hệ thống đang được kiểm thử thực sự yêu cầu.

Nếu kiểm thử hiệu năng có giá trị như vậy, tại sao không có nhiều đội ngũ thực hiện nó?

## Các lý do phổ biến để không thực hiện kiểm thử hiệu năng

Những lo ngại này thường bắt nguồn từ những quan niệm sai lầm về chi phí và độ phức tạp cần thiết của việc kiểm thử hiệu năng.

### Ứng dụng của chúng ta quá nhỏ

Ý tưởng cho rằng chỉ các tập đoàn lớn hoặc các ứng dụng phức tạp mới yêu cầu kiểm thử hiệu năng chủ yếu xuất phát từ quan niệm sai lầm rằng kiểm thử hiệu năng cần phải mô phỏng hàng trăm hoặc thậm chí hàng nghìn người dùng. Trên thực tế, việc đo lường cách một hệ thống hoạt động chỉ với một nhóm nhỏ người dùng có thể mang lại lợi ích đáng kể. Ngay cả việc mô phỏng một người dùng duy nhất cũng có thể làm nổi bật các nút thắt cổ chai trong ứng dụng mà lẽ ra sẽ không bị phát hiện. Những sự kém hiệu quả về hiệu năng gây tốn kém cũng có thể tồn tại trong các hệ thống nhỏ.

### Nó tốn kém hoặc mất thời gian

Kiểm thử hiệu năng _có thể_ tốn kém và mất thời gian, nhưng các đội ngũ có thể lựa chọn loại hoạt động phù hợp với ngân sách về chi phí và thời gian của họ. Chi phí của việc _không_ kiểm thử hiệu năng thường lớn hơn nhiều so với khoản đầu tư ban đầu vào một số hoạt động kiểm thử hiệu năng.

### Nó yêu cầu kiến thức kỹ thuật sâu rộng

Các loại kiểm thử hiệu năng khác nhau yêu cầu các mức độ kiến thức kỹ thuật khác nhau. Tuy nhiên, về bản chất, kiểm thử hiệu năng không phức tạp hơn hay ít phức tạp hơn các hình thức kiểm thử khác. Các đội ngũ có thể chọn từ một loạt các hoạt động kiểm thử hiệu năng tùy theo khả năng tiếp nhận độ phức tạp của họ. Việc truy cập một trang web trong khi xem thời gian từ bảng Network của DevTools trong trình duyệt là một loại kiểm thử hiệu năng mang lại giá trị tức thì với rất ít nỗ lực.

### Chúng ta không có môi trường hiệu năng

Kiểm thử hiệu năng không phải lúc nào cũng có nghĩa là kiểm thử tải (load testing), và ngay cả kiểm thử tải không phải lúc nào cũng liên quan đến việc ép ứng dụng đến điểm gãy. Một số cách để đánh giá hiệu năng không yêu cầu môi trường kiểm thử hiệu năng chuyên dụng, ví dụ:

- Unit tests cho hiệu năng trong quá trình phát triển
- API tests trong quá trình kiểm thử hệ thống (System testing)
- Và giám sát tổng hợp (synthetic monitoring) hoặc các bài kiểm tra tải thấp trong môi trường production.

### Khả năng quan sát quan trọng hơn kiểm thử hiệu năng

Việc có một nền tảng khả năng quan sát (observability) hoàn thiện khiến nhiều người từ bỏ kiểm thử hiệu năng để ủng hộ việc giám sát hiệu năng ứng dụng trong môi trường production. Tuy nhiên, hiệu quả của observability phụ thuộc vào việc có dữ liệu để quan sát, và nếu không có khả năng tạo dữ liệu nhân tạo, hiệu năng ứng dụng thường chỉ được quan sát sau khi các nút thắt cổ chai đã đi vào hoạt động thực tế.

Với kiểm thử hiệu năng, các đội ngũ có thể mô phỏng các kịch bản người dùng phong phú _trước khi_ các vấn đề hiệu năng tiềm ẩn được phát hành ra production, làm cho observability trở nên hữu ích trong cả môi trường kiểm thử. Một số loại kiểm thử, chẳng hạn như phục hồi sau thảm họa (disaster recovery), kỹ thuật hỗn loạn (chaos engineering), và kiểm thử độ tin cậy (reliability testing), cũng giúp các đội ngũ chuẩn bị cho những thất bại không thể tránh khỏi.

Cả kiểm thử hiệu năng và observability đều là những thành phần thiết yếu để cải thiện chất lượng của một hệ thống.

### Điện toán đám mây là vô hạn; chúng ta luôn có thể mở rộng

Hiện nay, khi nhiều ứng dụng được lưu trữ trên đám mây, việc nghĩ rằng mở rộng theo chiều ngang hoặc chiều dọc sẽ loại bỏ nhu cầu kiểm thử hiệu năng là điều rất dễ xảy ra. Tuy nhiên, niềm tin này có thể dẫn đến việc tăng chi phí đáng kể nếu tính hiệu quả không được xem xét.

Khả năng mở rộng (Scalability) chỉ là _một_ khía cạnh của hiệu năng ứng dụng cần được kiểm thử. Ngay cả các hệ thống được mở rộng hiệu quả vẫn có thể chậm hoặc dễ gặp lỗi và mất điện (outages) khiến ứng dụng không thể sử dụng được. Trên thực tế, việc mở rộng hệ thống có thể làm trầm trọng thêm một số loại lỗi dây chuyền (cascading failures), chẳng hạn như bão thử lại (retry storms) và vấn đề đám đông ồ ạt (thundering-herd problem).

## Kiểm tra kiến thức của bạn

### Câu hỏi 1

Điều nào sau đây _không_ phải là lý do tại sao chúng ta nên thực hiện kiểm thử hiệu năng?

A: Chúng ta muốn làm cho việc sử dụng ứng dụng của mình trở thành một trải nghiệm thú vị hơn cho khách hàng.

B: Ai biết được điều gì có thể xảy ra trong production? Chúng ta muốn sẵn sàng cho mọi thứ.

C: Chúng ta muốn thay thế ngăn xếp observability của mình bằng một bộ kiểm thử chủ động hơn.

### Câu hỏi 2

Ai đó nói với bạn rằng kiểm thử hiệu năng quá tốn kém. Cách phản hồi tốt là gì?

A: Ngày càng có nhiều chuyên gia đưa kiểm thử hiệu năng vào CV của họ, vì vậy chúng ta có thể thuê ai đó với giá rẻ.

B: Ghi lại một kịch bản (script) và phát lại nó tốn rất ít chi phí, vì vậy chúng ta có thể làm điều đó.

C: Các lỗi hiệu năng tiềm ẩn sẽ tốn kém hơn để sửa sau khi chúng được triển khai lên production.

D: A và B.

### Câu hỏi 3

Phát biểu nào sau đây là đúng?

A: Kiểm thử hiệu năng luôn là về việc tạo ra tải người dùng hoặc lưu lượng truy cập cao đối với các máy chủ ứng dụng.

B: Kiểm thử hiệu năng là một hoạt động đòi hỏi chuyên môn đặc biệt để thực hiện.

C: Kiểm thử hiệu năng yêu cầu một môi trường giống như production.

D: Kiểm thử hiệu năng cải thiện tinh thần tổng thể của đội ngũ bằng cách xây dựng sự tự tin vào những gì ứng dụng có thể chịu đựng được.

### Đáp án

1. C. Kiểm thử hiệu năng và observability bổ sung cho nhau. Chúng không thay thế nhau, mà thay vào đó cùng hoạt động để cải thiện sự tin tưởng vào hệ thống.
2. C. Các nút thắt cổ chai về hiệu năng và các vấn đề khác thường tốn kém hơn để sửa khi được xác định trong production. Kiểm thử hiệu năng chủ động có thể bù đắp nhiều hơn chi phí của nó khi nó xác định và khắc phục các lỗi này sớm hơn trong quá trình phát triển.
3. D. A sai vì kiểm thử hiệu năng có thể được thực hiện ở mức tải thấp hơn. B sai vì bất kỳ ai cũng có thể thực hiện kiểm thử hiệu năng, không chỉ các chuyên gia kiểm thử hiệu năng. C sai vì kiểm thử hiệu năng có thể được thực hiện trong môi trường phát triển, staging và kiểm thử.
