---
title: Song song cấp độ chỉ lệnh
weight: 3
---

Khi lập trình viên nghe từ *song song*, họ nghĩ ngay tới *song song đa nhân*, phân công tính toán thành *những luồng* bán độ lập, chúng hoạt động cùng nhau để xử lý một vấn đề cụ thể.

Kiểu song song này chủ yếu để giảm *độ trễ* và đạt được *khả năng mở rộng* chứ không phải làm tăng tính *hiệu quả*. Bạn có thể giải quyết bài toán lớn gấp 10 lần bằng thuật toán song song, nhưng nó cũng sẽ tốn 10 lần tài nguyên tính toán. Mặc dầu phần cứng song song ngày càng trở nên [dồi dào](/hpc/complexity/hardware) và thuật toán song song trở nên ngày càng quan trọng, chúng ta sẽ chỉ nói về nhân đơn của CPU mà thôi. Có một kiểu song song, đã có sẵn trong nhân đơn CPU, mà bạn có thể dùng miễn phí.

Phần cứng song song bây giờ ở khắp mọi nơi. Khi bạn mở trang web này trong trình duyệt, nó được tải về từ máy chủ CPU 50 lõi, sau đó được phân tích cú pháp bởi CPU máy tính để bàn 8 lõi và sau đó được hiển thị bằng GPU 400 lõi. Không phải lúc nào tất cả các lõi đều liên quan đến việc tải trang web - chúng có thể đang làm những việc khác. Song song hóa giúp giảm *độ trễ*. Điều quan trọng là vậy, nhưng hiện tại, mối quan tâm chính của chúng tôi không phải là *khả năng mở rộng*, mà là *hiệu quả* của các thuật toán.

Chia sẻ các phép tính tự nó là một nghệ thuật, nhưng hiện tại, chúng tôi muốn học cách sử dụng các tài nguyên mà chúng tôi đã có một cách hiệu quả hơn.

Trong khi song song đa lõi giống như đang "ăn gian", nhiều dạng song song tồn tại "miễn phí". Các thuật toán thích ứng cho phần cứng song song rất quan trọng để đạt được *khả năng mở rộng*. Trong phần đầu của cuốn sách này, chúng ta sẽ coi kỹ thuật này là "ăn gian". Chúng tôi chỉ thực hiện các tối ưu hóa thực sự miễn phí và tốt nhất là không lấy đi tài nguyên từ các tiến trình khác có thể đang chạy đồng thời.


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

### So sánh tương tự với hệ thống giáo dục

Hãy xem xét cách thức hoạt động của hệ thống giáo dục:

1. Các chủ đề được dạy cho các nhóm học sinh thay vì các cá nhân vì việc truyền phát những nội dung giống nhau cho mọi người cùng một lúc sẽ hiệu quả hơn.
2. Một lượng học sinh được chia thành các nhóm do các giáo viên khác nhau phụ trách; bài tập và các tài liệu khóa học được chia sẻ giữa các nhóm.
3. Mỗi năm cùng một khóa học được giảng dạy cho sinh viên mới nhập học mới để các giáo viên luôn bận rộn.

Những đổi mới này làm tăng đáng kể *thông lượng* của toàn hệ thống, mặc dù *độ trễ* (thời gian tốt nghiệp của một sinh viên) vẫn không thay đổi (và có thể tăng một chút vì dạy kèm được cá nhân hóa hiệu quả hơn).

Bạn có thể tìm thấy nhiều điểm tương đồng của hệ thống giáo dục với với các CPU hiện đại:

1. CPU sử dụng [SIMD] (/hpc/simd) để thực hiện cùng một hoạt động trên một khối các điểm dữ liệu khác nhau (bao gồm 16, 32 hoặc 64 byte).
2. Có nhiều đơn vị thực thi có thể xử lý các lệnh này đồng thời trong khi chia sẻ các tiện ích CPU khác (thường là 2-4 đơn vị thực thi).
3. Các chỉ lệnh được xử lý theo kiểu đường ống (tiết kiệm số chu kỳ kiểu như bằng số năm học từ mẫu giáo lên tiến sĩ).

Ngoài ra, một số khía cạnh khác cần lưu ý:

- Các đường dẫn thực thi trở nên khác nhau theo thời gian và cần các đơn vị thực thi khác nhau.
- Một số chỉ lệnh có thể bị dừng vì nhiều lý do khác nhau.
- Một số chỉ lệnh thậm chí còn được suy đoán (thực hiện trước thời hạn), nhưng sau đó bị loại bỏ.
- Một số chỉ lệnh có thể được chia thành một số hoạt động vi mô riêng biệt có thể tự tiến hành.

Lập trình các bộ xử lý pipelined và superscalar có những thách thức riêng của nó, mà chúng ta sẽ giải quyết trong chương này.