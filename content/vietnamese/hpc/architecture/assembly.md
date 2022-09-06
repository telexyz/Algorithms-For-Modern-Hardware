---
title: Hợp ngữ
weight: 1
published: true
---

CPU được điều khiển bởi ngôn ngữ máy - một luồng mã lệnh nhị phân chỉ định:
- mã số lệnh (hay còn gọi là *opcode*)
- *toán hạng* của nó là gì (nếu có)
- và nơi lưu trữ *kết quả* (nếu được tạo ra).


Một phiên bản ngôn ngữ máy thân thiện với con người hơn nhiều, được gọi là *hợp ngữ*, sử dụng các mã ghi nhớ để tham chiếu đến các lệnh mã máy và các tên tượng trưng để tham chiếu đến các thanh ghi và các vị trí lưu trữ khác.

Và đây là cách bạn cộng hai số (`*c = *a + *b`) trong hợp ngữ Arm:
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

Assembly là một ngôn ngữ rất tối thiểu vì nó cần phải như vậy. Nó phản ánh ngôn ngữ máy càng chặt chẽ càng tốt, đến mức gần như có sự tương ứng 1: 1 giữa mã máy và hợp ngữ. Trên thực tế, bạn có thể biến bất kỳ chương trình đã biên dịch nào trở lại dạng hợp ngữ của nó bằng cách sử dụng một quy trình được gọi là *tháo rời* [^disassembly] - mặc dù mọi thứ không cần thiết như nhận xét sẽ không được giữ nguyên.

[^disassembly]: Trên Linux, để tháo rời một chương trình đã biên dịch, bạn có thể gọi `objdump -d {path-to-binary}`.

