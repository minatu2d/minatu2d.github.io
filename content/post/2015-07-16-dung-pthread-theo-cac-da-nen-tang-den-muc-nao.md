---
layout: post
title: Cơ bản về pthread
date: 2015-07-16 23:18:38.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
- thread
tags:
- Draft
- pthread
meta:
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '12800494909'
  _thumbnail_id: '69'
  _wpcom_is_markdown: '1'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2015/07/16/dung-pthread-theo-cac-da-nen-tang-den-muc-nao/"
---
Ngày trước, khi tìm hiểu về Java, rồi Qt, nghe đến thuật ngữ đa nền (multi-platform). kì thực cái multi platform đó sẽ được phát triển như thế nào. Nó có thực sự dễ dàng như họ quảng cáo? Họ thường quảng cáo rằng, chỉ cần thay đổi cấu hình bằng một vài click chuột thì có thể build lại một dự án bất kì của Qt từ OS này sang OS khác.

Khi gặp những ứng dụng chạy đa nền tảng mình mới thấy quảng cáo chỉ miêu tả rất nhỏ thôi. Các dự án thực tế luôn có cực kì nhiều vấn đề. Nó không phải là hello world mà chỉ bằng vài click là đa nền tảng được. Ta biết rằng, những thư viện như Qt hỗ trợ rất nhiều. Nhưng thông thường các dự án chỉ dùng 1 phần thôi, còn lại họ tự viết hoặc sử dụng lại ở đâu đó. Vì vậy, nếu nói một dự án pure Qt thì đương nhiên dễ dàng chuyển đổi giữa các platform. Còn một dự án không phải là pure Qt thì rất nhiều vấn đề phát sinh.

Nói lan man rồi, phần chính của bài này là nói về **pthread**.  
Pthread được giới thiệu là thư viện implement chuẩn POSIX đặc tả việc tạo nhiều luồng tính toán song song.  
Hầu hết các implement của pthread là trên Linux. Ta hiểu POSIX miểu tả các API ( tức là các tên hàm và chức năng của chúng). Khi viết các hàm đó, thì tùy mỗi nền tảng mà người ta sẽ có những các khác nhau để thực hiện được nhiệm vụ được miêu tả trong đặc tả của các hàm này.

Trong bài này ta sẽ tìm hiểu một số thứ về pthread trên Linux và Window, chắc bài này sẽ khá đầu "lâu" nên thui mỗi hôm viết một tẹo vậy.

Thứ tự thực hiện sẽ như sau:

I. Những thứ cần biết về pthread API  
II. Test thử trên Window.  
III. Test thử trên Linux.


---

# I. Những thứ cần biết về pthread.

## 1. Phân loại API của pthread
Dựa theo chức năng mà các API của pthread được chia làm 4 loại:
- Thread management: Là các hàm sử dụng để tạo, hủy, detached, join thread cũng như set/get các thuộc tính của thread nữa.  
- Mutexes: Là các hàm sử dụng để tạo, hủy, unlocking, locking mutex ("mutual exclusion" : vùng tranh chấp), cũng như set/get các thuộc tính của mutex.  
- Condition variables: Là các hàm để tạo, hủy, đợi hoặc phát tín hiệu dựa trên giá trị của một biến cụ thể.  
- Synchronization: Là các hàm dùng để quản lý việc read/write lock và barriers.  
- Nhưng hàm thuộc cùng 1 loại API ở trên sẽ có tên tương tự nhau.  
Ví dụ: pthread_create_xXX, pthread_join; pthread_mutex_XXX, pthread_cond_XXX etc

## 2. Cách tạo thread

- Điều đầu tiên người ta muốn biết luôn là làm thế nào để tạo thread sử dụng pthread, nó có nhiều tham số không. có dễ hiểu không. Cơ chế truyền xử ý của nó là gì? không biết có điều kiện gì khó khăn không???

- Để tạo thread sử dụng pthread, rất đơn giản, ta gọi hàm pthread_create ở bất kì đâu ta muốn gọi, chú ý sau khi chạy hàm này thì thread mới cũng sẽ chạy luôn, chứ không chờ được start đâu nhé.

```c
int pthread_create(pthread_t *restrict thread,  
const pthread_attr_t *restrict attr,  
void *(*start_routine)(void*), void *restrict arg);  
```

