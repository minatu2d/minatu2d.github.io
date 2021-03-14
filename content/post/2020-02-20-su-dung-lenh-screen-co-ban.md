---
title: "Sử dụng lệnh screen cơ bản"
description: "Sử dụng lệnh screen cơ bản"
date: "2020-02-23T01:20:01+09:00"
thumbnail: ""
toc: true
published: true
categories:
  - basic
  - cmd
tags:
  - cmd
  - screen
---

`screen` là một phần mềm (chạy ở chế độ dòng lệnh) trên Linux rất hữu ích.

## 1. `screen` giải quyết vấn đề gì?
* `Vấn đề`: Khi bạn kết nối bằng `SSH` đến `server` qua 1 terminal, bạn có thể gặp các vấn đề sau:
    * 1. Chương trình bạn định chạy (train 1 con AI, down phim XXX) sẽ rất mất thời gian, bạn không thể đợi đến khi nó xong được. Bạn muốn nó vẫn chạy kể cả khi bạn tắt `terminal` đi về, hôm sau đến vẫn có cái `terminal` cũ.
    * 2. Bạn muốn chạy 2,3,... chương trình một lúc một lúc, chương trình nào cũng bắn lỗi ra `STDOUT`, và bạn phải kiểm tra được tình trạng lỗi của mỗi chương trình một cách riêng biệt để fix bug, điều chỉnh dựa trên `LOG` của nó.
    * 3. Bạn chán việc log lỗi chương trình bằng `redirect` chúng sang `file`.
* `screen` sinh ra để giải quyết các vấn đề ở trên.
* Chỉ với một câu lệnh, bạn có thể giữ lại toàn bộ trạng thái của `terminal`, giúp cho chương trình vẫn chạy bình thường, các `terminal log` vẫn được ghi lại.
* Chỉ với một kết nối `SSH` bạn có thể chạy bao nhiêu chương trình đồng thời tuỳ thích, mỗi chương trình sẽ có `terminal` (tức là `STDIN`,`STDOUT`,`STDERR` ) riêng.
* Chỉ bằng `screen`, bạn không cần mở `Desktop` lên để chỉ để mở một `tab terminal` mới để chạy chương trình của bạn.

## 2. Một số khái niệm khi sử dụng và install
* `session`: Tức là một `terminal ảo` do `screen` quản lý. Trên đó bạn làm được mọi thứ như trên `terminal` thật.
Chỉ khác một điều, khi bạn thoát (bằng lệnh `exit`) thì nó sẽ huỷ `terminal` ảo thôi, và trở về `terminal` thật.

* `windows`: Tức là một ô trên `terminal`, trong đó chưa một `terminal ảo` để bạn thực hiện lệnh.
* Cài đặt trên Ubuntu (với các Platform khác thì mình chịu)
```
sudo apt install screen
```

## 3. Các thao tác sử dụng cơ bản
### 3.1 Tạo một session mới và thoát khỏi
Giả sử ta có kết nối `SSH` trên một `terminal` thật với tên (chính title của cửa sổ ) : `minatu2d@z600:~/screen_samplesample (ssh)`

![screen_cmd_real_terminal](/post/images/screen_cmd_real_terminal.png)

* Ta chạy `screen` command với không tham số (hay là mặc định):
```bash
screen
```
    - Kết quả:
    ![screen_cmd_start_up](/post/images/screen_cmd_start_up.png)

    - Sau khi nhấn `Enter` hoặc `Space` thì ta sẽ thấy như sau:
    ![screen_cmd_in_new_session](/post/images/screen_cmd_in_new_session.png)

    - Đến đây, ta vẫn thấy một `terminal` như ở màn hình đầu tiên, ta vẫn có thể thực hiện mội lệnh bình thường.
    - Bạn có thể để ý `title` của cửa sổ thay đổi như nào trong 3 màn hình ở trên không?
        - Ở 2 màn hình dưới, `title` được đổi thành `screen` hay nói cách khác là `screen` đang được chạy đó.
        - Cụ thể hơn nữa là, `screen` command đang chạy và tạo cho chúng ta một `terminal ảo` hay còn gọi với tên `session` (như đã giải thích ở trên)

