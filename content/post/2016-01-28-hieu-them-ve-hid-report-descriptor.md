---
layout: post
title: Hiểu thêm về HID Report Descriptor
date: 2016-01-28 15:44:23.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
- Communication
tags:
- Draft
- HID
- USB
- USBDescriptor
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: '43452'
  _publicize_job_id: '19216329154'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/01/28/hieu-them-ve-hid-report-descriptor/"
---
Đang hì hục đọc sách các kiểu con đà điều để hiểu kĩ hơn về HID Report Descriptor (Đặc tả cấu trúc dữ liệu trao đổi của HID). Thì tìm được bài này, nó giải thích hầu hết những chỗ khó hiểu một cách dễ hiểu, và đặc biệt có ví dụ minh họa.

Giờ xin dịch lại bài này một cách khái quát nhất.

Vì để hiểu bài này cần biết đến một vài khái niệm về USB, về HID Device, nữa nên để xin tóm tắt nội dung bên dưới như sau. Qua giao tiếp USB, máy tính có thể giao tiếp với rất nhiều thiết bị từ USB Flash Memory (hay gọi là USB), chuột, bàn phím,..etc. Vì có rất nhiều thiết bị có thể kết nối được, nên để dễ phân biệt và dễ dàng cho việc phát triển Driver trên máy tính, người ta chia thành các lớp thiết bị. Có rất nhiều lớp không thể kể hết được, nhưng xin có 2 lớp chính đó là Mass Storage Device (chính là USB Flash Memory đấy), và HID Device (chính là chuột, bàn phím).

Về nguyên tắc, từ khi cắm 1 thiết bị USB bất kì cắm vào cổng, cần trải qua 3 giai đoạn giao tiếp giữa máy tính và thiết bị để có thể hiểu nhau. Đó là Reset (về điện đóm), Configured (load các driver tương ứng), Sử dụng được (khi chúng di chuyển được con chuột, hoặc gõ được phím).

Trong bài này giới thiệu dữ liệu trao đổi giữa HID Driver (nằm trên máy tính) và HID Device (hay về mặt phần mềm là Firmware chạy trên Device). Mà cụ thể nói về "HID Report Descriptor", cái được HID Device gửi lên HID Driver để báo cho Driver biết HID Device có những dữ liệu gì mà Driver có thể hỏi được, được tổ chức như thế nào.

