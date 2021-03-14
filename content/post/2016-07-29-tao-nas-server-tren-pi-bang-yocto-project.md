---
layout: post
title: Tạo một bản build Linux cho Raspberry PI bằng Yocto Project
date: 2016-07-29 08:06:07.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Embedded
- OpenEmbbeded
- YoctoProject
tags:
- Kernel
- LinuxImage
- RaspberryPI
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '25266356184'
  _oembed_99705e2ddd8703aa15dfd14465bdf35c: "{{unknown}}"
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/07/29/tao-nas-server-tren-pi-bang-yocto-project/"
---
Ban đầu, dự định sẽ tạo một NAS server theo link tham khảo bên dưới.  
Nhưng thấy ta nên tách riêng phần tạo bản phân phối Linux thành 1 bài riêng, rồi viết các nội dung liên quan đến customize thành các bài khác sẽ dễ hiểu hơn.  
Hơn nữa, phần tạo bản build basic sẽ cần được thảo luận kĩ hơn do có thể phát sinh nhiều vấn đề. Mà nếu không giải quyết được các vấn đề đó thì nội dung các bài khác sẽ không thể thực hiện được.  
Do vậy, bài này sẽ tập trung vào tạo bản basic **rpi-basic-image** thôi.

Link tham khảo:  
[http://mickey-happygolucky.hatenablog.com/entry/2015/02/14/002035](http://mickey-happygolucky.hatenablog.com/entry/2015/02/14/002035)

## 1. Tạo một bản build cho Raspberry PI từ Ubuntu bằng Yocto Project

Mục đích của bước này là tạo ra được một ảnh đĩa (file .img giống như Raspabian cung cấp cho người dùng tại [đây](https://www.raspberrypi.org/downloads/raspbian/), sau khi đã giải nén). Sử dụng kết quả của [Yocto Project](https://www.yoctoproject.org/) - một khung (framework) cho phép tạo ra một bản Linux có thể tùy biến được. Yocto Project cung cấp một bản phân phối Linux (gọi là Poky), người dùng có thể build cho rất nhiều target (gồm mainboard, chíp,etc) khác nhau.

Việc build một bản Linux cho RPI trong trường hợp này là quá trình build Poky cho board Raspberry PI, chip là ARM (không nhớ phiên bản lắm).

Nhưng thông tin có vẻ nghe khó hiểu và phức tạp, nhưng ta chỉ hiểu rằng ta Raspberry là một trong nhiều target mà Yocto Project hỗ trợ build OS cho. Vậy thôi, bắt đầu nào. Nó đơn giản hơn chúng ta nghĩ rất nhiều, chỉ tốn thời gian và khoảng trống ổ đĩa thôi.

À, cần ít nhất khoảng 30GB trước khi thực hiện các bước này, một mạng có tốc độ tốt sẽ giúp giảm thời gian hoàn thành các bước bên dưới còn nửa ngày chẳng hạn :)))

*   Trước khi bắt đầu, ta nên cài các gói cần thiết như dưới để đỡ phải cài lúc gặp lỗi:

```bash  
$ sudo apt-get install gawk wget git-core diffstat unzip texinfo gcc-multilib \  
build-essential chrpath socat libsdl1.2-dev xterm  
```

*   Tạo thư mục cần build, hiện tại **krogoth** đang là bản mới nhất:

```bash  
$mkdir raspberrpi_krogoth  
```

*   Tải các Poky, như cách mọi người vẫn làm với **git** :

```bash  
$cd raspberrpi_krogoth  
$  
$git clone -b krogoth git://git.yoctoproject.org/poky.git  
```

*   Như ta đã nói ở phần đầu, Poky là một bản phân phối Linux của Yocto Project, mặc định của nó không hỗ trợ RPI (chỉ có Texas Instruments Beaglebone - beaglebone và Freescale MPC8315E-RDB - mpc8315e-rdb mà thôi, xem thêm tại [đây](http://git.yoctoproject.org/cgit/cgit.cgi/poky/tree/README.hardware?h=krogoth)). Vì thế, ta phải tải phần hỗ trợ liên quan đến bo mạch (BSP) của RPI  
    **jethro** là bản mới nhất

```bash  
$cd poky  
$  
$git clone -b jethro git://git.yoctoproject.org/meta-raspberrypi  
```

Đến đây, ta đã được 1 nửa về mặt thao tác phải làm, nhưng chỉ mới 1/10 về mặt thời gian phải đợi. Xin hãy từ từ :))

Tiếp theo, ta bắt đầu vào build.  
- Thiết lập các biến môi trường  
Do poky đã cung cấp sẵn script thiết lập các biến này rồi.  
Ta chỉ cần chạy.

```bash  
$source poky/oe-init-build-env build  
```

(có thể không cần **buid** ở cuối)  
Sau khi chạy xong lệnh trên, đường dẫn sẽ tự động chuyển về thư mục build của Yocto. Ta bắt đầu làm mọi thứ từ đây.  
Nó có dạng kiểu như thế này:

```bash  
oedev@OEDEV-Ubuntu:~/raspberrypi_krogoth/build$  
```

*   Poky chia các gói phần mềm thành các layer (lớp), gói hỗ trợ RPI cũng vậy, vì không được hỗ trợ mặc định nên ta phải thêm vào bằng cách:

Mở file **build/conf/bblayers.conf** bằng _nano_, _vi_ hoặc bất cứ editor nào.  
Rồi thêm đường dẫn đến gói hỗ trợ bo mạch RPI.  
Sau khi sửa, nó sẽ có nội dung kiểu như sau:

