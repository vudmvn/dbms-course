---
title: "Lab: Kết nối nhiều bảng bằng WHERE và JOIN ON trong MySQL"
author: "Tên giảng viên"
duration: "90m"
difficulty: "Beginner-Intermediate"
prerequisites:
    - "Đã biết SELECT, FROM, WHERE"
    - "Đã biết khóa chính và khóa ngoại trong CSDL quan hệ"
    - "Đã biết truy vấn một bảng trên classicmodels"
summary: "Thực hành so sánh cách kết nối nhiều bảng bằng FROM nhiều bảng kết hợp WHERE và cú pháp JOIN ... ON; tìm hiểu điều kiện ON không nhất thiết phải là dấu bằng."
---

# Lab: Kết nối nhiều bảng bằng `WHERE` và `JOIN ... ON` trong MySQL

## 1. Mục tiêu học tập

Sau khi hoàn thành lab này, người học có thể:

1. Giải thích được cách kết nối nhiều bảng bằng `FROM` nhiều bảng và điều kiện `WHERE`.
2. Giải thích được cách kết nối nhiều bảng bằng `JOIN ... ON`.
3. So sánh được implicit join và explicit join.
4. Nhận biết được khi nào hai cách viết cho kết quả tương đương.
5. Hiểu rủi ro khi quên điều kiện nối bảng.
6. Phân biệt điều kiện nối bảng và điều kiện lọc dữ liệu.
7. Viết truy vấn nối 2 bảng và 3 bảng trên cơ sở dữ liệu `classicmodels`.
8. Giải thích được vì sao nên ưu tiên dùng `JOIN ... ON`.
9. Biết rằng điều kiện trong `ON` không bắt buộc phải là dấu `=`.
10. Viết được một số truy vấn có điều kiện `ON` dùng `<`, `>`, `<>`, `BETWEEN`, hoặc nhiều điều kiện kết hợp.

---

## 2. Cơ sở dữ liệu mẫu `classicmodels`

Trong lab này, ta sử dụng cơ sở dữ liệu mẫu `classicmodels`.

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
| `orderdetails.orderNumber -> orders.orderNumber` | Mỗi dòng chi tiết thuộc về một đơn hàng |
| `orderdetails.productCode -> products.productCode` | Mỗi dòng chi tiết ứng với một sản phẩm |
| `payments.customerNumber -> customers.customerNumber` | Mỗi khoản thanh toán thuộc về một khách hàng |
| `customers.salesRepEmployeeNumber -> employees.employeeNumber` | Khách hàng có thể được phụ trách bởi một nhân viên |
| `employees.officeCode -> offices.officeCode` | Mỗi nhân viên thuộc về một văn phòng |
| `employees.reportsTo -> employees.employeeNumber` | Nhân viên có thể báo cáo cho một quản lý |

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
DESC customers;
DESC orders;
DESC orderdetails;
DESC products;
DESC payments;
DESC employees;
DESC offices;
```

### Bài tập thực hành

**Bài 3.1.** Viết lệnh chọn cơ sở dữ liệu `classicmodels`.

**Bài 3.2.** Viết lệnh xem cấu trúc bảng `customers`.

**Bài 3.3.** Viết lệnh xem cấu trúc bảng `orders`.

**Bài 3.4.** Viết lệnh xem cấu trúc bảng `orderdetails`.

**Bài 3.5.** Xác định khóa ngoại trong bảng `orders`.

---

## 4. Kết nối nhiều bảng bằng `FROM` nhiều bảng và `WHERE`

Cách viết sau dùng nhiều bảng trong mệnh đề `FROM`, sau đó đặt điều kiện nối bảng trong mệnh đề `WHERE`:

```sql
SELECT 
    customers.customerNumber,
    customers.customerName,
    orders.orderNumber,
    orders.orderDate
