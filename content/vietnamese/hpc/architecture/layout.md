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

### Bộ đệm chỉ lệnh

Các chỉ lệnh được lưu và nạp sử dụng [bộ nhớ hệ thống](/hpc/cpu-cache) giống với dữ liệu, ngoại trừ tầng thấp nhất của bộ đệm được thay thế bởi bộ đệm chỉ lệnh riêng - bởi vì bạn sẽ không muốn dữ liệu được đọc sẽ ghi đè lên chỉ lệnh).

Bộ đệm chỉ lệnh là thiết yếu trong những trường hợp sau:
- không biết chỉ lệnh nào sẽ được thực hiện tiếp và cần nạp khối lệnh tiếp theo với độ [trễ thấp](/hpc/cpu-cache/latency),
- hoặc đang thực thi một khối lệnh độ dài lớn nhưng thời gian xử lý ngắn, và cần [băng thông cao](/hpc/cpu-cache/bandwidth).

Vì thế bộ nhớ hệ thống có thể là nút nghẽn cổ chai cho các chương trình có số lượng mã máy lớn. Hạn chế này làm giảm hiệu quả của các kỹ thuật thảo luận ở trên:

- [Inlining functions](../functions) không phải lúc nào cũng tối ưu, vì tuy nó làm giảm việc chia sẻ code nhưng lại làm tăng mã chương trình, nên cần nhiều lần tải mã lệnh vào bộ đệm hơn.

- [Unrolling loops](../loops) chỉ có lợi ở một mức độ nào đó, ngay cả khi số lần lặp được biết trước tại thời gian biên dịch: tại một số điểm, CPU sẽ phải tìm nạp cả lệnh và dữ liệu từ bộ nhớ chính, trong trường hợp đó, nó có thể sẽ bị tắc nghẽn bởi băng thông bộ nhớ.

- [Căn chỉnh mã](#code-alignment) lớn làm tăng kích thước mã máy, và đòi hỏi nhiều lần nạp vào bộ đệm chỉ lệnh hơn. Dành thêm một chu kỳ để tìm nạp là một hình phạt nhẹ hơn so với việc bỏ lỡ bộ nhớ đệm và chờ đợi các hướng dẫn được tìm nạp từ bộ nhớ chính.

Một khía cạnh khác là việc đặt các chuỗi lệnh được sử dụng thường xuyên trên cùng [các dòng bộ nhớ đệm](/hpc/cpu-cache/cache-lines) và các [trang bộ nhớ](/hpc/cpu-cache/paging) sẽ cải thiện [tính địa phưong hóa bộ nhớ đệm](/hpc/external-memory/locality). Để cải thiện việc sử dụng bộ đệm chỉ lệnh, bạn nên nhóm mã nóng với mã nóng và mã nguội với mã lạnh, và xóa mã chết (không sử dụng) nếu có thể. Nếu bạn muốn khám phá thêm ý tưởng này, hãy xem [Công cụ bố trí và tối ưu hóa nhị phân](https://engineering.fb.com/2018/06/19/data-infrastructure/accelerate-large-scale-applications-with-bolt/) của Facebook, công cụ này gần đây đã được hợp nhất vào LLVM.

### Các nhánh không bằng nhau

Giả sử rằng vì lý do nào đó bạn cần một hàm trợ giúp tính độ dài khoảng của hai số nguyên $x$ và $y$. Trong C, bạn sẽ viết như thế này:

```c++
int length(int x, int y) {
    if (x > y)
        return x - y;
    else
        return y - x;
}
```

Trong hợp ngữ x86, có rất nhiều cách triển khai đoạn mã ở trên, mỗi cách triển khai có hiệu năng khác nhau. Hãy bắt đầu bằng cách triển khai trực tiếp đoạn mã trên vào hợp ngữ:
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
Trong khi mã nguồn C trông có vẻ đối xứng, phiên bản hợp ngữ lại không như vậy. Kết quả là 1 nhánh sẽ chạy nhanh hơn nhánh còn lại một chút: nếu `x > y` CPU sẽ thực hiện 5 lệnh từ cmp tới ret. Trường hợp còn lại CPU thực hiện nhiều hơn 2 lệnh `jump`.

Có thể giả sử rằng hầu hết các trường hợp `x > y` vì ai lại đi tính khoảng cách ngược như vậy. Khi gặp trường hợp ngược, chỉ cần đảo `x` cho `y`:
```c++
int length(int x, int y) {
    if (x > y)
        swap(x, y);
    return y - x;
}
```

Mã hợp ngữ sẽ như thế này:

```nasm
length:
    cmp  edi, esi
    jle  normal     ; if x <= y, ko cần đổi chỗ, ta bỏ qua lệnh xchg
    xchg edi, esi
normal:
    sub  esi, edi
    mov  eax, esi
    ret
```

Tổng số lệnh giảm từ 8 xuống 6. Chưa hẳn đã tối ưu nhất vì trong trường hợp này `x > y` hầu như không xảy ra nên ta sẽ lãng phí 1 lệnh `xchg edi, esi` vì nó hầu như không được thực hiện. Ta sẽ mang nó ra khỏi dòng thực thi bằng cách:
 
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

Kỹ thuật này khá tiện lợi trong trường hợp cần xử lý ngoại lệ, và trong ngôn ngữ bậc cao bạn có thể đưa cho trình biên dịch [gợi ý](/hpc/compilation/situational) rằng nhánh nào sẽ dùng nhiều hơn các nhánh còn lại:
```c++
int length(int x, int y) {
    if (x > y) [[unlikely]]
        swap(x, y);
    return y - x;
}
```
Quá trình tối ưu trên chỉ có ích lợi khi bạn biết rõ nhành nào thường xuyên dùng, nhánh nào không. Nếu không biết rõ, một số [khía cạnh khác](/hpc/pipelining/hazards) còn quan trọng hơn cách bố trí mã lệnh. Trình biên dịch buộc phải sử dụng một lệnh "di chuyển có điều kiện" gần như tương đương với biểu thức `(x > y ? y - x : x - y)` hoặc gọi `abs(x - y)`:
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

Loại bỏ rẽ nhánh là một chủ đề quan trọng và ta sẽ dành [gần như một chương](/hpc/pipelining/branching) để nói về nó.
