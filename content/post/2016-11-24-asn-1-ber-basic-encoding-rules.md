---
layout: post
title: ASN.1 - BER (Basic Encoding Rules)
date: 2016-11-24 15:58:01.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
- Communication
tags:
- ASN1
- PER
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '29263673789'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/11/24/asn-1-ber-basic-encoding-rules/"
---
Tiếp tục về [ASN.1](https://lazytrick.wordpress.com/tag/asn1/).

Như bài đầu tiên, ASN.1 tách biệt phần định nghĩa dữ liệu (các file định nghĩa) với phần hiện thực dữ liệu (mỗi trường được biểu diễn bằng mấy byte, mấy bit, etc)

Ta sẽ tiếp tục nói về hiện thực dữ liệu.  
Các phương thức để hiện thực dữ liệu gồm có: **BER**, **CER/DER**, và **PER**.

Trong đó:  
**BER** : Basic Encoding Rules  
**CER/DER** : Canonical Encoding Rules/Distinguished Encoding Rules  
**PER** : Packet Encoding Rules

Bài này sẽ nói về **BER** và **CER/DER**, **PER** sẽ để ở bài sau.

Link nguồn chính:  
[http://www5d.biglobe.ne.jp/~stssk/asn1/index.html](http://www5d.biglobe.ne.jp/~stssk/asn1/index.html)

của bác [Ishida So](http://www5d.biglobe.ne.jp/~stssk/mail.html)

_Một chút về lịch sử_:  
**BER** ban đầu là phương pháp hiện thực duy nhất của ASN.1. Từ 1985, ASN.1 tách riêng phần định nghĩa và phần hiện thực dữ liệu ra, vì thế có thêm nhiều phương thức hiện thực khác được đưa ra.  
**CER/DER** được kế thừa từ **BER**, còn **PER** được đưa ra năm 1994.

### 1. Hiện thực hóa TLV (TLV Encoding)

Về mặt kĩ thuật, **BER** sử dụng **TLV** cho việc hiện thực dữ liệu.  
Khi Encode **TLV** sử dụng 3 yếu tố dưới đây, để biểu diễn từng loại dữ liệu.  
Khi Decode, dựa vào 3 yếu tố này để xác định kiểu, độ dài tướng ứng của mỗi loại dữ liệu.

**T** : Type, tức là kiểu dữ liệu, sử dụng **Tag**  
**L** : Length, tức là độ dài dữ liệu  
**V** : Value, tức là giá trị

Với kiểu cơ bản, phần giá trị (**V**) sẽ được biểu diễn theo dữ liệu của kiểu. Còn với kiểu cấu trúc, phần dữ liệu lại một lần nữa được biểu diễn bằng **TLV**.

![pic_1](/post/images/pic_1.png)

### 2. Thứ tự bit (Bit order)

**BER** sử dụng đơn vị 8 bit (1 octet) để biểu diễn.  
Tức là cho dù kiểu dữ liệu là 1 bit thì nó vẫn sử dụng 1 octet để biểu diễn.  
Thứ tự octet là Octet ý nghĩa cao -> Octet ý nghĩa thấp) theo chiều từ trái qua phải. Điều này hướng đến việc thích hợp với truyền dữ liệu qua mạng (Network Byte Order).

![pic_3](/post/images/pic_3.png)

Còn trong 1 octet, thứ tự và ý nghĩa được thể hiện như sau:

![pic_4](/post/images/pic_4.png)

### 3. Hiện thực hóa các thẻ

Thẻ (**Tag**) là khái niệm đã được nhắc đến trong [bài trước](https://lazytrick.wordpress.com/2016/11/16/asn-1-cu-phap-co-ban/)

Có 4 nhóm thẻ khác nhau là:  
- Phổ biến (UNIVERSAL)  
- Ứng dụng (APPLICATION)  
- Đặc trưng ngữ cảnh  
- Riêng (PRIVATE)

Ta hãy xem **BER** biểu diễn **T** như thế nào:  
- Bit thứ 8-7: Biểu diễn nhóm Thẻ  
- Bit thứ 6: Biểu diễn là kiểu cơ bản hay cấu trúc  
- Bit thứ 5-1: Biểu diễn số Thẻ  
Cụ thể như sau:

Chỉ số Thẻ

Các biểu diễn

Lớp：

００：Phổ biến

０１：Ứng dụng

１０：Đặc trưng ngữ cảnh

１１：Riêng

Ｐ／Ｃ：

０：Kiểu cơ bản

１：Kiểu cấu trúc

０～３０

![](/post/images/pic5.png)

３１～

![](/post/images/pic_6.png)

_Ví dụ_: Số điện thoại với định nghĩa và hiện thực tương ứng như bên dưới đây là một ví dụ cho 2 lần **TLV**.

```cpp  
TelephoneNumber ::= [1] NumericString  
```

![pic_8](/post/images/pic_8.png)

Khi muốn bỏ biểu diễn phần **T** đi, **BER** sử dụng từ khóa **IMPLICIT** trong định nghĩa của ASN.1.  
*Thế mới biết, từ khóa này ban đầu dành cho **BER**

```cpp  
TelephoneNumber ::= [1] IMPLICIT NumericString  
```

![pic_9](/post/images/pic_9.png)

Ngược lại với **IMPLICIT**, sử dụng **EXPLICIT** sẽ phải sử dụng **TLV** cho phần dữ liệu **V**.

### 4. Hiện thực độ dài dữ liệu(L)

*   Khi độ dài dữ liệu nhỏ <= 127, được gọi là Short Definite - Dạng ngắn, 1 octet được sử dụng để biểu diễn (thực sự sử dụng 7 bit, còn bit đầu là 0). - Khi độ dài >=128, 2 octet trở lên được sử dụng để biểu diễn, gọi là Long Definite - Dạng dài. Bit đầu của octet đầu tiên được set là 1. 7 bit tiếp theo miêu tả độ dài tính theo octet của độ dài dữ liệu (1-126).

![pic_12](/post/images/pic_12.png)  
Tức là nếu độ dài dữ liệu (**V**)là 128-255 thì 7 bit này sẽ có giá trị 1. Nếu độ dài dữ liệu từ 256-65535 thì 7 bit này sẽ có giá trị 2.  
- Giá trị 127 cũng có thể được sử dụng nếu cần thiết.

![pic_13](/post/images/pic_13.png)

Trong trường hợp của kiểu cấu trúc, phải hiện thực với độ dài có thể thay đổi (Inefinite). Trường độ dài (**L**) sẽ có độ dài là 1 octet, bit cao nhất được gán là 1, các bit còn lại là 0 (tức là **giá trị 0x80**). Phần giá trị **V** sẽ được hiện thực ở ngay phía sau, phần này sẽ được kết thúc bởi **2 octet toàn là 0**.

![pic_11](/post/images/pic_11.png)

### 5. Hiện thực phần giá trị

Mỗi giá trị phụ thuộc vào từng Tag tương ứng với nó.

Kiểu logic

(**BOOLEAN**)

Kiểu cơ bản, sử dụng 1 octet, giá trị 0 cho FALSE, giá trị khác 0 cho TRUE

Kiểu NULL

(**NULL**)

Kiểu cơ bản, độ dài bằng 0, không phục thuộc vào giá trị

Kiểu số nguyên

(**INTEGER**)

Kiểu cơ bản, giá trị được biểu diễn ở dạng bù 2 (có dấu).

Kiểu số thực

(**REAL**)

Đại khái là không giống với cách biểu diễn trên máy tính. (Giải thích dịp khác)

Kiểu liệt kê

(**ENUMERATED**)

Phụ thuộc vào các giá trị mà nó biểu diễn, sẽ được biểu diễn giống với kiểu nguyên

Kiểu dãy bit

(**BIT STRING**)

Có thể biểu diễn bằng cả kiểu cơ bản lẫn cấu trúc.  
Trường hợp sử dụng kiểu cơ bản, octet đầu tiên chứa giá trị bằng số bit không được sử dụng của octet cuối cùng. Dù nó là 0 hay 1 đi nữa đều được biểu diễn. Trong trường hợp này, nếu độ dài dãy bit là 0, thì no không thục thuộc vào dữ liệu chứa nữa.  
Trường hợp sử dụng kiểu cấu trúc, được biểu diễn bằng dãy kiểu cơ bản.

Kiểu OCTET (byte)

(**OCTET STRING**)

Có thể biểu diễn bằng cả kiểu cơ bản lẫn kiểu cấu trúc.  
Với kiểu cơ bản, thì biểu diễn bằng một dãy Octet thôi.  
Với kiểu cấu trúc, được biểu diễn bằng một dãy kiểu cơ bản khác.

Kiểu chỉ định đối tượng

(**OBJECT IDENTIFIER**)

Được biểu diễn bằng kiểu cơ bản. Chỉ số của đối tượng cũng được chỉ ra. (Chi tiết sẽ được giải thích sau)

Kiểu giới hạn kí tự

(**...String**)

Được biểu diễn giống với kiểu Octet

Kiểu lựa chọn

(**CHOICE**)

Kiểu lựa chọn bản thân nó không được biểu diễn bằng BER, BER chỉ hiện thực khi đã được lựa chọn rồi thôi.

Kiểu xếp tuần tự

(**SEQUENCE**)

Biểu diễn bằng kiểu cấu trúc. Nội dung sẽ là nội dung của môi thành phần.

Kiểu mảng

(**SEQUENCE OF**)

Giống với kiểu xếp tuần tự

Kiểu tập hợp hỗn hợp

(**SET**)

Thứ tự tùy ý sẽ bị bỏ đi, và được biểu diễn giống với kiểu xếp tuần tự.

Kiểu tập hợp đồng nhất

(**SET-OF**)

Giống với kiểu sếp tuần tự.