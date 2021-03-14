---
layout: post
title: USB cho dev (Chap.04 - Các loại Endpoint)
date: 2016-03-26 03:11:25.000000000 +09:00
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
  _publicize_job_id: '21136961790'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/03/26/usb-cho-dev-chap-04-cac-loai-endpoint/"
---
## Đặc tả USB định nghĩa 4 loại Transfer/EndPoint

1. Control Transfer

2. Interrupt Transfer

3. Isochronous Transfer

4. Bulk Transfer

### 1. Control Transfer (Truyền điều khiển)

Control Transfer thường được sử dụng cho Commands và Status. Đây là loại truyền chủ yếu được sử dụng trong suốt quá trình Enumeration. Ở dạng truyền này, các gói tin được gửi theo phương trâm best effort delivery (gửi đến khi nào nhận được thì tính tiếp). Độ dài mỗi packet trong dạng truyền control ở mỗi speed lại khác nhau. Ở low speed, nó là 8 byte; Ở High Speed cho phép độ dài 8, 16, 32 và 64 nữa; Ở Full Speed thì phải là 64 byte.

Trong dạng truyền Control, mỗi lần truyền sẽ có 3 stage, mỗi Stage là một nhóm packet và được thực hiện liên tiếp nhau.

**Đầu tiên là Setup Stage.**

Ở Stage này, request sẽ được chuyển đến phía Function Device. Quá trình này được thực hiện trong 3 packet. Packet đầu tiên là Setup Token, chứa Address và Endpoint Number. Tiếp theo là Data Packet, luôn chứa PID Type ở Data0 và một Setup Packet chỉ rõ loại request (Setup Packet sẽ được giải thích chương 6). Packet cuối cùng là một handshaked, dùng để cho biết việc nhận đã xảy thành công hay chưa. Nếu phía Function Device nhận thành công Setup Data (gồm CRC và PID đều OK), nó sẽ trả lời phía Host bằng ACK. Ngược lại, nó có thẻ bỏ qua, không cần gửi lại gì hết. Phía Function không thể gửi một gói STALL hoặc NAK để trả lời Setup Packet được.

![contset](/post/images/contset.gif)

**Tiếp theo là Data Stage.**

Stage này không phải lúc nào cũng có. Nó chứa một hoặc nhiều transfer IN hoặc OUT. Request Setup sẽ cho biết lượng dữ liệu được truyền. Nếu lượng dữ liệu phải truyền vượt quá kích thước tối đa của một Packet, thì dữ liệu sẽ được gửi lần lượt theo từng gói.

Có 2 kịch bản gửi dữ liệu, đó là IN và OUT.

**IN :**

Khi Host muốn nhận dữ liệu, nó sẽ gửi một IN Token. Phía Function nhận được IN Token sẽ kiểm tra xem trường PID (bao gồm phần đảo bit) có hợp lệ hay không. Nếu hợp lệ, device Function sẽ trả lại bằng một packet DATA chứa dữ liệu cần gửi hoặc packet STALL nếu EndPoint có lỗi hoặc NAK nếu EndPoint đang có xử lý khác (tất nhiên các trường hợp lỗi sẽ không có dữ liệu được gửi đi).

![](/post/images/contdata.gif)

![](/post/images/transkey.gif)

**OUT:**

Khi host cần gửi một gói dữ liệu điều khiển, nó sẽ gửi đi một token OUT với phần payload là dữ liệu cần gửi. Nếu có bất cứ phần nào của token OUT hoặc packet Data bị lỗi, thì phía Function sẽ bỏ qua các packet này. Ngược lại, nếu dữ liệu hợp lệ và buffer Endpoint chỉ định đang rỗng nữa thì phía Functions sẽ đẩy dữ liệu vào buffer đó. Rồi Function sẽ gửi lại một ACK cho Host khi nó nhận đầy đủ dữ liệu. Còn nếu buffer Endpoint không rỗng (tức đang có dữ liệu chờ xử lý), nó sẽ gửi lại phía Host một NAK. Còn lại, khi endpoint gặp bất cứ lỗi nào khác, Function sẽ trả lại STALL cho Host.

**Cuối cùng là Status Stage**

Sẽ thông báo trạng thái của toàn bộ quá trình, nội dung của stage này phụ thuộc vào hướng truyền. Thông báo về Status luôn được thực hiện bởi Functions.

**IN :** Nếu host gửi một Token IN để nhận dữ liệu, thì phía Host phải xác nhận dữ liệu đã được nhận thành công. Việc xác nhận này được thực hiện khi Host gửi một token OUT với độ dài payload là zero. Việc nhận được token này sẽ coi như là ACK. Nếu không nhận được hoặc nhận lỗi, phía Host sẽ coi là STALL. Trường hợp còn lại sẽ coi là NACK, để báo rằng Host sẽ lặp lại Status Stage sau đó.

