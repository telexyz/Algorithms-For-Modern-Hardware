---
title: Bố cục mã máy
weight: 10
published: true
---

Các kỹ sư máy tính thích chia nhỏ [đường ống của CPU](/hpc/pipelining) thành hai phần: *front-end*, nơi các lệnh được tìm nạp từ bộ nhớ và được giải mã; và *back-end*, nơi các lệnh được lên lịch và thực thi. Thông thường, hiệu suất bị tắc nghẽn bởi giai đoạn thực thi và vì lý do này, hầu hết nỗ lực của chúng ta trong cuốn sách này sẽ được dành cho việc tối ưu hóa xung quanh phần cuối.

Nhưng đôi khi điều ngược lại có thể xảy ra khi front-end không cung cấp lệnh cho back-end đủ nhanh. Điều này có thể xảy ra vì nhiều lý do, cuối cùng tất cả đều liên quan đến cách mã máy được trình bày trong bộ nhớ và ảnh hưởng đến hiệu suất, chẳng hạn như xóa mã không sử dụng, hoán đổi các nhánh lệnh "nếu" hoặc thậm chí thay đổi thứ tự khai báo hàm khiến hiệu suất cải thiện hoặc xấu đi.

### CPU Front-End

Trước khi mã máy được chuyển thành các lệnh và CPU hiểu lập trình viên muốn gì, trước tiên nó cần phải trải qua hai giai đoạn quan trọng mà chúng ta quan tâm: *tìm nạp* và *giải mã*.

Trong giai đoạn **tìm nạp**, CPU chỉ cần tải một đoạn byte có kích thước cố định từ bộ nhớ chính, chứa các mã nhị phân của một số lệnh. Kích thước khối này thường là 32 byte trên x86, mặc dù nó có thể khác nhau trên các máy khác nhau. Một sắc thái quan trọng là khối này phải được [căn chỉnh](/hpc/cpu-cache/cache-lines): địa chỉ của đoạn phải bằng bội số kích thước của nó.

<!-- todo: what happens when an instruction crosses the boundary? -->

