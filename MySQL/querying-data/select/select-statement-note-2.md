---
title: "Tutorial: CTE vs Subquery trong MySQL"
author: "Tên giảng viên"
duration: "90m"
difficulty: "Intermediate"
prerequisites:
    - "Đã biết SELECT, WHERE, JOIN"
    - "Đã biết hàm tổng hợp, GROUP BY"
    - "Đã biết khái niệm truy vấn con cơ bản"
summary: "So sánh Subquery và CTE trong SQL/MySQL: cách viết, ưu điểm, nhược điểm, tình huống sử dụng và ví dụ thực hành trên classicmodels."
---

# Tutorial: CTE vs Subquery trong MySQL

## 1. Mục tiêu học tập

Sau khi hoàn thành tutorial này, người học có thể:

1. Giải thích được **Subquery** là gì.
2. Giải thích được **CTE** là gì.
3. Viết cùng một bài toán bằng Subquery và bằng CTE.
4. So sánh ưu điểm và nhược điểm của Subquery.
5. So sánh ưu điểm và nhược điểm của CTE.
6. Biết khi nào nên dùng Subquery.
7. Biết khi nào nên dùng CTE.
8. Nhận biết các lỗi thường gặp khi dùng Subquery và CTE.
9. Viết truy vấn dễ đọc hơn bằng cách tách bài toán thành nhiều bước.
10. Ứng dụng Subquery và CTE trên cơ sở dữ liệu mẫu `classicmodels`.

---

## 2. Cơ sở dữ liệu mẫu `classicmodels`

Trong tutorial này, ta sử dụng cơ sở dữ liệu mẫu `classicmodels`.

Một số bảng thường dùng:

| Bảng | Ý nghĩa |
|---|---|
| `customers` | Khách hàng |
| `orders` | Đơn hàng |
| `orderdetails` | Chi tiết đơn hàng |
| `products` | Sản phẩm |
| `payments` | Thanh toán |
| `employees` | Nhân viên |
| `offices` | Văn phòng |

Một số quan hệ thường dùng:

| Quan hệ | Ý nghĩa |
|---|---|
| `orders.customerNumber -> customers.customerNumber` | Mỗi đơn hàng thuộc về một khách hàng |
| `payments.customerNumber -> customers.customerNumber` | Mỗi khoản thanh toán thuộc về một khách hàng |
| `orderdetails.orderNumber -> orders.orderNumber` | Mỗi dòng chi tiết thuộc về một đơn hàng |
| `orderdetails.productCode -> products.productCode` | Mỗi dòng chi tiết ứng với một sản phẩm |
| `customers.salesRepEmployeeNumber -> employees.employeeNumber` | Khách hàng có thể được phụ trách bởi nhân viên bán hàng |
| `employees.reportsTo -> employees.employeeNumber` | Nhân viên có thể báo cáo cho một quản lý |

Chọn cơ sở dữ liệu:

```sql
USE classicmodels;
```

---

## 3. Subquery là gì?

**Subquery** hay **truy vấn con** là một câu lệnh `SELECT` được đặt bên trong một truy vấn khác.

Ví dụ:

```sql
SELECT productCode, productName, MSRP
FROM products
WHERE MSRP > (
    SELECT AVG(MSRP)
    FROM products
);
```

Ý nghĩa:

1. Truy vấn con:

```sql
SELECT AVG(MSRP)
FROM products;
```

2. Truy vấn ngoài lấy các sản phẩm có `MSRP` lớn hơn giá trị trung bình đó.

Subquery thường xuất hiện trong:

| Vị trí | Vai trò |
|---|---|
| `WHERE` | Lọc dữ liệu dựa trên kết quả truy vấn khác |
| `SELECT` | Tạo thêm cột tính toán |
| `FROM` | Tạo bảng tạm trong truy vấn, còn gọi là derived table |
| `HAVING` | Lọc nhóm dựa trên kết quả truy vấn khác |

---

## 4. CTE là gì?

**CTE** là viết tắt của **Common Table Expression**.

Trong MySQL, CTE được viết bằng mệnh đề `WITH`.

Ví dụ:

```sql
WITH average_price AS (
    SELECT AVG(MSRP) AS avgMSRP
    FROM products
)
SELECT p.productCode, p.productName, p.MSRP
FROM products p
JOIN average_price ap
    ON p.MSRP > ap.avgMSRP;
```

Ý nghĩa:

