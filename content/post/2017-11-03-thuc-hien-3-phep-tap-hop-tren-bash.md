---
layout: post
title: Thực hiện 3 phép tập hợp trên bash
date: 2017-11-03 09:02:16.000000000 +09:00
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
  _publicize_job_id: '11035386516'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2017/11/03/thuc-hien-3-phep-tap-hop-tren-bash/"
---

Gần đây do phải làm việc crawling dữ liệu, nên có một chút ít động đến các phép toán tập hợp.  
Bài này sử dụng để note lại các câu lệnh để thực hiện các phép toán phổ biến với tập hợp.  
Nội dung được dịch từ [bài viết](http://www.catonmat.net/blog/set-operations-in-unix-shell/) khá chi tiết của pro [Peter Krumins](http://www.catonmat.net/about/) chủ trang [http://www.catonmat.net/](http://www.catonmat.net/)

## 1. Phép giao

Có rất nhiều cách để tìm giao của 2 tập hơn trong bash.

Cách đầu tiên, đơn giản nhất, dùng `uniq -d` với sự hỗ trợ của `sort`  
Giả sử có 2 tập (tức là file text luôn) `set1`, `set2` (mỗi phần tử nằm trên một dòng).  
Câu lệnh sau sẽ tìm giao của chúng:

```shell
$set set1 set2 | uniq -d  
```

Cách thứ hai, sử dụng câu lệnh `comm`, câu lệnh sử dụng để so sánh 2 files đã được sắp xếp.  
Câu lệnh sẽ như sau:

```shell
$ comm -12 <(sort set1) <(sort set2)  
```

Cách thứ ba, sử dụng câu lệnh `join`, câu lệnh sử dụng để nối 2 hoặc nhiều files (thường được dùng kết hợp với `split`).  
Với câu lệnh này, việc tìm giao hai tập sẽ như sau:

```bash  
$ join <(sort set1) <(sort set2)  
```

## 2. Phép hợp

Đây là phép tập hợp dễ làm nhất.

Cách đầu tiên, sử dụng `cat`.  
Ta cũng giả sử có 2 tập set1, set2 như phần 1.  
Như ta đã biết, ta đã có câu lệnh `cat` để nối lại, rồi in ra một, hai hay nhiều files.

```shell  
$cat set1 set2  
```

Giờ ta phải loại bỏ các phần tử lặp lại và chỉ để lại một từ kết của của lệnh `cat` ở trên.  
Đơn giản nhất là `sort` rồi cho qua `uniq` để lấy dãy duy nhất các phần tử.

```shell 
$cat set1 set2 | sort | uniq  
```

Hoặc khỏi cần dùng `cat` mà `sort` trực tiếp luôn.

```shell 
$sort set1 set2 | uniq  
```

Thậm chí, có thể bỏ qua của `uniq` bằng tham số `-u` của `sort`:\`\`\`shell

```shell 
$sort -u set1 set2  
```

**Nếu 2 tập đã được sắp xếp rồi**, bạn có thể lấy hợp của hai tập nhanh hơn bằng tham số `-m` của `sort`.  
Tham số này nghĩa là `merge`.  
Câu lệnh có thể như sau:

```bash  
$sort -m set1 set2 | uniq  
```

Đương nhiên, có thể bỏ `uniq` nếu thay `-m` bằng `-um` như đã nói ngay trên.

## 3. Phép hiệu

Tức là lấy các phần từ thuộc chỉ một trong 2 tập mà thôi.  
Ta cũng giả sử có 2 tập set1, set2 như phần 1.

Đầu tiên, câu lệnh phù hợp nhất trong trường hợp này là `comm`.  
Câu lệnh sau sẽ in ra các phần tử chỉ thuộc set1 mà không thuộc set2:

```bash  
$ comm -23 <(sort set1) <(sort set2)  
```

Về câu lệnh `comm`, ở phần 1 chúng ta cũng đã có dùng với tham số `-12` rồi.  
Tiện thể giải thích một chút về tham số này:  
**1** : Không in những dòng chỉ thuộc file 1  
**2** : Không in những dòng chỉ thuộc file 2  
**3** : Không in những dòng thuộc cả 2 file  
Trong trường hợp này, sử dụng `-23` sẽ không in ra các dòng thuộc file 2 và cả hai file.