---
layout: post
title: Nếu wireshark trên Windows gặp lỗi, hãy thử dùng dumpcap
date: 2015-07-28 15:04:10.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Khác
tags: []
meta:
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '13173376479'
  _wpcom_is_markdown: '1'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2015/07/28/neu-wireshark-tren-windows-gap-loi-hay-thu-dung-dumpcap/"
---
`Wireshark`, một phần mềm quá phổ biến để phân tích gói tin cho dù là có dây hay không dây, 1 dây hay 2 dây, tất tần tật. Gì em nó cũng làm được.

`Wireshark` vốn đuợc phát triển cho `Solaris` và `Linux`. Và thư viện đồ họa hiện đang sử dụng là `gtk`.  
`Gtk` là vốn là thư viện đựoc phát triển từ dự án GIMP ( GNOME Graphic Toolkit). Và nó được phát triển cho hệ điều hành `Linux`.  
Giống như bao chương trình dạng của Unix khác, ban đầu chúng đều chạy ở chế độ dòng lệnh.  
Do được nhiều người sử dụng nên họ viết GUI cho nó.  
Và cũng giống hầu hết các siêu tool khác như `emacs`, `glade`...etc. Chúng đuợc port sang Window bằng cách sử dụng các bản build cho Window của thư viện gtk và các thư viện liên quan.  
Độ ổn định của gtk và các thư đồ họa nền Linux mà Wireshark đang sử dụng là quá tuyệt vời só với các tool đuợc porting khác rồi.  
Nhưng không phải lúc nào cũng muợt mà. Nó vẫn có thể gặp lỗi liên quan đến mấy thư viện này.  
Phần đa các lỗi liên quan đến một loạt thư viện dll đuợc porting sang Windows từ Linux.  
Mà thường những lỗi này sẽ dấn đến crash chưong trình ngay, khỏi chạy gì nữa.

Trong trường hợp đó, ta có thể tìm kiếm sự giúp đỡ từ cộng đồng người sử dụng và developer của Wireshark, một trong những cộng đồng mã nguồn mở active nhất hiện nay.

Có một phưong pháp vẫn cho ta kết quả capture gói tin mà không phải sử dụng đến các thư viện đồ họa, vốn rất thiếu ổn định ở trên.

Đó là `tshark` và `dumcap`.

Khi đồ họa bị crash hãy nghĩ đến chúng, biết đâu chúng sẽ cho ta kết quả không ngờ. :)))