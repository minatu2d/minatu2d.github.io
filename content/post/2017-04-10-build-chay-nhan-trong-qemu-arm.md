---
layout: post
title: Build, chạy nhân Linux trong QEMU ARM
date: 2017-04-10 14:07:48.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
toc : true
status: publish
categories:
- Basic
- Compile
tags:
- busybox
- qemu
- rootfs
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '3838876360'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2017/04/10/build-chay-nhan-trong-qemu-arm/"
---
Chạy một bản Linux tối giản trên Qemu ARM

Bài này sẽ làm một ví dụ để chỉ ra sự liên quan giữa  
các thành phần của một hệ thống Linux thông qua  
việc build, chạy Kernel trên board **vexpress-a9** mô phỏng bằng Qemu.

Đó là Kernel, Root file system, Busybox, Init.

* * *

## 1. "Chém" chút về quá trình khởi động của Linux


*   Về cơ bản, quá trình khởi động Linux có 2 giai đoạn.  
    Giai đoạn đầu là load kernel lên RAM và chạy  
    Giai đoạn thứ hai là kernle sẽ tự động mount hệ thống file  
    rồi các ứng dụng trên hệ thống file được mount.  
    Bất cứ bản Linux nào, dù lớn, dù bé đều thì tối thiểu phải có 2 giai đoạn ở trên.  
    
*   Để chạy Linux trên các board sử dụng chip ARM, ta cần 2 điều kiện  
    Thứ nhất, kernel hỗ trợ board.  
    Thứ hai, ta phải có hoặc tạo file device tree chứa thông tin  
    các device mà kernel có thể truy cập được khi kernel chạy
    

## 2. Chuẩn bị môi trường

Ubuntu 16.04 LTS hoặc các môi trường Linux tương tự và tương đối mới  
Giả sử ta sẽ thao tác một thư mục tên có tên **qemu_arm_test** trong HOME

## 3. Tải về

