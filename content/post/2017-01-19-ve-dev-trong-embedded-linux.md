---
layout: post
title: Về /dev trong Embedded Linux
date: 2017-01-19 16:19:32.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
- Buildroot
tags:
- device file
- Kernel
- systemd
- udev
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '933215210'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2017/01/19/ve-dev-trong-embedded-linux/"
---
Đây là nội dung **pick up** từ [manual](https://buildroot.org/downloads/manual/manual.html) của Buildroot.

Nó mô tả khá rõ về **/dev** trong hệ thống Linux, cùng  
với các giải pháp dành cho hệ thống Embedded Linux.

### 6.2 /dev management

Trên 1 hệ thống Linux, thư mục _/dev_ chứa các file đặc biệt, được gọi là  
_device files_ (hay các file thiết bị), cho phép ứng dụng phía user truy cập  
đến các thiết bị phần cứng mà Linux kernel quản lý. Nếu không có những file đặc  
biệt này, ứng dụng phía user không thể sử dụng được các thiết bị phần cứng, thậm  
chí chú có được Linux kernel nhận ra một cách đúng đắn đi chăng nữa.

Dưới _System configuration_ của Buildroot, có _/dev management_, có 4 giải pháp  
khác nhau để hangle thư mục _/dev_ này:

*   Solution 1: **Tĩnh sử dụng bảng thiết bị**  
    Đây là cách khá cũ để handle các file thiết bị trong Linux. Bằng cách này, các file  
    thiết bị sẽ được lưu trữ không đổi (persistently)trên hệ thống file gốc  
    (root filesystem) (chúng luôn không đổi qua mỗi lần reboot), và không có quá trình  
    tự động thêm, xóa những file thiết bị này khi phần được thêm, gỡ từ hệ thống. Buildroot  
    cho phép tạo ra một tập các file thiết bị này sử dụng _bảng thiết bị_ (device table),  
    mặc định chúng sẽ nằm ở _system/device_table_dev.txt_ trong source code của Buildroot.  
    File này sẽ được sử dụng khi Buildroot tạo ảnh hệ thống file cuối cùng  
    (final root filesystem image), vì thế nó sẽ không xuất hiện trong thư mục  
    _output/target_. Tùy chọn _BR2_ROOTFS_STATIC_DEVICE_TABLE_ cho phép thay đổi bảng  
    thiết bị mặc định mà Buildroot sẽ sử dụng. Bạn có thể tạo bảng thiết bị của riêng  
    cho dự án của mình.
*   Solution 2: **Động chỉ sử dụng devtmpfs thôi**  
    _devtmpfs_ là một hệ thống file "ảo" (chỉ nằm trên RAM) trong Linux kernel, được  
    được vào ở phiên bản kernel 2.6.32 (nếu bạn sử dụng kernel cũ hơn, thì không thể  
    sử dụng option này). Khi được mount vào _/dev_ hệ thống file ảo này sẽ tự động làm  
    cho các file thiết bị xuất hiện hoặc không xuất hiện tương ứng với thiết bị phần  
    cứng được thêm hay được rút khỏi hệ thống. Hệ thống file này sẽ không được bảo  
    toàn sau mỗi lần reboot, đơn giản vì nó được thay đổi động bởi kernel. Khi muốn  
    sử dụng _devtmpfs_, trong cấu hình kernel, các tham số sau phải được bật:  
    _CONFIG_DEVTMPFS_ và _CONFIG_DEVTMPFS_MOUNT_. Khi Buildroot thực hiện việc build  
    Linux kernel cho hệ thống của bạn thì nó sẽ đảm bảo chắc chắn 2 option trên được  
    bật. Còn khi bạn tự build Linux kernel bên ngoài Buildroot, bạn phải có trách nhiệm  
    bật 2 option đó (Nếu không làm thế, hệ thống sau khi build sẽ không khởi động được  
    đâu)
    
*   Solution 3: **Động sử dụng devtmpfs + mdev**  
    Cách này vẫn dựa trên hệ thống file ảo _devtmpfs_ đã nói ở trên (vì thế để sử dụng  
    được thì phải bật 2 option: _CONFIG_DEVTMPFS_ và _CONFIG_DEVTMPFS_MOUNT_) khi  
    cấu hình Linux kernel, và nó thêm một tiện ích phía userspace, đó là _mdev_ phía  
    trên của _devtmpfs_.　_mdev_ là một chương trình, một phần của Busybox, chương trình  
    này sẽ được kernel gọi mỗi khi 1 thiết bị được thêm hoặc loại bỏ khỏi hệ thống.  
    Nhờ file cấu hình tại _/etc/mdev.conf_, _mdev_ có thể được cấu hình một cách linh  
    hoạt. Ví dụ, set quyền hoặc ownership cho file thiết bị, gọi một script hoặc ứng  
    dụng bất cứ khi nào một thiết bị xuất hiện hoặc biến mất, etc. Về cơ bản, nó cho  
    phép phía userspace tham gia vào sự kiện thêm hoặc loại bỏ thiết bị. Ví dụ, việc  
    load tự động kernel modules khi thiết bị xuất hiện trên hệ thống. _mdev_ cũng đóng  
    vai trò quan trọng nếu bạn có thiết bị yêu cầu firmware để hoạt động được, nó sẽ  
    chịu trách nhiệm đẩy nội dung chuyển nội dung firmware cho kernel. _mdev_ là phiên  
    _nhẹ nhàng_ của _udev_. Thông tin chi tiết về _mdev_ và cú pháp file cấu hình có  
    thể xem tại [đây](http://git.busybox.net/busybox/tree/docs/mdev.txt)
*   Solution 4: **Động sử dụng devtmpfs + eudev**  
    Cách này cũng tiếp tục dựa trên hệ thống file ảo _devtmpfs_ đã nói ở trên, nhưng  
    thêm một ứng dụng chạy daemon (ngầm) bên trên _devtmpfs_, đó là _eudev_. _eudev_  
    là ứng dụng chạy ngầm nhận lời gọi từ kernel khi một thiết bị được thêm hoặc loại  
    bỏ khỏi hệ thống. Nó _nặng_ hơn so với _mdev_, nhưng cung cấp tính mềm dẻo tốt hơn.  
    _eudev_ là một phiên bản chạy độc lập của _udev_ - chính là ứng dụng chạy ngầm phía  
    userspace được sử dụng trong hầu hết các bản phân phối Linux desktop. Hiện tại nó  
    cũng là 1 phần của _systemd_ nữa.

Các developer của Buildroot khuyến nghị sử dụng **Solution 2** (Động chỉ sử dụng  
devtmpfs). Nếu bạn cần phía userspace được thông báo khi có sự kiện thêm/loại bỏ  
thiết bị hoặc firmware khi cần, thì **Solution 3** là một lựa chọn tốt.

Có 1 chú ý ở đây, nếu _systemd_ được chọn làm _init system_, thì _/dev_ sẽ được  
thực hiện bởi _udev_ cung cấp từ chính _systemd_.