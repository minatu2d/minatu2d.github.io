---
title: "Vài chú ý khi setup mail server"
description: "Vài điều đã giải quyết khi setup mail server"
date: "2020-01-01T01:01:01+09:00"
thumbnail: ""
categories:
  - basic
  - fixproblem
tags:
  - mailserver
  - pdf
---
`Mail Server` là một trong những ứng dụng phổ biến bậc nhất trong hầu hết các hệ thống thông tin.
Theo mình, ngay cả khi các ứng dụng tin nhắn nhóm như Slack, Chatwork, FB Messenger đã thì việc sử dụng mail vẫn rất được ưa chuộng do những tính chất đặc trưng của nớ: như formal hơn, lưu trữ chắc chắn hơn...

Lần đầu tiên mình được trực tiếp sờ vào việc cài đặt của một mail server. Thực ra ở vị trí mới của mình thì có lẽ không nên đụng vào mấy thứ đó thì sẽ hiệu quả hơn. Nhưng biết làm sao được, mình thích đi `giải quyết, tìm hiểu vẫn đề kĩ thuật` nên mò vào tìm hiểu.

Nói vậy đủ rồi, bắt đầu vào.

### 1. Các khái niệm và giao thức của hệ thống Mail
#### 1.1 Sự khác nhau giữa `WebMail` và `ClientMail`
**Đầu tiên nhất**, đó là sự khác nhau giữa sử dụng `WebMail` và `MailClient` truyền thống (tôi gọi là truyền thống vì ra đường trước `WebMai` rất nhiều).
`WebMail` nghĩa là gửi/nhận mail ngay trên trình duyệt web. Cái này giờ phổ biến nhất, gần như áp đảo.

`MailClient` nghĩa là, thay vì sử dụng trình duyệt, thì có một ứng dụng chuyên dụng để thực hiện việc gửi/nhận mail.

Nhiều người sẽ hỏi: Tại sao trên `web` cũng gửi/nhận được thì cần cái ứng dụng `MailClient` để làm cái gì.
Đó là vấn đề lịch sử: Khi `web` chưa phổ biến như bây giờ, lúc đó đơn giản web là mở ra vài trang tin thì `web` và `mail` là các dịch vụ khá tương đương thôi.

Còn giờ người ta có thể làm mọi thứ trên `web` thì bỗng nhiên `web` trở thành một cái nền tảng cho các dịch vụ khác chứ không đơn thuần là một dịch vụ tương đương như `email` nữa.
![WebMail and ClientMail](/post/images/2020-01-01-vai-chu-y-khi-setup-mailserver-tren-ubuntu/WebMai_And_MailClient.png)


#### 1.2 Các khái niệm và hoạt động  cơ bản
**_Về khái niệm_**:

Ta sẽ nói về một số khái niệm khi sử dụng `MailClient` (trường hợp `WebMail` hầu như không khác mấy).

- `Mail` : Là nội dung (bao gồm cả file đính kèm) được trao đổi giữa 2 người bất kì thông qua địa chỉ của họ.

- `Mail Server`: Là một cái máy tính có nhiệm vụ chứa `mail` gửi/nhận của các user mà nó quản lý.

- `SMTP` : Là giao thức (các thức giao tiếo) giữa 2 `Mail Server`.

- `POP3` hoặc `POP3S`: Là giao thức để tải mail xuống, bao gồm toàn bộ nội dung của `Mail`. Chữ `S` ở cuối là chỉ `Secured` tức là giao thức `POP3` trên đường truyền được mã hoá. 
Chứ bản thân `POP3` thì không được mã hoá.

- `IMAP` hoặc `IMAPS`: Là giao thức để tải `header` của `Mail` xuống. Việc này làm tăng tính hiệu quả của việc Check mail vì người dùng có thể xem `Header` rồi mới quyết định có thực sự tải cả `Mail` đó xuống hay không.

- `Mail Header`: Phần header của `Mail` bao gồm những thông tin như : Tiêu đề, danh sách gửi nhận, danh sách CC, danh sách BCC.

- `CC` : Đồng gửi đến. Nghĩa là mọi người ở trong danh sách này đều nhận được `Mail` và đều biết người khác cũng danh sách trong CC đó cũng nhận được `Mail`.

- `BCC`: Đồng gửi đến, nhưng mỗi người không biết có ai khác nhận được `Mail` hay không.

- `Mail Relay Server`: Chưa rõ lắm

**_Gửi mail_**

- **Step 1**: Sau khi User soạn mail và nhân nút "Send" trên `MailClient`, `MailClient` sẽ gửi `Mail` đó đến `Mail Server` của chính mình (chứ không phải của người nhận) bằng giao thức `SMTP`.

- **Step 2**: Sau khi `Mail Server` của người gửi nhận được yêu cầu gửi `Mail` tư user, nó sẽ sử dụng tiếp giao thức `SMTP` để gửi đến `Mail Server` của người nhận. Vậy là kết thúc việc gửi.

**_Về hoạt _**

- **Step 3**: Sau khi `Mail Server` nhận được mail đến thông quan giao thức `SMTP`, nó sẽ lưu lại ở đâu đó.

- **Step 4**: Phía user nhận cũng sẽ sử dụng một `MailClient`, thông qua giao thức `IMAP(S)` hoặc `POP3(S)` để lấy mail mới của chính nó về. 
Đến đây, ta cũng kết thúc việc nhận `Mail`

### 2.Các phần mềm `Server <=> Server` phổ biến (SMTP runner)
- Là phần mềm chạy trên `Server` để làm 3 nhiệm vu: Nhận email gửi từ chính User nó quản lý, nhận email được gửi từ `Mail Server` khác gửi đến cho User của nó, chuyển tiếp mail gửi từ User của nó đến `Mail Server` khác.

#### 2.1 Postfix
- Một trong những phần mềm "già" nhất, phổ biến nhất.

#### 2.2 Zimbra
#### 2.3 Sendmail

### 3. Các phần mềm `Server <=> Client` phổ biến (IMAP, POP3 runner)
- 

### 4. Thêm record vào `DNS Records`
### 4. Cài đặt cụ thể trên Ubuntu 18.04
#### 4.1 Cài đặt phần mềm `Server <=> Server`: Postfix
##### 4.1.1 Cài đặt
##### 4.1.2 Cấu hình
##### 4.4.3 Kiểm tra hoạt động
#### 4.2 Cài đặt phần mềm `Server <=> Client`: Dovecot
##### 4.2.1 Cài đặt
##### 4.2.1 Cấu hình
##### 4.2.3 Kiểm tra hoạt động
#### 4.3 Gửi và nhận email