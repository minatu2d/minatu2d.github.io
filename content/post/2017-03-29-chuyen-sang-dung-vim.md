---
layout: post
title: Chuyển sang dùng VI(M)
date: 2017-03-29 13:37:36.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Khác
tags:
- Linux
- Vim
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '3404588707'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2017/03/29/chuyen-sang-dung-vim/"
---
Chuyển sang dùng Vi
-------------------

Vi - Editor khá nhiều tuổi, có lẽ còn nhiều tuổi hơn của mình.  
Là editor phổ biến nhất trên hệ thống dòng lệnh Linux, Unix hoặc tương tự.  
Có Linux, bạn gần như sẽ có thể dùng Vi. Mà Linux thì có ở rất rất nhiều nơi.  
Có phải vì nó mặc định nên nó phổ biến???  
Mình từng nghĩ vậy hoặc nghĩ chắc nó nhẹ nên người ta cài sẵn nó thôi  
chứ chức năng hoặc độ tiện dụng chắc tệ lắm.  
Vì thực ra, ở trường mình từng dùng 1, 2 lần thôi. Vì khi mới bắt đầu biết đến  
Linux, thì nó cũng có giao diện đồ họa khá tiện rồi. Thầy giáo lại là một  
emacs-fan nữa nên ít khi phải dùng Vi.  
Vi thường xuất hiện trong những câu chuyện chém gió về những pro chưa từng gặp,  
những hacker kiệt xuất, etc...Nào là pro toàn dùng Vim thôi, hay bọn hacker chắc  
dùng Vim kinh lắm...  
Đến gần đây, khi càng ngày càng muốn theo Embedded Linux, mình vẫn chưa sử dụng  
Vi bao giờ vì đơn giản, mọi thứ mình đang dùng (Notepadplusplus, Eclipse) khá ổn.  
Thế nhưng, từ giờ chắc phải suy nghĩ về việc học sử dụng nghiêm túc em Vim này.  
Nó đến từ việc mình bị bắt phải dùng khi bắt đầu việc mới.  
Thực ra mình cũng không ngại đâu, sẽ nhớ được thôi. Bí quá thì copy ra ngoài notepad++  
rồi lại copy vào.:)))  
Nhưng không, mình đã lầm, Vim có nhiều thứ hơn mình nghĩ.  
Mới đang ở giai đoạn bắt đầu sử dụng Vim như là editor chính, cả công việc lẫn ở nhà.  
Nên cứ note tạm mấy cái mình thấy hữu ích và có thể xem đi xem lại thôi.

1. Chỉ gõ khi phải gõ
----------------------

*   Vim có nhiều mode, nhưng có thể chia ra làm 2 mode phổ biến: Command mode và Edit mode
*   Ở góc dưới trái của màn hình luôn hiển thị trạng thái hiện tại là mode nào.
*   Command Mode: Nghĩa là các phím sẽ mang một ý nghĩa thực thi lệnh nào đó trên đơn vị là  
    kí tự, từ, hàng, hoặc liên quan đến chuyển mode. Ví dụ: xóa ở vị trí con trỏ "x", thêm dòng  
    mới là "o", ghi chuyển bằng "h", "j", "k", "l" chẳng hạn.
*   Edit mode: Tức là mode bạn có thể gõ như các editor thông thường khác.  
    Thông thường khi mới chuyển sang, chúng ta thường chọn mode Edit luôn vì nó gần với hầu hết  
    các Editor khác.  
    Tuy nhiên, các đúng nên là: Chỉ thực sự cần chuyển Edit mode khi cần thôi, sau đó nên  
    trở về Command Mode

2. Các thao tác di chuyển
--------------------------

**hjkl : Trái dưới trên phải**:  
- Thay vì sử dụng các phím mũi tên:  
Trong VIM việc di chuyển con trỏ được thực hiện bởi 4 phím "HJKL" trong Command Mode  
- Tức là khi định di chuyển, ta nhớ nhấn Esc rồi sử dụng 4 phím kia.  
- Cái này rất dễ quen thôi.

**0-$ tương đương Home - End**  
Di chuyển vể đầu, cuối dòng

**Ctrl-u, Ctrl-d : Page Up - Page Down**  
Chuyển trang sau hoặc trang trước

**1-G(shift-g), 0-G(shift-g) : Đầu file, cuối file**  
G (Shift-G) là lệnh nhảy dòng  
1-G nhảy về dòng 1, tức là đầu file  
0-G vì không có dòng 0, nên cái này được quy định nhảy về cuối file

**f-chữ cái, hoặc F (shift-F)-chữ cái**  
Để di chuyển nhanh đến từ có chứa chữ cái đầu nào đó  
Ta có thể bấm f(forward), sau đó là chữ cái.  
Cách này đặc biệt hữu dụng với những dòng mà dài hoặc rất dài  
Sử dụng F (shift-f) nếu muốn nhảy theo chiều ngược lại.

**w, b: di chuyển qua các từ**  
Khi muốn di chuyển sang từ tiếp theo hoặc về từ trước đó  
có thể sử dụng w và b.  
Mình ít dùng 2 phím này

**/ hoặc ? <từ khóa>: Để tìm kiếm**  
Tương đương với Ctrl + f ở hầu hết editor khác.  
Tuy nhiên, mục từ khóa hỗ trợ biểu thức chính quy luôn.  
Hay nói cách khác, chức năng tìm kiếm chính là 1 engine xử lý biểu thức  
chính quy luôn.  
Để dịch chuyển sang kết quả sau/trước sử dụng n và N(shift-n)

3. Một số thao tác cơ bản
--------------------------

### 3.1 Bôi đen

Với mình, đây là thao tác phổ biến nhất. :))  
**Sử dụng Visual mode (V)**  
Bấm phím v (nhỏ) trong Command Mode để chuyển sang Visual Mode  
Sau khi chuyển sang rồi, sử dụng các phím di chuyển để tạo dùng bôi đen  
Cái này tương tự việc ta giữ phím Shift trong các Editor khác.  
Nhưng với VIM không cần giữ phím gì hết.

**Sử dụng V (Shift-V)**  
để đôi đen 1 dòng

### 3.2 Copy, Cut, Paste

**Bôi đen rồi p**  
- Sau khi đã bôi đen, sử dụng  
_Copy_ -> phím y  
_Cut_ -> phím x  
- Di chuyển rồi paste bằng : p

**yyp : copy một dòng**  
Câu lệnh thần thánh, tạo ra một dòng giống hệt dòng hiện tại.  
Giải thích câu trên như sau: yy là copy dòng,p là paste ngay sau dòng hiện tại.

### 3.3 Xóa

**x : Xóa kí tự**  
**dd: Xóa cả dòng**  
**d$: Xóa đến cuối dòng**