---
layout: post
title: CMake - Một ví dụ đơn giản
date: 2015-08-16 04:12:53.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Compile
tags:
- Cmake
- Make
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '13772923733'
  _thumbnail_id: '73'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2015/08/16/cmake-mot-vi-du-don-gian/"
---
Trong bài tôi đã giới thiệu qua về CMake. Như ta đã biết nó cung cấp tính tăng giúp việc sinh ra Makefile một cách hiệu quả. Nhất là đối với các dự án phức tạp. Nó cũng cung cấp thêm các bộ sinh khác để sinh cấu trúc quản lý source cho các IDE khác nhau như Visual Studio, KDE.  
Trong giới hạn, tôi sẽ nói về việc sử dụng CMake để build một simple project trên cả Windows và Linux.

Ta vẫn xoay quanh ví dụ kinh điển là tạo chương trình Hello World, nhưng yêu cầu hiển thị câu đó trên ít nhất 3 ngôn ngữ. ja,en,vi. Chương trình sẽ viết bằng ngôn ngữ C.

([Link tham khảo](http://derekmolloy.ie/hello-world-introductions-to-cmake/))  
Ví dụ được thực hiện trong thư mục _/home/oedev/Code/CMake_

Nội dung chính gồm các phần sau,  
1. Chương trình Hello World cho En.  
2. Thêm ngôn ngữ Ja, Vi vào chương trình trên.  
3. Thêm ngôn ngữ Ja, Vi trong một tình huống khác.

## 1.Chương trình Hello World cho En

Ta sẽ có file source như sau:  `mainapp.c`

```c
#include<stdio.h>

//  
// Greeting message in En  
//  
void greeting_en()  
{  
  printf("Hello world n");  
}

//  
// Main functions  
//  
int main(int argc, char* argv[])  
{  
  greeting_en();

  return 0;  
}  
```

File sử dụng cho CMake là 1 file CMakeLists.txt

```bash 
#  
# Điều kiện về version tối thiểu để đọc được file CMakeList.txt này.  
#  
cmake_minimum_required(VERSION 2.8.12)

#  
# Tên project thường sẽ là tên file chạy  
#  
project (helloworld)

#  
# Định nghĩa mối liên quan giữa file chạy và file nguồn  
#  
add_executable(hellworld mainapp.c)  
```

Đặt cả 2 file trên vào cùng 1 thư mục:  
Nội dung được kiểm tra bằng lệnh `tree `như sau

```bash  
CMake  
├── CMakeLists.txt  
└── mainapp.c  
```

File `CMakeList.txt` chính là file định nghĩa source sử dụng ngôn ngữ mà `CMake` hiểu được.  
Đến đây, ta thực hiện công việc chính của `CMake`. Đó là sinh ra `Makefiles` cho project đơn giản này. Ta phải chuyển vào bên trong thư mục `CMake` trước khi chạy lệnh dưới đây:

```bash  
$ cmake .  
```

Dấu `.` phía sau rất quan trọng, hay có thể thay nó bằng `$(PWD)` trong trường hợp này.  
Tham số thứ 2 là thư mục chưa file `CMakeFile.txt`
Nội dung chạy xong nên như thế này:

```bash
oedev@OECrossDev:~/Code/CMake$ cmake .  
-- The C compiler identification is GNU 4.8.2  
-- The CXX compiler identification is GNU 4.8.2  
-- Check for working C compiler: /usr/bin/cc  
-- Check for working C compiler: /usr/bin/cc -- works  
-- Detecting C compiler ABI info  
-- Detecting C compiler ABI info - done  
-- Check for working CXX compiler: /usr/bin/c++  
-- Check for working CXX compiler: /usr/bin/c++ -- works  
-- Detecting CXX compiler ABI info  
-- Detecting CXX compiler ABI info - done  
-- Configuring done  
-- Generating done  
-- Build files have been written to: /home/oedev/Code/CMake  
```

Nội dung thư mục sau khi chạy lệnh trên:

```bash 
oedev@OECrossDev:~/Code/CMake$ tree -L 2  
.  
├── CMakeCache.txt  
├── CMakeFiles  
│   ├── 2.8.12.2  
│   ├── cmake.check_cache  
│   ├── CMakeDirectoryInformation.cmake  
│   ├── CMakeOutput.log  
│   ├── CMakeTmp  
│   ├── hellworld.dir  
│   ├── Makefile2  
│   ├── Makefile.cmake  
│   ├── progress.marks  
│   └── TargetDirectories.txt  
├── cmake_install.cmake  
├── CMakeLists.txt  
├── mainapp.c  
└── Makefile  
```

Như ta thấy, có rất nhiều file và thư mục được tạo ra khi ta chạy `CMake`. Nhưng trong giới hạn bài này ta chỉ quan tâm đến `Makefile` thôi.  
Ta cũng thấy rằng chương trình chương được build, hay nói cách khác `CMake` chỉ làm nhiệm vụ của nó đến đây thôi. Tức là nhiệm vụ của một công cụ sinh `Makefiles`.

Giờ muốn build project ta vừa tạo lúc đầu (chỉ có 1 file .c), ta sẽ chạy Makefiles thôi.

```bash 
oedev@OECrossDev:~/Code/CMake$ make  
Scanning dependencies of target hellworld  
[100%] Building C object CMakeFiles/hellworld.dir/mainapp.c.o  
Linking C executable hellworld  
[100%] Built target hellworld  
```

Kết quả thu được như sau

```bash
oedev@OECrossDev:~/Code/CMake$ ls  
CMakeCache.txt CMakeFiles cmake_install.cmake CMakeLists.txt hellworld mainapp.c Makefile  
```

Ta đã thây file _helloword_ được sinh ra, đây chính là file chạy:

```bash 
oedev@OECrossDev:~/Code/CMake$ ./hellworld  
Hello world  
```

Giờ ta sẽ thêm lời chào bằng ngôn ngữ Ja, Vi vào chương trình trên.

## 2.Thêm ngôn ngữ Ja, Vi vào chương trình trên.

Ta sẽ tổ chức lại source như sau:

```bash  
oedev@OECrossDev:~/Code/CMake$ tree  
.  
├── CMakeLists.txt  
├── include  
│   ├── greetings_en.h  
│   ├── greetings_ja.h  
│   └── greetings_vi.h  
└── src  
├── greetings_en.c  
├── greetings_ja.c  
├── greetings_vi.c  
└── mainapp.c  
```

Do cấu trúc thư mục đã thay đổi, vị trí các file nguồn cũng thế.  
Ta cần thay đổi file `CMakeList.txt` để phù hợp với thay đổi này.  
Nội dung file này sẽ thành như sau:

```bash
#  
# Điều kiện về version  
#  
cmake_minimum_required(VERSION 2.8.12)

#  
# Tên project  
#  
project (helloworld)

#  
# Khai bảo thư mục chứa file header (.h)  
#  
include_directories(include)

#  
# Thêm từ file nguồn bằng lệnh *set*  
#  
set(SOURCES src/mainapp.c src/greetings_en.c src/greetings_ja.c src/greetings_vi.c)

#  
# Thêm một tập các file bằng một bộ lọc trong thư mục chưa source  
# Đây là cách nhanh và phổ biến hơn.  
#  
#file(GLOB SOURCES &quot;src/*.c&quot;)

#  
# Định nghĩa sự liên quan giữa file chạy và các file nguồn.  
#  
add_executable(hellworld ${SOURCES})  
```

Ta sẽ chạy `cmake` để sinh Makefiles và build chương trình với `Makefiles` xem kết quả thế nào:  
Với các sử dụng hàm `set`:  `Chạy cmake`

```bash 
oedev@OECrossDev:~/Code/CMake$ cmake .  
-- The C compiler identification is GNU 4.8.2  
-- The CXX compiler identification is GNU 4.8.2  
-- Check for working C compiler: /usr/bin/cc  
-- Check for working C compiler: /usr/bin/cc -- works  
-- Detecting C compiler ABI info  
-- Detecting C compiler ABI info - done  
-- Check for working CXX compiler: /usr/bin/c++  
-- Check for working CXX compiler: /usr/bin/c++ -- works  
-- Detecting CXX compiler ABI info  
-- Detecting CXX compiler ABI info - done  
-- Configuring done  
-- Generating done  
-- Build files have been written to: /home/oedev/Code/CMake  
```

_Chạy Make_

```bash 
oedev@OECrossDev:~/Code/CMake$ make  
Scanning dependencies of target hellworld  
[ 25%] Building C object CMakeFiles/hellworld.dir/src/mainapp.c.o  
[ 50%] Building C object CMakeFiles/hellworld.dir/src/greetings_en.c.o  
[ 75%] Building C object CMakeFiles/hellworld.dir/src/greetings_ja.c.o  
[100%] Building C object CMakeFiles/hellworld.dir/src/greetings_vi.c.o  
Linking C executable hellworld  
[100%] Built target hellworld  
```

_Kiểm tra thư mục_

```bash  
oedev@OECrossDev:~/Code/CMake$ tree -L 2  
.  
├── CMakeCache.txt  
├── CMakeFiles  
│   ├── 2.8.12.2  
│   ├── cmake.check_cache  
│   ├── CMakeDirectoryInformation.cmake  
│   ├── CMakeOutput.log  
│   ├── CMakeTmp  
│   ├── hellworld.dir  
│   ├── Makefile2  
│   ├── Makefile.cmake  
│   ├── progress.marks  
│   └── TargetDirectories.txt  
├── cmake_install.cmake  
├── CMakeLists.txt  
├── hellworld  
├── include  
│   ├── greetings_en.h  
│   ├── greetings_ja.h  
│   └── greetings_vi.h  
├── Makefile  
└── src  
├── greetings_en.c  
├── greetings_ja.c  
├── greetings_vi.c  
└── mainapp.c  
```

_Kết quả chạy chương trình_

```bash
oedev@OECrossDev:~/Code/CMake$ ./hellworld  
Hello world  
Konichiwa!!!  
Xin chao !!!!  
```

Khi thay _set_ command bằng command file (GLOB) thì cho kết quả gần như giống hệt nhau.

### 2.1 Tạo thư mục để build riêng

À, có một điểm cần nói ở đây, vừa ta có thấy rất nhiều file (ngoài Makefile) được sinh ra trong quá trình cmake chạy.  
Nếu để các file đó chung với thư mục các file source, header ta vừa tạo sẽ gây ra rắc rối, khó quản lý.  
Ta sẽ tống các file đó vào 1 thư mục, gọi là thư mục _build_.  
Cấu trúc thư mục sẽ thay đổi như sau:

```bash  
oedev@OECrossDev:~/Code/CMake$ tree  
.  
├── build  
├── CMakeLists.txt  
├── include  
│   ├── greetings_en.h  
│   ├── greetings_ja.h  
│   └── greetings_vi.h  
└── src  
├── greetings_en.c  
├── greetings_ja.c  
├── greetings_vi.c  
└── mainapp.c  
```

Giờ ta sẽ thực hiện toàn bộ quá trình _cmake_ và _make_ bên trong thư mục _build_

```bash 
oedev@OECrossDev:~/Code/CMake/build$ cmake ..  
-- The C compiler identification is GNU 4.8.2  
-- The CXX compiler identification is GNU 4.8.2  
-- Check for working C compiler: /usr/bin/cc  
-- Check for working C compiler: /usr/bin/cc -- works  
-- Detecting C compiler ABI info  
-- Detecting C compiler ABI info - done  
-- Check for working CXX compiler: /usr/bin/c++  
-- Check for working CXX compiler: /usr/bin/c++ -- works  
-- Detecting CXX compiler ABI info  
-- Detecting CXX compiler ABI info - done  
-- Configuring done  
-- Generating done  
-- Build files have been written to: /home/oedev/Code/CMake/build  
```

Ta sử dụng _.._ thay vì _._ như ở các câu lệnh trước, vì ta đã chuyển vào thư mục _./build_ nên file _CMakeList.txt_ ở thư mục cha của thư mục này. Ta phải nói cho cmake biết điều đó.  
Tiếp tục chạy lệnh _make_, ta sẽ có nội dung thư mục _CMake_ sẽ như sau:

```bash 
oedev@OECrossDev:~/Code/CMake$ tree -L 3  
.  
├── build  
│   ├── CMakeCache.txt  
│   ├── CMakeFiles  
│   │   ├── 2.8.12.2  
│   │   ├── cmake.check_cache  
│   │   ├── CMakeDirectoryInformation.cmake  
│   │   ├── CMakeOutput.log  
│   │   ├── CMakeTmp  
│   │   ├── hellworld.dir  
│   │   ├── Makefile2  
│   │   ├── Makefile.cmake  
│   │   ├── progress.marks  
│   │   └── TargetDirectories.txt  
│   ├── cmake_install.cmake  
│   ├── hellworld  
│   └── Makefile  
├── CMakeLists.txt  
├── include  
│   ├── greetings_en.h  
│   ├── greetings_ja.h  
│   └── greetings_vi.h  
└── src  
├── greetings_en.c  
├── greetings_ja.c  
├── greetings_vi.c  
└── mainapp.c  
```

Như ta thấy, toàn bộ nội dung của quá trình chạy _cmake_ và _make_ được giữ trong thư mục _./build_, phần đinh nghĩa source code bên ngoài không bị ảnh hưởng gì hết.  
Để chạy chương trình, ta phải vào thư mục _./build_  
Kết quả vẫn là :

```bash  
oedev@OECrossDev:~/Code/CMake/build$ ./hellworld  
Hello world  
Konichiwa!!!  
Xin chao !!!!  
```

## 3. Thêm ngôn ngữ Spain(es) trong một tình huống khác

Tình huống khác ở đây là gì? Tôi muốn nói đến tình huống như sau:  
Giả sử ta có một thư viện chứa lời chào cho 3 ngôn ngữ En, Ja, Vi.  
Ta cần thêm ngôn ngữ Es để có một chương trình với loài chào của 4 ngôn ngữ En, Ja, Vi, Es.

Trước hết, ta nên hiểu thư viện như thế nào? Trong C/C++ để sử dụng lại được mã nguồn hoặc để bảo vệ mã nguồn. Người ta thường build những mã nguồn muốn sử dụng lại hoặc đưa cho người khác thành các thư viện.  
Về mặt file ta thấy được, thư viện sẽ có 2 loại: thư viện tĩnh(.a) và thư viện động(.so).  
Trong các file đó chứa source đã được đưa về dang nào đó của các hàm mà ta đã viết trong file .c của source.

Nếu thư viện chỉ có thế thôi thì chưa đủ, ta cần phải biết thư viện đó cung cấp gì cho bên ngoài có thể sử dụng được. Có thể là biến toàn cục, có thể là hàm.  
Các hàm và biến này phải đặt đâu đó trong các file header. Không phải luôn luôn do người viết thư viện cung câp nha. Rồi khi build với source mà ta viết, trình biên dịch sẽ biết cách compile hoặc link cho thích hợp.  
File header thường được cung cấp kèm theo các file .a(.so). Nếu không có, thì phải có tài liệu mô tả.  
Có cả tài liệu và file header là điều lý tưởng nhất.

Giả định rằng, ta có thư viện chưa xử lý hiển thị 3 loài chào En, Ja, Vi. Nhưng ta chỉ có file header mô tả prototype của các hàm mà không thấy được source của nó.

### 3.1 Build thư viện tĩnh

Kết quả của đoạn này sẽ là file thư viện _greetings_en_ja_vi.a_ và 3 file header đi kèm.  
Sau đó ta có thể xóa 3 file source(.c) đi được.  
Về cấu trúc file, do thư viện không chứa hàm _main()_ được, ta phải bỏ file _mainapp.c_ đi.  
Nhớ nhé, thư viện thì chỉ cần file header thôi là dùng được rồi.  
Thư viện mẫu mực sẽ không cần đến xem code, ta chỉ cần xem mô tả API hoặc file header là có thể dùng được rồi.  
Cấu trúc file mới sẽ như thế này:

```bash  
oedev@OECrossDev:~/Code/CMake$ tree -L 2  
.  
├── CMakeLists.txt  
├── include  
│   ├── greetings_en.h  
│   ├── greetings_ja.h  
│   └── greetings_vi.h  
└── src  
├── greetings_en.c  
├── greetings_ja.c  
└── greetings_vi.c  
```

Giờ, ta phải sửa lại file `CMakeList.txt`, nó sẽ thành như sau:  

```bash  
#  
# Điều kiện phiên bản nhỏ nhất của CMake  
#  
cmake_minimum_required(VERSION 2.8.12)

#  
# Tên project  
#  
project (multilang_greetings)

#  
# Thêm thư mục chứa các file header (.h)  
#  
include_directories(include)

#  
# Thêm một tập các file source  
#  
file(GLOB SOURCES &quot;src/*.c&quot;)

#  
# Thư viện bao gồm những source nào.  
# Loại thư viện là STATIC (tĩnh)  
# Tên thư viện có thể khác tên *project*  
#  
add_library(greetings_enjavi STATIC ${SOURCES})  
```

Chạy tiếp _cmake_ và _make_ trong thư mục _build_:  
Ta sẽ được kết quả như sau:  
Chạy _Cmake_, kết quả gần như không thay đổi so với trước:

```bash 
oedev@OECrossDev:~/Code/CMake/build$ cmake ..  
-- The C compiler identification is GNU 4.8.2  
-- The CXX compiler identification is GNU 4.8.2  
-- Check for working C compiler: /usr/bin/cc  
-- Check for working C compiler: /usr/bin/cc -- works  
-- Detecting C compiler ABI info  
-- Detecting C compiler ABI info - done  
-- Check for working CXX compiler: /usr/bin/c++  
-- Check for working CXX compiler: /usr/bin/c++ -- works  
-- Detecting CXX compiler ABI info  
-- Detecting CXX compiler ABI info - done  
-- Configuring done  
-- Generating done  
-- Build files have been written to: /home/oedev/Code/CMake/buil  
```

Chạy _make_:

```bash 
oedev@OECrossDev:~/Code/CMake/build$ make  
Scanning dependencies of target greetings_enjavi  
[ 33%] Building C object CMakeFiles/greetings_enjavi.dir/src/greetings_vi.c.o  
[ 66%] Building C object CMakeFiles/greetings_enjavi.dir/src/greetings_ja.c.o  
[100%] Building C object CMakeFiles/greetings_enjavi.dir/src/greetings_en.c.o  
Linking C static library libgreetings_enjavi.a  
[100%] Built target greetings_enjavi  
```

Ta thấy rằng tên thư viện sinh ra là _libgreetings_enjavi.a_, như vậy tiền tố _lib_ được thêm vào tên thư viện ta đã khai báo với _add_library_ trong file CMakeList.txt.  
Ta có thể kiểm tra xem thư viện này chứa cái gì, bằng câu lệnh _ar_

```bash  
oedev@OECrossDev:~/Code/CMake/build$ ar -t libgreetings_enjavi.a  
greetings_vi.c.o  
greetings_ja.c.o  
greetings_en.c.o  
```

Ta sẽ thấy không có file chạy nào được sinh ra trong thư mục build. Đúng như mong muốn, ta cần build thư viện.  
Ta cũng kiểm tra kích thước file này:

```bash 
oedev@OECrossDev:~/Code/CMake/build$ ls -lh libgreetings_enjavi.a  
-rw-rw-r-- 1 oedev oedev *4.9K*  8月 17 23:13 libgreetings_enjavi.a  
```

Đó là 4.9K (dung lượng NET), còn trên đĩa chắc phải hơn.

### 3.2 Build thư viện động

Ta đã build thư viện tĩnh, được file .a (một file nén chưa 3 file .o). Kích thước 4.9K(NET)  
Giờ ra build thư viện động.  
Vậy động với tĩnh khác nhau ở chỗ nào, tại sao phải build cả động làm gì.  
Chuyện khá dài, tôi sẽ trình bày ngắn gọn. Thư viện tĩnh, tức là tất cả xử lý của mọi hàm đều nằm trong thư viện đó rồi. Ví dụ, nó gọi hàm _printf_ trong source .c, thì khi tạo thư viện tĩnh, nó ôm toàn bộ xử lý của printf vào trong file .a mà ta vừa tạo.  
Còn thư viện động, thì không làm thế, nó lưu lại thông tin báo là, chỗ này gọi hàm printf với các tham số x,y,z gì đó. Khi nào chạy thì hỏi tiếp xem hàm đó ở đâu, xử lý thế nào rồi mới truyền các tham số để xử lý.

Cấu trúc các file source, header không thay đổi gì so với khi build thư viện tĩnh.  
Ta cần thay đổi file _CMakeList.txt_ một chút:  
Nó sẽ có nội dung như thế này:

```bash  
#  
# Điều kiện phiên bản nhỏ nhất của CMake  
#  
cmake_minimum_required(VERSION 2.8.12)

#  
# Tên project  
#  
project (multilang_greetings)

#  
# Thêm thư mục chứa các file header (.h)  
#  
include_directories(include)

#  
# Thêm một tập các file source  
#  
file(GLOB SOURCES &quot;src/*.c&quot;)

#  
# Thư viện bao gồm những source nào.  
# Loại thư viện là SHARED(động, thực ra không sát nghĩa cho lắm)  
# Tên thư viện có thể khác tên *project*  
#  
add_library(greetings_enjavi SHARED ${SOURCES})  
```

Kết quả của khi chạy _CMake_ là không thay đổi gì, còn khi chạy _make_, nó sẽ như thế này:

```bash 
oedev@OECrossDev:~/Code/CMake/build$ make  
Scanning dependencies of target greetings_enjavi  
[ 33%] Building C object CMakeFiles/greetings_enjavi.dir/src/greetings_vi.c.o  
[ 66%] Building C object CMakeFiles/greetings_enjavi.dir/src/greetings_ja.c.o  
[100%] Building C object CMakeFiles/greetings_enjavi.dir/src/greetings_en.c.o  
Linking C shared library libgreetings_enjavi.so  
[100%] Built target greetings_enjavi  
```

Thư viện được sinh ra là _libgreetings_enjavi.so_, cũng bắt đầu với tiền tố _lib_, nhưng phần mở rộng là .so.  
Với thư viện động, ta sẽ kiểm tra nội dung như sau:

```bash 
oedev@OECrossDev:~/Code/CMake/build$ ldd libgreetings_enjavi.so  
linux-vdso.so.1 =&gt;  (0x00007fff3e7fc000)  
libc.so.6 =&gt; /lib/x86_64-linux-gnu/libc.so.6 (0x00007f6cf04e7000)  
/lib64/ld-linux-x86-64.so.2 (0x00007f6cf0acd000)  
```

Thực ra mình cũng không hiểu kĩ mấy dòng này, tốt nhất không giải thích gì. ;))

