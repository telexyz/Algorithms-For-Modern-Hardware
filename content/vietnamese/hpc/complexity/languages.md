---
title: Ngôn ngữ lập trình
aliases:
  - /hpc/analyzing-performance
weight: 2
published: true
---

Nếu bạn đang đọc những dòng này thì có lẽ đâu đó trên con đường lập trình bạn đã bắt đầu quan tâm tới hiệu năng.

Với tôi là vào thời còn học trung học, tôi nhận ra rằng làm trang web và lập trình thông dụng sẽ không giúp mình vào đại học và tôi tìm được một thế giới thú vị: olympic lập trình thuật toán. Tôi là một lập trình viên khá ổn, đặc biệt là đối với một học sinh trung học, nhưng tôi chưa bao giờ thực sự tự hỏi phải mất bao nhiêu thời gian để thực thi chương trình do mình viết ra, điều đó có vẻ không mấy quan trọng. Nhưng đột nhiên nó bắt đầu quan trọng: mỗi bài toán olympic có giới hạn thời gian nghiêm ngặt. Tôi bắt đầu đếm các hoạt động của chương trình. Và tự hỏi mình có thể khiến chương trình làm được bao nhiêu hoạt động trong một giây?

Tôi không biết nhiều về kiến trúc máy tính để có câu trả lời chính xác. Nhưng tôi nghĩ ra một quy tắc: "2-3GHz có nghĩa là 2 đến 3 tỷ lệnh được thực hiện mỗi giây và trong một vòng lặp đơn giản thực thao tác trên các phần tử mảng, tôi cũng cần tăng bộ đếm vòng lặp, kiểm tra điều kiện kết thúc vòng lặp, lập chỉ mục mảng và các việc tương tự, vì vậy hãy thêm chỗ cho 3-5 chỉ lệnh cho mỗi việc như vậy" và sau đó chia tổng số chỉ lệnh phải thực hiện cho $5 \cdot 10^8$ là ra ước tính thời gian thực thi. Nó không đúng hoàn toàn, nhưng đếm số lượng thao tác mà thuật toán của tôi cần phải thực hiện và chia nó cho số này là một quy tắc ước lượng tốt cho trường hợp của tôi.

Câu trả lời thực sự, tất nhiên, phức tạp hơn nhiều và phụ thuộc nhiều vào loại "hoạt động" bạn có trong đầu. Nó có thể thấp tới $10^7$ cho những thứ như [đuổi theo con trỏ](/hpc/cpu-cache/latency) và cao tới $10^{11}$ cho đại số tuyến tính [tăng tốc bằng SIMD](/hpc/simd). Để minh họa những khác biệt nổi bật, chúng ta sẽ sử dụng các thử nghiệm về phép nhân ma trận cài đặt trong các ngôn ngữ khác nhau — và đào sâu hơn vào cách máy tính thực hiện chúng.

## Các loại ngôn ngữ lập trình

Bộ xử lý có thể được coi là *một máy trạng thái*. Nó giữ *trạng thái* của mình trong một số *thanh ghi* có độ dài cố định, một trong số đó là con trỏ lệnh, cho biết vị trí bộ nhớ của lệnh tiếp theo sẽ được đọc và thực thi. Lệnh này có thể sửa đổi các thanh ghi và di chuyển con trỏ lệnh tới lệnh tiếp theo sẽ được thực thi, v.v.

Những chỉ lệnh này được gọi là *mã máy* và được mã hoá nhị phân, máy tính dùng những chỉ lệnh này để điều khiển bộ xử lý. Chúng khó đọc và rất khó làm việc. Vì vậy một trong những điều đầu tiên con người làm sau khi tạo máy tính là nghĩ ra các *ngôn ngữ lập trình* - để trừu tượng hoá phần lớn chi tiết về cách máy tính hoạt động nhằm đơn giản hoá quá trình điều khiển máy tính. Ngày nay không có ai thao tác trực tiếp trên mã máy nữa. Thay vào đó, chúng ta sử dụng ngôn ngữ lập trình cấp cao hơn để cung cấp chỉ lệnh cho bộ xử lý.

Một ngôn ngữ lập trình về cơ bản chỉ là một giao diện. Bất kỳ chương trình nào được viết trong đó chỉ là một biểu diễn cấp cao hơn đẹp hơn mà tại một số thời điểm cần được chuyển đổi thành mã máy để thực thi trên CPU - và có một số cách khác nhau để làm điều đó:

* Từ quan điểm của lập trình viên, có hai loại ngôn ngữ: *biên dịch*, cần được tiền xử lý trước khi thực thi; và *thông dịch*, được thực hiện trong thời gian chạy bằng một chương trình riêng biệt được gọi là trình thông dịch.
* Từ góc độ máy tính, cũng có hai loại ngôn ngữ: ngôn ngữ *bản địa*, thực thi trực tiếp mã máy; và *được quản lý*, dựa trên hệ thống thời gian chạy để thực thi.

Vì chạy mã máy trong trình thông dịch không có ý nghĩa, nên tổng cộng ba loại ngôn ngữ lập trình:

