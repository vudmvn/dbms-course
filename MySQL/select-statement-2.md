---
title: "Tutorial: Sử dụng SELECT và JOIN với classicmodels"
author: "Tên giảng viên"
duration: "120m"
difficulty: "Beginner - Intermediate"
prerequisites:
    - "Đã biết khái niệm bảng, cột, dòng trong CSDL quan hệ"
    - "Đã biết khóa chính và khóa ngoại"
    - "Đã biết SELECT cơ bản"
    - "Đã biết cách mở MySQL client hoặc MySQL Workbench"
summary: "Thực hành truy vấn dữ liệu bằng SELECT và kết hợp nhiều bảng bằng JOIN trên cơ sở dữ liệu mẫu classicmodels."
---

# Tutorial: Sử dụng câu lệnh `SELECT` và `JOIN` với `classicmodels`

## 1. Mục tiêu học tập

Sau khi hoàn thành tutorial này, người học có thể:

1. Nhắc lại được vai trò của câu lệnh `SELECT`.
2. Truy vấn dữ liệu từ một bảng bằng `SELECT ... FROM ...`.
3. Lọc dữ liệu bằng `WHERE`.
4. Sắp xếp dữ liệu bằng `ORDER BY`.
5. Giới hạn số dòng kết quả bằng `LIMIT`.
6. Hiểu vì sao cần dùng `JOIN` khi dữ liệu nằm ở nhiều bảng.
7. Giải thích được mối liên hệ giữa khóa chính và khóa ngoại khi viết `JOIN`.
8. Viết được truy vấn `INNER JOIN` để lấy dữ liệu có liên hệ ở cả hai bảng.
9. Viết được truy vấn `LEFT JOIN` để giữ lại toàn bộ dòng của bảng bên trái.
10. Viết được truy vấn nối nhiều hơn hai bảng.
11. Viết được truy vấn `SELF JOIN` đơn giản trên bảng có khóa ngoại tự tham chiếu.
12. Nhận biết một số lỗi thường gặp khi viết `JOIN`.

---

## 2. Giới thiệu cơ sở dữ liệu mẫu `classicmodels`

`classicmodels` là cơ sở dữ liệu mẫu dùng để thực hành SQL. Cơ sở dữ liệu mô phỏng hoạt động của một công ty bán các mô hình xe, tàu, máy bay và một số sản phẩm sưu tầm.

Một số bảng chính trong cơ sở dữ liệu:

| Bảng | Ý nghĩa |
|---|---|
| `productlines` | Nhóm hoặc dòng sản phẩm |
| `products` | Thông tin sản phẩm |
| `offices` | Văn phòng của công ty |
| `employees` | Nhân viên |
| `customers` | Khách hàng |
| `payments` | Thanh toán của khách hàng |
| `orders` | Đơn hàng |
| `orderdetails` | Chi tiết từng dòng sản phẩm trong đơn hàng |

Trong tutorial này, ta tập trung vào hai nhóm kiến thức:

1. Truy vấn dữ liệu bằng `SELECT`.
2. Kết hợp dữ liệu từ nhiều bảng bằng `JOIN`.

---

### Bài tập thực hành

**Bài 2.1.** Liệt kê 8 bảng chính trong cơ sở dữ liệu `classicmodels`.

**Bài 2.2.** Bảng nào lưu thông tin sản phẩm?

**Bài 2.3.** Bảng nào lưu thông tin khách hàng?

**Bài 2.4.** Bảng nào lưu thông tin đơn hàng?

**Bài 2.5.** Bảng nào lưu chi tiết các sản phẩm trong từng đơn hàng?

---

## 3. Chuẩn bị môi trường

Trước khi thực hành, cần chắc chắn rằng cơ sở dữ liệu đã được tạo và được chọn bằng lệnh:

```sql
USE classicmodels;
```

Để xem danh sách các bảng:

```sql
SHOW TABLES;
```

Để xem cấu trúc của một bảng:

```sql
DESC customers;
```

Ví dụ xem cấu trúc bảng `products`:

```sql
DESC products;
```

Ví dụ xem cấu trúc bảng `orders`:

```sql
DESC orders;
```

---

### Bài tập thực hành

**Bài 3.1.** Viết lệnh chọn cơ sở dữ liệu `classicmodels`.

**Bài 3.2.** Viết lệnh xem danh sách bảng trong cơ sở dữ liệu hiện tại.

**Bài 3.3.** Viết lệnh xem cấu trúc bảng `products`.

**Bài 3.4.** Viết lệnh xem cấu trúc bảng `customers`.

**Bài 3.5.** Viết lệnh xem cấu trúc bảng `orderdetails`.

---

## 4. Sơ đồ quan hệ chính trong `classicmodels`

Để viết `JOIN`, cần hiểu các cặp bảng được liên kết với nhau qua khóa chính và khóa ngoại.

Một số quan hệ quan trọng:

| Bảng con | Khóa ngoại | Bảng cha | Khóa chính |
|---|---|---|---|
| `products` | `productLine` | `productlines` | `productLine` |
| `employees` | `officeCode` | `offices` | `officeCode` |
| `employees` | `reportsTo` | `employees` | `employeeNumber` |
| `customers` | `salesRepEmployeeNumber` | `employees` | `employeeNumber` |
| `payments` | `customerNumber` | `customers` | `customerNumber` |
| `orders` | `customerNumber` | `customers` | `customerNumber` |
| `orderdetails` | `orderNumber` | `orders` | `orderNumber` |
| `orderdetails` | `productCode` | `products` | `productCode` |

Có thể hình dung một số luồng dữ liệu thường dùng:

```text
customers -> orders -> orderdetails -> products -> productlines
customers -> payments
customers -> employees -> offices
employees -> employees
```

---

### Bài tập thực hành

