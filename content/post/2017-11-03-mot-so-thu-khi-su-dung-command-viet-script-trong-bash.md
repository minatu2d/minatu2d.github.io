---
layout: post
title: Một số thứ khi sử dụng command, viết script trong bash
date: 2017-11-03 06:31:11.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
tags:
- Bash
- script
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '11031715743'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2017/11/03/mot-so-thu-khi-su-dung-command-viet-script-trong-bash/"
---
Bài này xin được trích rút một số thứ hữu ích từ bài viết rất hay tên [The art of command line](https://github.com/jlevy/the-art-of-command-line) và mình cũng đã dịch thử (chưa được review) ở  
[Nghệ thuật sử dụng dòng lệnh](https://github.com/minatu2d/the-art-of-command-line/blob/master/README-vi.md)

## 1. Hạn chế gõ lại câu lệnh cho dù là rất ngắn

Hãy sử dụng chức năng tìm đoạn đang gõ trong history để làm giảm công gõ lại  
Hai chức năng tìm xuôi và tìm ngược được bash cung cấp rồi.  
Chỉ việc gán phím tắt là sử dụng được thôi.  
Cách làm được miêu tả trong bài [Phím tắt cho bash](https://lazytrick.wordpress.com/2017/04/02/phim-tat-cho-bash/).  
Đơn giản là thêm đoạn sau vào `~/.bashrc`

```bash  
# Add some shortcut key for quickly search history commands  
# e -> Alt key  
# C -> Ctrl key  
bind '"\ep": history-search-backward'  
# Use Alt + p  
bind '"\en": history-search-forward'  
# Use Alt + n  
```

## 2. Sử dụng **xargs** để biến mọi thứ thành tham số

`args` để biến mọi thứ thành tham số cho lệnh khác.  
Ví dụ muốn thực hiện tạo bản backup (thêm đuôi __backup_ chẳng hạn) cho các file `.txt` trong thư mục chẳng hạn.

```bash  
$ls *.txt|xargs -I hello cp hello hello_backup  
```

Khi muốn tạo một lệnh phức tạp sử dụng tham số được lấy từ `xargs`, hoặc không chắc chắn về câu lệnh thì hãy in toàn bộ đoạn câu lệnh ra bằng cách đặt trong lệnh `echo`  
Ví dụ với đoạn backup ở trên:

```bash  
$ls *.txt|xargs -I hello echo 'cp hello hello_backup'  
```

Ta sẽ có danh sách các câu lệnh định viết được hiển thị ra màn hình.  
Vì thế có thể dễ dàng kiểm tra xem câu lệnh đã viết đúng chưa, hoặc thử thực hiện một câu mẫu xem kết quả như mong muốn không rồi mới thực hiện thực sự.  
Với mình, mình thường viết như này:

```bash  
$ls *.txt|xargs -I hello echo 'cp hello hello_backup && echo "Finished hello"' > tmp_script.sh  
```

Sau đó mình sẽ có scipt tên là `tmp_script` chứa những câu lệnh định chạy.  
Giờ lấy một câu trong đó chạy thử, nếu ok thì chạy script đó là xong.

## 3. Hãy thiết lập header cẩn thận cho bash script

Ta biết rằng, đã là script thì các lệnh được chạy lần lượt đến khi phải dừng thì thôi.  
Nếu viết những script mà thực hiện các thao tác quan trọng như move, xóa, sửa, hay nói chung là có thể làm mất dữ liệu quan trọng mà ta dính phải những lỗi cơ bản liên quan đến tên file, tham số thì sẽ gây thiệt hại và một sự mất công không hề nhỏ.  
Để tránh những lỗi đó trong bài tham khảo ở trên có đưa ra một cấu hình khi thực hiện script.

```bash  
set -euo pipefail  
trap "echo 'Trapped: Script failed: see failed command above'" ERR  
```

Theo kinh nghiệm của mình, **đoạn trên nên được đặt ở phần đầu mẫu khi viết bash script.**  
Giải thích đoạn trên:  
`set -e` : dừng khi bất cứ câu lệnh nào trong script trả về lỗi khác 0 (lỗi thường được quy định là nonzero)  
`set -u` : dừng lại khi gặp một biến được sử mà chưa hề được set trước đó  
`set -o pipefail` : dừng lại khi gặp lỗi liên quan đến pipe  
`trap` : chạy đoạn lệnh khi script gặp lỗi, đoạn trên là dừng và thông báo là có lỗi đó.  
Ngoài ra, có thể thêm `set -x` nếu muốn debug script, tức là mỗi câu lệnh được hiển thị ra màn hình ngay trước khi nó được thực hiện (bao gồm các giá trị biến, tham số truyền cho nó)

## 4. Biết về giới hạn **128K**

Chính là độ dài tối đa của một câu lệnh.  
Có thể bạn sẽ tự hỏi, mấy ai viết nổi câu lệnh dài từng đó.  
Nếu viết bằng tay, thì đúng là như thế.  
Nhưng ở đây nói đến độ dài câu lệnh sau khi được phân giải hết các giá trị mở rộng như `*`, `..`, `?`, etc.  
Chắc chắn bạn quen với câu lệnh dạng:

```bash  
$ls *.html  
```

Độ dài khi ta gõ ở đây là 6 + 1 (kí trự cuối chuỗi) = 7, nhưng độ dài thực sự khi được thực hiện thì không hẳn như thế vì bash (các interpreter cũng thế) sẽ thực hiện thao tác mở rộng để thay thế kí tự `*` ở trên.  
Câu lệnh được chạy sẽ có dạng `ls file1.html file2.html ...`  
Nếu trong thư mục bạn thực hiện lệnh trên có 20K files, mà mỗi file dài chừng 7 kí tự, tức là ngoài đuôi html nó chỉ cần có thêm 3 kí tự nữa thôi thì, đã mất 140K kí tự để đặt chúng cạnh nhau rồi.  
Khi đó sẽ có lỗi `Argument list too long` xảy ra.  
Khi thực hiện với lượng file lớn, tốt nhất không nên dùng những kí tự mở kiểu này khi duyệt file.  
Hãy sử dụng câu lệnh `find` để thay thế vì nó thích hợp hơn.

## 5. Xử lý với các tên file bắt đầu bằng kí tự **-** (minus)

Giả sử khi truyền một file vào câu lệnh, mà tên file lại bắt đầu bằng **-** thì có thể gây lỗi.  
Đó là vì bash bị hiểu kí tự **-** là bắt đầu của options.  
Ví dụ bạn không thể mở file có tên `-abc.txt` bằng câu lệnh sau được:

```bash  
$vi -abc.txt  
```

Nó sẽ hiện lỗi `Unknown option argument: "-abc.txt"`  
Để giải quyết cái này, ta chỉ cần thêm đường dẫn cho tên file là được.  
Bất cứ đường dẫn nào đều ok.  
Ví dụ, câu lệnh dưới đây sẽ ok.

```bash  
$vi ./-abc.txt  
```

Những tên file có kí tự ** -** (trước - là space) cũng vậy. Vì thế, trong trường hợp truyền một biến chứa tên file cho một câu lệnh nên đặt bên trong một cặp nháy kép.