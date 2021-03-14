---
layout: post
title: Một ứng dụng Web đơn giản sử dụng CGI/Perl sẽ cần những gì?
date: 2015-07-11 13:52:18.000000000 +09:00
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
  _publicize_job_id: '12626400480'
  _wpcom_is_markdown: '1'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2015/07/11/mot-ung-dung-web-don-gian-su-dung-cgiperl-se-can-nhung-gi/"
---
Với những ai đã từng làm nhiều với `CGI` và `Perl` sẽ dễ dàng đưa ra phương án tối ưu nhất.

Còn với những ai mới bắt đầu như mình , hoặc chỉ cần 1 chút thôi chẳng hạn thì có thể dễ bị lạc đường giữa rất nhiều framework hoặc thư viện của perl.

Vừa rồi mình có làm một ứng dụng Web

* Server Side : CGI sử dụng ngôn ngữ perl. (Tức là các file xử lý viết bằng ngôn ngữ Perl, và được server Apache gọi xử lý theo kiểu CGI (sử dụng module mod_perl).
* Phía Client là Javascript và HTML.

Mình là người mới hoàn toàn với `Perl`. Đã từng 1 chút với `PHP`.


## 1. Liên hệ với những gì đã biết
Tuy tìm hiểu thấy mọi người nói là giống nhau. Giống nhau ở đây được hiểu theo nghĩa khi viết 1 xử lý bằng `Perl` và `PHP` sẽ tương đối giống nhau.

Ai cũng biết ta không nên chế tạo lại "chiếc bánh xe", cần chọn công cụ để reuse giúp quá trình thực hiện dự án nhanh và phù hợp nhất với yêu cầu.

Yêu cầu dự án của tôi không cần nhiều, chỉ đơn giản cũng khoảng 20 trang giao diện Web, đọc một số file cấu hình, ghi những gì người dùng phía Client gửi đến ra file. Yêu cầu rất đơn giản, nhưng có 1 điểm, số màn hình sẽ có thể thay đổi trong tương lai mà tôi lại không đủ thời gian để đưa ra một thiết kế from scratch.

Bạn đầu, tôi viết chay bằng `Perl` trong `CGI`, phân tách các http request gửi đến để routing đến xử lý phù hợp. Code ban đầu hoàn toàn là dạng cấu trúc kiểu theo flow xử lý chứ chẳng có tí hướng đối tượng nào.

## 2. Có phao cứu sinh
Rồi tôi tìm thấy `CGI::Application` (đây là cách viết một module trong Perl), nó cung cấp 1 cách khá hệ thống các thành phần để phát triển một ứng dụng Web sử dụng CGI/Perl. Nhưng xử lý liên quan đến Session, xuất ra view, routing, phương thức debug, các xử lý lỗi đã được module này cung cấp.

Việc cần làm là sử dụng các thành phần đó cho ứng dụng của mình.

Thông thương một ứng dụng web bất kì nào cũng có 3 thành phần theo mô hình `MVC`. Cái này là kinh điển rồi.

- **M** : (Model) là nói dữ liệu có thể là file, nhưng thường là database.

- **V** : (View) là nói đến phần show ra cho người dùng cuối.(người ta hay gọi là giao diện đó)

- **C** : (Control) là nói đến phần xử lý logic từ yêu cầu của người dùng và data để đưa ra cái view theo mong muốn của người dùng.

Trong cái ứng dụng `CGI/Perl` của tôi, do `CGI::Application` đã cung cấp phần khung bao gồm:

1.  Một cơ chế đọc các request và routing đến xử lý mong muốn, cái này giúp việc đưa request đến Control phù hợp.
19.  Một cơ chế xuất view từ temple (cấu trúc giống HTML + có nhúng các phần có thể thay đổi) sử dụng module HTML::Template
20.  Cơ chế sử dụng session.  
    Session sử dụng có quản lý của CGI::Application, nó dựa trên CGI::Session nhưng được sử dụng một cách thông nhất bên trong CGI::Application.

Do yêu cầu dự án khá đơn giản, những việc cần làm của tôi chỉ là học cách sử dụng `HTML::Template`, thêm các xử lý file.