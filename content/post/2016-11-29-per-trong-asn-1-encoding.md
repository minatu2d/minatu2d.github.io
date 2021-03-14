---
layout: post
title: PER trong ASN.1 Encoding
date: 2016-11-29 14:48:30.000000000 +09:00
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
  _publicize_job_id: '29431538473'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/11/29/per-trong-asn-1-encoding/"
---
**PER** là phương thức biểu diễn dữ liệu ngắn gọn và xúc tích nhất có thể của ASN.1. Thay vì sử dụng **TLV** như **BER**, **PER** sử dụng _Preamble_ (diễn cho nhiều hoặc trạng thái bị lược bỏ của một dữ liệu bên trong), giá trị kích thước(cũng có thể bị lược bỏ), giá trị (cũng có thể bị lược bỏ), hay gọi là **PLV**.

Đơn vị biểu diễn của **PER** không phải Octet mà là Bit. Mỗi phần tử sẽ thuộc 1 trong 3 loại sau:  
1. Trường dư bit, tức là dãy bit không thể biểu diễn bằng một số nguyên các Octet.  
2. Trường nguyên bit, có thể biểu diễn bằng một số nguyên các Octet.  
3. Danh sách trường, chứa cả các thành phần là nguyên bit và dư bit.

  
Trong **BER**, dù không biết các định nghĩa ASN.1 của một đoạn dữ liệu, ta vẫn có thể giải mã ở mức độ nào đó dựa vào chỉ số các Thẻ (theo cấu trúc **TLV**). Với **PER**, trường độ dài, và nội dung (chỉ xuất hiện khi thực sự cần) phụ thuộc rất chặt vào định nghĩa ASN.1 của nó, vì thế khi Encode/Decode ta nhất định phải tham khảo các định nghĩa đó.  
Trong **BER**, vì sử dụng các thẻ (Tag) để phân biệt kiểu, nên ta dễ dàng mở rộng các kiểu **CHOICE**, **SET**, **SEQUENCE**. Trong **PER**, vì không có Thẻ nên tính mở rộng phải được tính đến từ trước (bước viết định nghĩa ASN.1).  
Khi định nghĩa chưa các thành phần mở rộng, việc hiện thực sẽ sử dụng **Preamble** để xác định xem trong dữ liệu có phần mở rộng hay không. Giá trị độ dài của những thành phần độ dài cố định (giới hạn bằng **SIZE** trong định nghĩa) sẽ không được đưa vào dữ liệu hiện thực (ngoại trừ một số ngoại lệ). Giá trị của **SEQUENCE**, **SET** khi hiện thực sẽ bắt đầu bằng **Preamble** để xác định sự tồn tại của các thành phần **OPTIONAL** bên trong. Còn **CHOICE** sẽ bắt đầu bằng chỉ số của cái được chọn.

Nói chung, xử lý khởi tạo cho **PER** rắc rối hơn, nhưng tốc độ Encode/Decode thì nhanh hơn **BER**, kích thước dữ liệu được Encode cũng nhỏ hơn, vì thế, rất thích hợp cho truyền qua mạng.

## 1. 4 kiểu hiện thực hóa (Encoding)

**PER** có 2 loại là: Kiểu cơ bản (**BASIC PER**), và kiểu chính quy (**CANONICAL-PER**).  
Mỗi loại trên lại có 2 loại con là: Sắp xếp lại (**ALIGNED**) và không sắp xếp lại (**UNALIGNED**).

Kiểu chính quy, thực ra là kiểu cơ bản được thêm một số điều kiện giới hạn. Nó được sử dụng nhiều trong các hệ thống **Relay**, **Chữ kí số**. Kiểu cơ bản, giá trị được biểu diễn theo một vài loại. Nhưng giới hạn lỏng hơn, nên Encoding theo kiểu cơ bản các là sẽ nhanh hơn kiểu chính quy thôi.

