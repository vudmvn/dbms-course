---
title: "Lab: Subquery với EXISTS, NOT EXISTS, ALL và ANY trong classicmodels"
author: "Tên giảng viên"
duration: "120 phút"
difficulty: "Intermediate"
prerequisites:
  - "Đã biết SELECT, WHERE, ORDER BY, DISTINCT và JOIN cơ bản"
  - "Đã biết các bảng customers, orders, orderdetails, products, payments, employees trong classicmodels"
  - "Đã biết khái niệm khóa chính và khóa ngoại"
summary: "Thực hành subquery trong MySQL với EXISTS, NOT EXISTS, ALL, ANY/SOME trên cơ sở dữ liệu mẫu classicmodels."
---

# Lab: Subquery với `EXISTS`, `NOT EXISTS`, `ALL` và `ANY` trong `classicmodels`

## Link tham khảo

- [GeeksforGeeks: SQL Subquery](https://www.geeksforgeeks.org/sql/sql-subquery/)
- [GeeksforGeeks: SQL EXISTS](https://www.geeksforgeeks.org/sql/sql-exists/)
- [GeeksforGeeks: SQL IN Operator](https://www.geeksforgeeks.org/sql/sql-in-operator/)
- [GeeksforGeeks: SQL Subquery with ANY and ALL](https://www.geeksforgeeks.org/sql/sql-subquery/)
- [GeeksforGeeks: SQL Correlated Subqueries](https://www.geeksforgeeks.org/sql/sql-correlated-subqueries/)

---

## 1. Mục tiêu học tập

Sau khi hoàn thành lab này, người học có thể:

1. Giải thích được khái niệm **subquery** và phân biệt subquery tương quan với subquery không tương quan.
2. Dùng `EXISTS` để kiểm tra sự tồn tại của các dòng thỏa điều kiện trong một bảng liên quan.
3. Dùng `NOT EXISTS` để tìm các đối tượng **không có** bản ghi liên quan.
4. Dùng `ALL` để so sánh một giá trị với **mọi giá trị** do subquery trả về.
5. Dùng `ANY` hoặc `SOME` để so sánh một giá trị với **ít nhất một giá trị** do subquery trả về.
6. Phân biệt ý nghĩa của `> ALL (...)` và `> ANY (...)`.
7. Nhận biết ảnh hưởng của tập rỗng và `NULL` khi dùng `ALL`, `ANY`, `NOT IN` và `NOT EXISTS`.
8. Viết được các truy vấn thực tế trên cơ sở dữ liệu `classicmodels`.

---

## 2. Giới thiệu các bảng sử dụng trong `classicmodels`

Trong lab này, các bảng được sử dụng chủ yếu là:

| Bảng | Khóa/chỉ mục liên quan | Ý nghĩa trong lab |
|---|---|---|
| `customers` | `customerNumber` | Khách hàng; có thể có đơn hàng và thanh toán |
| `orders` | `orderNumber`, `customerNumber` | Đơn hàng của khách hàng |
| `orderdetails` | `orderNumber`, `productCode` | Các dòng sản phẩm trong từng đơn hàng |
| `products` | `productCode`, `productLine` | Thông tin sản phẩm và giá |
| `payments` | `customerNumber`, `checkNumber` | Các khoản thanh toán của khách hàng |
| `employees` | `employeeNumber` | Nhân viên; một số nhân viên phụ trách khách hàng |
| `offices` | `officeCode` | Văn phòng; có thể có nhân viên làm việc |

### Một số quan hệ quan trọng

```text
customers 1 --- n orders
customers 1 --- n payments
orders    1 --- n orderdetails
products  1 --- n orderdetails
employees 1 --- n customers
offices   1 --- n employees
```

### Bài tập thực hành

**Bài 2.1.** Viết lệnh xem cấu trúc của các bảng `customers`, `orders`, `orderdetails`, `products` và `payments`.

**Bài 2.2.** Xác định các cặp cột thể hiện quan hệ giữa `customers` và `orders`.

**Bài 2.3.** Xác định các cặp cột thể hiện quan hệ giữa `orders`, `orderdetails` và `products`.

**Bài 2.4.** Xác định cột cho biết nhân viên bán hàng phụ trách một khách hàng.

**Bài 2.5.** Vẽ nhanh sơ đồ quan hệ giữa `customers`, `orders`, `orderdetails`, `products` và `payments`.

---

## 3. Chuẩn bị môi trường

Chọn cơ sở dữ liệu:

```sql
USE classicmodels;
```

Kiểm tra các bảng:

```sql
SHOW TABLES;
```

Xem cấu trúc bảng:

```sql
DESC customers;
DESC orders;
DESC orderdetails;
DESC products;
DESC payments;
```

### Bài tập thực hành

**Bài 3.1.** Viết lệnh chọn cơ sở dữ liệu `classicmodels`.

**Bài 3.2.** Viết lệnh xem danh sách các bảng trong cơ sở dữ liệu hiện tại.

**Bài 3.3.** Viết lệnh xem cấu trúc bảng `orders` và xác định cột liên kết với bảng `customers`.

**Bài 3.4.** Viết lệnh xem cấu trúc bảng `orderdetails` và xác định khóa chính gồm những cột nào.

**Bài 3.5.** Viết lệnh xem cấu trúc bảng `payments` và xác định các cột dùng để nhận diện một khoản thanh toán.

---

## 4. Khái niệm subquery

**Subquery** là một câu lệnh `SELECT` nằm bên trong một câu lệnh SQL khác. Kết quả của subquery được câu lệnh bên ngoài sử dụng để lọc dữ liệu, so sánh dữ liệu hoặc kiểm tra sự tồn tại.

Ví dụ: tìm các sản phẩm có giá bán đề xuất lớn hơn giá bán đề xuất trung bình của tất cả sản phẩm.

```sql
SELECT productCode, productName, MSRP
FROM products
WHERE MSRP > (
    SELECT AVG(MSRP)
    FROM products
);
```

Trong lab này, subquery thường nằm trong mệnh đề `WHERE`.

### 4.1. Subquery không tương quan

Subquery **không tương quan** không tham chiếu tới bảng ở câu lệnh ngoài. Về mặt khái niệm, nó có thể được tính độc lập với câu lệnh ngoài.

Ví dụ:

```sql
SELECT productCode, productName, MSRP
FROM products
WHERE MSRP > ALL (
    SELECT MSRP
    FROM products
    WHERE productLine = 'Motorcycles'
);
```

Subquery bên trong chỉ phụ thuộc vào bảng `products` với điều kiện cố định `productLine = 'Motorcycles'`.

### 4.2. Subquery tương quan

Subquery **tương quan** có tham chiếu tới một bảng hoặc alias của câu lệnh ngoài. Về mặt logic, subquery được xét theo từng dòng của câu lệnh ngoài.

Ví dụ: tìm khách hàng đã có ít nhất một đơn hàng.

```sql
SELECT c.customerNumber, c.customerName
FROM customers AS c
WHERE EXISTS (
    SELECT 1
    FROM orders AS o
    WHERE o.customerNumber = c.customerNumber
);
```

Trong đó, `c.customerNumber` là giá trị đến từ dòng khách hàng đang xét ở câu lệnh ngoài.

### Bài tập thực hành

**Bài 4.1.** Trong ví dụ tìm khách hàng có đơn hàng, hãy chỉ ra bảng ở câu lệnh ngoài và bảng ở subquery.

**Bài 4.2.** Giải thích vì sao subquery trong ví dụ trên là subquery tương quan.

**Bài 4.3.** Cho truy vấn sau. Hãy xác định đây là subquery tương quan hay không tương quan.

```sql
SELECT productCode, productName, MSRP
FROM products
WHERE MSRP < (
    SELECT AVG(MSRP)
    FROM products
);
```

**Bài 4.4.** Viết một truy vấn dùng subquery để lấy các khách hàng có `creditLimit` lớn hơn hạn mức tín dụng trung bình của tất cả khách hàng có giá trị không rỗng.

**Bài 4.5.** Viết một truy vấn dùng subquery để lấy các khoản thanh toán có `amount` lớn hơn số tiền thanh toán trung bình.

---

## 5. Mệnh đề `EXISTS`

### 5.1. Ý nghĩa

`EXISTS` kiểm tra xem subquery có trả về **ít nhất một dòng** hay không.

- Nếu subquery trả về ít nhất một dòng, `EXISTS (...)` có giá trị đúng.
- Nếu subquery không trả về dòng nào, `EXISTS (...)` có giá trị sai.

Cú pháp tổng quát:

```sql
SELECT column_list
FROM outer_table AS t1
WHERE EXISTS (
    SELECT 1
    FROM inner_table AS t2
    WHERE condition_uses_t1_and_t2
);
```

Trong subquery của `EXISTS`, danh sách sau `SELECT` không quyết định kết quả. Ta thường viết `SELECT 1` để thể hiện rõ rằng chỉ cần quan tâm đến **sự tồn tại của dòng**, không cần lấy dữ liệu cụ thể.

---

### 5.2. Ví dụ 1: Khách hàng đã từng đặt hàng

```sql
SELECT c.customerNumber, c.customerName
FROM customers AS c
WHERE EXISTS (
    SELECT 1
    FROM orders AS o
    WHERE o.customerNumber = c.customerNumber
)
ORDER BY c.customerNumber;
```

Ý nghĩa: với mỗi khách hàng `c`, truy vấn kiểm tra xem bảng `orders` có ít nhất một đơn hàng có cùng `customerNumber` hay không.

---

### 5.3. Ví dụ 2: Khách hàng đã từng thanh toán

```sql
SELECT c.customerNumber, c.customerName
FROM customers AS c
WHERE EXISTS (
    SELECT 1
    FROM payments AS p
    WHERE p.customerNumber = c.customerNumber
)
ORDER BY c.customerName;
```

---

### 5.4. Ví dụ 3: Nhân viên phụ trách ít nhất một khách hàng

```sql
SELECT e.employeeNumber, e.lastName, e.firstName, e.jobTitle
FROM employees AS e
WHERE EXISTS (
    SELECT 1
    FROM customers AS c
    WHERE c.salesRepEmployeeNumber = e.employeeNumber
)
ORDER BY e.employeeNumber;
```

---

### 5.5. Ví dụ 4: Sản phẩm đã xuất hiện trong ít nhất một dòng chi tiết đơn hàng

```sql
SELECT p.productCode, p.productName, p.productLine
FROM products AS p
WHERE EXISTS (
    SELECT 1
    FROM orderdetails AS od
    WHERE od.productCode = p.productCode
)
ORDER BY p.productLine, p.productName;
```

---

### 5.6. Ví dụ 5: Đơn hàng có ít nhất một dòng sản phẩm đặt từ 50 đơn vị trở lên

```sql
SELECT o.orderNumber, o.orderDate, o.status
FROM orders AS o
WHERE EXISTS (
    SELECT 1
    FROM orderdetails AS od
    WHERE od.orderNumber = o.orderNumber
      AND od.quantityOrdered >= 50
)
ORDER BY o.orderNumber;
```

### Bài tập thực hành

**Bài 5.1.** Liệt kê mã và tên các khách hàng có ít nhất một đơn hàng.

**Bài 5.2.** Liệt kê mã và tên các khách hàng có ít nhất một khoản thanh toán.

**Bài 5.3.** Liệt kê các văn phòng có ít nhất một nhân viên đang làm việc.

**Bài 5.4.** Liệt kê các sản phẩm đã từng được đặt trong `orderdetails`.

**Bài 5.5.** Liệt kê các đơn hàng có ít nhất một dòng chi tiết với `quantityOrdered > 80`.

**Bài 5.6.** Liệt kê các khách hàng đã có ít nhất một đơn hàng mang trạng thái `Shipped`.

**Bài 5.7.** Liệt kê các sản phẩm đã từng xuất hiện trong một đơn hàng có trạng thái `On Hold`.

---

## 6. Mệnh đề `NOT EXISTS`

### 6.1. Ý nghĩa

`NOT EXISTS` kiểm tra rằng subquery **không trả về dòng nào**.

Cú pháp tổng quát:

```sql
SELECT column_list
FROM outer_table AS t1
WHERE NOT EXISTS (
    SELECT 1
    FROM inner_table AS t2
    WHERE condition_uses_t1_and_t2
);
```

`NOT EXISTS` đặc biệt phù hợp với những câu hỏi dạng:

- Khách hàng nào chưa từng đặt hàng?
- Sản phẩm nào chưa từng được bán?
- Nhân viên nào chưa được gán khách hàng?
- Văn phòng nào chưa có nhân viên?

---

### 6.2. Ví dụ 1: Khách hàng chưa từng đặt hàng

```sql
SELECT c.customerNumber, c.customerName
FROM customers AS c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders AS o
    WHERE o.customerNumber = c.customerNumber
)
ORDER BY c.customerNumber;
```

---

### 6.3. Ví dụ 2: Sản phẩm chưa từng xuất hiện trong chi tiết đơn hàng

```sql
SELECT p.productCode, p.productName, p.productLine
FROM products AS p
WHERE NOT EXISTS (
    SELECT 1
    FROM orderdetails AS od
    WHERE od.productCode = p.productCode
)
ORDER BY p.productCode;
```

---

### 6.4. Ví dụ 3: Nhân viên không phụ trách khách hàng nào

```sql
SELECT e.employeeNumber, e.lastName, e.firstName, e.jobTitle
FROM employees AS e
WHERE NOT EXISTS (
    SELECT 1
    FROM customers AS c
    WHERE c.salesRepEmployeeNumber = e.employeeNumber
)
ORDER BY e.employeeNumber;
```

---

### 6.5. `NOT EXISTS` và `NOT IN`

Với dữ liệu thực tế, `NOT EXISTS` thường an toàn hơn `NOT IN (subquery)` khi subquery có thể chứa `NULL`.

Ví dụ có rủi ro:

```sql
SELECT c.customerNumber, c.customerName
FROM customers AS c
WHERE c.customerNumber NOT IN (
    SELECT o.customerNumber
    FROM orders AS o
);
```

Nếu subquery có một giá trị `NULL`, biểu thức `NOT IN` có thể cho kết quả không như mong đợi do logic ba giá trị của SQL. Với yêu cầu kiểm tra “không có bản ghi liên quan”, ưu tiên mẫu sau:

```sql
SELECT c.customerNumber, c.customerName
FROM customers AS c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders AS o
    WHERE o.customerNumber = c.customerNumber
);
```

### Bài tập thực hành

**Bài 6.1.** Liệt kê các khách hàng chưa từng đặt hàng.

**Bài 6.2.** Liệt kê các khách hàng chưa từng có khoản thanh toán.

**Bài 6.3.** Liệt kê các sản phẩm chưa từng xuất hiện trong `orderdetails`.

**Bài 6.4.** Liệt kê các nhân viên chưa phụ trách khách hàng nào.

**Bài 6.5.** Liệt kê các văn phòng không có nhân viên nào.

**Bài 6.6.** Liệt kê các khách hàng chưa có đơn hàng có trạng thái `Shipped`.

**Bài 6.7.** Liệt kê các sản phẩm chưa từng xuất hiện trong đơn hàng có trạng thái `Shipped`.

---

## 7. Mệnh đề `ALL`

### 7.1. Ý nghĩa

`ALL` được dùng sau một toán tử so sánh và yêu cầu phép so sánh phải đúng với **mọi giá trị** do subquery trả về.

Cú pháp:

```sql
expression comparison_operator ALL (subquery)
```

Các toán tử so sánh thường dùng:

```text
=   >   <   >=   <=   <>   !=
```

Ví dụ:

```sql
x > ALL (subquery)
```

có nghĩa là: `x` phải lớn hơn **mọi giá trị** được trả về bởi subquery.

Khi subquery không rỗng và không có `NULL`, có thể hiểu nhanh:

| Biểu thức | Cách hiểu gần đúng |
|---|---|
| `x > ALL (S)` | `x` lớn hơn giá trị lớn nhất trong `S` |
| `x >= ALL (S)` | `x` lớn hơn hoặc bằng giá trị lớn nhất trong `S` |
| `x < ALL (S)` | `x` nhỏ hơn giá trị nhỏ nhất trong `S` |
| `x <= ALL (S)` | `x` nhỏ hơn hoặc bằng giá trị nhỏ nhất trong `S` |

> Lưu ý: cách hiểu qua `MAX` và `MIN` cần thận trọng nếu tập kết quả có `NULL` hoặc rỗng. Phần 9 sẽ giải thích chi tiết hơn.

---

### 7.2. Ví dụ 1: Sản phẩm có MSRP cao hơn mọi sản phẩm thuộc dòng Motorcycles

```sql
SELECT p.productCode, p.productName, p.productLine, p.MSRP
FROM products AS p
WHERE p.productLine <> 'Motorcycles'
  AND p.MSRP > ALL (
      SELECT p2.MSRP
      FROM products AS p2
      WHERE p2.productLine = 'Motorcycles'
  )
ORDER BY p.MSRP;
```

---

### 7.3. Ví dụ 2: Sản phẩm có tồn kho thấp nhất trong từng dòng sản phẩm

```sql
SELECT p.productCode, p.productName, p.productLine, p.quantityInStock
FROM products AS p
WHERE p.quantityInStock <= ALL (
    SELECT p2.quantityInStock
    FROM products AS p2
    WHERE p2.productLine = p.productLine
)
ORDER BY p.productLine, p.quantityInStock;
```

Đây là subquery tương quan vì `p.productLine` thuộc bảng ở câu lệnh ngoài.

Nếu có nhiều sản phẩm cùng mức tồn kho thấp nhất, truy vấn sẽ hiển thị tất cả các sản phẩm đó.

---

### 7.4. Ví dụ 3: Khoản thanh toán lớn nhất của từng khách hàng

```sql
SELECT p.customerNumber, p.checkNumber, p.paymentDate, p.amount
FROM payments AS p
WHERE p.amount >= ALL (
    SELECT p2.amount
    FROM payments AS p2
    WHERE p2.customerNumber = p.customerNumber
)
ORDER BY p.customerNumber, p.amount DESC;
```

---

### 7.5. Ví dụ 4: Khách hàng có hạn mức tín dụng cao nhất trong từng quốc gia

```sql
SELECT c.customerNumber, c.customerName, c.country, c.creditLimit
FROM customers AS c
WHERE c.creditLimit IS NOT NULL
  AND c.creditLimit >= ALL (
      SELECT c2.creditLimit
      FROM customers AS c2
      WHERE c2.country = c.country
        AND c2.creditLimit IS NOT NULL
  )
ORDER BY c.country, c.creditLimit DESC;
```

### Bài tập thực hành

**Bài 7.1.** Liệt kê các sản phẩm có `MSRP` cao hơn mọi sản phẩm thuộc dòng `Motorcycles`.

**Bài 7.2.** Liệt kê các sản phẩm có `buyPrice` thấp nhất trong từng `productLine`.

**Bài 7.3.** Liệt kê các sản phẩm có `MSRP` cao nhất trong từng `productLine`.

**Bài 7.4.** Liệt kê khoản thanh toán lớn nhất của từng khách hàng. Nếu đồng hạng, hiển thị tất cả.

**Bài 7.5.** Liệt kê các khách hàng có `creditLimit` thấp nhất trong từng quốc gia, bỏ qua các giá trị `NULL`.

**Bài 7.6.** Liệt kê các nhân viên có `employeeNumber` nhỏ hơn mọi nhân viên có `jobTitle = 'Sales Rep'`.

---

## 8. Mệnh đề `ANY` và `SOME`

### 8.1. Ý nghĩa

`ANY` yêu cầu phép so sánh đúng với **ít nhất một giá trị** do subquery trả về.

`SOME` là từ đồng nghĩa của `ANY` trong MySQL.

Cú pháp:

```sql
expression comparison_operator ANY (subquery)
```

hoặc:

```sql
expression comparison_operator SOME (subquery)
```

Ví dụ:

```sql
x > ANY (subquery)
```

có nghĩa là: `x` lớn hơn **ít nhất một giá trị** do subquery trả về.

Khi subquery không rỗng và không có `NULL`, có thể hiểu nhanh:

| Biểu thức | Cách hiểu gần đúng |
|---|---|
| `x > ANY (S)` | `x` lớn hơn ít nhất một giá trị trong `S`, tương đương lớn hơn giá trị nhỏ nhất trong `S` |
| `x >= ANY (S)` | `x` lớn hơn hoặc bằng ít nhất một giá trị trong `S` |
| `x < ANY (S)` | `x` nhỏ hơn ít nhất một giá trị trong `S`, tương đương nhỏ hơn giá trị lớn nhất trong `S` |
| `x = ANY (S)` | tương đương về ý nghĩa với `x IN (S)` trong các trường hợp thông thường |

---

### 8.2. Ví dụ 1: Sản phẩm có MSRP cao hơn ít nhất một sản phẩm thuộc dòng Motorcycles

```sql
SELECT p.productCode, p.productName, p.productLine, p.MSRP
FROM products AS p
WHERE p.MSRP > ANY (
    SELECT p2.MSRP
    FROM products AS p2
    WHERE p2.productLine = 'Motorcycles'
)
ORDER BY p.MSRP;
```

Truy vấn này thường cho nhiều kết quả hơn truy vấn dùng `> ALL`, vì chỉ cần cao hơn **một** sản phẩm thuộc dòng `Motorcycles`.

---

### 8.3. Ví dụ 2: Sản phẩm không đắt nhất trong dòng sản phẩm của nó

```sql
SELECT p.productCode, p.productName, p.productLine, p.MSRP
FROM products AS p
WHERE p.MSRP < ANY (
    SELECT p2.MSRP
    FROM products AS p2
    WHERE p2.productLine = p.productLine
)
ORDER BY p.productLine, p.MSRP;
```

Một sản phẩm được chọn nếu trong cùng dòng sản phẩm tồn tại ít nhất một sản phẩm có MSRP cao hơn nó.

---

### 8.4. Ví dụ 3: Khoản thanh toán không phải nhỏ nhất của từng khách hàng

```sql
SELECT p.customerNumber, p.checkNumber, p.paymentDate, p.amount
FROM payments AS p
WHERE p.amount > ANY (
    SELECT p2.amount
    FROM payments AS p2
    WHERE p2.customerNumber = p.customerNumber
)
ORDER BY p.customerNumber, p.amount;
```

---

### 8.5. Ví dụ 4: Dùng `= ANY` để tìm khách hàng có đơn hàng đang bị giữ

```sql
SELECT c.customerNumber, c.customerName
FROM customers AS c
WHERE c.customerNumber = ANY (
    SELECT o.customerNumber
    FROM orders AS o
    WHERE o.status = 'On Hold'
)
ORDER BY c.customerNumber;
```

Có thể viết tương tự bằng `IN`:

```sql
SELECT c.customerNumber, c.customerName
FROM customers AS c
WHERE c.customerNumber IN (
    SELECT o.customerNumber
    FROM orders AS o
    WHERE o.status = 'On Hold'
)
ORDER BY c.customerNumber;
```

### Bài tập thực hành

**Bài 8.1.** Liệt kê các sản phẩm có `MSRP` cao hơn ít nhất một sản phẩm thuộc dòng `Motorcycles`.

**Bài 8.2.** Liệt kê các sản phẩm có `buyPrice` lớn hơn ít nhất một sản phẩm trong cùng `productLine`.

**Bài 8.3.** Liệt kê các sản phẩm có `quantityInStock` thấp hơn ít nhất một sản phẩm trong cùng `productLine`.

**Bài 8.4.** Liệt kê các khoản thanh toán lớn hơn ít nhất một khoản thanh toán khác của cùng khách hàng.

**Bài 8.5.** Dùng `= ANY` để liệt kê khách hàng có đơn hàng ở trạng thái `Disputed`.

**Bài 8.6.** Dùng `SOME` để liệt kê khách hàng có `creditLimit` lớn hơn ít nhất một khách hàng ở `France`; bỏ qua giá trị `NULL`.

---

## 9. So sánh `ALL`, `ANY` và `EXISTS`

| Mệnh đề | Câu hỏi cần trả lời | Ví dụ tư duy |
|---|---|---|
| `EXISTS (subquery)` | Có ít nhất một dòng thỏa điều kiện hay không? | Khách hàng có đơn hàng nào không? |
| `NOT EXISTS (subquery)` | Không có dòng nào thỏa điều kiện hay không? | Sản phẩm chưa từng được đặt hay không? |
| `x > ALL (subquery)` | `x` có lớn hơn mọi giá trị không? | Giá sản phẩm có cao hơn tất cả xe máy không? |
| `x > ANY (subquery)` | `x` có lớn hơn ít nhất một giá trị không? | Giá sản phẩm có cao hơn ít nhất một xe máy không? |
| `x = ANY (subquery)` | `x` có bằng ít nhất một giá trị không? | Khách hàng có nằm trong danh sách khách có đơn hàng bị giữ không? |

### Ví dụ trực quan

Giả sử subquery trả về tập giá trị:

```text
S = {10, 20, 30}
```

| Biểu thức | Kết quả với `x = 25` | Giải thích |
|---|---:|---|
| `25 > ALL (S)` | Sai | 25 không lớn hơn 30 |
| `25 > ANY (S)` | Đúng | 25 lớn hơn 10 và 20 |
| `25 < ALL (S)` | Sai | 25 không nhỏ hơn 10 |
| `25 < ANY (S)` | Đúng | 25 nhỏ hơn 30 |

---

## 10. Tập rỗng và giá trị `NULL`

### 10.1. Khi subquery trả về tập rỗng

Giả sử subquery không trả về dòng nào.

| Biểu thức | Kết quả logic |
|---|---|
| `EXISTS (subquery)` | Sai |
| `NOT EXISTS (subquery)` | Đúng |
| `x > ALL (subquery)` | Đúng |
| `x > ANY (subquery)` | Sai |

Ý nghĩa trực giác:

- “Lớn hơn **mọi phần tử** của một tập rỗng” không có phản ví dụ, nên được xem là đúng.
- “Lớn hơn **ít nhất một phần tử** của một tập rỗng” là sai vì không có phần tử nào để thỏa mãn.

### 10.2. Khi subquery có `NULL`

SQL dùng logic ba giá trị: `TRUE`, `FALSE`, `UNKNOWN`.

Nếu subquery trả về `NULL`, so sánh như `x > NULL` có kết quả `UNKNOWN`. Khi `UNKNOWN` xuất hiện trong điều kiện `WHERE`, dòng thường không được chọn.

Để tránh kết quả khó dự đoán, hãy lọc `NULL` trong subquery khi cần so sánh số hoặc ngày:

```sql
SELECT c.customerNumber, c.customerName, c.creditLimit
FROM customers AS c
WHERE c.creditLimit IS NOT NULL
  AND c.creditLimit > ALL (
      SELECT c2.creditLimit
      FROM customers AS c2
      WHERE c2.country = 'France'
        AND c2.creditLimit IS NOT NULL
  );
```

### 10.3. Vì sao `NOT EXISTS` thường phù hợp hơn `NOT IN`?

Nếu subquery trong `NOT IN` có `NULL`, điều kiện có thể trở thành `UNKNOWN` cho tất cả các dòng bên ngoài. `NOT EXISTS` kiểm tra trực tiếp quan hệ giữa dòng ngoài và dòng trong nên thường diễn đạt đúng hơn yêu cầu “không có bản ghi liên quan”.

### Bài tập thực hành

**Bài 10.1.** Giải thích vì sao `x > ALL (tập rỗng)` có giá trị đúng nhưng `x > ANY (tập rỗng)` có giá trị sai.

**Bài 10.2.** Hãy bổ sung điều kiện để truy vấn sau bỏ qua các hạn mức tín dụng `NULL`.

```sql
SELECT c.customerNumber, c.customerName, c.creditLimit
FROM customers AS c
WHERE c.creditLimit > ALL (
    SELECT c2.creditLimit
    FROM customers AS c2
    WHERE c2.country = 'USA'
);
```

**Bài 10.3.** Viết lại một truy vấn dùng `NOT IN` để tìm khách hàng chưa có đơn hàng bằng `NOT EXISTS`.

**Bài 10.4.** Nêu sự khác nhau giữa `EXISTS` và `IN` trong mục đích sử dụng, không cần xét tối ưu hóa thực thi.

---

## 11. `EXISTS` so với `JOIN`

`EXISTS` phù hợp khi mục tiêu là kiểm tra có/không có bản ghi liên quan.

Ví dụ tìm khách hàng có đơn hàng bằng `EXISTS`:

```sql
SELECT c.customerNumber, c.customerName
FROM customers AS c
WHERE EXISTS (
    SELECT 1
    FROM orders AS o
    WHERE o.customerNumber = c.customerNumber
);
```

Có thể viết bằng `JOIN`:

```sql
SELECT DISTINCT c.customerNumber, c.customerName
FROM customers AS c
JOIN orders AS o
    ON o.customerNumber = c.customerNumber;
```

Với `JOIN`, một khách hàng có nhiều đơn hàng sẽ tạo nhiều dòng kết quả; do đó cần `DISTINCT` nếu chỉ muốn mỗi khách hàng xuất hiện một lần. Với `EXISTS`, mục đích “có ít nhất một đơn hàng” được thể hiện trực tiếp.

> Không có một cú pháp luôn tốt hơn trong mọi tình huống. Hãy chọn biểu đạt rõ đúng yêu cầu; sau đó kiểm tra kế hoạch thực thi và chỉ mục khi tối ưu hiệu năng.

### Bài tập thực hành

**Bài 11.1.** Viết hai truy vấn trả về danh sách sản phẩm đã từng được đặt: một truy vấn dùng `EXISTS`, một truy vấn dùng `JOIN` kết hợp `DISTINCT`.

**Bài 11.2.** Giải thích vì sao truy vấn dùng `JOIN` có thể tạo bản ghi lặp nếu bỏ `DISTINCT`.

**Bài 11.3.** Viết truy vấn dùng `NOT EXISTS` để tìm nhân viên chưa phụ trách khách hàng nào. Giải thích vì sao dùng `INNER JOIN` không trực tiếp phù hợp với yêu cầu này.

---

## 12. Một số lỗi thường gặp

### Lỗi 1: Quên điều kiện tương quan trong `EXISTS`

Sai về mặt logic:

```sql
SELECT c.customerNumber, c.customerName
FROM customers AS c
WHERE EXISTS (
    SELECT 1
    FROM orders AS o
);
```

Nếu bảng `orders` có ít nhất một dòng, truy vấn trên sẽ trả về **mọi khách hàng**, vì subquery không liên hệ với khách hàng hiện tại.

Đúng:

```sql
SELECT c.customerNumber, c.customerName
FROM customers AS c
WHERE EXISTS (
    SELECT 1
    FROM orders AS o
    WHERE o.customerNumber = c.customerNumber
);
```

---

### Lỗi 2: Nhầm `ALL` với `ANY`

```sql
-- Cao hơn mọi sản phẩm thuộc Motorcycles
p.MSRP > ALL (...)

-- Cao hơn ít nhất một sản phẩm thuộc Motorcycles
p.MSRP > ANY (...)
```

---

### Lỗi 3: Không loại `NULL` khi so sánh với `ALL` hoặc `ANY`

Sai về mặt an toàn dữ liệu:

```sql
WHERE c.creditLimit > ALL (
    SELECT c2.creditLimit
    FROM customers AS c2
)
```

Nên chủ động loại `NULL` nếu cột có thể rỗng:

```sql
WHERE c.creditLimit IS NOT NULL
  AND c.creditLimit > ALL (
      SELECT c2.creditLimit
      FROM customers AS c2
      WHERE c2.creditLimit IS NOT NULL
  )
```

---

### Lỗi 4: Dùng cùng alias cho bảng ngoài và bảng trong

Sai hoặc khó đọc:

```sql
SELECT p.productCode
FROM products AS p
WHERE p.MSRP > ALL (
    SELECT p.MSRP
    FROM products AS p
);
```

Nên dùng alias khác nhau:

```sql
SELECT p.productCode
FROM products AS p
WHERE p.MSRP > ALL (
    SELECT p2.MSRP
    FROM products AS p2
);
```

### Bài tập thực hành

**Bài 12.1.** Sửa truy vấn sai sau để chỉ lấy khách hàng có ít nhất một đơn hàng.

```sql
SELECT c.customerNumber, c.customerName
FROM customers AS c
WHERE EXISTS (
    SELECT 1
    FROM orders AS o
);
```

**Bài 12.2.** Sửa truy vấn sau để tìm sản phẩm có MSRP cao hơn mọi sản phẩm thuộc dòng `Planes`.

```sql
SELECT productCode, productName, MSRP
FROM products
WHERE MSRP > ANY (
    SELECT MSRP
    FROM products
    WHERE productLine = 'Planes'
);
```

**Bài 12.3.** Sửa truy vấn sau để xử lý an toàn các giá trị `NULL` của `creditLimit`.

```sql
SELECT c.customerNumber, c.customerName, c.creditLimit
FROM customers AS c
WHERE c.creditLimit < ANY (
    SELECT c2.creditLimit
    FROM customers AS c2
    WHERE c2.country = 'USA'
);
```

---

## 13. Bài tập tổng hợp

### Bài 13.1. Khách hàng đã đặt hàng nhưng chưa từng thanh toán

Liệt kê mã và tên khách hàng thỏa cả hai điều kiện:

- Có ít nhất một đơn hàng.
- Không có khoản thanh toán nào.

Yêu cầu dùng cả `EXISTS` và `NOT EXISTS`.

---

### Bài 13.2. Sản phẩm đã từng được giao

Liệt kê các sản phẩm đã xuất hiện trong ít nhất một đơn hàng có `status = 'Shipped'`.

Gợi ý: dùng `EXISTS`; trong subquery có thể kết nối `orderdetails` và `orders`.

---

### Bài 13.3. Khách hàng có mọi đơn hàng đều đã giao

Liệt kê các khách hàng:

- Có ít nhất một đơn hàng.
- Không có đơn hàng nào có trạng thái khác `Shipped`.

Gợi ý: kết hợp `EXISTS` và `NOT EXISTS`.

---

### Bài 13.4. Khoản thanh toán cao nhất theo khách hàng

Liệt kê khoản thanh toán lớn nhất của mỗi khách hàng bằng `ALL`. Nếu một khách hàng có nhiều khoản đồng hạng, giữ lại tất cả.

---

### Bài 13.5. Sản phẩm không có tồn kho cao nhất trong dòng

Liệt kê các sản phẩm có `quantityInStock` thấp hơn ít nhất một sản phẩm khác cùng `productLine`.

Yêu cầu dùng `ANY` hoặc `SOME`.

---

### Bài 13.6. Khách hàng có hạn mức cao nhất theo quốc gia

Liệt kê khách hàng có `creditLimit` cao nhất trong từng quốc gia. Bỏ qua `creditLimit` rỗng. Dùng `ALL`.

---

### Bài 13.7. Kiểm tra `= ALL`

Viết truy vấn tìm khách hàng mà tất cả đơn hàng của họ đều có `status = 'Shipped'` bằng dạng so sánh `= ALL (...)`.

Lưu ý: cần thêm điều kiện để không chọn khách hàng chưa có đơn hàng.

---

### Bài 13.8. So sánh `> ALL` và `> ANY`

Viết hai truy vấn trên bảng `products`:

1. Sản phẩm có `MSRP` cao hơn mọi sản phẩm thuộc dòng `Ships`.
2. Sản phẩm có `MSRP` cao hơn ít nhất một sản phẩm thuộc dòng `Ships`.

So sánh số dòng kết quả của hai truy vấn và giải thích quan hệ giữa hai tập kết quả.

---

## 14. Đáp án gợi ý cho một số bài tập

> Các đáp án dưới đây chỉ là một cách viết. Có thể có truy vấn tương đương khác về cú pháp hoặc cách tổ chức điều kiện.

### Bài 4.4

```sql
SELECT c.customerNumber, c.customerName, c.creditLimit
FROM customers AS c
WHERE c.creditLimit > (
    SELECT AVG(c2.creditLimit)
    FROM customers AS c2
    WHERE c2.creditLimit IS NOT NULL
)
ORDER BY c.creditLimit DESC;
```

### Bài 5.3

```sql
SELECT o.officeCode, o.city, o.country
FROM offices AS o
WHERE EXISTS (
    SELECT 1
    FROM employees AS e
    WHERE e.officeCode = o.officeCode
)
ORDER BY o.officeCode;
```

### Bài 5.6

```sql
SELECT c.customerNumber, c.customerName
FROM customers AS c
WHERE EXISTS (
    SELECT 1
    FROM orders AS o
    WHERE o.customerNumber = c.customerNumber
      AND o.status = 'Shipped'
)
ORDER BY c.customerNumber;
```

### Bài 6.3

```sql
SELECT p.productCode, p.productName, p.productLine
FROM products AS p
WHERE NOT EXISTS (
    SELECT 1
    FROM orderdetails AS od
    WHERE od.productCode = p.productCode
)
ORDER BY p.productCode;
```

### Bài 6.6

```sql
SELECT c.customerNumber, c.customerName
FROM customers AS c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders AS o
    WHERE o.customerNumber = c.customerNumber
      AND o.status = 'Shipped'
)
ORDER BY c.customerNumber;
```

### Bài 7.2

```sql
SELECT p.productCode, p.productName, p.productLine, p.buyPrice
FROM products AS p
WHERE p.buyPrice <= ALL (
    SELECT p2.buyPrice
    FROM products AS p2
    WHERE p2.productLine = p.productLine
)
ORDER BY p.productLine, p.buyPrice;
```

### Bài 7.4

```sql
SELECT p.customerNumber, p.checkNumber, p.paymentDate, p.amount
FROM payments AS p
WHERE p.amount >= ALL (
    SELECT p2.amount
    FROM payments AS p2
    WHERE p2.customerNumber = p.customerNumber
)
ORDER BY p.customerNumber, p.amount DESC;
```

### Bài 8.3

```sql
SELECT p.productCode, p.productName, p.productLine, p.quantityInStock
FROM products AS p
WHERE p.quantityInStock < ANY (
    SELECT p2.quantityInStock
    FROM products AS p2
    WHERE p2.productLine = p.productLine
)
ORDER BY p.productLine, p.quantityInStock;
```

### Bài 8.5

```sql
SELECT c.customerNumber, c.customerName
FROM customers AS c
WHERE c.customerNumber = ANY (
    SELECT o.customerNumber
    FROM orders AS o
    WHERE o.status = 'Disputed'
)
ORDER BY c.customerNumber;
```

### Bài 10.2

```sql
SELECT c.customerNumber, c.customerName, c.creditLimit
FROM customers AS c
WHERE c.creditLimit IS NOT NULL
  AND c.creditLimit > ALL (
      SELECT c2.creditLimit
      FROM customers AS c2
      WHERE c2.country = 'USA'
        AND c2.creditLimit IS NOT NULL
  );
```

### Bài 11.1 — Cách dùng `EXISTS`

```sql
SELECT p.productCode, p.productName
FROM products AS p
WHERE EXISTS (
    SELECT 1
    FROM orderdetails AS od
    WHERE od.productCode = p.productCode
)
ORDER BY p.productCode;
```

### Bài 11.1 — Cách dùng `JOIN`

```sql
SELECT DISTINCT p.productCode, p.productName
FROM products AS p
JOIN orderdetails AS od
    ON od.productCode = p.productCode
ORDER BY p.productCode;
```

### Bài 13.1

```sql
SELECT c.customerNumber, c.customerName
FROM customers AS c
WHERE EXISTS (
    SELECT 1
    FROM orders AS o
    WHERE o.customerNumber = c.customerNumber
)
AND NOT EXISTS (
    SELECT 1
    FROM payments AS p
    WHERE p.customerNumber = c.customerNumber
)
ORDER BY c.customerNumber;
```

### Bài 13.2

```sql
SELECT p.productCode, p.productName, p.productLine
FROM products AS p
WHERE EXISTS (
    SELECT 1
    FROM orderdetails AS od
    JOIN orders AS o
        ON o.orderNumber = od.orderNumber
    WHERE od.productCode = p.productCode
      AND o.status = 'Shipped'
)
ORDER BY p.productCode;
```

### Bài 13.3

```sql
SELECT c.customerNumber, c.customerName
FROM customers AS c
WHERE EXISTS (
    SELECT 1
    FROM orders AS o
    WHERE o.customerNumber = c.customerNumber
)
AND NOT EXISTS (
    SELECT 1
    FROM orders AS o
    WHERE o.customerNumber = c.customerNumber
      AND o.status <> 'Shipped'
)
ORDER BY c.customerNumber;
```

### Bài 13.7

```sql
SELECT c.customerNumber, c.customerName
FROM customers AS c
WHERE EXISTS (
    SELECT 1
    FROM orders AS o
    WHERE o.customerNumber = c.customerNumber
)
AND 'Shipped' = ALL (
    SELECT o.status
    FROM orders AS o
    WHERE o.customerNumber = c.customerNumber
)
ORDER BY c.customerNumber;
```

---

## 15. Tóm tắt

- Subquery là câu lệnh `SELECT` nằm trong một câu lệnh SQL khác.
- Subquery tương quan tham chiếu tới bảng hoặc alias của câu lệnh ngoài.
- `EXISTS` dùng để kiểm tra có ít nhất một bản ghi liên quan.
- `NOT EXISTS` dùng để kiểm tra không có bản ghi liên quan.
- `ALL` yêu cầu điều kiện so sánh đúng với mọi giá trị do subquery trả về.
- `ANY`/`SOME` yêu cầu điều kiện so sánh đúng với ít nhất một giá trị do subquery trả về.
- `> ALL` có điều kiện chặt hơn `> ANY` trên cùng một tập kết quả.
- Khi dùng `ALL`, `ANY` hoặc `NOT IN`, cần chú ý đến tập rỗng và giá trị `NULL`.
- Với yêu cầu “có/không có quan hệ”, `EXISTS` và `NOT EXISTS` thường biểu đạt rõ hơn `JOIN` hoặc `NOT IN`.

---

## 16. Từ khóa chính

- Subquery
- Correlated Subquery
- EXISTS
- NOT EXISTS
- ALL
- ANY
- SOME
- IN
- NOT IN
- NULL
- Empty Set
- Three-Valued Logic
- `classicmodels`
- `customers`
- `orders`
- `orderdetails`
- `products`
- `payments`
