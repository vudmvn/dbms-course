---
title: "Lab: CTE và WITH trong MySQL với classicmodels"
author: "Tên giảng viên"
duration: "90m"
difficulty: "Intermediate"
prerequisites:
    - "Đã biết SELECT, WHERE, ORDER BY, LIMIT"
    - "Đã biết JOIN cơ bản"
    - "Đã biết hàm tổng hợp, GROUP BY, HAVING"
summary: "Thực hành Common Table Expression bằng WITH, nhiều CTE và WITH RECURSIVE trên cơ sở dữ liệu mẫu classicmodels."
---

# Lab: CTE và `WITH` trong MySQL với `classicmodels`

## 1. Mục tiêu học tập

Sau khi hoàn thành lab này, người học có thể:

1. Giải thích được CTE là gì.
2. Sử dụng `WITH` để tạo CTE đơn giản.
3. Viết CTE để làm rõ truy vấn dài.
4. Dùng CTE kết hợp với `WHERE`, `ORDER BY`, `LIMIT`.
5. Dùng CTE để tổng hợp dữ liệu.
6. Dùng CTE kết hợp với `JOIN`.
7. Khai báo nhiều CTE trong cùng một truy vấn.
8. Sử dụng `WITH RECURSIVE` cho dữ liệu phân cấp.
9. Nhận biết sự khác nhau giữa CTE thường và CTE đệ quy.
10. Tránh các lỗi thường gặp khi viết CTE.

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

Các quan hệ thường dùng trong lab:

| Quan hệ | Ý nghĩa |
|---|---|
| `orders.customerNumber -> customers.customerNumber` | Mỗi đơn hàng thuộc về một khách hàng |
| `payments.customerNumber -> customers.customerNumber` | Mỗi khoản thanh toán thuộc về một khách hàng |
| `orderdetails.orderNumber -> orders.orderNumber` | Mỗi dòng chi tiết thuộc về một đơn hàng |
| `orderdetails.productCode -> products.productCode` | Mỗi dòng chi tiết ứng với một sản phẩm |
| `customers.salesRepEmployeeNumber -> employees.employeeNumber` | Khách hàng có thể được phụ trách bởi nhân viên bán hàng |
| `employees.reportsTo -> employees.employeeNumber` | Nhân viên có thể báo cáo cho một quản lý |

---

## 3. Chuẩn bị môi trường

Chọn cơ sở dữ liệu:

```sql
USE classicmodels;
```

Xem cấu trúc các bảng thường dùng:

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

**Bài 3.2.** Viết lệnh xem cấu trúc bảng `payments`.

**Bài 3.3.** Viết lệnh xem cấu trúc bảng `orders`.

**Bài 3.4.** Viết lệnh xem cấu trúc bảng `employees`.

**Bài 3.5.** Xác định cột tự tham chiếu trong bảng `employees`.

---

## 4. CTE là gì?

**CTE** là viết tắt của **Common Table Expression**.

Trong MySQL, CTE được viết bằng mệnh đề `WITH`. CTE giúp đặt tên cho một kết quả trung gian để truy vấn dễ đọc hơn.

Ví dụ:

```sql
WITH usa_customers AS (
    SELECT customerNumber, customerName, city, country, creditLimit
    FROM customers
    WHERE country = 'USA'
)
SELECT *
FROM usa_customers;
```

Ý nghĩa:

1. `usa_customers` là kết quả trung gian.
2. Truy vấn cuối cùng đọc dữ liệu từ `usa_customers`.
3. CTE chỉ tồn tại trong phạm vi truy vấn đó.

Cú pháp tổng quát:

```sql
WITH cte_name AS (
    SELECT ...
)
SELECT ...
FROM cte_name;
```

---

## 5. CTE đơn giản với `WITH`

### Ví dụ 1: Sản phẩm thuộc dòng `Classic Cars`

```sql
WITH classic_cars AS (
    SELECT productCode, productName, productLine, MSRP
    FROM products
    WHERE productLine = 'Classic Cars'
)
SELECT *
FROM classic_cars
ORDER BY MSRP DESC;
```

### Ví dụ 2: Khách hàng ở `USA`