Link : [http://eleccelerator.com/tutorial-about-usb-hid-report-descriptors/](http://eleccelerator.com/tutorial-about-usb-hid-report-descriptors/)

Một USB HID Report là một trong những descriptor (đặc tả) mà Host yêu cầu từ thiết bị USB. Thiết bị USB sẽ trả lời yêu cầu này bằng các Report. Nhưng Report này sẽ nói cho Host biết các dữ liệu trong quá trình sử dụng mà Device gửi lên nên được hiểu như thế nào. Trong bài này, tác giả Frank-Zhang(Sony Engineer) đã giới thiệu theo từng ví dụ.

Đầu tiên, hãy vào trang [http://www.usb.org/developers/hidpage/](http://www.usb.org/developers/hidpage/) và tìm tài liệu có tên  “Device Class Definition for HID”(Định nghĩa lớp cho các thiết bị HID), trong bài này tác giả sẽ nói về phần cơ bản nhất để hiểu tài liệu này.

Thứ hai, có thể tải Tool tên là HID Descriptor Tool từ cùng link ở trên luôn. Tool này hỗ trợ việc Edit, Parse một miêu tả HID ở dạng Binary sang dạng người thiết kế dễ dàng đọc hơn.

## USB HID report descriptor là cái gì?

> Giao thức HID giúp việc implement thiết bị trở nên rất dễ dàng. Thiết bị tự định nghĩa các gói dữ liệu của nó và gửi đến Host trông qua các  “HID descriptor”.  HID descriptor là một mảng 1 chiều không hơn không kém, miêu tả gói mà thiết bị định gửi sau đó.  Nó bao gồm : thiết bị hỗ trợ bao nhiêu gói dữ liêu, các gói có kích thước ra sao,  và mục đích của mỗi byte trong dữ liệu. Ví dụ, một phím tính toán trên một keyboard có thể được miêu tả với Host rằng 2 trạng thái (nhấn/nhả) được miêu tả trong bít thứ 2, của byte thứ 6 trong packet số 4.(chú ý là đây chỉ là giả đinh thôi). Một thiết bị thông thường lưu trữ HID Descriptor của nó trong ROM và đương nhiên không cần phải Parse lại làm gì. Nhiều chuột và bàn phím hiện nay được implement chỉ với CPU-8bit.

– Wikipedia on Human Interface Device

Trong bài viết này, tác giả sẽ dạy chúng ta về USB HID Report Descriptors (Đặc tả mẫu báo cáo dữ liệu cho thiết bị USB HID) bằng một vài ví dụ cụ thể.

Để cho đơn giản, chúng ta hãy cùng tạo Report Descriptor cho một mouse chuẩn gồm: 3 nút bấm và toa độ di chuyển X, Y. Chúng ta tất nhiên sẽ muốn gửi thông tin về 3 nút kia và tọa độ di chuyển của chuột. Ta cần 1 bit để miêu tả mỗi nút, và 1 byte (giá trị có dấu) để miêu tả mỗi tọa độ. Ta có thể miêu tả đơn giản bằng một bảng các byte như sau:

Bit 7

Bit 6

Bit 5

Bit 4

Bit 3

Bit 2

Bit 1

Bit 0

Byte 0

Ko sd

Ko sd

Ko sd

Ko sd

Ko sd

Nút trái

Nút giữa

Nút phải

Byte 1

Giá trị có dấu miêu tả tọa độ tương đối X

Byte 2

Gia trị có dấu miêu tả tọa độ tương đối Y

Có thể miêu dữ liệu trên trong một cấu trúc ngôn ngữ C như thế này:

`struct` `mouse_report_t`

`{`

`uint8_t buttons;`

`int8_t x;`

`int8_t y;`

`}`

Giờ, ta sẽ miêu tả dữ liêu trên trong Report Descriptor.

Đầu tiên, ta miêu tả rằng, ta cần một nhóm 3 nút.

`USAGE_PAGE (Button)`

`USAGE_MINIMUM (Button 1)`

`USAGE_MAXIMUM (Button 3)`

Mỗi nút có trạng thái biểu diễn bằng 1 bít (0/1 tương đương với nhấn và không được nhấn).

`LOGICAL_MINIMUM (0)`

`LOGICAL_MAXIMUM (1)`

Và ta cần 1 nhóm 3 bít này, nó sẽ được biểu diễn như sau:

`REPORT_COUNT (3)`

`REPORT_SIZE (1)`

Cuối cùng, khai báo rằng đây là 1 input, nó cần được gửi đến máy tính:

`INPUT (Data,Var,Abs)`

Cuối cùng, 3 nút được biểu diễn bằng dãy sau:

`USAGE_PAGE (Button)`

`USAGE_MINIMUM (Button 1)`

`USAGE_MAXIMUM (Button 3)`

`LOGICAL_MINIMUM (0)`

`LOGICAL_MAXIMUM (1)`

`REPORT_COUNT (3)`

`REPORT_SIZE (1)`

`INPUT (Data,Var,Abs)`

OK rồi. nhưng đến đây chưa xong, ta nhớ rằng, cần miêu tả đến mỗi byte được sử dụng như thế nào. Ta ở đây còn 5 bit chưa sử dụng của byte đầu tiên. Ta cần phải báo cho Host biết nó sẽ được dùng làm gì. Hoặc không được dùng làm gì hết thì ta khai báo như sau:

`REPORT_COUNT (1)`

`REPORT_SIZE (5)`

`INPUT (Cnst,Var,Abs)`

Vậy là 3 nút bấm, trong byte đầu tiên luôn.

Giờ ta đến với tọa độ X, đầu tiên ta cũng cần khai báo rằng đây là X.

`USAGE_PAGE (Generic Desktop)`

`USAGE (X)`

Giá trị ó có thể nhận trong khoảng -127 đến 128, ta miêu tả như sau:

`LOGICAL_MINIMUM (-127)`

`LOGICAL_MAXIMUM (127)`

Cần 1 bộ 8 bit.

`REPORT_SIZE (8)`

`REPORT_COUNT (1)`

Và đây là dữ liệu input, cần gửi đến máy tính;

`INPUT (Data,Var,Rel)`

Và ta được dãy miêu tả tọa độ X như sau:

`USAGE_PAGE (Generic Desktop)`

`USAGE (X)`

`LOGICAL_MINIMUM (-127)`

`LOGICAL_MAXIMUM (127)`

`REPORT_SIZE (8)`

`REPORT_COUNT (1)`

`INPUT (Data,Var,Rel)`

Ta sư dụng trọn ven trọng 1 byte nên không có phần thừa ở đây.

Với tọa độ Y, ta cũng tương tự có tọa độ X, Y như sau:

`USAGE_PAGE (Generic Desktop)`

`USAGE (X)`

`LOGICAL_MINIMUM (-127)`

`LOGICAL_MAXIMUM (127)`

`REPORT_SIZE (8)`

`REPORT_COUNT (1)`

`INPUT (Data,Var,Rel)`

`USAGE_PAGE (Generic Desktop)`

`USAGE (Y)`

`LOGICAL_MINIMUM (-127)`

`LOGICAL_MAXIMUM (127)`

`REPORT_SIZE (8)`

`REPORT_COUNT (1)`

`INPUT (Data,Var,Rel)`

Mỗi trên vẫn chạy ngon nhưng để tiết kiệp bộ nhớ (vì Report này thường được lưu trên bộ nhớ rất nhỏ mà). Ta có thể rút như như sau mà không thay đổi ý nghĩa:

`USAGE_PAGE (Generic Desktop)`

`USAGE (X)`

`USAGE (Y)`

`LOGICAL_MINIMUM (-127)`

`LOGICAL_MAXIMUM (127)`

`REPORT_SIZE (8)`

`REPORT_COUNT (2)`

`INPUT (Data,Var,Rel)`

Toàn bộ 3 nút và tọa độ X, Y được miêu tả bằng dãy dữ liệu sau:

`USAGE_PAGE (Button)`

`USAGE_MINIMUM (Button 1)`

`USAGE_MAXIMUM (Button 3)`

`LOGICAL_MINIMUM (0)`

`LOGICAL_MAXIMUM (1)`

`REPORT_COUNT (3)`

`REPORT_SIZE (1)`

`INPUT (Data,Var,Abs)`

`REPORT_COUNT (1)`

`REPORT_SIZE (5)`

`INPUT (Cnst,Var,Abs)`

`USAGE_PAGE (Generic Desktop)`

`USAGE (X)`

`USAGE (Y)`

`LOGICAL_MINIMUM (-127)`

`LOGICAL_MAXIMUM (127)`

`REPORT_SIZE (8)`

`REPORT_COUNT (2)`

`INPUT (Data,Var,Rel)`

Đến đây vẫn chưa hết, ta cần thêm một vài trường bao bọc khối này để máy tính hiểu đây là một Report của Mouse.

Nó giống như thế này:

`USAGE_PAGE (Generic Desktop)`

`USAGE (Mouse)`

`COLLECTION (Application)`

`USAGE (Pointer)`

`COLLECTION (Physical)`

`... What we wrote already goes here`

`END COLLECTION`

`END COLLECTION`

Cuối cùng, dãy dưới đây mới thực sự là dữ liệu ta vừa tạo:

`0x05, 0x01,` `// USAGE_PAGE (Generic Desktop)`

`0x09, 0x02,` `// USAGE (Mouse)`

`0xa1, 0x01,` `// COLLECTION (Application)`

`0x09, 0x01,` `//   USAGE (Pointer)`

`0xa1, 0x00,` `//   COLLECTION (Physical)`

`0x05, 0x09,` `//     USAGE_PAGE (Button)`

`0x19, 0x01,` `//     USAGE_MINIMUM (Button 1)`

`0x29, 0x03,` `//     USAGE_MAXIMUM (Button 3)`

`0x15, 0x00,` `//     LOGICAL_MINIMUM (0)`

`0x25, 0x01,` `//     LOGICAL_MAXIMUM (1)`

`0x95, 0x03,` `//     REPORT_COUNT (3)`

`0x75, 0x01,` `//     REPORT_SIZE (1)`

`0x81, 0x02,` `//     INPUT (Data,Var,Abs)`

`0x95, 0x01,` `//     REPORT_COUNT (1)`

`0x75, 0x05,` `//     REPORT_SIZE (5)`

`0x81, 0x03,` `//     INPUT (Cnst,Var,Abs)`

`0x05, 0x01,` `//     USAGE_PAGE (Generic Desktop)`

`0x09, 0x30,` `//     USAGE (X)`

`0x09, 0x31,` `//     USAGE (Y)`

`0x15, 0x81,` `//     LOGICAL_MINIMUM (-127)`

`0x25, 0x7f,` `//     LOGICAL_MAXIMUM (127)`

`0x75, 0x08,` `//     REPORT_SIZE (8)`

`0x95, 0x02,` `//     REPORT_COUNT (2)`

`0x81, 0x06,` `//     INPUT (Data,Var,Rel)`

`0xc0,` `//   END_COLLECTION`

`0xc0` `// END_COLLECTION`

Đến đây, nếu bạn gặp vấn đề về một vài khái niệm, bạn có thể đặt câu hỏi với tác giả link gốc hoặc đặt câu hỏi bằng tiếng Việt với chủ blog để cùng thảo luận.