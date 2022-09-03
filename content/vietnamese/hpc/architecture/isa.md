---
title: Kiến trúc tập chỉ lệnh
weight: -1
---

Lập trình viên chúng ta rất thích xây dựng phần mềm dựa trên sự trừa tượng. Thử hình dung xem có bao nhiêu thứ xảy ra khi bạn tải một URL. Bạn gõ thứ gì đó trên bàn phím; các phím bấm bằng cách nào đó được hệ điều hành phát hiện và gửi đến trình duyệt; trình duyệt phân tích cú pháp URL và yêu cầu Hệ điều hành thực hiện một yêu cầu mạng; sau đó là DNS, định tuyến, TCP, HTTP, và tất cả các lớp OSI khác; Trình duyệt phân tích cú pháp HTML; JavaScript thể hiện sự kỳ diệu của nó; một số biểu diễn của một trang được gửi đến GPU để kết xuất; khung hình ảnh được gửi đến màn hình… và mỗi bước này có thể liên quan đến việc thực hiện hàng tá việc cụ thể khác.

Sự trừu tượng hoá giúp chúng ta giảm tất cả sự phức tạp này xuống một *giao diện* duy nhất mô tả những gì mà một *mô-đun* nhất định có thể làm mà không cần sửa chữa một cách triển khai cụ thể. Điều này mang lại lợi ích gấp đôi:

- Các kỹ sư làm việc trên các mô-đun cấp cao hơn chỉ cần biết giao diện (nhỏ hơn nhiều).
- Các kỹ sư làm việc trên bản thân mô-đun có quyền tự do tối ưu hóa và cấu trúc lại việc triển khai mô-đun miễn là nó tuân thủ các *hợp đồng* của mô-đun.

Các kỹ sư phần cứng cũng thích sự trừu tượng.

Các kỹ sư phần cứng cũng yêu thích sự trừu tượng. Một sự trừu tượng của CPU được gọi là *kiến trúc tập lệnh* (ISA), và nó xác định cách một máy tính sẽ hoạt động theo quan điểm của một lập trình viên. Tương tự như giao diện phần mềm, nó mang lại cho các kỹ sư máy tính khả năng cải tiến các thiết kế CPU hiện có đồng thời mang lại cho các lập trình viên sự tin tưởng rằng những thứ hoạt động trước đây sẽ không bị hỏng trên các chip mới hơn.

Về cơ bản, ISA xác định cách phần cứng sẽ diễn giải ngôn ngữ máy. Ngoài các lệnh và mã hóa nhị phân của chúng, ISA cũng xác định số lượng, kích thước và mục đích của các thanh ghi, mô hình bộ nhớ và mô hình đầu vào / đầu ra. Tương tự như giao diện phần mềm, ISA cũng có thể được mở rộng: trên thực tế, chúng thường được cập nhật, chủ yếu là theo cách tương thích ngược, các chỉ lệnh mới và chuyên biệt hơn được thêm vào để cải thiện hiệu suất.

### RISC và CISC

Trong lịch sử, đã có nhiều ISA được sử dụng. Nhưng không giống như [mã hóa ký tự và giao thức nhắn tin tức thời] (https://xkcd.com/927/), việc phát triển và duy trì một ISA hoàn toàn riêng biệt rất tốn kém, vì vậy các thiết kế CPU chính thống hội tụ ở một trong hai dòng:

- Chip **Arm**, được sử dụng trong hầu hết các thiết bị di động cũng như các thiết bị giống máy tính khác như TV, tủ lạnh thông minh, lò vi sóng, [máy tự động trên ô tô] (https://en.wikipedia.org/wiki / Tesla_Autopilot), v.v. Chúng được thiết kế bởi một công ty cùng tên của Anh, cũng như một số nhà sản xuất điện tử bao gồm Apple và Samsung.

- Chip **x86** [^x86], được thiết kế bởi Intel và AMD và được sử dụng trong hầu hết các máy chủ và máy tính để bàn, với một số ngoại lệ đáng chú ý như MacBook M1 của Apple, bộ xử lý Graviton của AWS và [siêu máy tính nhanh nhất thế giới](https://en.wikipedia.org/wiki/Fugaku_(supercomputer)), tất cả đều sử dụng CPU dựa trên Arm.

[^x86]: Các phiên bản 64-bit hiện đại của x86 được gọi là "AMD64", "Intel 64" hoặc bằng các tên trung lập hơn với nhà cung cấp là "x86-64" hoặc chỉ "x64". Một phần mở rộng 64-bit tương tự của Arm được gọi là "AArch64" hoặc "ARM64." Trong cuốn sách này, chúng tôi sẽ chỉ sử dụng "x86" và "Arm" ngụ ý các phiên bản 64-bit.

Sự khác biệt chính giữa chúng là sự phức tạp của kiến trúc, thiên về triết lý thiết kế hơn là một số thuộc tính được xác định nghiêm ngặt:

- CPU Arm là máy tính tập lệnh *giảm* (RISC). Chúng cải thiện hiệu suất bằng cách giữ cho tập lệnh nhỏ và được tối ưu hóa cao, mặc dù một số hoạt động ít phổ biến hơn phải được thực hiện với các chương trình con liên quan đến một số lệnh.

- CPU x86 là máy tính tập lệnh *phức tạp* (CISC). Chúng cải thiện hiệu suất bằng cách thêm nhiều chỉ lệnh chuyên biệt, đôi khi rất hiếm được sử dụng trong thực tế.

Ưu điểm chính của các thiết kế RISC là chúng tạo ra các chip đơn giản hơn và nhỏ hơn, giúp giảm chi phí sản xuất và sử dụng điện năng. Không có gì ngạc nhiên khi thị trường tự phân khúc với việc Arm thống trị các thiết bị đa năng, chạy bằng pin, và để lại mạng nơ-ron phức tạp và các tính toán trường Galois cho x86 cấp máy chủ, chuyên biệt cao.

Hai kiến trúc này giống nhau về mặt chức năng, cả hai đều chia sẻ các khái niệm như pipeline, cổng thực thi và SIMD, nhưng vì hầu hết độc giả quan tâm đến việc tối ưu hóa ứng dụng cho máy chủ và máy tính để bàn thông dụng, nên trong cuốn sách này chúng ta sẽ chủ yếu tập trung vào x86.