---
layout: post
title: 4 Stage khi biên dịch HelloWorld.c
date: 2017-01-01 22:00:04.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
- Compile
tags:
- GCC
- HelloWorld
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '301962049'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2017/01/01/4-stage-khi-bien-dich-helloworld-c/"
---
Gần đây, phải giải quyết giúp một vài vấn đề liên quan đến Cross-Compile. Có tìm hiểu kĩ một chú về Compiler, Linker, và Loader.  
Bài này xin nói về cơ bản về quá trình biên dịch một file source code (.c) sang dạng chạy được.  
Ví dụ: từ HelloWorld.c thành HelloWorld và chạy được như ví dụ dưới đây.

HelloWorld.c

```cpp  
#include  
int main()  
{  
print("Hello World \n");  
return 0;  
}  
```

Kết quả chạy:

```bash  
$./HelloWorld  
HelloWorld  
```

## 1. 4 Stage khi compiling

Để chuyển từ 1 file source code **HelloWorld.c** ở trên sang dạng chạy được **HelloWorld** bằng GCC sẽ có 4 stages sau.  
Kết quả của Stage trước là sẽ là đầu vào của Stage sau.

![compiling_4_stage](/post/images/compiling_4_stage.png)

Khi chúng ta gõ

```bash  
$gcc -o HelloWorld HelloWorld.c  
```

Thì chỉ kết quả của Stage sẽ được giữ lại, kết quả của các Stage trung gian không được lưu lại.

## 2. Chi tiết mỗi Stage

### 2.1 Stage 1 : Preprocessing (Tiền xử lý)

Ở Stage này, các tiền xử lý (bắt đầu băng dấu #) được xử lý. Ví dụ các chỉ thị dịch #ifdef, #if, #include sẽ được thay thế hết.  
Do vậy, các lỗi về sai đường dẫn file header sẽ được phát hiện. Kết quả các đoạn chị thị dịch #ifdef, #elif cũng sẽ được đưa ra.

Để hiển thị kết quả đầu ra của Stage này, ta có thể sử dụng command sau:

Khi sử dụng **gcc**:

```bash  
$gcc -E HelloWorld.c > HelloWorld.i  
```

Hoặc sử dụng **cpp** (C Preprocessor)

```bash  
$cpp HelloWorld.c > HelloWorld.i  
```

### 2.2 Stage 2 : Biên dịch sang mã Assembly

Ở Stage này, kết quả từ Stage 1 sẽ được biên dịch thành source code trong ngôn ngữ Assembly.  
Để hiển thị đầu ra của Stage này, ta sử dụng command sau:

```bash  
$gcc -S HelloWorld.c  
```

Ta sẽ được file **HelloWorld.s** trong thư mục hiện tại.

### 2.3 Stage 3 : Biên dịch sang mã máy

Ở Stage này, GCC sẽ biên dịch kết của từ Stage 2 (Assembly Code) sang dạng mã máy.  
Tức là GCC sẽ phải biết môi trường mà nó sẽ compile cho là gì. (Môi trường chạy hay Host)

Để lấy kết quả đầu ra của Stage này, ta sử dụng command sau:

```bash  
$gcc -c HelloWorld.c  
```

Ta sẽ được file **HelloWorld.o** nằm ở thư mục hiện tại.

### 2.4 Stage 4 : Link để tạo file chạy được

Ta đã có kết quả từ Stage là các file **.o**.  
Từ mỗi file **.c** sẽ tạo ra 1 file **.o**.

Đó chỉ là các file rời rạc, hay các file object hay module.

Để có 1 chương trình chạy được, ta cần liên kết hay là **Link** các file object lại.

Quá trình link về cơ bản, sẽ lấy nội dung (là hàm hoặc biến) rồi xếp lại theo một thứ tự để máy tính có thể hiểu và chạy được.

Có 2 loại link : Link tĩnh (các file **.o** và **.a**), và Link động (các file **.so**).

Đàu ra Link tĩnh là một file chứa toàn bộ mã máy kể cả các hàm thư viện chuẩn, chỉ cần load lên bộ nhớ là có thể chạy được. Vì thế kích thước thường rất lớn.

Ta có thể sử dụng command sau:

```bash  
$gcc -static -o HelloWorld HelloWorld.c  
```

Đầu ra của Link động là một file không chứa toàn bộ mã của các thư viện phụ thuộc. Nó có thể yêu cầu kernel load khi thực sự được chạy hoặc được gọi. Vì thế, kích thước file thường nhỏ.

GCC sẽ mặc định sử dụng Link động nên ta có thể sử dụng command sau:

```bash  
$gcc -o HelloWorld HelloWorld.c  
```