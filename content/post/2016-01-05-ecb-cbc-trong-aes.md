---
layout: post
title: ECB, CBC trong AES
date: 2016-01-05 14:48:34.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Security
tags:
- AES
meta:
  _wpcom_is_markdown: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '18439227781'
  _rest_api_published: '1'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/01/05/ecb-cbc-trong-aes/"
---
Một trong những phương pháp mã hóa dữ liệu được sử dụng như là một chuẩn. Đó là **AES** (Advanced Encryption Standard)

Về mã hóa, là quá trình biến dữ liệu rõ (**plain**) thành dữ liệu nhiễu (**encrypt**).

## 1. Hai cách ứng dụng

Mã hóa nói chung, thường sử dụng dụng một phương pháp toán học, mã học nào đó để làm nhiễu một cách chủ động thông tin ban đầu.

Phương pháp (method) mã hóa là các bước toán học để tạo ra dữ liệu nhiễu.

Một số loại mã hóa, người ta có thể giấu phương pháp đi, ai biết phương pháp thì mới đọc được dữ liệu nhiễu. Nhưng cách này không hay vì gặp phải nhiều vấn đề trong chia sẻ dữ liệu, tức là phải đi giải thích cả phương pháp lẫn các tham số đi kèm phương pháp đó khi người khác muốn sử dụng.

Một số loại khác, là công khai phương pháp mã hóa những giấu đi 1 yếu tố là giống như các tham số của phương pháp. Ví dụ là KEY. Ai có KEY, nếu áp dụng đúng phương pháp sẽ giải mã được.

## 2. Các từ khóa

Trường hợp AES,dù đã public, cũng khá dễ hiểu cho việc implement nhưng hiểu chứng minh thì cực kì khó với người không phải chuyên môn.

**Key** : Là key thôi, nó sẽ đuợc sử dụng khi áp dụng phương pháp mã Hóa.

**Block**: Đơn vị cho việc mã hóa, đơn vị tối thiểu để mã hóa, 1 khối dữ liệu bản plain sẽ cho ra một khối dữ liệu encrypted cùng độ dài.

Thông thường, để mã hóa một đoạn dữ liệu độ dài bất kì, người ta phải chia khối đó thành những Block đơn vị rồi mã hóa cho từng khối đó.

Để tăng độ phức tạp cho việc mã hóa, người ta có thể tạo ra mối liên hệ giữa các block với nhau.

## 3. ECB và CBC

2 khái niệm trong AES muốn nói đến ở đây là : ECB và CBC.

**ECB** : Các block được mã hóa riêng rẽ, một block bị hỏng hoàn toàn không ảnh hưởng đến việc giải mã các block khác

**CBC** : Khi mã hóa mỗi block thì ngoài việc sử dụng KEY ra thì nó còn sử dụng kết quả của Block trước đó làm tham số. Cứ như thế, khi hỏng 1 block có thể ảnh hưởng đến rất nhiều block khác nữa.