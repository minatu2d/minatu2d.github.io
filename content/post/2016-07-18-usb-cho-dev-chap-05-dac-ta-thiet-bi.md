---
layout: post
title: USB cho dev (Chap.05 – Đặc tả thiết bị)
date: 2016-07-18 09:19:07.000000000 +09:00
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
  _publicize_job_id: '24891061030'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/07/18/usb-cho-dev-chap-05-dac-ta-thiet-bi/"
---
Tất cả các thiết bị nằm trong 1 hệ thống phân cấp các miêu tả (hierachy of descriptors), miêu tả cho Host biết các thông tin về Thiết bị gì?Nhà sản xuất? Phiên bản của giao thức USB nó hỗ trợ?Các cách để cấu hình? Số lượng Endpoint và loại truyền tương ứng.

Các đặc tả phổ biết nhất bao gồm:

*   Miêu tả thiết bị (Device Descriptors)
*   Miêu tả các cấu hình (Configuration Descriptors)
*   Miêu tả giao diện (Interface Descriptors)
*   Miêu tả điểm đầu cuối (EndPoint Descriptors)
*   Các chuỗi sử dụng trong các miêu tả trên.

Mỗi thiết bị chỉ có 1 miêu tả thiết bị (Device Descriptors). Thông tin chứa trong mô tả thiết bị bao gồm phiên bản nào của USB mà thiết bị tuân theo, mã sản phẩm (Product IDs), mã nhà sản xuất (Vendor IDs), chính các thông tin này được sử dụng để xác định driver nào cần thiết để có thể giao tiếp được với thiết bị, cũng như số lượng cấu hình mà thiết bị có thể có. Số lượng cấu hình của thiết bị liên quan đến việc cấu hình mà thiết bị có thể theo (tất nhiên mỗi thời điểm chỉ 1).

Miêu tả cấu hình chỉ rõ các giá trị như điện năng sử dụng trong mỗi cấu hình, thiết bị là tự cấp nguồn (self-powered) hay lấy nguồn từ bus (bus-powered), rồi thì số lượng giao diện (interface) của mỗi cấu hình. Trong quá trình enumurated, Host sẽ đọc miêu tả thiết bị, và đưa ra quyết định cấu hình nào được sử dụng. Chỉ được phép sử dụng 1 cấu hình.

Ví dụ, một thiết bị có 2 cấu hình là sử dụng nguồn bus và sử dụng nguồn tự có. nếu thiết bị được cắm vào Host có nguồn ổn định nó có thể chọn cấu hình sử dụng bus, khi đó nó có thể hoạt động mà không cần một nguồn cắm ngoài nào khác. Ngược lại, nếu nó kết nối với một laptop chẳng hạn, nó có thể bật cấu hình sử dụng nguồn tự có, khi đó ta cần kết nối nó với một nguồn khác để có thể giao tiếp được.

Không nhất thiết 2 cấu hình phải có cách sử dụng nguồn khác nhau. Có thể 2 cấu hình sử dụng nguồn giống hệt nhau những khác nhau về các thông số khác như các giao diện, kết hợp các Endpoint. Chú rằng, khi thay đổi cấu hình, Host phải dừng toàn bộ các EndPoint đang sử dụng để chuyển sang cấu hình mới. Dù có thể chưa nhiều cấu hình, nhưng rất ít thiết bị có nhiều hơn 1 cấu hình.

![](/post/images/desctree.gif)

Miêu tả giao diện có thể được xem miêu tả của một nhóm Endpoint phối hợp với nhau để thực hiện một tính năng của thiết bị. Ví dụ: bạn có một thiết bị đa chức năng như fax/scanner/printer chẳng hạn. Miêu tả giao diện 1 có thể là các Endpoint của chức năng fax; giao diện thứ 2 có thể là của chức năng scanner và giao diện thứ 3 dành cho chức năng printer. Không giống miêu tả cấu hình, không có giới về số lượng được phép thực hiện. Tức là một thiết bị có thể có 1 hoặc nhiều miêu tả giao diện cùng chạy tại 1 thời điểm.

