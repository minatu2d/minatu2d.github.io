---
layout: post
title: Giới thiệu về lập trình Assembly trên Linux (AT&T Style không phải Intel Style)
date: 2017-02-11 12:47:50.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
tags:
- Asm
- Assembly
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '1742116569'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2017/02/11/gioi-thieu-ve-lap-trinh-assembly-tren-linux-att-style-khong-phai-intel-style/"
---
_Tham khảo_  
- Sách : AT&T Assembly Language, Richard Blum

## 1. Ngôn ngữ Assembly là gì?

*   Ở mức thấp nhất, Process chỉ hiểu **instruction code**
*   **Instruction Code** là các mã nhị phân chứa các thành phần: Instruction prefix, Opcode, ModR/M, SIB, Displacement, Data Element.
*   Người ta hoàn toàn có thể viết chương trình bằng **instruction code**, nhưng nó sẽ cực kì khó nhọc, bởi ta chỉ thấy không gì khác ngoài các byte nối tiếp nhau.
*   Việc sử dụng các ngôn ngữ bậc cao giúp việc viết chương trình dễ dàng hơn rất nhiều, vì trình biên dịch hoặc thông dịch đảm nhiệm việc chuyển mã ngôn ngữ bậc cao trực tiếp hoặc gián tiếp sang **instruction code** để chạy.
*   Tuy nhiên, việc sinh **instruction code** của trình biên dịch/thông dịch không phải luôn luôn hiệu quả.
*   Khi muốn động vào các kết quả được sinh ra từ trình biên dich/thông dịch ở trên, ta không khó có thể sử trực tiếp trên **instruction code**, một dạng dễ nhớ của các **instruction code** này được sinh ra để phục vụ việc này. Đó chính là **Assembly Language**.
*   Vì gần như được chuyển sang **instruction code** của Processor nên **Assemble Code** gần như được gắn với Processor nào đó.
*   Dù các lệnh của hầu hết bộ xử lý là tương tự nhau nhưng quy tắc gợi nhớ các mã **Assembly** dù chỉ khác nhau một chút thôi cũng khiến việc viết code cross-processor gần như là không thể rồi.
*   Ta nói **Assembly Language** là nói đến 1 ngôn ngữ, nhưng thực chất mỗi Processor có 1 **Assembly Language** riêng của nó.
*   Bởi vậy, để để lập trình với **Assembly** cho 1 Processor nào đó, ta cũng cần **Assembler** tương ứng cho Processor đó.

## 2. Intel ASM và AT&T ASM và ARM

*   Các bộ xử lý cùng kiến trúc thường ít có thay đổi về **Assembly Language**, tuy nhiên 2 kiến trúc khác nhau thì thường có **Assembly Language** khác nhau
*   **ARM** : Hiện tại chưa tìm hiểu được.
*   Nếu bỏ qua **ARM**, ta có thể chia ra làm 2 loại **Assembly Language** sau theo tiêu chí **lệnh gợi nhớ**, đó là **Intel** và **AT&T**.  
    Về sự khác nhau của 2 loại này:  
    Hầu hết sự khác biệt xuất hiện ở một dài định dạng cụ thể. Nhưng những điểm khác nhau chính giữa **Intel** và **AT&T** được đưa ra dưới đây:

> *   **AT&T** sử dụng **$** để kí hiệu, còn **Intel** thì không.  
>     Vì thế, khi biểu diễn giá trị **4**, AT&T sẽ sử dụng **$4**, còn **Intel** thì  
>     đơn giản viết **4**.
> *   **AT&T** sử dụng thứ tự về source (nguồn), destination (đích) ngược với **Intel**,  
>     Ví thế, để chuyển giá trị **4** vào thanh ghi **EAX**, **AT&T** sẽ viết là  
>     _movl $4, %eax_, còn **Intel** lại viết là _mov eax, 4_.
>     
> *   Ở Long calls và jumps cũng sử dụng cú pháp khác nhau để định nghĩa segment và  
>     offset. **AT&T** sử dụng _ljmp $section, $offset_, trong khi đó Intel sử dụng  
>     _jmp section:offset_
>     

## 3. ASM trên Linux

Để lập trình ASM trên Linux, ta sử dụng những công cụ sau:

*   Assembler :**as**  
    Đây là **GNU Assembler**, là cross-platform assembler phổ biến nhất trên Linux.  
    Như đã nói ở trên, mỗi **assemble language** thường gắn liền với assembler của nó. Vậy thì cross-platform ở trên là ý gì. Có nghĩa là compiler này có khả năng biên dịch nhiều loại assembly code khác nhau.  
    Cùng 1 source nhưng nó có thể sinh ra nhiều loại instruction code tùy theo processor được chỉ đinh.

*   Linker :**ld**  
    Ở ngôn ngữ C/C++ hầu như ít khi ta phải thao tác trực tiếp với linker. **gcc** thường sẽ làm tất rồi.  
    Nhưng đối với assembler thì không như thế, nó sẽ không tự động gọi linker đâu, ta phải tự gọi **linker** để tạo ra file chạy cho source chúng ta viết.  
    Và **linker** duy nhất sử dụng trên Linux chính là **GNU Linker**, hay **ld**.
    
*   Debugger : **gdb**  
    Chúng ta đã có 1 loại bài về **GDB** rồi, **gdb** debug được mọi chương trình trong Linux dù chúng được viết bằng C/C++ hay bây giờ là Assembly đi nữa.
    
