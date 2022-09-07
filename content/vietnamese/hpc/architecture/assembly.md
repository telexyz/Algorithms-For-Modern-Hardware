---
title: Hợp ngữ
weight: 1
published: true
---

CPU được điều khiển bởi ngôn ngữ máy - là một luồng mã lệnh nhị phân, mỗi lệnh bao gồm:
- mã số lệnh (hay còn gọi là *opcode*)
- *toán hạng* của nó là gì (nếu có)
- và nơi lưu trữ *kết quả* (nếu được tạo ra).

Một phiên bản ngôn ngữ máy thân thiện với con người hơn nhiều, được gọi là *hợp ngữ*, sử dụng các mã ghi nhớ để tham chiếu đến các lệnh mã máy, và các tên tượng trưng để tham chiếu đến các thanh ghi và các vị trí lưu trữ khác.

Đây là cách bạn cộng hai số (`*c = *a + *b`) trong hợp ngữ Arm:
```nasm
; *a = x0, *b = x1, *c = x2
ldr w0, [x0]    ; nạp 4 bytes từ vị trí x0 trỏ tới vào w0
ldr w1, [x1]    ; nạp 4 bytes từ vị trí x1 trỏ tới vào w1
add w0, w0, w1  ; cộng w0 với w1 và lưu kết quả vào w0
str w0, [x2]    ; ghi nội dung của vào vị trí x2 trỏ tới
```

Dưới đây là đoạn hợp ngữ x86 tương tự:
```nasm
; *a = rsi, *b = rdi, *c = rdx 
mov eax, DWORD PTR [rsi]  ; nạp 4 bytes từ vị trí rsi trỏ tới vào eax
add eax, DWORD PTR [rdi]  ; cộng nội dung ở vị trí được rdi trỏ tới vào eax
mov DWORD PTR [rdx], eax  ; ghi nội dung của eax vào nơi rdx trỏ tới
```

Hợp ngữ rất đơn giản theo nghĩa là nó không có nhiều cấu trúc cú pháp so với các ngôn ngữ lập trình cấp cao. Từ những gì bạn có thể quan sát từ các ví dụ trên:

- Chương trình là một dãy các lệnh, mỗi lệnh được viết như tên của nó, theo sau là một số toán hạng.
- Cú pháp `[thanh-ghi]` được sử dụng để "tham chiếu" một con trỏ được lưu trữ trong một thanh ghi; và với x86, bạn cần thêm thông tin kích thước vào trước (`DWORD` ở đây có nghĩa là 32 bit).
- Dấu `;` được sử dụng cho các chú thích dòng, tương tự như `#` và `//` trong các ngôn ngữ khác.

Assembly là một ngôn ngữ rất tối thiểu vì nó cần phản ánh ngôn ngữ máy càng chặt chẽ càng tốt, đến mức gần như có sự tương ứng 1:1 giữa mã máy và hợp ngữ. Trên thực tế, bạn có thể biến bất kỳ chương trình đã biên dịch nào trở lại dạng hợp ngữ bằng cách sử dụng một quy trình được gọi là *tháo rời* [^disassembly].

[^disassembly]: Trên Linux, để tháo rời một chương trình đã biên dịch, bạn có thể gọi `objdump -d {path-to-binary}`.

