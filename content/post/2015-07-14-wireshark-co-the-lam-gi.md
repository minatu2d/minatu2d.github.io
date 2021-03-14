---
layout: post
title: Wireshark có thể làm gì?
date: 2015-07-14 14:40:03.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Khác
- Network
tags:
- Wireshark
meta:
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '12721154421'
  _wpcom_is_markdown: '1'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2015/07/14/wireshark-co-the-lam-gi/"
---
Dù biết tool này đã lâu, từ hồi học ĐH có mở ra mở vào để làm mấy bài tập mạng theo kiểu "chống chế". Hơn nữa ngày đó, phần mạng (networking) mình cũng không nhiều kiến thức vì đang ham hố phần viết một phần mềm để đời, có giao diện chứ không đen ngòm như mấy cái thầy giáo dạy trên lớp.

Lại sắp lan man rồi, vào đề thôi.

Trong bài này sẽ trả lời mấy câu sau:

- 1.  Thế nào là phân tích traffic của mạng.
- 2.  Một chút hiểu biết cơ bản về phần cứng trong mạng (hub, switch, router...)
- 3.  Wireshark là phần mềm gì thế?
- 4.  Một số ứng dụng có thể thấy.

Ta sẽ đi trả lời tuần tự từng câu hỏi.

## 1.  Thế nào là phân tích traffic mạng.

- Trước hết traffic nghĩa là gì? Tại sao không gọi là dữ liệu mạng, hay tài nguyên mạng hay gì khác.

- Từ traffic ở đây được hiểu là lưu lượng đi qua, hay nội dung đi qua một chỗ nào đó. Trong mạng đó có thể là card mạng (chỗ nối dây mạng vào trên máy tính), hub, switch, hay router.... Ta nên hiểu mạng giống như một hệ thống đường xá.

- Dữ liệu của chúng ta giống như hàng hóa, được chuyên chở đến đích bằng một hệ thống đường dây kết nối điểm gửi và điểm nhận. Ta có thể hiểu như thế này, muốn chuyên hàng từ làng L1 của tỉnh T1, đến làng L2 của tỉnh T2. Người ta sẽ phải gửi hàng lên tỉnh, rồi từ tỉnh T1 gửi sang tỉnh T2 rồi từ T2 chuyển dần xuống làng L2. Vận chuyển dữ liệu cũng vậy, dữ liệu được đóng vào các gói tin rồi cho di chuyển qua đường mạng. Tuy nhiên, vì mạng quá chẳng chịt nên nhiều khi không biết phải chuyển đi đường nào là đúng nhất nên có những trường hợp phải chuyển đi mọi hướng, rồi đến đâu lại hỏi tiếp, đến khi nào tới nơi thì thôi.

- Đến mỗi điểm mà có giao cắt chẳng hạn, các gói tin này sẽ được người quản lý ở các điểm giao cắt đó chuyển đến điểm kế tiếp.

- Lưu thông trong mạng cũng vậy, không phải bao giờ gói tin cũng đuợc chuyển đến đúng điểm nhận bằng một đường duy nhất. Và rất nhiều trường hợp không thể chuyển đến điểm đích được.

- Khi gặp vấn đề này, người ta thường dùng cách phân tích traffic (các gói tin qua các điểm nối) để thấy gói đó được chuyển như thế nào, và từ đó tại sao lại không tới đích.

- Thêm một điểm cần nói đến ở đây, không phải sau khi ta nhấn nút gửi trên màn hình máy tính, thì máy tính tập hợp tất cả cái đó rồi quăng xuống dây mạng và vận chuyển đi. Giống như hàng hóa, nó cần được bọc lại, để tránh bị tò mò.

- Trong mạng có 7 tầng, không nhớ rõ lắm nữa. Tại mỗi tầng, dữ liệu đều được bọc lại, và lúc đó gói bé cũng như gói lớn bên ngoài giống nhau hết. Được gọi là đơn vị vận chuyển của tầng tương ứng.

**Ví dụ** : Web sẽ sử dụng giấy là HTTP để bọc dữ liệu, xuống phía dưới, Cả cục HTTP này lại được bọc bên trong 1 cục TCP, sau đó cả cục TCP đó được bọc tiếp trong một cục IP, cuối cùng IP mới chuyển cho mấy em đường dây để chuyển qua mạng. Bên nhận cũng có các tầng tương ứng, mỗi tầng sẽ bóc lớp tương ứng của nó thôi rồi chuyển cái có được bên trong lên cho tầng trên.

## 2. Một chút về phần cứng mạng.

### a. Hub

Hay còn gọi là repeater, nó có nhiều cổng, dữ liệu vào bất kì cổng nào sẽ được copy ra nhiều bản, mỗi bản gửi đến một cổng còn lại. Anh Hub này thực sự không có một chút trí nhớ nào, dù có chuyển bao nhiều lần đi nữa thì mỗi gói tin đến anh này đề copy bi ra rồi gửi mỗi bản đến các cổng còn lại. Và nó không thể nhận nhiều gói cùng một lúc trên các cổng khác nhau được.

