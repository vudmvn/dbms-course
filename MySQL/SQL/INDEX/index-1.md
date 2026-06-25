---
title: "Tutorial 1: Tạo và quản lý Index trong MySQL"
author: "Tên giảng viên"
duration: "150m"
difficulty: "Intermediate"
prerequisites:
  - "Đã biết SELECT, WHERE, ORDER BY, JOIN và CREATE TABLE"
  - "Đã biết dùng SHOW CREATE TABLE và SHOW INDEX"
  - "Có quyền CREATE, ALTER, INDEX và DROP trên schema lab"
summary: "Giải thích index là gì; thực hành tạo, liệt kê, phân tích và xóa single-column hoặc composite index trong schema index_lab."
---

# Tutorial 1: Tạo và quản lý Index trong MySQL

## Link tham khảo

- [MySQL Tutorial — MySQL Index](https://www.mysqltutorial.org/mysql-index/)
- [MySQL Tutorial — CREATE INDEX](https://www.mysqltutorial.org/mysql-index/mysql-create-index/)
- [MySQL Tutorial — DROP INDEX](https://www.mysqltutorial.org/mysql-index/mysql-drop-index/)
- [MySQL Tutorial — SHOW INDEXES](https://www.mysqltutorial.org/mysql-index/mysql-show-indexes/)
- [MySQL 8.4 Reference Manual — Optimization and Indexes](https://dev.mysql.com/doc/refman/8.4/en/optimization-indexes.html)
- [MySQL 8.4 Reference Manual — Column Indexes](https://dev.mysql.com/doc/refman/8.4/en/column-indexes.html)
- [MySQL 8.4 Reference Manual — CREATE INDEX](https://dev.mysql.com/doc/refman/8.4/en/create-index.html)
- [MySQL 8.4 Reference Manual — DROP INDEX](https://dev.mysql.com/doc/refman/8.4/en/drop-index.html)
- [MySQL 8.4 Reference Manual — SHOW INDEX](https://dev.mysql.com/doc/refman/8.4/en/show-index.html)

> **Phạm vi lab:** Index là database object ảnh hưởng đến storage, tốc độ ghi và execution plan. Chỉ tạo/xóa index trong schema `index_lab` hoặc môi trường test. Không thử trực tiếp trên bảng production hay bảng dùng chung nếu chưa đo workload, kiểm tra query plan và có kế hoạch rollback.

## 1. Mục tiêu học tập

Sau khi hoàn thành tutorial này, người học có thể:

1. Giải thích index là gì và trade-off giữa tốc độ đọc với chi phí ghi/lưu trữ.
2. Tạo single-column index bằng `CREATE INDEX` hoặc `ALTER TABLE`.
3. Tạo composite index cho mẫu truy vấn nhiều điều kiện.
4. Đọc metadata index bằng `SHOW INDEX`, `SHOW CREATE TABLE` và `INFORMATION_SCHEMA.STATISTICS`.
5. Xóa secondary index bằng `DROP INDEX` hoặc `ALTER TABLE ... DROP INDEX`.
6. Phân biệt primary key, secondary index và composite index ở mức thực hành.
7. Dùng `EXPLAIN` để kiểm tra kế hoạch trước/sau khi tạo index.
8. Chọn tên index rõ nghĩa và tránh tạo index trùng lặp.
9. Dùng `ANALYZE TABLE` như một bước làm mới statistics khi phù hợp.
10. Nhận biết trường hợp optimizer vẫn có thể không dùng index vừa tạo.

---

## 2. Index là gì?

Index là một cấu trúc dữ liệu giúp MySQL tìm row nhanh hơn cho một số điều kiện và thứ tự truy vấn.

Ví dụ:

```sql
SELECT customer_id, first_name, last_name, credit_limit
FROM customer_search
WHERE country = 'USA';
```

Nếu có index trên `country`, MySQL có thể dùng index để tìm nhanh các row khớp thay vì đọc mọi row trong table, tùy vào dữ liệu và optimizer plan.

Với B-tree index, MySQL có thể hỗ trợ tra cứu theo:

```text
=
>
>=
<
<=
BETWEEN
IN (...)
```

và một số mẫu `LIKE 'prefix%'`.

### 2.1. Index không làm mọi query nhanh hơn

Index có lợi khi query cần chọn một phần tương đối nhỏ dữ liệu hoặc khi index hỗ trợ join, sort hoặc grouping phù hợp.

Index có chi phí:

| Chi phí | Diễn giải |
|---|---|
| Storage | Mỗi index chiếm thêm disk/memory cache |
| INSERT | Mỗi row insert có thể phải thêm entry cho nhiều index |
| UPDATE | Update index key cần cập nhật index |
| DELETE | Delete row cần xóa entry trong index |
| Optimizer complexity | Quá nhiều index làm lựa chọn plan phức tạp hơn |
| DDL impact | Tạo/xóa index trên table lớn có thể tốn thời gian và ảnh hưởng workload |

Không tạo index cho mọi cột. Cần dựa vào query workload, `EXPLAIN`, dữ liệu thực tế và đo đạc.

### 2.2. Primary key và secondary index

| Loại | Ví dụ | Vai trò |
|---|---|---|
| Primary key | `PRIMARY KEY(customer_id)` | Định danh row; InnoDB dùng làm clustered index |
| Secondary index | `INDEX idx_country(country)` | Hỗ trợ lookup/sort/join ngoài primary key |
| Composite index | `INDEX idx_country_credit(country, credit_limit)` | Hỗ trợ mẫu query dùng nhiều cột theo thứ tự key parts |

### 2.3. Mô hình “mục lục sách”

Index có thể hình dung như mục lục của sách:

- Không có mục lục: đọc nhiều trang để tìm chủ đề.
- Có mục lục: tìm từ khóa → đến vùng trang có khả năng chứa nội dung.
- Mục lục cũng phải được cập nhật khi sách thay đổi.

Đây chỉ là trực giác. Database index có cấu trúc, cache, concurrency và optimizer phức tạp hơn.

### Bài tập thực hành

**Bài 2.1.** Giải thích index là gì bằng một hoặc hai câu.

**Bài 2.2.** Nêu hai lợi ích của index.

**Bài 2.3.** Nêu hai chi phí của việc thêm index.

**Bài 2.4.** Phân biệt primary key với secondary index.

**Bài 2.5.** Giải thích vì sao “tạo index cho mọi cột” là chiến lược không tốt.

---

## 3. Chuẩn bị schema `index_lab`

### 3.1. Tạo database và bảng lab

```sql
CREATE DATABASE IF NOT EXISTS index_lab;

USE index_lab;
```

Tạo bảng khách hàng dùng để thực hành:

```sql
DROP TABLE IF EXISTS customer_search;

CREATE TABLE customer_search (
    customer_id INT NOT NULL,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(120) NULL,
    country VARCHAR(50) NOT NULL,
    credit_limit DECIMAL(10,2) NOT NULL DEFAULT 0.00,
    PRIMARY KEY (customer_id)
) ENGINE = InnoDB;
```

Nạp dữ liệu từ `classicmodels`:

```sql
INSERT INTO customer_search (
    customer_id,
    first_name,
    last_name,
    email,
    country,
    credit_limit
)
SELECT customerNumber,
       COALESCE(contactFirstName, ''),
       COALESCE(contactLastName, ''),
       CONCAT('customer', customerNumber, '@example.test'),
       country,
       COALESCE(creditLimit, 0)
FROM classicmodels.customers;
```

Tạo bảng order đơn giản:

```sql
DROP TABLE IF EXISTS order_search;

CREATE TABLE order_search (
    order_number INT NOT NULL,
    customer_id INT NOT NULL,
    order_date DATE NOT NULL,
    required_date DATE NOT NULL,
    shipped_date DATE NULL,
    status VARCHAR(20) NOT NULL,
    PRIMARY KEY (order_number)
) ENGINE = InnoDB;
```

```sql
INSERT INTO order_search (
    order_number,
    customer_id,
    order_date,
    required_date,
    shipped_date,
    status
)
SELECT orderNumber,
       customerNumber,
       orderDate,
       requiredDate,
       shippedDate,
       status
FROM classicmodels.orders;
```

Tạo bảng product:

```sql
DROP TABLE IF EXISTS product_search;

CREATE TABLE product_search (
    product_code VARCHAR(15) NOT NULL,
    product_name VARCHAR(100) NOT NULL,
    product_line VARCHAR(50) NOT NULL,
    quantity_in_stock SMALLINT NOT NULL,
    buy_price DECIMAL(10,2) NOT NULL,
    msrp DECIMAL(10,2) NOT NULL,
    PRIMARY KEY (product_code)
) ENGINE = InnoDB;
```

```sql
INSERT INTO product_search (
    product_code,
    product_name,
    product_line,
    quantity_in_stock,
    buy_price,
    msrp
)
SELECT productCode,
       productName,
       productLine,
       quantityInStock,
       buyPrice,
       MSRP
FROM classicmodels.products;
```

### 3.2. Kiểm tra dữ liệu

```sql
SELECT COUNT(*) AS totalCustomers
FROM customer_search;

SELECT COUNT(*) AS totalOrders
FROM order_search;

SELECT COUNT(*) AS totalProducts
FROM product_search;
```

### 3.3. Lưu ý về quy mô dữ liệu lab

`classicmodels` thường nhỏ. Vì vậy, optimizer có thể chọn table scan dù index tồn tại, vì đọc toàn table nhỏ đôi khi rẻ hơn đi qua index.

Trong lab, mục tiêu chính là:

- Đọc DDL index.
- Quan sát `EXPLAIN`.
- So sánh `possible_keys`, `key`, `rows`, `Extra`.
- Hiểu mẫu query phù hợp với index.

Không lấy thời gian chạy trên dữ liệu nhỏ làm bằng chứng duy nhất về thiết kế production.

### Bài tập thực hành

**Bài 3.1.** Tạo database `index_lab`.

**Bài 3.2.** Tạo bảng `customer_search` và nạp dữ liệu từ `classicmodels.customers`.

**Bài 3.3.** Tạo bảng `order_search` và nạp dữ liệu từ `classicmodels.orders`.

**Bài 3.4.** Kiểm tra số row của ba bảng lab.

**Bài 3.5.** Giải thích vì sao small table có thể không dùng index dù index tồn tại.

---

## 4. Tạo index bằng `CREATE INDEX`

### 4.1. Single-column index

Cú pháp:

```sql
CREATE INDEX index_name
ON table_name (column_name);
```

Ví dụ tạo index hỗ trợ lọc theo quốc gia:

```sql
CREATE INDEX idx_customer_country
ON customer_search (country);
```

Kiểm tra:

```sql
SHOW INDEX FROM customer_search;
```

Dùng query:

```sql
EXPLAIN
SELECT customer_id, first_name, last_name, credit_limit
FROM customer_search
WHERE country = 'USA';
```

### 4.2. Naming convention

Tên index nên mô tả:

```text
idx_<table_or_feature>_<column_or_pattern>
```

Ví dụ:

```text
idx_customer_country
idx_order_customer_date
idx_product_line_msrp
```

Với unique index:

```text
uq_customer_email
```

Không đặt tên mơ hồ:

```text
idx1
index_a
new_index
```

### 4.3. Tạo index nhiều cột

```sql
CREATE INDEX idx_customer_country_credit
ON customer_search (country, credit_limit);
```

Đây là **một composite index**, không phải hai index độc lập.

Nó phù hợp với các query như:

```sql
SELECT customer_id, first_name, last_name, credit_limit
FROM customer_search
WHERE country = 'USA'
  AND credit_limit >= 50000.00;
```

Chi tiết về thứ tự cột của composite index sẽ học sâu ở Tutorial 2.

### 4.4. Tạo index trong `CREATE TABLE`

Index có thể được khai báo ngay khi tạo table:

```sql
CREATE TABLE example_orders (
    order_id INT NOT NULL,
    customer_id INT NOT NULL,
    order_date DATE NOT NULL,
    PRIMARY KEY (order_id),
    INDEX idx_customer_date (customer_id, order_date)
) ENGINE = InnoDB;
```

### 4.5. Tạo index bằng `ALTER TABLE`

```sql
ALTER TABLE product_search
ADD INDEX idx_product_line (product_line);
```

Hoặc:

```sql
ALTER TABLE product_search
ADD INDEX idx_product_line_msrp (product_line, msrp);
```

`CREATE INDEX` và `ALTER TABLE ... ADD INDEX` đều là cách hợp lệ để thêm secondary index. Chọn một style thống nhất trong project.

### Bài tập thực hành

**Bài 4.1.** Tạo index `idx_customer_country` trên `customer_search(country)`.

**Bài 4.2.** Tạo index `idx_order_status` trên `order_search(status)`.

**Bài 4.3.** Tạo composite index `idx_order_customer_date` trên `order_search(customer_id, order_date)`.

**Bài 4.4.** Dùng `ALTER TABLE` tạo index `idx_product_line` trên `product_search(product_line)`.

**Bài 4.5.** Viết query phù hợp với index `(customer_id, order_date)`.

---

## 5. Kiểm tra index bằng `SHOW INDEX`

### 5.1. Cú pháp

```sql
SHOW INDEX FROM table_name;
```

Các alias thường gặp:

```sql
SHOW INDEXES FROM table_name;
SHOW KEYS FROM table_name;
```

Ví dụ:

```sql
SHOW INDEX FROM order_search;
```

### 5.2. Các cột quan trọng

| Cột output | Ý nghĩa |
|---|---|
| `Table` | Tên table |
| `Non_unique` | `0` nếu unique, `1` nếu index cho phép trùng |
| `Key_name` | Tên index |
| `Seq_in_index` | Vị trí cột trong index |
| `Column_name` | Cột index key part |
| `Collation` | Hướng lưu key part, thường `A` hoặc `D` |
| `Cardinality` | Ước lượng distinct values |
| `Sub_part` | Prefix length nếu là prefix index |
| `Index_type` | Ví dụ `BTREE` |
| `Visible` | `YES`/`NO` nếu server hỗ trợ invisible index |

Ví dụ nhìn composite index:

```sql
SHOW INDEX FROM order_search
WHERE Key_name = 'idx_order_customer_date';
```

Nếu output có hai dòng:

```text
Seq_in_index = 1, Column_name = customer_id
Seq_in_index = 2, Column_name = order_date
```

thì thứ tự key parts là:

```text
(customer_id, order_date)
```

### 5.3. `SHOW CREATE TABLE`

`SHOW INDEX` tốt cho overview index metadata. `SHOW CREATE TABLE` cho DDL tổng thể:

```sql
SHOW CREATE TABLE order_search;
```

Dùng lệnh này để xem:

- Columns.
- Primary key.
- Secondary indexes.
- Unique indexes.
- Engine.
- Table options.

### 5.4. `INFORMATION_SCHEMA.STATISTICS`

Khi cần query metadata theo điều kiện:

```sql
SELECT INDEX_NAME,
       NON_UNIQUE,
       SEQ_IN_INDEX,
       COLUMN_NAME,
       COLLATION,
       CARDINALITY,
       SUB_PART,
       INDEX_TYPE,
       IS_VISIBLE
FROM INFORMATION_SCHEMA.STATISTICS
WHERE TABLE_SCHEMA = 'index_lab'
  AND TABLE_NAME = 'order_search'
ORDER BY INDEX_NAME, SEQ_IN_INDEX;
```

### Bài tập thực hành

**Bài 5.1.** Dùng `SHOW INDEX FROM customer_search`.

**Bài 5.2.** Dùng `SHOW INDEX FROM order_search WHERE Key_name = 'idx_order_customer_date';`.

**Bài 5.3.** Xác định thứ tự cột của `idx_order_customer_date`.

**Bài 5.4.** Dùng `SHOW CREATE TABLE order_search`.

**Bài 5.5.** Viết query `INFORMATION_SCHEMA.STATISTICS` liệt kê metadata index của `product_search`.

---

## 6. Dùng `EXPLAIN` để kiểm tra index usage

### 6.1. Cú pháp

```sql
EXPLAIN
SELECT ...
```

Ví dụ:

```sql
EXPLAIN
SELECT order_number, order_date, status
FROM order_search
WHERE customer_id = 103
  AND order_date >= '2004-01-01';
```

Các cột cần quan sát ban đầu:

| Cột | Cách đọc cơ bản |
|---|---|
| `possible_keys` | Các index optimizer có thể cân nhắc |
| `key` | Index được chọn thực tế; `NULL` nghĩa là không chọn index |
| `key_len` | Độ dài key part được dùng |
| `rows` | Số row optimizer ước lượng cần xem |
| `filtered` | Tỷ lệ row ước lượng qua filter |
| `Extra` | Thông tin bổ sung như `Using where`, `Using index`, `Using filesort` |

Ví dụ tạo index rồi so sánh:

```sql
EXPLAIN
SELECT order_number, order_date, status
FROM order_search
WHERE customer_id = 103
ORDER BY order_date;
```

Sau đó:

```sql
CREATE INDEX idx_order_customer_date
ON order_search (customer_id, order_date);
```

Chạy `EXPLAIN` lại.

### 6.2. Không coi `key = NULL` luôn là lỗi

Optimizer có thể quyết định full table scan nếu:

- Table rất nhỏ.
- Điều kiện trả về phần lớn row.
- Index không đủ selective.
- Cost estimate cho table scan thấp hơn.
- Query expression không phù hợp với index.

Cần đánh giá bằng query, plan và workload thực, không chỉ nhìn một dấu hiệu.

### 6.3. `ANALYZE TABLE`

Khi statistics cũ hoặc sau thay đổi dữ liệu lớn, có thể xem xét:

```sql
ANALYZE TABLE order_search;
```

Sau đó chạy lại `EXPLAIN`.

Không chạy maintenance operations tùy tiện trên production. Cần hiểu impact theo storage engine, version và workload.

### Bài tập thực hành

**Bài 6.1.** Chạy `EXPLAIN` cho query lọc `order_search` theo `customer_id`.

**Bài 6.2.** Xác định `possible_keys` và `key` trong output.

**Bài 6.3.** Chạy `EXPLAIN` cho query lọc theo `status`.

**Bài 6.4.** Chạy `ANALYZE TABLE order_search;` trong schema lab.

**Bài 6.5.** Nêu hai lý do optimizer có thể không dùng index trên table nhỏ.

---

## 7. Xóa index

### 7.1. `DROP INDEX`

Cú pháp:

```sql
DROP INDEX index_name
ON table_name;
```

Ví dụ:

```sql
DROP INDEX idx_customer_country
ON customer_search;
```

### 7.2. `ALTER TABLE ... DROP INDEX`

```sql
ALTER TABLE customer_search
DROP INDEX idx_customer_country;
```

Hai cách trên đều dùng để xóa secondary index.

### 7.3. Không dùng `DROP INDEX` để xóa primary key

Primary key là constraint/index đặc biệt.

Để xóa primary key:

```sql
ALTER TABLE table_name
DROP PRIMARY KEY;
```

Không làm việc này trong production chỉ để “thử index”. Primary key có thể liên quan foreign keys, clustered storage layout và application assumptions.

### 7.4. Kiểm tra trước khi drop

Trước khi drop:

```sql
SHOW INDEX FROM customer_search;
```

Sau drop:

```sql
SHOW INDEX FROM customer_search;
```

Cần kiểm tra:

- Index có thật sự redundant không?
- Có query hay foreign key phụ thuộc index không?
- Có cần dùng invisible index để thử impact trước không?
- Có migration/rollback script không?

### 7.5. DDL và transaction

Tạo/xóa index là DDL. Trong MySQL, DDL thường có transactional behavior khác DML và có thể gây implicit commit. Không giả định có thể `ROLLBACK` một `CREATE INDEX` hoặc `DROP INDEX` như rollback `INSERT`.

Trên table lớn, hoạt động DDL cần được lên kế hoạch theo version, engine và yêu cầu availability.

### Bài tập thực hành

**Bài 7.1.** Xóa `idx_customer_country` bằng `DROP INDEX`.

**Bài 7.2.** Tạo lại index đó bằng `CREATE INDEX`.

**Bài 7.3.** Xóa `idx_product_line` bằng `ALTER TABLE ... DROP INDEX`.

**Bài 7.4.** Giải thích vì sao không dùng `DROP INDEX` để xóa primary key.

**Bài 7.5.** Nêu hai kiểm tra cần làm trước khi drop index trên production.

---

## 8. Thiết kế index cơ bản

### 8.1. Bắt đầu từ workload

Không bắt đầu bằng câu hỏi:

```text
Table này có cột nào chưa có index?
```

Hãy bắt đầu bằng:

```text
Các query quan trọng nào chạy thường xuyên, chậm hoặc phục vụ nghiệp vụ chính?
```

Ví dụ workload:

```sql
SELECT *
FROM order_search
WHERE customer_id = ?
ORDER BY order_date DESC
LIMIT 20;
```

Index ứng viên:

```sql
CREATE INDEX idx_order_customer_date_desc
ON order_search (customer_id, order_date DESC);
```

Ví dụ:

```sql
SELECT *
FROM customer_search
WHERE country = ?
  AND credit_limit >= ?;
```

Index ứng viên:

```sql
CREATE INDEX idx_customer_country_credit
ON customer_search (country, credit_limit);
```

### 8.2. Tránh index dư thừa

Nếu đã có:

```text
(customer_id, order_date)
```

thì một index riêng chỉ trên `customer_id` có thể trở nên dư thừa cho một số workload, vì composite index có thể hỗ trợ leftmost key part `customer_id`.

Tuy nhiên, không drop index chỉ bằng quy tắc này. Cần kiểm tra:

- Unique/foreign-key requirements.
- Query patterns.
- Covering behavior.
- Existing plans.
- Write cost và object dependencies.

### 8.3. Cột nào thường là ứng viên?

Cần xem xét cột được dùng thường xuyên trong:

- `WHERE`.
- Join conditions.
- `ORDER BY`.
- `GROUP BY`.
- Foreign-key lookups.

Nhưng selectivity, thứ tự cột, expression, số row trả về và write workload đều quan trọng.

### Bài tập thực hành

**Bài 8.1.** Đề xuất index cho query lọc `customer_id` rồi sắp xếp `order_date DESC`.

**Bài 8.2.** Đề xuất index cho query lọc `country` và `credit_limit`.

**Bài 8.3.** Nêu ba nơi trong query thường là nguồn để tìm index candidate.

**Bài 8.4.** Giải thích vì sao cột xuất hiện trong `WHERE` không tự động có nghĩa phải tạo index.

**Bài 8.5.** Nêu ba yếu tố cần kiểm tra trước khi drop một index bị nghi dư thừa.

---

## 9. Bài tập tổng hợp

### Bài 9.1. Quản lý index customer

1. Tạo index trên `customer_search(country)`.
2. Tạo composite index trên `(country, credit_limit)`.
3. Liệt kê index bằng `SHOW INDEX`.
4. Dùng `EXPLAIN` cho query lọc country + credit limit.
5. Xóa index single-column nếu bạn xác nhận chỉ dùng để lab.

### Bài 9.2. Quản lý index order

1. Tạo index `(customer_id, order_date)`.
2. Dùng `SHOW CREATE TABLE`.
3. Chạy `EXPLAIN` cho query theo customer và date range.
4. Chạy `ANALYZE TABLE`.
5. So sánh plan trước/sau theo các cột `possible_keys`, `key`, `rows`.

### Bài 9.3. Metadata report

Viết query từ `INFORMATION_SCHEMA.STATISTICS` trả về:

1. Table name.
2. Index name.
3. Non-unique flag.
4. Sequence in index.
5. Column name.
6. Cardinality.
7. Visibility.

Chỉ áp dụng cho schema `index_lab`.

### Bài 9.4. Index decision

Với query:

```sql
SELECT product_code, product_name, msrp
FROM product_search
WHERE product_line = 'Motorcycles'
ORDER BY msrp DESC;
```

1. Đề xuất composite index.
2. Viết `CREATE INDEX`.
3. Dùng `EXPLAIN`.
4. Nêu vì sao data nhỏ có thể vẫn scan.
5. Nêu cách rollback lab index.

### Bài 9.5. Cleanup

Xóa các secondary index lab đã tạo, nhưng giữ primary keys.

---

## 10. Đáp án gợi ý

### Bài 4.3

```sql
CREATE INDEX idx_order_customer_date
ON order_search (customer_id, order_date);
```

### Bài 5.5

```sql
SELECT INDEX_NAME,
       NON_UNIQUE,
       SEQ_IN_INDEX,
       COLUMN_NAME,
       COLLATION,
       CARDINALITY,
       SUB_PART,
       INDEX_TYPE,
       IS_VISIBLE
FROM INFORMATION_SCHEMA.STATISTICS
WHERE TABLE_SCHEMA = 'index_lab'
  AND TABLE_NAME = 'product_search'
ORDER BY INDEX_NAME, SEQ_IN_INDEX;
```

### Bài 6.1

```sql
EXPLAIN
SELECT order_number, order_date, status
FROM order_search
WHERE customer_id = 103;
```

### Bài 7.1

```sql
DROP INDEX idx_customer_country
ON customer_search;
```

### Bài 8.1

```sql
CREATE INDEX idx_order_customer_date_desc
ON order_search (customer_id, order_date DESC);
```

### Bài 9.4

```sql
CREATE INDEX idx_product_line_msrp_desc
ON product_search (product_line, msrp DESC);

EXPLAIN
SELECT product_code, product_name, msrp
FROM product_search
WHERE product_line = 'Motorcycles'
ORDER BY msrp DESC;
```

### Cleanup

```sql
DROP INDEX IF EXISTS idx_customer_country
ON customer_search;

DROP INDEX IF EXISTS idx_customer_country_credit
ON customer_search;

DROP INDEX IF EXISTS idx_order_status
ON order_search;

DROP INDEX IF EXISTS idx_order_customer_date
ON order_search;

DROP INDEX IF EXISTS idx_order_customer_date_desc
ON order_search;

DROP INDEX IF EXISTS idx_product_line
ON product_search;

DROP INDEX IF EXISTS idx_product_line_msrp_desc
ON product_search;
```

> `DROP INDEX IF EXISTS` is supported by current MySQL versions. If your server rejects it, check `SHOW INDEX` first and use `DROP INDEX index_name ON table_name` only when the index exists.

---

## 11. Tóm tắt

- Index hỗ trợ lookup, join, sort hoặc grouping cho các query phù hợp.
- Index có chi phí storage và làm INSERT/UPDATE/DELETE đắt hơn.
- Dùng `CREATE INDEX` hoặc `ALTER TABLE ... ADD INDEX` để tạo secondary index.
- Dùng `SHOW INDEX`, `SHOW CREATE TABLE` và `INFORMATION_SCHEMA.STATISTICS` để kiểm tra metadata.
- Dùng `EXPLAIN` để quan sát plan; không mặc định mọi index phải được optimizer chọn.
- Dùng `DROP INDEX` hoặc `ALTER TABLE ... DROP INDEX` để xóa secondary index.
- Thiết kế index dựa trên workload, không dựa trên việc “mỗi cột một index”.
- Tạo/xóa index là DDL, cần thực hiện thận trọng trên table lớn hoặc production.

---

## 12. Từ khóa chính

- Index
- Secondary index
- Primary key
- Composite index
- CREATE INDEX
- ALTER TABLE ADD INDEX
- DROP INDEX
- SHOW INDEX
- INFORMATION_SCHEMA.STATISTICS
- EXPLAIN
- ANALYZE TABLE
- Cardinality
- Selectivity
- Query plan
- InnoDB
- DDL
