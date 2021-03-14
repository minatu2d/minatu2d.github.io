---
layout: post
title: CER/DER trong ASN.1 Encoding
date: 2016-11-25 14:43:11.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
- Communication
tags:
- ASN1
- CER
- DER
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '29295846168'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/11/25/phuong-thuc-cerder/"
---
Khi sử dụng BER để hiện thực dữ liêu, ta thấy rằng có khá nhiều chỗ tùy ý. Tức là cùng một thông tin có nhiều cách biểu diễn khác nhau. Khi sử dụng với trường hợp chữ kí Số, phát sinh ra nhiều vấn đề.

Để giải quyết những vấn đề đó, người ta thêm ràng buộc vào **BER**, và tạo ra **CER** và **DER**.

Sự khác nhau chủ yếu giữa **CER** và **DER** là về biểu diễn trường độ dài. **Length(L)**. Trong khi **DER** sử dụng một định dạng với độ dài cố định, thì **CER** sử dụng định dạng với độ dài không có định,  
  
Cụ thể, các điểm khác biệt được nêu dưới bảng sau:

CER

DER

Trường độ dài

Với kiểu cơ bản, độ dài =128 sẽ sử dụng  
dạng dài. Còn kiểu cấu trúc, sử dụng dạng thay đổi được.

Dù kiểu cơ bản, hay cấu trúc thì đều cùng 1 quy tắc, độ dài =128 thì sử dụng dạng dài.

Kiểu logic

TRUE luôn được biểu diễn là 1.

Kiểu dãy bit

Các bit không sử dụng được gán là 0.

Với những dãy bit được định nghĩa thành 1 kiểu khác thì  
các bit 0 phía sau sẽ được bỏ đi.

Trường hợp có giới hạn về kích thước kiểu, thì các bit 0 chỉ tồn tại khi cần thiết thôi.

Khi độ dài là 0, trường độ dài có giá trị 1, nội dung sẽ là 1 Octet toàn 0.

Kiểu dãy bit

Kiểu chuỗi Octet (byte)

Kiểu chuỗi kí tự

Nếu dưới 1000 Octet, sẽ được biểu diễn như kiểu Basic

Nếu có từ 1000 Octet trở lên, sẽ được biểu diễn giống với kiểu Cấu trúc,  
trừ đoạn cuối ra thì mỗi 1000 sẽ được biểu diễn.

Được biểu diễn như kiểu cơ bản

Kiểu tuần tự

Kiểu tập hợp

Không hiện thực các giá trị mặc định.

Kiểu tập hợp

Biểu diễn tương ứng cho từng thành phần theo thẻ (phổ biến, ứng dụng, đặc trưng ngữ cảnh, riêng)

Sẽ biểu diễn theo thẻ nhỏ nhất trong số Thẻ của các phần tử lựa chọn.

Trường hợp kiểu lựa chọn (không có Tag), sẽ biểu diễn theo phần tử được chọn  
(tùy theo phần tử được chọn mà biểu diễn sẽ thay đổi theo).

Kiểu tập hợp đồng nhất

Được biểu diễn theo thứ tự tăng dần của dãy Octet