FROM customers, orders
WHERE customers.customerNumber = orders.customerNumber;
```

Cách viết này thường gọi là **implicit join**.

Ý nghĩa:

1. Lấy dữ liệu từ bảng `customers` và bảng `orders`.
2. Ghép khách hàng với đơn hàng dựa trên `customerNumber`.
3. Chỉ giữ các dòng có mã khách hàng khớp nhau.

Có thể viết ngắn hơn bằng bí danh bảng:

```sql
SELECT 
    c.customerNumber,
    c.customerName,
    o.orderNumber,
    o.orderDate
FROM customers c, orders o
WHERE c.customerNumber = o.customerNumber;
```

### Bài tập thực hành

**Bài 4.1.** Dùng `FROM customers, orders` và `WHERE` để lấy tên khách hàng và mã đơn hàng.

**Bài 4.2.** Dùng `FROM orders, orderdetails` và `WHERE` để lấy mã đơn hàng, mã sản phẩm, số lượng đặt.

**Bài 4.3.** Dùng `FROM employees, offices` và `WHERE` để lấy tên nhân viên và thành phố văn phòng.

**Bài 4.4.** Dùng `FROM customers, payments` và `WHERE` để lấy tên khách hàng và số tiền thanh toán.

**Bài 4.5.** Dùng bí danh bảng để viết lại Bài 4.1.

---

## 5. Rủi ro khi quên điều kiện nối bảng

Nếu viết:

```sql
SELECT 
    c.customerName,
    o.orderNumber
FROM customers c, orders o;
```

thì truy vấn không có điều kiện nối bảng.

Khi đó SQL tạo ra **tích Descartes**:

```text
Mỗi khách hàng được ghép với mọi đơn hàng.
```

Nếu bảng `customers` có nhiều dòng và bảng `orders` có nhiều dòng, kết quả sẽ rất lớn và thường không đúng về mặt nghiệp vụ.

Ví dụ sai:

```sql
SELECT 
    c.customerName,
    o.orderNumber
FROM customers c, orders o;
```

Ví dụ đúng:

```sql
SELECT 
    c.customerName,
    o.orderNumber
FROM customers c, orders o
WHERE c.customerNumber = o.customerNumber;
```

### Bài tập thực hành

**Bài 5.1.** Giải thích vì sao truy vấn sau có thể tạo ra quá nhiều dòng:

```sql
SELECT c.customerName, o.orderNumber
FROM customers c, orders o;
```

**Bài 5.2.** Sửa truy vấn ở Bài 5.1 để chỉ ghép khách hàng với đơn hàng của chính họ.

**Bài 5.3.** Viết một truy vấn sai do thiếu điều kiện nối giữa `orders` và `orderdetails`.

**Bài 5.4.** Sửa truy vấn ở Bài 5.3.

**Bài 5.5.** Giải thích vì sao cần cẩn thận khi dùng `FROM` nhiều bảng.

---

## 6. Kết nối nhiều bảng bằng `JOIN ... ON`

Cú pháp hiện đại hơn là dùng `JOIN ... ON`.

Ví dụ:

```sql
SELECT 
    c.customerNumber,
    c.customerName,
    o.orderNumber,
    o.orderDate
FROM customers c
JOIN orders o
    ON c.customerNumber = o.customerNumber;
```

Cách viết này gọi là **explicit join**.

Trong đó:

| Thành phần | Ý nghĩa |
|---|---|
| `JOIN orders o` | Chỉ rõ bảng cần nối thêm |
| `ON c.customerNumber = o.customerNumber` | Chỉ rõ điều kiện nối bảng |
| `WHERE` | Dùng cho điều kiện lọc dữ liệu sau khi nối |

Ví dụ có thêm điều kiện lọc:

```sql
SELECT 
    c.customerNumber,
    c.customerName,
    o.orderNumber,
    o.orderDate,
    o.status
FROM customers c
JOIN orders o
    ON c.customerNumber = o.customerNumber