Mỗi miêu tả giao diện có 1 trường là **bInterfaceNumber** để chỉ số của Giao diện và **bAlternateSetting**, sẽ cho phép một giao diện thay đổi bất cứ lúc nào. Ví dụ, nếu một thiết bị có 2 giao diện, giao diện 1 và giao diện 2 chẳng hạn. Giao diện thứ nhất sẽ giá trị **bInterfaceNumber** được gán là zero (tức là giao diện đầu tiên), và **bAlternateSetting** cũng là 0 (tức là cấu hình mặc định). Giao diện thứ 2 sẽ có **bInterfaceNumber** là 1, tức là chỉ giao diện thứ 2 và cũng **bAlternateSetting** được gán là 0 (tức là cấu hình mặc định). Ta cũng có thể đưa ra một miêu tả giao diện khác với  **bInterfaceNumber** là 1 (giao diện 2), nhưng **bAlternateSetting** tức là chỉ một setting khác của giao diện 2. Khi cấu hình trên được sử dụng, tức là 2 miêu tả giao diện trên sẽ được sử dụng với **bAlternativeSettings** là zero. Tuy nhiên, trong quá trình sử dụng, Host có thể gửi một yêu cầu **SetInterface** đến interface 1 yêu cầu nó chuyển sang setting thay thế. Khi đó nó sẽ chạy trên một miêu tả giao diện khác.

![](/post/images/intftree.gif)Từ cấu hình trên, ta thấy chúng ta có được đến 2 cấu hình. Chúng ta vẫn có thể gửi dữ liệu qua interface zero, trong khi thay đổi setting Endpoint liên quan đến interface 1 mà không ảnh hưởng đến interface zero.

Mỗi miêu tả Endpoint được sử dụng để chỉ rõ loại truyền, hướng truyền, tần số polling (hỏi thiết bị) và kích thước lớn nhất của packet cho mỗi Endpoint. Endpoint Zero mặc định được sủ dụng là Endpoint điều khiển và nó không bao giờ có miêu tả.

Sự kết hợp các loại đặc tả (Composition of USB Descriptors)

Tất cả các miêu tả từ thiết bị, cấu hình, giao diện etc, đều có một định dạng chung nhất. Byte đầu tiên chỉ ra độ dài của miêu tả, trong khi byte thứ 2 chỉ loại đặc tả. Nếu độ dài của một đặc tả nhỏ hơn kích thước mà đặc trả USB đã định nghĩa thì Host sẽ bỏ qua luôn. Tuy nhiên, nếu kích thước lớn hơn thì Host sẽ bỏ phần thừa ra và tìm miêu tả kế thiếp từ vị trí cuối cùng độ dài thực tế.

Offset

Field

Size

Value

Ý nghĩa

0

bLength

1

Number

Kích thước Descriptor tính theo Bytes

1

bDescriptionType

1

Constant

Loại đặc tả

2

...

n

Các tham số của đặc tả

Miêu tả thiết bị (Device Descriptors)

Sẽ miêu tả toàn bộ thông tin về thiết bị. Tất nhiên, vì vậy nó chỉ có 1 thôi. Nó diễn tả một vài thông tin cơ bản, trong đó có một vài thông tin quan trọng như phiên bản USB nó hỗ trợ, kích thước tối đa của packet, nhà sản xuất (Vendor), mã sản phầm. Số lượng cấu hình mà thiết bị có. Định dạng được miêu tả dưới bảng sau:

Offset

Field

Size

Value

Description

0

bLength

1

Number

Kích thước của Descriptor (18 bytes)

1

bDescriptorType

1

Constant

Là loại Device Descriptors (0x01)

2

bcdUSB

2

BCD

Phiển bản đặc tả USB mà thiết bị tuân theo.

4

bDeviceClass

1

Class

Max class (Được gán bởi USB Org)

Nếu nó bằng 0 thì mỗi interface nên có class riêng của nó.

Nếu là 0xFF thì class này do Vendor tự quy định.

Ngoài ra thì mọi giá trị đều hợp lệ.

5

bDeviceSubClass

1

SubClass

Mã của Subclass  (Được gán bởi USB Org)

6

bDeviceProtocol

1

Protocol

Mã protocol (Assigned by USB Org)

7

bMaxPacketSize

1

Number

Kích thước lớn nhất của Packet khi sử dụng Endpoin Zero (mặc định phải có). Các giá trị có thể là 8, 16, 32, 64

8

idVendor

2

ID

Vendor ID (Assigned by USB Org)

10

idProduct

2

ID

Product ID (Assigned by Manufacturer)

12

bcdDevice

2

BCD

Device Release Number

14

