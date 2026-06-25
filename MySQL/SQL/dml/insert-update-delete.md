---
title: "Tutorial: Sử dụng câu lệnh INSERT, UPDATE và DELETE với classicmodels"
author: "Tên giảng viên"
duration: "120m"
difficulty: "Beginner"
prerequisites:
    - "Đã biết SELECT, WHERE và các bảng cơ bản trong classicmodels"
    - "Đã biết khái niệm khóa chính, khóa ngoại và NULL"
    - "Đã biết cách mở MySQL client hoặc MySQL Workbench"
summary: "Thực hành chèn, cập nhật và xóa dữ liệu bằng INSERT, UPDATE, DELETE trên cơ sở dữ liệu mẫu classicmodels."
---

# Tutorial: Sử dụng câu lệnh `INSERT`, `UPDATE` và `DELETE` với `classicmodels`

## Link tham khảo

- [MySQL INSERT Statement](https://dev.mysql.com/doc/refman/8.4/en/insert.html)
- [MySQL UPDATE Statement](https://dev.mysql.com/doc/refman/8.4/en/update.html)
- [MySQL DELETE Statement](https://dev.mysql.com/doc/refman/8.4/en/delete.html)
- [MySQL Transaction Statements](https://dev.mysql.com/doc/refman/8.4/en/commit.html)

## 1. Mục tiêu học tập

Sau khi hoàn thành tutorial này, người học có thể:

1. Hiểu vai trò của `INSERT`, `UPDATE` và `DELETE` trong SQL.
2. Chèn một hoặc nhiều dòng dữ liệu bằng `INSERT`.
3. Chèn dữ liệu lấy từ truy vấn bằng `INSERT ... SELECT`.
4. Cập nhật một hoặc nhiều cột bằng `UPDATE`.
5. Cập nhật dữ liệu bằng biểu thức và `CASE`.
6. Xóa đúng các dòng cần thiết bằng `DELETE ... WHERE`.
7. Hiểu ảnh hưởng của khóa chính và khóa ngoại khi thao tác dữ liệu.
8. Kiểm tra dữ liệu trước và sau khi thao tác bằng `SELECT`.
9. Sử dụng transaction để thực hành an toàn.

---

## 2. Giới thiệu về DML và `classicmodels`

**DML** (*Data Manipulation Language*) là nhóm câu lệnh thao tác với dữ liệu nằm trong bảng.

| Câu lệnh | Mục đích |
|---|---|
| `INSERT` | Chèn dòng dữ liệu mới |
| `UPDATE` | Cập nhật dòng dữ liệu đã tồn tại |
| `DELETE` | Xóa dòng dữ liệu |

Các bảng chính được dùng trong tutorial:

| Bảng | Ý nghĩa |
|---|---|
| `customers` | Thông tin khách hàng |
| `orders` | Thông tin đơn hàng |
| `orderdetails` | Các dòng sản phẩm của đơn hàng |
| `payments` | Thông tin thanh toán |
| `products` | Thông tin sản phẩm |

Quan hệ dữ liệu:

```text
customers (1) ─────< orders (1) ─────< orderdetails >───── (1) products
     │
     └─────────────< payments
```

Các quan hệ này dẫn đến ba lưu ý:

- Muốn chèn `orders`, khách hàng tương ứng phải tồn tại trong `customers`.
- Muốn chèn `orderdetails`, đơn hàng và sản phẩm tương ứng phải tồn tại.
- Khi xóa dữ liệu, thường phải xóa bảng con trước bảng cha.

### Bài tập thực hành

**Bài 2.1.** Câu lệnh nào dùng để chèn dữ liệu mới?

**Bài 2.2.** Câu lệnh nào dùng để cập nhật dữ liệu đã có?

**Bài 2.3.** Câu lệnh nào dùng để xóa dữ liệu?

**Bài 2.4.** Bảng nào lưu thông tin khách hàng?

**Bài 2.5.** Bảng nào lưu các dòng sản phẩm thuộc một đơn hàng?

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
DESC payments;
DESC products;
```

Xem dữ liệu mẫu:

```sql
SELECT customerNumber, customerName, city, country, creditLimit
FROM customers
ORDER BY customerNumber
LIMIT 10;
```

```sql
SELECT orderNumber, orderDate, status, customerNumber
FROM orders
ORDER BY orderNumber
LIMIT 10;
```

```sql
SELECT orderNumber, productCode, quantityOrdered, priceEach, orderLineNumber
FROM orderdetails
ORDER BY orderNumber, orderLineNumber
LIMIT 10;
```

### Bài tập thực hành

**Bài 3.1.** Viết lệnh chọn cơ sở dữ liệu `classicmodels`.

**Bài 3.2.** Viết lệnh xem cấu trúc bảng `customers`.

**Bài 3.3.** Viết lệnh xem cấu trúc bảng `orders`.

**Bài 3.4.** Viết lệnh xem cấu trúc bảng `orderdetails`.

**Bài 3.5.** Viết lệnh xem cấu trúc bảng `payments`.

---

## 4. Quy tắc an toàn khi thao tác dữ liệu

### 4.1. Kiểm tra bằng `SELECT` trước khi `UPDATE` hoặc `DELETE`

```sql
SELECT customerNumber, customerName, creditLimit
FROM customers
WHERE customerNumber = 103;
```

Sau khi xác nhận đúng dữ liệu mới chạy lệnh cập nhật hoặc xóa.

### 4.2. Không quên `WHERE`

Sai:

```sql
UPDATE customers
SET creditLimit = 0;
```

Sai:

```sql
DELETE FROM payments;
```

Hai câu trên có thể ảnh hưởng đến tất cả dòng của bảng.

### 4.3. Dùng transaction khi thực hành

```sql
START TRANSACTION;

-- INSERT, UPDATE hoặc DELETE để thực hành

ROLLBACK;
```

| Câu lệnh | Ý nghĩa |
|---|---|
| `START TRANSACTION` | Bắt đầu giao dịch |
| `ROLLBACK` | Hủy thay đổi chưa được lưu |
| `COMMIT` | Lưu vĩnh viễn thay đổi |

> Trong giờ thực hành, mỗi nhóm lệnh nên kết thúc bằng `ROLLBACK`. Chỉ dùng `COMMIT` khi giảng viên yêu cầu lưu dữ liệu.

### 4.4. Mã thực hành cố định

Các ví dụ chèn dữ liệu dùng các mã sau:

| Dữ liệu | Mã minh họa |
|---|---|
| Khách hàng | `900001`, `900002`, `900003` |
| Đơn hàng | `990001`, `990002`, `990003` |
| Thanh toán | `LAB-900001-01`, `LAB-900002-01` |

Nếu một mã đã tồn tại trong database thực hành, hãy thay bằng mã khác. Tutorial không sử dụng biến SQL.

### Bài tập thực hành

**Bài 4.1.** Viết câu `SELECT` kiểm tra khách hàng mã `900001` đã tồn tại hay chưa.

**Bài 4.2.** Viết câu `SELECT` kiểm tra đơn hàng mã `990001` đã tồn tại hay chưa.

**Bài 4.3.** Nêu rủi ro của lệnh `UPDATE customers SET creditLimit = 0;`.

**Bài 4.4.** Viết khung transaction có `START TRANSACTION` và `ROLLBACK`.

**Bài 4.5.** Phân biệt `ROLLBACK` và `COMMIT`.

---

# Phần A. Chèn dữ liệu bằng `INSERT`

## 5. Cú pháp cơ bản của `INSERT`

`INSERT` dùng để chèn một dòng mới vào bảng.

```sql
INSERT INTO table_name (column1, column2, ...)
VALUES (value1, value2, ...);
```

| Thành phần | Ý nghĩa |
|---|---|
| `INSERT INTO` | Chỉ định bảng nhận dữ liệu |
| `table_name` | Tên bảng |
| Danh sách cột | Các cột được gán giá trị |
| `VALUES` | Danh sách giá trị mới |

Ví dụ chèn khách hàng:

```sql
INSERT INTO customers (
    customerNumber,
    customerName,
    contactLastName,
    contactFirstName,
    phone,
    addressLine1,
    city,
    country,
    creditLimit
)
VALUES (
    900001,
    'Lab Database Co.',
    'Nguyen',
    'An',
    '+84 24 1234 5678',
    '1 Dai Co Viet',
    'Hanoi',
    'Vietnam',
    50000.00
);
```

### Lưu ý

- Số giá trị phải bằng số cột.
- Giá trị phải theo đúng thứ tự cột đã ghi.
- Chuỗi và ngày tháng đặt trong dấu nháy đơn.
- `NULL` không đặt trong dấu nháy đơn.
- Nên ghi rõ tên cột để tránh phụ thuộc vào thứ tự cột của bảng.

### Bài tập thực hành

**Bài 5.1.** Viết khung lệnh `INSERT INTO ... VALUES ...` để chèn một khách hàng.

**Bài 5.2.** Chèn khách hàng mã `900001`, tên `Your Name Lab Co.`, thành phố `Hanoi`, quốc gia `Vietnam`, hạn mức `40000.00`.

**Bài 5.3.** Chèn khách hàng mã `900002`, tên `Second Lab Co.`, thành phố `Da Nang`, quốc gia `Vietnam`, hạn mức `30000.00`.

**Bài 5.4.** Chèn khách hàng mã `900003`, tên `Third Lab Co.`, thành phố `Ho Chi Minh City`, quốc gia `Vietnam`, hạn mức `35000.00`.

**Bài 5.5.** Thử chèn lại khách hàng có mã `900001`. Ghi lại lỗi và giải thích nguyên nhân.

---

## 6. Chèn một dòng vào bảng `customers`

Ví dụ hoàn chỉnh:

```sql
START TRANSACTION;

INSERT INTO customers (
    customerNumber,
    customerName,
    contactLastName,
    contactFirstName,
    phone,
    addressLine1,
    addressLine2,
    city,
    state,
    postalCode,
    country,
    salesRepEmployeeNumber,
    creditLimit
)
VALUES (
    900001,
    'Lab Database Co.',
    'Nguyen',
    'An',
    '+84 24 1234 5678',
    '1 Dai Co Viet',
    NULL,
    'Hanoi',
    NULL,
    '100000',
    'Vietnam',
    NULL,
    50000.00
);

ROLLBACK;
```

### Bài tập thực hành

**Bài 6.1.** Chèn khách hàng mã `900010`, tên `Lab Customer Ten`, thành phố `Hanoi`, quốc gia `Vietnam`, hạn mức `20000.00`.

**Bài 6.2.** Chèn khách hàng mã `900011`, tên `Lab Customer Eleven`, có `addressLine2 = 'Building A'`, thành phố `Da Nang`, hạn mức `25000.00`.

**Bài 6.3.** Chèn khách hàng mã `900012`, đặt `addressLine2`, `state`, `salesRepEmployeeNumber` là `NULL`.

**Bài 6.4.** Chèn khách hàng mã `900013`, có `postalCode = '700000'`, quốc gia `Vietnam`.

**Bài 6.5.** Sau mỗi câu chèn, dùng `SELECT ROW_COUNT()` để kiểm tra số dòng được chèn.

---

## 7. Chèn nhiều dòng trong một câu lệnh

```sql
INSERT INTO customers (
    customerNumber,
    customerName,
    contactLastName,
    contactFirstName,
    phone,
    addressLine1,
    city,
    country,
    creditLimit
)
VALUES
(
    900002,
    'Lab Trading One',
    'Tran',
    'Binh',
    '+84 24 1111 1111',
    '2 Nguyen Trai',
    'Hanoi',
    'Vietnam',
    30000.00
),
(
    900003,
    'Lab Trading Two',
    'Le',
    'Chi',
    '+84 28 2222 2222',
    '3 Le Loi',
    'Ho Chi Minh City',
    'Vietnam',
    35000.00
);
```

### Bài tập thực hành

**Bài 7.1.** Dùng một câu `INSERT` để chèn khách hàng `900020` và `900021`.

**Bài 7.2.** Dùng một câu `INSERT` để chèn khách hàng `900022`, `900023`, `900024`.

**Bài 7.3.** Ba khách hàng ở Bài 7.2 phải có tên công ty, thành phố và hạn mức tín dụng khác nhau.

**Bài 7.4.** Chèn hai khách hàng cùng quốc gia `Vietnam` nhưng thuộc hai thành phố khác nhau.

**Bài 7.5.** Chèn ba khách hàng, trong đó một khách hàng có `addressLine2 = NULL`.

---

## 8. Chèn đơn hàng vào bảng `orders`

Để chèn đơn hàng, khách hàng tương ứng phải tồn tại trước.

```sql
INSERT INTO orders (
    orderNumber,
    orderDate,
    requiredDate,
    shippedDate,
    status,
    comments,
    customerNumber
)
VALUES (
    990001,
    '2026-06-21',
    '2026-06-28',
    NULL,
    'In Process',
    'Order created for INSERT practice',
    900001
);
```

### Bài tập thực hành

**Bài 8.1.** Chèn đơn hàng mã `990001` cho khách hàng `900001`.

**Bài 8.2.** Chèn đơn hàng mã `990002` cho khách hàng `900002`, trạng thái `In Process`.

**Bài 8.3.** Chèn đơn hàng mã `990003` cho khách hàng `900003`, trạng thái `On Hold`, `shippedDate = NULL`.

**Bài 8.4.** Chèn đơn hàng mã `990004` cho khách hàng `900010`, ngày đặt `2026-06-22`, ngày yêu cầu `2026-06-30`.

**Bài 8.5.** Thử chèn đơn hàng có `customerNumber = 9999999`. Ghi lại lỗi và giải thích.

---

## 9. Chèn dữ liệu bằng `INSERT ... SELECT`

`INSERT ... SELECT` chèn dữ liệu lấy từ kết quả truy vấn.

```sql
INSERT INTO orderdetails (
    orderNumber,
    productCode,
    quantityOrdered,
    priceEach,
    orderLineNumber
)
SELECT
    990001,
    productCode,
    2,
    MSRP,
    1
FROM products
ORDER BY productCode
LIMIT 1;
```

Ví dụ chọn sản phẩm thuộc dòng `Motorcycles`:

```sql
INSERT INTO orderdetails (
    orderNumber,
    productCode,
    quantityOrdered,
    priceEach,
    orderLineNumber
)
SELECT
    990001,
    productCode,
    3,
    MSRP,
    2
FROM products
WHERE productLine = 'Motorcycles'
ORDER BY productCode
LIMIT 1;
```

### Bài tập thực hành

**Bài 9.1.** Dùng `INSERT ... SELECT` chèn một dòng `orderdetails` cho đơn hàng `990001`.

**Bài 9.2.** Chọn sản phẩm thuộc dòng `Motorcycles`; đặt `quantityOrdered = 3`, `priceEach = MSRP`, `orderLineNumber = 1`.

**Bài 9.3.** Chọn sản phẩm thuộc dòng `Classic Cars`, chèn vào đơn hàng `990001` với `orderLineNumber = 2`.

**Bài 9.4.** Chọn sản phẩm thuộc dòng `Planes`, chèn vào đơn hàng `990002` với `quantityOrdered = 4`.

**Bài 9.5.** Chọn một sản phẩm bất kỳ, chèn vào đơn hàng `990003` với `quantityOrdered = 1`, `orderLineNumber = 1`.

---

## 10. Chèn dữ liệu vào bảng `payments`

Bảng `payments` thường có khóa chính ghép:

```text
(customerNumber, checkNumber)
```

```sql
INSERT INTO payments (
    customerNumber,
    checkNumber,
    paymentDate,
    amount
)
VALUES (
    900001,
    'LAB-900001-01',
    '2026-06-21',
    1500.00
);
```

### Bài tập thực hành

**Bài 10.1.** Chèn thanh toán cho khách hàng `900001`, mã séc `LAB-900001-01`, ngày `2026-06-21`, số tiền `1500.00`.

**Bài 10.2.** Chèn thanh toán cho khách hàng `900002`, mã séc `LAB-900002-01`, ngày `2026-06-22`, số tiền `2000.00`.

**Bài 10.3.** Chèn thanh toán cho khách hàng `900003`, mã séc `LAB-900003-01`, ngày `2026-06-23`, số tiền `1750.00`.

**Bài 10.4.** Thử chèn lại cặp khóa `(900001, 'LAB-900001-01')`. Giải thích lỗi.

**Bài 10.5.** Thử chèn thanh toán cho khách hàng `9999999`. Giải thích lỗi.

---

## 11. Lỗi thường gặp với `INSERT`

### Lỗi 1: Trùng khóa chính

```sql
INSERT INTO customers (
    customerNumber,
    customerName,
    contactLastName,
    contactFirstName,
    phone,
    addressLine1,
    city,
    country,
    creditLimit
)
VALUES (
    900001,
    'Another Lab Co.',
    'A',
    'B',
    '000',
    'Unknown',
    'Hanoi',
    'Vietnam',
    1000.00
);
```

### Lỗi 2: Số cột và số giá trị không khớp

Sai:

```sql
INSERT INTO customers (customerNumber, customerName, city)
VALUES (900010, 'Test Customer');
```

Đúng:

```sql
INSERT INTO customers (customerNumber, customerName, city)
VALUES (900010, 'Test Customer', 'Hanoi');
```

### Lỗi 3: Vi phạm khóa ngoại

```sql
INSERT INTO orders (
    orderNumber, orderDate, requiredDate, status, customerNumber
)
VALUES (
    990010, '2026-06-21', '2026-06-28', 'In Process', 9999999
);
```

### Bài tập thực hành

**Bài 11.1.** Sửa câu `INSERT` bị thiếu giá trị trong phần `VALUES`.

**Bài 11.2.** Viết một câu `INSERT` gây lỗi trùng khóa chính trên `customers`.

**Bài 11.3.** Viết một câu `INSERT` gây lỗi khóa ngoại trên `orders`.

**Bài 11.4.** Viết một câu `INSERT` có một cột nhận giá trị `NULL`.

**Bài 11.5.** Giải thích lợi ích của việc ghi rõ tên các cột khi chèn dữ liệu.

---

# Phần B. Cập nhật dữ liệu bằng `UPDATE`

## 12. Cú pháp cơ bản của `UPDATE`

`UPDATE` thay đổi dữ liệu của các dòng đã tồn tại.

```sql
UPDATE table_name
SET column1 = value1,
    column2 = value2,
    ...
WHERE condition;
```

Ví dụ:

```sql
UPDATE customers
SET creditLimit = 60000.00
WHERE customerNumber = 103;
```

Trong giờ thực hành:

```sql
START TRANSACTION;

UPDATE customers
SET creditLimit = 60000.00
WHERE customerNumber = 103;

ROLLBACK;
```

### Bài tập thực hành

**Bài 12.1.** Cập nhật `creditLimit` của khách hàng `103` thành `60000.00`.

**Bài 12.2.** Cập nhật thành phố của khách hàng `103` thành `Hanoi`.

**Bài 12.3.** Cập nhật số điện thoại của khách hàng `103` thành `+84 24 9999 9999`.

**Bài 12.4.** Cập nhật `creditLimit` của khách hàng `112` thành `80000.00`.

**Bài 12.5.** Sau mỗi câu `UPDATE`, dùng `SELECT ROW_COUNT()` để kiểm tra số dòng được cập nhật.

---

## 13. Cập nhật bằng biểu thức

Có thể dùng giá trị cũ để tạo giá trị mới.

```sql
UPDATE customers
SET creditLimit = creditLimit + 5000.00
WHERE customerNumber = 103;
```

```sql
UPDATE customers
SET creditLimit = creditLimit * 1.10
WHERE customerNumber = 112;
```

```sql
UPDATE orderdetails
SET quantityOrdered = quantityOrdered + 2
WHERE orderNumber = 10100
  AND productCode = 'S18_1749';
```

### Bài tập thực hành

**Bài 13.1.** Tăng `creditLimit` của khách hàng `103` thêm `5000.00`.

**Bài 13.2.** Tăng `creditLimit` của khách hàng `112` thêm 10%.

**Bài 13.3.** Tăng `quantityOrdered` của dòng `(10100, 'S18_1749')` thêm 1.

**Bài 13.4.** Giảm `priceEach` của dòng `(10100, 'S18_1749')` đi 5%.

**Bài 13.5.** Tăng `amount` của khoản thanh toán `(103, 'HQ336336')` thêm `100.00`.

---

## 14. Cập nhật nhiều cột trong một câu lệnh

```sql
UPDATE customers
SET city = 'Da Nang',
    phone = '+84 236 123 4567',
    creditLimit = 65000.00
WHERE customerNumber = 103;
```

```sql
UPDATE orders
SET status = 'On Hold',
    comments = 'Waiting for confirmation'
WHERE orderNumber = 10100;
```

### Bài tập thực hành

**Bài 14.1.** Cập nhật khách hàng `103`: thành phố `Da Nang`, số điện thoại `+84 236 123 4567`.

**Bài 14.2.** Cập nhật khách hàng `112`: quốc gia `Vietnam`, thành phố `Ho Chi Minh City`, hạn mức tín dụng `70000.00`.

**Bài 14.3.** Cập nhật đơn hàng `10100`: trạng thái `On Hold`, ghi chú `Waiting for confirmation`.

**Bài 14.4.** Cập nhật đơn hàng `10101`: trạng thái `In Process`, `shippedDate = NULL`.

**Bài 14.5.** Cập nhật thanh toán `(103, 'HQ336336')`: ngày thanh toán `2026-06-21`, số tiền `6500.00`.

---

## 15. Cập nhật có điều kiện bằng `CASE`

```sql
UPDATE orderdetails
SET priceEach = CASE
    WHEN quantityOrdered >= 30 THEN priceEach * 0.95
    ELSE priceEach
END
WHERE orderNumber = 10100
  AND productCode = 'S18_1749';
```

```sql
UPDATE orders
SET comments = CASE
    WHEN status = 'In Process' THEN 'Order is being processed'
    WHEN status = 'On Hold' THEN 'Order is waiting for confirmation'
    ELSE comments
END
WHERE orderNumber = 10100;
```

### Bài tập thực hành

**Bài 15.1.** Giảm `priceEach` của dòng `(10100, 'S18_1749')` đi 5% nếu `quantityOrdered >= 30`.

**Bài 15.2.** Cập nhật `comments` của đơn hàng `10100`: nếu trạng thái là `On Hold` thì ghi `Order is waiting for confirmation`, ngược lại giữ nguyên.

**Bài 15.3.** Cập nhật `creditLimit` của khách hàng `103`: nếu nhỏ hơn `50000.00` thì tăng thêm `10000.00`, ngược lại giữ nguyên.

**Bài 15.4.** Cập nhật `status` của đơn hàng `10101`: nếu `shippedDate IS NULL` thì đặt `On Hold`, ngược lại giữ nguyên trạng thái.

**Bài 15.5.** Giải thích vai trò của nhánh `ELSE` trong biểu thức `CASE`.

---

## 16. Cập nhật giá trị `NULL`

```sql
UPDATE orders
SET shippedDate = NULL
WHERE orderNumber = 10100;
```

Không viết:

```sql
UPDATE orders
SET shippedDate = 'NULL'
WHERE orderNumber = 10100;
```

```sql
UPDATE orders
SET shippedDate = '2026-06-25'
WHERE orderNumber = 10101;
```

### Bài tập thực hành

**Bài 16.1.** Cập nhật `shippedDate` của đơn hàng `10100` thành `NULL`.

**Bài 16.2.** Cập nhật `shippedDate` của đơn hàng `10101` thành `2026-06-25`.

**Bài 16.3.** Cập nhật `state` của khách hàng `103` thành `NULL`.

**Bài 16.4.** Cập nhật `salesRepEmployeeNumber` của khách hàng `103` thành `NULL`.

**Bài 16.5.** Giải thích sự khác nhau giữa `NULL` và `'NULL'`.

---

## 17. Lỗi thường gặp với `UPDATE`

### Lỗi 1: Thiếu `WHERE`

```sql
UPDATE customers
SET creditLimit = 0;
```

### Lỗi 2: Điều kiện quá rộng

```sql
UPDATE customers
SET creditLimit = creditLimit * 1.05
WHERE country = 'USA';
```

### Lỗi 3: Vi phạm khóa ngoại

```sql
UPDATE orders
SET customerNumber = 9999999
WHERE orderNumber = 10100;
```

### Bài tập thực hành

**Bài 17.1.** Sửa lệnh sau để chỉ cập nhật khách hàng `103`:

```sql
UPDATE customers
SET creditLimit = 0;
```

**Bài 17.2.** Viết câu `SELECT` cần chạy trước khi cập nhật khách hàng ở `USA`.

**Bài 17.3.** Viết câu `UPDATE` có thể gây lỗi khóa ngoại trên bảng `orders`.

**Bài 17.4.** Giải thích vì sao `WHERE country = 'USA'` có thể vẫn quá rộng.

**Bài 17.5.** Viết một câu `UPDATE` chỉ thay đổi một khách hàng bằng khóa chính.

---

# Phần C. Xóa dữ liệu bằng `DELETE`

## 18. Cú pháp cơ bản của `DELETE`

```sql
DELETE FROM table_name
WHERE condition;
```

Ví dụ:

```sql
DELETE FROM payments
WHERE customerNumber = 103
  AND checkNumber = 'HQ336336';
```

Trong giờ thực hành:

```sql
START TRANSACTION;

DELETE FROM payments
WHERE customerNumber = 103
  AND checkNumber = 'HQ336336';

ROLLBACK;
```

### Bài tập thực hành

**Bài 18.1.** Xóa khoản thanh toán `(103, 'HQ336336')`.

**Bài 18.2.** Xóa khoản thanh toán `(103, 'JM555205')`.

**Bài 18.3.** Xóa khoản thanh toán `(103, 'OM314933')`.

**Bài 18.4.** Xóa các khoản thanh toán của khách hàng `103` có số tiền nhỏ hơn `5000.00`.

**Bài 18.5.** Xóa các khoản thanh toán của khách hàng `103` có ngày thanh toán trước `2004-01-01`.

---

## 19. Xóa dữ liệu bằng khóa chính ghép

Bảng `payments` có thể được xác định bởi cặp `(customerNumber, checkNumber)`.

```sql
DELETE FROM payments
WHERE customerNumber = 103
  AND checkNumber = 'HQ336336';
```

Bảng `orderdetails` có khóa chính ghép `(orderNumber, productCode)`.

```sql
DELETE FROM orderdetails
WHERE orderNumber = 10100
  AND productCode = 'S18_1749';
```

### Bài tập thực hành

**Bài 19.1.** Xóa dòng `orderdetails` có `(orderNumber, productCode) = (10100, 'S18_1749')`.

**Bài 19.2.** Xóa tất cả dòng `orderdetails` có `orderNumber = 10100`.

**Bài 19.3.** Xóa các dòng `orderdetails` có `orderNumber = 10100` và `quantityOrdered < 30`.

**Bài 19.4.** Xóa các dòng `orderdetails` có `orderNumber = 10101` và `priceEach < 100.00`.

**Bài 19.5.** Giải thích vì sao dùng cả `orderNumber` và `productCode` thường an toàn hơn chỉ dùng `orderNumber`.

---

## 20. Xóa dữ liệu và khóa ngoại

Không nên xóa khách hàng trước khi xử lý đơn hàng và thanh toán liên quan.

```sql
DELETE FROM customers
WHERE customerNumber = 103;
```

Nếu còn dữ liệu con tham chiếu đến khách hàng này, DBMS có thể từ chối lệnh.

Thứ tự xóa thông thường:

```text
payments → orderdetails → orders → customers
```

Ví dụ với dữ liệu thực hành:

```sql
DELETE FROM payments
WHERE customerNumber = 900001;

DELETE FROM orderdetails
WHERE orderNumber = 990001;

DELETE FROM orders
WHERE orderNumber = 990001;

DELETE FROM customers
WHERE customerNumber = 900001;
```

### Bài tập thực hành

**Bài 20.1.** Giải thích vì sao xóa khách hàng `103` có thể gây lỗi.

**Bài 20.2.** Viết câu `DELETE` xóa các dòng `orderdetails` của đơn hàng `10100`.

**Bài 20.3.** Viết câu `DELETE` xóa đơn hàng `10100` sau khi các dòng `orderdetails` liên quan đã được xóa.

**Bài 20.4.** Sắp xếp thứ tự xóa đúng cho bốn bảng: `customers`, `orders`, `orderdetails`, `payments`.

**Bài 20.5.** Viết các câu `DELETE` theo thứ tự an toàn để xóa khách hàng `900001`, đơn hàng `990001`, các dòng chi tiết và thanh toán liên quan.

---

## 21. `DELETE`, `TRUNCATE` và `DROP`

| Câu lệnh | Tác dụng | Xóa cấu trúc bảng? | Dùng `WHERE`? |
|---|---|---:|---:|
| `DELETE` | Xóa một số hoặc toàn bộ dòng | Không | Có |
| `TRUNCATE TABLE` | Xóa nhanh toàn bộ dòng | Không | Không |
| `DROP TABLE` | Xóa cả bảng và dữ liệu | Có | Không |

Không dùng trong database thực hành dùng chung:

```sql
TRUNCATE TABLE customers;
```

```sql
DROP TABLE customers;
```

### Bài tập thực hành

**Bài 21.1.** Phân biệt `DELETE` và `TRUNCATE TABLE`.

**Bài 21.2.** Phân biệt `DELETE` và `DROP TABLE`.

**Bài 21.3.** Trong ba lệnh `DELETE`, `TRUNCATE TABLE`, `DROP TABLE`, lệnh nào dùng được `WHERE`?

**Bài 21.4.** Lệnh nào xóa cả cấu trúc bảng?

**Bài 21.5.** Vì sao không nên chạy `TRUNCATE TABLE payments` trong database dùng chung?

---

## 22. Lỗi thường gặp với `DELETE`

### Lỗi 1: Thiếu `WHERE`

```sql
DELETE FROM payments;
```

### Lỗi 2: Điều kiện quá rộng

```sql
DELETE FROM payments
WHERE customerNumber = 103;
```

### Lỗi 3: Xóa bảng cha trước bảng con

```sql
DELETE FROM customers
WHERE customerNumber = 103;
```

### Bài tập thực hành

**Bài 22.1.** Sửa câu sau để chỉ xóa khoản thanh toán `HQ336336` của khách hàng `103`:

```sql
DELETE FROM payments
WHERE customerNumber = 103;
```

**Bài 22.2.** Viết câu `SELECT` cần chạy trước khi xóa các dòng `orderdetails` của đơn hàng `10100`.

**Bài 22.3.** Giải thích rủi ro của lệnh `DELETE FROM orderdetails WHERE orderNumber = 10100;`.

**Bài 22.4.** Giải thích vì sao không nên xóa khách hàng trước khi xóa đơn hàng của khách hàng đó.

**Bài 22.5.** Viết một câu `DELETE` chỉ xóa một dòng `orderdetails` bằng khóa chính ghép.

---

## 23. Đáp án gợi ý cho một số bài tập

### Bài 5.2

```sql
INSERT INTO customers (
    customerNumber,
    customerName,
    contactLastName,
    contactFirstName,
    phone,
    addressLine1,
    city,
    country,
    creditLimit
)
VALUES (
    900001,
    'Your Name Lab Co.',
    'Nguyen',
    'An',
    '+84 24 1234 5678',
    '1 Dai Co Viet',
    'Hanoi',
    'Vietnam',
    40000.00
);
```

### Bài 8.1

```sql
INSERT INTO orders (
    orderNumber,
    orderDate,
    requiredDate,
    shippedDate,
    status,
    comments,
    customerNumber
)
VALUES (
    990001,
    '2026-06-21',
    '2026-06-28',
    NULL,
    'In Process',
    'Created for INSERT practice',
    900001
);
```

### Bài 9.2

```sql
INSERT INTO orderdetails (
    orderNumber,
    productCode,
    quantityOrdered,
    priceEach,
    orderLineNumber
)
SELECT
    990001,
    productCode,
    3,
    MSRP,
    1
FROM products
WHERE productLine = 'Motorcycles'
ORDER BY productCode
LIMIT 1;
```

### Bài 10.1

```sql
INSERT INTO payments (
    customerNumber,
    checkNumber,
    paymentDate,
    amount
)
VALUES (
    900001,
    'LAB-900001-01',
    '2026-06-21',
    1500.00
);
```

### Bài 12.1

```sql
UPDATE customers
SET creditLimit = 60000.00
WHERE customerNumber = 103;
```

### Bài 13.1

```sql
UPDATE customers
SET creditLimit = creditLimit + 5000.00
WHERE customerNumber = 103;
```

### Bài 14.3

```sql
UPDATE orders
SET status = 'On Hold',
    comments = 'Waiting for confirmation'
WHERE orderNumber = 10100;
```

### Bài 15.1

```sql
UPDATE orderdetails
SET priceEach = CASE
    WHEN quantityOrdered >= 30 THEN priceEach * 0.95
    ELSE priceEach
END
WHERE orderNumber = 10100
  AND productCode = 'S18_1749';
```

### Bài 18.1

```sql
DELETE FROM payments
WHERE customerNumber = 103
  AND checkNumber = 'HQ336336';
```

### Bài 19.1

```sql
DELETE FROM orderdetails
WHERE orderNumber = 10100
  AND productCode = 'S18_1749';
```

### Bài 20.5

```sql
DELETE FROM payments
WHERE customerNumber = 900001;

DELETE FROM orderdetails
WHERE orderNumber = 990001;

DELETE FROM orders
WHERE orderNumber = 990001;

DELETE FROM customers
WHERE customerNumber = 900001;
```

---

## 24. Tóm tắt

- `INSERT` dùng để chèn dữ liệu mới.
- `INSERT ... SELECT` dùng để chèn dữ liệu lấy từ truy vấn.
- `UPDATE` dùng để thay đổi dữ liệu đã tồn tại.
- `DELETE` dùng để xóa dữ liệu theo điều kiện.
- `WHERE` đặc biệt quan trọng với `UPDATE` và `DELETE`.
- Khóa chính giúp tránh dữ liệu trùng lặp.
- Khóa ngoại bảo đảm dữ liệu tham chiếu hợp lệ.
- Khi thực hành, nên dùng `START TRANSACTION` và `ROLLBACK`.
- Bài tập trong phần `INSERT` chỉ yêu cầu thao tác chèn; bài tập trong phần `UPDATE` chỉ yêu cầu thao tác cập nhật; bài tập trong phần `DELETE` chỉ yêu cầu thao tác xóa.
- Tutorial không dùng biến SQL dạng `@...`.

---

## 25. Từ khóa chính

- SQL
- DML
- INSERT
- INSERT ... SELECT
- UPDATE
- DELETE
- VALUES
- SET
- WHERE
- CASE
- NULL
- Primary Key
- Foreign Key
- Referential Integrity
- START TRANSACTION
- COMMIT
- ROLLBACK
- ROW_COUNT()
- classicmodels
