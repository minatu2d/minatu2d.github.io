---
layout: post
title: Bắt đầu về Docker
date: 2019-02-06 07:26:15.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
tags: []
meta:
  _wpcom_is_markdown: '1'
  timeline_notification: '1549437979'
  _publicize_job_id: '27333003684'
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2019/02/06/2015/"
---
Docker, là công nghệ rất nổi và có ảnh hưởng đến hầu hết developer.  
Được coi là một công nghệ ảo hóa ở mức hệ điều hành khi so với các phương pháp ảo hóa phần cứng khác.  
Theo mình hỏi, kĩ thuật để làm việc này không mới, nó dựa trên kĩ thuật **chroot** có được sử dụng từ khá lâu trong linux.

Nói là vậy, nhưng học để sử dụng thì cần bài bản chút, bài này mình sẽ tóm tắt lại trang **Overview** từ trang của **Docker**

## 0. Docker dùng cho mục đích gì?

## 1. Tối ưu hóa chu trình phát triển:

  
  

Phát triển (Dev) --> Kiểm thử (Test) --> Triển khai (Deploy) --> Sửa lỗi (Bug fixing) --> Kiểm thử

Khi sử dụng **container** (sẽ được giải thích ở phía dưới) do **docker** cũng cấp, tốc độ phát triển liên tục được thực hiện trơn tru hơn.

## 2. Đáp ứng triển khai và mở rộng (scaling)

**Container** của Docker cho phép chạy dễ dàng và nhẹ nhàng trên bất cứ môi trường điện toán nào.

## 3. Chạy được nhiều môi trường khác nhau

Vì lý do nhanh và nhẹ của nó nên tạo được nhiều môi trường ảo hóa hơn các công nghệ áo hóa khác trên cùng một phần cứng tính toán.

### 3.1. Docker Engine

Hoạt động theo mô hình **client-server**  
**Server** là ứng sẽ chạy suốt (ẩn).  
**Client** là ứng dụng được người sử dụng sẽ dùng để giao tiếp với **Server** bằng các **REST API**

*   ![](/post/images/engine-components-flow.png)
    

### 3.2. Kiến trúc của Docker

*   ![](/post/images/docker_architect.png)
    

Đây là kiến trúc của em nó:

Hãy giải thích những từ xuất hiện ở hình trên:

*   **DOCKER_HOST** : Phần Host hay Server
*   **Client** : Phần Client
*   **Registry** : Như một cái kho
*   **Docker daemon** : Một chương trình chạy ngầm quản lý các **Container** và **Images**
*   **Registry** : Nơi lưu trữ các **image** (ảnh) Docker. **Docker hub** là một ví dụ và là kho duy nhất được cài mặc định trên Docker lúc mới cài. Chúng ta có thể hoàn toàn tự tạo một kho của riêng mình.
*   **Images** : Là một danh sách chỉ thị viết theo quy tắc Docker quy định để tạo ra một **Docker container**. Thường, một ảnh sẽ dựa trên một ảnh khác rồi chỉnh sửa đi. Ví dụ: dựa trên ảnh **ubuntu**, bạn thêm các thành phần như Web server, ứng dụng vào để tạo thành ảnh khác.
*   **Container** : Là một **instance** chạy được của **image**. Nó được tạo, bật, tắt, di chuyển, xóa bỏ bằng các **Docker API** do phía **Client** tác động lên. Về cơ bản, các **container** độc lập với nhau về kết nối mạng. Một khi nó bị xóa, mọi thay đổi sẽ không được lưu lại trên thiết bị lưu trữ nữa.

### 3.3. Công nghệ sử dụng trong Docker

*   **Namespce** : Không gian chỉ định tên. Để cung cấp tài nguyên cho mỗi **container**. Docker sử dụng các tên sau trong **Linux**
    
    *   **pid** : cho quản lý tiến trình
    *   **net** : cho quản lý giao tiếp mạng (network)
    *   **ipc** : cho việc truy cập tài nguyên để giao tiếp giữa các **process**
    *   **mnt** : cho việc quản lý cacs điểm nối hệ thống file (filesystem mount point)
    *   **uts** : quản lý nhân, phiên bản
*   **Control groups** : Hay thường gọi là **cgroups**, là một công nghệ trong **Linux** cho phép giới hạn tài nguyên (vì trong **Linux** mọi thứ là file, nên ta có thể hiểu tài nguyên cũng là file) có thể truy cập của một ứng dụng
*   **Union file systems** : Nôm na là Hệ thống file kết hợp. Được **Docker** sử dụng cung cấp các lớp khi tạo **container**. Có nhiều biến thể khác nhau bao gồm: **AUFS**, **btrfs**, **vfs**, và **DeviceMapper**.
*   **Container format** : **Docker Engine** kết hợp tất cả những thứ trên gồm : **namespace**, **control groups**, **UnionFS** vào một định dạng. Hiện tại định dạng đang sử dụng là **libcontainer**. Trong tương lai, có thể nhiều định dạng khác được hỗ trợ như: **BSD Jails**, **Solaris Zones**.