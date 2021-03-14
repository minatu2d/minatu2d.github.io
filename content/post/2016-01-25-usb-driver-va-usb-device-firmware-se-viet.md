---
layout: post
title: USB cho Dev (Chp.01 - Giới thiệu)
date: 2016-01-25 14:18:10.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
- Communication
tags:
- Beyondlogic
- USB
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '19106661483'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/01/25/usb-driver-va-usb-device-firmware-se-viet/"
---
USB - Một chuẩn giao tiếp phổ biến nhất (tính đến 2016), hãy cùng tìm hiểu một chút về nó. Bài này không phải dành cho người sử dụng bằng nhứng con số về tốc độ, hay cách cắm vật lý. Bài này là một bài dịch, mình thấy cần rất hữu ích khi bắt đầu phát triển sử dụng USB.

Link gốc tại [http://www.beyondlogic.org/usbnutshell/usb1.shtml](http://www.beyondlogic.org/usbnutshell/usb1.shtml)

## Tóm tắt về USB

### Hiểu chuẩn USB để sử dụng trong phát triển

Nếu bạn là người bắt đầu công việc phát triển sử dụng USB, thì quả thật nó không dễ dàng gì. Có đến 650 trang tài liệu đặc tả [USB 2.0 specification](http://www.usb.org/developers/docs/usb_20.zip), mà đó mới chỉ là mô tả chuẩn chung thôi. Nếu phát triển một thiết bị cụ thể, giả sử là HID (bàn phím, chuột..etc) thì có khi bạn phải đọc thêm  97 trang tài liệu đặc tả thiết bị [USB Class Standards](http://www.usb.org/developers/devclass/) nữa. Nếu bạn là nhà phát triển Driver cho phía Host, thì bạn phải số tài liệu tương đương. Nhưng thật ra mà nói, chẳng có gì ở mấy cái tài liệu đặc tả đâu.

Thật may, bạn không cần phải nhằn hết toàn bộ tài liệu đặc về USB ở trên. Có một vài chapter chứa cả nội dung marketing, chủ yếu về những chi tiết liên quan đến giao tiếp mức tập, nhằm thuyết phục các nhà sản xuất IC phát triển chuẩn này. Chúng ta hay ngó qua các đề mục lớn của tài liệu đặc tả USB 2.0 để xem cần tập trung vào điểm chính nào.

Chapter

Tên

Mô tả

Vị trí (trang)

1

Introduction

Bao gồm mục đích viết tài liệu, phạm vi nội dung. Thông tin quan trọng nhất của chapter này tham chiếu đến Universal Serial Bus Device Class Specifications (Đặc tả các lớp thiết bị USB). **Không cần phải đọc chapter này.**

2

2

Terms and Abbreviations

Như chính cái tên của nó, chương này cần thiết với mọi chuẩn.

8

3

Background

Chỉ ra mục đích của USB là cung cấp cho người dùng cuối tính năng PnP (Cắm vào là chơi luôn). Giới thiệu về Low, Full, High Speed với mục đích marketing. **Khỏi cần đọc.**

4

4

Architectural Overview

**Đây chính là chỗ cân đọc.** Nó cung cấp một cách cơ bản về hệ thống sử dụng USB, bao gồm topology (dạng mô tả hình học), tốc độ truyền dữ liệu, các luồng dữ liệu, còn có cả các yêu cầu cơ bản về mặt điện nữa.

10

5

USB Data Flow Model

Chương này bắt đầu nói luồng dữ liệu được trên USB. Nó giới thiệu 2 khái niệm cơ bản EndPoint, Pipe và dành hầu như toàn bộ phần còn lại nói về các loại luồng dữ liệu (Controller, Interrupt, Isochronous và Bulk). Vì nói toàn bộ nên nó khá nặng với người lần đầu đọc, tuy nhiên đây là phần rất quan trọng để hiểu các phần sau.

60

6

Mechanical

Chapter này nói chi tiết vế 2 loại đầu kết nối. Thông tin quan trọng là Type A hướng đến kết nối với Down Stream, Type B là để kết nối đến Up Stream. Vì thế, không thể nối cổng up stream với nhau được. Dây có thể tháo dời được phải là Full/High Speed. Trong khi dây Low Speed phải cố định với thiết bị. **Bạn chỉ nên nhìn qua để biết** 2 khái niệm trên nếu bạn không phải là nhà sản xuất các dây kết nối cho USB. Các nhà thiết kế PCB có thể tìm thấy những thông tin hữu ích ở trang này.

33

7

Electrical

Chương này nói các tín hiệu điện bao gồm cường độ dòng, trở, rise/fall, gửi nhận tín hiệu, các mức logic etc. Phần quan trọng ở chương này là nói về việc xác định speed của thiết bị bằng 1 điện trở và cách xác định nguồn cấp cho thiết bị là tự cấp (self powered) hay lấy từ cổng kết nối (bus powered). Nếu bạn không phải là người tìm hiểu thiết kế USB Transceiver thì **có thể bỏ qua chương này**. Datasheet của thiết bị USB thông thường sẽ đưa ra chi tiết giá trị đúng để phù hợp với dòng của Bus.

75

8

Protocol Layer

Chỗ này chính là chỗ để chúng ta xem xét đến các tầng protocol. Chương này miêu tả các USB packet mức byte, bắt đầu từ các trường syn, pid, address, endpoint, CRC. Khi hiểu rõ mấy thứ đó, ta sẽ đi vào các lớp protocol sau, USB Packets. Hầu hết chúng ta (developer) không thể thấy được protocol ở mức thấp hơn mức này vì USB IC của thiết bị làm việc này rồi. **Tuy nhiên, việc hiểu rõ các trạng thái ở mỗi bước, rồi quá trình (hanshaking) bắt tay là rất có ích.**

45

9

USB Device Frame Work

Đây là chương được xem xét nhất trong toàn bộ đặc tả, thâm chí tôi từng in một vài tờ và ghim nó lên. Chi tiết về quá trình quá trình Bus Enumeration, các request code (set address, get descripor etc), chính là cái mà hầu hết USB Programmer, Designer thấy và quan tâm đến. **Chương này phải đọc**

36

10

USB Host Hardware and Software

Chương này nói về những gì liên quan đến host. Nó bao gồm cơ chế sinh frame, microframe, yêu cầu về host controller, cơ chế của phần mềm và mô hình driver cho USB. **Nếu bạn không liên quan đến phía Host (Driver, phần mềm phía trên driver etc) thì có thể bỏ qua.**

23

11

Hub Specification

Chi tiết về cách hoạt động của USB Hub. **Nếu bạn không phải đang thiết kế Hub thì bạn có thể bỏ qua chương này.**

143

Nào, giờ chúng ta sẽ phân loại các chương trên theo nhu cầu. Nếu bạn là một nhà phát triển Driver(phần mềm) cho thiết bị USB, thì bạn chỉ cần đọc các chương sau:

> 4 - Architectural Overview  
> 5 - USB Data Flow Model  
> 9 - USB Device Frame Work, and  
> 10 - USB Host Hardware and Software.

Nếu bạn là người thiết kế phần cứng thiết bị (điện tử) thì bạn cần đọc những chương sau:

> 4 - Architectural Overview  
> 5 - USB Data Flow Model  
> 6 - Mechanical, and  
> 7 - Electrical.

### Tóm tắt về USB dành cho nhà thiết kế thiết bị

Nào, hãy đối mặt với nói. Thứ nhất, hầu hết chúng ta ở đây để phát triển các thiết bị USB. Thứ hai, chúng ta tuy đã đọc mà vẫn không biết cách nào để implement thiết bị. Vì thế, 7 chương tiếp theo trong bài viết này, chúng ta sẽ tập trung vào những phần cần thiết, phù hợp cho việc phát một thiết bị USB. Nó giúp bạn hiểu rõ USB và những vấn đề liên quan, để từ đó nghiên cứu kĩ hơn để giải quyết những vẫn đề trong những ứng dụng cụ thể.

Chuẩn USB 1.1 khá rắc rối vì thế High Speed được ném sang USB 2.0. Để hiểu những nguyên lý cơ bản về USB, chúng tôi sẽ bỏ qua nhiều chỗ liên quan đến High Speed.

### Giới thiệu về Universal Serial Bus

USB phiên bản 1.1 hỗ trợ 2 mode tốc độ, đó là Full Speed 12Mbits/s và Low Speed 5Mbits/s. Ở Low Speed 5Mbits/s, ít ảnh hưởng của EMI ([Electromagnetic interference](https://en.wikipedia.org/wiki/Electromagnetic_interference) - giao thoa điện từ) nên việc chế tạo sẽ rẻ hơn do chi phí cho chất chế tạo thấp hơn. Ví dụ, thay vì dùng thạch anh thì dùng cộng hưởng giá rẻ hơn. Chuẩn USB 2.0, lúc đó vốn chưa có chỗ đứng trong các máy Desktop hồi đó, nhưng được nâng lên để hỗ trợ tối đa 480Mbits/s. Hay ngày nay ta gọi là High Speed Mode. Đây là đặc điểm để cạnh tranh với Firewire Serial Bus, vốn khá phổ biến lúc đó.

Tốc độ USB:

High Speed - 480Mbits/s  
Full Speed - 12Mbits/s  
Low Speed - 1.5Mbits/s

Bus trong giao tiếp USB được kiểm soát bởi Host và chỉ Host thôi. Chỉ có 1 Host trên 1 hệ thống Bus. Đặc ta cũng nói rõ, không support bất cứ sắp xếp nào có nhiều master. Tuy nhiên, trong [On-The-Go specification](http://www.beyondlogic.org/usb/otghost.htm) thì có một điểm mới được đưa vào USB 2.0, đó là Host Negotiations Protocol (giao thức thỏa thuận để làm Host) cho phép 2 thiết bị tự thỏa thuận với nhau để giữ vai trò là Host. Đặc điểm mới này nhằm hướng đến các kết nối point-to-point (điểm với điểm), ví dụ như một mobile phone và một máy cá nhân kết nối trực tiếp mà không thông qua hub hay cần một cấu hình cho nhiều thiết bị. USB Host đảm nhận tất cả transaction và lập lịch sử dụng bandwidth. Dữ liệu được gửi bằng nhiều dạng transaction dựa trên giao thức token-based.

Theo tôi, mô hình hình học (topology) của USB có nhiều điểm hạn chế. Ý định ban đầu của USB là giảm bớt số dây cổng kết nối trên PC. Mấy ông Apple thì nói rằng, ý tưởng của USB xuất phát từ Apple Desktop Bus, cái mà chuột và bàn phím cùng kết nối được thông qua 1 dây kết nối thôi.

Tuy nhiên, USB lại sử dụng mô hình Start(sao *), tương tự với Ethernet. Điều này dẫn được việc phải sử dụng Hub ở đâu đó trong hệ thống, rất có thể sẽ gây ra sự tăng lên về chi phí cho kích thước máy, dây cáp. Dù vậy thì thực tế nó cũng không đến mức đó. Nguyên nhân là rất nhiều thiết bị tích hợp sẵn USB Hub bên trong. Ví dụ, một cái bàn phím ta vẫn dùng để kết nối đến máy tính có thể chưa một Hub trong đó, một thiết bị USB khác như chuột, camera kĩ thuật số có thể dễ dàng kết nối vào bàn phím đó. Có một hàng sa số các thiết bị có tích hợp sẵn USB Hub  bên trong, điển hình là màn hình.

Mô hình Star có một số điểm mạnh so với thiết kế thiết bị dạng nối tiếp (imply daisy chaining). Đầu tiên, đó là việc kiểm soát nguồn cấp cho mỗi thiết bị được thực hiện riêng rẽ, vì nếu có xảy ra overcurrent thì không ảnh hưởng đến các thiết bị khác. Thứ hai, USB Hub hỗ trợ cả 3 loại tốc độ High, Full và Low Speed, các transaction của High và Full Speed được lọc ra vì thế thiết bị tốc độ thấp không phải thực hiện những transaction đó.

Một USB Bus hỗ trợ tốt đa đến 127 thiết bị kết nối cùng thời điểm. Khi cần nối thêm thiết bị, đơn giản là tăng port hoặc hub lên. Thời gian đầu, USB Host chỉ có 2 port, sau đó hầu hết nhà sản xuất bắt đầu thấy nó quá ít và tăng lên 4,5 port thậm chí có cả các port bên trong cho kết nối với ổ cứng nữa. Về phía host, thời đó hầu hết chỉ hỗ trợ 1 USB Host Controller vì thế 2 port thường chia sẻ USB Bandwith. Sau đó, yêu cầu kết nối tăng lên, chúng ta thấy cả những Host hỗ trợ nhiều USB Controller riêng rẽ.

USB Host Controller có tài liệu đặc tả riêng. Ở USB 1.1, có 2 loại đặc tả cho Host Controller. Loại thứ nhất là [UHCI (Universal Host Controller Interface)](http://developer.intel.com/design/USB/UHCI11D.htm) được phát triển bởi Intel, đặc trưng là giảm xử lý cho phần cứng và đặt nặng cho phần mềm (chủ yếu là Microsoft). Loại thứ 2 là [OHCI (Open Host Controller Interface)](http://www.compaq.com/productinfo/development/openhci.html), được phát triển bởi Compaq, Microsoft và National Semiconductor, đặt nặng xử lý cho phần cứng và đơn giản hóa phần mềm. Đây là một điển hình cho mối liên quan giữa phần cứng và phần mềm.

Ngoài ra, với sự ra đời của USB 2.0, một đặc tả Host Controller yêu cầu phải chi tiết ở mức thanh ghi được ra đời. Đó là  [EHCI (Enhanced Host Controller Interface)](http://developer.intel.com/technology/usb/ehcispec.htm). Nhưng công ty tham gia xây dựng bao gồm Intel, Compaq, NEC, Lucent và Microsoft. Họ cùng nhau xây dựng tiêu chuẩn mới và muốn chỉ cần 1 driver là đủ.

Về USB Driver, mục đích để hỗ trợ tốt hơn PnP, nên nó cho phép load/unload driver một cách linh hoạt. Phía người đơn giản chỉ cần cắm thiết bị vào, phía Host sẽ lấy những thông tin cần thiết bằng cách hỏi thiết bị đó, rồi load các driver tương ứng lên (chúng ta vẫn thấy một thông báo có thiết bị mới ở góc dưới phải màn hình quá trình này xảy ra). Người dùng cuối không cần phải lo lắng bất cứ điều gì liên quan đến khô khan như IRQs, địa chỉ khe cắm hay reboot lại máy. Khi không dùng nữa, chỉ cần rút ra, thế là xong, Host sẽ tự phát hiện và unload các driver đã load trước đó.

Để load đúng driver, thông thường Host dựa vào cặp giá trị PID/VID (Product ID/Vendor ID). VID được cung cấp bởi USB Implementor forum, nó được bán cho các nhà sản xuất thiết bị khi họ muốn muốn tham gia. Những thông tin về gia có thể tìm thấy ở [USB Implementor’s Website](http://www.usb.org/developers/vendor.html).

Có những tổ chức tiêu chuẩn khác cung cấp VID mở rộng cho mục đích phi thương mại như giảng dạy, nghiên cứu hoặc Hobbyist. USB Implementer forum không cung cấp cái này. Trong trường hợp này, bạn sẽ muốn một VID/PID để gán cho các thiết bị của bạn. Có thể nhận nó từ chính nhà sản xuất chíp USB, nó sẽ VID/PID của một sản phẩm phi thương mại. Hoặc một số nhà sản xuất cũng bán PID cho phép bạn sản xuất những thiết bị thương mai.

Quay trở lại với những gì sẽ thảo luận sau một hồi lan man. USB hỗ trợ 4 dạng transfer. Đó là Control, Interrupt, Bulk và Isochronous. Hầu hết sẽ được xem xét ở phần sau, những có 1 điểm muốn nhắc đến là về Isochronous. Nó cho phép thiết bị dự trù 1 lượng bandwidth với latency đảm bảo. Việc đảm bảo này là lý tưởng cho các thiết bị Audio, Video, nơi dữ liệu cần được đảm bảo liên tục. Mỗi dạng truyền cung cấp cho designer nhưng công cụ để tối ưu việc truyền dữ liệu như phát hiện lỗi (error detection), phục hồi (recovery), độ trẽ đảm bảo (guaranteed latency) và băng thông (bandwidth).