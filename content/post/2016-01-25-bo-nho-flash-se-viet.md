---
layout: post
title: Bộ nhớ Flash
date: 2016-01-25 14:29:09.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
- Embedded
tags:
- Draft
- Flash
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '19106965273'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/01/25/bo-nho-flash-se-viet/"
---
Để thành 1 bài thì hơi nhiều, những thôi đã để thành 1 bài thì vẫn phải viết.

## Lập trình với bộ nhớ Flash

Nếu là Flash memory, khi lập trình nhất định phải có một thao tác xóa trước một thao tác ghi. Sau khi xóa, giá trị tất cả các ô nhớ là 1, tức là nếu đọc ra ta sẽ thấy toàn **0xFF** thôi.

## Serial Flash

Có rất nhiều loại Flash, nhưng trên các ứng dụng embedded thì Serial Flash được sử dụng phổ biến nhất. Vì giao tiếp được bằng các chuẩn Serial như SPI, I2C nên nó dễ dàng ghép nối với các chip trên board.

## NAND và NOR

Có 2 điểm khác biệt giữa 2 loại chip nhớ này. Đó là

*   Kết nối giữa memory cell khác nhau. Trong NOR, các phần tử nhớ kết nối song song với nhau, nên cho phép lập trình với từng cái. Còn NAND thì được kết nối dạng nối tiếp
*   Giao diện để đọc, ghi khác nhau (NOR cho phép đọc Random-Access, NAND chỉ cho phép đọc theo Page)

Bảng so sánh giữa NAND và NOR

Attribute

NAND

NOR

Main Application

File Storage

Code execution

Storage capacity

High

Low

Cost per bit

Better

.

Active Power

Better

.

Standby Power

.

Better

Write Speed

Good

.

Read Speed

.

Good