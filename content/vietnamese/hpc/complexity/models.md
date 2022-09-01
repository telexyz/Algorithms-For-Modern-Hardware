---
title: Các mô hình tính toán
weight: 3
# draft: true
---
(( bản nháp ))

Trong lý thuyết khoa học máy tính cổ điển, những điều thực sự thú vị đã ngừng xảy ra vào những năm 70; mọi thứ trong quá khứ chỉ là những nỗ lực thay thế logarit trong tiệm cận bằng một thứ gì đó ít hơn một chút so với logarit. Nếu 50 năm trước, các thuật toán như vậy hy vọng rằng cuối cùng sẽ có đủ sức mạnh tính toán để xử lý các bộ dữ liệu lớn để thể hiện sự ưu việt so với các thuật toán khác kém hơn một cách tiệm cận, nhưng thực tế, ngày nay chúng ta biết chắc chắn rằng chúng sẽ không bao giờ làm được.

Đây là những gì cuốn sách này nói về: chấp nhận thực tế và tối ưu hoá cho phần cứng bạn có, ngoài độ phức tạp tiệm cận. Bạn có thể sẽ không học được một thuật toán nào nhanh hơn tiệm cận ở đây, nhưng bạn sẽ học cách siết chặt hiệu suất từ tất cả các bóng bán dẫn *không tăng theo hàm mũ* mà bạn có trong tay, đó là một kỹ năng có nhiều tác dụng hơn.

Máy tính vẫn đang trở nên nhanh hơn, nhưng theo cách trực giao với mô hình tính toán và chúng ta cần tạo ra những cái mới. Tại sao điều này lại quan trọng để tối ưu hoá các thuật toán ngoài Big O.

Mục tiêu của chương này là giải thích tại sao nó trở nên quan trọng hơn nhiều.

### Hơn cả Big O

Nhiều máy tính không phát triển theo chiều sâu nữa, mà đi theo chiều rộng.

Các mô hình thường "tính phí" chỉ một loại hoạt động.

Máy truy cập ngẫu nhiên mô hình RAM

Mô hình Word RAM

Mô hình bộ nhớ ngoài

Các loại mô hình RAM song song khác nhau

Độ phức tạp truyền thông

Có nhiều cách thức tạo mô hình tính toán khác nhau. Một số liên quan tới mật mã hoặc truyền thông nói chung. Một số phù hợp với công việc cho các nhà thiết kế phần cứng. "Mô hình bóng bán dẫn" đếm số lượng bóng bán dẫn hoặc "mô hình năng lượng" dựa trên các định luật entropy vật lý. Thuật toán lượng tử, hoặc thậm chí có thể là thuật toán do con người thực hiện, nơi bạn cũng cần quan tâm đến lỗi tính toán (do con người gây ra).