**Bài 4.1.** Xác định khóa ngoại trong bảng `products`.

**Bài 4.2.** Xác định khóa ngoại trong bảng `orders`.

**Bài 4.3.** Xác định hai khóa ngoại trong bảng `orderdetails`.

**Bài 4.4.** Xác định khóa ngoại trong bảng `customers`.

**Bài 4.5.** Vì sao bảng `employees` có thể nối với chính nó?

---

## 5. Nhắc lại cú pháp `SELECT` cơ bản

Câu lệnh `SELECT` dùng để lấy dữ liệu từ bảng.

Cú pháp tổng quát:

```sql
SELECT column1, column2, ...
FROM table_name;
```

Ví dụ lấy một số cột từ bảng `customers`:

```sql
SELECT customerNumber, customerName, city, country
FROM customers;
```

Ví dụ lấy một số cột từ bảng `products`:

```sql
SELECT productCode, productName, productLine, MSRP
FROM products;
```

Ví dụ lấy một số cột từ bảng `orders`:

```sql
SELECT orderNumber, orderDate, status, customerNumber
FROM orders;
```

---

### Bài tập thực hành

**Bài 5.1.** Viết truy vấn lấy `customerNumber`, `customerName`, `country` từ bảng `customers`.

**Bài 5.2.** Viết truy vấn lấy `productCode`, `productName`, `productLine` từ bảng `products`.

**Bài 5.3.** Viết truy vấn lấy `orderNumber`, `orderDate`, `status` từ bảng `orders`.

**Bài 5.4.** Viết truy vấn lấy `employeeNumber`, `lastName`, `firstName`, `jobTitle` từ bảng `employees`.

**Bài 5.5.** Viết truy vấn lấy `officeCode`, `city`, `country` từ bảng `offices`.

---

## 6. Lọc dữ liệu bằng `WHERE`

Mệnh đề `WHERE` dùng để chỉ lấy các dòng thỏa mãn điều kiện.

Ví dụ lấy khách hàng ở `USA`:

```sql
SELECT customerNumber, customerName, city, country
FROM customers
WHERE country = 'USA';
```

Ví dụ lấy sản phẩm thuộc dòng `Classic Cars`:

```sql
SELECT productCode, productName, productLine
FROM products
WHERE productLine = 'Classic Cars';
```

Ví dụ lấy đơn hàng có trạng thái `Cancelled`:

```sql
SELECT orderNumber, orderDate, status
FROM orders
WHERE status = 'Cancelled';
```

---

### Bài tập thực hành

**Bài 6.1.** Viết truy vấn lấy các khách hàng ở `France`.

**Bài 6.2.** Viết truy vấn lấy các sản phẩm thuộc dòng `Motorcycles`.

**Bài 6.3.** Viết truy vấn lấy các đơn hàng có trạng thái `Shipped`.

**Bài 6.4.** Viết truy vấn lấy các văn phòng ở `USA`.

**Bài 6.5.** Viết truy vấn lấy các nhân viên có `jobTitle = 'Sales Rep'`.

---

## 7. Sắp xếp và giới hạn kết quả

`ORDER BY` dùng để sắp xếp kết quả.

Ví dụ sắp xếp sản phẩm theo giá giảm dần:

```sql
SELECT productCode, productName, MSRP
FROM products
ORDER BY MSRP DESC;
```

`LIMIT` dùng để giới hạn số dòng kết quả.

Ví dụ lấy 5 sản phẩm có giá bán đề xuất cao nhất:

```sql
SELECT productCode, productName, MSRP
FROM products
ORDER BY MSRP DESC
LIMIT 5;
```

Ví dụ lấy 10 khách hàng có hạn mức tín dụng cao nhất:

```sql
SELECT customerNumber, customerName, creditLimit
FROM customers
ORDER BY creditLimit DESC
LIMIT 10;
```

---

### Bài tập thực hành

**Bài 7.1.** Viết truy vấn lấy 5 sản phẩm có `MSRP` cao nhất.

**Bài 7.2.** Viết truy vấn lấy 10 khách hàng có `creditLimit` cao nhất.

**Bài 7.3.** Viết truy vấn lấy 5 khoản thanh toán có `amount` lớn nhất.

**Bài 7.4.** Viết truy vấn lấy 10 đơn hàng mới nhất theo `orderDate`.

**Bài 7.5.** Viết truy vấn lấy 10 sản phẩm có `quantityInStock` thấp nhất.

---

## 8. Vì sao cần dùng `JOIN`?

Dữ liệu trong cơ sở dữ liệu quan hệ thường được tách thành nhiều bảng để tránh dư thừa.

Ví dụ:

- Bảng `orders` chỉ lưu `customerNumber`.
- Tên khách hàng lại nằm trong bảng `customers`.

Nếu muốn xem đơn hàng kèm tên khách hàng, cần kết hợp hai bảng:

```text
orders.customerNumber = customers.customerNumber
```

Tương tự:

- Muốn xem sản phẩm kèm mô tả dòng sản phẩm, nối `products` với `productlines`.
- Muốn xem khách hàng kèm nhân viên bán hàng phụ trách, nối `customers` với `employees`.
- Muốn xem chi tiết đơn hàng kèm tên sản phẩm, nối `orderdetails` với `products`.

---

### Bài tập thực hành

**Bài 8.1.** Muốn biết tên khách hàng của mỗi đơn hàng, cần nối hai bảng nào?

**Bài 8.2.** Muốn biết tên sản phẩm trong từng dòng chi tiết đơn hàng, cần nối hai bảng nào?

**Bài 8.3.** Muốn biết văn phòng của mỗi nhân viên, cần nối hai bảng nào?

**Bài 8.4.** Muốn biết dòng sản phẩm của mỗi sản phẩm, cần nối hai bảng nào?