![](/post/images/contsta1.gif)

![](/post/images/transkey.gif)

**OUT**: Nếu host gửi OUT token để gửi dữ liệu ra thì phía Function sẽ xác nhận việc nhận dữ liệu thành công bằng cách gửi lại một Packet có độ dài là Zero cho 1 Token IN mà Host sẽ gửi sau cùng. Tuy nhiên, nếu có lỗi xảy ra khi trả lại Zero Packet thì nó sẽ coi là STALL. Nếu Function ko thể trả lời IN Token do bận, nó sẽ báo lại là NACK để Host nhận lại Status trong Phase tiếp theo.

![](/post/images/contsta2.gif)

![](/post/images/transkey.gif)

**_Truyền Control : Một bức tranh lớn hơn_**

Giờ, chúng ta cùng xem những thứ chúng ta đã nói ở phần trước kết hợp với nhau như thế nào? Hãy xem ví dụ : Host muốn request Device Descriptor trong quá trình Enumeration thì những packet nào sẽ được gửi/nhận.

Đầu tiên, Host sẽ gửi Set up Token để báo cho function biết rằng nó sẽ gửi Setup packet tiếp sau. Trường địa chỉ ADDR sẽ chứa địa chỉ của device mà Host muốn request đến. EndPoint number là Zero, tức là Default Pipe.

Tiếp đó, Host sẽ gửi Data Packet (loại Setup) chưa 8 byte payload (chính là Device Descriptor Request được miêu tả trong Chapter 09 của USB Specification). Phía USB Function sau đó gửi ACK (để xác nhận) sau khi đã nhận Setup Packet đầy đủ và không lỗi. Nếu nội dung của Packet bị lỗi, Device sẽ bỏ qua Packet đó và không gửi ACK gì hết. Phía Host sẽ tự nhận ra và gửi lại Packet đó sau đó ít lâu.

**1. Setup Token**

 

**Sync**

 

**PID**

 

**ADDR**

 

**ENDP**

 

**CRC5**

 

**EOP**

Address & Endpoint Number

**2. Data0 Packet**

 

**Sync**

 

**PID**

 

**Data0**

 

**CRC16**

 

**EOP**

Device Descriptor Request

**3. Ack Handshake**

 

**Sync**

 

**PID**

 

**EOP**

Device Ack. Setup Pack

_Hình trên chưa 3 packet đã nói ở trên._

Giờ phía Device sẽ decode Packet và có được 8 byte. Sau khi xác định được 8 byte này là Device Descriptor Request, nó sẽ gửi Device Descriptor trong các packet bên dưới đây.

**1. In Token**

 

**Sync**

 

**PID**

 

**ADDR**

 

**ENDP**

 

**CRC5**

 

**EOP**

Address & Endpoint Number

**2. Data1 Packet**

 

**Sync**

 

**PID**

 

**Data1**

 

**CRC16**

 

**EOP**

First 8 Bytes of Device Descriptor

**3. Ack Handshake**

 

**Sync**

 

**PID**

 

**EOP**

Host Acknowledges Packet

**1. In Token**

 

**Sync**

 

**PID**

 

**ADDR**

 

**ENDP**

 

**CRC5**

 

**EOP**

Address & Endpoint Number

**2. Data0 Packet**

 

**Sync**

 

**PID**

 

**Data0**

 

**CRC16**

 

**EOP**

Last 4 bytes + Padding

**3. Ack Handshake**

 

**Sync**

 

**PID**

 

**EOP**

Host Acknowledges Packet

Ở hình trên, payload tối đa của 1 packet được giả định là có độ dài 8 byte. Vì thế, ta thấy Device Descriptor (độ dài 12 byte) được chia ra 2 lần gửi tương ứng với 2 lần nhận IN Token.

Sau khi Device Descriptor được gửi hết, thì Host sẽ yêu cầu phía Function xác nhận dữ liệu gửi bằng cách gửi OUT và mong muốn nhận được response với length là ZERO.

**1. Out Token**

 

**Sync**

 

**PID**

 

**ADDR**

 

**ENDP**

 

**CRC5**

 

**EOP**

Address & Endpoint Number

**2. Data1 Packet**

 

**Sync**

 

**PID**

 

**Data1**

 

**CRC16**

 

**EOP**

Zero Length Packet

**3. Ack Handshake**

 

**Sync**

 

**PID**

 

**EOP**

Device Ack. Entire Transaction

