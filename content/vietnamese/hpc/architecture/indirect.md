---
title: Phân nhánh gián tiếp
weight: 4
---

Trong quá trình lắp ráp, tất cả các nhãn được chuyển đổi thành địa chỉ (tuyệt đối hoặc tương đối) và sau đó được mã hóa thành các lệnh nhảy.

Bạn cũng có thể nhảy theo một giá trị không phải hằng số được lưu trữ bên trong một thanh ghi, được gọi là *bước nhảy được tính toán*:

```nasm
jmp rax
```

Điều này có một vài ứng dụng thú vị liên quan đến ngôn ngữ động và triển khai luồng điều khiển phức tạp hơn.

### Phân nhánh Multiway

Nếu bạn đã quên câu lệnh `switch` có tác dụng gì, thì đây là một quy trình con để tính điểm trung bình trong hệ thống chấm điểm của Mỹ:
```cpp
switch (grade) {
    case 'A':
        return 4.0;
        break;
    case 'B':
        return 3.0;
        break;
    case 'C':
        return 2.0;
        break;
    case 'D':
        return 1.0;
        break;
    case 'E':
    case 'F':
        return 0.0;
        break;
    default:
        return NAN;
}
```

Cá nhân tôi không nhớ lần cuối cùng tôi sử dụng câu lệnh switch trong bối cảnh phi giáo dục. Nói chung, các câu lệnh switch tương đương với một chuỗi "if, else if, else if, else if…", v.v. và vì lý do này, nhiều ngôn ngữ thậm chí không có chúng. Tuy nhiên, các cấu trúc luồng điều khiển như vậy rất quan trọng để triển khai trình phân tích cú pháp, trình thông dịch và các máy trạng thái khác, thường bao gồm một vòng lặp `while (true)` duy nhất và một câu lệnh `switch (state)` bên trong.

Khi chúng ta có quyền kiểm soát phạm vi giá trị mà biến có thể nhận, chúng ta có thể sử dụng thủ thuật sau đây bằng cách sử dụng các bước nhảy được tính toán. Thay vì tạo $n$ nhánh có điều kiện, chúng ta có thể tạo một *bảng nhánh* chứa các con trỏ / phương sai tới các vị trí có thể nhảy, và sau đó chỉ cần lập chỉ mục với biến `state` nhận các giá trị trong phạm vi $[0, n)$.

Các trình biên dịch sử dụng kỹ thuật này khi các giá trị được đóng gói dày đặc với nhau. Nó cũng có thể được triển khai một cách rõ ràng với *goto được tính toán*:
```cpp
void weather_in_russia(int season) {
    static const void* table[] = {&&winter, &&spring, &&summer, &&fall};
    goto *table[season];

    winter:
        printf("Freezing\n");
        return;
    spring:
        printf("Dirty\n");
        return;
    summer:
        printf("Dry\n");
        return;
    fall:
        printf("Windy\n");
        return;
}
```

### Dynamic Dispatch

Phân nhánh gián tiếp cũng là công cụ để thực hiện đa hình thời gian chạy.

Ví dụ, chúng ta có một lớp trừu tượng của `Animal` với phương thức ảo `.speak()` và hai cách triển khai cụ thể: một `Dog` sủa và một `Cat` kêu:

```cpp
struct Animal {
    virtual void speak() { printf("<abstract animal sound>\n");}
};

struct Dog {
    void speak() override { printf("Bark\n"); }
};

struct Cat {
    void speak() override { printf("Meow\n"); }
};

Dog sparkles;
Cat mittens;

Animal *catdog = (rand() & 1) ? &sparkles : &mittens;
catdog->speak();
```

Có nhiều cách để thực hiện hành vi này, C++ thực hiện nó bằng cách sử dụng *bảng phương thức ảo*.

Đối với tất cả các triển khai cụ thể của `Animal`, trình biên dịch độn (padding) tất cả các phương thức của chúng (nghĩa là chuỗi lệnh của chúng) để chúng có cùng độ dài cho tất cả các lớp (bằng cách chèn một số [chỉ dẫn lấp chỗ trống](../layout) sau khi `ret`) và sau đó chỉ cần ghi chúng tuần tự vào một nơi nào đó trong bộ nhớ lệnh. Sau đó, nó thêm trường *thông tin kiểu thời gian chạy* vào cấu trúc.

Với một lệnh gọi phương thức ảo, trường phương sai được tìm nạp từ phiên bản của một cấu trúc và một lệnh gọi hàm bình thường được thực hiện với nó, sử dụng thực tế là tất cả các phương thức và các trường khác của mọi lớp dẫn xuất có cùng phương sai.

Tất nhiên, điều này phát sinh một số chi phí:

- Bạn có thể cần 15 chu kỳ hoặc lâu hơn vì lý do xả đường ống tương tự như đối với [dự đoán sai nhánh](/hpc/pipelining).
- Trình biên dịch rất có thể sẽ không thể tự nội tuyến (inline) lệnh gọi hàm.
- Kích thước lớp tăng một vài byte hoặc nhiều hơn (tùy vào cách việc triển khai).
- Kích thước nhị phân tăng lên một chút.

Vì những lý do này, đa hình thời gian chạy thường được tránh dùng trong các ứng dụng quan trọng về hiệu suất.