### b. Switch

Về cơ bản là cũng chuyển gói tin đến từ một cổng vào sang một cổng ra khác. Tuy nhiên, switch "khôn" hơn hub, nó không copy gói đó rồi chuyển đến tất cả các cổng. Nó lưu lại một bảng để biết là cổng nào sẽ nói đến IP nào. Vì vậy lúc chuyển nó sẽ chuyển gói đó đến port tương ứng thôi.

### c. Router

Cái này mới gọi là pro nè. Nó chẳng những làm được việc chuyển gói tin đến các máy cắm vào nó một cách chính xác như switch, nó còn có khả năng chuyển gói tin ra ngoài nếu bản thân nó không biết địa chỉ đến của gói tin ở đâu.

## 3.  Wireshark là phần mềm gì?

- Là phần mềm mã nguồn mở được sử dụng phổ biến trong lĩnh vực phân tích traffic mạng.  
- Về mặt phần cứng, nó hoạt động được là nhờ vào chế độ promiscuous của thiết bị mạng.  
- Khi chạy ở chế độ này, card mạng sẽ cho ta biết tất cả các gói tin qua nó kể các cái gói tin mà đích đến không phải là card mạng đó.

- Có rất nhiều tool cho việc phân tích traffic mạng này. Nhưng nhờ rất nhiều ưu điểm vượt trội mà wireshark thường được sử dụng nhiều nhất. Có thể kể ra ở đây một vài ưu điểm sau:  
-- Số protocol hỗ trợ là quá nhiều, nếu bạn thấy có giao thức nào chưa được hỗ trợ hay gửi mail yêu cầu cho công đồng, nó sẽ có ở phiên bản tiếp theo. Thậm chí, bạn có thể tự viết một protocol rồi gửi yêu cầu được merge vào đến các maintainer. Nó cũng sẽ xuất hiện trong phiên bản tiếp theo nếu được chấp nhận.  
-- Giao diện thân thiện và kinh điển.　Wireshark có giao diện được coi như điển hình của một chương trình phân tích traffic mạng.

- Giá là zero. Vì là phần mềm mã nguồn mở theo giấu GPL. Chúng ta có thể sử dụng chúng với bất kì mục đích nào, kể cả thương mại lẫn không thương mại.

## 4. Một số số ứng dụng có thể thấy của wireshark.  

### Phân tích vấn đề không kết nối của 2 endpoint đã được kết nối vật lý với nhau.  

Vì gói tin được di chuyển qua rất nhiều thiết bị mạng trước khi đến máy đích.  

Ta cần kiểm tra tại mỗi node (có thể là hub, switch, router...) để biết được, gói tin có được truyển theo đường mong muốn không.

Với hub, ta chỉ cần nối một máy tính chạy wireshark vào một cổng bất kì của hub đó. Mọi gói tin vào hub sẽ qua máy tính phân tích của chúng ta.  
Với switch, do các cái gói tin sẽ được truyền đến 1 port để đi tiếp đến đích. Ta sẽ sử dụng 2 phương án:  
sw1 : Ta đặt một hub trên đường từ điểm muốn kiểm tra tới switch.  
sw2 : Ta sử dụng chế độ mirror trong switch để sao chép gói tin sang một port ta mong muốn.

Với router, ta sẽ sử dụng cách 1 của switch.

### Sử dụng stream để theo dõ follow dữ liệu được trao đổi giữa 2 endpoint.  
Với TCP, việc view stream sẽ cho ta thấy tất cả các gói tin được gửi giữa 2 điểm từ lúc socket được tạo đến lúc kết thúc việc bắt gói tin hoặc ngắt kết nối.  
Với UDP, việc view stream sẽ cho ta thấy thứ tự về thời gian các các gói tin UDP.

### Sử dụng Flow Graph  
Chức năng này cho ta thấy một biểu đồ ở dạng tuần tự. Nó mô tả những end point nào, giap tiếp với card mạng hiện tai. Trong một mạng nhỏ không kết nối internet, biểu đồ này cho thấy tổng quan thứ tự kết nối, truyển dữ liệu giữa các máy tính với nhau một cách dễ dàng nhất.

### IO Graph  
Chức năng này cho ta một biểu đồ histogram mô tả lưu lượng gói tin qua card mạng theo thời gian.

### Chức năng conversation  
Cho ta tổng quan về lượng dữ liệu đã trao đổi giữa các endpoint với máy tính ta đang phân tích.  
Bao nhiêu gửi đi, gửi cho những ai, bao nhiều nhận về, nhận từ những ai sẽ hiện ra rất rõ ràng.

Qua phần đầu của dự án hiện tại, mình mới để ý thấy chừng đó chức năng.  
Mình nghĩ rằng mình sẽ viết một phần mở rộng cho wireshark để hỗ trợ phân tích các message ứng dụng mình tự định nghĩa. Chưa có kế hoạch cụ thể.