Sắp xếp lại (**ALIGNED**), có nghĩa các dữ liệu được xếp sao cho vừa đủ số Octet (nếu không đủ thì các bit 0 sẽ được thêm vào). Kiểu không sắp (**UNALIGNED**), nghĩa là sử dụng mọi bit, không có bit nào không sử dụng.  
Vì thế, kiểu **UNALIGED** thường sẽ nhỏ hơn **ALIGNED**.  
2 kiểu này không tương thích với nhau, một Decoder dạng **ALIGNED** sẽ không thể decode được dữ liệu kiểu **UNALIGNED**.

## 2. Hiện thực số nguyên

### 2.1 Kiểu nguyên hữu hạn

Giả sử ta sẽ biểu diễn số một số nguyên N.  
Nếu giá trị nhỏ nhất **N_MIN** và giá trị lớn nhất **N_MAX** đã được biết (trong định nghĩa ASN.1) thì ta phải biểu diễn giá trị **N-NMIN**, tức là khi **N=N_MIN** thì giá trị **0** được biểu diễn.  
Nếu **N_MIN**=**N_MAX** thì chỉ có 1 giá trị, không cần phải biểu diễn gì nữa (được khuyết luôn, biết rồi không cần cho vào tốn chỗ).  
Khoảng giá trị thực tế sẽ biểu diễn sẽ >=0, sử dụng số không dấu, và sử dụng số bit nhỏ nhất để biểu diễn khoảng giá trị từ **N-N_MIN** đến **N_MAX**.

```c  
Số bit cần để biểu diễn = log2(N_MAX - N_MIN + 1)  
Không làm tròn, bỏ phần thập phân.  
```

a. Với **UNALIGNED**, các bit bị dư ra ở Octet cuối sẽ được sử dụng để biểu diễn các thành phần tiếp theo.

b. Với **ALIGNED**, tùy thuộc vào khoảng khoảng giá trị **range** = N_MAX - N_MIN + 1 mà cách biểu diễn sẽ thay đổi.

> 1.  **range 2. **range = 256**: Sử dụng 1 Octet để biểu diễn.
> 2.  **257<=range 4. **range >= 65537**: Biểu diễn theo dạng độ dài thay đổi. Số Octet L và rồi L Octet. **L = log256(N - N_MIN)**. Định dạng của L Octet này được nói phía dưới.

### 2.2 Các số nguyên không âm nhỏ

Thông thường, các khoảng có cả các khoảng giá trị nhỏ, nhưng khoảng giá trị không giới hạn cũng không ít. Kích thước của giá trị mở rộng của **SEQUENCE**, **SET**, chỉ số trong **CHOICE** là các số nguyên có khoảng giá trị nhỏ. Được gọi là Dạng các số nguyên không âm nhỏ thông thường.  
Giả sử số cần biểu diễn là N.  
a. Nếu **N = 64**: 1 bit đầu, giá trị 1, để mô tả là **ALIGNED**, 6 bit tiếp theo được biểu diễn cho **Khoảng số nguyên giới hạn** ở ngay phần dưới đây.

### 2.3 Khoảng số nguyên giới hạn

Khoảng được nói đến ở đây là có giới hạn nhỏ nhất N_MIN nhưng không có giới hạn trên. Được biểu diễn theo dạng độ dài thay đổi được.  
Đầu tiên là số lượng Octet tối thiểu để biểu diễn **N - N_MIN**, sau đó là L Octet tương ứng ở dạng không dấu.  
Tùy vào **UNALIGNED** hay **ALIGNED** mà các biểu diễn L Octet này sẽ khác nhau. Chi tiết được trình bày ở phần sau.

### 2.4 Kiểu nguyên không giới hạn.

Giả sử ta sẽ biểu diễn N mà nó không được định nghĩa một giới hạn nào cả. Đương nhiên sẽ sử dụng biểu diễn độ dài thay đổi.  
Đầu tiên là số lượng Octet L, sau đó là L Octet, nhưng L Octet này được biểu diễn ở dạng nhị phân có dấu (bù 2).  
Tùy vào **UNALIGNED** hay **ALIGNED** mà các biểu diễn L Octet này sẽ khác nhau. Xem ở phần sau. Chi tiết được trình bày ở phần sau.

