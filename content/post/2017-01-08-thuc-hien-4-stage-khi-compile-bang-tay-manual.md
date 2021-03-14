---
layout: post
title: Thực hiện 4 Stage khi Compile bằng tay (Manual)
date: 2017-01-08 06:18:19.000000000 +09:00
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
- Linker
- Linux
- Loader
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '524289058'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2017/01/08/thuc-hien-4-stage-khi-compile-bang-tay-manual/"
---
Ta đã có bài giới thiệu về [4 Stage khi Compiling](https://lazytrick.wordpress.com/2017/01/01/4-stage-khi-bien-dich-helloworld-c/) rồi. Đầu ra của Stage trước sẽ là đầu vào của Stage sau.

Trong compile thông thường dạng

```bash  
$gcc -o HelloWorld HelloWorld.c  
```

Với câu lệnh trên,ta sẽ không thấy kết quả của 3 Stage đầu tiên.

Để hiểu rõ hơn, chúng ta hãy thử thực hiện các Stage bằng tay xem liệu ta có thể tạo ra file chạy như câu lệnh compile trên hay không.

### 1. Thực hiện Stage 1 (Preprocessing)

Như trong bài trước, stage này sẽ lấy đầu vào là file .c và cho kết quả đầu ra là file **.i** (thông thường)

Đầu vào: **HelloWorld.c**  
Đầu ra: **HelloWorld.i**  
Câu lệnh thực hiện:

```bash  
$gcc -E HelloWorld.c -o HelloWorld.i  
```

Hoặc bằng câu lệnh **cpp**:

```bash  
$cpp HelloWorld.c -o HelloWorld.i  
```

File **HelloWorld.i** khá dài so với **HelloWorld.c**.

Về ý nghĩa, file **HelloWorld.i** vẫn là một file source C như bao file Source Code C khác mà thôi, chứ hoàn toàn chưa chuyển sang dạng khác.

### 2. Thực hiện Stage 2 (Compiling to Assembly Code)

Stage này sẽ chuyển từ source C chứa trong các file **HelloWorld.i** sang Assembly Code, file **HelloWorld.s**:

Đầu vào: **HelloWorld.i**  
Đầu ra: **HelloWorld.s**  
Câu lệnh thực hiện:

```bash  
$gcc -S HelloWorld.i -o HelloWorld.i  
```

### 3. Thực hiện Stage 3 (Compiling to Machine Code)

Bước này sẽ chuyển Assembly Code sang mã máy mà chương trình sẽ chạy:

Đầu vào: **HelloWorld.s**  
Đầu ra:**HelloWorld.o**

Câu lệnh thực hiện:

```bash  
$as HelloWorld.s -o HelloWorld.o  
```

**as** chính là Assembler, một trình biên dịch Assembly.

### 4. Thực hiện Stage 4 (Linking)

Linker trong Linux là **ld** hay tên đầy đủ là GNU Linker. Xem thêm ở [man ld](https://linux.die.net/man/1/ld)

*   **Lần 1**: Ta thấy 3 stage ở trên thực hiện không mấy khó khăn gì.  
    Cùng dạng câu lệnh như thế mà thực hiện thì sao.  
    Đây là kết quả:

```bash  
$ ld HelloWorld.o -o HelloWorld  
ld: warning: cannot find entry symbol _start; defaulting to 00000000004000b0  
HelloWorld.o: In function \`main':  
HelloWorld.c:(.text+0xa): undefined reference to \`puts'  
```

*   **Lần 2**: Đọc một chút về **Linker** trong man page.  
    Và hiểu ra một chút rằng, chương trình C ta viết sử dụng hàm **printf**, hàm này không phải ta tự viết.  
    Người mới học vẫn gọi là **hàm chuẩn**, hầu như không quan tâm nó đến từ đâu.  
    Nhưng ta đang làm manual mà. Bản thân ngôn ngữ C không bao gồm một thư viện, hay hàm nào cả.  
    Vậy hàm **printf** lấy từ đâu, ta phải chỉ cho **Linker** biết. Đó là thư viện **libc**, chứa những hàm cơ bản mà chúng ta bảo là chuẩn cho Linux.  
    Tham khảo ví dụ trong [man page](//linux.die.net/man/1/ld)),  
    ta thực hiện lại việc link bằng command:

```bash  
$ ld -o HelloWorld /lib/crt0.o HelloWorld.o -lc  
ld: cannot find /lib/crt0.o: No such file or directory  
```

Lỗi trên do không có file.  
Bỏ **/lib/crt0.o** đi thì sao:

```bash  
$ ld -o HelloWorld HelloWorld.o -lc  
ld: warning: cannot find entry symbol _start; defaulting to 0000000000400260  
```

Lỗi trên **entry_point** (địa chỉ hàm mà CPU sẽ nhảy vào đầu tiên để bắt đầu thực hiện chương trình) chưa được khai báo.  
Sửa như sau:

```bash  
$ld -o HelloWorld HelloWorld.o -lc --entry main  
```

Câu lệnh thành công, file **HelloWorld** được tạo ra.  
Tuy nhiên, khi chạy thì:

```bash  
./HelloWorld  
bash: ./HelloWorld: No such file or directory  
```

Lỗi trên là không có file nào như thế. WTF, lỗi gì lạ vậy, rõ ràng là có mà.

*   **Lần 3**: Sau một hồi hỏi thầy GG. Nguyên nhân có vẻ là do **dynamic loader linker** mà **HelloWorld** yêu cầu không có trong hệ thống.

Kiểm tra như sau:

```bash  
$ readelf -l HelloWorld|grep interpreter  
[Requesting program interpreter: /lib/ld64.so.1]  
$ ls /lib/ld64.so.1  
ls: cannot access '/lib/ld64.so.1': No such file or directory  
```

Kiểm tra dynamic loader linker từ file kết quả được build "không manual".

Tức là bằng câu lệnh:

```bash  
$gcc -o HelloWorld_auto HelloWorld.c  
$ readelf -l HelloWorld_auto |grep interpreter  
[Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]  
```

Ta thử "ép buộc" sử dụng dynamic loader linker hiện có thì sao.

```bash  
$ /lib64/ld-linux-x86-64.so.2 ./HelloWorld  
Hello World  
Segmentation fault (core dumped)  
```

Chạy được nhưng bị _Segmentation fault_.  
Ta không nên tìm hiểu sâu ở đây nữa.

*   **Lần 4**: Ta tự hỏi, **GCC** đã thực hiện quá trình **linking** khi tạo ra file **HelloWorld_auto** ở trên như thế nào?  
    Thật may, sau một hồi hỏi thầy GG.  
    Ta có thể thấy được toàn bộ tham số của câu lệnh biên dịch ở trên bằng tham số **-v** vào câu lệnh biên dịch.

```bash  
$gcc -v -o HelloWorld_auto HelloWorld.c  
```

Kết quả từ câu lệnh trên khá rắc rối, (có lẽ cần 1 bài khác để nói kĩ hơn về nó).  
Tuy nhiên ta chỉ quan tâm đến đoạn tham số của **collect2** (chính là Linker mà **GCC** sử dụng cho ngôn ngữ C) mà thôi.  
Nó như thế này:

```bash  
<br />COLLECT_GCC_OPTIONS='-v' '-o' 'HelloWorld_auto' '-mtune=generic' '-march=x86-64'  
/usr/lib/gcc/x86_64-linux-gnu/5/collect2 -plugin /usr/lib/gcc/x86_64-linux-gnu/5/liblto_plugin.so -plugin-opt=/usr/lib/gcc/x86_64-linux-gnu/5/lto-wrapper -plugin-opt=-fresolution=/tmp/ccy1PInh.res -plugin-opt=-pass-through=-lgcc -plugin-opt=-pass-through=-lgcc_s -plugin-opt=-pass-through=-lc -plugin-opt=-pass-through=-lgcc -plugin-opt=-pass-through=-lgcc_s --sysroot=/ --build-id --eh-frame-hdr -m elf_x86_64 --hash-style=gnu --as-needed -dynamic-linker /lib64/ld-linux-x86-64.so.2 -z relro -o HelloWorld_auto /usr/lib/gcc/x86_64-linux-gnu/5/../../../x86_64-linux-gnu/crt1.o /usr/lib/gcc/x86_64-linux-gnu/5/../../../x86_64-linux-gnu/crti.o /usr/lib/gcc/x86_64-linux-gnu/5/crtbegin.o -L/usr/lib/gcc/x86_64-linux-gnu/5 -L/usr/lib/gcc/x86_64-linux-gnu/5/../../../x86_64-linux-gnu -L/usr/lib/gcc/x86_64-linux-gnu/5/../../../../lib -L/lib/x86_64-linux-gnu -L/lib/../lib -L/usr/lib/x86_64-linux-gnu -L/usr/lib/../lib -L/usr/lib/gcc/x86_64-linux-gnu/5/../../.. /tmp/ccocGwHc.o -lgcc --as-needed -lgcc_s --no-as-needed -lc -lgcc --as-needed -lgcc_s --no-as-needed /usr/lib/gcc/x86_64-linux-gnu/5/crtend.o /usr/lib/gcc/x86_64-linux-gnu/5/../../../x86_64-linux-gnu/crtn.o  
```

Nhìn nản luôn, edit lại chút cho "rắc rối hơn"

```bash  
COLLECT_GCC_OPTIONS='-v' '-o' 'HelloWorld_auto' '-mtune=generic' '-march=x86-64'  
/usr/lib/gcc/x86_64-linux-gnu/5/collect2  
-plugin /usr/lib/gcc/x86_64-linux-gnu/5/liblto_plugin.so  
-plugin-opt=/usr/lib/gcc/x86_64-linux-gnu/5/lto-wrapper  
-plugin-opt=-fresolution=/tmp/ccy1PInh.res  
-plugin-opt=-pass-through=-lgcc  
-plugin-opt=-pass-through=-lgcc_s  
-plugin-opt=-pass-through=-lc  
-plugin-opt=-pass-through=-lgcc  
-plugin-opt=-pass-through=-lgcc_s  
--sysroot=/  
--build-id  
--eh-frame-hdr  
-m elf_x86_64  
--hash-style=gnu  
--as-needed  
-dynamic-linker /lib64/ld-linux-x86-64.so.2  
-z relro  
-o HelloWorld_auto  
/usr/lib/gcc/x86_64-linux-gnu/5/../../../x86_64-linux-gnu/crt1.o  
/usr/lib/gcc/x86_64-linux-gnu/5/../../../x86_64-linux-gnu/crti.o  
/usr/lib/gcc/x86_64-linux-gnu/5/crtbegin.o  
-L/usr/lib/gcc/x86_64-linux-gnu/5  
-L/usr/lib/gcc/x86_64-linux-gnu/5/../../../x86_64-linux-gnu  
-L/usr/lib/gcc/x86_64-linux-gnu/5/../../../../lib -L/lib/x86_64-linux-gnu  
-L/lib/../lib -L/usr/lib/x86_64-linux-gnu  
-L/usr/lib/../lib  
-L/usr/lib/gcc/x86_64-linux-gnu/5/../../.. /tmp/ccocGwHc.o  
-lgcc  
--as-needed -lgcc_s --no-as-needed  
-lc -lgcc  
--as-needed -lgcc_s --no-as-needed  
/usr/lib/gcc/x86_64-linux-gnu/5/crtend.o  
/usr/lib/gcc/x86_64-linux-gnu/5/../../../x86_64-linux-gnu/crtn.o  
```

Ồ, dù không thể hiểu hết các tham số nhưng ta cũng thấy một số tham số sau:

1.  **-dynamic-linker /lib64/ld-linux-x86-64.so.2**: chỉ ra dynamic linker nào được được sử dụng để ghi vào file kết quả.
2.  **-L<đường dẫn>** : chỉ định các đường dẫn chứa thư viện mặc đinh hay chúng ta vẫn nói là chuẩn.
3.  **--sysroot=/** : là thông số **cực kì quan trọng** trong cross-compling. Nó sẽ chỉ ra dường dẫn mà các đường dẫn trong qua trình biên dịch lấy đó là thư mục root.

Ok, ta thử chạy manual **Linker** với mớ tham số ở trên xem sao (nhớ xóa tham số **-o HelloWorld_auto** nữa.

```bash  
ld -o HelloWorld HelloWorld.o -plugin /usr/lib/gcc/x86_64-linux-gnu/5/liblto_plugin.so -plugin-opt=/usr/lib/gcc/x86_64-linux-gnu/5/lto-wrapper -plugin-opt=-pass-through=-lgcc -plugin-opt=-pass-through=-lgcc_s -plugin-opt=-pass-through=-lc -plugin-opt=-pass-through=-lgcc -plugin-opt=-pass-through=-lgcc_s --sysroot=/ --build-id --eh-frame-hdr -m elf_x86_64 --hash-style=gnu --as-needed -dynamic-linker /lib64/ld-linux-x86-64.so.2 -z relro /usr/lib/gcc/x86_64-linux-gnu/5/../../../x86_64-linux-gnu/crt1.o /usr/lib/gcc/x86_64-linux-gnu/5/../../../x86_64-linux-gnu/crti.o /usr/lib/gcc/x86_64-linux-gnu/5/crtbegin.o -L/usr/lib/gcc/x86_64-linux-gnu/5 -L/usr/lib/gcc/x86_64-linux-gnu/5/../../../x86_64-linux-gnu -L/usr/lib/gcc/x86_64-linux-gnu/5/../../../../lib -L/lib/x86_64-linux-gnu -L/lib/../lib -L/usr/lib/x86_64-linux-gnu -L/usr/lib/../lib -lgcc --as-needed -lgcc_s --no-as-needed -lc -lgcc --as-needed -lgcc_s --no-as-needed /usr/lib/gcc/x86_64-linux-gnu/5/crtend.o /usr/lib/gcc/x86_64-linux-gnu/5/../../../x86_64-linux-gnu/crtn.o  
```

Và đây là **HelloWorld**:

```bash  
$./HelloWorld  
Hello World  
```

### 5. Tham số cho GCC

Nếu muốn xem các file kết quả trung gian từ 3 Stage đầu, ta có thể thêm tham số **--save-temps** khi biên dịch.

```bash  
$gcc --save-temps -o HelloWorld HelloWorld.c  
$ls -lia  
total 52  
5097125 drwxrwxr-x 2 oedev oedev 4096 1月 8 15:09 .  
5097103 drwxrwxr-x 3 oedev oedev 4096 1月 8 15:09 ..  
4990864 -rwxrwxr-x 1 oedev oedev 8608 1月 8 15:09 HelloWorld  
4990859 -rw-rw-r-- 1 oedev oedev 70 1月 8 15:09 HelloWorld.c  
4990861 -rw-rw-r-- 1 oedev oedev 17121 1月 8 15:09 HelloWorld.i  
4990863 -rw-rw-r-- 1 oedev oedev 1504 1月 8 15:09 HelloWorld.o  
4990862 -rw-rw-r-- 1 oedev oedev 460 1月 8 15:09 HelloWorld.s

```