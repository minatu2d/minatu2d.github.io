---
layout: post
title: USB cho dev (Chap.03 - Giao thức)
date: 2016-03-21 15:19:47.000000000 +09:00
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
  _publicize_job_id: '20979922779'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/03/21/usb-cho-dev-chap-03-giao-thuc/"
---
Không giống như RS-232 và nhiều giao tiếp tuần tự khác, cái mà không định nghĩa dạng dữ liệu được gửi. USB được tạo bởi nhiều lớp protocol. Nghe có vẻ "nguy hiểm", nhưng cứ bình tõm, nó không ghê gớm đến thế đâu. Một khi bạn hiểu cái gì đang diễn ra thì cái bạn thực sự phải bỏ công sức vào chỉ là các lớp ở tầng trên thôi. Trong thực tế, hầu hết USB Controller I.C.s sẽ lo hết mấy tầng dưới rồi. Hay nói cách khác, mấy tầng đó nhà phát triển cũng không thấy được.

Mỗi USB Transaction bao gồm

*   SMột packet chứa Token (Giống như Header cho biết cái gì sẽ đến tiếp sau đó) - luôn có
*   Một packet chứa dữ liệu (có thể không có) , (chứa payload)
*   Một packet chứa trạng thái (được sử dụng để thông báo trạng thái truyền và cung cấp cơ chế kiểm soát lỗi nữa)

Như chúng ta đã thao luận trước, USB là dạng bus tập trung vào host. Host sẽ bắt đầu mọi transactions. Khi gửi gói đầu tiên, một token được sinh ra bởi host. Từ token đó, sẽ xác định cái gì dữ liệu nào được gửi tiếp sau đó. Dữ liệu đó có thể là để đọc, ghi và địa chỉ thiết bị là gì, cho endpoint nào. Packet sau token nói chung sẽ mang dữ liệu, cuối cùng là một gói để kết thúc quá trình. Gói cuối sẽ cho biết dữ liệu hoặc token được gửi thành công hay chưa.

Sau đây sẽ là các trường phổ biến trong một Packet USB. Chú ý rằng, dữ liệu truyền trên USB Bus là dạng LSBit First.

Sync

Tất cả các  Packet đều bắt đầu bằng trường này. Ở Low Speed và Full Speed thì trường này có độ dài 8 bit, còn ở High Speed

Các loại USB Packet

là 32 bit. Trường này được sử dụng để đồng bộ clock giữa bên gửi và bên nhận. 2 bit cuối của Sync báo rằng sau đó là điểm bắt đầu của trường PID.

PID

PID nghĩa là Packet ID. Trường này được sử dụng để xác định loại packet được gửi. Bảng sau mô tả loại packet và giá trị tương ứng.

Nhóm

Giá trị PID (Loại Packet)

Tên Packet

Token

0001

OUT Token

1001

IN Token

0101

SOF Token

1101

SETUP Token

Data

0011

DATA0

1011

DATA1

0111

DATA2

1111

MDATA

Handshake

0010

ACK Handshake

1010

NAK Handshake

1110

STALL Handshake

0110

NYET (No Response Yet)

Special

1100

PREamble

1100

ERR

1000

Split

0100

Ping

Như bảng trên ta cũng thấy PID chiếm 4 bit. Nhưng để đảm bảo nhận chính xác, giá trị đảo của 4bit này được thêm vào đuổi, thành ra có 8 bit cho trường PID.

PID0

PID1

PID2

PID3

nPID0

nPID1

nPID2

nPID3

ADDR

Trường địa chỉ, dùng để chỉ ra Packet này sẽ cho thiết bị nào. Có 7bit cho trường này, vì thế có tối đa 127 địa chỉ hợp (như ta đã nói từ đầu số thiết bị tối đa là 127) + 1 địa chỉ ADDR=0 là không hợp lệ. Địa chỉ Zero này không được gán cho bất cứ thiết bị nào, chẳng những thế những thiết bị khi chưa được cấp địa chỉ phải trả lời (bằng Packet) tới địa chỉ Zero.

ENDP

Đây là trường EndPoint. Độ dài 4 bit vì thế hỗ trợ tối đa 16 Endpoint. Có 1 trường hợp đặc biệt cho thiết bị Low Speed là chỉ có 2 Endpoint ở mỗi đầu gửi nhận (tức là tối đa 4 Endpoin thôi)

CRC

Giá trị này được sinh ra trên dữ liệu chín

