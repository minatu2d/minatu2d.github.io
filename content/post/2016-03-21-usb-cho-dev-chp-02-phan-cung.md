---
layout: post
title: USB cho Dev (Chp.02 - Phần cứng)
date: 2016-03-21 02:17:04.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
- USB
tags:
- Translate
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: '43452'
  _publicize_job_id: '20961706836'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/03/21/usb-cho-dev-chp-02-phan-cung/"
---
Tiếp theo bài chương đầu tiên về USB, chương này sẽ nói về phần cứng.

## Đầu kết nối (Connectors)

Mọi thiết bị có một upstream "chảy"đến host, và mọi host có một downstream "chảy" thiết bị. Các điểm kết nối với Upstream, downstream không phải ở dạng hoán đổi cơ học thì thế phải các kết nối vòng không hợp lệ (illegal loopback connections) như downstream chảy đến downstream chẳng hạn phải được loại bỏ ở hubs. Có 2 loại connector phổ biến là type A và type B. Như hình bên dưới đây:

![](/post/images/contypea.gif)

![](/post/images/contypeb.gif)

Type A USB Connector

Type B USB Connector

Cắm vào type A nghĩa là hướng đến upstream. Ta thường thấy khe cắm Type A (đầu cái) trên Host hoặc Hubs ta vẫn dùng hàng ngày. Cắm vào Type B nghĩa là hướng đến downstream, ta vẫn thấy đầu cắm loại này ở thiết bị.

Có một điều khá thú vị, đó là ta có thể hay bắt gặp các dây type A - type A (nối thằng) hay nối dài, gồm một dãy các các đầu chuyển cực (đực -> cái -> đực...) . Đúng ra những dây đó mâu thuẫn với đặc tả USB. Những số loại dây khác cũng thế, chúng thường được sử dụng là dây trung gian giữa 2 đầu type A và type B.