**Bài 8.5.** Muốn biết nhân viên bán hàng phụ trách khách hàng, cần nối hai bảng nào?

---

## 9. Cú pháp tổng quát của `INNER JOIN`

`INNER JOIN` trả về các dòng có dữ liệu khớp ở cả hai bảng.

Cú pháp:

```sql
SELECT table1.column1, table2.column2
FROM table1
INNER JOIN table2
    ON table1.key_column = table2.key_column;
```

Trong đó:

| Thành phần | Ý nghĩa |
|---|---|
| `FROM table1` | Bảng bắt đầu truy vấn |
| `INNER JOIN table2` | Bảng cần nối thêm |
| `ON` | Điều kiện nối hai bảng |
| `table1.key_column = table2.key_column` | Cặp khóa dùng để nối |

Ví dụ nối `orders` và `customers`:

```sql
SELECT 
    orders.orderNumber,
    orders.orderDate,
    orders.status,
    customers.customerName
FROM orders
INNER JOIN customers
    ON orders.customerNumber = customers.customerNumber;
```

---

### Bài tập thực hành

**Bài 9.1.** Viết truy vấn nối `orders` và `customers` để lấy `orderNumber`, `orderDate`, `status`, `customerName`.

**Bài 9.2.** Viết truy vấn nối `products` và `productlines` để lấy `productCode`, `productName`, `productLine`, `textDescription`.

**Bài 9.3.** Viết truy vấn nối `employees` và `offices` để lấy `employeeNumber`, `lastName`, `firstName`, `city`, `country`.

**Bài 9.4.** Viết truy vấn nối `payments` và `customers` để lấy `checkNumber`, `paymentDate`, `amount`, `customerName`.

**Bài 9.5.** Viết truy vấn nối `orderdetails` và `products` để lấy `orderNumber`, `productCode`, `productName`, `quantityOrdered`, `priceEach`.

---

## 10. Đặt bí danh cho bảng khi viết `JOIN`

Khi truy vấn có nhiều bảng, nên dùng bí danh để câu lệnh ngắn và dễ đọc.

Ví dụ:

```sql
SELECT 
    o.orderNumber,
    o.orderDate,
    o.status,
    c.customerName
FROM orders AS o
INNER JOIN customers AS c
    ON o.customerNumber = c.customerNumber;
```

Trong đó:

| Bí danh | Bảng đầy đủ |
|---|---|
| `o` | `orders` |
| `c` | `customers` |

Có thể bỏ từ khóa `AS` khi đặt bí danh cho bảng:

```sql
SELECT 
    o.orderNumber,
    o.orderDate,
    c.customerName
FROM orders o
INNER JOIN customers c
    ON o.customerNumber = c.customerNumber;
```

Khi mới học, nên dùng `AS` để truy vấn rõ nghĩa hơn.

---

### Bài tập thực hành

**Bài 10.1.** Viết lại truy vấn nối `orders` và `customers` bằng bí danh `o` và `c`.

**Bài 10.2.** Viết truy vấn nối `products` và `productlines` bằng bí danh `p` và `pl`.

**Bài 10.3.** Viết truy vấn nối `employees` và `offices` bằng bí danh `e` và `o`.

**Bài 10.4.** Viết truy vấn nối `payments` và `customers` bằng bí danh `p` và `c`.

**Bài 10.5.** Viết truy vấn nối `orderdetails` và `products` bằng bí danh `od` và `p`.

---

## 11. `INNER JOIN` giữa `products` và `productlines`

Bảng `products` có khóa ngoại `productLine` tham chiếu đến bảng `productlines`.

Mối liên hệ:

```text
products.productLine = productlines.productLine
```

Truy vấn:

```sql
SELECT 
    p.productCode,
    p.productName,
    p.productLine,
    pl.textDescription
FROM products AS p
INNER JOIN productlines AS pl
    ON p.productLine = pl.productLine;
```

Có thể thêm điều kiện lọc:

```sql
SELECT 
    p.productCode,
    p.productName,
    p.productLine,
    p.MSRP
FROM products AS p
INNER JOIN productlines AS pl
    ON p.productLine = pl.productLine
WHERE p.productLine = 'Classic Cars'
ORDER BY p.MSRP DESC
LIMIT 10;
```

---

### Bài tập thực hành

**Bài 11.1.** Nối `products` và `productlines`, lấy `productCode`, `productName`, `productLine`.

**Bài 11.2.** Nối `products` và `productlines`, lấy các sản phẩm thuộc `Motorcycles`.

**Bài 11.3.** Nối `products` và `productlines`, lấy 5 sản phẩm có `MSRP` cao nhất.

**Bài 11.4.** Nối `products` và `productlines`, lấy `productName`, `productVendor`, `textDescription`.

**Bài 11.5.** Nối `products` và `productlines`, lấy sản phẩm có `quantityInStock < 1000`.

---

## 12. `INNER JOIN` giữa `orders` và `customers`

Bảng `orders` có khóa ngoại `customerNumber` tham chiếu đến bảng `customers`.

Mối liên hệ:

```text
orders.customerNumber = customers.customerNumber
```

Truy vấn:

```sql
SELECT 
    o.orderNumber,
    o.orderDate,
    o.requiredDate,
    o.status,
    c.customerName,
    c.country
FROM orders AS o
INNER JOIN customers AS c
    ON o.customerNumber = c.customerNumber;
```

Thêm điều kiện lọc đơn hàng bị hủy:

```sql
SELECT 
    o.orderNumber,
    o.orderDate,
    o.status,
    c.customerName
FROM orders AS o
INNER JOIN customers AS c
    ON o.customerNumber = c.customerNumber
WHERE o.status = 'Cancelled';
```

---

### Bài tập thực hành

**Bài 12.1.** Nối `orders` và `customers`, lấy `orderNumber`, `orderDate`, `status`, `customerName`.

