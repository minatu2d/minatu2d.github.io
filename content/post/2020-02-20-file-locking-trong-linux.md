---
title: "File locking trong linux"
description: "Giới thiệu cơ bản về file locking ứng dụng trên linux"
date: "2020-02-29T21:17:00+09:00"
thumbnail: ""
toc: true
published: true
categories:
  - basic
  - cmd
tags:
  - cmd
  - locking
  - filelock
---

## 1. Giới thiệu 
* `File locking` (khoá file) là cơ chế để đảm bảo việc `đọc/ghi` file giữa nhiều `process` cùng lúc một cách an toàn.
* Bài này chúng ta sẽ giới thiệu xem cơ chế `Locking` sinh ra giải quyết vấn đề gì và các ví dụ sử dụng của nó trên `bash script` và `ngôn ngữ C`.

## 2. Nếu vấn đề theo cách muôn thủa
* Ví dụ mô tả vấn đề `cập nhật xen kẽ` trong bất cứ hệ thống thông tin nào.
    - Đó là vấn đề cập nhật số dư tài khoản (`balance`) được lưu trong file `balance.dat` thông quan 2 `process` là `A` và `B`, giả sử ban đầu trong tài khoản có `100 củ`, thứ tự thực hiện đúng sẽ như sau:
        - 1. **Process A**: Đọc giá trị tài khoản hiện tại, trừ đi `20 củ` và lưu kết quả vào `số dư`.
        - 2. **Process B**: Đọc giá trị tài khoản hiện tại, cộng vào `80 củ` và cũng lưu kết quả vào `số dư`.
    - Sau khi thực hiện xong, số dư tài khoản phải là : `100 - 20 + 80 = 160 củ`.
    - Tuy nhiên, nếu xảy ra vấn đề `cập nhật xen kẽ`, thứ tự thực hiện có thể như sau:
        - 1. **Process A**: Đọc giá trị `balance` hiện tại và chuẩn bị thực hiện tính toán.
        - 2. **Process B**: Đọc được cùng giá trị với **Process A** và cũng chuẩn bị thực hiện tính toán.
        - 3. **Process A**: Thực hiện trừ `20 củ` và lưu kết quả (`80 củ`) vào `balance`
        - 4. **Process B**: Do không biết giá trị nó đọc lúc trước đã bị thay đổi (giá trị nó đang giữa là `100 củ` trong khi giá trị của `balance` đã là `80 củ` rồi), nên nó vẫn thực hiện cộng `80` củ và lưu kết quả `100 + 80 = 180 củ` vào `balance`.
    - Kết quả là `180` là giá trị cuối cùng chứ không phải là `160` như mong muốn.

## 3. Hai cơ chế `Locking` trong ứng dụng `Linux` 
* `File locking` (ở đây là nói đến đơn vị `File` để phân biệt với `memory locking`) là cơ chế cho để hạn chế truy cập cùng 1 file của nhiều `process`. Có nghĩa là chỉ cho phép một `process` truy cập ở một thời điểm cụ thể nào đó, vì thế sẽ tránh được `cập nhật xen kẽ`.
* Ta hẳn đều biết câu lệnh được ví như `ôm boom cảm tử` trong linux: `rm -rf /` (**Tuyệt đối đừng thử nếu không biết!!!!!**). Nếu ta chạy nó, toà bộ hệ thống đang chạy sẽ bị xoá sạch **không còn một cái gì, nên nhớ là không còn một cái gì hết**.
* Tại sao lại kì quặc vậy, hệ thống đang chạy mà lại còn bị xoá đi là sao.
Đó chính là vì : **Linux thường không tự động thực hiện lock các file đang được mở**.
Cho nên cho dù hệ thống đang chạy với một tá file đang được mở đi nữa, thì nó vẫn có thể bị xoá sạch sẽ.
* Thực ra, Linux có hỗ trợ 2 cơ chế lock *lock* (gọi là 2 loại lock cũng được), đó là:
    - Cơ chế khoá cộng tác (Advisory Locking)
    - Cơ chế khoá độc quyền (Mandatory Locking)
* Chúng ta sẽ xem chi tiết thêm ở bên dưới.