* Bạn có thể thêm tham số cho câu lệnh ở trên:
    - Gán tên cho `session`:
    ```bash
    screen -S <Tên của session>
    ```
    - Thay đổi title của windows khi chạy vào session
    ```bash
    screen -t <Tên của của sổ>
    ```

* Để thoát khỏi `session` vừa được tạo và trở về `terminal thật`, ta sử dụng lệnh `exit` trong chính `terminal ảo` :
```
exit
```
    - Khi thoát khỏi thì `session` cũng sẽ bị huỷ và ta sẽ trở về `terminal thật` với `title` được thay đổi theo.

### 3.2 Kiểm tra xem có đang ở trong 1 session hay không
* Như bạn cũng thấy, khi chuyển từ `terminal thật` vào `terminal ảo`, gần như chúng ta không thấy có sự thay đổi nhiều.
Nên nhiều khi khó nhận biết là chúng ta đang chạy trong `terminal thật` hay `terminal ảo`.
    - Ví dụ :khi chúng ta định chạy lệnh `exit` chẳng hạn.

* Có một cách để kiểm tra chúng ta đang ở đâu, đó là kiểm giá trị biến môi trường `$STY` (biến này được `screen` command sử dụng để lưu tên của `session` hiện tại), câu lệnh:

```bash
echo $STY
```
    - Nếu có giá trị (eg: `11822.pts-1.z600`) -> ta đang ở trong `terminal ảo` hay `session` nào đó. 
        - Giá trị trả về sẽ cho ta biết tên của session đó.
    - Nếu không có gía trị -> ta đang ở `terminal thật`.

### 3.3 Nhả (detach) session hiện tại
- Ta muốn `session` vẫn sống, tức là chỉ ra ngoài `terminal thật` tạm thời và sẽ quay lại, ta gọi đó là `deattach`:
    - Tổ hợp phím: `Ctrl + A` + `D`.
