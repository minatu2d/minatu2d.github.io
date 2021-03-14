---
layout: post
title: Cài thêm driver usb-wifi adapter cho bản build Raspbery PI sử dụng Yocto Project
date: 2016-08-11 15:04:03.000000000 +09:00
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
- Driver
- Firmware
- RT2870
- WifiUSB
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: '43452'
  _publicize_job_id: '25687462974'
  _oembed_ae4a09cd37b70ce4745e923704db6c3c: "{{unknown}}"
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/08/11/cai-them-driver-usb-wifi-adapter-cho-ban-build-raspbery-pi-su-dung-yocto-project-chua-xong/"
---
Như ta đã làm trong bài trước, sau khi thực hiện việc setup các biến môi trường bằng lệnh **source**, ta thực hiện build tạo image có có tên là **rpi-basic-image** thông qua lệnh:

```bash  
$bitbake rpi-basic-image  
```

Thực ra còn 2 image khác ta có thể build đó là **rpi-hwup-image** **rpi-test-image**. Ta có thể thấy 2 file bb cho 2 image ở thư mục **meta-raspberrypi/recipes-core/images/**.

**rpi-hwup-image** : là image nhỏ nhất (có dịp sẽ thử)  
**rpi-basic-image**: là image ta vấn dùng đến bây giờ.  
**rpi-test-image** : là image dùng cho việc test. (có dịp sẽ thử)

Dù image chúng ta vẫn build (**rpi-basic-image**) được tích hợp nhiều nhất trong số 3 image ở trên nhưng không phải tất cả driver đều được cài đặt.  
Điển hình là USB Toggle - Wifi thông qua USB (Buffalo WLI-UC-GNM) không chạy được.  
Thông qua log hệ thống, ta thấy được rằng việc load firmware cho thiết bị đã không thành công vì thế không thể bật được thiết bị.

Mục đích của bài này là làm cho thiết bị trên có thể chạy được trên bản build **rpi-basic-image**, giống như Raspbian.

Trong bài này sẽ thử thực hiện 2 cách sau:

1.  Chuyển driver đã chạy tốt từ Raspbian sang bản build **rpi-basic-image**
2.  Build driver từ source code.

## 1.  Chuyển driver đã chạy tốt từ Raspbian sang bản build **rpi-basic-image**

Cách này về cơ bản là một "cheat", nó hơi khác với tinh thần ban đầu khi build toàn bộ từ source. Nhưng trong tình huống này, ta vẫn hay thử nó.

Trước hết, ta cần chạy lại **Raspbian** và kiểm tra các thông tin sau:  
- Các thông số phần cứng của USB-Wifi Adapter mà ta đang sử dụng  
- Lấy log hệ thống từ thời điểm thiết bị được cắm vào đến lúc sử dụng được.

Update:2016/09/05:  
Qua kết quả tìm hiểu từ [Driver và Firmware trong Linux](https://lazytrick.wordpress.com/2016/08/29/khai-niem-driver-va-firmware-trong-linux/), [Quá trình load firmware của Driver trong Linux](https://lazytrick.wordpress.com/2016/09/05/qua-trinh-load-firmware-cua-driver-trong-linux/), [Tìm Driver cho Linux](https://lazytrick.wordpress.com/2016/08/19/xac-dinh-cac-driver-module-dang-duoc-su-dung-tren-linux/)

Trong trường hợp này ta phải làm sao để firmware của USB Wifi Client có mặt ở **/lib/firmware/** khi Driver của thiết bị được chạy.  
Cách đơn giản nhát là thêm rcipy **linux-firmware** vào layer **meta-local**  
Tuy nhiên, để cấu hình sử dụng được USB Wifi Client ta cần thêm một tập các tool về Wifi nữa. Nó nằm trong gói **packagegroup-base-wifi**

Xem lại **meta-local** ở bài[](https://lazytrick.wordpress.com/2016/08/06/them-samba-server-vao-ban-build-linux-cho-raspberry-pi-bang-yocto-project/)

```bash  
poky/meta-local/  
├── conf  
│ └── layer.conf  
└── recipes-core  
└── images  
└── rpi-basic-image.bbappend  
```

Ta cần sửa lại **rpi-basic-image.bbappend** thành như sau:

```python  
IMAGE_INSTALL += &quot;samba \  
linux-firmware \  
packagegroup-base-wifi  
```

## 2. Build từ source code

Cách này sai lầm vì bản thân Driver đã có rồi chứ không phải chưa có như đã nghĩ lúc trước.  
Thứ còn thiếu là **firmware** cái sẽ được Driver tải vào để thực hiện cá chức năng vốn có của nó.