---
layout: post
title: Quá trình load firmware của Driver trong Linux
date: 2016-09-05 01:44:02.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Embedded
tags:
- Driver
- Firmware
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '26491667057'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/09/05/qua-trinh-load-firmware-cua-driver-trong-linux/"
---
### Để các kernel module sử dụng được các **firmware** thì cần 2 điều kiện sau:

1.  Khi kernel được build, các tham số sau phải được bật (ENABLE)  
    **CONFIG_FW_LOADER** : Cho phép load firmware  
    **CONFIG_EXTRA_FIRMWARE > CONFIG_EXTRA_FIRMWARE_DIR** : Đường dẫn chưa các firmware.
2.  Các firmware phải được copy vào thư mục **CONFIG_EXTRA_FIRMWARE_DIR** đã set ở trên.

### Một số điều lưu ý liên quan đến **firmware**:

1.  Các firmware là binary(closed source) được cung cấp từ các nhà sản xuất thiết bị,  
    nó không nằm trong luồng chính của kernel, được maintain ở địa chỉ sau:  
    (http://git.kernel.org/cgit/linux/kernel/git/firmware/linux-firmware.git)[http://git.kernel.org/cgit/linux/kernel/git/firmware/linux-firmware.git]
    
2.  Các firmware này không phụ thuộc vào platform chạy Linux, cho dùng là ARM, x86 hay MIPS, etc đều dùng chung 1 nguồn cả. Vì thế, nếu bản Linux Embedded xuất hiện lỗi thiếu firmware cho thiết bị mà chạy tốt trên Linux Desktop, thì cách giải quyết rất có thể là copy firmware bị thiếu từ Desktop sang Linux Embedded.