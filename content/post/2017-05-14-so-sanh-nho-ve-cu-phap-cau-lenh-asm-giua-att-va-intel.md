---
layout: post
title: So sánh nhỏ về cú pháp câu lệnh ASM giữa AT&T và Intel
date: 2017-05-14 08:43:10.000000000 +09:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Basic
tags:
- Assembly
meta:
  _wpcom_is_markdown: '1'
  _oembed_91f1860b4e461b6c3e3a5b3c28f89928: "{{unknown}}"
  _oembed_2a3f597a0090887522110b46d94fc2ae: "{{unknown}}"
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '5027491600'
  _oembed_3558a61d65622cb44fb7ad9f15206b74: "{{unknown}}"
  _oembed_b3a7108e6afe6b87369b9930e61a519d: "{{unknown}}"
  _oembed_804e0f57dd83bec78e679c18b718001a: "{{unknown}}"
  _oembed_70b4d7ec10a562a955bed1f3d2dd9558: "{{unknown}}"
author:
  login: minatu
  email: pvta2pn@gmail.com
  display_name: minatu
  first_name: トウー
  last_name: フン・ヴァン
permalink: "/2017/05/14/so-sanh-nho-ve-cu-phap-cau-lenh-asm-giua-att-va-intel/"
---

**Link gốc**:  
http://www.imada.sdu.dk/Courses/DM18/Litteratur/IntelnATT.htm

**Người dịch**:  
Ngôn ngữ Assembly không phải là ngôn ngữ tốt để viết ứng dụng,nhất là bây giờ đã là năm 2017. Có quá nhiều thứ hiệu quả và nhanh hơn. Tuy nhiên, khi cần tìm hiểu ở mức hệ điều hành, rồi driver thiết bị, thì việc đọc được Assembly là rất hữu ích.

**Nội dung**

Cú pháp ASM giữa Intel và AT&T có rất nhiều điềm khác nhau rõ rệt. Điều này nhiều khi dẫn đến sự nhầm lẫn khi một người đã học một ASM của Intel trước rồi sang học ASM của AT&T hoặc ngoặc lại. Bài này, chúng ta sẽ xem xét những cái khác biệt cơ bản đó.

## 1. Về tiền tố (Prefixes)

Cú pháp Intel thì không bất cứ tiền tố nào.  
Còn cú pháp AT&T:  
- thanh ghi được theo sau tiền tố **%**  
- giá trị được theo sau tiền tố **$**

Cú pháp Intel sử dụng hậu tố 'h' sau giá trị cơ số 16, 'b' sau giá trị cơ số 2.  
Nếu giá trị đầu tiên của giá trị cơ số 16 là chữ thì nó phải được theo sao bằng tiền tố 0.

```C  
// Intex Syntax  
mov eax,1  
mov ebx,0ffh  
int 80h

// AT&T Syntax  
movl $1,%eax  
movl $0xff,%ebx  
int $0x80  
```

## 2. Về chiều của toán tử (Direction of Operands)

Chiều của toán tử theo cú pháp Intel ngược với cú pháp AT&T.  
Trong cú pháp Intel, toán tử đầu tiên (hay tham số đầu tiên) là đích. Toán tử thứ 2 là nguồn.  
Trong cú pháp AT&T thì ngược lại. Đầu tiên là nguồn, thứ 2 là đích.

**Tham khảo thêm**:  
Có một chú ý về [lệnh MOV](https://lazytrick.wordpress.com/2017/05/14/lenh-mov-trong-assembly/) về giá trị đích, nguồn do người dịch tổng hợp

Ví dụ:

```c  
//Intex Syntax  
instr dest,source  
mov eax,[ecx]

//AT&T Syntax  
instr source,dest  
movl (%ecx),%eax  
```

## 3. Cách chỉ định bộ nhớ (Memory Operands)

Cách chỉ định địa chỉ cũng khác nhau.  
Trong cú pháp Intel, thanh ghi chứa địa chỉ được bao bởi 2 dấu ngoặc vuông `[]`.  
Trong cú pháp AT&T, được bao bởi 3 dấu ngoặc đơn `()`.

Ví dụ:

```c  
//Intex Syntax  
mov eax,[ebx]  
mov eax,[ebx+3]

//AT&T Syntax  
movl (%ebx),%eax  
movl 3(%ebx),%e  
```

Kiểu của AT&T có vẻ phức tạo hơn kiểu của Intel.  
Kiểu Intel, cách viết sẽ có dạng sau: `segreg[base+index * scale + disp]`  
Kiểu AT&T, sẽ là: `%segreg:disp(base,index,scale)`

Các giá trị `index/scale/disp/segreg` có thể bị lược bỏ.  
`Scale` nếu không được chỉ định thì `index` phai được chỉ định.  
Giá trị mặc định của `Scale` là .

Ví dụ:

```c  
//Intel Syntax  
instr foo,segreg:[base+index*scale+disp]  
mov eax,[ebx+20h]  
add eax,[ebx+ecx*2h  
lea eax,[ebx+ecx]  
sub eax,[ebx+ecx*4h-20h]

//AT&T Syntax  
instr %segreg:disp(base,index,scale),foo  
movl 0x20(%ebx),%eax  
addl (%ebx,%ecx,0x2),%eax  
leal (%ebx,%ecx),%eax  
subl -0x20(%ebx,%ecx,0x4),%eax  
```

Như bạn cũng thấy, kiểu AT&T khá khó hiểu.  
Cách viết `[base + index*scale + disp] trong dễ hiểu hơn nhiều cách viết`disp(base, index, scale)\`

## 4. Về hậu tố

Cái này khác nhau nhiều đây.  
Cú pháp AT&T sử dụng hậu tố gợi nhớ cho câu lệnh để ám chỉ kích thước toán tử (DEST, SRC).  
`l` nghĩa là `long`  
`w` nghĩa là `word`  
`b` nghĩa là `byte`  
Cú pháp Intel thì lại sử dụng hậu tố gợi nhớ cho bộ nhớ, phía địa chỉ.  
ví dụ:

```C  
byte ptr  
word ptr  
dword ptr  
// word tương ứng với "long"  
```

Nó tương tự giống định nghĩa kiểu dữ liệu trongC.

Ví dụ so sanh:

```c  
//Intel Syntax  
mov al,bl  
mov ax,bx  
mov eax,ebx  
mov eax, dword ptr [ebx]

//AT&T Syntax  
movb %bl,%al  
movw %bx,%ax  
movl %ebx,%eax  
movl (%ebx),%eax  
```