---
layout: post
title: '"Data structure alignment" là gì? Tại sao phải hiểu nó khi code C.'
date: 2016-10-26 06:50:13.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
- Compile
tags:
- Aligment
- C
- Casting
- Pointer
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '28242666696'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/10/26/data-structure-alignment-la-gi-tai-sao-phai-hieu-no-khi-code-c/"
---
Đây là một chủ đề hay, có rất nhiều resource bằng tiếng Việt khá dễ hiểu rồi.  
Bài này chỉ mô tả ngắn gọn một chút cùng với vài ví dụ thực nghiệm để hiểu các khái niệm về cơ bản về **Data Structure Aligment**.

Giả sử ta có một cấu trúc sau:

```C  
#include <stdint.h>  
typedef struct  
{  
uin8_t mem1;  
uin8_t mem2;  
uin32_t mem3;  
}ST_FOOL_1;  
typedef struct  
{  
uin8_t mem1;  
uin8_t mem2;  
uin8_t mem3;  
uin8_t mem4;  
uin8_t mem5;  
}ST_FOOL_2;  
```

Cấu trúc **ST_FOOL_1** sẽ được miêu tả trong 6 byte?  
Cấu trúc **ST_FOOL_2** sẽ được miêu tả trong 5 byte?  
Tùy theo tham số trong quá trình "Data structure alignment", mà cấu trúc trên sẽ được chứa trong số bộ nhớ cần thiết khác nhau.

#### 1. Về Sizeof

Khi có thay đổi về "data structure aligment" (ta tạm gọi là kích thước để nhóm), kết quả của **sizeof()** sẽ bị thay được phản ánh, và ngược lại từ kết quả của **sizeof()** ta sẽ đoán được trình biên dịch đã sử kích thước bao nhiêu.

Ta có ví dụ như sau:

```C  
#include<stdio.h>  
#include<stdint.h>  
typedef struct  
{  
uint8_t mem1;  
uint8_t mem2;  
uint32_t mem6;  
}ST_FOOL_1;

typedef struct  
{  
uint8_t mem1;  
uint8_t mem2;  
uint8_t mem3;  
uint8_t mem4;  
uint8_t mem5;  
}ST_FOOL_2;

int main()  
{  
printf("Sizeof(ST_FOOL_1) =%ld\n",sizeof(ST_FOOL_1));  
printf("Sizeof(ST_FOOL_2) =%ld\n",sizeof(ST_FOOL_2));

return 0;  
}  
```

Ta lưu vào file **sizeof_result.c**, sau đó biên dịch và chạy được kết quả sau:

```bash  
$gcc -o sizeof_result sizeof_result.c  
$./sizeof_result  
Sizeof(ST_FOOL_1) =8  
Sizeof(ST_FOOL_2) =5  
```

=> **Sizeof(ST_FOOL_1)** = 8, vậy GCC đã sử dụng Block 4 byte để "aligment" cho cấu trúc rồi.  
Vậy nếu Block 4 bytes, thì tại sao ST_FOOL_2 không nhảy lên 8 byte?  
Câu trả lời là khi và chỉ khi trình biên dịch không thể chứa member tiếp theo, nó sẽ bỏ qua một vài byte trống để nhảy đến Block 4 byte tiếp theo.  
Kích thước sẽ được tính đến bị trí mà member cuối cùng với tới.

Giờ ta thay đổi Block 4 bytes thành các giá trị khác để xem sự thay đổi của **sizeof()** như thế nào.