```sql
WITH usa_customers AS (
    SELECT customerNumber, customerName, city, country, creditLimit
    FROM customers
    WHERE country = 'USA'
)
SELECT *
FROM usa_customers
WHERE creditLimit > 100000;
```

### Ví dụ 3: Đơn hàng đã giao

```sql
WITH shipped_orders AS (
    SELECT orderNumber, orderDate, shippedDate, status, customerNumber
    FROM orders
    WHERE status = 'Shipped'
)
SELECT *
FROM shipped_orders
WHERE orderDate >= '2005-01-01';
```

### Bài tập thực hành

**Bài 5.1.** Tạo CTE `classic_cars` chứa sản phẩm thuộc dòng `Classic Cars`.

**Bài 5.2.** Từ CTE `classic_cars`, lấy sản phẩm có `MSRP > 100`.

**Bài 5.3.** Tạo CTE `usa_customers` chứa khách hàng ở `USA`.

**Bài 5.4.** Từ CTE `usa_customers`, lấy khách hàng có `creditLimit > 100000`.

**Bài 5.5.** Tạo CTE `shipped_orders` chứa đơn hàng có trạng thái `Shipped`.

---

## 6. CTE kết hợp `ORDER BY` và `LIMIT`

CTE có thể được dùng như một bảng tạm để tiếp tục sắp xếp và giới hạn dữ liệu.

### Ví dụ 1: Top 10 sản phẩm giá cao trong dòng `Classic Cars`

```sql
WITH classic_cars AS (
    SELECT productCode, productName, productLine, MSRP
    FROM products
    WHERE productLine = 'Classic Cars'
)
SELECT *
FROM classic_cars
ORDER BY MSRP DESC
LIMIT 10;
```

### Ví dụ 2: Top 5 khoản thanh toán lớn

```sql
WITH payment_list AS (
    SELECT customerNumber, checkNumber, paymentDate, amount
    FROM payments
)
SELECT *
FROM payment_list
ORDER BY amount DESC
LIMIT 5;
```

### Bài tập thực hành

**Bài 6.1.** Tạo CTE chứa các sản phẩm thuộc `Motorcycles`, sau đó lấy 5 sản phẩm có `MSRP` cao nhất.

**Bài 6.2.** Tạo CTE chứa các khách hàng ở `France`, sau đó sắp xếp theo `creditLimit` giảm dần.

**Bài 6.3.** Tạo CTE chứa các đơn hàng `Shipped`, sau đó lấy 10 đơn hàng mới nhất theo `orderDate`.

**Bài 6.4.** Tạo CTE chứa các khoản thanh toán năm 2004, sau đó lấy 5 khoản lớn nhất.

**Bài 6.5.** Tạo CTE chứa các sản phẩm tồn kho dưới 1000, sau đó sắp xếp tăng dần theo `quantityInStock`.

---

## 7. CTE để tổng hợp dữ liệu

CTE rất hữu ích khi cần tổng hợp trước, sau đó lọc hoặc nối với bảng khác.

### Ví dụ 1: Tổng thanh toán theo khách hàng

```sql
WITH payment_summary AS (
    SELECT 
        customerNumber,
        SUM(amount) AS totalPayment
    FROM payments
    GROUP BY customerNumber
)
SELECT 
    customerNumber,
    totalPayment
FROM payment_summary
ORDER BY totalPayment DESC;
```

### Ví dụ 2: Doanh thu từng đơn hàng

```sql
WITH order_summary AS (
    SELECT
        orderNumber,
        SUM(quantityOrdered * priceEach) AS orderTotal
    FROM orderdetails
    GROUP BY orderNumber
)
SELECT
    orderNumber,
    orderTotal
FROM order_summary
WHERE orderTotal > 50000
ORDER BY orderTotal DESC;
```

### Ví dụ 3: Số đơn hàng theo khách hàng

```sql
WITH order_count AS (
    SELECT
        customerNumber,
        COUNT(*) AS numberOfOrders
    FROM orders
    GROUP BY customerNumber
)
SELECT *
FROM order_count
WHERE numberOfOrders > 5;
```

### Bài tập thực hành

**Bài 7.1.** Dùng CTE tính tổng thanh toán theo khách hàng.