### 2.5 Biểu diễn độ dài

Trong **BER**, trường độ dài luôn luôn được đưa vào khi biểu diễn, và theo đơn vị Octet. Nhưng **PER** thì khác, nó chỉ xuất hiện khi thực sự cần mà thôi.  
Về biểu diễn độ dài dữ liệu,  
**BER** biểu diễn độ dài theo đơn vị Octet với mọi kiểu dữ liệu.  
**PER** biểu diễn theo độ đơn vị Bit cho dãy Bit, biểu diễn đơn vị Octet cho OCTET STRING và kiểu mở, dãy kí tự theo số kí tự, **SEQUENCE OF** và **SET OF** theo số phần tử.  
Trong ASN.1 thì độ dài cũng có thể là có khoảng giá trị là 0-vô cùng, vì thế cách biểu diễn của độ dài cũng bị ảnh hưởng.

a. Nếu giới hạn trên của độ dài là 65535 và nó là 1 giá trị cố định, thì không cần biểu diễn độ dài nữa. (tức là trường độ dài sẽ bi khuyết)  
b. Trường hợp độ dài bit của _Preamble_, giả sử độ dài là nBit, thì giá trị **nBit - 1** sẽ được biểu diễn theo **Các số nguyên không âm nhỏ** ở trên.  
c. Trường hợp **ALIGNED**:

> 1.  Nếu giới hạn trên của độ dài len 2. Nếu giới hạn trên của độ dài len từ 655356 trở lên hay là vô cùng:  
>     \>1. len >2. 128 <= len >3. len >= 16384: Giá trị độ len được biểu diễn một lần nữa giống như một giá trị.

d. Trường hợp **UNALIGNED**:

> 1.  Nếu giới hạn trên của len 2. Các trường hợp khác, ngoại trừ những kiểu không sử dụng Octet để biểu diễn thì sẽ được biểu diễn giống với trường hợp **ALIGNED**.

### 2.6 Kiểu mở

Là kiểu được thêm vào sau các dữ liệu được miêu tả. Nó không có giới hạn về độ dài, vì thế được biểu diễn theo kiểu độ dài thay đổi.  
Đầu tiên là độ dài L (biểu diễn bằng đơn vị Octet), sau đó là dãy L Octet.

## 3. Cách biểu diễn cụ thể cho mỗi kiểu

### 3.1 Kiểu logic (BOOL)

Sẽ biểu diễn bằng 1 bit thôi. TRUE được biểu diễn là 1, FALSE được biểu diễn là 0. Không có trường độ dài.

### 3.2 Kiểu số nguyên

Phạm vi biểu diễn của tập giá trị sẽ quyết định giá trị thuộc tập đó được biểu diễn như thế nào.

Các phạm vi biểu diễn có thể của tập số nguyên sẽ bao gồm:  
- Chỉ có 1 giá trị

```cpp  
Two ::= INTEGER (2)  
```

*   Khoảng giá trị có một đầu hữu hạn (cận trên hoặc cận dưới), và một đầu vô hạn (cận trên hoặc cận dưới), ta sẽ sử dụng **MIN**, **MAX** trong trường hợp này.

```cpp  
From3to15 ::= INTEGER (3..15)  
PositiveOrZeroNumber ::= Number (0..MAX)  
```

*   Tập giá trị chứa trong một tập có sẵn, quan hệ thuộc.
*   Tập giá trị là kết quả của các phép tập hợp như: Hợp, Giao, Loại trừ, Loại trừ đảo.
    

Số nguyên sẽ được biểu diễn theo các bước mô tả dưới đây:

