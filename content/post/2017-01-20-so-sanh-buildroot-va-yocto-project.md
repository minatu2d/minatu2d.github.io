---
layout: post
title: So sánh giữa Buildroot và Yocto Project
date: 2017-01-20 01:22:46.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
- Buildroot
- Compile
- Embedded
- YoctoProject
tags: []
meta:
  _wpcom_is_markdown: '1'
  _publicize_job_id: '948608049'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _wp_old_slug: '1784'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2017/01/20/so-sanh-buildroot-va-yocto-project/"
---
Bài này sẽ dịch lại [Slide](http://events.linuxfoundation.org/sites/events/files/slides/belloni-petazzoni-buildroot-oe_0.pdf) thảo luận giữa 2 diễn giả là Alexandre Belloni, Thomas Petazzoni  
từ Free Electrons tại Embedded Linux Conference 2016.

So sánh giữa Buildroot và OpenEmbedded/Yocto Project

## 1. Điểm chung

*   Đều là build-system cho Embedded Linux.  
    Mục tiêu là có thể customize, build hoàn chỉnh một Embedded Linux System.  
    Bao gồm: filesystem, toolchain, kernel, bootloaders
*   Đều được build từ source
*   Sử dụng cross-compilation
*   Rất actively trong cả dự án đang maintained và phát triển.
*   Được sử dụng rộng dãi trong công nghiệp.
*   Tài liệu tốt, nhiều khóa đào tạo.
*   Sử dụng Free Software (phần mềm tự do)  
    

## 2. Khác nhau

### 2.1 Về tư tưởng chung

*   **Buildroot**:
    *   Tập trung mạnh vào sự đơn giản
    *   Dễ sử dụng dễ hiểu và mở rộng
    *   Các trường hợp đặc biệt sẽ được thực hiện qua extension scripts.
    *   Sử dụng được ác công nghệ, ngôn ngữ đang tồn tại hiện nay:  
        kconfig, make
    *   Minimallist : phương trâm càng nhỏ càng tốt
    *   Không phụ thuộc vào mục đích sử dụng
    *   Cộng đồng mở, nhưng có vendor hoặc tổ chức nào quản lý
*   **Yocto**
    *   Hỗ trợ nhiều kiến trúc phổ biến
        *   OpenEmbedded: Chỉ hỗ trợ qemu
        *   Yocto Project : Thêm những nền tảng máy khác
    *   Chỉ cung cấp core recipes, sử dụng _layer_ để mở rộng và thêm gói  
        cũng như nền tảng phần cứng.
    *   Việc custom chỉ xảy ra ở các layer riêng biệt
    *   Hệ thống build đa năng: cố gắng linh hoạt nhất có thể để handle hầu hết các  
        trường hợp sử dụng.
    *   Cộng đồng mở, nhưng vẫn được quản lý bởi Yocto Project Advisory Board,  
        vốn được tạo bởi các công ty tài trợ
    *   OpenEmbedded là một dự án cộng đồng độc lập

### 2.2. Về output

*   **Buildroot**
    *   Ouput chính là một root filesystem image
        *   Cũng có thể kèm toolchain, kernel image, bootloader đi kèm.
    *   Hỗ trợ nhiều định dạng: ext2/3/4, ubifs, iso9660, etc.
    *   Không có gói binary, không có hệ thống quản lý gói
        *   Nhiều người gọi là **firmware generator**
        *   Không thể update qua các gói
        *   Cần update toàn bộ hệ thống, như Android.
        *   Không nên tin tưởng việc update 1 phần.
*   **Yocto Project**:
    *   Tạo bản phân phố, đầu ra là một loại các package.
        *   Hệ thống quản lý là tùy chọn cho hệ thống đích
        *   Có thể cài đặt, update chỉ 1 phần của hệ thống
    *   Cũng có thể sinh **root filesystem images* thông qua cài đặt các tool tạo  
        image. Hỗ trợ: ext2/3/4, ubifs, iso9660, etc...cũng có cả VM Images,　vmdk, vdi  
        qcow2
    *   Cuối cùng, có thể tạo ra 1 _disk image_
    *   Cũng có thể sinh một **SDK** kèm theo image, cho phép app developer compile,  
        test ứng dụng của họ mà không cần tích hợp chúng trong quá trình build.  
        Nhưng SDK phải được đồng bộ với thay đổi của image.

### 2.3. Cấu hình (Configuration)

*   **Buildroot**:
    *   Sử dụng lại _kconfig_ từ Linux kernel
    *   Giao diện đơn giản thông qua {menu,x,b,g} config
    *   Toàn bộ configuration được lưu trong 1 file duy nhất .config/defconf
    *   Định nghĩa toàn bộ các khía cảnh của hệ thống: kiến trúc, kernel version/config,  
        bootloader, user-space packages, etc.
    *   make menuconfig, make -> đơn giản
    *   Build cùng một hệ thống cho nhiều máy khác nhau: sẽ hoàn toàn riêng biệt
        *   Cần 1 tool để sinh defconfig từ nhiều fragements
        *   Có thể làm được những không quá dễ
        *   Quá trình build hoàn toàn tách biệt cho mỗi máy
*   **Yocto Project**:
    *   Quá trình configuration được chia ra làm nhiều phần:
        *   Distribution configuration: cấu hình gói general, toolchain, chọn libc.
        *   Machine configuration: định nghĩa architecture, đặc tính machine, BSP.
        *   Image recipe: Quy định package nào sẽ được cài dặt trên hệ thống đích
        *   Local configuration: bản phân phối với máy build luôn, có bao nhiêu thread  
            có thể sử dụng khi compiling, có hay không xóa bỏ code sau khi build.
    *   Cần biêt về nhiều lớp khác nhau mới có thể sử dụng chúng.
    *   Cho phép build cùng 1 image cho nhiều máy hoặc nhiều distribution hoặc nhiều ảnh cho 1 máy

### 2.4. Layers (Lớp)

*   **Buildroot**:
    *   Không có khái niệm này
    *   Tất cả package được maintained từ các kho chính thức
        *   Cho phép chất lượng rất tốt nhờ được review bởi experts
    *   Sử dụng *BR2_EXTERNAL
        *   Cho phép lưu trữ định nghĩa package, cấu hình và các thông tin khác.
        *   Chỉ có 1 _BR2_EXTERNAL_
        *   Sử dụng cho các package custom hoặc proprietary package
        *   Chỉ có thể add, không cho phép override.
*   **Yocto Project**:
    *   Cơ chế layer cho phép sửa/thêm package mới hoặc tạo ảnh mới
    *   Xóa bỏ sự ngăn cách giữa core build system, BSP và các chỉnh sửa
    *   Scales được, bên thứ 3 có thể cung cấp layer với BSPs của họ hoặc một tập  
        recipes để handle những ứng dụng đặc trưng
    *   Sử dụng layer cùng nhau phải đảm bảo tính tương thích và dùng chung OE branch  
        base.
    *   Cần để ý chất lượng layer, việc review vẫn chưa có tính hệ thống
    *   OpenEmbedded Metadata Index đưa ra danh sách các layer, recipe, máy hiện  
        mà framework này hỗ trợ tại http://layers.openemdedded.org/layerindex
    *   Có cơ chế override cho phép điều chỉnh recipe dựa trên máy hoặc distro.

### 2.5. Về toolchain

*   Tự build toolchain, dựa trên gcc, lựa chọn được thư viện C Libraries  
    (glibc, uClibc, musl)
*   Sử dụng các toolchain build sẵn từ bên ngoài:
    *   Có vẻ bên Buildroot thì dễ dàng hơn vì nó có sẵn bên trong
    *   Chỉ thực sự support tốt với những layer từ các vendor trong Yocto Project
*   **Buildroot**: _không có trong slide_
*   **Yocto Project** : _không có trong slide_

### 2.6 Về mức độ phức tạp

*   **Buildroot**:
    
    *   Được thiết kế đơn giản
    *   Mọi chức năng được đưa ra cho core đều phân tích trên tỉ số về  
        tính hữu dụng/độ phức tạp.
    *   Core logic được viết hoàn toàn bằng make, có ít hơn 1000 dòng code nhưng chứa  
        đến 230 dòng comment rồi.
    *   Thực sự dễ dàng để hiểu cái gì sẽ được thực hiện, tại sao và làm thế nào  
        package được build
    *   Các shell script đơn giản thực hiện các nhiệm vụ như download, giải nén,  
        building, installing các thành phần phần mềm theo thứ tự.
    *   Tài liệu chi tiết, resource cũng như thảo luận dồi dào.
    *   1 giờ nói là đủ để miêu tả hết bên trong (ELCE 2014)
    *   Những feedback từ IRC: hơn hẳn Yocto, hầu hết đều ngạc nhiên về sự dễ dàng  
        của nó. Đó là thứ đầu tiên thu hút tôi.
*   **Yocto Project**:
    *   Có lẽ cần phải học cẩn thận hơn.
    *   Core là **bitbake**, một dự án tách biệt được viết trong python(60k line code)
    *   Một tập các class định nghĩa các task phổ biến
    *   Recipe được viết như là sự trộn lại giữa ngôn ngữ riêng của bitbake, python,  
        và shell script.
    *   Tài liệu chi tiết nhưng nhiều biết cấu hình quá.
    *   Không phải luôn luôn dễ dàng hiểu best pratices  
        Ví dụ: Poky không nên được sử dụng cho sản phẩm, cấu hình chỉnh sửa distro/image  
        không nên đặt ở local.conf, hoặc xóa tmp/
    *   Nhiều người vẫn bị loạn khái niệm (Yocto Project, Poky, OpenEmbedded, bitbake)

### 2.7 Số lượng/sự đa dạng của package

*   **Buildroot**:
    *   Có 1800+ packages
    *   Graphics: X.org, Wayland, Qt4/Qt5, Gtk2/Gtk3, EFL
    *   Multimedia: Gsstreamer 0.10/1.x, ffmpeg, Kodi, OpenGL support một lượng  
        lớn platform
    *   Ngôn ngữ: Python 2/3, PHP, Lua, Perl, Erlang, Mono, Ruby, Node.js
    *   Networking: Apache, Samba, Dovecot, Exim, CUPS, rất nhiều server tools
    *   Init System: Busybox (default), initsysv, systemd
    *   Không support 1 toolchal trên target
*   **Yocto Project**:
    *   Hàng ngàn recipe: Khoảng 2200 trên oe-core, meta-openemdedded,  
        meta-qt5. Còn trên Metadata Index có khoảng 8400 (bao gồm cả trùng)
    *   Hầu hết package giống bên Buildroot
    *   Có nhiều layer cho ngôn ngữ hơn như: Java, Go, Rust, smalltalk
    *   Vẫn có cả layer cho Qt3
    *   Có thêm meta-virtualization (Docker, KVM, LXC, Xen) và meta-openstack layer

### 2.8 Cách quản lý phụ thuộc

*   **Buildroot**:
    *   Theo hướng tối giản hóa
        *   Nếu 1 chức năng có thể được disabled, thì nó nên mặc định bị disabled.
    *   Rất nhiều phụ thuộc tự động
        *   Nếu bạn enable OpenSSL, bạn có thể tự động có được SSL Support cho tất  
            các các package khác mà bạn đã enabled và có sử dụng SSL Support
    *   Có thể tạo ra một filesystem nhỏ mà không cần mất công gì nhiều.
*   **Yocto Project**:
    *   Package configuration được thực hiện ở mức distribution
        *   Enable OpenSSL sẽ bất tất cả các gói phụ thuộc của nó.
        *   Nhưng vẫn có thể disable riêng lẻ từng package trong đó
        *   Nhưng được cái nó lại chỉ enable cho tính năng cho từng gói
    *   Mặc dù có thể sửa đổi ở mức machine, nhưng nên tránh điều này để portability
    *   Mỗi recipe có thể định nghĩa một tập các feature mặc định để tạo  
        ra một cấu hình mặc định cho riêng nó.

### 2.9 Update và security

*   **Buildroot**
    *   Release sau mỗi 3 tháng: 2 tháng phát triển, 1 tháng để làm cho ổn định
    *   Mỗi lần release bao gồm cả security update và major update
    *   Cũng có thể thay đổi cả core nữa.
    *   Không có phiên bản LTS nhận update security dài hơi.  
        User phải tự kiểm soát cái những package họ muốn.
    *   Script để đánh giá các lỗi CVE chưa được fixed trong Buildroot configuration  
        được đưa lên.
*   **Yocto Project**:
    *   Release sau mỗi 6 tháng: 1 vào tháng 4, 1 vào tháng 10.
    *   Lập kế hoạch, roadmap được chi tiết và public
    *   4 milestones trong 3 tháng, giữa M1 và final release
    *   Ít nhất phiên bản trước và bản release hiện tại được maintained.  
        Chúng sẽ nhận được các gói security và fixed quan trọng nhưng không có recipe  
        upgrade.
    *   Các bản release cũ sẽ được maintained bởi cộng đồng, gọi là thôi.

### 2.10 Phát hiện và xử lý các thay đổi về cấu hình (detection of configuration changes)

*   **Buildroot**:
    *   Không cố gắng để smart
    *   Khi có 1 thay đổi trong cấu hình, nó không có gắng tìm ra xem có cái nào  
        cần build lại hay không.
    *   Một khi package đã được build rồi, Buildroot sẽ không build lại nó nữa trừ  
        khi ban yêu cầu nó
    *   Khi có thay đổi lớn ở cấu hình, nó sẽ build lại hết.
        *   Cái này có thể gây phiền toái với hệ thống lớn
    *   Khi có thay đổi nhỏ thường sẽ được xử lý để không phải build lại toàn bộ
        *   Cho dù là nhỏ thì cũng phải hiểu rõ cái đã thay đổi
    *   Không có cách nào để chia sẻ những cái thực thi chung giữa các cấu hình khác  
        nhau. 1 Configure 1 build.
*   **Yocto Project**:
    *   **bitbake** giữ một trạng thái gọi là **shared State Cache** (sstate-cache) cho  
        phép cải thiện việc builds.
    *   Nó phát hiện thay đổi ở đầu vào của task bằng việc tạo checksum cho các đầu  
        vào đó.
    *   Cache này được chia sẻ giữa các lần build. Nếu build cho các máy tương tự  
        nhau sẽ nhanh hơn nhiều.
    *   Nó cũng có thể share trạng thái này giữa nhiều máy host khác nhau  
        (ví dụ: 1 build server và 1 developer machine). Là 1 cách tuyệt vời để tăng  
        tốc quá trình build full, vốn rât tốn thời gian.

### 2.11 Kiến trúc hỗ trợ (architecture support)

*   **Buildroot**:
    *   Một lượng lớn kiến trúc được support
    *   Có : ARM (64), MIPS, PowerPC(64), x86/x86-64
    *   Một số kiến trúc đặc trưng:
        *   Xtensa, Blackfin, ARC, m68k, SPARC, Microblaze, NIOSII
        *   ARM no MMU, đặc biệt ARMv7-M1
    *   Đóng góp từ các architecture vendors
        *   MIPS bởi Imagination Technologies
        *   PowerPC64 bởi IBM
        *   ARC bởi Synopsys
        *   Blackfin bởi Analog Devices
*   **Yocto Project**:
    *   Các kiến trúc chính: ARM, MIPS, PowerPC, x86 và 64-bit
    *   Trong các layer riêng rẽ:
        *   Microblaze, NIOSII
    *   Các silicon vendor main　BSP layer của họ:
        *   meta-intel
        *   meta-altera (cả ARM và NIOSII)
        *   meta-atmel, meta-fsl, meta-ti
        *   meta-xilinx (cả ARM và Microblaze)
    *   Từ cộng đồng:
        *   meta-rockchip, meta-sunxi

### 2.12 Build tối giản

*   **Buildroot**:  
    Cho qemu_arm_versatile_defconfig sẽ mất 15 phút 25 giây, ảnh file là 2MB.
    
*   **Yocto Project**:  
    Build core-image-minimal cho qemuarm mất 50 phút 47 giây, ảnh file là 4.9M.  
    Nếu có sstate-cache tồn tại sẵn rồi, thì mát 1 phút 21 giây.
    

### 2.13 Một số nội dung từ Q&A của Buildroot

*   Không hỗ trợ complier trên target
*   Không hỗ trợ các file header cho development trên target
*   Không hỗ trợ document trên target
*   Không hỗ trợ quản lý gói cài trên target