**Bài 7.2.** Dùng CTE tính doanh thu từng đơn hàng.

**Bài 7.3.** Dùng CTE đếm số đơn hàng theo khách hàng.

**Bài 7.4.** Dùng CTE đếm số sản phẩm theo dòng sản phẩm.

**Bài 7.5.** Dùng CTE đếm số nhân viên theo văn phòng.

---

## 8. CTE kết hợp `JOIN`

CTE có thể được nối với bảng khác như một bảng thông thường.

### Ví dụ 1: Tổng thanh toán kèm tên khách hàng

```sql
WITH payment_summary AS (
    SELECT
        customerNumber,
        SUM(amount) AS totalPayment
    FROM payments
    GROUP BY customerNumber
)
SELECT
    c.customerNumber,
    c.customerName,
    ps.totalPayment
FROM customers c
JOIN payment_summary ps
    ON c.customerNumber = ps.customerNumber
ORDER BY ps.totalPayment DESC;
```

### Ví dụ 2: Doanh thu đơn hàng kèm tên khách hàng

```sql
WITH order_summary AS (
    SELECT
        orderNumber,
        SUM(quantityOrdered * priceEach) AS orderTotal
    FROM orderdetails
    GROUP BY orderNumber
)
SELECT
    o.orderNumber,
    o.orderDate,
    c.customerName,
    os.orderTotal
FROM orders o
JOIN customers c
    ON o.customerNumber = c.customerNumber
JOIN order_summary os
    ON o.orderNumber = os.orderNumber
ORDER BY os.orderTotal DESC;
```

### Bài tập thực hành

**Bài 8.1.** Dùng CTE tính tổng thanh toán theo khách hàng, sau đó nối với `customers` để lấy tên khách hàng.

**Bài 8.2.** Dùng CTE tính doanh thu từng đơn hàng, sau đó nối với `orders`.

**Bài 8.3.** Dùng CTE đếm số đơn hàng theo khách hàng, sau đó nối với `customers`.

**Bài 8.4.** Dùng CTE đếm số khách hàng theo nhân viên bán hàng, sau đó nối với `employees`.

**Bài 8.5.** Dùng CTE tính doanh thu từng sản phẩm, sau đó nối với `products`.

---

## 9. Nhiều CTE trong cùng một truy vấn

Có thể khai báo nhiều CTE trong cùng một mệnh đề `WITH`, cách nhau bằng dấu phẩy.

### Ví dụ: Tổng thanh toán và số đơn hàng theo khách hàng

```sql
WITH 
payment_summary AS (
    SELECT
        customerNumber,
        SUM(amount) AS totalPayment
    FROM payments
    GROUP BY customerNumber
),
order_count AS (
    SELECT
        customerNumber,
        COUNT(*) AS numberOfOrders
    FROM orders
    GROUP BY customerNumber
)
SELECT
    c.customerNumber,
    c.customerName,
    ps.totalPayment,
    oc.numberOfOrders
FROM customers c
LEFT JOIN payment_summary ps
    ON c.customerNumber = ps.customerNumber
LEFT JOIN order_count oc
    ON c.customerNumber = oc.customerNumber;
```

### Bài tập thực hành

**Bài 9.1.** Tạo hai CTE: tổng thanh toán theo khách hàng và số đơn hàng theo khách hàng; sau đó hiển thị cùng với tên khách hàng.

**Bài 9.2.** Tạo hai CTE: doanh thu từng đơn hàng và thông tin đơn hàng đã giao; sau đó nối hai CTE.

**Bài 9.3.** Tạo hai CTE: số khách hàng theo nhân viên và thông tin nhân viên bán hàng; sau đó hiển thị nhân viên có nhiều khách hàng.

**Bài 9.4.** Tạo hai CTE: doanh thu từng sản phẩm và thông tin sản phẩm thuộc `Classic Cars`; sau đó nối hai CTE.

**Bài 9.5.** Tạo hai CTE: khách hàng ở `USA` và tổng thanh toán theo khách hàng; sau đó hiển thị tổng thanh toán của khách hàng ở `USA`.

---

## 10. CTE đệ quy với `WITH RECURSIVE`

CTE đệ quy dùng để xử lý dữ liệu phân cấp.

