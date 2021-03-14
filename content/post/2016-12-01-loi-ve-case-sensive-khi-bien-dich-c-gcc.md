---
layout: post
title: Lỗi về Case-sensive khi biên dịch C (gcc)
date: 2016-12-01 13:08:38.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Compile
- Khác
tags:
- C
- Error
- FileSystem
- GCC
- Linux
meta:
  _wpcom_is_markdown: '1'
  _edit_last: '72243466'
  geo_public: '0'
  _publicize_job_id: '29506004430'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/12/01/loi-ve-case-sensive-khi-bien-dich-c-gcc/"
---
Khi phát triển các ứng dụng trên Linux, nhúng Linux, mình hầu như cài đặt và sử dụng một máy ảo (tạo bằng VMWare hoặc VirtualBox). Cài trình biên dịch **GCC** lên đó.  
Hầu như mình có thể làm mọi việc trên môi trường máy ảo đó trừ quản lý source.  
Vì cty mình vẫn sử dụng SVN với Client là Tortoise. Linux cũng có rất nhiều công cụ tuơng tự Tortoise nhưng để tránh những vấn đề không cần thiết, có thể làm phiền người khác liên quan đến tương thích SVN, mình vẫn chọn quản lý bằng Tortoise trên Windows.  

Vì 2 yêu cầu trên, mình thường bằng 1 cách nào đó setup một thư mục chung giữa 2 môi trường. Và cách phổ biến và có lẽ là đơn giản nhất là sử dụng module **vmhgfs** được cung cấp sẵn bở VMWare.  
Với cách này, chúng ta dễ dàng chia sẻ 1 thư mục từ Host (máy Windows) với máy Guest (chạy Ubuntu chẳng hạn).  
Nhưng gần đây, mình gặp 1 vấn đề với hệ thống file được **vmhgfs** hỗ trợ, khi build source trên thư mục chia sẻ với Host này.  
Vấn đề được trình bày bên dưới.

### 1. Cấu trúc source ví dụ

Giả sử có source như này:

```bash  
.  
├── hello  
├── hello.c  
├── inc_lowercase  
│ └── abc.h  
└── inc_uppercase  
└── Abc.h  
```

**hello.c**:

```c  
$ cat hello.c  
#include  
#include&quot;Abc.h&quot;  
#include&quot;abc.h&quot;

int main(int argc, char* argv[])  
{  
abc_t var1;  
Abc_t var2;

var1.x = 0;  
var2 = 3;

printf(&quot;OMG Hello world %d %ldn&quot;,var1.x, var2);  
return 0;  
}  
```

**inc_lowercase/abc.h**:

```c  
typedef struct _abc_  
{  
int x;  
}abc_t;  
```

**inc_uppercase/Abc.h**:

```c  
typedef long int Abc_t;  
```

### 2. So sánh thử

#### 2.1 Thư mục source nằm trên **_home_** của VMWare

Kết quả: Build ngon, chạy tốt:

```bash  
$ gcc -o hello hello.c -I./inc_uppercase -I./inc_lowercase  
$ ./hello  
OMG Hello world 0 3  
```

#### 2.1 Thư mục source nằm trên thư mục chia sẻ giữa VMWare (chạy Linux) với Host (chạy Windows) thông qua **vmhgfs**:

Giả sử ta để source ở _/mnt/hgfs_ :

Kết quả: Build lỗi, đương nhiên là không chạy được.

```bash  
duser@192.168.70.225:/mnt/hgfs$ gcc -o hello hello.c -I./inc_uppercase -I./inc_lowercase  
In file included from hello.c:3:0:  
abc.h:1:16: error: redefinition of ‘struct _abc_’  
typedef struct _abc_  
^  
In file included from hello.c:2:0:  
Abc.h:1:16: note: originally defined here  
typedef struct _abc_  
^  
In file included from hello.c:3:0:  
abc.h:4:2: error: conflicting types for ‘abc_t’  
}abc_t;  
^  
In file included from hello.c:2:0:  
Abc.h:4:2: note: previous declaration of ‘abc_t’ was here  
}abc_t;  
^  
hello.c: In function ‘main’:  
hello.c:8:2: error: unknown type name ‘Abc_t’  
Abc_t var2;  
^  
hello.c:13:2: warning: format ‘%ld’ expects argument of type ‘long int’, but argument 3 has type ‘int’ [-Wformat=]  
printf(&quot;OMG Hello world %d %ldn&quot;,var1.x, var2);  
^  
```

Ta thấy các lỗi ở đây khá là "lạ", nào là **redefine**, nào là **unkown type name**.  
**Nếu gặp lỗi này lần đầu thì không phải ai cũng lý giải được. :(**

Vấn đề ở đây nằm ở **/mnt/hgfs**.  
Có vẻ như **hgfs** không phân biệt **Abc.h** với **abc.h** nên khi GCC (dù chạy trên VM Ubuntu) đọc cái file này lên sẽ cùng 1 tên.

Ở đây ta thấy lỗi **typedef struct _abc** thì có thể đoán được rằng **abc.h** đã được include trước.  
Có vẻ như **hgfs** coi 2 file header trên với cùng 1 tên **abc.h**.

Không chỉ có **hgfs** mà nhiều hệ thống file khác cũng giống như thế.