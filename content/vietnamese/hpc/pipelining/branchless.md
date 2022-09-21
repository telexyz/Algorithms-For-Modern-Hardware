---
title: Lập trình không nhánh
weight: 3
published: true
---

Như chúng ta đã nói trong [phần trước](../branching), các nhánh không thể được dự đoán hiệu quả bởi CPU sẽ rất tốn kém vì chúng có thể gây ra tình trạng ngưng trệ đường ống trong thời gian dài để tìm nạp các lệnh mới sau khi một nhánh dự đoán sai. Trong phần này, chúng ta thảo luận về các phương tiện loại bỏ các nhánh ngay từ đầu.

### Dự đoán

Chúng tôi sẽ bắt đầu từ nghiên cứu điển hình trong phần trước - tạo một mảng các số ngẫu nhiên và tính tổng tất cả các phần tử nhỏ hơn 50:

```c++
for (int i = 0; i < N; i++)
    a[i] = rand() % 100;

volatile int s;

for (int i = 0; i < N; i++)
    if (a[i] < 50)
        s += a[i];
```

Mục đích của chúng ta là loại bỏ nhánh gây ra bởi lệnh `if`. Chúng ta có thể làm như sau:
```c++
for (int i = 0; i < N; i++)
    s += (a[i] < 50) * a[i];
```

Vòng lặp này tốn khoảng 7 chu kỳ cho mỗi lần lặp thay vì 14 chu kỳ. Và hiệu năng giữ nguyên khi chúng ta thay hằng số `50` bằng một giá trị khác, vì nó không phụ thuộc vào xác xuất rẽ nhánh.

Khoan đã, hình như vẫn còn nhánh ở đây? Lệnh `(a[i] < 50)` được dịch ra mã máy như thế nào?

Không có kiểu Boolean trong hợp ngữ, và cũng không có lệnh mã máy nào tạo ra 1 hoặc 0 dựa trên kết quả của phép so sánh, nhưng chúng ta có thể tính toán gián tiếp như sau `(a[i] - 50) >> 31`. Mẹo này dựa trên [cách trình bày nhị phân của số nguyên](/hpc/arithmetic/integer), dựa trên thực tế rằng biểu thức `a[i] - 50` mà là số âm (ngụ ý `a[i] < 50`), thì bit cao nhất của nó là 1, và ta có thể xác định nó bằng phép dịch chuyển bit.

```nasm
mov  ebx, eax   ; t = x
sub  ebx, 50    ; t -= 50
sar  ebx, 31    ; t >>= 31
imul eax, ebx   ; x *= t
```

Một mẹo phức tạp hơn là chuyển đổi bit dấu thành mặt nạ và sử dụng bitwis `and` thay cho phép nhân: `((a[i] - 50) >> 31 - 1) & a[i]`. Mẹo này tiết kiệm được 1 chu kỳ CPU, bởi lệnh `imul` chiếm tới 3 chu kỳ:

```nasm
mov  ebx, eax   ; t = x
sub  ebx, 50    ; t -= 50, có thể gây underflow (xem lưu ý dưới)
sar  ebx, 31    ; t >>= 31
; imul eax, ebx ; x *= t, thay bằng:
sub  ebx, 1     ; t -= 1 (causing underflow if t = 0)
and  eax, ebx   ; x &= t
```

Lưu ý rằng cách tối ưu trên không đúng về mặt kỹ thuật dưới góc nhìn của trình biên dịch: với 50 giá trị thấp nhất trong khoảng $[-2^{31}, - 2^{31} + 49]$ — kết quả sẽ không đúng vì underflow. Chúng ta biết tất cả các số đều trong khoảng 0 tới 100 nên điều đó sẽ không xảy ra nhưng trình biên dịch thì không biết điều đó.

Thay vì thực hiện thủ thuật số học này, trình biên dịch đã sử dụng lệnh đặc biệt `cmov` ("di chuyển có điều kiện") để gán giá trị dựa trên một điều kiện (được tính toán và kiểm tra bằng cách sử dụng thanh ghi cờ, giống như đối với các bước nhảy):
```nasm
mov     ebx, 0      ; cmov doesn't support immediate values, so we need a zero register
cmp     eax, 50
cmovge  eax, ebx    ; eax = (eax >= 50 ? eax : ebx=0)
```

