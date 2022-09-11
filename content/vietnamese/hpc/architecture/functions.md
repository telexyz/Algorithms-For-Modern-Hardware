---
title: Hàm và hồi quy
weight: 3
published: true
---

Để "gọi một hàm" trong hợp ngữ, bạn cần [nhảy](../loop) về đầu đoạn code của hàm đó và sau đó nhảy trở lại (sau khi hàm thực thi xong). Nhưng sau đó hai vấn đề quan trọng nảy sinh:

1. Điều gì sẽ xảy ra nếu hàm được gọi lưu trữ dữ liệu trong cùng thanh ghi?
2. "Trở lại" đâu?

Cả hai mối quan tâm này đều có thể được giải quyết bằng cách có một vị trí dành riêng trong bộ nhớ, nơi chúng ta có thể ghi tất cả thông tin cần trả về từ hàm trước khi gọi nó. Vị trí này được gọi là *ngăn xếp*.

### Ngăn xếp

Ngăn xếp phần cứng hoạt động giống ngăn xếp phần mềm và được triển khai tương tự với hai con trỏ:

- *Con trỏ cơ sở* đánh dấu điểm bắt đầu của ngăn xếp và được lưu trữ theo quy ước trong `rbp`.
- *Con trỏ ngăn xếp* đánh dấu phần tử cuối cùng của ngăn xếp và được lưu trữ theo quy ước trong `rsp`.

Khi bạn cần gọi một hàm, bạn đẩy tất cả các biến cục bộ của mình lên ngăn xếp (điều này bạn cũng có thể thực hiện trong các trường hợp khác; ví dụ: khi bạn hết thanh ghi), đẩy con trỏ lệnh hiện tại, rồi chuyển đến đầu (đoạn code) của hàm được gọi. Khi thoát khỏi một hàm, bạn nhìn vào con trỏ được lưu trữ trên đầu ngăn xếp, nhảy đến đó, sau đó đọc cẩn thận tất cả các biến được lưu trữ trên ngăn xếp trở lại các thanh ghi.

<!-- 
Khi một hàm bắt đầu, một *mở đầu hàm* được thực hiện: lưu con trỏ cơ sở trước đó trên ngăn xếp và đặt `rbp = rsp`.
 -->

Các tham số hàm và biến cục bộ được truy cập bằng cách cộng và trừ tương ứng, một độ lệch không đổi từ `ebp`.

Bản thân `ebp` trỏ đến con trỏ cơ sở của khung (chương trình, giống khung hình khi xem video) trước, cho phép stack walking trong trình gỡ lỗi và xem các biến cục bộ của khung khác. Chẳng hạn như dừng chương trình và xem hàm nào được gọi bởi hàm nào.

```nasm
push ebp      ; Lưu lại con trỏ của khung hiện tại
mov ebp, esp  ; Tạo con trỏ khung mới trỏ tới trên cùng ngăn xếp
sub esp, 20   ; Tạo ra 20 byte để lưu trữ cục bộ trong ngăn xếp
```

Nếu bạn bật chế độ tối ưu hóa khi biên dịch, nó sẽ bỏ qua con trỏ khung và sử dụng `ebp` như một thanh ghi bình thường và truy cập trực tiếp vào vùng nhớ cục bộ thông qua `esp`, nhưng điều này làm cho việc gỡ lỗi trở nên khó khăn hơn một chút vì trình gỡ lỗi không còn có thể truy cập trực tiếp vào các khung ngăn xếp của các lệnh gọi hàm trước đó nữa.

Bạn có thể thực hiện tất cả những điều đó với các thao tác và lệnh nhảy thông thường trong bộ nhớ, nhưng do tần suất sử dụng nó, có 4 hướng dẫn đặc biệt để thực hiện việc này:

- `push` ghi dữ liệu tại con trỏ ngăn xếp và giảm nó.
- `pop` đọc dữ liệu từ con trỏ ngăn xếp và tăng nó.
- `call` đặt địa chỉ của lệnh sau lên trên ngăn xếp và nhảy đến một nhãn.
- `ret` đọc địa chỉ trả về từ trên cùng của ngăn xếp và nhảy đến địa chỉ đó.

Bạn sẽ gọi chúng là "syntactic sugar" nếu chúng không phải là hướng dẫn phần cứng thực sự - chúng chỉ là những phần tương đương hợp nhất của các đoạn mã hai lệnh sau:
```nasm
; "push rax"
sub rsp, 8
mov QWORD PTR[rsp], rax

; "pop rax"
mov rax, QWORD PTR[rsp]
add rsp, 8

; "call func"
push rip ; <- con trỏ lệnh (mặc dù truy cập nó như vậy có thể là không được phép)
jmp func

; "ret"
pop  rcx ; <- chọn bất kỳ thanh ghi nào chưa sử dụng
jmp rcx
```

