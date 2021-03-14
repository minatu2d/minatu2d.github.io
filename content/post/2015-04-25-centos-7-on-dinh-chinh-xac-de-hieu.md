---
layout: post
title: CentOS 7 - Ổn định, chính xác, dễ hiểu
date: 2015-04-25 08:19:53.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Khác
tags: []
meta:
  _edit_last: '72243466'
  _publicize_pending: '1'
  _oembed_e5ce12076b2d3fd261592c3bdcbb498b: "{{unknown}}"
  geo_public: '0'
  _oembed_07e800cbfb2d6a575faf102545c9e4fd: "{{unknown}}"
  _oembed_2cadd9c9438fd2633d0d199eff6571cd: "{{unknown}}"
  _oembed_12c6bf413fc86f3f010a3fae910c7af8: "{{unknown}}"
  _oembed_dceef62e7c4d99f7b16679f0c1018739: "{{unknown}}"
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2015/04/25/centos-7-on-dinh-chinh-xac-de-hieu/"
---
Đang chán nản với cái ứng dụng C# vì không thích cái kiểu làm thì theo yêu cầu mà hỏi thì cái gì các sếp cũng không rõ. Thì anh Kusu ra hỏi.

```

Kusu : Mày biết CGI không?

Mình : (Ah ha, có việc khác hay hơn ư) Có, trả lời ngay,

Kusu : Bọn tao cần 1 cái web dùng CGI chạy trên CentOS (Wow, good xúc thôi), trong công ty ít người làm về server với Web lắm. T đang nghĩ, nếu mày biết thì ra làm luôn.

Mình; Ok.

```

Từ mùng 10-30/3/2015:

### 1. Cài đặt bau đầu  


- Cài đặt CentOS hơi khác với Ubuntu mình dùng trước đến giờ.

- Trên cùng cái đĩa DVD đó, nó cho phép nhiều tùy chọn để cài đặt.

tùy chọn ở đây là số lượng gói và tool được cài vào máy, tất nhiên các phần cơ bản liên quan đến mạng sẽ luôn được cài. Một vài tùy chọn như: Desktop Gnome, Desktop KDE, Desktop Xfe... Môi trường phát triển Web..

Mình chọn Minimal vì sử dụng VM Player tạo máy ảo để chạy CentOS + &nbsp;máy chỉ có 4G RAM(tù). Thực ra trước khi quyết định Minimal thì có cài thử cái Gnome lên, nhưng hỡi ôi, chậm trên cả tưởng tượng.

- Với cái minimal, thì rất ít tool được cài, một số thứ sẽ phải cài ngay như:

nettool : ifconfig, networking... cái nọ cái chai.

- Set up local responsibility gì đó, chắc là nó sẽ up cái kho phần mềm từ cái đống package trong DVD luôn (vì DVD có tùy chọn full nên chắc chắn nó sẽ có không ít gói phần mềm trong đó) (Cái này mình không làm. dùng mạng cài luôn).

- Rồi thiếu cái gì cũng google rồi, yum,... nọ kia thôi.

Nhưng thấy 1 cái, hướng dẫn trên net hầu như chính xác đến từ dòng, kết quả chạy hệt như hình mẫu.

### 2. Cài đặt Apache( trên CentOS được gọi là httpd - http "diamond")

- Tất nhiên là yum -y install httpd,

Cài xong, bật service cái chạy luôn, không phải nghĩ.

Thế là xong helloworld trên HTML

### 3. Cấu hình để chạy CGI/Perl (Hello World)

((Mai viết tiếp)). Giờ ngủ đã.

### 4. Cấu hình để phát triển&nbsp;Hello Customers.

### 5. Từ Plain CGI chuyển qua CGI-Application

Lúc mới đầu, thấy có cung cấp 1 cái lớp CGI, cung cấp request, response, các query

Nghĩ bụng, vậy là đủ cho 1 cái web đơn gian rồi, xử lý thì thiếu gì tìm đó.

Việc routing đơn là xử lý string thôi

http://127.0.0.1/webapp/abc.cgi?var1=value1&var2=value2.

Cứ dùng cgi -> query rồi if..else&nbsp; mà xử lý thôi.

Nói chung ứng dụng mà đặc biệt là web, cái nào cũng nên thực hiện cái mô hình MVC.

Tức là càng tách biệt 3 phần Model (việc truy cập data base) + View ( Phần hiển thị cho người dụng) + Control (phần xử lý các nghiệp vụ gì đó dựa trên đầu vào dữ liệu, đầu ra là kết quả hiện thị ra view).

Với CGI plain mỗi controller, ta sẽ dùng 1 lệnh if

```perl
actionValue = cgi -> query("action")
if (action == "action1") {

//<Xử lý 1>

} else if (action == "action2") {

//<Xử lý 2>

}
```

Phần database mình chưa động đến <Để bổ sung sau vậy>

Phần View, làm sao để chuyển cái output là giá trị đến người dùng 1 cách dễ hiểu nhất.

CGI thì không hỗ trợ việc nhúng code, nên ta dùng 1 plugin gọi là HTMLTemplate

template = HTMLTemplate -> Load("abc.tpl")// Miễn là text file thôi, kiểu gì cũng ok.

Ta được đối tượng HTMLTemplate,  
```perl
template -> param("VAR1", realValu1)

template -> param("VAR2", realValu2)
```

các biến VAR1, VAR2 được định nghĩa bên trong trong file template kia.

Chuyển qua CGI Application, hầu hết các xử lý đều không đổi.

Chỉ có phần điều hướng thì sử dụng khái niệm mode của CGIApplication

Nhưng cũng phải nói rằng, các xử lý về lỗi, các xử lý liên quan đến sesion sẽ

dễ dàng hơn rất nhiều khi dùng CGIApplication.