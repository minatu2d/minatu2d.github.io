---
title: "Vấn đề tốn bộ nhờ (Memory) và CPU khi export file bằng Apache POI"
description: "Vấn đề thường gán với bộ nhớ và CPU khi export file bằng Apache POI"
date: "2023-11-01T7:30:00+09:00"
thumbnail: "/post/images/20231101_export_file_problem/out_of_memory.png"
toc: true
published: true
categories:
  - basic
tags:
  - java
  - excel
  - apache poi
---

# Môi trường, điều kiện
- Ghi 1 triệu row ra file excel
- Môi trường:
  - AMD Ryzen 5 3600 6-Core Processor
  - Mem: 15Gi
## Khi sủ dụng với xssfworkbook
- Thời gian chạy : 24.7 s
```bash
➜  excel-performance-test mvn exec:java -Dexec.mainClass="com.example.ExcelPerformanceTestXSSFWorkbook"
[INFO] Scanning for projects...
[INFO]
[INFO] -----------------< com.example:excel-performance-test >-----------------
[INFO] Building excel-performance-test 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- exec-maven-plugin:3.1.0:java (default-cli) @ excel-performance-test ---
Execution time: 24486 ms
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  24.776 s
[INFO] Finished at: 2023-11-09T06:56:38+09:00
```
- CPU, Memory Usage :
  ![Alt text](/post/images/2023/20231101_export_file_problem/TestXSSFWorkbook.png)

## Khi sử dụng với sxssfworkbook
- Thời gian chạy : 5.1 s
```bash
➜  excel-performance-test mvn exec:java -Dexec.mainClass="com.example.ExcelPerformanceTestSXSSFWorkbook"
[INFO] Scanning for projects...
[INFO]
[INFO] -----------------< com.example:excel-performance-test >-----------------
[INFO] Building excel-performance-test 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- exec-maven-plugin:3.1.0:java (default-cli) @ excel-performance-test ---
Execution time: 4779 ms
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  5.106 s
[INFO] Finished at: 2023-11-09T06:58:12+09:00
[INFO] ------------------------------------------------------------------------
```
- CPU, Memory Usage
![Alt text](/post/images/2023/20231101_export_file_problem/TestSXSSFWorkbook.png)
# Đánh giá:
- Sử dụng `sxssfworkbook` cho thời gian chạy nhanh hơn : 24.7 s vs 5.1 s
- Sử dụng `sxssfworkbook` cho mức độ chiếm dụng CPU ít hơn
- Sử dụng `sxssfworkbook` thì ghi file thường xuyên hơn
  - Có thể điều chỉnh được bằng `Buffer` size (ở ví dụ trên đang để 1000 row)

# Một số chú ý
- Khi sử dụng `sxssfworkbook` thì một số thao tác `GetRow` sẽ không thể hoạt động được nữa.
- Tức là một phần của file `Excel` đã bị ghi ra rồi thì không thể tham chiếu lại nữa.
