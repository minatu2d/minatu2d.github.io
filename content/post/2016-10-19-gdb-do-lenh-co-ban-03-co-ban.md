---
layout: post
title: GDB dòng lệnh cơ bản (03 - Cơ bản)
date: 2016-10-19 14:27:19.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
- Debug
tags:
- C
- GDB
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: '43452'
  _publicize_job_id: '28010785840'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/10/19/gdb-do-lenh-co-ban-03-co-ban/"
---
Trong bài số [02](https://lazytrick.wordpress.com/2016/09/19/gdb-co-the-lam-gi/), ta đã nói đến những việc mà GDB có thể giúp chúng ta.

Về cơ bản GDB, có thể chạy để debug mọi chương trình, tuy nhiên nếu không muốn càng debug càng rối thì ta nên sử dụng tham số **-g** khi biên dịch để giúp quá trình debug xác định được vị trí mỗi đoạn binary trong source ban đầu.  
  
## 0. Source code
Trong bài này, ta hãy cùng xem cách sử dụng thực tế sẽ như thế nào.  
Giả sử ta có một source Hello world như sau:

```bash  
.  
├── include  
│   ├── greetings_en.h  
│   ├── greetings_ja.h  
│   └── greetings_vi.h  
└── src  
├── greetings_en.c  
├── greetings_ja.c  
├── greetings_vi.c  
├── mainapp  
└── mainapp.c  
```

Trong đó, các file có nội dung như sau:

```cpp  
// mainapp.c

#include  
#include"greetings_en.h"  
#include"greetings_ja.h"  
#include"greetings_vi.h"

//  
// Main functions  
//  
int main(int argc, char* argv[])  
{  
greeting_en();  
greeting_ja();  
greeting_vi();

return 0;  
}

// greetings_en.h  
void greeting_en();

// greetings_ja.h  
void greeting_ja();

// greetings_vi.h  
void greeting_vi();

// greetings_en.c  
#include  
void greeting_en()  
{  
printf("Hello world\n");  
}

// greetings_ja.c  
#include  
void greeting_ja()  
{  
printf("Konnichiwa \n");  
}

// greetings_vi.c  
#include  
void greeting_vi()  
{  
printf("Xin chao \n");  
}  
```

Lệnh biên dịch từ **src**:

```bash  
oedev@OEDEV-Ubuntu:~/src$gcc -g -o mainapp mainapp.c -I../include greetings_*.c  
```

Chú ý, để sử dụng GDB tốt nhất, ta nên thêm tham số **-g** (viết thường) như ở trên.  
Kết quả sau khi chạy sẽ như thế này:

```bash  
oedev@OEDEV-Ubuntu:~/src$./mainapp  
Hello world  
Konnichiwa  
Xin chao  
```

OK, đó là 1 chương trình đơn giản để làm ví dụ các phần bên dưới đây.

## 1. Chạy chương trình với GDB

-Ta chỉ cần thêm **gdb** vào đầu câu lệnh thông thường, ta sẽ thấy GDB được khởi động với giao diện sau:

```bash  
oedev@OEDEV-Ubuntu:~/src$gdb ./mainapp  
GNU gdb (Ubuntu 7.11.1-0ubuntu1~16.04) 7.11.1  
Copyright (C) 2016 Free Software Foundation, Inc.  
License GPLv3+: GNU GPL version 3 or later  
This is free software: you are free to change and redistribute it.  
There is NO WARRANTY, to the extent permitted by law. Type "show copying"  
and "show warranty" for details.  
This GDB was configured as "x86_64-linux-gnu".  
Type "show configuration" for configuration details.  
For bug reporting instructions, please see:  
.  
Find the GDB manual and other documentation resources online at:  
.  
For help, type "help".  
Type "apropos word" to search for commands related to "word"...  
Reading symbols from ./mainapp...done.  
(gdb)  
```

Ta thấy giao diện giao tiếp bằng dòng lệnh hiện ra, chương trình chính vẫn chưa được chạy.  
Đến đây, hầu hết là thông tin về phiên bản, rồi cấu hình của GBD thôi. Nhưng có 1 dòng ta sẽ để ý thấy.  
Đó là :

**_Reading symbols from ./mainapp...done._**

Nếu ta biên dịch mà không có **_-g_** (như đã nói ở trên), ta sẽ nhận một dòng khác như sau:

**_Reading symbols from ./mainapp...(no debugging symbols found)...done._**

Hiểu nôm na là, khi có **_-g_**, GDB sẽ load được các symbol (là tên các hàm sử dụng trong ứng dụng), còn không có thì nó không thể load được.

Rồi, ta mới chỉ thấy được rằng, GDB đã thực hiện xong các bước kiểm tra các symbol có trong file chạy và load các source code trong đường dẫn nó biết.

-Giờ ta bắt đầu chạy, nếu muốn GDB dừng lại ở hàm **main()**thì hãy dùng lệnh **start**, còn muốn chạy luôn không cần dừng thì hãy dùng **run**(viết ngắn là **_r_**).  
Khi ***start**:

```bash  
(gdb) start  
Temporary breakpoint 1 at 0x400535: file mainapp.c, line 11.  
Starting program: /home/oedev/code/debug_03/src/mainapp

Temporary breakpoint 1, main (argc=1, argv=0x7fffffffdcb8) at mainapp.c:11  
11 greeting_en();  
(gdb)  
```

## 2. Breakpoint

-Ta chỉ có thể đặt khi chương trình chính không chạy (chưa chạy hoặc đang bị paused).  
Mỗi breakpoint sẽ gắn với dòng tại 1 file nào đó.  
Về cơ bản, ta sử dụng lệnh **break** (viết ngắn **_b_**) để đặt breakpoint, đưa 2 tham số là tên file và chỉ số dòng vào là ok.  
Ví dụ, đặt breakpoint ở dòng 13 của **mainapp.c**:

```bash  
(gdb) b mainapp.c:13  
Breakpoint 2 at 0x400549: file mainapp.c, line 13.  
(gdb)  
```

GDB cũng hỗ trợ đặt break theo tên hàm nữa.  
Ví dụ, đặt breakpoint ở hàm **greeting_en()** trong file **greetings_en.c**:

```bash  
(gdb) b greetings_en.c:greeting_en  
Breakpoint 3 at 0x40055e: file greetings_en.c, line 4.  
```

-Xem danh sách các breakpoint bằng **info break**:

```bash  
(gdb) info break  
Num Type Disp Enb Address What  
2 breakpoint keep y 0x0000000000400549 in main at mainapp.c:13  
3 breakpoint keep y 0x000000000040055e in greeting_en at greetings_en.c:4  
```

Chú ý là, breakpoint đầu tiên ở đầu hàm main (do lệnh **start**) không được tính.

-Xóa một breakpoint  
Ta cũng có thể dùng cách tương tự như đặt chỉ thay command **break** bằng **clear** là ok.  
Nhưng có một cách đơn giản hơn.  
Ta để ý thấy mỗi khi đặt một breakpoint mới, ta thấy các dòng kết quả:

```bash  
Breakpoint 1..  
Breakpoint 2..  
```

Ta có thể xóa theo các tham số đó bằng lệnh **del**:

```bash  
(gdb) info break  
Num Type Disp Enb Address What  
2 breakpoint keep y 0x0000000000400549 in main at mainapp.c:13  
3 breakpoint keep y 0x000000000040055e in greeting_en at greetings_en.c:4  
(gdb) del 3  
(gdb) info break  
Num Type Disp Enb Address What  
2 breakpoint keep y 0x0000000000400549 in main at mainapp.c:13  
```

Dù xóa breakpoint 3 đi thì số 3 đó cũng không được sử dụng lại để đánh số cho các breakpoint được đánh dấu sau này (trừ khi khởi động lại GDB).

## 3. Các thao tác debug khi chương trình dừng tại breakpoint

Sau khi đặt breakpoint, ta tiến hành chạy chương trình.  
Vì chương trình đang dừng (paused), ta dùng lệnh **continue** (viết ngắn là **_c_**) để chạy tiếp. Chương trình sẽ dừng lại khi gặp breakpoint.  
Sau khi dừng lại, nó sẽ có dạng như sau:

```bash  
(gdb) c  
Continuing.  
Hello world  
Konnichiwa

Breakpoint 2, main (argc=1, argv=0x7fffffffdcb8) at mainapp.c:13  
13 greeting_vi();  
(gdb)  
```

-Kiểm tra xem chương trình xem đã bị dừng ở đâu  
Thực ra nhìn vào trạng thái dừng ở trên ta cũng biết được rằng, chương trình đang dừng ở **mainapp.c**, dòng 13, chỗ lời gọi hàm **greeting_vi()**.  
Có một cách trực quan hơn là lệnh **layout** (viết ngắn là **_l_**).

```bash  
(gdb) l  
8 //  
9 int main(int argc, char* argv[])  
10 {  
11 greeting_en();  
12 greeting_ja();  
13 greeting_vi();  
14  
15 return 0;  
16 }  
```

GDB sẽ hiển thị cả đoạn source xung quanh điểm đang bị dừng (còn giao diện trực quan hơn nữa sẽ được giới thiệu ở bài sau),

-Thực hiện dòng hiện tại (không nhảy sâu vào hàm nếu đang gọi hàm):  
Lệnh **next** (viết ngắn là **_n_**)

```bash  
(gdb) n  
Xin chao  
15 return 0;  
```

GDB sẽ dừng lại ở dòng tiếp theo, dòng 15 như ở trên (vì 14 không có gì)

-Thực hiện gọi sâu vào hàm  
Giả sử đang dừng ở lời gọi **greeting_vi()**,  
Sử dụng **step**, GDB sẽ chạy vào bên trong **greeting_vi()** thay vì thực hiện ngay toàn bộ xử lý trong nó giống như **next**.

```bash  
(gdb) step  
greeting_vi () at greetings_vi.c:4  
4 printf("Xin chao \n");  
(gdb) l  
1 #include  
2 void greeting_vi()  
3 {  
4 printf("Xin chao \n");  
5  
```

## 4. Xem giá trị khi chương trình dừng tại breakpoint

Trước khi đi vào phần này, ta sửa lại nội dung file **mainapp.c** ở trên thành như sau:

```cpp  
#include  
#include  
#include  
#include"greetings_en.h"  
#include"greetings_ja.h"  
#include"greetings_vi.h"

typedef struct _currency_  
{  
char iso4217Code[5];  
char name[255];  
}ST_CURRENCY;

typedef struct _country_  
{  
char capital[255];  
float area;  
uint32_t population;  
float density;  
ST_CURRENCY currency;  
int16_t timezone;  
char dateFormat[100];  
}ST_COUNTRY;

void printCountry(void *data)  
{  
ST_COUNTRY *country = (ST_COUNTRY*)data;

printf("Capital :%s\n",country->capital);  
printf("Area :%.2f km2\n",country->area);  
printf("Density :%.2f /km2\n",country->density);  
printf("Currency : %s (%s) \n",country->currency.iso4217Code, country->currency.name);  
printf("Timezone : GMT + %d \n",country->timezone);  
printf("Dateformat :%s\n",country->dateFormat);  
}  
//  
// Main functions  
//  
int main(int argc, char* argv[])  
{  
greeting_en();  
greeting_ja();  
greeting_vi();

/*Vietnam*/  
ST_COUNTRY vietnam;  
memset(&vietnam, 0x00, sizeof(england));

strcpy(vietnam.capital,"Hanoi");  
vietnam.area = 332698; // km2  
vietnam.population = 91700000;  
strcpy(vietnam.currency.iso4217Code,"VND");  
strcpy(vietnam.currency.name,"Dong");  
vietnam.timezone = +7;  
vietnam.density = 276.03;  
strcpy(vietnam.dateFormat,"dd/mm/yyyy");

printCountry(&vietnam);

return 0;  
}

```

Ta tiến hành chạy với GDB như trên và dừng ở lời gọi hàm **printCountry(&vietnam)**

-Sử dụng print (viết ngắn là p)  
Ta lần lượt sẽ hiển thị giá trị nguyên, thực, chuỗi, cấu trúc như sau:

```bash  
Breakpoint 1, main (argc=1, argv=0x7fffffffdcb8) at mainapp.c:58  
58 printCountry(&vietnam);  
(gdb) l  
53 strcpy(vietnam.currency.name,"Dong");  
54 vietnam.timezone = +7;  
55 vietnam.density = 276.03;  
56 strcpy(vietnam.dateFormat,"dd/mm/yyyy");  
57  
58 printCountry(&vietnam);  
59  
60 return 0;  
61 }  
(gdb) p vietnam.population  
$1 = 91700000  
(gdb) p vietnam.density  
$2 = 276.029999  
(gdb) p vietnam.capital  
$3 = "Hanoi", '000'  
(gdb) p vietnam  
$4 = {capital = "Hanoi", '000' , area = 332698,  
population = 91700000, density = 276.029999, currency = {  
iso4217Code = "VND000", name = "Dong", '000' },  
timezone = 7, dateFormat = "dd/mm/yyyy", '000' }  
(gdb)  
```

-Hiển thị giá trị biến cấu trúc đẹp hơn  
Bật chế độ in đẹp bằng **set print pretty on**, rồi thực hiện in như bình thường.

```bash  
(gdb) p vietnam  
$4 = {capital = "Hanoi", '000' , area = 332698,  
population = 91700000, density = 276.029999, currency = {  
iso4217Code = "VND000", name = "Dong", '000' },  
timezone = 7, dateFormat = "dd/mm/yyyy", '000' }  
(gdb) set print pretty on  
(gdb) p vietnam  
$5 = {  
capital = "Hanoi", '000' ,  
area = 332698,  
population = 91700000,  
density = 276.029999,  
currency = {  
iso4217Code = "VND000",  
name = "Dong", '000'  
},  
timezone = 7,  
dateFormat = "dd/mm/yyyy", '000'  
}  
```

Ta có thể thấy rõ sự khác bọt giữa khi pretty là off/on.

-Hiển thị giá trị con trỏ có ép kiểu  
Đặt breakpoint ở hàm **printCountry**, sau khi dừng ta sẽ có như sau:

```bash  
Breakpoint 2, printCountry (data=0x7fffffffd950) at mainapp.c:27  
27 ST_COUNTRY *country = (ST_COUNTRY*)data;  
(gdb) p data  
$6 = (void *) 0x7fffffffd950  
```

Tham số **data** được truyền vào là kiểu void. Nên khi ta dùng lệnh print thì chỉ thấy được địa chỉ con trỏ thôi. Ta có thể ép kiểu bất cứ kiểu nào.  
Nhưng vì biết đó là kiểu con trỏ của **ST_COUNTRY** rồi, ta có thể làm như sau:

```bash  
(gdb) p (*(ST_COUNTRY*)data)  
$7 = {  
capital = "Hanoi", '000' ,  
area = 332698,  
population = 91700000,  
density = 276.029999,  
currency = {  
iso4217Code = "VND000",  
name = "Dong", '000'  
},  
timezone = 7,  
dateFormat = "dd/mm/yyyy", '000'  
}  
```

-Hiển thị mảng  
Ở trên ta đã thấy chuỗi được hiển thị rồi, chuỗi cũng là mảng nên tả hãy thử hiện thị độ dài bất kì của mảng đó xem sao, cách làm phổ biến như sau:

```bash  
(gdb) x/10xb country->capital  
0x7fffffffd950: 0x48 0x61 0x6e 0x6f 0x69 0x00 0x00 0x00  
0x7fffffffd958: 0x00 0x00  
```

Câu lệnh trên sẽ hiển thị **10** bytes (**b** cuối cùng là byte) đầu tiên của mảng **country->capital**, ở dang cơ số 16(chính là **x** sau số 10).

## 5. Kiểm tra stacktrace

-backtrace (viết ngắn là bt) được sử dụng để cho biết luồng xử lý đi như thế nào để đến được điểm dang dừng lại, ta có thể sử dụng thêm **bt full** để thấy các lời gọi ở vị trí nào trong code.

```bash  
(gdb) bt  
#0 printCountry (data=0x7fffffffd950) at mainapp.c:29  
#1 0x00000000004007ef in main (argc=1, argv=0x7fffffffdcb8) at mainapp.c:58  
(gdb) bt full  
#0 printCountry (data=0x7fffffffd950) at mainapp.c:29  
country = 0x7fffffffd950  
#1 0x00000000004007ef in main (argc=1, argv=0x7fffffffdcb8) at mainapp.c:58  
vietnam = {  
capital = "Hanoi", '000' ,  
area = 332698,  
population = 91700000,  
density = 276.029999,  
currency = {  
iso4217Code = "VND000",  
name = "Dong", '000'  
},  
timezone = 7,  
dateFormat = "dd/mm/yyyy", '000'  
}  
```