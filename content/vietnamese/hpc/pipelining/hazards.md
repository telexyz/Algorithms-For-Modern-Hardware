---
title: Các mối nguy của đường ống
weight: 1
published: true
---

[Đường ống](../) cho phép bạn ẩn độ trễ của các lệnh bằng cách chạy chúng đồng thời, nhưng cũng tạo ra một số trở ngại tiềm ẩn - đặc trưng được gọi là *các mối nguy của đường ống*, tức là các tình huống khi lệnh tiếp theo không thể thực thi trong chu kỳ đồng hồ sau.

Có nhiều cách điều này có thể xảy ra:

* Một *mối nguy cấu trúc* xảy ra khi hai hoặc nhiều lệnh cần dùng cùng một phần của CPU (ví dụ: một đơn vị thực thi).
* Một *mối nguy dữ liệu* xảy ra khi bạn phải đợi một toán hạng được tính toán từ một số bước trước đó.
* Một *mối nguy điều khiển* xảy ra khi một CPU không thể biết nó cần thực hiện lệnh nào tiếp theo.

Cách duy nhất để xử lý các mối nguy đó là *dừng đường ống*: dừng tất cả các tiến trình cho tới khi nguyên nhân gây ra sự tắc nghẽn biến mất. Điều này tạo ra những *bong bóng* trong đường ống, khi mà các đơn vị thực thi không làm việc và không có việc hữu ích nào được hoàn thành.

![Sự tắc nghẽn đường ống ở giai đoạn thực thi lệnh](../img/bubble.png)

Các mối nguy khác nhau có các hình phạt khác nhau:

- Trong các mối nguy về cấu trúc, bạn phải đợi (thường là một chu kỳ nữa) cho đến khi đơn vị thực thi sẵn sàng. Chúng là những nút thắt cơ bản về hiệu suất và không thể tránh khỏi.

- Trong các mối nguy về dữ liệu, bạn phải đợi dữ liệu cần thiết được tính toán (độ trễ của *đường dẫn tới hạn*). Các mối nguy dữ liệu được giải quyết bằng cách cơ cấu lại các tính toán để đường dẫn tới hạn ngắn hơn.

- Trong các mối nguy kiểm soát, bạn thường phải xả toàn bộ đường ống và bắt đầu lại, lãng phí toàn bộ 15-20 chu kỳ. Chúng được giải quyết bằng cách loại bỏ hoàn toàn các nhánh hoặc làm cho chúng có thể dự đoán được để CPU có thể *suy đoán* một cách hiệu quả về những gì sẽ được thực thi tiếp theo.

Vì chúng có tác động rất khác nhau đến hiệu suất, chúng ta sẽ xem xét theo thứ tự giảm dần, bắt đàu từ những thứ nghiêm trọng nhất.