WHERE o.status = 'Shipped';
```

Ở đây:

```sql
ON c.customerNumber = o.customerNumber
```

là điều kiện nối bảng.

Còn:

```sql
WHERE o.status = 'Shipped'
```

là điều kiện lọc dữ liệu.

### Bài tập thực hành

**Bài 6.1.** Dùng `JOIN ... ON` để lấy tên khách hàng và mã đơn hàng.

**Bài 6.2.** Dùng `JOIN ... ON` để lấy mã đơn hàng, mã sản phẩm, số lượng đặt.

**Bài 6.3.** Dùng `JOIN ... ON` để lấy tên nhân viên và thành phố văn phòng.

**Bài 6.4.** Dùng `JOIN ... ON` để lấy tên khách hàng và số tiền thanh toán.

**Bài 6.5.** Dùng `JOIN ... ON` để lấy khách hàng và đơn hàng có trạng thái `Cancelled`.

---

## 7. Hai cách viết có tương đương không?

Với **INNER JOIN**, hai cách viết sau thường cho cùng kết quả.

### Cách 1: `FROM` nhiều bảng + `WHERE`

```sql
SELECT 
    c.customerName,
    o.orderNumber
FROM customers c, orders o
WHERE c.customerNumber = o.customerNumber;
```

### Cách 2: `JOIN ... ON`

```sql
SELECT 
    c.customerName,
    o.orderNumber
FROM customers c
JOIN orders o
    ON c.customerNumber = o.customerNumber;
```

Hai truy vấn trên đều lấy các cặp khách hàng - đơn hàng có `customerNumber` khớp nhau.

Tuy nhiên, `JOIN ... ON` thường rõ ràng hơn vì tách điều kiện nối bảng ra khỏi điều kiện lọc dữ liệu.

### Bài tập thực hành

**Bài 7.1.** Viết truy vấn lấy `customerName`, `orderNumber` bằng cách dùng `FROM customers, orders`.

**Bài 7.2.** Viết lại truy vấn Bài 7.1 bằng `JOIN ... ON`.

**Bài 7.3.** Viết truy vấn lấy `orderNumber`, `productCode`, `quantityOrdered` bằng hai cách.

**Bài 7.4.** Viết truy vấn lấy `employeeNumber`, `lastName`, `city` bằng hai cách.

**Bài 7.5.** So sánh độ dễ đọc của hai cách viết khi truy vấn có từ 3 bảng trở lên.

---

## 8. Nối 3 bảng: cách cũ và cách hiện đại

Ví dụ cần lấy:

- Tên khách hàng.
- Mã đơn hàng.
- Mã sản phẩm.
- Số lượng đặt.

Ta cần nối 3 bảng:

```text
customers -> orders -> orderdetails
```

### Cách viết bằng `FROM` nhiều bảng và `WHERE`

```sql
SELECT 
    c.customerName,
    o.orderNumber,
    od.productCode,
    od.quantityOrdered
FROM customers c, orders o, orderdetails od
WHERE c.customerNumber = o.customerNumber
  AND o.orderNumber = od.orderNumber;
```

### Cách viết bằng `JOIN ... ON`

```sql
SELECT 
    c.customerName,
    o.orderNumber,
    od.productCode,
    od.quantityOrdered
FROM customers c
JOIN orders o
    ON c.customerNumber = o.customerNumber
JOIN orderdetails od
    ON o.orderNumber = od.orderNumber;
```

Cách thứ hai dễ đọc hơn vì mỗi lần nối bảng đều có điều kiện `ON` đi kèm.

### Thêm điều kiện lọc

Ví dụ chỉ lấy các đơn hàng đã giao:

```sql
SELECT 
    c.customerName,
    o.orderNumber,
    od.productCode,
    od.quantityOrdered,
    o.status
FROM customers c
JOIN orders o
    ON c.customerNumber = o.customerNumber
JOIN orderdetails od
    ON o.orderNumber = od.orderNumber
