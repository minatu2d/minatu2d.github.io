---
layout: post
title: Tìm source code của USB Driver trên Linux
date: 2017-04-23 04:10:05.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
- Driver
- Firmware
- USB
tags: []
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '4291100745'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2017/04/23/tim-source-code-cua-usb-driver-tren-linux/"
---
Ở bài [Tìm driver cho Linux](https://lazytrick.wordpress.com/2016/08/19/xac-dinh-cac-driver-module-dang-duoc-su-dung-tren-linux/) cũng đã nói qua rồi, nhưng bài này muốn chỉ ra chi tiết hơn một chú cho nhưng ai muốn đọc source.

Cách tìm source code driver cho thiết bị USB

## 1. Nói qua về thiết bị USB


*   Linux kernel xác định driver phù hợp cho thiết bị bằng 2 thông tin chính.  
    Đó là **Vendor** (nhà sản xuất), và **Product** (sản phẩm).  
    Thông tin về **Vendor** được mô tả bằng một id, trong source thường là **idVendor**.  
    Thông tin về **Product** cũng được mô tả bằng một id, trong source thường là **idProduct**.  
    2 thông tin trên sẽ xác định duy nhất một loại thiết bị.  
    Thông tin về id của mỗi Vendor, Product được đăng kí với tổ chức hỗ trợ USB.

*   Driver không phải là tất cả để một thiết bị cắm vào hoạt động.  
    Nhiều thiết bị để hoạt động được thì còn cần thêm một cái là **firmware** nữa.  
    Firmware chứa mã chương trình mà driver sẽ tống vào thiết bị.  
    Sau khi thiết bị chạy được bằng firmware đó rồi.  
    Thì giao tiếp phía sau sẽ là giữa **firmware** và **driver**.  
    **Firmware** thường chứa những đoạn chương trình không thể công khai của Vendor.  
    Vì thế, cách này thường được dùng để giữ những tài sản công nghệ của họ.

## 2. Lấy thông tin Vendor, Product từ /var/log/syslog


*   Như ta đã biết, linux có một hệ thống file log, chứa những thông tin trong  
    quá trình hoạt động của nhân, các phần mềm, dịch vụ. Nó thường nằm ở _/var/log/_  
    Trong trường hợp này, ta chỉ quan tâm đến log của nhân, ta sẽ xem file _/var/log/syslog/_  
    Câu lệnh để xem ở chế độ _live_ là:

```bash  
$tail -f /var/log/syslog  
```

*   Để quan sát được toàn bộ log của quá trình từ lúc cắm thiết bị đến lúc dùng được  
    ta sẽ bật lệnh xem log ở trên ở một terminal khác cho tiện.  
    Nếu không, ta phải xem lại file log ở trên và tìm ra được tương ứng cho thiết bị.
    
    Để dễ hiểu hơn, ta sẽ lấy ví dụ về thiết bị cụ thể:
    
*   Thiết bị là USB-Serial Converter, có giao tiếp bằng USB.  
    Ta cùng xem sẽ lấy được thông tin gì từ log của thiết bị này:  
    Đoạn log ta lấy được như như sau:

```bash  
Apr 23 12:01:54 OEDEV-Ubuntu kernel: [ 5046.576063] usb 2-3.1: new full-speed USB device number 14 using xhci_hcd  
Apr 23 12:01:54 OEDEV-Ubuntu kernel: [ 5046.671613] usb 2-3.1: New USB device found, idVendor=0403, idProduct=6015  
Apr 23 12:01:54 OEDEV-Ubuntu kernel: [ 5046.671617] usb 2-3.1: New USB device strings: Mfr=1, Product=2, SerialNumber=3  
Apr 23 12:01:54 OEDEV-Ubuntu kernel: [ 5046.671620] usb 2-3.1: Product: FT230X Basic UART  
Apr 23 12:01:54 OEDEV-Ubuntu kernel: [ 5046.671622] usb 2-3.1: Manufacturer: FTDI  
Apr 23 12:01:54 OEDEV-Ubuntu kernel: [ 5046.671624] usb 2-3.1: SerialNumber: DJ00LL3D  
Apr 23 12:01:54 OEDEV-Ubuntu kernel: [ 5046.675291] ftdi_sio 2-3.1:1.0: FTDI USB Serial Device converter detected  
Apr 23 12:01:54 OEDEV-Ubuntu kernel: [ 5046.675344] usb 2-3.1: Detected FT-X  
Apr 23 12:01:54 OEDEV-Ubuntu kernel: [ 5046.675737] usb 2-3.1: FTDI USB Serial Device converter now attached to ttyUSB0  
Apr 23 12:01:54 OEDEV-Ubuntu mtp-probe: checking bus 2, device 14: "/sys/devices/pci0000:00/0000:00:14.0/usb2/2-3/2-3.1"  
Apr 23 12:01:54 OEDEV-Ubuntu mtp-probe: bus: 2, device: 14 was not an MTP device  
```

Ta thấy từ log trên có đoạn:

```text  
Apr 23 12:01:54 OEDEV-Ubuntu kernel: [ 5046.671613] usb 2-3.1: New USB device found, idVendor=0403, idProduct=6015  
```

Chính dòng này đang chứa id của **Vendor** và **Product**:  
Cụ thể: **idVendor** = 0403, **idProduct** = 6015

Về cơ bản, thông tin đến đây là đủ để tìm driver trong source của nhân rồi.  
Ngoài ra bạn cũng sẽ thấy một số thông tin về Vendor cũng như thiết bị trong log ở trên:

Ví dụ:  
Product: FT230X Basic UART  
Manufacturer: FTDI  
SerialNumber: DJ00LL3D

Ở đây, từ log trên ta cũng biết được một thông tin quan trọng khác là thiết bị đã chạy hay chưa.

```text  
Apr 23 12:01:54 OEDEV-Ubuntu kernel: [ 5046.675737] usb 2-3.1: FTDI USB Serial Device converter now attached to ttyUSB0  
```

Chính dòng này đã cho ta biết được là thiết bị đã chạy được.  
Vì sao?  
Nó đã được attached vào vị trí **/dev/ttyUSB0** để cho phép các ứng dụng khác sử dụng.  
Tức là ta có một cổng COM giống như trên windows rồi.  
Nhưng vì ở Linux thì mọi thứ là file nên cổng com cũng là một file.

Nếu không có bước này thì ta có thể coi như thiết bị chưa dùng được cho dù kernel đã nhận  
biết được có một thiết bị được cắm vào.

## 3. Lấy thông tin Vendor, Product từ lsusb

lsusb là câu lệnh liệt kê tất cả các thiết bị usb hiện có, kể cả thiết bị chưa sử dụng được.  
Output của nó kiểu như sau:

```text  
$ lsusb  
Bus 001 Device 002: ID 8087:8001 Intel Corp.  
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub  
Bus 003 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub  
Bus 002 Device 009: ID 064e:c231 Suyin Corp.  
Bus 002 Device 005: ID 8087:07dc Intel Corp.  
Bus 002 Device 013: ID 1f55:0004  
Bus 002 Device 014: ID 0403:6015 Future Technology Devices International, Ltd Bridge(I2C/SPI/UART/FIFO)  
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub  
```

2 giá trị được ngăn cách bởi **:** phía sau **ID** chính là id của **Vendor** và **Product**.

Nếu bản linux của bạn hỗ trợ lsusb, bạn có thể dễ dàng xác định bằng cách  
so sánh đầu ra của câu lệnh **lsusb** trước và sau khi cắm một thiết bị vào máy.

Ví dụ:  
Trước khi cắm thiết bị, output của **lsusb** là:

```text  
$ lsusb  
Bus 001 Device 002: ID 8087:8001 Intel Corp.  
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub  
Bus 003 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub  
Bus 002 Device 009: ID 064e:c231 Suyin Corp.  
Bus 002 Device 005: ID 8087:07dc Intel Corp.  
Bus 002 Device 013: ID 1f55:0004  
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub  
```

Sau khi cắm thiết bị, output của **lsusb** là:

```text  
$ lsusb  
Bus 001 Device 002: ID 8087:8001 Intel Corp.  
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub  
Bus 003 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub  
Bus 002 Device 009: ID 064e:c231 Suyin Corp.  
Bus 002 Device 005: ID 8087:07dc Intel Corp.  
Bus 002 Device 013: ID 1f55:0004  
Bus 002 Device 014: ID 0403:6015 Future Technology Devices International, Ltd Bridge(I2C/SPI/UART/FIFO)  
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub  
```

Như ta đã thấy, thiết bị mà ta vừa xác đinh ở trên chỉ xuất hiện ở lệnh **lsusb** sau.

## 4. Tìm source code từ thông tin Vendor, Product

Giả định rằng ta đã có id của **Vendor** hay **idVendor** và id của Product hay **idProduct**  
Ta tiếp tục sử dụng thiết bị ở trên:

Product: FT230X Basic UART  
Manufacturer: FTDI  
SerialNumber: DJ00LL3D  
idVendor = 0403  
idProduct = 6015

Ta cần tìm source của driver cho thiết bị này trong source của kernel.

Ta tải bạn kernel statble mới nhất tại [https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.10.12.tar.xz](https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.10.12.tar.xz), và giải nén:

```bash  
$ cd linux-4.10.8/  
$ ls  
arch COPYING Documentation fs ipc kernel Makefile modules.order README security tools vmlinux  
block CREDITS drivers include Kbuild lib mm Module.symvers samples sound usr vmlinux.o  
certs crypto firmware init Kconfig MAINTAINERS modules.builtin net scripts System.map virt  
```

Ta tìm source driver cho thiết bị trên như sau:

*   Tìm theo vendor id:

```bash  
$ grep -n -R 0x0403 drivers/usb  
drivers/usb/serial/ipaq.c:66: { USB_DEVICE(0x045E, 0x0403) }, /* Windows Powered Pocket PC 2002 */  
drivers/usb/serial/cp210x.c:203: { USB_DEVICE(0x1FB9, 0x0403) }, /* Lake Shore Model 475A Gaussmeter */  
drivers/usb/serial/ftdi_sio.h:449: * 8 idVendor 2 0x0403 Vendor ID  
drivers/usb/serial/ftdi_sio_ids.h:16:#define FTDI_VID 0x0403 /* Vendor Id */  
drivers/usb/serial/ftdi_sio_ids.h:47: * OpenRD Base, Client use VID 0x0403  
drivers/usb/serial/ftdi_sio_ids.h:233: * Almost all of these devices use FTDI's vendor ID (0x0403).  
drivers/usb/serial/ftdi_sio_ids.h:346: * Hameg HO820 and HO870 interface (using VID 0x0403)  
drivers/usb/serial/ftdi_sio_ids.h:1352: * MJS Gadgets HD Radio / XM Radio / Sirius Radio interfaces (using VID 0x0403)  
drivers/usb/serial/ftdi_sio_ids.h:1373: * Segway Robotic Mobility Platform USB interface (using VID 0x0403)  
drivers/usb/misc/ftdi-elan.c:86:#define USB_FTDI_ELAN_VENDOR_ID 0x0403

$ grep -n -R 0x6015 drivers/usb  
drivers/usb/serial/ftdi_sio_ids.h:26:#define FTDI_FTX_PID 0x6015 /* FT-X series (FT201X, FT230X, FT231X, etc) */  
```

Nếu ta đã biết về lập trình C, thì thường giá trị hardcode như 0403, 0615 sẽ được định nghĩa  
và sử dụng định nghĩa đó ở những nơi khác chứ không sử dụng trực tiếp giá trị.

Từ kết quả ở trên ta thấy có 2 dòng **define** cho giá trị **0x0403** và **0x6015**  
cùng ở file **drivers/usb/serial/ftdi_sio_ids.h**. Đó là

```text  
drivers/usb/serial/ftdi_sio_ids.h:16:#define FTDI_VID 0x0403 /* Vendor Id */  
drivers/usb/serial/ftdi_sio_ids.h:26:#define FTDI_FTX_PID 0x6015 /* FT-X series (FT201X, FT230X, FT231X, etc) *  
```

Ta hiểu rằng, 2 giá trị trên sẽ được sử dụng bằng tên đại diện **FTDI_VID** và **FTDO_FTX_PID** ở những nơi khác.

Ta tiếp tục tìm, vì một nhà sản xuất sẽ có rất nhiều thiết bị.  
Vì thế ta nên tìm thiết bị để có kết quả chính xác hơn. Trong trường hợp này là **FTDO_FTX_PID**

```text  
$ grep -n -R FTDI_FTX_PID  
drivers/usb/serial/ftdi_sio_ids.h:26:#define FTDI_FTX_PID 0x6015 /* FT-X series (FT201X, FT230X, FT231X, etc) */  
drivers/usb/serial/ftdi_sio.c:177: { USB_DEVICE(FTDI_VID, FTDI_FTX_PID) },  
```

Và đây, source cho driver của thiết bị nằm ở **drivers/usb/serial/ftdi_sio.c**.

## 5. Tham khảo

Để biết thêm về cách tạo, đọc một driver (đặc biệt của usb) trong linux.  
Có lẽ không chỗ nào tốt hơn chính là tài liệu (hoặc sách) do người viết (maintain) những driver đó.

Thực sự nên đọc cuốn sách Linux Device Driver 3rd Edition (hoàn toàn free) của chính những maintainer tại địa chỉ [https://lwn.net/Kernel/LDD3/](https://lwn.net/Kernel/LDD3/)

Một cuốn nữa (và cũng free) nên đọc là Linux in A Nutshell (link tại [http://files.kroah.com/lkn/](http://files.kroah.com/lkn/) ) cũng của một maintainer chính cho nhân Linux.