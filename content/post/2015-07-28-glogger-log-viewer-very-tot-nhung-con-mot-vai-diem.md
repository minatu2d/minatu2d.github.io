---
layout: post
title: Glogger - Log viewer tốt nhưng còn một vài điểm
date: 2015-07-28 15:21:24.000000000 +09:00
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
  _publicize_job_id: '13173861526'
  _wpcom_is_markdown: '1'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2015/07/28/glogger-log-viewer-very-tot-nhung-con-mot-vai-diem/"
---
Nói đến editor, ai cũng nghĩ đến Notepad++. Sakura (JP thôi), hay gì gì đó.  
Ừ thì đúng Notepad++ rất nhiều tính năng, rất nhanh, rất nhẹ.

Nhưng gần đây tôi phát hiện ra điểm yếu của nó, xử lý file lớn của nó rất tệ. Thậm chí nó còn gây mất dữ liệu.  
Tôi và bạn tôi đã thử với một file khoảng 600MB trở lên thì khó có thể thực hiện một phép copy & paste nào nữa.  
Nếu có thể thực hiện được đi nữa thì rất dễ gây sai dữ liệu hoặc mất dữ liệu.

Thực ra ngoài với những file đó tôi cũng chẳng làm gì nhiều, chỉ copy paste một vài đoạn thôi. chứ chưa edit sử dụng các chức khác. Mục đích của tôi lần đó là để xem thôi chứ edit sửa lại thì rất ít. Bạn tôi "thích" và tin tưởng Notepad++ cực kì, giống như tôi tin tưởng emacs hay vi vậy. Em nó luôn đúng và chính xác. Tôi có khuyên cậu ấy nếu chỉ dùng view log hay dùng glogger,  
tool như Notepad++ thích hợp cho việc edit thôi và với file dung lượng nhỏ thì tốt. Lớn chút là giật đùng đùng.

Tôi muốn nói đến cái tool đẻ view log rất mạnh tôi đang dùng. Đó là glogger, và là open source sữ dụng Qt.

Nhưng điểm mạnh của em nó.  

- 1. Giao diện view log trực quan.  
- 2. Sử dụng tab, giúp mở nhiều file.  
- 3. Có chức nằng tìm kiếm, hỗ trợ cả regex thì phải.  
- 4. Đọc file lớn cực tốt, và usability tìm kiếm, mở file lớn rất thân thiện.

Tuy nhiên hôm nay tôi thấy em nó có 2 điểm yếu, tôi thấy nó trên bản Window, bảng Linux tôi chưa thử:

- 1.  Mở file log nằm trên đĩa cứng hoặc ổ đĩa (network) thì ngon lành cành đào. Nhưng mở file qua network trên window thì bó tay.
- 2.  Mở file mà đường dẫn có chưa kí tự tiếng Nhật là chịu luôn.

tôi nghĩ 2 chức năng này không hẳn quá khó để add, tôi cứ note tạm ở đây.  
Có thời gian sẽ nghịch.