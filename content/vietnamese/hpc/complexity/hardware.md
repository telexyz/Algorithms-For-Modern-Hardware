---
title: Phần cứng hiện đại
weight: 1
ignoreIndexing: true
---

Nhược điểm chính của các siêu máy tính của những năm 1960 không phải là chúng chậm mà là chúng khổng lồ, sử dụng phức tạp và đắt đến mức chỉ có chính phủ của các siêu cường quốc mới có thể mua được. Kích thước là lý do chúng quá đắt: chúng đòi hỏi rất nhiều thành phần tuỳ chỉnh phải được lắp ráp rất cẩn thận bởi những người có bằng cấp cao về kỹ thuật điện, trong một quy trình không thể mở rộng để sản xuất hàng loạt.

Sự phát triển của vi mạch - mạch đơn, nhỏ, hoàn chỉnh - đã cách mạng hoá ngành công nghiệp và có lẽ là phát minh quan trọng nhất của thế kỷ 20. Một tủ máy tính trị giá hàng triệu đô la vào năm 1965 có thể đặt vừa trên [một miếng silicon 4mm × 4mm](https://en.wikipedia.org/wiki/MOS_Technology_6502)[^size] vào năm 1975 mà bạn có thể mua với giá 25 đô la. Sự cải thiện đáng kể về giá thành đã bắt đầu cuộc cách mạng máy tính gia đình trong thập kỷ tiếp theo, với các máy tính như Apple II, Atari 2600, Commodore 64 và IBM PC dành cho đại chúng.

[^size]: Kích thước thực tế của CPU là khoảng vài cm vì cần phải quản lý điện năng, tản nhiệt, và thuận tiện cho việc cắm nó vào bo mạch chủ.

### Vi mạch được tạo ra như thế nào

Vi mạch được "in" trên một lát tinh thể silicon bằng cách sử dụng một quá trình gọi là [in li-tô quang học](https://en.wikipedia.org/wiki/Photolithography), trong đó bao gồm:

1. phát triển và cắt lát [một tinh thể silicon rất tinh khiết](https://en.wikipedia.org/wiki/Wafer_(electronics)),
2. bao phủ một lớp photoresist - [một chất tan ra khi photon va vào](https://en.wikipedia.org/wiki/Photoresist),
3. bắn nó với photon theo mẫu hình đã được thiết lập,
4. [hoá khắc](https://en.wikipedia.org/wiki/Etching_(microfabrication)) các bộ phận đã phơi bày,
4. loại bỏ photoresist còn lại,

…và sau đó thực hiện 40-50 bước khác trong vài tháng để hoàn thành phần còn lại của CPU.

![](../img/lithography.png)

Với công nghệ photon, chúng ta có thể "khắc" mẫu lên một khu vực nhỏ hơn nhiều, tạo ra một mạch nhỏ với tất cả các thuộc tính mong muốn. Bằng cách này, quang học của những năm 1970 có thể khắc vài nghìn bóng bán dẫn lên diện tích kích thước bằng móng tay, mang lại cho vi mạch một số lợi thế mà máy tính trong thế giới vĩ mô không có:

- tốc độ đồng hồ cao hơn (trước đây bị giới hạn bởi tốc độ ánh sáng);
- khả năng mở rộng quy mô sản xuất;
- sử dụng ít vật liệu và ít năng lượng hơn rất nhiều, từ đó làm giảm giá thành.

### Dennard Scaling

Hãy xem xét những gì xảy ra khi chúng ta thu nhỏ một vi mạch xuống. Một mạch nhỏ hơn đòi hỏi ít vật liệu hơn và các bóng bán dẫn nhỏ hơn mất ít thời gian hơn để chuyển đổi (cùng với tất cả các quy trình vật lý khác trong chip), cho phép giảm điện áp và tăng tốc độ đồng hồ.

Một quan sát chi tiết hơn, được gọi là *Dennard Scaling*, tuyên bố rằng giảm kích thước bóng bán dẫn xuống 30%

- tăng gấp đôi mật độ bóng bán dẫn ($0.7^2 \approx 0.5$),
- tăng tốc độ đồng hồ lên 40% ($\frac{1}{0.7} \approx 1.4$),
- với *mật độ năng lượng* tổng thể giữ nguyên.

Vì chi phí sản xuất trên mỗi đơn vị là một hàm của diện tích và chi phí khai thác chủ yếu là chi phí năng lượng[^power], mỗi "thế hệ" mới có tổng chi phí gần như giống nhau, nhưng đồng hồ cao hơn 40% và gấp đôi số bóng bán dẫn, có thể được sử dụng ngay lập tức, ví dụ, thêm chỉ lệnh mới hoặc tăng kích thước từ - để theo kịp quá trình thu nhỏ xảy ra trong vi mạch bộ nhớ.

[^power]: Chi phí điện để chạy một máy chủ trong 2-3 năm gần bằng chi phí tự làm chip.

Do sự đánh đổi giữa năng lượng và hiệu suất bạn có thể thực hiện trong quá trình thiết kế, độ trung thực của chính quá trình chế tạo, chẳng hạn như "180nm" hoặc "65nm", tương đương với mật độ bóng bán dẫn, đã trở thành thương hiệu cho độ hiệu quả của CPU[^fidelity].

[^fidelity]: khi luật Moore bắt đầu chậm lại, các nhà sản xuất chip ngừng phác hoạ chip của họ bằng kích thước của các thành phần - bây giờ nó được dùng một thuật ngữ tiếp thị. [Một uỷ ban đặc biệt](https://en.wikipedia.org/wiki/International_Technology_Roadmap_for_Semiconductors) họp mặt hai năm một lần, họ lấy tên tiến trình trước đó, chia nó cho căn bậc hai của hai, làm tròn đến số nguyên gần nhất, tuyên bố kết quả là tên của tiến trình mới, và sau đó uống rượu ăn mừng. "nm" không có nghĩa là nanomet nữa.

Thu hẹp quang học là động lực chính đằng sau những cải tiến hiệu suất trong lịch sử phát triển máy tính. Gordon Moore, cựu giám đốc điều hành của Intel, vào năm 1975 dự đoán rằng số lượng bóng bán dẫn trong bộ vi xử lý sẽ tăng gấp đôi mỗi hai năm. Dự đoán của ông vẫn đúng cho đến ngày nay và được gọi là *luật Moore*.

![](../img/dennard.ppm)

Dennard Scaling và luật Moore không phải là luật vật lý, mà là những quan sát được thực hiện bởi các kỹ sư am hiểu. Cả hai đều bị hạn chế bởi vật lý cơ bản, giới hạn cuối cùng là kích thước của các nguyên tử silicon. Trên thực tế, Dennard Scaling đã dừng lại do các vấn đề về năng lượng.

Về mặt nhiệt động lực học, máy tính là một thiết bị rất hiệu quả để chuyển đổi năng lượng điện thành nhiệt. Nhiệt này cuối cùng cần phải được loại bỏ, và có những giới hạn vật lý về lượng năng lượng bạn có thể loại bỏ từ một tinh thể kích thước milimet. Các kỹ sư máy tính, nhằm tối đa hoá hiệu suất, sẽ chọn tốc độ đồng hồ tối đa có thể được trong khi mức tiêu thụ điện năng tổng thể vẫn giữ nguyên. Nếu bóng bán dẫn trở nên nhỏ hơn, chúng có điện dung ít hơn, có nghĩa là cần ít điện áp hơn để lật chúng, từ đó cho phép tăng tốc độ đồng hồ. 

Khoảng năm 2005-2007, chiến lược này đã ngừng hoạt động vì hiệu ứng rò rỉ: các mạch trở nên nhỏ đến mức từ trường của chúng bắt đầu làm cho các electron trong mạch lân cận di chuyển theo hướng chúng không được phép, gây ra sự gia tăng nhiệt không cần thiết và thỉnh thoảng làm xáo trộn bit.

Cách duy nhất để tránh điều này là tăng điện áp; và để cân bằng mức tiêu thụ điện, bạn cần giảm tần số đồng hồ, từ đó làm cho toàn bộ quá trình dần dần trở nên kém hiệu quả khi mật độ bóng bán dẫn tăng lên. Tại một số điểm, tỷ lệ đồng hồ không còn tăng thêm và xu hướng thu nhỏ bắt đầu chậm lại.


### Hiệu suất năng lượng

Có thể bạn sẽ bất ngờ, nhưng số liệu chính cho CPU hiện đại không phải là tần số đồng hồ, mà là các hoạt động hữu ích trên mỗi joule, hoặc, thực tế hơn, các hoạt động hữu ích trên mỗi đô la.

Trong lịch sử, ba biến thông số chính định hướng việc thiết kế vi mạch là *năng lượng*, *hiệu suất* và *diện tích* (PPA), thường được định nghĩa bằng watt, hertz và nanomet. Cho đến khoảng 2005, chi phí, chủ yếu là một hàm số của diện tích và hiệu suất, là tiêu chí quan trọng nhất, sau đó mới đến năng lượng. Nhưng khi các thiết bị di động chạy bằng pin bắt đầu thay thế máy tính cá nhân, công suất tiêu thụ điện nhanh chóng được đưa lên đầu danh sách, tiếp theo mới là diện tích và hiệu suất.


### Máy tính hiện đại

Dennard scaling đã kết thúc, nhưng luật Moore vẫn tồn tại.

Tốc độ đồng hồ ổn định, nhưng số lượng bóng bán dẫn vẫn đang tăng lên, cho phép tạo ra phần cứng mới, phần cứng *song song*. Thay vì theo đuổi các chu kỳ nhanh hơn, các thiết kế CPU bắt đầu tập trung vào việc thực hiện nhiều thứ hữu ích hơn trong một chu kỳ. Thay vì nhỏ hơn, các bóng bán dẫn đã thay đổi hình dạng.

Điều này dẫn đến các *kiến trúc ngày càng phức tạp* có khả năng thực hiện hàng chục, hàng trăm hoặc thậm chí hàng ngàn thứ khác nhau trong mỗi chu kỳ.

![Die shot của một lõi CPU Zen của AMD (~ 1.400.000.000 bóng bán dẫn)](../img/die-shot.jpg)

Dưới đây là một số cách tiếp cận cốt lõi trong việc sử dụng nhiều bóng bán dẫn:

- Chồng chéo việc thực thi nhiều lệnh để giữ (các phần khác nhau của) CPU luôn bận rộn (đường ống);
- Thực hiện các hoạt động mà không nhất thiết phải chờ các hoạt động trước đó hoàn thành (thực thi đầu cơ và không theo thứ tự);
- Thêm nhiều đơn vị thực thi để xử lý đồng thời các hoạt động độc lập (bộ xử lý siêu vô hướng);
- Tăng kích thước từ của máy, đến mức thêm các lệnh có khả năng thực hiện cùng một thao tác trên một khối 128, 256 hoặc 512 bit dữ liệu ([SIMD](/hpc/simd/));
- Thêm [nhiều lớp bộ nhớ đệm](/hpc/cpu-cache/) để tăng tốc độ truy cập [RAM và bộ nhớ ngoài](/hpc/external-memory/) (thời gian truy cập bộ nhớ không tuân theo quy luật mở rộng silicon);
- Thêm nhiều lõi giống hệt nhau trên chip (tính toán song song, GPU);
- Sử dụng nhiều chip trong bo mạch chủ và nhiều máy tính rẻ hơn trong trung tâm dữ liệu (tính toán phân tán);
- Sử dụng phần cứng tuỳ chỉnh để giải quyết một vấn đề cụ thể với hiệu năng tốt hơn (ASIC, FPGA).

Đối với các máy tính hiện đại, phương pháp "[hãy đếm tất cả các hoạt động](../)" để dự đoán hiệu suất thuật toán không chỉ hơi sai mà còn sai theo cấp độ lớn. Sự thay đổi của phần cứng hiện đại đòi hỏi các mô hình tính toán mới và các cách khác để đánh giá hiệu suất thuật toán.

### Sức mạnh máy tính

- Nhảy con trỏ và xử lý trong hầu hết các ngôn ngữ thông dịch: $10^7$
- Các hoạt động phân nhánh trong ngôn ngữ biên dịch: $10^8$
- Xử lý vô hướng không phân nhánh trong ngôn ngữ biên dịch: $10^9$
- Các ứng dụng SIMD phức tạp hoặc ràng buộc băng thông: $10^{10}$
- Đại số tuyến tính, đơn lõi: $10^{11}$
- CPU máy tính để bàn điển hình: $10^{12}$
- GPU điện thoại di động điển hình: $10^{12}$
- Card đồ hoạ tích hợp điển hình: $2 \cdot 10^{12}$
- Cấu hình chơi game cao cấp: $10^{13}$
- Phần cứng học sâu: $10^{14}$
- Giàn phần cứng học sâu: $10^{15}$
- Được coi là siêu máy tính: $10^{16}$
- Cấu hình đào tạo mô hình ngôn ngữ mạng thần kinh: $5 \cdot 10^{17}$
- Fugaku (#1): $2 \cdot 10^{18}$
- Folding@home: $3 \cdot 10^{18}$