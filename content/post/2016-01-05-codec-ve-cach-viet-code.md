---
layout: post
title: "[CodeC] Về cách viết code"
date: 2016-01-05 15:02:39.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- CodeSkill
- Khác
tags:
- C
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '18439615882'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/01/05/codec-ve-cach-viet-code/"
---
Việc code thường hay phải lặp đi lặp lại rất nhiều thao tác trong nhiều dự án khác nhau. Có thể kể đến nhưng scanf, fread, fwrite....print, debug...

Giờ có một design, tức là flow chart+ danh sách tên hàm.

Flow chart thì rõ là phải viết rồi, có thể đặt tên theo các xử lý, gắn các mã. Nó dù rất giống với phần implement nhưng ta có thể tách bạch được.

Phần danh sách tên hàm và các tham số, nếu lúc thiết kế danh sách tên hàm mà ta đã nghĩ luôn đến việc đặt tên sao cho chuẩn, sao cho nó là duy nhất trong scope của nó thì quả thật rất vất.

Thay vì thế, ta có thể dùng các tên hàm được gọi là chuẩn, tức là các hàm được định nghĩa trong các thư viện chuẩn của ngôn ngữ C.

Ta cứ viết thoải mái đi, không cần để ý nhiều đến hàm số, cũng không cần để ý hàm đó sẽ là hàm từ thư viện chuẩn hay một hàm stub ta phải viết.

Khi implement một khung cho thiết kế, ta thấy code sẽ rất rõ ràng, sáng sủa.

Tuy nhiên, việc không nghĩ đến các tên, hàm, cũng như tham số của nó khi trước có thể dẫn đến việc ta không thể build succecced được.

Có một cách rất đơn giản là, sử dụng define để loại bỏ các hàm mà ta chưa implement được cũng như các hàm bị trùng với các thư viện chuẩn.

Nếu đã có một hàm khác cho các xử lý bị loại bỏ đó, ta có thể dùng define để thay thế các đoạn code lúc trước, các này vẫn dảm bảo dịch ok mà code chương trình luôn đẹp và chuẩn.

Ví dụ: trong chương trình sau, dầu ra chuẩn là một terminal, chứ không phải một màn hình.

Ta vẫn viết code như ta code với dầu ra màn hình sử dụng, printf...

File C nào chưa hàm print cần thay thế, ta chỉ cần define lại là xong.