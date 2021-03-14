---
layout: post
title: Tìm Driver cho Linux
date: 2016-08-19 01:38:50.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
- Embedded
tags:
- Driver
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '25927133903'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/08/19/xac-dinh-cac-driver-module-dang-duoc-su-dung-tren-linux/"
---
Trong quá trình tìm cách cài đặt driver cho [bản build Raspberry PI](https://lazytrick.wordpress.com/2016/07/29/tao-nas-server-tren-pi-bang-yocto-project/) sử dụng Yocto, mình có tìm hiểu driver trong Linux và tìm được cuốn sách **Linux in A Nutshell** (link tại [đây](http://files.kroah.com/lkn/) ) của pro này.

Chương 7, chương mà tác giả đặc biệt "tự hào", nói về Customize một Linux Kernel.  
Trong chương này, tác giả cũng nói đến việc xác định các driver đang được sử dụng trên hệ thống hiện tại.  
  
Bài này xin tóm tắt nội dung đó, như một phần của việc tìm hiểu để chuyển Driver cho USB Wifi Client từ Raspbian sang [bản Linux tự build](https://lazytrick.wordpress.com/2016/07/29/tao-nas-server-tren-pi-bang-yocto-project/).

Về cơ bản, mỗi khi driver (module) được load, một chương trình là **udev** sẽ tạo một Node (một File, thư mục) trong thư mục **_/sys/_** chứa tất tần tật mọi thông tin cần thiết để các thành phần khác có thể sử dụng được thiết bị mà Driver đó điều khiển.

1.  Driver về Network
2.  Thiết bị PCI
3.  Driver cho thiết bị USB

## 1. Driver về Network

Các card mạng hiện có sẽ nằm dưới đường dẫn **/sys/class/net/**.

```bash  
$ ls /sys/class/net/  
eth0 eth1 eth2 lo  
```

Đây chính là các card mạng mà ta vẫn thấy khi sử sụng

```bash  
$ifconfig -a  
```

Để xác định Driver đang điều khiển các card mạng ở trên, ta làm như sau:

Với **eth0** chẳng hạn:

```bash  
$ basename \`readlink /sys/class/net/eth0/device/driver/module\`  
e1000  
```

Kết quả **e100** đó chính là tên Driver đang được sử dụng để điều khiển card mạng **eth0**

## 2. Driver cho thiết bị PCI

PCI là giao tiếp dạng Bus. Các thiết bị giao tiếp chuẩn này thường được gắn sẵn trên board của PC hoặc Laptop rồi. Ví dụ như card sound, network cũng thế.  
Mỗi thiết bị PCI được phân biệt bởi **VendorID** (mã nhà sản xuất) và **DeviceID** (mã thiết bị hay sản phẩm).  
Xác định driver của thiết bị PCI sẽ phức tạp hơn 1 chút.

Đầu tiên, liệt kê tất cả các thiết bị PCI có trong máy (có thể có những thiết bị chưa chạy)

```bash  
$lspci  
```

Kết quả câu lệnh trên sẽ cho ta một danh sách thiết bị. Giả sử trong đó có thiết bị sau:

```bash  
06:04.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL-8139/  
8139C/8139C+ (rev 10)  
```

Hãy để ý chuỗi **06:04.0** vì ta sẽ sử dụng nó để tìm kiếm trong **/sys/**

Xem danh sách các thiết bị có trong **/sys/**:

```bash  
$ cd /sys/bus/pci/devices/  
$ ls  
0000:00:00.0 0000:00:1d.0 0000:00:1e.0 0000:00:1f.3 0000:06:03.3  
0000:00:02.0 0000:00:1d.1 0000:00:1f.0 0000:06:03.0 0000:06:03.4  
0000:00:02.1 0000:00:1d.2 0000:00:1f.1 0000:06:03.1 0000:06:04.0  
0000:00:1b.0 0000:00:1d.7 0000:00:1f.2 0000:06:03.2 0000:06:05.0  
```

Ta thấy **06:04.0** cũng xuất hiện trong kết quả trên, vào đó xem sao:

```bash  
$ cd 0000:06:04.0  
```

Trong **0000:06:04.0**, có khá nhiều file, nhưng ta chỉ quan tâm đến 2 giá trị.  
Đó là **Vendor** và **Device**, chính là VendorID, và DeviceID mà ta cần tìm.

```bash  
$ cat vendor  
0x10ec  
$ cat device  
0x8139  
```

Từ 2 giá trị đã tìm được ở trên, ta sẽ tìm Driver thông qua source của Kernel.

**Bằng VendorID**:

```bash  
$ cd ~/linux/linux-2.6.17.8/  
$ grep -i 0x10ec include/linux/pci_ids.h  
#define PCI_VENDOR_ID_REALTEK 0x10ec  
```

**Bằng DeviceID**:

```bash  
$ cd ~/linux/linux-2.6.17.8/  
$ grep -i 0x8139 include/linux/pci_ids.h  
#define PCI_DEVICE_ID_REALTEK_8139 0x8139  
```

Ta thấy rằng **PCI_VENDOR_ID_REALTEK**, **PCI_DEVICE_ID_REALTEK_8139** là định nghĩa liên quan đến thiết bị ta cần tìm.

Tiếp tục tìm kiếm trong source code của nhân để xem có driver cho thiết bị của ta không.

```bash  
$ grep -Rl PCI_VENDOR_ID_REALTEK *  
include/linux/pci_ids.h  
drivers/net/r8169.c  
drivers/net/8139too.c  
drivers/net/8139cp.c  
```

Mỗi Driver PCI sẽ chưa danh sách các thiết bị mà nó hỗ trợ. Vì thế.  
Ngoài file .h đầu tiên ta vừa thấy ở trên, ta lần lượt xem 3 file .c để xem thiết bị của chúng ta có được hỗ trợ không.

Đầu tiên,  
**drivers/net/r8169.c**  
Có 1 đoạn định nghĩa như sau:

```C  
static struct pci_device_id rtl8169_pci_tbl[] = {  
{ PCI_DEVICE(PCI_VENDOR_ID_REALTEK, 0x8169), },  
{ PCI_DEVICE(PCI_VENDOR_ID_REALTEK, 0x8129), },  
{ PCI_DEVICE(PCI_VENDOR_ID_DLINK, 0x4300), },  
{ PCI_DEVICE(0x16ec, 0x0116), },  
{ PCI_VENDOR_ID_LINKSYS, 0x1032, PCI_ANY_ID, 0x0024, },  
{0,},  
};  
```

Tham số tư 2 sau, VendorID là Device. Không thấy có DeviceID (0x8139) cho thiết bị của chúng.  
-> Driver này không hỗ trợ thiết bị ta đang tìm rồi.

Thứ 2,  
**drivers/net/8139too.c**  
Có đoạn sau sử dụng định nghĩa Vendor

```C  
if (pdev->vendor == PCI_VENDOR_ID_REALTEK &&  
pdev->device == PCI_DEVICE_ID_REALTEK_8139 && pci_rev >= 0x20) {  
dev_info(&pdev->dev,  
"This (id %04x:%04x rev %02x) is an enhanced 8139C+ chip\n",  
pdev->vendor, pdev->device, pci_rev);  
dev_info(&pdev->dev,  
"Use the \"8139cp\" driver for improved performance and  
stability.\n");  
}  
```

Đoạn comment sau có ý rằng nên dùng Driver 8139cp cho DeviceID 8139.  
Ồ, nó đây chăng.

```C  
Use the 8139cp driver for improved performance and  
stability.  
```

Thứ 3,  
**drivers/net/8139cp.c**  
Có đoạn code sau:

```C  
static struct pci_device_id cp_pci_tbl[] = {  
{ PCI_VENDOR_ID_REALTEK, PCI_DEVICE_ID_REALTEK_8139,  
PCI_ANY_ID, PCI_ANY_ID, 0, 0, },  
{ PCI_VENDOR_ID_TTTECH, PCI_DEVICE_ID_TTTECH_MC322,  
PCI_ANY_ID, PCI_ANY_ID, 0, 0, },  
{ },  
};  
MODULE_DEVICE_TABLE(pci, cp_pci_tbl);  
```

Đúng nó đây rồi, cùng VendorID, Device mà chúng ta đang tìm luôn.

Vậy là ta đã biết Driver nào support thiết bị của chúng ta ở trên.

Việc sau cùng là build lại kernel với Driver đó được bật.

Tác giả Kroah cũng đưa ra các step để thực hiện việc tìm Driver cho thiết bị PCI như sau:  
1. Xác định PCI bus ID của thiết bị muốn tìm driver, sử dụng câu lệnh **_lspci_**.  
2. Chuyển vào đường dẫn thư mục **_/sys/bus/pci/devices/0000:<bus_id>_**, với **_bus_id_** là PCI bus ID tìm được ở Bước 1.  
3. Xác định giá trị ID của vendor và device trong thư mục chuyển vào ở Bước 2.  
4. Tìm trong file **_<kernel source>/include/linux/pci_ids.h_**, xem có định nghĩa nào cho giá trị của ID của PCI vendor và device mà ta đã xác định được ở Bước 3 không.  
5. Tìm tất cả những file chưa tham chiếu đến các định nghĩa tìm được ở Bước 4. Vendor và DeviceId thường sẽ nằm trong 1 cấu trúc của thiết bị PCI, có tên là **struct pci_device_id**. Thường mỗi file .c sẽ tương ứng là 1 Driver.  
6. Tìm trong **_Makefiles_** cấu hình source của kernel xem có định nghĩa nào dạng **CONFIG_** cho driver ta cần build không. Ta có thể tìm bằng lệnh dưới đây:

```bash  
$ find -type f -name Makefile | xargs grep DRIVER_NAME  
```

7.Tìm đến giá trị **CONFIG_** trong kernel để cho phép driver được build khi build lại kernel.

## 3. Thiết bị USB

Việc tìm kiếm Driver cho thiết bị sử dụng giao tiếp USB khá giống với PCI ở phần 2.

Các bước tượng tự như sau:  
1. Xác định ID USB vendor và product ID của thiết bị bạn muốn tìm Driver. Sử dụng câu lệnh **lsusb** ở trước và sau khi thiết bị được cắm vào máy.  
2. Tìm trong source code của kernel xem có ID của vendor và product của thiết bị ta đã tìm ở Bước 1 không. Cả Vendor ID và Product ID sẽ nằm trong 1 cấu trúc mô tả thiết bị USB, có tên là **struct usb_device_id**. Từ đó xác định được Driver phải build để hỗ trợ thiết bị.  
3. Tìm trong Makefiles xem có định nghĩa **CONFIG_** cho Driver ở Bước 2 không, sử dụng câu lệnh:

```bash  
$ find -type f -name Makefile | xargs grep DRIVER_NAME  
```

4.Mở lại cấu hình build kernel và bật các định nghĩa tìm được ở Bước 3.