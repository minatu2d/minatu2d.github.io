---
layout: post
title: Cài đặt GDB (02 - Cài đặt)
date: 2016-09-20 08:42:17.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
- Debug
tags:
- GDB
- Installation
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '27012952747'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/09/20/cai-dat-gdb-02-cai-dat/"
---
Khi sử dụng GDB để debug 1 chương trình thì chương trình đó gọi là **target program**.  
Khi nói về vị trí của GDB dùng để debug và target program, ta sẽ có 2 cách trường hợp sử dụng sau:

1.  GDB và **target program** cùng ở 1 máy : Thường sử dụng với chính các chương trình được dev, rồi build, rồi chạy trên máy đó. Đây là trường hợp chúng ta hay thấy nhất, đó là khi phát triển các app desktop.
2.  GDB và **target program** không cùng trên 1 máy : Tức là GDB sẽ chạy trên 1 máy để debug 1 chương trình chạy trên máy khác thông qua Serial hoặc Network. Cái này hay thấy khi phát triển ứng dụng nhúng. Xem thêm tại [đây](https://lazytrick.wordpress.com/2016/08/03/cac-khai-niem-trong-cross-compiling/).

Một số distro khi được cài đặt, đã bao gồm **GDB** rồi. Có thể kiểm tra bằng command dưới đây:

```bash  
$gdb --version  
```

Dưới đây, sẽ đưa ra hướng dẫn cách cài đặt GDB trên máy sử dụng để debug. Nó có thể là máy chạy **target program**, có thể không.

## 1. Cài đặt trên Debian (Debian-based distro)

Debian 8.x chưa được cài mặc định GDB, hãy sử dụng command sau để cài nó:

```bash  
#aptitude install gdb  
```

Chú ý rằng, ở đây phải sử dụng quyền **_root_** hoặc **_sudo_**.  
Trên một số bản Debian như 8.x thì hoàn toàn có thể dùng cách cài đặt như Ubuntu được.

## 2. Cài đặt trên Ubuntu (Ubuntu-based distro)

Trên các bản Ubuntu gần đây như [14.04](http://releases.ubuntu.com/trusty/ubuntu-14.04.4-desktop-i386.manifest), [16.04](http://releases.ubuntu.com/xenial/ubuntu-16.04-desktop-i386.manifest) **GDB** đều được được cài mặc định.  
Hoặc nếu chưa được cài, thì hãy chạy command sau để cài nó :

```bash  
$sudo apt-get install gdb  
```

### 2. Cài đặt trên CentOS, Fedora

Trên CentOS, Fedora, ta cài bằng lệnh sau dưới quyền root.

```bash  
#yum install gdb  
```

Nếu có đĩa DVD cài đặt, có thể tải gói cài từ DVD bằng lệnh sau:

```bash  
#yum install --disablerepo=\* --enablerepo=c7-media install gdb  
```

## 3. Cài đặt trên Raspbian

Raspbian dựa trên Debian, có thể cài theo cách cài trên Ubuntu hoặc Debian đều được.

```bash  
#aptitude install gdb  
```

hoặc

```bash  
$sudo apt-get install gdb  
```

## 4. Cài đặt từ source code

Ta có thể tải một bản gdb bất kì tại [đây](ftp://sourceware.org/pub/gdb/releases/) để tiến hành cài đặt.

Ở đây, bản mới nhất là **7.11.1** để build, cài đặt.  
Chú ý là, ta có thể cài đặt **GDB** trên 1 đường dẫn bất kì của user đang sử dụng mà không cần phải cài vào các đường dẫn mặc định như **/usr/bin**, hay **/usr/local/bin**, etc.  
Và quá trình này không cần phải là **root**.  
Thực hiện với quyền root nhiều khi còn phát sinh vấn đề.

### 4.1. Tạo thư mục để build

Command:

```bash  
$ #1. Creating local directory  
$pwd  
/home/oedev  
$mkdir gdb  
$cd gdb  
$pwd  
/home/oedev/gdb  
$mkdir gdb_install  
```

Ta sẽ cài đặt **GDB** vào ***gdb_install**.

### 4.2. Tải source code

```bash  
$ #2. Download source code  
$wget ftp://sourceware.org/pub/gdb/releases/gdb-7.11.1.tar.gz  
```

### 4.3. Giải nén source code

```bash  
$ #3. Extrating the source code  
$tar -xzvf gdb-7.11.1.tar.gz  
$ls  
gdb-7.11.1 gdb-7.11.1.tar.gz  
$cd gdb-7.11.1  
```

### 4.4. Cấu hình (Configure)

Khi ta chỉ chạy GDB trên chính máy hiện tại, và debug các chương trình chạy trên máy này thôi thì command đơn giản là :

```bash  
$./configure --prefix=/home/oedev/gdb/gdb_install  
```

Sẽ mất vài giây để chạy xong.

### 4.5. Biên dịch (Compiling)

Sau khi **configure**, thì sẽ là **make**:

```bash  
$make  
```

Cần 5-10 phút để build xong.

### 4.6. Cài đặt (Installing)

Vì ta đã cấu hình đường dẫn ở bước configure rồi nên ở đây, chỉ cần chạy lệnh sau là **GDB** sẽ được cài đặt vào được dẫn đó (không cần quyền root):

```bash  
$make install  
```

Kiểm tra thư mục cài đặt

```bash  
oedev@OEDEV-Ubuntu:~/gdb/gdb_install$ tree -L 2  
.  
├── bin  
│   ├── gcore  
│   ├── gdb  
│   └── gdbserver  
├── include  
│   ├── ansidecl.h  
│   ├── bfd.h  
│   ├── bfdlink.h  
│   ├── dis-asm.h  
│   ├── gdb  
│   ├── plugin-api.h  
│   └── symcat.h  
├── lib  
│   ├── libbfd.a  
│   ├── libbfd.la  
│   ├── libinproctrace.so  
│   ├── libopcodes.a  
│   └── libopcodes.la  
└── share  
├── gdb  
├── info  
├── locale  
└── man  
```

Giờ ta có thể sử dụng các file trong thư mục bin rồi.  
Ta chủ yếu sử dụng 2 file là **bin/gdb** và **bin/gdbserver**.  
Chi tiết sẽ được nói trong các bài tiếp theo.