**Bài 12.2.** Lấy các đơn hàng có trạng thái `Cancelled` kèm tên khách hàng.

**Bài 12.3.** Lấy các đơn hàng của khách hàng ở `France`.

**Bài 12.4.** Lấy 10 đơn hàng mới nhất kèm tên khách hàng.

**Bài 12.5.** Lấy các đơn hàng có `shippedDate IS NULL` kèm tên khách hàng.

---

## 13. `INNER JOIN` giữa `employees` và `offices`

Bảng `employees` có khóa ngoại `officeCode` tham chiếu đến bảng `offices`.

Mối liên hệ:

```text
employees.officeCode = offices.officeCode
```

Truy vấn:

```sql
SELECT 
    e.employeeNumber,
    e.lastName,
    e.firstName,
    e.jobTitle,
    o.city,
    o.country
FROM employees AS e
INNER JOIN offices AS o
    ON e.officeCode = o.officeCode;
```

Thêm điều kiện lấy nhân viên ở văn phòng `USA`:

```sql
SELECT 
    e.employeeNumber,
    e.lastName,
    e.firstName,
    e.jobTitle,
    o.city,
    o.country
FROM employees AS e
INNER JOIN offices AS o
    ON e.officeCode = o.officeCode
WHERE o.country = 'USA';
```

---

### Bài tập thực hành

**Bài 13.1.** Nối `employees` và `offices`, lấy tên nhân viên và thành phố văn phòng.

**Bài 13.2.** Lấy các nhân viên làm việc ở `USA`.

**Bài 13.3.** Lấy các nhân viên có `jobTitle = 'Sales Rep'` kèm quốc gia văn phòng.

**Bài 13.4.** Lấy các nhân viên làm việc ở văn phòng `Paris`.

**Bài 13.5.** Lấy danh sách nhân viên, thành phố, quốc gia và sắp xếp theo `country`, `city`.

---

## 14. `INNER JOIN` giữa `customers` và `employees`

Bảng `customers` có khóa ngoại `salesRepEmployeeNumber` tham chiếu đến bảng `employees`.

Mối liên hệ:

```text
customers.salesRepEmployeeNumber = employees.employeeNumber
```

Truy vấn:

```sql
SELECT 
    c.customerNumber,
    c.customerName,
    c.country,
    e.employeeNumber,
    e.lastName,
    e.firstName
FROM customers AS c
INNER JOIN employees AS e
    ON c.salesRepEmployeeNumber = e.employeeNumber;
```

Lưu ý: `INNER JOIN` chỉ trả về các khách hàng đã có nhân viên bán hàng phụ trách.

Nếu một khách hàng có `salesRepEmployeeNumber IS NULL`, khách hàng đó không xuất hiện trong kết quả `INNER JOIN`.

---

### Bài tập thực hành

**Bài 14.1.** Nối `customers` và `employees`, lấy tên khách hàng và tên nhân viên phụ trách.

**Bài 14.2.** Lấy các khách hàng ở `USA` kèm nhân viên phụ trách.

**Bài 14.3.** Lấy các khách hàng ở `France` kèm nhân viên phụ trách.

**Bài 14.4.** Lấy khách hàng có `creditLimit > 100000` kèm nhân viên phụ trách.

**Bài 14.5.** Sắp xếp kết quả theo họ nhân viên, sau đó theo tên khách hàng.

---

## 15. `INNER JOIN` giữa `orderdetails` và `products`

Bảng `orderdetails` có khóa ngoại `productCode` tham chiếu đến bảng `products`.

Mối liên hệ:

```text
orderdetails.productCode = products.productCode
```

Truy vấn:

```sql
SELECT 
    od.orderNumber,
    od.productCode,
    p.productName,
    od.quantityOrdered,
    od.priceEach
FROM orderdetails AS od
INNER JOIN products AS p
    ON od.productCode = p.productCode;
```

Có thể tạo cột tính toán cho thành tiền từng dòng:

```sql
SELECT 
    od.orderNumber,
    p.productName,
    od.quantityOrdered,
    od.priceEach,
    od.quantityOrdered * od.priceEach AS lineTotal
FROM orderdetails AS od
INNER JOIN products AS p
    ON od.productCode = p.productCode;
```

---

### Bài tập thực hành

**Bài 15.1.** Nối `orderdetails` và `products`, lấy `orderNumber`, `productName`, `quantityOrdered`, `priceEach`.

**Bài 15.2.** Tạo thêm cột `lineTotal = quantityOrdered * priceEach`.

**Bài 15.3.** Lấy các dòng chi tiết có `quantityOrdered >= 50`.

**Bài 15.4.** Lấy các dòng chi tiết của sản phẩm thuộc dòng `Classic Cars`.

**Bài 15.5.** Sắp xếp kết quả theo `orderNumber`, sau đó theo `orderLineNumber`.

---

## 16. Nối ba bảng: `orders`, `customers`, `orderdetails`

Khi cần thông tin nằm ở ba bảng, có thể viết nhiều `JOIN` liên tiếp.

Ví dụ lấy đơn hàng, khách hàng và các dòng chi tiết:

```sql
SELECT 
    o.orderNumber,
    o.orderDate,
    o.status,
    c.customerName,
    od.productCode,
    od.quantityOrdered,
    od.priceEach
FROM orders AS o
INNER JOIN customers AS c
    ON o.customerNumber = c.customerNumber
INNER JOIN orderdetails AS od
    ON o.orderNumber = od.orderNumber;
```

Ý nghĩa:

1. Nối `orders` với `customers` để biết đơn hàng thuộc khách hàng nào.
2. Nối tiếp với `orderdetails` để biết đơn hàng có những dòng sản phẩm nào.

---

### Bài tập thực hành