```bash  
oedev@OEDEV-Ubuntu:~/raspberrypi_krogoth/build$ cat conf/bblayers.conf

# POKY_BBLAYERS_CONF_VERSION is increased each time build/conf/bblayers.conf  
# changes incompatibly  
POKY_BBLAYERS_CONF_VERSION = &amp;quot;2&amp;quot;

BBPATH = &amp;quot;${TOPDIR}&amp;quot;  
BBFILES ?= &amp;quot;&amp;quot;

BBLAYERS ?= &amp;quot; \  
/home/oedev/raspberrypi_krogoth/poky/meta \  
/home/oedev/raspberrypi_krogoth/poky/meta-poky \  
/home/oedev/raspberrypi_krogoth/poky/meta-yocto-bsp \  
/home/oedev/raspberrypi_krogoth/poky/meta-raspberrypi \ # mới thêm vào  
&amp;quot;  
```

*   Tiếp theo, ta phải thay đổi Target Machine,

Bản phân phối Poky của Yocto Project hỗ trợ rất nhiều Target Machine, trong đó có những Board mặc định của TI, Freescale hoặc chương trình mô phỏng phần cứng Qemu.

Raspberry PI không được hỗ trợ mặc định nên ta phải thêm nó vào.

Mở file **build/conf/local.conf **và sửa như sau:

```bash  
....  
#MACHINE ?= "qemuarm64"  
#MACHINE ?= "qemumips"  
#MACHINE ?= "qemumips64"  
#MACHINE ?= "qemuppc"  
#MACHINE ?= "qemux86"  
#MACHINE ?= "qemux86-64"  
#  
# There are also the following hardware board target machines included for  
# demonstration purposes:  
#  
#MACHINE ?= "beaglebone"  
#MACHINE ?= "genericx86"  
#MACHINE ?= "genericx86-64"  
#MACHINE ?= "mpc8315e-rdb"  
#MACHINE ?= "edgerouter"  
#  
# This sets the default machine to be qemux86 if no other machine is selected:  
MACHINE ??= "qemux86"

# Thêm vào cho RaspberryPI  
MACHINE ?= "raspberrypi"  
....  
```

*   Giờ bắt đầu build và bỏ máy, ở phần chạy lệnh **source** phía trên, ta có thể thấy một đoạn được hiện ra với hướng dẫn kiểu **bitbake XXX**.  
    Ta cũng làm tương tự như vậy

```bash  
$bitbake rpi-basic-image  
```

Bước này sẽ thực hiện việc tải các source cần thiết, thực hiện các sửa, thực hiện tạo compiler, build source, rồi tạo image.  
**rpi-basic-image** : là 1 một images mà framework có thể tạo, ta có thể xem danh sách các images khác bằng đường dẫn bên dưới đây:

```bash  
oedev@OEDEV-Ubuntu:~/raspberrypi_krogoth/poky/meta-raspberrypi$ ls recipes-core/images/  
rpi-basic-image.bb rpi-hwup-image.bb rpi-test-image.bb  
```

Trong quá trình thực hiện, có thể gặp phải lỗi kiểu như sau:

Error.01 : Việc tải bị timeout: chỉ có cách thực hiện lại một vài lần  
Tức là chạy lại câu lệnh **bitbake rpi-basic-image** ở phía trên.  
Error.02 : Không tìm thấy file **mkknlimg**:  
Lỗi sẽ hiện ra kiểu như thế này:

```bash  
DEBUG: Executing shell function do_rpiboot_mkimage  
/build/tmp/sysroots/x86_64-linux/usr/lib/rpi-mkimage/mkknlimg: not found  
WARNING: exit code 127 from a shell command.  
```

Đây là lỗi đường dẫn, hãy sửa bằng cách:

```bash  
#Tìm kiếm mkknlimg xem có ở đâu đó không.  
$ locate mkknlimg  
```

Sẽ có kết quả dạng:  
_.../tmp/sysroots/x86_64-linux/usr/libexec/mkknlimg_  
Ta đi sửa cho đúng là ok.  
Mở file _poky/meta-raspberrypi/recipes-kernel/linux/linux-raspberrypi.inc_  
Tìm đến hàm _do_rpiboot_mkimage()_ sửa thành như sau:

```python
#Comment dòng cũ này: ${STAGING_DIR_NATIVE}/usr/lib/rpi-mkimage/mkknlimg ...  
${STAGING_DIR_NATIVE}/usr/libexec/mkknlimg ...  
```

Error.03 : Việc tải **bcm2835-bootfiles** dường như bị đứng lại, thực ra không phải vậy, nó nặng đến 3GB, nếu đường truyền chậm thì rất có thể phải đợi rất lâu, ta có cả giác nó không hề làm việc. Nhưng hãy cố gắng đợi nha. :))

Error.04 : Lỗi khi kiểm tra check sum của image được build ra, lỗi này được sửa bằng cách thêm 1 dòng vào file dưới đây:

_poky/meta-raspberrypi/classes/sdcard_image-rpi.bbclass_

Nội dung đoạn thêm sẽ như thế này:

```python 
IMAGEDATESTAMP = "${@time.strftime('%Y.%m.%d',time.gmtime())}"  
#Nội dung thêm vào:  
IMAGE_CMD_rpi-sdimg[vardepsexclude] = "IMAGEDATESTAMP"  
```

Sau khi thực hiện xong bước build images, ta sẽ thu được images trong thư mục  
**build/tmp/deploy/images/raspberrypi** với tên **rpi-basic-image-raspberrypi.rpi-sdimg**.

Ta có thể ghi vào thẻ nhớ để để boot RPI bình thường

Việc ghi ra thẻ nhớ thì làm theo hướng dẫn dành cho Raspbian ở link [này](https://www.raspberrypi.org/documentation/installation/installing-images/linux.md).

Update: 2017/02/14: Cám ơn bạn HocARM review và phát phát hiện ra chỗ thiếu phần configure Target Machine.