Đây là nguyên mẫu từ sách giáo khoa (https://computing.llnl.gov/tutorials/pthreads/man/pthread_create.txt)  
Trông rất khó hiểu, ta sẽ đơn giản mấy tên tham số 1 chút:

Ta bỏ qua những từ khóa khó hiểu : restrict, ta tập trung vào nhiệm vụ mỗi tham số để hiểu được cần phải truyển cái gì vào đó.

- _p_thread_t thread_ : Một giá trị được sử dụng như một định danh cho thread mới được tạo ra, nó sẽ được hàm này trả về, chúng ta không cần tạo trước giá trị, ta chỉ cần truyển con trỏ của một biến kiểu p_thread_t là đủ. Giá trị của biến truyển vào sẽ được thay đổi bên trong hàm tạo.  
Từ sau đó, ta sẽ sử dụng nó cho các hàm khác với thread nếu cần.  
- _p_thread_attr_t attr_ : Một giá trị được sử dụng để thiết lập một số thuộc tính của thread mới được tạo. Tuy nhiên, nếu không có yêu cầu gì đặc biệt. Ta cứ sử dụng một con trỏ NULL để truyển vào, khi đó thread mới được tạo sẽ có các thuộc tính mặc định và chạy được.  
- _oid _(_start_routine)(void*)_ : nhìn là biết đó là cái gì. Xử lý hay chính là hàm được chạy trong thread ta sẽ tạo.  
Hàm này được chạy ngay sau khi kết thúc hàm tạo.  
- _void *arg_ : Chính là tham số sẽ được truyền vào hàm khi thread chạy hàm đó.

Một ví dụ về tạo thread bằng pthread

Chú ý rằng, thread sẽ đựoc chạy ngay sau khi kết thúc hàm tạo.

## 3. Dừng một thread

- Ta đã tạo và làm cho nó chạy, giờ dừng lại thì làm thế nào?  
- Để một chiếc xe đang chạy dừng lại, ta có mấy tình huống sau: xe hết xăng, hoặc dùng phanh + giảm ga để cho nó dừng.  
- Việc dừng một thread cũng tương tự như thế: Nếu thread tự dừng, tức là xử lý của thread kết thúc thì không có gì phải bàn rồi, điều ấy thật tuyệt vời nếu như nó dùng đúng lúc ta muốn. Nếu ta muốn nó dừng lại tại một thời điểm ta mong muốn bất chấp nó có muốn dừng hay không thì làm thế nào??? Cái này mới gọi là chuối đây. Chắc cũng phải tạo ra ma sát để phanh rồi mới dừng đuợc chứ nhỉ.

### 3.1 Tự stop tường minh

- Đầu tiên ta nói đến tự nguyện dừng lại. Nghĩa là mọi thread đều tự gọi một hàm để yêu cầu đựoc dừng lại chứ không bị ai bắt ép gì cả, tức là thread đó thực hiện xong xử lý của nó, nó kết thúc.  
- Hàm đó là _pthread_exit(NULL);_  
- Nếu hàm này đuợc gọi trong thread ngoài main(), nó sẽ thực hiện việc dừng thread đang chạy.  
- Nếu hàm này đuợc gọi trong thread main(), nó sẽ đợi đến khi các thread con đuợc thoát tường mình rồi nó mới thoát.

```c
#include<stdio.h>
#include<pthread.h>  
#define NUM_THREADS 5

void *PrintHello(void *threadid)  
{  
  long tid;  
  tid = (long)threadid;  
  printf("Hello World! It's me, thread %ld!\n", tid);  
  pthread_exit(NULL);  
}

int main (int argc, char *argv[])  
{  
  pthread_t threads[NUM_THREADS];  
  int rc;  
  long t;  
  for(t=0; t < NUM_THREADS; t++){  
  printf("In main: creating thread %ld\n", t);  
  rc = pthread_create(&threads[t], NULL, PrintHello, (void *)t);  
  if (rc){  
    printf("ERROR: return code from pthread_create() is %d\n", rc);  
    exit(-1);  
  }  
}

/* Last thing that main() should do */  
pthread_exit(NULL);  
}  
```

### 3.2 Dừng bắt buộc (Để sau viết tiếp)

- Có một số trường hợp, khi có một yêu cầu bắt buộc phải dừng 1 thread đang chạy từ 1 thead khác chẳng hạn.  
Ví dụ, người dùng yêu cầu tải 1 file lớn từ internet, ta cắm xử lý đó vào một thread. Tuy nhiên, ngay sau đó người dùng không còn muốn file đó nữa, họ cancel, lúc này ta phải dừng được thread đang chạy kia để không tốn xử lý vô ích.  
(Sẽ đuợc bổ sung sau)

## 4. Cơ chế trao đổi dữ liệu và đồng bộ

- Hiểu đơn giản là giống như, mỗi người thực hiên một việc, nhưng nó có liên quan đến nhau.  
- Người này thực hiện đến một số bước cần kết quả của người kia. Cần gọi điện, hay gửi mail hay gì đó để báo cho nhau biết.  
- Thread cũng như thế, cái trao đổi ở đây là dữ liệu.  
- Trên máy tính để xác định dữ liệu ở đâu người ta dùng địa chỉ. Hay nói cách khác, các thread muốn trao đổi với nhau thông qua 2 cách:  
+ Nhờ bên thứ 3 truyền dữ liệu đó đến bên kia.  
+ 2 bên cùng biết 1 địa chỉ chung nào đó, dùng trao đổi qua đó.  
À, phân loại ra như vậy thôi chứ rất hiếm khi có ứng dụng nào chỉ dùng 1 cách trao đổi, thường sẽ kết hợp cả 2 để có hiệu quả cao nhất.  
Ta sẽ nói về cái phức tạp hơn trước. Đó là 2 bên cùng biết địa chỉ chung rồi đọc/ghi vào đó.

### 4.1 Mutex

Mutual Exclusion, được sử dụng khi có nhiều thread cùng ghi vào một vùng địa chỉ.  
Đây là 1 ví dụ kinh điển nếu không sử dụng mutex:

[![mutex_first_sample](/post/images/mutex_first_sample.png?w=300)](https://lazytrick.files.wordpress.com/2015/07/mutex_first_sample.png)

Từ ví dụ trên ta thấy, tài khoản ban đầu là $1000, nạp 2 lần $200 mà số tiền cuối trong tài khoản khi hết xử lý là $1200, mất $200. Làm ăn kiểu này không ổn (tất nhiên trong thực tế chắc chắn không có chuyện này rồi)

Để giải quyết tình trạng trên, ta sẽ sử mutex để lock đoạn xử lý liên quan đến Balance.  
Hay ở trên từ bước _Read balance_ đển bước _Update Balance_ ta phải để chúng trong 1 khối mà có xử lý khác chen vào.

Các hàm mà pthread cung cấp cho để sử dụng mutex:

### 4.1.1 Tạo, hủy mutex

Hàm tạo:

```c
int pthread_mutex_init(pthread_mutex_t *restrict mutex,  
const pthread_mutexattr_t *restrict attr);  
```

Ta thấy tham số ở đây là địa chỉ biến ta sử dụng làm mutex và attr.  
Nếu attr là NULL thì mutex sẽ được khởi tạo với thuộc tính mặc định.

> Attempting to initialize an already initialized mutex results in unde-fined behavior.

(Trích https://computing.llnl.gov/tutorials/pthreads/man/pthread_mutex_destroy.txt)

Thực hiện init nhiều lần cùng 1 biến là 1 thao tác mà kết quả không xác định.  
Hàm hủy:

```c
int pthread_mutex_destroy(pthread_mutex_t *mutex);  
```

Tham số là địa chỉ của 1 biến mutex mà ta đã init trước đó.

##### 4.1.2. Sử dụng mutex

_Hàm lock:_

```c  
int pthread_mutex_lock(pthread_mutex_t *mutex);  
```

Hàm sẽ lock mutex được truyền trong tham số.  
Nếu mutex nó định lock đang ở trạng thái bị lock rồi thì nó sẽ đợi đến khi mutex này được gỡ lock.

_Hàm unlock:_

```c 
int pthread_mutex_unlock(pthread_mutex_t *mutex);  
```

Hàm này thực hiện gỡ lock cho một mutex mà nó đã lock.  
Nếu có thread nào đang bị block vì cố gắng lock mutex này, thì một trong số đó sẽ có quyền lock mutex đó.

Ta thấy rằng, hàm lock ở trên sẽ block nếu như mutex đang ở trạng thái lock.  
Nếu ta không muốn bị block mà muốn chuyển qua xử lý tiếp theo thì sao.  
Hàm dưới đây là giải pháp.

_Hàm trylock_

```c 
int pthread_mutex_trylock(pthread_mutex_t *mutex);  
```

Hàm này có nhiệm vụ chính giống hàm Lock nhưng chỉ khác 1 điểm là nếu mutex đó đang bị lock thì nó sẽ trả về ngay mà không đợi đến khi mutex đó được unlock nữa.

### 4.2 Biến điều kiện

- Hay còn gọi là Condition Variables. Nó cung cấp một cách khác để đồng bộ hóa. Trong khi mutex thực hiện việc đồng bộ bằng cách điều khiển quyền truy cập dữ liệu đó tại các "critical" code. Thì condition variable cung cấp cách truy cập dữ liệu dựa trên điều kiện về giá trị của chính dữ liệu đó.

- Để làm điều này mà không sử dụng condition variables, thì programmer cần phải có một xử lý polling dữ liệu để xem điều kiện có được thỏa mãn không, rồi mới thực hiện các xử lý tiếp theo. Condition variables sinh ra để giải quyết vấn đề đó mà không cần phải polling.
- Một condition variable luôn được sử dụng kết hợp với mutex.

#### 4.2.1 Tạo, hủy biến điều kiện

Hàm tạo:

```c 
int pthread_cond_init(pthread_cond_t *restrict cond,  
const pthread_condattr_t *restrict attr);  
```

- Hàm này sẽ khởi tạo một biến điều kiện với một giá trị attr được truyền.  
- Nếu attr bằng NULL thì biến điều kiện sẽ được khởi tạo với các attr mặc định.

Hàm hủy:

```c
int pthread_cond_destroy(pthread_cond_t *cond);  
```

- Hàm này sẽ hủy biến điều kiện được truyền qua tham số.  
- Ta có thể initialize lại biến này để sử dụng như mới. Còn tác thao tác khác nhằm sử dụng lại giá trị trước khi hủy đều không xác định kết quả.

#### 4.2.2 Sử dụng biến điều kiện

Có 2 thao tác được sử dụng cho biến điều kiện: đợi và signal (báo hiệu)  
Hàm đợi:

```c
int pthread_cond_timedwait(pthread_cond_t *restrict cond,  
  pthread_mutex_t *restrict mutex,  
  const struct timespec *restrict abstime);  
int pthread_cond_wait(pthread_cond_t *restrict cond,  
pthread_mutex_t *restrict mutex);  
```

- Cả 2 hàm này sẽ block trên biến điều kiện.  
- Hàm này phải được gọi bên trong 1 đoạn xử lý từ lock 1 mutex đến unlock mutex đó.  
- Ta thấy rằng mutex được truyền vào như là 1 tham số của 2 hàm đợi trên.  
- Nếu mutex được truyền vào mà chưa được lock thì kết quả của 2 hàm đợi này là không xác định.  
- Hàm này sẽ tự động release mutex đang bị khóa để một đoạn xử lý khác đang cố gắng block mutex có thể chạy tiếp.  
Hàm phát tín hiệu:

```c
int pthread_cond_broadcast(pthread_cond_t *cond);  
int pthread_cond_signal(pthread_cond_t *cond);  
```

- Cả 2 hàm đều sẽ unblock (cho thread bị dừng được chạy tiếp) các thread đang bị blocked bởi biến điều kiện truyền vào qua tham số.  
- Hàm đầu tiên sẽ unblock tất cả các thread bị block bởi biến điều kiện được truyền vào.  
- Hàm thứ hai sẽ block 1 thread trong các thread mà biến điều kiện đó gây ra blocked.  
- Bộ lập lịch sẽ xác định thứ tự các thread sẽ được unlock.  
- Vậy khi các thread được unlock thì nó chạy tiếp như thế nào?  
- Ta đã thấy 2 hàm gây ra block ở trên là `pthread_cond_wait()` và `pthread_cond_timedwait()`. Khi đoạn block được unblock thì lời gọi 2 hàm này sẽ được trả về. Việc chạy như thế nào sau đó được quyết định bởi policy của bộ lập lịch.  
- Có 1 điểm đáng chú ý ở đây, 2 hàm pthread_cond_broadcast() và pthread_cond_signal() không chứa bất cứ thông tin nào về mutex, vậy nên nó có thể được gọi bởi bất kì thread nào, ngay cả khi thread đó chẳng giữ mutex nào hết. Tuy nhiên, nếu thao tác lập lịch dự đoán được yêu cầu thì mutex bao quanh đoạn biến điều kiện sẽ bị lock bởi 2 hàm trên.  
- Việc gọi 2 hàm trên không có tác dụng gì nếu chẳng có thread bị block trên biến điều kiện được đưa vào.

## 5. Các công cụ để theo dõi pthread

Ở đây, ta chỉ nói đến các công cụ sử dụng dòng lệnh vì nó sẽ được sử dụng phổ biến trong các hệ thống *NIX.

### 5.1 ps

Ai cũng biết kết quả của lệnh này là in ra màn hình danh sách các process đang chạy.  
Ta có thể sử dụng 1 trong các tham số sau để biết có bao nhiêu thread được đang được chạy từ 1 process.  
**-Lf** (có thể khác nhau trên mỗi hệ thống)

```bash  
% ps -Lf  
UID PID PPID LWP C NLWP STIME TTY TIME CMD  
blaise 22529 28240 22529 0 5 11:31 pts/53 00:00:00 a.out  
blaise 22529 28240 22530 99 5 11:31 pts/53 00:01:24 a.out  
blaise 22529 28240 22531 99 5 11:31 pts/53 00:01:24 a.out  
blaise 22529 28240 22532 99 5 11:31 pts/53 00:01:24 a.out  
blaise 22529 28240 22533 99 5 11:31 pts/53 00:01:24 a.out  
```

Ở đây ta thấy, process 22529 tạo ra 5 thread từ 22529 đến 22533.  
**-T**

```c 
% ps -T  
PID SPID TTY TIME CMD  
22529 22529 pts/53 00:00:00 a.out  
22529 22530 pts/53 00:01:49 a.out  
22529 22531 pts/53 00:01:49 a.out  
22529 22532 pts/53 00:01:49 a.out  
22529 22533 pts/53 00:01:49 a.out  
```

Ta cũng thấy kết quả tương tự ở đây.

**-Lm**

```c
% ps -Lm  
PID LWP TTY TIME CMD  
22529 - pts/53 00:18:56 a.out  
- 22529 - 00:00:00 -  
- 22530 - 00:04:44 -  
- 22531 - 00:04:44 -  
- 22532 - 00:04:44 -  
- 22533 - 00:04:44 -  
```

Ta cũng thấy kết quả tương tự ở đây.

- Nhiều lúc mình tự hỏi tại sao người ta nghĩ ra nhiều lệnh thực hiện cũng một mục đích như vậy làm gì. Mình nghĩ nếu đê trả lời sẽ có 2 ý thế này, thứ nhất cộng đồng Linux là mã nguồn mở, mọi người có thể tự do thêm các tính năng mà mình muốn. Thứ 2, có nhiều option thực hiện các chức năng tương tự nhau, nhưng cách implment khác nhau, vì thế khi triển khai Linux trên các hệ thống nghèo tài nguyên CPU, bộ nhớ(hệ nhúng), người ta không thể cài đặt tất cả chúng được, chỉ một tập nhỏ các option được hỗ trợ thôi. Điều này cực kì quan trọng.

### 5.2 Lệnh top

- Là lệnh theo dõi hiệu năng của máy ở chế độ gần như thời gian thực.  
- Trong bảng dưới đây ta sẽ thấy process 18010 tạo ra 3 thread khác là 18012, 18013, 18014.

![](/post/images/topH.gif)

## 6. Một số thông tin hữu ích khác

-  Khi thoát chương trình từ thread main ( tức là thread tạo chạy hàm _main()_), chúng ta nên gọi _pthread_exit()_ ở cuối để đảm bảo thread main sẽ chờ để thoát hết các thread nó tạo ra trước đó.
- thread joinable và thread không phải joinable.
-- một thread joinable là 1 thread cho phép thread biết thời điểm nó kết thúc và đợi nó kết thúc.  
-- một thread không phải thread joinable là 1 thread mà không được quản lý thời điểm nó kết thúc hay không thread nào biết mà đợi nó kết thúc được.

-  Deatach thread: Đây là một hành động.  

    Khi detached một thread nghĩa là ta làm cho nó thành không joinable.  

    nếu là joinable sẽ cần nhiều thông tin để quản lý thread hơn, nên ta hãy deatach thread khi thực sự cảm thấy không cần để chúng là joinable nữa.  

    Với option cho attr lúc tạo thread, ta cũng có thể tạo ra thread mà trạng thái là không joinable luôn.

# II. Test thử trên Window.

(Vì bài này đã quá dài, và để dễ nắm bắt nên tôi sẽ để phần ví dụ này sang (một post khác khi hoàn thành bài về CMake)

# III. Test thử trên Linux.

[Ví dụ cơ bản với pthread trên Linux](https://lazytrick.wordpress.com/2017/05/20/vi-du-co-ban-voi-pthread-tren-linux/)