---
layout: post
title: Ví dụ cơ bản với pthread trên Linux
date: 2017-05-20 15:50:20.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
tags:
- pthread
- thread
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '5254155553'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2017/05/20/vi-du-co-ban-voi-pthread-tren-linux/"
---

## 1. Tạo và dừng thread

Source code gốc lấy từ [link](https://computing.llnl.gov/tutorials/pthreads/samples/hello.c)  

```c
/******************************************************************************
 * FILE: hello.c
 * DESCRIPTION:
 * Chuong trinh demo thao tac tao thread, tat thread
 * AUTHOR: Blaise Barney
 * LAST REVISED: 08/09/11
 *
 * TRANSLATE to Vietnamese by Minatu
 * COMPILE CMD : $gcc -o hello hello.c -lpthread
 ******************************************************************************/
//
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#define NUM_THREADS 5

void *PrintHello(void *threadid) {
  long tid;
  tid = (long)threadid;
  printf("Hello World! Thread #%ld o day !!!n", tid);
  pthread_exit(NULL);
}

int main(int argc, char *argv[]) {
  pthread_t threads[NUM_THREADS];
  int rc;
  long tId;

  for (tId = 0; tId < NUM_THREADS; tId++) {
    printf(" Ham main() : Thread Main : Tao...thread moi: %ldn", tId);
    rc = pthread_create(&threads[tId], NULL, PrintHello, (void *)tId);
    if (rc) {
      printf("ERROR; loi khi chay pthread_create(), ma loi: %dn", rc);
      exit(-1);
    }
  }

  /* Last thing that main() should do */
  pthread_exit(NULL);
}
}  
```

Kết quả sau khi biên dịch và chạy:

```bash
oedev@OEDEV-Ubuntu:code$ gcc -o hello hello.c -lpthread  
oedev@OEDEV-Ubuntu:code$ ./hello  
Ham main() : Thread Main : Tao...thread moi: 0  
Ham main() : Thread Main : Tao...thread moi: 1  
Hello World! Thread #0 o day !!!  
Ham main() : Thread Main : Tao...thread moi: 2  
Hello World! Thread #1 o day !!!  
Ham main() : Thread Main : Tao...thread moi: 3  
Ham main() : Thread Main : Tao...thread moi: 4  
Hello World! Thread #2 o day !!!  
Hello World! Thread #3 o day !!!  
Hello World! Thread #4 o day !!!  
```

Như ta thấy, các thread được tạo và chạy hàm `PrintHello` rồi hiển thị giá trị `tId`  
được truyền khi tạo trong nội dung của nó.  
Ta chú ý ở đây, thông tin hiện ra màn hình có thể không theo **thứ tự đâu**.  
Tức là có thể dù thread 1 được tạo trước thread 2 nhưng chưa chắc chữ `Hello World`  
của nó sẽ ra trước thread 2.

Nói kĩ hơn về các hàm sử dụng ở trên:

```c  
rc = pthread_create(  
&threads[tId],  
NULL,  
PrintHello,  
(void *)tId);  
```

Trong đó,  
- Tham số thứ nhất: `&amp;threads[tId]` : Chứa các thông tin của thread sau khi được tạo.  
Bất cứ thao tác nào liên quan đến thread này đều thông qua biến này.

*   Tham số thứ 2 : `NULL` : Tức là không set thêm thuộc tính nào cho thread sẽ được tạo,  
    lấy hết mặc định.
*   Tham số thứ 3 : `PrintHello` : Hàm (hoặc con trỏ hàm) sẽ chạy trong thread sau khi được tạo.  
    Thông thường hàm này sẽ chạy mãi bằng một vòng lặp vô tận chẳng hạn.
    
*   Tham số thứ 4 : `(void*)tId` : Tham số này là một con trỏ thôi, kiểu gì cũng được.  
    Nó sẽ được truyền như là tham số như thread thực hiện hàm đã truyền ở tham số thứ 3.  
    Trong trường hợp này hàm `PrintHello` nhận tham số `void *` vì thế nên ép kiểu sang  
    `void *` cho `tId`. Và ta cũng thấy, giá trị `tId` được ép kiểu trở lại trong hàm `PrintHello`
    

Tiếp theo đến

```c 
pthread_exit(NULL);  
```

*   Tham số của hàm này ở đây được sử dụng để truyền ra cho một thread khác đang đợi bằng `join`  
    nó kết thúc, trong trường hợp này không sử dụng nên giá trị nó là NULL.

## 2. Thêm một chút xử lý cho thread

*   Giả sử mỗi thread chạy một vòng lặp vô hạn, sẽ in giá trị tăng dần đến các terminal (được bật sẵn)  
    khác nhau.
*   Để đơn giản, ta sử dụng 4 terminal.  
    1 Terminal (gọi là Terminal 0) để cho thread chính hay chính cái xử lý chứa hàm `main()`  
    3 Terminal (gọi là Terminal 1,2,3) còn lại sẽ sử dụng để 5 thread con được tạo hiển thị giá trị:  
    Thread 1, 2 sẽ hiển thị Terminal 1  
    Thread 3, 4 sẽ hiển thị Terminal 2  
    Thread 5 sẽ hiển thị Terminal 3

*   Mỗi terminal trong Linux có thể được truy cập qua một đường dẫn file.  
    Ta có thể lấy đường dẫn của terminal ta đang sử dụng bằng lệnh `tty`  
    Giả sử 4 terminal của chúng ta có đường dẫn tương ứng là:  
    `/dev/pts/15` đến `/dev/pts/17`
    
*   Ta sẽ sửa lại hàm `PrintHello` ở trên như sau:
    

```c
const char *TERMINAL_DEV_FILES[] = {
    "/dev/pts/19", "/dev/pts/19", "/dev/pts/20", "/dev/pts/20", "/dev/pts/21",
};

void PrintHello(void *threadid) {
  FILE *fo = NULL;
  long count = 0;
  long tid;
  tid = (long)threadid;
  fo = fopen(TERMINAL_DEV_FILES[tid], "a");
  if (fo != NULL) {
    count = 0;
    while (1) {
      fprintf(fo, "Thread %ld : %ldn", tid + 1, ++count);
      if (count == 100)
        break;
      sleep(3);
    }
  }
  fclose(fo);
  printf("Bye!!!! Thread #%ld o day !!!n", tid);
  pthread_exit(NULL);
}
```

Giử sử thread chính sẽ được chạy từ một terminal khác.  
Kết quả sẽ như sau:  
[https://youtu.be/Gpy6m8B19qc](https://youtu.be/Gpy6m8B19qc)