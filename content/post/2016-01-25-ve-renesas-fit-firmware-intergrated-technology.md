---
layout: post
title: Về Renesas FIT - Firmware Intergrated Technology
date: 2016-01-25 14:31:47.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Embedded
- Firmware
- Microcontroller
- RTOS
tags:
- FIT
- Renesas
meta:
  _wpcom_is_markdown: '1'
  _publicize_job_id: '19107042030'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/01/25/ve-renesas-fit-firmware-intergrated-technology/"
---
Kinh nghiệm lập trình với Microcontroller chưa nhiều, chỉ mới 3 năm không liên tục. Nhưng cũng thấy phần nào được một ít gọi là cái hay của một công nghệ (thực ra cứ gọi là công nghệ thôi, chứ nhiều lúc công nghệ với kĩ thuật cũng khá gần nhau). Đó là FIT  viết tắt của Firmware Intergration Technology (FIT) của Renesas.

Theo những gì Renesas cung cấp, công nghệ này được sử dụng trong các dòng Chip RX từ RX63M, RX64M...RX113).

Trong bài sẽ đi giới thiệu khái quát về FIT.

1.  FIT là gì?
2.  Đặt trưng của FIT
3.  Môi trường phát triển cho FIT
4.  Các thành phần phần mềm sử dụng trong FIT

* * *

## 1. FIT là gì?

FIT - Firmware Intergration Technology (đọc là FIT) là một khái niệm hoàn toàn mới để nhấn mạnh vào việc đơn giản hóa việc "ráp" (embedding) các driver thiết bị ngoại vi, mô đun chức năng của phần mềm, vì thế, nó nâng cao khả năng chuyển đổi môi trường chạy "portability" giữa các dòng Microcontroller RX, nhằm mục đích giảm nhẹ quá trình phát triển ứng dụng, quản lý các tài nguyên phần mềm khi sử dụng dòng RX trong phát triển ứng dụng.

## 2. Đặt trưng của FIT

FIT sẽ định nghĩa một cách rò ràng như dưới đây cho mỗi source code ví dụ ( cả middleware và drivers) liên quan đến thiết bị thuộc dòng RX:

*   Các thiết lập khởi tạo cho MCU
*   Làm thế nào định nghĩa một mạch đích
*   Các file cấu hình
*   Tên của các hàm
*   Giao diện với ứng dụng

Đây là những định nghĩa thiết yếu để phát triển ứng dụng trên bất cứ Microcontroller nào. Nếu theo cách thông thường, ta sẽ có rất nhiều source code khác nhau, và cần phải thay đổi sample khi "ráp" chúng với ứng dụng. Với FIT, vì đã định nghĩa những rule cho những thông tin này, nên mỗi source sample có thể được "ráp" với ứng dụng một cách dễ dàng. Theo cách đó, mỗi driver thiết bị ngoại vi, middeware support FIT sẽ có những giao diện đồng nhất với phía ứng dụng. Điều này sẽ rất có lợi khi porting giữa các RX Microcontroller.

## 3. Môi trường phát triển cho FIT

E2Studio là một IDE dựa trên mã nguồn mở Eclipse, có một giao diện thân thiện kiểu WYSIWYG giúp việc "ráp" các module FIT một cách dễ dàng. FIT modules cũng được hỗ trợ bở CS+, một IDE do Renesas tự phát triển.

## 4. Các thành phần phần mềm sử dụng trong FIT

FIT bao gồm 1 gói hỗ trợ Board chạy MCU (Board Support Package - BSP), các module chức năng và driver thiết bị ngoại vi, module middleware và module dành cho giao diện với ứng dụng.

*   BSP : thực hiện việc thiết lập các thông số khởi tạo cho MCU, thiết lập Clock, thiết lập Board.
*   Các module chức năng và driver cho thiết bị ngoại vi: là những driver, cái điều khiển các chức năng dành cho thiết bị ngoại vi của RX Microcontroller.
*   Middleware Modules: Là những phần mềm phổ biến nằm giữa driver và ứng dụng, ví dụ như TCP/IP Stack và FileSystem.
*   Module giao diện với ứng dụng: là giao diện (API) mà ứng dụng sẽ gọi để thực hiện các chức năng mong muốn, ví dụ như giao diện theo chuẩn POSIX API chẳng hạn.

![fit1](/post/images/fit1.jpg)

Dưới đây là một so sánh giữa FIT và non-FIT:

![fit01](/post/images/fit01.jpg)

## 5. Việc phát triển

Khi sử dụng các dòng chíp hỗ trợ FIT, developer có thể tải rất nhiều source code cho mỗi thiết bị trên trang chủ của Renesas.

Source code này được chia ra làm các module, gọi là các FIT Module.

Mỗi module thường bao gồm 3 phần:

*   Tài liệu mô tả (ví dụ SPI, I2C...). Trong tài liệu mô tả sẽ có hướng dẫn chi tiết làm thế nào để add module đó vào Project chưa source code của ứng dụng (Cho IDE E2Studio và CS+), các API cung cấp cho ứng dụng, các options để compile thành công. Mọi thứ đều được giải thích chi tiết
*   Một thư mục source code, sẽ có 2 thư mục. Thư mục thứ nhất chứa source về các xử lý chính của chức năng đó. Thư mục thứ 2 là file config. File này được sử dụng đẻ cấu hình module tương ứng.

_*Tương quan :_ Công nghệ FIT này có tư tưởng khá giống với mô hình sử dụng trong OpenEmbedded. Khác là OpenEmbedded giúp lắp ráp các thành phần phần mềm để tạo ra một bản build như mong muốn. còn FIT tạo ra một Firmware. FIT định nghĩa clear từ đâu, còn BitBake thì sinh ra cho việc tương thích những thứ có sẵn.

Hmm, thực ra cũng khá giống nhau đấy chứ nhỉ.