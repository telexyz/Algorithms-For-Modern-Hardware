---
title: Khi nào nên tối ưu hoá
weight: 4
# draft: true
---

Tại sao các công ty như Google yêu cầu giải quyết các bài toán thuật toán bảng trắng trong các cuộc phỏng vấn của họ? Lý do là các tổ chức có hơn 10000 kỹ sư rất khác với thế giới bên ngoài.

Chắc chắn, khi bạn tiến bộ, những câu hỏi lập trình đơn giản này sẽ được thay thế bằng các vấn đề thiết kế hệ thống phức tạp và đánh giá khả năng quản lý. Lý do các câu hỏi phức tạp hơn chưa được hỏi là bởi vì bạn không nhận được vào cho một vai trò cao cấp ngay khi vừa tốt nghiệp đại học.

- - -

Thiết kế thuật toán là kỹ năng bạn dùng tới 20% thời gian lập trình nhưng lại quyết định 80% sự thành công. Và có những lĩnh vực mà hiệu năng là tính năng quyết định sống còn.

> Big-O không phải thứ các công ty phần mềm muốn. Big-O không đảm bảo chương trình viết ra sẽ tối ưu mà nó chỉ đảm bảo sẽ tránh được những thuật toán quá chậm. Những thuật toán không có khả năng mở rộng.

- - -

Bạn sẽ thật sự thất vọng nếu bạn có kinh nghiệm lập trình cạnh tranh. Bạn sẽ không thể giải quyết những loại vấn đề này, ngay cả khi họ hỏi họ trong một cuộc phỏng vấn. Để giải quyết chúng, bạn cần loại bằng cấp khác. Thuật toán tối ưu tiệm cận đã tồn tại, bạn cần tối ưu hoá yếu tố không đổi. Thật không may, chỉ có một số ít các trường đại học dạy điều đó.

## Các cấp độ tối ưu hoá

Các lập trình viên có thể được xếp hạng về khả năng tối ưu hoá phần mềm của họ:

0. *Người mới*. Những người không nghĩ về hiệu suất chút nào. Họ thường viết bằng ngôn ngữ cấp cao. Hầu hết các "lập trình viên" dừng lại ở mức này.

1. *Sinh viên đại học*. Những người biết về ký hiệu Big O và quen thuộc với các cấu trúc và phương pháp tiếp cận. Người thực hành LeetCode và CodeForces đang ở mức này. Đây cũng là yêu cầu trong việc tuyển dụng vào các công ty lớn.

2. *Sinh viên sau đại học*. Những người biết rằng không phải tất cả các lệnh được thực thi bằng nhau; biết các mô hình chi phí khác như mô hình bộ nhớ ngoài (B-tree, sắp xếp bên ngoài), mô hình từ (bitset), hoặc tính toán song song, nhưng vẫn ở mức lý thuyết.

3. *Nhà phát triển chuyên nghiệp*. Những người biết thời gian thực tế của các lệnh. Nhận thức được rằng dự đoán rẽ nhánh sai là tốn kém, và bộ nhớ được chia thành các dòng bộ trong nhớ đệm. Biết một số kỹ thuật SIMD cơ bản. 

4. *Kỹ sư hiệu suất*. Biết chính xác những gì xảy ra bên trong phần cứng. Biết sự khác biệt giữa độ trễ và băng thông, biết về cổng. Biết cách sử dụng SIMD và phần còn lại của tập lệnh một cách hiệu quả. Có thể đọc mã Assembly và sử dụng profile (để đo lường hiệu suất).

5. *Nhân viên Intel*. Biết chi tiết cụ thể về kiến trúc vi mô. Điều này nằm ngoài phạm vi của các kỹ sư bình thường.

Trong đọc cuốn sách này, chúng tôi hy vọng rằng người đọc bắt đầu ở đâu đó quanh cấp độ 1, và hy vọng khi đọc xong nó sẽ đến 4.

Bạn cũng nên trải qua các cấp độ này khi thiết kế thuật toán. Đầu tiên làm cho nó hoạt động đã, sau đó chọn một loạt các thuật toán tối ưu tiệm cận hợp lý. Sau đó suy nghĩ về cách họ sẽ làm việc về hoạt động bộ nhớ của họ hoặc khả năng thực hiện song song (ngay cả khi bạn xem xét các chương trình đơn luồng, thì vẫn sẽ có rất nhiều cơ chế song song tồn tại bên trong một lõi), và sau đó tiến tới cài đặt tối ưu. Nên tránh tối ưu hoá sớm, như Knuth đã từng nói.

---

Đối với hầu hết các trang web, hiệu quả không quan trọng, nhưng *độ trễ* thì có. Đồng thời, một máy chủ có lõi chuyên dụng và 1GB ram (là một lượng lớn tài nguyên vô lý cho một dịch vụ web đơn giản) có khấu hao khoảng một phần triệu đô-la mỗi giây. (Tác giả muốn nói là còn nhiều thứ có thể tối ưu cho Web)

Amazon đã làm một thử nghiệm trong đó họ để người dùng trải nghiệm dịch vụ của mình với độ trễ giả tạo và phát hiện ra rằng độ trễ 100ms làm giảm doanh thu. Điều này xảy ra với hầu hết các dịch vụ khác

Việc giảm thiểu độ trễ thường có thể được thực hiện với tính toán song song, đó là lý do tại sao các hệ thống phân tán được ưa chuộng. Cuốn sách này liên quan đến việc cải thiện *hiệu quả* của các thuật toán, điều này giúp làm giảm độ trễ như là  một sản phẩm phụ của quá trình tối ưu.

Tuy nhiên, vẫn có những trường hợp sử dụng khi có sự đánh đổi giữa chất lượng và giá thành của máy chủ. (Và khi đó với tài nguyên hạn chế, việc tối ưu hóa trở nên quan trọng hơn bao giờ hết.)

- Tìm kiếm có thứ bậc. Thường có nhiều lớp mô hình chính xác hơn nhưng chậm hơn. Bạn càng xếp hạng nhiều tài liệu trên mỗi lớp, thì chất lượng cuối cùng càng tốt.

- Trò chơi. Chúng thú vị hơn trên quy mô lớn, nhưng sức mạnh tính toán cũng tăng lên. Điều này bao gồm cả AI.

- Huấn luyện AI sử dụng dữ liệu lớn như mô hình ngôn ngữ. Các mô hình nặng hơn yêu cầu tính toán nhiều hơn. Điểm nghẽn trong đó không phải là dữ liệu, mà là hiệu quả.


## Ước tính tác động

Đôi khi, việc tối ưu hóa cần phải xảy ra những lựa chọn bậc cao hơn.

SIMDJSON tăng tốc độ phân tích cú pháp JSON, nhưng có thể tốt hơn nếu không sử dụng JSON ngay từ đầu. Hãy sử dụng Protobuf hoặc các định dạng nhị phân phẳng.

Chi phí để thực hiện, lỗi, khả năng bảo trì. Hoàn toàn ổn khi hầu hết các phần mềm trên thế giới không đạt hiệu năng cao nhất để ưu tiên các chi phí trên.

Trở thành một lập trình viên giỏi hơn có nghĩa là gì? Các chương trình nhanh hơn? Tốc độ làm việc nhanh hơn? Ít lỗi hơn? Nó là sự kết hợp của những cái đó.

Việc tối ưu hóa trình biên dịch hoặc cơ sở dữ liệu là những ví dụ về các hoạt động có tác động cao vì chúng là nền tảng để các phần mềm khác được xây dựng nên.