1.  Nếu có **dấu mở rộng** (Extension Mark), thì sẽ có 1 bit (_Preamble_) để biểu diễn (là **UNALIGNED**) trạng thái mở rộng.  
    Nếu giá trị được biểu diễn thuộc khoảng giá trị được mô tả ban đầu (chưa trở thành mở rộng) thì bit này được gán 0.  
    Nếu giá trị đó nằm ngoài khoảng ban đầu, thì bit này được gán là 1.  
    Nếu bit này là 0, ta sẽ tiếp tục ở bước **2.**, nếu bằng 1, ta sẽ tiếp tục ở bước **5.**.  
    Khi biểu diễn số nguyên, ta mới chỉ cần để ý đến cận trên, cận dưới của nó mà thôi. Cho dù khi sử dụng **dấu mở rộng** thì nếu giá trị thực nằm trong khoảng cận trên, cận dưới thì giá trị đó sẽ được biểu diễn giống với trường hợp không có dấu **mở rộng**.

*   Nếu dựa vào một điều kiện nào đó, ta chỉ có 1 giá trị thôi, thì giá trị này sẽ không được biểu diễn nữa (tất nhiên cả **L** và **V** đều bị khuyết). Biểu diễn chỉ đến đây là hết.
    
*   Nếu định nghĩa miêu tả một giới hạn hữu hạn, giá trị sẽ được biểu diễn giống với số nguyên hữu hạn miêu tả ở trên. Biểu diễn chỉ đến đây là hết.
    
*   Nếu chỉ có giới hạn dưới được định nghĩa, giá trị sẽ được biểu diễn theo dạng Khoảng số nguyên giới hạn ở trên. Biểu diễn chỉ đến đây là hết.
    
*   Biểu diễn một số nguyên không có giới hạn. Biểu diễn cũng kết thúc ở đây.
    

Các trường hợp được tóm tắt dưới bảng sau:

**

### 3.3 Kiểu liệt kê (ENUMERATED)

Kiểu liệt kê được biểu diễn theo cách được mô tả dưới đây.

1.  Nếu có dấu mở rộng, _Preamble_ là 1 bit đầu tiên. Nếu giá trị cần biểu diễn nằm trong khoảng biểu diễn ban đầu, thì bit này sẽ có giá trị 0. Ngược lại, bit này sẽ có giá trị là 0. Chi tiết các biểu diễn được miêu tả ở bước 2 nếu là 0, ở bước 3 nếu là 1.

**

*   Các giá trị trong kiểu liệt kê sẽ luôn được tính từ 0 (nếu ban đầu không bắt đầu từ 0, thì sẽ được đánh số lại). Kiểu liệt kê dạng này sẽ được biểu diễn giống với Số nguyên hữu hạn.
    
*   Sau khi loại bỏ khoảng giá trị khi chưa mở rộng, ta sẽ đánh số lại từ 0. Phần giá trị cần biểu diễn thêm sẽ được biểu diễn như là Số nguyên không âm nhỏ.
    

Các bước được tóm tắt dưới bảng sau:

**

### 3.4 Kiểu số thực (REAL)

Được biểu diễn gần như giống với **CER/DER**.  
Đầu tiên là biểu diễn độ dài, sau đó là giá trị biểu diễn ở dạng **CER/DER**. Giá trị này sẽ được biểu diễn theo đơn vị **OCTET** hoặc **BIT** phụ thuộc vào **ALIGNGED** hay **UNALIGNED** hay không.

### 3.5 Kiểu dãy BIT

Khi biểu diễn, những điều kiện sau đây sẽ được tính đến.

*   Kích thước: Giới hạn về đọ dài. Cũng giống với kiểu số nguyên, giá trị về mặt độ dài có thể thay đổi tùy theo định nghĩa độ dài cố định, hoặc định nghĩa khoảng độ dài.  
    Ví dụ

```cpp  
Exactly31BitsString ::= BIT STRING (SIZE (31))  
StringOf31BitsAtTheMost ::= BIT STRING (SIZE (0..31))  
```

*   Giới hạn được tạo bằng các phép tập hợp như phép giao, phép hợp, phép loại trừ, phép loại trừ đảo.