Miền bộ nhớ giữa `rbp` và `rsp` được gọi là *khung ngăn xếp (stack frame)*, và đây là nơi các biến cục bộ của hàm được lưu trữ. Nó được phân bố khi chương trình bắt đầu chạy, và nếu bạn đẩy nhiều dữ liệu vào ngăn xếp hơn dung lượng của nó (8MB theo mặc định trên Linux), bạn sẽ gặp phải lỗi *tràn ngăn xếp*. (`rbp`: begin pointer, `rsp`: stack pointer)

Bởi vì các hệ điều hành hiện đại không thực sự cung cấp cho bạn các trang bộ nhớ cho đến khi bạn đọc hoặc ghi vào không gian địa chỉ của chúng, bạn có thể tự do chỉ định kích thước ngăn xếp rất lớn, hoạt động giống như một giới hạn về lượng bộ nhớ ngăn xếp có thể được sử dụng chứ không phải con số cố định mà mọi chương trình phải sử dụng.

Thật tiện lợi khi lưu con trỏ khung `rbp` ở đầu một hàm và thay thế nó bằng `rsp` - bằng cách này, khi rời khỏi một hàm, bạn có thể khôi phục lại `rbp` và quên tất cả các biến cục bộ của nó. Chuỗi này được gọi là *mở đầu hàm* và trông giống như đoạn mã sau (thường được trình biên dịch tối ưu hóa):

```nasm
push rbp     ; bảo tồn con trỏ khung hiện tại
mov rbp, rsp ; tạo một con trỏ khung mới trỏ đến đỉnh hiện tại của ngăn xếp
sub rsp, 20  ; cấp phát 20 byte trong ngăn xếp cho các biến cục bộ
```

<!--

Note that the data in the stack is written top-to-bottom. This is just a convention: it could be the other way around. When you need to "leave" a function or a visibility scope such as the body of an `if` or a `for`, you can just increase the stack pointer.

Lưu ý rằng dữ liệu trong ngăn xếp được ghi từ trên xuống dưới. Đây chỉ là một quy ước: nó có thể ngược lại. Khi bạn cần "rời khỏi" một hàm hoặc một phạm vi (scope), chẳng hạn như phần thân của dấu `if` hoặc `for`, bạn chỉ cần tăng con trỏ ngăn xếp.

 -->

 ### Quy ước gọi hàm

