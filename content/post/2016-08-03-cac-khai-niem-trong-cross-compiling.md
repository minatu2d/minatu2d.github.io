---
layout: post
title: Phân biệt Build vs Host vs Target
date: 2016-08-03 01:24:36.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
- Compile
- Embedded
tags:
- CrossCompile
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '25414361866'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/08/03/cac-khai-niem-trong-cross-compiling/"
---
Khái niệm Cross-compiling là rất phổ biến khi phát triển các hệ thống nhúng.  
Với người mới, hiểu rõ khái niệm là rất quan trọng.

Bài này sẽ cố gắng phân biệt 3 khái niệm về môi trường. Đó là **Host Enviroment**, **Build Enviroment**, và **Target Enviroment**. Có thể dịch nôm na là **Môi trường chủ**, **Môi trường biên dịch**, và **Môi trường chạy đích**.  
Vì có thể dẫn đến hiểu nhầm hoặc không rõ nghĩa nên chúng ta nên sử dụng trực tiếp thì hơn.

Trước khi bắt đầu với các ví dụ, ta nên xem qua một số nguồn diễn giải 3 khái niệm này.

## 1. Diễn giải từ tài liệu của Autotools.

Trong link diễn giải về **Canonical System** (Hệ thống quy chuẩn) sử dụng trong [Autotools](https://autotools.io/index.html), có đoạn sau:

> When using autoconf, there are three system definitions (or machine definitions) that are used to identify the “actors” in the build process; each definition relates to a similarly-named variable which will be illustrated in detail later. These three definitions are:
> 
> host (CHOST)
> 
> The system that is going to run the software once it is built, which is the main actor. Once the software has been built, it will execute on this particular system.
> 
> build (CBUILD)
> 
> The system where the build process is being executed. For most uses this would be the same as the host system, but in case of cross-compilation the two obviously differ.
> 
> target (CTARGET)
> 
> The system against which the software being built will run on. This actor only exists, or rather has a meaning, when the software being built may interact specifically with a system that differs from the one it's being executed on (our host). This is the case for compilers, debuggers, profilers and analyzers and other tools in general.

Dịch nôm na là :

Khi sử dụng autoconf, có 3 định nghĩa hệ thống (hoặc có hiểu là định nghĩa máy) được sử dụng để chỉ rõ các "actor" (tác nhân) trong quá trình build, mỗi định nghĩa sẽ có liên quan đến những giá trị biến có tên tương tự được trình ở phần sau. 3 định nghĩa này là:

host (**CHOST**)  
Đây là hệ thống mà phần mềm mà ta đang muốn build sẽ chạy trên đó. Khi phần mềm được build rồi, thì nó chỉ chạy trên hệ thống như thế này mà thôi.

build (**CBUILD**)  
Đây là hệ thống mà quá trình build sẽ chạy trên đó. Hầu hết sẽ cùng hệ thống với host, nhưng trong trường hợp cross-compilation (biên dịch chéo) 2 hệ thống này lại khác nhau.

target (**CTARGET**)  
Cũng là hệ thống mà phần mềm đang muốn build sẽ chạy trên đó. Nhưng tác nhân này chỉ tồn tại, đúng hơn là có ý nghĩa khi phần mềm đang được build có thể tương tác với một hệ thống khác cái hệ thống nó đang chạy trên (host). Đây là trường hợp khi build **compilers** (trình biên dịch), **debuggers** (trình gỡ rối), **profilers** (trình đánh giá) and **analyzers** (trình phân tích) hoặc các tool khác tương tự.

## 2. Diễn giải từ GNU GCC

Trong tài liệu về miêu tả các khái niệm sử dụng trong [GNU GCC](https://gcc.gnu.org/onlinedocs/gccint/index.html#Top) có một đoạn giản thích về các [Configure Term](https://gcc.gnu.org/onlinedocs/gccint/Configure-Terms.html#Configure-Terms) (các khái niệm khi cấu hình), có đoạn sau:

> There are three system names that the build knows about: the machine you are building on (build), the machine that you are building for (host), and the machine that GCC will produce code for (target). When you configure GCC, you specify these with --build=, --host=, and --target=.
> 
> Specifying the host without specifying the build should be avoided, as configure may (and once did) assume that the host you specify is also the build, which may not be true.
> 
> If build, host, and target are all the same, this is called a native. If build and host are the same but target is different, this is called a cross. If build, host, and target are all different this is called a canadian (for obscure reasons dealing with Canada's political party and the background of the person working on the build at that time). If host and target are the same, but build is different, you are using a cross-compiler to build a native for a different system. Some people call this a host-x-host, crossed native, or cross-built native. If build and target are the same, but host is different, you are using a cross compiler to build a cross compiler that produces code for the machine you're building on. This is rare, so there is no common way of describing it. There is a proposal to call this a crossback.

Dịch nôm na như sau:

Có 3 tên hệ thống mà quá trình build phải biết: máy mà bạn đang chạy build trên đó (build), máy mà bạn đang build phần mềm cho nó (host), và cái máy mà GCC sẽ tạo mã cho nó (target). Khi cấu hình GCC, bạn sẽ chỉ ra 3 tên thống này bằng các tham số **--build=, --host=, and --target=**.

Tránh chỉ định hệ thống host mà không chỉ định rõ hệ thống build, khi đó quá trình configure có thể giả định rằng hệ host cũng là hệ build, và có thể không đúng với bạn mong muốn.

Nếu hệ thống **build**, **host**, và **target** giống nhau, thì cái này gọi là **native**. Nếu hệ **build** và **host** giống nhau nhưng hệ thống **target** lại khác, thì cái này gọi là **cross**.

Nếu hệ thống **build**, **host**, và **target** hoàn toàn khác nhau thì cái này gọi là canadian (có 1 lý do khá khó hiểu liên quan đến một Đảng chính trị ở Canada và những background của những người làm việc ở thời gian đó).

Nếu hệ thống **host** và **target** are giống nhau, nhưng hệ thống **build** thì khác, thì tức là bạn đang sử dụng một **cross-compiler** (trình biên dịch chéo) để tạo **native code** cho một hệ thống khác. Một số người gọi cái này là **host-x-host**, **crossed native**, hoặc **cross-built native**.

Nếu hệ thống build và **target** giống nhau, nhưng hệ thống **host** thì khác, thì có nghĩa là bạn đang sử dụng **cross compiler** (trình biên dịch chéo) để build một trình **cross compiler** (trình biên dịch chéo khác) cái mà sẽ sinh code cho hệ thống mà bạn đang build trên đó. Trường hợp này rất hiếm, cho nên không được miêu tả ở đây. Nhiều khi nó được gọi là crossback.

OK, nếu đến đây mà bạn đã phân biệt được **Build-Host-Target** rồi thì không cần xem các ví dụ nữa.

1.  Build GDB để debug trên RaspberryPI