WHERE o.status = 'Shipped';
```

### Bài tập thực hành

**Bài 8.1.** Dùng `FROM` nhiều bảng và `WHERE` để nối `customers`, `orders`, `orderdetails`.

**Bài 8.2.** Viết lại Bài 8.1 bằng `JOIN ... ON`.

**Bài 8.3.** Thêm điều kiện chỉ lấy đơn hàng có `status = 'Shipped'`.

**Bài 8.4.** Nối `orders`, `orderdetails`, `products` để lấy mã đơn hàng, tên sản phẩm, số lượng đặt.

**Bài 8.5.** Nối `customers`, `orders`, `orderdetails`, `products` để lấy tên khách hàng, mã đơn hàng, tên sản phẩm, số lượng đặt.

---

## 9. Vì sao nên ưu tiên dùng `JOIN ... ON`?

Nên ưu tiên dùng `JOIN ... ON` vì:

1. Dễ đọc hơn.
2. Dễ thấy bảng nào nối với bảng nào.
3. Dễ tách điều kiện nối bảng và điều kiện lọc dữ liệu.
4. Giảm rủi ro quên điều kiện nối.
5. Cần thiết khi học `LEFT JOIN`, `RIGHT JOIN` hoặc các kiểu nối ngoài.

So sánh:

```sql
SELECT 
    c.customerName,
    o.orderNumber,
    od.productCode
FROM customers c, orders o, orderdetails od
WHERE c.customerNumber = o.customerNumber
  AND o.orderNumber = od.orderNumber
  AND o.status = 'Shipped';
```

với:

```sql
SELECT 
    c.customerName,
    o.orderNumber,
    od.productCode
FROM customers c
JOIN orders o
    ON c.customerNumber = o.customerNumber
JOIN orderdetails od
    ON o.orderNumber = od.orderNumber
WHERE o.status = 'Shipped';
```

Ở cách thứ hai:

- `ON` dùng để nối bảng.
- `WHERE` dùng để lọc dữ liệu.

---

## 10. Khác biệt quan trọng với `LEFT JOIN`

Với `INNER JOIN`, điều kiện trong `ON` và `WHERE` đôi khi có thể cho kết quả giống nhau.

Nhưng với `LEFT JOIN`, vị trí đặt điều kiện có thể làm thay đổi kết quả.

### Điều kiện lọc đặt trong `ON`

```sql
SELECT 
    c.customerNumber,
    c.customerName,
    o.orderNumber,
    o.status
FROM customers c
LEFT JOIN orders o
    ON c.customerNumber = o.customerNumber
   AND o.status = 'Cancelled';
```

Ý nghĩa:

- Lấy tất cả khách hàng.
- Nếu khách hàng có đơn hàng `Cancelled`, hiển thị đơn hàng đó.
- Nếu khách hàng không có đơn hàng `Cancelled`, vẫn giữ khách hàng, nhưng thông tin đơn hàng là `NULL`.

### Điều kiện lọc đặt trong `WHERE`

```sql
SELECT 
    c.customerNumber,
    c.customerName,
    o.orderNumber,
    o.status
FROM customers c
LEFT JOIN orders o
    ON c.customerNumber = o.customerNumber
WHERE o.status = 'Cancelled';
```

Ý nghĩa:

- Nối khách hàng với đơn hàng.
- Sau đó chỉ giữ dòng có `o.status = 'Cancelled'`.
- Những khách hàng không có đơn hàng `Cancelled` bị loại bỏ.
- Kết quả gần giống `INNER JOIN` đối với điều kiện này.

### Bài tập thực hành

**Bài 10.1.** Viết `LEFT JOIN` lấy tất cả khách hàng và các đơn hàng `Cancelled` nếu có, đặt điều kiện `status` trong `ON`.

**Bài 10.2.** Viết `LEFT JOIN` lấy khách hàng và đơn hàng, sau đó đặt điều kiện `status = 'Cancelled'` trong `WHERE`.

**Bài 10.3.** So sánh ý nghĩa của hai truy vấn trên.

**Bài 10.4.** Viết truy vấn lấy tất cả khách hàng và thông tin thanh toán nếu có.

**Bài 10.5.** Viết truy vấn lấy các khách hàng chưa có đơn hàng bằng `LEFT JOIN`.

---

## 11. Điều kiện trong `ON` có bắt buộc là dấu `=` không?

Không.

Điều kiện trong `ON` là một biểu thức logic, nên có thể dùng nhiều toán tử khác nhau:

```sql
=
<>
>
<
>=
<=
BETWEEN
LIKE
AND
OR
```

Tuy nhiên, khi nối bảng theo khóa chính - khóa ngoại, điều kiện phổ biến nhất vẫn là dấu `=`.

Ví dụ phổ biến:

```sql
SELECT 
    c.customerName,
    o.orderNumber