### 3.1 Cơ chế khoá cộng tác (Advisory Locking)
* Nó không phải là một cách thức bắt buộc. 
* Nó chỉ chạy khi các `process` tham gia tự giác cộng tác bằng cách `chiếm lấy khoá`.
* Hay nói cách khác, `khoá cộng tác` này sẽ bị làm ngơ nếu `process` cố tình không chiếm lấy khoá.
* Tiếp tục lấy ví dụ về trường hợp cập nhật `balance.dat` ở trên.
* Trường hợp **Process B** không cộng tác (hoặc không tự giác):
    - 0. Gỉa sử giá trị ban đầu của `balance` vẫn là `100 củ`.
    - 1. **Process A**: thực hiện `chiếm lấy khoá` trên file `balance.dat` và thực hiện mở file, đọc được giá trị `100 củ` lên.
        - Chúng ta phải hiểu rằng, `khoá` mà **Process A** đã chiếm không được set bởi hệ điều hành hay hệ thống file.
        - Vì thế, cho dù **Process A** đã khoá đi nữa thì **Process B** vẫn thoải mái đọc, ghi, thậm chí là xoá file đó thông qua `system call`.
        - Nếu **Process B** thực hiện như thế thật, thì ta nói rằng : **Process B** không cộng tác với **Process A** hay đơn giản là **không tự giác**.

* Trường hợp **Process B** có cộng tác (hoặc tự giác):
    - 0. Gỉa sử giá trị ban đầu của `balance` vẫn là `100 củ`.
    - 1. **Process A**: Thực hiện `chiếm lấy khoá` trên file `balance.dat` và thực hiện mở file, đọc được giá trị `100 củ` lên.
    - 2. **Process B**: Thực hiện việc cố gắng `chiếm lấy khoá` trước đọc file.
        - Vì **Process A** đã `locked` rồi nên **Process B** sẽ đợi **Process A** nhả ra.
    - 3. **Process A**: Thực hiện trừ `20 củ` và lưu kết quả (`80 củ`) vào `balance`.
        - Rồi nó cũng thực hiện `nhả (release) lock` luôn.
    - 4. **Process B**: Giờ nó chiếm được `lock` rồi, nó sẽ đọc file và được giá tị `80 củ`.
        - Rồi nó thực hiện logic của nó (cộng thêm `80 củ`) và lưu vào file `balance.dat`.
        - Cuối cùng nó cũng `nhả lock` để các `process` khác có thể dùng.
        
