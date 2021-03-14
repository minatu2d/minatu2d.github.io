---
layout: post
title: Khoá học online đầu tiên về Machine Learning ( note 1) - Khó ~~
date: 2018-05-23 02:18:56.000000000 +09:00
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
  _rest_api_published: '1'
  timeline_notification: '1527041939'
  _rest_api_client_id: "-1"
  _publicize_job_id: '18150752615'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2018/05/23/khoa-hoc-online-dau-tien-ve-machine-learning-note-1-kho/"
---
Cuối cùng cũng kết thúc khoá học Online đầu tiên về Machine Learnning.  
Do tiếng Anh cùi, nên dù nhìn sub liên tục nhưng vẫn vô số lần tua lại để xem.  
Khoá học rất hay, thầy Andrew NG giảng khá kĩ các khái niệm quan trọng.  
Dù mình biết Machine Learning nặng về Toán, đặc biệt Toán Tối ưu như machinelearningcoban.com viết, nhưng khoá này không tập trung vào những cái đó.  
Nó tập trung vào việc Intuition - Trực giác về Machine Learning.  
Ví dụ như giải quyết vấn đề gì. Quá trình Train là đi tìm cái gì. Quá trình Test làm gì. Rồi cách đánh giá một mô hình Machine Learning.

Với người mới như mềnh, có vẻ khoá này hợp lý.

Thôi, tạm note nội dung tóm tắt từng tuần ra đây cho đỡ quên. Và cũng đỡ phải nhớ.

## Tuần 1 : Giới thiêu

Lướt sóng qua một chút về Machine Learning.  
- Supervised Learning là gì.  
- Unsupervised Learning là gì.

Giới thiệu, mô hình ( mới đầu mình nghe thì khó tưởng tượng từ này, đại khái là cách giải trong toán) đầu tiên là **Linear Regression With One Variable**.

Bắt đầu từ bài toán tìm _f(x)_ thoả mãn : _y = f(x)_, biết rằng _x=x1_ thì _y=y1_, _x=x2_ thì _y=y2_, etc... . Biết thêm rằng nữa f(x) có dạng : _ax + b_.  
Mình nhớ không nhầm với bài toán trên, cách giải là coi a,b là biến, thay các giá trị x1, x2, y1, y2.. đã biết vào rồi giải. Cộng trừ nhân chia gì đó chắc sẽ ra thôi.  
Khi tìm được a,b rồi thì cho x=x3 thì hỏi y bằng bao nhiêu chẳng hạn thì quá đơn giản rồi. **Luôn luôn bằng, luôn luôn chính xác*.

Nhưng trong Machine Learning thì hơi khác một chút. Nó cũng giống việc giải phương trình là đi tìm a,b nhưng không phải để đúng với mọi điều kiện dạng x=x1 thì y phải bằng y1, x=x2 thì b phải bằng y2.

Giả sử sau khi tìm được a,b rồi, với _x=x1_ nó tìm ra _y1_p=ax1 + b_, với _x=x2_ nó tìm ra  
y2_p. Các giá trị này chỉ y1_p, y2_p chỉ cần gần gần y1, y2 đã cho là được. Vì sao lại thế, vì nhiệm vụ của nó là đoán một quy luật sát với tất cả các điểm có dạng **(x1, y1), (x2, y2) thôi, chứ không phải đoán chính xác tất cả. Sau đó với quy luật đoán được, nó thực hiện đoán tốt với các điểm x3, x4 chưa biết.

Cái đoán để tìm ra quy luật hay chính là hàm số đó người ta gọi là **fit**.  
Cái hàm số chưa biết đó được gọi với cái tên **Hyphothesis**, với mình đây là từ mới toanh.  
Quá trình tìm ra quy luật từ dữ liệu ban đầu gọi là **train**  
Quá trình kiểm tra quy luật với những dữ liệu chưa có lúc **train**, được gọi là **test**.

Tiếp theo, nó giới thiệu về **Cost Function**, tức là hàm **trả giá có sự sai khác khi đoán**.  
Dùng luôn ví dụ ở trên, cái giá phải trả chính là sự sai khác giữa giá trị đoán và giá trị mong muốn. Giữa y1 và y1_p, y2 và y2_p etc.  
Người ta hay dùng khoảng cách Eclide ( hay gọi là Norm-2) để tính **cái giá ** này.  
Mục đích trong quá trình train là làm cho **cái giá phải trả** là nhỏ nhất.  
Hay tối thiểu hoá hàm **Cost Function**.

Tiếp nữa, đến Gradient Descent, là một phương pháp để tối thiểu hoá **Cost Function** nó ở trên.  
Hãy tưởng tượng đến Parabol ở cổng Giải Phóng của BKHN. Nó bị ngược, nếu nó quay lại thành hình chữ U chẳng hạn, thì trên đồ thị ta dễ dàng thấy điểm nhỏ nhất của nó. Chính là điểm nằm ở **đít** chữ U hay Parabol đó.  
Giả sử hàm **trả giá** cũng có dạng tương tự như thế ( cũng bậc 2 mà), ta phải làm thế nào? Vẽ lên thì cũng thấy đấy. Nhưng người ta có cách khác dựa vào hiểu biết về đạo hàm. Hồi cấp 3, tôi đã học thuộc là đạo hàm chính là hệ số góc của đường tiếp tuyê đến hàm số tại chính điểm đó.  
Hoá ra, chính từ cái hệ số góc đó người ta sẽ biết được là đồ thị tại điểm đó là cong hay thẳng hay xiên. Thế là họ có phương pháp cứ đi theo từng điểm của đồ thị rồi di chuyển dần theo chiều ngược với tiếp tuyến thì thể nào cũng về điểm thấp nhất, à nhầm, điểm cực trị mới đúng.