FROM customers c
JOIN orders o
    ON c.customerNumber = o.customerNumber;
```

Ở đây dùng `=` vì:

```text
orders.customerNumber tham chiếu customers.customerNumber
```

---

## 12. Ví dụ `ON` dùng `<` hoặc `>`

Ví dụ tìm các cặp sản phẩm trong cùng dòng sản phẩm, trong đó sản phẩm thứ nhất có giá thấp hơn sản phẩm thứ hai:

```sql
SELECT 
    p1.productLine,
    p1.productName AS product1,
    p1.MSRP AS price1,
    p2.productName AS product2,
    p2.MSRP AS price2
FROM products p1
JOIN products p2
    ON p1.productLine = p2.productLine
   AND p1.MSRP < p2.MSRP;
```

Điều kiện nối gồm:

```sql
p1.productLine = p2.productLine
```

và:

```sql
p1.MSRP < p2.MSRP
```

Điều kiện thứ hai không phải là dấu `=`.

### Bài tập thực hành

**Bài 12.1.** Viết truy vấn tìm các cặp sản phẩm cùng `productLine`, trong đó `p1.MSRP < p2.MSRP`.

**Bài 12.2.** Thay điều kiện thành `p1.buyPrice < p2.buyPrice`.

**Bài 12.3.** Lọc thêm chỉ xét các sản phẩm thuộc `Classic Cars`.

**Bài 12.4.** Chỉ hiển thị `productLine`, `product1`, `price1`, `product2`, `price2`.

**Bài 12.5.** Giải thích vì sao đây là self join.

---

## 13. Ví dụ `ON` dùng `<>`

Ví dụ lấy các cặp nhân viên khác nhau trong cùng một văn phòng:

```sql
SELECT 
    e1.officeCode,
    e1.employeeNumber AS employee1,
    e1.lastName AS employee1LastName,
    e2.employeeNumber AS employee2,
    e2.lastName AS employee2LastName
FROM employees e1
JOIN employees e2
    ON e1.officeCode = e2.officeCode
   AND e1.employeeNumber <> e2.employeeNumber;
```

Điều kiện:

```sql
e1.officeCode = e2.officeCode
```

giúp lấy nhân viên cùng văn phòng.

Điều kiện:

```sql
e1.employeeNumber <> e2.employeeNumber
```

giúp tránh ghép một nhân viên với chính họ.

Tuy nhiên, truy vấn này có thể tạo cặp lặp:

```text
A - B
B - A
```

Để tránh cặp lặp, có thể dùng:

```sql
SELECT 
    e1.officeCode,
    e1.employeeNumber AS employee1,
    e1.lastName AS employee1LastName,
    e2.employeeNumber AS employee2,
    e2.lastName AS employee2LastName
FROM employees e1
JOIN employees e2
    ON e1.officeCode = e2.officeCode
   AND e1.employeeNumber < e2.employeeNumber;
```

### Bài tập thực hành

**Bài 13.1.** Viết truy vấn lấy các cặp nhân viên khác nhau trong cùng văn phòng bằng `<>`.

**Bài 13.2.** Viết lại truy vấn để tránh cặp lặp bằng `<`.

**Bài 13.3.** Bổ sung `firstName` cho từng nhân viên.

**Bài 13.4.** Chỉ lấy các cặp nhân viên trong văn phòng có `officeCode = '1'`.

**Bài 13.5.** Giải thích sự khác nhau giữa `<>` và `<` trong ví dụ này.

---

## 14. Ví dụ `ON` dùng điều kiện ngày tháng

Ví dụ ghép đơn hàng với các khoản thanh toán của cùng khách hàng, trong đó ngày thanh toán sau ngày đặt hàng:

```sql
SELECT
    o.orderNumber,
    o.orderDate,
    o.customerNumber,
    p.checkNumber,
    p.paymentDate,
    p.amount
