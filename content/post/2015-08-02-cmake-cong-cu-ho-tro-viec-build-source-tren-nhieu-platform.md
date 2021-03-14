---
layout: post
title: CMake - Công cụ hỗ trợ việc build source trên nhiều platform.
date: 2015-08-02 20:38:27.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Compile
tags:
- Cmake
meta:
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _thumbnail_id: '73'
  _publicize_job_id: '13341161610'
  _wpcom_is_markdown: '1'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2015/08/02/cmake-cong-cu-ho-tro-viec-build-source-tren-nhieu-platform/"
---
- Đuợc làm việc cùng những pro đầy kinh nghiệm thực sự rất thú vị, ngoài việc chia sẻ hiểu biết của mình, còn đuợc hóng nhiều kĩ năng sử dụng tool của các pro nữa. Mình thấy rằng, không có tool nào gọi là thần thành, mọi tool chỉ giúp tăng tốc độc với những người mới và không đủ thời gian để hiểu sâu. Còn với những người đã nhiều kinh nghiệm, họ sử dụng tool theo các điểm mạnh của chúng và kết hợp rất nhiều tool một cách nhịp nhàng sao cho công việc tiến hành nhanh nhất.

- Qua dự án vừa rồi, dù đã nghe qua về `CMake`. Nhưng cũng giống vô vàn các tool khác, ta nhiều khi không biết cái nào sẽ đến, và cái nào sẽ cần trong dự án tiếp theo. Thôi thì, duyên gặp em nào thì cũng để cho em ấy chút kỉ niệm trên blog này.

1. Về Build Cross-Platform
2. CMake giải quyết vấn đề gì.
3. Các tính năng mạnh của CMake

Trước hết, ta cứ làm một số mục chính để lần sau viết tiếp còn biết đang ở chỗ nào.

## 1. Nói một chút về build-cross platform

- Ta biết rằng, trên các hệ thống *NIX, các phần mềm hầu hết là mã nguồn mở được nhiều người phát triển. Và hầu hết trong chúng sử dụng một cách build source đơn giản, và rất mạnh. Đó là `Make` hay các `Makefiles`. Cùng với việc giảm thiểu đến mức tối đa việc tạo ra một kịch bản build cho các hệ thông nàng, các tool đi kèm cũng được sinh ra: `autotools`, `autobuild`,...etc.  
- Nhưng với sự phổ biến của `Linux`, càng ngày khiến người không còn chỉ biết có Window nữa, mà song hành với nó còn có nhiều hệ điều hành khác. Việc phát triển phần mềm mà có thể chạy trên càng nhiều hệ điều hành(Platform) sẽ là lý tưởng.  
- Ta sẽ nói đến việc làm thế nào để porting một chương trình từ `Linux` sang `Windows`.  
- Nếu nói hết ra ta thấy sẽ rất nhiều việc phải làm, nào là porting thư viện sử dụng, porting source đã viết, rồi quản lý source ra sao khi phải thay đổi source...  
- Ở đây, tôi muốn nói đến việc quản lý source và tạo ra kịch bản build cho 2 nền tảng Linux và Windows một cách bài bản, thống nhất, dễ hiểu, đáng tin cậy là một yêu cầu gần như bắt buộc.

## 2. CMake giải quyết vấn đề gì.

- Trên `Linux`, hay các hệ *NIX, `programmer` thường thích dùng Command Line hơn vì nó quen với họ, vì họ cảm thấy nó dễ dùng cho công việc của họ.  
- Trên `Windows` thì không như thế, họ được cung cấp một IDE (Visual Studio..) để phát triển ứng dụng.  
- Giả sử ta muốn porting một ứng dụng từ `Linux` sang `Window`, ta không thể yêu cầu họ phải sử dụng dòng lệnh, phải viết Make file được. Điều đó khiến cho giá phát triển sẽ bị đẩy lên cao do phải đào tạo programmer trên Window biết sử dụng command line, rồi không tận dụng được kinh nghiệm của họ với IDE, rồi việc phát triển song song 2 phiên bản sẽ gặp rất nhiều vấn đề...  
- Giống như rất nhiều công cụ khác giúp tăng hiệu quả và độ tin cậy trong việc develop ứng dụng cross-platform. Điểm mạnh lớn nhất của `CMake` là trên cùng 1 source mà `CMake` quản lý, nó có khả năng tạo ra kịch bản build bằng Makefile cho Linux, tạo ra Project file cho Visual Studio...  
- Source code được giữa gần như trong suốt giữa 2 môi trường này.  
- Ta sẽ tìm hiểu ở phần sau kĩ hơn xem `CMake` nó làm thế nào, và có điều gì khác thường ở đây.  
- Có một điểm cần nói ở đây, trước khi có `CMake` thì có nhiều công cụ khác cũng hộ trợ việc này nhưng đều dựa trên ngôn ngữ Make hay tạo ra Makefile như là kết quả cuối cùng. CMake tự định nghĩa riêng một ngôn ngữ của nó giúp việc định nghĩa các thành phần của ứng dụng nó quản lý một cách dễ dàng nhất.

## 3. Các tính năng mạnh của CMake

Những tính năng được gọi là mạnh của `CMake`, có thể kể đến 2 tính năng sau:  
- Tạo ra kịch bản build cho ra hiệu suất làm việc cao trên các hệ thống nó support.  
- Tao ra kịch kịch bản build mà tách riêng với source code, (tính năng này có từ Autotools), điều hành đặc biết hữu ích khi ta muốn thực hiện nhiều kịch bản build trên cùng 1 source code.  
(Ta thấy trên `Window`, các project thường chứa luôn source code trong đó, nhưng `CMake` tạo ra project file mà các file source ở một chỗ khác)
 
Ta có thể nói rằng, `CMake` là một bộ sinh Makefile.  
Ví dụ sử dụng `CMake` mình sẽ để trong một post khác để tránh làm post này quá dài.