Trong `classicmodels`, bảng `employees` có quan hệ tự tham chiếu:

```text
employees.reportsTo -> employees.employeeNumber
```

Điều này biểu diễn nhân viên báo cáo cho quản lý nào.

### Ví dụ: Cây quản lý nhân viên

```sql
WITH RECURSIVE employee_hierarchy AS (
    SELECT
        employeeNumber,
        lastName,
        firstName,
        reportsTo,
        jobTitle,
        0 AS level
    FROM employees
    WHERE reportsTo IS NULL

    UNION ALL

    SELECT
        e.employeeNumber,
        e.lastName,
        e.firstName,
        e.reportsTo,
        e.jobTitle,
        eh.level + 1 AS level
    FROM employees e
    JOIN employee_hierarchy eh
        ON e.reportsTo = eh.employeeNumber
)
SELECT *
FROM employee_hierarchy
ORDER BY level, employeeNumber;
```

Ý nghĩa:

1. Phần đầu lấy nhân viên không có quản lý trực tiếp.
2. Phần sau tìm các nhân viên báo cáo cho người đã có trong CTE.
3. Quá trình lặp tiếp tục cho đến khi không còn nhân viên cấp dưới.

### Bài tập thực hành

**Bài 10.1.** Viết CTE đệ quy lấy toàn bộ cây quản lý từ nhân viên không có `reportsTo`.

**Bài 10.2.** Bổ sung cột `level` để biểu diễn cấp bậc quản lý.

**Bài 10.3.** Sắp xếp kết quả theo `level`, sau đó theo `employeeNumber`.

**Bài 10.4.** Chỉ hiển thị các cột `employeeNumber`, `firstName`, `lastName`, `jobTitle`, `level`.

**Bài 10.5.** Thử lọc các nhân viên có `level >= 2`.

---

## 11. So sánh CTE thường và CTE đệ quy

| Loại CTE | Cú pháp | Mục đích |
|---|---|---|
| CTE thường | `WITH cte_name AS (...)` | Đặt tên cho kết quả trung gian |
| Nhiều CTE | `WITH cte1 AS (...), cte2 AS (...)` | Chia truy vấn phức tạp thành nhiều bước |
| CTE đệ quy | `WITH RECURSIVE cte_name AS (...)` | Xử lý dữ liệu phân cấp hoặc lặp |

---

## 12. Một số lỗi thường gặp

### Lỗi 1: Quên dấu phẩy giữa nhiều CTE

Sai:

```sql
WITH 
cte1 AS (
    SELECT ...
)
cte2 AS (
    SELECT ...
)
SELECT ...
```

Đúng:

```sql
WITH 
cte1 AS (
    SELECT ...
),
cte2 AS (
    SELECT ...
)
SELECT ...
```

---

### Lỗi 2: Dùng tên CTE trùng với tên bảng gây khó đọc

Không nên:

```sql
WITH customers AS (
    SELECT *
    FROM customers
    WHERE country = 'USA'
)
SELECT *
FROM customers;
```

Nên dùng tên rõ hơn:

```sql
WITH usa_customers AS (
    SELECT *
    FROM customers
    WHERE country = 'USA'
)
SELECT *
FROM usa_customers;
```

---

### Lỗi 3: Quên truy vấn chính sau phần `WITH`

Sai:

```sql
WITH usa_customers AS (
    SELECT *
    FROM customers
    WHERE country = 'USA'
);
```

Đúng:

```sql
WITH usa_customers AS (
    SELECT *
    FROM customers
    WHERE country = 'USA'
)
SELECT *
FROM usa_customers;
```

---

### Lỗi 4: CTE đệ quy thiếu phần cơ sở hoặc phần đệ quy

Một CTE đệ quy thường cần:

1. Phần cơ sở.
2. `UNION ALL`.
3. Phần đệ quy.

---

### Lỗi 5: Điều kiện nối sai trong CTE đệ quy

Nếu điều kiện nối sai, kết quả có thể thiếu dữ liệu, lặp sai hoặc chạy quá lâu.

---

## 13. Bài tập tổng hợp

**Bài 13.1.** Dùng CTE để lấy 10 sản phẩm `Classic Cars` có `MSRP` cao nhất.

