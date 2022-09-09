---
title: Vòng lặp và điều kiện
weight: 2
---

Hãy xem xét ví dụ dưới đây:
```nasm
loop:
    add  edx, DWORD PTR [rax]
    add  rax, 4
    cmp  rax, rcx
    jne  loop
```
Nó tính tổng của mảng số nguyên 32-bit.

Thành phần chính của vòng lặp là lệnh `add edx, DWORD PTR [rax]`: nó tải dữ liệu từ biến lặp `rax` và cộng dồn vào `edx`. Tiếp theo chúng ta dịch chuyển biến lặp về phía trước 4 byte với lệnh `add rax, 4`. Sau đó một lệnh phức tạp hơn chút xíu được thực hiện.

### Lệnh Jump

Hợp ngữ không có các lệnh điều hướng như `if`, `for`, `function`. Nó chỉ có lệnh `goto` hay còn gọi là jump trong thế giới lập trình bậc thấp. Vị trí nhảy có thể là một địa chỉ tuyệt đối trong bộ nhớ, hoặc tương đối với địa chỉ hiện tại, hoặc thậm chí [được tính toán trong thời gian chạy](../indirect). 

Để tránh phải đau đầu khi quản lý trực tiếp các địa chỉ này, bạn có thể đánh dấu bất kỳ lệnh nào bằng một chuỗi ký tự theo sau là `:`, rồi sử dụng chuỗi này làm nhãn thay thế bằng địa chỉ tương đối của lệnh được đánh dấu khi được chuyển đổi thành mã máy.

Lệnh nhảy không điều kiện `jmp` chỉ có thể dùng để thực hiện kiểu vòng lặp `while (true)`. Một họ các lệnh nhảy **có điều kiện** thực sự được sử dụng để điều khiển rẽ nhánh.

Thật hợp lý khi nghĩ rằng các điều kiện này được tính như là giá trị `bool` ở đâu đó và được chuyển cho các bước nhảy có điều kiện dưới dạng toán hạng: xét cho cùng, đây là cách nó hoạt động trong các ngôn ngữ lập trình. Nhưng đó không phải là cách nó được thực hiện trong phần cứng. Các phép toán có điều kiện sử dụng một thanh ghi `FLAGS` đặc biệt, thanh ghi này trước tiên cần được điền bằng cách chạy các lệnh để thực hiện một số loại kiểm tra.

Trong ví dụ của chúng ta, `cmp rax, rcx` so sánh `rax` với con trỏ cuối mảng `rcx`. Điều này cập nhật thanh ghi `FLAGS` và bây giờ nó có thể được sử dụng bởi `jne loop`, nó sẽ tìm kiếm một bit nhất định để biết liệu hai giá trị có bằng nhau hay không và sau đó nhảy trở lại đầu vòng lặp hoặc tiếp tục hướng dẫn tiếp theo (phá vỡ vòng lặp).

### Mở vòng lặp

Một điều bạn có thể nhận thấy về vòng lặp ở trên là có rất nhiều chi phí để xử lý một phần tử duy nhất. Trong mỗi chu kỳ, chỉ có một lệnh hữu ích được thực thi và 3 lệnh còn lại chỉ để gia tăng biến đếm và cố gắng tìm hiểu xem vòng lặp đã hoàn thành chưa.

Những gì chúng ta có thể làm là  *mở cuộn* vòng lặp bằng cách nhóm các lần lặp lại với nhau - tương đương với mã như thế này trong C:
```c++
for (int i = 0; i < n; i += 4) {
    s += a[i];
    s += a[i + 1];
    s += a[i + 2];
    s += a[i + 3];
}
```

Trong hợp ngữ nó sẽ trông như thế này:
```nasm
loop:
    add  edx, [rax]
    add  edx, [rax+4]
    add  edx, [rax+8]
    add  edx, [rax+12]
    add  rax, 16
    cmp  rax, rsi
    jne  loop
```

Giờ đây chúng ta chỉ cần sử dụng 3 lệnh điều khiển vòng lặp cho 4 lệnh thực sự hữu ích (một sự cải thiện hiệu năng từ $\frac{1}{4}$ lên $\frac{4}{7}$).

Trong thực tế, việc mở các vòng lặp không phải lúc nào cũng nâng cao hiệu suất bởi vì các bộ xử lý hiện đại không thực sự thực hiện từng hướng dẫn một, mà duy trì [hàng đợi lệnh đang chờ xử lý](/hpc/pipelining) để hai hoạt động độc lập có thể được thực thi đồng thời.

Đây cũng là trường hợp của chúng ta: tốc độ tăng thực từ việc giải nén sẽ không gấp bốn lần, bởi vì các hoạt động tăng bộ đếm và kiểm tra xem chúng ta đã hoàn thành chưa độc lập với phần thân của vòng lặp và có thể được lên lịch chạy đồng thời với nó. Nhưng vẫn có thể có lợi ích khi [yêu cầu trình biên dịch](/hpc/compilation/situational) mở cuộc nó ở một mức độ nào đó.

### Một cách tiếp cận thay thế

You don't have to explicitly use `cmp` or a similar instruction to make a conditional jump. Many other instructions either read or modify the `FLAGS` register, sometimes as a by-product enabling optional exception checks.

Bạn không cần phải sử dụng `cmp` hoặc một lệnh tương tự để thực hiện một bước nhảy có điều kiện. Nhiều hướng dẫn khác có thể đọc hoặc sửa đổi thanh ghi `FLAGS`, đôi khi là một sản phẩm phụ cho phép kiểm tra ngoại lệ tùy chọn.

Ví dụ: `add` luôn đặt một số cờ, biểu thị cho dù kết quả là 0 hay là âm, cho dù xảy ra tràn hay tràn dưới (an overflow or an underflow), v.v. Lợi dụng cơ chế này, các trình biên dịch thường tạo ra các vòng lặp như sau:
```nasm
    mov  rax, -100  ; giả sử 100 là khích thước mảng
loop:
    add  edx, DWORD PTR [rax + 100 + rcx]
    add  rax, 4
    jnz  loop       ; nếu rax chưa là 0 thì lặp lại
```

Mã này khó đọc hơn một chút đối với con người, nhưng nó ngắn hơn một lệnh trong vòng lặp khi so với ví dụ gốc, điều này có thể ảnh hưởng đáng kể đến hiệu suất.

Ví dụ gốc:
```nasm
loop:
    add  edx, DWORD PTR [rax]
    add  rax, 4
    cmp  rax, rcx
    jne  loop
```
