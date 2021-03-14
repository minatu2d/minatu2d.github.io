---
layout: post
title: Thêm samba server vào bản build Linux cho Raspberry PI bằng Yocto Project
date: 2016-08-06 16:05:42.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Communication
- Embedded
- OpenEmbbeded
- YoctoProject
tags:
- Samba
- SMB
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: '43452'
  _publicize_job_id: '25529293725'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/08/06/them-samba-server-vao-ban-build-linux-cho-raspberry-pi-bang-yocto-project/"
---
Ta đã nói đến việc build một bản phân phối Linux cho Raspberry PI ở bài [Tạo một bản phân phối Linux cho Raspberry PI bằng Yocto Project](https://lazytrick.wordpress.com/2016/07/29/tao-nas-server-tren-pi-bang-yocto-project/).

Một trong những giao thức truy cập file phổ biến nhất hiên nay là SMB, vốn ban đầu được hỗ trợ trên các máy Windows, dùng cho giao thức chia sẻ file trong mạng nội bộ. Trên Linux, để tạo một server như thế, người ta dùng Samba (cái tên cũng na ná nhỉ).

Trong bài này, sẽ trình bày cách đưa **Samba** vào bản phân phối Linux đã được build ở bài trước.

  
Trước khi bắt đầu các thao tác dưới đây, ta cần thiết lập các biến môi trường cần thiết bằng câu lệnh từ bài trước:

```bash  
$source poky/oe-init-build-env build  
```

Thư mục **build** sẽ ở ngoài thư mục **poky**

## Thêm samba server vào bản build

Yocto Project cung cấp một bộ khung cho phép thêm các phần mềm vào image được build ra. Ta có thêm bất cứ phần mềm nào, nó có thể là source tự build hoặc một phần mềm được viết kịch bản từ build để thêm vào images có từ trước.

Trường hợp của **samba server**, đây vốn là một phần mềm khá phổ biến khi muốn tạo một server để truy cập giống như chia sẻ file giữa các máy trên windows vậy.  
Vì thế, kịch bản tải source, thực hiện các bản sửa, compiple đã có sẵn rồi.  
Nó là 1 **recipes** (file chứa kịch bản download source đến compile) thuộc layer **meta-openembedded/meta-networking/**

(Để tìm hiểu thêm các khái niệm về **recipe** và **layer** của Yocto Project xin hãy hỏi thầy Google)

Như một cách để confirm, ta có thể truy cập vào source code của layer trên tại:

[https://github.com/openembedded/meta-openembedded/tree/master/meta-networking](https://github.com/openembedded/meta-openembedded/tree/master/meta-networking)

Recipe của **samba server** nằm ở **meta-networking/recipes-connectivity/samba/**.

Ta bắt đầu quá trình thêm ngay bây giờ  
- Đầu tiên, ta phải tải layer trên về thư mục **poky**, giống như cách tải BSP ở [bài trước](https://lazytrick.wordpress.com/2016/07/29/tao-nas-server-tren-pi-bang-yocto-project/), phiên bản mới nhất của **meta-openembedded** là **krogoth**.  
Ta sẽ thực hiện các câu lệnh để tải về như dưới đây:

```bash  
oedev@OEDEV-Ubuntu:~/raspberrypi_krogoth/poky$ pwd  
/home/oedev/raspberrypi_krogoth/poky  
oedev@OEDEV-Ubuntu:~/raspberrypi_krogoth/poky$ git clone -b krogoth git://github.com/openembedded/meta-openembedded.git  
```

Không mất quá thời gian để tải, ta sẽ có thư mục **meta-openembedded** nằm bên dưới **Poky**, tức là cùng thư mục với **meta-raspberrypi** đã tải ở phần trên.

*   Giờ ta phải thêm layer chứa **samba server** vào  
    Để thêm layer, ta sẽ thực hiện giống như đã thêm với layer BSP ở trên thôi:  
    Mở file **build/conf/bblayers.conf** và thêm đường dẫn của layer networking.  
    Tuy nhiên, ta sẽ gặp lỗi ngay vì chỉ thêm networking sẽ không đủ, vì trong file mô tả của layer networking (đường dẫn **meta-networking/conf/layer.conf**) có nội dung như sau:

```python  
# We have a conf and classes directory, add to BBPATH  
BBPATH .= ":${LAYERDIR}"

# We have a packages directory, add to BBFILES  
BBFILES += "${LAYERDIR}/recipes-*/*/*.bb \  
${LAYERDIR}/recipes-*/*/*.bbappend"

BBFILE_COLLECTIONS += "networking-layer"  
BBFILE_PATTERN_networking-layer := "^${LAYERDIR}/"  
BBFILE_PRIORITY_networking-layer = "5"

# This should only be incremented on significant changes that will  
# cause compatibility issues with other layers  
LAYERVERSION_networking-layer = "1"

LAYERDEPENDS_networking-layer = "core"  
LAYERDEPENDS_networking-layer += "openembedded-layer"  
LAYERDEPENDS_networking-layer += "meta-python"

LICENSE_PATH += "${LAYERDIR}/licenses"

# used by waf-samba.bbclass  
WAF_CROSS_ANSWERS_PATH = "${LAYERDIR}/files/waf-cross-answers"  
```

Ta sẽ thấy 3 dòng có chữ LAYERDEPENDS, hay là các layer mà layer này phụ thuộc.  
Tức là để build được layer này, ta phải build cả 2 layer kia nữa.

Vì thế, ta phải sửa file **build/conf/bblayers.conf** thành như sau:

```bash  
# POKY_BBLAYERS_CONF_VERSION is increased each time build/conf/bblayers.conf  
# changes incompatibly  
POKY_BBLAYERS_CONF_VERSION = "2"

BBPATH = "${TOPDIR}"  
BBFILES ?= ""

BBLAYERS ?= " \  
/home/oedev/raspberrypi_krogoth/poky/meta \  
/home/oedev/raspberrypi_krogoth/poky/meta-poky \  
/home/oedev/raspberrypi_krogoth/poky/meta-yocto-bsp \  
/home/oedev/raspberrypi_krogoth/poky/meta-raspberrypi \  
/home/oedev/raspberrypi_krogoth/poky/meta-openembedded/meta-python \  
/home/oedev/raspberrypi_krogoth/poky/meta-openembedded/meta-oe \  
/home/oedev/raspberrypi_krogoth/poky/meta-openembedded/meta-networking \  
/home/oedev/raspberrypi_krogoth/poky/meta-local \  
```

*   Tiếp theo, thêm **samba server** vào ảnh:  
    Theo tài liệu [Yocto Project Development Manual] (http://www.yoctoproject.org/docs/1.8/dev-manual/dev-manual.html), có 2 cách để thêm vào image. Thứ nhất là, thêm vào biến **IMAGE_INSTALL_append** có trong file **build/conf/local.conf**. Cách thứ 2, tạo một layer mới, là layer sau cùng được build, layer chỉ chứa gói phần mềm **samba** mà thôi.  
    Cách thứ 2 sẽ ít ảnh hưởng hơn cách thứ nhất vì ta không phải động đến file có sẵn là **local.conf**. Vì thế, ta sẽ làm theo cách 2.

Tạo cây thư mục với đường dẫn như sau:

```bash  
poky/meta-local/  
├── conf  
│   └── layer.conf  
└── recipes-core  
└── images  
└── rpi-basic-image.bbappend  
```

Trong đó:  
**layer.conf**: là file cấu hình layer **local** này, có có mẫu chung rồi.  
trong trường này, ta sẽ sử dụng nội dung sau:

```bash  
# We have a conf and classes directory, add to BBPATH  
BBPATH .= ":${LAYERDIR}"

# We have recipes-* directories, add to BBFILES  
BBFILES += "${LAYERDIR}/recipes-*/*/*.bb ${LAYERDIR}/recipes*/*/*.bbappend"

BBFILE_COLLECTIONS += "local"  
BBFILE_PATTERN_local = "^${LAYERDIR}/"  
BBFILE_PRIORITY_local = "7"  
```

Ta nên để **PRIOIRTY** càng cao càng tốt, tức là độ ưu tiên thấp khi build.

**rpi-basic-image.bbappend** sẽ có nội dung như sau:

```bash  
IMAGE_INSTALL += "samba \  
"  
```

*   Cuối cùng ta thực hiện lại lệnh build ảnh như đã build ở phần 1 thôi.  
    Sau khi thực hiện xong, ta sẽ có file ảnh mới với kích thước lớn hơn hẳn  
    (128MB so với 197MB thì phải)

<Một số hình ảnh kiểm chứng>