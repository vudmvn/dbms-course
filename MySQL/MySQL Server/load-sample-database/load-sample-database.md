---
layout: page
title: "Nạp database mẫu classicmodels vào MySQL Server"
---

# Nạp database mẫu `classicmodels` vào MySQL Server

**Nguồn tham khảo:** MySQL Tutorial - [How to Load the Sample Database into MySQL Server](https://www.mysqltutorial.org/getting-started-with-mysql/how-to-load-sample-database-into-mysql-database-server/)
## 1. Mục tiêu tutorial

Trong tutorial này, bạn sẽ học cách nạp database mẫu `classicmodels` vào **MySQL Server** bằng chương trình dòng lệnh `mysql`.

Sau khi hoàn thành, bạn có thể:

1. Tải database mẫu `classicmodels`.
2. Giải nén file database mẫu.
3. Kết nối tới MySQL Server bằng `mysql` command-line client.
4. Dùng lệnh `source` để nạp file `.sql` vào MySQL Server.
5. Kiểm tra database `classicmodels` đã được tạo thành công hay chưa.
6. Xử lý một số lỗi thường gặp khi import database.

---

## 2. Chuẩn bị

Trước khi bắt đầu, cần đảm bảo:

- Máy tính đã cài **MySQL Server**.
- MySQL Server đang chạy.
- Máy tính có chương trình `mysql` command-line client.
- Bạn biết tài khoản và mật khẩu đăng nhập MySQL, ví dụ:
  - Username: `root`
  - Password: mật khẩu đã đặt khi cài MySQL
- Bạn đã tải file database mẫu `classicmodels`.

---

## 3. Tổng quan các bước thực hiện

Quy trình nạp database mẫu gồm 5 bước chính:

1. Tải database mẫu `classicmodels`.
2. Giải nén file tải về.
3. Kết nối tới MySQL Server bằng chương trình `mysql`.
4. Dùng lệnh `source` để chạy file SQL.
5. Kiểm tra database bằng lệnh `SHOW DATABASES`.

---

## 4. Bước 1: Tải classicmodels database

Trước tiên, tải database mẫu `classicmodels` từ phần **MySQL sample database**.

File tải về thường ở định dạng `.zip`, ví dụ:

```text
sampledatabase.zip
```

File `.zip` này chứa file SQL dùng để tạo database, tạo bảng và thêm dữ liệu mẫu.

---

## 5. Bước 2: Giải nén file tải về

Sau khi tải file `.zip`, hãy giải nén vào một thư mục tạm.

Trên Windows, có thể giải nén vào thư mục:

```text
C:\temp
```

Sau khi giải nén, bạn có thể có file:

```text
C:\temp\mysqlsampledatabase.sql
```

Nếu dùng macOS, Linux hoặc Unix, bạn có thể giải nén vào bất kỳ thư mục nào thuận tiện, ví dụ:

```text
/tmp/mysqlsampledatabase.sql
```

hoặc:

```text
/home/user/Downloads/mysqlsampledatabase.sql
```

---

## 6. Bước 3: Kết nối tới MySQL Server bằng mysql client

Mở **Command Prompt** trên Windows hoặc **Terminal** trên macOS/Linux.

Sau đó chạy lệnh:

```cmd
mysql -u root -p
```

Ý nghĩa của lệnh:

| Thành phần | Ý nghĩa |
|---|---|
| `mysql` | Chạy chương trình MySQL command-line client |
| `-u root` | Kết nối bằng user `root` |
| `-p` | Yêu cầu nhập mật khẩu sau khi chạy lệnh |

Sau khi chạy lệnh, MySQL sẽ yêu cầu nhập mật khẩu:

```text
Enter password: ********
```

Mật khẩu này là mật khẩu của user `root` mà bạn đã đặt khi cài MySQL.

Lưu ý: Khi nhập mật khẩu, màn hình có thể không hiển thị ký tự nào. Đây là hành vi bình thường để bảo mật.

Nếu đăng nhập thành công, bạn sẽ thấy dấu nhắc:

```sql
mysql>
```

Điều này cho biết bạn đã kết nối thành công tới MySQL Server.

---

## 7. Bước 4: Nạp database bằng lệnh source

Sau khi đã vào môi trường MySQL với dấu nhắc:

```sql
mysql>
```

hãy dùng lệnh `source` để chạy file SQL.

Nếu file SQL nằm tại:

```text
C:\temp\mysqlsampledatabase.sql
```

thì chạy lệnh:

```sql
source c:/temp/mysqlsampledatabase.sql
```

Lưu ý:

- Trong MySQL command-line client, nên dùng dấu `/` thay vì `\` trong đường dẫn.
- Đường dẫn Windows:

```text
C:\temp\mysqlsampledatabase.sql
```

có thể viết trong MySQL là:

```text
c:/temp/mysqlsampledatabase.sql
```

Sau khi chạy lệnh, MySQL sẽ thực thi các câu lệnh trong file SQL để:

- Tạo database `classicmodels`.
- Tạo các bảng.
- Thêm dữ liệu mẫu vào các bảng.

---

## 8. Bước 5: Kiểm tra database đã được nạp chưa

Sau khi chạy lệnh `source`, kiểm tra danh sách database bằng lệnh:

```sql
SHOW DATABASES;
```

Nếu nạp thành công, kết quả sẽ có database `classicmodels`:

```text
+--------------------+
| Database           |
+--------------------+
| classicmodels      |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```

Database `classicmodels` xuất hiện trong danh sách nghĩa là quá trình import đã thành công.

---

## 9. Kiểm tra chi tiết bên trong classicmodels

Sau khi thấy database `classicmodels`, chọn database này:

```sql
USE classicmodels;
```

Kiểm tra danh sách bảng:

```sql
SHOW TABLES;
```

Kết quả thường gồm các bảng:

```text
customers
employees
offices
orderdetails
orders
payments
productlines
products
```

Có thể kiểm tra nhanh dữ liệu khách hàng:

```sql
SELECT customerNumber, customerName, country
FROM customers
LIMIT 10;
```

Hoặc kiểm tra dữ liệu sản phẩm:

```sql
SELECT productCode, productName, productLine
FROM products
LIMIT 10;
```

---

## 10. Giải thích lệnh source

Lệnh `source` trong MySQL command-line client dùng để thực thi nội dung của một file SQL.

Ví dụ:

```sql
source c:/temp/mysqlsampledatabase.sql
```

có nghĩa là:

> Đọc toàn bộ file `mysqlsampledatabase.sql` và chạy các câu lệnh SQL trong file đó trên MySQL Server.

File SQL thường chứa các câu lệnh như:

```sql
CREATE DATABASE classicmodels;
USE classicmodels;
CREATE TABLE customers (...);
INSERT INTO customers VALUES (...);
```

Nhờ vậy, thay vì phải gõ từng câu lệnh bằng tay, ta chỉ cần chạy một lệnh `source`.

---

## 11. Luồng xử lý khi nạp database

Khi chạy lệnh:

```sql
source c:/temp/mysqlsampledatabase.sql
```

quy trình diễn ra như sau:

1. `mysql` client đọc file `mysqlsampledatabase.sql`.
2. Client gửi các câu lệnh SQL trong file tới MySQL Server.
3. MySQL Server thực thi các câu lệnh:
   - Tạo database.
   - Tạo bảng.
   - Thêm dữ liệu.
4. Server trả kết quả về cho client.
5. Người dùng kiểm tra lại bằng `SHOW DATABASES` và `SHOW TABLES`.

Có thể hiểu đơn giản:

```text
File SQL → mysql client → MySQL Server → classicmodels database
```

---

## 12. Một số lỗi thường gặp

### 12.1. Lỗi không tìm thấy file SQL

Nếu chạy:

```sql
source c:/temp/mysqlsampledatabase.sql
```

nhưng MySQL báo lỗi không tìm thấy file, có thể do:

- File không nằm đúng thư mục.
- Tên file bị sai.
- Đường dẫn bị sai.
- File chưa được giải nén.

Cách xử lý:

- Kiểm tra lại file có tồn tại không.
- Kiểm tra đúng tên file chưa.
- Đưa file vào thư mục đơn giản như:

```text
C:\temp
```

- Dùng đường dẫn dạng:

```sql
source c:/temp/mysqlsampledatabase.sql
```

---

### 12.2. Lỗi chưa đăng nhập vào MySQL

Lệnh `source` cần được chạy bên trong môi trường MySQL.

Bạn phải thấy dấu nhắc:

```sql
mysql>
```

rồi mới chạy:

```sql
source c:/temp/mysqlsampledatabase.sql
```

Nếu bạn đang ở Command Prompt thông thường, cần đăng nhập trước:

```cmd
mysql -u root -p
```

---

### 12.3. Lỗi sai mật khẩu root

Nếu nhập sai mật khẩu, có thể gặp lỗi:

```text
Access denied for user 'root'@'localhost'
```

Cách xử lý:

- Kiểm tra lại mật khẩu.
- Kiểm tra đúng username chưa.
- Đảm bảo MySQL Server đang chạy.
- Nếu quên mật khẩu, cần thực hiện quy trình reset password.

---

### 12.4. Lỗi MySQL Server chưa chạy

Nếu MySQL Server chưa chạy, bạn sẽ không kết nối được bằng:

```cmd
mysql -u root -p
```

Trên Windows, có thể kiểm tra service MySQL bằng:

```cmd
sc.exe query type= service | findstr /I "mysql"
```

Nếu thấy service như `MySQL80` hoặc `MySQL91`, kiểm tra trạng thái:

```cmd
sc.exe query MySQL91
```

Nếu đang chạy, kết quả sẽ có:

```text
STATE              : 4  RUNNING
```

---

### 12.5. Lỗi do đường dẫn có khoảng trắng

Nếu file nằm trong thư mục có khoảng trắng, việc gọi `source` có thể gây khó khăn.

Ví dụ:

```text
C:\Users\Minh Vu\Downloads\mysqlsampledatabase.sql
```

Cách đơn giản nhất là chuyển file vào thư mục:

```text
C:\temp
```

Sau đó chạy:

```sql
source c:/temp/mysqlsampledatabase.sql
```

---

## 13. Bài tập thực hành

### Bài tập 1

Tải và giải nén database mẫu `classicmodels`.

**Yêu cầu:**

- Giải nén file vào thư mục:

```text
C:\temp
```

- Kiểm tra có file:

```text
mysqlsampledatabase.sql
```

---

### Bài tập 2

Kết nối tới MySQL Server bằng lệnh:

```cmd
mysql -u root -p
```

**Yêu cầu:**

- Nhập đúng mật khẩu.
- Kiểm tra đã xuất hiện dấu nhắc:

```sql
mysql>
```

---

### Bài tập 3

Nạp database mẫu bằng lệnh:

```sql
source c:/temp/mysqlsampledatabase.sql
```

**Yêu cầu:**

- Quan sát quá trình chạy lệnh.
- Ghi lại nếu có lỗi phát sinh.

---

### Bài tập 4

Kiểm tra database đã được nạp thành công:

```sql
SHOW DATABASES;
```

**Yêu cầu:**

- Xác nhận có database `classicmodels`.

---

### Bài tập 5

Chọn database và kiểm tra bảng:

```sql
USE classicmodels;
SHOW TABLES;
```

**Yêu cầu:**

- Ghi lại danh sách bảng trong database `classicmodels`.

---

## 14. Câu hỏi ôn tập

### 14.1. Câu hỏi tự luận

**Câu 1.** Lệnh `mysql -u root -p` có ý nghĩa gì?

---

**Câu 2.** Lệnh `source c:/temp/mysqlsampledatabase.sql` dùng để làm gì?

---

**Câu 3.** Vì sao nên giải nén file SQL vào thư mục đơn giản như `C:\temp`?

---

**Câu 4.** Làm thế nào để kiểm tra database `classicmodels` đã được tạo thành công?

---

**Câu 5.** Sau khi chọn `USE classicmodels;`, lệnh nào dùng để xem danh sách bảng?

---

### 14.2. Câu hỏi trắc nghiệm

**Câu 1.** Lệnh nào dùng để kết nối tới MySQL Server bằng user `root`?

A. `mysql -u root -p`  
B. `connect root mysql`  
C. `start classicmodels`  
D. `show mysql root`  

**Câu 2.** Lệnh `-p` trong `mysql -u root -p` có ý nghĩa gì?

A. Chọn port mặc định  
B. Yêu cầu nhập mật khẩu  
C. In danh sách database  
D. Tạo database mới  

**Câu 3.** Lệnh nào dùng để nạp file SQL trong MySQL command-line client?

A. `load table`  
B. `open file`  
C. `source`  
D. `run database`  

**Câu 4.** Lệnh nào dùng để xem danh sách database?

A. `SHOW TABLES;`  
B. `USE classicmodels;`  
C. `SELECT DATABASE();`  
D. `SHOW DATABASES;`  

**Câu 5.** Nếu import thành công, database nào sẽ xuất hiện?

A. `classicmodels`  
B. `samplecars`  
C. `mysqlworkbench`  
D. `ordersdb`  

---

## 15. Tóm tắt

Trong tutorial này, chúng ta đã học cách nạp database mẫu `classicmodels` vào MySQL Server bằng chương trình `mysql`.

Các bước chính gồm:

1. Tải database mẫu.
2. Giải nén file `.zip`.
3. Kết nối tới MySQL Server:

```cmd
mysql -u root -p
```

4. Chạy lệnh `source` để nạp file SQL:

```sql
source c:/temp/mysqlsampledatabase.sql
```

5. Kiểm tra database:

```sql
SHOW DATABASES;
```

6. Chọn database và xem bảng:

```sql
USE classicmodels;
SHOW TABLES;
```

Sau khi hoàn thành, bạn có thể dùng database `classicmodels` để thực hành các bài học MySQL tiếp theo.

---

## 16. Từ khóa chính

- MySQL Server
- mysql command-line client
- classicmodels
- sample database
- SQL file
- source command
- root user
- password
- SHOW DATABASES
- USE database
- SHOW TABLES
- Command Prompt
- Terminal
- import database
