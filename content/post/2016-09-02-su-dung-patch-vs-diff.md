---
layout: post
title: Sử dụng Patch và Diff
date: 2016-09-02 08:53:59.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
tags:
- Command
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '26405898873'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/09/02/su-dung-patch-vs-diff/"
---
**Patch**, hay người ta vẫn gọi là các bản vá, víu gì đó. Cũng thấy khái niệm này từ lâu rồi. Nhưng chưa bao giờ phát sinh nhu cầu sử dụng vì đa số các dự án mình từng làm đều dùng Server SVN tập trung. Mọi thay đổi đều có quản lý chặt chẽ, cũng ít khi rẽ quá nhiều nhánh (branch).  
Gần đây, bắt buộc phải nghĩ đến việc sử dụng 2 tool này vì phải update source trên máy thật (một board mạch) chỉ hỗ trợ truyền Serial. :((  
Sau nhiều lần truyền source, dù đã nén bằng **_tar_**, cũng mất đến 20-30 phút, mình không thể chịu nổi nữa. Thế là hỏi thầy google. Thấy có 2 command khá thông dụng để giải quyết vấn đề này.

Tư tưởng khá đơn giản, thay vì update toàn bộ source bằng cách nén source muốn test và chuyển vào máy thật như mình đang làm, giờ chỉ tải phần thay đổi vào máy rồi đưa phần thay đối đó lên source cũ.

Cũng có 1 cách là lấy tập các file thay đổi rồi ghi đè, nhưng cách đó có vẻ không hiệu quả bằng cách dùng 2 command **Patch** và **Diff**

**Diff** sẽ tạo ra file chứa các thay đổi của phiên bản cũ so với phiên bản mới, file .patch. (thường được thực hiện trên môi trường phát triển)  
**Patch** sẽ đưa các thay đổi được ghi trong file .patch sang source cũ. (thường được thực hiện trên môi trường chạy - tức Runtime)  
Chú ý rằng, **patch**, **diff** chỉ thực hiện với các file text thôi.  
Các file binary thì phải dùng lệnh khác.

Ta hãy bắt đầu sử dụng với một vài tình huống thực tế dựa trên các thao tác cơ bản là thêm, sửa, xóa.

Giả sử ta có 1 source như sau:

1.  Thêm file
2.  Sửa file
3.  Xóa file
4.  Đổi tên file
5.  Thêm thư mục
6.  Xóa thư mục

Giả sử ta đã có 1 phiên bản source với 2 phiên bản sau:

```bash  
greetings-1.0  
├── main.c  
└── version.h  
greetings-1.0-mod  
├── greetings.h  
├── main.c  
└── version.h  
```

## 1. Thêm file

Các file **_main.c_**, ***version.h** ở cả 2 bản **1.0** và **1.0-mod** giống hệt nhau.  
Giờ ta sẽ thực hiện cập nhật các thay đổi từ **1.0-mod** sang cho **1.0**.

Tạo **patch** file sử dụng [diff](http://linux.die.net/man/1/diff).

```bash  
diff -uprN greetings-1.0 greetings-1.0-mod > patch_1.0-mod.patch  
```

Nội dung file patch sẽ như sau:

```bash  
$ cat patch_1.0-mod.patch  
diff -uprN greetings-1.0/greetings.h greetings-1.0-mod/greetings.h  
--- greetings-1.0/greetings.h 1970-01-01 09:00:00.000000000 +0900  
+++ greetings-1.0-mod/greetings.h 2016-09-12 13:43:09.128523777 +0900  
@@ -0,0 +1,5 @@  
+#ifndef __GREETINGS_HEADER__  
+#define __GREETINGS_HEADER__  
+  
+void greetings_vi();  
+#endif  
```

Thực hiện bản patch này trên biên bản **1.0** bằng command [patch](http://linux.die.net/man/1/patch) như sau:

```bash  
$cd greetings-1.0  
$patch -u < ../patch_1.0-mod.patch  
patching file greetings.h  
```

Source của **1.0** đã được patch, kiểm tra với [tree](http://linux.die.net/man/1/tree)

```bash  
greetings-1.0  
├── greetings.h  
├── main.c  
└── version.h  
greetings-1.0-mod  
├── greetings.h  
├── main.c  
└── version.h  
```

Vì một lý do nào đó, ta muốn quay lại phiên bản **1.0** ban đầu, tức là lúc chưa patch. Ta có thể thực hiện như sau:

```bash  
$cd greetings-1.0  
$patch -uR < ../patch_1.0-mod.patch  
patching file greetings.h  
```

**Patch** sẽ rollback toàn bộ nội ung đã thay đổi ở bước trên, nội dung **1.0** sẽ trở lại như đầu, khi chưa có file **_greetings.h_**.

## 2. Sửa file

Ta copy phiên bản **1.0** thanh **1.0-mod2** rồi thêm 1 dòng vào **main.c**, để nó trở thành như sau:

```C  
#include<stdio.h>  
#include"version.h"

int main(int argc, char* argv[])  
{  
// Add in v1.0-mod2  
printf("--This is sample source for patch/diff usage demo\n");

printf("--Version--info--\n");  
printf("MAJOR =%d\n",VERSION_MAJOR);  
printf("MINOR =%d\n",VERSION_MINOR);  
printf("REVSION =%d\n",REVISION_NUMBER);

return 0;  
}  
```

Và thay đổi **REVISON_NUMBER** trong **_vesion.h_** lên **2**.  
Ok, ta có 2 file thay đổi.

Tạo patch:

```bash  
diff -uprN greetings-1.0 greetings-1.0-mod2 > patch_1.0-mod2.patch  
```

Nội dung **_patch_1.0-mod2.patch_**:

```bash  
diff -uprN greetings-1.0/main.c greetings-1.0-mod2/main.c  
--- greetings-1.0/main.c 2016-09-12 10:45:17.584502129 +0900  
+++ greetings-1.0-mod2/main.c 2016-09-12 14:05:17.883121135 +0900  
@@ -3,6 +3,9 @@

int main(int argc, char* argv[])  
{  
+ // Add in v1.0-mod2  
+ printf("--This is sample source for patch/diff usage demo\n");  
+  
printf("--Version--info--\n");  
printf("MAJOR =%d\n",VERSION_MAJOR);  
printf("MINOR =%d\n",VERSION_MINOR);  
diff -uprN greetings-1.0/version.h greetings-1.0-mod2/version.h  
--- greetings-1.0/version.h 2016-09-12 11:46:31.463883113 +0900  
+++ greetings-1.0-mod2/version.h 2016-09-12 14:09:00.274072101 +0900  
@@ -3,5 +3,5 @@  
// Version :  
#define VERSION_MAJOR 1  
#define VERSION_MINOR 0  
-#define REVISION_NUMBER 0  
+#define REVISION_NUMBER 2  
#endif  
```

Apply bản patch này lên **1.0**:

```bash  
$cd greetings-1.0  
$patch -u < ../patch_1.0-mod2.patch  
patching file main.c  
patching file version.h  
```

Như vậy **1.0** sẽ được cập nhật.  
Tất nhiên ta có thể sử dụng tham số ***-R** (Reverse) để xóa bỏ toàn bộ thay đổi mà bản patch mod2 đã làm.

## 3. Xóa file

Ta copy **1.0** thành **1.0-mod3**, và xóa file **main.c**:

Làm tương tự các bước như trên ta sẽ có  
Nội dung **_patch_1.0-mod3.patch_**:

```bash  
$ cat patch_1.0-mod3.patch  
diff -uprN greetings-1.0/main.c greetings-1.0-mod3/main.c  
--- greetings-1.0/main.c 2016-09-12 14:13:52.187548541 +0900  
+++ greetings-1.0-mod3/main.c 1970-01-01 09:00:00.000000000 +0900  
@@ -1,12 +0,0 @@  
-#include<stdio.h>  
-#include"version.h"  
-  
-int main(int argc, char* argv[])  
-{  
- printf("--Version--info--\n");  
- printf("MAJOR =%d\n",VERSION_MAJOR);  
- printf("MINOR =%d\n",VERSION_MINOR);  
- printf("REVSION =%d\n",REVISION_NUMBER);  
-  
- return 0;  
-}  
```

Ta thực hiện bản patch này trên **1.0**, và có kết quả như sau:

```bash  
greetings-1.0  
└── version.h  
greetings-1.0-mod3  
└── version.h  
```

Thử rollback lại rollback lại bản **patch** vừa rồi, ta vẫn có được kết quả như mong muốn:

```bash  
greetings-1.0  
├── main.c  
└── version.h  
greetings-1.0-mod3  
└── version.h  
```

## 4. Đổi tên file

Ta copy **1.0** thành **1.0-mod4**, rồi đổi **_main.c_** thành **_main_app.c_**.  
Ta vẫn thực hiện được các bước như ở 2,3,4 và có được kết quả mong muốn.  
Xem qua file patch, ta có thể thấy phép đổi tên sẽ tương đương với phép xóa file có tên cũ và thêm file có tên mới với cũng nội dung.

```bash  
$ cat patch_1.0-mod4.patch  
diff -uprN greetings-1.0/main_app.c greetings-1.0-mod4/main_app.c  
--- greetings-1.0/main_app.c 1970-01-01 09:00:00.000000000 +0900  
+++ greetings-1.0-mod4/main_app.c 2016-09-12 14:23:11.569704749 +0900  
@@ -0,0 +1,12 @@  
+#include<stdio.h>  
+#include"version.h"  
+  
+int main(int argc, char* argv[])  
+{  
+ printf("--Version--info--\n");  
+ printf("MAJOR =%d\n",VERSION_MAJOR);  
+ printf("MINOR =%d\n",VERSION_MINOR);  
+ printf("REVSION =%d\n",REVISION_NUMBER);  
+  
+ return 0;  
+}  
diff -uprN greetings-1.0/main.c greetings-1.0-mod4/main.c  
--- greetings-1.0/main.c 2016-09-12 14:20:47.563705673 +0900  
+++ greetings-1.0-mod4/main.c 1970-01-01 09:00:00.000000000 +0900  
@@ -1,12 +0,0 @@  
-#include<stdio.h>  
-#include"version.h"  
-  
-int main(int argc, char* argv[])  
-{  
- printf("--Version--info--\n");  
- printf("MAJOR =%d\n",VERSION_MAJOR);  
- printf("MINOR =%d\n",VERSION_MINOR);  
- printf("REVSION =%d\n",REVISION_NUMBER);  
-  
- return 0;  
-}  
```

## 5. Thêm thư mục

Ta cũng copy **1.0** thành **1.0-mod5**, và thêm vào thư mục trống **_lib_**.  
Khi thực hiện **diff**, ta không thấy có sự khác nhau nào được sinh ra.  
vì thế không có bản **patch** nào trong trường hợp thêm 1 thư mục rỗng.

Việc thay đổi tham số của câu lệnh diff không tính đến ở đây.

Giờ ta thêm 1 file vào thư mục **_lib_** xem sao?  
Cấu trúc thư mục mới của **1.0-mod5**:

```bash  
greetings-1.0-mod5  
├── lib  
│   └── abc.c  
├── main.c  
└── version.h  
```

Ta tiến hành lại các bước tạo patch và apply patch như đã làm.  
Patch file có nội dung như sau:

```bash  
$ cat patch_1.0-mod5.patch  
diff -uprN greetings-1.0/lib/abc.c greetings-1.0-mod5/lib/abc.c  
--- greetings-1.0/lib/abc.c 1970-01-01 09:00:00.000000000 +0900  
+++ greetings-1.0-mod5/lib/abc.c 2016-09-12 14:35:25.920931214 +0900  
@@ -0,0 +1,6 @@  
+#include<stdio.h>  
+  
+void abc_initialize()  
+{  
+ // Code later  
+}  
```

Khi thực hiện apply patch file trên **1.0**, lệnh ta vẫn dùng sẽ không tạo ra thư mục mà ta chỉ thấy có file xuất hiện thôi.  
Đã thử thay đổi 1 vài tham số, và lệnh dưới đây đã chạy:

```bash  
$cd greetings-1.0  
$ patch -u -p1 < ../patch_1.0-mod5.patch  
$ tree .  
├── lib  
│   └── abc.c  
├── main.c  
└── version.h  
```

**p1** : ở đây được hiểu là sẽ bỏ qua 1 lần **/**.

## 6. Xóa thư mục

Giờ ta lấy copy **1.0-mod5** thanh **1.0-mod5r**, rồi xóa bỏ thư mục **_lib_**.  
Nội dung bản patch như sau:

```bash  
oedev@OEDEV-Ubuntu:~/geek/patchdiff$ cat patch_1.0-mod5r.patch  
diff -uprN greetings-1.0-mod5/lib/abc.c greetings-1.0-mod5r/lib/abc.c  
--- greetings-1.0-mod5/lib/abc.c 2016-09-12 14:35:25.920931214 +0900  
+++ greetings-1.0-mod5r/lib/abc.c 1970-01-01 09:00:00.000000000 +0900  
@@ -1,6 +0,0 @@  
-#include  
-  
-void abc_initialize()  
-{  
- // Code later  
-}  
```

Khi apply patch, thì tương tự như trên ta cần thêm **_p0_**

```bash  
$patch -u -p1 <../patch_1.0-mod5r.patch  
patching file lib/abc.c  
```