*   C/C++ Compiler :**gcc**  
    Thực ra không trực tiếp liên quan lắm. Nhưng như trong 1 bài nói về **Tìm hiểu thêm về biên dịch HelloWorl**, ta cũng đề cập đến khả năng sinh Asssembly Code của **gcc** rồi.  
    Có thể chúng ta sẽ sử dụng **gcc** để sinh mã ASM rồi optimize chúng thay vì viết từ zero.
    
*   Object code dissassembler (Dịch ngược asm):**objdump**  
    Cho phép xem instruction code và mã dịch ngược về ASM của nó.
    
*   Profiler : **gprof**  
    Là công cụ đánh giá performance của phần mềm trong Linux. Nó cho phép xác định hàm nào tốn nhiều thời gian xử lý nhất. Từ đó, ta có thể tập trung cải tiến để tăng hiệu năng.
    

## 4. Một chương trình Assembly đơn giản

Chương trình sau sẽ lấy tên của CPU và hiển thị ra dòng **The processor Vendor ID is XXXX **

*   Source code

```asm  
#Author : Richard Blum  
#cpuid.s Sample program to extract the processor Vendor ID  
.section .data  
output:  
.ascii “The processor Vendor ID is 'xxxxxxxxxxxx'\n”  
.section .text  
.globl _start  
_start:  
movl $0, %eax  
cpuid  
movl $output, %edi  
movl %ebx, 28(%edi)  
movl %edx, 32(%edi)  
movl %ecx, 36(%edi)  
movl $4, %eax  
movl $1, %ebx  
movl $output, %ecx  
movl $42, %edx  
int $0x80  
movl $1, %eax  
movl $0, %ebx  
int $0x80  
```

Lưu lại và đặt tên là **cpuid.s**:  
- Biên dịch bằng Assembler:

```bash  
$as -o cpuid.o cpuid.s  
```

Output: **cpuid.o**

*   Link để tạo file chạy:

```bash  
ld -o cpuid cpuid.o  
```

Output: **cpuid**

*   Chạy

```bash  
$ ./cpuid  
The processor Vendor ID is 'GenuineIntel'  
```

*   Debug  
    Ta cần biên dịch với option khác thì mới debug được.

```bash  
$as -o cpuid.o cpuid.s  
ld -o cpuid cpuid.o  
```

Ta sẽ đặt break point ở **_start** và xem thử vị trí dừng:

```bash  
$ gdb ./cpuid  
GNU gdb (Ubuntu 7.11-0ubuntu1) 7.11  
Copyright (C) 2016 Free Software Foundation, Inc.  
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>  
This is free software: you are free to change and redistribute it.  
There is NO WARRANTY, to the extent permitted by law. Type "show copying"  
and "show warranty" for details.  
This GDB was configured as "i686-linux-gnu".  
Type "show configuration" for configuration details.  
For bug reporting instructions, please see:  
<http://www.gnu.org/software/gdb/bugs/>.  
Find the GDB manual and other documentation resources online at:  
<http://www.gnu.org/software/gdb/documentation/>.  
For help, type "help".  
Type "apropos word" to search for commands related to "word"...  
Reading symbols from ./cpuid...done.  
(gdb) b *_start  
Breakpoint 1 at 0x8048074: file cpuid.s, line 10.  
(gdb) r  
Starting program: /home/osboxes/code/asm/cpuid

Breakpoint 1, _start () at cpuid.s:10  
10 movl $0, %eax  
(gdb) l  
5 .section .text  
6 .globl _start  
7 _start:  
8 # Using $0 for value 0  
9 # Using %eax for register EAX  
10 movl $0, %eax  
11 cpuid  
12 movl $output, %edi  
13 movl %ebx, 28(%edi)  
14 movl %edx, 32(%edi)  
```

*   Dissassemler:  
    Cho **cpuid.o**:

```bash  
$ objdump -d cpuid.o

cpuid.o: file format elf32-i386

Disassembly of section .text:

00000000 :  
0: b8 00 00 00 00 mov $0x0,%eax  
5: 0f a2 cpuid  
7: bf 00 00 00 00 mov $0x0,%edi  
c: 89 5f 1c mov %ebx,0x1c(%edi)  
f: 89 57 20 mov %edx,0x20(%edi)  
12: 89 4f 24 mov %ecx,0x24(%edi)  
15: b8 04 00 00 00 mov $0x4,%eax  
1a: bb 01 00 00 00 mov $0x1,%ebx  
1f: b9 00 00 00 00 mov $0x0,%ecx  
24: ba 2a 00 00 00 mov $0x2a,%edx  
29: cd 80 int $0x80  
2b: b8 01 00 00 00 mov $0x1,%eax  
30: bb 00 00 00 00 mov $0x0,%ebx  
35: cd 80 int $0x80  
```

Cho **cpuid**:

```bash  
$ objdump -d cpuid

cpuid: file format elf32-i386

Disassembly of section .text:

08048074 :  
8048074: b8 00 00 00 00 mov $0x0,%eax  
8048079: 0f a2 cpuid  
804807b: bf ab 90 04 08 mov $0x80490ab,%edi  
8048080: 89 5f 1c mov %ebx,0x1c(%edi)  
8048083: 89 57 20 mov %edx,0x20(%edi)  
8048086: 89 4f 24 mov %ecx,0x24(%edi)  
8048089: b8 04 00 00 00 mov $0x4,%eax  
804808e: bb 01 00 00 00 mov $0x1,%ebx  
8048093: b9 ab 90 04 08 mov $0x80490ab,%ecx  
8048098: ba 2a 00 00 00 mov $0x2a,%edx  
804809d: cd 80 int $0x80  
804809f: b8 01 00 00 00 mov $0x1,%eax  
80480a4: bb 00 00 00 00 mov $0x0,%ebx  
80480a9: cd 80 int $0x80  
```