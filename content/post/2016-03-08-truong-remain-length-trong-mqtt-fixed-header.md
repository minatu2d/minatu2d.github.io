---
layout: post
title: Trường Remain Length trong MQTT Fixed Header
date: 2016-03-08 15:39:23.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
- Communication
- IoT
tags:
- mqtt
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: '43452'
  _publicize_job_id: '20555779060'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/03/08/truong-remain-length-trong-mqtt-fixed-header/"
---
Trước kia, đã từng học rất nhiều thứ về hệ điều hành. Nhưng hầu hết những nguyên lý được nói đến đều lấy Windows, hoặc Linux(*Unix), hoặc Mac làm tham chiếu đến những nội dung được học.

Ai cũng biết sẽ có một phần mềm hệ thống gọi là Kernel, nó rất quan trọng nó lập lịch các tiến trình, quản lý bộ nhớ, các driver... Thực sự cũng từng đọc code tham khảo (dành cho Academy của M$ về Windows NT), nhưng thực sự vẫn chưa có một hình dung tương đối về cái Kernel kia.

Gần đây, do chạy thử các Sample trên môi trường Non-OS, rồi OS. Nên vỡ ra được một số điều. Tất nhiên chưa được kiểm chứng kĩ lượng, đây chỉ là sự ghép các kiến thức rời rạc, rồi nắn cho nó nghe có vẻ logic. Không sao, cứ đưa ra dự đoán, chúng sẽ được kiểm chứng bằng trải nghiệm thực tế trong công việc.

Cái NORTi (một hệ điều hành implement các API cho RTOS của iTRON Project)

## 1. Diện mạo bên ngoài:

Không giống với Linux hay Windows, ta thường thấy một con trỏ nhấp nháy trên màn hình, để người dùng nhập lệnh hay một màn hình đồ họa cho phép thao tác bằng chuột và keyboard. Và hầu hết ta sẽ có quá trình cài đặt 2 hệ điều hành này vào máy trước khi chạy.

Còn với NORTi, nó là RTOS dành cho microcontroller (có thể sử dụng ở cả máy tính nữa), nó được biên dịch với các ứng dụng, sau đó sẽ được đưa vào Microcontroller.

Quá trình biên dịch giống hệt việc ta build một ứng dụng trên Linux có sử dụng một Library (cái loại mà chỉ có mã binary, không có source code ấy).

Kết quả của quá trình build trên Linux là File chạy mà Linux chạy được.

Còn kết quả của quá trình build với NORTi là một file binary + file memory map.

Binary và map sẽ được nạp vào Microcontroller để chạy. Ta vẫn gọi là nạp Firmware.

Khi chưa có firmware thì con microcontroller này ko khác một viên gạch hay một thanh tre có cùng kích thước và trọng lượng. Chẳng có tiếng Beep vào giống như ta mở PC. Có chăng có 1 điểm giống với PC là có vài cái đèn sẽ sáng khi có nguồn điện.

## 2. Các thiết bị ngoại vi

Về cơ bản, RTOS sẽ chạy trên các chip được tích hợp rất nhiều chức năng. Nhưng chức năng ấy là gì. Nói ra thì nhiều vô kể, nhìn vào danh sách chức năng của một con Chip, ta những tưởng nó là hệ điều hành, một "nồi lẩu" với rất nhiều chức năng. Tất nhiên, rất ít ứng dụng cụ thể nào sử dụng hết được.

Đó là giao tiếp tuần tự (Serial) như SPI, I2C..., hay các giao tiếp song song như Pararell, các giao tiếp mạng, các giao tiếp chuyên dụng như USB, CAN.

Bản thân con chip sẽ cung cấp các thanh ghi cho phía ứng dụng để có thể cấu hình, rồi truyền đầu vào, nhận đầu ra.

Về mặt phần mềm, việc phải làm là đọc hiểu cách hoạt động của các chức năng được cung cấp đó thông qua Datasheet, Userguide, và dễ tiếp cận nhất là qua các Sample Source.

Về phần cứng, không rõ nên cũng không ghi ra đây.

## 3. Ngắt

Nguyên tắc giao tiếp với thiết bị ngoại vi của máy tính từ xưa đến này thông qua 2 cách là Ngắt và Polling.

Nhiều chip hỗ trợ ngắt ở nhiều mức ưu tiêu khác nhau. Việc này có rất nhiều lợi ích. Lợi ích lớn nhất là có phép ưu tiên một xử lý có độ ưu tiên cao không bị ngắt quãng. Ví dụ như một số đoạn lập lịch trong nhân không thể bị ngắt quãng được.

## 4. Cái gì diễn ra khi NORT được khởi động

Các ứng dụng trên Microcontroller thì thường được chạy cách như sau:

Khởi tạo các vùng trên RAM, Clock của hệ thống.

-------->

Khởi tạo các thiết bị ngoại vi (gồm bật cái nào, tắt cái nào, mở ngắt nào, tắt ngắt nào)

-------->

Chạy ứng dụng (có thể là hệ điều hành, có thể là hàm main của một sample code)

Mỗi CPU thì có một địa chỉ mặc định để nhảy đến sau khi tự nó hoàn thành các kiểm tra bên trong nó. Việc chúng ta load firmware xuống Microcontroller là việc ta nạp một dãy rất dài các lệnh máy mà lệch bắt đầu phải được đặt ở vị trí mặc định của Chip.

Trong trường hợp của NORTi, hoặc các hệ điều hành ITRON-based khác, nó cung cấp một đoạn chương trình nhỏ thực hiện các bước khởi tạo (có thể theo luồng phía trên). Kết thúc đoạn chương trình đó, nó sẽ nhảy tới địa chỉ hàm main() để chay ứng dụng ta đã viết.

## 5. Quá trình multi-tasking được thực hiện như thế nào?

Hầu hết các dòng Chip có thể cài được RTOS sẽ cung cấp 1 chức năng cho phép bật tắt việc xảy ra ngắt. Tức là bằng một thao tác ghi trên một ghi R nào đó, phần mềm có thể ngăn tạp thời toàn bộ, hoặc một số ngắt xảy ra. Cũng bằng thao tác ghi trên thanh R đó (tất nhiên với giá trị khác), phần mềm có thể cho phép các ngắt đã bị tạm dừng tiếp tục đươc thực hiện.

Việc này có ý nghĩa hết sức quan trọng, ta thấy rằng để có hệ điều hành RTOS, ta phải có một kernel. Kernel này cũng là một phần mềm, nhưng nó thao tác trực tiếp với các thanh ghi của CPU chứ không phải thông qua các API dễ hiểu. Kernel có rất nhiều đoạn code được gọi là critical (hay nói cách khác là không thể bị ngắt ở chỗ đó được).

Để thực hiện này, nó sẽ làm theo flow sau:

Ngăn không cho các ngắt ảnh hưởng đến nhân (tạm thời)

------>

Thực hiện xử lý critical của kernel.

------>

Cho phép lại các ngắt đã bị ngăn

Vì chức năng dừng ngắt này được cung cấp bởi CPU nên phần mềm không phải tự tạo một cơ chế như thế.

Chế Lock hay Unlock cũng có thể được thực hiện bằng cách giống như trên.

## 6. Một số chú ý khi sử dụng ITRON

Nếu MCU không hỗ trợ nhiều hơn 2 độ ưu tiên cho ngắt mà chỉ có tắt, mở thôi thì ta không cần quan tâm đến việc