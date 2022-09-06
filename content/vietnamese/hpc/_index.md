---
title: Thuật toán cho phần cứng hiện đại
menuTitle: Hiệu năng cao
weight: 5
noToc: true
---

Dịch từ cuốn "Algorithms for Modern Hardware" viết bởi [Sergey Slotin](http://sereja.me/) ([sách gốc](https://github.com/algorithmica-org/algorithmica), [mã nguồn đi kèm](https://github.com/sslotin/scmm-code)).

Đối tượng cuốn sách bao gồm các lập trình viên hiệu năng, các nhà nghiên cứu thuật toán ứng dụng và các sinh viên ngành khoa học máy tính muốn tìm cách tăng tốc cài đặt thuật toán từ $O(n \log n)$ sang $O(n \log \log n)$.

### Vài lời từ tác giả

Tôi muốn thay đổi cách bộ môn khoa học máy tính - chính xác hơn là thiết kế thuật toán - đang được dạy. Cụ thể hơn:

Có 2 cuốn sách kinh điển là nền tảng xây dựng hầu hết các khóa học về thiết kế giải thuật. Cả 2 cuốn đều rất xuất sắc, nhưng [cuốn thứ nhất](https://en.wikipedia.org/wiki/The_Art_of_Computer_Programming) đã 50 tuổi, [cuốn còn lại](https://en.wikipedia.org/wiki/Introduction_to_Algorithms) đã 30 tuổi, và kể từ đó [máy tính đã thay đổi rất nhiều](/hpc/complexity/hardware). Độ phức tạp tiệm cận không còn là yếu tố quyết định duy nhất nữa.

Trong thiết kế thuật toán hiện đại, điểm cốt yếu là bạn chọn cách cài đặt sử dụng tốt nhất có thể các loại cơ chế song song khác nhau có sẵn trong phần cứng hơn là chọn một thuật toán có số lượng tính toán ít nhất trên lý thuyết - cách cài đặt tận dụng tối đa sức mạnh của phần cứng.

Tuy nhiên, chương trình giảng dạy khoa học máy tính ở các trường đại học hầu như bỏ qua sự thay đổi này. Ngoài vài khóa học đang khắc phục điều đó như "[Performance Engineering of Software Systems](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-172-performance-engineering-of-software-systems-fall-2018/)" from MIT, "[Programming Parallel Computers](https://ppc.cs.aalto.fi/)" from Aalto University, và các trang hướng dẫn Denis Bakhvalov's "[Performance Ninja](https://github.com/dendibakh/perf-ninja)" — hầu hết các khóa học khác vẫn coi phần cứng hiện đại như những năm 1990s.

Tôi thực sự muốn performance engineering được dạy ngay sau khóa học giới thiệu về thuật toán. Hoàn thành cuốn sách này là điểm khởi đầu. Việc tạo ra một khoá học mới đòi hỏi nhiều hơn thế: bạn cần một chương trình giảng dạy cân bằng, cơ sở hạ tầng cho khoá học, trình bày giáo án, các bài tập thực hành ... vì vậy sau khi hoàn thành cuốn sách này, tôi sẽ tạo ra các tài liệu và công cụ cho việc giảng dạy performance engineering - tôi mong muốn được hợp tác với những người khác có cùng ý tưởng.

Nội dung của toàn bộ giáo án bao gồm:

### Phần I: Performance Engineering

Phần đầu tiên giới thiệu về kiến trúc máy tính và tối ưu hoá các thuật toán đơn luồng. Các chủ đề về tối ưu hoá CPU: như bộ nhớ đệm, SIMD và pipelining; và cung cấp các ví dụ ngắn gọn viết bằng C++, tiếp theo là các case studies điển hình lớn, mà khi áp dụng các kiến thức về performance engineering thường tăng tốc độ đáng kể so với một số thuật toán hoặc cấu trúc dữ liệu có sẵn trong STL (thư viện mẫu chuẩn của C++).

Mục lục dự kiến:
```
0. Lời nói đầu

1. Mô hình phức tạp
 1.1. Phần cứng hiện đại
 1.2. Ngôn ngữ lập trình
 1.3. Mô hình tính toán
 1.4. Khi nào nên tối ưu hoá

2. Kiến trúc máy tính
 1.1. Các kiến trúc tập chỉ lệnh (instruction set)
 1.2. Hợp ngữ (Assembly)
 1.3. Vòng lặp và điều kiện
 1.4. Hàm và đệ quy
 1.5. Phân nhánh gián tiếp
 1.6. Bố cục mã máy
 1.7. Các cuộc gọi hệ thống
 1.8. Ảo hoá

3. Song song cấp chỉ lệnh
 3.1. Pipeline Hazards
 3.2. Chi phí phân nhánh
 3.3. Lập trình không phân nhánh
 3.4. Bảng chỉ lệnh
 3.5. Lịch trình chỉ lệnh
 3.6. Tính toán thông lượng
 3.7. Giới hạn hiệu suất lý thuyết

4. Biên dịch
 4.1. Các giai đoạn biên dịch
 4.2. Cờ và mục tiêu
 4.3. Tối ưu hoá tình huống
 4.4. Contract Programming
 4.5. Non-Zero-Cost Abstractions
 4.6. Tính toán trong lúc biên dịch
 4.7. Tối ưu hoá số học
 4.8. Trình biên dịch có thể và không thể làm gì

5. Profiling
 5.1. Thiết bị đo đạc
 5.2. Hồ sơ thống kê
 5.3. Mô phỏng chương trình
 5.4. Máy phân tích mã máy
 5.5. Điểm chuẩn (Benchmarking)
 5.6. Nhận kết quả chính xác

6. Số học
 6.1. Số dấu phẩy động
 6.2. Số học khoảng thời gian
 6.3. Phương pháp Newton
 6.4. Căn bậc hai nghịch đảo nhanh
 6.5. Số nguyên
 6.6. Chia số nguyên
 6.7. Thao tác bit
(6.8. Nén dữ liệu)

7. Lý thuyết số
 7.1. Nghịch đảo mô-đun
 7.2. Nhân Montgomery
(7.3. Trường hữu hạn)
(7.4. Sửa lỗi)
 7.5. Mật mã học
 7.6. Băm
 7.7. Tạo số ngẫu nhiên

8. Bộ nhớ ngoài
 8.1. Phân cấp bộ nhớ
 8.2. Bộ nhớ ảo
 8.3. Mô hình bộ nhớ ngoài
 8.4. Sắp xếp bên ngoài
 8.5. Xếp hạng danh sách
 8.6. Chính sách trục xuất
 8.7. Thuật toán không để ý bộ nhớ cache
 8.8. Địa phương không gian và tạm thời
(8.9. B-Trees)
(8.10. Thuật toán tuyến tính)
(9.13. Quản lý bộ nhớ)

9. Bộ nhớ RAM & CPU
 9.1. Băng thông bộ nhớ
 9.2. Độ trễ bộ nhớ
 9.3. Dòng bộ nhớ cache
 9.4. Chia sẻ bộ nhớ
 9.5. Song song cấp độ bộ nhớ
 9.6. Tìm nạp trước
 9.7. Căn chỉnh và đóng gói
 9.8. Lựa chọn thay thế con trỏ
 9.9. Tính liên kết bộ nhớ cache
 9.10. Phân trang bộ nhớ
 9.11. AoS và SoA

10. Song song SIMD
 10.1. Nội tại và các loại vectơ
 10.2. Di chuyển dữ liệu
 10.3. Giảm (reduction)
 10.4. Mặt nạ và pha trộn
 10.5. Xáo trộn trong thanh ghi
 10.6. Tự động vectơ hoá và SPMD

11. Algorithm Case Studies
 11.1. GCD nhị phân
(11.2. Sàng số nguyên tố)
 11.3. Phân tích thừa số nguyên
 11.4. Hồi quy hậu cần
 11.5. Big Integers & Karatsuba Algorithm BB
 11.6. Biến đổi Fourier nhanh
 11.7. Biến đổi lý thuyết số
 11.8. Argmin với SIMD
 11.9. Prefix Sum with SIMD
 11.10. Đọc các số nguyên thập phân
 11.11. Viết các số nguyên thập phân
(11.12. Đọc và viết Floats)
(11.13. Tìm kiếm chuỗi)
 11.14. Sắp xếp
 11.15. Nhân ma trận

12. Nghiên cứu điển hình về cấu trúc dữ liệu
 12.1. Tìm kiếm nhị phân
 12.2. Cây B tĩnh
(12.3. Cây tìm kiếm)
 12.4. Cây phân đoạn
(12.5. Thử nghiệm)
(12.6. Truy vấn tối thiểu phạm vi)
 12.7. Bảng băm
(12.8. Bitmap)
(12.9. Bộ lọc xác suất)
```

Trong số những điều thú vị mà chúng tôi sẽ tăng tốc:

- GCD nhanh hơn 2 lần (so với `std::gcd`)
- Tìm kiếm nhị phân nhanh hơn 8-15 lần (so với `std::lower_bound`)
- Cây phân đoạn nhanh hơn 5-10 lần (so với cây Fenwick)
- Bảng băm nhanh hơn 5 lần (so với `std::unordered_map`)
- Popcount nhanh hơn 2 lần (so với việc gọi liên tục `popcnt`)
- Phân tích cú pháp chuỗi số nguyên nhanh hơn 35 lần (so với `scanf`)
- Sắp xếp nhanh hơn x? lần (so với `std::sort`)
- Tổng nhanh hơn 2 lần (so với `std::accumulate`)
- Tổng tiền tố nhanh hơn 2-3 lần (so với triển khai ngây thơ)
- Argmin nhanh hơn 10 lần (so với triển khai ngây thơ)
- Tìm kiếm mảng nhanh hơn 10 lần (so với `std::find`)
- Cây tìm kiếm nhanh hơn 15 lần (so với `std::set`)
- Nhân ma trận nhanh hơn 100 lần (so với "for-for-for")
- Phân tích số nguyên kích thước từ tối ưu (~ 0,4ms trên số nguyên 60 bit)
- Thuật toán Karatsuba tối ưu
- FFT tối ưu

Khối lượng: 450-600 trang
Ngày phát hành: Q3 2022

### Phần II: Các thuật toán song song

Xử lý đồng thời, mô hình song song, chuyển đổi ngữ cảnh, luồng xanh (green-threads), thời gian chạy đồng thời (concurrent runtimes), sự gắn kết bộ nhớ cache, nguyên thuỷ đồng bộ hoá, OpenMP, reductions, scans, xếp hạng danh sách, thuật toán đồ thị, cấu trúc dữ liệu không khoá, tính toán không đồng nhất, CUDA, kernels, warps, blocks, nhân ma trận, sắp xếp.

Khối lượng: 150-200 trang
Ngày phát hành: 2023-2024?


### Phần III: Tính toán phân tán

<!-- (I might need some help from here on.) -->

Mạng, truyền tin nhắn, mô hình diễn viên, thuật toán ràng buộc truyền thông, nguyên thuỷ phân tán, all-reduce, MapReduce, xử lý luồng (stream processing), lập kế hoạch truy vấn, lưu trữ, chia nhỏ (sharding), nén, cơ sở dữ liệu phân tán, tính nhất quán, độ tin cậy, lập lịch, công cụ quy trình làm việc (workflow engines), điện toán đám mây.

Ngày phát hành: ???

### Phần IV: Đồng thiết kế phần mềm và phần cứng

<!-- (TODO: come up with a better title — one that emphasizes that this part is mainly about the software-hardware boundary and not PL/IC design.) -->

LLVM IR, compiler optimizations & back-end, interpreters, JIT-compilation, Cython, JAX, Numba, Julia, OpenCL, DPC++, oneAPI, XLA, (basic) Verilog, FPGAs, ASICs, TPUs and other AI accelerators.

Ngày phát hành: ???

### Ghi nhận

Cuốn sách chủ yếu dựa trên các bài đăng trên blog, tài liệu nghiên cứu, các cuộc nói chuyện hội nghị và các tác phẩm khác của nhiều người:

- [Agner Fog](https://agner.org/optimize/)
- [Daniel Lemire](https://lemire.me/en/#publications)
- [Andrei Alexandrescu](https://erdani.com/index.php/about/)
- [Chandler Carruth](https://twitter.com/chandlerc1024)
- [Wojciech Muła](http://0x80.pl/articles/index.html)
- [Malte Skarupke](https://probablydance.com/)
- [Travis Downs](https://travisdowns.github.io/)
- [Brendan Gregg](https://www.brendangregg.com/blog/index.html)
- [Andreas Abel](http://embedded.cs.uni-saarland.de/abel.php)
- [Jakob Kogler](https://cp-algorithms.com/)
- [Igor Ostrovsky](http://igoro.com/)
- [Steven Pigeon](https://hbfs.wordpress.com/)
- [Denis Bakhvalov](https://easyperf.net/notes/)
- [Paul Khuong](https://pvk.ca/)
- [Pat Morin](https://cglab.ca/~morin/)
- [Victor Eijkhout](https://www.tacc.utexas.edu/about/directory/victor-eijkhout)
- [Robert van de Geijn](https://www.cs.utexas.edu/~rvdg/)
- [Edmond Chow](https://www.cc.gatech.edu/~echow/)
- [Peter Cordes](https://stackoverflow.com/users/224132/peter-cordes)
- [Geoff Langdale](https://branchfree.org/)
- [Matt Kulukundis](https://twitter.com/JuvHarlequinKFM)
- [Georg Sauthoff](https://gms.tf/)
- [Danila Kutenin](https://danlark.org/author/kutdanila/)
- [Ivica Bogosavljević](https://johnysswlab.com/author/ibogi/)
- [Matt Pharr](https://pharr.org/matt/)
- [Jan Wassenberg](https://research.google/people/JanWassenberg/)
- [Marshall Lochbaum](https://mlochbaum.github.io/publications.html)
- [Pavel Zemtsov](https://pzemtsov.github.io/)
- [Gustavo Duarte](https://manybutfinite.com/)
- [Nyaan](https://nyaannyaan.github.io/library/)
- [Nayuki](https://www.nayuki.io/category/programming)
- [InstLatX64](https://twitter.com/InstLatX64)
- [ridiculous_fish](https://ridiculousfish.com/blog/)
- [Z boson](https://stackoverflow.com/users/2542702/z-boson)
- [Creel](https://www.youtube.com/c/WhatsACreel)


### Miễn trừ trách nhiệm: Lựa chọn công nghệ 

Các ví dụ trong cuốn sách này sử dụng C++, GCC, x86-64, CUDA và Spark.

Những công nghệ này được chọn vì chúng phổ biến và ổn định nhất; do đó hữu ích hơn cho người đọc. Tôi có thể chọn C / Zig / Rust / Carbon, LLVM, ARM, OpenCL và Dask trong lần tái bản tới.