1. CTE `average_price` tính giá bán đề xuất trung bình.
2. Truy vấn chính dùng kết quả của CTE để lấy sản phẩm có `MSRP` lớn hơn trung bình.

Cú pháp tổng quát:

```sql
WITH cte_name AS (
    SELECT ...
)
SELECT ...
FROM cte_name;
```

CTE chỉ tồn tại trong phạm vi của truy vấn hiện tại.

---

## 5. So sánh nhanh Subquery và CTE

| Tiêu chí | Subquery | CTE |
|---|---|---|
| Cách viết | Đặt truy vấn con bên trong truy vấn chính | Đặt truy vấn trung gian ở đầu bằng `WITH` |
| Độ ngắn gọn | Gọn với truy vấn đơn giản | Dài hơn một chút với truy vấn đơn giản |
| Độ dễ đọc | Có thể khó đọc khi lồng nhiều tầng | Dễ đọc hơn khi truy vấn phức tạp |
| Tái sử dụng trong cùng truy vấn | Khó tái sử dụng | Có thể tham chiếu CTE nhiều lần |
| Phù hợp với bài toán nhiều bước | Không thuận tiện bằng CTE | Rất phù hợp |
| Xử lý dữ liệu phân cấp | Không thuận tiện | Có thể dùng `WITH RECURSIVE` |
| Hỗ trợ trong MySQL | Có từ lâu | MySQL 8.0 trở lên |

---

## 6. Ví dụ 1: Sản phẩm có giá cao hơn trung bình

### Cách 1: Dùng Subquery

```sql
SELECT productCode, productName, MSRP
FROM products
WHERE MSRP > (
    SELECT AVG(MSRP)
    FROM products
)
ORDER BY MSRP DESC;
```

### Cách 2: Dùng CTE

```sql
WITH average_price AS (
    SELECT AVG(MSRP) AS avgMSRP
    FROM products
)
SELECT p.productCode, p.productName, p.MSRP
FROM products p
JOIN average_price ap
    ON p.MSRP > ap.avgMSRP
ORDER BY p.MSRP DESC;
```

### Nhận xét

Trong ví dụ này, Subquery ngắn gọn hơn.

CTE giúp đặt tên rõ ràng cho kết quả trung gian `average_price`, nhưng có thể hơi dài nếu bài toán đơn giản.

### Bài tập thực hành

**Bài 6.1.** Viết truy vấn lấy sản phẩm có `buyPrice` lớn hơn `buyPrice` trung bình bằng Subquery.

**Bài 6.2.** Viết lại Bài 6.1 bằng CTE.

**Bài 6.3.** Viết truy vấn lấy khách hàng có `creditLimit` lớn hơn `creditLimit` trung bình bằng Subquery.

**Bài 6.4.** Viết lại Bài 6.3 bằng CTE.

**Bài 6.5.** So sánh cách viết nào ngắn hơn trong các bài trên.

---

## 7. Ví dụ 2: Tổng thanh toán theo khách hàng

Bài toán: Tính tổng tiền thanh toán của từng khách hàng, sau đó lấy những khách hàng có tổng thanh toán lớn hơn `100000`.

### Cách 1: Dùng Derived Table

Derived table là subquery trong mệnh đề `FROM`.

```sql
SELECT
    payment_summary.customerNumber,
    payment_summary.totalPayment
FROM (
    SELECT
        customerNumber,
        SUM(amount) AS totalPayment
    FROM payments
    GROUP BY customerNumber
) AS payment_summary
WHERE payment_summary.totalPayment > 100000;
```

### Cách 2: Dùng CTE

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
WHERE totalPayment > 100000;
```

### Nhận xét

Hai cách có ý nghĩa tương tự.

CTE thường dễ đọc hơn vì phần tính toán trung gian được đặt tên ở đầu truy vấn.

### Bài tập thực hành

**Bài 7.1.** Dùng derived table để tính số đơn hàng theo khách hàng, sau đó lấy khách hàng có nhiều hơn 5 đơn hàng.

**Bài 7.2.** Viết lại Bài 7.1 bằng CTE.

**Bài 7.3.** Dùng derived table để tính doanh thu từng đơn hàng từ `orderdetails`.

**Bài 7.4.** Viết lại Bài 7.3 bằng CTE.

**Bài 7.5.** Với bài toán tổng hợp nhiều bước, cách nào dễ đọc hơn?

---

## 8. Ví dụ 3: Nối kết quả tổng hợp với bảng khác

Bài toán: Tính tổng thanh toán theo khách hàng và hiển thị tên khách hàng.

### Cách 1: Dùng Subquery trong `FROM`

```sql
SELECT
    c.customerNumber,
    c.customerName,
    ps.totalPayment
