---
layout: post
title: Một chút về RGB VGA, Digital RGB
date: 2016-02-03 15:31:36.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
tags:
- DigitalSignal
- RGB
- VGA
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: '43452'
  _publicize_job_id: '19425798237'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/02/03/mot-chut-ve-rgb-vga-digital-rgb/"
---
MCU phải giao tiếp với chip ADV7401, tìm mãi tài liệu tiếng Việt mà không tìm được (chắc do tìm kém). Mất hơn 1 hôm mới hiểu được cơ bản chức năng của nó làm gì.

Mô tả ngắn gọn ở đây để sau đỡ quên vậy:

ADV7401 : Chip chuyển đổi tín hiệu truyền hình Analog (PAL, NTSC...) sang dạng số, hoặc số hóa tín hiệu RGB/VGA về dạng Digital RGB.

Về chức năng, mình đang có một tín hiệu VGA, nhưng do không hiểu các khái niệm về đầu ra, đầu vào của chip này nên cũng mất một thời gian mới rõ được.

Bài này sẽ note lại một số keyword xuất hiện trong Datasheet của ADV7401.

1.  Video Decoding là quá trình làm gì.
2.  Graphic Digitization là quá trình làm gì.
3.  720p, 1080i nói lên điều gì.
4.  VGA, SXGA nói lên điều gì.
5.  Một số keyword khác, có tìm hiểu nhưng không liên quan.

* * *

### 1. Video Decoding là quá trình gì.

Video Decoding, ghe thôi đã biết là nặng nề rồi. Mình vẫn nghe thấy phải card hình xịn xem video mới mượt, hoặc đại loại như thế. Các có liên quan gì đến card xịn, liên quan đến xử lý hình ảnh đây.

Vì ta đang tìm hiểu cho việc lập trình, mà lập trình phải I/O. Rốt cuộc, cái Video Decoding nó nhận đầu vào là cái gì, và đầu ra của nó là cái gì???

Ta thấy ngay chữ Decoding, thì chắc là giải mã cái gì rồi. Đó là Video. Cái ta thấy trên màn hình. Nhưng ở Video ở đây muốn nói đến Analog Video hay Video sử dụng mà dạng trước khi lên màn hình của nó là tín hiệu Analog (tức là tần số, các phrase, cường độ..). Video hay Tivi sử dụng Analog ra đời trước khi Tivi tín hiệu số phổ biến. Tivi tín hiệu Analog vẫn được sử dụng đến ngay nay, tuy nhiên trong các thiết bị hiển thị trong máy tính thì hầu hết là Video số. Video số có nghĩa là nó sẽ chưa dữ liệu về màu sắc của mỗi điểm ảnh trên màn hình. Tín hiệu Analog thì không như thế, nó tái tạo các thông số về tần số, cường độ tại điểm nhận thôi. (không biết có đúng ko nữa!!!)

Video Decoding ở đây là quá trình lấy tín hiệu Video Analog làm đầu vào và xuất ra đầu ra Video Digital (

### 2. Graphic Digitization là quá trình làm gì.

Graphic nghĩa là liên quan đến điểm ảnh liên quan đến RGB rồi. Còn Digitization là số hóa. Có nghĩa là số hóa cái gì đó để tạo ra dữ liệu dạng số hay bảng giá trị RGB của mỗi pixel.

Như ta đã biết, máy tính (máy tính Case và một số laptop) thường có một cổng ghi VGA (thuật ngữ này sẽ thảo luận sau) có 15 lỗ, nối với màn hình để hiển thị nội dung cho ta.

Tín hiệu ở máy tính xử lý là RGB (bảng mã màu cho mỗi Pixel), còn tín hiệu truyền qua dây đương nhiên là dạng Analog rồi. Nếu coi case máy tính là bên gửi, màn hình là bên nhận. Thì bảng RGB được truyền qua dây bằng tín hiệu Analog đến màn hình, rồi được Controller của màn hình tái tạo rồi hiển thị cho ta thấy kết quá.

Graphics Digitization là quá trình mà Controller của màn hình sẽ làm, nó nhận đầu vào là Analog (chú ý đây là Analog của tín hiệu số phía máy tính gửi sang) rồi xuất ra đầu ra là một bảng RGB cho ra màn hình.

### 3. 720p, 1080i là gì.

p : progressive scanning hay noninterlaced scanning

i : interlaced scanning

Hình ảnh được hiển thị lên màn hình thông qua các frame (mỗi frame thông thường là trạng thái các điểm trên một đường ngang), việc update trạng thái màn hình chính là việc lặp đi lặp lại cập nhật trạng thái các đường này từ trên xuống dưới. Tần số cập nhật sẽ đủ lớn để mắt không nhận ra là có sự thay đổi cục bộ một phần của màn hình. Cái này gọi là scanning (quét màn hình).

Có 2 cách để quét màn hình, là progressive và interlace,

Progressive nghĩa là các đường từ trên xuống lần lượt được cập nhật trong 1 lần scan.

Interlaced nghĩa là các đường lẻ được cập nhật hết rồi đến các đường chẳn trong 1 lần scan. Vì thế nó cần 2 lần scan để cập nhật hết toàn bộ màn hình.

Progressive sẽ cập nhật toàn bộ các line cho mỗi lần scan nên phải có tần số cao hơn Interlace, mà cụ thể bằng 1/2.

720p ở đây là muốn nói đến của 720 đường ngang được cập nhật bằng progressive.

1080i ở đây là nói đến có 1080 đường ngang được cập nhật bằng Interlaced.

### 4. VGA, SVGA là gì.

Là tên đại diện cho độ phân giải màn hình,

![800px-vector_video_standards2-svg](/post/images/800px-vector_video_standards2-svg.png)