Vì vậy, đoạn mã trên thực sự gần với việc sử dụng toán tử bậc ba như thế này:
```c++
for (int i = 0; i < N; i++)
    s += (a[i] < 50 ? a[i] : 0);
```

Cả hai biến thể đều được trình biên dịch tối ưu hóa và tạo ra mã hợp ngữ sau:

```nasm
    mov     eax, 0
    mov     ecx, -4000000
loop:
    mov     esi, dword ptr [rdx + a + 4000000]  ; load a[i]
    cmp     esi, 50
    cmovge  esi, eax                            ; esi = (esi >= 50 ? esi : eax=0)
    add     dword ptr [rsp + 12], esi           ; s += esi
    add     rdx, 4
    jnz     loop                                ; "iterate while rdx is not zero"
```

Kỹ thuật này được gọi là *dự đoán*, và nó gần như tương đương với thủ thuật đại số:
$$
x = c \cdot a + (1 - c) \cdot b
$$


Bằng cách này, bạn có thể loại bỏ sự phân nhánh, nhưng điều này phải trả giá bằng việc đánh giá *cả* hai nhánh và lệnh `cmov`. Bởi vì việc đánh giá nhánh ">=" không tốn chi phí nào, nên hiệu suất chính xác bằng [trường hợp "luôn có"](../branching/#branch-prediction) trong phiên bản rẽ nhánh.

### Khi dự đoán có lợi

Sử dụng dự đoán loại bỏ một [mối nguy điều khiển](../hazards) nhưng lại tạo ra một mối nguy dữ liệu. Vẫn gây ra tắc nghẽn đường ống nhưng bớt tốn kém hơn: bạn chỉ cần chờ `cmov` mà không phải xả bỏ toàn bộ đường ống như trong như trong trường hợp dự đoán sai.

Tuy nhiên, có nhiều tình huống, để nguyên mã nhánh sẽ hiệu quả hơn. Đây là trường hợp khi chi phí tính toán *cả hai* nhánh thay vì chỉ *một* lớn hơn chi phí của việc dự đoán sai nhánh. Thường thì mã rẽ nhánh thắng khi nhánh có thể được dự đoán với xác suất lớn hơn `75%`.

![](../img/branchy-vs-branchless.svg)

This 75% threshold is commonly used by the compilers as a heuristic for determining whether to use the `cmov` or not. Unfortunately, this probability is usually unknown at the compile time, so it needs to be provided in one of several ways:

- We can use [profile-guided optimization](/hpc/compilation/situational/#profile-guided-optimization) which will decide for itself whether to use predication or not.
- We can use [likeliness attributes](../branching#hinting-likeliness-of-branches) and [compiler-specific intrinsics](/hpc/compilation/situational) to hint at the likeliness of branches: `__builtin_expect_with_probability` in GCC and `__builtin_unpredictable` in Clang.
- We can rewrite branchy code using the ternary operator or various arithmetic tricks, which acts as sort of an implicit contract between programmers and compilers: if the programmer wrote the code this way, then it was probably meant to be branchless.

The "right way" is to use branching hints, but unfortunately, the support for them is lacking. Right now [these hints seem to be lost](https://bugs.llvm.org/show_bug.cgi?id=40027) by the time the compiler back-end decides whether a `cmov` is more beneficial. There is [some progress](https://discourse.llvm.org/t/rfc-cmov-vs-branch-optimization/6040) towards making it possible, but currently, there is no good way of forcing the compiler to generate branch-free code, so sometimes the best hope is to just write a small snippet in assembly.

<!--

Because this is very architecture-specific.

in the absence of branch likeliness hints

While any program that uses a ternary operator is equivalent to a program that uses an `if` statement

The codes seem equivalent. My guess is that the compiler doesn't know that `s + a[i]` does not cause integer overflow.

(The compiler can't optimize it because it's technically [not allowed to](/hpc/compilation/contracts): despite `y - x` being valid, `x - y` could over/underflow, causing undefined behavior. Although fully correct, I guess the compiler just doesn't date executing it.)

Branchless computing tricks like this one are especially important in all sorts of parallel algorithms.

The `cmov` variant doesn't care about probabilities of branches. It only wins if the branch probability if 75% chance, which usually is the heuristic threshold set in compilers.

This is a legal optimization, but I guess an implicit contract has evolved between application programmers and compiler engineers that if you write a ternary operator, then you kind of telling that it is likely going to be an unpredictable branch.

The general technique is called *branchless* or *branch-free* programming. Predication is the main tool of it, but there are more complicated ways.

-->

<!--

Let's do a few more examples as an exercise.

```c++
int max(int a, int b) {
    return (a > b) * a + (a <= b) * b;
}
```

```c++
int max(int a, int b) {
    return (a > b ? a : b);
}
```


```c++
int abs(int a, int b) {
    return max(diff, -diff);
}
```

```c++
int abs(int a, int b) {
    int diff = a - b;
    return (diff < 0 ? -diff : diff);
}
```

```c++
int abs(int a) {
    return (a > 0 ? a : -a);
}
```

```c++
int abs(int a) {
    int mask = a >> 31;
    a ^= mask;
    a -= mask;
    return a;
}
```

-->

### Larger Examples

**Strings.** Oversimplifying things, an `std::string` is comprised of a pointer to a null-terminated `char` array (also known as a "C-string") allocated somewhere on the heap and one integer containing the string size.

A common value for a string is the empty string — which is also its default value. You also need to handle them somehow, and the idiomatic approach is to assign `nullptr` as the pointer and `0` as the string size, and then check if the pointer is null or if the size is zero at the beginning of every procedure involving strings.

However, this requires a separate branch, which is costly (unless the majority of strings are either empty or non-empty). To remove the check and thus also the branch, we can allocate a "zero C-string," which is just a zero byte allocated somewhere, and then simply point all empty strings there. Now all string operations with empty strings have to read this useless zero byte, but this is still much cheaper than a branch misprediction.

**Binary search.** The standard binary search [can be implemented](/hpc/data-structures/binary-search) without branches, and on small arrays (that fit into cache) it works ~4x faster than the branchy `std::lower_bound`:

```c++
int lower_bound(int x) {
    int *base = t, len = n;
    while (len > 1) {
        int half = len / 2;
        base += (base[half - 1] < x) * half; // will be replaced with a "cmov"
        len -= half;
    }
    return *base;
}
```

Other than being more complex, it has another slight drawback in that it potentially does more comparisons (constant $\lceil \log_2 n \rceil$ instead of either $\lfloor \log_2 n \rfloor$ or $\lceil \log_2 n \rceil$) and can't speculate on future memory reads (which acts as prefetching, so it loses on very large arrays).

In general, data structures are made branchless by implicitly or explicitly *padding* them so that their operations take a constant number of iterations. Refer to [the article](/hpc/data-structures/binary-search) for more complex examples.

<!--

The only downside of the branchless implementation is that it potentially does more memory reads: 

There are typically two ways to achieve this:

And in general, data structures can be "padded" to be made constant size or height.

That there are no substantial reasons why compilers can't do this on their own, but unfortunately this is just how it is right now.

-->

**Data-parallel programming.** Branchless programming is very important for [SIMD](/hpc/simd) applications because they don't have branching in the first place.

In our array sum example, removing the `volatile` type qualifier from the accumulator allows the compiler to [vectorize](/hpc/simd/auto-vectorization) the loop:

```c++
/* volatile */ int s = 0;

for (int i = 0; i < N; i++)
    if (a[i] < 50)
        s += a[i];
```

It now works in ~0.3 per element, which is mainly [bottlenecked by the memory](/hpc/cpu-cache/bandwidth).

The compiler is usually able to vectorize any loop that doesn't have branches or dependencies between the iterations — and some specific small deviations from that, such as [reductions](/hpc/simd/reduction) or simple loops that contain just one if-without-else. Vectorization of anything more complex is a very nontrivial problem, which may involve various techniques such as [masking](/hpc/simd/masking) and [in-register permutations](/hpc/simd/shuffling).

<!--

**Binary exponentiation.** However, when it is constant

When we can iterate in small batches, [autovectorization](/hpc/simd/autovectorization) speeds it up 13x.

-->
