---
title: "Vấn đề chia sẻ Volumne giữa nhiều Docker Container"
description: "Sử dụng lệnh screen cơ bản"
date: "2020-02-23T01:20:01+09:00"
thumbnail: ""
toc: true
published: false
categories:
  - basic
  - docker
tags:
  - container
  - docker
  - volume
  - locking
  - datalost
---

Sự tiện lợi của `Docker` thì không cần phải nói thêm.
Mình là một beginner khi sử dụng `Docker`, gần như làm các dự án `maintain` là chủ yếu.
Vì thế, kinh nghiệm sử dụng `Docker` một cách thuần thục chắc chắn chưa nhiều.
Nhưng gần đây, mình khá bất ngờ khi đọc được 1 bài viết nói về `sự tiện lợi` quá mức của `Docker` làm người ta quyên đi những hạn chế của nó.

Bài này sẽ nói về 1 trong số những hạn chế, mà mình đã phải vật lội với nó.
Đó chính là Docker không có cơ chế `Locking`.
Mình đã viết 1 vài về locking ở đây :[]()

## Docker không hề có cơ chế locking file trong `volume` được mount vào
- Docker sử dụng cơ chế `copy on write`.

## Ứng dụng chạy trong `Container` phải tự làm điều đó.
- 