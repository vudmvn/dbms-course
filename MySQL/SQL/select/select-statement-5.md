---
title: "Lab: Subquery trong MySQL với classicmodels"
author: "Tên giảng viên"
duration: "90m"
difficulty: "Intermediate"
prerequisites:
    - "Đã biết SELECT, WHERE, ORDER BY, LIMIT"
    - "Đã biết JOIN cơ bản"
    - "Đã biết hàm tổng hợp cơ bản"
summary: "Thực hành truy vấn con trong WHERE, SELECT, FROM, subquery tương quan, IN, NOT IN, EXISTS và NOT EXISTS trên cơ sở dữ liệu mẫu classicmodels."
---

# Lab: Subquery trong MySQL với `classicmodels`

## Link tham khảo

- [MySQL Subquery](https://www.mysqltutorial.org/mysql-basics/mysql-subquery/)
- [MySQL Derived Table](https://www.mysqltutorial.org/mysql-basics/mysql-derived-table/)

## 1. Mục tiêu học tập

Sau khi hoàn thành lab này, người học có thể:

1. Giải thích được subquery là gì.
2. Viết subquery trong mệnh đề `WHERE`.
3. Sử dụng subquery trả về một giá trị.
4. Sử dụng subquery trả về nhiều dòng với `IN`.
5. Sử dụng `NOT IN` đúng cách khi dữ liệu có thể chứa `NULL`.
6. Viết subquery trong phần `SELECT`.
7. Viết subquery tương quan.
8. Sử dụng `EXISTS` và `NOT EXISTS`.
9. Viết subquery trong mệnh đề `FROM`, còn gọi là derived table.
10. Nhận biết các lỗi thường gặp khi viết subquery.

---

## 2. Giới thiệu cơ sở dữ liệu mẫu `classicmodels`

`classicmodels` là cơ sở dữ liệu mẫu mô phỏng hoạt động của một công ty bán các mô hình xe, tàu, máy bay và các sản phẩm sưu tầm.

Một số bảng chính:

| Bảng | Ý nghĩa |
|---|---|
| `productlines` | Nhóm hoặc dòng sản phẩm |
| `products` | Thông tin sản phẩm |
| `offices` | Văn phòng |
| `employees` | Nhân viên |
| `customers` | Khách hàng |
| `payments` | Thanh toán |
| `orders` | Đơn hàng |
| `orderdetails` | Chi tiết đơn hàng |

Các quan hệ thường dùng:

| Quan hệ | Ý nghĩa |
|---|---|
| `orders.customerNumber -> customers.customerNumber` | Mỗi đơn hàng thuộc về một khách hàng |
| `payments.customerNumber -> customers.customerNumber` | Mỗi khoản thanh toán thuộc về một khách hàng |
| `orderdetails.orderNumber -> orders.orderNumber` | Mỗi dòng chi tiết thuộc về một đơn hàng |
| `orderdetails.productCode -> products.productCode` | Mỗi dòng chi tiết ứng với một sản phẩm |
| `customers.salesRepEmployeeNumber -> employees.employeeNumber` | Khách hàng có thể được phụ trách bởi nhân viên bán hàng |
| `products.productLine -> productlines.productLine` | Mỗi sản phẩm thuộc một dòng sản phẩm |

---

## 3. Chuẩn bị môi trường

Chọn cơ sở dữ liệu:

```sql
USE classicmodels;
```

Xem danh sách bảng:

```sql
SHOW TABLES;
```

Xem cấu trúc một số bảng:

```sql
DESC products;
DESC customers;
DESC orders;
DESC orderdetails;
DESC payments;
DESC employees;
```

### Bài tập thực hành

**Bài 3.1.** Viết lệnh chọn cơ sở dữ liệu `classicmodels`.

**Bài 3.2.** Viết lệnh xem cấu trúc bảng `products`.

**Bài 3.3.** Viết lệnh xem cấu trúc bảng `customers`.

**Bài 3.4.** Viết lệnh xem cấu trúc bảng `orders`.

**Bài 3.5.** Viết lệnh xem cấu trúc bảng `orderdetails`.

---

## 4. Subquery là gì?

**Subquery** hay **truy vấn con** là một câu lệnh `SELECT` được đặt bên trong một truy vấn khác.

Ví dụ:

```sql
SELECT productCode, productName, buyPrice
FROM products
WHERE buyPrice > (
    SELECT AVG(buyPrice)
    FROM products
);
```

Ý nghĩa:

1. Truy vấn con tính giá mua trung bình của toàn bộ sản phẩm.
2. Truy vấn ngoài lấy các sản phẩm có `buyPrice` lớn hơn giá mua trung bình đó.

Subquery có thể xuất hiện ở nhiều vị trí:

| Vị trí | Vai trò |
|---|---|
| `WHERE` | Lọc dữ liệu dựa trên kết quả truy vấn khác |
| `SELECT` | Tạo thêm cột tính toán từ truy vấn con |
| `FROM` | Tạo bảng tạm trong truy vấn |
| `HAVING` | Lọc nhóm dựa trên kết quả truy vấn khác |

---

## 5. Subquery trả về một giá trị

Subquery trả về một giá trị thường dùng với:

```sql
=, <>, >, <, >=, <=
```

### Ví dụ 1: Sản phẩm có giá mua lớn hơn giá mua trung bình

```sql
SELECT productCode, productName, buyPrice
FROM products
WHERE buyPrice > (
    SELECT AVG(buyPrice)
    FROM products
);
```

### Ví dụ 2: Khoản thanh toán lớn hơn khoản thanh toán trung bình

```sql
SELECT customerNumber, checkNumber, paymentDate, amount
FROM payments
WHERE amount > (
    SELECT AVG(amount)
    FROM payments
);
```

### Ví dụ 3: Sản phẩm có giá bán đề xuất cao nhất

```sql
SELECT productCode, productName, MSRP
FROM products
WHERE MSRP = (
    SELECT MAX(MSRP)
    FROM products
);
```

### Bài tập thực hành

**Bài 5.1.** Lấy các sản phẩm có `MSRP` lớn hơn `MSRP` trung bình.

**Bài 5.2.** Lấy các sản phẩm có `buyPrice` nhỏ hơn `buyPrice` trung bình.

**Bài 5.3.** Lấy các khoản thanh toán có `amount` lớn hơn `amount` trung bình.

**Bài 5.4.** Lấy sản phẩm có `quantityInStock` lớn nhất.

**Bài 5.5.** Lấy khách hàng có `creditLimit` bằng hạn mức tín dụng lớn nhất.

---

## 6. Subquery trả về nhiều dòng với `IN`

Khi subquery trả về nhiều dòng, ta thường dùng `IN`.

### Ví dụ 1: Khách hàng đã từng đặt hàng

```sql
SELECT customerNumber, customerName
FROM customers
WHERE customerNumber IN (
    SELECT customerNumber
    FROM orders
);
```

### Ví dụ 2: Sản phẩm đã từng xuất hiện trong đơn hàng

```sql
SELECT productCode, productName
FROM products
WHERE productCode IN (
    SELECT productCode
    FROM orderdetails
);
```

### Ví dụ 3: Nhân viên đang phụ trách ít nhất một khách hàng

```sql
SELECT employeeNumber, lastName, firstName, jobTitle
FROM employees
WHERE employeeNumber IN (
    SELECT salesRepEmployeeNumber
    FROM customers
    WHERE salesRepEmployeeNumber IS NOT NULL
);
```

### Bài tập thực hành

**Bài 6.1.** Lấy danh sách khách hàng đã từng có đơn hàng.

**Bài 6.2.** Lấy danh sách khách hàng đã từng thanh toán.

**Bài 6.3.** Lấy danh sách sản phẩm đã từng được đặt trong `orderdetails`.

**Bài 6.4.** Lấy danh sách nhân viên đang phụ trách ít nhất một khách hàng.

**Bài 6.5.** Lấy danh sách dòng sản phẩm có ít nhất một sản phẩm trong bảng `products`.

---

## 7. Subquery với `NOT IN`

`NOT IN` dùng để lấy các dòng không thuộc tập kết quả của subquery.

### Ví dụ 1: Khách hàng chưa từng đặt hàng

```sql
SELECT customerNumber, customerName
FROM customers
WHERE customerNumber NOT IN (
    SELECT customerNumber
    FROM orders
);
```

### Ví dụ 2: Sản phẩm chưa từng được đặt

```sql
SELECT productCode, productName
FROM products
WHERE productCode NOT IN (
    SELECT productCode
    FROM orderdetails
);
```

### Lưu ý về `NULL`

Nếu subquery có thể trả về `NULL`, `NOT IN` có thể cho kết quả không như mong muốn.

Nên viết:

```sql
SELECT employeeNumber, lastName, firstName
FROM employees
WHERE employeeNumber NOT IN (
    SELECT salesRepEmployeeNumber
    FROM customers
    WHERE salesRepEmployeeNumber IS NOT NULL
);
```

### Bài tập thực hành

**Bài 7.1.** Lấy danh sách khách hàng chưa từng có đơn hàng.

**Bài 7.2.** Lấy danh sách khách hàng chưa từng thanh toán.

**Bài 7.3.** Lấy danh sách sản phẩm chưa từng được đặt.

**Bài 7.4.** Lấy danh sách nhân viên chưa phụ trách khách hàng nào.

**Bài 7.5.** Lấy danh sách dòng sản phẩm chưa có sản phẩm nào.

---

## 8. Subquery trong phần `SELECT`

Subquery trong `SELECT` thường dùng để tạo thêm cột kết quả.

### Ví dụ 1: Mỗi khách hàng và số lượng đơn hàng

```sql
SELECT 
    c.customerNumber,
    c.customerName,
    (
        SELECT COUNT(*)
        FROM orders o
        WHERE o.customerNumber = c.customerNumber
    ) AS orderCount
FROM customers c;
```

### Ví dụ 2: Mỗi khách hàng và tổng tiền đã thanh toán

```sql
SELECT
    c.customerNumber,
    c.customerName,
    (
        SELECT SUM(p.amount)
        FROM payments p
        WHERE p.customerNumber = c.customerNumber
    ) AS totalPayment
FROM customers c;
```

### Ví dụ 3: Mỗi sản phẩm và số lần xuất hiện trong chi tiết đơn hàng

```sql
SELECT
    p.productCode,
    p.productName,
    (
        SELECT COUNT(*)
        FROM orderdetails od
        WHERE od.productCode = p.productCode
    ) AS orderLineCount
FROM products p;
```

### Bài tập thực hành

**Bài 8.1.** Hiển thị mỗi khách hàng và số lượng đơn hàng của khách hàng đó.

**Bài 8.2.** Hiển thị mỗi khách hàng và tổng số tiền đã thanh toán.

**Bài 8.3.** Hiển thị mỗi sản phẩm và số dòng đơn hàng có chứa sản phẩm đó.

**Bài 8.4.** Hiển thị mỗi nhân viên và số khách hàng mà nhân viên đó phụ trách.

**Bài 8.5.** Hiển thị mỗi dòng sản phẩm và số sản phẩm thuộc dòng sản phẩm đó.

---

## 9. Subquery tương quan

**Subquery tương quan** là subquery có tham chiếu đến bảng của truy vấn bên ngoài.

### Ví dụ 1: Khách hàng có hơn 5 đơn hàng

```sql
SELECT 
    c.customerNumber,
    c.customerName
FROM customers c
WHERE (
    SELECT COUNT(*)
    FROM orders o
    WHERE o.customerNumber = c.customerNumber
) > 5;
```

### Ví dụ 2: Sản phẩm có tồn kho lớn hơn trung bình của chính dòng sản phẩm đó

```sql
SELECT 
    p.productCode,
    p.productName,
    p.productLine,
    p.quantityInStock
FROM products p
WHERE p.quantityInStock > (
    SELECT AVG(p2.quantityInStock)
    FROM products p2
    WHERE p2.productLine = p.productLine
);
```

### Bài tập thực hành

**Bài 9.1.** Lấy các khách hàng có nhiều hơn 3 đơn hàng.

**Bài 9.2.** Lấy các khách hàng có tổng tiền thanh toán lớn hơn 100000.

**Bài 9.3.** Lấy các sản phẩm có `buyPrice` lớn hơn `buyPrice` trung bình của chính dòng sản phẩm đó.

**Bài 9.4.** Lấy các sản phẩm có `MSRP` lớn hơn `MSRP` trung bình của chính dòng sản phẩm đó.

**Bài 9.5.** Lấy các nhân viên đang phụ trách nhiều hơn 5 khách hàng.

---

## 10. `EXISTS`

`EXISTS` kiểm tra subquery có trả về ít nhất một dòng hay không.

### Ví dụ 1: Khách hàng có ít nhất một đơn hàng

```sql
SELECT customerNumber, customerName
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customerNumber = c.customerNumber
);
```

### Ví dụ 2: Sản phẩm đã từng được đặt

```sql
SELECT productCode, productName
FROM products p
WHERE EXISTS (
    SELECT 1
    FROM orderdetails od
    WHERE od.productCode = p.productCode
);
```

### Bài tập thực hành

**Bài 10.1.** Dùng `EXISTS` để lấy khách hàng có ít nhất một đơn hàng.

**Bài 10.2.** Dùng `EXISTS` để lấy khách hàng có ít nhất một khoản thanh toán.

**Bài 10.3.** Dùng `EXISTS` để lấy sản phẩm đã từng được đặt.

**Bài 10.4.** Dùng `EXISTS` để lấy nhân viên có ít nhất một khách hàng được phân công.

**Bài 10.5.** Dùng `EXISTS` để lấy dòng sản phẩm có ít nhất một sản phẩm.

---

## 11. `NOT EXISTS`

`NOT EXISTS` kiểm tra subquery không trả về dòng nào.

### Ví dụ 1: Khách hàng chưa có đơn hàng

```sql
SELECT customerNumber, customerName
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customerNumber = c.customerNumber
);
```

### Ví dụ 2: Sản phẩm chưa từng được đặt

```sql
SELECT productCode, productName
FROM products p
WHERE NOT EXISTS (
    SELECT 1
    FROM orderdetails od
    WHERE od.productCode = p.productCode
);
```

`NOT EXISTS` thường an toàn hơn `NOT IN` khi dữ liệu có thể chứa `NULL`.

### Bài tập thực hành

**Bài 11.1.** Dùng `NOT EXISTS` để lấy khách hàng chưa có đơn hàng.

**Bài 11.2.** Dùng `NOT EXISTS` để lấy khách hàng chưa có khoản thanh toán.

**Bài 11.3.** Dùng `NOT EXISTS` để lấy sản phẩm chưa từng được đặt.

**Bài 11.4.** Dùng `NOT EXISTS` để lấy nhân viên chưa phụ trách khách hàng nào.

**Bài 11.5.** Dùng `NOT EXISTS` để lấy dòng sản phẩm chưa có sản phẩm.

---

## 12. Subquery trong mệnh đề `FROM`: Derived table

Subquery trong `FROM` tạo ra một bảng tạm thời chỉ tồn tại trong truy vấn.

### Ví dụ 1: Tổng thanh toán theo khách hàng

```sql
SELECT 
    customerNumber,
    totalPayment
FROM (
    SELECT 
        customerNumber,
        SUM(amount) AS totalPayment
    FROM payments
    GROUP BY customerNumber
) AS payment_summary
WHERE totalPayment > 100000;
```

### Ví dụ 2: Doanh thu từng đơn hàng

```sql
SELECT
    orderNumber,
    orderTotal
FROM (
    SELECT
        orderNumber,
        SUM(quantityOrdered * priceEach) AS orderTotal
    FROM orderdetails
    GROUP BY orderNumber
) AS order_summary
WHERE orderTotal > 50000;
```

### Bài tập thực hành

**Bài 12.1.** Tạo derived table tính tổng thanh toán theo khách hàng, sau đó lấy khách hàng có tổng thanh toán lớn hơn 100000.

**Bài 12.2.** Tạo derived table tính doanh thu từng đơn hàng, sau đó lấy đơn hàng có doanh thu lớn hơn 50000.

**Bài 12.3.** Tạo derived table đếm số đơn hàng theo khách hàng, sau đó lấy khách hàng có nhiều hơn 5 đơn hàng.

**Bài 12.4.** Tạo derived table tính số sản phẩm theo từng dòng sản phẩm.

**Bài 12.5.** Tạo derived table tính số nhân viên theo từng văn phòng.

---

## 13. Một số lỗi thường gặp

### Lỗi 1: Subquery trả về nhiều dòng nhưng dùng toán tử `=`

Sai:

```sql
SELECT customerNumber, customerName
FROM customers
WHERE customerNumber = (
    SELECT customerNumber
    FROM orders
);
```

Đúng:

```sql
SELECT customerNumber, customerName
FROM customers
WHERE customerNumber IN (
    SELECT customerNumber
    FROM orders
);
```

---

### Lỗi 2: Quên bí danh cho derived table

Sai:

```sql
SELECT *
FROM (
    SELECT customerNumber, SUM(amount) AS totalPayment
    FROM payments
    GROUP BY customerNumber
);
```

Đúng:

```sql
SELECT *
FROM (
    SELECT customerNumber, SUM(amount) AS totalPayment
    FROM payments
    GROUP BY customerNumber
) AS payment_summary;
```

---

### Lỗi 3: Dùng `NOT IN` với subquery có thể chứa `NULL`

Nên lọc `NULL`:

```sql
SELECT employeeNumber, lastName, firstName
FROM employees
WHERE employeeNumber NOT IN (
    SELECT salesRepEmployeeNumber
    FROM customers
    WHERE salesRepEmployeeNumber IS NOT NULL
);
```

Hoặc dùng `NOT EXISTS`.

---

## 14. Bài tập tổng hợp

**Bài 14.1.** Lấy danh sách sản phẩm có `MSRP` lớn hơn `MSRP` trung bình của toàn bộ sản phẩm.

**Bài 14.2.** Lấy danh sách khách hàng đã từng có ít nhất một đơn hàng bằng `EXISTS`.

**Bài 14.3.** Lấy danh sách khách hàng chưa từng thanh toán bằng `NOT EXISTS`.

**Bài 14.4.** Dùng derived table để tính tổng tiền thanh toán theo khách hàng, sau đó chỉ lấy khách hàng có tổng tiền lớn hơn 100000.

**Bài 14.5.** Lấy các sản phẩm chưa từng xuất hiện trong `orderdetails` bằng `NOT EXISTS`.

---

## 15. Đáp án gợi ý

### Bài 5.1

```sql
SELECT productCode, productName, MSRP
FROM products
WHERE MSRP > (
    SELECT AVG(MSRP)
    FROM products
);
```

### Bài 6.1

```sql
SELECT customerNumber, customerName
FROM customers
WHERE customerNumber IN (
    SELECT customerNumber
    FROM orders
);
```

### Bài 7.4

```sql
SELECT employeeNumber, lastName, firstName
FROM employees
WHERE employeeNumber NOT IN (
    SELECT salesRepEmployeeNumber
    FROM customers
    WHERE salesRepEmployeeNumber IS NOT NULL
);
```

### Bài 8.1

```sql
SELECT 
    c.customerNumber,
    c.customerName,
    (
        SELECT COUNT(*)
        FROM orders o
        WHERE o.customerNumber = c.customerNumber
    ) AS orderCount
FROM customers c;
```

### Bài 9.3

```sql
SELECT 
    p.productCode,
    p.productName,
    p.productLine,
    p.buyPrice
FROM products p
WHERE p.buyPrice > (
    SELECT AVG(p2.buyPrice)
    FROM products p2
    WHERE p2.productLine = p.productLine
);
```

### Bài 10.1

```sql
SELECT customerNumber, customerName
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customerNumber = c.customerNumber
);
```

### Bài 11.3

```sql
SELECT productCode, productName
FROM products p
WHERE NOT EXISTS (
    SELECT 1
    FROM orderdetails od
    WHERE od.productCode = p.productCode
);
```

### Bài 12.1

```sql
SELECT 
    customerNumber,
    totalPayment
FROM (
    SELECT 
        customerNumber,
        SUM(amount) AS totalPayment
    FROM payments
    GROUP BY customerNumber
) AS payment_summary
WHERE totalPayment > 100000;
```

---

## 16. Tóm tắt

- Subquery là truy vấn nằm trong truy vấn khác.
- Subquery trả về một giá trị có thể dùng với `=`, `>`, `<`, `>=`, `<=`.
- Subquery trả về nhiều dòng thường dùng với `IN`.
- `NOT IN` cần cẩn thận nếu subquery có thể trả về `NULL`.
- Subquery tương quan tham chiếu tới bảng của truy vấn bên ngoài.
- `EXISTS` kiểm tra có tồn tại dòng thỏa mãn điều kiện hay không.
- `NOT EXISTS` kiểm tra không tồn tại dòng thỏa mãn điều kiện.
- Derived table là subquery trong mệnh đề `FROM`.

---

## 17. Từ khóa chính

- SQL
- MySQL
- Subquery
- Nested query
- Correlated subquery
- Scalar subquery
- `IN`
- `NOT IN`
- `EXISTS`
- `NOT EXISTS`
- Derived table
- `classicmodels`