**

*   Sử dụng dấu mở rộng cho mục đích có thể sử dụng trong tương lai.
    

Biểu diễn dãy BIT được mô tả theo các bước như sau:

1.  Nếu là một dãy bit đã được định nghĩa **TÊN** thì các bit không sử dụng phía sau sẽ bị loại bỏ.

*   Nếu có giới hạn về kích thước, trong phạm vi thỏa mãn giới hạn, các giá trị luôn được biểu diễn sao cho có độ dài BIT là nhỏ nhất.  
    Có thể thêm các bit 0, hoặc xóa bit.
    
*   Nếu trong giới hạn kích thước có **dấu mở rộng**, thì cũng tương tự các trường hợp khác. Bit đầu được sử dụng là _Preamble_. Nếu giá trị cần biểu diễn nằm trong phạm vi ban đầu, thì bit này giá trị bằng 0. Ngược lại, bit đó sẽ có giá trị bằng 1 (để mô tả đây là 1 giá trị mở rộng).  
    Với trường hợp _Preamble_ bằng 0, ta đến tiếp bước 4. Còn lại ta đến bước 7.
    
*   Nếu giới hạn trên của kích thước là 0. Thì không cần biểu diễn giá trị này. Kết thúc quá trình biểu diễn.
    
*   Nếu kích thước là cố định (cận trên = cận dưới), và nhỏ hơn hoặc bằng 16, thì không cần biểu diễn độ dài nữa. Biểu diễn luôn toàn bộ giá trị với độ dài cố định đó. Không biểu diễn bằng đơn vị Octet. Kết thúc biểu diễn cho giá trị này ở đây.
    
*   Nếu kích thước là cố định, và >=17, <= 65536. Cũng không cần biểu diễn độ dài, mà biểu diễn thẳng dãy bit đó luôn. Trường hợp **ALIGNED** có thể biểu diễn theo đơn vị Octet.
    
*   Đầu tiên biểu diễn độ dài dãy bit. Nếu giới hạn trên của kích thước hữu hạn thì sử dụng số nguyên hữu hạn. Nếu không có cận trên, thì biểu diễn số nguyên giới hạn bộ phận. Sau đó mới đến dãy bit miêu tả độ dài.
    

Các trường hợp biểu diễn được tóm tắt dưới bảng sau:

**

### 3.6 Kiểu OCTET (OCTET STRING)

Ta cần quan tâm đến các thông số dưới đây:  
- Kích thước: Giới hạn về độ dài.  
- Các phép toán tập hợp dùng để giới hạn: Phép hợp, giao, loại trừ, loại trừ đảo.  
- Dấu mở rộng: Miêu tả khả năng sử dụng trong tương lai.