USB has four different packet types. Token packets indicate the type of transaction to follow, data packets contain the payload, handshake packets are used for acknowledging data or reporting errors and start of frame packets indicate the start of a new frame.

h của Packet (phần Payload). Trường CRC của Packet Data có độ dài 16 bit,các Packet khác có độ dài cố định 5 bit.

EOP

EOP tức là End Of Packet. Đoạn phía sau này không hiểu lắm.

Các loại USB Packet

Như đã trình bày trong bảng về trường PID, chúng ta có 4 nhóm (loại lớn) Packet khác nhau. Packet Token sẽ xác định loại transaction, Packet data sẽ chứa Payload. Packet Handshake được sử dụng cho báo cáo tình trạng gửi nhận, thông báo lỗi. Cuối cùng, Packet Start Of Frame để báo hiệu bắt đầu 1 frame mới.

Packet Token

Có 3 loại Packet Token

IN - Báo cho USB Device biết host muốn đọc thông tin từ nó

OUT - Báo cho USB Device biết host gửi thông tin cho nó

SETUP - Báo cho USB Device biết host sẽ truyền thông tin điều khiển

Mỗi Packet Token phải tuân theo định dạng sau:

**Sync**

**PID**

**ADDR**

**ENDP**

**CRC5**

**EOP**

Packet Data

Có 2 loại Packet data, mỗi loại đều có thể truyền tối đa 1024 byte dữ liệu.

Data0

Data1

Ở High Speed mode, có thể sử dụng thêm 2 PID khác nữa là DATA2 và MDATA.

Packet data sẽ có định dạng sau:

**Sync**

**PID**

**Data**

**CRC16**

**EOP**

Ở Low Speed, cho phép tối đa 8 bytes payload.(phần Data ở định dạng trên)

Ở Full Speed,  cho phép tối đa 1023 byte payload.

Ở High Speed, cho phép tối đa 1024 bytes.

Data phải được gửi thành nhiều byte.

Packet Handshake

Có 3 loại Packet Handshake là :

ACK - Cho biết Packet đã được gửi nhận thành công chưa

NAK - Cho biết Device không tạm thời không thể gửi hoặc nhận dữ liệu. Ngoài ra, gói này cũng được sử dụng trong Transaction dạng Interrupt để báo cho host biêt rằng device chẳng ó gì để gửi.

STALL - Device báo rằng trạng thái hiện tại cần cần thiệp từ phía Host.

Packet Handshake sẽ có dạng sau:

**Sync**

**PID**

**EOP**

Packet Start Of Frame:

Hay Packet SOF, chứa dữ liệu là một giá trị 11 bit. Được gửi bởi Host theo chu kì 1ms+- 500ns với thiết bị Full Speed, và chu kì 1us - 0.0625us trên High Speed.

Định dạng của SOF sẽ như sau:

**Sync**

**PID**

**Frame Number**

**CRC5**

**EOP**

USB Functions (hay phía USB Device)

Khi chúng ta nói về một USB Device, chúng ta thường nghĩ ngay đến USB Peripheral. Nhưng một USB Device có nghĩa là một USB Transceiver Device. Nó có thể sử dụng ở phía Host, Peripheral, USB Hub hoặc Host Controller IC Device hoặc là một USB Peripheral Device thật. Chuẩn mà nói, người ta gọi phải là USB Function. Ví dụ, là USB Device có khả năng hoặc chức năng như máy Printer, Zip Drive hoặc các thiết bị hiện đại khác.

Vậy thì, đến lúc này rồi, chúng ta có nên biết mọi thứ cấu tạo nên USB Packet không? Câu trả lời là không. Bạn tốt nhất hãy quên bao nhiêu bit cho trường PID đi là vừa, đừng quá tập trung mấy cái đó. Bởi vì, thật may mắn là hấu hết USB Functions thực hiện từ dưới cho đến tầng transaction (chúng ta sẽ nói ở những Chapter sau) trong phần cứng rồi. Lý do, chúng ta phải biết những thông tin chi tiết phía trên là vì USB Functions sẽ thông báo các lỗi như PID Encoding Error. Nếu không lướt qua phần đó, tôi e rằng sau này các bạn sẽ hỏi Một PID Encoding Error là cái gì thế. Nếu bạn đoán rằng từ cái tên của nó rằng 4 bit cuối của PID sẽ không match với 4 bit còn lại sau khi được đảo, thì bạn đã đúng.

