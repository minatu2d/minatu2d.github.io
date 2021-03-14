---
layout: post
title: Khái quát về FAT
date: 2015-12-27 13:39:16.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
description: FAT comprehensive overview
toc: true
authorbox: true
categories:
- Basic
tags:
- FAT
- FileSystem
meta:
  _wpcom_is_markdown: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '18156938026'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2015/12/27/khai-quat-ve-fat/"
---
Trong quá trình porting sang hệ điều hành ITRON-based (NORTi), đã có dịp tìm hiểu về FAT, thấy bài của anh ELMちゃん này đầy đủ những thứ mình muốn biết về FAT nên sẽ dịch lại cả bài ở đây.

Link gốc: [http://elm-chan.org/docs/fat.html](http://elm-chan.org/docs/fat.html)

Trong tài liệu này, về cơ bản là dựa trên tài liệu vể [FAT32 Spec](http://www.microsoft.com/whdc/system/platform/firmware/fatgen.mspx) , nhưng sẽ được giản luợc và bô sung thêm phần giải thích khái niệm mà bản gốc không có. Trong thực tế, các hệ thống chuẩn (DOS/Windows) cũng hoạt động không hoàn toàn giống với tài liệu Spec, tôi sẽ giải thích các khái niệm dựa trên hoạt động thực tế. Thêm một điều nữa là, trong tài liệu có thể có những nội dung theo ý cá nhân của tôi, nên khi triển khai hệ thống thực tế thì cần confirm chắc chắn yêu cầu thực tế rồi mới làm.

Mục lục

1.  Phần mở đầu
2.  Các thuật ngữ sử dụng bên trong tài liệu
3.  Cơ bản về hệ thống file FAT
4.  Boot Sector và BPB
5.  Công thức tính kích thước các vùng
6.  FAT và Cluster
7.  Thông số quyết định loại FAT
8.  Truy cập FAT Entry
9.  Sự liên quan giữa File và Sector
10.  Về cấu trúc FSInfo của FAT và Backup của Boot Sector
11.  Về cấu trúc Directory
12.  Các thao tác với Directory
13.  Định dạng nhãn thời gian
14.  Độ dài tên File
15.  Pham vi tên và Code page
16.  Matching giữa LFN và SFN
17.  Việc tạo SFN và việc thêm kí tự
18.  Về LFN Entry
19.  Tính tương thích
20.  Về việc chia các Partitions (về mặt vật lý)

# 1. Phần mở đầu

Hệ thống file tức là nói về hệ thống quản lý dữ liệu trên thiết bị nhớ bỏ sung. Có nhiều hệ thống File, những tài liệu này chỉ giới hạn nói về các cấu dữ liệu được lưu trữ của hệ thống file FAT.

Từ 1980, lần đầu tiên FAT được hỗ trợ trên MS-DOS. Ở thời điểm đó, nó được phát triển với mục tiêu tạo ra một hệ thống File phù hợp với ổ đĩa floppy 500KB trở xuống. Về sau, khi dung lượng lớn dần thì nó được mở rộng thêm. FAT nghĩa là File Allocation Table, hay nói cách khác muốn nói đến việc quản lý, cấp phát bảng này. Ngày nay, có 3 loại FAT được sử dụng: FAT12, FAT16, FAT32. Điểm khác nhau cơ bản nằm ở các kích thước FAT Entry (giống như kích thước của index của mảng 1 chiều), tương ứng với số Bit để biểu diễn mỗi index là 12, 16, 32. Và chúng tương thích ngược hoàn toàn, tức là hệ thống hỗ trợ FAT16 thì bao gồm cả FAT12, và hệ thống FAT32 thì bao gồm cả FAT12/16 nữa.

# 2. Các thuật ngữ sử dụng bên trong tài liệu

`0xACB..`  : bắt đầu nghĩa là biểu diễn kí tự Hexacimal( hệ 16), còn lại là kí tự Decimal (hệ 10).

`1K, 2K..` : K ở đây biểu diễn đơn vị bằng 1024 (hay `2^10`)

`1M, 2M..` : M ở đây biểu diễn đỡn vị bằng 1048576 (hay `2^20`)

`1G, 2G..` : G ở đây biểu diễn đơn vị bằng 103741824 (hay `2^30`)

`1T, 2T..` : T ở đây biểu diễn `2^40`.

Các đoạn code trong tài liệu mặc định viết bằng ngôn ngữ C, và có cú pháp không chặt chẽ. Có nhiều đoạn không mô tả rõ ràng giữa 32 bit và 16 bit. Giả định rằng lập trình viên hiểu rõ kích thước của các kiểu dữ liệu này cũng như các cách để tránh việc chuyển đổi kiểu dữ liệu. Thêm nữa, toàn bộ kiểu dữ liệu sử dụng là không dấu, tuyệt đối không sử dụng kiểu dữ liệu có dố để tránh những kết quả không như mong muốn.

# 3. Cơ bản về hệ thống file FAT

## 3.1 Dung lượng FAT

Tức là nói về dung lượng luân lý (ổ đĩa luân lý) của một hệ thống `FAT` hoàn chỉnh. Dung lượng luân lý này thường bao gồm 3 hoặc 4 thành phần. Các vùng được cấu tạo từ 1 hoặc nhiều Sector (định nghĩa phía dưới). Và được đặt theo thứ tự như sau:

1.  Vùng dữ trữ  (chứa dữ liệu các thành phần của ổ đĩa)
2.  Vùng `FAT` (bảng cấp phát vùng dữ liệu - giống như index của mảng)
3.  Vùng thư mục gốc (trong `FAT32` không có vùng này)
4.  Vùng dữ liệu ( chứa nội dung của File và thư mục, giống như giá trị các phần tử mảng)

## 3.2 Sector

Là đơn vị nhỏ nhất khi hệ thống file đọc, ghi từ thiết bị lưu trữ. Về kích thước, có 3 loại phổ biến được sử dụng là `512/1024/2048` byte. Các sector được đánh chỉ số từ đầu đến hết ổ đĩa luân lý, chỉ số bắt đầu là 0. Ổ đĩa luôn lý không phải luôn luôn bắt đầu từ đầu của thiết bị lưu trữ, tức là người ta có thể cài đặt `FAT` chỉ cho một vùng trên thiết bị thôi chẳng hạn. Vì thế, trong tài liệu này, khi nói đến Sector thì có nghĩa là nói tính từ Sector 0 của ổ đĩa luân lý. Khi nói về vị trí tuyệt đối trên thiết bị lưu trữ, thì sẽ là Sector vật lý.

## 3.3 Định dạng dữ liệu

Hệ thống `File FAT` ban đầu được phát triển cho x86 CPU sử dụng trong `IBM PC`. Vì điểm quan trọng này nên, toàn bộ dữ liệu được lưu trữ của hệ thống `File FAT` là `Litle-Endian`. Vì vậy trên một `Big-Endian CPU` khác, thì khi truy cập các cấu trúc `BPB/FAT/Directory` thì cần phải chuyển đổi Endian trước khi sử dụng tính toán. Truy cập các trường dữ liệu `16/32` bit thì không nhất thiết được `Aligment`. Cho dù CPU có là `Litle Endian` đi nữa vẫn có trường hợp truy cập các trường dữ liệu không được Aliment cho kết quả không đúng, vì thế cần truy cập theo đơn vị byte. Thêm một vấn đề nữa, là truy cập kiểu từng `WORD` chỉ có thẻ thực hiện trong môi trường hỗ trợ các tùy chọn `Packing`, nó sẽ dần đến code có portability rất kém. Chính vì 2 lý do trên, mà khi thao tác với `FAT`, dữ liệu sẽ được truy cập như là mảng byte 1 chiều, việc này khiên cho việc đọc code trở nên khó khăn hơn nhưng tăng tính `Portability` của code.

## 4. Boot Sector và BPB

- Trong FAT luân lý, cấu trúc dữ liệu quan trọng nhất đó là BPB (BIOS Parameter Block) vì tất cả các thông tin liên quan đến FAT luân lý đều nằm ở đây. BPB nằm bên trong Boot Sector. Cái Sector này có thể được gọi với tên khác như VBR (Volumne Boot Record) hoặc PBR (Private Boot Record), và là Sector đầu tiên trong ổ đĩa FAT luân lý.

- Vùng BPB này hay dùng để chứa những thông tin mở rộng khi hệ thống File FAT thêm các chức năng mới. Đó chính là lý do mà BPB này có nhiều tên gọi đến vậy. Ở MS-DOSv1.0 thì chưa có vùng BPB này. Do phiên bản đầu tiên chỉ dành cho đĩa mêm Floppy 5.25 inches, 2 mặt hoặc 1 mặt, và để phân biệt loại 1 mặt hay mặt, nó sẽ đọc 8 bit trong FAT[0] (tức là ngay sau Boot Sector).

- Trong phiên bản MS-DÓSv2.0, việc xác định trên sẽ thông qua việc đọc BPB trong Boot Sector, kể từ đó phương pháp sử dụng byte đầu tiên của FAT[0] không còn được sử dụng nữa. Hiện tại, tất cả FAT luân lý đều phải chứa trường này. Về sau BPP này được mở rộng thêm khá nhiều dẫn đến việc xác định các tham số của ổ đĩa FAT luân lý gây ra nhiều hiểu lầm hoặc không rõ ràng.

- Đầu tiên, do sử phổ biến của ổ đĩa cứng (Hard Disk - Dung lượng lớn hơn trước) nên việc sử dụng FAT12 gây ra nhiều hạn chế về hiệu xuất cũng như giới hạn số lượng File nó quản lý được. MS-DOS ra đời để hỗ trợ FAT16 (thay đổi lần 1). Tuy nhiên, không lâu sau đó, nó cũng gặp vấn đề về khả năng quản lý. Đó là kích thước index bảng FAT của FAT16 chỉ có 16 bit nên nó chỉ quản lý được tối đa 2^26 (65536) Sector (nếu 1 Sector là 512 thì size lớn nhất nó quản lý được là 32M Byte), vẫn ổn khi dung lượng ổ đĩa cứng tăng rất nhanh. Thế là MS-DÓSv3.31 ra đời, hỗ trợ FAT32 (32 bit index), ngoài ra nó cũng hỗ trợ FAT12 (kích thước tối đa 128M Byte - Sector kích thước 32Byte ), FAt16(kích thước tối đa 2G Byte - Sector kích thước 32 Byte ) (thay đổi lần 2).

- Cuối cùng, trong Windows 95 OSR2 thì FAT32 đã hỗ trợ đến những ổ cứng dung lượng lớn hơn 2G Byte rồi, khi mà thời điểm trước đó FAT16 chỉ hỗ trợ đến 2GB. Trong spec của FAT32, có nói sẽ hỗ trợ đến2T Byte( Sector là 512 Byte). Và đây là lần thay đổi cuối cùng. Sau đó, thì Microsoft có khuyến nghị người dùng chuyển sang  NTFSS hoặc exFAT cho những thiết bị lưu trữ lớn hơn 32GB.

- Trong bảng dưới đây, tôi sẽ liệt kê một số trường dữ liệu của BPB, nó sẽ bắt đầu bằn BPB_. Những trường bắt đầu bằng BS_ thì không liên quan đến BPB mà nó chỉ nằm trong Boot Sector.

- Trong phần BPB của ổ đĩa luân lý FAT32 từ trường đầu tiên đến trường BPB_TotSec32 nó giống với FAT12, FAT16. Từ trường đó về sau thì có sự khác biệt. Từ byte 36 trở đi, thì mỗi trường chỉ thuộc về 1 loại mà thôi (hoặc là FAT32, hoặc là FAT12/16. Các xác định loại FAT nào sẽ được trình bày bên dưới:

Tên

Offset

Size

Giải thích

BS_jmpBoot

0

3

Mã lệnh để nhảy đến boot trap。Trường này có 2 định dạng phổ biến sau;  
0xEB, 0x??, 0x90 (Short Jump+NOP)  
0xE9, 0x??, 0x?? (Near Jump)  
?? là một giá trị tùy ý phụ thuộc vào địa chỉ nó sẽ nhảy đến。Trường hợp trường này bị mất hoặc ko có giá trị đúng thì Windows sẽ không thể nhận ra ổ đĩa này.

BS_OEMName

3

8

"MSWIN4.1"là giá trị được khuyến nghị nên đặt ở đây。Ngoài ra "MSDOS5.0" cũng hay được sử dụng。Giá trị của trường này hay bị hiểu nhầm như sau. Trường này đơn giản chỉ là 1 cá dãy kí tự, là tên. Trên các hệ điều hành của Microsoft thì không để ý đến trường này. Nhưng có thể một lúc nào đó giá trị này được xét đến với FAT Driver. Vì thế việc set giá trị ở đây để giảm tối thiểu vấn đề không tương thích đó. Dù nói là set gì cũng được nhưng vẫn có thể xảy ra tình trạng không nhận dạng được ổ đĩa FAT luân lý. Về ý nghĩa, trường này đại khái là tên của hệ thống đã format hay đã tạo ra FAT luân lý thôi.

BPB_BytsPerSec

11

2

Kích thước của Sector. Giá trị hợp lệ là một trong những giá trị sau 512/1024/2048/4096. Microsoft sẽ support các giá trị này. Nhiều hệ thống chỉ support 512, nhưng không kiểm tra trường này có bằng 512 hay không, chỗ này để giảm vấn đề tương thích ta cần sử dụng đúng giá trị của Sector Size. Đơn giản chỉ để tránh những hiểu nhầm không đáng có. Giá trị này phải bằng với kích thước của Sector mà hệ thống đang dùng để lưu trữ.

BPB_SecPerClus

13

1

Số Sector cấp phát tối thiểu. Trong ổ đĩa FAT luân lý sử dụng một khái  niệm gọi là Cluster. Để chỉ một dãy liên tục từ 1 Sector trở nên. Số Sector chưa trong 1 Cluster phải là cơ số của 2. Tức là nó phải là 1 trong các số 1, 2, 4, ..., 128. Kích thước của Cluster tính theo Byte bằng (BPB_BytsPerSec * BPB_SecPerClus) và nó không thể lớn hơn 32K Byte được. Các phiên bản Window 98, NT.. về sau có sử dụng cluster có kích thước 64K, 128K, 256K lớn hơn 32KB cũ. Những FAT được tạo trên các hệ thống này có thể không thể được nhận ra trên các hệ thống cũ.

BPB_RsvdSecCnt

14

2

Số Sector chứa vùng dự trữ, trường này luôn hơn 0 (Vì ít nhất nó phải chưa BPB hay chính nó). Với FAT12/16 thì trường này chỉ cần 1 là đủ. Vì vùng dữ trữ của FAT12/16 chứa trong 1 Sector là đủ. Cũng có nhiều FAT Driver không kiểm tra trường này. Trên FAT32 trường này có giá trị 32. Microsoft support các giá trị từ 1 trở lên.

BPB_NumFATs

16

1

Số phân vùng FAT, phải được set là 2. Thực ra chỉ cần >= 1 là được nhưng thỉnh thoảng cũng có trường hợp phần mềm FAT hoặc FAT Driver của OS kiểm tra điều kiện bằng hay không chẳng hạn. Driver của Microsoft thì hỗ trợ cả giá trị khác 2 nhưng vẫn khuyến nghị không nên dùng giá trị khác 2.  
Giá trị chuẩn bằng 2 có ý nghĩa là FAT luân có một bản back up để khi xảy ra mất mát dữ liệu sẽ giảm khả năng mất dữ liệu. Ở các thiết bị không phải là đĩa như thẻ nhớ, flash thì vùng này không được sử dụng. Vì không có vùng backup nên ta set giá trị 1 cũng được. Có thể một lúc nào đó FAT Driver không nhận ra được cũng nên.

BPB_RootEntCnt

17

2

Trong FAT12/16 thì trường này được sử dụng để chỉ số lượng Directory trong Root Directory. Giá trị của trường này phải được set để sao kích thước của vùng Directory, tức là  BPB_RootEntCnt * 32 (kích thước mỗi Directory Entry) bằng số chưa BPB_BytsPerSec(kích thước 1 Sector) . Để tương thích được rộng nhất thì FAT16 phải sử dụng Sector size =512. Trên FAT32 thì trường này không được sử dụng và nó luôn phải được set là 0.

BPB_TotSec16

19

2

Tổng số sector của ổ đĩa luân lý (độ dài 16 bit là giá trị cũ)。Giá trị này là tổng số Sector chứa cả 4 vùng đã nêu ở phần trên. Ở FAT12/16 giá trị này mà lớn hơn 0x10000 thì trường này không được sử dụng và phải được set là 0. Tổng số Sector sẽ được lấy từ giá trị  BPB_TotSec32にbên dưới. Trong FAT32 thì trường này nhất định phải set là 0 (tức là không dùng).

BPB_Media

21

1

Giá trị ngăn cách cỗ định bên của đĩa cứng, giá trị cố định chuẩn là 0xF8. Với các đĩa không có ngăn cách thì giá trị này thường là  0xF0. Ngoài ra những giá trị dưới đây cũng được sử dụng 0xF0, 0xF8, 0xF9, 0xFA, 0xFB, 0xFC, 0xFD, 0xFE, 0xFF. Ngoài ra một điểm rất quan trọng khác nữa là giá trị này phải bằng với giá trị đặt ở 8 bit thấp của FAT[0]. Cái này xuất hiện từ MS-DOSv1.0, hiện tại thì không được sử dụng nữa.

BPB_FATSz16

22

2

Số Sector có trong 1 phân vùng FAT. Trường này chỉ sủ dụng được ở FAT12/FAT16 thôi. Trong FAT32 trường này nhất định phải set là 0 (không sử dụng) và trường BPB_FATSz32 sẽ được sử dụng thay thế. Kích thước bằng Sector của ổ đĩa luân lý sẽ bằng : giá trị trường này* BPB_NumFATs.

BPB_SecPerTrk

24

2

Số Sector có trong 1 Track (Rãnh đĩa). Trường này liên quan đến thị chi tiết về hình học của thiết bị lưu trữ. Sử dụng ở ổ đĩa máy IBM PC, ngoài ra thì nó chẳng có ý nghĩa gì nữa.

BPB_NumHeads

26

2

Số lượng Head (đầu đọc), giống với Track, trường này liên quan đến cấu tạo hình học của đĩa. Chỉ được sử dụng trong IBM PC, không còn được sử dụng nữa.

BPB_HiddSec

28

4

Số sector không thuộc ổ đĩa luân lý, tức là nằm phía trước địa chỉ bắt đầu cài đặt ổ đĩa luân lý. Nói chung, trong máy IBM PC, Phần này liên quan đến việc truy cập các thông tin BIOS được hệ thống lưu trữ lại. Trong trường hợp ổ luân lý bắt đầu từ đầu của thiết bị lưu trữ như ổ Floppy chẳng hạn thì trường này có giá trị là 0.

BPB_TotSec32

32

4

Tổng số Sector của ổ đĩa luân lý (độ dài 32 bit là giá trị mới). Đây là kích thước tính bằng Sector của 4 vùng đã miêu tả ở phần trên. Trong FAT12/16 khi số Sector nhỏ hơn hoặc bằng 0x10000 thì trường này phải gán là 0, trường BPB_TotSec16 sẽ được sử dụng thay thế. Trong FAT32 trường này luôn được sử dụng.

Các trường trong bảng sau sẽ khác biệt giữa FAT12/16 với FAT32. Vì thế, trước khi tham khảo, cần quyết định xem dùng loại nào. Có những trường chỉ có trong FAT32 mà không hề có trong FAT12/FAT16.

Các trường tính từ Offset 36 trong FAT12/16

名前

Offset

Size

解説

BS_DrvNum

36

1

Số thứ tự ổ đĩa được sử dụng trong đĩa của BIOS IBM PC. Trường này được sử dụng trong Boot Trap của MS-DOS. Với Floppy Disk thì nó là 0x00, còn với Disk cố định thì nó là 0x80. Cái này thực tế sẽ phục vụ cho hệ điều hành.

BS_Reserved1

37

1

Dự trữ( dùng cho WindowsNT), khi format thì cần set 0 cho giá trị này.

BS_BootSig

38

1

Boot Signature mở rộng (0x29). Giá trị này sẽ quyết định sự tồn tại của 2\3 giá trị kế tiếp.

BS_VolID

39

4

Số serial của ổ đĩa luân lý. Trường này với trường BS_VolLab được sử dụng như là dấu hiệu để support trên Removable Storage. Nó cũng giúp cho FAT Driver phát hiện ra việc rút Removable Store không đúng. Cái ID này thường được sinh từ giờ hệ thống

BS_VolLab

43

11

Nhãn của ổ đĩa luân lý. Trường này với trường được ghi ở Root Directory sẽ giống nhau. Khi FAT Driver thay đổi trường này ở 1 trong 2 thì cũng phải thay thay đổi cả 2. Trong trường hợp không được set nhãn thì giá trị sẽ là "NO NAME ".

BS_FilSysType

54

8

Loại FAT "FAT12   ", "FAT16   " hoặc là FAT     ".Nhìn vào có thể nghĩ rằng trường này sẽ có tác dụng khi xác định loại FAT. Nhưng không phải vậy, nó không thuộc BPB và FAT Driver của Micosoft cũng không dùng nó để quyết định kiểu FAT. Vì một số FAT Driver vẫn dùng nó để kiểm tra nên để tăng tối đa tính tương thích thì trường này nên được gán đúng kiểu của FAT hiện tại.

Từ offset 36, các trường này chỉ có trong FAT32

名前

Offset

Size

解説

BPB_FATSz32

36

4

Số Sector có trong 1 FAT luân lý. Kích thước của cả ổ đĩa luân lý tính bằng Sector: Giá trị này * BPB_NumFAT. Việc tham khảo trường này là cần thiết trước khi quyết định loại FAT. Cho dù trong FAT12/16 đi nữa cũng không vấn đề gì vì , trường BPB_FATSz16 sẽ được sử dụng và trong FAT32 nó sẽ bị bỏ qua.

BPB_ExtFlags

40

2

Bít 3～0: FAT active tính từ 0. Nếu bit số 7 là 1 thì trường này được sử dụng.  
Bít 6～4: Dự trữ.Bit 7:  bằng 0 thì toàn bộ FAT sẽ được mirroring. Nếu là 1 thì chỉ có FAT được chỉ định ở bit 3~0 là active thôi.Bit 15～8: dự trữ.

BPB_FSVer

42

2

Version của FAT32. Byte cao là Major, Byte thấp là Minor. Trường này không phải để sử dụng cho việc mount ổ đĩa FAT32 cũ mà là dùng để mở rộng FAT32. Trong tài liệu này sẽ nói về version 0.0 thôi.

Những tool Ultility khi truy cập đến ổ luân lý mà thấy giá trị lơn hơn giá trị mà nó có thì sẽ không cần làm gì thêm. Ngoài ra, FAT32 Driver kiểm tra trường này nếu nó lớn hơn thời điểm viết Driver thì cũng không cần mount gì.

BPB_RootClus

44

4

Vị trí của Senctor đầu tiên trong vùng Directory. Thông thường thì là 2 (hay là Sector đầu tiên sau Boot Sector), nhưng giá trị 2 là không bắt buộc.  
Đương nhiên, nếu vị trí Root Directory thay đổi thì phải đặt đúng cái Sector đầu của nó thôi. Nếu vị trí này bị sai hoặc bị xóa mất(0) thì Tool phục hồi đĩa phải thực hiện việc tìm Root Directory thôi.

BPB_FSInfo

48

2

Bên trong vùng dự trữ của FAT32 có vị trí Sector đặt cấu trúc FSINFO.　Thường là 1. Tức là Sector sau Boot Sector.

BPB_BkBootSec

50

2

Nếu khác giá trị 0, thì nó sẽ là giá của Sector chưa bản Backup của Boot Sector.

BPB_Reserved

52

12

Trường dự phòng cho việc mở rộng trong tương lai. Tại thời điểm format ổ đĩa, chúng nên được gán là Zero.

BS_DrvNum

64

1

Giống với FAT12/16

BS_Reserved1

65

1

Giống với FAT12/16

BS_BootSig

66

1

Giống với FAT12/16

BS_VolID

67

4

Giống với FAT12/16

BS_VolLab

71

11

Giống với FAT12/16

BS_FilSysType

82

8

Luôn luôn là "FAT32   ", trường này không liên quan đến việc quyết định loại FAT nào.

Trong Boot Sector, có một điểm rất quan trọng nữa, các byte ở offset 510, 511 của Sector này phải lần lượt là 0x55, 0xAA. Nếu 2 giá trị này không được đọc đúng thì Boot Sector này là invalid. Rất nhiều tài liệu về FAT hiểu làm và cho rằng rằng 2 byte cuối của Boot Sector phải là 0x55, và 0xAA. Điều trên chỉ đúng khi Boot Sector có kích thước là 512 byte. Nhưng trong trường hợp khác thì không thể đúng được. Ta phải hiểu là luôn luôn là vị trị offset 510, 511 (có thể là cả 2 byte cuối cũng được). Trong tài liệu của Microsoft thì vị trí offset 512 trở đi đều là 0 hết.

Kích thước ổ đĩa luân lý, (trường BPB_TotSec16 hoặc 32) thì có thể nhỏ hơn kích thước của thiết bị nhớ hoặc vùng lưu trữ đó. Vì các vùng nhỡ được chia ra làm các ô nhớ cố định để lưu trữ nên có thể xảy ra tình trạng sau.

Có thể có nhiều đơn vị nhớ không lưu trữ hết gây lãng phí một lượng dùng trống còn lại. Đây là điều hoàn toàn bình thường trong FAT. Tuy nhiên, nếu giá trị kích thước luân lý lớn hơn kích thước thật của thiết bị nó được cài đặt, thì sẽ xảy ra rất nhiều vấn đều. FAT Driver khi phát hiện được vấn đề này cần từ chối việc mount ổ đĩa đó.

## 5. Công thức tính kích thước các vùng.

Từ các thông số của BPB thì ta có thể tính chính xác được vị trí của các vùng và kích thước mỗi vùng trên ổ đĩa luân lý.

Vùng FAT là vùng ngay sau vùng dự trữ, vì thế kích thước và vị trị của vùng FAT sẽ được tính như công thức dưới đây:

> FatStartSector = BPB_ResvdSecCnt;  
> FatSectors = BPB_FATSz * BPB_NumFATs;

Tiếp theo, kích thước và vị trí của vùng thư mục gốc:

> RootDirStartSector = FatStartSector + FatSectors;  
> RootDirSectors = (32 * BPB_RootEntCnt + BPB_BytsPerSec - 1) / BPB_BytsPerSec;

32 ở đây là kích thước một Directory Entry (đơn vị Byte).  
Bỏ qua phần thập phân.  
Ở công thức trên, kết quả đã đã bao gồm đủ Sector để chưa toàn bộ Root Directory rồi.  
Ở FAT32, thì vì trường BPB_RootEntCnt luôn là 0 nên kích thước của vùng Root Directory cũng luôn là 0.  
Cứ thế, vung dữ liệu được tính theo công thức dưới đây;

> DataStartSector = RootDirStartSector + RootDirSectors;  
> DataSectors = BPB_TotSec - DataStartSector;

Trường hợp thiết bị lưu trữ được chia thành các đoạn, có thể vị trí bắt đầu cài đặt ổ đĩa luôn lý  
không hẳn là vị trí bắt đầu của thiết bị lưu trữ nên Sector luân lý và chỉ số trên thiết bị lưu trữ có thể khác nhau.

## 6. FAT và Cluster

Sau đây là một vùng rất quan trọng, đó chính là vùng FAT. Chức năng của vùng FAT là ghi lại, quản lý vị trí (một dãy các cluster) trên vùng Data của các File trong hệ thống.

Vùng Data được chia ra đều ra thành các đơn vị lưu trữ gọi là cluster. Cluster không phải là có kích thước tùy ý mà phải bằng một số lần cố định (chính là trường BPB_SecPerClus) Sector. Vùng FAT sẽ quản lý một danh sách các số và ánh xạ 1:1 đến các cluster này. Mỗi giá trị được quả lý trong vùng FAT tương ứng với một Cluster trong vùng Data. Có 1 điểm đặc biệt ở đây, 2 Entry đầu trong vùng FAT (FAT[0], FAT[1] thì không sử dụng hay nói cách khác là dự trữ, nó không chỉ ánh xạ đến một cluster hết. Tính từ FAT[2] (sẽ tương ứng với Cluster 2), FAT[3] sẽ tương ứng với Cluster 3, cứ như thế cho đến hết. Nói cách khác, chỉ số Cluster được tính từ 2. Về những chỉ số trong vùng FAT, có thể xem kĩ tại đây.

Vì FAT này cực kì quan trọng, nếu Sector chưa nó bị hỏng có thể ảnh hướng rất lớn. Khi sử dụng FAT backup, thông qua giá trị BPB_NumFATs thì kích thước vùng FAT này sẽ bằng BPB_FATSz * BPB_NumFATs. Thông thường khi có tha tác chỉnh sửa, nó sẽ copy và cập nhật ở cả 2 vùng.

## 7. Thông số quyết định loại FAT

Về vấn đề này có rất nhiều nhầm lẫn dấn đến việc xảy ra nhiều lỗi khác nhau. Thực tế, việc quyết định loại FAT được thực hiện theo một logic hết sức đơn giản dưới đây:

**Kiểu FAT (FAT12/FAT16/FAT32) được quyết định dựa trên số cluster của ổ đĩa luân lý, ngoài ra không có cách định nào khác.**

Số cluster ở đây là số lượng cluster có ở vùng Data, hay nói cách khác là giá trị mà kích thước vùng Data chia cho kích thước cluster.

CountofClusters = DataSectors / BPB_SecPerClus;

Nếu có số dư thì nó sẽ bị bỏ đi tức là chỉ lấy giá trị nguyên của phép chi trên thôi. Sau khí có được số lượng Cluster rồi thì, thực hiện bước sau để xác định kiểu FAT:

Nếu số cluster nhỏ hơn 4085 thì ổ đĩa luân lý là FAT12

Nếu số cluster >= 4086 và = 65526 thì ổ đĩa luân lý là FAT32.

Đây là cách duy nhất để xác định loại FAT. Ở đây có thể thấy, FAT12 không thể có nhiều hơn 4085 cluster được, FAT16 không thể có ít hơn 4086 cluster, hoặc nhiều hơn 65525 được. Nếu ổ đĩa FAT luân lý nào không tuân theo luật này rất dễ xảy ra việc FAT Driver không nhận đúng loại FAT dẫn đến những sai xót phá hỏng dữ liệu.

Tuy nhiên, những giá trị biên này không được quy định rõ ràng. Từ số lượng cluster ta có thể có được các giá trị biên như ở trên. Nhưng thực tế, những giá trị biên này cũng không thông nhất ( Cluster có thể nhiều loại 1,2,8,10,16...). Ví dụ, giá trị biên của số cluster lớn nhất chẳng hạn, trong FAT Spec thì nói là 4084, trong tài liệu MSDN thì viết là 4085, trên Windows thì 4085 cho Driver và 4086 cho Disk Tool. Nếu lấy 1 cái làm chuẩn thì có trên một cái khác lại thành không chuẩn. Vì thế, để đảm bảo việc tương thích tốt nhất cho source của FAT, ta nên tránh lấy số cluster gần với các giá trị biên. Nên lấy giá trị cách giá trị biên quy định trong FAT Spec khoảng 16 đơn vị là đẹp.

Thêm nữa, có nhiều FAT Driver không thực hiện chặt chẽ quy tắc ở trên để xác định loại FAT mà dựa vào chuỗi kí tự ở trường BS_FilSysType để xác định kiểu. Vì thế, để sử dụng như là cách sau cùng để xác định đúng kiểu FAT, ta cũng nên gán giá trị trường này đúng với kiểu thực của FAT hiện tại.

Từ quy tắc trên ta có thể suy ra cách xác định kiểu FAT từ kích thước của ổ đĩa luân lý. Ví dụ: Kích thước Sector là 512 byte, kích thước Cluster từ 512 Byte ~ 32K Byte.

Kiểu FAT

Kích thước ổ đĩa luân lý

FAT12

～128M Byte

FAT16

2M～2G Byte

FAT32

32M～2T Byte

Ta có bảng như trên.

## 8. Truy cập FAT Entry

Liên quan đến FAT, có một thông số nữa cũng hết sức quan trọng. Đó là cách truy cập vùng FAT (hay các FAT Entry). Đầu tiên, ta cần hiểu rằng mỗi Cluster số N thì sẽ có 1 entry tương ứng (1-1) ở đâu đó trong vùng FAT. FAT16/FAT32 thì khá đơn giản, FAT Entry giống 1 mảng 1 chiều các số nguyên 16\32 bit. So với mảng trên bộ nhớ thì có 1 vài điểm khác. Thứ nhất, các chỉ số không nằm trên một vùng liên tục, ta chỉ có thể chắc chắn từ Sector đầu tiên chứa FAT, thì toàn bộ FAT sẽ nằm trên nhiều Sector liên tục nhau thôi. Vị dụ, muốn xác định Entry của Cluster N (FAT[N]) thì ta thực hiện tính toán như sau:

Vị trí Entry của FAT16:
    ThisFATSecNum = BPB_ResvdSecCnt + (N * 2 / BPB_BytsPerSec);

    ThisFATEntOffset = (N * 2) % BPB_BytsPerSec;

Vị trí Entry của FAT32:
    ThisFATSecNum = BPB_ResvdSecCnt + (N * 4 / BPB_BytsPerSec);

    ThisFATEntOffset = (N * 4) % BPB_BytsPerSec;

Dựa vào công thức ở trên, giá trị của FAT Entry chính là giá 2 hoặc 4 byte kể từ vị trí bắt đầu được xác định bởi chỉ số của Sector. Thứ tự byte trong trường hợp này luôn là Litle-Endian. Trong FAT16/FAT32 thì chỉ số FAT Entry không thể đến số Sector giới hạn được.

Liên quan đến FAT32, có một điểm rất quan trọng nữa. Tuy FAT Entry trong FAT32 chiếm 4 byte, nhưng 4 bit cao thì không được sử dụng mà để dự trữ thôi. Tức là chỉ còn 28 bit thấp được sử dụng. Các bít dự trữ được set là 0, và không được phép thay đổi những giá trị này. Chính vì thế, khi đọc FAT Entry từ ổ đĩa luân lý FAT32, ta phải sử dụng Bit Mask 0x0FFFFFFF với phép AND. Sau đó, khi ghi xuống, ta vẫn đảm bảo 4 bit đầu không thay đổi.

Khi lấy ra:

Lấy ra Entry của FAT32:
    ReadSector(SecBuff, ThisFATSecNum);

    ThisEntryVal = *(uint32*)&SecBuff[ThisFATEntOffset] & 0x0FFFFFFF;

Khi ghi lại:

Lấy ra Entry của FAT32:
    ReadSector(SecBuff, ThisFATSecNum);

    tmp = *(uint32*)&SecBuff[ThisFATEntOffset];
    tmp = (tmp & 0xF0000000) | (NewEntryVal & 0x0FFFFFFF);
    *(uint32*)&SecBuff[ThisFATEntOffset] = tmp;

    WriteSector(SecBuff, ThisFATSecNum);

Với FAT12, mỗi FAT Entry sử dụng 12 bit (1.5 byte) nên có phức tạp hơn một chút. Tuơng ứng với chỉ số Cluster là chẵn hay lẻ ta sẽ có công thức tương ứng như sau:

Vị trí Entry của FAT12:
    ThisFATSecNum = BPB_ResvdSecCnt + ((N + (N / 2)) / BPB_BytsPerSec);
    ThisFATEntOffset = (N + (N / 2)) % BPB_BytsPerSec;

Lấy ra Entry của FAT FAT12:
    ReadSector(SecBuff, ThisFATSecNum);

    if (N & 1) {    /* Cluster có chỉ số là lẻ */
        ThisEntryVal = (SecBuff[ThisFATEntOffset] >> 4)
                     | ((uint16)SecBuff[ThisFATEntOffset + 1] << 4);
    } else {        /* Cluster có chỉ số chẵn */
        ThisEntryVal = SecBuff[ThisFATEntOffset]
                     | ((uint16)(SecBuff[ThisFATEntOffset + 1] & 0x0F) << 8);
    }

Set Entry của FAT12:
    ReadSector(SecBuff, ThisFATSecNum);

    if (N & 1) {    /* Cluster có chỉ số là lẻ */
        SecBuff[ThisFATEntOffset] = (SecBuff[ThisFATEntOffset] & 0x0F)
                                  | (NewEntryVal <> 4;
    } else {        /* Cluster có chỉ số là chẵn */
        SecBuff[ThisFATEntOffset] = NewEntryVal;
        SecBuff[ThisFATEntOffset + 1] = (SecBuff[ThisFATEntOffset + 1] & 0xF0)
                                      | ((NewEntryVal >> 8) & 0x0F);
    }

    WriteSector(SecBuff, ThisFATSecNum);

Đoạn code trên vẫn bị sai,

![fat](/post/images/fat.png)

## 9. Mối liên hệ giữa File và các cluster

Các File nằm trễn ổ đĩa luân lý được quản lý theo đơn vị là Directory (chi tiết về Directory sẽ được nói ở bên dưới). Trong Directory, có chưa tên file, cluster đầu tiên trong dãy cluster của File, những thông tin này là cửa vào để truy cập FAT. Dữ liệu mỗi File là một dãy các cluster nối nhau (link list). Từ cluster đầu tiên, ta sẽ biết được cluster đầu tiên lưu trữ File. Cluster 0, 1 chỉ dùng cho dự trữ, vì thế Cluster sẽ từ 2. Một ổ đĩa luân lý có N Cluster có nghĩa là các Cluster đó có chỉ số từ 2 đến N+1 và số FAT Entry là N+1. Những File có kích thước là 0 thì sẽ không được cấp phát Cluster nào hết. Từ chỉ số hợp lệ của một Cluster, ta có thể tính được vị trí của Sector đầu tiên trong Cluster đó.

FirstSectorofCluster = DataStartSector + (N - 2) * BPB_SecPerClus;

Hay nói cách khác, từ dữ liệu được lưu trữ từ Cluster số 2 trở đi. Khi kích thước File lớn, nó có thể được lưu trữ trên nhiều Cluster. FAT Entry tương ứng cho mỗi Cluster đó sẽ chưa chỉ số của Cluster tiếp theo. Vì vậy, cứ đi theo thứ tự đó, ta sẽ truy cập đến bất cứ Sector nào của File. Và tất nhiên, không thể theo thứ tự ngược lại được. FAT Entry của Cluster cuối cùng trong dãy Cluster mô tả dữ liệu 1 File sẽ có 1 dấu hiệu đặc biệt để nhận biết (EOC). Giá trị này sẽ không trùng với bất cứ Cluster nào đang có để đảm bảo tính đúng đắn. Tùy theo mỗi loại FAT sẽ sẽ sử dụng các giá trị khác nhau:

*   FAT12: 0xFF8～0xFFF (thường đại diện là 0xFFF)
*   FAT16: 0xFFF8～0xFFFF (thường đại diện là 0xFFFF)
*   FAT32: 0x0FFFFFF8～0x0FFFFFFF (thường đại diện là 0x0FFFFFFF)

Thêm một giá trị đặc biệt nữa, đó là dấu hiệu dành cho Cluster lỗi (hay ta vẫn gọi là Bad Cluster). Cluster lỗi nghĩa nói đến Cluster chưa Sector không sử dụng được nữa do hỏng hóc của thiết bị nhớ. Trong quá trình Format, phục hồi, hay dọn dẹp, khi một Cluster được phát hiện là hỏng, thì ta phải dấu lại để sau không đưa dữ liệu lên đó nữa. Các giá trị dùng để đánh dấu Cluster hỏng lần lượt như sau, FAT12 là 0xFF7, FAT16 là 0xFFF7, FAT32 là 0x0FFFFFF7.

Trong FAT12 thì không thể có Cluster hợp lệ nào có chỉ số trùng với những giá trị trên được. Tuy nhiên, trong FAT32, só Cluster tối đa không được quy định rõ, nên vẫn có khả năng một Cluster hợp lệ có trùng chỉ số với giá trị đánh dấu. Nhưng ổ đĩa loại này sẽ gây ra nhiều phức tạp cho các tool Disk Ultility. Vì thế, khi tạo cần tránh những điều này. Với FAT32 ta nên lấy giá trị cận trên của số Cluster là 268435444.

Trên một số hệ thống, nó có giới hạn số Cluster tại thời điểm tạo hệ thống đó. Ví dụ, trên Windows9X/Me thì kích thước hỗ trợ tối chỉ tới 16MB thôi, nên FAT32 không thể sử dụng nhiều hơn 4M Cluster được.

Cluster  chỉ số 0 FAT[0] vì không được sử dụng nên không được cấp phát để lưu trữ bất cứ File nào. Ngoài ra người ta cũng dùng giá trị khác để đánh dấu những Cluster đang sử dụng, hoặc Cluster bị hỏng không thể cấp phát. Số Cluster còn lại mà chưa được sử dụng thì không được lưu ở đâu hết. Vì thế, muốn biết ổ đĩa còn trống bao nhiều, không còn cách nào khác là duyệt toàn bộ FAT Entry. FAT32 có số Cluster tương đối lớn, nên trong trường hợp này người sử dụng thông tin hỗ trợ (từ trường FSInfo).

2 FAT Entry đầu tiên là FAT[0], FAT[1] không được sử dụng để lưu trữ thông tin. Khi forrmat ổ đĩa, tùy vào mỗi loại FAT mà 2 Entry này được gán giá trị khác nhau:

*   FAT12: FAT[0] = 0xF??; FAT[1] = 0xFFF;
*   FAT16: FAT[0] = 0xFF??; FAT[1] = 0xFFFF;
*   FAT32: FAT[0] = 0xFFFFFF??; FAT[1] = 0xFFFFFFFF;

"??" có giá trị bằng giá trị ở trường BPB_Media. FAT[0] cũng không mang chức năng gì đặc biệt. Còn FAT[1], trong trường hợp FAT16/FAT32 thì chúng được sử dụng như sau:

*   Lịch sử trạng thái khi máy được tắt (FAT16: bit15、FAT32: bit31): 1:Bình thường, 0: Xuất hiện việc bị tắt bất thường.
*   Lịch sử lỗi phần cứng (FAT16: bit14、FAT32: bit30): 1: Không có lỗi、0: Có lỗi phần cứng.

Những bit trên để chỉ đến lỗi có thể phát sinh trên ổ đĩa. Nhưng OS support cái này sẽ dựa vào đó để biết được trạng thái của ổ đĩa luân lý lúc trước mà có thao tác tự động phụ hồi, kiểm tra lỗi thích hợp.

Liên quan đến vùng FAT có 2 điểm quan trọng. Thứ nhất, không nhất thiết phải dùng đến hết đến Sector cuối cùng.  Hầu hết FAT sử dụng một dãy Sector nhất định thôi.FAT Driver không được phép giả định về giá trị của phần chưa được sử dụng. Khi tạo ổ đĩa luân lý, phải khởi tạo vùng FAT về zero hết (Trừ FAT[0], FAT[1]). Thêm một điều nữa, trường thể hiện kích thước của ổ đĩa (BPB_FATSz16/32) nhiều khi cũng được gán giá trị lớn hơn thực tế. Vì thế, có thể có những Sector không được sử dụng nằm ở phía sau ổ đĩa FAT. Vì thế, để điều chỉnh lại các vùng này, người ta sinh ra các thao tác Aligment để điều chỉnh lại.

Dưới đây là bảng tóm tắt các khoảng giá trị của FAT Entry

FAT12

FAT16

FAT32

解説

0x000

0x0000

0x00000000

Không sử dụng

0x001

0x0001

0x00000001

Dự trữ

0x002:0xFF6

0x0002:0xFFF6

0x00000002:0x0FFFFFF6

Đang sử dụng (chưa phải Cluster cuối, phía sau còn Cluster khác nữa trong day mô tả dữ liệu)

0xFF7

0xFFF7

0x0FFFFFF7

Sector bị hỏng

0xFF8:0xFFF

0xFFF8:0xFFFF

0x0FFFFFF8:0x0FFFFFFF

Dang được sửdunjg (là Cluster cuối của dãy mô tả dữ liệu)

1.  Về cấu trúc FSInfo của FAT và Backup của Boot Sector

Về kích thước lớn nhất của vùng FAT, trong FAT12 là 6KB, FAT16 là 128KB, FAT32 khoảng vài MB trở lên. Vì kích thước vùng FAT của FAT32 khá lớn, nên phải tránh tối đa phải duyệt lại toàn bộ tất cả khi tìm kiếm một Cluster trống. Vì thế, trong FAT32 nó có hỗ trợ thêm một cấu trúc nữa, là FSInfo. Cấu trúc chứa trọn vẹn trong 1 Sector. Chỉ số của Sector chứa cấu trúc này được xác định bởi giá trị của trường BPB_FSInfo trong bảng BPB.

Cấu trúc FSInfo  của FAT32(chỉ FAT32 mới có)

Tên

Offset

Size

解説

FSI_LeadSig

0

4

Cố định là 0x41615252. Trong thực tế, trường này là dấu hiệu đầu tiên dùng để xác định Sector FSInfo.

FSI_Reserved1

4

480

Dự trữ, không sử dụng, khi format phải được gán hết là 0.

FSI_StrucSig

484

4

Cố định 0x61417272. Trong thực tế, trường này là dấu hiệu thứ 2 để xác định Sector FS_Info.

FSI_Free_Count

488

4

Số lượng Cluster còn trống. Đây là giá trị tham chiếu, không nhất thiết phải đúng. Khi nó được gán là 0xFFFFFFFF thì có nghĩa là không sử dụng trường này.Khi trương này ở trạng thái sử dụng thì ít nhất ta cũng kiểm tra được giá trị số Cluster của ổ đĩa luân lý có hợp lệ hay không.

FSI_Nxt_Free

492

4

Chỉ số của Cluster cuối cùng được cấp phát. Đây cũng chỉ là giá trị tham chiếu thôi. Nếu nó là 0xFFFFFFFF thì cũng có nghĩa là không dùng. Khi FAT Driver muốn tìm một Cluster trống, thì giá trị ở đây được sử dụng như chỉ số bắt đầu. Nếu giá trị ở đây không hợp lệ, thì nó phải tìm từ FAT[2].

FSI_Reserved2

496

12

Dự trự. Khi format ổ đĩa, nó được khởi tạo toàn bộ là 0. Thực tế đến giờ vẫn không dùng.

FSI_TrailSig

508

4

Cố định là 0xAA550000. Trong thực tế, giá trị này được sử dụng như là dấu hiệu nhận biết thứ 3 cho Sector FSInfo.

Ngoài ra, FAT32 còn có một chức năng nữa, đó là Backup Boot Record ( Vùng khởi động dự phòng). Ở trong FA12/FAT16 không có chức năng này, thay vào đó nó sử dụng mã kiểm tra vòng (CRC). Nhưng điều này để tăng cường khả năng phục hồi tronh trường hợp hỏng hóc bất thường. Vị trí Sector chứa vùng Backup Boot Record ở trên được xác định qua trường BPB_BkBootSec. Giá trị này được khuyến cáo mạnh mẽ rằng nên set là 6. Các boot loader hoặc FAT Driver nên được thiết kê trong trường hợp đọc Boot Sector thất bại, thì nó sẽ đọc Boot Record từ Sector số 6.

Ổ đĩa FAT32, có đến 3 Sector liên tiếp là Boot Sector, tức là từ Boot Sector đầu tiên, ta có thêm 2 bản copy nữa. Đương nhiên các Byte 510, 511 của các Sector này lần lượt sẽ nhận giá trị 0x55, 0xAA.

## 10. Về cấu trúc Directory

Đến đây, ta khoan hãy nghĩ đến những File có tên File dài, ta cứ giả định File có tên File đủ ngắn. Trong ổ đĩa FAT luân lý, thư mục là một loại File đặc biệt, người ta dựa vào một thuộc tính đặc biệt của File để xác định nó là Thư mục hay không. Thuộc tính này được lưu trữ bằng cấu trúc trong bảng Directory Table, có kích thước 2 byte. Vì thế, sẽ lưu trữ tối đa 65536 Entry.

Thư mục luôn tồn tại trong ổ đĩa luân lý là Root Directory. Là thư mục ở tầng cao nhất. Trong FAT12/FAT16, nó nằm ngay sau vùng FAT, và có kích thước không thay dổi. Độ dài của bảng Directory chính là giá trị của trường BPB_RootEntCnt. Trong FAT32, cả File và Sub Directory cũng đưowjc lưu. Sub Directory chỉ khác với Directory ở chỗ nó không trỏ đến một Directory Entry nào cả. Sector bắt đầu của Root Directory được xác định bởi giá trị BPB_RootClus.

Ngoài ra còn Sub Directory còn phải giữa thông tin của Dot Entry nữa. Đó là "." và "..". Dưới đây là bảng mô tả của một cấu trúc lưu trữ Directory.

名前

Offset

Size

解説

DIR_Name

0

11

Tên của File ngắn + phần mở rộng.

DIR_Attr

11

1

Các thuộc tính của File. Kết hợp các giá trị sau (2 bít cao đầu tiên không sử dụng)   
0x01: ATTR_READ_ONLY (Chỉ đọc)  
0x02: ATTR_HIDDEN (Ẩn)  
0x04: ATTR_SYSTEM (Hệ thống)  
0x08: ATTR_VOLUME_ID (Nhãn ổ đĩa)  
0x10: ATTR_DIRECTORY (Thư mục)  
0x20: ATTR_ARCHIVE (Lưu trữ)  
0x0F: ATTR_LONG_FILE_NAME (File có tên dài)

DIR_NTRes

12

1

Cờ ghi tên File ngắn bằng chữ thường(đây là 1 option).  
0x08:Toàn bộ là chữ thường.  
0x10:Phần mở rộng là chữ thường

DIR_CrtTimeTenth

13

1

Giá trị hỗ trợ độ chính xác cho DIR_CrtTime. DIR_CrtTime có độ chínhxác là 2s. 2s này sẽ được chia làm 200 phần đánh số từ 0-199. Nếu không support trường này, thì gán bằng 0.

DIR_CrtTime

14

2

Thời điểm (giờ phút giây) tạo File(đâ cũng là option). Trường hợp không support thì cũng gán là 0 và không thay đổi giá trị.

DIR_CrtDate

16

2

Thời điểm (ngày tháng năm) tạo(là option). Trong trường hợp không support thì gán là 0.

DIR_LstAccDate

18

2

Ngày cuối cùng File được mở(là option). Không có thôngtin về giờ. Khi thực hiện thao tác ghi, phải gán giá trị giống với giá trị DIR_WrtDate.Trong trường hợp không support thì cũng gán là 0 và không thay đổi giá trị.

DIR_FstClusHI

20

2

16 bit cao của Cluster đầu tiên. Nếu là FAT12/16 thì gán hết bằng 0 và không thay đổi giá trị.

DIR_WrtTime

22

2

Giờ cuối cùng File được thực hiện thao tác ghi(Ghi xong đóng lại). Phải hỗ trợ.

DIR_WrtDate

24

2

Ngày cuối cùng File được thực hiện thao tác ghi. Phải hỗ trợ.

DIR_FstClusLO

26

2

16 bit cuối của Cluster bắt đầu nội dung File. Nếu File có kích thước là 0, thì trường này cũng là 0.

DIR_FileSize

28

4

Kích thước File tính theo Byte. Nếu là Directory thì giá trị là 0.

Byte đầu tiên trong trường DIR_NAME có ý nghĩa rất quan trọng đối với trạng thái của Entry tương ứng. Nếu nó là 0xE5 hoặc 0x00 thì đây là Entry trống, nó có thể được sừ dụng. Nếu ngoài 2 giá trị trên, thì Entry đó sẽ được hiểu là đang sử dụng. Khi giá trị là 0, thì có được biết đến là Entry cuối cùng đang được sử dụng. Các Entry tiếp sau đó, phải được đảm bảo cũng là 0x00. Điều này sẽ giúp việc tìm kiếm dễ dàng hơn vì không phải tìm đến các Entry không có dữ liệu nữa. Với các tên File mà có byte đầu là 0xE5 giá trị 0xE5 sẽ được chuyển thành 0x05.

Trường DIR_NAME có 11 byte, bao gồm 8 byte phần tên File và 3 byte phần mở rộng. Khi ấy, dấu "." ngăn cách giữa tên File và phần mở rộng sẽ bị xóa bỏ. Nếu tên tổng độ dài của tên File và phần mở rộng không đủ 11 byte, thì các byte ở giữa sẽ được fill là 0x20 (space, dấu cách).

Trường hợp sử dụng kí tự của bảng mã khác ngoài ASCII thì bảng mã máy hiện tại sẽ được sử dụng. Dươi đây là một ví dụ thực tế:

ファイル名         DIR_Name          解説
"FILENAME.TXT"    "FILENAMETXT"     Dấu "." ngăn cách bị xóa.
"DOG.AVI"         "DOG     AVI"     Không đủ 8+3 byte thì sẽ được padding ở giữa.
"File.Txt"        "FILE    TXT"     Chuyển hết thành các kí tự chữ IN HOA
"file.TXT"        "FILE    TXT"     Giống ở trên.
"蜃気楼.JPG"      "・気楼  JPG"      "蜃"の第一バイトが0xE5なので0x05に置き換え
"NO_EXT"          "NO_EXT     "     Không có phần mở rộng
"NO_EXT."         "NO_EXT     "     Giống ở trên
".cnf"                              *không thể ko có tên File
"new file.txt"                      *không sử dụng space trong tên File
"file[1].c++"                       ※kí tự [],+ là không được phép
"longext.jpeg"                      ※dài quá 8+3 cũng không được
"arch.tar.gz"                       ※dài quá 8+3 cũng không được.

Các kí tự ASCII có thể sử dụng trong Filename là:

_0～9 A～Z ! # $ % & ' ( ) - @ ^ _ \` { } ~_

Và các kí tự mở rộng 0x80 - 0xFF). Các kí tự in thường a~z không sử dụng được. Nó không gây lỗi nhưng sẽ được tự động chuyển sang IN HOA. Về kí tự mở rộng, với mỗi ngôn ngữ thì nó sẽ khác nhau. Nhưng tất cả đều được chuyển sang dạng chữ IN HOA rồi chuyển về dạng kí tự ASCII. Chính vì thế, việc tương thích giữa các ngôn ngữ là rất thấp. Về tính tương thích trong môi trường tiếng Nhật, hãy tham khảo phần nói về tính tương thích ở bên dưới.

Các File trong một thư mục thì phải có tên duy nhất. Không thể tồn tại 2 Entry có cùng DIR_Name được. Trường DIR_Attr có giá trị được kết hợp bởi các flag từ bảng sau:

ファイル属性

Flag

Giải thích

ATTR_READ_ONLY

Chống ghi. Không được phép ghi hoặc xóa File này.

ATTR_HIDDEN

Không hiển thị trong thư mục thông thường (phụ thuộc vào hệ thống sử dụng)

ATTR_SYSTEM

File hệ thống (phụ thuộc vào hệ thống sử dụng)

ATTR_DIRECTORY

Đây là thư mục, có chứa thư mục con. Được giải thích bên dưới

ATTR_ARCHIVE

Được set khi có thay đổi trong File. Nó chủ yếu được sử dụng khi các ToolBackup muốn tìm kiếm các file đã thay đổi.

ATTR_VOLUME_ID

Chỉ có 1 Entry sử dụng cờ này. Đó chính Root Directory. Đây là tên của ổ đĩa luân lý luôn. Các trường DIR_FstClusHI, DIR_FstClusLO, DIR_FileSize phải luôn được giữ giá trị 0. Nó được dụng độc lập.

ATTR_LONG_NAME

Bao rằng File có tên dài, và tên trong phần DIR_NAME chỉ là 1 phần thôi. Sẽ được giải thích kĩ hơn ở phần sau.

## 12. Các thao tác với Directory

Tạo File

Khi tạo File, nó sẽ tìm đến một Directory trống trong bảng Directory, rồi tạo Entry cho tên File đó. Nếu tìm đến cuối Cluster mà vẫn không có vùng trống nào thì sẽ cấp phát thêm Cluster mới và thêm vào dãy các Cluster. Số Entry không được phép lớn hơn 65536. Nếu bảng Directory là tĩnh (tức là Root Directory trong FAT12/FAT16) mà đã đầy thì không thể tạo thêm được Entry nữa. Trong Directory Entry mới được tạo, cờ ATTR_ARCHIVE trong trường DIR_Attr sẽ được bật, các trường DIR_FstClusHI, DIR_FstClusLO, DIR_FileSize sẽ được khởi tạo giá trị 0. Khi thực hiện ghi vào File, kích thước File tăng lên > 0, thì sẽ có Cluster mới cho dữ liệu được tạo. Chỉ số của Cluster đó được gán và 2 trường DIR_FstClusHI, DIR_FstClusLO sẽ được gán giá trị tương ứng. Rồi khi kích thước File lớn thêm nữa, Cluster hiện tại không đủ chứa, nó sẽ cấp thêm Cluster. Và dãy Cluster lưu trữ dữ liệu được cập nhật.

Tạo Directory

Khi tạo thưc mụ con, sau khi tạo Entry cho thư mục con đó, trường DIR_Attr sẽ được gán 2 cờ ATTR_DIRECTORY và ATTR_ARCHIVE. Đây chính là điểm khác biệt duy nhất giữa Directory và File. Directory cũng không mang thông tin về kích thước, đương nhien trường DIR_FileSize sẽ bằng 0. Các trường DIR_FstClusHI, DIR_FstClusLO sẽ được gán chỉ số từ Cluster đầu tiên được cấp cho Directory này.

2 Entry đầu tiên của Sub Directory phải là "." và "..". Dot Entry không tồn tại trong Root Directory, chỉ có trong Sub Directory. TrườngDIR_NAME của DIR[0] là ".", của DIR[1] là "..". Thuộc tính ATTR_DIRECTORY phải được set cho 2 thư mục này. Trường lưu vị trí Cluster của "." Entry sẽ lấy chính giá trị của thư mục hiện tại. Còn của ".." thì lấy giá trị của thư mục cha. Nếu thư mục cha là Root Directory thì gán 0.

Xóa File

Khi xóa File, gán giá trị 0xE5 trong Directory Entry. Nếu File nằm trên nhiều Cluster thì gán các FAT Entry tương ứng là 0.

Xóa Directory

Cũng giống với việc xóa File nhưng. Nếu bảng Directory không trống ( ngoài top Entry ra không còn Entry nào khác) thì không xóa được. Nếu xóa thư mục chứa File đi, thì những Cluster bị chiếm bởi các File đó trở thành "Lost Cluster". Vì thế, khi xóa thư mục, phải xóa tất cả các thành phần bên trong đi.

Nhãn ổ đĩa

Là tên được gán cho ổ đĩa FAT, được lưu trong trong Root Entry. Khác với tên của Directory Entry, nhãn đĩa có thể trùng với tên một Directory Entry khác, được phép có kí tự space. Tối đa cũng là 11 byte nhưng không được phép có kí tự ".".

Nhãn không thể sử dụng chức năng LFN được. Khi setup nhãn trong MS-DOS, giá trị nhãn trong Root Directory và trường BS_VolLab nằm trong Boot Sector phải đồng nhất. Trong Windows, thì không có điều này. Trên Windows, nếu nhãn bắt đầu bằng 0xE5 thì sẽ xảy ra vấn đề (nếu không chuyển nó sang 0x05 thì không thể ghi được). Vì thế, nó không cần sử dụng như ở trên.

## 13. Định dạng nhãn thời gian

Trong Directory Entry, có mấy trường liên quan đến thời gian. Hầu hết các Driver chỉ hỗ trợ 2 trương sau đây, đó là DIR_WrtTime, DIR_WrtDate. Các trường khác không được hỗ trợ. Các trường không được hỗ trợ sẽ được giữ giá trị 0. Ngày tháng giờ sẽ có định dạng theo dạng sau:

Tên trường

Các bit

DIR_WrtDate  
DIR_CrtDate  
DIR_LstAccDate

Bit 15～9: Số năm tính từ 1980(0～127). 1980～2107.  
Bit 8～5: tháng (1～12).  
Bit 4～0: ngày (1～31).

DIR_WrtTime  
DIR_CrtTime

Bit 15～11: Giờ  (0～23).  
Bit 10～5: phút (0～59).  
Bit 4～0: giây (theo đơn vị 2 giây) (0～29) tương ứng là 0～58 giây.

## 14. Tên File dài

Khi thêm tính năng hỗ trợ tên File dài (LFN) vào FAT, thì cần đảm bảo tính tương thích với các hệ thống trước đó. Cụ thể, bao gồm những điều sau:

*   Trên các hệ thống cũ, nó sẽ không nhận thức được sự tồn tại của LFN.
*   Mục đích đầu tiên là để nó không bị dê dàng thấy được trên MS-DOS và hệ thống File của Windows cũ. Nếu muốn thấy được sự tồn tại thì phải bằng một phép tìm kiếm Wildcard.
*   Tên File dài sẽ được đặt ở bị trí gần về mặt vật lý với Directory Entry. Hay nói cách khác nó sẽ có quan hệ trực tiếp với SFN Entry, và không làm ảnh hưởng đến perform chung.

-Dù có được phát hiện về sự tồn tại đi chăng nữa, thì cũng sẽ không làm ảnh hưởng xấu đến toàn hệ thống File. Các tool Disk sẽ không sử dụng các API thông thường mà sử dụng các truy cập trục tiếp đến các Sector. Vì phần LFN này coi nhưng không được biết đến với các xử lý thông thường. Vì thế, nó cũng không ảnh hưởng đến việc hỏng hóc của dữ liệu.

Để thỏa mãn những điều kiện trên, LFV được định nghĩa bằng một thuộc tính đặc trưng của Directory Entry. Ở Section trước đã được giải thích rồi, trong thuộc tính của Directory Entry có một thuộc tính là ATTR_LONG_NAME. Từ cái thuộc tính này, sẽ biết được tên trong cái Entry hiện tại là một phần của LFN (Long FileName) Entry. Khi tìm kiếm đến các Entry như thế, thì biết được sự khác biệt giữa LFN và SFN trong bảng dưới đây:

Cấu trúc thư mục của LFN

Tên

Offset

Size

Giải thích

LDIR_Ord

0

1

Là chí số về thứ tự của EntrytrongchuỗiEntry. Khi cờ LAST_LONG_ENTRY được bật (0x40)thì có nghĩa là phần cuối cùng trong chỗ Entry

_Name1

1

10

Tên. Kí tự thứ nhất đến kí tự thứ 5.

LDIR_Attr

11

1

Thuộc tính của LFN. Biểu diễn rằng đây là một phần của LFN Entry , giá trị phải là ATTR_LONG_NAME.

LDIR_Type

12

1

Loại LFN. Luôn là 0, giá trị khác 0 là giá trị dự trữ.

LDIR_Chksum

13

1

Checksume bao gồm LFN Entry và SFN Entry.

LDIR_Name2

14

12

Tên. Kí tự thứ 6 đến 11.

LDIR_FstClusLO

26

2

Để tránh những lỗi do Tool Disk cũ gây ra, trường này nên được gán là 0.

LDIR_Name3

28

4

Tên. Kí tự thứ 12 đến 13.

LFN Entry luôn đi kèm với một SFN Entry mà nó bổ trợ, vì thế không có ý nghĩa gì nếu LFN tồn tại độc lập. Tên File hoặc là chỉ sử dụng SFN hoặc sử dụng cả SFN, LFN. Điều này sẽ đảm bảo tính tương thích ngược với các hệ thống không hỗ trợ LFN. Khi có sử dụng LFN thì tên sẽ là tên đầy đủ. Khi không sử dụng LFN thì LFN Entry sẽ không được nhận ra và việc access vẫn bằng SFN. Với hệ thống hỗ trợ LFN thì có thể sử dụng cả 2 phương án để truy cập File. Các Entry LFN chỉ mô tả các thông tin về tên File, cho dù không có đi chăng nữa thì việc truy cập SFN vẫn không vấn đề gì.

Lưu trữ tên File cho File"MultiMediaCard System Summary.pdf" 

Vị trí

Byte đầu

Trường tên

Thuộc tính

Nội dung

DIR[N-3]

0x43

ary.pdf

--VSHR

LFN thứ 33(lfn[26..38])

DIR[N-2]

0x02

d System Summ

--VSHR

LFN2 thứ 2(lfn[13..25])

DIR[N-1]

0x01

MultiMediaCar

--VSHR

LFN thứ 1(lfn[0..12])

DIR[N]

'M'

MULTIM~1PDF

A-----

SFN

Mỗi tên trong LFN Entry chỉ có 13 kí tự . Do số kí tự tối đa hỗ trợ cho tên File là 255 kí tự nên tên File sẽ chiếm tối đa 20 Entry. Mỗi kí tự sử dụng trong tên File sử dụng Encode là UCS-2 (tức là mỗi kí tự gồm 16 bit). Nếu số kí tự không chia hết cho 13 thì tên sẽ kết thúc bằng kí tự Null (U+0000) và phần trống sẽ được fill bằng (U+FFFF).

Các LFN Entry được xếp liên tiếp ngay trước SFN Entry. Nếu không thỏa mãn điều này thì coi như là không sử dụng được.

## 15. Phạm vi đặt tên và Page Code

Với tên file Ngắn

Được phép sử dụng 8 byte phần tên và 3 byte phần mở rộng. Hay ta vẫn gọi là dạng 8.3. Độ dài tối đa của đường dẫn đầy đủ: 64 byte (cho Path) + 12 byte (cho tên 8.3) + 3 byte cho tên ổ đĩa = 97 byte( không tính NULL ở cuối). Các kí tự có thể dùng để đặt tên gồm kí tự tiếng Anh, kí tự thuộc khoảng (0x80 - 0xFF) và một phần của kí hiệu ASCII ("_$ % ' - _ @ ~ \` ! ( ) { } ^ # &"._

Khi lưu trữ, tên được lưu trữ trong SFN (sử dụng OEM Code Page), không cho phép chữ thường nên các tên đặt chữ thường sẽ bị chuyển hết về chữ hoa.

Tên file Dài

Tối đa có 255 kí tự phần tên + 3 kí tự phần mở rộng, tổng sẽ có 259 kí tự (+1 cho dấu .) chưa kể kí tự NULL. Các kí tự có thể dùng để đặt tên bao gồm: Tất cả các kí tự của SFN, thêm các dấu "_+ , ; = [ ]_" của ASCII, dấu chấm có thể xuất hiện nhiều lần (không chi riêng phần ngăn cách giữa tên và phần mở rộng), dấu cách. Tuy nhiên, không có phép dấu "." hoặc dấu Space ở cuối tên. Có thể tool tạo vẫn cho phép nhưng trong Window thì gần như không thể nhập trực tiếp được.

Tên sẽ được lưu trữ như đã nhập chứ không bị chuyển đổi sang chữ Hoa. Các kí tự sử dụng được Encode bằng UCS-2, bảng kí tự Unicode.

Vì SFN và LFN sử dụng bảng mã cũng như cách Encode khác nhau nên khi implment cần phải làm làm cho chúng tương thích. Nếu Code page là SBCS thì không xảy ra vấn đề gì, nhưng nếu là DBCS thì kích thước bản sẽ chênh nhau vài trăm K nên sẽ xảy ra nhiều vấn đề. Nhất là đối với các hệ nhúng với tài nguyên hạn chế.

## 16. Giữa SFN và LFN

Đầu tiên, tên được ghi trong SFN có dạng SFN, còn tên được ghi trong LFN thì có dạng LFN. Cùng một tên nhưng được ghi đồng thời và 2 chỗ. Trong thư mục, không thể có file bất kể SFN/LFN mà có tên giống nhau được. Vì thế các tên "LongFileName.txt", "longfilename.txt", "LONGFILENAME.TXT"

Sẽ cùng được nhìn nhận như nhau. Vì vậy, trong phép tìm kiếm không có phân biệt chữ Hoa vs chữ thường.

Khi lấy danh sách File từ thư mục, nếu không thể chuyển các kí tự của LFN sang OEM Code thì FAT Driver sẽ hoặc là thay thế các kí tự không chuyển bằng Underscore (_) hoặc là sử dụng tên trong SFN để thay thế.

## 17. Việc tạo SFN và các kí tự đại diện

Trong hệ thống LFN thì các API đương nhiên luôn sử dụng LFN. Đồng thời, nó cũng phải tạo cả SFN nữa, luôn luôn phải đồng thời không được phép tách ra. Điều kiện này để tránh những rắc rối trên hệ thống và ứng dụng khác nhau, cũng như sự thống nhất về tên giữa SFN và LFN. Việc tạo tên SFN (bao gồm cả phần tên, phần kí tự thay thế, phần mở rộng) tuân theo các bước sau:

1.  Xóa các kí tự cách và dấu "." ở cuối
2.  Khi gặp dấu cách trong tên File, thì xóa hết và bật cờ báo có kí tự không thể chuyển đổi.
3.  Khi gặp dấu chấm "." trong ở đầu tên File thì xóa hết và cũng bật cờ báo có kí tự không thể chuyển đổi.
    
4.  Khi vẫn còn có từ 2 dấu chấm "." trở nên thì xóa chỉ để lại dấu chấm ỏ phía sau, bật cờ có kí tự không thể chuyển đổi.
    
5.  Các kí tự in thường (chữ cái và mở rộng) chuyển hết sang chữ Hoa.
    
6.  Nếu nhập vào Unicode thì cần chuyển sang OEM Code, nếu kí nào không chuyển được thì thay thế bằng Underscore "_", bật cờ báo có kí tự không chuyển đổi được.
    
7.  Với tên của SFN, điền hết space vào giữa để đủ độ dài 8.3.
    
8.  Khi tạo SFN, thì cứ từ kí tự đầu tiên trong tên LFN copy sang SFN. Việc copy kết thúc khi thỏa mãn một trong những điều kiện sau: hết tên tròn LFN, phát hiện dấu ".", đã copy được 8 byte. Nếu vượt quá độ dài 8 byte thì bật cờ báo có kí tự không chuyển đổi được.
    
9.  Tạo phần mở rộng của SFN, nếu có tồn tại kí tự "." thì copy từ kí tự ngay phía sau kí tự "." đến khi xảy ra một trong các điều kiện sau: hết chuỗi, đã copy 3 byte). Nếu có nhiều hơn 3 byte thì bật cờ báo có kí tự không chuyển đổi được.
    

Đến đây, ta đã xong phần tên (gồm cả phần mở rộng) của SFN rồi. Ta xét đến cờ báo có kí tự không chuyển đổi được hay không. Nếu nó được bật, sẽ thêm các kí tự đại diện khác vào SFN. Định dạng các kí tự đại diện này có dạng "~n" (n có giá trị từ  1 đến 6 chữ số trong hệ cơ số 10). Nếu thêm các kí tự đại diện này vào mà làm cho phần tên (trước dấu ".") vượt quá 8 byte thì đoạn cuối của phần tên sẽ bị xóa đi.

Theo cách này, nếu trong thư mục đó đã có một SFN Entry có tên đó rồi, thì sẽ thay đổi giá trị n để tìm ra tên chưa được sử dụng. Thông thường, giá trị n này được tăng dần từ 1, nhưng cũng có trường hợp sử dụng một thuật toán sinh  khác. Ví dụ trong trường hợp FAT của Windows NT, giá trị được tìm kiếm từ 1 nhưng từ lần thứ 5 trở đi, nó sẽ sử dụng giá trị có trong Buffer nào đó chứ không tăng n lên mãi.

## 18. Những trường hợp không sinh ra LFN Entry

Nếu một tên File tuân theo quy tắc 8.3 rồi thì LFN và SFN sẽ giống hệt nhau (bỏ qua trường hợp khác nhau về chữ Hoa chữ thường). Nếu tên thỏa mãn thêm các trường hợp sau thì LFN tương ứng sẽ không được sinh ra nữa.

*   Không chứa kí tự chữ ASCII in thường.
*   Không chứa kí tự mở rộng (0x80 trở lên)
    

Lại trong Windows NT, khi thỏa mãn các điều kiện sau thì LFN sẽ được bỏ qua

*   Chứa kí tự in ASCII in thường ở phần tên hoặc phần mở rộng hoặc cả 2 phần.
*   Không phải là sự lẫn lỗn giữa chữ Hoa và chữ thường.
    
*   Phần mở rộng không chứa kí tự mỏ rộng(0x80 trở lên)
    

Nếu thỏa mãn nhưng điều kiện trên, bản chữ thường sẽ được lưu và trường  DIR_NTRes  , LFN Entry sẽ bị bỏ qua. Những File sau đây thỏa mãn điều kiện này : "lower12.dll", "system32", "FDCMD.exe"  , trong Windows NT có rất nhiều File được đặt tên theo dạng này. Cũng có một số hệ thống không support việc lưu trữ ở trường DIR_NTRes  (ví dụ Windows 9x chẳng hạn) thì chúng sẽ sử dụng toàn bộ tên dạng chữ Hoa được lưu ở SFN thôi. Vì FAT System không phân biệt chữ Hoa, chữ thường khi tìm kiếm, nên không vấn đề gì hết. Nhưng trong một số hệ thống Unix, thì việc này sẽ xảy ra vấn đề.

## 19. Tính tương thích giữa các hệ thống

Hệ thống không hỗ trợ LFN

Việc hỗ trợ LFN trên các Disk cố định đương nhiên là việc rất cần thiết rồi. Trên các thiết bị nhớ di động cũng được support. Vì các thiết bị di động có thể sử dụng ở nhiều nơi, với nhiều hệ thống khác nhau nên yêu cầu về tính tương thích là vô cùng quan trọng. Chức năng LFN phải được implement để ngay cả trên các hệ thống không support LFN cũng không xảy ra mất mất dữ liệu. Ví dụ, một ổ đĩa sử dụng LFN cũng có thể truy cập được trên hệ thống không hỗ trợ LFN. Không những thế, việc truy cập phải đảm bảo rằng không có sự sửa chữa hay chuyển đổi dữ liệu nào. Trừ khi có thao tác tạo mới Directory Entry thì chúng sẽ được tạo theo cách hệ thống sử dụng.

Trên các hệ thống không hỗ trợ LFN, các LFN Entry dù có tồn tại cũng không được nhận biết, vì thế các LFN Entry có thể bị hỏng bởi các thao tác với thư mục. Dù các LFN Entry có bị hỏng đi chăng nữa thì, phần dữ liệu của File vẫn không bị mất. Tất nhiên phần tên LFN có thể bị mất nhưng, ta luôn truy cập được bằng SFN.

Tương thích với tiếng Nhật

Các loại OEM Page Code khác nhau

Mac OS X

## 20. Về lưu trữ

Dù không liên quan trực tiếp đến phần File System đang nói đến, nhưng khi tạo ra một hệ thống Embedded, ta không thể không nhắc đến phần lưu trữ vật lý được.

Với những thiết bị lưu trữ cố định (nghĩa là ko phải kiểu copy vào rồi tháo ra mang sang chỗ khác như USB, hay thẻ nhớ ấy) thì người ta thường chia ra làm nhiều partition để sử dụng. Ví dụ 8GB được chia ra làm 3 Partitions tương ứng là 4G, 2G, 2G. Cũng có trường hợp người ta chỉ sử dụng 1 disk logic trên 1 đĩa vật lý thôi.

Về tool để partitition, thì người ta hay sử dụng 2 tool là FDISK và SFD (Super Floppy Disk). Ở những Disk thời trước, trong Sector vật lý đầu tiên, thường lưu trữ bảng phân vùng, tức là bảng mô tả Disk được chia ra như thế nào. Về sau, những Floppy Disk không còn được chia ra nữa. File System được tính từ sector vật lý đầu tiên luôn.

FDISK được được sử dụng trên các Disk cố định và một số thiết bị nhớ di động (Removable Media) như SD Card, CF Card, USB Memory. SFD được sử dụng trên các thiết bị còn lại như Floppy Disk, thẻ từ, MicroDrive. Trên Windows, với thiết bị nhớ di động cả định dạng của FDISK (loại chỉ 1 phần vùng), và SFD, còn FDISK loại nhiều phân vùng không được Support. Ngược ại, với Disk cố định, thì SFD không được hỗ trợ.

Bảng sau đây mô tả các trường của MBR:

Các trường của MBR(Sector vật lý 0)

Tên

Offset

Kích thước

Giải thích

MBR_bootcode

0

446

Mã khởi động. Mã này phụ thuộc vào hệ thống. Giá trị 0 nghĩa là không sử dụng.

MBR_Partation1

446

16

Bảng phân vùng Entry 1. Mô tả trạng thái và phạm vi của phân vùng. (Chi tiết về trường này ở bên dưới)

MBR_Partation2

462

16

Bảng phân vùng Entry 2.

MBR_Partation3

478

16

Bảng phân vùng Entry 3.

MBR_Partation4

494

16

Bảng phân vùng Entry 4.

MBR_Sig

510

2

Phải là 0xAA55. Mã xác thức MBR. Nếu giá trị khác giá trị trên sẽ coi như MBR không hợp lệ.

Mỗi Bảng phân vùng Entry được mô tả như bảng dưới đây. Có tối đa 4 Entry thôi, có nghĩa là có tối đa 4 phân vùng có thể sử dụng. Cũng có thuật ngữ về phân vùng mở rộng (Extended Partition) tức là có thẻ chia phân mỗi phân vùng vật lý thành nhiều phân vùng con, nhưng xin không nhắc đến ở đây.

Các trường trong bảng phần vùng Entry

Tên

Offset

Kích thước

Giải thích

PT_BootID

0

1

Nhãn boot.  
0x00: Không phải phân vùng boot  
0x80:Phân vùng boot.  
Boot có nghĩa là có thể từ phân vùng tương ứng, nhưng đây là 1 trường phụ thuộc hệ thống sử dụng. Chỉ có thể chỉ định 1 phân vùng trong số 4 phân vùng để boot.

PT_StartHd

1

1

Chỉ số của Head trỏ đến Sector đầu tiên theo định dạng CHS..(0~254)

PT_StartCySc

2

2

Chỉ số của Cylinder (10 bit thấp 0=1023) và chỉ số sector bên trong Track(6 bit cao 1~63)

PT_System

4

1

Loại phân vùng (có tính đại diện đại diện)。  
0x00:Phân vùng trống  
0x01: FAT12 (CHS/LBA, Có không quá 65536)  
0x04: FAT16 (CHS/LBA, Có không quá 65536)  
0x05: Phân vùng mở rộng (CHS/LBA)  
0x06: FAT12/16 (CHS/LBA, Có từ 65536 sector)  
0x07: HPFS/NTFS/exFAT (CHS/LBA)  
0x0B: FAT32 (CHS/LBA)  
0x0C: FAT32 (LBA)  
0x0E: FAT12/16 (LBA)  
0x0F: Phân vùng mở rộng (LBA)

PT_EndHd

1

1

Chỉ số của Header của Sector cuối ùng của phân vùng(0～254). (Theo định dạng CHS)

PT_EndCySc

2

2

Chỉ số của Cylinder của Sector cuối cùng (10 bit thấp :0～1023)và Chỉ số sector bên trong Track (6 bit cao:1～63). (theo định dạng CHS)

PT_LbaOfs

8

4

Chỉ số đầu tiên của Sector theo định dạng LBA(1～0xFFFFFFFF)。

PT_LbaSize

12

4

Kích thước phân vùng theo định dạng LBA(1～0xFFFFFFFF)。

Trong bảng phân vùng, không phải tất cả các Entry đều được sử dụng. Thông thường, chỉ 1 phân vùng được tạo, 3 Entry lại để trống.

Ở trên có nói đến 2 thuật ngữ CHS và LBA. CHS nghĩa là mô tả theo tính chất vật lý của đĩa, nhưng phụ thuộc vào hệ thống. LBA nghĩa là chỉ cần 2 thông tin là Sector đầu tiên và số Sector sẽ dùng là đủ. Do bị giới hạn về dung lương nên khi vượt quá phạm vi dung lượng thì các trường CHS sẽ bị loại.