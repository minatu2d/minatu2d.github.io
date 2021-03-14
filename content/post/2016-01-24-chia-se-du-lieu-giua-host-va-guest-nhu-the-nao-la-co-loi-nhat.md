---
layout: post
title: Chia sẻ dữ liệu giữa Host và Guest như thế nào là có lợi nhất?
date: 2016-01-24 13:14:00.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Khác
tags: []
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: '43452'
  _publicize_job_id: '19072027706'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/01/24/chia-se-du-lieu-giua-host-va-guest-nhu-the-nao-la-co-loi-nhat/"
---
Ngày nay, dù chỉ cần có 1 mày tính thì người ta vẫn có thể sử dụng nhiều môi trường hệ điều hành khác nhau. Cái đó gọi là ảo hóa.

Tức là trên một máy tính chạy một hệ điều hành cụ thể. Ta cài đặt một ứng dụng mô phỏng một máy tính trên đó. Ta sẽ có bao nhiều máy tính tùy vào khả năng phần cứng của máy thôi.

2 phần mềm được sử dụng free nhiều nhất hiện tại là VirtualBox và VMWare Player.

Khi sử dụng những phần mềm này, ta có thể tạo ra nhiều môi trường hệ điều hành khác nhau tùy thuộc vào nhu cầu. Việc chuyển qua, chuyển lại giữa các môi trường đơn giản là chuyển qua giữa các ứng dụng.

Trong quá trình phát triển phần mềm, khi đã sử dụng máy ảo, hay môi trường ảo.

Thì điều đầu tiên người ta nghĩ đến là việc chia sẻ dữ liệu với môi trường ảo đó như thế nào??? Bởi vì không đẩy dữ liệu vào hoặc lấy dữ liệu ra từ máy ảo thì việc tạo ra cái môi trường ảo này hầu hình chẳng có ích gì.

Một cách rất hay được sử dụng là chia sẻ giữa 2 máy này bằng internet.

Vì máy ảo cũng nối mạng, và máy thật cũng nối mạng nên cách đơn giản nhất là đẩy lên internet, 2 bên cùng truy cập.

Cách này có một điểm dở là tốc độ bị cản trở bởi internet. Dù hữu ích thật nhưng, 2 chiếc máy quá gần nhau, sao phải đẩy dữ liệu đến một nơi xa tít rồi lại gọi về.

Cách này hoàn toàn không khả thi với các dữ liệu lớn tới hàng GB.

Một khó khăn nữa trong việc chia sẻ dữ liệu giữa Host-Guest (Host là máy mà chạy chương trình tạo máy ảo,　Guest:　là cái máy được tạo ra bởi phần mềm chạy trên máy Host) đó là việc tương thích môi trường.

Nếu Host là Window, Guest cũng là Window. 2 máy Window trao đổi qua mạng Host-Guest rất đơn giản. Ta mapping tài nguyên của máy Guest thành 1 ổ đĩa của máy Host. Sau đó việc chia sẻ dữ liệu rất dễ dàng, chỉ đơn giản là copy & paste. Hơn thế nữa, việc mapping thành một thư mục giúp các ứng dụng chạy trên máy Host hoạt động trơn tru không hề thấy sự khác biệt nào vì ở đĩa thật với ổ đĩa ảo có khác gì nhau đâu, chỉ là cái tên truy cập thôi. Đặc điểm này rất quan trọng trong một số ứng dụng chuyên dụng.

Có 2 cách để làm việc này.

1.  Sử dụng tính nắng Share Folder có sẵn của VMWare hoặc VirtualBox.
2.  Cài đặt một server SMB (sử dụng giao thức Directory của Window), rồi từ máy Host Access đến SMB Server đó để truy cập các tài nguyên mà Server này quản lý trong máy Guest.