*   Linux kernel bản stable mới nhất  
    Vào trang [http://www.kernel.org/](http://www.kernel.org/), rồi tải bản stable mới nhất  
    Hoặc lấy link rồi sử dụng wget như sau:

```bash  
$ wget https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.10.8.tar.xz  
```

*   Busybox bản stable mới nhất  
    Vào trang [http://busybox.net](http://busybox.net), rồi tải bản stable mới nhất  
    Hoặc lấy link rồi sử dụng wget như sau:

```bash  
$ wget http://busybox.net/downloads/busybox-1.26.2.tar.bz2  
```

## 4. Cài đặt
-----------

*   Cài đặt cross-compiler chạy trên Linux, build code cho ARM  
    Có rất nhiều loại Toolchain dành cho ARM mà chúng ta có thể cài trên Linux.  
    Danh sách có thể tham khảo tại [đây](http://elinux.org/Toolchains).  
    Ở đây chúng ta sẽ sử dụng Linaro (ARM) vì cài đặt khá dễ dàng trên Ubuntu.  
    Câu lệnh như sau:

```bash  
$sudo add-apt-repository ppa:linaro-maintainers/toolchain  
$sudo apt-get install gcc-arm-linux-gnueabi  
```

Sau khi cài đặt xong, ta có thể thấy các command sau:  
**arm-linux-gnueabi-gcc**, **arm-none-eabi-gcc**

*   Cài đặt Qemu ARM để mô phỏng board **vexpress-a9**  
    Qemu thì quá nổi tiếng rồi, một phần mềm máy ảo là nền tảng cho hầu hết các công  
    nghệ ảo phần cứng ngày nay.

```bash  
$sudo apt-get install qemu qemu-system-arm  
```

Để khởi động thử board **vexpress-a9**

```bash  
$qemu-system-arm -M vexpress-a9 -m 512  
```

Nếu lên được một màn hình windows rồi tắt là ok, chứng tỏ các tham số là tạm ổn.  
Chỉ là máy chưa biết chạy cái gì thôi.

## 5. Biên dịch

Ta sẽ tiến hành build kernel rồi Busybox

### 5.1 Biên dịch Kernel từ MainStream

*   Giải nén kernel

```bash  
$unxz linux-4.10.8.tar.xz  
$tar -xvf linux-4.10.8.tar  
```

*   Cấu hình và build kernel bằng Toolchain ta đã cài ở trên

```bash  
$cd linux-4.10.8  
$make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- vexpress_defconfig  
$make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- all  
```

Khi khi quá trình build kết thúc, bạn sẽ thấy nhân được tạo trong  
đường dẫn _linux-4.10.8/arch/arm/boot/zImage_.  
Chốc nữa ta sẽ khởi động bằng _zImage_ đó.

### 5.2 Biên dịch Busybox từ MainStream

*   Giải nén

```bash  
$bzip2 -d -k busybox-1.26.2.tar.bz2  
```

*   Cấu hình và build Busybox bằng Toolchain đã cài ở trên  
    Sử dụng cấu hình mặc định

```text  
$cd busybox-1.26.2  
$make ARCH=arm CROSS_COMPLIE=arm-linux-gnueabi- defconfig  
```

Chúng ta sẽ không sử dụng shared library (tức là không có run-tim loader-linker)  
vì thế phải sửa cấu hình build của Busybox để chỉ hỗ trợ buildstatic thôi.

```bash  
$make ARCH=arm CROSS_COMPLIE=arm-linux-gnueabi- menuconfig  
```

Chú ý là, có thể phải cài thư viện _libcurses-dev_ thì mới chạy được  
giao diện cấu hình.

Sau đó chỉnh sửa ở đường dẫn sau:  
_Busybox Settings > Build Options > Build BusyBox as a static binary (no shared libs)_

Sau đó build Busybox bằng câu lệnh sau:

```bash  
$make ARCH=arm CROSS_COMPLIE=arm-linux-gnueabi- install  
```

Kết quả build sẽ nằm ở thư mục __install_.  
Nội dung thư mục này sẽ như sau:

```bash  
_install$ ls  
bin linuxrc sbin usr  
```

## 6. Tạo file system chứa Busybox

Như đã nói ở trên, ta cần một file system để chứa ứng dụng phía user-space.  
Ít nhất phải có **init program**, là chương trình đầu tiên mà kernel sẽ chạy.  
Process của chương trình này luôn là 1, và cha của mọi tiến trình trong Linux.

Vậy, một file system tối thiệu liệu cần những gì.  
Thật may mắn, kết quả sau khi build Busybox đã cho ta gần như đủ 1 file system rồi.  
Tuy nhiên, ta cần một vài file cấu hình cho **init proram** nữa.

Được dẫn của **init program** trong file system có thể được chỉ định trong tham  
số khởi động kernel.Nếu không được chỉ định thì nó sẽ tìm theo thức tự sau:  
Theo source code từ [kernel/init/main.c:990](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git/tree/init/main.c#n990)

```cpp  
if (!try_to_run_init_process("/sbin/init") ||  
!try_to_run_init_process("/etc/init") ||  
!try_to_run_init_process("/bin/init") ||  
!try_to_run_init_process("/bin/sh"))  
```

Busybox là một tập các utility của Linux, nó cũng có một **init program** nằm ở _/sbin/init_ luôn. Ta có thể xem trong nội dung thư mục *_install** ở trên.

Trong bài này, ta sẽ tham khảo ví dụ tạo file system của Busybox cho đĩa mềm (floppy disk).  
Cụ thể sẽ làm như sau:  
Tại thư mục source của _Busybox_:

```cpp  
$cd examples/bootfloopy  
$ls  
bootfloppy.txt etc mkrootfs.sh quickstart.txt  
display.txt mkdevs.sh mksyslinux.sh syslinux.cfg  
```

Ta sẽ sử dụng thư mục _etc_ và thư mục __install_ (được tạo khi build Busybox)  
để tạo file system.

Đầu tiên, tạo thư mục chứa nội dung toàn bộ file system:

```bash  
$mkdir rootfs_content  
$cp -fR busybox-1.26.2/examples/etc rootfs_content  
$cp -fR busybox-1.26.2/_install/* rootfs_content  
$ls rootfs_content  
bin etc linuxrc sbin usr  
```

Ta sẽ tạo thêm một vài thư mục phụ nữa:

```bash  
$cd rootfs_content  
$mkdir dev home mnt proc root sys tmp var  
```

Cuối cùng ta có cấu trúc thư mục như sau:

```text  
├── bin  
├── dev  
├── etc  
├── home  
├── linuxrc -> bin/busybox  
├── mnt  
├── proc  
├── root  
├── sbin  
├── sys  
├── tmp  
├── usr  
└── var  
```

Ta cần sửa một chút file cấu hình của **init program**.  
Nó nằm ở _/etc/inittab_, sửa nội dung thành như sau:

```text  
::sysinit:/etc/init.d/rcS  
::respawn:-/bin/sh  
::ctrlaltdel:/bin/umount -a -r  
```

Giờ ta sẽ tạo file system từ thư mục **rootfs_content** ở trên.  
Đầu tiên, tạo một file system ext3 trống:

```bash  
$dd if=/dev/zero of=BusyBoxRootFS.ext3 bs=1M count=32  
$mkfs.ext3 BusyBoxRootFS.ext3  
```

Sau đó, mount file system vừa tạo trong file _BusyBoxRootFS.ext3_  
vào để tạo nội dung cho nó (nhớ phải có tham số -o nha):

```text  
$sudo mount -t ext3 BusyBoxRootFS.ext3 tmpfs/ -o loop  
$cp -fR rootfs_content/* tmpfs/  
```

Cuối cùng là _umount_ nó:

```bash  
$umount tmpfs  
```

Giờ file hệ thống chứa các ứng dụng user-space nằm trong file system bên trong file _BusyboxRootFS.ext3_ rồi.

## 7. Chạy

Ta kiểm tra lại những thức đã làm trước khi chạy  
**qemu-system-arm** : đã cài  
**kernel** : đã build  
**file system** : đã tạo

Giờ ta sẽ chạy, câu lệnh như sau:

```text  
$qemu-system-arm -M vexpress-a9 \  
-dtb linux-4.10.8/arch/arm/boot/dts/vexpress-v2p-ca9.dtb \  
-kernel linux-4.10.8/arch/arm/boot/zImage \  
-append "root=/dev/mmcblk0 console=ttyAMA0" \  
-sd BusyBoxRootFS.ext3 \  
-serial stdio  
```

![QEMU](/post/images/qemu_arm.png)

Trên cửa sổ dòng lệnh sẽ hiện ra _terminal_ của máy Qemu:

Cùng với một màn hình có chim cánh cụt:

Ta kiểm tra máy đang chạy bằng

```text  
#uname -a  
#cat /proc/cpuinfo  
```

Sẽ thấy máy chạy là ARM, và phiên bản nhân là 4.10.8 (mới nhất)

## 8. Tham khảo

1.  Learning from you blog - https://learningfromyoublog.wordpress.com/
2.  Free Embedded - https://learningfromyoublog.wordpress.com/
3.  Busybox - https://www.busybox.net/
4.  BusyBox simplifies embedded Linux systems - https://www.ibm.com/developerworks/library/l-busybox/
5.  Linux stable - https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git/
6.  QEMU System ARM - https://qemu.weilnetz.de/doc/qemu-doc.html#ARM-System-emulator