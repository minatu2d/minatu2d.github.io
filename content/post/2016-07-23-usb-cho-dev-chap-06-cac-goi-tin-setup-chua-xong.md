---
layout: post
title: USB cho dev (Chap.06 – Các gói tin Setup)
date: 2016-07-23 05:46:28.000000000 +09:00
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
  _publicize_job_id: '25068611680'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/07/23/usb-cho-dev-chap-06-cac-goi-tin-setup-chua-xong/"
---
Mỗi thiết bị USB phải trả lời các gói tin Setup (Setup packets) trên Endpoint mặc định (Endpoint Zero). Các gói tin Setup được sử dụng cho việc phát hiện thiết bị, cấu hình, cũng như lấy các thông tin khác như các thông tin về chức năng, địa chỉ thiết bị, kiểm tra trạng thái các Endpoint.

Chuẩn USB yêu cầu Host sẽ mon muốn về mặt thời gian từ phát hiện đến lấy đầy đủ các thông tin trên trong vòng không quá 5 giây. Ngoài ra, nó cũng quy định chặt hơn cho từng Request từ phía Host.

*   Yêu cầu thông tin chuẩn của thiết bị (Standard Device request) mà không có dữ liệu (hay DATA Stage) thì phải hoàn thành trong vòng 50ms.
*   Nếu yêu cầu có dữ liệu kèm theo (có DATA stage) thì dữ liệu phải được trao đổi sau ít nhất 500ms sau request được gửi đến.
    *   Mỗi gói dữ liệu phải gửi trong 500ms sau khi gói trước được gửi
    *   Thông báo trạng thái (Status Stage) phải hoàn thành trong vòng 50ms sau khi gửi gói tin cuối cùng.
*   Lệnh SetAddresss (có chưa phần Data) phải được thực hiện và trả lại Status trong vòng 50ms. Phía thiết bị có 2ms để thực hiện thay địa chỉ trước khi request tiếp theo được gửi đến.Nhưng yêu cầu về thời gian ở trên hoàn toàn có thể thực hiện được ngay cả với những thiết bị chậm nhất đi nữa. Nhưng những giới hạn về thời gian này có thể gây khó khăn trong quá trình debug. Trong vòng 50ms, chẳng thể có kí tự debug nào được gửi đi ở 9600bps ở dạng serial không đồng bộ hoặc một Emulator cho mỗi step chạy ở các thanh ghi. Chính vì thế, Khi debug USB, người cần có một vài phương pháp khác. (cái này chưa bàn đến)

Mỗi request sẽ bắt đầu bằng 8 byte, theo định dạng sau:

Offset

Field

Size

Value

Description

0

bmRequestType

1

Bit-Map

**D7 Hướng truyền DATA**  
0 = Host to Device  
1 = Device to Host  
**D6..5 Loại**  
0 = Standard  
1 = Class  
2 = Vendor  
3 = Reserved  
**D4..0 Phía nhận**  
0 = Device  
1 = Interface  
2 = Endpoint  
3 = Other  
4..31 = Reserved  

1

bRequest

1

Value

Request

2

wValue

2

Value

Value

4

wIndex

2

Index or Offset

Index

6

wLength

2

Count

Số lượng byte truyền trong 1 Data Stage

**bmRequestType**: Xác định chiều của request (phần DATA), loại request, cũng như loại thông tin sẽ tiếp nhận.

**bRequest**: Muốn nói rằng, đây là Request.

Thông thường bmRequestType được đọc, và sẽ được chia ra các xử lý theo các handler như Standard Device request Handler, Standard Interface request Handler, hoặc Standard Endpoint request Handler, hay Class Device request handler.etc. Tất nhiên, cách xử lý trên không bắt buộc, biệc đọc nội dung packet Setup cũng như xử lý phụ thuộc hoàn toàn các thiết kế phía Device của bạn. Một số chọn cách đọc bRequest trước tiên sau đó mới xác định đến loại thông tin sẽ tiếp nhận.

Các request chuẩn (Standard request) là phổ biến ở tất cả các thiết bị USB, nó sẽ được nói chi tiết ở bên dưới. Classr request phổ biến cho các lớp Driver. Ví dụ, tất cả các thiết bị tuân theo HID class thì đều có một tập các request class chung. Tập class này khác với Communication class, và tất nhiên khác với Mass Storage class.

