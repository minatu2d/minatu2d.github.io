---
title: "Cảm nhận về zsh sau 1 tháng sử dụng basic"
description: "Cảm nhận về zsh sau 1 tháng sử dụng basic"
date: "2020-02-23T00:30:01+09:00"
thumbnail: ""
categories:
  - basic
  - cmd
tags:
  - zsh
  - bash
---
Sau khi update chiếc máy `MAC` (máy công ty), terminal mặc định trong máy có recommend sang sử dụng `zsh` thay vì `SHELL` mặc định trước đó (`bash` thì phải) thì mới bắt đầu sử dụng thử `zsh`.
Đọc qua một vài lời khen thấy nó khá hay nên quyết định chuyển sang dùng `zsh` xem sao. Xem có đúng **như lời đồn** không.
Bài này xin tóm tắt một số ý đã tìm hiểu, cảm nhận được.

### 0. Một số tính năng `đáng giá`
* Theo trang [howtogeek.com](https://www.howtogeek.com/362409/what-is-zsh-and-why-should-you-use-it-instead-of-bash/), zsh nổi bật với những tính năng sau:
    * Thự động chuyển thư mục `cd" bằng cách gõ tên thư mục :Hay nói cách khác, giờ khi chuyển thư mục ta có thể bỏ `cd` đi. Cũng hay đấy chứ.
    * Tự động phân giải đường dẫn: Ví dụ “/u/lo/b” có thể được phân giả thành “/usr/local/bin”
    * Tự động sửa lỗi chính và tự hoàn thành nếu viết sai
    * Hỗ trợ `plugin` và `theme`

## 1. Cài đặt dễ dàng
* Cài đặt trên `MAC` bằng `brew install zsh`.
* Cài đặt trên Ubuntu (hoặc tương tự):`sudo apt install zsh`.
* Cài đặt trên các nền tảng khác có thể tham khảo ở [đây](https://github.com/ohmyzsh/ohmyzsh/wiki/Installing-ZSH)

## 2. Có một thứ gọi là `oh my zsh`
* Một cái tên rất `dị`, chi tiết tại [github của oh my zsh](https://github.com/ohmyzsh/ohmyzsh) hoặc tại [https://ohmyz.sh/](https://ohmyz.sh/)
* Là một `framework` được `fan` của `zsh` tạo ra để quản lý các `thiết lập`, `plugin` của `zsh`.
* Có đến hơn `200` plugin và hơn `140` theme.
* Cài đặt `oh my zsh` cũng cực kì đơn giản.
    * Cài bằng `curl`: 
    ```bash
    sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
    ```

    * Cài bằng `wget`
    ```bash
    sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
    ```

## 3. Hỗ trợ `theme`
* Một `theme` điển hình là [powerlevel9k](https://github.com/bhilburn/powerlevel9k), dù có một chút vấn đề về hiển thị `font` khi cài font này. 
    * Ví dụ:
    ![powerline9k_sample](/post/images/zsh_oh_my_zsh_powerline9k.png)

## 4. Có vẻ như chỉ có 1 file cấu hình là `~/.zsh`
* Không giống như `bash`, file cấu hình được chia ra ở nhiều cấp độ như: `/etc/bash.bashrc`, `~/.bashrc`, etc.

## 4. Tương thích hầu hết với `bash`
- Tức là `tập lệnh`, `shortcut key` gần như giống nhau

## 5. Việc gõ có vẻ trở nên nhanh
- Có thể chính là do chức năng tự động hoàn thành, tự tìm kiếm ở của `zsh`.