---
layout: post
title: Driver và Firmware trong Linux
date: 2016-08-29 08:25:25.000000000 +09:00
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
- Firmware
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '26257719657'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/08/29/khai-niem-driver-va-firmware-trong-linux/"
---
Gần đây, khi tìm hiểu cách cài đặt driver cho USB Wifi, mình có tìm hiểu thêm về quá trình tạo nhân Linux. Đặc biệt, việc thiết lập cấu hình trước khi build tạo ảnh của kernel.  
Mình thấy ngoài **Driver**, tức là thành phần trung gian giữa ứng dụng và phần cứng, còn có 1 khái niệm nữa. Đó là **Firmware**.  
  
Bài này sẽ dịch lại [trang](https://wiki.ubuntu.com/Kernel/Firmware), để hiểu qua về Firmware trong Linux Kernel.

## 1. Firmware ở đây là gì?

Nhiều thiết bị có tới 2 thành phần để tạo nên chức năng của chúng khi trong hệ điều hành. Đầu tiên, đó là một **Driver**, là phần mềm cho phép hệ điều hành nói chuyện được với phần cứng của thiết bị. Thứ hai, đó là **Firmware**, thường là một mã nhị phân được upload trực tiếp vài thiết bị để nó có thể hoạt động được. Bạn có thể nghĩ Firmware ở đây là cách để lập trình phần cứng bên trong thiết bị. Trong thực tế, hầu hết **Firmware** đều có dạng (closed-source), thường không có source code đi kèm, và được phân phối miễn phí.

## 2. Lấy Firmware như thế này ở đâu?

**Firmware** thường được maintained bởi chính các công ty phát triển thiết bị. Trên Windows, **firmware** thường là một phần của **Driver** bạn cài đặt để sử dụng thiết bị đó, và tất nhiên, user không hề thấy nó. Trên Linux, **firmware** đến từ rất nhiều nguồn. Một số có ngay trong source của nhân. Số khác được phân phối từ [upstream](http://git.kernel.org/cgit/linux/kernel/git/firmware/linux-firmware.git). Một số khác thì thật không may, chúng không được phân phối miễn phí.

Trên Ubuntu, **firmware** có thể đến từ 1 trong các nguồn dưới đây:  
- Gói **_linux-image_**  
- Gói **_linux-firmware_**  
- Gói **_linux-firmware-nonfree_**  
- Gói driver riêng biệt  
- Các nguồn khác như Driver CD, Email, Website.

Chú ý rằng, gói **_linux-firmware-nonfree_** không được cài mặc định.  
Các **firmware** đã được cài đặt sẽ nằm ở **/lib/firmware/**.

## 3. Các Firmware này được sử dụng lúc nào?

Khi driver cho các thiết bị này được chạy, nó sẽ "yêu cầu" các **firmare** tương ứng từ **_lib/firmware/_**  
Quá trình về cơ bản sẽ như sau:  
1. Driver yêu cầu file Firmware.  
2. Kernel sẽ gửi 1 yêu cầu đến **udev** để hỏi Firmware  
3. **udev** sẽ chạy 1 script để load dữ liệu từ firmware vào 1 file đặc biệt được tạo bởi Kernel.  
4. Kernels sẽ đọc đọc Firmware từ file đó và chuyển cho Driver.  
5. Driver sau đó sẽ bằng cách nào đó load firmware vào thiết bị.

**Vị dụ về USB Wifi Buffalo sử dụng chip Ralink**

Thông thường các module khi yêu cầu **External Firmware**, nhân sẽ tìm trong đường dẫn **/lib/firmware**. Đường dẫn này được set trong biến **CONFIG_EXTRA_FIRMWARE_DIR** khi cấu hình để build kernel.

Trong trường hợp của chúng ta, module đã được load nhưng firmware dành cho thiết bị không có trong **/lib/firmware**, có lẽ ta chỉ cần copy vào thư mục trên là vấn đề sẽ được giải quyết.