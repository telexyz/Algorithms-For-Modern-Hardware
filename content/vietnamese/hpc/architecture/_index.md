---
title: Kiến trúc máy tính
aliases: [/hpc/analyzing-performance/assembly]
weight: 2
---

Khi tôi bắt đầu tự học cách tối ưu hóa chương trình, một sai lầm lớn mà tôi mắc phải là chủ yếu dựa vào phương pháp thực nghiệm mà không hiểu máy tính thực sự hoạt động như thế nào, tôi sẽ hoán đổi bán ngẫu nhiên các vòng lặp lồng nhau, sắp xếp lại số học, kết hợp các điều kiện rẽ nhánh, các inline function bằng tay và làm theo tất cả các mẹo hiệu suất khác mà tôi đã nghe từ những người khác, hy vọng một cách mù quáng chương trình sẽ được cải thiện.

Thật không may, đây là cách hầu hết các lập trình viên tiếp cận tối ưu hóa. Hầu hết các tài liệu giảng dạy về hiệu suất không dạy bạn lý luận về hiệu suất phần mềm một cách định tính. Thay vào đó, họ đưa ra lời khuyên chung cho bạn về các phương pháp triển khai nhất định - và trực giác về hiệu suất nói chung rõ ràng là không đủ.

Nó có lẽ đã giúp tôi tiết kiệm hàng chục, nếu không phải hàng trăm giờ nếu tôi học kiến trúc máy tính *trước khi* lập trình thuật toán. Vì vậy, ngay cả khi hầu hết mọi người không *hào hứng* với nó, chúng ta sẽ dành vài chương đầu tiên để nghiên cứu cách hoạt động của CPU và bắt đầu với việc học hợp ngữ.
