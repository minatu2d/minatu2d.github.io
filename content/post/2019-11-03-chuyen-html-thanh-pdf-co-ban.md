---
title: "Chuyển HTML thành PDF cơ bản"
description: "Các chuyển một HTML thành PDF ở mức cơ bản"
date: "2019-11-03T17:41:31+09:00"
thumbnail: ""
categories:
  - basic
  - fixproblem
tags:
  - html2pdf
  - pdf
---

### 1. Tại sao cần chuyển sang `PDF`
- Nhiều lý do để người ta sử dụng PDF thay vì các định dạng khác.
  - Về cơ bản không thể edit nên đảm bảo thống nhất về nội dung hiển thị
  - Ít bị lỗi FONT
  - Tiện lợi khi gửi, nhận: bên nhận và bên gửi luôn thấy chung 1 dữ liệu và ít khi sửa được.
- Vì `PDF` gần như tiêu chuẩn để trao đổi dạng tài liệu `read-only`
- Đọc được ở trên hầu hết mọi nền tảng: máy tính, điện thoại..
  - Trình duyết Web như `Firefox`, `Chrome` còn cho phép đọc luôn mà ko cần cài thêm gì.

### 2. Công cụ hỗ trợ chuyển từ `HTML` sang `PDF`
- Về cơ bản, `HTML` là `plan-text`(các `thẻ` (tag) được kết hợp với nhau), chúng ta sẽ không thể có nội dung nếu không có `Web Browser` để mở nó.
- Vì thế, muốn chuyển : `HTML` sang `PDF`, ta cần 1 `Web Browser` để mở lên, sau đó dùng chính `Web Browser` đó để `PDF` hoá. Trên hầu hết trình duyệt web đều có chức năng này.
- Vậy khi lập trình thì sao? Có thư viện nào hỗ trợ không?
- Câu trả lời là : Có. Đó là [wkhtmltopdf](https://github.com/wkhtmltopdf/wkhtmltopdf)

### 2. Thử nghiệm và một số vấn đề
#### 2.1 Thử xem
- Giả sử convert 1 post [này]({{< ref "2015-08-02-cmake-cong-cu-ho-tro-viec-build-source-tren-nhieu-platform.md" >}})
```bash
$ wkhtmltopdf post/2015-08-02-cmake-cong-cu-ho-tro-viec-build-source-tren-nhieu-platform/index.html ./result.pdf
Loading pages (1/6)
Warning: Failed to load file:///css/style.css (ignore)
Warning: Failed to load file:///js/menu.js (ignore)
Warning: Failed to load file://https-blog-lazytrick-com.disqus.com/embed.js (ignore)
Counting pages (2/6)
Resolving links (4/6)
Loading headers and footers (5/6)
Printing pages (6/6)
Done
```
Và kết quả: [result.pdf](/post/pdfs/2019-11-03-chuyen-html-thanh-pdf-co-ban/result_default.pdf). Một file trắng xoá luôn.

- Như ta cũng thấy ở trên, có một số `warning` khi ta chạy câu lệnh.
- Sau khi tìm hiểu, thì biết được rằng:
  - `wkhtmltopdf` không thể load được các file kể trên
  - Các resource được chỉ định bằng đường dẫn tuyệt đối thì vẫn load được.
  - Các resource có đường dẫn tương đối thì load lỗi.

#### 2.2 Chỉnh sửa một chút
- Copy các thư mục: `css`,`js` vào thư mục chạy lệnh `wkhtmltopdf` để nó có thể phù hợp với đường dẫn đang bị lỗi.
- Copy cả file html vào thư mục chạy lệnh luôn -> phù hợp với đường dẫn tương đối
- Ta điều chỉnh một chút trong source HTML để các file bị lỗi ở trên thành tương đối:
  - `/css/style.css` -> `./css/style.css`
  - `/js/menu.js` -> `./js/menu.js`
- Chạy lại câu lệnh lần nữa:
```shell
$wkhtmltopdf index.html result_1.pdf
Loading pages (1/6)
Warning: Failed to load file://https-blog-lazytrick-com.disqus.com/embed.js (ignore)
Counting pages (2/6)
Resolving links (4/6)
Loading headers and footers (5/6)
Printing pages (6/6)
Done
```
Và kết qủa là: [result_1.pdf](/post/pdfs/2019-11-03-chuyen-html-thanh-pdf-co-ban/result_1.pdf)
Tốt hơn rất nhiều rồi.
Tuy một số `style` chưa đúng với web page.

### 3. Tích hợp trong ứng dụng khác:
- Thư viện `wkhtmltopdf` chủ yếu bằng `C` nên chắc chắn sẽ dễ dàng được binding sang các ngôn ngữ khác.
- Một số ngôn ngữ đã hỗ trợ:
  - PHP : https://github.com/KnpLabs/snappy
  - Ruby: https://rubygems.org/gems/wkhtmltopdf-binary-edge
  - Python: https://pypi.org/project/pdfkit/