FROM orders o
JOIN payments p
    ON o.customerNumber = p.customerNumber
   AND p.paymentDate >= o.orderDate;
```

Điều kiện nối gồm:

```sql
o.customerNumber = p.customerNumber
```

và:

```sql
p.paymentDate >= o.orderDate
```

Lưu ý: trong schema `classicmodels`, bảng `payments` không lưu trực tiếp `orderNumber`. Vì vậy, truy vấn này chỉ ghép theo khách hàng và thời gian; không khẳng định khoản thanh toán đó chắc chắn thuộc đúng đơn hàng đó.

### Bài tập thực hành

**Bài 14.1.** Viết truy vấn ghép `orders` với `payments` theo cùng `customerNumber`.

**Bài 14.2.** Bổ sung điều kiện `p.paymentDate >= o.orderDate` trong `ON`.

**Bài 14.3.** Bổ sung điều kiện chỉ lấy đơn hàng trong năm 2005.

**Bài 14.4.** Sắp xếp kết quả theo `customerNumber`, `orderDate`, `paymentDate`.

**Bài 14.5.** Giải thích giới hạn nghiệp vụ của truy vấn này.

---

## 15. Ví dụ `ON` dùng khoảng giá

Có thể tự tạo bảng mức giá tạm thời rồi nối sản phẩm với mức giá tương ứng.

```sql
SELECT 
    p.productCode,
    p.productName,
    p.MSRP,
    price_band.bandName
FROM products p
JOIN (
    SELECT 'Low' AS bandName, 0 AS minPrice, 50 AS maxPrice
    UNION ALL
    SELECT 'Medium', 50, 100
    UNION ALL
    SELECT 'High', 100, 1000
) AS price_band
    ON p.MSRP > price_band.minPrice
   AND p.MSRP <= price_band.maxPrice;
```

Ở đây điều kiện `ON` không nối theo khóa ngoại. Nó nối sản phẩm với một khoảng giá phù hợp.

Có thể hiểu:

```text
Nếu MSRP nằm trong khoảng của mức giá nào thì sản phẩm thuộc mức giá đó.
```

### Bài tập thực hành

**Bài 15.1.** Chạy truy vấn phân loại sản phẩm theo ba mức `Low`, `Medium`, `High`.

**Bài 15.2.** Đổi khoảng giá thành `Budget`, `Standard`, `Premium`.

**Bài 15.3.** Chỉ phân loại sản phẩm thuộc dòng `Motorcycles`.

**Bài 15.4.** Sắp xếp kết quả theo `bandName`, sau đó theo `MSRP`.

**Bài 15.5.** Giải thích vì sao điều kiện `ON` ở đây không phải là khóa chính - khóa ngoại.

---

## 16. Tóm tắt so sánh

| Nội dung | `FROM A, B WHERE ...` | `JOIN ... ON ...` |
|---|---|---|
| Tên gọi | Implicit join | Explicit join |
| Phong cách | Cũ hơn | Hiện đại hơn |
| Điều kiện nối | Đặt trong `WHERE` | Đặt trong `ON` |
| Điều kiện lọc | Cũng đặt trong `WHERE` | Thường đặt trong `WHERE` |
| Độ rõ ràng | Dễ lẫn nối bảng và lọc dữ liệu | Rõ ràng hơn |
| Nguy cơ quên điều kiện nối | Cao hơn | Thấp hơn |
| Phù hợp với `LEFT JOIN` | Không thuận tiện | Rõ ràng và chuẩn hơn |

---

## 17. Một số lỗi thường gặp

### Lỗi 1: Quên điều kiện nối bảng

Sai:

```sql
SELECT c.customerName, o.orderNumber
FROM customers c, orders o;
```

Đúng:

```sql
SELECT c.customerName, o.orderNumber
FROM customers c, orders o
WHERE c.customerNumber = o.customerNumber;
```

Hoặc tốt hơn:

```sql
SELECT c.customerName, o.orderNumber
FROM customers c
JOIN orders o
    ON c.customerNumber = o.customerNumber;
