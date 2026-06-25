---
title: "Tutorial: Quản lý vòng đời Table trong MySQL"
author: "Tên giảng viên"
duration: "240m"
difficulty: "Beginner–Intermediate"
prerequisites:
  - "Đã biết CREATE DATABASE, SELECT, INSERT, UPDATE và DELETE"
  - "Đã biết PRIMARY KEY, FOREIGN KEY, UNIQUE, CHECK, NOT NULL và DEFAULT cơ bản"
  - "Có quyền CREATE, ALTER, DROP, INDEX và DML trên schema thực hành"
summary: "Thực hành CREATE TABLE, AUTO_INCREMENT, ALTER TABLE, RENAME TABLE, DROP TABLE, temporary tables, TRUNCATE TABLE và generated columns bằng schema table_lifecycle_lab."
---

# Tutorial: Quản lý vòng đời Table trong MySQL

## Link tham khảo

### MySQL Tutorial

- [MySQL CREATE TABLE](https://www.mysqltutorial.org/mysql-basics/mysql-create-table/)
- [MySQL AUTO_INCREMENT](https://www.mysqltutorial.org/mysql-basics/mysql-auto-increment/)
- [MySQL ALTER TABLE](https://www.mysqltutorial.org/mysql-basics/mysql-alter-table/)
- [MySQL Rename Table](https://www.mysqltutorial.org/mysql-basics/mysql-rename-table/)
- [MySQL Drop Column](https://www.mysqltutorial.org/mysql-basics/mysql-drop-column/)
- [MySQL Add Column](https://www.mysqltutorial.org/mysql-basics/mysql-add-column/)
- [MySQL DROP TABLE](https://www.mysqltutorial.org/mysql-basics/mysql-drop-table/)
- [MySQL Temporary Table](https://www.mysqltutorial.org/mysql-basics/mysql-temporary-table/)
- [MySQL TRUNCATE TABLE](https://www.mysqltutorial.org/mysql-basics/mysql-truncate-table/)
- [MySQL Generated Columns](https://www.mysqltutorial.org/mysql-basics/mysql-generated-columns/)

### MySQL Reference Manual

- [CREATE TABLE Statement](https://dev.mysql.com/doc/refman/8.4/en/create-table.html)
- [ALTER TABLE Statement](https://dev.mysql.com/doc/refman/8.4/en/alter-table.html)
- [RENAME TABLE Statement](https://dev.mysql.com/doc/refman/8.4/en/rename-table.html)
- [DROP TABLE Statement](https://dev.mysql.com/doc/refman/8.4/en/drop-table.html)
- [CREATE TEMPORARY TABLE Statement](https://dev.mysql.com/doc/refman/8.4/en/create-temporary-table.html)
- [TRUNCATE TABLE Statement](https://dev.mysql.com/doc/refman/8.4/en/truncate-table.html)
- [Generated Columns](https://dev.mysql.com/doc/refman/8.4/en/create-table-generated-columns.html)

> **Phạm vi lab:** Các lệnh trong bài thay đổi cấu trúc hoặc xóa dữ liệu. Dùng schema `table_lifecycle_lab`; không chạy nguyên xi trên `classicmodels`, production database hoặc server dùng chung.

---

## 1. Mục tiêu học tập

Sau khi hoàn thành tutorial này, người học có thể:

1. Tạo table bằng `CREATE TABLE` với data type, constraints, indexes và engine phù hợp.
2. Dùng `AUTO_INCREMENT` để tạo numeric surrogate key.
3. Kiểm tra definition table bằng `SHOW CREATE TABLE`, `SHOW COLUMNS`, `DESCRIBE` và `SHOW INDEX`.
4. Thêm, xóa, sửa và đổi tên column bằng `ALTER TABLE`.
5. Thêm/xóa indexes, unique constraints và foreign keys phù hợp.
6. Đổi tên table bằng `RENAME TABLE` hoặc `ALTER TABLE ... RENAME`.
7. Dùng `DROP TABLE` an toàn theo thứ tự dependency.
8. Tạo temporary table theo session.
9. Phân biệt rõ `DELETE FROM` với `TRUNCATE TABLE`.
10. Tạo virtual và stored generated columns.
11. Nhận biết implicit commit, DDL dependency và migration risk.
12. Viết script schema có thể chạy lại từ database rỗng.

---

## 2. Vòng đời của table và rủi ro DDL

Một table thường đi qua chuỗi:

```text
Design
  -> CREATE TABLE
  -> INSERT / UPDATE / DELETE data
  -> ALTER TABLE khi yêu cầu thay đổi
  -> Add / drop indexes
  -> RENAME TABLE trong migration có kiểm soát
  -> CREATE TEMPORARY TABLE cho intermediate result
  -> DELETE hoặc TRUNCATE dữ liệu theo mục tiêu
  -> DROP TABLE khi đã hoàn tất retention và dependency review
```

| Statement | Mục đích | Rủi ro chính |
|---|---|---|
| `CREATE TABLE` | Tạo object mới | Sai data type/constraint ngay từ đầu |
| `ALTER TABLE` | Thay đổi structure | Lock, implicit commit, mất compatibility |
| `RENAME TABLE` | Đổi tên/move table | Phá application/view/routine |
| `DROP TABLE` | Xóa definition và toàn bộ data | Mất dữ liệu, mất trigger |
| `CREATE TEMPORARY TABLE` | Bảng tạm riêng session | Nhầm với permanent table cùng tên |
| `TRUNCATE TABLE` | Làm rỗng nhanh table | Không rollback như DML, không chạy delete trigger |
| Generated column | Derived value theo expression | Expression/usage không phù hợp |

### 2.1. DDL không phải DML thông thường

`INSERT`, `UPDATE`, `DELETE` chủ yếu tác động row data.

`CREATE TABLE`, `ALTER TABLE`, `DROP TABLE`, `RENAME TABLE` và `TRUNCATE TABLE` là DDL hoặc có semantics DDL. Không mặc định rằng các câu này có thể rollback như `UPDATE` trong một transaction.

Quy trình an toàn:

1. Xác nhận đúng database/table/environment.
2. Chạy `SHOW CREATE TABLE` và sao lưu DDL hiện có.
3. Kiểm tra foreign keys, views, routines, triggers, events và application code.
4. Test trên local/staging.
5. Có migration script và rollback/recreate plan.
6. Kiểm tra lại metadata và data sau thay đổi.

### Bài tập thực hành

**Bài 2.1.** Phân biệt DDL với DML.

**Bài 2.2.** Liệt kê ba lệnh trong bài có thể làm mất dữ liệu nếu dùng sai.

**Bài 2.3.** Nêu ba bước an toàn trước `ALTER TABLE` trên production.

**Bài 2.4.** Giải thích vì sao không nên giả định DDL luôn rollback được.

**Bài 2.5.** Nêu lợi ích của schema lab riêng.

---

## 3. Chuẩn bị schema `table_lifecycle_lab`

> **Cảnh báo:** Script này xóa schema lab nếu nó đã tồn tại.

```sql
DROP DATABASE IF EXISTS table_lifecycle_lab;

CREATE DATABASE table_lifecycle_lab
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_0900_ai_ci;

USE table_lifecycle_lab;
```

### 3.1. Bảng gốc

```sql
CREATE TABLE products (
    product_id INT UNSIGNED NOT NULL AUTO_INCREMENT,
    product_name VARCHAR(120) NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (product_id),
    CONSTRAINT ck_products_unit_price
        CHECK (unit_price >= 0)
) ENGINE = InnoDB;
```

```sql
CREATE TABLE order_lines (
    line_id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    product_id INT UNSIGNED NOT NULL,
    quantity INT UNSIGNED NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    PRIMARY KEY (line_id),
    CONSTRAINT ck_order_lines_quantity
        CHECK (quantity > 0),
    CONSTRAINT ck_order_lines_unit_price
        CHECK (unit_price >= 0),
    CONSTRAINT fk_order_lines_product
        FOREIGN KEY (product_id)
        REFERENCES products(product_id)
        ON UPDATE RESTRICT
        ON DELETE RESTRICT
) ENGINE = InnoDB;
```

`import_staging` không có FK, dùng riêng cho phần `TRUNCATE TABLE`:

```sql
CREATE TABLE import_staging (
    staging_id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    source_file VARCHAR(100) NOT NULL,
    raw_product_name VARCHAR(200) NOT NULL,
    raw_price_text VARCHAR(50) NULL,
    imported_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (staging_id)
) ENGINE = InnoDB;
```

### 3.2. Dữ liệu mẫu

```sql
INSERT INTO products (product_name, unit_price)
VALUES
    ('Keyboard', 450000.00),
    ('Mouse', 250000.00),
    ('Monitor', 3200000.00);
```

```sql
INSERT INTO order_lines (product_id, quantity, unit_price)
VALUES
    (1, 2, 450000.00),
    (2, 1, 250000.00),
    (3, 1, 3200000.00);
```

```sql
INSERT INTO import_staging (
    source_file,
    raw_product_name,
    raw_price_text
)
VALUES
    ('products_2026_01.csv', 'Keyboard', '450000'),
    ('products_2026_01.csv', 'Mouse', '250000');
```

### 3.3. Kiểm tra metadata

```sql
SHOW TABLES;

SHOW CREATE TABLE products;

SHOW COLUMNS FROM order_lines;

SHOW INDEX FROM products;
```

### Bài tập thực hành

**Bài 3.1.** Tạo `table_lifecycle_lab`.

**Bài 3.2.** Tạo `products`, `order_lines`, `import_staging`.

**Bài 3.3.** Insert ít nhất ba products và ba order lines.

**Bài 3.4.** Dùng `SHOW CREATE TABLE products;`.

**Bài 3.5.** Dùng `DESCRIBE order_lines;` và giải thích output.

---

# Phần A. CREATE TABLE và AUTO_INCREMENT

## 4. CREATE TABLE

### 4.1. Cú pháp khái quát

```sql
CREATE TABLE [IF NOT EXISTS] table_name (
    column_name data_type [column_options],
    ...,
    [table_constraints]
) [table_options];
```

Ví dụ:

```sql
CREATE TABLE categories (
    category_id INT UNSIGNED NOT NULL AUTO_INCREMENT,
    category_name VARCHAR(100) NOT NULL,
    description VARCHAR(255) NULL,
    PRIMARY KEY (category_id),
    CONSTRAINT uq_categories_name
        UNIQUE (category_name)
) ENGINE = InnoDB;
```

### 4.2. Các quyết định khi tạo table

| Thành phần | Câu hỏi thiết kế |
|---|---|
| Table name | Tên có phản ánh entity rõ ràng không? |
| Data type | Range, precision, format có phù hợp dữ liệu không? |
| NULL/NOT NULL | Giá trị có thật sự có thể thiếu không? |
| Primary key | Row được định danh thế nào? |
| UNIQUE | Business identifier nào không được trùng? |
| FK | Table phụ thuộc table nào? |
| CHECK/DEFAULT | Domain/default nào enforce tại DB? |
| Engine | Có cần transaction và foreign key? |
| Index | Query workload nào thực sự cần hỗ trợ? |

### 4.3. CREATE TABLE IF NOT EXISTS

```sql
CREATE TABLE IF NOT EXISTS categories (
    category_id INT UNSIGNED NOT NULL AUTO_INCREMENT,
    category_name VARCHAR(100) NOT NULL,
    PRIMARY KEY (category_id)
) ENGINE = InnoDB;
```

`IF NOT EXISTS` chỉ giúp tránh lỗi khi table đã tồn tại. Nó **không** so sánh definition hiện tại với definition bạn viết và không tự thêm cột/constraint còn thiếu.

Kiểm tra table thật sự bằng:

```sql
SHOW CREATE TABLE categories;
```

### 4.4. CREATE TABLE ... LIKE và CREATE TABLE ... AS SELECT

Copy skeleton:

```sql
CREATE TABLE products_backup LIKE products;
```

Tạo table từ query result:

```sql
CREATE TABLE products_copy AS
SELECT *
FROM products;
```

Không coi hai cách là tương đương:

| Cách | Mục tiêu |
|---|---|
| `CREATE TABLE ... LIKE` | Tạo table dựa trên definition source |
| `CREATE TABLE ... AS SELECT` | Tạo table từ result set; không mặc định giữ mọi key/constraint/index của source |

### Bài tập thực hành

**Bài 4.1.** Tạo table `categories`.

**Bài 4.2.** Thêm unique constraint cho `category_name`.

**Bài 4.3.** Dùng `SHOW CREATE TABLE categories;`.

**Bài 4.4.** Tạo `products_backup LIKE products`.

**Bài 4.5.** Tạo `products_copy AS SELECT * FROM products` rồi so sánh metadata của hai table copy.

---

## 5. AUTO_INCREMENT

### 5.1. Khái niệm

`AUTO_INCREMENT` là attribute của integer column, để MySQL tự sinh sequence number khi insert row.

```sql
CREATE TABLE suppliers (
    supplier_id INT UNSIGNED NOT NULL AUTO_INCREMENT,
    supplier_name VARCHAR(120) NOT NULL,
    PRIMARY KEY (supplier_id)
) ENGINE = InnoDB;
```

Insert không cần đưa ID:

```sql
INSERT INTO suppliers (supplier_name)
VALUES
    ('Alpha Trading'),
    ('Beta Imports');
```

```sql
SELECT *
FROM suppliers
ORDER BY supplier_id;
```

### 5.2. Cách dùng tốt nhất

Cách khuyến nghị: bỏ column AUTO_INCREMENT khỏi `INSERT`.

```sql
INSERT INTO suppliers (supplier_name)
VALUES ('Gamma Distribution');
```

Hoặc:

```sql
INSERT INTO suppliers (
    supplier_id,
    supplier_name
)
VALUES (
    NULL,
    'Delta Office'
);
```

### 5.3. LAST_INSERT_ID()

Lấy generated value sau INSERT trong cùng session:

```sql
INSERT INTO suppliers (supplier_name)
VALUES ('Epsilon Electronics');

SELECT LAST_INSERT_ID() AS newSupplierId;
```

### 5.4. Rule quan trọng

Một table:

- Chỉ có một `AUTO_INCREMENT` column.
- Column đó phải được index.
- Không có `DEFAULT` value.
- Thường được dùng làm `PRIMARY KEY`.

Ví dụ:

```sql
product_id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
PRIMARY KEY (product_id)
```

### 5.5. AUTO_INCREMENT không phải business number

Không dùng auto-generated ID thay cho số nghiệp vụ phải theo format/sequence riêng, ví dụ:

```text
INV-2026-000001
ORD-HN-2026-001
STU20260001
```

`AUTO_INCREMENT` phù hợp cho surrogate key. Business code nên là column riêng, có format/UNIQUE rule phù hợp.

### 5.6. Không giả định ID liên tục tuyệt đối

Không dùng ID để suy luận:

```text
Số row hiện tại.
Số hóa đơn hợp lệ.
Không có gap trong lịch sử.
```

Delete, failed insert, concurrent inserts hoặc behavior engine/configuration có thể tạo gap. ID là identifier kỹ thuật, không phải số chứng từ nghiệp vụ liên tục.

### 5.7. Đặt lại giá trị bắt đầu trong lab

```sql
ALTER TABLE suppliers
AUTO_INCREMENT = 100;
```

Không dùng thao tác này để “lấp gap” production.

### Bài tập thực hành

**Bài 5.1.** Tạo table `suppliers`.

**Bài 5.2.** Insert ba suppliers không chỉ định ID.

**Bài 5.3.** Insert thêm supplier và gọi `LAST_INSERT_ID()`.

**Bài 5.4.** Thử tạo table có hai AUTO_INCREMENT columns.

**Bài 5.5.** Giải thích vì sao invoice number không nên được đồng nhất với auto-increment ID.

---

# Phần B. ALTER TABLE

## 6. ALTER TABLE

### 6.1. Cú pháp

```sql
ALTER TABLE table_name
    alter_option [, alter_option] ...;
```

`ALTER TABLE` có thể:

- Add/drop/modify/rename columns.
- Add/drop indexes.
- Add/drop foreign keys.
- Add/drop/check constraints.
- Rename table.
- Đổi charset/collation hoặc table options.

### 6.2. Checklist trước ALTER production table

1. Table có bao nhiêu rows và kích thước bao nhiêu?
2. Có application query/write đồng thời không?
3. Có FK, view, trigger, routine, event hay ETL dependency không?
4. Có risk mất/truncate data không?
5. Server version/engine hỗ trợ algorithm/locking gì?
6. Có backup definition/data không?
7. Có rollback plan không?
8. Đã test local/staging chưa?

### 6.3. Kiểm tra trước/sau thay đổi

```sql
SHOW CREATE TABLE products;
```

```sql
SHOW COLUMNS FROM products;
```

```sql
SHOW INDEX FROM products;
```

### Bài tập thực hành

**Bài 6.1.** Chạy `SHOW CREATE TABLE products;` trước khi alter.

**Bài 6.2.** Nêu ba dependency cần kiểm tra trước ALTER.

**Bài 6.3.** Nêu một risk của ALTER trên table lớn.

**Bài 6.4.** Viết skeleton ALTER thêm hai cột.

**Bài 6.5.** Nêu hai lý do cần test DDL trên staging.

---

## 7. ADD COLUMN

### 7.1. Thêm một cột

```sql
ALTER TABLE products
ADD COLUMN sku VARCHAR(30) NULL
AFTER product_id;
```

Kiểm tra:

```sql
SHOW COLUMNS FROM products;
```

### 7.2. Thêm cột có default

```sql
ALTER TABLE products
ADD COLUMN is_active BOOLEAN NOT NULL DEFAULT TRUE
AFTER unit_price;
```

Cần xác định:

- Existing rows nhận giá trị gì?
- New rows có thể bỏ cột không?
- Default có đúng nghiệp vụ không?

### 7.3. Thêm nhiều cột

```sql
ALTER TABLE products
ADD COLUMN description VARCHAR(255) NULL AFTER product_name,
ADD COLUMN updated_at DATETIME NOT NULL
    DEFAULT CURRENT_TIMESTAMP
    ON UPDATE CURRENT_TIMESTAMP
    AFTER created_at;
```

### 7.4. Thêm NOT NULL column vào table có data

Thực hiện theo nhiều bước:

```sql
ALTER TABLE products
ADD COLUMN reorder_level INT NULL;
```

```sql
UPDATE products
SET reorder_level = 0
WHERE reorder_level IS NULL;
```

```sql
ALTER TABLE products
MODIFY COLUMN reorder_level INT NOT NULL DEFAULT 0;
```

### 7.5. FIRST và AFTER

```sql
ALTER TABLE products
ADD COLUMN brand_name VARCHAR(100) NULL
AFTER product_name;
```

Thứ tự cột không nên là API contract. Luôn ghi explicit column list:

```sql
SELECT product_id, sku, product_name, unit_price
FROM products;
```

Không dựa vào `SELECT *` để mapping application theo vị trí.

### Bài tập thực hành

**Bài 7.1.** Thêm `sku VARCHAR(30)` vào products.

**Bài 7.2.** Thêm `is_active BOOLEAN NOT NULL DEFAULT TRUE`.

**Bài 7.3.** Thêm `description VARCHAR(255)`.

**Bài 7.4.** Thêm `reorder_level` qua nullable -> update -> NOT NULL.

**Bài 7.5.** Giải thích vì sao application không nên phụ thuộc vào thứ tự cột.

---

## 8. DROP COLUMN

### 8.1. Cú pháp

```sql
ALTER TABLE table_name
DROP [COLUMN] column_name;
```

Ví dụ:

```sql
ALTER TABLE products
DROP COLUMN description;
```

### 8.2. Drop nhiều cột

```sql
ALTER TABLE products
DROP COLUMN description,
DROP COLUMN brand_name;
```

### 8.3. Kiểm tra trước khi drop

- Cột có data cần archive không?
- View/routine/trigger/event có dùng cột không?
- Application/API/ORM có query cột không?
- Cột có nằm trong index/FK/constraint/generated expression không?
- Có release plan để application ngừng dùng cột trước không?
- Có migration rollback plan không?

### 8.4. Không dùng drop để “ẩn” dữ liệu

Nếu cần hạn chế hiển thị, cân nhắc view, role/privilege, policy application hoặc deprecation. Drop column làm mất data trong cột đó.

### Bài tập thực hành

**Bài 8.1.** Add `obsolete_note VARCHAR(100)`.

**Bài 8.2.** Update vài rows có `obsolete_note`.

**Bài 8.3.** Dùng `SHOW COLUMNS` xác minh.

**Bài 8.4.** Drop `obsolete_note`.

**Bài 8.5.** Liệt kê năm dependency/risk trước khi drop column production.

---

## 9. MODIFY, CHANGE và RENAME COLUMN

### 9.1. MODIFY COLUMN

Giữ tên, đổi definition:

```sql
ALTER TABLE products
MODIFY COLUMN sku VARCHAR(50) NULL;
```

Khi `MODIFY`, ghi đầy đủ attributes cần giữ.

Ví dụ nếu old column là:

```sql
is_active BOOLEAN NOT NULL DEFAULT TRUE
```

thì không nên viết:

```sql
ALTER TABLE products
MODIFY COLUMN is_active BOOLEAN;
```

vì có thể mất `NOT NULL` hoặc `DEFAULT`.

Viết đầy đủ:

```sql
ALTER TABLE products
MODIFY COLUMN is_active BOOLEAN NOT NULL DEFAULT TRUE;
```

### 9.2. CHANGE COLUMN

Đổi tên và/hoặc definition:

```sql
ALTER TABLE products
CHANGE COLUMN product_name display_name VARCHAR(150) NOT NULL;
```

### 9.3. RENAME COLUMN

Nếu chỉ đổi tên:

```sql
ALTER TABLE products
RENAME COLUMN product_name TO display_name;
```

Đổi tên column vẫn có thể phá application SQL, views, routines hoặc ETL. Hãy kiểm tra dependencies.

### 9.4. Thay đổi data type

Ví dụ mở rộng giá:

```sql
ALTER TABLE products
MODIFY COLUMN unit_price DECIMAL(12,2) NOT NULL;
```

Trước khi thu hẹp length:

```sql
SELECT MAX(CHAR_LENGTH(product_name)) AS maxNameLength
FROM products;
```

### Bài tập thực hành

**Bài 9.1.** Đổi SKU từ `VARCHAR(30)` sang `VARCHAR(50)`.

**Bài 9.2.** Rename `product_name` sang `display_name`.

**Bài 9.3.** Đổi lại `display_name` thành `product_name`.

**Bài 9.4.** Mở rộng `unit_price` thành `DECIMAL(12,2)`.

**Bài 9.5.** Giải thích vì sao MODIFY cần ghi đủ NOT NULL/DEFAULT.

---

## 10. Add/drop index, unique constraint và foreign key

### 10.1. Add ordinary index

```sql
ALTER TABLE products
ADD INDEX idx_products_sku (sku);
```

```sql
SHOW INDEX FROM products;
```

### 10.2. Add UNIQUE

Sau khi data đã sạch:

```sql
ALTER TABLE products
ADD CONSTRAINT uq_products_sku
UNIQUE (sku);
```

Kiểm tra duplicates trước:

```sql
SELECT sku, COUNT(*) AS total_rows
FROM products
WHERE sku IS NOT NULL
GROUP BY sku
HAVING COUNT(*) > 1;
```

### 10.3. Drop index

```sql
ALTER TABLE products
DROP INDEX idx_products_sku;
```

Dùng tên index hiển thị bởi `SHOW INDEX`/`SHOW CREATE TABLE`.

### 10.4. Add FK sau khi tạo table

```sql
CREATE TABLE product_notes (
    note_id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    product_id INT UNSIGNED NOT NULL,
    note_text VARCHAR(255) NOT NULL,
    PRIMARY KEY (note_id)
) ENGINE = InnoDB;
```

```sql
ALTER TABLE product_notes
ADD CONSTRAINT fk_product_notes_product
FOREIGN KEY (product_id)
REFERENCES products(product_id)
ON UPDATE RESTRICT
ON DELETE CASCADE;
```

Trước khi add FK, kiểm tra existing child rows, type compatibility và referential action.

### 10.5. Drop FK

```sql
ALTER TABLE product_notes
DROP FOREIGN KEY fk_product_notes_product;
```

Supporting index có thể vẫn tồn tại; kiểm tra trước khi drop thêm index.

### Bài tập thực hành

**Bài 10.1.** Add `idx_products_sku`.

**Bài 10.2.** Set distinct SKU values rồi add `UNIQUE`.

**Bài 10.3.** Dùng SHOW INDEX.

**Bài 10.4.** Tạo `product_notes` và add FK bằng ALTER.

**Bài 10.5.** Giải thích vì sao cần kiểm tra orphan rows trước add FK.

---

# Phần C. RENAME TABLE và DROP TABLE

## 11. RENAME TABLE

### 11.1. Cú pháp

```sql
RENAME TABLE old_table_name
TO new_table_name;
```

```sql
RENAME TABLE import_staging
TO product_import_staging;
```

Hoặc:

```sql
ALTER TABLE product_import_staging
RENAME TO import_staging;
```

### 11.2. Đổi nhiều table

```sql
RENAME TABLE
    table_a TO table_a_old,
    table_b TO table_a,
    table_a_old TO table_b;
```

Chỉ dùng cho migration có plan rõ. MySQL thực hiện rename theo thứ tự và có metadata locking.

### 11.3. Dependencies cần kiểm tra

- Application SQL/ORM mapping.
- Views.
- Procedures/functions.
- Triggers/events.
- ETL/report scripts.
- Documentation and tests.
- Permissions granted per table.

### 11.4. Không dùng suffix table như version control

Tránh:

```text
products_final
products_final_v2
products_final_v3_new
```

Ưu tiên migration scripts có version, Git và deployment process.

### Bài tập thực hành

**Bài 11.1.** Rename `import_staging` thành `product_import_staging`.

**Bài 11.2.** Rename lại bằng ALTER TABLE.

**Bài 11.3.** Kiểm tra SHOW TABLES trước/sau.

**Bài 11.4.** Nêu bốn dependency trước rename production table.

**Bài 11.5.** Giải thích vì sao suffix table không thay migration strategy.

---

## 12. DROP TABLE

### 12.1. Cú pháp

```sql
DROP TABLE table_name;
```

Cleanup-safe:

```sql
DROP TABLE IF EXISTS product_notes;
```

### 12.2. DROP TABLE xóa gì?

- Table definition.
- Toàn bộ data.
- Indexes.
- Partition data/definition nếu table partitioned.
- Triggers gắn với table.

Không dùng DROP TABLE khi chỉ muốn:

- Xóa rows: `DELETE`/`TRUNCATE`.
- Xóa column: `ALTER TABLE DROP COLUMN`.
- Xóa index: `DROP INDEX`/`ALTER TABLE DROP INDEX`.

### 12.3. Foreign key dependency

`order_lines` reference `products`, nên cleanup theo thứ tự:

```sql
DROP TABLE IF EXISTS order_lines;
DROP TABLE IF EXISTS products;
```

Không tắt foreign-key checks tùy tiện để drop production objects.

### 12.4. IF EXISTS

```sql
DROP TABLE IF EXISTS product_notes;
```

`IF EXISTS` giúp cleanup script chạy lại. Xem diagnostic bằng:

```sql
SHOW WARNINGS;
```

### Bài tập thực hành

**Bài 12.1.** Tạo `drop_demo` và thêm rows.

**Bài 12.2.** Drop `drop_demo`.

**Bài 12.3.** Xác minh bằng SHOW TABLES LIKE.

**Bài 12.4.** Thử drop products khi order_lines còn tồn tại.

**Bài 12.5.** So sánh DROP TABLE, TRUNCATE và DELETE.

---

# Phần D. Temporary tables

## 13. CREATE TEMPORARY TABLE

### 13.1. Khái niệm

Temporary table:

- Chỉ nhìn thấy trong session tạo nó.
- Tự bị drop khi session đóng.
- Có thể trùng tên permanent table; temporary table che permanent table trong session đó.
- Cần privilege `CREATE TEMPORARY TABLES` để tạo.

### 13.2. Temporary aggregate table

```sql
CREATE TEMPORARY TABLE tmp_product_sales (
    product_id INT UNSIGNED NOT NULL,
    total_quantity BIGINT NOT NULL,
    total_value DECIMAL(14,2) NOT NULL,
    PRIMARY KEY (product_id)
) ENGINE = InnoDB;
```

```sql
INSERT INTO tmp_product_sales (
    product_id,
    total_quantity,
    total_value
)
SELECT product_id,
       SUM(quantity),
       SUM(quantity * unit_price)
FROM order_lines
GROUP BY product_id;
```

```sql
SELECT p.product_name,
       t.total_quantity,
       t.total_value
FROM tmp_product_sales AS t
JOIN products AS p
    ON p.product_id = t.product_id
ORDER BY t.total_value DESC;
```

### 13.3. Temporary table from SELECT

```sql
CREATE TEMPORARY TABLE tmp_expensive_products AS
SELECT product_id,
       product_name,
       unit_price
FROM products
WHERE unit_price >= 1000000.00;
```

### 13.4. Session isolation

Session A tạo `tmp_demo`; Session B không nhìn thấy table này. Session B có thể tạo temporary table riêng cũng tên `tmp_demo`.

Temporary table không phải shared cache/communication table giữa users.

### 13.5. Drop temporary table rõ ràng

```sql
DROP TEMPORARY TABLE IF EXISTS tmp_product_sales;
```

Keyword `TEMPORARY` giúp tránh xóa nhầm permanent table cùng tên.

### Bài tập thực hành

**Bài 13.1.** Tạo tmp_product_sales.

**Bài 13.2.** Insert aggregate data từ order_lines.

**Bài 13.3.** Join temporary table với products.

**Bài 13.4.** Tạo tmp_expensive_products AS SELECT.

**Bài 13.5.** Drop hai temporary tables an toàn.

---

## 14. Temporary table cùng tên permanent table

Trong một session, tránh:

```sql
CREATE TEMPORARY TABLE products (...);
```

vì từ đó:

```sql
SELECT * FROM products;
```

đọc temporary table thay vì permanent table trong session đó.

Dùng convention:

```text
tmp_
stage_
work_
session_
```

Ví dụ:

```text
tmp_product_sales
stage_import_products
work_customer_ranking
```

### Bài tập thực hành

**Bài 14.1.** Giải thích behavior khi temporary table trùng permanent table name.

**Bài 14.2.** Đề xuất ba prefixes cho temporary table.

**Bài 14.3.** Viết lệnh drop `tmp_import` an toàn.

**Bài 14.4.** Giải thích vì sao temporary table không phù hợp shared cache.

**Bài 14.5.** Nêu use case reporting phù hợp với temporary table.

---

# Phần E. TRUNCATE TABLE

## 15. TRUNCATE TABLE

### 15.1. Cú pháp

```sql
TRUNCATE TABLE import_staging;
```

```sql
SELECT COUNT(*) AS remaining_rows
FROM import_staging;
```

### 15.2. DELETE và TRUNCATE

| Tiêu chí | DELETE FROM | TRUNCATE TABLE |
|---|---|---|
| Nhóm statement | DML | DDL |
| Xóa toàn bộ rows | Có | Có |
| WHERE | Có | Không |
| DELETE trigger | Có thể chạy | Không chạy |
| Rollback như DML | Có thể tùy transaction/engine | Không; implicit commit |
| AUTO_INCREMENT | Thường không reset chỉ vì delete | Reset về start value |
| FK reference từ child | Theo FK rules | Có thể fail nếu table bị table khác reference |
| Use case | Xóa có điều kiện/audit/transaction | Reset toàn bộ staging/test table phù hợp |

### 15.3. Không có WHERE

Sai:

```sql
TRUNCATE TABLE import_staging
WHERE source_file = 'products_2026_01.csv';
```

Đúng khi cần selective delete:

```sql
DELETE FROM import_staging
WHERE source_file = 'products_2026_01.csv';
```

### 15.4. Trigger và foreign keys

`TRUNCATE TABLE` không kích hoạt `ON DELETE` trigger và không phù hợp nếu workflow cần archive/audit per row.

Không dùng:

```sql
TRUNCATE TABLE products;
```

khi `order_lines` vẫn reference products.

### 15.5. Khi dùng TRUNCATE

Chỉ dùng khi:

- Xóa toàn bộ rows.
- Table là staging/cache/test.
- Không cần DELETE trigger/audit.
- Không cần DML rollback.
- Đã xác nhận FK dependencies.
- Đúng environment/table.

### Bài tập thực hành

**Bài 15.1.** Thêm 3 rows vào import_staging.

**Bài 15.2.** TRUNCATE import_staging.

**Bài 15.3.** Insert row mới và kiểm tra staging_id.

**Bài 15.4.** Viết DELETE theo source_file.

**Bài 15.5.** Nêu bốn khác biệt DELETE và TRUNCATE.

---

# Phần F. Generated columns

## 16. Generated columns

### 16.1. Khái niệm và cú pháp

Generated column do MySQL tính từ expression:

```sql
column_name data_type
[GENERATED ALWAYS] AS (expression)
[VIRTUAL | STORED]
```

Ví dụ:

```sql
line_total DECIMAL(12,2)
GENERATED ALWAYS AS (quantity * unit_price) STORED
```

### 16.2. VIRTUAL và STORED

| Loại | Tính toán | Storage | Use case |
|---|---|---|---|
| VIRTUAL | Khi row được đọc | Không materialize trong row | Derived value/query convenience |
| STORED | Khi INSERT/UPDATE | Lưu value | Derived value cần materialize/indexing phù hợp |

Nếu không ghi loại, default là `VIRTUAL`.

### 16.3. Stored generated example

```sql
CREATE TABLE order_lines_generated (
    line_id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    product_name VARCHAR(120) NOT NULL,
    quantity INT UNSIGNED NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    line_total DECIMAL(12,2)
        GENERATED ALWAYS AS (quantity * unit_price) STORED,
    PRIMARY KEY (line_id),
    CONSTRAINT ck_olg_quantity CHECK (quantity > 0),
    CONSTRAINT ck_olg_unit_price CHECK (unit_price >= 0)
) ENGINE = InnoDB;
```

```sql
INSERT INTO order_lines_generated (
    product_name,
    quantity,
    unit_price
)
VALUES
    ('Keyboard', 2, 450000.00),
    ('Monitor', 1, 3200000.00);
```

```sql
SELECT line_id,
       product_name,
       quantity,
       unit_price,
       line_total
FROM order_lines_generated;
```

### 16.4. Không gán value tùy ý

Không viết:

```sql
INSERT INTO order_lines_generated (
    product_name,
    quantity,
    unit_price,
    line_total
)
VALUES (
    'Mouse',
    1,
    250000.00,
    1.00
);
```

Bỏ generated column khỏi INSERT/UPDATE để MySQL tự tính.

```sql
INSERT INTO order_lines_generated (
    product_name,
    quantity,
    unit_price
)
VALUES (
    'Mouse',
    1,
    250000.00
);
```

### 16.5. Update base columns

```sql
UPDATE order_lines_generated
SET quantity = 3
WHERE product_name = 'Keyboard';
```

```sql
SELECT product_name,
       quantity,
       unit_price,
       line_total
FROM order_lines_generated
WHERE product_name = 'Keyboard';
```

### 16.6. Virtual generated example

```sql
CREATE TABLE product_text_demo (
    product_id INT UNSIGNED NOT NULL AUTO_INCREMENT,
    product_name VARCHAR(100) NOT NULL,
    brand_name VARCHAR(100) NOT NULL,
    display_label VARCHAR(220)
        GENERATED ALWAYS AS (
            CONCAT(brand_name, ' - ', product_name)
        ) VIRTUAL,
    PRIMARY KEY (product_id)
) ENGINE = InnoDB;
```

```sql
INSERT INTO product_text_demo (
    product_name,
    brand_name
)
VALUES
    ('Wireless Mouse', 'Logitech'),
    ('Mechanical Keyboard', 'Keychron');
```

```sql
SELECT product_id,
       brand_name,
       product_name,
       display_label
FROM product_text_demo;
```

### 16.7. Index generated column

```sql
CREATE INDEX idx_order_lines_generated_total
ON order_lines_generated (line_total);
```

Bắt đầu từ workload:

```sql
SELECT *
FROM order_lines_generated
WHERE line_total >= 1000000.00;
```

Sau đó dùng `EXPLAIN`, data size, selectivity và write cost để đánh giá index.

### 16.8. Restrictions chính

Generated column expression không dùng:

- Subquery.
- User/system/local variables.
- Stored functions/loadable functions.
- AUTO_INCREMENT làm generated column hoặc base column expression.

Generated column phù hợp khi value chỉ phụ thuộc base columns trong cùng row/table. Không dùng cho:

```text
Order total = SUM(line items) across rows
Customer status from other tables
Discount from current promotion table
Inventory across warehouses
```

### Bài tập thực hành

**Bài 16.1.** Tạo order_lines_generated.

**Bài 16.2.** Insert rows không chỉ định line_total.

**Bài 16.3.** Update quantity và kiểm tra line_total.

**Bài 16.4.** Tạo product_text_demo có display_label VIRTUAL.

**Bài 16.5.** Add index line_total và chạy EXPLAIN cho predicate theo total.

---

## 17. Add generated column vào table có sẵn

### 17.1. Add stored column

```sql
ALTER TABLE order_lines
ADD COLUMN line_total DECIMAL(12,2)
    GENERATED ALWAYS AS (quantity * unit_price) STORED
    AFTER unit_price;
```

```sql
SHOW CREATE TABLE order_lines;
```

```sql
SELECT line_id,
       product_id,
       quantity,
       unit_price,
       line_total
FROM order_lines;
```

### 17.2. Add virtual column

```sql
ALTER TABLE products
ADD COLUMN price_label VARCHAR(180)
    GENERATED ALWAYS AS (
        CONCAT(product_name, ' / ', unit_price)
    ) VIRTUAL;
```

Đây là demo. Trong production, presentation formatting phức tạp thường nên nằm ở app/view/report layer thay vì generated column.

### 17.3. Generated column không thay thế business calculation phức tạp

Dùng khi derived value:

- Chỉ dùng columns cùng row.
- Expression đơn giản/deterministic.
- Không lookup table khác.
- Không phụ thuộc user/session/workflow.

### Bài tập thực hành

**Bài 17.1.** Add stored line_total vào order_lines.

**Bài 17.2.** Kiểm tra SHOW CREATE TABLE order_lines.

**Bài 17.3.** Update quantity và kiểm tra line_total.

**Bài 17.4.** Add virtual price_label vào products.

**Bài 17.5.** Nêu ba trường hợp không dùng generated column.

---

# Phần G. Tổng hợp

## 18. Checklist migration table

| Trường | Cần trả lời |
|---|---|
| Change ID | Ví dụ `DDL-ADD-SKU-2026-03` |
| Mục tiêu | Thêm SKU để tích hợp dữ liệu nhập hàng |
| Tables | products |
| DDL | ALTER TABLE ADD COLUMN |
| Data migration | Existing rows có SKU hay NULL? |
| Dependencies | API, queries, views, ETL, constraints, indexes |
| Test | SHOW CREATE TABLE, insert/update/query |
| Rollback | DROP COLUMN nếu chưa có data business cần giữ |
| Risks | Lock, duration, storage, app compatibility |

### Bài tập thực hành

**Bài 18.1.** Viết checklist cho thêm sku.

**Bài 18.2.** Viết checklist cho drop description.

**Bài 18.3.** Viết checklist cho TRUNCATE import_staging.

**Bài 18.4.** Viết checklist cho rename products.

**Bài 18.5.** Nêu lý do migration cần rollback plan.

---

## 19. Bài tập tổng hợp

### Bài 19.1. Product schema evolution

1. Tạo products và sample data.
2. Add sku, description, is_active, reorder_level.
3. Backfill reorder_level.
4. Add unique constraint SKU.
5. Rename product_name thành display_name rồi đổi lại.
6. Kiểm tra final SHOW CREATE TABLE.
7. Viết rollback plan cho hai thay đổi.

### Bài 19.2. Temporary reporting table

1. Tạo temporary table tổng hợp quantity/value theo product.
2. Insert aggregate từ order_lines.
3. Join products.
4. Chỉ hiển thị total value > 1,000,000.
5. Drop temporary table.
6. Mở session khác và kiểm tra session isolation.
7. Giải thích vì sao đây không phải shared table.

### Bài 19.3. DELETE versus TRUNCATE

1. Add rows import_staging.
2. DELETE theo source_file.
3. Add lại rows.
4. TRUNCATE toàn bộ table.
5. Check count.
6. Insert row mới và check AUTO_INCREMENT.
7. Nêu lý do không TRUNCATE products.

### Bài 19.4. Generated order totals

1. Tạo order_lines_generated.
2. Insert không line_total.
3. Update quantity/unit_price.
4. Check generated total.
5. Add index line_total.
6. Query predicate line_total >= 1000000.
7. EXPLAIN và nhận xét data size.

### Bài 19.5. Controlled cleanup

1. Drop temporary tables.
2. Drop child tables trước parent.
3. Drop generated demos.
4. Drop schema.
5. Dùng IF EXISTS.
6. Viết warning lab only.
7. Kiểm tra SHOW DATABASES LIKE.

---

## 20. Đáp án gợi ý

### Bài 19.1

```sql
ALTER TABLE products
ADD COLUMN sku VARCHAR(50) NULL AFTER product_id,
ADD COLUMN description VARCHAR(255) NULL AFTER product_name,
ADD COLUMN is_active BOOLEAN NOT NULL DEFAULT TRUE AFTER unit_price,
ADD COLUMN reorder_level INT NULL AFTER is_active;
```

```sql
UPDATE products
SET reorder_level = 0
WHERE reorder_level IS NULL;
```

```sql
ALTER TABLE products
MODIFY COLUMN reorder_level INT NOT NULL DEFAULT 0;
```

```sql
ALTER TABLE products
ADD CONSTRAINT uq_products_sku UNIQUE (sku);
```

### Bài 19.2

```sql
CREATE TEMPORARY TABLE tmp_product_sales (
    product_id INT UNSIGNED NOT NULL,
    total_quantity BIGINT NOT NULL,
    total_value DECIMAL(14,2) NOT NULL,
    PRIMARY KEY (product_id)
) ENGINE = InnoDB;
```

```sql
INSERT INTO tmp_product_sales (
    product_id,
    total_quantity,
    total_value
)
SELECT product_id,
       SUM(quantity),
       SUM(quantity * unit_price)
FROM order_lines
GROUP BY product_id;
```

```sql
SELECT p.product_name,
       t.total_quantity,
       t.total_value
FROM tmp_product_sales AS t
JOIN products AS p
    ON p.product_id = t.product_id
WHERE t.total_value > 1000000.00
ORDER BY t.total_value DESC;
```

```sql
DROP TEMPORARY TABLE IF EXISTS tmp_product_sales;
```

### Bài 19.3

```sql
DELETE FROM import_staging
WHERE source_file = 'products_2026_01.csv';
```

```sql
TRUNCATE TABLE import_staging;
```

### Bài 19.4

```sql
CREATE INDEX idx_order_lines_generated_total
ON order_lines_generated (line_total);

EXPLAIN
SELECT *
FROM order_lines_generated
WHERE line_total >= 1000000.00;
```

### Bài 19.5

```sql
DROP TEMPORARY TABLE IF EXISTS
    tmp_product_sales,
    tmp_expensive_products;

DROP TABLE IF EXISTS product_notes;
DROP TABLE IF EXISTS order_lines_generated;
DROP TABLE IF EXISTS order_lines;
DROP TABLE IF EXISTS products_backup;
DROP TABLE IF EXISTS products_copy;
DROP TABLE IF EXISTS product_text_demo;
DROP TABLE IF EXISTS import_staging;
DROP TABLE IF EXISTS suppliers;
DROP TABLE IF EXISTS categories;
DROP TABLE IF EXISTS products;

DROP DATABASE IF EXISTS table_lifecycle_lab;
```

---

## 21. Lỗi thường gặp

### Lỗi 1. Dùng CREATE TABLE IF NOT EXISTS như migration tool

Nó không tự thay đổi table đã tồn tại. Dùng `SHOW CREATE TABLE`, rồi viết ALTER/migration rõ ràng.

### Lỗi 2. Quên attributes khi MODIFY/CHANGE

Có thể vô tình mất `NOT NULL`, `DEFAULT`, `COMMENT` hoặc AUTO_INCREMENT.

### Lỗi 3. Drop column/table không review dependency

Có thể phá view, routine, trigger, event, foreign key, report hoặc app.

### Lỗi 4. Dùng TRUNCATE khi cần DELETE trigger/audit

TRUNCATE không kích hoạt ON DELETE trigger.

### Lỗi 5. Temporary table trùng permanent table name

Temporary table che permanent table trong session hiện tại.

### Lỗi 6. Gán value tùy ý cho generated column

Generated value phải do MySQL tính; đừng đưa giá trị business tùy ý vào cột này.

### Lỗi 7. Dùng AUTO_INCREMENT làm invoice number không có gap

Tách surrogate key và business identifier nếu business rule cần numbering riêng.

### Lỗi 8. Add UNIQUE khi data có duplicate

Kiểm tra data trước bằng `GROUP BY ... HAVING COUNT(*) > 1`.

### Bài tập thực hành

**Bài 21.1.** Sửa cách dùng sai CREATE TABLE IF NOT EXISTS để thêm column.

**Bài 21.2.** Nêu command kiểm tra definition trước MODIFY.

**Bài 21.3.** Nêu case phải DELETE thay vì TRUNCATE.

**Bài 21.4.** Nêu case phù hợp temporary table.

**Bài 21.5.** Nêu case generated column phù hợp và không phù hợp.

---

## 22. Tóm tắt

- `CREATE TABLE` tạo columns, keys, constraints, indexes và table options.
- `AUTO_INCREMENT` phù hợp surrogate numeric key; mỗi table chỉ có một auto-increment column, nó phải được index, và không phải business sequence không gap.
- `ALTER TABLE` cho phép add/drop/modify/rename columns, indexes và constraints; luôn review dependencies.
- `RENAME TABLE` có thể ảnh hưởng application và stored objects.
- `DROP TABLE` xóa definition, data và trigger; drop child tables trước parent tables nếu có FK.
- Temporary table chỉ tồn tại trong session tạo nó.
- `TRUNCATE TABLE` làm rỗng table theo DDL semantics: không WHERE, không DELETE trigger, implicit commit, reset auto-increment.
- Generated column là value computed từ expression: VIRTUAL không materialize; STORED materialize.
- Mọi DDL nên có script, backup/review, test và rollback plan.

---

## 23. Từ khóa chính

- CREATE TABLE
- CREATE TABLE IF NOT EXISTS
- SHOW CREATE TABLE
- AUTO_INCREMENT
- LAST_INSERT_ID()
- ALTER TABLE
- ADD COLUMN
- DROP COLUMN
- MODIFY COLUMN
- CHANGE COLUMN
- RENAME COLUMN
- RENAME TABLE
- DROP TABLE
- DROP TEMPORARY TABLE
- Temporary Table
- TRUNCATE TABLE
- DELETE FROM
- Generated Column
- VIRTUAL
- STORED
- CHECK
- UNIQUE
- FOREIGN KEY
- InnoDB
- DDL
- Implicit Commit
- Migration
- Rollback Plan