FROM customers c
JOIN (
    SELECT
        customerNumber,
        SUM(amount) AS totalPayment
    FROM payments
    GROUP BY customerNumber
) AS ps
    ON c.customerNumber = ps.customerNumber
ORDER BY ps.totalPayment DESC;
```

### Cách 2: Dùng CTE

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

### Nhận xét

CTE giúp truy vấn chính sạch hơn:

```sql
FROM customers c
JOIN payment_summary ps
```

Khi nhìn truy vấn chính, ta dễ hiểu rằng `payment_summary` là một bảng kết quả đã được chuẩn bị trước.

### Bài tập thực hành

**Bài 8.1.** Dùng subquery trong `FROM` để tính số đơn hàng theo khách hàng và hiển thị `customerName`.

**Bài 8.2.** Viết lại Bài 8.1 bằng CTE.

**Bài 8.3.** Dùng subquery trong `FROM` để tính doanh thu từng đơn hàng và hiển thị `orderDate`.

**Bài 8.4.** Viết lại Bài 8.3 bằng CTE.

**Bài 8.5.** Giải thích vì sao CTE làm truy vấn chính dễ đọc hơn.

---

## 9. Ví dụ 4: Dùng nhiều kết quả trung gian

Bài toán: Với mỗi khách hàng, hiển thị:

- Tên khách hàng.
- Tổng số đơn hàng.
- Tổng tiền thanh toán.

### Cách 1: Dùng nhiều derived table

```sql
SELECT
    c.customerNumber,
    c.customerName,
    oc.numberOfOrders,
    ps.totalPayment
FROM customers c
LEFT JOIN (
    SELECT
        customerNumber,
        COUNT(*) AS numberOfOrders
    FROM orders
    GROUP BY customerNumber
) AS oc
    ON c.customerNumber = oc.customerNumber
LEFT JOIN (
    SELECT
        customerNumber,
        SUM(amount) AS totalPayment
    FROM payments
    GROUP BY customerNumber
) AS ps
    ON c.customerNumber = ps.customerNumber;
```

### Cách 2: Dùng nhiều CTE

```sql
WITH
order_count AS (
    SELECT
        customerNumber,
        COUNT(*) AS numberOfOrders
    FROM orders
    GROUP BY customerNumber
),
payment_summary AS (
    SELECT
        customerNumber,
        SUM(amount) AS totalPayment
    FROM payments
    GROUP BY customerNumber
)
SELECT
    c.customerNumber,
    c.customerName,
    oc.numberOfOrders,
    ps.totalPayment
FROM customers c
LEFT JOIN order_count oc
    ON c.customerNumber = oc.customerNumber
LEFT JOIN payment_summary ps
    ON c.customerNumber = ps.customerNumber;
```

### Nhận xét

Khi có nhiều kết quả trung gian, CTE thường rõ ràng hơn nhiều.

Mỗi bước được đặt tên:

```text
order_count
payment_summary
```

Người đọc có thể hiểu truy vấn như một quy trình gồm nhiều bước.

### Bài tập thực hành

**Bài 9.1.** Viết truy vấn dùng nhiều derived table để hiển thị tên khách hàng, số đơn hàng và tổng thanh toán.

**Bài 9.2.** Viết lại Bài 9.1 bằng nhiều CTE.

**Bài 9.3.** Tạo CTE `order_count` để đếm số đơn hàng theo khách hàng.

**Bài 9.4.** Tạo CTE `payment_summary` để tính tổng thanh toán theo khách hàng.

**Bài 9.5.** Nối hai CTE trên với bảng `customers`.

---

## 10. Ví dụ 5: Subquery trong `SELECT` và CTE

Bài toán: Hiển thị mỗi khách hàng và số đơn hàng của khách hàng đó.

### Cách 1: Subquery trong `SELECT`

```sql
SELECT
    c.customerNumber,
    c.customerName,
    (
        SELECT COUNT(*)
        FROM orders o
        WHERE o.customerNumber = c.customerNumber
    ) AS numberOfOrders
