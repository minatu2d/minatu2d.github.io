---
layout: post
title: Một chút về Driver cho USB Device trong Linux
date: 2016-08-15 05:47:10.000000000 +09:00
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
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '25797273465'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/08/15/mot-chut-ve-driver-cho-usb-device-trong-linux/"
---
Trong loại bài dịch [trước đây](https://lazytrick.wordpress.com/2016/01/25/usb-driver-va-usb-device-firmware-se-viet/) nói về USB, tôi có nhắc một chút đến việc load đúng driver thì phía Host làm thế nào?

Như ta đã biết, khi một thiết bị USB được cắm vào máy (Host), phía Host sẽ thực hiện một loạt thao tác từ xác định nguồn (bus, hay self), lấy thông tin tốc độ, các thông tin về descriptor (thiết bị, giao diện, các Endpoint). Cuối cùng để sử dụng các tính năng chính mà thiết bị cung cấp, thì hệ điều hành phía Host, dù là Windows, hay Linux, hay MacOS sẽ phải load driver tương ứng với thiết bị.

Để xác định đúng driver cần load, phía Host thường sử dụng 2 thông số. Đó là **Product ID/Vendor ID**

Trường hợp hệ điều hành Linux, hầu hết Driver cho thiết bị được tích hợp sẵn trong source của nhân. Tất nhiên, với mỗi cấu hình build khác nhau, người ta có thể biên dịch cùng hoặc bỏ qua.  
Trong linux, mỗi driver sẽ cho Kernel biết một danh sách các thiết bị (mỗi thiết bị thường được đặc trưng bởi 2 thông số ở **Product ID/Vendor ID** đã nói phía trên).  
Kernel sẽ lựa chọn driver phù hợp nhất.

Như một tìm hiểu nhỏ, cho bài viết [Cài thêm driver cho bản build Raspberry PI sử dụng Yocto](https://lazytrick.wordpress.com/2016/08/11/cai-them-driver-usb-wifi-adapter-cho-ban-build-raspbery-pi-su-dung-yocto-project-chua-xong/).

Ta xét đến thiết bị USB-Wifi Adapter của Buffalo, link sản phẩm tại [đây](http://buffalo.jp/product/wireless-lan/client/wli-uc-gnm2/).

Thiết bị chạy tốt trên Ubuntu, Raspbian vì thế ta có thể thấy các thông số của thiết bị tại log của 2 hệ điều hành này.  
Từ đó ta thấy được:

1.  Thiết bị sử dụng chip không dây của Ralink (xem thêm tại https://en.wikipedia.org/wiki/Ralink)
2.  Trong source của nhân linux, dường như có 1 loạt thiết bị không dây Ralink đang được hỗ trợ.  
    Xem ở đây ta sẽ thấy: [http://lxr.free-electrons.com/source/drivers/net/wireless/ralink/rt2x00/rt2800usb.c#L968](http://lxr.free-electrons.com/source/drivers/net/wireless/ralink/rt2x00/rt2800usb.c#L968)

Vậy thì tại sao bản build **rpi-basic-image** lại không có.???

1.  Trong log mà ta có được, có xuất hiện một dòng liên quan đến firmware request.  
    Vậy có quan hệ gì nữa Driver trong Kernel, module, và firmware??  
    Dường như kernel cho phép load firmware để hỗ trợ các driver mà nó sử dụng.

Update 2016/09/06:  
Từ kêt quả tìm hiểu về [Driver và Firmware trong Linux](https://lazytrick.wordpress.com/2016/08/29/khai-niem-driver-va-firmware-trong-linux/), [Quá trình load firmware của Driver trong Linux](https://lazytrick.wordpress.com/2016/09/05/qua-trinh-load-firmware-cua-driver-trong-linux/), [Tìm Driver cho Linux](https://lazytrick.wordpress.com/2016/08/19/xac-dinh-cac-driver-module-dang-duoc-su-dung-tren-linux/)

Ta có thể thấy rằng, với thiết bị USB Wifi Client của chúng ta, ta chỉ cần tạo thư mục **/lib/firmware/** và copy firmware còn thiếu vào là sẽ chạy được.  
Tuy nhiên để tạo ra bản build chưa sẵn các firmware này thì ta phải thêm recipy **linux-firmware** vào image.