### 3.3 Thêm ngôn ngữ Es

Giờ ta đã có trong tay siêu thư viện với 2 phiên bản, tĩnh và động.  
Ta sẽ viết code sử dụng chúng, đồng thời thêm chức năng lời chào bằng tiếng Spain nữa.  
+ Với thư viện tĩnh  
Gần như rất giống thư viện động, chỉ khác tên file thư viện thôi.  
Ta chỉ cần thay _libgreetings_enjavi.so_ trong file _CMakeList.txt_ ở phần dưới bằng _libgreetings_enjavi.a_, ta đã build ở phần trên thôi.

+ Với thư viện động
Cấu trúc thư mục sẽ như sau:

```bash 
oedev@OECrossDev:~/Code/CMake$ tree -L 2  
.  
├── build  
├── CMakeLists.txt  
├── include  
│   ├── greetings_en.h  
│   ├── greetings_es.h  
│   ├── greetings_ja.h  
│   └── greetings_vi.h  
├── lib  
│   └── libgreetings_enjavi.so  
└── src  
├── greetings_es.c  
└── mainapp.c  
```

Ta cũng cần sửa lại file _CMakeList.txt_ nữa  
Nó sẽ thành như thế này:  
_CMakeList.txt_

```bash  
#  
# Ràng buộc về version  
#  
cmake_minimum_required(VERSION 2.8.12)

#  
# Tên project  
#  
project (multilang_greetings)

#  
# Khai báo nơi chưa thư viện (.so)  
#  
set (PROJECT_LINK_LIBS libgreetings_enjavi.so)  
link_directories(lib)

#  
# Nơi chưa các file header(.h)  
#  
include_directories(include)

#  
# Thêm source .c bằng lệnh set  
#  
#set(SOURCES src/greetings_es.c)

#  
# Thêm source .c bằng file GLOB  
#  
file(GLOB SOURCES &quot;src/*.c&quot;)

#  
# File chạy sẽ được tạo như thế nào  
# Sử dụng thư viện đã khai báo qua PROJECT_LINK_LIBS  
#  
add_executable(helloworld ${SOURCES})  
target_link_libraries(helloworld ${PROJECT_LINK_LIBS})  
```

Sau đó, ta cũng chạy _CMake_ và _Make_ để tạo ra file chạy:  
Kết quả của việc chạy CMake hầu như vẫn vậy.  
Còn kết quả của _Make_ sẽ như thế này:

```bash
oedev@OECrossDev:~/Code/CMake/build$ make  
Scanning dependencies of target helloworld  
[ 50%] Building C object CMakeFiles/helloworld.dir/src/mainapp.c.o  
[100%] Building C object CMakeFiles/helloworld.dir/src/greetings_es.c.o  
Linking C executable helloworld  
[100%] Built target helloworld  
```

File chạy là _helloworld_  
Kết quả chạy của file này sẽ như thế này:

```bash 
oedev@OECrossDev:~/Code/CMake/build$ ./helloworld  
Hello world  
Konichiwa!!!  
Xin chao !!!!  
Hola!!!  
```