USB 2.0 cũng có một hiệu đính để giới thiệu về đầu kết nối mini-usb B. Chi tiết về đầu kết nối này có thể xem tại [Mini-B Connector Engineering Change Notice](http://www.usb.org/developers/data/ecn1.pdf). Lý do là, type B quá to để tích hợp vào những thiết bị điện tử yêu cầu nhỏ gọn tối đa như điện thoại, thiết bị cầm tay.

Gần đây, đặc tả về OTG [On-The-Go specification](http://www.beyondlogic.org/usb/otghost.htm) ra đời, cho phép kết nối ngang hàng Peer-to-Peer giữa 2 thiết bị USB. Chuẩn này được đưa ra cho phép USB Host có thể nằm trên các thiết bị như điện thoại, thiết bị cầm tay. Tôi đoán rằng, chúng ta sẽ thấy tràn ngập thiết bị được trang bị mini-USB trong thời gian sớm thôi.

Pin Number

Cable Colour

Function

1

Red

VBUS (5 volts)

2

White

D-

3

Green

D+

4

Black

Ground

Các màu dây bên trong bên trong cáp USB, việc gán màu giúp các nhà phát triển dễ dàng hơn trong việc xác định tín hiệu. Đặc tả USB chỉ ra nhiều tham số về điện khác nhau cho cáp USB. Nếu bạn có muốn tìm hiểu kĩ hơn nên đọc đặc tả của USB 1.0. Trong tài liệu đó, người ta gợi ý rằng nên để các dây toàn màu màu xám sương. Nhưng trong USB 1.1, 2.0 thì chúng được thay thế bằng các màu tự nhiên hơn Đen, Xám.

PCB Designer có lẽ muốn tham khảo ở Chapter 6 để biết thêm những chi tiết về pinout.

### Về điện (Electrical)

Trừ khi bạn là người đang mong muốn thiết kế mức silicon cho USB Device/Transceiver hoặc USB Host/ Hubs. Không phải là tất cả, nhưng bạn có thể tham khảo kĩ hơn ở Chapter 7. Chúng ta chỉ lướt qua những điểm cơ bản ở đây thôi.

Như chúng ta đã thảo luận ở bài trước, USB có nhiều dạng truyền dữ liệu khác nhau. Nó được Encode sử dụng NRZI, các bit được chuyển đảm bảo theo dòng. Ở thiết bị low và full speed, mức logic 1 được xác định ở mức điện áp cho D+ là 2.8V với trở kháng là 15K ohm, nối đất;cho D- ở mức dưới 0.3V và cũng qua trở kháng 1.5K Ohm được kéo bởi điện ap 3.6V. Mức logic 0 là những trường hợp còn lại, hay D- lớn hơn 2.8V và D+ nhỏ hơn 0.3V với một trở kháng kéo xuống/lên tương ứng như ở trên.

Tốc độ là low/full speed được xác định thông qua giá trị dòng qua điện trở 90 ohm +/- 15%. Vì thế, giá trị này rất quan trọng trọng việc chọn dòng phù hợp tương ứng với các điện trở ở D+/D-.

High Speed sử dụng dòng cố định 17.78mA cho các tín hiệu của nó nhàm giảm nhiễu.

### Xác định tốc độ

Một thiết bị phải thiết lập tốc độ cho chính nó bằng cách Pull D+ hoặc D- đến 3.3V. Một thiết bị Full Speed, như hình bên dưới đây, sẽ sử dụng một điện trở Pull Up đến gắn với D+. Giá trị điện trở này cũng được phía Host/Hub sử dụng để phát hiện thiết bị được cắm vào. Nếu không có điện trở pull up này, thì phía Host/Hub sẽ coi như không có gì được kết nối vào bus. Một số thiết bị tích hợp điện trở này bên trong thành phần mạch của nó, và cho phép bật hay không bằng xử lý trong firmware, mốt số khác thì sử dụng điện trở gắn ngoài chip.

Ví dụ: Philips Semiconductor đưa ra SoftConnect Technology. Khi kết nối thiết bị hỗ trợ công nghệ này vào bus, chức năng của thiết bị USB này sẽ được khởi động trước cả khi nó bật hay tắt điện trở pull up để host xác định tốc độ của nó. Nếu điện trở này được bật ngay, và được kết nối đến Vbus thì rất có thể phía Host sẽ xác định kết nối và liên tục gửi yêu cầu reset cũng như hỏi các thông số khác khi mà controller phía device còn chưa được khởi động.

Một số nhà sản xuất khác như Cypress Semiconductor, sử dụng điện trở lập trình lại được cho các thiết bị EzUSB của họ. Khi đó, một thiết bị sau khi được nhận là một chức năng nào đó, có thể được ngắt kết nối bằng xử lý trên firmware, sau đó kết nối như một chức năng khác. Tất cả được thực hiện mà không cần sự tác động của người dùng. Rất nhiều thiết bị EzUSB không sử dụng Flash hoặc OTP ROM để chưa code.  Chúng được load load khi có connect thôi.

![](/post/images/fspullup.gif)

Hình 2 : Thiết bị Full Speed có một điện trở pull up nối với D+

![](/post/images/lspullup.gif)

Hình 3 : Thiết bị Low Speed có một điện trở Pull Up nối với D-

Bạn sẽ nhận thấy là, việc xác định High Speed không được nhắc đến. Thực ra, thiết bị High Speed sẽ bắt đầu kết nối như là một thiết bị Full Speed. Khi đã được đã gắn vào Bus, nó sẽ thiết lập chế độ High Speed ở bước Reset nếu Hub/Host support tốc độ này. Khi chế độ High Speed được bật, thì điện trở Pull Up sẽ được loại bỏ để giữ cân bằng line.

Thiết bị tuân theo USB 2.0 không yêu cầu bắt buộc support High Speed. Điều này cho phép thiết bị có thể được sản xuất với giá rẻ hơn nếu không có yêu cầu khắt khe về tốc độ. Một thiết bị low speed cũng thế, không yêu cầu support Full Speed nếu nó không cần thiết cho thiết bị.

Một thiết bị High Speed không cần support low speed, nó chỉ cần support Full Speed vì sẽ sử dụng ở bước connect ban đầu. Tuân thủ USB 2.0, downstreamxuống thiết bị phải support cả 3 mode về tốc độ.

### Nguồn (VBus)

Một trong những điểm mạnh của USB là khả năng cung cấp nguồn cho thiết bị. Thiết bị có thể lấy nguồn từ Bus vì thế không cần trang bị thêm module cung cấp nguồn nào khác. Tuy nhiên, có nhiều trường hợp quá trông cậy vào đặc điểm này mà không tính đến hết các yếu khác.

Một thiết bị USB miêu tả dòng tiêu thụ (chính xác đến 2mA) của nó trong miêu tả cấu hình của nó (chúng ta sẽ xem xét chi tiết ở phần sau). Một thiết bị không thể nhận một dòng cao hơn cái được thiết trong suốt quá trình emuration được kể cả nó không có nguồn cấp ngoài đi nữa. Có 3 lớp cung cấp nguồn chu USB:

Cấp nguồn thấp qua bus: Low-power bus powered functions  
Cấp nguồn cao bus: High-power bus powered functions  
Tự cấp nguồn : Self-powered functions

Chức năng cung cấp nguồn thấp qua bus sẽ cung cấp nguồn cho thiết bị từ Vbus, nó không thể có dòng lớn hơn 1 unit được. Đặc tả USB định nghĩa 1 unit tải tương đương với 100mA. Chức năng cung cấp nguồn thấp qua bus này phải được thiết kế để làm việc điện áp từ 4.40V đến 5.25V được đo ở phía UpStream (host/hub). Với những thiết bị 3.3V thì cần phải có LCD Regulators.

Chức năng cung cấp nguồn cao qua bus sẽ cung cấp nguồn cho thiết bị từ Vbú, nó không thể có dòng lớn hơn 1 unit cho đến khi được configured xong. Sau đó, nó có thể sử dụng tối đa 5 units (500mA) nếu cấu hình của nó yêu cầu. Chức năng này phải được thực hiện với điện áp nhỏ nhất là 4.4V khi phát hiện kết nối và enumerated. Sau khi configure xong, thì điện áp nguồn phía đầu cắm upstream phải nằm trong khoảng 4.75V - 5.25V.

Chức năng tự cấp nguồn, có thể nhận được nguồn tối đa 1 unit từ bus, còn lại nhận từ nguồn bên ngoài. Nếu nguồn bên ngoài bị ngắt, thì ngay lúc đó phải đảm bảo không lấy nhiều hơn 1 unit từ bus. Chức năng tự cung cấp nguồn này sẽ dễ để thiêt kết hơn vì không xảy ra nhiều vấn đề liên quan đến nguồn tiêu thụ. Với 1 unit từ bus, đủ để quá trình enumuration và configure thiết bị diễn ra mà không cần nguồn thứ 2.

Không có thiết bị USB nào được phép điều chỉnh VBus từ upstream. Nếu VBus hỏng, tín hiệu qua các chân D+/D- sẽ bị ngắt trong 10 s.

Một vấn đề khác với VBus là dòng ban đầu phải được giới hạn. Điều này được đề cập đến trong phần 7.2.4.1 của đặc tả. Dòng sẽ hóa giải bởi một điện dung vừa đủ giữa VBus và ground. Trong đặc tả chỉ giá trị lớn nhất của điện dung có thể sử dụng là 10uF. Một điều nữa, khi rút thiết bị khỏi kết nối, dòng hiện tại sẽ bị ném qua dây USB một sự tăng áp lớn có thể xảy trên các đầu mở dây. Để ngăn điều này, một điện dung nhỏ nhất 1uF được chỉ định.

Với những thiết bị sử dụng nguồn từ bus, nó không thể có dòng lớn hơn 500mA, thường là không thể. Bạn có tự hỏi đây là giá trị gì không? Có lẽ là Suspend Mode.

### Suspend Current

Suspend mode là cần thiết cho tất cả các thiết bị. Ở chế độ suspend, một số ràng buộc khác bắt buộc phải tuân theo.