FROM customers c;
```

### Cách 2: CTE + `LEFT JOIN`

```sql
WITH order_count AS (
    SELECT
        customerNumber,
        COUNT(*) AS numberOfOrders
    FROM orders
    GROUP BY customerNumber
)
SELECT
    c.customerNumber,
    c.customerName,
    oc.numberOfOrders
FROM customers c
LEFT JOIN order_count oc
    ON c.customerNumber = oc.customerNumber;
```

### Nhận xét

Subquery trong `SELECT` dễ viết khi chỉ cần tính thêm một thông tin đơn giản.

CTE phù hợp hơn khi:

- Cần dùng kết quả đếm ở nhiều chỗ.
- Cần nối thêm nhiều bảng khác.
- Truy vấn bắt đầu dài và khó đọc.

### Bài tập thực hành

**Bài 10.1.** Dùng subquery trong `SELECT` để hiển thị mỗi khách hàng và tổng tiền thanh toán.

**Bài 10.2.** Viết lại Bài 10.1 bằng CTE.

**Bài 10.3.** Dùng subquery trong `SELECT` để hiển thị mỗi nhân viên và số khách hàng phụ trách.

**Bài 10.4.** Viết lại Bài 10.3 bằng CTE.

**Bài 10.5.** So sánh độ dễ đọc của hai cách viết.

---

## 11. Ưu điểm của Subquery

Subquery có một số ưu điểm:

### 11.1. Gọn với truy vấn đơn giản

Ví dụ:

```sql
SELECT productCode, productName, MSRP
FROM products
WHERE MSRP > (
    SELECT AVG(MSRP)
    FROM products
);
```

Với bài toán đơn giản như trên, subquery ngắn và dễ hiểu.

### 11.2. Tự nhiên khi dùng trong `WHERE`

Ví dụ lấy khách hàng đã từng đặt hàng:

```sql
SELECT customerNumber, customerName
FROM customers
WHERE customerNumber IN (
    SELECT customerNumber
    FROM orders
);
```

### 11.3. Không cần đặt tên bước trung gian

Nếu truy vấn con chỉ dùng một lần và đơn giản, subquery giúp tránh tạo thêm tên CTE không cần thiết.

---

## 12. Nhược điểm của Subquery

Subquery có một số nhược điểm:

### 12.1. Khó đọc khi lồng nhiều tầng

Ví dụ truy vấn có nhiều subquery lồng nhau sẽ nhanh chóng khó theo dõi.

### 12.2. Khó tái sử dụng kết quả trung gian

Nếu cùng một subquery cần dùng nhiều lần, ta có thể phải viết lặp lại.

### 12.3. Derived table dài làm truy vấn chính rối

Ví dụ:

```sql
FROM customers c
JOIN (
    SELECT ...
) AS ps
```

Nếu derived table dài, phần `FROM` của truy vấn chính trở nên khó đọc.

### 12.4. Dễ quên bí danh cho derived table

Trong MySQL, subquery trong `FROM` cần có bí danh:

```sql
SELECT *
FROM (
    SELECT customerNumber, SUM(amount) AS totalPayment
    FROM payments
    GROUP BY customerNumber
) AS payment_summary;
```

---

## 13. Ưu điểm của CTE

CTE có nhiều ưu điểm khi truy vấn phức tạp.

### 13.1. Dễ đọc hơn

CTE giúp chia truy vấn thành các bước có tên rõ ràng.

```sql
WITH payment_summary AS (
    SELECT customerNumber, SUM(amount) AS totalPayment
    FROM payments
    GROUP BY customerNumber
)
SELECT *
FROM payment_summary;
```

### 13.2. Tốt cho bài toán nhiều bước

Ví dụ:

```sql
WITH
order_count AS (...),
payment_summary AS (...)
SELECT ...
```

Mỗi CTE tương ứng với một bước xử lý.

### 13.3. Có thể tham chiếu nhiều lần trong cùng truy vấn

Nếu cần dùng cùng một kết quả trung gian nhiều lần, CTE giúp tránh viết lặp lại.

### 13.4. Hỗ trợ truy vấn đệ quy

CTE có thể dùng `WITH RECURSIVE` để xử lý dữ liệu phân cấp.

Ví dụ bảng `employees` có cột `reportsTo`, biểu diễn quan hệ quản lý - nhân viên.

---

## 14. Nhược điểm của CTE

CTE cũng có một số nhược điểm:

### 14.1. Dài hơn với truy vấn rất đơn giản

Ví dụ lấy sản phẩm có giá cao hơn trung bình, CTE có thể dài hơn subquery.

### 14.2. Cần MySQL 8.0 trở lên

CTE được hỗ trợ từ MySQL 8.0. Nếu dùng phiên bản MySQL cũ hơn, có thể không chạy được.

### 14.3. Không phải lúc nào cũng nhanh hơn

CTE giúp truy vấn dễ đọc hơn, nhưng không đảm bảo luôn chạy nhanh hơn subquery.

Hiệu năng phụ thuộc vào:

- Hệ quản trị cơ sở dữ liệu.
- Bộ tối ưu truy vấn.
- Chỉ mục.
- Kích thước dữ liệu.
- Cách viết truy vấn.

### 14.4. Có thể bị lạm dụng

Nếu truy vấn rất đơn giản, dùng CTE có thể làm câu SQL dài hơn không cần thiết.

---

## 15. Khi nào nên dùng Subquery?

Nên dùng Subquery khi:

1. Truy vấn con ngắn và chỉ dùng một lần.
2. Cần lọc dữ liệu bằng một giá trị tính được.
3. Cần dùng `IN`, `NOT IN`, `EXISTS`, `NOT EXISTS`.
4. Bài toán có thể diễn đạt tự nhiên bằng `WHERE`.
5. Người đọc có thể hiểu truy vấn ngay mà không cần tách bước.

Ví dụ phù hợp:

```sql
SELECT productCode, productName, MSRP
FROM products
WHERE MSRP > (
    SELECT AVG(MSRP)
    FROM products
);
```

---

## 16. Khi nào nên dùng CTE?

Nên dùng CTE khi:

1. Truy vấn có nhiều bước xử lý.
2. Cần tạo nhiều kết quả trung gian.
3. Cần dùng lại cùng một kết quả trung gian.
4. Truy vấn có nhiều phép tổng hợp rồi nối bảng.
5. Truy vấn bằng derived table trở nên dài và khó đọc.
6. Cần xử lý dữ liệu phân cấp bằng `WITH RECURSIVE`.

Ví dụ phù hợp:

```sql
WITH
order_count AS (
    SELECT customerNumber, COUNT(*) AS numberOfOrders
    FROM orders
    GROUP BY customerNumber
),
payment_summary AS (
    SELECT customerNumber, SUM(amount) AS totalPayment
    FROM payments
    GROUP BY customerNumber
)
SELECT
    c.customerNumber,
    c.customerName,
    oc.numberOfOrders,
    ps.totalPayment