Tiếp theo là giai đoạn **giải mã**: CPU xem xét đoạn byte này, loại bỏ mọi thứ xuất hiện trước con trỏ lệnh và chia phần còn lại của chúng thành các lệnh. Các lệnh máy được mã hóa bằng cách sử dụng số byte thay đổi: những lệnh đơn giản và rất phổ biến như `inc rax` chiếm một byte, trong khi một số phức tạp với các hằng số được mã hóa và tiền tố sửa đổi hành vi có thể mất tới 15 byte. Vì vậy, từ một 32 byte khối,  nhiều hơn một lệnh có thể được giải mã, nhưng không vượt quá một giới hạn được gọi là *độ rộng giải mã*. Trên CPU của tôi ([Zen 2](https://en.wikichip.org/wiki/amd/microarchitectures/zen_2)), độ rộng giải mã là 4, có nghĩa là trên mỗi chu kỳ, có thể giải mã tối đa 4 lệnh và chuyển sang giai đoạn tiếp theo.

Các giai đoạn hoạt động theo kiểu đường ống: nếu CPU có thể nói (hoặc [dự đoán](/hpc/pipelining/branching/)) khối lệnh nào nó cần tiếp theo, thì giai đoạn tìm nạp không đợi lệnh cuối cùng trong hiện tại khối được giải mã và tải khối tiếp theo ngay lập tức.

<!--

Decoded Stream Buffer (DSB)

Loop Stream Detector (LSD)

-->

### Căn chỉnh mã

Những thứ khác tương đương nhau, các trình biên dịch thường thích các lệnh có mã máy ngắn hơn, bởi vì theo cách này, nhiều lệnh hơn có thể phù hợp với một khối tìm nạp 32B duy nhất và cũng vì nó làm giảm kích thước của tệp nhị phân. Nhưng đôi khi ngược lại được ưu tiên, do thực tế là các khối lệnh được tải về phải được căn chỉnh.

Hãy tưởng tượng rằng bạn cần thực hiện một chuỗi lệnh bắt đầu trên byte cuối cùng của khối 32B được căn chỉnh. Bạn có thể thực hiện lệnh đầu tiên mà không bị chậm trễ thêm, nhưng đối với các lệnh tiếp theo, bạn phải đợi thêm một chu kỳ để thực hiện tìm nạp lệnh khác. Nếu khối mã được căn chỉnh trên ranh giới 32B, thì tối đa 4 lệnh có thể được giải mã và sau đó được thực thi đồng thời (trừ khi chúng quá dài hoặc phụ thuộc lẫn nhau).

Bởi vậy các trình biên dịch thường thực hiện tối ưu hóa mà thoạt nhìn trông có vẻ vô ích: đôi khi nó thích dùng các hướng dẫn có mã máy dài hơn và thậm chí chèn các hướng dẫn giả mà không làm gì cả [^nop] để có được các vị trí nhảy chính được căn chỉnh tại ranh giới phù hợp với hàm mũ của 2.

[^nop]: Những hướng dẫn như vậy được gọi là no-op hoặc NOP. Trên x86, "cách chính thức" để không làm gì là `xchg rax, rax` (hoán đổi một thanh ghi với chính nó): CPU nhận ra nó và không dành thêm chu kỳ để thực thi nó, ngoại trừ giai đoạn giải mã.

Trong GCC, bạn có thể sử dụng cờ `-falign-label=n` để chỉ định một chính sách căn chỉnh cụ thể, [thay thế](https://gcc.gnu.org/onlineocs/gcc/Optimize-Options.html) `-labels` với `-function`, `-loops` hoặc `-jumps` nếu bạn muốn chọn lọc hơn. Ở mức độ tối ưu hóa `-O2` và `-O3`, nó được bật mặc định - mà không cần căn chỉnh cụ thể, trong trường hợp đó, nó sử dụng giá trị mặc định phụ thuộc vào từng máy (thường là hợp lý).

### Instruction Cache

The instructions are stored and fetched using largely the same [memory system](/hpc/cpu-cache) as for the data, except maybe the lower layers of cache are replaced with a separate *instruction cache* (because you wouldn't want a random data read to kick out the code that processes it).

The instruction cache is crucial in situations when you either:

- don't know what instructions you are going to execute next, and need to fetch the next block with [low latency](/hpc/cpu-cache/latency),
- or are executing a long sequence of verbose-but-quick-to-process instructions, and need [high bandwidth](/hpc/cpu-cache/bandwidth).

The memory system can therefore become the bottleneck for programs with large machine code. This consideration limits the applicability of the optimization techniques we've previously discussed:

- [Inlining functions](../functions) is not always optimal, because it reduces code sharing and increases the binary size, requiring more instruction cache.
- [Unrolling loops](../loops) is only beneficial up to some extent, even if the number of iterations is known during compile time: at some point, the CPU would have to fetch both instructions and data from the main memory, in which case it will likely be bottlenecked by the memory bandwidth.
- Huge [code alignments](#code-alignment) increase the binary size, again requiring more instruction cache. Spending one more cycle on fetch is a minor penalty compared to missing the cache and waiting for the instructions to be fetched from the main memory.

Another aspect is that placing frequently used instruction sequences on the same [cache lines](/hpc/cpu-cache/cache-lines) and [memory pages](/hpc/cpu-cache/paging) improves [cache locality](/hpc/external-memory/locality). To improve instruction cache utilization, you should  group hot code with hot code and cold code with cold code, and remove dead (unused) code if possible. If you want to explore this idea further, check out Facebook's [Binary Optimization and Layout Tool](https://engineering.fb.com/2018/06/19/data-infrastructure/accelerate-large-scale-applications-with-bolt/), which was recently [merged](https://github.com/llvm/llvm-project/commit/4c106cfdf7cf7eec861ad3983a3dd9a9e8f3a8ae) into LLVM.

### Unequal Branches

Suppose that for some reason you need a helper function that calculates the length of an integer interval. It takes two arguments, $x$ and $y$, but for convenience, it may correspond to either $[x, y]$ or $[y, x]$, depending on which one is non-empty. In plain C, you would probably write something like this:

```c++
int length(int x, int y) {
    if (x > y)
        return x - y;
    else
        return y - x;
}
```

In x86 assembly, there is a lot more variability to how you can implement it, noticeably impacting performance. Let's start with trying to map this code directly into assembly:

```nasm
length:
    cmp  edi, esi
    jle  less
    ; x > y
    sub  edi, esi
    mov  eax, edi
done:
    ret
less:
    ; x <= y
    sub  esi, edi
    mov  eax, esi
    jmp  done
```

While the initial C code seems very symmetrical, the assembly version isn't. This results in an interesting quirk that one branch can be executed slightly faster than the other: if `x > y`, then the CPU can just execute the 5 instructions between `cmp` and `ret`, which, if the function is aligned, are all going to be fetched in one go; while in case of `x <= y`, two more jumps are required.

It may be reasonable to assume that the `x > y` case is *unlikely* (why would anyone calculate the length of an inverted interval?), more like an exception that mostly never happens. We can detect this case, and simply swap `x` and `y`:

```c++
int length(int x, int y) {
    if (x > y)
        swap(x, y);
    return y - x;
}
```

The assembly would go like this, as it typically does for the if-without-else patterns:

```nasm
length:
    cmp  edi, esi
    jle  normal     ; if x <= y, no swap is needed, and we can skip the xchg
    xchg edi, esi
normal:
    sub  esi, edi
    mov  eax, esi
    ret
```

The total instruction length is 6 now, down from 8. But it is still not quite optimized for our assumed case: if we think that `x > y` never happens, then we are wasteful when loading the `xchg edi, esi` instruction that is never going to be executed. We can solve this by moving it outside the normal execution path:

```nasm
length:
    cmp  edi, esi
    jg   swap
normal:
    sub  esi, edi
    mov  eax, esi
    ret
swap:
    xchg edi, esi
    jmp normal
```

This technique is quite handy when handling exceptions cases in general, and in high-level code, you can give the compiler a [hint](/hpc/compilation/situational) that a certain branch is more likely than the other:

```c++
int length(int x, int y) {
    if (x > y) [[unlikely]]
        swap(x, y);
    return y - x;
}
```

This optimization is only beneficial when you know that a branch is very rarely taken. When this is not the case, there are [other aspects](/hpc/pipelining/hazards) more important than the code layout, that compel compilers to avoid any branching at all — in this case by replacing it with a special "conditional move" instruction, roughly corresponding to the ternary expression `(x > y ? y - x : x - y)` or calling `abs(x - y)`:

```nasm
length:
    mov   edx, edi
    mov   eax, esi
    sub   edx, esi
    sub   eax, edi
    cmp   edi, esi
    cmovg eax, edx  ; "mov if edi > esi"
    ret
```

Eliminating branches is an important topic, and we will spend [much of the next chapter](/hpc/pipelining/branching) discussing it in more detail.

<!--

This architecture peculiarity

When you have branches in your code, there is a variability in how you can place their instruction sequences in the memory — and surprisingly, .

```nasm
length:
    mov   edx, edi
    mov   eax, esi
    sub   edx, esi
    sub   eax, edi
    cmp   edi, esi
    cmovg eax, edx  ; "mov if edi > esi"
    ret
```

Granted that `x > y` never or almost never happens, the branchy variant will be 2 instructions shorter.

https://godbolt.org/z/bb3a3ahdE

(The compiler can't optimize it because it's technically [not allowed to](/hpc/compilation/contracts): despite `y - x` being valid, `x - y` could over/underflow, causing undefined behavior. Although fully correct, I guess the compiler just doesn't date executing it.)

We will spend [much of the next chapter](/hpc/pipelining/branching) discussing it in more detail.

You don't have to decode the things you are not going to execute anyway.

In general, you want to, and put rarely executed code away — even in the case of if-without-else patterns.

-->
