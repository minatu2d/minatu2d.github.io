---
title: "ChatGPT là gì? Tại sao Developer nên lo lắng"
description: "ChatGPT là gì? Tại sao nên lo lắng về công việc developer"
date: "2023-07-07T15:30:00+09:00"
thumbnail: ""
toc: true
published: true
categories:
  - basic
  - ai
  - llm
tags:
  - chatgpt
  - developer
---

# Đăng kí sử dụng
- Vào địa chỉ chat.openai.com sau đó đăng kí 1 account
    - Có thể đăng kí bằng tài khoản Google, Microsoft hoặc Apple
    - Hoặc dùng email bất kì
- Ta sẽ có một giao diện chat để sử dụng.


# Hỏi thử
## Bảo ChatGPT tự giới thiệu về chính ChatGPT
- Hỏi : Viết một bài giới thiệu về ChatGPT cho Developer.
- Trả lời :

ChatGPT là một mô hình ngôn ngữ thông minh được phát triển bởi OpenAI dựa trên kiến trúc GPT-3.5. Với khả năng mô phỏng và tương tác ngôn ngữ tự nhiên, ChatGPT mang đến cho bạn một công cụ mạnh mẽ để xây dựng ứng dụng, dịch vụ và sản phẩm mới sử dụng trí tuệ nhân tạo.

Dựa trên hơn 175 tỷ tham số, ChatGPT hiểu và phản ứng với ngôn ngữ của bạn một cách linh hoạt và tự nhiên. Bạn có thể sử dụng ChatGPT để tạo ra các trợ lý ảo, hệ thống hỏi đáp tự động, chatbot và nhiều ứng dụng khác mà có thể tương tác với người dùng dễ dàng.

Với ChatGPT, bạn có thể gửi các câu truy vấn hoặc văn bản và nhận được phản hồi tức thì. Bạn có thể yêu cầu ChatGPT để thực hiện các tác vụ như viết văn bản, tạo mã, tìm kiếm thông tin, giải đáp câu hỏi, hoặc thậm chí chơi trò chơi với nó. ChatGPT có khả năng hiểu và tạo ra ngôn ngữ tự nhiên, giúp bạn xây dựng trải nghiệm tương tác tốt hơn cho người dùng cuối của mình.

Để tích hợp ChatGPT vào dự án của bạn, OpenAI cung cấp API ChatGPT, giúp bạn dễ dàng tương tác với mô hình từ ứng dụng hoặc trang web của mình. API cung cấp một giao diện đơn giản để gửi yêu cầu và nhận phản hồi từ ChatGPT, cho phép bạn tùy chỉnh ứng dụng theo nhu cầu của mình.

Tuy nhiên, hãy nhớ rằng ChatGPT cũng có một số hạn chế. Đôi khi nó có thể tạo ra thông tin không chính xác hoặc vô lý. Nếu bạn cung cấp thông tin không rõ ràng hoặc câu hỏi mơ hồ, ChatGPT có thể trả lời sai hoặc không phù hợp. Do đó, việc kiểm soát và kiểm tra kết quả trả về từ ChatGPT là rất quan trọng.

OpenAI đang không ngừng cải thiện ChatGPT dựa trên phản hồi và yêu cầu của cộng đồng. Điều này đảm bảo rằng bạn sẽ luôn nhận được trợ giúp và hỗ trợ để sử dụng ChatGPT tốt nhất trong dự án của mình.

Hãy bắt đầu khám phá ChatGPT ngay hôm nay và tận dụng sức mạnh của trí tuệ nhân tạo để xây dựng những ứng dụng tuyệt vời và sáng tạo!

- Đánh giá : Giọng văn rất tự nhiên và ổn.

## Bảo ChatGPT code
### Python
- Hỏi: 
```
Write a python program to read a CSV-format file.
Each field of CSV should be ID, Name, Birthday, all of them is String type.
```
- Trả lời:

Certainly! Here's a Python program that reads a CSV file with fields ID, Name, and Birthday, where each field is of type string:

```python
import csv

def read_csv_file(file_path):
    data = []
    with open(file_path, 'r') as csvfile:
        reader = csv.reader(csvfile)
        header = next(reader)  # Read the header row
        if header != ['ID', 'Name', 'Birthday']:
            print("Invalid CSV format. Please make sure the header matches 'ID', 'Name', 'Birthday'.")
            return data

        for row in reader:
            if len(row) != 3:
                print("Invalid row:", row)
                continue

            id_, name, birthday = row
            data.append({'ID': id_, 'Name': name, 'Birthday': birthday})

    return data

# Example usage
file_path = 'data.csv'  # Replace with your CSV file path
csv_data = read_csv_file(file_path)

# Print the data
for entry in csv_data:
    print("ID:", entry['ID'])
    print("Name:", entry['Name'])
    print("Birthday:", entry['Birthday'])
    print("---")

```

Make sure to replace 'data.csv' with the actual file path of your CSV file. The program reads each row of the CSV file, validates the header, and stores the data in a list of dictionaries, where each dictionary represents a row with the field names as keys.

Please note that this program assumes that the CSV file has the header row with the fields 'ID', 'Name', and 'Birthday' in that order. If the file structure differs, you can modify the program accordingly.

- Đánh giá:
    - Code chạy được
    - Có cả bắt lỗi, comment nữa.


- Cảm nhận
    - Thôi rồi lượm ơi...Cái máy tính nó biết code rồi....

# Một vài cảm nhận và dự đoán
## Cảm nhận
- Với sự ra đời của ChatGPT cũng như các mô hình ngôn ngữ lớn, có thể nói sẽ có một cuộc cách mạng về năng suất lao động sẽ xảy ra.
- Giống như chiếc máy vi tính khi mới ra đời.
- Không ai để ý đến việc nó có thể làm gì ngoài chức năng đánh văn bản.
- Và rồi, nó len lỏi vào mọi ngóc ngách của cuộc sống
- Chiếc điện thoại thông minh cũng vậy.
- Không ai nghĩ là chiếc điện thoại ngoài để nghe gọi ra thì thêm những thứ khác vào để làm cái gì.
- Và rồi, tất cả để dùng nó một cách tự nhiên, như một vật bất li thân.

## Dự đoán
- Với sự ra đời của ChatGPT, công việc của lập trình viên sẽ thay đổi ra sao.
- 1. Người ta coi biết sử dụng ChatGPT để code là một skill như các skill khác như biết sử dụng Git, AWS vậy.
- 2. ChatGPT sẽ khó có thể sinh code từ những lời nói miệng của khách hàng (chính khách hàng còn không biết), nhưng nó có thể hiểu hệ thống và sửa chính xác những gì người lập chính viên muốn.
- 3. Việc tạo ra code base của dự án đơn giản sẽ giảm chi phí đáng kể. Các công việc customize sau đó có thể sẽ không thay đổi chi phí. Vì thời gian (cost) để developer sửa các yêu cầu nhỏ sẽ vẫn nhỏ hơn chi phí để mô tả một cách chính xác cho ChatGPT sửa những chỗ này.
- 4. Có lẽ dự án sẽ đi theo hướng
    - Có một người biết kĩ thuật và có khả năng sử dụng ChatGPT sẽ xây dựng source base
    - Một nhóm khác sẽ làm cho source base đó chạy
    - Một nhóm khác sẽ sửa các yêu cầu nhỏ của khách hàng.