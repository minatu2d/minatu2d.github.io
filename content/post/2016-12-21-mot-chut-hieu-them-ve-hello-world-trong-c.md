---
layout: post
title: Một chút hiểu thêm về "Hello World" trong C.
date: 2016-12-21 05:31:09.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
- Compile
tags:
- Compiler
- HelloWorld
- Linker
- Loader
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '30216149769'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/12/21/mot-chut-hieu-them-ve-hello-world-trong-c/"
---
Gần đây, gặp một số vấn đề về Loader-Linker giữa môi trường build và môi trường chạy trong Cross-Compiling.

Có thể bất kì chương nào trong Linux cũng vậy. Nhưng chỉ xét một chường trình được build bằng C thì  
một chương trình sẽ được chạy 2 cách dưới đây

### 1. Chương trình hoàn toàn là tĩnh

*   Không có symbol nào cần phải được (resolved) trước khi chạy.
*   Không yêu cầu bất cứ một thư viện run-time nào.
*   Kernel cứ thế load vào, rồi nhảy đến vị trí của Program Entry là chạy.
*   Có kích thước lớn
*   Để build được nó thì các thư viện phụ thuộc khi build cũng phải có phiên bản static.  
    Lấy ví dụ **HelloWorld**

```c  
#include  
int main(int argc, char *args[])  
{  
printf("Hello, Ajinomoto \n");  
return 0;  
}  
```

Khi build tĩnh bằng (hầu như phải có **-static**)

```bash  
gcc -o helloworld_static helloworld.c -static  
```

Kích thước file chạy sẽ như sau:

```bash  
$ ls -liah helloworld_static  
2910638 -rwxrwxr-x 1 duser duser 717K Dec 9 13:37 helloworld_static  
```

**717KB** cho HelloWorld trên Ubuntu 14.04

Kiểm tra bằng **objdump**:

```bash  
$ objdump -p helloworld_static

helloworld_static: file format elf32-i386

Program Header:  
LOAD off 0x00000000 vaddr 0x08048000 paddr 0x08048000 align 2**12  
filesz 0x000a0c77 memsz 0x000a0c77 flags r-x  
LOAD off 0x000a0f40 vaddr 0x080e9f40 paddr 0x080e9f40 align 2**12  
filesz 0x00001040 memsz 0x000023c4 flags rw-  
NOTE off 0x000000f4 vaddr 0x080480f4 paddr 0x080480f4 align 2**2  
filesz 0x00000044 memsz 0x00000044 flags r--  
TLS off 0x000a0f40 vaddr 0x080e9f40 paddr 0x080e9f40 align 2**2  
filesz 0x00000010 memsz 0x00000028 flags r--  
STACK off 0x00000000 vaddr 0x00000000 paddr 0x00000000 align 2**4  
filesz 0x00000000 memsz 0x00000000 flags rw-  
RELRO off 0x000a0f40 vaddr 0x080e9f40 paddr 0x080e9f40 align 2**0  
filesz 0x000000c0 memsz 0x000000c0 flags r--  
```

Nhìn có vẻ khó hiểu, như chỉ để ý thấy chỉ có phần **Program Header** mà không còn phần nào khác.

### 2. Chương trình là động

*   Chứa symbol cần được phân giải trước khi chạy.
*   Hầu như các command trên Linux đều là dạng này.
*   Loader-Linker (có đường dẫn tuyệt đối) được ghi bên trong file chạy  
    tức là nếu yêu cầu loader-linker /lib/ld-linux.so.2 mà môi trường chạy không có thì sẽ có lỗi rất lạ kiểu Không tìm thấy file, hoặc định dạng lỗi, etc.
*   Tên của thư viện phụ thuộc (không có đường dẫn) được ghi bên trong file chạy.
*   Kích thước file chạy nhỏ.

Tiếp tục thử với **Hello World**

```bash  
gcc -o helloworld helloworld.c  
```

Kiểm tra kích thước file chạy:

```bash  
$ ls -liah helloworld_dynamic  
2910636 -rwxrwxr-x 1 duser duser 7.2K Dec 9 13:37 helloworld_dynamic  
```

**7.2K** cho **HelloWorld**

Kiểm tra bằng **objdump**:

