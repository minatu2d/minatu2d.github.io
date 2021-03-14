---
layout: post
title: SCSI - Giao tiếp với USB Memory
date: 2016-03-15 14:54:41.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
- Communication
tags:
- SCSI
- USB
- USBMemory
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: '43452'
  _publicize_job_id: '20781352948'
  _oembed_ce5bc9ca30e8028766fc92e5d1bee89a: "{{unknown}}"
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/03/15/scsi-giao-tiep-voi-usb-memory/"
---
![8GB-USB-Flash-Drive-Rotation-2-0-USB-DISK-Model-Pen-drive-memory-stick-8G-HOT.jpg](/post/images/8gb-usb-flash-drive-rotation-2-0-usb-disk-model-pen-drive-memory-stick-8g-hot.jpg)

Gần đây, do phải tìm hiểu khả năng porting USB Memory Driver sang NORTi nên đã có dịp tìm hiểu và tự confirm trên code một số điều liên quan đến thiết bị nhớ USB (hay ta vẫn gọi là USB Flash Memory). "USB Flash Memory" bao gồm USB: là tên giao diện cả mềm, cứng; Flash : là chất liệu của chip nhớ, Memory : là chỉ thiết bị nhớ nói chung.

Ở một số bài trước, tôi cũng viết về một số điểm chính của giao tiếp USB.

Về cơ bản, phần mềm mà programer có thể viết cho USB hầu như được xác định từ sau bước tín hiệu. Tức là phía ứng dụng mức thấp (tức driver ấy) sẽ sử dụng các thanh và các ngắt được cung cấp để setup, nhận dữ liệu, gửi dữ liêu, phát hiện kết nối.

Về phía người dùng ta sẽ thấy có một vài loại thiết bị sử dụng USB như chuột, bàn phím, thiết bị nhớ (ta vẫn gọi là USB).

Bài này sẽ nói 1 chút về thiết bị nhớ sử dụng giao tiếp USB hay USB Memory.

## 1. Quá trình Configure

Là quá trình lúc cắm vào đến khi phần mềm phía Host hiểu đây là một thiết bị nhớ.

Quá trình này chỉ xác định đây là một thiết bị Mass Storage Class, có thể lưu trữ được.

Mà chỉ xác định rằng đây là 1 thiết bị nhớ, bằng cách nào đó có thể truy cập (đọc, ghi) theo địa chỉ hoặc Block.

Trong quá trình Configure này, controller phía USB Memory sẽ giao tiếp với phần mềm phía Host. Nó sẽ gửi các thông số cần thiết để Host nhận ra nó và đưa ra cách giao tiếp phù hợp.

Một trong những thông số quan trọng nhất là tập lệnh giao tiếp với Controller trên USB Memory.

<Sẽ có bảng ở đây>

## 2. Các truyền dữ liệu nào được sử dụng

Như đã nói ở trên, hầu hết các USB hiện nay sẽ hỗ trợ giao tiếp bằng tập lệnh SCSI.

Nghĩa là như thế nào, thông qua đường Bus của phần cứng, phía Host sẽ gửi các lệnh trong tập lệnh SCSI đến Controller trên USB Memory để lấy thông tin, đọc, ghi...

Nhiệm vụ của Driver phía Host là sử dụng các lệnh này đẻ truy cập đến đúng vùng nhớ.

Để ứng dụng phía trên dễ dàng sử dụng USB, người ta thường cài đặt trên USB Memory một hệ thống File. Thông thường là FAT16, FAT32..

## 3. Làm thế nào để truy cập đến thiết bị nhớ.

Controller phía USB Memory sẽ cho phép phía Driver truy cập đến từng đơn vị nhớ Flash.

Ta cũng biết rằng, bản thân Flash là loại bộ nhớ mà mọi thao tác ghi lại phải kèm với một thao tác xóa dữ liệu trước đó. Đơn vị để đọc, ghi, xóa được quy định trên mỗi loại chip nhớ, thông thường USB Memory sẽ sử dụng đơn vị là 512 byte. Nhiều loại chip nhớ có dung lượng nhỏ sẽ sử dụng đơn vị là 256 Byte, 4Kilo Byte, 64Kbyte.

Phía Driver (nói về phần mềm) sẽ gửi các yêu cầu cho phía Controller của USB Memory thông qua các Commands. Các command này phải được cả phía Controller của USB Memory hiểu thì mới thực hiện đúng được. Thông thường người ta sẽ dựa trên một tập commands được quy định rõ ràng.

Thông qua giá trị _bInterfaceSubClass_ mà phía Controller gửi lên cho Driver lúc 2 trao đổi để nhận ra nhau. Thì phía Driver sẽ biết phía Controller hỗ trợ tập Command nào.

Tất cả các giá trị có thể được miêu tả trong trang 11 của tài liệu mô tả về Mass Storage Class. [Ở đây](http://www.usb.org/developers/docs/devclass_docs/Mass_Storage_Specification_Overview_v1.4_2-19-2010.pdf)

![Screenshot from 2016-03-19 21:41:08](/post/images/screenshot-from-2016-03-19-214108.png)

Tập command được hỗ trợ phổ biến nhất hiện tại là SCSI Transparent command set.

Tức giá trị _bInterfaceSubClass _sẽ là _0x06_

Tập command trên khá nhiều nhưng phổ có 3 command phổ được sử dụng phổ biến là

INQUIRY : Đọc những thông số cơ bản của USB Memory.

READ(10) : Là lệnh đọc dữ liệu từ USB Memory. Có nhiều câu lệnh READ với những tham số khác nhau, vì câu lệnh này có độ dài 10 byte nên thường được gọi là READ10.

WRITE : Ghi dữ liệu vào USB Memory.

*Ghi chú: SCSI sử dụng trong bài viết là nói về một tập các định nghĩa câu lệnh (về mặt phần mềm), ngoài ra SCSI cũng được biết đến là một giao diện phần cứng phổ biến trước đây. Thực ra SCSI Command Set ban đầu được sử dụng trong giao tiếp SCSI (phần cứng), nhưng sau này phần cứng bị tuy đã không còn sử dụng nhưng những câu lệnh giao tiếp vẫn được sử dụng trong một giao diện phần cứng khác. Điển hình ta thấy ở đây là USB Memory.

Thế mới biết, phần cứng có tuổi thọ ngắn hơn phần mềm rất nhiều.