- Ngôn ngữ được thông dịch, chẳng hạn như Python, JavaScript hoặc Ruby.
- Các ngôn ngữ được biên dịch để chạy với hệ thống thời gian chạy (hoặc máy ảo), chẳng hạn như Java, C# hoặc Erlang (và các ngôn ngữ được tạo ra sau để hoạt động trên nền tảng của hệ thống thời gian chạy hoặc máy ảo đó chẳng hạn như Scala, F# hoặc Elixir).
- Các ngôn ngữ bản địa được biên dịch ra mã máy, chẳng hạn như C, Go hoặc Rust.

Không có cách "đúng" duy nhất để thực thi các chương trình máy tính: mỗi cách tiếp cận có những lợi ích và nhược điểm riêng. Thông dịch qua hệ thống thời gian chạy và máy ảo cung cấp tính linh hoạt và cho phép một số tính năng có vẻ sướng như kiểu động, thay đổi mã thời gian chạy và quản lý bộ nhớ tự động, nhưng chúng đi kèm với một số đánh đổi hiệu suất không thể tránh khỏi, mà chúng ta sẽ nói ở dưới đây.

### Ngôn ngữ thông dịch

Đây là một ví dụ về ma trận trận $1024 \times 1024$ theo định nghĩa trong sách giao khoa viết bằng thuần Python:

```python
import time
import random

n = 1024

a = [[random.random()
      for row in range(n)]
      for col in range(n)]

b = [[random.random()
      for row in range(n)]
      for col in range(n)]

c = [[0
      for row in range(n)]
      for col in range(n)]

start = time.time()

for i in range(n):
    for j in range(n):
        for k in range(n):
            c[i][j] += a[i][k] * b[k][j]

duration = time.time() - start
print(duration)
```
Mã này chạy mất 630 giây, tức là hơn 10 phút! Hãy thử phân tích: 

CPU chạy với tần số đồng hồ 1.4GHz, nghĩa là nó thực hiện $1.4 \cdot 10^9$ chu kỳ mỗi giây, tổng cộng gần $10^{15}$ cho toàn bộ tính toán, và khoảng 880 chu kỳ cho mỗi phép nhân trong vòng lặp trong cùng.

Điều này không có gì đáng ngạc nhiên nếu bạn xem xét những điều Python cần làm để hiểu lập trình viên muốn gì:

- nó phân tích biểu thức `c[i][j] += a[i][k] * b[k][j]`;
- cố gắng tìm ra `a`, `b`, và `c` là gì và tra cứu tên của chúng trong một bảng băm đặc biệt với thông tin kiểu;
- hiểu rằng `a` là một danh sách, tìm nạp toán tử `[]` của danh sách, lấy con trỏ của `a[i]`, tìm ra nó cũng là một danh sách, tiếp tục tìm nạp toán tử `[]` của `a[i]`, nhận được con trỏ của `a[i][k]`, và sau đó là giá trị phần tử;
- tìm kiếm kiểu của nó, tìm ra rằng nó là một số thực dấu phẩy động, và tìm nạp phương thức thực hiện toán tử `*`;
- làm những điều tương tự cho `b` và `c`, và cuối cùng cộng thêm kết quả vào `c[i][j]`.

Trình thông dịch của các ngôn ngữ được sử dụng rộng rãi như Python được tối ưu hoá tốt và họ có thể bỏ qua một số bước trên khi thực hiện lặp lại cùng một đoạn mã. Nhưng vẫn còn một số chi phí khá đáng kể không thể tránh khỏi. Nếu chúng ta loại bỏ tất cả các loại kiểm tra và đuổi theo con trỏ này, có lẽ chúng ta có thể nhận được số chu kỳ trên mỗi phép nhân nhân có tỷ lệ ngày càng gần với 1.

### Ngôn ngữ biên dịch được quản lý

Mã nguồn nhân ma trận như trên được triển khai trong Java:

```java
import java.util.Random;

public class Matmul {
    static int n = 1024;
    static double[][] a = new double[n][n];
    static double[][] b = new double[n][n];
    static double[][] c = new double[n][n];

    public static void main(String[] args) {
        Random rand = new Random();

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                a[i][j] = rand.nextDouble();
                b[i][j] = rand.nextDouble();
                c[i][j] = 0;
            }
        }

        long start = System.nanoTime();

        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                for (int k = 0; k < n; k++)
                    c[i][j] += a[i][k] * b[k][j];
                
        double diff = (System.nanoTime() - start) * 1e-9;
        System.out.println(diff);
    }
}
```

Nó chạy mất 10 giây, tương đương với khoảng 13 chu kỳ CPU cho mỗi phép nhân - nhanh hơn 63 lần so với Python. Lưu ý rằng chúng ta cần đọc các phần tử của `b` không theo trình tự bộ nhớ.