**Bài 16.1.** Nối `orders`, `customers`, `orderdetails`, lấy `orderNumber`, `customerName`, `productCode`, `quantityOrdered`.

**Bài 16.2.** Lấy các dòng chi tiết của các đơn hàng có `status = 'Shipped'`.

**Bài 16.3.** Lấy các dòng chi tiết của khách hàng ở `USA`.

**Bài 16.4.** Lấy 20 dòng chi tiết mới nhất theo `orderDate`.

**Bài 16.5.** Tạo thêm cột `lineTotal = quantityOrdered * priceEach`.

---

## 17. Nối bốn bảng: đơn hàng, khách hàng, chi tiết, sản phẩm

Để hiển thị tên sản phẩm thay vì chỉ có `productCode`, nối thêm bảng `products`.

```sql
SELECT 
    o.orderNumber,
    o.orderDate,
    c.customerName,
    p.productName,
    od.quantityOrdered,
    od.priceEach
FROM orders AS o
INNER JOIN customers AS c
    ON o.customerNumber = c.customerNumber
INNER JOIN orderdetails AS od
    ON o.orderNumber = od.orderNumber
INNER JOIN products AS p
    ON od.productCode = p.productCode;
```

Có thể thêm thành tiền dòng:

```sql
SELECT 
    o.orderNumber,
    o.orderDate,
    c.customerName,
    p.productName,
    od.quantityOrdered,
    od.priceEach,
    od.quantityOrdered * od.priceEach AS lineTotal
FROM orders AS o
INNER JOIN customers AS c
    ON o.customerNumber = c.customerNumber
INNER JOIN orderdetails AS od
    ON o.orderNumber = od.orderNumber
INNER JOIN products AS p
    ON od.productCode = p.productCode
ORDER BY o.orderDate DESC, o.orderNumber;
```

---

### Bài tập thực hành

**Bài 17.1.** Nối `orders`, `customers`, `orderdetails`, `products`, lấy tên khách hàng và tên sản phẩm.

**Bài 17.2.** Lấy các dòng đơn hàng của sản phẩm thuộc dòng `Motorcycles`.

**Bài 17.3.** Lấy các dòng đơn hàng của khách hàng ở `France`.

**Bài 17.4.** Lấy 20 dòng đơn hàng mới nhất kèm tên sản phẩm.

**Bài 17.5.** Tạo thêm cột `lineTotal` và sắp xếp theo `lineTotal` giảm dần.

---

## 18. Nối năm bảng: thêm `productlines`

Nếu muốn hiển thị cả mô tả dòng sản phẩm, nối thêm bảng `productlines`.

```sql
SELECT 
    o.orderNumber,
    o.orderDate,
    c.customerName,
    p.productName,
    p.productLine,
    pl.textDescription,
    od.quantityOrdered,
    od.priceEach
FROM orders AS o
INNER JOIN customers AS c
    ON o.customerNumber = c.customerNumber
INNER JOIN orderdetails AS od
    ON o.orderNumber = od.orderNumber
INNER JOIN products AS p
    ON od.productCode = p.productCode
INNER JOIN productlines AS pl
    ON p.productLine = pl.productLine;
```

Truy vấn này đi theo chuỗi:

```text
orders -> customers
orders -> orderdetails -> products -> productlines
```

---

### Bài tập thực hành

**Bài 18.1.** Viết truy vấn nối 5 bảng trên để lấy `orderNumber`, `customerName`, `productName`, `productLine`.

**Bài 18.2.** Lấy các dòng đơn hàng thuộc dòng sản phẩm `Classic Cars`.

**Bài 18.3.** Lấy các dòng đơn hàng thuộc dòng sản phẩm `Ships`.

**Bài 18.4.** Lấy các dòng đơn hàng của khách hàng ở `USA` và sản phẩm thuộc `Motorcycles`.

**Bài 18.5.** Lấy 30 dòng đầu tiên, sắp xếp theo `orderDate` giảm dần.

---

## 19. `LEFT JOIN`: giữ lại dữ liệu ở bảng bên trái

`LEFT JOIN` trả về toàn bộ dòng của bảng bên trái, kể cả khi không có dòng khớp ở bảng bên phải.

Cú pháp:

```sql
SELECT table1.column1, table2.column2
FROM table1
LEFT JOIN table2
    ON table1.key_column = table2.key_column;
```

Ví dụ lấy tất cả khách hàng và nhân viên bán hàng phụ trách nếu có:

```sql
SELECT 
    c.customerNumber,
    c.customerName,
    c.salesRepEmployeeNumber,
    e.employeeNumber,
    e.lastName,
    e.firstName
FROM customers AS c
LEFT JOIN employees AS e
    ON c.salesRepEmployeeNumber = e.employeeNumber;
```

Khách hàng chưa có nhân viên phụ trách vẫn xuất hiện, nhưng các cột từ bảng `employees` sẽ là `NULL`.

---

### Bài tập thực hành

**Bài 19.1.** Dùng `LEFT JOIN` lấy tất cả khách hàng và nhân viên phụ trách nếu có.

**Bài 19.2.** Dùng `LEFT JOIN` lấy các khách hàng chưa có nhân viên phụ trách.

**Bài 19.3.** Dùng `LEFT JOIN` lấy tất cả khách hàng và các khoản thanh toán nếu có.

**Bài 19.4.** Dùng `LEFT JOIN` lấy tất cả sản phẩm và dòng sản phẩm tương ứng.

**Bài 19.5.** Dùng `LEFT JOIN` lấy tất cả nhân viên và thông tin văn phòng tương ứng.

---

## 20. Tìm dòng không có dữ liệu liên quan bằng `LEFT JOIN`

Một ứng dụng phổ biến của `LEFT JOIN` là tìm các dòng chưa có bản ghi liên quan.

Ví dụ tìm khách hàng chưa có nhân viên bán hàng phụ trách:

