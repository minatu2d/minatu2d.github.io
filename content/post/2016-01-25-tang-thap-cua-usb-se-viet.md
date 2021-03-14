---
layout: post
title: Tầng thấp của USB
date: 2016-01-25 14:17:00.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
- Communication
tags:
- USB
meta:
  _wpcom_is_markdown: '1'
  _publicize_job_id: '19106631500'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/01/25/tang-thap-cua-usb-se-viet/"
---
USB - Khỏi cần nói thì nó cũng đã quá nổi tiếng về sự phổ biến rồi. Gần như mọi thứ đều mặc định phải có kết nối USB, cổng USB gần như là bắt buộc trên máy tính, và rất nhiều thiết bị điện tử ta thấy. Từ USB1.1 đến 2.0, rồi gần đây nhất là 3.0. Rồi gần đây người ta có nhắc nhiều đến USB Type C.  
Tất cả những mĩ từ đó dùng để PR tính năng của sản phẩm đến người dùng. Tất nhiên blog này tôi không muốn nói về những thứ đó. Tôi muốn về những thứ tôi nghĩ là cần phải biết để lập trình được với USB. Đó là phần mềm chạy trên thiết bị của 2 đầu dây USB.

Bài đầu tiên này sẽ nói về tầng giao tiếp gần phần cứng nhất. Vì thế tôi muốn đặt tên nó là Tầng tháp của USB. Thực ra nó cũng không thấp lắm vì tầng tín hiệu điện tử mới thực là thấp.  
Nhưng kết nối USB có một điểm khác biệt với các kết nối Serial trước đó. Trong một số kết nối Serial hay Parallel trước USB. Người lập trình hoàn toán có thể tác động lên từng Pin của kết nối để thực hiện việc giao tiếp với thiết bị.  
USB không như thế. Người ta thường tích hợp sãn những module phần cứng ở cả 2 đầu của sợi dây. Vì thế việc trao đổi dữ liệu ở mức tín hiệu số đã được làm sẵn rồi. Việc phần mềm 2 đầu sợi dây phải làm là : thiếp lập thông số cho các phần cứng đó, giao nhận, gửi dữ liệu đi, đón nhận các thông báo từ cứng thông qua các ngắt và thanh ghi của phần cứng.

Ta cũng nên mô tả bố cục của bài tóm lược để tránh lan man

1.  Cấu tạo phần cứng, các loại kết nối
2.  Phía thiết bị
3.  Phía Host
4.  Các trạng thái kết nối
5.  Phải hỏi mới được trả lời
6.  Tốc độ