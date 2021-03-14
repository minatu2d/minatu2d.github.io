---
layout: post
title: Cờ Retain và QoS trong message PUBLISH
date: 2016-02-21 15:41:14.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- IoT
tags:
- mqtt
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: '43452'
  _publicize_job_id: '20028531205'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/02/21/khai-quat-ve-mqtt/"
---
Giao thức MQTT dựa trên mô hình publisher/subscriber (nhà cung cấp/người sử dụng), publisher có dữ liệu trên một lĩnh nào đó (được đại diện bằng các topic), phía subscriber (người mong muốn thông tin) sẽ mong muốn thông tin trên môt lĩnh vực nào đó (cũng được miêu tả bằng topic). Môt anh trung gian ở giữa sẽ nhận từ rất nhiều anh cung cấp trên rất nhiều lĩnh vực, sau đó xem xét, thông tin phù hợp để gửi đến những người sử dụng mà anh trung gian này biết.

Message PUBLISH được gửi từ publisher đến server (hay broker, hay chính là anh trung gian) để cung cấp thông tin.

Bài này nói về 2 trường của message PUBLISH, đó là QoS và Retain. Và 2 trường này tuy nằm ở phần **header cố định** (có trên mọi message). Về trường QoS chỉ được sử dụng ở 3 Message, trong đó PUBLISH (dùng cả 3 level 0, 1, 2), SUBSCRIBE và UNSUBSCRIBE (chỉ dùng level 1). Bài này chỉ nói về message PUBLISH thôi.

1.  Trường QoS
2.  Trường Retain
3.  Ví dụ thực tế với mosquitto

### 1. Trường QoS (Byte:0, Bit : 2-1, Size:2 bit)

Trường này sẽ dùng để miêu tả mong muốn mà Publisher muốn server đảm bảo việc truyền message đến các subscriber. Có 3 mức độ mà subscriber có thể yêu cầu.

_00b_ -> cùng lắm là 1 lần. không sử dụng thêm phương thức đảm bảo độ tin cậy nào khác mà tin tưởng vào TCP/IP luôn, cho chiều gửi của PUBLISH.

Như bảng sau:

Publisher (bên cung cấp)

Message và hướng gửi

Server

QoS = 0

PUBLISH  
`---------->`

**Hành động :**

Đẩy message đến các Subscribers hiện tại.

_01b_ -> Ít nhất 1 lần, khi Server nhận được message này. Nó thực hiện lần lượt 3 bước " lưu trữ tạm thời message vừa nhận được, gửi message nhận được đến các subscriber hiện tại, cuối cùng xóa message đã lưu. Sau khi thực hiện được bước cuối cùng (xóa message), nó sẽ gửi PUBACK (PUBLISH ACK) về phía Publisher đã gửi gói PUBLISH để thông báo rằng việc chuyển đi đã kết thúc. Nếu phía Publisher sau một thời gian nhất định không nhận được PUBACK nó sẽ coi như bị timeout và gửi lại gói PUBLISH này (với trường DUP được set 1)

Như bảng sau:

Subscriber(bên cung cấp)

Message và hướng gửi

Server

QoS = 1  
DUP = 0  
Message ID = x**Hành động:** Lưu tạm thời message sẽ gửi

PUBLISH  
`---------->`

**Hành động:**

*   Lưu trữ message
*   Đẩy các message đến các Subscribers
*   Xóa message

**Hành động:** Hủy message đã lưu

PUBACK  
`<----------`

Phía Server (broker) khi nhận được gói PUBLISH (có trường DUP được set 1), nó sẽ thực hiện các hành động như ở trên. (chính vì thế, có thể xảy ra tình trạng 1 subscriber nhận cùng 1 nội dung đến 2 lần)

_10b_ -> Chính xác một lần, có nghĩa là các message được gửi lên từ Publisher qua gói PUBLISH sẽ chuyển đến các Subscriber đúng 1 lần. Ở mức QoS này (mức cao nhất), sẽ không cho phép việc chuyển lặp lại message.

Có 2 kịch bản khi sử dụng level này, phía Server (broker) sẽ tùy chọn implement option nào.

Luồng chuyển message được miêu ta như bảng sau:

Client (bên cung cấp)

Message and hướng gửi

Server

QoS = 2  
DUP = 0  
Message ID = x**Hành động:** Lưu message

PUBLISH  
`---------->`

**Hành động:** Lưu message

_hoặc_

**Hành động:**

*   Lưu message ID
*   Chuyển message đến các subsriber

PUBREC  
`<----------`

Message ID = x

Message ID = x

PUBREL  
`---------->`

**Hành động:**

*   Chuyển message đến các subscriber
*   Xóa message

_or_

**Action:** Xóa message

**Hành động:** Quên message đã lưu

PUBCOMP  
`<----------`

Message ID = x

### 2. Trường Retain (Byte:0, Bit:0)

Chỉ được sử dụng ở PUBLISH. Mục đich để báo với Server biết nên giữ lại message sau khi chuyển đến các Subscriber hiện tại. Message giữ lại sẽ được sử dụng để gửi đến các Subsriber mới subsriber topic mà message này thuộc về.

Trường này sẽ có tác dụng trong trường hợp khi mà Publisher cần thông báo đến các Subsribers mới biết về sự tồn tại của nó. Nó giống như là để lại dấu vết.

Tuy nhiên, đối với các Subsriber đang kết nối đến Server thì trường Retain này sẽ không được Set. Điều này giúp Subscriber phân biệt được đâu là message "mới toanh", đâu mà message được giữ từ trước đó.

Các message phải được lưu lại ngay cả khi khởi động lại server.

Để xóa message đã yêu cầu lưu lại, Publisher chỉ cần gửi một message cùng topic nhưng với trường payload bằng 0.

### 3. Ví dụ thực tế với mosquitto

Ta sẽ thực nghiệm với mosquitto

*   Trường QoS : level = 00b
*   Trường QoS : level = 01b
*   Trường QoS : level = 10b
*   Trường Retain = 0
*   Trường Retain = 1