Trong Gradient Descent, việc thực hiện update hệ số phải đồng thời.  
Tức là sau khi tính được Gradient (dựa trên đạo hàm) tại một điểm hoặc tập điểm (từ đầu vào), ta phải thực hiện việc update hệ số đồng thời.

Ở trên chỉ là đường tiếp tuyến cho không gian 2 chiều. Khi nhiều chiều, người ta sẽ có mặt tiếp tuyến rồi siêu phẳng tiếp tuyến. Với mình, từ 3 chiều không tưởng tượng được tốt rồi. :((

Phần cuối của tuần, là ôn lại ( thực ra với mình nó như mới rồi vì thì Toán xong môn nào mình thường xoá hết dữ liệu môn đó bằng một trận AOE ) về đại số, ma trận.  
Mình ôn để biết mấy khái niệm về Inverse, Transpose, Matrix vậy thôi. Học như lần đầu có khác, cái gì cũng thú vị VKL :))

## Tuần 2 : Linear Regression With Multiple Variables

Nói về Linear Regression with Multiple Variables với các khái niệm tương ứng với One Variable ở tuần 1.  
Đừng tưởng tượng đến đồ thị làm gì, khó VKL.

Tiếp theo, nhắc đến 2 khái niệm quan trọng là **feature scaling** và **mean normalization**  
**Feature Scaling** là chia tất cả giá trị đầu vào cho một giá trị đủ lớn ( thường là hiệu giữa max và min )  
**Mean Normalization** là trừ tất cả giá trị đầu vào cho giá trị trung bình của chúng.

Tiếp đó, có nhắc đến các khái niệm **Cost Function** và **Gradient

Tiếp theo nói đến, Polymonial Regression, tức là lấy mũ giá trị đầu vào.  
Thay vì chỉ dùng _ax + b_, người ta có thể dùng _ax^100 + b_.

Rồi nó đến các giải bài toán tìm hệ số thuần túy ( tức là sao cho tất cả các dấu bằng giữa y1=y1_p, y2=y2_p) bằng các sử dụng ma trận. Với cách này, chú ý có cả trường hợp vô nghiệm và vô số nghiệm.

Tuần này, cũng có bài tập đầu tiên phải làm và nộp.  
Tất cả bài tập trong khoá sẽ phải viết bằng ngôn ngữ Matlab (Matrix Laborary).  
Matlab mất phí nên dùng Octave.  
Viêc nộp bài cũng trên Octave luôn, khá tiện.

## 3. Tuần 3 : Logistic Regression

Trong Machine Learning có 2 lớp bài toán người ta hay gặp.  
Đó là Classification và Regression.  
**Regression** giống như ví dụ ở trên, đoán giá trị là một tập số liên tục. Thường là thuộc tập số thực.  
**Classification** hay còn gọi là phân loại, đoán thuộc loại nào.

Tuy nhiên, vì một vài lý do lịch sử mà thuật toán Logistic Regression dù là thuật toán dùng cho bài toán phân loại (chính xác hơn là phân loại nhị phân - tức chỉ có 2 loại), nhưng lại có cái tên rất Regression.

Hàm **sigmoid** được giới thiệu.  
Để tính xác xuất thuộc vào một lớp (đang xét bài toán chỉ có 2 lớp thôi).  
Nó thực hiện hàm **sigmoid** trên tích giữa vector hệ số và đầu vào.  
Giống nhữa Linear Regression:  
Zi = Xi * Theta

Áp dụng hàm **signmoid** lên tích này:

g(Zi) = 1 / (1 + e^-z)

Sau đó, khái niệm **Decision Boudary** (biên quyết định) cũng được giới thiệu.  
Tức là nhìn trên đồ thị đơn giản nó sẽ đường thằng (trong nhiều chiều hơn là mặt phẳng, siêu phẳng) để phân chia 2 vùng tuơng ứng với 2 lớp

Hàm **Cost Function** của Logistic Regression cũng được đưa ra.  
Sau đó là cách tính **Gradient**

Kể từ đây ta có thể hiểu rằng, đi đâu làm gì thì đều cần nhớ đến 2 thứ

**Cost Function** và cách tính **Gradient**

Trong tuần này, cũng giới thiệu một khái niệm mới nữa, đó là **Regularization**.  
Mình nghĩ, để giải thích rõ ràng về tại sao lại có khái niệm này thì hơi khó.  
Trong khóa học có giới thiệu về **Overfitting**, rồi **Generalize** (tính tổng quan), rồi **Underfetting**.