```

---

### Lỗi 2: Đặt mọi điều kiện trong `WHERE`, làm truy vấn khó đọc

Khó đọc:

```sql
SELECT c.customerName, o.orderNumber, od.productCode
FROM customers c, orders o, orderdetails od
WHERE c.customerNumber = o.customerNumber
  AND o.orderNumber = od.orderNumber
  AND o.status = 'Shipped';
```

Dễ đọc hơn:

```sql
SELECT c.customerName, o.orderNumber, od.productCode
FROM customers c
JOIN orders o
    ON c.customerNumber = o.customerNumber
JOIN orderdetails od
    ON o.orderNumber = od.orderNumber
WHERE o.status = 'Shipped';
```

---

### Lỗi 3: Dùng `LEFT JOIN` nhưng đặt điều kiện của bảng bên phải trong `WHERE`

Cần cẩn thận với truy vấn dạng:

```sql
SELECT 
    c.customerNumber,
    c.customerName,
    o.orderNumber
FROM customers c
LEFT JOIN orders o
    ON c.customerNumber = o.customerNumber
WHERE o.status = 'Cancelled';
```

Điều kiện trong `WHERE` có thể loại bỏ các dòng `NULL` sinh ra từ `LEFT JOIN`, khiến kết quả không còn giữ đầy đủ bảng bên trái.

---

### Lỗi 4: Không dùng bí danh bảng khi nối nhiều bảng

Khó đọc:

```sql
SELECT customers.customerName, orders.orderNumber
FROM customers
JOIN orders
    ON customers.customerNumber = orders.customerNumber;
```

Dễ đọc hơn:

```sql
SELECT c.customerName, o.orderNumber
FROM customers c
JOIN orders o
    ON c.customerNumber = o.customerNumber;
```

---

## 18. Bài tập tổng hợp

### Bài 18.1

Viết truy vấn lấy `customerName`, `orderNumber`, `orderDate` bằng cách dùng `FROM customers, orders` và điều kiện `WHERE`.

### Bài 18.2

Viết lại Bài 18.1 bằng `JOIN ... ON`.

### Bài 18.3

Nối 3 bảng `customers`, `orders`, `orderdetails` bằng cách dùng `FROM` nhiều bảng và `WHERE`.

### Bài 18.4

Viết lại Bài 18.3 bằng `JOIN ... ON`.

### Bài 18.5

Nối 4 bảng `customers`, `orders`, `orderdetails`, `products` để lấy:

- `customerName`
- `orderNumber`
- `productName`
- `quantityOrdered`
- `priceEach`

### Bài 18.6

Dùng `LEFT JOIN` để lấy tất cả khách hàng và đơn hàng `Cancelled` nếu có. Đặt điều kiện `status = 'Cancelled'` trong `ON`.

### Bài 18.7

Viết truy vấn so sánh các cặp sản phẩm cùng dòng sản phẩm, trong đó sản phẩm thứ nhất có `MSRP` nhỏ hơn sản phẩm thứ hai.

### Bài 18.8

Viết truy vấn lấy các cặp nhân viên khác nhau trong cùng văn phòng và tránh cặp lặp.

### Bài 18.9

Tạo bảng mức giá tạm bằng subquery trong `FROM`, sau đó phân loại sản phẩm theo `MSRP`.

### Bài 18.10

Giải thích bằng lời: khi nào nên dùng điều kiện trong `ON`, khi nào nên dùng điều kiện trong `WHERE`.

---

## 19. Đáp án gợi ý

### Bài 18.1

```sql
SELECT 
    c.customerName,
    o.orderNumber,
    o.orderDate
FROM customers c, orders o
WHERE c.customerNumber = o.customerNumber;
```

### Bài 18.2

```sql
SELECT 
    c.customerName,
    o.orderNumber,
    o.orderDate
