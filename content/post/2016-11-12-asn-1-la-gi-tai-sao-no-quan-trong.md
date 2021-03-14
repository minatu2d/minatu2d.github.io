---
layout: post
title: ASN.1 là gì? Tại sao nó quan trọng.
date: 2016-11-12 08:51:58.000000000 +09:00
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
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: '43452'
  _publicize_job_id: '28830131927'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/11/12/asn-1-la-gi-tai-sao-no-quan-trong/"
---
Mình thấy ASN.1 khá quan trọng nhất là trong thiết kế giao thức trao đổi dữ liệu giữa các thiết bị tính toán, đặc biệt là các thiết bị tài nguyên nhỏ.

Google tiếng Việt thấy khá ít thông tin về ASN.1.  
Mình dự định sẽ tìm hiểu thêm một chút về ASN.1 và làm loại bài viết về nó.

Nội dung bài này sẽ giới thiệu sơ qua về ASN.1.  
Bài viết tham khảo rất nhiều từ bài viết của anh [Ishida So](http://www5d.biglobe.ne.jp/~stssk/asn1/index.html)

1.  ASN.1 là gì?
2.  Một số ứng dụng
3.  Làm sao để sử dụng ASN.1

### 1. ASN.1 là gì?

Tìm kiếm trên google sẽ thấy [bài viết](http://laptrinh.vn/d/3211-gioi-thieu-ve-asn1.html) từ 2013 của tác giả [Mr.datnh](http://laptrinh.vn/m/2-mrdatnh.html) trên diền đàn [http://laptrinh.vn](http://laptrinh.vn).  
Nội dung có vẻ được dịch từ [wikipedia](https://en.wikipedia.org/wiki/Abstract_Syntax_Notation_One) nhưng có vẻ hơi lủng củng, mình sẽ dịch lại để nó "lủng củng" hơn nữa.

Trên wikipedia viết:

> Abstract Syntax Notation One (ASN.1) is a standard and notation that describes rules and structures for representing, encoding, transmitting, and decoding data in telecommunications and computer networking. The formal rules enable representation of objects that are independent of machine-specific encoding techniques. Formal notation makes it possible to automate the task of validating whether a specific instance of data representation abides by the specifications. In other words, software tools can be used for the validation.[1]
> 
> ASN.1 is a joint standard of the International Organization for Standardization (ISO), International Electrotechnical Commission (IEC), and International Telecommunication Union Telecommunication Standardization Sector ITU-T, originally defined in 1984 as part of CCITT X.409:1984. ASN.1 moved to its own standard, X.208, in 1988 due to wide applicability. The substantially revised 1995 version is covered by the X.680 series. The latest revision of the X.680 series of recommendations is the 5.0 Edition, published in 2015.

Mình sẽ dịch lại như sau:

> Kí pháp trừu tượng một (Abstract Syntax Notation One) (viết tắt là ASN.1) là một bộ các tiêu chuẩn và kí hiệu miêu tả các quy tắc và cấu trúc cho biểu diễn, mã hóa (encoding), truyền đi và giải mã dữ liệu, thường được sử dụng trong viễn thông, và mạng thiết tị tính toán (computer networking). Các quy tắc chính thức này cho phép biểu diễn một đối tượng bất kì mà không phụ thuộc vào máy hoặc kĩ thuật mã hóa cụ thể nào. Kí pháp này cũng cho phép thực hiện một cách tuần tự để biết được một đoạn dữ liệu nào đó có tuân theo quy tắc cụ thể không. Hay nói cách khác, người ta có thể dùng phần mềm để thực hiện việc kiểm tra này.
> 
> ASN.1 một chuẩn được thống nhất giữa ISO, IEC, và ITU-T. Ban đầu được mô tả là một phần của chuẩn CCITT X.409:1984 năm 1984. ASN.1 được chuyển thành một chuẩn riêng, tên là X.208 vào năm 1998 cho phép sử dụng rộng rãi. Đến năm 1995, chuẩn này nằm trong X.680. Và phiên bản mới nhất là 5.0 của X.680 được đưa ra năm 2015.

Từ giờ ta sẽ gọi là **kí pháp ASN.1**.

Một ví dụ đơn giản của kí pháp ASN.1:

```cpp  
TestModule DEFINITIONS ::= BEGIN -- Định nghĩa một module  
HinhChuNhat ::= SEQUENCE { -- Tên của kiểu mô tả đối tượng Hình chữ nhật  
chieuRong INTEGER, -- Chiều rộng  
chieuDai INTEGER, -- Chiều dài  
} -- Kết thúc HinhChuNhat  
END -- Kết thúc TestModule  
```

Ta thấy nó trông giống định nghĩa một cấu trúc trong ngôn ngữ C thôi. Nhưng thực sự không hoàn toàn giống.  
Ví dụ trên không cho biết một đối tượng **HinhChuNhat** được miêu tả như thế nào trên bộ nhớ.

**Ví dụ**: đối tượng _hinhA_ có kiểu là **HinhChuNhat** với _chieuRong_ = 20, _chieuDai_ = 30 ta cũng chưa thể biết được đối tượng _hinhA_ đó được biểu diễn trên bộ nhớ ra sao, _chieuRong_, _chieuDai_ sẽ được biểu diễn bằng mấy byte khi cả đối tượng _hinhA_ được biểu diễn.

Còn trong ngôn ngữ C chẳng hạn, khi ta định nghĩa một cấu trúc. Ngay từ lúc biên dịch, ta đã biết được mỗi biến kiểu cấu trúc đó, mỗi trường được biểu diễn như thế nào trên bộ nhớ rồi. Tham khảo thêm ở bài về [Data Structure Aligment](https://lazytrick.wordpress.com/2016/10/26/data-structure-alignment-la-gi-tai-sao-phai-hieu-no-khi-code-c/)

### 2. Một số ứng dụng của ASN.1

Là ngôn ngữ để định nghĩa các giao thức truyền thông.  
Nó được sử dụng ở rất nhiều nơi như: Chữ kí số, LDAP, Kerberos.  
Có thể coi ASN.1 có vai trò giống như XML (và các công nghệ tương tự) nhưng nó có trước và được ứng dụng từ khá lâu rồi.  
Như đã nói ở phần trên, ASN.1 tách phần định nghĩa với phần định dạng dữ liệu thực tế ra.  
Loại định dang thông thường nhất là **BER**, bổ sung thêm một số giới hạn vào sẽ có **CER** và **DER**, loại nhỏ nhất, compact nhất là **PER**.

### 3. Làm sao để sử dụng ASN.1

```cpp  
ASN.1 definitions  
+  
|  
|  
+-----------v-----------+  
| |  
| ASN.1 Compiler |  
| |  
+-----------+-----------+  
|  
v  
Encoder/Decoder  
```

Như ta đã nó ở trên ASN.1 chia làm 2: phần định nghĩa và phần đinh dạng dữ liệu thực tế.

Để có thể sử dụng ASN.1 trong ứng dụng,

#### a. ASN.1 definitions

Chứa định nghĩa của toàn bộ các đối tượng sẽ sử dụng ASN.1 trong ứng dụng.

#### b. Encoder/Decoder

Đây là source code hoặc binary (ở dạng thư viện.a hoặc .so kèm them các file header) chứa các hàm Encode/Decode dữ liệu.  
Thông thường các kiểu đã được định nghĩa sẽ xuất hiện một structure tương ứng trong các file header kèm theo.

Encode: lấy input là một biến cấu trúc (Structure của ngôn ngữ C chẳng hạn), sinh ra output là một mảng byte.

Decode: lấy đầu vào là một mảng byte, sinh ra output là một biến cấu trúc, cái đã dùng để Encode ở bước trên.

Bên gửi dữ liệu sẽ Encode và quẳng vào đường truyền.  
Bên nhận dữ liệu sẽ Decode dứ liệu nhận được thành cấu trúc có thể sử dụng ở phía ứng dụng lớp trên.

#### c. ASN.1 Compiler

Là phần mềm sẽ đọc file định nghĩa ASN.1 (file text) rồi sinh ra bộ Encoder/Decoder (các hàm sử dụng để Encoding/Decoding) cho tất cả các kiểu được định trong file đầu vào.  
Compiler có khá ít sản phẩm opensource, các sản phẩm closed-source bán với gía rất là đắt.  
Ta sẽ làm quen thực tế với open-source ASN.1 Compiler trong 1 bài tiếp của Serial này.