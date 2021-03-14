---
layout: post
title: 2 cách sử dụng thư viện trong Linux
date: 2016-06-12 13:20:13.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
- Compile
tags:
- Library
- Linking
- ObjectFile
meta:
  _wpcom_is_markdown: '1'
  _thumbnail_id: '694'
  _rest_api_published: '1'
  _rest_api_client_id: '43452'
  _publicize_job_id: '23759558350'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/06/12/2-cach-su-dung-thu-vien-trong-linux/"
---
Mình đang gặp 1 vấn đề là 1 hàm _F1()_ trong thư viện động mình viết _(lib1.so)_ không hoạt động đúng như mong muốn. Trong hàm _F1()_có gọi một loại hàm _F21(), F22(), F23()...F2n()_ từ 1 thư viện tĩnh _(lib2.a)_ khác.

Khi build không gặp lỗi,

Khi biên dịch thư viện động này với 1 file **sample.c** để chưa **main()** để chạy thử và gọi hàm _F1() _từ thư viện động _lib1.so_. Kết quả vẫn mong muốn.

Tuy nhiên, khi ném thư viện đó vào ứng dụng khác (mà mình không biết họ có sử dụng cùng cách làm với ví dụ của mình không) thì các hàm _F2x() _có vẻ vẫn chạy nhưng kết quả không như mong muốn, và luôn đưa ra những kết quả giống nhau.

Tìm hiểu 1 chút về Dynamic Libraries, thì thấy có vẻ chưa hiểu kĩ lắm nên sẽ tìm 1 vài bài để dịch, đọc. Có thể chưa phải là đúng tutor cho đúng problem. Nhưng cứ dịch đã, hiểu được sẽ giải thích được và tìm ra được nguyên nhân ở trên.

Đầu tiên là bài **[Anatomy of Linux dynamic libraries](http://www.ibm.com/developerworks/library/l-dynamic-libraries/) **của** [M. Tim Jones](http://www.ibm.com/developerworks/library/l-dynamic-libraries/#authorN10024), **từ IBM.

## Mổ xẻ về thư viện động trong Linux

**Process và API**

Dynamically linked shared libraries (thư viện chia sẻ liên kết động hay gọi tắt là thư viện động) là một tính năng quan trong của GNU/Linux®. Nó cho phép thực thi các chức nưng một cách mềm dẻo hơn bằng cách thực thi các hàm ở thời điểm chạy. Vì chỉ được gọi khi cần thiết, nên sẽ giảm các tiêu tốn bộ nhớ không cần thiết khi chúng chưa được gọi. Bài viết này tập trung vào quá trình tạo và sử dụng thư viện động, cung cấp chi tiết về các công cụ để xem xét chúng cũng xem áp dụng nó như thế nào.

* * *

Các thư viện dược thiết kế với mục đích đóng gói 1 chức năng trở thành một unit đơn. Các unit này có thể được cùng sử dụng giữa nhiều developers. Vì thế ta mới có cách gọi **modular programming. ** (hay chương trình được build từ nhiều modules). Linux hỗ trỡ 2 loại thư viện với những lợi thế và bất lợi riêng của nó. Đầu tiên là thư viện tĩnh (static library) sẽ chứa các chức năng sẽ được kéo vào program ở thời điểm biên dịch. Loại thứ 2 là thư viện động (dynamic libraries), nó sẽ được load và gắn vào để chạy khi chạy chương trình.

Hình bên dưới miêu tả một cách đơn giản về các loại thư viện:

![figure1](/post/images/figure1.gif)

Về thư viện tĩnh, do được đưa vào ứng dụng tại thời điểm biên dịch. Vì thế, với dụng ứng dụng nhỏ, chúng vẫn phù hợp. Với những ứng dụng, cần nhiều thư viện, thì thư viện động, sẽ giảm dung lượng lưu trữ và bộ nhớ phải sử dụng. Ví khi cần, chúng mới được load vào bộ nhớ để sử dụng. Nhiều chương trình có thể cùng sử dụng 1 bản copy của thư viện động. Còn với thư viện tĩnh, mỗi chương trình sẽ giữ 1 bản copy của riêng nó.

Thư viện động 2 cách sử dụng :liên kết động ở thời điểm chạy và load động bởi chương trình. Bài này sẽ tập trung vào 2 cách sử dụng này.

GNU/Linux cung cấp 2 cách để làm việc với thư viện động (chúng được kế thừa từ Sun Solaris). Bạn có thể liên kết động chương trình của bạn với thư viện động và Linux sẽ thực hiện việc load khi cần thiết khi ứng dụng của bạn yêu cầu (mà nó chưa được load trước đó). Và một cách sử dụng khác nữa, là lựa chọn hàm cần gọi để load vào, cái này gọi là **dynamic loading. **Với **dynamic loading, **một chương trình có thể chỉ định để loại một thư viện cụ thể (nếu nó chưa được load) và gọi 1 hàm cụ thể trong đó (hình dưới miêu tả điều này). Các phần mềm hỗ trợ plugins là một ví dụ phổ biến của cách sử dụng này. Tôi sẽ đưa ví dụ ở phần sau. :)

