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

**Unconditional** jump `jmp` can only be used to implement `while (true)` kind of loops or stitch parts of a program together. A family of **conditional** jumps is used to implement actual control flow.

It is reasonable to think that these conditions are computed as `bool`-s somewhere and passed to conditional jumps as operands: after all, this is how it works in programming languages. But that is not how it is implemented in hardware. Conditional operations use a special `FLAGS` register, which first needs to be populated by executing instructions that perform some kind of check.

In our example, `cmp rax, rcx` compares the iterator `rax` with the end-of-array pointer `rcx`. This updates the `FLAGS` register, and now it can be used by `jne loop`, which looks up a certain bit there that tells whether the two values are equal or not, and then either jumps back to the beginning or continues to the next instruction, thus breaking the loop.

### Loop Unrolling

One thing you might have noticed about the loop above is that there is a lot of overhead to process a single element. During each cycle, there is only one useful instruction executed, and the other 3 are incrementing the iterator and trying to find out if we are done yet.

What we can do is to *unroll* the loop by grouping iterations together — equivalent to writing something like this in C:

```c++
for (int i = 0; i < n; i += 4) {
    s += a[i];
    s += a[i + 1];
    s += a[i + 2];
    s += a[i + 3];
}
```

In assembly, it would look something like this:

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

Now we only need 3 loop control instructions for 4 useful ones (an improvement from $\frac{1}{4}$ to $\frac{4}{7}$ in terms of efficiency), and this can be continued to reduce the overhead almost to zero.

In practice, unrolling loops isn't always necessary for performance because modern processors don't actually execute instructions one-by-one, but maintain a [queue of pending instructions](/hpc/pipelining) so that two independent operations can be executed concurrently without waiting for each other to finish.

This is our case too: the real speedup from unrolling won't be fourfold, because the operations of incrementing the counter and checking if we are done are independent from the loop body, and can be scheduled to run concurrently with it. But may still be beneficial to [ask the compiler](/hpc/compilation/situational) to unroll it to some extent.

### An Alternative Approach

You don't have to explicitly use `cmp` or a similar instruction to make a conditional jump. Many other instructions either read or modify the `FLAGS` register, sometimes as a by-product enabling optional exception checks.

For example, `add` always sets a number of flags, denoting whether the result is zero, is negative, whether an overflow or an underflow occurred, and so on. Taking advantage of this mechanism, compilers often produce loops like this:

```nasm
    mov  rax, -100  ; replace 100 with the array size
loop:
    add  edx, DWORD PTR [rax + 100 + rcx]
    add  rax, 4
    jnz  loop       ; checks if the result is zero
```

This code is a bit harder to read for a human, but it is one instruction shorter in the repeated part, which may meaningfully affect performance.

<!--

### A More Complex Example

Let's do a more complicated example.

```c++
int collatz(int n) {
    int cnt = 0;
    while (n != 1) {
        cnt++;
        if (n & 2 == 1)
            n = 3 * n + 1;
        else
            n = n / 2;
    }
    return cnt;
}
```

It is a notoriously difficult math problem that seems ridiculously simple.

Make use of [lea instruction](../assembly).

E.g., if you want to make a computational experiment [Collatz conjecture](https://en.wikipedia.org/wiki/Collatz_conjecture), you may use `lea rax, [rax + rax * 2 + 1]`, and then try to `sar` it.

Another way is to check add.

Eliminating branching. Or at least making it easier for the compiler to predict which instructions are going to be executed next.

tzcnt

cmov

Need to somehow link it to branchless programming and layout article. We now have 3 places introducing the concept.

Many other operations set something in the `FLAGS` register. For example, add often. It is useful to, and then decrement or increment it to save on instruction. Like a while loop:

```
while (n--) {
    // ...
}
```

There is an important "conditional move" operation.

-->