Ta sẽ sử dụng chỉ thị dịch **pack** (chi tiết ở [đây](https://gcc.gnu.org/onlinedocs/gcc-4.4.4/gcc/Structure_002dPacking-Pragmas.html) :

Cách sử dụng:

```C  
#include<stdio.h>  
#include<stdint.h>

#pragma pack(1) // phải là lũy thừa của 2, tức là 1,2,4,8..  
typedef struct  
{  
uint8_t mem1;  
uint8_t mem2;  
uint32_t mem6;  
}ST_FOOL_1;

typedef struct  
{  
uint8_t mem1;  
uint8_t mem2;  
uint8_t mem3;  
uint8_t mem4;  
uint8_t mem5;  
}ST_FOOL_2;

int main()  
{  
printf("Sizeof(ST_FOOL_1) =%ld\n",sizeof(ST_FOOL_1));  
printf("Sizeof(ST_FOOL_2) =%ld\n",sizeof(ST_FOOL_2));

return 0;  
}  
```

Ta sẽ có kết quả như sau:  
- **pack(1)**:

```bash  
$./sizeof_result  
Sizeof(ST_FOOL_1) =6  
Sizeof(ST_FOOL_2) =5  
```  
- **pack(2)**:  
```bash  
$./sizeof_result  
Sizeof(ST_FOOL_1) =6  
Sizeof(ST_FOOL_2) =5  
```  
- **pack(4)**:  
```bash  
$./sizeof_result  
Sizeof(ST_FOOL_1) =8  
Sizeof(ST_FOOL_2) =5  
```

#### 2. Khi sử dụng Casting con trỏ

Sử dụng casting pointer giữa mảng và cấu trúc là một cách để convert (chuyển đổi) nội dung giữa 1 biến cấu trúc và mảng.

Giả sử một ứng dụng truyền nhận sử dụng command chẳng hạn, bên gửi sẽ chuyển command sang mảng byte, rồi gửi đi. Phía nhận sẽ làm ngược lại.

```C  
uint8_t arrData[100];

ST_FOOL *pFool = (ST_FOOL*)&arrData[0];

```

Trở lại với ví dụ ở phía trên, giả sử ta có cấu trúc command giống như **ST_FOOL_1**, **ST_FOOL_2** đã định nghĩa ở trên.

Với source như sau:

```C  
#include<stdio.h>  
#include<stdint.h>  
#include<string.h>

typedef struct  
{  
uint8_t mem1;  
uint8_t mem2;  
uint32_t mem6;  
}ST_FOOL_1;

typedef struct  
{  
uint8_t mem1;  
uint8_t mem2;  
uint8_t mem3;  
uint8_t mem4;  
uint8_t mem5;  
}ST_FOOL_2;

int main()  
{  
int i = 0;

printf("====Structure --> Array====\n");  
uint8_t outputArr[16];

memset(outputArr, 0x00, sizeof(outputArr));  
ST_FOOL_1 *pFool_1 = (ST_FOOL_1*)&outputArr[0];  
pFool_1->mem1 = 0x12;  
pFool_1->mem2 = 0x34;  
pFool_1->mem6 = 0x56789AB;

printf("pFool_1 :\n");  
printf(".mem1 = 0x%02X\n",pFool_1->mem1);  
printf(".mem2 = 0x%02X\n",pFool_1->mem2);  
printf(".mem6 = 0x%08X\n",pFool_1->mem6);

printf("Output array(hex) :\n");  
for (i = 0; i < sizeof(ST_FOOL_1); i ++)  
{  
printf ("%02X ",outputArr[i]);  
}  
printf("\n\n");

memset(outputArr, 0x00, sizeof(outputArr));  
ST_FOOL_2 *pFool_2 = (ST_FOOL_2*)&outputArr[0];  
pFool_2->mem1 = 0x12;  
pFool_2->mem2 = 0x34;  
pFool_2->mem3 = 0x56;  
pFool_2->mem4 = 0x78;  
pFool_2->mem5 = 0x90;

printf("pFool_2 :\n");  
printf(".mem1 = 0x%02X\n",pFool_2->mem1);  
printf(".mem2 = 0x%02X\n",pFool_2->mem2);  
printf(".mem3 = 0x%02X\n",pFool_2->mem3);  
printf(".mem4 = 0x%02X\n",pFool_2->mem4);  
printf(".mem5 = 0x%02X\n",pFool_2->mem5);

printf("Output array (hex):\n");  
for (i = 0; i < sizeof(ST_FOOL_2); i ++)  
{  
printf ("%02X ",outputArr[i]);  
}  
printf("\n\n");

printf("====Array --> Structure===\n");  
uint8_t inputArr[16] = {  
0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08,  
0x09, 0x10, 0x11, 0x12, 0x13, 0x14, 0x15, 0x16  
};

printf("Input array:\n");  
for (i = 0; i < 16; i ++)  
{  
printf ("%02X ",inputArr[i]);  
}  
printf("\n");

ST_FOOL_1 *pFool_3 = (ST_FOOL_1*)&inputArr[0];  
printf("pFool_3 :\n");  
printf(".mem1 = 0x%02X\n",pFool_3->mem1);  
printf(".mem2 = 0x%02X\n",pFool_3->mem2);  
printf(".mem6 = 0x%08X\n",pFool_3->mem6);

printf("\n");

ST_FOOL_2 *pFool_4 = (ST_FOOL_2*)&inputArr[0];  
printf("pFool_4 :\n");  
printf(".mem1 = 0x%02X\n",pFool_4->mem1);  
printf(".mem2 = 0x%02X\n",pFool_4->mem2);  
printf(".mem3 = 0x%02X\n",pFool_4->mem3);  
printf(".mem4 = 0x%02X\n",pFool_4->mem4);  
printf(".mem5 = 0x%02X\n",pFool_4->mem5);

return 0;  
}  
```

Ta sẽ kiểm tra các kết quả với các trường hợp aligment khác nhau.  
- Khi không sử dụng **pack**, kết quả sẽ như sau:

```bash  
====Structure --> Array====  
pFool_1 :  
.mem1 = 0x12  
.mem2 = 0x34  
.mem6 = 0x056789AB  
Output array(hex) :  
12 34 00 00 AB 89 67 05 // 2 byte 0x00 0x00 bị để trống

pFool_2 :  
.mem1 = 0x12  
.mem2 = 0x34  
.mem3 = 0x56  
.mem4 = 0x78  
.mem5 = 0x90  
Output array (hex):  
12 34 56 78 90 // dữ liệu đúng

====Array --> Structure===  
Input array:  
01 02 03 04 05 06 07 08 09 10 11 12 13 14 15 16  
pFool_3 :  
.mem1 = 0x01  
.mem2 = 0x02  
.mem6 = 0x08070605// 2 byte 0x03 0x04 bị mất

pFool_4 :  
.mem1 = 0x01  
.mem2 = 0x02  
.mem3 = 0x03  
.mem4 = 0x04  
.mem5 = 0x05  
```

*   Khi sử dụng **pack(1)**:

```C  
====Structure --> Array====  
pFool_1 :  
.mem1 = 0x12  
.mem2 = 0x34  
.mem6 = 0x056789AB  
Output array(hex) :  
12 34 AB 89 67 05 // các member được sếp sát nhau

pFool_2 :  
.mem1 = 0x12  
.mem2 = 0x34  
.mem3 = 0x56  
.mem4 = 0x78  
.mem5 = 0x90  
Output array (hex):  
12 34 56 78 90

====Array --> Structure===  
Input array:  
01 02 03 04 05 06 07 08 09 10 11 12 13 14 15 16  
pFool_3 :  
.mem1 = 0x01  
.mem2 = 0x02  
.mem6 = 0x06050403// các member được đẩy sát nhau

pFool_4 :  
.mem1 = 0x01  
.mem2 = 0x02  
.mem3 = 0x03  
.mem4 = 0x04  
.mem5 = 0x05  
```

### 3. Kết luận:

Khi viết một cấu trúc mà ta có ý định sử dụng nó cho việc casting sang kiểu Array hoặc từ kiểu Array ta nên chú ý đặc điểm này, thông thường **pack(4)** được sử dụng mặc định, vì thế ta nên sắp xếp các member trong cấu trúc sao cho chứa đủ trong mỗi 4 byte.  
Hoặc ta nên chỉ rõ sử dụng **pack(1)**, **pack(2)** hoặc **pack(4)**. Thường là 1 hoặc 4.

Ví dụ với trường hợp cấu trúc **ST_FOOL_1** ở trên, ta có thể viết lại như sau để đảm bảo không có byte trống khi convert.

```C  
typedef struct  
{  
uint8_t mem1;  
uint8_t mem2;  
uint8_t padding[2];  
uint32_t mem6;  
}ST_FOOL_1;  
```