**Bài 13.2.** Dùng CTE tính tổng thanh toán theo khách hàng.

**Bài 13.3.** Dùng CTE và `JOIN` để hiển thị `customerName` và `totalPayment`.

**Bài 13.4.** Dùng CTE tính doanh thu từng đơn hàng.

**Bài 13.5.** Dùng CTE và `JOIN` để hiển thị `orderNumber`, `orderDate`, `customerName`, `orderTotal`.

**Bài 13.6.** Dùng hai CTE để hiển thị tên khách hàng, tổng số đơn hàng và tổng số tiền thanh toán.

**Bài 13.7.** Dùng CTE đệ quy để hiển thị cây quản lý nhân viên.

**Bài 13.8.** Dùng CTE đếm số khách hàng theo nhân viên bán hàng.

**Bài 13.9.** Dùng CTE tính doanh thu từng sản phẩm.

**Bài 13.10.** Dùng nhiều CTE để lọc khách hàng ở `USA` và hiển thị tổng thanh toán của họ.

---

## 14. Đáp án gợi ý

### Bài 5.1

```sql
WITH classic_cars AS (
    SELECT productCode, productName, productLine, MSRP
    FROM products
    WHERE productLine = 'Classic Cars'
)
SELECT *
FROM classic_cars;
```

### Bài 7.1

```sql
WITH payment_summary AS (
    SELECT 
        customerNumber,
        SUM(amount) AS totalPayment
    FROM payments
    GROUP BY customerNumber
)
SELECT *
FROM payment_summary;
```

### Bài 8.1

```sql
WITH payment_summary AS (
    SELECT
        customerNumber,
        SUM(amount) AS totalPayment
    FROM payments
    GROUP BY customerNumber
)
SELECT
    c.customerNumber,
    c.customerName,
    ps.totalPayment
FROM customers c
JOIN payment_summary ps
    ON c.customerNumber = ps.customerNumber;
```

### Bài 9.1

```sql
WITH 
payment_summary AS (
    SELECT
        customerNumber,
        SUM(amount) AS totalPayment
    FROM payments
    GROUP BY customerNumber
),
order_count AS (
    SELECT
        customerNumber,
        COUNT(*) AS numberOfOrders
    FROM orders
    GROUP BY customerNumber
)
SELECT
    c.customerNumber,
    c.customerName,
    ps.totalPayment,
    oc.numberOfOrders
FROM customers c
LEFT JOIN payment_summary ps
    ON c.customerNumber = ps.customerNumber
LEFT JOIN order_count oc
    ON c.customerNumber = oc.customerNumber;
```

### Bài 10.1

```sql
WITH RECURSIVE employee_hierarchy AS (
    SELECT
        employeeNumber,
        lastName,
        firstName,
        reportsTo,
        jobTitle,
        0 AS level
    FROM employees
    WHERE reportsTo IS NULL

    UNION ALL

    SELECT
        e.employeeNumber,
        e.lastName,
        e.firstName,
        e.reportsTo,
        e.jobTitle,
        eh.level + 1 AS level
    FROM employees e
    JOIN employee_hierarchy eh
        ON e.reportsTo = eh.employeeNumber
)
SELECT *
FROM employee_hierarchy
ORDER BY level, employeeNumber;
```

---

## 15. Tóm tắt

- CTE là kết quả trung gian được đặt tên trong một truy vấn.
- CTE được viết bằng mệnh đề `WITH`.
- CTE giúp truy vấn dài dễ đọc và dễ bảo trì hơn.
- Có thể dùng CTE với `WHERE`, `ORDER BY`, `LIMIT`, `JOIN`, `GROUP BY`.
- Có thể khai báo nhiều CTE trong cùng một truy vấn.
- `WITH RECURSIVE` dùng để xử lý dữ liệu phân cấp.
- Bảng `employees` trong `classicmodels` có thể dùng để thực hành CTE đệ quy vì có quan hệ `reportsTo`.

---

## 16. Từ khóa chính

- SQL
- MySQL
- CTE
- Common Table Expression
- `WITH`
- `WITH RECURSIVE`
- Recursive CTE
- `UNION ALL`
- `JOIN`
- `GROUP BY`
- `classicmodels`
