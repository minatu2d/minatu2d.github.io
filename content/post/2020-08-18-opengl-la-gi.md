---
title: "OpenGL là gì?"
description: "Giới thiệu chung về OpenGL"
date: "2020-08-14T10:09:00+09:00"
thumbnail: ""
toc: true
published: true
categories:
  - basic
  - opengl
  - computer_graphic
tags:
  - opengl
  - glfw
  - glu
  - 3D
---

## 0. Tại sao đi tìm hiểu OpenGL làm cái gì?
- Gần đây, phải quay sang làm 1 dự án có liên quan đến Computer Graphic, cụ thể là trong không gian 3 chiều (3D).
- Thực sự, đối với mình nó lại hoàn toàn mới.
Ngoài chút kiến thức toán như: Vector, Ma trận (Matrix), vector đơn vị, vài phép cơ bản của Vector thì mình không còn tí kiến thức Đại số nào nữa.
- Vì thế, cũng mất khá nhiều thời gian để đọc lại các kiến thức đại số liên quan đến các phép biến đổi cơ bản : Tịnh tiến (translate), phép co giãn (scale), phép quay (rotation).
- Tuy không hiểu rõ, nhưng việc đọc lại, học lại cũng giúp ích rất nhiều trong việc debug.
Tuy nhiên, khi động đến phần "Lighting" (tương tác với nguồn sáng) trong không gian 3D thì đọc lý thuyết thôi là không đủ.
- Vì thế, để hiểu kĩ hơn mình đã tìm hiểu về một vài Graphics Toolkit để chạy thử.
- Nổi tiếng nhất đó là : `OpenGL`. (Viết tắt từ: Open Graphic Library).
- Bài này sẽ nói về những thứ cơ bản khi tiếp cận với `OpenGL`.

## 1. OpenGL, GLUT, GLFW, GLEW, GLAD
- `OpenGL`: https://www.khronos.org/opengl/wiki/Main_Page
  -  Là thư viện chuyên biệt cho render (tính toán hiển thị) và các tính toán khác liên quan đến đồ hoạ máy tính. Nó không phụ thuộc platform (Eng: platform-independent).
  - Ví dụ: Khi ta chiếu 1 hình hộp xuống một mặt phẳng thì `OpenGL` sẽ cung cấp các hàm để ta thực hiện các việc đó.
  - Các công việc khác như : Khởi tạo, hiển thị không phải là `platform-independent` nữa, mỗi nền tảng sẽ có cách khác nhau.
  - `OpenGL` chỉ làm nhiệm vụ tính toán những gì cần thiết để hiển thị thôi.
  - Được viết bằng C/C++.
  - Có binding trong nhiều ngôn ngữ khác, vì thế ở nhiều ngôn ngữ khác cũng có thể sử dụng được.

- `GLUT`: https://www.opengl.org/resources/libraries/glut/
  - Viết tắt của **The OpenGL Utility Toolkit**.
  - Ra đời rất sớm để giải quyết vấn đề khi người sử dụng `OpenGL` phải tự implement các phần liên quan cho mỗi platform khác nhau.
  - Toolkit này giúp việc viết code đỡ bị phụ thuộc vào platform hơn. Hay nó sẽ thực hiện các phần do khác biệt nền tảng đó và cung cấp giao diện đồng nhất cho người phát triển.
  - Tuy nhiên, theo thông tin trên [trang chủ](https://www.opengl.org/resources/libraries/glut/) thì `GLUT` đã lâu rồi không còn được `maintain` nữa.
  - Nó cũng gợi ý chuyển qua [`FreeGLUT`](http://freeglut.sourceforge.net/) rồi. (Hình như `GLUT` thay đổi về `License` thì phải)
- `GLFW`: https://www.glfw.org/
  - Viết tắt của gì thì không biết.
  - Là thư viện cung cấp các chức năng thiết yếu liên quan đến cửa sổ, context, xử lý các loại input như bàn phím chuột liên quan đến `OpenGL`.
- `GLEW` : http://glew.sourceforge.net/
  - Viết tắt là gì thì không biết luôn.
  - Là thư viện cho phép xác định các phần mở rộng của OpenGL và load nó lên.
- `GLAD` : https://github.com/Dav1dde/glad
  - Một loader cho `OpenGL`.

## Tại sao OpenGL cần Loader?
- Như ta đã biết thư viện `OpenGL` được viết bằng C/C++.
- Thông thường việc sử dụng thư viện, ta chỉ cần `các file header` và `binary chứa code đã được biên dịch`.
- Tuy nhiên, với `OpenGL` thì nó hơi khác một chút.
- 