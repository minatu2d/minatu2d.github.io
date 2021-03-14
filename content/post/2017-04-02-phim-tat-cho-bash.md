---
layout: post
title: Phím tắt cho bash
date: 2017-04-02 11:16:36.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- CodeSkill
- Khác
tags:
- Bash
- Linux
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '3545952822'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2017/04/02/phim-tat-cho-bash/"
---
Mặc định, bash sử dụng `emacs` mode, có thể chuyển sang _vi_ mode được.  
Nếu sử dụng ở chế độ mặc định, thì dưới đây là một số shortcut hữu ích khi sử dụng.

### Chiều ngang : Di chuyển cơ bản

Ctrl + b : (Backward) Di chuyển con trỏ sang trái về trước 1 kí tự  
Ctrl + f : (Fordware) Di chuyển con trỏ sang phải một kí tự.  
Ctrl + d : (Delete) Xóa kí tự ở vị trí con trỏ

### Chiều ngang : Di chuyển nhanh

Ctrl + a : (Ahead) Di chuyển con trỏ về command  
Ctrl + e : (End) Di chuyển con trỏ về cuối command  
Alt + f : (Fordward) Di chuyển con trỏ sang từ tiếp theo  
Alt + b : (Backward) Di chuyển con trỏ về từ phía trước

(một từ được định nghĩa gồm aphabets, digits, không chứa kí hiệu)

### Cut và Paste

Ctrl + k : (Kill) Cắt lấy đoạn từ vị trí con trỏ đến cuối command  
Alt + d : (Delete) Xóa trừ ở vị trí con trỏ  
Ctrl + y : (Yank) Dán vào bị trí hiện tại của con trỏ  
Ctrl + r : (Retrieve) Tìm command trong history theo các kí tự được gõ sau đó

### Một số khác

Trên đây là các chức năng gần như mặc định, có thể sử dụng được ngay.  
Còn một số chức năng khá hữu ích nữa, ta có thể gắn shortcut cho nó bằng cách sau:

```bash  
# Add some shortcut key for quickly search history commands  
# e -> Alt key  
# C -> Ctrl key  
bind '"\ep": history-search-backward'  
bind '"\en": history-search-forward'  
```

Tìm kiếm command bắt đầu bằng một từ bất kì trang history  
2 phím này cực kì hữu dụng để giảm số phim mũi tên  
phải bấm khi muốn chạy một command dài đã gõ trước đó khá lâu.  
Khi đó, ta chỉ cần gõ từ khóa bắt đầu rồi Alt + p để chọn command  
Mong muốn