### 3.2 Cơ chế khoá độc quyền (Mandatory Locking)
* Trước khi nói về `khoá độc quyền`, chúng ta hãy nhớ trong đầu là **Khoá độc quyền trong Linux là không đáng tin (unreliable)**
    - Trong tài liệu `man page`, ([link](http://man7.org/linux/man-pages/man2/fcntl.2.html)) về hàm C `fcntl` có đoạn sau:
    >Mandatory locking
    >
    >   Warning: the Linux implementation of mandatory locking is unreliable.
    >   See BUGS below.  Because of these bugs, and the fact that the feature
    >   is believed to be little used, since Linux 4.5, mandatory locking has
    >   been made an optional feature, governed by a configuration option
    >   (CONFIG_MANDATORY_FILE_LOCKING).  This is an initial step toward
    >   removing this feature completely.
    Lược dịch:
    > Implement của `mandatory locking` trong `Linux` là không đáng tin.
    > Sẽ thêm lý do ở BUGS bên dưới. Chính vì những BUG đó, từ bản `Linux 4.5`, phần implement
    > của `mandatory locking` sẽ được coi là `tính năng optional`, được quyết định có trong 
    > kernel hay không bằng option `CONFIG_MANDATORY_FILE_LOCKING`. Đây là bước đầu của việc 
    > loại bỏ hoàn toàn tính năng này trong tương lai.
* Không giống `khoá cộng tác` ở trên, `khoá độc quyền` không yêu cầu bất cứ sự cộng tác nào từ phía `process` khác
* Một khi khoá này được kích hoạt trên file nào đó, hệ điều hành sẽ không cho bất cứ `process` nào khác đọc/ghi file đó nữa.
* Để sử dụng được loại khoá này trên Linux, thì hệ thống phải thoả mãn 3 yêu cầu sau:
    - 1. Phải `mount` hệ thống file với `mand` option 
        - Câu lệnh sẽ có dạng: `mount -o mand <FILESYSTEM> <MOUNTPOINT>
    - 2. Phải bật bit `set-group-ID` và tắt bit `group-execute` cho những file mà ta muốn `lock`. 
        - Câu lệnh sẽ có dạng: [chmod](http://man7.org/linux/man-pages/man1/chmod.1.html) g+s, g-x <files>
    - 3. `Kernel` phải được build với option `CONFIG_MANDATORY_FILE_LOCKING` được bật.

## 4. Xem danh sách lock đang được sử dụng
* Sử dụng  `lslock` (có sẵn trong package [util-linux](https://en.wikipedia.org/wiki/Util-linux)
    - Kết quả:Trong đó cột `M` sẽ cho biết `khoá cộng tác` (giá trị = 1) hay `khoá độc quyền` (giá trị = 0)
    ![locking_cmd_ls_locks](/post/images/locking_cmd_lslocks.png)


* Sử dụng `cat /proc/locks`
    - Kết quả: Ở đây, cột thứ 3 từ trái sang cho ta biết nó là `khoá cộng tác` (ADVISORY) hay `khoá độc quyền` (MANDATORY)
    ![locking_cmd_cat_proc_locks](/post/images/locking_cmd_cat_proc_locks.png)
    
## 5. Sử dụng `Locking` trong `script`
* Ta sẽ xem các ví dụ cho `khoá cộng tác` là chủ yếu.
* Lệnh [flock](http://man7.org/linux/man-pages/man1/flock.1.html) từ [util-linux](https://en.wikipedia.org/wiki/Util-linux) cho phép ta thực hiện `lock` file. 
* Để `lock` 1 file cho một câu lệnh bất kì, ta sử dụng:
```
flock <file path > <command>
```
* Giả sử ta làm thử với ví dụ, cập nhật số dư tài khoản:

* Ta giả sử có 1 file text `balance.dat` chứa số dư hiện tại, ta có 2 `process` **A** và **B** được chạy bởi 2 script để thực hiện việc cập nhật tài khoản.
* Ta chuẩn bị 1 script là `update_balance.sh` để thực hiện việc update số dư chung cho cả **A** và **B**
* **update_balance.sh** sẽ chạy script sau:
```bash
#!/bin/bash
file="balance.dat"
value=$(cat $file)
echo "Read current balance:$value"

#sleep 10 seconds to simulate business calculation
progress=10
while [[ $progress -lt 101 ]]; do
    echo -n -e "\033[77DCalculating new balance..$progress%"
    sleep 1
    progress=$((10+progress))
done
echo ""

value=$((value+$1))
echo "Write new balance ($value) back to $file."
echo $value > "$file"
echo "Done."
```
* Tạo file `balance.dat` bằng lệnh:
```bash
echo "100" > balance.dat
```
* Script **a.sh** mà **Process A** chạy sẽ như sau:
```bash
#!/bin/bash
#-----------------------------------------
# process A: lock the file and subtract 20
# from the current balance
#-----------------------------------------
flock --verbose balance.dat ./update_balance.sh '-20'
```

* Trường hợp 1: **Process B** không quan tâm đến việc `chiếm khoá`, cứ access như bình thường:
    - Script **b.sh** mà **Process B** chạy sẽ như sau:
    ```bash
    #!/bin/bash
    #----------------------------------------
    # process B: add 80 to the current balance in a
    # non-cooperative way
    #----------------------------------------
    ./update_balance.sh '80'
    ```
    - Khi chạy sẽ như sau:
    ![file_locking_non_cooporative](/post/images/file_locking_non_cooporative.gif)

* Trường hợp 2: **Process B** `chiếm khoá` rồi mới access file:
    - Script **b.sh** mà **Process B** chạy sẽ như sau:
    ```bash
    #!/bin/bash
    #----------------------------------------
    # process B: add 80 to the current balance
    # in a cooperative way
    #----------------------------------------
    flock --verbose balance.dat ./update_balance.sh '80'
    ```
    - Khi chạy sẽ như sau:
    ![file_locking_cooperative](/post/images/file_locking_cooperative.gif)

## 6. Sử dụng `Locking` trong `C`

## 7. Links tham khảo