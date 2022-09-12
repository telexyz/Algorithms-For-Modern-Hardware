---
title: Song song cấp độ chỉ lệnh
weight: 3
---

Khi lập trình viên nghe từ *song song*, họ nghĩ ngay tới *song song đa nhân*, phân công tính toán thành *những luồng* bán độ lập, chúng hoạt động cùng nhau để xử lý một vấn đề cụ thể.

Kiểu song song này chủ yếu để giảm *độ trễ* và đạt được *khả năng mở rộng* chứ không phải làm tăng tính *hiệu quả*. Bạn có thể giải quyết bài toán lớn gấp 10 lần bằng thuật toán song song, nhưng nó cũng sẽ tốn 10 lần tài nguyên tính toán. Mặc dầu phần cứng song song ngày càng trở nên [dồi dào](/hpc/complexity/hardware) và thuật toán song song trở nên ngày càng quan trọng, chúng ta sẽ  chỉ nói về nhân đơn của CPU mà thôi. Có một kiểu song song, đã có sẵn trong nhân đơn CPU, mà bạn có thể dùng miễn phí.

Note: khi nói về hiệu năng, người ta hay nói tới `băng thông` và `độ trễ`

<!--

This technique only applies 

Parallel hardware is now everywhere. When you opened this page in your browser, it was retrieved by a 50-core server CPU, then parsed by an 8-core desktop CPU, and then rendered by a 400-core GPU. Not all cores were involved with serving you this page at all times — they might have been doing something else.

Parallelism helps in reducing *latency*. It is important, but for now, our main concern is not *scalability*, but *efficiency* of algorithms.

Sharing computations is an art in itself, but for now, we want to learn how to use resources that we already have more efficiently.

While multi-core parallelism is "cheating," many form of parallelism exist "for free."

Adapting algorithms for parallel hardware is important for achieving *scalability*. In the first part of this book, we will consider this technique "cheating." We only do optimizations that are truly free, and preferably don't take away resources from other processes that might be running concurrently.

-->

### Đường ống chỉ lệnh

Để thực thi một lệnh, bộ xử lý phải làm rất nhiều công việc chuẩn bị, bao gồm:

- Fetch: **nạp** một đoạn mã máy từ bộ nhớ,
- Decode: **giải mã** nó và chia nhỏ ra thành nhiều lệnh,
- Execute: **thực thi** những lệnh đó, có thể bao gồm các thao tác trên **bộ nhớ** (Memory), và
- Write: **ghi** kết quả vào thanh ghi.

Toàn bộ chuỗi hoạt động này là *khá lâu*. Nó mất đến 15-20 chu kỳ CPU ngay cả đối với một lệnh đó đơn giản như `cộng`hai giá trị được lưu trữ thanh ghi lại với nhau. Để giảm thiểu độ trễ này, các CPU hiện đại sử dụng *pipelining*: sau khi một lệnh đi qua giai đoạn đầu tiên, chúng bắt đầu xử lý lệnh tiếp theo ngay lập tức mà không cần đợi lệnh trước đó hoàn thành đầy đủ.

![](img/pipeline.png)

Các nhà sản xuất phần cứng thích sử dụng thuật ngữ *số chu kỳ trên một lệnh* (CPI) hơn là "độ trễ trung bình của lệnh" như là một cách để đo lường độ hiệu năng của CPU. Nó cũng là một [độ đo tốt](/hpc/profiling/benchmarking) cho việc thiết kế thuật toán.

CPI hoàn hảo của một bộ xử lý đường ống hóa (pipelined processor) sẽ tiệm cần với 1, thậm chí còn thấp hơn nữa nếu chúng ta làm rộng đường ống sao cho có nhiều hơn một lệnh được thực hiện cùng lúc. Bởi vì bộ đệm của ALU có thể được chia sẻ, sẽ rẻ hơn nếu cho thêm một lõi riêng biệt nữa. Kiến trúc như thế có khả năng chạy nhiều hơn một lệnh trên một chu kỳ, và được gọi là *siêu vô hướng* (superscalar), các CPU hiện đại đều là siêu vô hướng.

Bạn chỉ có thể tận dụng được thế mạnh của siêu vô hướng nếu tập lệnh chứa những nhóm lệnh độc lập với nhau về logic (ví dụ như sự phụ thuộc trước sau), và có thể được thực thi riêng biệt. Các lệnh không tới theo một thứ tự thuận tiện, vì thế khi có thể, các CPU hiện đại thực hiện chúng "không theo thứ tự" (ooo: out-of-order) để tận dụng sức mạnh phần cứng và giảm thiểu tình trạng ngưng trệ đường ống.

Cách thức hoạt động của ooo là một chủ đề [thảo luận nâng cao hơn](/scheduling), hiện tại, bạn có thể giả định rằng CPU duy trì một bộ đệm các lệnh đang chờ xử lý xử lý và thực thi chúng ngay sau khi các giá trị toán hạng của nó được tính toán và có một đơn vị thực thi đang rảnh rỗi.

### An Education Analogy

Consider how our education system works:

1. Topics are taught to groups of students instead of individuals as broadcasting the same things to everyone at once is more efficient.
2. An intake of students is split into groups led by different teachers; assignments and other course materials are shared between groups.
3. Each year the same course is taught to a new intake so that the teachers are kept busy.

These innovations greatly increase the *throughput* of the whole system, although the *latency* (time to graduation for a particular student) remains unchanged (and maybe increases a little bit because personalized tutoring is more effective).

You can find many analogies with modern CPUs:

1. CPUs use [SIMD parallelism](/hpc/simd) to execute the same operation on a block of different data points (comprised of 16, 32, or 64 bytes).
2. There are multiple execution units that can process these instructions simultaneously while sharing other CPU facilities (usually 2-4 execution units).
3. Instructions are processed in pipelined fashion (saving roughly the same number of cycles as the number of years between kindergarten and PhD).

<!-- You can continue "up:" there are multiple school branches (cores), multiple schools (computers), etc. -->

In addition to that, several other aspects also match:

- Execution paths become more divergent with time and need different execution units.
- Some instructions may be stalled for various reasons.
- Some instructions are even speculated (executed ahead of time), but then discarded.
- Some instructions may be split in several distinct micro-operations that can proceed on their own.

Programming pipelined and superscalar processors presents its own challenges, which we are going to address in this chapter.
