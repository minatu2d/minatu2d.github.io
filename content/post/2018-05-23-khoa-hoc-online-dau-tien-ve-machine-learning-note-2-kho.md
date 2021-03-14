---
layout: post
title: Khóa học online đầu tiên về Machine Learning (note 2) - Khó ~~
date: 2018-05-23 07:55:50.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Machine Learning
tags:
- CourserA
meta:
  _wpcom_is_markdown: '1'
  timeline_notification: '1527062153'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '18159097208'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2018/05/23/khoa-hoc-online-dau-tien-ve-machine-learning-note-2-kho/"
---
Tiếp theo [bài trước](../2016-11-12-asn-1-la-gi-tai-sao-no-quan-trong) trong loạt note về khóa học online đầu tiên về Machine Learning.

## Tuần 4 : Neurtal Networks : Biểu diễn

Như đã biết, khi nói về dữ liệu, ta thường nói đến một loạt đặc trưng.  
Ví dụ: người thì có chiều cao, cân nặng, sở thích, etc.  
Và ta cũng đã biết về tạo cá đặc trưng đa thức khi các đặc trưng bậc 1 không đủ để miêu tả tốt dữ liệu.

Giả sử, ta có một dữ liệu **X** có các đặc trưng **x1, x2, x3, x4, x5, x6**.  
Giờ ta cần thêm các đặc trưng dạng đa thức bậc 2, tức là ta phải thêm các dạng **x1^2, x1x2, x2^2..**.  
Sẽ có khoảng O(N^2) đặc trưng như thế. (Với N là số đặc trưng ban đầu).  
Khi N tăng, con số trên sẽ tăng rất nhanh.  
Khi số bậc tăng nữa, con số kia còn tăng nhanh hơn.  
=> Gần như không thể sử dụng các biểu diễn này vi số chiều quá lớn. (???)

Và **Neural Network** là một cách biểu diễn thay thế khi có một **hyphotheses** với nhiều đặc trưng.  
Lấy ý tưởng từ việc mô phỏng đơn giản cách hoạt động của não bộ trong thần kinh học.

Tiép đến, nói về biểu diễn của **Neural Network**. Ta có thể hiểu là có nhiều đầu vào (mỗi đầu vào là một đặc trưng) được kết nối (các dây - hay chính là hệ số tuơng ứng với mỗi dây nối) đến 1 đầu ra thông qua một hàm (thường là không linear), để tạo ra chỉ một đầu ra duy nhất.  
Hàm được nói ở trên gọi là **activation function**. Thường là hàm **sigmoid** được nhắc đến ở bài trước.

Tuần này cũng nói đến cách biểu diễn thành các lớp.  
Lớp đầu tiên là Input (được đánh số là 1), mỗi giá tri ở lớp này tuơng ứng với một đặc trưng của đầu vào.  
Lớp cuối cùng là Output (gọi là lớp thứ L - giả sử có L lớp) chứa giá trị đầu sau khi qua các lớp ẩn.  
Các lớp ở giữa Input và Output là các lớp ẩn, được đánh số từ 2.  
Mỗi Neuron trong lớp ẩn và lớp Output là kết quả từ hàm **activation** của tổng của tichtất cả các Neuron từ lớp ngay trước đó nhân kết nối tương ứng. Mỗi Neuron có tập kết nối khác nhau. Mỗi kết nối đó chính là một tham số cần phải được điều chỉnh trong quá trình **Training**.  
Ta biết hàm **sigmoid** hay bộ phân lớp sử dụng hàm **sigmoid** dùng để phân loại 2 lớp.  
Khi cần phân loại K lớp, ta cần K nàm **sigmoid** tuơng ứng thì ta cần **K** bộ phân lớp. Mỗi bộ phân lớp đó sẽ phân biệt một đầu vào thuộc lớp đó hay không, và xác suất là bao nhiêu.  
Tuơng ứng ta sẽ có **K** đầu ra ứng với **K** bộ phân lớp, K đầu ra này chính là đầu ra của lớp Output ở trên.

Trong tuần này, cũng nhắc đến cách biểu diễn **one-hot**, khi phân loai nhiều lớp.

## Tuần 5: Neural Network : Quá trình "học"

Hàm **Cost Function**, hay tính khoảng cách giữa đầu ra dự đoán (giá trị tại lớp Output) với đầu ra thực tế. Có sử dụng **Regularization** để tránh Overfitting.  
Và nhớ là giá trị **Cost Function** là một số thực cho mỗi tập dữ liệu đầu vào.

Các tính **Gradient** cũng được làm tuơng tự. Bao gồm cảu **Regularization**. Số chiều của **Gradient** sẽ bằng số chiều tham số.

Hàm **Cost Function** tính được bằng khoảng các giữa đầu ra thật và đầu ra dự đoán chỉ dành cho lớp cuối cùng thôi.  
Nhưng ở Neural Network, có cả những lớp ẩn. Mỗi lớp ẩn có số Neuron khác nhau. Mỗi Neuron chính là một hàm với các kết nối có độ lớn khác nhau nữa.  
Ta phải đi update các hàm số (hay chính là độ lớp các kết nôi) của các hàm (hay các Neuron) đó nữa.  
Backgropagation Alogorithm sẽ thực hiện việc đó.  
Việc giải thích tại sao thuật toán trên lại chạy khá nặng về toán học, mình có đọc qua nhưng không nhớ được mấy. (Thấy nhiều i,j,k lắm, đau đầu thật).  
Đại khái là, nó sẽ tính độ lỗi hay **mức độ phải trả giả cho cho việc dự đoán** tại mỗi Neuron trong mỗi lớp ẩn. Việc tính này dựa vào độ lỗi từ lớp ngay sau.  
Từ giá trị đó sẽ tính tiếp các giá trị cho từng kết nối của Neuron đó (hệ số) tới các giá trị lớp ngay trước.  
Nghĩa là tính ngược từ lớp cuối lên. Tất nhiên là bỏ qua lớp Input.

Tiếp theo, việc tính đạo với hàm số phức tạp không hề dễ, lại còn dễ nhầm. Cần có một bước kiểm tra tính đúng đắn của giá trị đạo hàm. Cái này dựa trên tính chất của vốn có của đạo hàm để thực hiện một bước kiểm tra đạo hàm.

Tiếp theo, đến khởi tạo hệ số. Phải là random. Không thể khởi tạo toàn 0 cho các tham số được. Người ta thường khởi tạo giá trị trong một khoảng [-epsilon, epsilon] nào đó.