FROM customers c
LEFT JOIN order_count oc
    ON c.customerNumber = oc.customerNumber
LEFT JOIN payment_summary ps
    ON c.customerNumber = ps.customerNumber;
```

---

## 17. Bảng quyết định nhanh

| Tình huống | Nên dùng |
|---|---|
| Lọc theo giá trị trung bình, lớn nhất, nhỏ nhất | Subquery |
| Kiểm tra tồn tại dữ liệu liên quan | `EXISTS` hoặc `NOT EXISTS` |
| Truy vấn con chỉ dùng một lần và ngắn | Subquery |
| Cần tổng hợp trước rồi nối với bảng khác | CTE |
| Cần nhiều bước trung gian | CTE |
| Cần dùng lại cùng một kết quả trung gian | CTE |
| Cần xử lý cây quản lý, dữ liệu phân cấp | Recursive CTE |
| Người mới đọc truy vấn cần hiểu từng bước | CTE |

---

## 18. Lỗi thường gặp

### Lỗi 1: Subquery trả về nhiều dòng nhưng dùng `=`

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

### Lỗi 3: Quên truy vấn chính sau CTE

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

### Lỗi 4: Dùng CTE nhưng đặt tên không rõ nghĩa

Không nên:

```sql
WITH t1 AS (...), t2 AS (...)
SELECT ...;
```

Nên đặt tên theo ý nghĩa:

```sql
WITH
order_count AS (...),
payment_summary AS (...)
SELECT ...;
```

---

## 19. Bài tập tổng hợp

### Bài 19.1

Lấy các sản phẩm có `MSRP` lớn hơn `MSRP` trung bình bằng Subquery.

### Bài 19.2

Viết lại Bài 19.1 bằng CTE.

### Bài 19.3

Tính tổng thanh toán theo khách hàng bằng derived table.

### Bài 19.4

Viết lại Bài 19.3 bằng CTE.

### Bài 19.5

Hiển thị tên khách hàng, tổng số đơn hàng và tổng thanh toán bằng nhiều CTE.

### Bài 19.6

Hiển thị mỗi nhân viên và số khách hàng mà nhân viên đó phụ trách bằng subquery trong `SELECT`.

### Bài 19.7

Viết lại Bài 19.6 bằng CTE.

### Bài 19.8

Dùng `EXISTS` để lấy khách hàng có ít nhất một đơn hàng.

### Bài 19.9

Dùng `NOT EXISTS` để lấy sản phẩm chưa từng được đặt.

### Bài 19.10

Viết một đoạn ngắn giải thích khi nào nên dùng Subquery và khi nào nên dùng CTE.

---

## 20. Đáp án gợi ý

### Bài 19.1

```sql
SELECT productCode, productName, MSRP
FROM products
WHERE MSRP > (
    SELECT AVG(MSRP)
    FROM products
);
```

### Bài 19.2

```sql
WITH average_price AS (
    SELECT AVG(MSRP) AS avgMSRP
    FROM products
)
SELECT p.productCode, p.productName, p.MSRP
FROM products p
JOIN average_price ap
    ON p.MSRP > ap.avgMSRP;
