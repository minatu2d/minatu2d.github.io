---
layout: post
title: "[USB]Các khái niệm về USB"
date: 2015-12-22 16:41:13.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
- Communication
- USB
tags: []
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '18034655339'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2015/12/22/usbcac-khai-niem-ve-usb/"
---
Kết nối USB, chắc chắn ai cũng biết. Mỗi người đểu sở hữu ít nhất một thiết bị có kết nối này.

Tôi cũng đã từng nghĩ nó sẽ khá dễ dàng để hiểu, để lập trình giống như người dùng vẫn hiểu về nó.

Đúng là với người dùng cuối, một thiết bị có kết nối thì khả năng có nó có thể kết với máy tính như smartphone, máy nghe nhạc, bộ sạc.... rất dễ hiểu, dễ gần.

Nhưng gần đây khi được tiếp cận với USB ở dạng driver và firmware cho thiết bị, tôi thấy nó không dễ dàng như tôi tưởng. Nó đủ phực tạp để chúng ta tiếp tục nói về nó.

Trong bài này, tôi sẽ giới thiệu một số khái niệm tôi đọc và rút ra được trong quá trình làm. Nó gồm cả những khái niệm về phần cứng, lẫn phần mềm cho dù về mặt phần cứng mình không được tự tin lắm.

1.  Chúng ta vẫn thấy mỗi máy tính có 2, 3 có khi 4 cổng USB, những tất cả chúng đều nối đến một điểm mà ở đó hệ điều hành điều khiển, quan sát được tất cả các thiết bị cắm vào.
2.  Dù nhiều cổng nhưng cùng nối đến 1 điểm, cái này gọi là Bus, dữ liệu tất cả các cổng đều truyền qua các Bus này, vì thế, khi số thiết bị đang sử dụng đạt đến một số lượng nhất định thì tốc độ sẽ bị chậm.
3.  Chắc chắn chúng ta từng nghe đến USB Hub, ví dụ khi máy chỉ có 2 cổng USB mà ta muốn dùng đến 3 cổng là chuột, bàn phím và 1 cái USB. Khi đó nối một chiếc USB Hub 4 cổng hay gì đó vào bất kì cổng USB nào. Tức là cần 1 đầu vào cho máy tính mà có đến 4 đầu cho các thiết bị bên ngoài. Quá là tiện.
4.  Về mặt lý thuyết, USB Host hỗ trợ đến 127 thiết bị được cắm vào chúng (không tính Hub)
5.  Người ta hay nói đến tốc độ của USB là Low-Speed, Full-Speed, High-Speed. Đó là muốn nói đến tương quan về mặt thời gian cần thiết để truyền một khối dữ liệu nhất đinh. Không mang ra để làm thước đo về mặt hiệu năng sử dụng được. Ta nên hiểu tốc độ đặc trưng cho một lớp thiết bị. Ví dụ như bàn phím, hầu hết chuột thì chỉ cần tốc độ Full là đủ lắm rồi. Có thiết bị truyền tải Video, thì sẽ cần High để đảm bảo độ trễ giới hạn.
6.  Khi một thiết bị USB được cắm vào máy tính hay là Host thì sẽ có 3 qua trình lớn xả ra cho đến khi sử dụng. Đó là quá trình Reset(về mặt điện đóm), quá trình nhận dạng (tức là Host hiểu đây là thiết bị gì, phải thao tác với nó như thế nào), sau đó mới đến trạng thái sử dụng.
7.  Đặc trưng truyền dữ liệu tầng ứng dụng với thiết bị là khi nào Host hỏi, thì thiết bị mới trả lời (kèm theo dữ liệu), Host không hỏi thì thiết bị không gửi gì hết (trừ trường hợp Wake-up).
8.  Nếu thiết bị không thể được nhận dạng bởi Window, ta sẽ thấy xuất hiện một hộp thoại thông báo "Unrecognize device " hiện ra.
9.  Nếu thiết bị hỏng cả mạch rồi, thì đến cái đèn báo (nếu có) cũng sẽ không sáng. 