```sql
SELECT 
    c.customerNumber,
    c.customerName,
    c.salesRepEmployeeNumber
FROM customers AS c
LEFT JOIN employees AS e
    ON c.salesRepEmployeeNumber = e.employeeNumber
WHERE e.employeeNumber IS NULL;
```

Ví dụ tìm khách hàng chưa từng có đơn hàng:

```sql
SELECT 
    c.customerNumber,
    c.customerName
FROM customers AS c
LEFT JOIN orders AS o
    ON c.customerNumber = o.customerNumber
WHERE o.orderNumber IS NULL;
```

Ví dụ tìm sản phẩm chưa từng xuất hiện trong chi tiết đơn hàng:

```sql
SELECT 
    p.productCode,
    p.productName
FROM products AS p
LEFT JOIN orderdetails AS od
    ON p.productCode = od.productCode
WHERE od.orderNumber IS NULL;
```

---

### Bài tập thực hành

**Bài 20.1.** Tìm các khách hàng chưa có nhân viên bán hàng phụ trách.

**Bài 20.2.** Tìm các khách hàng chưa từng có đơn hàng.

**Bài 20.3.** Tìm các sản phẩm chưa từng được đặt trong `orderdetails`.

**Bài 20.4.** Tìm các nhân viên không phụ trách khách hàng nào.

**Bài 20.5.** Tìm các dòng sản phẩm không có sản phẩm nào.

---

## 21. So sánh `INNER JOIN` và `LEFT JOIN`

| Tiêu chí | `INNER JOIN` | `LEFT JOIN` |
|---|---|---|
| Kết quả trả về | Chỉ dòng khớp ở cả hai bảng | Tất cả dòng bảng trái, kèm dòng khớp nếu có |
| Dòng không có liên hệ | Bị loại khỏi kết quả | Vẫn xuất hiện |
| Cột từ bảng phải nếu không khớp | Không xuất hiện dòng đó | Có giá trị `NULL` |
| Dùng khi nào | Chỉ cần dữ liệu đã có liên hệ | Muốn giữ toàn bộ dữ liệu của bảng chính |
| Ví dụ | Đơn hàng kèm khách hàng | Tất cả khách hàng, kể cả chưa có đơn hàng |

Ví dụ `INNER JOIN`:

```sql
SELECT c.customerNumber, c.customerName, o.orderNumber
FROM customers AS c
INNER JOIN orders AS o
    ON c.customerNumber = o.customerNumber;
```

Ví dụ `LEFT JOIN`:

```sql
SELECT c.customerNumber, c.customerName, o.orderNumber
FROM customers AS c
LEFT JOIN orders AS o
    ON c.customerNumber = o.customerNumber;
```

---

### Bài tập thực hành

**Bài 21.1.** So sánh kết quả của `INNER JOIN` và `LEFT JOIN` giữa `customers` và `orders`.

**Bài 21.2.** Khi cần giữ lại toàn bộ khách hàng, nên dùng loại `JOIN` nào?

**Bài 21.3.** Khi chỉ cần khách hàng đã có đơn hàng, nên dùng loại `JOIN` nào?

**Bài 21.4.** Viết `INNER JOIN` giữa `customers` và `orders`.

**Bài 21.5.** Viết `LEFT JOIN` giữa `customers` và `orders`.

---

## 22. `SELF JOIN`: nối một bảng với chính nó

`SELF JOIN` dùng khi một bảng có quan hệ với chính nó.

Trong `classicmodels`, bảng `employees` có cột:

```text
reportsTo
```

Cột này tham chiếu đến:

```text
employees.employeeNumber
```

Nghĩa là một nhân viên có thể báo cáo cho một nhân viên khác.

Ví dụ lấy nhân viên và quản lý trực tiếp:

```sql
SELECT 
    e.employeeNumber AS employeeId,
    e.lastName AS employeeLastName,
    e.firstName AS employeeFirstName,
    m.employeeNumber AS managerId,
    m.lastName AS managerLastName,
    m.firstName AS managerFirstName
FROM employees AS e
LEFT JOIN employees AS m
    ON e.reportsTo = m.employeeNumber;
```

Trong đó:

| Bí danh | Ý nghĩa |
|---|---|
| `e` | Nhân viên |
| `m` | Quản lý |

Dùng `LEFT JOIN` để vẫn giữ lại nhân viên không có quản lý trực tiếp, ví dụ Chủ tịch công ty.

---

### Bài tập thực hành

**Bài 22.1.** Viết truy vấn lấy danh sách nhân viên và quản lý trực tiếp.

**Bài 22.2.** Đổi bí danh cột thành `NhanVien`, `QuanLy`.

**Bài 22.3.** Lấy các nhân viên không có quản lý trực tiếp.

**Bài 22.4.** Lấy các nhân viên có quản lý trực tiếp.

**Bài 22.5.** Sắp xếp kết quả theo họ của quản lý, sau đó theo họ của nhân viên.

---

## 23. Kết hợp `JOIN` với `WHERE`, `ORDER BY`, `LIMIT`

Một truy vấn có `JOIN` thường có thứ tự:

```sql
SELECT ...
FROM table1
JOIN table2
    ON ...
WHERE ...
ORDER BY ...
LIMIT ...;
```

Ví dụ lấy 10 đơn hàng mới nhất kèm tên khách hàng:

```sql
SELECT 
    o.orderNumber,
    o.orderDate,
    o.status,
    c.customerName
FROM orders AS o
INNER JOIN customers AS c
    ON o.customerNumber = c.customerNumber
ORDER BY o.orderDate DESC
LIMIT 10;
```

Ví dụ lấy 10 dòng đơn hàng có giá trị dòng lớn nhất:

```sql
SELECT 
    o.orderNumber,
    c.customerName,
    p.productName,
    od.quantityOrdered,
    od.priceEach,
    od.quantityOrdered * od.priceEach AS lineTotal
FROM orders AS o
INNER JOIN customers AS c
    ON o.customerNumber = c.customerNumber
INNER JOIN orderdetails AS od
    ON o.orderNumber = od.orderNumber
INNER JOIN products AS p
    ON od.productCode = p.productCode
ORDER BY lineTotal DESC
LIMIT 10;
```

---

### Bài tập thực hành

**Bài 23.1.** Lấy 10 đơn hàng mới nhất kèm tên khách hàng.

**Bài 23.2.** Lấy 10 dòng đơn hàng có `lineTotal` lớn nhất.

**Bài 23.3.** Lấy các dòng đơn hàng thuộc `Classic Cars`, sắp xếp theo `lineTotal` giảm dần.

**Bài 23.4.** Lấy 20 dòng đơn hàng của khách hàng ở `USA`, sắp xếp theo `orderDate` giảm dần.

**Bài 23.5.** Lấy 20 dòng đơn hàng có trạng thái `Cancelled` hoặc `Disputed`.

---

## 24. Một số lỗi thường gặp khi viết `JOIN`

### Lỗi 1: Quên điều kiện `ON`

Sai:

```sql
SELECT o.orderNumber, c.customerName
FROM orders AS o
INNER JOIN customers AS c;
```

Đúng:

```sql
SELECT o.orderNumber, c.customerName
FROM orders AS o
INNER JOIN customers AS c
    ON o.customerNumber = c.customerNumber;
```

---

### Lỗi 2: Nối sai cột

Sai:

```sql
SELECT o.orderNumber, c.customerName
FROM orders AS o
INNER JOIN customers AS c
    ON o.orderNumber = c.customerNumber;
```

Đúng:

```sql
SELECT o.orderNumber, c.customerName
FROM orders AS o
INNER JOIN customers AS c
    ON o.customerNumber = c.customerNumber;
```

---

### Lỗi 3: Tên cột bị mơ hồ

Sai:

```sql
SELECT customerNumber, orderNumber
FROM customers AS c
INNER JOIN orders AS o
    ON c.customerNumber = o.customerNumber;
```

Trong truy vấn trên, `customerNumber` xuất hiện ở cả hai bảng nên có thể gây lỗi hoặc gây khó đọc.

Đúng:

```sql
SELECT c.customerNumber, o.orderNumber
FROM customers AS c
INNER JOIN orders AS o
    ON c.customerNumber = o.customerNumber;
```

---

### Lỗi 4: Dùng `WHERE` làm mất ý nghĩa của `LEFT JOIN`

Truy vấn sau có thể biến `LEFT JOIN` thành gần giống `INNER JOIN` nếu lọc cột của bảng phải trong `WHERE`:

```sql
SELECT c.customerNumber, c.customerName, o.orderNumber
FROM customers AS c
LEFT JOIN orders AS o
    ON c.customerNumber = o.customerNumber
WHERE o.status = 'Shipped';
```

Nếu muốn giữ toàn bộ khách hàng, cần cẩn thận khi đặt điều kiện lọc.

Một cách xử lý là đưa điều kiện vào phần `ON`:

```sql
SELECT c.customerNumber, c.customerName, o.orderNumber, o.status
FROM customers AS c
LEFT JOIN orders AS o
    ON c.customerNumber = o.customerNumber
   AND o.status = 'Shipped';
```

---

### Bài tập thực hành

**Bài 24.1.** Sửa lỗi truy vấn sau:

```sql
SELECT o.orderNumber, c.customerName
FROM orders AS o
INNER JOIN customers AS c;
```

**Bài 24.2.** Sửa lỗi truy vấn sau:

```sql
SELECT o.orderNumber, c.customerName
FROM orders AS o
INNER JOIN customers AS c
    ON o.orderNumber = c.customerNumber;
```

**Bài 24.3.** Sửa lỗi truy vấn sau:

```sql
SELECT customerNumber, orderNumber
FROM customers AS c
INNER JOIN orders AS o
    ON c.customerNumber = o.customerNumber;
```

**Bài 24.4.** Giải thích vì sao truy vấn `LEFT JOIN` có điều kiện `WHERE o.status = 'Shipped'` có thể loại bỏ khách hàng chưa có đơn hàng.

**Bài 24.5.** Viết lại truy vấn `LEFT JOIN` sao cho vẫn giữ toàn bộ khách hàng, nhưng chỉ hiển thị đơn hàng có trạng thái `Shipped` nếu có.

---

## 25. Bài tập tổng hợp

**Bài 25.1.** Viết truy vấn lấy danh sách đơn hàng gồm:

```text
orderNumber, orderDate, status, customerName, country
```

**Bài 25.2.** Viết truy vấn lấy danh sách chi tiết đơn hàng gồm:

```text
orderNumber, productName, quantityOrdered, priceEach, lineTotal
```

**Bài 25.3.** Viết truy vấn lấy danh sách khách hàng và nhân viên phụ trách gồm:

```text
customerName, country, employeeNumber, employeeName
```

Trong đó `employeeName` có thể ghép từ `firstName` và `lastName`.

**Bài 25.4.** Viết truy vấn lấy danh sách nhân viên và văn phòng làm việc gồm:

```text
employeeNumber, employeeName, jobTitle, city, country
```

**Bài 25.5.** Viết truy vấn lấy 20 dòng đơn hàng mới nhất gồm:

```text
orderDate, orderNumber, customerName, productName, quantityOrdered, priceEach
```

**Bài 25.6.** Viết truy vấn lấy các khách hàng chưa có nhân viên bán hàng phụ trách.

**Bài 25.7.** Viết truy vấn lấy các khách hàng chưa từng có đơn hàng.