```

### Bài 19.3

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
) AS payment_summary;
```

### Bài 19.4

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
FROM payment_summary;
```

### Bài 19.5

```sql
WITH
order_count AS (
    SELECT
        customerNumber,
        COUNT(*) AS numberOfOrders
    FROM orders
    GROUP BY customerNumber
),
payment_summary AS (
    SELECT
        customerNumber,
        SUM(amount) AS totalPayment
    FROM payments
    GROUP BY customerNumber
)
SELECT
    c.customerNumber,
    c.customerName,
    oc.numberOfOrders,
    ps.totalPayment
FROM customers c
LEFT JOIN order_count oc
    ON c.customerNumber = oc.customerNumber
LEFT JOIN payment_summary ps
    ON c.customerNumber = ps.customerNumber;
```

### Bài 19.6

```sql
SELECT
    e.employeeNumber,
    e.lastName,
    e.firstName,
    (
        SELECT COUNT(*)
        FROM customers c
        WHERE c.salesRepEmployeeNumber = e.employeeNumber
    ) AS numberOfCustomers
FROM employees e;
```

### Bài 19.7

```sql
WITH customer_count AS (
    SELECT
        salesRepEmployeeNumber,
        COUNT(*) AS numberOfCustomers
    FROM customers
    WHERE salesRepEmployeeNumber IS NOT NULL
    GROUP BY salesRepEmployeeNumber
)
SELECT
    e.employeeNumber,
    e.lastName,
    e.firstName,
    cc.numberOfCustomers
FROM employees e
LEFT JOIN customer_count cc
    ON e.employeeNumber = cc.salesRepEmployeeNumber;
```

### Bài 19.8

```sql
SELECT customerNumber, customerName
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customerNumber = c.customerNumber
);
```

### Bài 19.9

```sql
SELECT productCode, productName
FROM products p
WHERE NOT EXISTS (
    SELECT 1
    FROM orderdetails od
    WHERE od.productCode = p.productCode
);
```

---

## 21. Tóm tắt

Các ý chính trong tutorial:

- Subquery là truy vấn con nằm bên trong truy vấn khác.
- CTE là kết quả trung gian được đặt tên bằng mệnh đề `WITH`.
- Subquery thường gọn hơn cho truy vấn đơn giản.
- CTE thường dễ đọc hơn cho truy vấn nhiều bước.
- Derived table là subquery nằm trong mệnh đề `FROM`.
- CTE thường giúp thay thế derived table dài để truy vấn dễ đọc hơn.
- Subquery phù hợp khi cần lọc bằng một giá trị hoặc kiểm tra tồn tại.
- CTE phù hợp khi cần tổng hợp trước, nối sau, hoặc chia truy vấn thành nhiều bước.
- CTE không đảm bảo luôn nhanh hơn subquery.
- Hiệu năng phụ thuộc vào dữ liệu, chỉ mục và bộ tối ưu của hệ quản trị CSDL.

---

## 22. Từ khóa chính

- SQL
- MySQL
- Subquery
- Nested query
- Derived table
- CTE
- Common Table Expression
- `WITH`
- `WITH RECURSIVE`
- `EXISTS`
- `NOT EXISTS`
- `IN`
- `NOT IN`
- Query readability
- Query maintainability
- `classicmodels`