### 2. Interrupt Transfers (Dạng truyền ngắt)

Như ta đã biết, ngắt được sinh ra từ thiết bị ở bất cứ thời điểm cho phép nào. Tuy nhiên, trong USB thì phía thiết bị (Function) vẫn phải đợi Host hỏi rồi mới báo là có ngắt hay không.

**Đặc điểm của Interrupt Transfers:**

*   Có độ trễ được đảm bảo
*   1 hướng
*   Có phát hiện lỗi và retry

Interrupt Transfer thông thường là không có chu kì cố định, thiết bị nhỏ sẽ bắt đầu truyển trong một độ trễ đảm bảo. Các Interrupt request sinh ra từ thiết bị sẽ được giữ đến khi host hỏi .

**Đặc trưng về payload của data:**

*   Kích thước tối đa của payload cho low-speed là 8 bytes.
*   Kich thước tối đa của payload cho Full-Speed là 64 bytes.
*   Kích thước tối đa của payload cho High-Speed là 1024 bytes.

![](/post/images/transint.gif)

![](/post/images/transkey.gif)

Hình trên miêu tả định dạng phổ biến của IN và OUT Interrupt:

**IN** : Phía Host sẽ định kì poll (hỏi) phía Function xem có ngắt hay chưa. Tần số polling này được quy định trong phần EndPoint Description của thiết bị đó. Mỗi lần hỏi host sẽ gửi một token IN. Nếu token này bị hỏng, phía Function sẽ bỏ qua packet và tiếp tục theo dõi bus để đợi token mới.

Nếu có ngắt đang được xếp bên trong thiết bị, thì Function sẽ gửi một packet trả loại phía Host và kèm theo dữ liệu tương ứng với ngắt đó. Host sau khi nhận được trả lời, nó sẽ báo lại bằng 1 ACK. Nếu dữ liệu trả lại bị hỏng, thì Host sẽ không trả lại gì hết. Nếu không có ngắt nào xảy ra khi Function nhận được poll từ Host, nó sẽ báo lại cho Host bằng NAK. Nếu có lỗi xảy ra trên Function thì nó sẽ gửi lại STALL cho Host.

**OUT**: Khi một host muốn gửi dữ liệu ngắt đến thiết bị bằng ngắt, nó sẽ đây một OUT token, tiếp theo đó là dữ liệu. Nếu bất kì phần nào của OUT token hoặc dữ liệu bị hỏng, phía Function sẽ bỏ qua packet. Nếu buffer của EndPoint bị rỗng, Function sẽ copy dữ liệu vào, rồi báo lại cho Host biết bằng 1 ACK. Nếu buffer của EndPoint không rỗng, tức là dữ liệu trước đó chưa được xử lý, thì Function sẽ gửi trả lại Host 1 gói NACK. Tuy nhiên, nếu lỗi xảy ra liên tục, và thấy rằng nên gửi lại thì nó sẽ gửi cho Host 1 STALL.

### 3. Isochronous Transfer (Dạng truyền đồng bộ)

Là dạng truyền xảy ra liên tục và có chu kì ổn định. Được sử dụng khi truyền những thông tin đặc trưng như âm thanh, hình ảnh. Nếu delay hoặc gửi lại xảy ra khi truyền âm thanh, hình ảnh, thì chúng ta sẽ thấy một số âm thanh có vẻ kì cục, kiểu như xước đĩa etc. Hoặc các nhạc không đồng bộ với hình ảnh, ta có thể dễ dàng nhận thấy chúng. Tuy nhiên, nếu 1 packet hoặc 1 frame bị drop một vài lần thì nó ít khi bị chú ý bởi người xem, nghe.

**Dạng truyền đồng bộ sẽ cung cấp:**

*   Một bandwidth đảm bảo
*   Độ trẽ được giới hạn
*   Một đường ống (stream pipe) vô hướng
*   Phát hiện lỗi thông qua CRC, nhưng không yêu cầu gửi lại
*   Chỉ sử dụng ở Full hoặc High Speed mode
*   Không data toggling

Kích thước lớn nhất của dữ liệu truyền (payload) được chỉ rõ trong phần mô tả EndPoint của một Endpoint truyền đồng bộ. Nó có thể truyền tới 1023 bytes với Full Speed và 1024 bytes với High Speed. Kích thước lớn nhất của payload sẽ ảnh hướng đến bandwitdh yêu cầu đến bus. Vì thế, phải thận trọng để chọn payload size phù hợp. Nếu sử dụng một payload lớn, thì nên sử dụng các định nghĩa cho alternative interface (interface thay thế khi cần) để thực hiện nhiều payload size. Nếu trong quá trình enumeration, host không thể cho phép bandwidth mà Endpoint yêu cầu do việc hạn chế sử dụng bandwidth, thỉnh thoảng sẽ phải hạn bandwidth xuống chứ không hẳn là không thể sử dụng được. Dữ liệu được gửi trong Endpoint có thể ít hơn con số đã được chấp nhận tại bước enumeration. Có thể thay đổi trong mỗi transaction.

