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

### Moving Data

Some instructions may have the same mnemonic, but have different operand types, in which case they are considered distinct instructions as they may perform slightly different operations and take different times to execute. The `mov` instruction is a vivid example of that, as it comes in around 20 different forms, all related to moving data: either between the memory and registers or just between two registers. Despite the name, it doesn't *move* a value into a register, but *copies* it, preserving the original.

When used to copy data between two registers, the `mov` instruction instead performs *register renaming* internally — informs the CPU that the value referred by register X is actually stored in register Y — without causing any additional delay except for maybe reading and decoding the instruction itself. For the same reason, the `xchg` instruction that swaps two registers also doesn't cost anything.

As we've seen above with the fused `add`, you don't have to use `mov` for every memory operation: some arithmetic instructions conveniently support memory locations as operands.

<!--

Some operations are fused like `add r m` or `inc m` (this is one of the rare instructions that doesn't use any register values as operands).

When address is used,

Mirroring

-->

### Addressing Modes

Memory addressing is done with the `[]` operator, but it can do more than just reinterpret a value stored in a register as a memory location. The address operand takes up to 4 parameters presented in the syntax:

```
SIZE PTR [base + index * scale + displacement]
```

where `displacement` needs to be an integer constant and `scale` can be either 2, 4, or 8. What it does is calculate the pointer `base + index * scale + displacement` and dereferences it.

<!-- You can use them in any order: the assembler will figure it out. -->

Using complex addressing is [at most one cycle slower](/hpc/cpu-cache/pointers) than dereferencing a pointer directly, and it can be useful when you have, for example, an array of structures and want to load a specific field of its $i$-th element.

Addressing operator needs to be prefixed with a size specifier for how many bits of data are needed:

- `BYTE` for 8 bits
- `WORD` for 16 bits
- `DWORD` for 32 bits
- `QWORD` for 64 bits

There is also a more rare `TBYTE` for [80 bits](/hpc/arithmetic/float), and `XMMWORD`, `YMMWORD`, and `ZMMWORD` for [128, 256, and 512 bits](/hpc/simd) respectively. All these types don't have to be written in uppercase, but this is how most compilers emit them.

The address computation is often useful by itself: the `lea` ("load effective address") instruction calculates the memory address of the operand and stores it in a register in one cycle, without doing any actual memory operations. While its intended use is for actually computing memory addresses, it is also often used as an arithmetic trick that would otherwise involve 1 multiplication and 2 additions — for example, you can multiply by 3, 5, and 9 with it.

It also frequently serves as a replacement for `add` because it doesn't need a separate `mov` instruction if you need to move the result somewhere else: `add` only works in the two-register `a += b` mode, while `lea` lets you do `a = b + c` (or even `a = b + c + d` if one of them is a constant).

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