Những người phát triển trình biên dịch và hệ điều hành cuối cùng đã đưa ra [quy ước](https://wiki.osdev.org/Calling_Conventions) về cách viết và gọi các hàm. Những quy ước này giúp hình thành một số [kỳ công kỹ nghệ phần mềm quan trọng](/hpc/compilation/stage) như chia việc biên dịch thành các đơn vị riêng biệt, sử dụng lại các thư viện đã được biên dịch và thậm chí viết chúng bằng các ngôn ngữ lập trình khác nhau.

Hãy xem xét ví dụ sau trong C:
```c
int square(int x) {
    return x * x;
}

int distance(int x, int y) {
    return square(x) + square(y);
}
```

Theo quy ước, một hàm phải nhận các tham số của nó trong `rdi`,`rsi`, `rdx`, `rcx`, `r8`, `r9` (và phần còn lại trong ngăn xếp nếu chúng không đủ), đặt giá trị trả về vào `rax`, rồi trả về. Do đó, `square`, là một hàm một tham số đơn giản, có thể được triển khai như thế này:
```nasm
square:             ; x = edi, ret = eax
    imul edi, edi
    mov  eax, edi
    ret
```

Each time we call it from `distance`, we just need to go through some trouble preserving its local variables:

Mỗi lần chúng ta gọi nó từ `distance`, chúng ta cần bảo toàn các biến cục bộ của nó:
```nasm
distance:           ; x = rdi/edi, y = rsi/esi, ret = rax/eax
    push rdi
    push rsi
    call square     ; eax = square(x)
    pop  rsi
    pop  rdi

    mov  ebx, eax   ; save x^2
    mov  rdi, rsi   ; move new x=y

    push rdi
    push rsi
    call square     ; eax = square(x=y)
    pop  rsi
    pop  rdi

    add  eax, ebx   ; x^2 + y^2
    ret
```

Có rất nhiều sắc thái khác, nhưng chúng ta sẽ không đi vào chi tiết vì cuốn sách này nói về hiệu suất và cách tốt nhất để giải quyết các lệnh gọi hàm thực sự là tránh thực hiện chúng ngay từ đầu.

### Nội tuyến (Inlining)

Việc di chuyển dữ liệu vào và ra khỏi ngăn xếp tạo ra chi phí đáng kể cho các nhỏ. Lý do bạn phải làm điều này là, nói chung, bạn không biết liệu hàm được gọi có đang sửa đổi các thanh ghi nơi bạn lưu trữ các biến cục bộ của mình hay không. Nhưng khi bạn biết được hàm `square` thực sự làm gì, bạn có thể giải quyết vấn đề này bằng cách lưu trữ dữ liệu trong các thanh ghi mà bạn biết rằng sẽ không bị sửa đổi.
```nasm
distance:
    call square
    mov  ebx, eax
    mov  edi, esi
    call square
    add  eax, ebx
    ret
```

This is better, but we are still implicitly accessing stack memory: you need to push and pop the instruction pointer on each function call. In simple cases like this, we can *inline* function calls by stitching the callee's code into the caller and resolving conflicts over registers. In our example:

Đã tốt hơn, nhưng chúng ta vẫn đang truy cập ngầm vào bộ nhớ ngăn xếp: bạn cần phải đẩy và lấy con trỏ hướng dẫn trên mỗi lệnh gọi hàm. Trong những trường hợp đơn giản như thế này, chúng ta có thể gọi hàm *nội tuyến* bằng cách ghép mã của hàm được gọi vào hàm gọi và giải quyết xung đột trên các thanh ghi:
```nasm
distance:
    imul edi, edi       ; edi = x^2
    imul esi, esi       ; esi = y^2
    add  edi, esi
    mov  eax, edi       ; there is no "add eax, edi, esi", so we need a separate mov
    ret
```

This is fairly close to what optimizing compilers produce out of this snippet — only they use the [lea trick](../assembly) to make the resulting machine code sequence a few bytes smaller:
Điều này khá gần với những gì mà trình biên dịch tối ưu hóa tạo - chỉ có điều họ sử dụng [lea trick](../assembly) để làm cho chuỗi mã máy nhỏ hơn vài byte:
```nasm
distance:
    imul edi, edi       ; edi = x^2
    imul esi, esi       ; esi = y^2
    lea  eax, [rdi+rsi] ; eax = x^2 + y^2
    ret
```

In situations like these, function inlining is clearly beneficial, and compilers mostly do it [automatically](/hpc/compilation/situational), but there are cases when it's not — and we will talk about them [in a bit](../layout).

Trong những tình huống như thế này, nội tuyến hàm rõ ràng là có lợi và các trình biên dịch chủ yếu làm điều đó [tự động](/hpc/compilation/situational), nhưng có những trường hợp thì không - và chúng ta sẽ nói về chúng [sau một chút](../layout).

### Loại bỏ lệnh gọi đuôi

Nội tuyến được thực hiện dễ dàng khi hàm được gọi không thực hiện bất kỳ lệnh gọi hàm nào khác, hoặc ít nhất nếu có thì các lệnh gọi này không phải là đệ quy. Hãy chuyển sang một ví dụ phức tạp hơn. Hãy xem xét phép tính đệ quy khi tính giai thừa:
```cpp
int factorial(int n) {
    if (n == 0)
        return 1;
    return factorial(n - 1) * n;
}
```

Mã hợp ngữ tương ứng:
```nasm
; n = edi, ret = eax
factorial:
    test edi, edi   ; test if a value is zero
    jne  nonzero    ; (the machine code of "cmp rax, 0" would be one byte longer)
    mov  eax, 1     ; return 1
    ret
nonzero:
    push edi        ; save n to use later in multiplication
    sub  edi, 1
    call factorial  ; call f(n - 1)
    pop  edi
    imul eax, edi
    ret
```

Vẫn có thể làm cho hàm đệ quy trở thành "call-less" bằng cách cấu trúc lại nó. Đây là trường hợp khi hàm là *đệ quy đuôi*, tức là nó trả về ngay sau khi thực hiện một lệnh gọi đệ quy. Vì không có hành động nào được yêu cầu sau lệnh gọi, nên cũng không cần lưu trữ bất kỳ thứ gì trên ngăn xếp và một lệnh gọi đệ quy có thể được thay thế một cách an toàn bằng một bước nhảy về đầu - biến hàm thành một vòng lặp một cách hiệu quả.

Để làm cho hàm `giai thừa` của chúng ta có dạng đệ quy đuôi, chúng ta có thể truyền đối số là "phép nhân hiện tại" cho nó:
```cpp
int factorial(int n, int p = 1) {
    if (n == 0)
        return p;
    return factorial(n - 1, p * n);
}
```

Hàm này có thể dễ dàng được xếp lại thành một vòng lặp:
```nasm
; assuming n > 0
factorial:
    mov  eax, 1
loop:
    imul eax, edi
    sub  edi, 1
    jne  loop
    ret
```

Lý do chính tại sao hàm đệ quy có thể chậm là nó cần đọc và ghi dữ liệu vào ngăn xếp, trong khi các thuật toán đệ quy lặp lại và đệ quy đuôi thì không. Khái niệm này rất quan trọng trong lập trình hàm, nơi không có vòng lặp và tất cả những gì bạn có thể sử dụng là các hàm. Nếu không loại bỏ lệnh gọi đuôi, các chương trình chức năng sẽ đòi hỏi nhiều thời gian và bộ nhớ hơn để thực thi.