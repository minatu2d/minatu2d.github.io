---
layout: post
title: AWK - Sử dụng cơ bản
date: 2016-09-07 09:25:06.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
tags:
- Command
- Linux
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '26572803644'
  _thumbnail_id: '1257'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/09/07/awk-su-dung-co-ban/"
---
**AWK** : 1 trong 3 tool (cùng với **grep** và **sed**) mạnh dùng xử lý chuỗi, xuất hiện ban đầu ở Unix, và được mặc định có trong bất cứ bản phân phối Linux nào.

Sau một hồi tìm hiểu **awk** trên [TutorialPoint](http://www.tutorialspoint.com/awk/index.htm).

**awk** là một một tool để xử lý một chuỗi, đầu vào có thể là file, là output của một câu lệnh khác. Đơn vị xử lý là dòng, tức là nó đọc vào từng dòng text từ dữ liệu đầu vào rồi thực hiện các xử lý tương ứng. AWK cung cấp hẳn 1 ngôn ngữ để so khớp input cũng như xuất output. Ngôn ngữ này cực kì giống ngôn ngữ C, nên rất dễ làm quen. Nó cung cấp 1 cơ chế matching mạnh là regular express nữa cho xử lý input.

**Khi làm việc với AWK, ta hãy hiểu rằng, từng line trong input sẽ được thực hiện qua đoạn script mà ta cung cấp cho Awk.**

  
AWK sẽ làm việc theo flow sau:

![awk_workflow](/post/images/awk_workflow.png)

```bash  
$awk 'BEGIN {} {} END {}'  
```

Có 3 khối : **BEGIN**, Body (không có từ khóa), **END**.

Khối **BEGIN**: được thực hiện chỉ duy nhất 1 lần, trước khi bắt đầu đọc dữ liệu.  
Khối **END**: được thực hiện chỉ duy nhất 1 lần, sau khi thực hiện xong phần xử lý chính.  
Khối Body: không có từ khóa bắt đầu, được thực hiện với mỗi line dữ liệu đầu vào.

Một số ví dụ đơn giản.

## 1.Viết lại pgrep

[pgrep](http://linux.die.net/man/1/pgrep) là tìm process ID theo keyword được nhập vào. Lệnh này được sử dụng trong rất nhiều trường hợp như tìm process ứng dụng, kết hợp [kill](http://linux.die.net/man/1/kill) để tắt 1 loạt ứng dụng, etc.

Giả sử với **_ps uax_**, ta có kết quả sau:

```bash  
root 3853 0.0 0.0 21024 480 ? Ss 18:52 0:00 /usr/sbin/vmware-authdlauncher  
root 4028 0.0 0.0 241144 5928 ? Ss 18:52 0:00 /usr/sbin/nmbd -D  
root 4073 0.0 0.1 338924 15976 ? Ss 18:52 0:00 /usr/sbin/smbd -D  
root 4077 0.0 0.0 330820 5792 ? S 18:52 0:00 /usr/sbin/smbd -D  
root 4079 0.0 0.0 338932 7484 ? S 18:52 0:00 /usr/sbin/smbd -D

```

Ta sẽ thực hiện lấy PID của các process có chưa từ khóa **_daemon_**:

```bash  
$ps aux|grep daemon|awk {print $2}  
```