Lưu ý rằng hai đoạn mã trên không chỉ khác nhau về mặt cú pháp. Cả hai đều là mã được tối ưu hóa do trình biên dịch tạo ra, nhưng phiên bản Arm sử dụng 4 lệnh, trong khi phiên bản x86 sử dụng 3. Lệnh `add eax, [rdi]` được gọi là *lệnh hợp nhất* thực hiện lệnh tải và lệnh cộng chỉ trong một lần - đây là một trong những đặc thù của [CISC](../isa#risc-vs-cisc).

### Chỉ lệnh và thanh ghi

Vì lý do lịch sử, chỉ lệnh trong hầu hết các ngôn ngữ hợp ngữ đều rất ngắn gọn. Khi mọi người thường viết hợp ngữ bằng tay và viết nhiều lần cùng một tập hợp các hướng dẫn thông dụng, chỉ cần bớt đi  một ký tự phải gõ là đỡ tốn rất nhiều công sức.

Ví dụ: `mov` để "lưu/tải một từ", `inc` để "tăng thêm 1", `mul` là "nhân" và `idiv` dành cho "phép chia số nguyên". Bạn có thể tra cứu mô tả của một hướng dẫn theo tên của nó trong [một trong các tài liệu tham khảo x86](https://www.felixcloutier.com/x86/).

Hầu hết các lệnh đều ghi kết quả của chúng vào toán hạng đầu tiên, toán hạng này cũng có thể tham gia vào việc tính toán như trong ví dụ `add eax, [rdi]` mà chúng ta đã thấy trước đây. Toán hạng có thể là thanh ghi, giá trị không đổi hoặc vị trí bộ nhớ.

**Các thanh ghi** được đặt tên là `rax`,` rbx`, `rcx`,` rdx`, `rdi`,` rsi`, `rbp`,` rsp` và `r8`-`r15`, tổng số 16 thanh ghi. Những "chữ cái" được đặt tên như vậy vì lý do lịch sử: `rax` là "bộ tích lũy", `rcx` là "bộ đếm", `rdx` là "dữ liệu", v.v. - nhưng tất nhiên, chúng còn được sử dụng cho nhiều mục đích khác nữa.

Ngoài ra còn có các thanh ghi 32-, 16-bit và 8-bit có tên tương tự (`rax` →` eax` → `ax` →` al`). Chúng không hoàn toàn tách biệt mà là *bí danh*: 32 bit thấp nhất của `rax` là` eax`, 16 bit thấp nhất của `eax` là` ax`, v.v. Điều này được thực hiện để tiết kiệm dung lượng khuôn chip trong khi duy trì khả năng tương thích, và đó cũng là lý do tại sao các casting các kiểu cơ bản trong các ngôn ngữ lập trình biên dịch thường không mất chi phí.

Đây chỉ là các thanh ghi *mục đích chung* mà bạn có thể, với [một số ngoại lệ](../functions), sử dụng theo cách bạn muốn trong hầu hết các chỉ lệnh. Ngoài ra còn có một tập hợp các thanh ghi riêng cho [số học dấu phẩy động] (/hpc/arithmetic/float), một loạt các thanh ghi rất rộng được sử dụng trong [SIMD] (/hpc/simd) và một số thanh ghi đặc biệt cần thiết cho [luồng điều khiển] (../loops).

**Hằng số** chỉ là giá trị số nguyên hoặc dấu phẩy động: `42`,` 0x2a`, `3,14`,` 6,02e23`. Chúng thường được gọi là *giá trị tức thì* vì chúng được nhúng ngay vào mã máy. Bởi vì nó có thể làm tăng đáng kể độ phức tạp của mã hóa lệnh, một số lệnh không hỗ trợ các giá trị tức thì hoặc chỉ cho phép một tập con cố định của chúng. Trong một số trường hợp, bạn phải tải một giá trị không đổi vào một thanh ghi và sau đó sử dụng nó thay vì một giá trị tức thì. Ngoài các giá trị số, còn có các hằng số chuỗi như `hello` hoặc` world\n` với tập con nhỏ các phép toán của riêng chúng.

### Di chuyển dữ liệu

Một số lệnh có thể có cùng cách ghi nhớ, nhưng có các kiểu toán hạng khác nhau, trong trường hợp đó, chúng được coi là các lệnh riêng biệt vì chúng có thể thực hiện các thao tác hơi khác nhau và mất thời gian thực hiện khác nhau. Lệnh `mov` là một ví dụ sinh động về điều đó, vì nó có khoảng 20 dạng khác nhau, tất cả đều liên quan đến dữ liệu di chuyển: giữa bộ nhớ và thanh ghi hoặc chỉ giữa hai thanh ghi. Mặc dù tên là di chuyển nhưng nó không *di chuyển* một giá trị vào một thanh ghi, mà *sao chép* nó, giữ nguyên bản gốc.

Khi được sử dụng để sao chép dữ liệu giữa hai thanh ghi, lệnh `mov` sẽ thực hiện *đổi tên thanh ghi*  và thông báo cho CPU rằng giá trị được tham chiếu bởi thanh ghi X thực sự được lưu trữ trong thanh ghi Y - mà không gây thêm bất kỳ độ trễ nào ngoại trừ việc có thể đọc và giải mã chỉ lệnh. Vì lý do tương tự, lệnh `xchg` hoán đổi hai thanh ghi cũng không tốn bất kỳ chi phí nào.

Như chúng ta đã thấy ở trên với `add` hợp nhất, bạn không cần phải sử dụng `mov` cho mọi thao tác trên bộ nhớ: một số lệnh số học hỗ trợ để cho thuận tiện đã  các toán hạng dưới dạng vị trí bộ nhớ.

### Chế độ địa chỉ

Việc xác định địa chỉ bộ nhớ được thực hiện với toán tử `[]`, nhưng nó có thể làm được nhiều việc hơn là chỉ diễn giải lại một  vị trí bộ nhớ được lưu trữ trong thanh ghi. Toán hạng địa chỉ có tối đa 4 tham số được trình bày trong cú pháp:
``
KÍCH THƯỚC PTR [cơ sở + chỉ số * tỷ lệ + dịch chuyển]
``

trong đó `dịch chuyển` cần là một hằng số nguyên và` tỷ lệ` có thể là 2, 4 hoặc 8. Những gì nó làm là tính toán con trỏ `cơ sở + chỉ số * tỷ lệ + dịch chuyển` và tham chiếu đến nó.

<!-- You can use them in any order: the assembler will figure it out. -->

Sử dụng địa chỉ phức tạp [chậm hơn tối đa một chu kỳ] (/hpc/cpu-cache/pointers) so với tham chiếu trực tiếp con trỏ và nó có thể hữu ích khi bạn có, chẳng hạn như một mảng cấu trúc và muốn tải một trường cụ thể của phần tử $i$-th.

Toán tử định địa chỉ cần được bắt đầu bằng một mã định kích thước cho số lượng bit của dữ liệu:

- `BYTE` cho 8 bits
- `WORD` cho 16 bits
- `DWORD` cho 32 bits
- `QWORD` cho 64 bits

Ngoài ra còn có `TBYTE` cho [80 bit](/hpc/arithmetic/float) và `XMMWORD`, `YMMWORD` và` ZMMWORD` tương ứng với [128, 256 và 512 bit](/hpc/simd).

Bản thân việc tính toán địa chỉ thường hữu ích: lệnh `lea` ("load effective address") tính toán địa chỉ bộ nhớ của toán hạng và lưu trữ nó trong một thanh ghi được thực hiện trong một chu kỳ mà không cần thực hiện bất kỳ thao tác bộ nhớ nào. Mặc dù mục đích sử dụng của nó là để tính toán các địa chỉ bộ nhớ, nhưng nó cũng thường được sử dụng như một thủ thuật số học liên quan đến 1 phép nhân và 2 phép cộng - ví dụ: bạn có thể nhân với 3, 5 và 9 với nó.

Nó cũng thường thay thế cho `add` vì nó không cần lệnh `mov` riêng biệt nếu bạn cần di chuyển kết quả đến một nơi khác: `add` chỉ hoạt động trong chế độ hai thanh ghi `a += b` , trong khi `lea` cho phép bạn thực hiện `a = b + c` (hoặc thậm chí `a = b + c + d` nếu một trong số chúng là hằng số).

### Alternative Syntax

There are actually multiple *assemblers* (the programs that produce machine code from assembly) with different assembly languages, but only two x86 syntaxes are widely used now. They are commonly called after the two companies that used them and had a dominant influence on programming during that era:

- The *AT&T syntax*, used by default by all Linux tools.
- The *Intel syntax*, used by default, well, by Intel.

These syntaxes are also sometimes called *GAS* and *NASM* respectively, by the names of the two primary assemblers that use them (*GNU Assembler* and *Netwide Assembler*).

We used Intel syntax in this chapter and will continue to preferably use it for the rest of the book. For comparison, here is how the same `*c = *a + *b` example looks like in AT&T asm:

```asm
movl (%rsi), %eax
addl (%rdi), %eax
movl %eax, (%rdx)
```

The key differences can be summarized as follows:

1. The *last* operand is used to specify the destination.
2. Registers and constants need to be prefixed by `%` and `$` respectively (e.g., `addl $1, %rdx` increments `rdx`).
3. Memory addressing looks like this: `displacement(%base, %index, scale)`.
4. Both `;` and `#` can be used for line comments, and also `/* */` can be used for block comments.

And, most importantly, in AT&T syntax, the instruction names need to be "suffixed" (`addq`, `movl`, `cmpq`, etc.) to specify what size operands are being manipulated:

- `b` = byte (8 bit)
- `w` = word (16 bit)
- `l` = long (32 bit integer or 64-bit floating-point)
- `q` = quad (64 bit)
- `s` = single (32-bit floating-point)
- `t` = ten bytes (80-bit floating-point)

In Intel syntax, this information is inferred from operands (which is why you also need to specify sizes of pointers).

Most tools that produce or consume x86 assembly can do so in both syntaxes, so you can just pick the one you like more and don't worry.