Lưu ý rằng hai đoạn mã trên không chỉ khác nhau về mặt cú pháp. Cả hai đều là mã được tối ưu hóa do trình biên dịch tạo ra, nhưng phiên bản Arm sử dụng 4 lệnh, trong khi phiên bản x86 sử dụng 3. Lệnh `add eax, [rdi]` được gọi là *lệnh hợp nhất* thực hiện lệnh tải và lệnh cộng chỉ trong một lần - đây là một trong những đặc thù của [CISC](../isa#risc-vs-cisc).

### Chỉ lệnh và thanh ghi

Vì lý do lịch sử, chỉ lệnh trong hầu hết các hợp ngữ đều rất ngắn gọn. Khi mọi người thường viết hợp ngữ bằng tay và viết nhiều lần cùng một tập hợp các hướng dẫn thông dụng, chỉ cần bớt đi một ký tự phải gõ là đã đỡ tốn rất nhiều công sức.

Ví dụ: `mov` để "lưu/tải một từ", `inc` để "tăng thêm 1", `mul` là "nhân" và `idiv` dành cho "phép chia số nguyên". Bạn có thể tra cứu mô tả của chỉ lệnh theo tên của tại [một trong những tài liệu tham khảo x86](https://www.felixcloutier.com/x86/).

Hầu hết các lệnh đều ghi kết quả của chúng vào toán hạng đầu tiên, toán hạng này cũng có thể tham gia vào việc tính toán như trong ví dụ `add eax, [rdi]` mà chúng ta đã thấy trước đây. Toán hạng có thể là thanh ghi, giá trị không đổi hoặc vị trí bộ nhớ.

**Các thanh ghi** được đặt tên là `rax`,` rbx`, `rcx`,` rdx`, `rdi`,` rsi`, `rbp`,` rsp` và `r8`-`r15`, tổng số 16 thanh ghi. Những "chữ cái" được đặt tên như vậy vì lý do lịch sử: `rax` là "bộ tích lũy", `rcx` là "bộ đếm", `rdx` là "dữ liệu", v.v. - nhưng tất nhiên, chúng còn được sử dụng cho nhiều mục đích khác nữa.

Ngoài ra còn có các thanh ghi 32-, 16-bit và 8-bit có tên tương tự (`rax` → `eax` → `ax` → `al`). Chúng không hoàn toàn tách biệt mà là *bí danh*: 32 bit thấp nhất của `rax` là` eax`, 16 bit thấp nhất của `eax` là` ax`, v.v. Chúng dùng để duy trì khả năng tương thích, và đó cũng là lý do tại sao các casting các kiểu cơ bản trong các ngôn ngữ lập trình biên dịch thường không mất thêm chi phí (vì dữ liệu đã nằm sẵn trong thanh ghi và chỉ việc gọi đúng tên thanh ghi tương ứng với kiểu ta muốn).

Đây là các thanh ghi mà bạn có thể sử dụng cho *mục đích chung* - sử dụng theo cách bạn muốn trong hầu hết các chỉ lệnh (cũng có [một số ngoại lệ](../functions)). Ngoài ra còn có một tập hợp các thanh ghi riêng cho [số học dấu phẩy động](/hpc/arithmetic/float), một loạt các thanh ghi rất rộng được sử dụng trong [SIMD](/hpc/simd) và một số thanh ghi đặc biệt cần thiết cho [điều khiển luồng](../loops).

**Hằng số** chỉ là giá trị số nguyên hoặc dấu phẩy động: `42`,` 0x2a`, `3,14`,` 6,02e23`. Chúng thường được gọi là *giá trị tức thì* vì chúng được nhúng ngay vào mã máy. Bởi vì nó có thể làm tăng đáng kể độ phức tạp của mã hóa lệnh, một số lệnh không hỗ trợ các giá trị tức thì hoặc chỉ cho phép một tập con cố định của chúng. Trong một số trường hợp, bạn phải tải một giá trị không đổi vào một thanh ghi và sau đó sử dụng nó thay vì một giá trị tức thì. Ngoài các giá trị số, còn có các hằng số chuỗi như `hello` hoặc` world\n` với các phép toán của riêng chúng.

### Di chuyển dữ liệu

Một số lệnh có thể có cùng cách ghi nhớ, nhưng có các kiểu toán hạng khác nhau, trong trường hợp đó, chúng được coi là các lệnh riêng biệt vì chúng có thể thực hiện các thao tác hơi khác nhau và mất thời gian thực hiện khác nhau. Lệnh `mov` là một ví dụ sinh động về điều đó vì nó có khoảng 20 dạng khác nhau, tất cả đều liên quan đến dữ liệu di chuyển: giữa bộ nhớ và thanh ghi hoặc chỉ giữa hai thanh ghi. Mặc dù tên là di chuyển nhưng nó không *di chuyển* mà *sao chép* một giá trị vào một thanh ghi, và giữ nguyên bản gốc.

Khi được sử dụng để sao chép dữ liệu giữa hai thanh ghi, lệnh `mov` sẽ thực hiện *đổi tên thanh ghi*  và thông báo cho CPU rằng giá trị được tham chiếu bởi thanh ghi X thực sự được lưu trữ trong thanh ghi Y - mà không gây thêm bất kỳ độ trễ nào ngoại trừ việc đọc và giải mã chỉ lệnh. Vì lý do tương tự, lệnh `xchg` hoán đổi hai thanh ghi cũng không tốn bất kỳ chi phí nào.

Như chúng ta đã thấy ở trên với `add` hợp nhất, bạn không cần phải sử dụng `mov` cho mọi thao tác liên quan tới bộ nhớ: một số lệnh số học có hỗ trợ các toán hạng dưới dạng vị trí bộ nhớ.

### Chế độ địa chỉ

Việc xác định địa chỉ bộ nhớ được thực hiện với toán tử `[]`, nhưng nó có thể làm được nhiều việc hơn là chỉ diễn giải một vị trí bộ nhớ được lưu trữ trong thanh ghi. Toán hạng địa chỉ có tối đa 4 tham số được trình bày theo cú pháp:
``
KÍCH THƯỚC PTR [cơ sở + chỉ số * tỷ lệ + dịch chuyển]
``

trong đó `dịch chuyển` cần là một hằng số nguyên và `tỷ lệ` có thể là 2, 4 hoặc 8. Những gì nó làm là tính toán con trỏ `cơ sở + chỉ số * tỷ lệ + dịch chuyển` và tham chiếu đến nó.

<!-- You can use them in any order: the assembler will figure it out. -->

Sử dụng địa chỉ phức tạp [chậm hơn tối đa một chu kỳ](/hpc/cpu-cache/pointers) so với tham chiếu trực tiếp con trỏ và nó có thể hữu ích khi bạn có, chẳng hạn như một mảng cấu trúc và muốn tải một trường cụ thể của phần tử thứ $i$.

Toán tử định địa chỉ cần được bắt đầu bằng một mã xác định số lượng bit của dữ liệu:

- `BYTE` cho 8 bits
- `WORD` cho 16 bits
- `DWORD` cho 32 bits
- `QWORD` cho 64 bits

Ngoài ra còn có `TBYTE` cho [80 bit](/hpc/arithmetic/float) và `XMMWORD`, `YMMWORD` và` ZMMWORD` tương ứng với [128, 256 và 512 bit](/hpc/simd).

Lệnh `lea` ("load effective address") tính toán địa chỉ bộ nhớ của toán hạng và lưu trữ nó trong một thanh ghi được thực hiện trong một chu kỳ mà không cần thực hiện bất kỳ thao tác bộ nhớ nào. Mặc dù mục đích sử dụng của nó là để tính toán các địa chỉ bộ nhớ, nhưng nó cũng thường được sử dụng như một thủ thuật số học liên quan đến 1 phép nhân và 2 phép cộng - ví dụ: bạn có thể nhân với 3, 5 và 9 với nó.

Nó cũng thường thay thế cho `add` vì nó không cần lệnh `mov` riêng biệt nếu bạn cần di chuyển kết quả đến một nơi khác thì `add` chỉ hoạt động trong chế độ hai thanh ghi `a += b` , trong khi `lea` cho phép bạn thực hiện `a = b + c` (hoặc thậm chí `a = b + c + d` nếu một trong số chúng là hằng số).

### Cú pháp thay thế

Thực tế có nhiều *chương trình lắp ráp* (`asemblers`: chương trình tạo ra mã máy từ hợp ngữ) với các hợp ngữ khác nhau, nhưng hiện nay chỉ có hai cú pháp x86 được sử dụng rộng rãi. Chúng thường được gọi theo tên của hai công ty sử dụng chúng và có ảnh hưởng chi phối đến lĩnh vực lập trình trong thời đại đó:

- Cú pháp *AT&T*, được sử dụng theo mặc định bởi tất cả các công cụ Linux.
- Cú pháp *Intel*, được sử dụng theo mặc định bởi Intel.

Các cú pháp này đôi khi cũng được gọi tương ứng là *GAS* và *NASM*, theo tên của hai chương trình lắp ráp chính sử dụng chúng (*GNU Assembler* và *Netwide Assembler*).

Chúng tôi đã sử dụng cú pháp Intel trong chương này và sẽ tiếp tục sử dụng nó cho phần còn lại của cuốn sách. Để so sánh, đây là ví dụ `*c = *a + *b` giống như thế nào trong AT&T asm:

```asm
movl (%rsi), %eax
addl (%rdi), %eax
movl %eax, (%rdx)
```

Những khác biệt chính có thể được tóm tắt như sau:

1. Toán hạng *cuối cùng* được sử dụng để chỉ định đích.
2. Thanh ghi và hằng số cần phải có tiền tố tương ứng là `%` và `$` (ví dụ: `addl $ 1,% rdx` gia số `rdx`).
3. Định địa chỉ bộ nhớ có dạng như sau: `displacement (%base, %index, scale)`.
4. Cả `;` và `#` đều có thể được sử dụng cho các bình luận dòng, và `/* */` cũng có thể được sử dụng cho các bình luận khối.

Và quan trọng nhất, trong cú pháp AT&T, các tên lệnh cần được đặt "hậu tố" (`addq`, `movl`, `cmpq`, v.v.) để chỉ định kích thước của toán hạng đang được thao tác:
- `b` = byte (8 bit)
- `w` = word (16 bit)
- `l` = long (32 bit integer or 64-bit floating-point)
- `q` = quad (64 bit)
- `s` = single (32-bit floating-point)
- `t` = ten bytes (80-bit floating-point)

Trong cú pháp Intel, những thông tin trên được suy ra từ toán hạng (đó là lý do tại sao bạn phải chỉ định kích thước con trỏ).