---
layout: post
title: "[OE] Tại sao vẫn dùng FAT"
date: 2015-12-27 13:17:33.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
tags:
- FAT
- FileSystem
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '18156579994'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2015/12/27/oe-tai-sao-van-dung-fat/"
---
Bài hôm nay nói về một hệ thống file được sử dụng khá nhiều trên các phần mềm chạy trên mạch.

Dù hiện nay có rất nhiều hệ thống file được nói đến như NTFS, Ext3, Ext4...Hầu hết những hệ thống file đó được sử dụng chủ yếu trên các máy có  năng lực tính toán cao và khả năng lưu trữ lớn. Thế còn với những máy có năng lực tính toán hạn chế, dung lượng lưu trữ nhỏ thì sẽ sử dụng hệ thống file nào. Theo ý kiến cá nhân, hầu hết sẽ là FAT, một hệ thống file được ra đời từ những năm 1970.

Nói về FAT, so với những phiên bản đầu thì càng về sau càng được cải tiến đi rất nhiều. Tuy có nhiều cải tiến đi nữa thì FAT cũng khó mà phù hợp với những hệ thống lưu trữ cực lớn vì bản thân nó có giới hạn về dung lượng lưu trữ.

Từ FAT12, FAT16, đến FAT32, mọi thứ đã thay đổi rất nhiều. Nhưng phần cốt lõi của FAT thì không thay đổi. FAT đơn giản về cấu tạo, dễ dàng (tính chất tương đối) để implement, dễ hiểu nữa.

Một số đặc điểm của FAT có thể nêu ở đây:

1.  Một hệ thống file FAT khi được cài đặt sẽ có 3 phần: Phần BPB (BPB - BIOS Parameter Block), Phần FAT Entry (đánh chỉ số), Phần thư mục gốc, và cuối cùng là Phần lưu trữ dữ liệu. Phần BPB sẽ chứa các thông tin về thiết lập của FAT hiện tại, như kích thước sector, kích thước cluster,...Phần FAT Entry được sử dụng như một mảng 1 chiều, mỗi vị trí chưa giá trị của cluster tiếp theo. Kích thước theo bit của giá trị sẽ tương ứng với FAT12, FAT16, hay FAT32. Ví dụ: FAT12 thì giá trị đuợc biểu diễn ở 1 byte + 1/2 byte, FAT16 thì giá trị được biểu diễn ở 2 byte, FAT32 thì giá trị được biểu diễn trong 4 byte. Phần thư mục gốc, phần này chứa thông tin về file, thư mục của thư mục gốc. Phần lưu trữ dữ liệu, phần này là dữ liệu chúng ta lưu trữ.
2.  Không có các phương thức bảo mật hay kiểm tra độ bảo toàn của dữ liệu được lưu trữ.
3.  Có thể cài đặt trên các thiết bị nhớ có dung lương rất nhỏ, tầm dưới 1MB.
4.  Không yêu cầu nhiều xử lý phức tạp như tạo index cho việc tìm kiếm...
5.  Được hầu hết các hệ điều hành Support.
6.  Hoàn toàn miễn phí (trừ một số chức năng nâng cao)