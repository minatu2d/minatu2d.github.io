---
layout: post
title: Thiết lập IP mặc định của bản build Linux cho Raspberry PI bằng Yocto Project
date: 2016-08-06 17:08:17.000000000 +09:00
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
- Recipe
- StaticIP
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: '43452'
  _publicize_job_id: '25530623083'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/08/06/thiet-lap-ip-mac-dinh-cua-ban-build-linux-cho-raspberry-pi-bang-yocto-project-chua-xong/"
---
Để tiếp tục customize bản OS được build trong bài [trước](https://lazytrick.wordpress.com/2016/07/29/tao-nas-server-tren-pi-bang-yocto-project/).  
Hôm nay ta sẽ thực hiện một nhiệm nhỏ. Đó là thiết lập một IP cho bản build.  
Tức là ta sẽ thiết lập 1 IP mặc định được gán cho Raspberry PI khi nó được khởi động bằng bản build của chúng ta.

Như thường lệ ta cần thiết lập các biến môi trường trước khi định thực hiện bất cứ thay đổi nào trên bản build.

```bash  
$source poky/oe-init-build-env build  
```

## Gán IP tĩnh cho bản build

### 1. Cách 1: Thay đổi trong **rootfs** (Đã không thực hiện được - bỏ qua và đến cách 2 bên dưới)

Về cơ bản, để tạo được một bản build linux, thì chắc chắn đâu đó phải có 1 **rootfs** (rootfs là gì được giải thích tại [đây](https://lazytrick.wordpress.com/2016/08/03/cac-khai-niem-trong-cross-compiling/).

Và ta cũng biết rằng, để setup IP cho một máy chạy Linux (những bản dựa trên Debian/Ubuntu), ta sẽ hỏi google và tìm được link [này](http://www.cyberciti.biz/faq/linux-configure-a-static-ip-address-tutorial/).

Ta cần vào **/etc/netwwork/interface** rồi edit lại đoạn setup **eth0** với mạng có dây mặc định, **wlan0** với mạng không dây mặc định.

Từ 2 thông tin trên, Yocto có gì liên quan đến 2 thứ trên.

1.  Đầu thay đổi **/etc/network/interfaces** có trong rootfs mặc định  
    Từ thư mục **build**, ta tìm bằng câu lệnh sau:

```bash  
oedev@OEDEV-Ubuntu:~/raspberrypi_krogoth/build$ locate /etc/network/interfaces  
/etc/network/interfaces  
/etc/network/interfaces.d  
/home/oedev/raspberrypi_krogoth/build/tmp/work/arm1176jzfshf-vfp-poky-linux-gnueabi/init-ifupdown/1.0-r7/image/etc/network/interfaces  
/home/oedev/raspberrypi_krogoth/build/tmp/work/arm1176jzfshf-vfp-poky-linux-gnueabi/init-ifupdown/1.0-r7/package/etc/network/interfaces  
/home/oedev/raspberrypi_krogoth/build/tmp/work/arm1176jzfshf-vfp-poky-linux-gnueabi/init-ifupdown/1.0-r7/packages-split/init-ifupdown/etc/network/interfaces  
/home/oedev/raspberrypi_krogoth/build/tmp/work/raspberrypi-poky-linux-gnueabi/rpi-basic-image/1.0-r0/rootfs/etc/network/interfaces  
```

Ta thấy có 1 kết quả là **_...rpi-basic-image/1.0-r0/rootfs/etc/network/interfaces_**.  
Ôi, chính em nó đây rồi.  
Rồi xơi thôi.

*   Mở file trên ra và sửa thành như sau, tất nhiên nhớ copy ra trước khi sửa nha.

```bash  
# /etc/network/interfaces -- configuration file for ifup(8), ifdown(8)

# The loopback interface  
auto lo  
iface lo inet loopback

# Wireless interfaces  
iface wlan0 inet dhcp  
wireless_mode managed  
wireless_essid any  
wpa-driver wext  
wpa-conf /etc/wpa_supplicant.conf

iface atml0 inet dhcp

# Wired or wireless interfaces  
auto eth0  
#  
# Modify START  
#  
#iface eth0 inet dhcp  
iface eth0 inet static  
address 192.168.1.100  
netmask 255.255.255.0  
network 192.168.1.1  
gateway 192.168.1.1  
#  
# Modify END  
#

iface eth1 inet dhcp

# Ethernet/RNDIS gadget (g_ether)  
# ... or on host side, usbnet and random hwaddr  
iface usb0 inet static  
address 192.168.7.2  
netmask 255.255.255.0  
gateway 192.168.0.1  
broadcast 192.168.0.255

# Bluetooth networking  
iface bnep0 inet dhcp  
```

*   Cuối cùng, build lại image.  
    Nếu chỉ chạy **bitbake rpi-basic-image** thì sẽ không có ảnh mới được tạo.  
    Vì thay đổi trên **rootfs** không làm cho bitbake hiểu được.  
    Bitbake chỉ chú ý đến các thay đổi trên layer, hoặc recipes thôi.  
    vì thế ta phải ép bitbake build tích hợp lại rootfs để tạo ra ảnh mới.

```bash  
oedev@OEDEV-Ubuntu:~/raspberrypi_krogoth/build$ bitbake rpi-basic-image -c rootfs -f  
```

*   Sau đó ghi file image được build ra từ đường dẫn **build/tmp/deploy/images/raspberrypi/** vào thẻ nhớ và kiểm chứng nào.  
    Một số hình ảnh thực tế

Cách làm này không ổn, vì mình đã hiểu sai về cách tạo ảnh của bitbake.  
Các rootfs được tạo mới từ các package khác chứ không phải có từ rootfs có sẵn để mình có thể tìm thấy được. Vì thế, rootfs sẽ được lấy từ 1 file nén ở đâu để làm base. Sau đó thực hiện thêm "gia vị" bằng các "recipe" trên cái base đó.

### 2. Ghi đè **/etc/interface** bằng thêm vào recipes đã có sẵn

Qua đọc mailing list của cộng đồng người dùng Yocto, mình đã nhận ra rằng, mọi thay đổi từ các file cấu hình, cài đặt phần mềm mới nên thực hiện thông qua các gia vị (recipe) chứ không nên thực hiện thay đổi rootfs.

Câu trả lời cho hầu hết các trường hợp muốn setup file cấu hình mặc định khi build ảnh nằm ở [đây](https://lists.yoctoproject.org/pipermail/yocto/2016-July/031004.html). Có đoạn như sau:

```text  
FYI there is a shortcut to creating these - the "recipetool appendfile"  
command. For example, say you have your desired custom inittab file locally as  
"myinittab" and you want the bbappend to be created in meta-mylayer then you  
would run:

recipetool appendfile meta-mylayer /etc/inittab myinittab

This will automatically determine which recipe is providing the file and how  
to overwrite it with your version. You do need to have built that recipe for  
this to work, but then that will already be the case if you have built the  
image beforehand.

Cheers,  
Paul

--

Paul Eggleton  
Intel Open Source Technology Centre  
```

Áp dụng với trường hợp của **/etc/network/interface**:  
Ta sẽ thực hiện sửa layer **meta-local** với mục đích sẽ thay thế file **/etc/network/interface** mặc định trong rootfs bằng file của chúng ta.

Đầu tiên, từ thư mục **BUILD** (biến môi trường là **BUILDIRR**tạo một file sẽ thay thế, ta copy 1 bản từ file mặc định và edit nó:

```bash  
# /etc/network/interfaces -- configuration file for ifup(8), ifdown(8)

# The loopback interface  
auto lo  
iface lo inet loopback

# Wireless interfaces  
iface wlan0 inet dhcp  
wireless_mode managed  
wireless_essid any  
wpa-driver wext  
wpa-conf /etc/wpa_supplicant.conf

iface atml0 inet dhcp

# Wired or wireless interfaces  
auto eth0  
#  
# Modify START  
#  
#iface eth0 inet dhcp  
iface eth0 inet static  
address 192.168.0.100  
netmask 255.255.255.0  
gateway 192.168.0.1  
broadcast 192.168.0.255  
#  
# Modify END  
#  
iface eth1 inet dhcp

# Ethernet/RNDIS gadget (g_ether)  
# ... or on host side, usbnet and random hwaddr  
iface usb0 inet static  
address 192.168.7.2  
netmask 255.255.255.0  
network 192.168.7.0  
gateway 192.168.7.1

# Bluetooth networking  
iface bnep0 inet dhcp  
```

Sau đó ta thực hiện lệnh sau để thêm vào recipe **init-ifupdown**

```bash  
$ recipetool appendfile ../poky/meta-local /etc/network/interfaces ./interface  
```

Sau khi thực hiện xong, ta sẽ thấy recipe **init-ifupdown** được thêm vào **meta-local**, cũng như **bbappend** file cũng được thêm vào luôn.  
Đây là nội dung của **meta-local** sau đó:

```bash  
../poky/meta-local/  
├── conf  
│   └── layer.conf  
└── recipes-core  
├── images  
│   └── rpi-basic-image.bbappend  
└── init-ifupdown  
├── init-ifupdown  
│   └── interfaces  
└── init-ifupdown_1.0.bbappend  
```

Như vậy ta thấy rằng, **recipetool** tự động thêm vào 1 **recipe** đã có trước đó.

Giờ ta tiến hành build lại ảnh cho RPI là xong:  
Vẫn câu lệnh quen thuộc từ thư mục **build**:

```bash  
$bitbake rpi-basic-image  
```

Trong kết quả có thể xuất hiện 2 lỗi về **hash mismatch**, cái này không sao hết. File ảnh mới vẫn được tạo, mọi thứ vẫn chạy bình thường.

### 3. Thực hiện thêm action vào bước tạo rootfs

(chưa thực hiện)