Java là một *ngôn ngữ biên dịch*, nhưng không phải *ngôn ngữ bản địa*. Chương trình đầu tiên biên dịch thành *bytecode*, sau đó được thông dịch bởi một máy ảo (JVM). Để đạt được hiệu suất cao hơn, các phần thường xuyên được thực thi của mã, chẳng hạn như vòng lặp `for` trong cùng, được biên dịch thành mã máy trong thời gian chạy và sau đó được thực thi mà hầu như không có chi phí phát sinh. Kỹ thuật này được gọi là *biên dịch khi cần thiết* JIT.

Biên dịch JIT không phải là một tính năng của chính ngôn ngữ, mà là cách triển khai nó. Có một phiên bản Python được biên dịch JIT có tên [PyPy](https://www.pypy.org/), chỉ cần khoảng 12 giây để thực thi mã ở trên mà không có bất kỳ thay đổi nào.

### Ngôn ngữ biên dịch bản địa

Giờ tới lượt ngôn ngữ C:

```cpp
#include <stdlib.h>
#include <stdio.h>
#include <time.h>

#define n 1024
double a[n][n], b[n][n], c[n][n];

int main() {
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            a[i][j] = (double) rand() / RAND_MAX;
            b[i][j] = (double) rand() / RAND_MAX;
        }
    }

    clock_t start = clock();

    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++)
            for (int k = 0; k < n; k++)
                c[i][j] += a[i][k] * b[k][j];

    float seconds = (float) (clock() - start) / CLOCKS_PER_SEC;
    printf("%.4f\n", seconds);
    
    return 0;
}
```
Phải mất 9 giây chạy chương trình trên với biên dịch ở mức `gcc -O3`.

Một cải tiến không đáng kể - lợi thế 1-3 giây so với Java và PyPy có thể được quy cho thời gian chạy bổ sung của biên dịch JIT - nhưng chúng ta chưa tận dụng lợi thế của hệ sinh thái tốt hơn nhiều của trình biên dịch C. Nếu chúng ta thêm cờ `-march=native` và `-ffast-math`, thời gian đột nhiên giảm xuống còn 0,6 giây!

Điều này xảy ra là nhờ chúng ta [giao tiếp với trình biên dịch](/hpc/compilation/flags/) mô hình chính xác của CPU chúng ta đang chạy (`-march=native`) và cho nó tự do sắp xếp lại [tính toán số thực dấu phẩy động](/hpc/arithmetic/float) (`-ffast-math`), và do đó, tận dụng lợi thế của CPU và sử dụng [vectơ hóa](/hpc/simd) để đạt được tốc độ này.

Không phải là không thể điều chỉnh các trình biên dịch JIT của PyPy và Java để đạt được hiệu suất tương tự, nhưng chắc chắn nó dễ dàng hơn cho các ngôn ngữ biên dịch trực tiếp sang mã gốc.

### BLAS

Cuối cùng, chúng ta hãy xem khả năng tối ưu hoá do chuyên gia thực hiện. Chúng ta sẽ kiểm tra một thư viện đại số tuyến tính được tối ưu hoá được sử dụng rộng rãi có tên [OpenBLAS](https://www.openblas.net/). Cách dễ nhất để sử dụng nó thông qua Python và gọi nó từ `numpy`:

```python
import time
import numpy as np

n = 1024

a = np.random.rand(n, n)
b = np.random.rand(n, n)

start = time.time()

c = np.dot(a, b)

duration = time.time() - start
print(duration)
```
Nó chỉ mất ~0.12 giây: tăng tốc ~5x so với phiên bản tự động vectơ hóa với mã nguồn C, và tăng tốc ~5250x so với bản Python ban đầu!

Bạn thường không thấy nhiều cải tiến đáng kể như vậy. Việc thực hiện phép nhân ma trận dày đặc trong OpenBLAS được thực hiện bởi [5000 dòng code Assembly được viết thủ công](https://github.com/xianyi/OpenBLAS/blob/develop/kernel/x86_64/dgemm_kernel_16x2_haswell.S) cho từng kiến trúc một. Trong các chương sau, chúng tôi sẽ giải thích từng kỹ thuật có liên quan, và sau đó [quay trở lại](/hpc/algorithms/matmul) ví dụ này và phát triển chương trình ngang cấp BLAS của riêng mình chỉ dưới 40 dòng mã lệnh C.

### Bài học rút ra

Bài học quan trọng ở đây là sử dụng ngôn cấp thấp không mặc định mang lại cho bạn hiệu suất; nhưng nó cung cấp cho bạn khả năng *kiểm soát* hiệu suất.

Nhiều lập trình viên cũng có một quan niệm sai lầm rằng việc sử dụng các ngôn ngữ lập trình khác nhau có một sự thay đổi đáng kể về số thao tác có thể thực hiện trên một giây. Suy nghĩ theo cách này và [so sánh hiệu năng giữa các ngôn ngữ](https://benchmarksgame-team.pages.debian.net/benchmarksgame/index.html) không có nhiều ý nghĩa: ngôn ngữ lập trình về cơ bản chỉ là công cụ lấy đi *một số* kiểm soát hiệu suất để đổi lấy sự thuận tiện. Bất kể môi trường thực thi nào, lập trình viên vẫn là người chịu trách nhiệm chính trong việc sử dụng tối đa các cơ hội mà phần cứng cung cấp.