- **Ví dụ**: chúng ta phải chạy 1 task để download một file lớn, lấy ví dụ: [Ubuntu ultimate edition 5.0 LTS](https://jaist.dl.sourceforge.net/project/ultimateedition/ultimate-edition-5.0-x64-lite-mate.iso)
    - Bật một `session` với tên `Download` từ `terminal thật`
    ```bash
    screen -S "Download"
    ```

    - Sau khi vào `terminal ảo` rồi, chạy câu lệnh download bằng `wget`
    ```bash
    wget https://jaist.dl.sourceforge.net/project/ultimateedition/ultimate-edition-5.0-x64-lite-mate.iso
    ```
    ![screen_cmd_download_session](/post/images/screen_cmd_download_session.png)

    - Nhả `session` hiện tại bằng `Ctrl + A` + `D`. Và trở lại `terminal thật`
    ![screen_cmd_after_detach](/post/images/screen_cmd_after_detach.png)

    - Giả sử, sau một thời gian muốn quay lại xem tiến độ download đến đâu, ta thực hiện từ `terminal thật`:
    ```bash
    screen -r "Download"
    ```
    - Ta lại quay trở lại `session` mà ta đã `detach`.
    ![screen_cmd_after_reattach](/post/images/screen_cmd_after_reattach.png)

### 3.4 Gắn lại (re-attach) session cũ
* Ở phần trên ta cũng có thực hiện rồi, ở `terminal thật`, ta thực hiện lệnh
```
screen -r <tên hoặc id của session muốn re-attach>
```
    - Và không cần phải gõ đầy đủ `tên` hay `id` của `session`, ta chỉ cần gõ đủ để `screen` phân biệt và tìm ra duy nhất là được.

* Nếu ta không nhớ `session` có `tên` hoặc `id` là gì, ta có thể xem danh sách `session` hiện tại bằng:
    - Tham số `-ls`, như ở dưới đây, ta có 3 `session` đã ở trạng thái `detach`
    ```zsh
    $ screen -ls
    There are screens on:
        17489.Testing AI	(02/23/2020 02:06:04 PM)	(Detached)
        17446.Training AI	(02/23/2020 02:05:49 PM)	(Detached)
        16556.Download	(02/23/2020 01:36:07 PM)	(Detached)
    3 Sockets in /run/screen/S-anhca.
    ```

### 3.5 Dừng session
* **Dừng tự nhiên**: Tức là từ trong 1 `terminal ảo` hay `session`, ta thực hiện tự thoái
    - Bằng lệnh `exit`, đơn giản ta chạy lệnh này trong `terminal ảo` muốn thoát.
    ```bash
    exit
    ```
    - Bằng `shortcut key`: `Ctrl + A` + `K`. (`K` nghĩa là `Kill` đó). Khi thực hiện tổ hợp phím này trong `terminal ảo`, rồi nhấn thêm `y` nữa là ta sẽ thoát khỏi `session` vào trở về `terminal thật`:
    ![screen_cmd_kill_session](/post/images/screen_cmd_kill_session.png)

* **Dừng ép buộc**: Tức là dừng một `terminal ảo` từ bên ngoài, ở đây là từ `terminal thật`.
    - Sau khi có được `tên` hoặc `id` của `session` muốn tắt. Từ bên ngoài `terminal thật`, thực hiện lệnh sau:
    ```bash
    screen -X -S <name hoặc id của session> quit
    ```

### 3.6 Ghi lại log của toàn bộ 1 session
* Việc lấy log khi chạy chương trình là một việc làm phổ biến. Một số trường hợp sử dụng:
    - Lấy làm bằng chứng chạy chương trình 
    - Lấy log để phân tích lỗi ...
* `screen` cung cấp cơ chế cho phép ghi lại toàn bộ log của `terminal ảo` ra file.
    - Hay nói các khác, `screen` lưu lại toàn bộ gồm: lệnh đã gõ, kết quả hiển thị ra của mỗi câu lệnh vào file.
* Mặc định, `screen` không ghi log file đâu, chúng ta sẽ bật nó:
    - Trong 1 `terminal ảo`, ta gõ `Ctrl + A` + `H` (chữ hoa). (`H` ở đây là `History` đấy), thì bắt đầu tại thời điểm đó `log` sẽ được ghi lại.
    ![screen_cmd_start_logging](/post/images/screen_cmd_start_logging.png)
    - Sau khi ta thực hiện một vài lệnh hoặc chạy chương trình, ta sẽ dừng việc ghi log lại, với cùng tổ hợp phím : `Ctrl + A` + `H`.
    ![screen_cmd_close_logging](/post/images/screen_cmd_close_logging.png)
    - Như bạn cũng thấy, tên file là `screen.0` hiện ở góc trái dưới của màn hình.
    Đây là tên file mặc định. Số `0` ở đây là số thứ tự do `screen` tự đánh dấu cho mỗi `session`
* Để xem nội dung của file `screen.0` ở trên, ta sử dụng: `less -r`. 
Vì do có một số kí tự điều kiển được ghi lại trong file log nên nếu xem bằng `vim` hoặc `cat` ta sẽ thấy chúng.
    - Tôi recommend sử dụng `less -r` trong trường hợp này:
    ```
    less -r screen.0
    ```
    ![screen_cmd_view_log](/post/images/screen_cmd_view_log.png)

## 4. Kết và một số phím tắt
* Nói chúng, `screen` là một công cụ rất hữu dụng trong môi trường có nhiều user cùng sử dụng một tài nguyên tính toán thông qua kết nối từ xa: `SSH`, `Telnet`, `Serial`.
* Một số phím tắt:

    | Shortcut key | Chức năng |
    | --------------- | --------------- |
    | `Ctrl + A` + `d` | Detach session hiện tại |
    | `Ctrl + A` + `H` (H : chữ hoa) | Bắt đầu/kết thúc ghi log cho 1 session |
    | `Ctrl + A` + `k` | Thoát (và huỷ) session hiện tại  |
    | `Ctrl + A` + `?` | Xem danh sách phím tắt  |