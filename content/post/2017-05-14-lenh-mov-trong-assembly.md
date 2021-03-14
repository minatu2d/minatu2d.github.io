---
layout: post
title: Lệnh MOV trong Assembly
date: 2017-05-14 07:59:46.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
tags:
- Assembly
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '5026625320'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2017/05/14/lenh-mov-trong-assembly/"
---

**Link gốc**: http://www5c.biglobe.ne.jp/~ecb/assembler/2_1.html

## 1. Lệnh MOV


Lệnh MOV, có thể nói là lệnh cơ bản nhất, có tần số sử dụng nhiều nhất trong bất cứ hệ tính toàn nào.  
Làm bất cứ cái gì, đều có bao gồm `sao chép` dữ liệu trong đó. Nói là chức năng `sao chép` thôi, chứ thực sự nó bao hàm sao chép, truyền dữ liệu. Đó có thể là từ bộ nhớ (các loại RAM) vào thanh ghi, từ thanh ghi vào bộ nhớ, từ một giá trị nào đó xuống thanh ghi hoặc bộ nhớ chẳng hạn.  
Tuy nhiên hãy nhớ rằng **KHÔNG THỂ CHUYỂN DỮ LIỆU TỪ BỘ NHỚ SANG BỐ NHỚ ĐƯỢC**,  
bởi vì **BỘ NHỚ ĐƯỢC TRUY CẬP THEO BUS (20bit hoặc 32bit), MỖI LẦN TRUY CẬP BỘ NHỚ CHỈ CÓ THỂ TRUY CẬP MỘT ĐỊA CHỈ MÀ THÔI**  

`

Lệnh MOV có 2 tham số, được gọi là **Operand**, nó giống như bên dưới đây:

```asm
MOV DEST, SOURCE  
```

Lệnh này theo cú pháp Intel nha.  
Chuyển (Copy) dữ liệu từ **SRC** đến **DEST**  
**SRC**, **DEST**: là địa chỉ thanh ghi hoặc bộ nhớ, bỏ trường hợp cả 2 là bộ nhớ.

Một số ví dụ:

```asm
MOV AX, BX : Copy nội dung từ thanh ghi BX sang thanh ghi AX  
MOV AL,10 : Lưu giá trị 10 vào thanh ghi AL  
```

`Một số chú ý`:  
- Không thể chuyển dữ liệu giữa bộ nhớ với nhau.  
- Nguồn và đích chuyển dữ liệu phải có cùng kích thước  
- Không thể chuyển giá giá trị vào thanh ghi segment`