iManufacturer

1

Index

Chỉ số của chuỗi mô tả nhà sản xuất

15

iProduct

1

Index

Chĩ số của chuỗi mô tả sản phẩm

16

iSerialNumber

1

Index

Chỉ số của mã Serial của sản phẩm

17

bNumConfigurations

1

Integer

Số cấu hình hiện có của thiết bị

*   bcdUSB : phiên bản cao nhất của USB mà thiết bị hỗ trợ. Giá trị ở dạng 0xJJMN, JJ là phiên bản major, M là miên bản minor, N là sub minor. Ví dụ USB 2.0 sẽ là 0x0200, USB1.1 sẽ là 0x1100, và USB1.0 sẽ là 0x1000.
*   bDeviceClass, bDeviceSubClass và bDeviceProtocol : Là thông số chủ yếu để hệ điều hành phía Host xác định driver cho thiết bị. Thông thường, chỉ bDeviceClass được set ở mức miêu tả thiết bị, còn 2 thông số còn lại được xác định tại các mức miêu tả interface. Điều này cho phép một thiết bị hỗ trợ nhiều class.
*   bMaxPacketSize : Kích thước packet của Endpoint Zero. Tất cả các thiết bị bắt buộc phải support EndPoint Zero.
*   idVendor và idProduct : được hệ điều hành phía Host sử dụng để tìm driver cho thiết bị, VendorID được gán bởi _[USB-IF](http://www.usb.org). _
*   bcdDevice: có cùng định dạng với bcdUSB và được sử dụng để cung cấp phiên bản của thiết bị. Giá trị này sẽ được gán bởi developer người phát triển thiết bị.
*   3 chuỗi miêu tả sẽ tồn tại để cung cấp những thông tin chi tiết về nhà sản xuất, sản phẩm, số serial. Không bắt buộc phải có 3 chuỗi này, nếu không có 2 chuỗi này thì index zero sẽ được sử dụng.
*   bNumConfigurations : định nghĩa số lượng cấu hình mà thiết bị hỗ trợ tại tốc độ hiện tại của nó.

Miêu tả cấu hình (Configuration Descriptors)

Một thiết bị có thể có một vài cấu hình khác nhau mặc dù hầu hết đơn giản và chỉ có 1 mà thôi. Miêu tả cấu hình sẽ chứa các thông tin như nguồn thiết bị sử dụng là gì? năng lượng tối đa tiêu thụ là bao nhiêu?Số lượng giao diện mà nó có? Vì thế, ví dụ ta có thể có 2 cấu hình, thứ nhất được dùng khi sử dụng nguồn từ bus (bus powered), thứ 2 được dùng khi sử dụng nguồn chính nó (self-powered). Hoặc ta có thể có một cấu hình mà sử dụng phương thức truyền khác với cấu hình còn lại. etc.

Tất cả những cấu hình này được thiết bị gửi đến Host. Phía Host sẽ đánh giá xem nên sử dụng cấu hình nào, và gửi yêu cầu **SetConfiguration** về phía thiết bị Function với tham số là giá trị được lấy ra từ trường **bConfigurationValue** của 1 trong những cấu hình nó được gửi cho.

Offset

Field

Size

Value

Description

0

bLength

1

Number

Kích thước của miêu tả cấu hình tính theo byte

1

bDescriptorType

1

Constant

Loại mô tả là Configuration (0x02)

2

wTotalLength

2

Number

Tổng số byte trả về từ phần dữ liệu

4

bNumInterfaces

1

Number

Số lượng giao diện (interfaces)

5

bConfigurationValue

1

Number

Gía trị được sử dụng như là tham số khi set cấu hình sử dụng

6

iConfiguration

1

Index

Chỉ số của chuỗi miêu tả về cấu hình này

7

bmAttributes

1

Bitmap

D7 : (bit số 7) không sử dụng, set to 1. (USB 1.0 thì nghĩa là Bus Powered)

D6 : Nếu được set, nó sẽ là self-powered

D5 : Dùng để Remote Wakeup (một tín hiệu từ thiết bị đến Host mà không đợi Host hỏi)

D4..0 : Không sử dụng, được gán toàn bộ là 0.

8

bMaxPower

1

mA

Lượng điện tiêu thụ tính theo đơn vị  2mA

Khi một mô tả cấu hình được đọc, nó sẽ trả về toàn bộ cấu trúc cây của cấu hình, bao gồm toàn bộ các interface liên quan, các miêu tả Endpoint nữa. Trường **wTotalLength** miêu tả số lượng byte trong cấu trúc được gửi về Host.

![](/post/images/confsize.gif)

*   bNumInterfaces : là số lượng interface có trong cấu hình này
*   bConfigurationValue : được sử dụng bởi **SetConfiguration** để chọn cấu hình này.
*   iConfiguration : chỉ số của chuỗi mô tả cấu hình này (là chuỗi dễ đọc)
*   bmAttributes : các tham số cho cấu hình này, bao gồm : nếu là self-powered powered thì D6 (bit số 6) được set. Bit D7 được sử dụng để xác định là bus-power hay không chỉ trong trường hợp USB1.0, các phiên bản sau sử dụng giá trị bMaxPower. Nếu thiết bị có sử dụng nguồn từ bus thì bắt buộc phải khai báo giá trị bMaxPower. Bit D5 được sử dụng để xác định xem có hỗ trợ remote wakeup (phía Function chủ động đánh thức Host khi Host đang ở trạng thái nghỉ)
*   bMaxPower : quy định giá trị lớn nhất của dòng mà thiết bị sẽ chạy. Đơn vị ở đây là 2m Ampe. Thiết bị được phép chạy tối đa lên đến 500m Ampe. Các thiết bị high speed mà sử dụng nguồn bus thì không được phép lớn hơn 500m Ampe. Nếu một thiết bị mất nguồn bên ngoài

Mô tả giao diện (Interface Descriptors)

Giao diện có thể được thấy là một nhóm các End Point thực hiện một chức năng của thiết bị. Miêu tả giao diện tuân theo các định dạng sau:

Offset

Field

Size

Value

Description

0

bLength

1

Number

Kích thước của miêu tả tính bằng byte (9 Bytes)

1

bDescriptorType

1

Constant

Là loại miêu tả giao diện (0x04)

2

bInterfaceNumber

1

Number

Số lượng giao diện

3

bAlternateSetting

1

Number

Gía trị được sử dụng thay đổi giao diện tức thời(alternative setting)

4

bNumEndpoints

1

Number

Số lượng Endpoints có trong Interface

5

bInterfaceClass

1

Class

Mã class (lấy từ danh sách được quy định bởi USB Org)

6

bInterfaceSubClass

1

SubClass

Mã Subclass (lấy từ danh sách được quy định bởi USB Org)

7

bInterfaceProtocol

1

Protocol

Mã giao thức (Protocol) (lấy từ danh sách được quy định bởi USB Org)

8

iInterface

1

Index

Chỉ số của các chuỗi của interface này trong miêu tả chuỗi

*   bInterfaceNumber: chỉ số của interface hiện tại. Giá trị này bắt đầu từ Zero và tăng lên với mỗi interface mới.
*   bAlternativeSetting: được sử dụng cho giao diện thay thế tức thời thông qua Set Interface Request.
*   bNumEndpoints: số lượng Endpoint được sử dụng trong Interface. Giá trị này không bao gồm Endpoint Zero, mỗi Endpoint được miêu tả kĩ ở phần sau.
*   bInterfaceClass, bInterfaceSubClass, bInterfaceProtocol: sử dụng để chỉ ra class nào được support (lớp thiết bị nào) (ví dụ: HID - chuột, bàn phím chẳng hạn, Communication - USB2Serial chẳng hạn, Mass Storage - USB Flash chẳng hạn). Các giá trị này sẽ giúp cho việc phân chia driver cho thiết bị thành các lớp nên giảm việc phải viết driver cho từng thiết bị cụ thể.
*   iInterface: Chỉ số của chuỗi mô tả cho interface này.

Mô tả EndPoints

Miêu tả các Endpoint khác Endpoin Zero. End Point luôn được được sử dụng là Control Endpoint (Endpoint điều khiển thiết bị) trước khi bất cứ miêu ta nào được Host yêu cầu. Định dạng trả sẽ theo bảng dưới đây:

Offset

Field

Size

Value

Description

0

bLength

1

Number

Kích thước của miêu tả tính bằng byte (7 bytes)

1

bDescriptorType

1

Constant

Là miêu tả EndPoint (0x05)

2

bEndpointAddress

1

Endpoint

Địa chỉ Endpoint  
Bits 0..3b Số Endpoint (tối đa là 16)  
Bits 4..6b Không sử dụng  
Bits 7 Hướng, 0 = Đi ra, 1 = Đi vào (không tính Endpoint Control)  

3

bmAttributes

1

Bitmap

Bits 0..1 Loại truyền

00 = Control (điều khiển)

01 = Isochronous (đồng bộ)

10 = Bulk (hàng loạt)

11 = Interrupt (ngắt)

Bits 2..7 Không sử dụng nếu là Endpoint Isochronous.  
Bits 3..2 = Loại đồng bộ (chỉ có Iso 0x01)

00 = No Synchonisation (không đồng bộ)

01 = Asynchronous (đồng bộ bất đối xứng)

10 = Adaptive (đồng bộ thích nghi)

11 = Synchronous (đồng bộ đối xứng)

Bits 5..4 = Loại sử dụng (chỉ có mode Iso 0x01)

00 = Data Endpoint (Endpoint data)

01 = Feedback Endpoint (Endpoint đáp lại)

10 = Explicit Feedback Data Endpoint (Endpoint trả lại data)

11 = Reserved (Không sử dụng )

4

wMaxPacketSize

2

Number

Kích thước lớn nhất cho Packet Size mà có thể gửi/nhận trên Endpoint này.

6

bInterval

1

Number

Tần suất để polling Data từ Endpoint. Value in frame counts. Ignored for Bulk & Control Endpoints. Isochronous must equal 1 and field may range from 1 to 255 for interrupt endpoints.

*   bEndpointAddress: Tức là Endpoint nào
*   bmAttributes: Loại truyền dữ liệu sử dụng trên EndPoint đó. Nó là 1 trong 4 loại đã nói ở các chương trước: Control, Interrupt, Isochronous, Bulk. Nếu một Endpoint Isochronous được sử dụng, thì các thuộc tính khác cũng được sử dụng theo, còn lại nó không có ý nghĩa gì hết.
*   wMaxPacketSize: Kích thước tối đa mà payload có thể chứa khi truyền nhận trên Endpoint này.
*   bInterval: Tần số polling (tần số hỏi dữ liệu). Đơn vị được tính là frame. (1ms cho low/full speed, và 125us cho high speed)

Miêu tả chuỗi (String Descriptor)

Phần miêu tả này chưa thông tin về các chuỗi miêu tả các loại Descriptor khác, giúp người đọc dễ dàng nhận biết các thông tin. Các chuỗi này là không bắt buộc. Các loại miêu tả không sử dụng các chuỗi dễ đọc thì trường tham chiếu đến chuối dễ đọc nên để là ZERO.

Các chuỗi này được Encode theo định dạng của tổ chức [Unicode](http://www.unicode.org/), có thể hỗ trợ đa ngôn ngữ. Các chuỗi được miêu tả cũng vậy, nên support nhiều ngôn ngữ theo các mã ngôn ngữ được miêu tả ở [Universal Serial Bus Language Identifiers (LANGIDs) version 1.0](http://www.usb.org/developers/data/USB_LANGIDs.pdf).

Offset

Field

Size

Value

Description

0

bLength

1

Number

Kích thước của miêu tả (tính bằng byte)

1

bDescriptorType

1

Constant

Đây là miêu tả chuỗi (0x03)

2

wLANGID[0]

2

number

Ngôn ngữ thứ nhất (e.g. 0x0409 English - United States)

4

wLANGID[1]

2

number

Ngôn ngữ thứ 2(e.g. 0x0c09 English - Australian)

n

wLANGID[x]

2

number

Ngôn ngữ thứ x (e.g. 0x0407 German - Standard)

Miêu tả chuỗi trên là miêu tả của Miêu tả chuỗi Zero. Host nên đọc các thông tin miêu tả này để tìm ra ngôn ngữ nào được support. Nếu một ngôn ngữ được hỗ trợ, nó sẽ được tham chiếu thông qua LanguageID được gán vào giá trị trường wIndex của Get Descriptor(String).

Tất các các chuỗi được miêu tả sau đó sẽ có dạng sau:

Offset

Field

Size

Value

Description

0

bLength

1

Number

Kích thước ở dạng byte

1

bDescriptorType

1

Constant

Dang miêu tả chuỗi (0x03)

2

bString

n

Unicode

Chuỗi được Encode bởi chuẩn Unicode