![](/post/images/transiso.gif)

![](/post/images/transkey.gif)

Hình trên mô tả định dạng truyền dồng bộ với IN và OUT token. Transaction truyền động bộ không có bước handshake và cũng không thể thông báo lỗi dạng STALL, HALT được.

### 4. Bulk Transfer (Dạng truyền hàng loạt)

Có thể được sử dụng cho truyền những dữ liệu rất lớn. Ví dụ, một dữ liệu để print hoặc một ảnh được scan từ scanner. Dạng truyền hàng loạt cung cấp cơ chế phát hiện lỗi thông qua CRC16 và có cơ chế để yêu cầu gửi lại nếu lỗi để đảm bảo dữ liệu được truyền/nhận không lỗi.

Dạng truyền này sử dụng số bandwidth chưa được sử dụng còn lại sau khi tất cả các dạng truyền khác đã được cấp. Nếu bus quá bận với các dạng truyền khác thì dạng truyền hàng loạt sẽ rất nhỏ giọt. Vì thế, dạng truyền này nên sử dụng với các dữ liệu không yêu cầu đáp ứng về mặt thời gian, cũng như đảm bảo về độ trễ.

**Dạng truyền hàng loạt:**

*   Sử dụng để truyền dữ liệu lớn
*   Phát hiện lỗi bằng CRC, với cơ chế đảm bảo truyền nhận
*   Không đảm bảo bandwidth tối thiểu
*   Truyền theo đường ống (stream pipe) vô hướng
*   Chỉ sử dụng ở mode Full và High Speed.

Dạng truyền hàng loạt chỉ được hỗ trợ ở Full và High Speed. Kích thước tối đa của payload mỗi packet có thể là 8,16,32,64 byte với Full Speed, và lên đến 512 bytes với High Speed. Dù dữ liệu truyền thực tế nhỏ hơn kích thước lớn nhất, nó cũng không cần padding zero. Một lần truyền hàng loạt được xem là hoàn thành khi nó truyền chính xác số dữ liệu được yêu cầu. Packet truyền đi có thể nhỏ hơn kích thước lớn nhất của EndPoint hoặc là Zero-length packet.

![](/post/images/contdata.gif)

![](/post/images/transkey.gif)

Hình trên chỉ ra định dạng của 1 bulk IN và bulk OUT transaction:

**IN** : Khi Host sẵn sàng nhận dữ liệu hàng loạt, nó sẽ gửi một IN token. Gói này sẽ bị Function bỏ qua nếu có lỗi. Function sẽ gửi trả lại dữ liệu bằng gói DATA nếu nhận đúng gói IN. STALL được gửi khi phía Function có lỗi, NACK được gửi nếu phía Function chưa có dữ liệu hoặc EndPoint đang bận.

**OUT**: Khi host muốn gửi đến Function dữ liệu bulk, nó sẽ gửi một OUT Token, sau đó là gói dữ liệu. Nếu OUT token hoặc Data Packet bị hỏng, thì Function sẽ bỏ qua Packet này. Nếu buffer của EndPoint là Empty, thì Function sẽ copy dữ liệu vào buffer rồi trả lại Host gói ACK. Nếu endpoint buffer chưa rỗng do dữ liệu trước chưa được xử lý, thì Function sẽ trả lại NAK. Nếu có lỗi xảy ra, Function sẽ gửi lại STALL.

## Quản lý BandWidth

Host chịu trách nhiệm quản lý bandwidth của bus. Việc này được thực hiện trong quá trình enumeration khi cấu hình các Endpoint đồng bộ, End Point ngắt. Đặc tả của USB nêu chỉ rõ giới hạn để đảm bảo rằng không cho phép quá 90% số frame được cấp phương thức truyền có tính chu kì (Interrupt và Isochronous) khi ở Full-Speed. Ở High-Speed, giới hạn trên được giảm xuống còn 80% microframe có thể cấp cho các phương thức truyền có tính chu kì.

Vì thế, bạn sẽ thấy rằng, sẽ có một đủ bandwidth đế mức không dùng hết với các phương thức truyền có tính chu kì. Chỉ còn khoảng 10% dành cho control nếu nó được sử dụng, còn lại mới đến lượt bulk transfer.