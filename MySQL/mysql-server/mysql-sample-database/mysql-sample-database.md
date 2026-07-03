---
layout: page
title: "MySQL Sample Database: classicmodels"
permalink: /MySQL/mysql-sample-database/
---

# MySQL Sample Database: `classicmodels`

**Nguồn tham khảo:** MySQL Tutorial - [MySQL Sample Database
](https://www.mysqltutorial.org/getting-started-with-mysql/mysql-sample-database/)

## 1. Mục tiêu tutorial

Trong tutorial này, bạn sẽ tìm hiểu về cơ sở dữ liệu mẫu **classicmodels** trong MySQL.

Sau khi hoàn thành, bạn có thể:

1. Hiểu `classicmodels` là gì và vì sao thường được dùng khi học MySQL.
2. Biết các nhóm dữ liệu chính trong database `classicmodels`.
3. Nhận diện các bảng quan trọng trong schema.
4. Hiểu vai trò của sơ đồ ERD khi học truy vấn MySQL.
5. Chuẩn bị database mẫu để thực hành các bài học MySQL tiếp theo.

---

## 2. classicmodels database là gì?

`classicmodels` là một cơ sở dữ liệu mẫu thường được dùng để học và thực hành MySQL.

Cơ sở dữ liệu này mô phỏng hoạt động của một doanh nghiệp bán lẻ các mô hình xe cổ thu nhỏ (*scale models of classic cars*).

Database này chứa các dữ liệu nghiệp vụ điển hình như:

- Khách hàng.
- Sản phẩm.
- Dòng sản phẩm.
- Đơn hàng bán.
- Chi tiết từng dòng hàng trong đơn hàng.
- Thanh toán của khách hàng.
- Nhân viên.
- Văn phòng bán hàng.
- Cấu trúc tổ chức nhân sự.

Vì có dữ liệu khá giống với một hệ thống kinh doanh thực tế, `classicmodels` rất phù hợp để thực hành nhiều chủ đề MySQL từ cơ bản đến nâng cao.

---

## 3. Vì sao nên dùng classicmodels để học MySQL?

Database `classicmodels` hữu ích vì:

1. **Có dữ liệu thực tế hơn các ví dụ quá nhỏ**

   Thay vì chỉ có một hoặc hai bảng đơn giản, `classicmodels` có nhiều bảng liên kết với nhau.

2. **Phù hợp để học truy vấn SQL**

   Có thể thực hành các câu lệnh như:

   - `SELECT`
   - `WHERE`
   - `ORDER BY`
   - `GROUP BY`
   - `JOIN`
   - Subquery
   - View
   - Stored Procedure

3. **Có quan hệ giữa các bảng**

   Người học có thể luyện cách truy vấn dữ liệu từ nhiều bảng thông qua khóa chính và khóa ngoại.

4. **Phù hợp với bài toán kinh doanh**

   Dữ liệu mô phỏng hoạt động bán hàng nên dễ đặt câu hỏi phân tích như:

   - Khách hàng nào mua nhiều nhất?
   - Sản phẩm nào bán chạy?
   - Nhân viên nào phụ trách nhiều khách hàng?
   - Doanh thu theo quốc gia là bao nhiêu?
   - Đơn hàng nào chưa được thanh toán?

---

## 4. Tải MySQL sample database

Bạn có thể tải database mẫu `classicmodels` từ trang cung cấp tài liệu MySQL tutorial.

File tải về thường có dạng nén `.zip`.

Ví dụ:

```text
sampledatabase.zip
```

Sau khi tải về, cần giải nén file này để lấy file SQL dùng để import vào MySQL Server.

---

## 5. Giải nén file database mẫu

Sau khi tải file `.zip`, bạn cần dùng một chương trình giải nén.

Một công cụ miễn phí phổ biến là:

```text
7-Zip
```

Trang tải:

```text
www.7-zip.org
```

Sau khi giải nén, bạn thường sẽ nhận được một file SQL, ví dụ:

```text
mysqlsampledatabase.sql
```

File này chứa các câu lệnh SQL để:

- Tạo database.
- Tạo các bảng.
- Thêm dữ liệu mẫu vào các bảng.

---

## 6. Nạp classicmodels vào MySQL Server

Sau khi có file SQL, bạn có thể nạp database mẫu vào MySQL Server.

Có hai cách phổ biến:

1. Dùng **mysql command-line client**.
2. Dùng **MySQL Workbench**.

---

## 7. Cách 1: Import bằng mysql command-line client

Giả sử file `mysqlsampledatabase.sql` nằm tại thư mục:

```text
C:\Users\YourName\Downloads\mysqlsampledatabase.sql
```

Bạn có thể mở Command Prompt và chạy:

```cmd
mysql -u root -p < "C:\Users\YourName\Downloads\mysqlsampledatabase.sql"
```

Ý nghĩa:

- `mysql`: gọi chương trình MySQL command-line client.
- `-u root`: đăng nhập bằng user `root`.
- `-p`: yêu cầu nhập mật khẩu.
- `<`: chuyển nội dung file SQL vào MySQL để thực thi.
- `"C:\Users\YourName\Downloads\mysqlsampledatabase.sql"`: đường dẫn tới file SQL.

Sau khi chạy lệnh, nhập mật khẩu MySQL nếu được yêu cầu.

---

## 8. Cách 2: Import bằng MySQL Workbench

Có thể import database bằng MySQL Workbench theo các bước sau:

### Bước 1: Mở MySQL Workbench

Mở MySQL Workbench và kết nối tới MySQL Server.

### Bước 2: Mở chức năng import

Vào menu:

```text
Server > Data Import
```

### Bước 3: Chọn file SQL

Chọn tùy chọn import từ file SQL, sau đó chọn file:

```text
mysqlsampledatabase.sql
```

### Bước 4: Chạy import

Nhấn nút **Start Import** để bắt đầu nạp dữ liệu.

### Bước 5: Kiểm tra database

Sau khi import xong, trong danh sách schema sẽ có database:

```text
classicmodels
```

---

## 9. Kiểm tra database sau khi import

Sau khi import, bạn có thể kiểm tra bằng lệnh:

```sql
SHOW DATABASES;
```

Nếu import thành công, bạn sẽ thấy database:

```text
classicmodels
```

Sau đó chọn database để sử dụng:

```sql
USE classicmodels;
```

Kiểm tra danh sách bảng:

```sql
SHOW TABLES;
```

Kết quả sẽ gồm các bảng như:

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

---

## 10. Schema của classicmodels database

Schema của `classicmodels` gồm các bảng chính sau:

| Bảng | Ý nghĩa |
|---|---|
| `customers` | Lưu dữ liệu khách hàng |
| `products` | Lưu danh sách các mô hình xe cổ |
| `productlines` | Lưu các dòng sản phẩm |
| `orders` | Lưu các đơn hàng bán |
| `orderdetails` | Lưu chi tiết từng dòng sản phẩm trong mỗi đơn hàng |
| `payments` | Lưu các khoản thanh toán của khách hàng |
| `employees` | Lưu thông tin nhân viên và cấu trúc báo cáo trong tổ chức |
| `offices` | Lưu dữ liệu văn phòng bán hàng |

---

## 11. Ý nghĩa các bảng trong classicmodels

### 11.1. Bảng `customers`

Bảng `customers` lưu thông tin khách hàng.

Ví dụ dữ liệu có thể gồm:

- Mã khách hàng.
- Tên khách hàng.
- Người liên hệ.
- Số điện thoại.
- Địa chỉ.
- Thành phố.
- Quốc gia.
- Nhân viên bán hàng phụ trách.
- Hạn mức tín dụng.

Bảng này thường được dùng để thực hành các truy vấn về khách hàng và doanh thu.

---

### 11.2. Bảng `products`

Bảng `products` lưu danh sách sản phẩm.

Trong ngữ cảnh `classicmodels`, sản phẩm là các mô hình xe cổ thu nhỏ.

Thông tin sản phẩm có thể gồm:

- Mã sản phẩm.
- Tên sản phẩm.
- Dòng sản phẩm.
- Tỷ lệ mô hình.
- Nhà cung cấp.
- Mô tả.
- Số lượng trong kho.
- Giá mua.
- Giá bán đề xuất.

---

### 11.3. Bảng `productlines`

Bảng `productlines` lưu các dòng sản phẩm.

Ví dụ:

- Classic Cars.
- Motorcycles.
- Planes.
- Ships.
- Trains.
- Trucks and Buses.
- Vintage Cars.

Bảng này giúp phân loại sản phẩm theo nhóm.

---

### 11.4. Bảng `orders`

Bảng `orders` lưu thông tin đơn hàng.

Thông tin thường gồm:

- Mã đơn hàng.
- Ngày đặt hàng.
- Ngày yêu cầu giao hàng.
- Ngày giao hàng thực tế.
- Trạng thái đơn hàng.
- Mã khách hàng.

---

### 11.5. Bảng `orderdetails`

Bảng `orderdetails` lưu chi tiết từng dòng hàng trong đơn hàng.

Một đơn hàng có thể gồm nhiều sản phẩm, vì vậy cần bảng `orderdetails` để lưu:

- Mã đơn hàng.
- Mã sản phẩm.
- Số lượng đặt.
- Giá bán từng sản phẩm.
- Thứ tự dòng hàng.

Bảng này rất quan trọng khi tính doanh thu.

---

### 11.6. Bảng `payments`

Bảng `payments` lưu các khoản thanh toán của khách hàng.

Thông tin có thể gồm:

- Mã khách hàng.
- Mã giao dịch thanh toán.
- Ngày thanh toán.
- Số tiền thanh toán.

Bảng này thường được dùng để phân tích tình hình thanh toán và công nợ.

---

### 11.7. Bảng `employees`

Bảng `employees` lưu thông tin nhân viên.

Thông tin có thể gồm:

- Mã nhân viên.
- Họ tên.
- Email.
- Chức danh.
- Văn phòng làm việc.
- Người quản lý trực tiếp.

Bảng này cũng thể hiện cấu trúc tổ chức thông qua quan hệ nhân viên báo cáo cho nhân viên khác.

---

### 11.8. Bảng `offices`

Bảng `offices` lưu dữ liệu các văn phòng bán hàng.

Thông tin có thể gồm:

- Mã văn phòng.
- Thành phố.
- Số điện thoại.
- Địa chỉ.
- Quốc gia.
- Khu vực.

---

## 12. ER Diagram của classicmodels

ER Diagram giúp người học nhìn thấy quan hệ giữa các bảng trong database `classicmodels`.

<p align="center">
  <img src="images/mysql-sample-database.png" alt="MySQL Sample Database ER Diagram" width="750">
</p>

<p align="center">
  <em>Hình 1. ER Diagram của database classicmodels.</em>
</p>

Nên in hoặc mở sơ đồ ERD khi học MySQL để dễ theo dõi các quan hệ giữa bảng.

Một số quan hệ quan trọng:

- `customers` liên kết với `orders`.
- `orders` liên kết với `orderdetails`.
- `orderdetails` liên kết với `products`.
- `products` liên kết với `productlines`.
- `customers` liên kết với `payments`.
- `customers` liên kết với `employees` thông qua nhân viên phụ trách.
- `employees` liên kết với `offices`.
- `employees` có quan hệ tự tham chiếu để biểu diễn nhân viên báo cáo cho quản lý.

---

## 13. Một số truy vấn kiểm tra nhanh

Sau khi import database, có thể chạy một số truy vấn sau.

### 13.1. Chọn database

```sql
USE classicmodels;
```

### 13.2. Xem danh sách bảng

```sql
SHOW TABLES;
```

### 13.3. Xem một vài khách hàng

```sql
SELECT customerNumber, customerName, country
FROM customers
LIMIT 10;
```

### 13.4. Xem danh sách sản phẩm

```sql
SELECT productCode, productName, productLine, buyPrice, MSRP
FROM products
LIMIT 10;
```

### 13.5. Xem các đơn hàng gần đây

```sql
SELECT orderNumber, orderDate, status, customerNumber
FROM orders
ORDER BY orderDate DESC
LIMIT 10;
```

### 13.6. Tính doanh thu từng dòng đơn hàng

```sql
SELECT orderNumber,
       productCode,
       quantityOrdered,
       priceEach,
       quantityOrdered * priceEach AS lineRevenue
FROM orderdetails
LIMIT 10;
```

---

## 14. Ví dụ truy vấn JOIN đơn giản

### 14.1. Lấy đơn hàng kèm tên khách hàng

```sql
SELECT o.orderNumber,
       o.orderDate,
       o.status,
       c.customerName
FROM orders o
JOIN customers c
  ON o.customerNumber = c.customerNumber
LIMIT 10;
```

Truy vấn này kết hợp bảng `orders` và `customers` thông qua khóa `customerNumber`.

---

### 14.2. Lấy sản phẩm kèm dòng sản phẩm

```sql
SELECT p.productCode,
       p.productName,
       p.productLine,
       pl.textDescription
FROM products p
JOIN productlines pl
  ON p.productLine = pl.productLine
LIMIT 10;
```

Truy vấn này giúp xem sản phẩm thuộc dòng sản phẩm nào.

---

### 14.3. Tính doanh thu theo đơn hàng

```sql
SELECT od.orderNumber,
       SUM(od.quantityOrdered * od.priceEach) AS totalRevenue
FROM orderdetails od
GROUP BY od.orderNumber
ORDER BY totalRevenue DESC
LIMIT 10;
```

Truy vấn này tính tổng doanh thu của từng đơn hàng.

---

## 15. Các chủ đề MySQL có thể học với classicmodels

Database `classicmodels` có thể dùng để học nhiều chủ đề MySQL:

| Chủ đề | Ví dụ thực hành |
|---|---|
| Basic SELECT | Lấy danh sách khách hàng, sản phẩm |
| WHERE | Lọc khách hàng theo quốc gia |
| ORDER BY | Sắp xếp đơn hàng theo ngày |
| GROUP BY | Tính doanh thu theo khách hàng |
| JOIN | Kết hợp khách hàng với đơn hàng |
| Subquery | Tìm khách hàng có doanh thu cao hơn trung bình |
| View | Tạo view tổng hợp doanh thu |
| Stored Procedure | Viết thủ tục thống kê đơn hàng |
| Function | Tạo hàm tính doanh thu |
| Trigger | Theo dõi thay đổi dữ liệu |
| Index | Tối ưu truy vấn theo khóa |

---

## 16. Bài tập thực hành

### Bài tập 1

Import database `classicmodels` vào MySQL Server.

**Yêu cầu:**

1. Kiểm tra database đã xuất hiện bằng lệnh:

```sql
SHOW DATABASES;
```

2. Chọn database:

```sql
USE classicmodels;
```

3. Xem danh sách bảng:

```sql
SHOW TABLES;
```

---

### Bài tập 2

Truy vấn danh sách 10 khách hàng đầu tiên.

**Yêu cầu:**

Hiển thị:

- `customerNumber`
- `customerName`
- `city`
- `country`

Gợi ý:

```sql
SELECT customerNumber, customerName, city, country
FROM customers
LIMIT 10;
```

---

### Bài tập 3

Truy vấn danh sách sản phẩm thuộc dòng sản phẩm `Classic Cars`.

**Yêu cầu:**

Hiển thị:

- `productCode`
- `productName`
- `productLine`
- `MSRP`

---

### Bài tập 4

Tính tổng doanh thu của từng đơn hàng.

**Yêu cầu:**

Sử dụng bảng `orderdetails`, tính:

```text
quantityOrdered * priceEach
```

và nhóm theo `orderNumber`.

---

### Bài tập 5

Viết truy vấn JOIN để hiển thị đơn hàng kèm tên khách hàng.

**Yêu cầu:**

Kết hợp hai bảng:

- `orders`
- `customers`

thông qua trường:

```text
customerNumber
```

---

## 17. Câu hỏi ôn tập

### 17.1. Câu hỏi tự luận

**Câu 1.** `classicmodels` database mô phỏng nghiệp vụ gì?

---

**Câu 2.** Vì sao `classicmodels` phù hợp để học MySQL?

---

**Câu 3.** Bảng `orderdetails` có vai trò gì?

---

**Câu 4.** Vì sao cần bảng `productlines` nếu đã có bảng `products`?

---

**Câu 5.** ER Diagram giúp ích gì khi học database `classicmodels`?

---

### 17.2. Câu hỏi trắc nghiệm

**Câu 1.** `classicmodels` là database mẫu mô phỏng lĩnh vực nào?

A. Bán lẻ mô hình xe cổ thu nhỏ  
B. Quản lý bệnh viện  
C. Quản lý thư viện số  
D. Quản lý mạng xã hội  

**Câu 2.** Bảng nào lưu dữ liệu khách hàng?

A. `products`  
B. `customers`  
C. `orders`  
D. `offices`  

**Câu 3.** Bảng nào lưu chi tiết từng dòng sản phẩm trong đơn hàng?

A. `payments`  
B. `employees`  
C. `orderdetails`  
D. `productlines`  

**Câu 4.** Bảng nào lưu dữ liệu thanh toán của khách hàng?

A. `products`  
B. `orders`  
C. `offices`  
D. `payments`  

**Câu 5.** Bảng nào lưu thông tin nhân viên và cấu trúc báo cáo trong tổ chức?

A. `employees`  
B. `customers`  
C. `orders`  
D. `productlines`  

---

## 18. Tóm tắt

- `classicmodels` là database mẫu trong MySQL.
- Database này mô phỏng một doanh nghiệp bán mô hình xe cổ thu nhỏ.
- Schema gồm các bảng chính như `customers`, `products`, `productlines`, `orders`, `orderdetails`, `payments`, `employees` và `offices`.
- Database này phù hợp để học các câu lệnh SQL từ cơ bản đến nâng cao.
- ER Diagram giúp người học hiểu quan hệ giữa các bảng.
- Sau khi import, có thể dùng `SHOW DATABASES`, `USE classicmodels` và `SHOW TABLES` để kiểm tra.
- `classicmodels` đặc biệt hữu ích để thực hành `JOIN`, `GROUP BY`, truy vấn tổng hợp, view và stored procedure.

---

## 19. Từ khóa chính

- MySQL
- Sample Database
- classicmodels
- Schema
- ER Diagram
- customers
- products
- productlines
- orders
- orderdetails
- payments
- employees
- offices
- SQL
- SELECT
- JOIN
- GROUP BY
- Stored Procedure
- View
