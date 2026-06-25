---
title: "Tutorial: Views trong MySQL"
author: "Tên giảng viên"
duration: "120m"
difficulty: "Intermediate"
prerequisites:
  - "Đã biết SELECT, WHERE, JOIN, GROUP BY và HAVING"
  - "Đã biết các bảng customers, employees, offices, orders, orderdetails, payments trong classicmodels"
  - "Có quyền CREATE VIEW, SHOW VIEW và DROP trên cơ sở dữ liệu thực hành"
summary: "Thực hành tạo, sử dụng, kiểm tra, đổi tên và xóa view; đồng thời hiểu thuật toán xử lý view, updatable view và WITH CHECK OPTION."
---

# Tutorial: Views trong MySQL

## Link tham khảo

- [MySQL Tutorial — MySQL Views](https://www.mysqltutorial.org/mysql-views/)


## 1. Mục tiêu học tập

Sau khi hoàn thành tutorial này, người học có thể:

1. Giải thích được view là gì và phân biệt view với bảng cơ sở.
2. Tạo view bằng `CREATE VIEW` và thay thế view bằng `CREATE OR REPLACE VIEW`.
3. Hiểu ba thuật toán xử lý view: `MERGE`, `TEMPTABLE`, `UNDEFINED`.
4. Nhận biết được view nào có thể cập nhật dữ liệu và view nào không thể cập nhật.
5. Tạo updatable view và thao tác `UPDATE`, `INSERT`, `DELETE` qua view khi phù hợp.
6. Dùng `WITH CHECK OPTION` để ngăn thay đổi làm dòng dữ liệu không còn thỏa điều kiện của view.
7. Phân biệt `LOCAL` và `CASCADED` trong `WITH CHECK OPTION`.
8. Liệt kê view trong database và xem thông tin view qua `INFORMATION_SCHEMA`.
9. Xem câu lệnh tạo view bằng `SHOW CREATE VIEW`.
10. Đổi tên, xóa một hoặc nhiều view an toàn.

---

## 2. View là gì?

Một **view** là một đối tượng cơ sở dữ liệu được định nghĩa bằng một câu lệnh `SELECT`. View có thể được truy vấn gần giống như bảng:

```sql
SELECT *
FROM view_name;
```

Tuy nhiên, view thường không lưu dữ liệu riêng như một bảng cơ sở. Khi truy vấn view, MySQL sử dụng định nghĩa `SELECT` của view để lấy dữ liệu từ bảng hoặc view bên dưới.

Ví dụ, thay vì nhiều lần viết truy vấn lấy khách hàng tại Hoa Kỳ:

```sql
SELECT customerNumber, customerName, city, state, creditLimit
FROM customers
WHERE country = 'USA';
```

ta có thể tạo view:

```sql
CREATE VIEW v_usa_customers AS
SELECT customerNumber, customerName, city, state, creditLimit
FROM customers
WHERE country = 'USA';
```

Sau đó truy vấn ngắn gọn:

```sql
SELECT *
FROM v_usa_customers;
```

### 2.1. Lợi ích của view

| Lợi ích | Giải thích |
|---|---|
| Đơn giản hóa truy vấn | Đóng gói một truy vấn dài hoặc dùng lặp lại nhiều lần |
| Tái sử dụng logic | Một định nghĩa có thể được dùng bởi nhiều truy vấn hoặc ứng dụng |
| Hạn chế dữ liệu hiển thị | Chỉ đưa ra các cột và dòng cần thiết cho một nhóm người dùng |
| Ổn định giao diện truy vấn | Ứng dụng có thể truy vấn view thay vì phụ thuộc trực tiếp vào bảng |
| Hỗ trợ phân quyền | Có thể cấp quyền trên view thay vì cấp quyền trực tiếp trên toàn bộ bảng |

### 2.2. View không thay thế bảng cơ sở

View không phải là giải pháp để thay thế thiết kế bảng, khóa chính, khóa ngoại hoặc constraint. View chủ yếu cung cấp một **lớp truy cập logic** trên dữ liệu đã tồn tại.

> Lưu ý: `CREATE VIEW`, `ALTER VIEW`, `DROP VIEW` là các thao tác DDL. Trong MySQL, không nên kỳ vọng `ROLLBACK` sẽ hoàn tác việc tạo hoặc xóa view. Khi thực hành, cần dọn dẹp view bằng `DROP VIEW IF EXISTS`.

### Bài tập thực hành

**Bài 2.1.** Giải thích view khác bảng cơ sở ở điểm nào.

**Bài 2.2.** Nêu hai lợi ích của việc dùng view trong hệ thống bán hàng.

**Bài 2.3.** Cho biết view có thể được tạo từ một bảng, nhiều bảng hay từ một view khác.

**Bài 2.4.** Giải thích vì sao view không thay thế primary key và foreign key.

**Bài 2.5.** Nêu một tình huống nên dùng view để giới hạn các cột được hiển thị cho người dùng.

---

## 3. Chuẩn bị môi trường

Chọn cơ sở dữ liệu mẫu:

```sql
USE classicmodels;
```

Kiểm tra các bảng sẽ dùng:

```sql
SHOW TABLES;
```

Xem một số dữ liệu mẫu:

```sql
SELECT customerNumber, customerName, city, country, creditLimit
FROM customers
ORDER BY customerNumber
LIMIT 10;
```

```sql
SELECT employeeNumber, firstName, lastName, jobTitle, officeCode
FROM employees
ORDER BY employeeNumber
LIMIT 10;
```

```sql
SELECT officeCode, city, country, territory
FROM offices
ORDER BY officeCode;
```

Trước khi tạo lại view cùng tên, xóa các view thực hành cũ nếu có:

```sql
DROP VIEW IF EXISTS
    v_usa_customers,
    v_customer_contacts,
    v_sales_reps,
    v_customer_payment_summary,
    v_editable_customers,
    v_usa_customers_check,
    v_usa_customers_base,
    v_high_credit_local,
    v_high_credit_cascaded,
    v_usa_customers_renamed;
```

### Bài tập thực hành

**Bài 3.1.** Viết lệnh chọn cơ sở dữ liệu `classicmodels`.

**Bài 3.2.** Viết lệnh xem cấu trúc bảng `customers`.

**Bài 3.3.** Viết lệnh xem cấu trúc bảng `employees`.

**Bài 3.4.** Viết truy vấn xem 10 khách hàng đầu tiên cùng quốc gia và hạn mức tín dụng.

**Bài 3.5.** Viết lệnh xóa an toàn view `v_usa_customers` nếu view này đã tồn tại.

---

# Phần A. Tạo và sử dụng view

## 4. Tạo view bằng `CREATE VIEW`

### 4.1. Cú pháp cơ bản

```sql
CREATE [OR REPLACE]
    [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}]
VIEW view_name [(column_name, ...)]
AS
select_statement
[WITH [CASCADED | LOCAL] CHECK OPTION];
```

Dạng đơn giản nhất:

```sql
CREATE VIEW view_name AS
SELECT ...;
```

Ví dụ tạo view khách hàng tại Hoa Kỳ:

```sql
CREATE VIEW v_usa_customers AS
SELECT customerNumber,
       customerName,
       city,
       state,
       creditLimit
FROM customers
WHERE country = 'USA';
```

Truy vấn view:

```sql
SELECT *
FROM v_usa_customers
ORDER BY customerName;
```

### 4.2. Tạo view có alias rõ ràng

```sql
CREATE VIEW v_customer_contacts AS
SELECT customerNumber AS CustomerNumber,
       customerName AS CustomerName,
       contactLastName AS ContactLastName,
       contactFirstName AS ContactFirstName,
       phone AS PhoneNumber,
       country AS Country
FROM customers;
```

View phải có tên cột duy nhất. Vì vậy, khi dùng biểu thức hoặc join có cột trùng tên, nên đặt alias.

### 4.3. Tạo view từ nhiều bảng

Ví dụ tạo view danh sách sales representative cùng văn phòng:

```sql
CREATE VIEW v_sales_reps AS
SELECT e.employeeNumber,
       CONCAT(e.firstName, ' ', e.lastName) AS employeeName,
       e.email,
       e.jobTitle,
       o.city AS officeCity,
       o.country AS officeCountry
FROM employees AS e
JOIN offices AS o
    ON e.officeCode = o.officeCode
WHERE e.jobTitle = 'Sales Rep';
```

Truy vấn:

```sql
SELECT *
FROM v_sales_reps
ORDER BY officeCountry, officeCity, employeeName;
```

### 4.4. Tạo view dùng aggregate

```sql
CREATE VIEW v_customer_payment_summary AS
SELECT c.customerNumber,
       c.customerName,
       COUNT(p.checkNumber) AS paymentCount,
       COALESCE(SUM(p.amount), 0) AS totalPayment
FROM customers AS c
LEFT JOIN payments AS p
    ON c.customerNumber = p.customerNumber
GROUP BY c.customerNumber, c.customerName;
```

View này hữu ích cho báo cáo, nhưng do có `COUNT`, `SUM` và `GROUP BY`, nó không phải updatable view.

### 4.5. `CREATE OR REPLACE VIEW`

Nếu muốn thay thế định nghĩa của một view đã có:

```sql
CREATE OR REPLACE VIEW v_usa_customers AS
SELECT customerNumber,
       customerName,
       city,
       state,
       phone,
       creditLimit
FROM customers
WHERE country = 'USA';
```

> Không dùng `SELECT *` khi tạo view nếu muốn kiểm soát rõ cột được công bố. Nếu bảng cơ sở có thêm cột sau này, các cột mới không tự động xuất hiện trong view đã tạo bằng `SELECT *`.

### Bài tập thực hành

**Bài 4.1.** Tạo view `v_french_customers` chứa `customerNumber`, `customerName`, `city`, `creditLimit` của khách hàng ở `France`.

**Bài 4.2.** Tạo view `v_product_prices` chứa `productCode`, `productName`, `productLine`, `buyPrice`, `MSRP` và cột tính toán `priceDifference = MSRP - buyPrice`.

**Bài 4.3.** Tạo view `v_office_contacts` chứa `officeCode`, `city`, `phone`, `country`, `territory` từ bảng `offices`.

**Bài 4.4.** Tạo view `v_sales_rep_list` kết hợp `employees` và `offices`; chỉ hiển thị nhân viên có `jobTitle = 'Sales Rep'`.

**Bài 4.5.** Dùng `CREATE OR REPLACE VIEW` để bổ sung cột `phone` vào view `v_french_customers`.

---

## 5. Cột của view, alias và các lưu ý khi tạo view

### 5.1. Tên cột trong view

Theo mặc định, tên cột của view lấy từ `SELECT` bên trong định nghĩa. Có thể chỉ định rõ danh sách tên cột:

```sql
CREATE VIEW v_product_margin (
    productCode,
    productName,
    buyPrice,
    msrp,
    grossMargin
) AS
SELECT productCode,
       productName,
       buyPrice,
       MSRP,
       MSRP - buyPrice
FROM products;
```

Số tên trong danh sách cột phải bằng số cột trong `SELECT`.

### 5.2. Cột biểu thức nên có alias

Sai hoặc khó đọc:

```sql
CREATE VIEW v_product_margin_bad AS
SELECT productCode,
       MSRP - buyPrice
FROM products;
```

Nên viết:

```sql
CREATE VIEW v_product_margin_good AS
SELECT productCode,
       MSRP - buyPrice AS grossMargin
FROM products;
```

### 5.3. View có thể dựa trên view khác

```sql
CREATE VIEW v_usa_high_credit_customers AS
SELECT customerNumber,
       customerName,
       city,
       creditLimit
FROM v_usa_customers
WHERE creditLimit >= 100000;
```

Khi tạo view dựa trên view khác, cần quản lý tên và dependency cẩn thận. Nếu view bên dưới bị xóa hoặc đổi tên, view bên trên có thể không còn hoạt động.

### Bài tập thực hành

**Bài 5.1.** Tạo view `v_customer_addresses` có các cột được đặt lại tên là `CustomerNo`, `CustomerName`, `Address`, `City`, `Country`.

**Bài 5.2.** Tạo view `v_order_line_value` có `orderNumber`, `productCode`, `quantityOrdered`, `priceEach` và `lineTotal = quantityOrdered * priceEach`.

**Bài 5.3.** Tạo view `v_product_margin_named` bằng danh sách tên cột sau tên view.

**Bài 5.4.** Tạo view `v_usa_customer_credit` dựa trên `v_usa_customers`, chỉ chứa khách hàng có `creditLimit >= 50000`.

**Bài 5.5.** Giải thích vì sao cột biểu thức như `MSRP - buyPrice` nên được đặt alias khi tạo view.

---

# Phần B. Thuật toán xử lý view

## 6. View processing algorithms

MySQL hỗ trợ ba giá trị cho `ALGORITHM`:

| Algorithm | Ý nghĩa |
|---|---|
| `MERGE` | MySQL ghép định nghĩa view vào truy vấn đang tham chiếu view |
| `TEMPTABLE` | MySQL tạo bảng tạm từ kết quả view rồi truy vấn bảng tạm đó |
| `UNDEFINED` | MySQL tự chọn; thường ưu tiên `MERGE` nếu có thể |

### 6.1. `ALGORITHM = MERGE`

```sql
CREATE ALGORITHM = MERGE VIEW v_usa_customers AS
SELECT customerNumber,
       customerName,
       city,
       creditLimit
FROM customers
WHERE country = 'USA';
```

Khi chạy:

```sql
SELECT *
FROM v_usa_customers
WHERE creditLimit > 50000;
```

về mặt khái niệm, MySQL có thể xử lý gần tương đương:

```sql
SELECT customerNumber,
       customerName,
       city,
       creditLimit
FROM customers
WHERE country = 'USA'
  AND creditLimit > 50000;
```

`MERGE` thường hiệu quả hơn và cần thiết cho nhiều view có thể cập nhật.

### 6.2. `ALGORITHM = TEMPTABLE`

```sql
CREATE ALGORITHM = TEMPTABLE VIEW v_customer_payment_summary AS
SELECT c.customerNumber,
       c.customerName,
       COUNT(p.checkNumber) AS paymentCount,
       COALESCE(SUM(p.amount), 0) AS totalPayment
FROM customers AS c
LEFT JOIN payments AS p
    ON c.customerNumber = p.customerNumber
GROUP BY c.customerNumber, c.customerName;
```

Với `TEMPTABLE`, MySQL lấy kết quả view vào bảng tạm rồi xử lý truy vấn phía ngoài. View được xử lý theo cách này không thể cập nhật.

### 6.3. `ALGORITHM = UNDEFINED`

```sql
CREATE ALGORITHM = UNDEFINED VIEW v_product_prices AS
SELECT productCode,
       productName,
       productLine,
       buyPrice,
       MSRP
FROM products;
```

Nếu không ghi `ALGORITHM`, MySQL cũng dùng cơ chế lựa chọn mặc định tương tự `UNDEFINED`.

### 6.4. Khi nào không thể dùng `MERGE`?

Một số cấu trúc buộc MySQL phải materialize hoặc khiến view không phù hợp cho `MERGE`, ví dụ:

- `DISTINCT`
- aggregate functions như `SUM`, `COUNT`, `AVG`
- `GROUP BY`
- `HAVING`
- `UNION` hoặc `UNION ALL`
- một số `LIMIT` hoặc subquery

### Bài tập thực hành

**Bài 6.1.** Tạo view `v_usa_customers_merge` với `ALGORITHM = MERGE`.

**Bài 6.2.** Tạo view `v_payment_summary_temp` với `ALGORITHM = TEMPTABLE`, dùng `SUM(amount)` theo `customerNumber`.

**Bài 6.3.** Tạo view `v_all_product_lines_undefined` với `ALGORITHM = UNDEFINED`.

**Bài 6.4.** Xác định view nào sau đây phù hợp với `MERGE`: view chỉ dùng `WHERE`, view dùng `GROUP BY`, hay view dùng `UNION`.

**Bài 6.5.** Giải thích vì sao view dùng `GROUP BY` thường không thể là updatable view.

---

# Phần C. Updatable views

## 7. Updatable view là gì?

Một view có thể update được khi MySQL có thể ánh xạ rõ một dòng trong view về một dòng tương ứng ở bảng cơ sở. Nói đơn giản, thường cần quan hệ một-một giữa các dòng của view và bảng bên dưới.

Ví dụ view đơn bảng, không aggregate:

```sql
CREATE VIEW v_editable_customers AS
SELECT customerNumber,
       customerName,
       contactLastName,
       contactFirstName,
       phone,
       addressLine1,
       city,
       country,
       creditLimit
FROM customers;
```

View này có thể dùng cho một số thao tác `UPDATE` hoặc `DELETE` trên dữ liệu cơ sở, nếu người dùng có quyền phù hợp.

Ví dụ cập nhật qua view trong transaction:

```sql
START TRANSACTION;

UPDATE v_editable_customers
SET creditLimit = creditLimit + 1000.00
WHERE customerNumber = 103;

SELECT customerNumber, customerName, creditLimit
FROM customers
WHERE customerNumber = 103;

ROLLBACK;
```

### 7.1. Các cấu trúc thường làm view không update được

Một view không updatable nếu chứa, trong số các trường hợp phổ biến:

- aggregate function hoặc window function
- `DISTINCT`
- `GROUP BY`
- `HAVING`
- `UNION` hoặc `UNION ALL`
- chỉ literal, không có bảng cơ sở để cập nhật
- `ALGORITHM = TEMPTABLE`

Ví dụ non-updatable view:

```sql
CREATE VIEW v_customer_payment_summary AS
SELECT customerNumber,
       COUNT(*) AS paymentCount,
       SUM(amount) AS totalPayment
FROM payments
GROUP BY customerNumber;
```

Không thể thực hiện:

```sql
UPDATE v_customer_payment_summary
SET totalPayment = 0;
```

### 7.2. Insertable view

Một view updatable chưa chắc đã insertable.

Để `INSERT` qua view, view cần có đủ các cột bắt buộc của bảng cơ sở mà không có giá trị mặc định, và các cột view cần là tham chiếu cột đơn giản, không phải biểu thức.

Ví dụ thực hành an toàn hơn là kiểm tra khả năng insertable qua metadata thay vì chèn trực tiếp vào bảng `customers` dùng chung:

```sql
SELECT TABLE_NAME, IS_UPDATABLE
FROM INFORMATION_SCHEMA.VIEWS
WHERE TABLE_SCHEMA = 'classicmodels'
  AND TABLE_NAME = 'v_editable_customers';
```

### Bài tập thực hành

**Bài 7.1.** Tạo view `v_editable_customer_phone` chỉ từ bảng `customers`, gồm `customerNumber`, `customerName`, `phone`, `city`, `country`.

**Bài 7.2.** Trong transaction, cập nhật `phone` của khách hàng `103` qua `v_editable_customer_phone`, sau đó `ROLLBACK`.

**Bài 7.3.** Tạo view `v_nonupdatable_customer_count` dùng `COUNT(*)` và `GROUP BY country`.

**Bài 7.4.** Giải thích vì sao `v_nonupdatable_customer_count` không thể dùng để cập nhật `customerCount`.

**Bài 7.5.** Dùng `INFORMATION_SCHEMA.VIEWS` để kiểm tra `IS_UPDATABLE` của `v_editable_customer_phone` và `v_nonupdatable_customer_count`.

---

## 8. Updatable view với join: lưu ý

MySQL có thể cho phép một số view có join được update, nhưng điều kiện thực tế phức tạp hơn view đơn bảng. Để mới học, nên coi các view join chủ yếu phục vụ đọc và báo cáo.

Ví dụ view join để đọc:

```sql
CREATE VIEW v_customer_sales_rep AS
SELECT c.customerNumber,
       c.customerName,
       c.city AS customerCity,
       c.country AS customerCountry,
       e.employeeNumber AS salesRepNumber,
       CONCAT(e.firstName, ' ', e.lastName) AS salesRepName
FROM customers AS c
LEFT JOIN employees AS e
    ON c.salesRepEmployeeNumber = e.employeeNumber;
```

Một số view join có thể update được một bảng thành phần trong những điều kiện nhất định. Tuy nhiên:

- Không nên giả định toàn bộ cột của view join đều update được.
- `DELETE` qua join view bị hạn chế hơn `UPDATE`.
- `INSERT` qua join view chỉ phù hợp trong những trường hợp rất cụ thể.

Trong thực tế giảng dạy cơ bản, nên dùng view đơn bảng để luyện thao tác cập nhật.

### Bài tập thực hành

**Bài 8.1.** Tạo view `v_customer_sales_rep` như ví dụ trên.

**Bài 8.2.** Dùng view `v_customer_sales_rep` để xem khách hàng chưa có sales representative.

**Bài 8.3.** Dùng view `v_customer_sales_rep` để xem khách hàng ở `USA` cùng tên sales representative.

**Bài 8.4.** Giải thích vì sao view join thường phù hợp hơn cho truy vấn đọc so với thực hành cập nhật cơ bản.

**Bài 8.5.** Xác định bảng nào đóng vai trò chính trong view `v_customer_sales_rep` khi muốn thay đổi `customerName`.

---

# Phần D. WITH CHECK OPTION

## 9. Tạo view với `WITH CHECK OPTION`

Một view có điều kiện `WHERE` có thể cho phép `INSERT` hoặc `UPDATE` khiến dòng dữ liệu không còn thỏa điều kiện view. `WITH CHECK OPTION` được dùng để ngăn tình huống đó.

Ví dụ:

```sql
CREATE VIEW v_usa_customers_check AS
SELECT customerNumber,
       customerName,
       contactLastName,
       contactFirstName,
       phone,
       addressLine1,
       city,
       state,
       postalCode,
       country,
       creditLimit
FROM customers
WHERE country = 'USA'
WITH CHECK OPTION;
```

Nếu một dòng đang thuộc view, thao tác sau sẽ bị từ chối nếu làm quốc gia không còn là `USA`:

```sql
START TRANSACTION;

UPDATE v_usa_customers_check
SET country = 'France'
WHERE customerNumber = 112;

ROLLBACK;
```

Nếu khách hàng `112` không thuộc `USA` trong bản dữ liệu của bạn, hãy dùng `SELECT` trước để chọn một `customerNumber` có `country = 'USA'`.

```sql
SELECT customerNumber, customerName, country
FROM v_usa_customers_check
ORDER BY customerNumber
LIMIT 10;
```

### 9.1. Không dùng CHECK OPTION thì sao?

```sql
CREATE VIEW v_usa_customers_no_check AS
SELECT customerNumber,
       customerName,
       country,
       creditLimit
FROM customers
WHERE country = 'USA';
```

Về nguyên tắc, một update qua view không có `WITH CHECK OPTION` có thể làm dòng vừa được sửa không còn xuất hiện khi truy vấn lại view. Điều này có thể gây khó hiểu và làm sai logic nghiệp vụ.

### Bài tập thực hành

**Bài 9.1.** Tạo view `v_french_customers_check` chứa khách hàng ở `France` với `WITH CHECK OPTION`.

**Bài 9.2.** Dùng `SELECT` lấy một khách hàng từ `v_french_customers_check` trước khi thử cập nhật.

**Bài 9.3.** Trong transaction, thử cập nhật `country` của một dòng thuộc `v_french_customers_check` sang `USA`. Ghi lại kết quả.

**Bài 9.4.** Tạo view `v_high_credit_customers_check` với điều kiện `creditLimit >= 50000` và `WITH CHECK OPTION`.

**Bài 9.5.** Trong transaction, thử giảm `creditLimit` của một dòng trong `v_high_credit_customers_check` xuống `1000.00`. Giải thích kết quả.

---

## 10. `LOCAL` và `CASCADED` với `WITH CHECK OPTION`

`LOCAL` và `CASCADED` xác định phạm vi kiểm tra khi một view được xây dựng từ view khác.

Nếu không ghi rõ, MySQL mặc định dùng:

```sql
WITH CASCADED CHECK OPTION
```

### 10.1. Ví dụ view cơ sở

```sql
CREATE VIEW v_usa_customers_base AS
SELECT customerNumber,
       customerName,
       contactLastName,
       contactFirstName,
       phone,
       addressLine1,
       city,
       country,
       creditLimit
FROM customers
WHERE country = 'USA';
```

### 10.2. `LOCAL CHECK OPTION`

```sql
CREATE VIEW v_high_credit_local AS
SELECT customerNumber,
       customerName,
       contactLastName,
       contactFirstName,
       phone,
       addressLine1,
       city,
       country,
       creditLimit
FROM v_usa_customers_base
WHERE creditLimit >= 50000
WITH LOCAL CHECK OPTION;
```

`LOCAL` kiểm tra điều kiện của view hiện tại. Với các view bên dưới, điều kiện chỉ được áp dụng nếu chính view bên dưới có `WITH CHECK OPTION` thích hợp.

### 10.3. `CASCADED CHECK OPTION`

```sql
CREATE VIEW v_high_credit_cascaded AS
SELECT customerNumber,
       customerName,
       contactLastName,
       contactFirstName,
       phone,
       addressLine1,
       city,
       country,
       creditLimit
FROM v_usa_customers_base
WHERE creditLimit >= 50000
WITH CASCADED CHECK OPTION;
```

`CASCADED` kiểm tra điều kiện của view hiện tại và áp dụng kiểm tra xuống các view bên dưới như một chuỗi ràng buộc. Nó phù hợp khi muốn đảm bảo mọi điều kiện phân lớp đều được giữ.

### 10.4. Tóm tắt

| Tùy chọn | Phạm vi kiểm tra |
|---|---|
| Không ghi tùy chọn | Mặc định `CASCADED` |
| `LOCAL` | Kiểm tra điều kiện view hiện tại; các view dưới chỉ bị ràng buộc nếu chúng đã có check option phù hợp |
| `CASCADED` | Kiểm tra điều kiện view hiện tại và các điều kiện của view bên dưới trong chuỗi |

> Trong thiết kế ứng dụng, `CASCADED` thường rõ ràng hơn khi các view xếp nhiều tầng và tất cả điều kiện cần được bảo toàn.

### Bài tập thực hành

**Bài 10.1.** Tạo `v_usa_customers_base` với điều kiện `country = 'USA'`.

**Bài 10.2.** Tạo `v_high_credit_local` từ `v_usa_customers_base` với điều kiện `creditLimit >= 50000` và `LOCAL CHECK OPTION`.

**Bài 10.3.** Tạo `v_high_credit_cascaded` từ `v_usa_customers_base` với điều kiện `creditLimit >= 50000` và `CASCADED CHECK OPTION`.

**Bài 10.4.** Nêu điều kiện dữ liệu cần thỏa khi một dòng được cập nhật qua `v_high_credit_cascaded`.

**Bài 10.5.** Giải thích vì sao `CASCADED` thường an toàn hơn `LOCAL` khi view được xây dựng thành nhiều tầng.

---

# Phần E. Hiển thị và quản lý view

## 11. Liệt kê view trong database

### 11.1. Dùng `SHOW FULL TABLES`

```sql
SHOW FULL TABLES FROM classicmodels
WHERE Table_type = 'VIEW';
```

`SHOW TABLES` thông thường liệt kê cả table và view nhưng không phân biệt rõ loại đối tượng. `SHOW FULL TABLES` cho biết `Table_type` là `BASE TABLE` hoặc `VIEW`.

### 11.2. Dùng `INFORMATION_SCHEMA.VIEWS`

```sql
SELECT TABLE_NAME,
       CHECK_OPTION,
       IS_UPDATABLE,
       DEFINER,
       SECURITY_TYPE
FROM INFORMATION_SCHEMA.VIEWS
WHERE TABLE_SCHEMA = 'classicmodels'
ORDER BY TABLE_NAME;
```

Cột `IS_UPDATABLE` giúp kiểm tra cờ MySQL xác định view có thể update được hay không.

### 11.3. Lọc theo tên view

```sql
SHOW FULL TABLES FROM classicmodels
LIKE 'v\_%';
```

### Bài tập thực hành

**Bài 11.1.** Liệt kê toàn bộ view trong database `classicmodels` bằng `SHOW FULL TABLES`.

**Bài 11.2.** Liệt kê `TABLE_NAME`, `CHECK_OPTION`, `IS_UPDATABLE` của tất cả view bằng `INFORMATION_SCHEMA.VIEWS`.

**Bài 11.3.** Liệt kê các view có tên bắt đầu bằng `v_customer`.

**Bài 11.4.** Tìm các view có `IS_UPDATABLE = 'YES'`.

**Bài 11.5.** Tìm các view có `CHECK_OPTION` khác `NONE`.

---

## 12. Xem định nghĩa view bằng `SHOW CREATE VIEW`

Cú pháp:

```sql
SHOW CREATE VIEW view_name;
```

Ví dụ:

```sql
SHOW CREATE VIEW v_usa_customers;
```

Kết quả cho biết câu lệnh MySQL lưu để tạo view, thường bao gồm:

- `ALGORITHM`
- `DEFINER`
- `SQL SECURITY`
- câu `SELECT` định nghĩa view
- `WITH CHECK OPTION` nếu có

Có thể xem theo dạng dọc trong MySQL client:

```sql
SHOW CREATE VIEW v_usa_customers\G
```

### Bài tập thực hành

**Bài 12.1.** Xem định nghĩa của `v_usa_customers`.

**Bài 12.2.** Xem định nghĩa của `v_sales_reps`.

**Bài 12.3.** Xem định nghĩa của `v_customer_payment_summary`.

**Bài 12.4.** Xác định `ALGORITHM` mà MySQL ghi nhận cho một view bạn đã tạo.

**Bài 12.5.** Xác định view nào có `WITH CHECK OPTION` bằng `SHOW CREATE VIEW`.

---

## 13. Đổi tên view

MySQL không có câu lệnh `RENAME VIEW` riêng. Có thể dùng `RENAME TABLE` để đổi tên một view:

```sql
RENAME TABLE v_usa_customers TO v_usa_customers_renamed;
```

Sau đó truy vấn bằng tên mới:

```sql
SELECT *
FROM v_usa_customers_renamed
LIMIT 10;
```

Đổi lại tên cũ nếu cần:

```sql
RENAME TABLE v_usa_customers_renamed TO v_usa_customers;
```

### Lưu ý

- Cần kiểm tra xem có view, stored procedure hoặc ứng dụng nào đang tham chiếu tên cũ hay không.
- Đổi tên view có thể làm các đối tượng phụ thuộc vào tên cũ không hoạt động.
- Nếu chỉ muốn thay đổi định nghĩa, thường dùng `CREATE OR REPLACE VIEW` hoặc `ALTER VIEW` thay vì rename.

### Bài tập thực hành

**Bài 13.1.** Đổi tên `v_usa_customers` thành `v_usa_customers_renamed`.

**Bài 13.2.** Truy vấn `v_usa_customers_renamed` để xác nhận đổi tên thành công.

**Bài 13.3.** Đổi tên `v_usa_customers_renamed` trở lại `v_usa_customers`.

**Bài 13.4.** Giải thích sự khác nhau giữa đổi tên view và thay đổi định nghĩa view.

**Bài 13.5.** Nêu một rủi ro khi đổi tên view trong hệ thống đang chạy.

---

## 14. Xóa view bằng `DROP VIEW`

Cú pháp:

```sql
DROP VIEW [IF EXISTS] view_name [, view_name] ...;
```

Ví dụ xóa một view:

```sql
DROP VIEW v_product_prices;
```

Dùng `IF EXISTS` để tránh lỗi nếu view không tồn tại:

```sql
DROP VIEW IF EXISTS v_product_prices;
```

Xóa nhiều view:

```sql
DROP VIEW IF EXISTS
    v_french_customers,
    v_product_prices,
    v_office_contacts;
```

> `DROP VIEW` chỉ xóa định nghĩa view; không xóa dữ liệu của các bảng cơ sở.

### Bài tập thực hành

**Bài 14.1.** Xóa view `v_french_customers` nếu view tồn tại.

**Bài 14.2.** Xóa đồng thời `v_product_prices` và `v_office_contacts` bằng một lệnh.

**Bài 14.3.** Viết lệnh `DROP VIEW` an toàn để xóa `v_sales_rep_list` nếu có.

**Bài 14.4.** Giải thích `DROP VIEW` khác `DROP TABLE` như thế nào.

**Bài 14.5.** Sau khi xóa một view, dùng câu lệnh nào để xác nhận view không còn trong database?

---

## 15. Một số lỗi thường gặp

### Lỗi 1. Dùng trùng tên với bảng cơ sở

Trong cùng một database, table và view dùng chung namespace. Không thể tạo view có tên trùng với bảng đã tồn tại.

Sai:

```sql
CREATE VIEW customers AS
SELECT *
FROM customers;
```

### Lỗi 2. Tên cột view bị trùng

Sai hoặc có thể bị từ chối:

```sql
CREATE VIEW v_bad AS
SELECT c.customerNumber,
       p.customerNumber
FROM customers AS c
JOIN payments AS p
    ON c.customerNumber = p.customerNumber;
```

Nên đặt alias:

```sql
CREATE VIEW v_good AS
SELECT c.customerNumber AS customerNumber,
       p.checkNumber AS checkNumber
FROM customers AS c
JOIN payments AS p
    ON c.customerNumber = p.customerNumber;
```

### Lỗi 3. Cho rằng mọi view đều update được

View dùng `GROUP BY`, aggregate, `DISTINCT`, `UNION`, `HAVING` hoặc `TEMPTABLE` thường không thể cập nhật.

### Lỗi 4. Dùng `SELECT *` và kỳ vọng view tự nhận cột mới

Nếu view được tạo bằng `SELECT *`, các cột được chốt tại thời điểm tạo. Cột mới thêm vào bảng cơ sở không tự xuất hiện trong view cũ.

### Lỗi 5. Không dùng `WITH CHECK OPTION` khi view thể hiện quy tắc nghiệp vụ

Ví dụ view chỉ dành cho khách hàng `USA`. Nếu không có `WITH CHECK OPTION`, update qua view có thể làm dòng không còn thuộc `USA`.

### Lỗi 6. Xóa hoặc đổi tên view đang được view khác tham chiếu

Các view phụ thuộc có thể bị lỗi khi truy vấn sau đó.

### Bài tập thực hành

**Bài 15.1.** Sửa lỗi tên cột trùng trong ví dụ `v_bad`.

**Bài 15.2.** Nêu ba cấu trúc khiến một view thường không update được.

**Bài 15.3.** Giải thích vì sao không nên dùng `SELECT *` cho view cần ổn định lâu dài.

**Bài 15.4.** Nêu tình huống cần dùng `WITH CHECK OPTION`.

**Bài 15.5.** Viết một quy trình ngắn để kiểm tra dependency trước khi đổi tên hoặc xóa view.

---

## 16. Bài tập tổng hợp

### Bài 16.1. View phục vụ nhân viên bán hàng

Tạo view `v_sales_customer_summary` bao gồm:

- `customerNumber`, `customerName`, `country`, `creditLimit` từ `customers`.
- Tên sales representative từ `employees`.
- Chỉ hiển thị khách hàng đã có sales representative.

Sau đó truy vấn view, sắp xếp theo quốc gia và tên khách hàng.

### Bài 16.2. View báo cáo thanh toán

Tạo view `v_customer_payment_report` gồm:

- `customerNumber`, `customerName`.
- Số khoản thanh toán.
- Tổng tiền thanh toán.
- Chỉ giữ khách hàng có tổng thanh toán lớn hơn `50000`.

Xác định view này có update được không và giải thích.

### Bài 16.3. Updatable view có CHECK OPTION

Tạo view `v_usa_customer_credit_check` từ bảng `customers`:

- Chỉ chứa khách hàng tại `USA`.
- Có `WITH CHECK OPTION`.
- Bao gồm các cột cần thiết để có thể đọc và cập nhật dữ liệu khách hàng.

Trong transaction, thử cập nhật một dòng thuộc view sao cho `country` đổi thành `France`, quan sát kết quả, rồi `ROLLBACK`.

### Bài 16.4. Quản lý metadata view

Với một view bạn đã tạo:

1. Liệt kê view bằng `SHOW FULL TABLES`.
2. Xem `IS_UPDATABLE` qua `INFORMATION_SCHEMA.VIEWS`.
3. Xem định nghĩa bằng `SHOW CREATE VIEW`.
4. Đổi tên view.
5. Đổi lại tên cũ.

### Bài 16.5. Dọn dẹp view thực hành

Viết một câu `DROP VIEW IF EXISTS` xóa tất cả view đã tạo trong bài này.

---

## 17. Đáp án gợi ý cho một số bài tập

### Bài 4.1

```sql
CREATE VIEW v_french_customers AS
SELECT customerNumber,
       customerName,
       city,
       creditLimit
FROM customers
WHERE country = 'France';
```

### Bài 4.2

```sql
CREATE VIEW v_product_prices AS
SELECT productCode,
       productName,
       productLine,
       buyPrice,
       MSRP,
       MSRP - buyPrice AS priceDifference
FROM products;
```

### Bài 4.4

```sql
CREATE VIEW v_sales_rep_list AS
SELECT e.employeeNumber,
       CONCAT(e.firstName, ' ', e.lastName) AS employeeName,
       e.email,
       o.city AS officeCity,
       o.country AS officeCountry
FROM employees AS e
JOIN offices AS o
    ON e.officeCode = o.officeCode
WHERE e.jobTitle = 'Sales Rep';
```

### Bài 6.2

```sql
CREATE ALGORITHM = TEMPTABLE VIEW v_payment_summary_temp AS
SELECT customerNumber,
       SUM(amount) AS totalPayment
FROM payments
GROUP BY customerNumber;
```

### Bài 7.2

```sql
START TRANSACTION;

UPDATE v_editable_customer_phone
SET phone = '+00 000 000 000'
WHERE customerNumber = 103;

ROLLBACK;
```

### Bài 9.4

```sql
CREATE VIEW v_high_credit_customers_check AS
SELECT customerNumber,
       customerName,
       creditLimit
FROM customers
WHERE creditLimit >= 50000
WITH CHECK OPTION;
```

### Bài 11.2

```sql
SELECT TABLE_NAME,
       CHECK_OPTION,
       IS_UPDATABLE
FROM INFORMATION_SCHEMA.VIEWS
WHERE TABLE_SCHEMA = 'classicmodels'
ORDER BY TABLE_NAME;
```

### Bài 12.1

```sql
SHOW CREATE VIEW v_usa_customers;
```

### Bài 13.1

```sql
RENAME TABLE v_usa_customers TO v_usa_customers_renamed;
```

### Bài 14.2

```sql
DROP VIEW IF EXISTS
    v_product_prices,
    v_office_contacts;
```

### Bài 16.5

```sql
DROP VIEW IF EXISTS
    v_french_customers,
    v_product_prices,
    v_office_contacts,
    v_sales_rep_list,
    v_usa_customers,
    v_customer_contacts,
    v_sales_reps,
    v_customer_payment_summary,
    v_editable_customers,
    v_editable_customer_phone,
    v_usa_customers_check,
    v_usa_customers_base,
    v_high_credit_local,
    v_high_credit_cascaded,
    v_usa_customers_renamed,
    v_customer_sales_rep,
    v_sales_customer_summary,
    v_customer_payment_report,
    v_usa_customer_credit_check;
```

---

## 18. Tóm tắt

- View là đối tượng được định nghĩa bởi `SELECT` và được truy vấn gần giống bảng.
- `CREATE VIEW` tạo view; `CREATE OR REPLACE VIEW` thay thế định nghĩa view đã tồn tại.
- `MERGE`, `TEMPTABLE`, `UNDEFINED` là các cách MySQL xử lý view.
- View đơn bảng, không aggregate thường có khả năng update; view dùng `GROUP BY`, aggregate, `DISTINCT`, `UNION`, `HAVING` thường không update được.
- `WITH CHECK OPTION` bảo vệ điều kiện `WHERE` của view khi cập nhật hoặc chèn dữ liệu qua view.
- `LOCAL` và `CASCADED` xác định phạm vi kiểm tra trong chuỗi view lồng nhau; mặc định là `CASCADED`.
- `SHOW FULL TABLES`, `INFORMATION_SCHEMA.VIEWS`, `SHOW CREATE VIEW` hỗ trợ kiểm tra metadata và định nghĩa view.
- Có thể đổi tên view bằng `RENAME TABLE` và xóa view bằng `DROP VIEW IF EXISTS`.
- Tạo/xóa/đổi tên view cần quản lý dependency cẩn thận và luôn có câu lệnh dọn dẹp trong môi trường lab.

---

## 19. Từ khóa chính

- View
- CREATE VIEW
- CREATE OR REPLACE VIEW
- ALTER VIEW
- Updatable View
- Insertable View
- View Algorithm
- MERGE
- TEMPTABLE
- UNDEFINED
- WITH CHECK OPTION
- LOCAL
- CASCADED
- SHOW FULL TABLES
- INFORMATION_SCHEMA.VIEWS
- SHOW CREATE VIEW
- RENAME TABLE
- DROP VIEW
- classicmodels
