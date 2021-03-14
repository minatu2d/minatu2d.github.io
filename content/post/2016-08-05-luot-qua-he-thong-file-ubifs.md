---
layout: post
title: Hệ thống File Ubifs
date: 2016-08-05 01:50:50.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
tags:
- FileSystem
- Flash
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '25481202906'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2016/08/05/luot-qua-he-thong-file-ubifs/"
---
Gần đây, có thấy trên Board mạch phát triển của mình có xuất hiện 1 process là **ubifs**. Qua tìm hiểu mới biết rằng, em nó là **UBIFS**, được phát triển năm 2007 bởi [Nokia](http://www.nokia.com/) với sự giúp đỡ của [University of Szeged](http://www.u-szeged.hu/), Hungary.

Bắt đầu được đưa vào mainline của Linux 2.6.27 năm 2008.

Hỏi thấy GG thấy được 1 [bài giới thiệu chung về hệ thống file cho thiết bị nhớ Flash](https://kythuatmaytinh.wordpress.com/2008/05/26/h%E1%BB%87-th%E1%BB%91ng-file-flash-tren-linux/) của anh [Le Dinh Thao](https://www.facebook.com/lddthao) trên blog [kithuatmaytinh](https://kythuatmaytinh.wordpress.com/).

Trong bài có nói tới các vấn đề khi hệ thống sử dụng Flash gặp phải và có nói đến 2 hệ thống File khá triển vọng là **LogFS** và **UBIFS**  
  
Bài này chỉ hướng đến mục tiêu tìm hiểu về **UBIFS** thông qua việc dịch tài liệu dưới đây: [http://www.linux-mtd.infradead.org/doc/ubifs.html](http://www.linux-mtd.infradead.org/doc/ubifs.html)

## 1. Note quan trọng

Đầu tiên, một thứ chúng ta cần hiểu ngay đó là, UBIFS là hệ thống file không giả định rằng thiết bị nhớ có dạng block (như **_Hard drive, MMC/SD, USB Flash, SSDs, etc_**), vì thế nó khác hoàn toàn các hệ thống file truyền thống như Fat, eFAT, etc. UBIFS được thiết kế để làm việc với **raw flash** (flash thô), không có block nào ở đây hết. Ta có thể ví dụ về trường hợp giống như **_MMC Card_**, mỗi **_MMC Card_** cung cấp ra bên ngoài rằng nó là 1 thiết bị dạng block thông qua FTL (Flash Translation Layer) - được thực hiện bởi phần cứng. Phần cứng này sẽ mô phỏng lại bộ nhớ bên trong sao cho giống với 1 block device cho các giao tiếp bên ngoài. Nếu chưa hiểu rõ về sự khác nhau này, hãy đọc ngay phần cuối của bài.

## 2. Khái quát

Như ta đã nói lúc đầu, **UBIFS** là file system mới được phát triển bởi các kĩ sư của Nokia với sự trợ giúp từ [the University of Szeged](http://www.inf.u-szeged.hu/tanszekek/szoftverfejlesztes/starten.xml). **UBIFS** được xem như là ra đời sau **JFFS2**.

Về **JFFS2**, nó làm việc bên trên các thiết bị MTD. Còn **UBIFS** làm việc trên **UBI Volumnes** (Ổ đĩa UBI), không làm việc trên các thiết bị MTD được. Hay nói cách khác, để làm việc được, thì cần 3 hệ thống phục như sau:  
- **_MTD subsystem_**, cung cấp giao diện duy nhất truy cập đến chip nhớ Flash. Subsystem này cung cấp 1 entry trong device của Linux (thường có dạng /dev/mtd0) để cho các ứng dụng khác truy cập đến **flash thô**.  
- **_UBI subsystem_**, hệ thống sẽ quản lý wear-leveling, các ổ đĩa vật lý trên thiết bị Flash. Nó làm việc phía trên các thiết bị MTD, cung cấp UBI Volumne (ổ đĩa logic UBI). Hay nói cách khác ổ đĩa logic UBI là mức trên của thiết bị MTD, không phải quan tâm đến nhưng vấn đề mà MTD giải quyết như wearing, bad blocks.  
- **_UBIFS file system_**, làm việc phía trên các ổ đĩa logic UBI.

Dưới đây là một số đặc trung của **UBIFS**:  
- **scalability**  
UBIFS cơ thể làm việc tốt khi kích thước Flash được mở rộng vì các thông số liên quan trực tiếp đến UBIFS phụ thuộc rất ít vào dung lượng Flash. UBIFS (đừng nhầm với UBI nha) có thể làm việc tốt với Flash có kích thước lên đến hàng trăm GBs. Tuy nhiên, như miêu tả ở trên UBIFS phụ thuộc vào UBI subsystem, vốn có một số hạn chế để scalability. Dù gì đi nữa, UBIFS/UBI cũng làm việc tốt hơn nhiều JFFS2, cho dù có xảy ra "thắt nút cổ chai" thì người ta có thể thay thế UBI bằng UBI2 mà không cần thay đổi UBIFS.  
- **fast mount** (mount nhanh)  
Không giống JFFS2, UBIFS không cần scan toàn bộ thiết bị nhớ khi mount. Chỉ mất khoảng vài mili giây để mount, và không phụ thuộc vào kích thước của flash. Tuy nhiên, thời gian khởi tạo UBI subsytem phụ thuộc vào kích thước flash, và phải được xem xét kĩ hơn.  
- **hỗ trợ write-back** (cho phép ghi không đồng bộ)  
Đây là một cải tiến rất đáng giá, ít nhất so với JFFS2 (dù có buffer những rất nhỏ, nó gần như là đồng bộ dữ liệu với thiết bị nhớ sau mỗi phép ghi).  
- **chịu được lỗi khi bị tắt đột ngột** (tolerance to unclean reboots)  
UBIFS là hệ thống file dạng nhật kí "jounaling" (các thao tác sẽ được ghi lại) vì thế nó có thể chịu được lỗi xảy ra khi bị hệ thống bị crash hoặc bị reboot bất ngờ. UBIFS sẽ thực hiện lại các thao tác được ghi trong "nhật kí" của nó, và mất thêm 1 khoảng thời gian nhất định. Nhưng vì nó không cần phải scan toàn bộ hệ thống file nên việc mount cùng lắm chỉ diễn ra trong vài giây thôi.

## 3. Chịu lỗi khi bị cắt nguồn

Cả UBI (tức là Subsystem) và UBIFS ngay từ ban đầu đều được thiết kế để có khả năng chống lỗi khi bị cắt nguồn bất ngờ.

## 4. UBIFS và MLC NAND Flash

## 5. Lỗi Unstable bits

Nói về lỗi phát sinh khi bị cắt nguồn điện lúc đang xóa hoặc ghi.

## 6. Source code

## 7. Mailing list

## 8. Tools

Hiện tại chỉ có 1 tool duy nhất phái user-space là **_mkfs.ubifs_** để tạo một "ảnh" UBIFS.

## 9. Khả năng mở rộng (Scalability)

Vì USBFS sử dụng cấu trúc cây (B+ Tree) nên nó sẽ lớn lên theo logarit của kích thước flash. Tuy nhiên, UBI Subsystem thì phụ thuộc tuyến tính vào kích thước Flash, vì thế UBIFS/UBI sẽ phụ thuộc tuyến tính. Tác giả của UBIFS tin rằng có thể cải tiến trở thành phụ thuộc theo logarit ở phiên bản UBI. Hiện tại UBI làm việc OK cho dung lượng từ 2-16GiB, tất nhiên nó cũng phụ thuộc vào tốc độ I/O nữa.

## 10. Hỗ trợ write-back (ghi không đồng bộ)

UBIFS hỗ trợ write-back. Điều đó có nghĩa là các thay đổi liên quan đến file sẽ không được ghi ngay vào Flash, nó sẽ được cache và ghi sau đó khi thực sự cần thiết. Điều này sẽ giúp cho các thao tác I/O giảm thời gian thực hiệnl, làm tăng hiệu năng. "Write-back caching" vốn là kĩ thuật phổ biến được sử dụng trong rất nhiều hệ thống file khác, như Ext3, XFS.  
Ngược lại, JFFS2 lại không hỗ trợ write-back, tất cả các thao tác với hệ thống file đều được thực hiện ngay lập tức lên thiết bị nhớ. Thực ra không hoàn toàn đúng vì JFFS2 có sử dụng một buffer nhỏ bằng kích thước 1 Page. Buffer này chứa dữ liệu cuối cùng được yêu cầu ghi, vì thế nó sẽ được đẩy hết ra khi đầy. Lượng dữ liệu được cached là rất nhỏ, nó gần như hoạt động giống các hệ thống file đồng bộ khác.  
Trong 1 số trường hợp, khi nội dung muốn được phản ánh ngay lên thiết bị nhớ, thì phía ứng dụng sẽ phải thực hiện 1 hành động thêm. Nếu không, sẽ xảy ra trường hợp file bị hỏng hoặc mất khi mất nguồn đột ngột. Điều này thường xảy ra trên các hệ thống Embedded.

Xem lại một chú về man page của **_write_**:

* * *

> $ man 2 write  
> ....  
> NOTES  
> A successful return from write() does not make any guarantee that data  
> has been committed to disk. In fact, on some buggy implementations, it  
> does not even guarantee that space has successfully been reserved for  
> the data. The only way to be sure is to call fsync(2) after you are  
> done writing all your data.  
> ...

* * *

Điều được miêu tả ở **NOTES** phía trên hoàn toàn đúng trong trường hợp của UBIFS (ngoại trừ đoạn "some buggy implementations", vì UBIFS có sử dụng một cách rõ ràng không gian cho dữ liệu chưa được ghi xuống). Điều này cũng đúng JFFS2 và một vài hệ thống file khác.  
Tuy nhiên, nhiều programmer không chú ý đến đoạn write-back này. Có thể họ không đọc kĩ đoạn man page ở trên.  
Ví dụ: Giả sử có 1 ứng dụng không chú ý đoạn có write-back này, nó có thể chạy tốt trên JFFS2 (vì hầu như các thao tác với JFFS2) là đồng bộ. Tất nhiên, nó vẫn có thể gây bug khi chuyển sang hệ thống file khác hỗ trợ write-back như UBIFS chẳng hạn.  
Dưới đây là một check list để các bạn có thể tham khảo khi chuyển từ JFFS2 sang UBIFS:  
- Nếu bạn muốn chuyển sang chạy ở chế độ đồng bộ (synchronous mode), hãy sử dụng tham số **-o sync** khi mount UBIFS. Tuy nhiên, hiệu năng của hệ thống file sẽ bị giảm đáng kể đấy - hãy cẩn thận. Một điều quan trọng cần nhớ nữa là, khi sử dụng mode đồng bộ thì UBIFS không thể bằng JFFS2 (vốn được thiết kế để đồng bộ)  
- **Hãy luôn nhớ rằng**, phải gọi **_fsync()_** cho tất cả các file quan trọng với ứng dụng của bạn. Tất nhiên, đừng dùng cả với các file tạm, nó sẽ làm hại hiệu năng đấy.  
- Nếu bạn muốn tối ưu hơn nữa về mặt hiệu năng, bạn có thể sử dụng **_fdatasync()_**, các dữ liệu chính thay đổi sẽ được đồng bộ còn các thông tin meta-data như **_mtime, permission_** sẽ không được đồng bộ ngay lúc đó. Sử dụng **_fdatasync()_** trong trường hợp phải gọi nhiều lần **_fsync()_** sẽ cho hiệu quả tốt hơn.  
- Trong **_shell_**, lệnh **sync** cũng có chức năng tương tự, nó sẽ đồng bộ cho toàn hệ thống file. Đôi lúc không phải là cách tốt nhất.  
- Bạn cũng có thể sử dụng cờ **0_SYNC** khi gọi **_open()_**. Khi đó, tất cả các thao tác **_write_** sẽ đồng bộ dữ liệu ngay lập tức trước khi trả về. Nó sẽ khá tiện dụng nhưng sẽ không tốt cho hiệu năng so với sử dụng **_fsync()_**, vì **_fsync_** cho phép đồng bộ nhiều dữ liệu sử dụng **_write_** chỉ bằng 1 lần gọi.  
- Còn 1 cách nữa, đó là thay đổi thông tin **_inode_** để yêu cầu hệ thống file tự đồng bộ giúp. Trong ngôn ngữ C, nó tương được với việc sử dụng **FS_IOC_SETFLAGS** khi gọi **_ioctl()_**.  
**_mkfs.ubifs_** sẽ kiểm tra cờ "sync" trong FS tree, các file có thuộc tính đồng bộ sẽ được đồng bộ.

**Chúng ta có thể tin rằng, check list trên đúng với cả các hệ thống file Linux khác nữa, bao gồm cả JFFS2**

Chi tiết hơn về **fsync(), fdatasync()**.

**fsync()**: Có thể sử dụng được cả với **directory**, tức là sẽ thực hiện đồng bộ **_inode meta-data_** liên quan đến directory. Khi một directory được set cờ "sync" thì tất cả các directory con của nó cũng kế thừa cờ này, kể cả file, directory mới. Khả năng này sẽ rất hữu dụng khi tạo một directory tree (ví dụ /etc).

**fdatasync()**: khi sử dụng với directory thì không có tác dụng gì thêm thì UBIFS nếu có đồng bộ cho directory thì đồng bộ cả các entry chứ không riêng gì dữ liệu. Và cả cờ "dirsync" cũng không có tác dụng gì hết.

Cả hai hàm này làm việc với các file descritor (fd - kiểu INT), chứ không phải các Stream, FILE* (con trỏ FILE). Để thực hiện đồng bộ cho các Stream (con trỏ FILE), đầu tiên cần xác định file descriptor của nó bằng **_fileno()_**, rồi đây hết dữ liệu ra stream bằng **_flush()_**, cuối cùng gọi **fsync()**, hoặc **fdatasync()**. **_flush()_** ở đây có được hiểu là đồng bộ ở mức độ buffer của ***libc**, còn **fsync()**, **fdatasync()** là đồng bộ ở mức kernel.

## 11. Các thông số liên quan đến write-back trên

Linux cung cấp một vài điểm trung gian giữa user-space và kernel space ở "/proc/sys/vm", cho phép tinh chỉnh các thông số liên quan đến write-back. Khi thay đổi thông số ở các điểm nối này, nó sẽ ảnh hưởng đến toàn bộ hệ điều hành. Chi tiết có tại "Documentation/sysctl/vm.txt" trong source của kernel. Dươi đây là một số điểm đáng chú ý:  
- **dirty_writeback_centisecs**: Chu kì cho để đánh thức thread chịu trách nhiệm thực hiện đồng bộ dữ liệu ra thiết bị nhớ.  
- **dirty_expire_centisecs** : thời gian tối đa để dữ liệu nằm trong buffer đến nó được thực sự ghi vào thiết bị nhớ bở thread write-back. Tức là thread write-back sẽ được đánh thức sau mỗi **dirty_writeback_centisecs**, và sẽ ghi những dữ liệu nằm trong buffer **dirty_expire_centisecs** trước đó. Đơn vị là centi-seconds.  
- **dirty_background_ratio** : Tỉ lệ bộ nhớ dùng để chưa dữ liệu tạm trên tổng số bộ nhớ (RAM) hiện có. Nếu lượng dữ liệu tạm vượt qua con số này, thì thread write-back sẽ được đánh thức để thực hiện ghi dữ liệu tạm vào thiết bị nhớ cho đến khi chiếm đủ nhỏ không gian tổng bộ nhớ mới thôi. Kể các các dữ liệu chưa nằm đủ lâu cũng bị ghi sang.  
- **dirty_ratio** : lượng dữ liệu tối đa buffer có thể chứa. Nếu đến giới hạn thì dữ liệu phải được đồng bộ trước khi thêm bất cứ dữ liệu tạm nào.

Chú ý, UBIFS có một write-buffer nhỏ được đồng bộ cứ mỗi 3-5 giây. Điều này có nghĩa là hầu như tất cả các dữ liệu "dirty" sẽ bị được giữ khoảng **dirty_expire_centisecs** centi-seconds. Nhưng một vài KiB sẽ bị giữ lại 3-5 giây.

## 12. UBIFS write-buffer

UBIFS là hệ thống file không đồng bộ, điều ấy quá rõ rồi. Giống như bao hệ thống file khác trong Linux, nó cũng sử dụng page cache. Nó nằm trong cơ chế quản lý bộ nhớ cơ bản của Linux rồi. Các page cache có thể có kích thước rất lớn. Bạn ghi ram file chẳng hạn, dữ liệu sẽ được đưa vào page cache, được đánh dấu là "dirty", câu lệnh **_write()_** sẽ trả về ngay (trừ khi file được mở với đặc tính đồng bộ). Sau đó, dữ liệu mới thực sự được ghi vào thiết bị.

Write-buffer là một buffer được thêm bởi UBIFS, nó đứng trung gian giữa page cache và flash. **Tức là write-back thread ghi dữ liệu đến write-buffer chứ không phải đến trực tiếp flash.** É hé, vãi~~.

Mục đích đưa ra write-buffer là nhằm tăng tốc độ ghi trên thiết bị NAND flash. Thông thường, NAND Flash chứa các NAND Pages, thường có kích thước 512, 2KiB, 4KiB. NAND page là đơn vị đọc/ghi nhỏ nhất của NAND Flash. (Á há.)

Kích thước của write-buffer sẽ tương ứng với kích thước NAND page (nó thường rất nhỏ so với Page cache). Mục đích của nó là tích trữ những thao tác ghi nhỏ để tạo thành một số nguyên lần NAND Page thay vì ghi từng phần. Quả thực, bạn có thể tưởng tự như sau, nếu có 4 thao tác ghi, mỗi cái 512 byte, mỗi thao tác cách nhau 1 giây, và ta có NAND page kích thước 2KiB. Khi đó, nếu không sử dụng write-buffer thì bạn cần ghi tới 4 NAND Page (tức là 8KiB), sẽ phí mất 6KiB. Còn nếu sử dụng write-buffer, bạn có khả năng tiết kịp 6KiB này.  
Tất nhiên, ví dụ vừa rồi là một trường hợp lý tưởng thôi, một số trường hợp chúng sẽ chẳng thu lại được thêm tí hiệu quả nào như I/O đồng bộ chẳng hạn hoặc dữ liệu giữa 2 lần ghi quá lâu. Để xử lý cái đó, write-buffer được gắn 1 cái timer, 3-5 giây, nó sẽ đẩy hết dữ liệu xuống flash ngay cả khi nó chưa đầy (một số nguyên lần NAND Page). Chúng ta phải làm điều đó để đảm bảo tính bảo toàn của dữ liệu.  
Một vài note liên quan đến đồng bộ và các hàm đã nói ở trên:  
- **"sync()"** : Đồng bộ toàn bộ dữ liệu trong write-buffer.  
- **"fsync(fd)"** : Đồng bộ toàn bộ dữ liệu của **fd** trong write-buffer.  
- File **synchronous** được mở với **0_SYNC** không bị ảnh hưởng bởi write-buffer.  
- write-buffer cũng bị vô hiệu hóa nếu file system được mounted với tham số **"-o sync"**

Từ đó, ta có thể suy ra rằng, dữ liệu dù đã được đánh dấu phải ghi vào flash thì nó vẫn có thể bị giữ lại 3-5 giây trước khi vào flash thực sự.

## 13. So sánh mode đồng bộ trong UBIFS với hoạt động của JFFS2

Khi UBIFS được mount ở mode đồng bộ (**-o sync**), thì tất cả các thao tác sau đó đó đều là đồng bộ. Điều đó đồng nghĩa rằng, dữ liệu sẽ được ghi vào flash trước khi các hàm **_write()_** trả về.

Ví dụ, nếu ghi một file 10MiB dữ liệu vào file f.dat sử dụng hàm **_write()_** và UBIFS đã được mount trong mode đồng bộ, thì UBIFS UBIFS sẽ đảm bảo rằng cả 10MiB đó + phần meta-data (kích thước, date) sẽ được ghi vào flash trước khi hàm **_write()_** trả về. Ngay sau đó, có tắt nguồn đi nữa, thì dữ liệu vẫn còn lại trong bộ nhớ flash. (Chắc chắn lắm!!!)

Ta cũng có được điều tương tự khi bản thân file f.dat được mở với option **O_SYNC** hoặc có thuộc tính **sync** được bật trong meta-data.

Có một hệ thống file khác mà bản thân thiết kế của nó đã gần như đồng bộ rồi (không tính đến một write-buffer khá nhỏ), đó là **JFFS2**. Nghe có vẻ chế độ đồng bộ của UBIFS cũng giống như thế, nhưng thực sự không phải như vậy. Có thể nghĩ như này, JFFS2 vốn được thiết kế đã vậy, nên sự đảm bảo của nó ở chế độ đồng bộ sẽ tốt hơn UBIFS ở mode đồng bộ.

Trong JFFS2, tất cả meta-data (như **_atime/mtime/ctime, inode size, UID/ GUID_**) đều được lưu ở phần header của data node. Mỗi data node chưa đúng 4KiB (dữ liệu đã được nén). Điều đó có nghĩa rằng, thông tin meta-data lặp lại ở rất nhiều nơi. Mỗi lần ghi dữ liệu, nó cũng update cả inode size nữa. Khi JFFS2 được mount, nó sẽ scan toàn bộ media, đê tìm data node gần đây nhất rồi lấy kích thước inode từ đó.

Trong thực tế, JFFS2 sẽ ghi theo thứ tự từ đầu đến cuỗi đoạn dữ liệu. Khi xảy ra mất nguồn trong lúc đang ghi, phần dữ liệu phía sau sẽ bị mất.

Khi nói về data và meta-data, có 1 sự khác nhau nhỏ giữa UBIFS và JFFS2 là, UBIFS lưu node data và meta-data ở các vùng riêng, còn JFFS2 thì lưu chung. Và UBIFS không lặp lại các meta-data ở nhiều nơi. Một điều nữa muốn nói đến là thứ tự ghi meta-data và data, cái nào được ghi xuống trước. UBIFS không cho phép ở bất kì thời điểm nào, cái data được lớn hơn meta-data. Tức là khi phải ghi dữ liệu data mới có kích thước lớn hơn thì nó phải ghi meta-data xuống trước rồi mới ghi phần node data xuống. Vì thế, có thể có trường hợp, đang ghi data mà bị đứt đoạn chẳng hạn, thì bạn sẽ có 1 lỗ trống dữ liệu mà không thể dùng lại được.  
Dưới đây là 1 ví dụ:  
- User tạo 1 file rỗng là **f.dat**. Là file đồng bộ hoặc UBIFS đã được mount ở mode đồng bộ. Gọi hàm **_write()_** cho 10MiB buffer.  
- Kernel sẽ copy 10MiB đó vào page cache. Kích thước của Inode cho file sẽ được thay thành 10MiB (vẫn trên memory), và được đánh dấu là "dirty". Đến đây, vẫn chưa có gì được ghi vào flash, nếu tắt nguồn ngay sau đó thì file **f.dat** vẫn rỗng.  
- UBIFS thấy nó cần thực hiện đồng bộ. Nó bắt đầu quá trình đó theo thứ tự sau. Đầu tiên, ghi thông tin inode xuống trước, từ là inode đó giờ có kích thước 10MiB. Nếu nguồn bị mất ở ngay sau điểm này, thì user sẽ có 1 file được ghi là 10Mib những dữ liệu toàn zero vì có vùng dữ liệu đâu.  
- UBIFS bắt đầu ghi dữ liệu xuống. Nếu nguồn bị cắt khi ở điểm nào đó trong lúc ghi thì phần còn lại của dữ liệu coi như bị mất.  
Trên đây là các tình huống khi sử dụng mode đồng bộ.

Chú ý rằng, với mode không đồng bộ, ta vẫn gặp vấn đề tương tự khi thread write-back đang thực hiện mà bị ngắt giữa chừng.

Vì thế, I/O đồng bộ trong UBIFS kém đảm bảo hơn JFFS2.  
Một ứng dụng lý tưởng nên xem xét đến cả trường hợp nội dung của file mong muốn bị đứt đoạn khi ghi xuống trước đó. Cả **ext3** cũng không thể đảm bảo giống như JFFS2 được.

Dù vậy, nhưng trong một số trường hợp UBIFS vẫn phải đóng vai để thực hiện các chức năng của JFFS2. Nó có thể làm được như thế nhưng cần một chút công sức cho việc này và hiện tại nó chưa được implement trên open source của nó. Bạn có thể tự làm nó, hoặc nhờ tác giả nếu thực sự cần nó cho ứng dụng của bạn.

## 14. Đồng bộ trong một số ứng dụng viết không cẩn thận.

Như đã miêu tả chương 10,UBIFS là hệ thống file không đồng bộ, các ứng dụng yêu cầu đồng bộ cho các file mà nó sử dụng khi cần thiết. Một số hệ thống file trên Linux như XFS cũng yêu cầu nên như thế.

Tuy nhiên, rất nhiều ứng dụng bỏ qua hoặc không thực hiện việc này một cách cẩn thận. Dẫn đến có một cuộc tranh luận rất lớn giữa developer phía user-space và kernel về tính năng "cấp phát trì hoãn" (delayed allocation). Bạn có thể xem thêm tại [Theodore Tso's blog post](http://thunk.org/tytso/blog/2009/03/12/delayed-allocation-and-the-zero-length-file-problem/). Thông tin chi tiết có thể tìm thấy ở [LWN article](http://lwn.net/Articles/326471/)

Tóm lại, tranh luận nổ ra xung quanh 2 trường hợp. Thứ nhất là về [atomic re-name (đổi tên)](http://www.linux-mtd.infradead.org/faq/ubifs.html#L_atomic_change), các chương trình phía user-space không đồng bộ bản copy trước khi đổi tên nó (tức là đổi tên 1 file mà chưa thực sự có trên thiết bị nhớ??!!). Thứ hai, một số ứng dụng xóa nội dung file rồi thay đổi nội dung file đó. Vẫn chưa có 1 thỏa thuận cuối cùng. Nhưng các developer ext4 phàn nàn răng "chúng ta không thể bỏ qua thế giới thực được", và đã có 2 thay đổi trong ext4 để giải quyết vấn đề này.

Nói đại khái như này, thay đổi đầu tiên là đồng bộ file khi chúng được đóng nếu trước đó chúng bị xóa hết nội dung bên trong (TRUNCATE). Đây là một "hack" nếu nhìn từ phương diện thiết kế hệ thống file, nó hướng đến các nhóm ứng dụng mà mở file, xóa nội dung, rồi thêm mới nhưng không thèm yêu cầu hệ thông file đồng bộ (!!??). Thay đổi thứ hai, đó là đồng bộ ngay khi file bị đổi tên.

Ồ, đến đây, dù có thể không chính xác. Nhưng ta có thể đoán rằng, vì ext4 không ghi file một cách đồng bộ, và chúng thực sự sử dụng write-back nên khả năng dính chưởng lên quan đến hiệu năng không quá cao. Chỉ riêng với trường hợp file bị "truncate" thì sẽ được đồng bộ ngay sau khi nó được đóng. Và trường hợp đổi tên file, thì ext4 sẽ ghi nội dung trước khi thay đổi meta-data của file đó.

Tuy nhiên, người viết ứng dụng không bao giờ nên dựa dẫm vào những thay đổi này, vì nó không "portable" đâu. Hãy thực hiện đồng bộ một cách cẩn thận, ext thực hiện những thay đổi trên vì có nhiều ứng dụng bị ảnh hướng quá thôi.

Chúng ta cũng nên có kết hoạch để có những tính năng trên trong UBIFS, tuy nhiên hiên hiện tại thì chưa. Tất nhiên, bất kì ai cũng có thể làm nó.

## 15. Compression (Nén)

UBIFS hỗ trợ nén dữ liệu bên trong nó, có nghĩa là dữ liệu được nén trước khi ghi xuống thiết bị nhớ, và được giải nén sau khi đọc lên. Việc nén này hoàn toàn trong suốt với người dùng. Dữ liệu được nén chỉ là những file thông thường thôi. Các node device, thư mục không được nén. Meta-data và các thông tin index cũng không được nén.

Đến thời điểm hiện tại, UBIFS hỗ trợ **LZO** và **zlib**. **Zlib** có tỉ lệ nén tốt hơn, nhưng **LZO** thì cho tốc độ nén/giải nén cao hơn. LZO được sử dụng mặc định trong câu lệnh tạo ảnh hệ thống file **_mkfs.ubifs_**. Tất nhiên, bạn có thể tắt tính năng nén bằng tham số "-x".

UBIFS sẽ chia dữ liệu thành cụp 4KiB và nén từng cụm độc lập. 4KiB, chưa hẳn là tối ưu vì dữ liệu ban đầu lớn thì kích thước từng cụm càng lớn thì sẽ càng nhanh, nhưng nó sẽ có lợi khi sử dụng không gian nhớ flash. Ví dụ, một file-system thực tế cho ARM Platform sẽ có kích thước giảm đến 40% khi dùng LZO, 50% khi dùng với Zlib. Điều này có nghĩa là, nếu bạn đặt 1 rootfs image vào hệ thống file UBIFS có dung lượng 256MiB thì bạn có thể sẽ vẫn có khoảng 100MB vùng trống dành cho việc khác (nghe hay đấy chứ!!!). Tuy nhiên, nó chỉ là có thể, vì nếu nội dung bạn đưa vào UBIFS không thuộc loại dữ liệu có thể nén được tốt ví dụ như mp3 chẳng hạn thì có thể ta không có được lượng dữ liệu trống như mong đợi.

Trong UBIFS, có thể bật/tắt việc nén dữ liệu cho từng inode bằng cách set/clear cờ compression. Cờ này sẽ được kế thừa từ inode cha. Có nghĩa là mọi file được tạo dưới directory nào đó, sẽ có trạng thái compression giống với directory chứa nó. Hãy xem thêm ở [đây](http://www.linux-mtd.infradead.org/faq/ubifs.html#L_comproff) để biết thêm chi tiết.

Cũng có nhiều ví dụ kết hợp của LZO và Zlib nữa, xem thêm phần hỏi đáp tại [đây](http://www.linux-mtd.infradead.org/faq/ubifs.html#L_favor_lzo).

Có một sự khác nhau nhỏ giữa JFFS2 trong việc nén dữ liệu. UBIFS thì sử dụng "crypto-API deflat", trong khi đó JFFS2 sử dụng trực tiếp zlib. Đấn đến, dùng cùng sư dụng zlib nhưng JFFS2 sử dụng deflat level 3, windows bit 15, còn UBIFS thì sử dụng level 6 và windows bit là -11 (sử dụng dấu -, để làm cho zlib không tạo header lên dữ liệu đầu ra. Qua thực nghiệm, JFFS2 có tỉ lệ nén thấp hơn 1 chút, tốc đọc giải nén cũng chậm hơn 1 chút, nhưng tốc độ nén thì cao hơn.

## 16. Checksumming

Bất cứ thông tin nào được UBIFS khi xuống thiết bị nhớ đều có CRC-32 checsum. UBIFS bảo vệ cả dữ liệu và meta-data bằng CRC. Mỗi khi meta-data được đọc, CRC phải được verified. CRC-32 khá mạnh trong việc kiểm tra dữ liệu dạng này. UBI subsystem cũng vậy nó verified mỗi meta-data.

Mặc định, thì dữ liệu CRC sẽ không được verified nhằm tăng tốc độ đọc của hệ thống file. Nhưng UBIFS thì cho thực hiện việc verification này khi sư dụng tham số **_chk_data_crc_** lúc mount. Nó sẽ làm giảm tốc độ đọc xuống 1 chút nhưng tăng tính bảo toàn dữ liệu. Khi sử dụng đến mức verified này thì bạn hoàn toàn có thể yên tầm là bất cứ thông tin nào được đọc lên từ UBIFS sẽ được verified và mọi hỏng hóc đều được phát hiện.

Chú ý, hiện tại UBIFS không cho phép tắt CRC-32 khi ghi dữ liệu, bởi vì quá trình Recovery của UBIFS phụ thuộc vào nó. Quá trình Recovery sẽ đọc dữ liệu không "sạch" từ lần tắt cuối và "replay" lại nhật kí, UBIFS sẽ phải phát hiện các hỏng hóc, các file đâng ghi dở để loại bỏ chúng, và ở đây nó phụ thuộc vào CRC-32.

Nói một cách khác, cho dù có "tắt" kiểm tra dữ liệu CRC đi nữa, thì bạn vẫn có CRC-32 checksum cho mỗi đoạn dữ liệu. Bất cứ khi nào bạn cảm thấy hệ thống file có dấu hiệu bị hỏng đâu đó, bạn có thể bật tham số **_chk_data_crc_** lên.

Chú ý quan trong: các phiên bản trước 2.6.39 sử dụng check dữ liệu CRC-32 mặc đinh và sử dụng tham số **_no_chk_data_crc_** khi muốn tắt nó.

## 17. Read-ahead (Đọc trước)

Là kĩ thuật tối ưu hóa, khi yêu cầu hệ thống file được nhiều hơn số dữ liệu user mong muốn. Ý tưỡng xuất phát từ việc đọc 1 file thường theo 1 mạch từ đầu đến cuối chẳng hạn. Ví thế, khi bắt đầu đọc, file system sẽ cố gắng đọc dữ liệu tiếp theo trước khi user thực sự yêu cầu chúng.

Linux VFS cung cấp khả năng "đọc trước" cho ứng dụng mà không phụ thuộc vào hệ thống file nó sử dụng bên dưới. Nó làm việc với hầu hết các file system dựa tên block kiểu truyền thống. Tuy nhiên, hiện tại chưa làm việc tốt được với UBIFS. UBIFS làm việc dựa trên UBI API (từ UBI Subsystem), bên dưới nữa là MTD API - là các hàm chạy đồng bộ. MTD đơn giản chỉ thực hiện việc thao tác với Flash media, nó không có hàng đợi request nào hết. Có nghĩa là VSF sẽ block các ứng dụng sử dụng UBIFS và yêu cầu chúng đợi tiến trình đọc trước (read-ahead process). Ngược lại, các API dành cho Block-device thường là không đồng bộ, mỗi lời gọi không cần phải đợi.

VFS read-ahead được thiết kết cho Hard Drives, nó phù hợp với Hard-drive hơn. Nhưng thiết bị nhớ raw flash rất khác so với Hard Drive. Nó không cần một thời gian điểm tìm điểm đọc "seek time" như Hard Drive làm vì thế những kĩ thuật như thế này không cần cho flash.

Có thể nói rằng, VFS sẽ làm cho UBIFS trở lên chậm hơn thay vì cải tiến tốc độ của nó. Ví thế, UBIFS tắt VFS Read-ahead. Nhưng bên trong UBIFS có 1 cơ chế gần giống với "read-ahead", được gọi là "bulk-read". Bạn cũng thể sử dụng nó bằng cách thêm tham số **_bulk_read_** khi mount.

Một vài thiết bị nhớ flash cho phép đọc dữ liệu nhanh hơn nếu trước dữ liệu đó được đọc trước đó 1 hoặc 1 vài lần. Ví dụ, OneNAND có thể "read-whilte-load" nếu nó đọc nhiều trang NAND. Vì UBIFS có thể tận dụng khả năng này khi đọc một khối dữ liệu lớn, và đó chính xác là những gì "bulk-read" làm.

Nếu UBIFS thấy một file được đọc qua block liên tiếp (ít nhất có 3 block 4KiB liên tiếp nhau được đọc), và các block liên tiếp này nằm cùng 1 "block" trên flash chẳng hạn, nó sẽ đọc trước một lượng dữ liệu, để sau đó được quẳng vào file cache để khi có request có thể trả về ngay.

Đây là 1 ví dụ cụ thể, giả sử user bắt đầu đọc 1 file từ đầu đến cuối. Giả định rằng, file đó không bị phân mảnh trên flash media. Giả sử LED25 chứa các node data của file này, các node logical, và thật may các node này nằm liên tiếp nhau trên flash media. Giả sử dụng user yêu cầu đọc LED25 offset 0. Khi đó, UBIFS sẽ đọc toàn bộ các data node có ở LED25, và đẩy chúng vào file cache. Từ đó, các đoạn dữ liệu tiếp theo của file sẽ có sẵn trên file cache rồi nên các thao tác đọc sẽ rất nhanh.

Lại nói về "bulk-read", khi hệ thống file bị phân mảnh quá nhiều trên thiết bị nhớ, thì bulk-read có thể làm giảm hiệu năng. Mặc dù bản thân UBIFS không làm phân mảnh hệ thống file, nhưng UBIFS cũng không cố gắng sắp xếp lại. Ví dụ, bạn ghi 1 file tại 1 thời điểm, thì nó sẽ không bị phân mảnh. Nhưng nếu ghi nhiều file cùng lúc, thì có thể nó sẽ bị phân mảnh đấy (nó phụ thuộc vào hoạt động của write-back thread khi đồng bộ các thay đổi), và UBIFS cũng không tự động sắp xếp lại các file bị phân mảnh đâu. Tuy nhiên, điều này vẫn có thể thực hiện được trong tương lai nhưng làm 1 cái chạy sắp xếp lại ở background chẳng hạn. Ví dụ như có thêm một thông tin nhật kí tại mỗi inode để tránh các cho data khác inode bị xen kẽ vào nhau chẳng hạn. Vâng, đó là những thứ sẽ được cải tiến, có thể là bạn sẽ làm nó.

## 18. Space cho superuser

UBIFS có một vài vùng trống dành cho superuser (root), có nghĩa là ngay cả khi với normal-user thì hệ thống file đã đầy, thì vẫn có chỗ cho superuser. Ext2 cũng có khả năng tương tự. Khi sử dụng **_mkfs.ubifs_**, chỉ cần thêm **-R** là được.

Hiện tại,thì chỉ root mới có thế sử dụng vùng dành riêng này. Nhưng UBIFS có khả năng giới hạn một nhóm user nào đó bằng cách giữ thông tin của các user đó và thực hiện kiểm tra khi truy cập chẳng hạn. Tuy nhiên, **_mkfs.ubifs_** chưa có option nào cho việc này, và nó không quá khó để thực hiện nó.

Một chú ý nữa, khi mount, UBIFS sẽ in ra thông tin về kích thước vùng trống này.

## 19. Các thuộc tính mở rộng

UBIFS hiện tại đã hỗ trợ các thuộc tính mở rộng rồi, nó có thể được sử dụng khi cấu hình tương ứng được bật (không cần thêm tham số khi mount). Nó hỗ trợ cho việc sử dụng **user**, **trusted**, **security**. Tuy nhiên, chưa hỗ trợ **ACL** (Access Control List).

Chú ý, hiện tại **_mkfs.ubifs_** bỏ qua thuộc tính mở rộng, nó không ghi chúng vào file ảnh hệ thống file được tạo ra.

## 20. Các options khi mount

Dưới đây là một vài tham số khi mount dành riêng cho UBIFS:  
- chk_data_crc (mặc định) - kiểm tra phần CRC-32.  
- no_chk_data_crc - không kiểm tra phần dữ liệu CRC-32  
- bulk_read - Cho phép sử dụng bulk-read (đọc khối lớn)  
- no_builk_read - Không cho phép sử dụng bulk-read (đọc theo khối lớn)

Ví dụ:

```bash  
$ mount -o no_chk_data_crc /dev/ubi0_0 /mnt/ubifs  
```

Mount hệ thống file UBIFS vào điểm **_/mnt/ubifs_**, không kiểm tra CRC data.  
Ngoài ra, UBIFS còn có mode đồng bộ nữa. Nó sẽ hoạt động mà không có writeback, write-buffer. Chú ý rằng, **atime** không được UBIFS hỗ trợ.

## 21. Vấn đề về tính toán không khoảng trống trên Flash

Những hệ thống file truyền thống khác như ext2 chẳng hạn, dễ dàng tính toán kích thước vùng trống. Việc tính toán đó khá chính xác, và users cũng quen với các con số đó rồi. Tuy nhiên, trong tình huống của UBIFS, nó hoàn toàn không giống như thế. Nó không thể tính toán hính xác lượng vùng trống còn lại được, nên nó sẽ làm khó users. Thay vào đó, nó đưa ra con số nhỏ nhất (**_minimum_**), thừng ít hơn thực tế. Thỉnh thoảng còn sai lệch nhiều nữa. Ví dụ, UBIFS trả về kết quả (thông qua lời gọi **_statfs()_**) rằng không có vùng trống nữa, nhưng nó vẫn có thể ghi thêm được kha khá. Vãi~~~

Hay nói về sự khác biệt này, UBIFS thường "nói dối" về lượng vùng trống nó có. Dù nó có thể đưa ra những con số tin lơn hơn, nhưng thường không bao giờ báo nhiều hơn nó có và rất hiếm khi báo chính xác được. Dẫn đến điều này không phải vì tác giả của UBIFS "quá ngốc" mà là có những lý do khác nữa. Xin xem tiếp phần dưới đây.

### Ảnh hưởng của việc nén dữ liệu

Như ta đã biết, UBIFS sử dụng nén dữ liệu bên trong nó (LZO và ZLIB). Phía user thì lại thường mong muốn hệ thống file bao cáo rằng nó còn bao nhiêu byte, hơn là có thể tạo được file kích thước bao nhiêu nữa. Vì được nén nên, vùng trống có thể lưu trữ được bao nhiêu dữ liệu của người dùng sẽ phụ thuộc vào tỉ lệ nén mà UBIFS thực hiện trên dữ liệu cụ thể (chứ không cố định).

Khi tính toán vùng trống còn lại, nó không biết trước được là user sẽ ghi những dữ liệu gì sau đó. Nên nó không thể lấy 1 tỉ lệ nén cố định được, và giả định trường hợp xấu nhất là dữ liệu không được nén.

Ồ, chỉ như trên thôi thì cũng không có gì to tát. Tuy nhiên, việc nén này sẽ trở thành vấn đề khi tính toán vùng trống nếu như nó hoạt động với write-back thread. Nói chính xác hơn, UBIFS không biết được dữ liệu "dirty" được nén như thế nào ngoài cách thực sự nén nó mà thôi.

### Ảnh hưởng của việc write-back

Giả sử tình huống như thế này, có X bytes dữ liệu "dirty" trong page cache. Chúng sẽ được đẩy ra thiết bị flash thôi, nhưng đến lúc đó thì vẫn nằm trên RAM. UBIFS sẽ cần **_X + O_** bytes trên thiết bị flash để ghi dữ liệu này, O ở đây là dữ liệu overhead (như index, các header, etc).

Vấn đề ở đây là, UBIFS không thể tính chính xác X và O được, và nó "bi quan" rằng sẽ là tình huống xấu nhất. Bởi vậy, khi thực sự đẩy ra ngoài flash, nó có thể dùng số vùng trống ít hơn đã giả định. Dưới đây là 1 ví dụ như thế:

```bash  
$ df  
Filesystem 1K-blocks Used Available Use% Mounted on  
ubi0:ubifs 49568 49568 0 100% /mnt/ubifs  
$ sync  
$ df  
Filesystem 1K-blocks Used Available Use% Mounted on  
ubi0:ubifs 49568 39164 7428 85% /mnt/ubifs  
```

Đầu tiên, **df** báo không còn vùng trống nữa, nhưng sau lệnh **sync**, nó lại báo còn **15%**, (éo đỡ được). Vì có rất nhiều dữ liệu "dirty" trong cache, nên UBIFS đã giả định là cần từng đó vùng trống trên Flash. Nhưng khi thực sự đẩy ra (vì có cả nén) nên nó không chiếm nhiều đến thế.

Dưới đây là một số lý do chính giải thích tại sao UBIFS muốn nhiều vùng trống hơn đến lúc nó thực sự cần:  
- 1 trong những lý do đó là có nén dữ liệu. Dữ liệu trước khi được nén sẽ nằm trong cache, UBIFS biết làm thế nào chúng được nén và lấy tổng dữ liệu chưa nén đó làm kích thước dự kiến. Tuy nhiên, dự liệu thực tế được nén lại với tỉ lệ khá tốt (từ những dữ liệu vốn đã nén rồi như file **.tar.gz**, **.mp3**). Điều này dẫn đến thực tế dữ liệu sau khi nén nhỏ hơn dữ liệu dự kiến rất nhiều.  
- Xuất phát từ thiết kế của UBIFS, không bao giờ "cross logical eraseblock", tức là 1 block chỉ có thể thuộc 1 node thôi. Vì thế, luôn có một vùng nhỏ không được sử dụng ở cuối block. Lượng vùng trống không thể sử dụng này sẽ phụ thuộc vào thứ tự data được ghi hoặc thay đổi. Và như thường lệ, UBIFS giả định "bi quan" rằng, sẽ có số vùng không dùng được nhiều nhất. Điều đó là một nguyên nhân nữa dấn đến ước tính vùng trống bị sai quá nhiều so với thực tế.

À, tất nhiên, UBIFS sẽ đưa ra kết quả chính xác hơn rất nhiều nếu nó đã được thực hiện đồng bộ.

### Hao hụt (Wastage)

Như đã nói ở trên, UBIFS không bao giờ "cross logical eraseblock". Hãy xem xét những con số dưới đây.  
- kích thước lớn nhất của UBIFS node (chưa nén) : 4256 bytes.  
- kích thước nhỏ nhất của UBIFS node (phần dữ liệu là 8 bytes) : 56 bytes  
- phụ thuộc vào độ dài tên file, các directory entry node : 56-304 bytes.  
- một LED thông thường có kích thước 126KiB với NAND Flash, 128 KiB eraseblock vật lý và 2048 NAND page (hoặc 124KiB nếu NAND chịp không hỗ trợ sub-pages)

Vì thế, với những trường hợp là tất cả đều có kích thước lớn nhất 4256 byte, thì UBIFS sẽ phí mất **((126*1024) MOD 4256 )** bytes. Vì dữ liệu trog thực tế rất đa dạng nên, số byte bị lãng phí sẽ nằm trong khoảng từ 0-4255.

UBIFS đang cải tiến để đẩy các node kích thước nhỏ như directory entry vào cuối của LEB để giảm lượng không gian bị lãng phí, nhưng nó vẫn phải là lý tưởng và vấn còn rất nhiều vũng trống khá lớn mà không được sử dụng nằm ở phần cuối của eraseblock.

Khi được hỏi về vùng trống còn lại, UBIFS sẽ không biết loại dữ liệu nào sẽ được ghi vào vùng trống trong tương lai, Vì thế, nó "bi quan" giả định rằng sẽ bị lãng phí lớn nhất, tức là 4255 byte/1 LEB. Trong thực tế, thì nó lạc quan hơn, nên có thể đưa ra con số nhỏ hơn 4255 được. Tuy nhiên, UBIFS sẽ không đưa ra con số như thế, nó đưa ra con số nhỏ nhất của vùng trống mà ứng dụng phía user-space có thể biết.

Chính vì giải thích ở trên mà, với LEB kích thước càng lớn thì các con số UBIFS đưa ra càng chính xác.

### Vùng dirty

Dirt Space (Vùng bẩn) là vùng flash bị chiếm dụng bởi UBIFS. Từng ít nhất 1 lần được sử dụng nhưng do các thao tác thay đổi, xóa dữ liệu nên nó bị invalidated nên không còn được sử dụng nữa. Ví dụ, nội dung của một file bị thay đổi toàn bộ chẳng hạn, các node data của dữ liệu cũ sẽ bị invalidated, các node data mới sẽ được ghi vào flash. Các invalidated node sẽ tạo thành "dirty space", hay vùng bẩn. Còn có nhiều tình huống khác để sinh ra các vùng này nữa.

UBIFS không có cơ chế thực sự trong thiết kế để sử dụng các vùng này vì dùng dữ liệu tương ứng đó không phải là **0xFF** (trong Flash thì cho biết là chưa có dữ liệu). Để sử dụng lại được các vùng này, UBIFS phải thực hiện "dọn dẹp" các LEBs (garbage-collect). Ý tưởng sử dụng Garbage Collector là thu hồi "vùng bẩn" giống cách JFFS2 đang làm. Xem [tài liệu thiết kế của JFFS2](http://sources.redhat.com/jffs2/jffs2.pdf) để biết thêm chi tiết.

Đại khái là, bộ dọn dẹp (GC) của UBIFS sẽ tìm 1 LEB có node "dirt space", chuyển các node hợp lệ từ LEB đó sang 1 LEB mà GC đã chuẩn bị sẵn. Sau khi thực hiện xong, ta sẽ có một không gian trống ở phía cuối LEB mà GC đã chuẩn bị. GC lại tiếp tục tìm 1 LEB khác mà có node "dirt space", tiếp tục làm như thế. Khi LEB mà GC chuẩn bị đó đầy, UBIFS sẽ tìm 1 LEB rỗng, và đưa nó cho GC để tiếp tục công việc. Quá trình này kết thúc khi, không tìm được LEB có vùng "dirty space" nào nữa.

UBIFS có một khái niệm là đơn vị I/O nhỏ nhất (minimum I/O unit size), cái sẽ đặc trưng cho kích thước nhỏ nhất của dữ liệu được ghi xuống Flash (xem thêm tại [đây](http://www.linux-mtd.infradead.org/faq/ubi.html#L_find_min_io_size) ). Thông thường, UBIFS sẽ làm việc trên các Flash có NAND page lớn, kích thước I/O nhỏ nhất thường là 2KiB.

Trong tình huống mà GC tìm được một LEB có kích thước vùng "dirty space" nhỏ hơn đơn vị I/O, khi đó sau khi lấy chuyển các nodes từ LEB đó sang LEB mà GC trung gian mà GC đã chuẩn bị, thì UBIFS vẫn phải ghi một kích thước bằng kích thước I/O nhỏ nhất xuống Flash (vì đơn vị nhỏ nhất rồi mà), kết quả là chẳng thu dọn được gì.

Như vậy đấy, UBIFS GC có gắng để không lãng phí không gian lưu trữ, nhưng không phải lúc nào cũng làm được, GC thì chưa thể hoạt động lý tưởng được. UBIFS không phải lúc nào cũng thu hồi được vùng "dirty space" nếu kích thước vùng đó nhỏ hơn kích thước đơn vị I/O.

Khi được hỏi về kích thước không gian trống còn lại, UBIFS sẽ coi như vùng dirty là vùng có thể ghi được dữ liệu mới vì sau khi dọn dẹp, nó sẽ trở thành vùng trống mà. Nhưng như đã miêu tả ở trên đó, UBIFS không thể thu hồi tất cả vùng "dirty space" và chuyển nó sang free được. Trường hợp xấu nhất, là nó không biết chính xác bao nhiêu "dirty space" có thể thu hồi. Vì thế, nó vẫn sử dụng tính toán "bi quan".

Dung lượng lưu trữ càng lớn thì vùng "dirty space" càng phình ra, thông tin về vùng trống càng thiếu chính xác.

Muốn chạy GC, thì hay gọi **statfs()**. Tuy nhiên, điều này dẫn đến **statfs()** sẽ rất chậm. Hiện tại chưa có nhưng, có thể làm cho GC chạy ở background giống như JFFS2 đang làm để giảm thời gian thu hồi vùng "dirty space" hơn.

### Kích thước của index thì không xác định

Như đã biết, UBIFS lưu index của nó ở trên Flash. Cần 1 dụng lượng lưu trữ nhất định để chứa nó. Ngoài ra còn có cả UBIFS journal (dữ liệu nhật kí),