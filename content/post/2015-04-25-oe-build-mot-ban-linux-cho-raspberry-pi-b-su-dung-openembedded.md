---
layout: post
title: "[OE] Build một bản Linux cho Raspberry PI B+ sử dụng OpenEmbedded"
date: 2015-04-25 15:30:15.000000000 +09:00
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
- RaspberryPI
meta:
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _oembed_b3731bff579b26a876efbe94aa00fa59: "{{unknown}}"
  _oembed_bc66e91ab4e74e140821c8c69d3058b6: "{{unknown}}"
  _oembed_be61d38dae8b3f4c0afa8a85e4e22ea9: "{{unknown}}"
  _oembed_4066d1fdce6c4e38eedf17c0446c3dc4: "{{unknown}}"
  _oembed_77061a24d380b09847fb19678640c312: "{{unknown}}"
  _oembed_2c1abe0f3df73c658a81eeb8ca4da50c: "{{unknown}}"
  _oembed_30dcf9076808cbc265524b81d37da5f9: "{{unknown}}"
  _wpcom_is_markdown: '1'
  _publicize_job_id: '26770590217'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2015/04/25/oe-build-mot-ban-linux-cho-raspberry-pi-b-su-dung-openembedded/"
---
Poky là một hệ distro linux ở dạng tham chiếu của Yocto Project.

OpenEmbedded là một phần trong đó.

Nào thế đủ rồi, ta đi vào phần chính.

### 1. Về Yocto project và ứng dụng cho Rasberry PI
Lần trước, tôi có viết một hứng về việc [tạo ra một ảnh cho Raspberry](http://www.cnx-software.com/2012/07/31/84-mb-minimal-raspbian-armhf-image-for-raspberry-pi/) dựa trên Raspbian (chụp lại ảnh của một hệ thống đang chạy). Với kết quả lúc trước, thì vấn đề là nó không thực sự nhỏ hơn, khi giải nén ra nó vẫn chiếm khoảng 414MB.

Nào, ngay hôm nay, tôi sẽ hướng dẫn để tạo một ảnh của hệ thống (Linux distro) sử dụng Yocto Project,

1 platform cho phép bạn build một Embedded Linux Distribution phù hợp chính xác với yêu cầu của dự án.

Một điểm nữa là, quá trình build có thể được **configure với các configure file**. Nó khá năng suất trong việc tạo ra một bản distro chỉ với rất ít thao tác.

Nếu bạn muốn image, bạn có thể download về ngày qua [link ](http://www.cnx-software.com/raspberry-pi/rpi-basic-image-raspberrypi-20130702123605.rootfs.rpi-sdimg.7z)này.

Sau khi download về rồi thì giải nén nó, ghi vào SD card. Rồi chạy thôi.

Còn nếu bạn muốn customize nó thì tiếp tục đọc.

Tôi đã thử với hướng dẫn từ [pimpmypi](http://www.pimpmypi.com/blog/blogPost.php?blogPostID=7 " pimpmypi"). Nhưng nó khá cũ, giờ mọi thứ dường như trở nên đơn gian hơn.

### 2. Thực hiện

Với môi trường thực hiện là Ubuntu 14.04 LTS.

#### 2.1 Lấy source
Đầu tiên chúng ta lấy source của poky và meta layer dành cho Raspberry PI.

```bash
mkdir yocto

cd yocto

git clone -b dylan git://git.yoctoproject.org/poky.git

cd poky

git clone -b dylan git://git.yoctoproject.org/meta-raspberrypi
```

### 2.2 Thiết lập biến môi trường
Chạy lệnh sau để setup một số biến môi trường

```bash

. oe-init-build-env build

```

Nhất định phải có dấu cách sau dấu "." . (Dấu này đại diện cho lệnh `source`)

Sau khi chạy xong lệnh trên và succecced.

Con trỏ lệnh sẽ tự động chuyển về thư mục build.

### 2.3 Build
Chúng sẽ edit 1 file cấu hình để giúp bitbake sử dụng CPU của PC chúng ta build tốt hơn.

Giúp tiết kiệm thời gian build.

```bash

Edit conf/local.conf

BB_NUMBER_THREADS = "9"

PARALLEL_MAKE = "-j 9"

MACHINE ?= "raspberrypi"

GPU_MEM = "16"
```

Trường nào không có thì bỏ qua, kết quả cuối cùng không thay đổi tuy nhiên, nếu cấu hình chính xác

với thông tin của máy sẽ giúp quá trình build nhanh hơn.

Cuối cùng, để OE build một image cho Raspberry PI, ta cần thêm lớp của RPI vào danh sách các lớp

mà OE có thể biết.

```bash

Edit conf/bblayers

BBLAYERS ?= " \

poky/meta \

poky/meta-yocto \

poky/meta-yocto-bsp \

poky/meta-raspberrypi \

"
```

Giờ là lệnh build, có 2 option cho file image. Đó là **rpi-basic-image** và **rpi-hwup-image**.

Cả 2 đều là minimal image. Nhưng **rpi-basic-image** có sẵn ssh-server-dropbear và splash

(cho splash screen).

Chúng ta sẽ build **rpi-basic-image**

Lệnh build như sau
```bash

`bitbake rpi-basic-image`

```

Quá trình build này phụ thuộc vào perform của máy và tốc độ mạng.

Có thể mất cả tiếng đồng hồ.

### 2.4 Kết quả

Khi quá trình build này hoàn thành, bạn sẽ thấy file ảnh nằm ở đây

`tmp/deploy/images/rpi-basic-image-raspberrypi.rpi-sdimg`

Ghi file đó ra SD Card là bạn đã có thể boot ngay vào PI được rồi.

User là root và không cần pass.

### 2.5 Đặc điểm của bản build

Bản linux này bao gồm những đặc điểm sau

* shell chạy là : busybox, đương nhiên là thư viện là uglibc rồi.

~~+ Không tích hợp driver cho wifi dongle.~~

* Chưa thấy code car wlan0.
* Có sắn driver cho Ethernet

Tình trạng sau khi khởi động của máy như thế này

![Featured image](/post/images/dsc_0084.jpg?w=300)

Trong hình trên, dù đã kết nối wireless dongle với PI qua USB rồi.

Kiểm tra lại log hệ thống để xem, PI đã nhận dạng được thiết bị này chưa.

Log như hình sau, ở đây t sử dụng lệnh

```bash

`tail -f /var/log/messages `

```

Ta sẽ thấy log dưới đây khi cắm wire dongle vào PI.

![Featured image](/post/images/dsc_0086.jpg?w=300)

Từ màn hình log ta thấy, thiết bị dường như đã được nhận dạng thành công.

Hiện tại có thể đã lỗi tại dòng

phy3: Failed to initialize wep : -2

Mình có 2 suy đoán ở đây

* Có thể phần wireless thiếu file cấu hình nên xảy ra tình trạng này.
99.  Có thể vẫn do lỗi driver, thiếu cái gì đó vì nếu chỉ khởi động bị failure thì phần hàm ifconfig phải list ra được chứ nhi.

Giờ sẽ thử làm theo hướng dẫn này, đưa file cấu hình wifi vào PI

http://www.jann.cc/2012/08/08/building_freescale_s_community_yocto_bsp_for_the_olinuxino.html#extending-the-wifi-support

Trong thư mục build của yocto, add thêm dòng này vào **conf/local.conf**  
IMAGE_INSTALL_append = " wpa-supplicant wireless-tools dhcp-client linux-firmware"