Và cuối cùng, là các request mà các vendor tự định nghĩa. Nhưng request này được người thiết kế thiết bị tự đặt, các thiết bị thường không giống nhau. Và tất nhiên, người ta có thể làm bất cứ thứ gì người ta muốn.

Một request phổ biến có thế hướng đến nhiều đối tượng khác nhau (thiết bị, giao diện, hay Endpoint). GetStatus Standard request chẳng hạn, nó có thể cho thiết ị, giao diện hoặc Endpoint. Khi hướng đến thiết bị, nó sẽ trả về trạng thái của remote wake up và nếu thiết bị là tự cung cấp nguồn. Tuy nhiên, nếu cùng request đó được gửi đến interface thì nó luôn trả lại zero, còn đến Endpoint thì nó lại trả lại một dừng tạm thời cho EndPoint đó.

Giá trị wValue và wIndex cho phép các tham số được kèm theo request. wLength chỉ số lượng byte được truyền ở DATA stage.

Request chuẩn (Standard Request)

Ở Section 9.4 trong đặc tả về USB, có nói rằng, "Standard request" phải được thực hiện trên mọi USB Device. Tài liệu đó cũng cung cấp một bảng của các request có thể. Ta hãy coi rằng phía firmware của thiết bị USB sẽ thực hiện đọc các gói Setup chưa chi chúng ra để xử lý tướng ứng vỡi mỗi loại đối tượng là Device, Interface hay Endpoint.

Request chuẩn cho Device (Standard Device Request)

Có 8 loại request chuẩn cho thiết bị, được liệt kê ở bảng dưới đây:

bmRequestType

bRequest

wValue

wIndex

wLength

Data

1000 0000b

GET_STATUS (0x00)

Zero

Zero

Two

Device Status

0000 0000b

CLEAR_FEATURE (0x01)

Feature Selector

Zero

Zero

None

0000 0000b

SET_FEATURE (0x03)

Feature Selector

Zero

Zero

None

0000 0000b

SET_ADDRESS (0x05)

Device Address

Zero

Zero

None

1000 0000b

GET_DESCRIPTOR (0x06)

Descriptor Type & Index

Zero or Language ID

Descriptor Length

Descriptor

0000 0000b

SET_DESCRIPTOR (0x07)

Descriptor Type & Index

Zero or Language ID

Descriptor Length

Descriptor

1000 0000b

GET_CONFIGURATION (0x08)

Zero

Zero

1

Configuration Value

0000 0000b

SET_CONFIGURATION (0x09)

Configuration Value

Zero

Zero

None

*   GET_STATUS : Device phải trả lại 2 byte theo dạng dưới đây:

D15

D14

D13

D12

D11

D10

D9

D8

D7

D6

D5

D4

D3

D2

D1

D0

Reserved

Remote Wakeup

Self Powered

Nếu D0 được set, thì có nghĩa thiết bị là nguồn tự cung, ngược lại thì đó là nguồn Bus.

Nếu D1 được set, thì có nghĩa là Remote wakep up được bật, thiết bị có thể wake up Host khi Host đang suspended. Bit này có thể được thay đôi bởi SET_FEATURE hoặc CLEAR_FEATURE.

*   CLEAR_FEATURE và SET_FEATURE: Sử dụng để set có hay không các feature. Khi nó hướng đên Device thì chỉ có 2 feature có thể set. Đó là DEVICE_REMOTE_WAKEUP và TEST_MODE. Test mode cho phép thiết bị diễn đạt nhiều điều kiện khác nhau. Chi tiết phần này được nói đến ở đặc tả USB 2.0.
*   SET_ADDRESS: Sử dụng để gán một địa chỉ duy nhất cho thiết bị trong quá trình Enumuration. Chính là giá trị của trường wValue, tối đa đến 127. Ở request này, thiết bị dù đã có giá trị địa chỉ nhưng vẫn phải đợi thông báo trạng thái trước khi thực sự set địa chỉ cho chính nó. Còn với các request khác, nó thường được thực hiện trước khi Status đến.
*   SET_DESCRIPTOR/GET_DESCRIPTOR: được sử dụng để trả về một descriptor trong wValue. Một request cho miêu tả cấu hình sẽ trả về tất cả các thông tin của miêu tả thiết bị, các interface, các Endpoint nữa. Có 1 chú ý ở đây:
    *   Endpoint Descriptor: không thể được truy cập trực tiếp từ GET_DESCRIPTOR/SET_DESCRIPTOR request.
    *   Interface Descriptor: giống với Endpoint Descriptor.
    *   String Descriptor: cho phép truy cập trực tiếp