FROM customers c
JOIN orders o
    ON c.customerNumber = o.customerNumber;
```

### Bài 18.3

```sql
SELECT 
    c.customerName,
    o.orderNumber,
    od.productCode,
    od.quantityOrdered
FROM customers c, orders o, orderdetails od
WHERE c.customerNumber = o.customerNumber
  AND o.orderNumber = od.orderNumber;
```

### Bài 18.4

```sql
SELECT 
    c.customerName,
    o.orderNumber,
    od.productCode,
    od.quantityOrdered
FROM customers c
JOIN orders o
    ON c.customerNumber = o.customerNumber
JOIN orderdetails od
    ON o.orderNumber = od.orderNumber;
```

### Bài 18.5

```sql
SELECT 
    c.customerName,
    o.orderNumber,
    p.productName,
    od.quantityOrdered,
    od.priceEach
FROM customers c
JOIN orders o
    ON c.customerNumber = o.customerNumber
JOIN orderdetails od
    ON o.orderNumber = od.orderNumber
JOIN products p
    ON od.productCode = p.productCode;
```

### Bài 18.6

```sql
SELECT 
    c.customerNumber,
    c.customerName,
    o.orderNumber,
    o.status
FROM customers c
LEFT JOIN orders o
    ON c.customerNumber = o.customerNumber
   AND o.status = 'Cancelled';
```

### Bài 18.7

```sql
SELECT 
    p1.productLine,
    p1.productName AS product1,
    p1.MSRP AS price1,
    p2.productName AS product2,
    p2.MSRP AS price2
FROM products p1
JOIN products p2
    ON p1.productLine = p2.productLine
   AND p1.MSRP < p2.MSRP;
```

### Bài 18.8

```sql
SELECT 
    e1.officeCode,
    e1.employeeNumber AS employee1,
    e1.lastName AS employee1LastName,
    e2.employeeNumber AS employee2,
    e2.lastName AS employee2LastName
FROM employees e1
JOIN employees e2
    ON e1.officeCode = e2.officeCode
   AND e1.employeeNumber < e2.employeeNumber;
```

### Bài 18.9

```sql
SELECT 
    p.productCode,
    p.productName,
    p.MSRP,
    price_band.bandName
FROM products p
JOIN (
    SELECT 'Low' AS bandName, 0 AS minPrice, 50 AS maxPrice
    UNION ALL
    SELECT 'Medium', 50, 100
    UNION ALL
    SELECT 'High', 100, 1000
) AS price_band
    ON p.MSRP > price_band.minPrice
   AND p.MSRP <= price_band.maxPrice;
```

---

## 20. Tóm tắt

Các ý chính trong lab:

- `FROM A, B WHERE A.id = B.a_id` là cách nối bảng kiểu cũ, còn gọi là implicit join.
- `JOIN B ON A.id = B.a_id` là cách nối bảng rõ ràng hơn, còn gọi là explicit join.
- Với `INNER JOIN`, hai cách này thường cho cùng kết quả nếu điều kiện nối giống nhau.
- Nếu quên điều kiện nối, truy vấn có thể tạo tích Descartes.
- Nên dùng `JOIN ... ON` vì dễ đọc, dễ bảo trì và ít lỗi hơn.
- Trong `JOIN ... ON`, điều kiện nối không bắt buộc phải là dấu `=`.
- `ON` có thể dùng `<`, `>`, `<>`, `BETWEEN`, `AND`, `OR`.
- Với `LEFT JOIN`, đặt điều kiện trong `ON` và `WHERE` có thể cho kết quả khác nhau.
- Nên dùng `ON` cho điều kiện nối bảng, và dùng `WHERE` cho điều kiện lọc sau khi nối.

---

## 21. Từ khóa chính

- SQL
- MySQL
- `FROM`
- `WHERE`
- `JOIN`
- `ON`
- Implicit join
- Explicit join
- Inner join
- Left join
- Cartesian product
- Self join
- Non-equi join
- Primary key
- Foreign key
- `classicmodels`
