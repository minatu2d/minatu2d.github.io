---
title: "Tensorflow và AVX"
date: 2019-10-31T22:17:58+09:00
published: true
type: post
parent_id: '0'
status: publish
toc: true
categories:
- fixproblem
tags:
- tensorflow
- avx
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
---

## 1. Lỗi liên quan đến `CPU instruction` của tensorflow
- Định cài đặt `tensorflow-gpu` trên con máy già chạy `Intel Xeon X5672` và có gắng `GTX 1060 3G` để nghịch chút.
- Nhưng khi kiểm tra xem có chạy được không:
```python
>>> import tensorflow as tf
Illegal instruction (コアダンプ)
```
- Vâng, là lỗi run-time - chỉ xảy ra khi chạy, chứ cài thì chưa xuất hiện vấn đề.

## 2. `AVX` là gì
- Ta tìm hiểu một chút về lỗi kể trên, thì hầu hết đề nói rằng: máy sử dụng CPU không hỗ trợ đầy tập lệnh mà tensorflow đang dùng. 
- Phổ biến là thiếu là `AVX - Advanced Vector Extension`.
- Theo wikipedia:

> **Advanced Vector Extension** là phần mở rộng cho bộ vi xử lý kiến trúc x86 (cả Intel và AMD), được Intel đề xuất năm 2008. Lần đầu tiên được hỗ trợ trên bộ xử lý `Sandy Bridge` năm Q1/2011. Sau đó đến Q3/2011 thì AMD cũng hỗ trợ. `AVX` cung cấp tính năng mới cùng các lệnh mới và cấu trúc thực hiện cũng khác.

- Trong đó, `AVX` có giới thiệu một loại phép tính toán gọi là `fused multipl-add` (dịch là tổ hợp nhân cộng). Phép tính này giúp làm tăng đáng kể việc tính toán số học như: phép nhân ma trận (dot-product, convolution). Nó có thể làm tăng tốc độ lên gấp 3 với các phép tính kiểu này.
- Một nữa, tất cả những điều này chỉ dành cho `CPU` thôi.

## 3. Tại sao lại dùng nó và không dùng `AVX`
- Thông thường, `tensorflow` vẫn có thể built được mà không cần `phần mở rộng của CPU` như `SSE4.1`, `SSE4.2`, `AVX`, `AVX2`, `FMA`...etc. Vì như thế sẽ tương thích với nhiều `CPU` nhất có thể.
- Và tốc độ của nó so với `GPU` thì ko đáng kể gì. Bởi vậy, với quy mô chạy hơi lớn một chút thì ta nên dùng `GPU`.

## 4. Giải quyết vấn đề lỗi với `AVX` như thế nào
- Tải phiên bản cũ từ `1.5` trở về trước:
```
pip install tensorflow==1.5
```