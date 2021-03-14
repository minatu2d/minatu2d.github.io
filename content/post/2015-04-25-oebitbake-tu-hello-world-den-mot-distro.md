---
layout: post
title: "[OE]Bitbake - Từ Hello World đến một Distro"
date: 2015-04-25 07:50:24.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Embedded
- OpenEmbbeded
- YoctoProject
tags:
- Bitbake
- Distro
- HelloWorld
meta:
  _edit_last: '72243466'
  geo_public: '0'
  _oembed_b99dc982d254783d84da9909eedb2426: "{{unknown}}"
  _wpcom_is_markdown: '1'
  _publicize_job_id: '20014763206'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2015/04/25/oebitbake-tu-hello-world-den-mot-distro/"
---
![user-configuration](/post/images/user-configuration.png)

Bitbake là một công cụ cốt lõi của [Yocto Project](https://www.yoctoproject.org/about). Nó bao gồm 1 bộ thông dịch các script được viết trong các file recipe (công thức tạo phần mềm), và thực hiện các lệnh trong đó. Nó mô tả lại và tự động hóa qúa trình người ta đưa một phần mềm vào một distro.

Về việc đưa một phần mềm vào distro, ta có thể thấy nó bao gồm vài step chính. Từ việc tải source code (ở đây là tải source code chứ không phải các gói đã được build sẵn đâu nhé, nó gần giống với ArchLinux, Gentoo và hoàn toàn khác với Ubuntu) , thực hiện các bản patch (sửa source hoặc kịch bản build đề phù hợp mục đích sử dụng), biên dịch, cuối cùng là tích hợp vào distro (kèm theo các thông số cấu hình).

Mục đích của bitbake là tạo ra một quy trình tự động mà đầu vào là các file kịch bản, bitbake sẽ tự làm các công việc còn lại. Ý tưởng của bitbake thực sự rất hay, dù rằng để áp dụng thực tế một cách nuột nà thì cần khá nhiều công sức để hiểu cách nó làm việc với các công cụ truyền thống.

(Link gốc của [bài](http://hambedded.org/blog/2012/11/24/from-bitbake-hello-world-to-an-image/))

OpenEmbedded (OE) và Yocto Project (YP) sử dụng **Bitbake** là công cụ chính.

Sẽ rất có ích cho việc hiểu về OE nếu ta hiểu BitBake ở mức độ nào đó thông qua các task (task ở đây xoanh quanh các xử lý của source code như tải, patch, build như đã nỏi ở trên), các class được định nghĩa trong OE.

Tài liệu này với mong muốn đưa ra một bức tranh mô tả việc tạo ra một Image (ảnh của một Distro) sử dụng OE và YP. Nó sẽ bắt đầu từ một project rất đơn giản sử dụng bitbake. Thông qua project đó sẽ giải thích các khái niệm quan trọng từ các OE class, cuối cùng là chỉ ra quá trình tạo một Image.

1.  Về Bitbake
2.  Tải Bitbake

## 1. BITBAKE là cái gì?



Về cơ bản, khi được chạy nó đọc vào các tasks định nghĩa trong các bitbake metadata(.bbclass, .bb), làm rõ các phụ thuộc (phụ thuộc) giữa chúng, đặt chúng theo một thứ tự và chạy các task cho các metadata đó. Ví dụ như cài dể build một gói A thì phải build gói B trước, tải source nó ở địa chỉ nào etc, các thông tin như thế sẽ nằm trong các file metadata.

(Ở đây có khái niệm metadata, tức là phần siêu dữ liệu chứ không phải dữ liệu chính.)

Bitbake là 1 hệ thống build source code, data của nó là source code, các bản patch.

Còn metadata là các file phục vụ quá trình buid như định nghĩa các biến môi trường, trình biên dịch, máy đích để chạy code sau khi biên dịch.

Bạn có thể nghĩ 1 task như 1 function sẽ được thiết lập để chạy trước hoặc sau một function (task) khác.

Trước khi chúng ta bắt đầu với ví dụ Hello World. Tôi muốn nhắc lại rằng, cái mà  bitbake sẽ đọc và hiểu là các file metadata. Nó có cú pháp riêng của nó giống như một ngôn ngữ lập trình vậy. Nó không giống như các chương trình shell phổ biến khác, đừng nhầm lẫn đấy.

Bạn có thể đọc thêm về ở metadata section của bitbake manual (đang được dịch)

## 2. LẤY BITBAKE VỀ

```bash 
wget http://git.openembedded.org/bitbake/snapshot/bitbake-1.16.0.tar.bz2  
tar xvf bitbake-1.16.0.tar.bz2  
cd bitbake-1.16.0&lt;/code&gt;
```

Thay vì cài đặt, trong bài này ta sẽ chạy nó từ thư mục buil bằng cách add PATH.

```bash 
# FIXME: how to disable xml doc generation? It takes a little time for this doc.  
python setup.py build  
export PYTHONPATH=`pwd`/build/lib
```

## 3. Build thử
Chúng ta sẽ đi vào một project cụ thẻ.  _Hello World Project_
Bắt đầu với những ví dụ kinh điển nhất, chúng ta sẽ tìm hiểu cách BitBake làm việc bằng cách chơi với chúng. 
Trong phần này, chúng ta sẽ cover những phần sau:

- Task là cái gì, được định nghĩa như thế nào, task được ghi đè như thế nào.  
- Flag là cái gì, Flag được sử dụng như thế nào trong BitBake.  

Với những thành phần như vậy chúng ta sẽ xem làm thế nào chúng được kết hợp với nhau trong một hệ thống build để taojra một Embedded Linux Image. Sau đó chúng ta sẽ vào xem Code của OpenEmbedded và xem những gì đã học được thực hiện như thế nào.

Chú ý : Phần này được viết với sự trợ giúp từ những <a href="http://www.mail-archive.com/yocto@yoctoproject.org/msg09379.html">email </a>được gửi trong Yocto Project Mailing List.

Trong thư mục bitbake được giải nén ở trên, bạn sẽ thấy có file class/base.class.  
Nó bao gồm 1 vài hàm cho việc in ra các info, warning, 3 hàm showdata, listtasks, build.

Khi bitbake được chay để build một recipe, mặc định rằng bất kì recipe nào cũng kế thừa file này. Đây là một nơi lý tưởng cho chúng ta định nghĩa build logic. Nếu bạn đã quen với việc build prorams cho UNIX-like OS, bạn chắc chắn sẽ biết rằng, việc build bao gồm những công đoạn sau. 

- Fetch source về.
- Giải nén
- Patch
- Cấu hình
- Biên dịch (making)
- Cài đặt (nếu cần thiết)

Giờ cùng xem những công được này được thực hiện như thế nào.  
### 3.1 Thêm task
```
Syntax : addtask
```
Câu trên chỉ giống như khai bao tên hàm.  
Code của task được định nghĩa như sau:

```python  
do_ {

}  
```

Phần code này có thể là python hoặc shell.  
Chúng ta cùng định nghĩa build logic.  
Giờ edit file `base.bbclas` và thay đổi nó thành như thế này:

```python
bbplain() {
  echo "$*"  
}

bbnote() {
  echo "NOTE: $*"  
}

bbwarn() {
  echo "WARNING: $*"  
}

bberror() {
  echo "ERROR: $*"  
}

bbfatal() {
  echo "ERROR: $*"
  exit 1  
}

addtask fetch  
python base_do_fetch() {
  bb.note("BASE_DO_FETCH")  
}

addtask unpack after do_fetch  
python base_do_unpack() {
  bb.note("BASE_DO_UNPACK")  
}

addtask patch after do_unpack  
base_do_patch() {
  bbnote "BASE_DO_PATCH"  
}

addtask configure after do_patch  
base_do_configure() {
  bbnote "BASE_DO_CONFIGURE"  
}

addtask make after do_configure  
base_do_make() {
  bbnote "BASE_DO_MAKE"  
}

addtask install after do_make  
base_do_install() {
  bbnote "BASE_DO_INSTALL"  
}

addtask build after do_install  
base_do_build() {
  bbnote "DO_BUILD"  
}

EXPORT_FUNCTIONS do_fetch do_unpack do_patch do_configure do_make do_install do_build

```

Như bạn đã thấy, ở code ở trên có 2 hàm là `python`, còn lại là `shell`.  
Việc in ra tên function ở trên khiến cho việc đọc lại logic dễ dàng hơn từ logs.

Chúng ta sẽ thấy được build logic từ cách làm trên nhưng sẽ tốt hơn nếu tách riêng chúng ra thành module cho việc logging.  
Chúng ta cùng tạo một **logging.bblass** trong thư mục _classes/_. File **logging.bbclass** trông sẽ giống như sau:

```python
`bbplain() {
  echo "$*"  
}`

bbnote() {
  echo "NOTE: $*"  
}

bbwarn() {
  echo "WARNING: $*"  
}

bberror() {
  echo "ERROR: $*"  
}

bbfatal() {
  echo "ERROR: $*"
  exit 1  
}

```

Chúng ta sẽ kế thừa module logging bằng cách như sau, file này là base.bbclass như đã được  
nói đến ở trên.

```python
inherit logging

addtask fetch  
python base_do_fetch() {
  bb.note("BASE_DO_FETCH")  
}
```

Khi file này được parse, các hàm được định nghĩa ở loggging sẽ được kế thừa.  
Kế thừa là một trong rất nhiều cách để thực hiện module hóa trong BitBake.

### 3.2 Thêm 1 lớp mới

Tạo một thư mục tên **meta-test** trong thư mục hiện tại (thư mục giải nén bitbake).  
Giờ chúng ta phải cho BitBake biết rằng chúng ta có 1 layer.  
Cách làm như sau, tạo file **conf/bblayers.conf** với nội dung như sau:  

```python
BBPATH := "${TOPDIR}"  
BBFILES ?= ""  
BBLAYERS = " \  
${TOPDIR}/meta-test \  
"
```

Biến ${TOPDIR} ta thấy chưa được nhắc đến. Khi parse đến file này mà biến vẫn chưa định nghĩa, thì BitBake sẽ tự địn nghĩa.  
Trong source code của BitBake có đoạn như sau:

```
_lib/bb/parse/parse_py/ConfHandler.py:36._   
(...)

def init(data):  
topdir = data.getVar('TOPDIR')  
if not topdir:  
data.setVar('TOPDIR', os.getcwd())

(...)
```

Bạn sẽ thấy rằng, nếu chưa được định nghĩa thì TOPDIR sẽ là đường dẫn đến thư mục chạy BitBake.  
Vậy các biến được gán giá trị ở đây :BBPATH, BBFILES, BBLAYERS sẽ được sử dụng để nhằm mục đích gì?  
Trong manual của Bitbake có đoạn như sau:
http://docs.openembedded.org/bitbake/html/ch02s03.html

> Bitbake will first search the current working directory for an optional  
> "conf/bblayers.conf" configuration file. This file is expected to contain a  
> BBLAYERS variable which is a space delimited list of 'layer' directories. For  
> each directory in this list a "conf/layer.conf" file will be searched for and  
> parsed with the LAYERDIR variable being set to the directory where the layer was  
> found. The idea is these files will setup BBPATH and other variables correctly  
> for a given build directory automatically for the user.
>
> Bitbake will then expect to find 'conf/bitbake.conf' somewhere in the user  
> specified BBPATH. That configuration file generally has include directives  
> to pull in any other metadata (generally files specific to architecture,  
> machine, local and so on.

dịch ra là

> Đầu tiên BitBake sẽ search trong thư mục hiện tại để tìm file conf/bblayers.conf.  
> Nhiệm vụ của file này thông thường sẽ chứa giá trị biến BBLAYERS. Biến này là một danh sách  
> các thư mục của các layer.(như ta vừa tạo ở trên đó là 1 thư mục cho 1 layer).  
> Mỗi thư mục trong danh sách trên, BitBake sẽ tìm kiếm lần lượt.  
> Trong mỗi thư mục, nó sẽ tìm kiếm conf/layers.conf trước tiên. Trong file layers.conf này  
> nó sẽ đọc để gán giá trị cho biến LAYERDIR. Biến này, tiếp theo đó được sử dụng để xác định  
> thư mục có chứa các LAYER.  
> Ý tưởng việc sinh ra các file này nhằm mục đích sẽ gán giá trị biến BBPATH và các biến môi trường khác  
> một cách chính xác để phục vụ việc build.

Biến BBPATH cũng tương tự như biến môi trường PATH.
Các file configuration có thể chưa các file configure khác.  
Việc xác định đường dẫn tuyết đối của các file này được thực hiện bằng thao tác search  
đường dấn tương đối trong các thư mục được liệt kê trong BBPATH, cái tìm thấy trước sẽ được sử dụng.

Như đã nói ở trên, ta cần tạo file conf/layer.conf cho layer meta-test.  
Nội dung sẽ như sau:  

```
BBPATH .= ":${LAYERDIR}"

BBFILES += "${LAYERDIR}/recipes-_/_/_.bb \  
${LAYERDIR}/recipes-_/_/_.bbappend"

BBFILE_COLLECTIONS += "test"  
BBFILE_PATTERN_test := "^${LAYERDIR}/"  
BBFILE_PRIORITY_test = "5"
```

Chúng ta sẽ thêm layer của chúng ta vào BBPATH để Bitbake có thể tìm thấy.  
Bằng biến BBFILES, chúng ta sẽ chỉ cho Bitbake thấy rằng, chúng ta có những file với mẫu như vậy trong thư mục layer của chúng ta.

Chúng ta hãy tạo một recipe trong thư mục.  
`mkdir -p meta-test/recipes-example/firstrecipe  
vim meta-test/recipes-example/firstrecipe/firstrecipe_0.0.bb`

File **firstrecipe_0.0.bb** sẽ có nội dung như sau:  
`DESCRIPTION = "Our first recipe for bitbake hello world"`

Ta sẽ có thư mục meta-test được tổ chức như dưới đây:

```bash
|-- classes  
| |-- base.bbclass  
|  -- logging.bbclass  
|-- conf  
| |-- bblayers.conf  
| -- bitbake.conf  
|-- meta-test  
| |-- conf  
| |  -- layer.conf  
| -- recipes-example  
| -- firstrecipe  
| -- firstrecipe_0.0.bb  
```

Với cấu trúc thư mục ở trên, ta có thể chạy Bitbake được rồi.  
`./bin/bitbake firstrecipe -vDD`

Đến đây, sau khi chạy thành công ta ta sẽ thấy được logic buid package thông qua các log.  
Đường dẫn log **tmp/work/firstrecipe-0.0-r0/temp/**  
trong đường dẫn trên kiểm tra file "**run_TASK.PID**" ta sẽ thấy được cái gì đã được chạy trong  
các task, file **log.task_order** sẽ định nghĩa thứ tự chạy các task của chúng ta.

Nào, giờ hãy thêm code vào các task đã được định nghĩa để hiểu cơ bản chức năng của hệ thống build này.

## 4. Ghi đè task

Việc định nghĩa logic build các package chưa đủ để tạo một hệ thống build thực sự nào.  
Có nhiều cách để thực hiện việc cấu hình, biên dịch, cài đặt thông qua các tool kinh điển như autotoool,  
cmake, scons, etc.. Ta sẽ override các hàm **do_configure, do_make, do_instal** cho các recipe khác nhau.

Giả sử **firstrecipe** sử dụng autotools.  
có nghĩa là nó sẽ có **configure script**, sau đó chúng ta biên dịch sử dụng **make**,  
và cài đặt sử dụng **make install**.  
Chúng ta sẽ bắt đầu ngay với việc tạo file **autotools.bbclass** trong **classes/**  
với nội dung như sau:

```python
autotools_do_configure() {  
  bbnote "AUTOTOOLS_CONFIGURE"  
}

autotools_do_make() {  
  bbnote "AUTOTOOLS_MAKE"  
}

autotools_do_install() {  
  bbnote "AUTOTOOLS_INSTALL"  
}

EXPORT_FUNCTIONS do_configure do_make do_install

```

Kế thừa autotools package trong **firstrecipe**  
```python
DESCRIPTION = "Our first recipe for bitbake hello world"`

inherit autotools
```

chúng ta cùng sẽ chạy lại BitBake với các hàm kế thừa từ autotool package

`./bin/bitbake firstrecipe -vDD`

Kiểm tra 2 file log trong thư mục temp để thấy các hàm mới đã được gọi (một cách tự động)  
Khi không có kế thừa trong recipi thì code chạy sẽ như thế này.  
```python

do_configure() {  
  base_do_configure
}

base_do_configure() {  
  bbnote "BASE_DO_CONFIGURE"
}

bbnote() {  
  echo "NOTE: $*"
}
```
Hàm của lớp base BitBake base class sẽ được gọi.  
Khi sử dụng kế thừa thì nó thế này:

```python
do_configure() {  
  base_do_configure
}

base_do_configure() {  
  bbnote "BASE_DO_CONFIGURE"
}

bbnote() {  
  echo "NOTE: $*"
}
```

Việc gọi các hàm từ lớp kế thừa được thực hiện thông qua từ khóa EXPORT_FUNCTIONS.  
Khi từ khóa này được sử dụng được dunjgthif do_xxx sẽ được map đến autotools_do_xxx.

Kết luận nhỏ : Đến đây, ta có thể hiểu được bộ khung hay logic của quá trình build package sử dụng  
BitBake.

## 5. Áp dụng trên OpenEmbedded

OpenEmbedded(OE) cung cấp một hệ thống build hoàn chỉnh và rất nhiều recipe.  
Nó có rất code, script để phục vụ các giai đoạn trong quá trình như:  
tải source code, patching (chỉnh sửa lại để phù hợp mục đích build thường là để  
tương tích với target), biên dịch, cài đặt, đóng gọi package ipk, deb, rpm, cả việc  
tạo ảnh hệ thống nữa.  
Ngoài ra OE còn cung cấp các tính năng khác như caching,... Sẽ bàn ở một bài khác.

[![](/post/images/yocto-environment.png)](https://www.yoctoproject.org/docs/current/yocto-project-qs/figures/yocto-environment.png) Cấu trúc của OpenEmbedded

Từ hình ta thấy các quá trình chuyển từ code sang các package, image, SDK.  
Trong bài này, chúng ta không bàn về chi tiết.  
Từ những điều đã biết ở trên về Hell World, hãy thử chúng được áp dụng như thế nào trong OE.

Tải source code  

```bash
wget http://downloads.yoctoproject.org/releases/yocto/yocto-1.3/poky-danny-8.0.tar.bz2  
tar xvf poky-danny-8.0.tar.bz2  
cd poky-danny-8.0
```

Như chúng ta đã nói ở trên, bitbake.conf sẽ được parse khi bitbake được chạy. File là một trong những file cấu hình quan trọng của OE.  
Vị trí file này

**meta/conf/bitbake.conf**

File cấu hình này có thể được module hóa, ở đây chúng được chia ra làm nhiều file.  
Nội dung file sẽ có dòng như thế này:

```bash
include conf/build/${BUILD_SYS}.conf  
include conf/target/${TARGET_SYS}.conf  
include conf/machine/${MACHINE}.conf  
include conf/machine-sdk/${SDKMACHINE}.conf  
include conf/distro/${DISTRO}.conf
```

Các file configure này sẽ được search trong các thư mục được định nghĩa ở BBPATH.  
Điều đó có nghĩa là khi BitBake chạy, những file cấu hình này sẽ được searched trong tất các  
các layer bạn định nghĩa. Cái tìm thấy đầu tiên sẽ được sử dụng.  
Nhớ rằng, đường dẫn của tất cả các LAYER sẽ được thêm vào BBPATH.  
và có đến 99% số biến được định nghĩa trong `bitbake.conf`, còn lại là các file include.

## 6. Các task

Để hiểu về kiến trúc của OE, điều quan trọng đầu tiên ta phải biết là các task được định nghĩa ở đâu.  
Chúng ta đã được thấy file **base.bbclass** là một nơi lý tưởng để định nghĩa logic và các code chúng.  
tất cả các file bbclass được định nghĩa trong _meta/classes/_.  
Mở file base.bbclass và search "addtask" keyword, ta sẽ thấy nội dung sẽ giống như thế này:

```python
...  
addtask fetch

...  
addtask unpack after do_fetch

...  
addtask configure after do_patch

...  
addtask compile after do_configure

...  
addtask install after do_compile

...  
addtask build after do_populate_sysroot

...  
addtask cleansstate after do_clean

...  
addtask cleanall after do_cleansstate
```

Tuy nhiên, trong file này không chỉ có những task này được định nghĩa.  
Có những task được định nghĩa trong **{patch, staging}.**bbclass.  
Chúng ta cùng nhau xem task nào được định nghĩa.

```python
// patch.bbclass  
addtask patch after do_unpack`

// staging.bbclass  
addtask populate_sysroot after do_install  
addtask do_populate_sysroot_setscene
```

Những task này cũng vẫn chưa đủ. Trong file bitbake.còn, ta thấy file conf.distro/defaultsetup.conf được  
include. Nó định nghĩa các packagage sau đây được kế thừa.  

`_package_ipk, insane, debian devshell sstate license_  `

```python
USER_CLASSES ?= ""  
PACKAGE_CLASSES ?= "package_ipk"  
INHERIT_INSANE ?= "insane"  
INHERIT_DISTRO ?= "debian devshell sstate license"  
INHERIT += "${PACKAGE_CLASSES} ${USER_CLASSES} ${INHERIT_INSANE} ${INHERIT_DISTRO}"
```

Trong các bbclass file, có 1 file liên quan tưới việc quản lý các gói/  
Ta cùng tìm hiểu 1 loại dễ nhất là package_ipk.bblcass. File này được kế thừa package.bbclass.  
File package.bbclass sẽ định nghĩa logic build, như thế này:

```python
...
addtask package before do_build after do_install

...  
addtask do_package_setscene

...  
addtask package_write before do_build after do_package
```

Thứ tự là install -> packaging.  
Vì vậy task cũng đượ địnhg nghĩa như vậy.  
Với mỗi loại package khác nhau, ipk, deb, rpm...  
Chúng ta cùng tìm hiểu loại dễ nhất là ipk.  
Nó định nghĩa như sau :

```python
...  
addtask do_package_write_ipk_setscene

...  
python do_package_write_ipk () {  
bb.build.exec_func("read_subpackage_metadata", d)  
bb.build.exec_func("do_package_ipk", d)  
}

...  
addtask package_write_ipk before do_package_write after do_package
```

Như bạn thấy, khi package_write_ipk được chạy, nó sẽ gọi **read_subpackage_metadata**  
và thực hiện hàm do_package_ipk để tạo ipk package.  
Nhưng gói khác cũng tương tự như vậy.

## 7. Package Group

Quan hệ giữa các gọi được sử dụng để cung cấp một gói ảo (virtual package), cái mà có cùng chức năng.  
cái này xử lý thông qua **package groups** trong OE.  
Package group có thể được xem như package ảo, chúng có các gói phụ thuộc.

Package groups nằm ở _recipes-packagegroups_. Chúng là các bitbake recipes như "busybox" hoặc  
"tinylogin". Tuy nhiên chúng không có source để build, chúng phụ thuộc vào những gói khác để cung cấp  
chức năng. Bên dưới là danh sách các package groups trong OE-core layer  
ta hãy xem có những group nào

```bash
% find meta/recipes-*/packagegroups -type d | xargs tree --charset=utf

meta/recipes-core/packagegroups  
|-- nativesdk-packagegroup-sdk-host.bb  
|-- packagegroup-base.bb  
|-- packagegroup-core-boot.bb  
|-- packagegroup-core-buildessential.bb  
|-- packagegroup-core-nfs.bb  
|-- packagegroup-core-sdk.bb  
|-- packagegroup-core-ssh-dropbear.bb  
|-- packagegroup-core-ssh-openssh.bb  
|-- packagegroup-core-standalone-sdk-target.bb  
|-- packagegroup-core-tools-debug.bb  
|-- packagegroup-core-tools-profile.bb  
|-- packagegroup-core-tools-testapps.bb  
|-- packagegroup-cross-canadian.bb  
|-- packagegroup-self-hosted.bb  
meta/recipes-devtools/packagegroups-- packagegroup-core-device-devel.bb  
meta/recipes-extended/packagegroups  
|-- packagegroup-core-basic.bb  
|-- packagegroup-core-lsb.bb  
meta/recipes-gnome/packagegroups  
|-- packagegroup-core-sdk-gmae.bb  
|-- packagegroup-core-standalone-gmae-sdk-target.bb`-- packagegroup-sdk-gmae.inc  
meta/recipes-graphics/packagegroups  
|-- packagegroup-core-clutter.bb  
|-- packagegroup-core-gtk-directfb.bb  
|-- packagegroup-core-x11-base.bb  
|-- packagegroup-core-x11-xserver.bb  
|-- packagegroup-core-x11.bb  
meta/recipes-qt/packagegroups  
|-- nativesdk-packagegroup-qte-toolchain-host.bb  
|-- packagegroup-core-qt.bb  
|-- packagegroup-core-qt4e.bb-- packagegroup-qte-toolchain-target.bb  
meta/recipes-sato/packagegroups  
|-- packagegroup-core-x11-sato.bb
```
Quá nhiều, ta chỉ xét 1 cái thôi. Đó là **packagegroup-core-boot**  
Đây là group sẽ yêu cầu các package để có thể boot hệ thống.  
Nào cùng tìm hiểu về nó

```python
meta/recipes-core/packagegroups/packagegroup-core-boot.bb  

#``Copyright (C) 2007 OpenedHand Ltd.
==================================`

`#`

SUMMARY = "Minimal boot requirements"  
DESCRIPTION = "The minimal set of packages required to boot the system"  
LICENSE = "MIT"  
DEPENDS = "virtual/kernel"  
PR = "r10"

inherit packagegroup

PACKAGE_ARCH = "${MACHINE_ARCH}"

#

Set by the machine configuration with packages essential for device bootup
==========================================================================

#  
MACHINE_ESSENTIAL_EXTRA_RDEPENDS ?= ""  
MACHINE_ESSENTIAL_EXTRA_RRECOMMENDS ?= ""

For backwards compatibility after rename
========================================

RPROVIDES_${PN} = "task-core-boot"  
RREPLACES_${PN} = "task-core-boot"  
RCONFLICTS_${PN} = "task-core-boot"

Distro can override the following VIRTUAL-RUNTIME providers:
============================================================

VIRTUAL-RUNTIME_dev_manager ?= "udev"  
VIRTUAL-RUNTIME_login_manager ?= "tinylogin"  
VIRTUAL-RUNTIME_init_manager ?= "sysvinit"  
VIRTUAL-RUNTIME_initscripts ?= "initscripts"  
VIRTUAL-RUNTIME_keymaps ?= "keymaps"

RDEPENDS_${PN} = "\  
base-files \  
base-passwd \  
busybox \  
${@base_contains("MACHINE_FEATURES", "rtc", "busybox-hwclock", "", d)} \  
${VIRTUAL-RUNTIME_initscripts} \  
${@base_contains("MACHINE_FEATURES", "keyboard", "${VIRTUAL-RUNTIME_keymaps}", "", d)} \  
modutils-initscripts \  
netbase \  
${VIRTUAL-RUNTIME_login_manager} \  
${VIRTUAL-RUNTIME_init_manager} \  
${VIRTUAL-RUNTIME_dev_manager} \  
${VIRTUAL-RUNTIME_update-alternatives} \  
${MACHINE_ESSENTIAL_EXTRA_RDEPENDS}"

RRECOMMENDS_${PN} = "\  
${MACHINE_ESSENTIAL_EXTRA_RRECOMMENDS}"
```

Biến **RDEPENDS_${PN}** chưa các package sẽ bitbake build.  
Các package này cũng được control bằng các tham số được định nghĩa ở trên như  
rtc, keyboard,....

## 8. Minimal image

Các recipes Image, tức là chưa đầy đủ các package cho 1 hệ thống.  
nằm ở **recipes-*/images/**. Có nhiều loại image với các mục đích khác nhau.  
Danh sách có thể được list ra như sau

```bash
% find meta/recipes-*/images -type d | xargs tree --charset=utf

meta/recipes-core/images  
|-- build-appliance-image  
| |-- Yocto_Build_Appliance.vmx  
| |-- Yocto_Build_Appliance.vmxf  
|-- build-appliance-image.bb  
|-- core-image-base.bb  
|-- core-image-minimal-dev.bb  
|-- core-image-minimal-initramfs.bb  
|-- core-image-minimal-mtdutils.bb-- core-image-minimal.bb  
meta/recipes-core/images/build-appliance-image  
|-- Yocto_Build_Appliance.vmx  
|-- Yocto_Build_Appliance.vmxf  
meta/recipes-extended/images  
|-- core-image-basic.bb  
|-- core-image-lsb-dev.bb  
|-- core-image-lsb-sdk.bb-- core-image-lsb.bb  
meta/recipes-graphics/images  
|-- core-image-clutter.bb  
|-- core-image-gtk-directfb.bb  
|-- core-image-x11.bb  
meta/recipes-qt/images-- qt4e-demo-image.bb  
meta/recipes-rt/images  
|-- core-image-rt-sdk.bb  
|-- core-image-rt.bb  
meta/recipes-sato/images  
|-- core-image-sato-dev.bb  
|-- core-image-sato-sdk.bb-- core-image-sato.bb
```

Minimal image là dễ hiểu nhất. Chúng ta sẽ tìm hiểu 1 chút về nó.  
core-image-minimal.  
Vậy làm thế nào core-image-minimal được built.

File recipie của core-image-minimal như sau

```bash

meta/recipes-core/images/core-image-minimal.bb  

DESCRIPTION = "A small image just capable of allowing a device to boot."

IMAGE_INSTALL = "packagegroup-core-boot ${ROOTFS_PKGMANAGE_BOOTSTRAP} ${CORE_IMAGE_EXTRA_INSTALL}"

IMAGE_LINGUAS = " "

LICENSE = "MIT"

inherit core-image

IMAGE_ROOTFS_SIZE = "8192"

remove not needed ipkg informations
===================================

ROOTFS_POSTPROCESS_COMMAND += "remove_packaging_data_files ; "
```

Như các bạn thấy, biến IMAGE_INSTALL chưa **packagegroup-core-boot**  
Và nó cũng kế thừa một lớp core-image bbclass.

Tìm hiểu qua về core-image

```python
meta/classes/core-image.bbclass

Common code for generating core reference images
================================================


Copyright (C) 2007-2011 Linux Foundation
========================================`


LIC_FILES_CHKSUM = "file://${COREBASE}/LICENSE;md5=3f40d7994397109285ec7b81fdeb3b58 \  
file://${COREBASE}/meta/COPYING.MIT;md5=3da9cfbcb788c80a0384361b4de20420"

IMAGE_FEATURES control content of the core reference images
===========================================================


By default we install packagegroup-core-boot and packagegroup-base packages - this gives us
===========================================================================================

working (console only) rootfs.
==============================


Available IMAGE_FEATURES:
=========================


- x11 - X server
================

- x11-base - X server with minimal environment
==============================================

- x11-sato - OpenedHand Sato environment
========================================

- tools-debug - debugging tools
===============================

- tools-profile - profiling tools
=================================

- tools-testapps - tools usable to make some device tests
=========================================================

- tools-sdk - SDK (C/C++ compiler, autotools, etc.)
===================================================

- nfs-server - NFS server
=========================

- ssh-server-dropbear - SSH server (dropbear)
=============================================

- ssh-server-openssh - SSH server (openssh)
===========================================

- qt4-pkgs - Qt4/X11 and demo applications
==========================================

- package-management - installs package management tools and preserves the package manager database
===================================================================================================

- debug-tweaks - makes an image suitable for development, e.g. allowing passwordless root logins
================================================================================================

- dev-pkgs - development packages (headers, etc.) for all installed packages in the rootfs
==========================================================================================

- dbg-pkgs - debug symbol packages for all installed packages in the rootfs
===========================================================================

- doc-pkgs - documentation packages for all installed packages in the rootfs
============================================================================

PACKAGE_GROUP_x11 = "packagegroup-core-x11"  
PACKAGE_GROUP_x11-base = "packagegroup-core-x11-base"  
PACKAGE_GROUP_x11-sato = "packagegroup-core-x11-sato"  
PACKAGE_GROUP_tools-debug = "packagegroup-core-tools-debug"  
PACKAGE_GROUP_tools-profile = "packagegroup-core-tools-profile"  
PACKAGE_GROUP_tools-testapps = "packagegroup-core-tools-testapps"  
PACKAGE_GROUP_tools-sdk = "packagegroup-core-sdk packagegroup-core-standalone-sdk-target"  
PACKAGE_GROUP_nfs-server = "packagegroup-core-nfs-server"  
PACKAGE_GROUP_ssh-server-dropbear = "packagegroup-core-ssh-dropbear"  
PACKAGE_GROUP_ssh-server-openssh = "packagegroup-core-ssh-openssh"  
PACKAGE_GROUP_package-management = "${ROOTFS_PKGMANAGE}"  
PACKAGE_GROUP_qt4-pkgs = "packagegroup-core-qt-demoapps"

IMAGE_FEATURES_REPLACES_foo = 'bar1 bar2'
=========================================

Including image feature foo would replace the image features bar1 and bar2
==========================================================================

IMAGE_FEATURES_REPLACES_ssh-server-openssh = "ssh-server-dropbear"

IMAGE_FEATURES_CONFLICTS_foo = 'bar1 bar2'
==========================================

An error exception would be raised if both image features foo and bar1(or bar2) are included
============================================================================================

python __anonymous() {

Ensure we still have a splash screen for existing images
========================================================

if base_contains("IMAGE_FEATURES", "apps-console-core", "1", "", d) == "1":  
bb.warn("%s: apps-console-core in IMAGE_FEATURES is no longer supported; adding \"splash\" to enable splash screen" % d.getVar("PN", True))  
d.appendVar("IMAGE_FEATURES", " splash")  
}

CORE_IMAGE_BASE_INSTALL = '\  
packagegroup-core-boot \  
packagegroup-base-extended \  
\  
${CORE_IMAGE_EXTRA_INSTALL} \  
'

CORE_IMAGE_EXTRA_INSTALL ?= ""

IMAGE_INSTALL ?= "${CORE_IMAGE_BASE_INSTALL}"

inherit image

Create /etc/timestamp during image construction to give a reasonably sane default time setting
==============================================================================================

ROOTFS_POSTPROCESS_COMMAND += "rootfs_update_timestamp ; "

Zap the root password if debug-tweaks feature is not enabled
============================================================

ROOTFS_POSTPROCESS_COMMAND += '${@base_contains("IMAGE_FEATURES", "debug-tweaks", "", "zap_root_password ; ",d)}'

Allow openssh accept empty password login if both debug-tweaks and ssh-server-openssh are enabled
=================================================================================================

ROOTFS_POSTPROCESS_COMMAND += '${@base_contains("IMAGE_FEATURES", "debug-tweaks ssh-server-openssh", "openssh_allow_empty_password; ", "",d)}'
```

Do biến `IMAGE_INSTALL` được định nghĩa trước khi kế thừa lớp recipe `core-image`. Vì thế, giá trị đó vẫn không bị thay đổi khi  
parse đến `core-image.bbclass`.  
Ỏ đây ta thấy có xử lý postprogress _rootfs_update_timestamp_  
Và cũng phải nhắc đến việc core-image kế thừa từ image

## 9. Vậy điều thú vị xảy ra ở đâu, có phải trong file image.bbclass??


Cuối cùng, đây là nơi mọi chuyện sẽ xảy ra. Hay nói ngắn gọn, ảnh sẽ được tạo từ ipk, deb, ..rpm package.  
Vì ipk được sử dụng trong ở trên rồi ) nên ta sẽ tiếp tục với ipk.  
Danh sách các package được đưa vào image được đưa ra trong file **package_ipk.bbclass**  
Nhưng gói trong đó được cài đặt sử dụng **opkg** vào thư mục **rootfs**

Vì file image.bbclas rất dài. Ta sẽ chỉ tập trung vào phần code quan trọng thôi đê hiểu quá trình tạo ra  
1 image.

Để tạo image cũng như cài đặt vào rootfs.  
Đầu tiên các package phải được build đã.

Tuy nhiên chúng ta không định nghĩa bất cứ phụ thuộc vào, mà chỉ là set giá trị biến IMAGE_INSTALL

```python
meta/classes/image.bbclass

...  
RDEPENDS += "${IMAGE_INSTALL} ${LINGUAS_INSTALL} ${NORMAL_FEATURE_INSTALL} ${ROOTFS_BOOTSTRAP_INSTALL}"

...

definition of which packages should be installed on image. PACKAGE_INSTALL
==========================================================================

variable is used in rootfs creation code in rootfs_ipk.bbclass
==============================================================

export PACKAGE_INSTALL ?= "${IMAGE_INSTALL} ${ROOTFS_BOOTSTRAP_INSTALL} ${FEATURE_INSTALL}"  
...  
addtask rootfs before do_build
```

Class này phụ thuộc vào biến IMAGE_INSTALL và các biến khác vì thế. Các package cần được build trước.  
Ta cũng thấy một task là _rootfs_ chạy trước task build. Vì thế khi mà tất cả các task khác được chạy  
xong thì do_rootfs sẽ bắt đầu việc tạo image.

Sau khi check một vài thư mục, do_rootfs sẽ gọi đến hàm tạo cho loại package tương tứng.

```python
meta/classes/image.bbclass, do_rootfs()

rootfs_${IMAGE_PKGTYPE}_do_rootfs
```

ví dụ  
trong trường của chúng ta sử dụng ipk, nó sẽ như thế này :  
**rootfs_ipk_do_rootfs**

Cùng xem 1 chút về hàm tạo **rootfs_ipk_do_rootfs** này  
rootfs_ipk_do_rootfs() tạo các thư mục cần thiết cho opkg làm việc, ghi một số file trạng thái và update chings nó  
bằng : _opk-cl update_.