**Bài 25.8.** Viết truy vấn lấy các nhân viên và quản lý trực tiếp của họ.

**Bài 25.9.** Viết truy vấn lấy các dòng đơn hàng thuộc sản phẩm `Classic Cars`.

**Bài 25.10.** Viết truy vấn lấy 10 dòng đơn hàng có `lineTotal` lớn nhất.

---

## 26. Đáp án gợi ý cho một số bài tập

### Bài 5.1

```sql
SELECT customerNumber, customerName, country
FROM customers;
```

### Bài 6.1

```sql
SELECT customerNumber, customerName, city, country
FROM customers
WHERE country = 'France';
```

### Bài 7.1

```sql
SELECT productCode, productName, MSRP
FROM products
ORDER BY MSRP DESC
LIMIT 5;
```

### Bài 9.1

```sql
SELECT 
    o.orderNumber,
    o.orderDate,
    o.status,
    c.customerName
FROM orders AS o
INNER JOIN customers AS c
    ON o.customerNumber = c.customerNumber;
```

### Bài 10.2

```sql
SELECT 
    p.productCode,
    p.productName,
    p.productLine,
    pl.textDescription
FROM products AS p
INNER JOIN productlines AS pl
    ON p.productLine = pl.productLine;
```

### Bài 12.2

```sql
SELECT 
    o.orderNumber,
    o.orderDate,
    o.status,
    c.customerName
FROM orders AS o
INNER JOIN customers AS c
    ON o.customerNumber = c.customerNumber
WHERE o.status = 'Cancelled';
```

### Bài 13.1

```sql
SELECT 
    e.employeeNumber,
    e.lastName,
    e.firstName,
    o.city,
    o.country
FROM employees AS e
INNER JOIN offices AS o
    ON e.officeCode = o.officeCode;
```

### Bài 14.1

```sql
SELECT 
    c.customerNumber,
    c.customerName,
    e.employeeNumber,
    e.firstName,
    e.lastName
FROM customers AS c
INNER JOIN employees AS e
    ON c.salesRepEmployeeNumber = e.employeeNumber;
```

### Bài 15.2

```sql
SELECT 
    od.orderNumber,
    p.productName,
    od.quantityOrdered,
    od.priceEach,
    od.quantityOrdered * od.priceEach AS lineTotal
FROM orderdetails AS od
INNER JOIN products AS p
    ON od.productCode = p.productCode;
```

### Bài 17.1

```sql
SELECT 
    o.orderNumber,
    o.orderDate,
    c.customerName,
    p.productName,
    od.quantityOrdered,
    od.priceEach
FROM orders AS o
INNER JOIN customers AS c
    ON o.customerNumber = c.customerNumber
INNER JOIN orderdetails AS od
    ON o.orderNumber = od.orderNumber
INNER JOIN products AS p
    ON od.productCode = p.productCode;
```

### Bài 19.2

```sql
SELECT 
    c.customerNumber,
    c.customerName,
    c.salesRepEmployeeNumber
FROM customers AS c
LEFT JOIN employees AS e
    ON c.salesRepEmployeeNumber = e.employeeNumber
WHERE e.employeeNumber IS NULL;
```

### Bài 20.2

```sql
SELECT 
    c.customerNumber,
    c.customerName
FROM customers AS c
LEFT JOIN orders AS o
    ON c.customerNumber = o.customerNumber
WHERE o.orderNumber IS NULL;
```

### Bài 22.1

```sql
SELECT 
    e.employeeNumber AS employeeId,
    e.firstName AS employeeFirstName,
    e.lastName AS employeeLastName,
    m.employeeNumber AS managerId,
    m.firstName AS managerFirstName,
    m.lastName AS managerLastName
FROM employees AS e
LEFT JOIN employees AS m
    ON e.reportsTo = m.employeeNumber;
```

### Bài 23.2

```sql
SELECT 
    o.orderNumber,
    c.customerName,
    p.productName,
    od.quantityOrdered,
    od.priceEach,
    od.quantityOrdered * od.priceEach AS lineTotal
FROM orders AS o
INNER JOIN customers AS c
    ON o.customerNumber = c.customerNumber
INNER JOIN orderdetails AS od
    ON o.orderNumber = od.orderNumber
INNER JOIN products AS p
    ON od.productCode = p.productCode
ORDER BY lineTotal DESC
LIMIT 10;
```

---

## 27. Tóm tắt

Các kiến thức chính trong tutorial:

- `SELECT` dùng để truy vấn dữ liệu.
- `FROM` chỉ định bảng chính cần đọc.
- `WHERE` dùng để lọc dữ liệu.
- `ORDER BY` dùng để sắp xếp kết quả.
- `LIMIT` dùng để giới hạn số dòng kết quả.
- `JOIN` dùng để kết hợp dữ liệu từ nhiều bảng.
- `INNER JOIN` chỉ giữ các dòng có dữ liệu khớp ở hai bảng.
- `LEFT JOIN` giữ toàn bộ dòng của bảng bên trái.
- `ON` là nơi đặt điều kiện nối giữa hai bảng.
- Khi viết `JOIN`, nên dùng bí danh bảng để câu lệnh ngắn và dễ đọc.
- Cần nối đúng cặp khóa chính và khóa ngoại.
- `SELF JOIN` dùng khi một bảng có quan hệ với chính nó.
- Có thể nối nhiều bảng bằng cách viết nhiều mệnh đề `JOIN` liên tiếp.

---

## 28. Từ khóa chính

- SQL
- SELECT
- FROM
- WHERE
- ORDER BY
- LIMIT
- JOIN
- INNER JOIN
- LEFT JOIN
- SELF JOIN
- ON
- Alias
- Primary Key
- Foreign Key
- customers
- orders
- orderdetails
- products
- productlines
- employees
- offices
- classicmodels
