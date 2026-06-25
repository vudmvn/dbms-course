---
title: "Tutorial 1: MySQL Triggers — Nền tảng, CREATE/DROP/SHOW và INSERT Triggers"
author: "Tên giảng viên"
duration: "180m"
difficulty: "Intermediate"
prerequisites:
  - "Đã biết INSERT, UPDATE, DELETE, constraints và transaction cơ bản"
  - "Đã biết DELIMITER, BEGIN ... END, IF và SIGNAL trong stored programs"
  - "Có quyền CREATE, ALTER, DROP và TRIGGER trên schema lab"
summary: "Giới thiệu MySQL row-level triggers; thực hành tạo, gọi, kiểm tra và xóa trigger; xây dựng BEFORE INSERT trigger duy trì summary table và AFTER INSERT trigger ghi audit log."
---

# Tutorial 1: MySQL Triggers — Nền tảng, `CREATE`/`DROP`/`SHOW` và `INSERT` Triggers

## Link tham khảo

- [MySQL Tutorial — MySQL Triggers](https://www.mysqltutorial.org/mysql-triggers/)
- [MySQL Tutorial — Create Trigger](https://www.mysqltutorial.org/mysql-triggers/create-trigger/)
- [MySQL Tutorial — BEFORE INSERT Trigger](https://www.mysqltutorial.org/mysql-triggers/mysql-before-insert-trigger/)
- [MySQL Tutorial — AFTER INSERT Trigger](https://www.mysqltutorial.org/mysql-triggers/mysql-after-insert-trigger/)
- [MySQL Tutorial — Drop Trigger](https://www.mysqltutorial.org/mysql-triggers/mysql-drop-trigger/)
- [MySQL Tutorial — Show Triggers](https://www.mysqltutorial.org/mysql-triggers/mysql-show-triggers/)
- [MySQL 8.4 Reference Manual — CREATE TRIGGER](https://dev.mysql.com/doc/refman/8.4/en/create-trigger.html)
- [MySQL 8.4 Reference Manual — Trigger Syntax and Examples](https://dev.mysql.com/doc/refman/8.4/en/trigger-syntax.html)
- [MySQL 8.4 Reference Manual — SHOW TRIGGERS](https://dev.mysql.com/doc/refman/8.4/en/show-triggers.html)
- [MySQL 8.4 Reference Manual — DROP TRIGGER](https://dev.mysql.com/doc/refman/8.4/en/drop-trigger.html)

> **Phạm vi lab:** Trigger chạy tự động khi DML tác động lên table. Vì vậy, lab dùng schema riêng `trigger_lab`; không thêm trigger vào `classicmodels` hoặc production database. Phần thiết lập bên dưới sẽ xóa và tạo lại `trigger_lab`, do đó chỉ chạy trên máy cá nhân hoặc môi trường lab được phép.

---

## 1. Mục tiêu học tập

Sau khi hoàn thành tutorial này, người học có thể:

1. Giải thích trigger là gì và khi nào trigger phù hợp.
2. Phân biệt `BEFORE` với `AFTER`, và `INSERT`/`UPDATE`/`DELETE`.
3. Giải thích MySQL chỉ có row-level trigger, không có statement-level trigger.
4. Tạo trigger bằng `CREATE TRIGGER`.
5. Dùng `NEW.column_name` đúng ngữ cảnh trong INSERT trigger.
6. Dùng `BEFORE INSERT` để chuẩn hóa/kiểm tra dữ liệu và duy trì summary table.
7. Dùng `AFTER INSERT` để ghi audit log sau khi row đã insert thành công.
8. Kiểm tra trigger bằng `SHOW TRIGGERS`, `SHOW CREATE TRIGGER` và `INFORMATION_SCHEMA.TRIGGERS`.
9. Xóa trigger bằng `DROP TRIGGER`.
10. Nhận biết các lỗi thường gặp liên quan delimiter, trigger timing và auto-increment.

---

## 2. Trigger là gì?

Trigger là một stored program có tên, được gắn với một table và tự động chạy khi một event dữ liệu xảy ra trên table đó.

MySQL hỗ trợ các event:

```text
INSERT
UPDATE
DELETE
```

và hai thời điểm:

```text
BEFORE
AFTER
```

Ví dụ:

```text
BEFORE INSERT
AFTER INSERT
BEFORE UPDATE
AFTER UPDATE
BEFORE DELETE
AFTER DELETE
```

### 2.1. Row-level trigger

MySQL chỉ hỗ trợ **row-level trigger**.

Nếu câu lệnh sau insert ba rows:

```sql
INSERT INTO sales (...)
VALUES (...), (...), (...);
```

thì `BEFORE INSERT` trigger và `AFTER INSERT` trigger, nếu tồn tại, được kích hoạt ba lần: một lần cho mỗi row.

MySQL không có statement-level trigger kiểu “chạy một lần cho cả câu INSERT”, bất kể câu lệnh ảnh hưởng bao nhiêu rows.

### 2.2. Các ứng dụng phù hợp

| Nhu cầu | Trigger có thể phù hợp |
|---|---|
| Validation nâng cao theo nghiệp vụ | Kiểm tra giá trị/transition trước DML |
| Chuẩn hóa dữ liệu đầu vào | Trim, uppercase, default business value trong `BEFORE` trigger |
| Audit | Ghi lại giá trị cũ/mới sau update hoặc row vừa insert |
| Summary table nhỏ | Cập nhật tổng hợp gần thời điểm DML |
| Archive khi delete | Lưu row bị xóa vào archive table |

### 2.3. Khi trigger không phải lựa chọn đầu tiên

Với validation đơn giản, ưu tiên constraints:

```text
NOT NULL
CHECK
UNIQUE
FOREIGN KEY
PRIMARY KEY
```

Trigger cũng không nên thay thế hoàn toàn application/service logic phức tạp, vì nó chạy ngầm tại database và có thể khó trace, debug, benchmark hoặc version-control nếu không quản lý cẩn thận.

### 2.4. Ưu điểm và hạn chế

| Ưu điểm | Hạn chế |
|---|---|
| Tự động bảo vệ/duy trì logic ở database layer | Có thể tạo side effect “ẩn” với application |
| Áp dụng cho mọi client ghi vào table | Tăng chi phí mỗi row bị thay đổi |
| Hữu ích cho audit và validation | Khó debug hơn SQL thông thường |
| Giữ một số invariant gần dữ liệu | Không thay thế constraints và thiết kế tốt |

### Bài tập thực hành

**Bài 2.1.** Giải thích trigger là gì bằng một hoặc hai câu.

**Bài 2.2.** Nêu ba event mà MySQL trigger hỗ trợ.

**Bài 2.3.** Nêu khác biệt giữa `BEFORE` và `AFTER`.

**Bài 2.4.** Nếu một lệnh `INSERT` thêm 50 rows, row-level trigger chạy bao nhiêu lần?

**Bài 2.5.** Nêu một trường hợp nên dùng `CHECK` thay vì trigger.

---

## 3. Cú pháp `CREATE TRIGGER`

### 3.1. Cú pháp khái quát

```sql
CREATE TRIGGER trigger_name
trigger_time trigger_event
ON table_name
FOR EACH ROW
trigger_body;
```

Trong đó:

```text
trigger_time  = BEFORE | AFTER
trigger_event = INSERT | UPDATE | DELETE
```

Ví dụ một trigger chỉ có một statement:

```sql
CREATE TRIGGER trg_example
BEFORE INSERT
ON some_table
FOR EACH ROW
SET NEW.some_column = TRIM(NEW.some_column);
```

### 3.2. Trigger body nhiều câu lệnh

Khi body có nhiều statement, dùng:

```sql
DELIMITER $$

CREATE TRIGGER trigger_name
BEFORE INSERT
ON table_name
FOR EACH ROW
BEGIN
    statement_1;
    statement_2;
END$$

DELIMITER ;
```

`DELIMITER` là lệnh của MySQL client/Workbench để phân biệt dấu `;` bên trong body với điểm kết thúc toàn bộ statement `CREATE TRIGGER`.

### 3.3. Quy ước đặt tên

Trong tutorial này dùng:

```text
trg_<timing-abbrev><event-abbrev>_<table>_<purpose>
```

Ví dụ:

```text
trg_bi_sales_prepare_summary
trg_ai_sales_audit
trg_bu_sales_validate_summary
trg_au_sales_audit
trg_bd_sales_protect
trg_ad_sales_archive_summary
```

Viết tắt:

| Prefix | Ý nghĩa |
|---|---|
| `bi` | BEFORE INSERT |
| `ai` | AFTER INSERT |
| `bu` | BEFORE UPDATE |
| `au` | AFTER UPDATE |
| `bd` | BEFORE DELETE |
| `ad` | AFTER DELETE |

Trigger names phải unique trong cùng schema.

### 3.4. Trigger chỉ gắn với permanent table

Không thể gắn trigger với:

```text
TEMPORARY table
View
```

Trigger phải gắn với table thường/permanent table.

### Bài tập thực hành

**Bài 3.1.** Viết cú pháp tổng quát của `CREATE TRIGGER`.

**Bài 3.2.** Viết skeleton của một `AFTER DELETE` trigger có body `BEGIN ... END`.

**Bài 3.3.** Giải thích vì sao cần đổi delimiter khi trigger có nhiều statement.

**Bài 3.4.** Đặt tên trigger phù hợp cho BEFORE UPDATE trigger kiểm tra giá trị `amount` trong table `payments`.

**Bài 3.5.** Nêu hai loại object không thể gắn MySQL trigger.

---

## 4. `OLD` và `NEW`

Trong trigger body, MySQL dùng `OLD` và `NEW` để truy cập values của row bị ảnh hưởng.

| Event | `OLD` | `NEW` |
|---|---:|---:|
| `INSERT` | Không dùng | Dùng |
| `UPDATE` | Dùng | Dùng |
| `DELETE` | Dùng | Không dùng |

### 4.1. INSERT trigger

Trong INSERT trigger, chỉ có row mới:

```sql
NEW.amount
NEW.customer_code
```

Không có:

```sql
OLD.amount
```

vì trước INSERT chưa có row cũ.

### 4.2. UPDATE trigger

Trong UPDATE trigger:

```sql
OLD.amount
```

là giá trị trước update.

```sql
NEW.amount
```

là giá trị sẽ được dùng sau update.

### 4.3. DELETE trigger

Trong DELETE trigger:

```sql
OLD.amount
```

là giá trị của row sắp bị xóa.

Không có `NEW`.

### 4.4. Thay đổi `NEW` trong BEFORE trigger

`OLD` luôn read-only.

Trong `BEFORE INSERT` hoặc `BEFORE UPDATE`, có thể thay đổi `NEW.column_name`:

```sql
SET NEW.customer_code = UPPER(TRIM(NEW.customer_code));
```

Trong `AFTER` trigger, row change đã hoàn tất nên `SET NEW.column_name = ...` không có hiệu lực.

### 4.5. Lưu ý với AUTO_INCREMENT

Trong `BEFORE INSERT`, `NEW.auto_increment_column` có giá trị `0` thay vì sequence number cuối cùng sẽ được tạo tự động.

Vì vậy:

- Dùng `AFTER INSERT` nếu audit cần ID auto-increment đã được sinh.
- Không dựa vào `NEW.sale_id` trong BEFORE INSERT nếu `sale_id` là auto-increment.

### Bài tập thực hành

**Bài 4.1.** Trong BEFORE INSERT trigger, dùng `OLD.amount` có hợp lệ không? Giải thích.

**Bài 4.2.** Trong AFTER DELETE trigger, dùng `NEW.amount` có hợp lệ không? Giải thích.

**Bài 4.3.** Viết statement chuẩn hóa `NEW.customer_code` thành uppercase trong BEFORE INSERT.

**Bài 4.4.** Trong UPDATE trigger, viết expression tính chênh lệch `NEW.amount - OLD.amount`.

**Bài 4.5.** Giải thích vì sao audit cần auto-increment ID thường phù hợp hơn với AFTER INSERT.

---

## 5. Thiết lập schema `trigger_lab`

> **Cảnh báo:** Script này xóa toàn bộ schema `trigger_lab` nếu nó đã tồn tại.

```sql
DROP DATABASE IF EXISTS trigger_lab;

CREATE DATABASE trigger_lab
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_0900_ai_ci;

USE trigger_lab;
```

### 5.1. Table nghiệp vụ chính

```sql
CREATE TABLE sales (
    sale_id BIGINT NOT NULL AUTO_INCREMENT,
    sale_date DATE NULL,
    customer_code VARCHAR(20) NULL,
    amount DECIMAL(12,2) NULL,
    status ENUM('DRAFT', 'POSTED', 'CANCELLED')
        NOT NULL DEFAULT 'DRAFT',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
        ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (sale_id)
) ENGINE = InnoDB;
```

### 5.2. Summary table

```sql
CREATE TABLE daily_sales_summary (
    sale_date DATE NOT NULL,
    sale_count INT NOT NULL DEFAULT 0,
    total_amount DECIMAL(14,2) NOT NULL DEFAULT 0.00,
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
        ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (sale_date)
) ENGINE = InnoDB;
```

### 5.3. Audit table

```sql
CREATE TABLE sales_event_log (
    event_id BIGINT NOT NULL AUTO_INCREMENT,
    sale_id BIGINT NULL,
    event_type VARCHAR(20) NOT NULL,
    event_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    customer_code VARCHAR(20) NULL,
    old_amount DECIMAL(12,2) NULL,
    new_amount DECIMAL(12,2) NULL,
    old_status VARCHAR(20) NULL,
    new_status VARCHAR(20) NULL,
    actor_login VARCHAR(288) NULL,
    note VARCHAR(255) NULL,
    PRIMARY KEY (event_id)
) ENGINE = InnoDB;
```

### 5.4. Kiểm tra DDL

```sql
SHOW CREATE TABLE sales;
SHOW CREATE TABLE daily_sales_summary;
SHOW CREATE TABLE sales_event_log;
```

### Bài tập thực hành

**Bài 5.1.** Tạo schema `trigger_lab`.

**Bài 5.2.** Tạo bảng `sales`.

**Bài 5.3.** Tạo bảng `daily_sales_summary`.

**Bài 5.4.** Tạo bảng `sales_event_log`.

**Bài 5.5.** Dùng `SHOW CREATE TABLE` kiểm tra DDL của `sales`.

---

## 6. BEFORE INSERT trigger: validate, normalize và duy trì summary

### 6.1. Mục tiêu

Trigger `trg_bi_sales_prepare_summary` thực hiện ba tác vụ trước mỗi row được insert vào `sales`:

1. Kiểm tra `amount` phải dương.
2. Gán `sale_date` là ngày hiện tại nếu input là `NULL`.
3. Chuẩn hóa `customer_code`, sau đó tăng summary theo ngày.

### 6.2. Tạo trigger

```sql
DELIMITER $$

CREATE TRIGGER trg_bi_sales_prepare_summary
BEFORE INSERT ON sales
FOR EACH ROW
BEGIN
    IF NEW.amount IS NULL OR NEW.amount <= 0 THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'Sale amount must be greater than 0';
    END IF;

    IF NEW.sale_date IS NULL THEN
        SET NEW.sale_date = CURRENT_DATE();
    END IF;

    IF NEW.customer_code IS NULL
       OR CHAR_LENGTH(TRIM(NEW.customer_code)) = 0 THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'Customer code is required';
    END IF;

    SET NEW.customer_code = UPPER(TRIM(NEW.customer_code));

    INSERT INTO daily_sales_summary (
        sale_date,
        sale_count,
        total_amount
    )
    VALUES (
        NEW.sale_date,
        1,
        NEW.amount
    )
    ON DUPLICATE KEY UPDATE
        sale_count = sale_count + 1,
        total_amount = total_amount + NEW.amount;
END$$

DELIMITER ;
```

### 6.3. Kiểm tra trigger

```sql
SHOW TRIGGERS FROM trigger_lab;
```

```sql
SHOW CREATE TRIGGER trigger_lab.trg_bi_sales_prepare_summary;
```

### 6.4. Test insert hợp lệ

```sql
INSERT INTO sales (
    sale_date,
    customer_code,
    amount,
    status
)
VALUES
    ('2026-01-15', '  c001  ', 125.50, 'DRAFT'),
    ('2026-01-15', 'c002', 80.00, 'POSTED'),
    (NULL, 'c003', 50.00, 'DRAFT');
```

Kiểm tra base table:

```sql
SELECT sale_id,
       sale_date,
       customer_code,
       amount,
       status
FROM sales
ORDER BY sale_id;
```

Kiểm tra summary:

```sql
SELECT sale_date,
       sale_count,
       total_amount
FROM daily_sales_summary
ORDER BY sale_date;
```

Kết quả cần quan sát:

- `c001` được đổi thành `C001`.
- Row có `sale_date = NULL` nhận ngày hiện tại.
- Summary tăng cho từng row inserted.

### 6.5. Test validation

Thử insert amount không hợp lệ:

```sql
INSERT INTO sales (
    sale_date,
    customer_code,
    amount,
    status
)
VALUES (
    '2026-01-15',
    'C004',
    -10.00,
    'DRAFT'
);
```

Statement phải thất bại do `SIGNAL`.

Với InnoDB, khi trigger khiến statement thất bại, các thay đổi do statement đó thực hiện, bao gồm thay đổi summary trong trigger, được rollback cùng statement.

### 6.6. Tại sao dùng BEFORE INSERT?

`BEFORE INSERT` phù hợp khi cần:

- Chặn row không hợp lệ trước khi row được ghi.
- Sửa/normalize `NEW` values trước khi insert.
- Duy trì dữ liệu liên quan trong cùng statement, với hiểu biết rõ về transaction/rollback.

Tuy nhiên, summary logic phức tạp có thể trở nên khó bảo trì. Khi quy tắc tổng hợp nhiều chiều hoặc có concurrency lớn, cần cân nhắc service layer, stored procedure có kiểm soát, materialized reporting pipeline hoặc batch job.

### Bài tập thực hành

**Bài 6.1.** Tạo trigger `trg_bi_sales_prepare_summary`.

**Bài 6.2.** Insert một sale có customer code chứa khoảng trắng và chữ thường; kiểm tra giá trị lưu.

**Bài 6.3.** Insert một sale có `sale_date = NULL`; kiểm tra date được gán.

**Bài 6.4.** Thử insert amount bằng `0`; ghi lại lỗi nhận được.

**Bài 6.5.** Kiểm tra `daily_sales_summary` sau khi insert hai sales cùng ngày.

---

## 7. AFTER INSERT trigger: audit row đã insert

### 7.1. Mục tiêu

`AFTER INSERT` trigger ghi lại event sau khi row `sales` insert thành công.

Điểm quan trọng: trong AFTER INSERT, `NEW.sale_id` đã có auto-increment value được sinh cho row.

### 7.2. Tạo trigger

```sql
DELIMITER $$

CREATE TRIGGER trg_ai_sales_audit
AFTER INSERT ON sales
FOR EACH ROW
BEGIN
    INSERT INTO sales_event_log (
        sale_id,
        event_type,
        customer_code,
        old_amount,
        new_amount,
        old_status,
        new_status,
        actor_login,
        note
    )
    VALUES (
        NEW.sale_id,
        'INSERT',
        NEW.customer_code,
        NULL,
        NEW.amount,
        NULL,
        NEW.status,
        USER(),
        'Sale inserted'
    );
END$$

DELIMITER ;
```

### 7.3. Test AFTER INSERT

```sql
INSERT INTO sales (
    sale_date,
    customer_code,
    amount,
    status
)
VALUES (
    '2026-01-16',
    'c005',
    200.00,
    'DRAFT'
);
```

Kiểm tra audit:

```sql
SELECT event_id,
       sale_id,
       event_type,
       customer_code,
       old_amount,
       new_amount,
       actor_login,
       note,
       event_time
FROM sales_event_log
ORDER BY event_id;
```

### 7.4. `USER()` và `CURRENT_USER()`

Trong trigger:

- `USER()` mô tả account/client connection đã gây ra statement.
- `CURRENT_USER()` trong trigger trả về account dùng để kiểm tra privilege khi trigger chạy, thường là trigger definer.

Khi audit “ai gửi statement”, `USER()` thường hữu ích hơn. Khi audit quyền/definer behavior, cần hiểu `CURRENT_USER()`.

### 7.5. AFTER trigger chỉ chạy sau row operation thành công

Nếu BEFORE trigger hoặc insert row thất bại, AFTER INSERT trigger không chạy.

Đây là một lý do phổ biến để dùng AFTER trigger cho audit “row đã được ghi thành công”.

### Bài tập thực hành

**Bài 7.1.** Tạo trigger `trg_ai_sales_audit`.

**Bài 7.2.** Insert một sale hợp lệ và kiểm tra audit log.

**Bài 7.3.** Xác định `sale_id` trong base table và audit table có khớp không.

**Bài 7.4.** Thử insert amount âm và kiểm tra rằng không có audit row mới được tạo.

**Bài 7.5.** Giải thích vì sao AFTER INSERT phù hợp hơn BEFORE INSERT khi audit cần auto-increment ID.

---

## 8. Liệt kê, xem definition và xóa trigger

### 8.1. `SHOW TRIGGERS`

Liệt kê tất cả trigger trong database:

```sql
SHOW TRIGGERS FROM trigger_lab;
```

Lọc theo table name:

```sql
SHOW TRIGGERS FROM trigger_lab
LIKE 'sales';
```

> `LIKE` trong `SHOW TRIGGERS` khớp **tên table**, không phải tên trigger.

Lọc theo trigger name dùng `WHERE`:

```sql
SHOW TRIGGERS FROM trigger_lab
WHERE `Trigger` LIKE 'trg_ai%';
```

### 8.2. `SHOW CREATE TRIGGER`

```sql
SHOW CREATE TRIGGER trigger_lab.trg_ai_sales_audit;
```

Lệnh này hữu ích trước khi sửa hoặc xóa trigger, vì MySQL không có `ALTER TRIGGER` để thay trực tiếp trigger body.

### 8.3. `INFORMATION_SCHEMA.TRIGGERS`

```sql
SELECT TRIGGER_NAME,
       EVENT_MANIPULATION,
       EVENT_OBJECT_TABLE,
       ACTION_TIMING,
       ACTION_ORDER,
       DEFINER,
       CREATED
FROM INFORMATION_SCHEMA.TRIGGERS
WHERE TRIGGER_SCHEMA = 'trigger_lab'
ORDER BY EVENT_OBJECT_TABLE,
         ACTION_TIMING,
         EVENT_MANIPULATION,
         ACTION_ORDER;
```

### 8.4. `DROP TRIGGER`

Cú pháp:

```sql
DROP TRIGGER [IF EXISTS] [schema_name.]trigger_name;
```

Ví dụ:

```sql
DROP TRIGGER IF EXISTS trigger_lab.trg_ai_sales_audit;
```

Sau khi drop, trigger không còn audit các insert mới. Muốn khôi phục, chạy lại script `CREATE TRIGGER`.

Nếu drop table, tất cả trigger gắn với table đó cũng bị drop.

### 8.5. Quy trình thay đổi trigger

```text
1. SHOW CREATE TRIGGER để sao lưu definition.
2. Lưu script migration/version control.
3. DROP TRIGGER IF EXISTS.
4. CREATE TRIGGER với definition mới.
5. Test DML representative.
6. Kiểm tra data side effects và metadata.
```

### Bài tập thực hành

**Bài 8.1.** Liệt kê tất cả trigger của `trigger_lab`.

**Bài 8.2.** Dùng `SHOW TRIGGERS ... LIKE 'sales'`.

**Bài 8.3.** Dùng `SHOW TRIGGERS ... WHERE` để lọc trigger có prefix `trg_bi`.

**Bài 8.4.** Dùng `SHOW CREATE TRIGGER` để lưu definition của `trg_ai_sales_audit`.

**Bài 8.5.** Viết các bước an toàn để thay đổi body của một trigger.

---

## 9. Các lỗi thường gặp

### Lỗi 1. Quên `FOR EACH ROW`

Sai:

```sql
CREATE TRIGGER trg_bad
BEFORE INSERT ON sales
BEGIN
    SET NEW.customer_code = UPPER(NEW.customer_code);
END;
```

Đúng:

```sql
CREATE TRIGGER trg_good
BEFORE INSERT ON sales
FOR EACH ROW
SET NEW.customer_code = UPPER(NEW.customer_code);
```

### Lỗi 2. Quên đổi delimiter cho body nhiều statement

Nếu body có `BEGIN ... END`, cần đổi delimiter trong mysql client/Workbench script.

### Lỗi 3. Dùng `OLD` trong INSERT hoặc `NEW` trong DELETE

Sai:

```sql
-- In an INSERT trigger
SET v_amount = OLD.amount;
```

Sai:

```sql
-- In a DELETE trigger
SET v_amount = NEW.amount;
```

### Lỗi 4. Sửa `NEW` trong AFTER trigger

`SET NEW.column = ...` chỉ có ý nghĩa trong BEFORE INSERT/UPDATE trigger.

### Lỗi 5. Dựa vào auto-increment ID trong BEFORE INSERT

`NEW.sale_id` trong BEFORE INSERT có thể là `0` trước khi server sinh ID.

### Lỗi 6. Cho trigger quá nhiều business logic

Trigger dài, gọi nhiều table, nhiều rule và nhiều side effect sẽ khó debug và có thể làm DML chậm. Tách logic hoặc dùng service/procedure có kiểm soát khi nghiệp vụ vượt quá mức đơn giản.

### Bài tập thực hành

**Bài 9.1.** Sửa trigger thiếu `FOR EACH ROW`.

**Bài 9.2.** Sửa script trigger nhiều statement nhưng quên delimiter.

**Bài 9.3.** Nêu alias hợp lệ trong DELETE trigger.

**Bài 9.4.** Nêu timing phù hợp để chỉnh `NEW.customer_code`.

**Bài 9.5.** Nêu hai lý do trigger có quá nhiều logic gây khó bảo trì.

---

## 10. Bài tập tổng hợp

### Bài 10.1. BEFORE INSERT validation

Tạo trigger `trg_bi_sales_amount_limit`:

1. Chạy trước insert vào `sales`.
2. Reject `amount <= 0`.
3. Reject `amount > 1000000.00`.
4. Gán `sale_date = CURRENT_DATE()` khi input là `NULL`.
5. Chuẩn hóa customer code thành uppercase.

> Khi thực hiện, cần gộp logic với trigger BEFORE INSERT hiện có hoặc thay thế trigger hiện có bằng version mới. Không tạo một trigger trùng tên.

### Bài 10.2. AFTER INSERT audit

Tạo trigger audit mới hoặc sửa `trg_ai_sales_audit` để:

1. Ghi `sale_id`.
2. Ghi customer code.
3. Ghi new amount.
4. Ghi status.
5. Ghi `USER()` và timestamp.

### Bài 10.3. Summary verification

1. Insert ba sales cùng ngày.
2. Tính tổng trực tiếp từ `sales`.
3. So sánh với `daily_sales_summary`.
4. Nêu lý do summary có thể bị sai nếu logic update/delete chưa xử lý.
5. Dự đoán phần nào của Tutorial 2 sẽ giải quyết việc này.

### Bài 10.4. Trigger metadata report

Viết query `INFORMATION_SCHEMA.TRIGGERS` hiển thị:

1. Trigger name.
2. Event.
3. Timing.
4. Subject table.
5. Definer.
6. Created time.
7. Action order.

### Bài 10.5. Cleanup có kiểm soát

1. Dùng `SHOW CREATE TRIGGER` để sao lưu hai trigger.
2. Drop hai trigger.
3. Insert một sale mới.
4. Xác minh summary/audit không còn thay đổi tự động.
5. Tạo lại trigger từ script đã sao lưu.

---

## 11. Đáp án gợi ý

### Bài 10.1 — Version mở rộng của BEFORE INSERT trigger

```sql
DROP TRIGGER IF EXISTS trigger_lab.trg_bi_sales_prepare_summary;

DELIMITER $$

CREATE TRIGGER trg_bi_sales_prepare_summary
BEFORE INSERT ON sales
FOR EACH ROW
BEGIN
    IF NEW.amount IS NULL
       OR NEW.amount <= 0
       OR NEW.amount > 1000000.00 THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'Sale amount must be between 0 and 1000000';
    END IF;

    IF NEW.sale_date IS NULL THEN
        SET NEW.sale_date = CURRENT_DATE();
    END IF;

    IF NEW.customer_code IS NULL
       OR CHAR_LENGTH(TRIM(NEW.customer_code)) = 0 THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'Customer code is required';
    END IF;

    SET NEW.customer_code = UPPER(TRIM(NEW.customer_code));

    INSERT INTO daily_sales_summary (
        sale_date,
        sale_count,
        total_amount
    )
    VALUES (
        NEW.sale_date,
        1,
        NEW.amount
    )
    ON DUPLICATE KEY UPDATE
        sale_count = sale_count + 1,
        total_amount = total_amount + NEW.amount;
END$$

DELIMITER ;
```

### Bài 10.2

```sql
SELECT event_id,
       sale_id,
       event_type,
       customer_code,
       new_amount,
       new_status,
       actor_login,
       event_time
FROM sales_event_log
WHERE event_type = 'INSERT'
ORDER BY event_id;
```

### Bài 10.3

```sql
SELECT sale_date,
       COUNT(*) AS actual_sale_count,
       SUM(amount) AS actual_total_amount
FROM sales
GROUP BY sale_date
ORDER BY sale_date;
```

```sql
SELECT sale_date,
       sale_count,
       total_amount
FROM daily_sales_summary
ORDER BY sale_date;
```

### Bài 10.4

```sql
SELECT TRIGGER_NAME,
       EVENT_MANIPULATION AS eventType,
       ACTION_TIMING AS timing,
       EVENT_OBJECT_TABLE AS subjectTable,
       DEFINER,
       CREATED,
       ACTION_ORDER
FROM INFORMATION_SCHEMA.TRIGGERS
WHERE TRIGGER_SCHEMA = 'trigger_lab'
ORDER BY EVENT_OBJECT_TABLE,
         ACTION_TIMING,
         EVENT_MANIPULATION,
         ACTION_ORDER;
```

---

## 12. Tóm tắt

- MySQL trigger là named stored program gắn với table và tự chạy khi `INSERT`, `UPDATE` hoặc `DELETE`.
- MySQL chỉ có row-level trigger; trigger chạy một lần cho mỗi row bị ảnh hưởng.
- `BEFORE` trigger phù hợp để validate/normalize `NEW` values.
- `AFTER` trigger phù hợp để audit khi row operation đã thành công.
- INSERT dùng `NEW`; UPDATE dùng cả `OLD` và `NEW`; DELETE dùng `OLD`.
- `OLD` read-only; chỉ BEFORE INSERT/UPDATE mới có thể gán `NEW.column`.
- Dùng `SHOW TRIGGERS`, `SHOW CREATE TRIGGER` và `INFORMATION_SCHEMA.TRIGGERS` để kiểm tra metadata.
- Dùng `DROP TRIGGER` để xóa; muốn sửa trigger body thì dùng drop + create.
- Trigger cần được quản lý như code: có migration script, test và naming convention.

---

## 13. Từ khóa chính

- Trigger
- Row-level trigger
- BEFORE
- AFTER
- INSERT
- UPDATE
- DELETE
- OLD
- NEW
- CREATE TRIGGER
- DROP TRIGGER
- SHOW TRIGGERS
- SHOW CREATE TRIGGER
- INFORMATION_SCHEMA.TRIGGERS
- SIGNAL
- DELIMITER
- Audit log
- Summary table
- Trigger definer