```bash  
$ objdump -p helloworld_dynamic

helloworld_dynamic: file format elf32-i386

Program Header:  
PHDR off 0x00000034 vaddr 0x08048034 paddr 0x08048034 align 2**2  
filesz 0x00000120 memsz 0x00000120 flags r-x  
INTERP off 0x00000154 vaddr 0x08048154 paddr 0x08048154 align 2**0  
filesz 0x00000013 memsz 0x00000013 flags r--  
LOAD off 0x00000000 vaddr 0x08048000 paddr 0x08048000 align 2**12  
filesz 0x000005c0 memsz 0x000005c0 flags r-x  
LOAD off 0x00000f08 vaddr 0x08049f08 paddr 0x08049f08 align 2**12  
filesz 0x00000118 memsz 0x0000011c flags rw-  
DYNAMIC off 0x00000f14 vaddr 0x08049f14 paddr 0x08049f14 align 2**2  
filesz 0x000000e8 memsz 0x000000e8 flags rw-  
NOTE off 0x00000168 vaddr 0x08048168 paddr 0x08048168 align 2**2  
filesz 0x00000044 memsz 0x00000044 flags r--  
EH_FRAME off 0x000004e4 vaddr 0x080484e4 paddr 0x080484e4 align 2**2  
filesz 0x0000002c memsz 0x0000002c flags r--  
STACK off 0x00000000 vaddr 0x00000000 paddr 0x00000000 align 2**4  
filesz 0x00000000 memsz 0x00000000 flags rw-  
RELRO off 0x00000f08 vaddr 0x08049f08 paddr 0x08049f08 align 2**0  
filesz 0x000000f8 memsz 0x000000f8 flags r--

Dynamic Section:  
NEEDED libc.so.6  
INIT 0x080482b0  
FINI 0x080484b4  
INIT_ARRAY 0x08049f08  
INIT_ARRAYSZ 0x00000004  
FINI_ARRAY 0x08049f0c  
FINI_ARRAYSZ 0x00000004  
GNU_HASH 0x080481ac  
STRTAB 0x0804821c  
SYMTAB 0x080481cc  
STRSZ 0x0000004a  
SYMENT 0x00000010  
DEBUG 0x00000000  
PLTGOT 0x0804a000  
PLTRELSZ 0x00000018  
PLTREL 0x00000011  
JMPREL 0x08048298  
REL 0x08048290  
RELSZ 0x00000008  
RELENT 0x00000008  
VERNEED 0x08048270  
VERNEEDNUM 0x00000001  
VERSYM 0x08048266

Version References:  
required from libc.so.6:  
0x0d696910 0x00 02 GLIBC_2.0  
```

### 3. Hiển thị thông về loader-linker

Khi chương trình động được chạy, một loader-linker sẽ được kernel gọi lên để load và link các thư viện động.  
Gọi là loader-linker.  
Thường sẽ có nằm ở **/lib/ld.ABC.so** hoặc tương tự như thế.  
Loader-linker này sẽ tìm các thư viện được yêu cầu từ Search Path, sau đó link lại để tạo ra chương trình chạy.  
Search Path này được cấu thành từ nhiều loại như: từ các path được đặt bên trong **/etc/ld.conf.so**, từ biến **LD_LIBRARY_PATH**, etc. Xem thêm tại: [ld.so](http://man7.org/linux/man-pages/man8/ld.so.8.html)

Một cách để xem các thông tin debug của loader-linker này:  
Giả sử với **HelloWorld** ở trên:

```bash  
$ LD_DEBUG=help ./helloworld_dynamic  
Valid options for the LD_DEBUG environment variable are:

libs display library search paths  
reloc display relocation processing  
files display progress for input file  
symbols display symbol table processing  
bindings display information about symbol binding  
versions display version dependencies  
scopes display scope information  
all all previous options combined  
statistics display relocation statistics  
unused determined unused DSOs  
help display this help message and exit

To direct the debugging output into a file instead of standard output  
a filename can be specified using the LD_DEBUG_OUTPUT environment variable.  
```

Trên là Help, ta thử với **libs** sẽ có kết quả như sau:

```bash  
$ LD_DEBUG=libs ./helloworld_dynamic  
7066: find library=libc.so.6 [0]; searching  
7066: search cache=/etc/ld.so.cache  
7066: trying file=/lib/i386-linux-gnu/libc.so.6  
7066:  
7066:  
7066: calling init: /lib/i386-linux-gnu/libc.so.6  
7066:  
7066:  
7066: initialize program: ./helloworld_dynamic  
7066:  
7066:  
7066: transferring control: ./helloworld_dynamic  
7066:  
Hello, Ajinomoto  
7066:  
7066: calling fini: ./helloworld_dynamic [0]  
7066:  
```

doime 09-12-2016