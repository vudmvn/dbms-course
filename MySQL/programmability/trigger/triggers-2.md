---
title: "Tutorial 2: MySQL Triggers — UPDATE, DELETE, Validation và Audit"
author: "Tên giảng viên"
duration: "180m"
difficulty: "Intermediate"
prerequisites:
  - "Đã hoàn thành Tutorial 1 về trigger foundations và INSERT triggers"
  - "Schema trigger_lab và các table sales, daily_sales_summary, sales_event_log đã tồn tại"
  - "Đã biết OLD/NEW, SIGNAL, IF và DELIMITER"
summary: "Thực hành BEFORE UPDATE validation, AFTER UPDATE auditing, BEFORE DELETE protection và AFTER DELETE archival; đồng bộ summary table khi sales được cập nhật hoặc xóa."
---

# Tutorial 2: MySQL Triggers — `UPDATE`, `DELETE`, Validation và Audit

## Link tham khảo

- [MySQL Tutorial — BEFORE UPDATE Trigger](https://www.mysqltutorial.org/mysql-triggers/mysql-before-update-trigger/)
- [MySQL Tutorial — AFTER UPDATE Trigger](https://www.mysqltutorial.org/mysql-triggers/mysql-after-update-trigger/)
- [MySQL Tutorial — BEFORE DELETE Trigger](https://www.mysqltutorial.org/mysql-triggers/mysql-before-delete-trigger/)
- [MySQL Tutorial — AFTER DELETE Trigger](https://www.mysqltutorial.org/mysql-triggers/mysql-after-delete-trigger/)
- [MySQL 8.4 Reference Manual — Trigger Syntax and Examples](https://dev.mysql.com/doc/refman/8.4/en/trigger-syntax.html)
- [MySQL 8.4 Reference Manual — SIGNAL Statement](https://dev.mysql.com/doc/refman/8.4/en/signal.html)
- [MySQL 8.4 Reference Manual — Restrictions on Stored Programs](https://dev.mysql.com/doc/refman/8.4/en/stored-program-restrictions.html)

> **Phạm vi lab:** Tutorial này tiếp tục dùng `trigger_lab` từ Tutorial 1. Các trigger cập nhật summary và audit table là minh họa học thuật. Hệ thống thực cần đánh giá concurrency, transaction, retention, compliance và performance trước khi đặt logic tương tự trên bảng nghiệp vụ lớn.

---

## 1. Mục tiêu học tập

Sau khi hoàn thành tutorial này, người học có thể:

1. Dùng `OLD` và `NEW` để kiểm tra thay đổi trong UPDATE trigger.
2. Dùng `BEFORE UPDATE` để validate transition và chuẩn hóa dữ liệu.
3. Đồng bộ summary table khi `amount` hoặc `sale_date` thay đổi.
4. Dùng `AFTER UPDATE` để log old/new values.
5. Dùng null-safe equality operator `<=>` để phát hiện thay đổi.
6. Dùng `BEFORE DELETE` để ngăn xóa row theo business rule.
7. Dùng `AFTER DELETE` để archive row bị xóa và cập nhật summary.
8. Hiểu error trong trigger làm statement gây trigger thất bại.
9. Phân biệt `DELETE` với `TRUNCATE TABLE` về trigger activation.
10. Thiết kế test cases cho trigger update/delete.

---

## 2. Chuẩn bị extension tables

Chọn schema:

```sql
USE trigger_lab;
```

Tạo audit table chi tiết cho UPDATE:

```sql
CREATE TABLE IF NOT EXISTS sales_change_log (
    change_id BIGINT NOT NULL AUTO_INCREMENT,
    sale_id BIGINT NOT NULL,
    changed_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    actor_login VARCHAR(288) NULL,
    old_sale_date DATE NULL,
    new_sale_date DATE NULL,
    old_customer_code VARCHAR(20) NULL,
    new_customer_code VARCHAR(20) NULL,
    old_amount DECIMAL(12,2) NULL,
    new_amount DECIMAL(12,2) NULL,
    old_status VARCHAR(20) NULL,
    new_status VARCHAR(20) NULL,
    PRIMARY KEY (change_id)
) ENGINE = InnoDB;
```

Tạo archive table cho DELETE:

```sql
CREATE TABLE IF NOT EXISTS deleted_sales_archive (
    archive_id BIGINT NOT NULL AUTO_INCREMENT,
    original_sale_id BIGINT NOT NULL,
    sale_date DATE NULL,
    customer_code VARCHAR(20) NULL,
    amount DECIMAL(12,2) NULL,
    status VARCHAR(20) NULL,
    original_created_at DATETIME NULL,
    original_updated_at DATETIME NULL,
    deleted_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    deleted_by VARCHAR(288) NULL,
    PRIMARY KEY (archive_id)
) ENGINE = InnoDB;
```

Kiểm tra:

```sql
SHOW CREATE TABLE sales_change_log;
SHOW CREATE TABLE deleted_sales_archive;
```

### Bài tập thực hành

**Bài 2.1.** Chọn schema `trigger_lab`.

**Bài 2.2.** Tạo `sales_change_log`.

**Bài 2.3.** Tạo `deleted_sales_archive`.

**Bài 2.4.** Dùng `SHOW CREATE TABLE` kiểm tra DDL của hai table.

**Bài 2.5.** Nêu mục đích khác nhau của `sales_change_log` và `deleted_sales_archive`.

---

## 3. BEFORE UPDATE trigger: validation và summary maintenance

### 3.1. Business rules lab

Trigger `trg_bu_sales_validate_summary` áp dụng các rule:

1. `amount` phải dương.
2. `customer_code` không được rỗng và được normalize uppercase.
3. Không được chuyển status từ `POSTED` về `DRAFT`.
4. Không cho thay đổi row đã `CANCELLED`, trừ khi giá trị update thực tế không thay đổi.
5. Nếu `sale_date` hoặc `amount` thay đổi, summary cũ được giảm và summary mới được tăng.

### 3.2. Tạo trigger

```sql
DROP TRIGGER IF EXISTS trigger_lab.trg_bu_sales_validate_summary;

DELIMITER $$

CREATE TRIGGER trg_bu_sales_validate_summary
BEFORE UPDATE ON sales
FOR EACH ROW
BEGIN
    IF NEW.amount IS NULL OR NEW.amount <= 0 THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'Sale amount must be greater than 0';
    END IF;

    IF NEW.sale_date IS NULL THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'Sale date is required';
    END IF;

    IF NEW.customer_code IS NULL
       OR CHAR_LENGTH(TRIM(NEW.customer_code)) = 0 THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'Customer code is required';
    END IF;

    SET NEW.customer_code = UPPER(TRIM(NEW.customer_code));

    IF OLD.status = 'POSTED'
       AND NEW.status = 'DRAFT' THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'Posted sales cannot return to DRAFT';
    END IF;

    IF OLD.status = 'CANCELLED'
       AND (
           NOT (OLD.sale_date <=> NEW.sale_date)
           OR NOT (OLD.customer_code <=> NEW.customer_code)
           OR NOT (OLD.amount <=> NEW.amount)
           OR NOT (OLD.status <=> NEW.status)
       ) THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'Cancelled sales cannot be changed';
    END IF;

    IF NOT (OLD.sale_date <=> NEW.sale_date)
       OR NOT (OLD.amount <=> NEW.amount) THEN

        UPDATE daily_sales_summary
        SET sale_count = sale_count - 1,
            total_amount = total_amount - OLD.amount
        WHERE sale_date = OLD.sale_date;

        DELETE FROM daily_sales_summary
        WHERE sale_date = OLD.sale_date
          AND sale_count = 0;

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
    END IF;
END$$

DELIMITER ;
```

### 3.3. Tại sao dùng `<=>`?

Trong SQL, biểu thức:

```sql
OLD.sale_date <> NEW.sale_date
```

có thể trả `NULL` khi một bên là `NULL`, không phải `TRUE`/`FALSE` rõ ràng.

Toán tử null-safe equality:

```sql
OLD.sale_date <=> NEW.sale_date
```

trả:

```text
1 nếu hai giá trị bằng nhau, kể cả NULL với NULL
0 nếu khác nhau
```

Vì vậy:

```sql
NOT (OLD.sale_date <=> NEW.sale_date)
```

là cách rõ ràng để kiểm tra “đã thay đổi” khi column có thể `NULL`.

### 3.4. Test update hợp lệ

Tạo data test riêng:

```sql
INSERT INTO sales (
    sale_date,
    customer_code,
    amount,
    status
)
VALUES
    ('2026-02-01', 'upd001', 100.00, 'DRAFT'),
    ('2026-02-01', 'del001', 60.00, 'DRAFT'),
    ('2026-02-01', 'post001', 80.00, 'POSTED'),
    ('2026-02-01', 'can001', 40.00, 'CANCELLED');
```

Update amount:

```sql
UPDATE sales
SET amount = 125.00
WHERE customer_code = 'UPD001';
```

Kiểm tra:

```sql
SELECT sale_id, sale_date, customer_code, amount, status
FROM sales
WHERE customer_code = 'UPD001';
```

```sql
SELECT sale_date, sale_count, total_amount
FROM daily_sales_summary
WHERE sale_date = '2026-02-01';
```

### 3.5. Test transition không hợp lệ

```sql
UPDATE sales
SET status = 'DRAFT'
WHERE customer_code = 'POST001';
```

Statement phải bị chặn.

Thử thay đổi cancelled row:

```sql
UPDATE sales
SET amount = 50.00
WHERE customer_code = 'CAN001';
```

Statement cũng phải bị chặn.

### 3.6. Lưu ý về summary table

Summary trigger chỉ minh họa một cách duy trì aggregate. Trong production:

- Cần xử lý INSERT/UPDATE/DELETE đầy đủ, như tutorial này.
- Cần tính đến concurrent updates.
- Cần xác định source of truth là base table hay summary table.
- Cần có reconciliation query/job để phát hiện drift.
- Có thể không phù hợp khi aggregation quá nhiều chiều hoặc write volume rất cao.

### Bài tập thực hành

**Bài 3.1.** Tạo `trg_bu_sales_validate_summary`.

**Bài 3.2.** Update `UPD001` từ `100.00` thành `125.00`; kiểm tra summary.

**Bài 3.3.** Update `UPD001` sang `sale_date = '2026-02-02'`; kiểm tra summary của cả hai ngày.

**Bài 3.4.** Thử chuyển `POSTED` về `DRAFT`.

**Bài 3.5.** Giải thích vì sao dùng `<=>` khi kiểm tra thay đổi của cột có thể NULL.

---

## 4. AFTER UPDATE trigger: audit old/new values

### 4.1. Mục tiêu

`AFTER UPDATE` trigger ghi log sau khi row đã update thành công.

Audit log chỉ được ghi khi ít nhất một business field thay đổi:

```text
sale_date
customer_code
amount
status
```

### 4.2. Tạo trigger

```sql
DROP TRIGGER IF EXISTS trigger_lab.trg_au_sales_audit;

DELIMITER $$

CREATE TRIGGER trg_au_sales_audit
AFTER UPDATE ON sales
FOR EACH ROW
BEGIN
    IF NOT (OLD.sale_date <=> NEW.sale_date)
       OR NOT (OLD.customer_code <=> NEW.customer_code)
       OR NOT (OLD.amount <=> NEW.amount)
       OR NOT (OLD.status <=> NEW.status) THEN

        INSERT INTO sales_change_log (
            sale_id,
            actor_login,
            old_sale_date,
            new_sale_date,
            old_customer_code,
            new_customer_code,
            old_amount,
            new_amount,
            old_status,
            new_status
        )
        VALUES (
            NEW.sale_id,
            USER(),
            OLD.sale_date,
            NEW.sale_date,
            OLD.customer_code,
            NEW.customer_code,
            OLD.amount,
            NEW.amount,
            OLD.status,
            NEW.status
        );

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
            'UPDATE',
            NEW.customer_code,
            OLD.amount,
            NEW.amount,
            OLD.status,
            NEW.status,
            USER(),
            'Sale updated'
        );
    END IF;
END$$

DELIMITER ;
```

### 4.3. Test audit

```sql
UPDATE sales
SET amount = 150.00,
    status = 'POSTED'
WHERE customer_code = 'UPD001';
```

Kiểm tra log chi tiết:

```sql
SELECT change_id,
       sale_id,
       old_amount,
       new_amount,
       old_status,
       new_status,
       actor_login,
       changed_at
FROM sales_change_log
ORDER BY change_id;
```

Kiểm tra event log:

```sql
SELECT event_id,
       sale_id,
       event_type,
       old_amount,
       new_amount,
       old_status,
       new_status,
       note
FROM sales_event_log
ORDER BY event_id;
```

### 4.4. Chỉ log meaningful changes

Column `updated_at` có thể tự thay đổi do `ON UPDATE CURRENT_TIMESTAMP`.

Nếu audit trigger log mọi UPDATE không điều kiện, các update không đổi business values cũng có thể tạo log noise.

Vì vậy trigger dùng điều kiện `<=>` để chỉ log khi business fields đổi.

### 4.5. Audit không thay thế mọi audit requirement

Audit trigger đơn giản có thể thiếu:

- Application request ID.
- Business user ID tách biệt database account.
- Client IP chuẩn hóa.
- Reason code.
- Tamper resistance.
- Retention, encryption và compliance controls.

Trong hệ thống cần audit nghiêm ngặt, thiết kế audit cần được xem xét ở cả application, database, logging platform và security policy.

### Bài tập thực hành

**Bài 4.1.** Tạo `trg_au_sales_audit`.

**Bài 4.2.** Update amount của một sale DRAFT và xem `sales_change_log`.

**Bài 4.3.** Update status từ DRAFT sang POSTED và kiểm tra old/new status.

**Bài 4.4.** Chạy một UPDATE không làm thay đổi business field, sau đó kiểm tra có audit row mới hay không.

**Bài 4.5.** Nêu ba thông tin audit trigger đơn giản có thể chưa đủ cho hệ thống production.

---

## 5. BEFORE DELETE trigger: bảo vệ row

### 5.1. Rule lab

Không cho phép delete sale đã `POSTED`.

Lý do minh họa:

```text
POSTED sale đại diện giao dịch đã hoàn tất.
Nếu cần đảo giao dịch, nên có quy trình cancel/reversal thay vì delete vật lý.
```

### 5.2. Tạo trigger

```sql
DROP TRIGGER IF EXISTS trigger_lab.trg_bd_sales_protect;

DELIMITER $$

CREATE TRIGGER trg_bd_sales_protect
BEFORE DELETE ON sales
FOR EACH ROW
BEGIN
    IF OLD.status = 'POSTED' THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'Posted sales cannot be deleted';
    END IF;
END$$

DELIMITER ;
```

### 5.3. Test

Delete DRAFT sale:

```sql
DELETE FROM sales
WHERE customer_code = 'DEL001';
```

Delete POSTED sale:

```sql
DELETE FROM sales
WHERE customer_code = 'POST001';
```

Lệnh thứ hai phải bị chặn.

### 5.4. BEFORE DELETE phù hợp khi nào?

Dùng BEFORE DELETE khi:

- Cần chặn delete theo status/ownership/rule.
- Cần validate trước khi row biến mất.
- Cần trả lỗi rõ ràng bằng `SIGNAL`.

Nhiều hệ thống thay delete bằng soft delete:

```text
is_deleted
deleted_at
deleted_by
status = CANCELLED
```

Soft delete cũng có trade-off: query cần filter đúng, unique constraints cần thiết kế cẩn thận, data retention tăng.

### Bài tập thực hành

**Bài 5.1.** Tạo `trg_bd_sales_protect`.

**Bài 5.2.** Xóa một sale DRAFT.

**Bài 5.3.** Thử xóa một sale POSTED.

**Bài 5.4.** Nêu hai tình huống business phù hợp để chặn DELETE.

**Bài 5.5.** So sánh hard delete với soft delete ở mức khái quát.

---

## 6. AFTER DELETE trigger: archive và summary reconciliation

### 6.1. Mục tiêu

Sau khi row delete thành công, trigger:

1. Lưu row cũ vào `deleted_sales_archive`.
2. Trừ count/amount khỏi summary table.
3. Xóa summary row nếu count bằng 0.
4. Ghi event log.

### 6.2. Tạo trigger

```sql
DROP TRIGGER IF EXISTS trigger_lab.trg_ad_sales_archive_summary;

DELIMITER $$

CREATE TRIGGER trg_ad_sales_archive_summary
AFTER DELETE ON sales
FOR EACH ROW
BEGIN
    INSERT INTO deleted_sales_archive (
        original_sale_id,
        sale_date,
        customer_code,
        amount,
        status,
        original_created_at,
        original_updated_at,
        deleted_by
    )
    VALUES (
        OLD.sale_id,
        OLD.sale_date,
        OLD.customer_code,
        OLD.amount,
        OLD.status,
        OLD.created_at,
        OLD.updated_at,
        USER()
    );

    UPDATE daily_sales_summary
    SET sale_count = sale_count - 1,
        total_amount = total_amount - OLD.amount
    WHERE sale_date = OLD.sale_date;

    DELETE FROM daily_sales_summary
    WHERE sale_date = OLD.sale_date
      AND sale_count = 0;

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
        OLD.sale_id,
        'DELETE',
        OLD.customer_code,
        OLD.amount,
        NULL,
        OLD.status,
        NULL,
        USER(),
        'Sale deleted and archived'
    );
END$$

DELIMITER ;
```

### 6.3. Test archive

Insert a DRAFT sale:

```sql
INSERT INTO sales (
    sale_date,
    customer_code,
    amount,
    status
)
VALUES (
    '2026-02-03',
    'ARC001',
    75.00,
    'DRAFT'
);
```

Delete it:

```sql
DELETE FROM sales
WHERE customer_code = 'ARC001';
```

Check archive:

```sql
SELECT archive_id,
       original_sale_id,
       sale_date,
       customer_code,
       amount,
       status,
       deleted_by,
       deleted_at
FROM deleted_sales_archive
ORDER BY archive_id;
```

Check summary:

```sql
SELECT *
FROM daily_sales_summary
WHERE sale_date = '2026-02-03';
```

### 6.4. DELETE và TRUNCATE khác nhau

`DELETE FROM sales` kích hoạt DELETE triggers row-by-row.

`TRUNCATE TABLE sales` không kích hoạt DELETE triggers, vì `TRUNCATE` không thực hiện DELETE statement theo semantics trigger.

Do đó, không dùng `TRUNCATE` trên table có archive/summary delete logic nếu bạn kỳ vọng trigger sẽ archive dữ liệu hoặc cập nhật summary.

### Bài tập thực hành

**Bài 6.1.** Tạo `trg_ad_sales_archive_summary`.

**Bài 6.2.** Insert rồi delete một sale DRAFT; kiểm tra archive.

**Bài 6.3.** Kiểm tra event log sau delete.

**Bài 6.4.** Kiểm tra summary table sau delete.

**Bài 6.5.** Giải thích vì sao `TRUNCATE TABLE` không thay thế `DELETE FROM` khi cần DELETE trigger.

---

## 7. Error behavior và transaction considerations

### 7.1. Khi BEFORE trigger lỗi

Nếu BEFORE trigger phát sinh error:

- Row operation tương ứng không được thực hiện.
- AFTER trigger không chạy.
- Statement gây trigger thất bại.

Ví dụ `SIGNAL SQLSTATE '45000'` trong BEFORE UPDATE/DELETE sẽ chặn DML.

### 7.2. Khi AFTER trigger lỗi

Nếu AFTER trigger lỗi:

- Statement gây trigger cũng thất bại.
- Với transactional table như InnoDB, các thay đổi của statement thường rollback.
- Với nontransactional table, một số thay đổi trước thời điểm lỗi có thể còn tồn tại.

Do đó, lab dùng InnoDB cho mọi table.

### 7.3. Trigger không quản lý transaction

Không dùng trong trigger:

```sql
START TRANSACTION;
COMMIT;
ROLLBACK;
```

Trigger tham gia vào statement/transaction của caller; không tự kết thúc transaction.

### 7.4. Multi-row DML

Với:

```sql
UPDATE sales
SET amount = amount * 1.05
WHERE status = 'DRAFT';
```

trigger chạy một lần cho mỗi row cập nhật. Audit table có thể nhận nhiều rows; summary logic cũng chạy nhiều lần.

Cần đánh giá overhead trước khi dùng trigger row-by-row với batch DML lớn.

### Bài tập thực hành

**Bài 7.1.** Nêu điều gì xảy ra khi BEFORE trigger phát sinh `SIGNAL`.

**Bài 7.2.** Nêu điều gì xảy ra với AFTER trigger nếu row operation trước đó thất bại.

**Bài 7.3.** Giải thích vì sao không dùng `COMMIT` trong trigger.

**Bài 7.4.** Nếu một UPDATE ảnh hưởng 1,000 rows, AFTER UPDATE trigger chạy bao nhiêu lần?

**Bài 7.5.** Nêu một lý do batch update lớn có thể làm audit trigger tốn chi phí.

---

## 8. Các lỗi thường gặp

### Lỗi 1. Dùng `=` thay vì `<=>` để so sánh nullable values

Không nên chỉ viết:

```sql
IF OLD.sale_date <> NEW.sale_date THEN
    ...
END IF;
```

nếu column có thể `NULL`.

Dùng:

```sql
IF NOT (OLD.sale_date <=> NEW.sale_date) THEN
    ...
END IF;
```

### Lỗi 2. Archive từ `NEW` trong DELETE trigger

DELETE trigger chỉ có:

```sql
OLD.column_name
```

### Lỗi 3. Cập nhật chính table đang bị update/delete

Không được để trigger modify table đã đang được statement gây trigger sử dụng để đọc hoặc ghi.

Ví dụ, AFTER UPDATE trigger trên `sales` không nên chạy:

```sql
UPDATE sales
SET ...
WHERE sale_id = NEW.sale_id;
```

Điều này vi phạm restriction và có nguy cơ recursion/side effect khó kiểm soát.

### Lỗi 4. Không xử lý summary khi update/delete

Nếu INSERT trigger tăng summary nhưng UPDATE/DELETE không điều chỉnh summary, total sẽ drift so với base table.

### Lỗi 5. Dùng DELETE trigger với expectation rằng TRUNCATE cũng kích hoạt

`TRUNCATE TABLE` không kích hoạt DELETE trigger.

### Bài tập thực hành

**Bài 8.1.** Sửa điều kiện so sánh nullable column dùng `<=>`.

**Bài 8.2.** Sửa AFTER DELETE trigger dùng `NEW.amount`.

**Bài 8.3.** Giải thích restriction về table đã được invoking statement sử dụng.

**Bài 8.4.** Nêu cách summary drift có thể xảy ra.

**Bài 8.5.** Nêu một cách an toàn để reset data lab khi DELETE trigger có side effect.

---

## 9. Bài tập tổng hợp

### Bài 9.1. Update validation

Mở rộng BEFORE UPDATE trigger để:

1. Không cho `amount > 500000.00`.
2. Không cho đổi `customer_code` khi status là `POSTED`.
3. Vẫn cho đổi amount khi status là DRAFT.
4. Vẫn cập nhật summary đúng.
5. Test cả success và failure case.

### Bài 9.2. Update audit

Mở rộng `sales_change_log` hoặc trigger để:

1. Ghi reason code placeholder.
2. Chỉ log khi amount thay đổi.
3. Lưu actor login.
4. Lưu old/new amount.
5. Kiểm tra audit output.

### Bài 9.3. Delete protection + archive

1. Chặn delete POSTED.
2. Cho delete DRAFT.
3. Archive DRAFT đã delete.
4. Ghi delete event.
5. Đồng bộ summary.

### Bài 9.4. Reconciliation query

Viết query so sánh:

- `COUNT(*)` và `SUM(amount)` từ `sales` theo `sale_date`.
- `sale_count` và `total_amount` từ `daily_sales_summary`.

Liệt kê ngày có mismatch.

### Bài 9.5. Multi-row update impact

1. Insert ba DRAFT sales cùng ngày.
2. Update tất cả amount tăng 10%.
3. Kiểm tra số audit rows được tạo.
4. Kiểm tra summary.
5. Giải thích vì sao trigger chạy row-by-row.

---

## 10. Đáp án gợi ý

### Bài 9.4 — Reconciliation query

```sql
SELECT COALESCE(base.sale_date, summary.sale_date) AS sale_date,
       base.actual_sale_count,
       base.actual_total_amount,
       summary.sale_count AS summary_sale_count,
       summary.total_amount AS summary_total_amount
FROM (
    SELECT sale_date,
           COUNT(*) AS actual_sale_count,
           SUM(amount) AS actual_total_amount
    FROM sales
    GROUP BY sale_date
) AS base
LEFT JOIN daily_sales_summary AS summary
    ON summary.sale_date = base.sale_date

UNION ALL

SELECT summary.sale_date,
       NULL AS actual_sale_count,
       NULL AS actual_total_amount,
       summary.sale_count,
       summary.total_amount
FROM daily_sales_summary AS summary
LEFT JOIN (
    SELECT sale_date
    FROM sales
    GROUP BY sale_date
) AS base
    ON base.sale_date = summary.sale_date
WHERE base.sale_date IS NULL;
```

### Bài 9.5

```sql
INSERT INTO sales (
    sale_date,
    customer_code,
    amount,
    status
)
VALUES
    ('2026-02-10', 'BATCH001', 100.00, 'DRAFT'),
    ('2026-02-10', 'BATCH002', 200.00, 'DRAFT'),
    ('2026-02-10', 'BATCH003', 300.00, 'DRAFT');
```

```sql
UPDATE sales
SET amount = amount * 1.10
WHERE sale_date = '2026-02-10'
  AND status = 'DRAFT';
```

```sql
SELECT COUNT(*) AS updateAuditRows
FROM sales_change_log
WHERE changed_at >= CURRENT_DATE();
```

---

## 11. Tóm tắt

- BEFORE UPDATE trigger phù hợp để validate transition, normalize `NEW`, và duy trì dữ liệu phụ thuộc trước update.
- AFTER UPDATE trigger phù hợp để ghi audit old/new values sau update thành công.
- BEFORE DELETE trigger phù hợp để bảo vệ row theo rule nghiệp vụ.
- AFTER DELETE trigger phù hợp để archive row cũ và điều chỉnh derived data.
- UPDATE trigger có cả `OLD` và `NEW`; DELETE trigger chỉ có `OLD`.
- `<=>` giúp so sánh nullable values đúng cách.
- Error trong trigger làm statement gây trigger thất bại; InnoDB rollback statement changes.
- Trigger chạy một lần cho mỗi row, nên cần cân nhắc batch DML và performance.
- `TRUNCATE TABLE` không kích hoạt DELETE triggers.
- Không để trigger modify chính table đang được statement gây trigger sử dụng.

---

## 12. Từ khóa chính

- BEFORE UPDATE
- AFTER UPDATE
- BEFORE DELETE
- AFTER DELETE
- OLD
- NEW
- Null-safe equality
- <=>
- SIGNAL
- Audit log
- Archive table
- Summary reconciliation
- InnoDB
- Transaction rollback
- TRUNCATE TABLE
- Row-by-row processing
- Business rule