*   GET_CONFIGURATION/SET_CONFIGURATION: Được sử dụng để lấy thông tin cấu hình thiết bị hoặc set cấu hình. Trạng thái trả về ở Status Stage là 1 byte giá trị. Nếu là 0, thì có nghĩa thiết bị chưa được configured, nếu khác 0 thì có nghĩa thiết bị đã được configured. SET_CONFIGURATION được sử dụng để cho phép một thiết bị chính thức hoạt động. Nó chứa giá trị của bConfigurationValue mà Host muốn thiết lập.

Request Giao diện chuẩn

Định nghĩa 5 loại yêu cầu giao diện chuẩn, chi tiết được nêu dưới bảng sau. Chỉ có 2 request là được sử dụng một cách rõ ràng.

bmRequestType

bRequest

wValue

wIndex

wLength

Data

1000 0001b

GET_STATUS (0x00)

Zero

Interface

Two

Interface Status

0000 0001b

CLEAR_FEATURE (0x01)

Feature Selector

Interface

Zero

None

0000 0001b

SET_FEATURE (0x03)

Feature Selector

Interface

Zero

None

1000 0001b

GET_INTERFACE (0x0A)

Zero

Interface

One

Alternate Interface

0000 0001b

SET_INTERFACE (0x11)

Alternative Setting

Interface

Zero

None

wIndex: Tức là chỉ số của Interface mà request này hướng đến, định dạng được nêu ra ở dưới đây:

D15

D14

D13

D12

D11

D10

D9

D8

D7

D6

D5

D4

D3

D2

D1

D0

Reserved

Interface Number

*   GET_STATUS: được sử dụng để trả về trạng thái của interface. Giá trị thường sẽ là 2 byte (0x00, 0x00)
*   CLEAR_FEATURE/SET_FEATURE: Sử dụng để set các giá trị boolean. Khi request cho giao diện thì không có đặc trưng này.
*   GET_INTERFACE/SET_INTERFACE: sẽ có trường Alternative Interface mà ta đã miêu tả ở các chương trước.

Request Endpoint chuẩn (Standard Endpoint Request)

Có 4 loại khác nhau được đưa ra dưới đây:

bmRequestType

bRequest

wValue

Windex

wLength

Data

1000 0010b

GET_STATUS (0x00)

Zero

Endpoint

Two

Endpoint Status

0000 0010b

CLEAR_FEATURE (0x01)

Feature Selector

Endpoint

Zero

None

0000 0010b

SET_FEATURE (0x03)

Feature Selector

Endpoint

Zero

None

1000 0010b

SYNCH_FRAME (0x12)

Zero

Endpoint

Two

FrameNumber

*   wIndex: tham chiếu đến Endpoint và hướng của chúng. Định dạng sẽ như bên dưới:

D15

D14

D13

D12

D11

D10

D9

D8

D7

D6

D5

D4

D3

D2

D1

D0

Reserved

Dir

Reserved

Endpoint Number

*   GET_STATUS:trả lại 2 bytes trạng thái (Halted/Stalled) của Endpoint. Định dạng của 2 byte đó như sau:

*   D15
    
    D14
    
    D13
    
    D12
    
    D11
    
    D10
    
    D9
    
    D8
    
    D7
    
    D6
    
    D5
    
    D4
    
    D3
    
    D2
    
    D1
    
    D0
    
    Reserved
    
    Halt
    

*   CLEAR_FEATURE/SET_FEATURE: Sử dụng để set Endpoint Features. Hiện tại chỉ có 1 feature selector, ENDPOINT_HALT(0x00) giúp Host có thể ngưng stall hoặc reset một Endpoint. Chỉ những Endpoint khác Endpoint mặc định mới nên sử dụng chức năng này.
*   Synch frame: sử dụng để báo một frame đồng bộ Endpoint.