![figure2](/post/images/figure2.gif)

## Liên kết động với Linux

Nào, giờ chúng ta cùng xem xét quá trình sử dụng của thư viên chia sẻ liên kết động trong Linux. Khi user bắt đầu ứng dụng, thực chất là sẽ thực hiên một Executable and Linking Format (ELF) image (ảnh về định dạng liên kết và chạy). Nhân sẽ bắt đầu quá trình load ELF image vào không gian nhớ ảo của user (user space virtual memory). Nhân sẽ dựa trên một SECTION gọi là **.interp** của image, cái sẽ cho dynamic linker biết thư viện nào được sử dụng đến trong ứng dụng (ở hình dưới ta thấy có /lib/ld-linux.so). Phần này tương tự với định nghĩa về interpreter trong các file Script của UNIX®(như #!/bin/sh)., nhưng nó được sử dụng trong 1 nghĩa cảnh khác mà thôi.

```shell
## Listing 1. Using readelf to show program headers
mtj@camus:~/dl$ readelf -l dl

Elf file type is EXEC (Executable file)
Entry point 0x8048618
There are 7 program headers, starting at offset 52

Program Headers:
  Type           Offset   VirtAddr   PhysAddr   FileSiz MemSiz  Flg Align
  PHDR           0x000034 0x08048034 0x08048034 0x000e0 0x000e0 R E 0x4
  INTERP         0x000114 0x08048114 0x08048114 0x00013 0x00013 R   0x1
      [Requesting program interpreter: /lib/ld-linux.so.2]
  LOAD           0x000000 0x08048000 0x08048000 0x00958 0x00958 R E 0x1000
  LOAD           0x000958 0x08049958 0x08049958 0x00120 0x00128 RW  0x1000
  DYNAMIC        0x00096c 0x0804996c 0x0804996c 0x000d0 0x000d0 RW  0x4
  NOTE           0x000128 0x08048128 0x08048128 0x00020 0x00020 R   0x4
  GNU_STACK      0x000000 0x00000000 0x00000000 0x00000 0x00000 RW  0x4

  ...

mtj@camus:~dl$
```

Ở đây, ld-linux.so là một interpreter nhưng bản thân nó cũng chính là thư viện cha sẻ ELF. Tuy nhiên nó lại được biên dịch tĩnh và không cần một thư viện phụ thuộc nào hết. Khi quá trình dynamic linking được thực hiên, kernel sẽ gọi dynamic linker (tức là ELF Intepreter) để khởi tạo. Rồi nó sẽ load các thư viện chia sẻ cần thiết(nếu thư viện đó chưa được load vào). Sau khi load xong, nó sẽ thực hiện bước reloation cho các thư viện chia sẻ sẽ sử dụng trong ứng dụng. Thông thường biến môi trường, LDLIBRARY_PATH sẽ định nghĩa nơi chứa những thư viện chia sẻ hiện có. Sau khi kết thúc việc load, reloations thì quyền điều khiển được trả về địa chỉ đầu tiên để chạy chương trình.

Relocations được thực hiện thông qua một cơ chế gián tiếp gọi là Global Offset Table (GOT) và Procedure Linkage Table (PLT). Hai bảng này sẽ cung cấp địa chỉ của external function và data của các thư viện chia sẻ mà ld-linux.so load được trong quá trình relocations. Có nghĩa là code sẽ không thay đổi chỉ có 2 bảng này phải điều chỉnh thôi. Relocations được thực hiện ngay khi load xong hoặc bất cứ khi nào hàm được gọi. (Xem thêm phần Dynamic loading with Linux).

Khi relocation xong, dynamic linker sẽ cho phép các thư viện chạy code khởi tạo của nó. Chức năng này cho phép thư viện khởi tạo dữ liệu cần thiết cho việc sử dụng sau đó. Phần code này được định nghĩa ở section **.init **của ELF image. Ngược lại, khi thư viện bị unload thì dynamic linker cũng sẽ thực hiện code kết thúc tương ứng (được định nghĩa ở section .**fini. **Khi hàm khởi tạo được gọi, dynamic linker sẽ trả điều khiển về cho ELF image.

## Load động trên Linux

Ngoài khả năng tự động load và link thư viện cho chương trình tương ứng, Linux cũng cung cấp cách để cho chính ứng dụng có quyền điều khiển quá trình load và link này.  Cái đó gọi là load động. Nó khác các Liên kết động ở trên là, trong mã của ứng dụng không biết trước sẽ sử dụng hàm nào trong thư viện. Khi thực hiện load động, ứng dụng có quyền chỉ rõ thư viện cụ thể nào nó load và sử dụng để chạy (gọi hàm bên trong ứng dụng). Tuy có vẻ khác như thế, nhưng những gì ta đã biết về ELF image ở trên vẫn không thay đổi. ld-linux dynamic linker vẫn thực hiên như là ELF loader, Interpreter.

Các Dynamic Loading (DL) API được cung cấp, sẽ cho phép load và sử dụng thư viện trong user-space (program). Hình bên dưới mô tả danh sách các API, số lượng tuy khá nhỏ (dù lượng xử lý bên trong các API này không hề nhỏ) nhưng chừng đó là đủ để cho ta mọi thứ cần thiết.

`dlopen`

Tạo một đối tượng để chương trình dùng nó truy cập đến file thư viện

`dlsym`

Lấy địa chỉ của một symbol thông qua đối tượng đã tạo bởi `dlopen`.

`dlerror`

Trả về một chuỗi mô tả của lỗi cuối cùng vừa xảy ra.

`dlclose`

Đóng đối tượng truy cập file đã mở trước đó.

Bắt đầu bằng việc gọi hàm dlopen, tham số cần cung cấp là đường dẫn file và 1 mode. Lời gọi dlopen sẽ trả về 1 handle, và được sử dụng ở các bước tiếp theo. Mode ở đây sẽ nói cho linker biết khi nào sẽ thực hiện việc relocation.

Có 2 giá trị cho mode : RTLD_NOW và RTLD_LAZY.

RTLD_NOW : yêu cầu linker thực hiện toàn bộ các relocation ngay lập tức.

RTLD_LAZY : yêu cầu linker thực hiện relocation khi cần thiết thôi.

Ở Mode này, được thực hiện trong lúc chuyển tiếp các yêu cầu mà chưa được relocate trước đó. Bằng cách này, dymick linker xem xét cần thực hiện một tham chiếu mới nào không, nếu nó chưa được thực hiện. Dynamic linker sẽ không lặp lại quá trình relocate.

Có 2 mode khác cũng được sử dụng kết hợp với 2 mode ở trên (bằng bitwise). Đó là RTLD_LOCAL và RTLD_GLOBAL. Khi không muốn các chương trình khác sử dụng các symbol mà ứng dụng đã load vào thì hãy dùng RTLD_LOCAL. Ngược lại, muốn để nó toàn cục thì ta hãy sử dụng RTLD_GLOBAL.

Hàm dlopen cũng tự động load các thư viện phục thuộc với thư viện chính. Có nghĩa là, nếu bạn mở một truy cập đến một thư viện động, thì các thư viện khac cũng tự động được load vào. Hàm này sẽ trả về một biến, và biến này sẽ được sử dụng ở các hàm tiếp theo.

Prototype của dlopen:

```c
#include <dlfcn.h>

void ***dlopen**( const char *file, int mode );
```

Khi đã có được đối tượng trả về từ hàm trên, gọi là một handle đến thư viện động (ELF Object). Bạn có thể xác định địa chỉ của bất cứ symbol nào bằng cách gọi hàm **dlsym**. Hàm này sẽ truyền vào tên symbol muốn xác định hay chính là tên của 1 hàm chưa bên trong object đó. Giá trị trả về của hàm này sẽ là địa chỉ được tìm thấy của symbol trong object.

```c
void ***dlsym**( void *restrict handle, const char *restrict name );
```

Khi có lỗi xảy ra khi gọi API trên, bạn có thể sử dụng **dlerror**  để xác định lỗi vừa xảy ra thông qua một chuỗi kí tự. Hàm bên dưới đây, không có tham số và trả về một chuỗi. Hàm này sẽ chỉ trả về kết quả nếu lời gọi ngay trước nó xảy ra lỗi.

```c
char ***dlerror**();
```

Cuối cùng, khi không cần truy cập nữa, ứng dụng chỉ việc gọi **dlclose** để báo cho OS biết hãy dọn dẹp các tham chiếu không cần thiết đi. Việc dọn dẹp này được thực hiện thông qua một biến đếm tham chiếu, vì thế nhiều user có thể dùng chung 1 đối tượng mà không bị tranh chấp. Tất cả các symbol có thể được đọc ra thông qua **dlsym. **Khi không dùng đến nữa, ta nên đóng lại bằng hàm:

```c
char ***dlclose**( void *handle );
```

##  Ví dụ sử dụng Dynamic loading

Bây giờ, chúng ta sẽ làm một ví dụ sử dụng các API bạn vừa thấy phía trên. Trong ứng dụng này, ta sẽ tạo một shell (giao diện dòng lệnh) đơn giản cho phép chỉ định một thư viện để load, 1 hàm để chạy với tham số sẽ được nhập vào luôn. Chúng ta sẽ gọi hàm thông qua load symbol (hàm) bằng các hàm đã tìm hiểu ở phía trên, sau đó truyền vào các tham số tự định nghĩa.

```c
### Toàn bộ source của ví dụ ở đây:

#include <stdio.h>
#include <dlfcn.h>
#include <string.h>

#define MAX_STRING      80


void invoke_method( char *lib, char *method, float argument )
{
  void *dl_handle;
  float (*func)(float);
  char *error;

  /* Open the shared object */
  dl_handle = **dlopen**( lib, RTLD_LAZY );
  if (!dl_handle) {
    printf( "!!! %s\n", dlerror() );
    return;
  }

  /* Resolve the symbol (method) from the object */
  func = **dlsym**( dl_handle, method );
  error = **dlerror**();
  if (error != NULL) {
    printf( "!!! %s\n", error );
    return;
  }

  /* Call the resolved method and print the result */
  printf("  %f\n", (*func)(argument) );

  /* Close the object */
  dlclose( **dl_handle** );

  return;
}


int main( int argc, char *argv[] )
{
  char line[MAX_STRING+1];
  char lib[MAX_STRING+1];
  char method[MAX_STRING+1];
  float argument;

  while (1) {

    printf("> ");

    line[0]=0;
    fgets( line, MAX_STRING, stdin);

    if (!strncmp(line, "bye", 3)) break;

    sscanf( line, "%s %s %f", lib, method, &argument);

    invoke_method( lib, method, argument );

  }

}
```

Để build source trên, ta sử dụng GCC (GNU Compiler Collection), với option **-rdynamic**, để báo với GCC Linker rằng, hãy thêm tất cả các symbol (hàm, biến toàn cục) vào bảng dynamic symbol (bảng symbol động), sẽ cho phép truy vết ngược khi sử dụng **dlopen**. Một tham số nữa là **-ldl **để báo rằng chương trình sẽ linked với thư viện **dllib**.

```shell
gcc -rdynamic -o dl dl.c -ldl
```

Ở hàm _main()_ ở trên, ta thấy nó thực hiện một nhiệm vụ khá đơn giản là đọc vào và pass 3 tham số (tên thư viện, tên hàm, tham số kiểu float). Nếu nhập vào "bye" ứng dụng sẽ thoát. Nếu 3 tham số thỏa mãn điều kiện, thì nó sẽ được pass qua hàm invoke_method.

Dưới đây là một vài ví dụ chạy của ví dụ ở trên:

```shell
mtj@camus:~/dl$ gcc -rdynamic -o dl dl.c -ldl
mtj@camus:~/dl$ ./dl
> libm.so cosf 0.0
  1.000000
> libm.so sinf 0.0
  0.000000
> libm.so tanf 1.0
  1.557408
> bye
mtj@camus:~/dl$
```

### Các tools sử dụng

Linux cung cấp một loạt tool để xem, đọc các đối tượng ELF (các thư viện chia sẻ động cũng là một loại đối tượng ELF). Một trong những tool hữu ích nhất là **ldd, **dùng để liệt kết các thư viện chia sẻ mà đối tượng ELF phụ thuộc vào. Khi sử dụng **ldd**, ta sẽ có kết quả như dưới đây:

```shell
mtj@camus:~/dl$ ldd dl
 linux-vdso.so.1 => (0x00007ffcd20e5000)
 libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007fa3acdb5000)
 libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fa3ac9f0000)
 /lib64/ld-linux-x86-64.so.2 (0x0000561d20bc7000)
mtj@camus:~/dl$
```

Ta thấy kết quả **ldd** ở trên có

_linux-vdso.so_ : đây là một thư viện đặc biệt, không liên quan đến file trong hệ thống file.

_libdl.so_: chứa các hàm liên quan đến Dynamic Library (thư viện động).

_libc.s_: Thư viện GNU C.

_/lib64/ld-linux-x86-64.so.2_ : Loader của Linux.

Một công cụ hết sức mạnh mẽ, cho phép đọc nội dung đối tượng ELF, đó là **readelf**. Một trong những cách sử dụng phổ biến của công cụ này là xác định các phần tử relocation trong đối tượng (phần tử relocation có nghĩa là các hàm chưa được xác định địa chỉ khi biên dịch, nó sẽ được xác định khi chạy).

Với ví dụ ta đã compile ở trên:

```shell
mtj@camus:~/dl$ readelf -r dl

Relocation section '.rela.dyn' at offset 0x7b0 contains 2 entries:
 Offset Info Type Sym. Value Sym. Name + Addend
000000601ff8 000700000006 R_X86_64_GLOB_DAT 0000000000000000 __gmon_start__ + 0
000000602080 001300000005 R_X86_64_COPY 0000000000602080 stdin + 0

Relocation section '.rela.plt' at offset 0x7e0 contains 11 entries:
 Offset Info Type Sym. Value Sym. Name + Addend
000000602018 000100000007 R_X86_64_JUMP_SLO 0000000000000000 strncmp + 0
000000602020 000300000007 R_X86_64_JUMP_SLO 0000000000000000 __stack_chk_fail + 0
000000602028 000400000007 R_X86_64_JUMP_SLO 0000000000000000 printf + 0
000000602030 000500000007 R_X86_64_JUMP_SLO 0000000000000000 __libc_start_main + 0
000000602038 000600000007 R_X86_64_JUMP_SLO 0000000000000000 fgets + 0
000000602040 000700000007 R_X86_64_JUMP_SLO 0000000000000000 __gmon_start__ + 0
000000602048 000800000007 R_X86_64_JUMP_SLO 0000000000000000 dlopen + 0
000000602050 000900000007 R_X86_64_JUMP_SLO 0000000000000000 __isoc99_sscanf + 0
000000602058 000a00000007 R_X86_64_JUMP_SLO 0000000000000000 dlclose + 0
000000602060 000d00000007 R_X86_64_JUMP_SLO 0000000000000000 dlsym + 0
000000602068 000e00000007 R_X86_64_JUMP_SLO 0000000000000000 dlerror + 0
mtj@camus:~/dl$
```

Từ danh sách này, ta có thể thấy rất nhiều hàm yêu cầu phải relocation như đến libc.so, các lời gọi liên quan đến load thư viên động (libdl.so). Hàm **__libc_start_main**, được gọi đến hàm **main()** trong source code.

Một công cụ khác nữa cũng hay được sử dụng đó là **objdump, **sẽ hiển thị các thông tin về file đối tượng, **nm** sẽ liệt kê các hàm, từ đối file đối tượng. Ngoài ra, ta cũng có thể gọi trực tiếp Linker  liên kết động bằng cách sau:

```shell
mtj@camus:~/dl$ /lib/ld-linux.so.2 ./dl
> libm.so expf 0.0
  1.000000
>
```

**ld-linux.so.2** với tham số --list sẽ tương đương với lệnh **ldd**

PS: Đã giải quyết được vấn đề nói ở đầu bài, các hàm F2x() đã không được gọi. Lý do là, các hàm này đã tồn tại ở một thư viện trước đó. Mà thực ra là phiên bản thư viện động của thư viện **lib2.a**. Chỉ load trước thư viện tự build động vào trước, thì hiện tương trên sẽ hết.