Biểu diễn một giá trị kiểu dãy **Octet** được thực hiện theo các bước dưới đây (về cơ bản, ngoài việc khác nhau về đơn vị ra thì gần như giống hệt kiểu dãy **Bit**.

1.  Nếu kích thước bị giới hạn, có sử dụng dấu mở rộng, thì 1 bit đầu được sử dụng là _Preamble_. Nếu giá trị nằm trong khoảng ban đầu (chưa mở rộng), thì giá trị _Preamble_ là 0, ngược lại sẽ là 1. Bằng 0 thì đến tiếp bước 2, bằng 1 đến bước 5.

**

*   Nếu kích thước là 0, thì chẳng có gì để biểu diễn cho giá trị này. Kết thúc biểu diễn của giá trị đó luôn.
    
*   Nếu kích thước cố định (cận trên = cận dưới), và =2, và = 65537 thì cách biểu diễn sẽ khác đi một chút, tuy nhiên do rất ít xảy ra nên không được nói đến ở đây.
    
*   Tiếp theo, từng giá trị sẽ được biểu diễn. Các yếu tốt **OPTIONAL**, **DEFAULT** mà không tồn tại sẽ bị bỏ qua.
    
*   Ở dạng biểu diễn chính quy (CER), khi yếu tố có nhãn **DEFAULT** nhận chính giá trị mặc định đó thì sẽ không được biểu diễn. Dạng biểu diễn cơ bản **BER** cũng thế. Trường hợp kiểu cấu trúc, có biểu diễn hay không phụ thuộc hoàn toàn vào Encoder.
    
*   Trường hợp không có dấu mở rộng, đến đây ra kết thúc biểu diễn.
    
*   Số dấu mở rộng, gọi là N-ext sẽ được biểu diễn bằng **Số nguyên nhỏ không âm thông thường**
    
*   Tiếp theo là N-ext bit để mô tả giá trị cần biểu diễn là mở rộng hay không mở rộng.
    
*   Tiếp theo, từng giá trị sẽ được biểu diễn tương ứng. Dãy giá trị biểu diễn của từng yếu tố sẽ được coi là giá trị của kiểu Mở.
    

### 3.9 Kiểu mảng.

Khi biểu diễn giá trị kiểu mảng, ta cần xem xét các yếu tố sau:

*   Kích thước: Giới hạn độ dài  
    Ví dụ:

```cpp  
ListOf5Strings ::= SEQUENCE (SIZE (5)) OF PrintableString  
ListOf5StringsOf5Characters ::=SEQUENCE (SIZE (5)) OF PrintableString (SIZE (5))  
```

*   Kết quả của cá phép tập hợp: hợp, giao, loại trừ
*   Dấu mở rộng: chỉ định các vị trí có thể sử dụng trong tương lai.

1.  Nếu kích thước có thể mở rộng, thì bit đầu tiên được sử dụng như là _Preamble_ (không làm tròn thành byte cho bit này). Như biểu diễn các kiểu khác, nếu giá trị kích thước cần biểu diễn nằm trong khoảng ban đầu, thi giá trị _Preamble_ bằng 0, sau đó đến bước 2. Ngược lại sẽ bằng 1, và biểu diễn số phần tử bằng kiểu *Số nguyên giới hạn bộ phận, rồi cuối cùng là biểu diễn của các phần tử.

*   Nếu kích thước chỉ là 1 giá trị cố định, và <= 65536, thì giá trị kích thước không được biểu diễn nữa. Chỉ n phần tử được biểu diễn thôi, rồi kết thúc.
    
*   Số phần tử sẽ được biểu diễn bằng kiểu số nguyên giới hạn trong trường hợp có giới hạn trên hữu hạn, được biểu diễn bằng kiểu giới hạn bộ phận khi không cận trên. Cuối cùng là các phần tử.
    

**

### 3.10 Kiểu tập hợp (SET)

Các yếu tố sau cần xem xét khi biểu diễn kiểu này.

*   Kích thước: giới hạn độ dài
*   Các phép tập hợp: phép hợp, giao, loại trừ, loại trừ đảo.
*   Dấu mở rộng: cho mục đích tương lai.

Trong biểu diễn chính quy (CER), các phần tử của SET được biểu diễn theo thứ tự tăng dần của số BIT của mỗi phần tử.  
Biểu của SET được biểu diễn giống với **SEQUENCE OF**.

### 3.12 Kiểu lựa chọn

1.  Nếu có sử dụng dấu mở rộng, bit đầu tiên được sử dụng là _Preamble_. Nếu giá trị lựa chọn nằm trong số các giá trị ban đầu, thì _Preamble_ sẽ bằng 0, ngược lại bằng 1.

**

*   Trường hợp kiểu có nhiều lựa chọn, index sẽ được biểu diễn trong khoảng giá trị từ 0 đến số lựa chọn - 1. Đối với các lựa chọn nằm trong khoảng ban đầu (chứ không phải đã mở rộng), xếp theo thứ tự của tag, index sẽ được đánh từ 0. Thứ tự giữa các class sẽ như sau Lớp chung (**UNIVERSAL**), lớp ứng dụng (**APPLICATION**), lớp đặc trưng ngữ cảnh, lớp sử dụng riên (**PRIVATE**). Với các thẻ cùng 1 lớp, thì thẻ số nhỏ sẽ đứng trước, thẻ số lớn hơn xếp sau. Đối với kiểu mà không có thẻ tag, thì thứ tự được sếp theo tên của từng thành phần, sau đó index được đánh dấu vẫn từ 0. Giá trị index sẽ được biểu diễn là số nguyên nếu nó là giá trị chưa mở rộng, hoặc được biểu diễn là số nguyên nhỏ không âm nếu là một giá trị đã mở rộng.  
    Nếu chỉ có 1 lựa chọn, thì index không cần biểu diễn.
    
*   Cuối cùng, từng thành phần sẽ được biểu diễn. Các giá trị trước mở rộng sẽ được biểu diễn bình thường, giá trị mở rộng sẽ biểu diễn thành giá trị kiểu OPEN.
    

**

### 3.13 Kiểu chỉ định đối tượng (OBJECT IDENTIFIER)

Đầu tiên là số **Octet** sau đó là biểu diễn giá trị bằng **BER**.

### 3.14 Kiểu kí tự hạn chế

Xét đến một số yếu tố sau:

*   Kế thừa từ kiểu Alphabet

```cpp  
Morse ::= PrintableString (FROM ("."|"-"|" "))  
IDCardNumber ::=NumericString (FROM ("0".."9"))  
PushButtonDialSequence ::=IA5String (FROM ("0".."9"|"*"|"#"))  
```

*   Kích thước: giới hạn độ dài
*   Phép chứa trong : Kết hợp từ những kiểu khác nhau.

```cpp  
LongWeekEnd ::= Day (WeekEnd|monday)  
```

*   Các phép tập hơn: giao, hợp, loại trừ, loại trừ đảo
*   Dấu mở rộng: dấu mở rộng trong sử dụng trong tương lai.

Quá trình sẽ được thực hiện theo các bước sau:

1.  Về cơ bản, sau khi biểu diễn số kí tự, thì từng kí tự sẽ được biểu diễn.
    
2.  Mỗi kí tự sẽ được biểu diễn bằng số bit ít nhất. Trường hợp **UNALIGNED**, giả sử có N kí tự cần biểu diễn thì sẽ cần Log2(N) bit để biểu diễn. Trường hợp **ALIGNED** bằng đơn vị Octet, thì sẽ cần bit là lũy thừa của 2 nhỏ nhất lớn hơn Log2(N). Thứ tự mỗi kí tự sẽ được sếp theo thứ tự của chính giá trị của nó. Bắt đầu từ 0. Trường hợp biểu diễn cho giá trị mở rộng, thì những giá trị chưa nằm trong dãy mở rộng sẽ được biểu diễn giống với lúc trước mở rộng, coi như không bị ảnh hưởng gì từ dấu mở rộng.
    
3.  Nếu có dấu mở rộng, thì giống như những kiểu khác ở trên. Bit đầu được sử dụng là _Preamble_. Giá trị cần biểu diễn thuộc khoảng ban đầu, thì _Preamble_ bằng 0, sau đó ta đến bước 4. Ngược lại sẽ được gán là 1, thì ta đến tiếp bước 6.
    
4.  Nếu kích thước là cố định, chuỗi biểu diễn có độ dài dưới 2 Octet. Thì biểu diễn độ dài được bỏ qua, ta chỉ biểu diễn giá trị ở dạng không làm tròn thành Octet rồi kết thúc.
    
5.  Nếu kích thước cố định, và số kí tự cần biểu diễn >=2, <= 65536 thì cũng vẫn biểu diễn giống bước 4 nhưng được phép làm tròn thành Octet trong trường hợp **ALIGNED**.
    
6.  Độ dài dãy kí tự được biểu diễn theo kiểu số nguyên giới hạn (nếu có cận trên), hoặc theo kiểu số nguyên bộ phận (nếu không có cận trên), tiếp theo đó là dãy kí tự.
    

**