---
title: "Tutorial 2: MySQL Index Types và chiến lược chọn Index"
author: "Tên giảng viên"
duration: "210m"
difficulty: "Intermediate–Advanced"
prerequisites:
  - "Đã hoàn thành Tutorial 1 về tạo và quản lý index"
  - "Đã biết EXPLAIN và composite index cơ bản"
  - "MySQL 8.x được khuyến nghị cho invisible, descending và functional indexes"
summary: "Thực hành unique, prefix, invisible, descending, composite, clustered, cardinality và functional indexes; đồng thời xây dựng tư duy chọn index theo query pattern."
---

# Tutorial 2: MySQL Index Types và chiến lược chọn Index

## Link tham khảo

- [MySQL Tutorial — MySQL Index](https://www.mysqltutorial.org/mysql-index/)
- [MySQL Tutorial — UNIQUE Index](https://www.mysqltutorial.org/mysql-index/mysql-unique-index/)
- [MySQL Tutorial — Prefix Index](https://www.mysqltutorial.org/mysql-index/mysql-prefix-index/)
- [MySQL Tutorial — Invisible Index](https://www.mysqltutorial.org/mysql-index/mysql-invisible-index/)
- [MySQL Tutorial — Descending Index](https://www.mysqltutorial.org/mysql-index/mysql-descending-index/)
- [MySQL Tutorial — Composite Index](https://www.mysqltutorial.org/mysql-index/mysql-composite-index/)
- [MySQL Tutorial — Clustered Index](https://www.mysqltutorial.org/mysql-index/mysql-clustered-index/)
- [MySQL Tutorial — Index Cardinality](https://www.mysqltutorial.org/mysql-index/mysql-index-cardinality/)
- [MySQL Tutorial — Functional Index](https://www.mysqltutorial.org/mysql-index/mysql-functional-index/)
- [MySQL 8.4 Reference Manual — Column Indexes](https://dev.mysql.com/doc/refman/8.4/en/column-indexes.html)
- [MySQL 8.4 Reference Manual — Invisible Indexes](https://dev.mysql.com/doc/refman/8.4/en/invisible-indexes.html)
- [MySQL 8.4 Reference Manual — Descending Indexes](https://dev.mysql.com/doc/refman/8.4/en/descending-indexes.html)
- [MySQL 8.4 Reference Manual — Clustered and Secondary Indexes](https://dev.mysql.com/doc/refman/8.4/en/innodb-index-types.html)
- [MySQL 8.4 Reference Manual — Functional Key Parts](https://dev.mysql.com/doc/refman/8.4/en/create-index.html)

> **Phiên bản:** Các ví dụ invisible, descending và functional index yêu cầu server hỗ trợ feature tương ứng. Hãy chạy `SELECT VERSION();` trước khi thực hành. Nếu server cũ không hỗ trợ một feature, học khái niệm và bỏ qua phần DDL đó.

## 1. Mục tiêu học tập

Sau khi hoàn thành tutorial này, người học có thể:

1. Tạo và dùng unique index để duy trì distinctness.
2. Giải thích prefix index và đánh đổi giữa kích thước index với selectivity.
3. Dùng invisible index để kiểm thử tác động của việc bỏ index mà chưa drop ngay.
4. Dùng descending index cho query có mixed sort order.
5. Thiết kế composite index theo leftmost-prefix principle.
6. Giải thích clustered index và secondary index trong InnoDB.
7. Đọc cardinality từ `SHOW INDEX` như một estimate.
8. Tạo functional index cho expression phù hợp.
9. So sánh functional index với query rewrite theo range predicate.
10. Chọn index theo workload thay vì áp dụng “một quy tắc cho mọi query”.

---

## 2. Chuẩn bị

Chạy Tutorial 1 trước để có schema `index_lab` với:

```text
customer_search
order_search
product_search
```

Chọn schema:

```sql
USE index_lab;
```

Kiểm tra server version:

```sql
SELECT VERSION() AS mysqlVersion;
```

Kiểm tra storage engine:

```sql
SHOW TABLE STATUS
WHERE Name IN ('customer_search', 'order_search', 'product_search');
```

Kiểm tra index hiện có:

```sql
SHOW INDEX FROM customer_search;
SHOW INDEX FROM order_search;
SHOW INDEX FROM product_search;
```

> Trong toàn bộ tutorial, hãy tạo index có prefix `idx_` hoặc `uq_`, kiểm tra bằng `SHOW INDEX`, và drop index lab sau khi hoàn thành nếu không còn dùng.

### Bài tập thực hành

**Bài 2.1.** Kiểm tra MySQL version.

**Bài 2.2.** Kiểm tra engine của ba table lab.

**Bài 2.3.** Liệt kê index hiện có của `order_search`.

**Bài 2.4.** Nêu lý do cần biết version trước khi thử functional index.

**Bài 2.5.** Nêu lý do index lab cần naming convention.

---

## 3. Unique indexes

### 3.1. Khái niệm

Unique index đảm bảo không có hai row có cùng giá trị key (theo quy tắc `NULL` của MySQL).

Cú pháp:

```sql
CREATE UNIQUE INDEX index_name
ON table_name (column_name);
```

Ví dụ:

```sql
CREATE UNIQUE INDEX uq_customer_email
ON customer_search (email);
```

Sau đó, insert email đã tồn tại sẽ bị từ chối:

```sql
INSERT INTO customer_search (
    customer_id, first_name, last_name, email, country, credit_limit
)
VALUES (
    999001, 'Test', 'Duplicate',
    'customer103@example.test', 'USA', 1000.00
);
```

### 3.2. UNIQUE index và PRIMARY KEY

| Đặc điểm | PRIMARY KEY | UNIQUE index |
|---|---|---|
| Số lượng trên table | Một | Có thể nhiều |
| NULL | Không cho phép | MySQL thường cho phép nhiều `NULL` nếu column nullable |
| Vai trò | Định danh chính của row | Bảo đảm distinctness cho business key |
| InnoDB | Dùng làm clustered index | Là secondary index trừ khi được InnoDB chọn làm clustered index do không có PK |

Nếu business rule yêu cầu email luôn có và luôn unique:

```sql
ALTER TABLE customer_search
MODIFY email VARCHAR(120) NOT NULL;

CREATE UNIQUE INDEX uq_customer_email
ON customer_search (email);
```

Nếu email optional, cần xác định rõ `NULL` có ý nghĩa gì trong nghiệp vụ.

### 3.3. Không dùng UNIQUE chỉ để tăng tốc

Unique index có thể hỗ trợ lookup, nhưng mục đích chính là integrity. Không tạo unique index nếu data hợp lệ có thể có duplicate values.

### Bài tập thực hành

**Bài 3.1.** Tạo unique index `uq_customer_email`.

**Bài 3.2.** Thử insert một email đã tồn tại và quan sát lỗi.

**Bài 3.3.** Dùng `SHOW INDEX` xác định `Non_unique` của `uq_customer_email`.

**Bài 3.4.** Giải thích khác nhau giữa primary key và unique index.

**Bài 3.5.** Nêu một cột trong hệ thống bán hàng có thể là ứng viên unique index.

---

## 4. Prefix indexes

### 4.1. Khái niệm

Prefix index chỉ index phần đầu của chuỗi.

Cú pháp:

```sql
CREATE INDEX index_name
ON table_name (string_column(prefix_length));
```

Ví dụ:

```sql
CREATE INDEX idx_customer_email_prefix
ON customer_search (email(12));
```

Prefix index hữu ích khi:

- Chuỗi dài.
- Full-column index quá lớn.
- Những ký tự đầu đủ phân biệt phần lớn giá trị truy vấn.
- Query thường dùng `LIKE 'prefix%'`.

### 4.2. Prefix với TEXT/BLOB

Với `TEXT` hoặc `BLOB`, MySQL yêu cầu prefix length khi tạo ordinary index.

Độ dài prefix bị giới hạn theo engine, row format, character set và byte limits. Đặc biệt với charset nhiều byte như `utf8mb4`, số ký tự phải được chọn có xét đến số byte mỗi ký tự.

Không hard-code một prefix length phổ quát. Cần đo:

- Tính selectivity theo prefix.
- Kích thước index.
- Mẫu search thực tế.
- Charset/collation.

### 4.3. Chọn prefix length

Có thể so sánh distinct prefixes:

```sql
SELECT
    COUNT(*) AS totalRows,
    COUNT(DISTINCT LEFT(email, 4)) AS distinctPrefix4,
    COUNT(DISTINCT LEFT(email, 8)) AS distinctPrefix8,
    COUNT(DISTINCT LEFT(email, 12)) AS distinctPrefix12,
    COUNT(DISTINCT email) AS distinctFullEmail
FROM customer_search;
```

Prefix càng dài thường càng selective, nhưng index càng lớn.

### 4.4. Cảnh báo với UNIQUE prefix index

Ví dụ:

```sql
CREATE UNIQUE INDEX uq_email_prefix
ON customer_search (email(12));
```

Câu này chỉ đảm bảo prefix 12 ký tự unique, không đảm bảo full email unique theo cách mong muốn. Trong đa số trường hợp business key như email, cần full unique index, không dùng unique prefix index.

### 4.5. Prefix index không thay thế full-text search

Prefix index phù hợp với pattern bắt đầu bằng chuỗi:

```sql
WHERE email LIKE 'customer10%'
```

Nó không phải giải pháp tổng quát cho:

```sql
WHERE email LIKE '%example%'
```

vì leading wildcard thường không dùng B-tree prefix index theo cách mong muốn. Với search văn bản phong phú, xem xét FULLTEXT hoặc công cụ search phù hợp.

### Bài tập thực hành

**Bài 4.1.** Tạo `idx_customer_email_prefix` trên `email(12)`.

**Bài 4.2.** Dùng `SHOW INDEX` kiểm tra cột `Sub_part`.

**Bài 4.3.** Viết query đếm distinct prefix length 4, 8, 12 của `email`.

**Bài 4.4.** Viết query `LIKE 'customer10%'` phù hợp với prefix search.

**Bài 4.5.** Giải thích vì sao không nên dùng unique prefix index để enforce full email uniqueness.

---

## 5. Invisible indexes

### 5.1. Khái niệm

Invisible index tồn tại và vẫn được duy trì khi data thay đổi, nhưng optimizer mặc định không dùng nó để lập execution plan.

Tạo invisible index:

```sql
CREATE INDEX idx_customer_country_invisible
ON customer_search (country) INVISIBLE;
```

Đổi một index hiện có thành invisible:

```sql
ALTER TABLE customer_search
ALTER INDEX idx_customer_country INVISIBLE;
```

Đưa lại visible:

```sql
ALTER TABLE customer_search
ALTER INDEX idx_customer_country VISIBLE;
```

### 5.2. Khi nào dùng?

Invisible index hữu ích để kiểm thử câu hỏi:

```text
Nếu index này không còn tồn tại với optimizer, các query bị ảnh hưởng thế nào?
```

Thay vì:

```text
DROP INDEX → chạy test → phải build lại index nếu cần
```

có thể:

```text
ALTER INDEX ... INVISIBLE → test → VISIBLE hoặc DROP
```

### 5.3. Các giới hạn quan trọng

- Primary key không thể invisible.
- Một unique index có thể không được phép invisible nếu InnoDB đang dùng nó như implicit clustered key do table không có primary key phù hợp.
- Invisible index vẫn chiếm storage và vẫn được update bởi DML.
- Unique invisible index vẫn enforce uniqueness.
- Optimizer mặc định bỏ qua invisible indexes; có thể kiểm thử đặc biệt với `use_invisible_indexes` theo docs, nhưng không nên đổi global behavior cho mục đích lab đơn giản.

### 5.4. Kiểm tra visibility

```sql
SHOW INDEX FROM customer_search;
```

Hoặc:

```sql
SELECT INDEX_NAME, IS_VISIBLE
FROM INFORMATION_SCHEMA.STATISTICS
WHERE TABLE_SCHEMA = 'index_lab'
  AND TABLE_NAME = 'customer_search';
```

### Bài tập thực hành

**Bài 5.1.** Tạo index visible trên `customer_search(country)` nếu chưa có.

**Bài 5.2.** Đổi index đó thành invisible.

**Bài 5.3.** Dùng `SHOW INDEX` hoặc `INFORMATION_SCHEMA.STATISTICS` kiểm tra visibility.

**Bài 5.4.** Chạy `EXPLAIN` trước và sau khi index invisible cho query lọc `country`.

**Bài 5.5.** Đổi index về visible, sau đó giải thích vì sao invisible không có nghĩa index ngừng được DML maintenance.

---

## 6. Descending indexes

### 6.1. Khái niệm

Descending index lưu key part theo hướng giảm dần. Nó đặc biệt hữu ích cho **mixed sort order** trong composite index.

Ví dụ:

```sql
CREATE INDEX idx_order_status_date_desc
ON order_search (status ASC, order_date DESC);
```

Query phù hợp:

```sql
SELECT order_number, order_date, status
FROM order_search
WHERE status = 'Shipped'
ORDER BY order_date DESC
LIMIT 10;
```

### 6.2. Mixed order là điểm quan trọng

Ví dụ index:

```sql
(status ASC, order_date DESC)
```

có thể phù hợp với:

```sql
ORDER BY status ASC, order_date DESC
```

Trong MySQL InnoDB, descending key parts giúp optimizer dùng forward scan cho những thứ tự có kết hợp ASC/DESC. Đây là trường hợp đáng chú ý hơn so với chỉ một cột đơn lẻ.

### 6.3. Lưu ý

- Descending indexes được hỗ trợ cho InnoDB.
- Không phải mọi query có `ORDER BY ... DESC` đều cần index DESC mới.
- Query vẫn cần thỏa điều kiện về index order, filter, join và selectivity.
- Dùng `EXPLAIN` để kiểm tra `Extra`, đặc biệt tình huống có/không có `Using filesort`.

### 6.4. So sánh

```sql
CREATE INDEX idx_order_status_date_asc
ON order_search (status, order_date);
```

và:

```sql
CREATE INDEX idx_order_status_date_desc
ON order_search (status ASC, order_date DESC);
```

Thử:

```sql
EXPLAIN
SELECT order_number, order_date
FROM order_search
WHERE status = 'Shipped'
ORDER BY order_date DESC;
```

Data nhỏ có thể khiến difference không rõ. Mục tiêu là đọc plan và hiểu index order.

### Bài tập thực hành

**Bài 6.1.** Tạo `idx_order_status_date_desc`.

**Bài 6.2.** Chạy `EXPLAIN` cho query `WHERE status = 'Shipped' ORDER BY order_date DESC`.

**Bài 6.3.** Tạo index `(status, order_date)` và so sánh metadata với index DESC.

**Bài 6.4.** Nêu một query có mixed order mà descending composite index có thể hỗ trợ.

**Bài 6.5.** Giải thích vì sao không nên tạo index DESC cho mọi cột chỉ vì ứng dụng có `ORDER BY ... DESC`.

---

## 7. Composite indexes và leftmost-prefix principle

### 7.1. Composite index

```sql
CREATE INDEX idx_customer_country_credit
ON customer_search (country, credit_limit);
```

Index có thứ tự:

```text
country → credit_limit
```

Nó có thể phù hợp với:

```sql
WHERE country = 'USA'
```

và:

```sql
WHERE country = 'USA'
  AND credit_limit >= 50000.00
```

Nhưng thường không phải lựa chọn trực tiếp tốt cho:

```sql
WHERE credit_limit >= 50000.00
```

vì query bỏ qua leftmost key part `country`.

### 7.2. Thứ tự key parts

Khi chọn `(a, b, c)`, phải dựa vào query patterns, không chỉ dựa vào “cardinality cao nhất trước”.

Các câu hỏi cần trả lời:

1. Cột nào xuất hiện trong equality conditions?
2. Cột nào dùng range?
3. Cột nào dùng để sort/group sau khi filter?
4. Query nào quan trọng nhất?
5. Có index đã tồn tại hỗ trợ leftmost prefix không?
6. Write workload có chấp nhận thêm index không?

### 7.3. Ví dụ order

Query:

```sql
SELECT order_number, order_date, status
FROM order_search
WHERE customer_id = 103
  AND order_date BETWEEN '2004-01-01' AND '2004-12-31'
ORDER BY order_date DESC;
```

Index ứng viên:

```sql
CREATE INDEX idx_order_customer_date
ON order_search (customer_id, order_date);
```

`customer_id` equality trước; `order_date` range/sort tiếp theo.

### 7.4. Composite index không thay thế hoàn toàn mọi index đơn

Một index `(a, b)` có thể hỗ trợ queries dùng `a`, nhưng không tự động thay thế index đơn `b` nếu workload thường lọc chỉ `b`.

Cần kiểm tra usage thực tế trước khi xóa index.

### Bài tập thực hành

**Bài 7.1.** Tạo `idx_customer_country_credit`.

**Bài 7.2.** Viết query sử dụng cả `country` và `credit_limit`.

**Bài 7.3.** Viết query chỉ dùng leftmost key part `country`.

**Bài 7.4.** Viết query chỉ dùng `credit_limit`, rồi giải thích hạn chế của index `(country, credit_limit)`.

**Bài 7.5.** Đề xuất composite index cho query theo `customer_id` và `order_date`.

---

## 8. Clustered indexes trong InnoDB

### 8.1. Clustered index là gì?

Mỗi InnoDB table có clustered index chứa row data. Thông thường, primary key là clustered index.

```sql
CREATE TABLE example (
    id BIGINT NOT NULL,
    code VARCHAR(30) NOT NULL,
    PRIMARY KEY (id)
) ENGINE = InnoDB;
```

Trong table này, `PRIMARY KEY(id)` là clustered index.

### 8.2. Nếu không có primary key

InnoDB chọn theo thứ tự:

1. Primary key được khai báo.
2. Unique index đầu tiên với toàn bộ key columns `NOT NULL`.
3. Nếu không có hai loại trên, InnoDB sinh hidden clustered index (`GEN_CLUST_INDEX`) trên synthetic 6-byte row ID.

Do đó, nên thiết kế primary key rõ ràng cho mỗi InnoDB table.

### 8.3. Secondary index liên hệ clustered index

Trong InnoDB, record của secondary index chứa:

- Secondary key columns.
- Primary key columns của row.

Vì vậy primary key dài làm secondary indexes lớn hơn. Primary key ngắn, ổn định thường có lợi cho storage của secondary indexes.

### 8.4. Không nhầm clustered index với “index sorted mọi cách”

InnoDB table chỉ có một clustered index. Row data được tổ chức theo clustered key, không có nghĩa table được tối ưu đồng thời cho mọi `ORDER BY`.

### Bài tập thực hành

**Bài 8.1.** Xác định clustered index của `customer_search`.

**Bài 8.2.** Nêu thứ tự InnoDB chọn clustered index khi table không khai báo primary key.

**Bài 8.3.** Giải thích vì sao long primary key làm secondary indexes lớn hơn.

**Bài 8.4.** Nêu lý do một table InnoDB nên có explicit primary key.

**Bài 8.5.** Giải thích vì sao table chỉ có một clustered index.

---

## 9. Index cardinality

### 9.1. Khái niệm

Cardinality là ước lượng mức độ distinct của giá trị trong index key.

Ví dụ:

- `customer_id` gần như unique → cardinality cao.
- `status` chỉ có vài trạng thái → cardinality thấp.
- `country` có số distinct vừa phải.

Xem cardinality:

```sql
SHOW INDEX FROM order_search;
```

Hoặc:

```sql
SELECT INDEX_NAME,
       SEQ_IN_INDEX,
       COLUMN_NAME,
       CARDINALITY
FROM INFORMATION_SCHEMA.STATISTICS
WHERE TABLE_SCHEMA = 'index_lab'
  AND TABLE_NAME = 'order_search'
ORDER BY INDEX_NAME, SEQ_IN_INDEX;
```

### 9.2. Cardinality là estimate

Cardinality từ metadata là statistic do server duy trì; không coi đó là số distinct chính xác trong mọi thời điểm.

So sánh:

```sql
SELECT COUNT(DISTINCT customer_id) AS distinctCustomers
FROM order_search;
```

với `Cardinality` của index key part `customer_id`.

### 9.3. Cardinality không phải quy tắc duy nhất

High cardinality thường có thể hữu ích cho selective lookup, nhưng index design còn phụ thuộc:

- Query predicates.
- Composite key order.
- Row distribution.
- Range conditions.
- Table size.
- Write workload.
- Covering potential.
- Optimizer estimates.

### Bài tập thực hành

**Bài 9.1.** Dùng `SHOW INDEX` xem cardinality của `order_search`.

**Bài 9.2.** Đếm `COUNT(DISTINCT customer_id)` trong `order_search`.

**Bài 9.3.** So sánh cardinality của primary key và `status`.

**Bài 9.4.** Nêu lý do cardinality metadata có thể khác `COUNT(DISTINCT ...)`.

**Bài 9.5.** Giải thích vì sao high cardinality không tự động bảo đảm index là đúng cho mọi query.

---

## 10. Functional indexes

### 10.1. Khi expression ngăn index thường được dùng

Giả sử có index thường:

```sql
CREATE INDEX idx_order_date
ON order_search (order_date);
```

Query sau áp dụng function lên cột:

```sql
SELECT COUNT(*)
FROM order_search
WHERE YEAR(order_date) = 2004;
```

Index `idx_order_date` không nhất thiết được dùng theo cách tối ưu cho expression `YEAR(order_date)`.

Trước hết, hãy cân nhắc rewrite query theo range:

```sql
SELECT COUNT(*)
FROM order_search
WHERE order_date >= '2004-01-01'
  AND order_date < '2005-01-01';
```

Range predicate thường đơn giản và tận dụng index thường trên `order_date`.

### 10.2. Tạo functional index

Khi query expression thực sự cần thiết và version hỗ trợ, tạo index trên expression:

```sql
CREATE INDEX idx_order_year
ON order_search ((YEAR(order_date)));
```

**Dấu ngoặc kép** quanh expression là bắt buộc:

```sql
((YEAR(order_date)))
```

Không viết:

```sql
(YEAR(order_date))
```

như cách khai báo column index thông thường.

### 10.3. Kiểm tra

```sql
EXPLAIN
SELECT COUNT(*)
FROM order_search
WHERE YEAR(order_date) = 2004;
```

### 10.4. Những giới hạn quan trọng

Functional key parts:

- Không thể chỉ là một column name được bọc thêm parentheses.
- Không được dùng làm foreign key key part.
- Có các restrictions tương tự generated columns.
- Không cho phép subquery, parameters, variables, stored functions hoặc loadable functions trong expression.
- Không dùng prefix length trực tiếp trên functional key part.
- Không dùng cho primary key; unique functional index được hỗ trợ trong trường hợp phù hợp.

Functional index được hiện thực qua hidden virtual generated columns, vì vậy nó vẫn chiếm storage cho index và cần được thiết kế có mục đích.

### Bài tập thực hành

**Bài 10.1.** Tạo normal index `idx_order_date` trên `order_search(order_date)`.

**Bài 10.2.** Dùng `EXPLAIN` cho `WHERE YEAR(order_date) = 2004`.

**Bài 10.3.** Viết query range thay cho `YEAR(order_date) = 2004`.

**Bài 10.4.** Tạo functional index `idx_order_year` nếu server hỗ trợ.

**Bài 10.5.** Giải thích vì sao functional index không phải lựa chọn đầu tiên nếu query có thể rewrite thành range condition đơn giản.

---

## 11. Chọn loại index: bảng quyết định

| Nhu cầu | Index/giải pháp cần xem xét |
|---|---|
| Business key bắt buộc distinct | Unique index |
| Chuỗi dài, search prefix | Prefix index |
| Thử tác động bỏ index | Invisible index |
| `ORDER BY` mixed ASC/DESC | Descending composite index |
| Lọc/join nhiều cột theo pattern cố định | Composite index |
| Lookup theo biểu thức lặp lại | Functional index hoặc query rewrite |
| Table InnoDB chưa có PK | Thiết kế explicit primary key |
| Query plan nghi do statistics cũ | `ANALYZE TABLE`, sau đó `EXPLAIN` |
| Leading wildcard text search | Không dùng prefix B-tree như giải pháp chính; xem Full-Text Search |

### Bài tập thực hành

**Bài 11.1.** Chọn index phù hợp để đảm bảo `email` unique.

**Bài 11.2.** Chọn index phù hợp cho `WHERE customer_id = ? ORDER BY order_date DESC`.

**Bài 11.3.** Chọn phương án phù hợp khi cần thử drop index nhưng muốn rollback nhanh.

**Bài 11.4.** Chọn phương án đầu tiên cho `WHERE YEAR(order_date) = 2004`: query rewrite hay functional index? Giải thích.

**Bài 11.5.** Nêu trường hợp prefix index phù hợp và không phù hợp.

---

## 12. Bài tập tổng hợp

### Bài 12.1. Unique + prefix

1. Tạo `uq_customer_email`.
2. Tạo `idx_customer_email_prefix`.
3. Dùng `SHOW INDEX` so sánh `Non_unique` và `Sub_part`.
4. Thử duplicate full email.
5. Giải thích vì sao hai index này có mục tiêu khác nhau.

### Bài 12.2. Composite + descending

1. Tạo index `(status, order_date DESC)`.
2. Chạy query lọc `status` và sort `order_date DESC`.
3. Dùng `EXPLAIN`.
4. Tạo index `(status, order_date ASC)` để so sánh metadata.
5. Xóa index không cần cho lab.

### Bài 12.3. Cardinality

1. Xem `SHOW INDEX FROM order_search`.
2. So sánh cardinality `customer_id`, `status`, `order_number`.
3. Chạy `COUNT(DISTINCT ...)` tương ứng.
4. Chạy `ANALYZE TABLE order_search`.
5. Nêu nhận xét về estimate và selectivity.

### Bài 12.4. Functional index

1. Tạo normal index `order_date`.
2. So sánh `YEAR(order_date)=2004` với date range.
3. Dùng `EXPLAIN`.
4. Tạo functional index nếu server hỗ trợ.
5. Nêu trường hợp nên tránh functional index.

### Bài 12.5. InnoDB design review

1. Xác định primary key của ba table lab.
2. Nêu clustered index của từng table.
3. Liệt kê secondary indexes.
4. Nêu ảnh hưởng của primary key dài tới secondary index.
5. Viết một khuyến nghị primary-key design cho bảng transaction lớn.

---

## 13. Đáp án gợi ý

### Bài 4.1

```sql
CREATE INDEX idx_customer_email_prefix
ON customer_search (email(12));
```

### Bài 5.2

```sql
ALTER TABLE customer_search
ALTER INDEX idx_customer_country INVISIBLE;
```

### Bài 6.1

```sql
CREATE INDEX idx_order_status_date_desc
ON order_search (status ASC, order_date DESC);
```

### Bài 7.5

```sql
CREATE INDEX idx_order_customer_date
ON order_search (customer_id, order_date);
```

### Bài 9.2

```sql
SELECT COUNT(DISTINCT customer_id) AS distinctCustomers
FROM order_search;
```

### Bài 10.3

```sql
SELECT COUNT(*)
FROM order_search
WHERE order_date >= '2004-01-01'
  AND order_date < '2005-01-01';
```

### Bài 10.4

```sql
CREATE INDEX idx_order_year
ON order_search ((YEAR(order_date)));
```

### Cleanup

```sql
DROP INDEX IF EXISTS uq_customer_email
ON customer_search;

DROP INDEX IF EXISTS idx_customer_email_prefix
ON customer_search;

DROP INDEX IF EXISTS idx_customer_country_credit
ON customer_search;

DROP INDEX IF EXISTS idx_order_status_date_desc
ON order_search;

DROP INDEX IF EXISTS idx_order_status_date_asc
ON order_search;

DROP INDEX IF EXISTS idx_order_date
ON order_search;

DROP INDEX IF EXISTS idx_order_year
ON order_search;
```

---

## 14. Tóm tắt

- Unique index bảo vệ distinctness; primary key là identity chính của row.
- Prefix index giảm kích thước index bằng cách index prefix của string, nhưng có thể giảm selectivity.
- Invisible index giúp kiểm thử việc bỏ index mà chưa drop nó; index vẫn được maintain.
- Descending key parts hữu ích đặc biệt cho mixed ASC/DESC order trong InnoDB.
- Composite index phụ thuộc mạnh vào thứ tự key parts và leftmost-prefix principle.
- InnoDB dùng primary key làm clustered index; secondary index records chứa primary-key columns.
- Cardinality là estimate về distinctness, không phải số chính xác tuyệt đối.
- Functional index cần double parentheses và chỉ nên dùng khi expression thật sự cần thiết sau khi cân nhắc query rewrite.
- Index strategy luôn phải dựa vào query patterns, data distribution, `EXPLAIN` và write cost.

---

## 15. Từ khóa chính

- Unique index
- Prefix index
- Invisible index
- Descending index
- Composite index
- Leftmost-prefix principle
- Clustered index
- Secondary index
- InnoDB
- Cardinality
- Selectivity
- Functional index
- Functional key part
- Generated column
- EXPLAIN
- ANALYZE TABLE
- Query rewrite
