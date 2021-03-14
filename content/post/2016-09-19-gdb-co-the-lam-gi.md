---
layout: post
title: GDB có thể làm gì? (01 - Khả năng của GDB)
date: 2016-09-19 11:58:27.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
- Debug
tags:
- GDB
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: '43452'
  _publicize_job_id: '26981475434'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/09/19/gdb-co-the-lam-gi/"
---
Trước khi nói về chủ đề chính là "GDB có thể làm gì". GDB hay những phần mềm như GDB được viết ra để giải quyết vấn đề gì.

## 1. Phầm mềm Debugger sinh ra giải quyết cái gì?

Phần mềm thực sự được tạo ra ở bước implementation, nó hiện thực những nội dung được mô tả trong thiết kế.  
Vì con người viết code tạo ra phần mềm, mà con người không phải lúc nào cũng luôn làm đúng như những gì họ đã nghĩ, đã ý định, đã thiết kế. Những cái đó người ta gọi là **BUG**.

Một điều nữa, khi đã biên dịch (ở đây không xét đến ngôn ngữ thông dịch nhé), các code mà con người viết ra sẽ trở thành mã nhị phân.

Khi phần mềm không chạy đúng như mong muốn, thì làm thế nào. Cũng giống như giải quyết các vấn đề khác trong thực tế, người ta quan sát hiện tượng một hoặc vài lần để phán đoán nguyên nhân, sau đó đưa ra cách giải quyết. Lấy ví dụ việc cho nước khi nấu cơm chẳng hạn, 1 vạch nước thì cứng, 4 vạch lại nhão, thử 2, 3 thì lại ngon. Với phần mềm cũng tương tự như thế, vì không thể sửa trực tiếp trên mã nhị phân (0101010..01) được, người ta có thể tái hiện lại hiện tượng nhiều lần rồi phán đoán, tìm ra đoạn code gây lỗi rồi sửa, sau đó thử. Cứ thế, đến khi đúng thì thôi.

Với cách đó thì không thể giải quyết được những **BUG** khó quan sát, **BUG** phức tạp được. **DEBUGGER** được sinh ra để trợ giúp việc này.

Có người còn nói "Quá trình debug là quá trình quay chậm lại BUG đã xảy ra", để thấy rõ nguyên nhân.

## 2. GDB có thể làm gì.

GDB, là viết tắt của GNU Debugger. Ta sẽ tham chiếu đến các chức năng mà một Debugger (là phần mềm) có, được định nghĩa ở [wikipedia(https://en.wikipedia.org/wiki/Debugger)  
- **một công cụ để lấy thông tin (a query processor)** các thông tin thường lấy như trạng thái các thanh ghi, địa chỉ các ô nhớ mà CPU đang xử lý ở 1 thời điểm nào đó.  
- **một công cụ để ánh xạ symbol (a symbol resolver)** ánh xạ trạng thái tại 1 điểm đang chạy đến vị trí trên source code tương ứng (file nào, dòng nào). Đây là chức năng được dùng phổ biến nhất, giúp developer dễ dàng xác định đoạn source code chưa đúng.  
- **một trình thông dịch biểu thức (a expression interpreter)** chuyển các biểu thức do developer viết ra thành điều kiện hay action tương ứng tác động lên chương trình đang chạy  
- **một giao diện giao tiếp cung cấp cho ứng dụng phía trên** (a debug support interface)  
- **quay chậm xử lý (step by step hoặc program animation)**  
- **dừng có điều kiện** thông qua các breakpoint.  
- **sửa rồi chạy tiếp** Sửa thông trạng thái chương trình như thay đổi giá trị các biến, các thanh ghi rồi cho phép chạy tiếp với trạng thái mới (một số trường hợp giúp nhảy qua các đoạn lỗi hoặc chưa ổn định)

## 3. Giao diện sử dụng GDB

Giống như bao chương trình GNU khác. Giao diện dòng lệnh là mặc định.  
Nhưng bên cạnh đó, có rất nhiều IDE hỗ trợ sử dụng GDB. Tức là thay vì ta phải thực hiện từng command trên dòng lệnh của GDB cung cấp. Ta có thể sử dụng IDE để làm việc này nhanh hơn.  
Chỉ tính riêng cho ngôn ngữ C/C++, các IDE phổ biến trên Linux đều hỗ trợ việc này như: Eclipe, Netbean, QtCreator, Anjuta, Codeblock, CodeLite, KDevelop.

Ta sẽ xem qua từng ví dụ cụ thể ở các bài sau.