![](/post/images/endpoint.gif)Hầu hết USB Function đều có một loại buffer, thông thường mỗi cái độ dài 8 bytes. Mỗi buffer này sẽ thuộc 1 End Point,  EP0 IN, EP0 OUT chẳng hạn. Lấy một ví dụ thế này, Host gửi cho Device một Device Descripter Request (yêu cầu lây miêu tả về Device). Hardware phía USB Function sẽ đọc Packet Setup và xem trường Address (địa chỉ đến) để xác định xem Packet này có phải dành cho nó không. Nếu là Packet gửi đến cho nó, thì nó sẽ copy payload vào buffer của EndPoint (được xác định bởi trường EndPoint cũng trong Packet Setup). Đồng thời, phần cứng cũng gửi một Packet Handshake để báo rằng nó đã nhận Packet về phía Host. Phần cứng sẽ sinh ra một ngắt bên trong nó để báo cho phía semiconductor/micro-controller biết về Packet nó mới nhận. Đó, phần cứng làm đến mức đó đấy.

Phần mềm sẽ phải xử lý ngắt, đọc dữ liệu từ buffer của EndPoint, rồi parse Device Descriptor Request.

EndPoints

Có thể hiểu là nguồn hay bể chứa dữ liệu. Như ta đã biết USB là xoanh quanh Host, tức là mọi giao tiếp được bắt đầu từ phía Host, EndPoints là điểm cuối của kênh giao tiếp và nằm phía USB Functions. Ở mức phần mềm chẳng hạn, giả sử driver gửi một Packet đến EP1 của thiết bị. Dữ liệu này được ném từ Host, và trôi xuống EP1 OUT Buffer. Firmware phía USB Functions cứ thế đọc dữ liệu này. Rồi xử lý tóe loe gì đó. Sau đó nếu nó muốn trả dữ liệu nào đó về Host, USB Functions lại không thể tống dữ liệu đó vào cái Channel lúc trước được (vì ngược chiều, đâm nhau thì toi). Vì thế, nó ghi dữ liệu vào EP1 IN Bufer. Dữ liệu được ghi vào cứ nằm ở đấy đến khi nào Host gửi Packet IN (yêu cầu lấy dữ liệu) thì mới được chuyển đi. EndPoint cũng được xem như điểm trung chuyển giữa phần cứng của Function Device và Firmware chạy trên Function device.

Tất cả các Device (của Host và Function) đều có EndPoint Zero. Đây là EndPoint duy nhất nhận toàn bộ request Control/Status trong suốt quá trình Enumeration, cũng như lúc thiết bị giao tiếp trên bus.

Pipes

Như ta đã nói ở trên, EndPoint là điểm trung giữa phần cứng và phần mềm trong USB Function Device. Tuy nhiên, nâng cao lên 1 chút ở mức phần mềm (dù vẫn thuộc Firmware - Driver Level) thì người ta thường sử dụng một khái niệm khác nữa. Đó là các Pipes. Một Pipe là một kết nối logical giữa Host và 1 hoặc nhiều Endpoint. Pipe có một tập các tham số kèm theo chúng như bandwidth là bao nhiêu, loại truyền  (transfer type) (Control, Bulk, Iso, Interrupt), hướng của luồng dữ liệu chạy qua, kích thước Buffer của Pipe nữa.  Lấy ví dụ về Default Pipe là một Pipe 2 chiều nối Host với EndPoint Zero IN và EndPoint Zero OUT, loại truyền là Control Transfer.

Về cơ bản, có 2 loại Pipe lớn:

Stream Pipe (Pipe dạng dòng) : truyền dữ liệu không định dạng trước, khi sử dụng loại này bạn có đơn giản là có thể gửi bất cứ dữ liệu gì ở 1 đầu, và lấy dữ liệu ra ở đầu còn lại. Luồng dữ liệu sử dụng Pipe này thường được định nghĩa trước, hoặc là IN hoặc là OUT. Pipe dạng dòng hỗ trợ phương thức truyền Bulk, Isochronous và Interrupt. Stream có thể được điều khiển (controlled về phần mềm) bởi Host hoặc Device.

Message Pipi (Pipe truyền messsage) : Dùng để truyền dữ liệu đã được định nghĩa theo USB Format. Được điều khiển cũng như xuất phát từ Host. Dữ liệu được truyền đi theo hướng mong muốn dựa trên request từ Host. Nó hỗ trợ truyền dữ liệu cả 2 hướng và chỉ hỗ trợ Control Transfer thôi.