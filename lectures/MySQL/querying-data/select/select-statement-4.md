---
title: "Lab: GROUP BY, HAVING và các truy vấn tổng hợp với classicmodels"
author: "Tên giảng viên"
duration: "120m"
difficulty: "Beginner - Intermediate"
prerequisites:
    - "Đã biết SELECT, FROM, WHERE, ORDER BY, LIMIT"
    - "Đã biết các hàm tổng hợp cơ bản: COUNT, SUM, AVG, MIN, MAX"
    - "Đã biết JOIN cơ bản là một lợi thế"
summary: "Thực hành nhóm dữ liệu, tính toán thống kê và lọc nhóm bằng GROUP BY và HAVING trên cơ sở dữ liệu mẫu classicmodels."
---

# Lab: `GROUP BY`, `HAVING` và các truy vấn tổng hợp với `classicmodels`

## Link tham khảo

- [MySQL GROUP BY](https://www.mysqltutorial.org/mysql-basics/mysql-group-by/)
- [MySQL HAVING](https://www.mysqltutorial.org/mysql-basics/mysql-having/)
- [MySQL HAVING COUNT](https://www.mysqltutorial.org/mysql-basics/mysql-having-count/)

## 1. Mục tiêu học tập

Sau khi hoàn thành lab này, người học có thể:

1. Giải thích được vai trò của `GROUP BY` trong SQL.
2. Sử dụng các hàm tổng hợp `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`.
3. Nhóm dữ liệu theo một cột hoặc nhiều cột.
4. Phân biệt được `WHERE` và `HAVING`.
5. Viết truy vấn lọc dữ liệu trước khi nhóm bằng `WHERE`.
6. Viết truy vấn lọc kết quả sau khi nhóm bằng `HAVING`.
7. Sắp xếp kết quả tổng hợp bằng `ORDER BY`.
8. Giới hạn kết quả tổng hợp bằng `LIMIT`.
9. Kết hợp `GROUP BY` với `JOIN` để thống kê dữ liệu từ nhiều bảng.
10. Tránh được các lỗi phổ biến khi dùng `GROUP BY` và `HAVING`.

---

## 2. Giới thiệu cơ sở dữ liệu mẫu `classicmodels`

`classicmodels` là cơ sở dữ liệu mẫu mô phỏng hoạt động của một công ty bán các mô hình xe, tàu, máy bay và sản phẩm sưu tầm.

Một số bảng thường dùng trong lab này:

| Bảng | Ý nghĩa | Một số cột quan trọng |
|---|---|---|
| `products` | Sản phẩm | `productCode`, `productName`, `productLine`, `quantityInStock`, `buyPrice`, `MSRP` |
| `productlines` | Dòng sản phẩm | `productLine`, `textDescription` |
| `customers` | Khách hàng | `customerNumber`, `customerName`, `country`, `city`, `salesRepEmployeeNumber`, `creditLimit` |
| `orders` | Đơn hàng | `orderNumber`, `orderDate`, `status`, `customerNumber` |
| `orderdetails` | Chi tiết đơn hàng | `orderNumber`, `productCode`, `quantityOrdered`, `priceEach` |
| `payments` | Thanh toán | `customerNumber`, `checkNumber`, `paymentDate`, `amount` |
| `employees` | Nhân viên | `employeeNumber`, `lastName`, `firstName`, `jobTitle`, `officeCode`, `reportsTo` |
| `offices` | Văn phòng | `officeCode`, `city`, `country`, `territory` |

Trước khi thực hành, chọn cơ sở dữ liệu:

```sql
USE classicmodels;
```

Xem danh sách bảng:

```sql
SHOW TABLES;
```

Xem cấu trúc một bảng:

```sql
DESC products;
```

### Bài tập thực hành

**Bài 2.1.** Viết lệnh chọn cơ sở dữ liệu `classicmodels`.

**Bài 2.2.** Viết lệnh xem cấu trúc bảng `products`.

**Bài 2.3.** Viết lệnh xem cấu trúc bảng `orders`.

**Bài 2.4.** Viết lệnh xem cấu trúc bảng `orderdetails`.

**Bài 2.5.** Xác định bảng nào chứa thông tin thanh toán của khách hàng.

---

## 3. Ý tưởng của truy vấn tổng hợp

Trong nhiều bài toán, ta không chỉ muốn xem từng dòng dữ liệu riêng lẻ, mà muốn trả lời các câu hỏi dạng thống kê:

- Có bao nhiêu khách hàng ở mỗi quốc gia?
- Mỗi dòng sản phẩm có bao nhiêu sản phẩm?
- Tổng số tiền mỗi khách hàng đã thanh toán là bao nhiêu?
- Mỗi đơn hàng có tổng giá trị là bao nhiêu?
- Nhân viên bán hàng nào phụ trách nhiều khách hàng?

Các câu hỏi này thường cần:

1. **Chọn nhóm dữ liệu** bằng `GROUP BY`.
2. **Tính toán trên mỗi nhóm** bằng hàm tổng hợp.
3. **Lọc nhóm** bằng `HAVING` nếu cần.

---

## 4. Các hàm tổng hợp thường dùng

| Hàm | Ý nghĩa | Ví dụ |
|---|---|---|
| `COUNT(*)` | Đếm số dòng | Đếm số khách hàng |
| `COUNT(column)` | Đếm số giá trị khác `NULL` trong một cột | Đếm số khách hàng có nhân viên phụ trách |
| `SUM(column)` | Tính tổng | Tổng tiền thanh toán |
| `AVG(column)` | Tính trung bình | Giá bán đề xuất trung bình |
| `MIN(column)` | Lấy giá trị nhỏ nhất | Giá thấp nhất |
| `MAX(column)` | Lấy giá trị lớn nhất | Giá cao nhất |

Ví dụ đếm tổng số khách hàng:

```sql
SELECT COUNT(*) AS totalCustomers
FROM customers;
```

Ví dụ tính tổng số tiền thanh toán:

```sql
SELECT SUM(amount) AS totalPayment
FROM payments;
```

Ví dụ tính giá bán đề xuất trung bình:

```sql
SELECT AVG(MSRP) AS avgMSRP
FROM products;
```

### Bài tập thực hành

**Bài 4.1.** Đếm tổng số sản phẩm trong bảng `products`.

**Bài 4.2.** Đếm tổng số khách hàng trong bảng `customers`.

**Bài 4.3.** Tính tổng số tiền trong bảng `payments`.

**Bài 4.4.** Tính giá mua trung bình của sản phẩm trong bảng `products`.

**Bài 4.5.** Tìm `MSRP` nhỏ nhất và lớn nhất trong bảng `products`.

---

## 5. Cú pháp cơ bản của `GROUP BY`

`GROUP BY` dùng để gom các dòng có cùng giá trị ở một hoặc nhiều cột thành từng nhóm.

Cú pháp tổng quát:

```sql
SELECT group_column, aggregate_function(column)
FROM table_name
GROUP BY group_column;
```

Ví dụ đếm số khách hàng theo quốc gia:

```sql
SELECT country, COUNT(*) AS numberOfCustomers
FROM customers
GROUP BY country;
```

Ví dụ đếm số sản phẩm theo dòng sản phẩm:

```sql
SELECT productLine, COUNT(*) AS numberOfProducts
FROM products
GROUP BY productLine;
```

### Bài tập thực hành

**Bài 5.1.** Đếm số khách hàng theo `country`.

**Bài 5.2.** Đếm số sản phẩm theo `productLine`.

**Bài 5.3.** Đếm số đơn hàng theo `status`.

**Bài 5.4.** Đếm số nhân viên theo `jobTitle`.

**Bài 5.5.** Đếm số văn phòng theo `country`.

---

## 6. Đặt bí danh cho kết quả tổng hợp

Khi dùng hàm tổng hợp, nên đặt bí danh bằng `AS` để kết quả dễ đọc.

Ví dụ:

```sql
SELECT country, COUNT(*) AS totalCustomers
FROM customers
GROUP BY country;
```

Không nên để kết quả hiển thị tên cột dạng `COUNT(*)`, vì khó đọc khi báo cáo hoặc khi sử dụng tiếp trong ứng dụng.

Ví dụ tính tổng tiền theo khách hàng:

```sql
SELECT customerNumber, SUM(amount) AS totalAmount
FROM payments
GROUP BY customerNumber;
```

### Bài tập thực hành

**Bài 6.1.** Đếm số khách hàng theo quốc gia, đặt bí danh là `SoKhachHang`.

**Bài 6.2.** Đếm số sản phẩm theo dòng sản phẩm, đặt bí danh là `SoSanPham`.

**Bài 6.3.** Tính tổng tiền thanh toán theo khách hàng, đặt bí danh là `TongThanhToan`.

**Bài 6.4.** Tính giá bán đề xuất trung bình theo dòng sản phẩm, đặt bí danh là `GiaTrungBinh`.

**Bài 6.5.** Đếm số đơn hàng theo trạng thái, đặt bí danh là `SoDonHang`.

---

## 7. Sắp xếp kết quả sau khi nhóm bằng `ORDER BY`

Sau khi nhóm dữ liệu, ta thường muốn sắp xếp kết quả theo giá trị tổng hợp.

Ví dụ sắp xếp các quốc gia theo số khách hàng giảm dần:

```sql
SELECT country, COUNT(*) AS totalCustomers
FROM customers
GROUP BY country
ORDER BY totalCustomers DESC;
```

Ví dụ sắp xếp dòng sản phẩm theo số lượng sản phẩm tăng dần:

```sql
SELECT productLine, COUNT(*) AS totalProducts
FROM products
GROUP BY productLine
ORDER BY totalProducts ASC;
```

### Bài tập thực hành

**Bài 7.1.** Đếm số khách hàng theo quốc gia và sắp xếp giảm dần theo số khách hàng.

**Bài 7.2.** Đếm số sản phẩm theo dòng sản phẩm và sắp xếp giảm dần theo số sản phẩm.

**Bài 7.3.** Tính tổng tiền thanh toán theo khách hàng và sắp xếp giảm dần theo tổng tiền.

**Bài 7.4.** Đếm số đơn hàng theo trạng thái và sắp xếp giảm dần theo số đơn hàng.

**Bài 7.5.** Tính giá trung bình theo dòng sản phẩm và sắp xếp giảm dần theo giá trung bình.

---

## 8. Giới hạn kết quả tổng hợp bằng `LIMIT`

Có thể dùng `LIMIT` sau `ORDER BY` để lấy các nhóm đứng đầu.

Ví dụ lấy 5 khách hàng có tổng thanh toán cao nhất:

```sql
SELECT customerNumber, SUM(amount) AS totalPayment
FROM payments
GROUP BY customerNumber
ORDER BY totalPayment DESC
LIMIT 5;
```

Ví dụ lấy 3 dòng sản phẩm có nhiều sản phẩm nhất:

```sql
SELECT productLine, COUNT(*) AS totalProducts
FROM products
GROUP BY productLine
ORDER BY totalProducts DESC
LIMIT 3;
```

### Bài tập thực hành

**Bài 8.1.** Lấy 5 quốc gia có nhiều khách hàng nhất.

**Bài 8.2.** Lấy 3 dòng sản phẩm có nhiều sản phẩm nhất.

**Bài 8.3.** Lấy 10 khách hàng có tổng thanh toán cao nhất.

**Bài 8.4.** Lấy 5 dòng sản phẩm có giá bán đề xuất trung bình cao nhất.

**Bài 8.5.** Lấy 3 trạng thái đơn hàng xuất hiện nhiều nhất.

---

## 9. Nhóm theo nhiều cột

Có thể nhóm dữ liệu theo nhiều cột. Khi đó, mỗi nhóm được xác định bởi tổ hợp giá trị của các cột đó.

Ví dụ đếm số khách hàng theo quốc gia và thành phố:

```sql
SELECT country, city, COUNT(*) AS totalCustomers
FROM customers
GROUP BY country, city
ORDER BY country ASC, city ASC;
```

Ví dụ đếm số đơn hàng theo năm và trạng thái:

```sql
SELECT YEAR(orderDate) AS orderYear, status, COUNT(*) AS totalOrders
FROM orders
GROUP BY YEAR(orderDate), status
ORDER BY orderYear ASC, status ASC;
```

### Bài tập thực hành

**Bài 9.1.** Đếm số khách hàng theo `country` và `city`.

**Bài 9.2.** Đếm số đơn hàng theo `status` và `customerNumber`.

**Bài 9.3.** Đếm số sản phẩm theo `productLine` và `productVendor`.

**Bài 9.4.** Đếm số nhân viên theo `officeCode` và `jobTitle`.

**Bài 9.5.** Đếm số đơn hàng theo năm của `orderDate` và `status`.

---

## 10. Dùng biểu thức trong `GROUP BY`

Không chỉ nhóm theo cột trực tiếp, ta còn có thể nhóm theo biểu thức.

Ví dụ nhóm đơn hàng theo năm:

```sql
SELECT YEAR(orderDate) AS orderYear, COUNT(*) AS totalOrders
FROM orders
GROUP BY YEAR(orderDate)
ORDER BY orderYear ASC;
```

Ví dụ nhóm thanh toán theo năm:

```sql
SELECT YEAR(paymentDate) AS paymentYear, SUM(amount) AS totalPayment
FROM payments
GROUP BY YEAR(paymentDate)
ORDER BY paymentYear ASC;
```

Ví dụ nhóm đơn hàng theo tháng trong từng năm:

```sql
SELECT 
    YEAR(orderDate) AS orderYear,
    MONTH(orderDate) AS orderMonth,
    COUNT(*) AS totalOrders
FROM orders
GROUP BY YEAR(orderDate), MONTH(orderDate)
ORDER BY orderYear ASC, orderMonth ASC;
```

### Bài tập thực hành

**Bài 10.1.** Đếm số đơn hàng theo năm của `orderDate`.

**Bài 10.2.** Tính tổng thanh toán theo năm của `paymentDate`.

**Bài 10.3.** Đếm số đơn hàng theo tháng và năm của `orderDate`.

**Bài 10.4.** Tính tổng thanh toán theo tháng và năm của `paymentDate`.

**Bài 10.5.** Đếm số đơn hàng theo năm và trạng thái.

---

## 11. Lọc dữ liệu trước khi nhóm bằng `WHERE`

`WHERE` được xử lý trước `GROUP BY`. Vì vậy, `WHERE` dùng để lọc các dòng dữ liệu gốc trước khi gom nhóm.

Ví dụ đếm số khách hàng theo thành phố, nhưng chỉ xét khách hàng ở `USA`:

```sql
SELECT city, COUNT(*) AS totalCustomers
FROM customers
WHERE country = 'USA'
GROUP BY city
ORDER BY totalCustomers DESC;
```

Ví dụ tính tổng tiền thanh toán theo khách hàng, nhưng chỉ xét các khoản thanh toán trong năm 2004:

```sql
SELECT customerNumber, SUM(amount) AS totalPayment
FROM payments
WHERE paymentDate BETWEEN '2004-01-01' AND '2004-12-31'
GROUP BY customerNumber
ORDER BY totalPayment DESC;
```

### Bài tập thực hành

**Bài 11.1.** Đếm số khách hàng theo thành phố, chỉ xét khách hàng ở `USA`.

**Bài 11.2.** Tính tổng thanh toán theo khách hàng, chỉ xét các khoản thanh toán trong năm 2004.

**Bài 11.3.** Đếm số đơn hàng theo trạng thái, chỉ xét các đơn hàng trong năm 2005.

**Bài 11.4.** Tính giá bán đề xuất trung bình theo dòng sản phẩm, chỉ xét sản phẩm có `MSRP > 100`.

**Bài 11.5.** Đếm số sản phẩm theo nhà cung cấp, chỉ xét dòng sản phẩm `Classic Cars`.

---

## 12. Lọc nhóm bằng `HAVING`

`HAVING` dùng để lọc kết quả sau khi đã nhóm dữ liệu.

Khác với `WHERE`, `HAVING` có thể dùng điều kiện trên hàm tổng hợp.

Ví dụ lấy các quốc gia có hơn 5 khách hàng:

```sql
SELECT country, COUNT(*) AS totalCustomers
FROM customers
GROUP BY country
HAVING COUNT(*) > 5
ORDER BY totalCustomers DESC;
```

Ví dụ lấy các khách hàng có tổng thanh toán lớn hơn 100000:

```sql
SELECT customerNumber, SUM(amount) AS totalPayment
FROM payments
GROUP BY customerNumber
HAVING SUM(amount) > 100000
ORDER BY totalPayment DESC;
```

### Bài tập thực hành

**Bài 12.1.** Lấy các quốc gia có nhiều hơn 5 khách hàng.

**Bài 12.2.** Lấy các dòng sản phẩm có nhiều hơn 10 sản phẩm.

**Bài 12.3.** Lấy các khách hàng có tổng thanh toán lớn hơn 100000.

**Bài 12.4.** Lấy các trạng thái đơn hàng có số đơn hàng lớn hơn 5.

**Bài 12.5.** Lấy các dòng sản phẩm có `MSRP` trung bình lớn hơn 100.

---

## 13. Kết hợp `WHERE` và `HAVING`

Một truy vấn có thể dùng cả `WHERE` và `HAVING`.

- `WHERE`: lọc từng dòng trước khi nhóm.
- `HAVING`: lọc từng nhóm sau khi nhóm.

Ví dụ: Trong năm 2004, tìm các khách hàng có tổng thanh toán lớn hơn 50000.

```sql
SELECT customerNumber, SUM(amount) AS totalPayment
FROM payments
WHERE paymentDate BETWEEN '2004-01-01' AND '2004-12-31'
GROUP BY customerNumber
HAVING SUM(amount) > 50000
ORDER BY totalPayment DESC;
```

Ý nghĩa:

1. Lấy dữ liệu từ bảng `payments`.
2. Chỉ giữ các dòng thanh toán trong năm 2004.
3. Nhóm theo `customerNumber`.
4. Tính tổng tiền mỗi khách hàng.
5. Chỉ giữ các khách hàng có tổng tiền lớn hơn 50000.
6. Sắp xếp giảm dần theo tổng tiền.

### Bài tập thực hành

**Bài 13.1.** Trong năm 2004, lấy các khách hàng có tổng thanh toán lớn hơn 50000.

**Bài 13.2.** Trong năm 2005, lấy các khách hàng có tổng thanh toán lớn hơn 30000.

**Bài 13.3.** Với các sản phẩm có `MSRP > 50`, lấy các dòng sản phẩm có nhiều hơn 5 sản phẩm.

**Bài 13.4.** Với khách hàng ở `USA`, lấy các thành phố có nhiều hơn 2 khách hàng.

**Bài 13.5.** Với đơn hàng có trạng thái khác `Cancelled`, thống kê số đơn hàng theo trạng thái và chỉ giữ nhóm có ít nhất 5 đơn hàng.

---

## 14. Thứ tự các mệnh đề trong truy vấn tổng hợp

Một truy vấn tổng hợp thường có thứ tự như sau:

```sql
SELECT group_column, aggregate_function(column)
FROM table_name
WHERE row_condition
GROUP BY group_column
HAVING group_condition
ORDER BY sort_column
LIMIT number;
```

Thứ tự viết thường gặp:

| Thứ tự | Mệnh đề | Vai trò |
|---|---|---|
| 1 | `SELECT` | Chọn cột hiển thị và hàm tổng hợp |
| 2 | `FROM` | Chọn bảng nguồn |
| 3 | `WHERE` | Lọc dòng trước khi nhóm |
| 4 | `GROUP BY` | Gom dòng thành nhóm |
| 5 | `HAVING` | Lọc nhóm sau khi tổng hợp |
| 6 | `ORDER BY` | Sắp xếp kết quả |
| 7 | `LIMIT` | Giới hạn số dòng kết quả |

Ví dụ hoàn chỉnh:

```sql
SELECT customerNumber, SUM(amount) AS totalPayment
FROM payments
WHERE paymentDate >= '2004-01-01'
GROUP BY customerNumber
HAVING SUM(amount) > 50000
ORDER BY totalPayment DESC
LIMIT 10;
```

### Bài tập thực hành

**Bài 14.1.** Viết truy vấn đầy đủ có `WHERE`, `GROUP BY`, `HAVING`, `ORDER BY`, `LIMIT` trên bảng `payments`.

**Bài 14.2.** Viết truy vấn đầy đủ có `WHERE`, `GROUP BY`, `HAVING`, `ORDER BY`, `LIMIT` trên bảng `orders`.

**Bài 14.3.** Viết truy vấn đầy đủ có `WHERE`, `GROUP BY`, `HAVING`, `ORDER BY`, `LIMIT` trên bảng `products`.

---

## 15. `COUNT(*)` và `COUNT(column)`

`COUNT(*)` đếm số dòng trong nhóm.

`COUNT(column)` chỉ đếm các dòng mà `column` khác `NULL`.

Ví dụ đếm tổng số khách hàng:

```sql
SELECT COUNT(*) AS totalCustomers
FROM customers;
```

Ví dụ đếm số khách hàng có nhân viên bán hàng phụ trách:

```sql
SELECT COUNT(salesRepEmployeeNumber) AS customersWithSalesRep
FROM customers;
```

Ví dụ theo quốc gia:

```sql
SELECT 
    country,
    COUNT(*) AS totalCustomers,
    COUNT(salesRepEmployeeNumber) AS customersWithSalesRep
FROM customers
GROUP BY country;
```

### Bài tập thực hành

**Bài 15.1.** So sánh `COUNT(*)` và `COUNT(salesRepEmployeeNumber)` trong bảng `customers`.

**Bài 15.2.** Theo từng quốc gia, đếm tổng số khách hàng và số khách hàng có nhân viên bán hàng phụ trách.

**Bài 15.3.** Theo từng văn phòng, đếm tổng số nhân viên và số nhân viên có quản lý trực tiếp.

**Bài 15.4.** Theo từng trạng thái đơn hàng, đếm tổng số đơn hàng và số đơn hàng có `shippedDate` khác `NULL`.

**Bài 15.5.** Giải thích tại sao `COUNT(*)` và `COUNT(column)` có thể cho kết quả khác nhau.

---

## 16. `COUNT(DISTINCT column)`

`COUNT(DISTINCT column)` dùng để đếm số giá trị khác nhau.

Ví dụ đếm số quốc gia có khách hàng:

```sql
SELECT COUNT(DISTINCT country) AS totalCountries
FROM customers;
```

Ví dụ đếm số dòng sản phẩm có trong bảng `products`:

```sql
SELECT COUNT(DISTINCT productLine) AS totalProductLines
FROM products;
```

Ví dụ đếm số khách hàng có thanh toán:

```sql
SELECT COUNT(DISTINCT customerNumber) AS customersWithPayments
FROM payments;
```

### Bài tập thực hành

**Bài 16.1.** Đếm số quốc gia khác nhau trong bảng `customers`.

**Bài 16.2.** Đếm số thành phố khác nhau trong bảng `customers`.

**Bài 16.3.** Đếm số dòng sản phẩm khác nhau trong bảng `products`.

**Bài 16.4.** Đếm số khách hàng có thanh toán trong bảng `payments`.

**Bài 16.5.** Đếm số sản phẩm đã xuất hiện trong bảng `orderdetails`.

---

## 17. Tính toán trên cột tổng hợp

Có thể tính toán trực tiếp trong hàm tổng hợp.

Ví dụ tính tổng giá trị từng dòng chi tiết đơn hàng:

```sql
SELECT 
    orderNumber,
    SUM(quantityOrdered * priceEach) AS orderTotal
FROM orderdetails
GROUP BY orderNumber
ORDER BY orderTotal DESC;
```

Ví dụ tính doanh thu theo sản phẩm:

```sql
SELECT 
    productCode,
    SUM(quantityOrdered * priceEach) AS productRevenue
FROM orderdetails
GROUP BY productCode
ORDER BY productRevenue DESC;
```

Ví dụ tính số lượng sản phẩm đã bán theo sản phẩm:

```sql
SELECT 
    productCode,
    SUM(quantityOrdered) AS totalQuantitySold
FROM orderdetails
GROUP BY productCode
ORDER BY totalQuantitySold DESC;
```

### Bài tập thực hành

**Bài 17.1.** Tính tổng giá trị của mỗi đơn hàng từ bảng `orderdetails`.

**Bài 17.2.** Tính doanh thu theo từng sản phẩm từ bảng `orderdetails`.

**Bài 17.3.** Tính tổng số lượng bán theo từng sản phẩm.

**Bài 17.4.** Lấy 10 đơn hàng có tổng giá trị cao nhất.

**Bài 17.5.** Lấy các sản phẩm có tổng số lượng bán lớn hơn 1000.

---

## 18. Kết hợp `GROUP BY` với `JOIN`

Trong thực tế, dữ liệu cần báo cáo thường nằm ở nhiều bảng. Khi đó, ta dùng `JOIN` trước, rồi `GROUP BY` sau.

Ví dụ đếm số đơn hàng theo tên khách hàng:

```sql
SELECT 
    c.customerNumber,
    c.customerName,
    COUNT(o.orderNumber) AS totalOrders
FROM customers c
JOIN orders o
    ON c.customerNumber = o.customerNumber
GROUP BY c.customerNumber, c.customerName
ORDER BY totalOrders DESC;
```

Ví dụ tính tổng giá trị mỗi đơn hàng kèm mã khách hàng:

```sql
SELECT 
    o.orderNumber,
    o.customerNumber,
    SUM(od.quantityOrdered * od.priceEach) AS orderTotal
FROM orders o
JOIN orderdetails od
    ON o.orderNumber = od.orderNumber
GROUP BY o.orderNumber, o.customerNumber
ORDER BY orderTotal DESC;
```

Ví dụ tính doanh thu theo dòng sản phẩm:

```sql
SELECT 
    p.productLine,
    SUM(od.quantityOrdered * od.priceEach) AS revenue
FROM products p
JOIN orderdetails od
    ON p.productCode = od.productCode
GROUP BY p.productLine
ORDER BY revenue DESC;
```

### Bài tập thực hành

**Bài 18.1.** Đếm số đơn hàng theo từng khách hàng, hiển thị `customerNumber`, `customerName`, `totalOrders`.

**Bài 18.2.** Tính tổng giá trị mỗi đơn hàng, hiển thị `orderNumber`, `customerNumber`, `orderTotal`.

**Bài 18.3.** Tính doanh thu theo dòng sản phẩm.

**Bài 18.4.** Tính tổng số lượng bán theo dòng sản phẩm.

**Bài 18.5.** Đếm số khách hàng theo từng nhân viên bán hàng, hiển thị mã nhân viên và số khách hàng.

---

## 19. `LEFT JOIN` và thống kê cả nhóm chưa có dữ liệu liên quan

`JOIN` thông thường chỉ giữ các bản ghi có dữ liệu khớp ở cả hai bảng.

`LEFT JOIN` giữ toàn bộ dòng ở bảng bên trái, kể cả khi không có dòng tương ứng ở bảng bên phải.

Ví dụ đếm số khách hàng theo nhân viên bán hàng, kể cả nhân viên chưa phụ trách khách hàng nào:

```sql
SELECT 
    e.employeeNumber,
    e.lastName,
    e.firstName,
    COUNT(c.customerNumber) AS totalCustomers
FROM employees e
LEFT JOIN customers c
    ON e.employeeNumber = c.salesRepEmployeeNumber
GROUP BY e.employeeNumber, e.lastName, e.firstName
ORDER BY totalCustomers DESC;
```

Ví dụ đếm số đơn hàng theo khách hàng, kể cả khách hàng chưa đặt đơn hàng:

```sql
SELECT 
    c.customerNumber,
    c.customerName,
    COUNT(o.orderNumber) AS totalOrders
FROM customers c
LEFT JOIN orders o
    ON c.customerNumber = o.customerNumber
GROUP BY c.customerNumber, c.customerName
ORDER BY totalOrders DESC;
```

### Bài tập thực hành

**Bài 19.1.** Đếm số khách hàng theo từng nhân viên bằng `LEFT JOIN`, kể cả nhân viên chưa có khách hàng.

**Bài 19.2.** Đếm số đơn hàng theo từng khách hàng bằng `LEFT JOIN`, kể cả khách hàng chưa có đơn hàng.

**Bài 19.3.** Đếm số sản phẩm theo dòng sản phẩm bằng `LEFT JOIN` giữa `productlines` và `products`.

**Bài 19.4.** Tìm các khách hàng chưa có đơn hàng bằng `LEFT JOIN`, `GROUP BY`, `HAVING`.

**Bài 19.5.** Tìm các nhân viên chưa phụ trách khách hàng nào bằng `LEFT JOIN`, `GROUP BY`, `HAVING`.

---

## 20. Lọc nhóm sau `LEFT JOIN`

Khi dùng `LEFT JOIN`, có thể kết hợp `GROUP BY` và `HAVING` để tìm các nhóm không có dữ liệu liên quan.

Ví dụ tìm khách hàng chưa có đơn hàng:

```sql
SELECT 
    c.customerNumber,
    c.customerName,
    COUNT(o.orderNumber) AS totalOrders
FROM customers c
LEFT JOIN orders o
    ON c.customerNumber = o.customerNumber
GROUP BY c.customerNumber, c.customerName
HAVING COUNT(o.orderNumber) = 0;
```

Ví dụ tìm nhân viên chưa phụ trách khách hàng nào:

```sql
SELECT 
    e.employeeNumber,
    e.lastName,
    e.firstName,
    COUNT(c.customerNumber) AS totalCustomers
FROM employees e
LEFT JOIN customers c
    ON e.employeeNumber = c.salesRepEmployeeNumber
GROUP BY e.employeeNumber, e.lastName, e.firstName
HAVING COUNT(c.customerNumber) = 0;
```

### Bài tập thực hành

**Bài 20.1.** Tìm các khách hàng có đúng 0 đơn hàng.

**Bài 20.2.** Tìm các khách hàng có từ 5 đơn hàng trở lên.

**Bài 20.3.** Tìm các nhân viên có đúng 0 khách hàng phụ trách.

**Bài 20.4.** Tìm các nhân viên có hơn 5 khách hàng phụ trách.

**Bài 20.5.** Tìm các dòng sản phẩm có nhiều hơn 10 sản phẩm.

---

## 21. Một số bài toán báo cáo thường gặp

### 21.1. Doanh thu theo dòng sản phẩm

```sql
SELECT 
    p.productLine,
    SUM(od.quantityOrdered * od.priceEach) AS revenue
FROM products p
JOIN orderdetails od
    ON p.productCode = od.productCode
GROUP BY p.productLine
ORDER BY revenue DESC;
```

### 21.2. Doanh thu theo khách hàng

```sql
SELECT 
    c.customerNumber,
    c.customerName,
    SUM(od.quantityOrdered * od.priceEach) AS revenue
FROM customers c
JOIN orders o
    ON c.customerNumber = o.customerNumber
JOIN orderdetails od
    ON o.orderNumber = od.orderNumber
GROUP BY c.customerNumber, c.customerName
ORDER BY revenue DESC;
```

### 21.3. Doanh thu theo năm

```sql
SELECT 
    YEAR(o.orderDate) AS orderYear,
    SUM(od.quantityOrdered * od.priceEach) AS revenue
FROM orders o
JOIN orderdetails od
    ON o.orderNumber = od.orderNumber
GROUP BY YEAR(o.orderDate)
ORDER BY orderYear ASC;
```

### 21.4. Số đơn hàng theo khách hàng

```sql
SELECT 
    c.customerNumber,
    c.customerName,
    COUNT(o.orderNumber) AS totalOrders
FROM customers c
JOIN orders o
    ON c.customerNumber = o.customerNumber
GROUP BY c.customerNumber, c.customerName
ORDER BY totalOrders DESC;
```

### 21.5. Tổng thanh toán theo khách hàng

```sql
SELECT 
    c.customerNumber,
    c.customerName,
    SUM(p.amount) AS totalPayment
FROM customers c
JOIN payments p
    ON c.customerNumber = p.customerNumber
GROUP BY c.customerNumber, c.customerName
ORDER BY totalPayment DESC;
```

### Bài tập thực hành

**Bài 21.1.** Viết truy vấn doanh thu theo dòng sản phẩm.

**Bài 21.2.** Viết truy vấn doanh thu theo khách hàng.

**Bài 21.3.** Viết truy vấn doanh thu theo năm.

**Bài 21.4.** Viết truy vấn số đơn hàng theo khách hàng.

**Bài 21.5.** Viết truy vấn tổng thanh toán theo khách hàng.

---

## 22. So sánh `WHERE` và `HAVING`

| Tiêu chí | `WHERE` | `HAVING` |
|---|---|---|
| Thời điểm xử lý | Trước khi nhóm | Sau khi nhóm |
| Lọc cái gì? | Lọc từng dòng dữ liệu gốc | Lọc từng nhóm sau tổng hợp |
| Dùng được hàm tổng hợp? | Thường không dùng trực tiếp | Có |
| Đi cùng | `SELECT`, `FROM` | `GROUP BY` |
| Ví dụ | `WHERE country = 'USA'` | `HAVING COUNT(*) > 5` |

Ví dụ đúng:

```sql
SELECT country, COUNT(*) AS totalCustomers
FROM customers
GROUP BY country
HAVING COUNT(*) > 5;
```

Ví dụ sai:

```sql
SELECT country, COUNT(*) AS totalCustomers
FROM customers
WHERE COUNT(*) > 5
GROUP BY country;
```

### Bài tập thực hành

**Bài 22.1.** Giải thích vì sao không nên dùng `WHERE COUNT(*) > 5`.

**Bài 22.2.** Viết truy vấn dùng `WHERE` để lọc khách hàng ở `USA` trước khi nhóm theo `city`.

**Bài 22.3.** Viết truy vấn dùng `HAVING` để lọc các quốc gia có hơn 5 khách hàng.

**Bài 22.4.** Viết truy vấn dùng cả `WHERE` và `HAVING` trên bảng `payments`.

**Bài 22.5.** Nêu một ví dụ thực tế cần dùng `HAVING`.

---

## 23. Một số lỗi thường gặp

### Lỗi 1: Dùng hàm tổng hợp trong `WHERE`

Sai:

```sql
SELECT country, COUNT(*) AS totalCustomers
FROM customers
WHERE COUNT(*) > 5
GROUP BY country;
```

Đúng:

```sql
SELECT country, COUNT(*) AS totalCustomers
FROM customers
GROUP BY country
HAVING COUNT(*) > 5;
```

---

### Lỗi 2: Quên `GROUP BY` khi chọn cột thường cùng hàm tổng hợp

Dễ gây lỗi hoặc kết quả khó hiểu:

```sql
SELECT country, COUNT(*) AS totalCustomers
FROM customers;
```

Nên viết:

```sql
SELECT country, COUNT(*) AS totalCustomers
FROM customers
GROUP BY country;
```

---

### Lỗi 3: Nhầm `COUNT(*)` và `COUNT(column)`

```sql
SELECT COUNT(*) AS totalCustomers,
       COUNT(salesRepEmployeeNumber) AS customersWithSalesRep
FROM customers;
```

Hai kết quả có thể khác nhau vì `COUNT(column)` không đếm giá trị `NULL`.

---

### Lỗi 4: Chọn cột không nằm trong `GROUP BY`

Không nên viết:

```sql
SELECT country, city, COUNT(*) AS totalCustomers
FROM customers
GROUP BY country;
```

Nên viết rõ nhóm theo cả hai cột nếu muốn hiển thị cả `country` và `city`:

```sql
SELECT country, city, COUNT(*) AS totalCustomers
FROM customers
GROUP BY country, city;
```

---

### Lỗi 5: Dùng `JOIN` làm tăng số dòng rồi tổng hợp sai

Khi `JOIN` bảng cha với bảng con, số dòng có thể tăng lên. Cần hiểu quan hệ giữa các bảng trước khi tổng hợp.

Ví dụ: một đơn hàng có nhiều dòng trong `orderdetails`, nên khi nối `orders` với `orderdetails`, mỗi đơn hàng có thể xuất hiện nhiều lần.

---

## 24. Bài tập tổng hợp cuối lab

### Bài 24.1. Thống kê khách hàng

Viết truy vấn hiển thị số khách hàng theo từng quốc gia, sắp xếp theo số khách hàng giảm dần.

### Bài 24.2. Thống kê sản phẩm

Viết truy vấn hiển thị số sản phẩm, giá mua trung bình và `MSRP` trung bình theo từng dòng sản phẩm.

### Bài 24.3. Thống kê thanh toán

Viết truy vấn hiển thị tổng thanh toán theo từng khách hàng, chỉ lấy các khách hàng có tổng thanh toán lớn hơn 100000.

### Bài 24.4. Thống kê đơn hàng

Viết truy vấn hiển thị số đơn hàng theo từng trạng thái trong năm 2004.

### Bài 24.5. Doanh thu theo đơn hàng

Viết truy vấn tính tổng giá trị từng đơn hàng từ bảng `orderdetails`.

### Bài 24.6. Doanh thu theo sản phẩm

Viết truy vấn tính doanh thu theo từng sản phẩm, sắp xếp giảm dần theo doanh thu.

### Bài 24.7. Doanh thu theo dòng sản phẩm

Viết truy vấn tính doanh thu theo từng dòng sản phẩm bằng cách kết hợp `products` và `orderdetails`.

### Bài 24.8. Khách hàng chưa có đơn hàng

Dùng `LEFT JOIN`, `GROUP BY`, `HAVING` để tìm khách hàng chưa có đơn hàng nào.

### Bài 24.9. Nhân viên chưa phụ trách khách hàng

Dùng `LEFT JOIN`, `GROUP BY`, `HAVING` để tìm nhân viên chưa phụ trách khách hàng nào.

### Bài 24.10. Báo cáo doanh thu theo năm

Viết truy vấn tính doanh thu theo từng năm dựa trên `orders` và `orderdetails`.

---

## 25. Đáp án gợi ý cho một số bài tập

### Bài 5.1

```sql
SELECT country, COUNT(*) AS totalCustomers
FROM customers
GROUP BY country;
```

### Bài 5.2

```sql
SELECT productLine, COUNT(*) AS totalProducts
FROM products
GROUP BY productLine;
```

### Bài 7.1

```sql
SELECT country, COUNT(*) AS totalCustomers
FROM customers
GROUP BY country
ORDER BY totalCustomers DESC;
```

### Bài 8.1

```sql
SELECT country, COUNT(*) AS totalCustomers
FROM customers
GROUP BY country
ORDER BY totalCustomers DESC
LIMIT 5;
```

### Bài 10.1

```sql
SELECT YEAR(orderDate) AS orderYear, COUNT(*) AS totalOrders
FROM orders
GROUP BY YEAR(orderDate)
ORDER BY orderYear ASC;
```

### Bài 11.2

```sql
SELECT customerNumber, SUM(amount) AS totalPayment
FROM payments
WHERE paymentDate BETWEEN '2004-01-01' AND '2004-12-31'
GROUP BY customerNumber
ORDER BY totalPayment DESC;
```

### Bài 12.1

```sql
SELECT country, COUNT(*) AS totalCustomers
FROM customers
GROUP BY country
HAVING COUNT(*) > 5
ORDER BY totalCustomers DESC;
```

### Bài 13.1

```sql
SELECT customerNumber, SUM(amount) AS totalPayment
FROM payments
WHERE paymentDate BETWEEN '2004-01-01' AND '2004-12-31'
GROUP BY customerNumber
HAVING SUM(amount) > 50000
ORDER BY totalPayment DESC;
```

### Bài 17.1

```sql
SELECT 
    orderNumber,
    SUM(quantityOrdered * priceEach) AS orderTotal
FROM orderdetails
GROUP BY orderNumber
ORDER BY orderTotal DESC;
```

### Bài 18.3

```sql
SELECT 
    p.productLine,
    SUM(od.quantityOrdered * od.priceEach) AS revenue
FROM products p
JOIN orderdetails od
    ON p.productCode = od.productCode
GROUP BY p.productLine
ORDER BY revenue DESC;
```

### Bài 19.2

```sql
SELECT 
    c.customerNumber,
    c.customerName,
    COUNT(o.orderNumber) AS totalOrders
FROM customers c
LEFT JOIN orders o
    ON c.customerNumber = o.customerNumber
GROUP BY c.customerNumber, c.customerName
ORDER BY totalOrders DESC;
```

### Bài 20.1

```sql
SELECT 
    c.customerNumber,
    c.customerName,
    COUNT(o.orderNumber) AS totalOrders
FROM customers c
LEFT JOIN orders o
    ON c.customerNumber = o.customerNumber
GROUP BY c.customerNumber, c.customerName
HAVING COUNT(o.orderNumber) = 0;
```

---

## 26. Tóm tắt

Các kiến thức chính trong lab:

- Hàm tổng hợp dùng để tính toán trên nhiều dòng dữ liệu.
- `GROUP BY` dùng để gom các dòng thành từng nhóm.
- Có thể nhóm theo một cột, nhiều cột hoặc biểu thức.
- `WHERE` lọc dữ liệu trước khi nhóm.
- `HAVING` lọc nhóm sau khi tổng hợp.
- `ORDER BY` có thể sắp xếp theo kết quả tổng hợp.
- `LIMIT` có thể dùng để lấy các nhóm đứng đầu.
- `COUNT(*)` đếm số dòng, còn `COUNT(column)` không đếm giá trị `NULL`.
- `COUNT(DISTINCT column)` đếm số giá trị khác nhau.
- Có thể kết hợp `GROUP BY` với `JOIN` để tạo báo cáo từ nhiều bảng.
- `LEFT JOIN` hữu ích khi muốn thống kê cả các đối tượng chưa có dữ liệu liên quan.

---

## 27. Từ khóa chính

- SQL
- MySQL
- classicmodels
- SELECT
- GROUP BY
- HAVING
- WHERE
- ORDER BY
- LIMIT
- COUNT
- COUNT DISTINCT
- SUM
- AVG
- MIN
- MAX
- JOIN
- LEFT JOIN
- Aggregate function
- Grouping
- Filtering groups
- Report query
