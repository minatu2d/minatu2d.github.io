---
layout: post
title: ASN.1 - Cú pháp cơ bản
date: 2016-11-16 15:49:05.000000000 +09:00
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
  _rest_api_client_id: "-1"
  _publicize_job_id: '28979635402'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/11/16/asn-1-cu-phap-co-ban/"
---
Tiếp theo loạt bài về ASN.1, bài này sẽ "dịch" tài liệu của anh [Isida So](http://www5d.biglobe.ne.jp/~stssk/mail.html) nói về cú pháp cơ bản của ASN.1.

Việc đầu tiên khi ứng dụng ASN.1 hoặc thiết kế một giao thức mới là viết định nghĩa cho các kiểu dữ liệu có thể được sử dụng.

ASN.1 là một ngôn ngữ vì thế, cũng giống với C, nó có các kiểu cơ bản và các cơ chế cho phép mở rộng để định nghĩa thêm các kiểu mới.

Ví dụ, khi muốn biểu diễn số điện thoại (**TelephoneNumber**), trong trường đã có kiểu dành cho chuỗi kí tự số (**NumericString**) thì ta chỉ việc viết như sau:

```cpp  
TelephoneNumber ::= NumericString  
```

1.  Các kiểu cơ bản
2.  Kiểu cấu trúc
3.  Thẻ (Tag)
4.  Kiểu bộ phận
5.  Quy ước mở rộng (Kí hiệu mở rộng)

Ta sẽ đi vào từng phần.

### 1. Các kiểu cơ bản

Các kiểu dữ liệu không thể phân tách thêm được nữa như số, chuỗi kí tự được gọi là các kiểu cơ bản. Nó sẽ gồm những kiểu cơ bản như sau:(không đầy đủ):

Kiểu lựa chọn

(**CHOICE**)

Chọn 1 kiểu từ nhiều kiểu

**Credit-card ::= SEQUENCE { number NumericString (SIZE (20)),
  expiry-date NumericString (SIZE (6)) -- MMYYYY -- }**

Kiểu sếp tuần tự

(**SEQUENCE**)

Nhiều kiểu được đặt theo một thứ tự nhất định

**Point ::= SEQUENCE { x INTEGER, y INTEGER }**

Kiểu mảng

(**SEQUENCE OF**)

Giống như kiểu mảng, các phần tử cùng 1 kiểu và có thứ tự

**Line ::= SEQUENCE OF Point**

Kiểu tập hợp hỗn hợp

(**SET**)

Tập hợp nhiều kiểu và không có thứ tự gì hết.

**Human ::= SET { name PrintableString, age INTEGER}**

Kiểu tập hợp đồng nhất

(**SET-OF**)

Tập hợp nhiều phần tử cùng kiểu

**PrimeNumber ::= SET OF INTEGER**

**Kiểu tùy ý**

Kiểu này có thể dùng bất cứ dữ nào.  
Tuy nhiên, do gán vô điều kiện như thế sẽ dẫn đến không hiện thực hóa được.  
Vì thế kiểu này không được khuyến khích dùng.

Kiểu ngoài

(**EXTERNAL**)

Một khiểu được định nghĩa ở đâu đó.  
Có thể sử dụng bất cứ dữ liệu nào. Kiểu ngoài (**EXTERNAL**) có thể được định nghĩa như ví dụ bên dưới đây.

**EXTERNAL ::= [UNIVERSAL 8] IMPLICIT SEQUENCE {
  direct-reference OBJECT IDENTIFIER OPTIONAL,
  indirect-reference INTEGER OPTIONAL,
  data-value-descriptor ObjectDescriptor OPTIONAL,
  encoding CHOICE {
    single-ASN1-type [0] ANY,
    octet-aligned [1] IMPLICIT OCTET STRING,
    arbitrary [2] IMPLICIT BIT STRING }}**

### 2. Kiểu có cấu trúc

Khi kết hợp các kiểu dữ liệu cơ bản với nhau, ta sẽ có **Kiểu có cấu trúc**. Ví dụ: thông tin mô tả 1 người như tên, số điện thoại, số fax trong danh bạ có thể được định nghĩa như sau:

```c  
TelephoneNumberRecord ::= SET {  
name VisibleString,  
tel TelephoneNumber,  
fax TelephoneNumber  
}  
```

Khi định nghĩa kiểu có cấu trúc, ta sẽ sử dụng thêm các định nghĩa sau:

Kiểu lựa chọn

(**CHOICE**)

Chọn 1 kiểu từ nhiều kiểu

**Credit-card ::= SEQUENCE { number NumericString (SIZE (20)),
  expiry-date NumericString (SIZE (6)) -- MMYYYY -- }**

Kiểu sếp tuần tự

(**SEQUENCE**)

Nhiều kiểu được đặt theo một thứ tự nhất định

**Point ::= SEQUENCE { x INTEGER, y INTEGER }**

Kiểu mảng

(**SEQUENCE OF**)

Giống như kiểu mảng, các phần tử cùng 1 kiểu và có thứ tự

**Line ::= SEQUENCE OF Point**

Kiểu tập hợp hỗn hợp

(**SET**)

Tập hợp nhiều kiểu và không có thứ tự gì hết.

**Human ::= SET { name PrintableString, age INTEGER}**

Kiểu tập hợp đồng nhất

(**SET-OF**)

Tập hợp nhiều phần tử cùng kiểu

**PrimeNumber ::= SET OF INTEGER**

**Kiểu tùy ý**

Kiểu này có thể dùng bất cứ dữ nào.  
Tuy nhiên, do gán vô điều kiện như thế sẽ dẫn đến không hiện thực hóa được.  
Vì thế kiểu này không được khuyến khích dùng.

Kiểu ngoài

(**EXTERNAL**)

Một khiểu được định nghĩa ở đâu đó.  
Có thể sử dụng bất cứ dữ liệu nào. Kiểu ngoài (**EXTERNAL**) có thể được định nghĩa như ví dụ bên dưới đây.

**EXTERNAL ::= [UNIVERSAL 8] IMPLICIT SEQUENCE {
  direct-reference OBJECT IDENTIFIER OPTIONAL,
  indirect-reference INTEGER OPTIONAL,
  data-value-descriptor ObjectDescriptor OPTIONAL,
  encoding CHOICE {
    single-ASN1-type [0] ANY,
    octet-aligned [1] IMPLICIT OCTET STRING,
    arbitrary [2] IMPLICIT BIT STRING }}**

Kiểu tập hợp(SET) hoặc sếp tuần tự có thể sử dụng **OPTIONAL** để mô một dữ liệu có thể được khuyết.

### 3. Thẻ (Tag)

Để tránh nhập nhằng trong định nghĩa kiểu, người ta sử dụng các Thẻ.  
Đặt biệt, trong trường hợp kiểu lựa chọn（Tuổi và số năm học đều là số nguyên chẳng hạn）hoặc, trong kiểu sếp tuần tự (SEQUENCE OF) hoặc kiểu tập hợp, có các thành phần có thể khuyến.

Để tránh nhập nhằng khi các thành phần đó bị khuyết, ta cũng sử dụng Thẻ (Tag).

Thẻ được viết bên trong 1 ngoặc vuông (như bên dưới).  
Ví dụ: Ở bên dưới kiểu dữ liệu được định nghĩa có địa chỉ, số điện thoại có thể khuyết.

ASN.1 có 4 nhóm thẻ:

*   Lớp phổ biến :

```c  
[UNIVERSAL <số>]  
```

Lớp này được sử dụng để chỉ các dữ liệu phổ biến, và chỉ được sử dụng trong các tài liệu ASN.1 chuẩn.  
Bao gồm :kiểu số nguyên, kiểu byte (octet), kiểu số thực, và kiểu liệt kê. (Enum).

*   Lớp ứng dụng

```c  
[APPLICATION <số>]  
```

Lớp này được sử dụng để chỉ các kiểu dữ liệu sử dụng trong một số ứng dụng cụ thể.

*   Lớp ngữ cảnh

```c  
[<số>]  
```

Chỉ có ý nghĩa trong định nghĩa hiện tại thôi.  
Lớp này được sử dụng nhiều nhất.

*   Lớp riêng

```c  
[PRIVATE <số>]  
```

Giới hạn sử dụng trong phạm vi một nhóm các công ty, tổ chức cụ thể. Có có trường hợp sử dụng lớp này khi muốn các protocol có sẵn trong các dự án cá nhân.

_Chú ý_: Kiểu lựa chọn không sử dụng Thẻ.

Sau đây là một số Thẻ của lớp Phổ biến:

Mã thẻ

Kiểu

[UNIVERSAL 0]

Sử dụng trong BER, dùng để đánh dấu kết thúc cho loại dữ liệu có độ dài thay đổi.

[UNIVERSAL 1]

Kiểu logic (BOOLEAN)

[UNIVERSAL 2]

Kiểu số nguyên(INTEGER)

[UNIVERSAL 3]

Kiểu dãy bit(BIT STRING)

[UNIVERSAL 4]

Kiểu octet (byte)(OCTET STRING)

[UNIVERSAL 5]

Kiểu không có gì(NULL)

[UNIVERSAL 6]

Kiểu chỉ định đối tượng(OBJECT IDENTIFIER)

[UNIVERSAL 7]

Kiểu mô tả đối tượng - Object Descriptor

[UNIVERSAL 8]

Kiểu ngoài(EXTERNAL), INSTANCE OF

[UNIVERSAL 9]

Kiểu số thực(REAL)

[UNIVERSAL 10]

Kiểu liệt kê(ENUMERATED)

[UNIVERSAL 11]

EMBEDDED PDV

[UNIVERSAL 12]

UTF8String

[UNIVERSAL 13]

RELATIVE-OID

[UNIVERSAL 16]

Kiểu sếp tuần tự(SEQUENCE), Kiểu mảng(SEQUENCE OF)

[UNIVERSAL 17]

Kiểu tập hợp hỗn hợp(SET), Kiểu tập hợp đồng nhất(SET-OF)

[UNIVERSAL 18]

NumericString

[UNIVERSAL 19]

PrintableString

[UNIVERSAL 20]

TeletexString, T61String

[UNIVERSAL 21]

VideotexString

[UNIVERSAL 22]

IA5String

[UNIVERSAL 23]

UTCTime

[UNIVERSAL 24]

GeneralizedTime

[UNIVERSAL 25]

GraphicString

[UNIVERSAL 26]

VisibleString, ISO646String

[UNIVERSAL 27]

GeneralString

[UNIVERSAL 28]

UniversalString

[UNIVERSAL 29]

CHARACTER STRING

[UNIVERSAL 30]

BMPString

### 4. Kiểu bộ phận

Kiểu bộ phận, tức là giới hạn một kiểu đã có để tạo ra kiểu mới.  
Tức là kiểu mới sẽ là con của kiểu cũ.  
Cú pháp định nghĩa như sau:

```c  
<Tên kiểu mới> ::= <Kiểu cũ đã có> ( <mô tả giới hạn> )  
```

Với kiểu **SET OF** và **SEQUENCE OF**, nếu phần mô tả giới hạn ở phía sau **OF** sẽ gây ra khó hiểu. Vì thế, để tránh điều đó **OF** sẽ được viết sau giới hạn.

_Ví dụ_: dưới đây mô tả một một tập hợp gồm 50 phần tử, mỗi phần tử là một chuỗi có độ dài từ 1-10 kí tự.

```c  
MemberList ::= SET SIZE(1..50) OF String(SIZE(1..10))  
```

Phần **SIZE** đầu tiên là để giới hạn cho **SET**, phần **SIZE** sau là để giới hạn cho **String**.

Thông thường, có 2 cách để giới hạn cho ASN.1, đó là giới hạn bằng kiểu bộ phận và giới hạn kiểu thông thường. Kiểu thông thường ở đây là giới hạn khi hiện thực.

Ở đây ta nó đến giới hạn bằng kiểu bộ phận thông qua các phép toán tập hợp và liệt kê.

##### Phép hợp (UNION) (đối với tập hợp)

Cú pháp:

```cpp  
<giới hạn 1>|<giới hạn 2>  
```

_Ví dụ_:

```cpp  
PrimeNumbers ::= INTEGER ( 2 | 3 | 5 | 7 | 11 | 13 )  
PushButtonDial ::= IA5String ("0"|"1"|"2"|"3"|"4"|"5"|"6"|"7"|"8"|"9"|"*"|"#")  
-- PushButtonDial là chuỗi 1 kí tự  
```

##### Phép giao (INTERSECTION) (đối với tập hợp)

Cú pháp:

```cpp  
<giới hạn 1>^<giới hạn 2>  
```

_Ví dụ_:

```cpp  
Example1 ::= INTEGER((1..10) ^ (5..15))  
Example1 ::= INTEGER(1..10)(5..15)  
```

##### Phép loại trừ (EXCEPT)

Cú pháp:

```cpp  
<giới hạn 1> EXCEPT <giới hạn 2>  
```

###### Phép loại trừ đảo (ALL EXCEPT)

Cú pháp:

```cpp  
<giới hạn 1> ALL EXCEPT <giới hạn 2>  
```

##### Định nghĩa tập giá trị

Lấy một tập giá trị cụ thể từ một kiểu đã có sẵn.

##### Kiểu chứa trong

Cú pháp:

```cpp  
INCLUDES <Kiểu đã có>  
```

Có cùng giá trị với kiểu đã có.  
Từ 1994, cho phép không cần viết INCLUDES

##### Khoảng giá trị

Cú pháp:

```cpp  
<Giới hạn dưới>..<Giới hạn trên>  
```

Lấy ra một khoảng giá trị từ một tập đã có.  
Nếu không muốn chỉ định giới hạn dưới, hãy sử dụng **MIN**.  
Nếu không muốn chỉ định giới hạn trên, hãy sử dụng **MAX**.

_Ví dụ_:

```cpp  
Interval ::= INTEGER (0..12)  
```

##### Giới hạn bằng kích thước (SIZE)

Cú pháp:

```cpp  
SIZE(<Giới hạn>)  
```

Giới hạn được sử dụng trong trường hợp này là độ dài bit (BIT STRING), độ dài byte (OCTET), độ dài chuỗi (STRING), số phần từ mảng (SET OF hoặc SEQUENCE OF):

_Ví dụ_:

```text  
name ::= PrintableString (SIZE (1..20)),  
Matrix ::= SEQUENCE SIZE (6) OF SEQUENCE SIZE (6) OF INTEGER(-100..100)  
-- Matrix có kích thước 6x6 phần tử,  
-- mỗi phần từ là giá trị nguyên nằm trong khoảng 100~100  
```

##### Giới hạn các kí tự Alphabet

Cú pháp:

```cpp  
FROM <một kiểu khác>  
```

Cách giới hạn này chỉ sử dụng cho chuỗi kí tự, được sử dụng để chỉ định tập giá trị được phép sử dụng trong chuỗi.

```cpp  
CapitalLettersAndSpaces ::=  
PrintableString (FROM ("A".."Z"|" "))  
```

##### Giới hạn bên trong

Cú pháp:

```cpp  
WITH COMPONENTS <Kiểu đã có>  
WITH COMPONENTS {<Giới hạn>, <Giới hạn>, <Giới hạn>}  
WITH COMPONENTS {..., <Giới hạn>}  
```

Sử dụng để giới hạn phần tử bên trong một kiểu cấu trúc.  
Câu khai báo đầu tiên sử dụng kiểu đã có làm cấu trúc luôn và giới hạn số phần cho cấu trúc đó.  
Câu khai báo thứ hai, ba sử dụng cho cấu trúc được tạo thành từ nhiều kiểu khác nhau.

_Ví dụ_:  
Địa chỉ (Address) có từ 3~6 dòng, mỗi dòng có 1~32 kí tự.

```cpp  
TextBlock ::= SEQUENCE OF VisibleString  
Address ::= TextBlock (SIZE (3..6))(WITH COMPONENT (SIZE (1..32)))  
```

### 5. Quy ước mở rộng (Extension mark)

Gần giống với cách biểu diễn giới hạn ở phía trên, chỉ khác là có thêm dấu 3 chấm (...), để biểu thị rằng tương lai có khả năng mở rộng.  
Trường hợp giới hạn cả phần mở rộng này, thì sau dấu 3 chấm, có thể viết thêm giới hạn vào.  
Có thể viết nhiều lần dấu 3 chấm để miêu tả giới hạn trên nhiều đoạn khác nhau (Tuy nhiên, qua mỗi bản ASN.1 thì điều này có thể không giống nhau).

_Ví dụ_:

```cpp  
A ::= INTEGER (0..10, ...)  
A ::= INTEGER (0..10, ..., 12)  
RadioButton ::= ENUMERATED { button1, button2, button3, ... }  
RadioButton ::= ENUMERATED { button1, button2, button3, ..., button4, button5 }  
```

Khi dùng **SEQUENCE**, **SET**, **CHOICE**, trongtrường hợp sử dụng nhiều dấu mở rộng, ASN.1 cho phép có thể sử dụng dấu ngoặc vuông để biểu diễn khi thêm dấu mở rộng vào một định nghĩa đã mở rộng.

_Ví dụ_:

```cpp  
Dimensions ::= SET { x INTEGER,  
y INTEGER,  
...,  
[[z INTEGER]] } -- version 2  
Afters ::= CHOICE { cheese IA5String,  
dessert IA5String,  
...,  
[[ coffee NULL ]] } -- version 2  
Afters ::= CHOICE { cheese IA5String,  
dessert IA5String,  
...,  
[[ coffee NULL,  
tea NULL ]] , -- version 2  
[[ cognac IA5String ]] } -- version 3  
Type ::= SEQUENCE { component1 INTEGER,  
component2 BOOLEAN, -- version 1  
... }  
Type ::= SEQUENCE { component1 INTEGER,  
component2 BOOLEAN,  
...,  
[[ component3 REAL ]], -- version 2  
... }  
```