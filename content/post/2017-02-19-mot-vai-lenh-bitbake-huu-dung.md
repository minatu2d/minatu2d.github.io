---
layout: post
title: Một vài lệnh Bitbake hữu dụng
date: 2017-02-19 00:14:48.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
- OpenEmbbeded
tags:
- Bitbake
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '2009984862'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2017/02/19/mot-vai-lenh-bitbake-huu-dung/"
---
Có một vài lệnh hữu dụng được cộng đồng sử dụng board NXP chia sẻ, mình sẽ note ở đây cho dễ tìm vậy. Link tại [đây](https://community.nxp.com/docs/DOC-94953).

**Lệnh Bitbake**

**Miêu tả  
**

_bitbake <image>_

_Nấu ra 1 "ảnh" (Image) _(Thêm tham số _-k đ_ể cho phép chạy đến hết kẻ cả có lỗi thực thi)

_bitbake <package> -c <task>_

`Thực hiện 1 task của package nào đó. Tên các task mặc định thường có``:` `_fetch,_` _`unpack`_, _`patch`_, _`configure`_, _`compile`_, _`install`_, _`package`_, _`package_write`_, and _`build.`_

Ví dụ_:_ Để "ép" bitbake compile lại kernel và _build lại ảnh cho board imx, ta sẽ sử dụng_ :

_$ bitbake_ _linux-imx_ _-f -c compile_

_$ bitbake linux-imx  
_

_bitbake <image >_ _-g -u depexp_  

Hiển thị các package phụ thuộc của 1 Image.

Ví dụ: Để hiển thị toàn bộ các pakage phụ thuộc của fsl-image-gui

_$ bitbake fsl-image-gui -g -u depexp_

Chú ý: Lệnh này sẽ mở  một UI window, vì thế cần thực hiện lệnh này trên 1 console của Desktop (chứ không phải console ảo hoặc remote, hoặ serial đâu nha).

_bitbake <package> -c  devshell_

Mở một shell mới với tất cả các biến cần thiết cho package được chỉ định.

_toaster_

Giao diện web cho Bitbake.

_bitbake <package> -c listtasks_

Hiển thị tất cả các task của 1 package.

_bitbake virtual/kernel_ _-c menuconfig_

Cấu hình lại kernel

bitbake <image> -c fetchall

Thực hiện tải source cho Image được chỉ định

bitbake-layers show-layers

Hiển thị các layers

bitbake-layers show-recipes "*-image-*"

Hiển thị các Image hiện có. Nếu không các kí tự star trong "*-images-*", nó sẽ show ra tất cả các Recipe hiện có đấy.

bitbake -g <_image>_ && cat pn-depends.dot | grep -v -e '-native' | grep -v digraph | grep -v -e '-image' | awk '{print $1}' | sort | uniq

Hiển thị tất cả các Package cua 1 Image

bitbake -g <_pkg>_ && cat pn-depends.dot | grep -v -e '-native' | grep -v digraph | grep -v -e '-image' | awk '{print $1}' | sort | uniq

Hiện thị tất cả các phụ thuộc của 1 Package

bitbake –v _<image>_ 2>&1 | tee _image__build.log

In kết quả ra màn hình console và lưu vào cả file nữa.

bitbake -s | grep _<pkg>_

Kiểm tra xem 1 package hiện có trong bản build hiện tại không.