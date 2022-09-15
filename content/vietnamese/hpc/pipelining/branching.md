---
title: Chi phí rẽ nhánh
weight: 2
published: true
---

Khi CPU gặp một lệnh nhảy, hay [bất kỳ kiểu rẽ nhánh nào](/hpc/architecture/indirect), nó không ngồi yên chờ cho điều kiện được tính toán xong mà thay vào đó, nó bắt đầu *thực thi một cách suy đoán* nhánh mà dường như có nhiều khả năng được thực hiện ngay lập tức. Trong quá trình thực thi, CPU tính toán số liệu thống kê về các nhánh được thực hiện trên mỗi lệnh và sau một thời gian, chúng bắt đầu dự đoán việc rẽ nhánh rất tốt bằng cách nhận ra các mẫu chung.

Vì lý do này, "chi phí" thực sự của một nhánh phần lớn phụ thuộc vào việc nó có dễ dự đoán bởi CPU hay không. Nếu việc dự đoán là ngẫu nhiên 50/50 thuần túy, bạn phải chịu một [mối nguy điều khiển](../hazards) và loại bỏ toàn bộ đường ống, mất thêm 15-20 chu kỳ để xây dựng lại. Và nếu nhánh luôn luôn hoặc không bao giờ được thực hiện, bạn hầu như không phải trả giá gì ngoại trừ việc kiểm tra điều kiện.

## Thử nghiệm

Chúng ta sẽ thử tạo một mảng các số nguyên ngẫu nhiên trong khoảng từ 0 tới 99:
```c++
for (int i = 0; i < N; i++)
    a[i] = rand() % 100;
```

Sau đó chúng ta sẽ thực hiện vòng lặp để cộng tổng các số nhỏ hơn 50 lại:
```c++
volatile int s;

for (int i = 0; i < N; i++)
    if (a[i] < 50)
        s += a[i];
```

Ta gán $N = 10^6$ và chạy vòng lặp này nhiều lần để hiệu ứng [cold cache](/hpc/cpu-cache/bandwidth) không ảnh hưởng tới kết quả. Ta đánh dấu biến cộng dồn là `volatile` (bắt buộc phải ghi giá trị vào bộ nhớ) để trình biên dịch không vec-tơ hóa vòng lặp, hoặc tối ưu hóa vòng lặp bằng cách này hay cách khác.

Với Clang, mã hợp ngữ của đoạn code trên sẽ như sau:

```nasm
    mov  rcx, -4000000  ; 10^6 * 4-byte
    jmp  body
counter:
    add  rcx, 4
    jz   finished       ; "jump if rcx became zero"
body:
    mov  edx, dword ptr [rcx + a + 4000000]
    cmp  edx, 49
    jg   counter
    add  dword ptr [rsp + 12], edx
    jmp  counter
```

Mục đích của chúng ta là mô phỏng lại hoàn toàn rẽ nhánh không dự đoán được, và đã khiến CPU phải dùng tới khoảng 14 chu kỳ cho một phần tử của vòng lặp. Chúng ta có thể giả sử rằng các nhánh được chuyển liên tục giữa `<` và `>=` và đường ống dự đoán sai ở mọi lần lặp. Vì thế ở mỗi hai lần lặp:

- Ta xả đường ống, khoảng 19 chu kỳ với Zen 2
- Ta cần một lệnh tải bộ nhớ và 1 lệnh so sánh, chúng tốn khoảng 5 chu kỳ. Ta có thể kiểm tra điều kiện của lần lặp chẵn và lẻ đồng thời, vì thế giả sử chúng chỉ tốn một lần cho mỗi 2 lần lặp.
- Với nhánh `<`, ta cần khoảng 4 chu kỳ nữa cho việc cộng `a[i]` vào biến volatile (lưu trong bộ nhớ) `s`.


Vì thế, trung bình, chúng ta cần $(4 + 5 + 19) / 2 = 14$ chu kỳ cho một lần lặp, trùng với những gì đo đạc được.

### Dự đoán nhánh

Ta có thể thây thế `50` bằng một tham số có thể điều chỉnh `P` để thay đổi xác suất của nhánh `<`:
```c++
for (int i = 0; i < N; i++)
    if (a[i] < P)
        s += a[i];
```

Với những giá trị khác nhau của `P`, ta được đồ thị như sau:

![](../img/probabilities.svg)

Đỉnh điểm của nó là 50-55%, đúng như dự đoán: sự sai lệch chi nhánh là thứ đắt nhất ở đây. Đồ thị này không đối xứng: chỉ cần khoảng 1 chu kỳ để chỉ kiểm tra các điều kiện không bao giờ được thỏa mãn (`P = 0`) và khoảng 7 chu kỳ cho phép cộng nếu nhánh luôn được lấy (`P = 100`).

