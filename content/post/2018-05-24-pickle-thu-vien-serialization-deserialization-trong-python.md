---
layout: post
title: Pickle - Thư viện Serialization/Deserialization trong Python
date: 2018-05-24 07:53:35.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Python
tags:
- Deserialization
- Pickle
- Serialization
meta:
  _wpcom_is_markdown: '1'
  timeline_notification: '1527148418'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '18198703290'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2018/05/24/pickle-thu-vien-serialization-deserialization-trong-python/"
---
Gần đây khi chạy thử một vài thuật toán Deep Learning, đặc biệt là các thuật toán cung cấp source code sử dụng thư viện Caffe, có gặp một vài vấn đề liên quan đến module **Pickle** trong Python.  
Thực ra, bình thường cũng không định viết gì cả. Nhưng chợt nhớ ra mình cũng quan tâm đến chủ đề **Serialization/Deserialization** này. Nó có vai trò giống như [ASN.1](../2016-11-12-asn-1-la-gi-tai-sao-no-quan-trong) mà có tìm hiểu hồi trước, hay phổ biến nhất của thể loại này là **Protocol Buffer** của Google, hay **Apache Thrift**. Thế là note lại cái để quên nó đi.

## 1. Pickle - Chuyển đổi một đối tượng Python sang dạng binary và ngược lại

*   Sử dụng cách thức chuyển đổi riêng (chỉ dành riêng cho Python), không gắn với bất cứ một chuẩn chuyển đổi nào.
*   Không có cơ chế định nghĩa dữ liệu tách biệt với việc mã hóa dữ liệu đó.  
    Dữ liệu được ghi ra phải là Python Object, và chỉ có thể đọc được trong Python.
*   Có thể ghi ra/load vào hầu hết đối tượng của các kiểu dữ liệu phổ biến
*   Các file pickle được ghi ra hoàn toàn có thể đọc được ở một môi trường Python khác. (Tất nhiên có tương thích về phiên bản protocol)

## 2. Pickle - là không secure

Mặc định, nó sẽ import tất cả các module có trong dữ liệu từ file được load lên.  
Thế này thì vãi quá.  
Bởi vậy, người ta khuyên không nên cứ thế load vào một file pickle không rõ nguồn gốc. vì éo ai biết nó load gì và sẽ chạy gì sau đó.