Biểu đồ này không phải là đơn phương thức: có một mức tối thiểu cục bộ khác vào khoảng 85-90%. Chúng sử dụng khoảng 6,15 chu kỳ cho lần lặp - nhanh hơn khoảng 10-15% so với khi chúng luôn lấy nhánh, do đó chúng cần thực hiện ít bổ sung hơn. Sự sai lệch nhánh ngừng ảnh hưởng đến hiệu suất tại đây bởi vì không phải toàn bộ bộ đệm lệnh bị loại bỏ, mà chỉ các hoạt động đã được dự đoán lên lịch. Về cơ bản, tỷ lệ dự đoán sai 10-15% đó là điểm cân bằng mà chúng ta có thể nhìn thấy đủ xa trong đường ống để không bị đình trệ nhưng vẫn tiết kiệm 10-15% khi sử dụng nhánh `>=` rẻ hơn.

Lưu ý rằng hầu như không tốn chi phí để kiểm tra điều kiện không bao giờ hoặc hầu như không bao giờ xảy ra. Đây là lý do tại sao các lập trình viên sử dụng các ngoại lệ thời gian chạy và kiểm tra trường hợp cơ sở rất nhiều: nếu chúng thực sự hiếm, chúng thực sự không tốn kém gì.

### Phát hiện mẫu

Trong ví dụ trên, mọi thứ cần thiết để dự đoán nhánh hiệu quả là một bộ đếm thống kê phần cứng. Nếu trước đây chúng ta sử dụng nhánh A thường xuyên hơn nhánh B, thì việc suy đoán nhánh A sẽ được thực thi tiếp là có lý. Bộ dự đoán nhánh trên các CPU hiện đại tiên tiến hơn đáng kể và có thể phát hiện các mẫu phức tạp hơn nhiều.

Hãy gán `P` trở lại 50, và sau đó sắp xếp mảng đầu tiên trước vòng lặp tổng kết chính:
```c++
for (int i = 0; i < N; i++)
    a[i] = rand() % 100;

std::sort(a, a + n);
```

Chúng ta vẫn đang xử lý các phần tử giống nhau, nhưng theo một thứ tự khác và thay vì 14 chu kỳ, giờ nó chạy nhiều hơn 4 một chút, chính xác là giá trị trung bình của chi phí thuần túy của nhánh `<` và nhánh `>=`.

Công cụ dự đoán nhánh có thể chọn các mẫu phức tạp hơn nhiều so với chỉ "luôn bên trái, rồi luôn bên phải" hoặc "trái-phải-trái-phải". Nếu chúng ta giảm kích thước của mảng $N$ xuống 1000 (mà không cần sắp xếp nó), thì công cụ dự đoán nhánh sẽ ghi nhớ toàn bộ chuỗi so sánh và điểm chuẩn đo lường lại ở khoảng 4 chu kỳ - trên thực tế, thậm chí còn ít hơn một chút so với trường hợp mảng được sắp xếp, bởi vì trong trường hợp trước đây, bộ dự đoán nhánh cần dành một chút thời gian để di chuyển giữa trạng thái "luôn có" và "luôn không".

### Gợi ý khả năng xuất hiện của các chi nhánh

Nếu bạn biết trước nhánh nào có nhiều khả năng hơn, bạn có thể [chuyển thông tin đó](/hpc/compilation/situational) đến trình biên dịch:
```c++
for (int i = 0; i < N; i++)
    if (a[i] < P) [[likely]]
        s += a[i];
```

Khi `P = 75`, mất khoảng 7,3 chu kỳ cho mỗi lần lặp, trong khi phiên bản gốc không có gợi ý cần khoảng 8,3.

Gợi ý này không loại bỏ nhánh hoặc giao tiếp bất cứ điều gì với bộ dự đoán nhánh, nhưng nó thay đổi [bố cục mã máy](/hpc/architecture/layout) theo cách cho phép CPU front-end xử lý nhánh có nhiều khả năng nhanh hơn một chút (mặc dù thường không quá một chu kỳ).

Việc tối ưu hóa này chỉ có lợi khi bạn biết nhánh nào có nhiều khả năng được thực hiện hơn trước giai đoạn biên dịch. Khi về cơ bản, nhánh là không thể đoán trước được, chúng ta có thể cố gắng loại bỏ nó hoàn toàn bằng cách sử dụng *dự đoán* - một kỹ thuật cực kỳ quan trọng mà chúng ta sẽ khám phá trong [phần tiếp theo](../branchless).

### Sự ghi nhận

Nghiên cứu điển hình này được lấy cảm hứng từ [câu hỏi Stack Overflow được ủng hộ nhiều nhất từ trước đến nay](https://stackoverflow.com/questions/11227809/why-is-processing-a-sorted-array-faster-than-processing-an-unsorted-array).
