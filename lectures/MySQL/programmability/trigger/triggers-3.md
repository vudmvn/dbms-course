---
title: "Tutorial 3: MySQL Triggers — Multiple Triggers, Metadata, Restrictions và Best Practices"
author: "Tên giảng viên"
duration: "180m"
difficulty: "Intermediate–Advanced"
prerequisites:
  - "Đã hoàn thành Tutorial 1 và 2 về INSERT, UPDATE, DELETE triggers"
  - "Có schema trigger_lab"
  - "Đã biết SHOW TRIGGERS, SHOW CREATE TRIGGER và INFORMATION_SCHEMA"
summary: "Thực hành nhiều trigger cùng table/event/timing bằng PRECEDES và FOLLOWS; quản lý metadata, security/definer, restrictions, test strategy và cleanup trigger_lab."
---

# Tutorial 3: MySQL Triggers — Multiple Triggers, Metadata, Restrictions và Best Practices

## Link tham khảo

- [MySQL Tutorial — Multiple Triggers](https://www.mysqltutorial.org/mysql-triggers/mysql-multiple-triggers/)
- [MySQL Tutorial — Show Triggers](https://www.mysqltutorial.org/mysql-triggers/mysql-show-triggers/)
- [MySQL 8.4 Reference Manual — CREATE TRIGGER](https://dev.mysql.com/doc/refman/8.4/en/create-trigger.html)
- [MySQL 8.4 Reference Manual — Trigger Syntax and Examples](https://dev.mysql.com/doc/refman/8.4/en/trigger-syntax.html)
- [MySQL 8.4 Reference Manual — Trigger Metadata](https://dev.mysql.com/doc/refman/8.4/en/trigger-metadata.html)
- [MySQL 8.4 Reference Manual — SHOW TRIGGERS](https://dev.mysql.com/doc/refman/8.4/en/show-triggers.html)
- [MySQL 8.4 Reference Manual — Restrictions on Stored Programs](https://dev.mysql.com/doc/refman/8.4/en/stored-program-restrictions.html)

> **Version note:** Current MySQL versions, including MySQL 8.0/8.4, allow multiple triggers with the same trigger event and action time on one table. Dùng `PRECEDES` hoặc `FOLLOWS` nếu thứ tự trigger có dependency. Với deployment cũ, luôn kiểm tra exact server version và release documentation trước khi áp dụng feature này.

---

## 1. Mục tiêu học tập

Sau khi hoàn thành tutorial này, người học có thể:

1. Tạo nhiều trigger cùng `BEFORE INSERT` hoặc cùng event/timing trên một table.
2. Giải thích default execution order và dùng `PRECEDES`/`FOLLOWS`.
3. Dùng `SHOW TRIGGERS`, `SHOW CREATE TRIGGER` và `INFORMATION_SCHEMA.TRIGGERS` để quản lý metadata.
4. Hiểu trigger `DEFINER` và privilege context cơ bản.
5. Nhận biết các trigger restrictions quan trọng.
6. Phân biệt trigger activation bởi DML với foreign-key cascade hoặc `TRUNCATE`.
7. Áp dụng quy trình thay đổi trigger bằng drop + create.
8. Viết test cases cho trigger side effects và error paths.
9. Nhận biết performance, replication và observability concerns.
10. Dọn dẹp schema lab an toàn sau khi hoàn thành.

---

## 2. Nhiều trigger cùng table, event và timing

### 2.1. Khả năng của MySQL

Một table có thể có nhiều trigger có cùng:

```text
timing  = BEFORE hoặc AFTER
event   = INSERT, UPDATE hoặc DELETE
```

Ví dụ: hai `BEFORE INSERT` trigger trên cùng table.

Nếu không quy định thứ tự, triggers có cùng event/timing chạy theo thứ tự được tạo.

Khi thứ tự quan trọng, dùng:

```sql
FOLLOWS existing_trigger_name
```

hoặc:

```sql
PRECEDES existing_trigger_name
```

sau `FOR EACH ROW`.

### 2.2. Cú pháp

```sql
CREATE TRIGGER trigger_name
BEFORE INSERT
ON table_name
FOR EACH ROW
FOLLOWS existing_trigger_name
trigger_body;
```

Hoặc:

```sql
CREATE TRIGGER trigger_name
BEFORE INSERT
ON table_name
FOR EACH ROW
PRECEDES existing_trigger_name
trigger_body;
```

`existing_trigger_name` phải là trigger trên cùng table, cùng event và cùng timing.

### 2.3. Khi nào thứ tự quan trọng?

Ví dụ:

```text
Trigger A: chuẩn hóa NEW.customer_code
Trigger B: validate customer_code theo format chuẩn hóa
```

Nếu Trigger B chạy trước Trigger A, validation có thể thất bại hoặc dùng dữ liệu chưa chuẩn hóa.

Tuy nhiên, nếu nhiều trigger cần phụ thuộc thứ tự chặt chẽ, đó cũng là tín hiệu cần xem lại thiết kế:

- Có thể gộp logic vào một trigger rõ ràng hơn.
- Có thể đưa workflow vào procedure/service layer.
- Có thể dùng constraints thay cho một phần validation.

### Bài tập thực hành

**Bài 2.1.** Nêu default order của nhiều trigger cùng event/timing.

**Bài 2.2.** Giải thích `FOLLOWS` khác `PRECEDES`.

**Bài 2.3.** Nêu một ví dụ trigger order ảnh hưởng logic.

**Bài 2.4.** Viết skeleton tạo trigger `FOLLOWS trg_bi_base`.

**Bài 2.5.** Nêu một lý do nên gộp trigger thay vì tạo quá nhiều trigger phụ thuộc nhau.

---

## 3. Lab: quan sát thứ tự trigger

### 3.1. Tạo demo tables

```sql
USE trigger_lab;

DROP TABLE IF EXISTS trigger_order_log;
DROP TABLE IF EXISTS trigger_order_demo;

CREATE TABLE trigger_order_demo (
    demo_id BIGINT NOT NULL AUTO_INCREMENT,
    demo_code VARCHAR(20) NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    PRIMARY KEY (demo_id)
) ENGINE = InnoDB;

CREATE TABLE trigger_order_log (
    execution_id BIGINT NOT NULL AUTO_INCREMENT,
    demo_code VARCHAR(20) NOT NULL,
    trigger_name VARCHAR(100) NOT NULL,
    note VARCHAR(255) NOT NULL,
    fired_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (execution_id)
) ENGINE = InnoDB;
```

### 3.2. Tạo base trigger

```sql
DROP TRIGGER IF EXISTS trigger_lab.trg_bi_demo_base;
DROP TRIGGER IF EXISTS trigger_lab.trg_bi_demo_precedes_base;
DROP TRIGGER IF EXISTS trigger_lab.trg_bi_demo_follows_base;
```

```sql
DELIMITER $$

CREATE TRIGGER trg_bi_demo_base
BEFORE INSERT ON trigger_order_demo
FOR EACH ROW
BEGIN
    INSERT INTO trigger_order_log (
        demo_code,
        trigger_name,
        note
    )
    VALUES (
        NEW.demo_code,
        'trg_bi_demo_base',
        'Base trigger fired'
    );
END$$

DELIMITER ;
```

### 3.3. Tạo trigger chạy trước base

```sql
DELIMITER $$

CREATE TRIGGER trg_bi_demo_precedes_base
BEFORE INSERT ON trigger_order_demo
FOR EACH ROW
PRECEDES trg_bi_demo_base
BEGIN
    INSERT INTO trigger_order_log (
        demo_code,
        trigger_name,
        note
    )
    VALUES (
        NEW.demo_code,
        'trg_bi_demo_precedes_base',
        'This trigger must run before base'
    );
END$$

DELIMITER ;
```

### 3.4. Tạo trigger chạy sau base

```sql
DELIMITER $$

CREATE TRIGGER trg_bi_demo_follows_base
BEFORE INSERT ON trigger_order_demo
FOR EACH ROW
FOLLOWS trg_bi_demo_base
BEGIN
    INSERT INTO trigger_order_log (
        demo_code,
        trigger_name,
        note
    )
    VALUES (
        NEW.demo_code,
        'trg_bi_demo_follows_base',
        'This trigger must run after base'
    );
END$$

DELIMITER ;
```

### 3.5. Test thứ tự

```sql
INSERT INTO trigger_order_demo (
    demo_code,
    amount
)
VALUES (
    'SEQ-001',
    100.00
);
```

```sql
SELECT execution_id,
       demo_code,
       trigger_name,
       note,
       fired_at
FROM trigger_order_log
WHERE demo_code = 'SEQ-001'
ORDER BY execution_id;
```

Kết quả mong đợi theo logic:

```text
trg_bi_demo_precedes_base
trg_bi_demo_base
trg_bi_demo_follows_base
```

### 3.6. Tại sao không log `NEW.demo_id` ở đây?

`demo_id` là auto-increment.

Trong BEFORE INSERT, giá trị `NEW.demo_id` có thể chưa là generated ID cuối cùng. Demo log dùng `demo_code` để quan sát thứ tự mà không phụ thuộc auto-increment value.

### Bài tập thực hành

**Bài 3.1.** Tạo hai demo tables.

**Bài 3.2.** Tạo `trg_bi_demo_base`.

**Bài 3.3.** Tạo trigger `PRECEDES trg_bi_demo_base`.

**Bài 3.4.** Tạo trigger `FOLLOWS trg_bi_demo_base`.

**Bài 3.5.** Insert `SEQ-001` và xác nhận thứ tự trong `trigger_order_log`.

---

## 4. Quản lý trigger metadata

### 4.1. `SHOW TRIGGERS`

```sql
SHOW TRIGGERS FROM trigger_lab;
```

Lọc theo subject table:

```sql
SHOW TRIGGERS FROM trigger_lab
LIKE 'sales';
```

> `LIKE` ở `SHOW TRIGGERS` lọc theo **table name**, không theo trigger name.

Lọc trigger name:

```sql
SHOW TRIGGERS FROM trigger_lab
WHERE `Trigger` LIKE 'trg_bi_demo%';
```

Lọc timing:

```sql
SHOW TRIGGERS FROM trigger_lab
WHERE `Timing` = 'BEFORE';
```

### 4.2. `SHOW CREATE TRIGGER`

```sql
SHOW CREATE TRIGGER trigger_lab.trg_bi_demo_base;
```

Dùng trước khi:

- Sửa trigger.
- Drop trigger.
- Tạo migration script.
- So sánh trigger giữa environments.
- Kiểm tra definer và SQL mode.

### 4.3. `INFORMATION_SCHEMA.TRIGGERS`

```sql
SELECT TRIGGER_NAME,
       EVENT_MANIPULATION,
       EVENT_OBJECT_SCHEMA,
       EVENT_OBJECT_TABLE,
       ACTION_TIMING,
       ACTION_ORDER,
       ACTION_STATEMENT,
       DEFINER,
       CREATED,
       SQL_MODE
FROM INFORMATION_SCHEMA.TRIGGERS
WHERE TRIGGER_SCHEMA = 'trigger_lab'
ORDER BY EVENT_OBJECT_TABLE,
         ACTION_TIMING,
         EVENT_MANIPULATION,
         ACTION_ORDER;
```

### 4.4. Metadata có thể giúp review gì?

| Metadata | Câu hỏi review |
|---|---|
| `TRIGGER_NAME` | Tên có thể hiện timing/event/purpose không? |
| `EVENT_OBJECT_TABLE` | Trigger gắn đúng subject table không? |
| `ACTION_TIMING` | BEFORE/AFTER có đúng logic không? |
| `ACTION_ORDER` | Multiple trigger order có rõ không? |
| `ACTION_STATEMENT` | Body có side effect không mong muốn không? |
| `DEFINER` | Account definer còn tồn tại/quyền còn phù hợp không? |
| `SQL_MODE` | Trigger được tạo dưới SQL mode nào? |

### Bài tập thực hành

**Bài 4.1.** Dùng `SHOW TRIGGERS FROM trigger_lab`.

**Bài 4.2.** Lọc trigger gắn với table `sales`.

**Bài 4.3.** Lọc trigger có tên bắt đầu bằng `trg_bi_demo`.

**Bài 4.4.** Dùng `SHOW CREATE TRIGGER` cho `trg_bi_demo_base`.

**Bài 4.5.** Viết query metadata có `ACTION_ORDER` cho mọi trigger trong lab.

---

## 5. Definer, privileges và security context

### 5.1. Trigger definer

Trigger có `DEFINER`.

Nếu không ghi rõ:

```sql
DEFINER = ...
```

definer mặc định là user chạy `CREATE TRIGGER`.

Khi trigger được kích hoạt, MySQL kiểm tra privileges theo definer context.

### 5.2. Quyền tạo trigger

Tạo trigger yêu cầu privilege:

```text
TRIGGER
```

trên subject table.

Ngoài ra, trigger definer cần các privileges cần thiết để thực hiện statements trong trigger body, ví dụ:

- `SELECT` nếu đọc `OLD`/`NEW` columns.
- `UPDATE` nếu dùng `SET NEW.column = ...`.
- `INSERT` lên audit table.
- `UPDATE` lên summary table.
- Các quyền khác theo SQL trong body.

### 5.3. `USER()` và `CURRENT_USER()`

Trong trigger:

```sql
SELECT USER();
```

gắn với client account gửi statement.

```sql
SELECT CURRENT_USER();
```

phản ánh account được dùng để kiểm tra trigger privileges, tức trigger definer context.

Vì vậy audit design cần xác định rõ muốn ghi:

- Database login của caller.
- Trigger definer.
- Application user/business user.
- Request identifier từ application.

### 5.4. Definer lifecycle

Nếu definer account bị drop hoặc mất required privileges, trigger có thể gặp lỗi khi được kích hoạt.

Vì vậy trigger deployment cần review:

- Definer có phải service account ổn định không?
- Account có privilege tối thiểu không?
- Account có bị dùng chung bừa bãi không?
- Backup/migration có preserve definer đúng không?

### Bài tập thực hành

**Bài 5.1.** Nêu privilege cần để tạo trigger.

**Bài 5.2.** Nêu quyền cần có nếu trigger insert vào audit table.

**Bài 5.3.** Giải thích khác nhau giữa `USER()` và `CURRENT_USER()` trong trigger.

**Bài 5.4.** Nêu một rủi ro khi trigger definer account bị xóa.

**Bài 5.5.** Nêu hai yêu cầu của least privilege cho trigger definer.

---

## 6. Các restrictions quan trọng

### 6.1. Không dùng trigger trên view hoặc temporary table

Trigger chỉ gắn với permanent base table.

Không tạo trigger trên:

```text
VIEW
TEMPORARY TABLE
INFORMATION_SCHEMA views
performance_schema views
```

### 6.2. Không dùng transaction control

Không dùng trong trigger:

```sql
START TRANSACTION;
COMMIT;
ROLLBACK;
```

Trigger tham gia transaction/statement của caller.

### 6.3. Không dùng dynamic SQL

Trigger không dùng:

```sql
PREPARE
EXECUTE
DEALLOCATE PREPARE
```

Do đó, trigger không phải nơi thích hợp để xây dynamic SQL theo string.

### 6.4. Không trả result set

Trigger không trả result set cho client.

Không dùng `SELECT` chỉ để hiển thị output như:

```sql
SELECT 'debug message';
```

Muốn debug lab, ghi vào log table hoặc dùng controlled test query sau DML.

### 6.5. Không modify table đã được invoking statement dùng

Một trigger không thể modify table đã đang được statement gây trigger sử dụng để đọc hoặc ghi.

Ví dụ trigger trên `sales` không thể tự `UPDATE sales ...` để cố sửa row sau update.

Điều này giúp tránh recursion/side effect không kiểm soát.

### 6.6. Foreign key actions và TRUNCATE

- Cascaded foreign-key actions không kích hoạt triggers.
- `TRUNCATE TABLE` không kích hoạt DELETE triggers.
- `DROP TABLE` cũng không kích hoạt DELETE trigger; trigger gắn với table bị drop cùng table.

### 6.7. `RETURN` không dùng trong trigger

Trigger không trả value nên không dùng `RETURN`.

Muốn thoát sớm từ block có label, có thể dùng `LEAVE` theo cú pháp stored program phù hợp.

### Bài tập thực hành

**Bài 6.1.** Nêu hai object không thể có trigger.

**Bài 6.2.** Nêu ba statement transaction control không được dùng trong trigger.

**Bài 6.3.** Giải thích vì sao dynamic SQL không dùng trong trigger.

**Bài 6.4.** Giải thích restriction “không modify table đã được invoking statement dùng”.

**Bài 6.5.** Nêu khác biệt giữa DELETE trigger activation với foreign-key cascade và TRUNCATE.

---

## 7. Error handling, SQL mode và behavior khi trigger fail

### 7.1. BEFORE và AFTER khi error

| Tình huống | Behavior khái quát |
|---|---|
| BEFORE trigger fail | Row operation tương ứng không được thực hiện |
| Row operation fail | AFTER trigger không chạy |
| AFTER trigger fail | Statement gây trigger fail |
| InnoDB statement fail | Thay đổi của statement thường rollback |
| Nontransactional table fail | Một số thay đổi trước lỗi có thể không rollback |

### 7.2. SQL mode được lưu khi trigger được tạo

MySQL lưu SQL mode đang có khi trigger được tạo và trigger body chạy theo mode đó.

Do đó:

- Thay đổi global/session SQL mode sau này không luôn làm trigger cũ đổi behavior.
- `SHOW TRIGGERS`/metadata có thể giúp review SQL mode.
- Migration nên tạo trigger dưới SQL mode có chủ đích và nhất quán.

### 7.3. Trigger cache và underlying metadata

MySQL docs nêu trigger cache không tự phát hiện mọi thay đổi metadata của underlying objects. Nếu trigger dùng table khác và table đó đổi metadata, cần kiểm tra/redeploy trigger theo change process thay vì giả định trigger tự thích nghi.

### Bài tập thực hành

**Bài 7.1.** Nêu điều gì xảy ra nếu BEFORE trigger phát sinh error.

**Bài 7.2.** Nêu điều gì xảy ra nếu AFTER trigger phát sinh error trên InnoDB table.

**Bài 7.3.** Nêu lý do SQL mode cần được review khi deploy trigger.

**Bài 7.4.** Giải thích vì sao thay đổi schema của table mà trigger dùng cần được kiểm tra trigger.

**Bài 7.5.** Nêu một cách test error path của trigger.

---

## 8. Best practices thiết kế trigger

### 8.1. Giữ trigger nhỏ và một mục đích

Tốt:

```text
trg_au_sales_audit
    → Ghi audit về sales update
```

Khó bảo trì:

```text
trg_sales_everything
    → Validate, pricing, inventory, notification, reporting,
      payroll, external workflow, batch summary...
```

### 8.2. Ưu tiên constraints cho integrity đơn giản

Dùng:

```text
NOT NULL
CHECK
UNIQUE
FOREIGN KEY
```

cho rules có thể diễn đạt declaratively.

Chỉ dùng trigger khi rule cần logic mà constraints không diễn đạt đủ.

### 8.3. Có migration/version-control

Không tạo trigger thủ công chỉ trong Workbench rồi quên lưu source.

Mỗi trigger nên có:

- File migration SQL.
- `DROP TRIGGER IF EXISTS` khi cần redeploy.
- `CREATE TRIGGER`.
- Test data/cases.
- Rollback/cleanup plan.
- Ghi chú definer/environment.

### 8.4. Test cả success và failure

Mỗi trigger cần test:

```text
- Trigger activation đúng event/timing.
- Data input hợp lệ.
- Input không hợp lệ.
- Multi-row DML.
- Transaction/error rollback.
- Side effects: audit, summary, archive.
- Permissions/definer nếu relevant.
```

### 8.5. Quan sát hiệu năng

Trigger chạy trên mỗi affected row.

Với DML lớn:

```sql
UPDATE sales
SET amount = amount * 1.02
WHERE status = 'DRAFT';
```

audit/summary trigger có thể tạo lượng lớn writes.

Cần đo:

- Rows affected.
- Trigger writes.
- Locking/concurrency.
- Transaction duration.
- Disk growth của audit table.
- Query latency.
- Retention/cleanup of audit tables.

### 8.6. Không dùng trigger để gửi external side effects trực tiếp

Trigger không nên là nơi gửi email, gọi HTTP API hoặc thực hiện workflow external trực tiếp.

Thay vào đó, có thể ghi event/outbox row vào table và để worker/application xử lý external side effect một cách retryable, observable và idempotent.

### Bài tập thực hành

**Bài 8.1.** Nêu một rule nên dùng CHECK thay vì trigger.

**Bài 8.2.** Nêu hai artifact nên có trong trigger migration.

**Bài 8.3.** Liệt kê năm test cases cho audit trigger.

**Bài 8.4.** Nêu ba performance metrics cần xem khi trigger chạy trên batch update.

**Bài 8.5.** Giải thích vì sao trigger không nên trực tiếp gửi email.

---

## 9. Bài tập tổng hợp

### Bài 9.1. Multiple trigger ordering

1. Tạo ba BEFORE INSERT trigger trên một demo table.
2. Một trigger dùng `PRECEDES`.
3. Một trigger dùng default order.
4. Một trigger dùng `FOLLOWS`.
5. Chứng minh thứ tự bằng log table.

### Bài 9.2. Trigger inventory report

Viết query từ `INFORMATION_SCHEMA.TRIGGERS` hiển thị:

1. Schema.
2. Table.
3. Trigger name.
4. Timing.
5. Event.
6. Action order.
7. Definer.
8. Created timestamp.
9. SQL mode.

Sắp xếp để review multiple triggers cùng table/event/timing.

### Bài 9.3. Trigger deployment review

Với trigger audit trên `sales`:

1. Xem `SHOW CREATE TRIGGER`.
2. Nêu source table và target tables.
3. Liệt kê privileges definer cần có.
4. Liệt kê failure cases.
5. Viết checklist trước release production.

### Bài 9.4. Restriction review

Cho mỗi yêu cầu, xác định hợp lệ hay không và giải thích:

1. Trigger trên view.
2. Trigger chạy `COMMIT`.
3. Trigger dùng `PREPARE`.
4. Trigger update chính subject table.
5. Trigger archive old row vào table khác.

### Bài 9.5. Audit capacity

Giả sử application update 100,000 sales rows mỗi ngày:

1. AFTER UPDATE audit trigger tạo bao nhiêu audit rows tối thiểu?
2. Nêu ba rủi ro storage/performance.
3. Nêu một retention strategy.
4. Nêu một indexing strategy cho audit table.
5. Nêu một cách kiểm tra audit trigger không tạo duplicate noise.

---

## 10. Đáp án gợi ý

### Bài 9.2

```sql
SELECT TRIGGER_SCHEMA,
       EVENT_OBJECT_TABLE AS subject_table,
       TRIGGER_NAME,
       ACTION_TIMING,
       EVENT_MANIPULATION,
       ACTION_ORDER,
       DEFINER,
       CREATED,
       SQL_MODE
FROM INFORMATION_SCHEMA.TRIGGERS
WHERE TRIGGER_SCHEMA = 'trigger_lab'
ORDER BY EVENT_OBJECT_TABLE,
         ACTION_TIMING,
         EVENT_MANIPULATION,
         ACTION_ORDER;
```

### Bài 9.4

| Yêu cầu | Hợp lệ? | Lý do |
|---|---:|---|
| Trigger trên view | Không | Trigger cần permanent table |
| Trigger chạy `COMMIT` | Không | Trigger không được begin/end transaction |
| Trigger dùng `PREPARE` | Không | Trigger không hỗ trợ dynamic SQL |
| Trigger update subject table | Không | Không được modify table đang được invoking statement dùng |
| Trigger archive row cũ vào table khác | Có thể | Hợp lệ nếu privileges, logic, transaction và workload phù hợp |

### Bài 9.5 — hướng trả lời

Nếu mọi update đều làm thay đổi business field và trigger chỉ log meaningful change, tối thiểu có thể là 100,000 audit rows/ngày. Cần xem disk growth, write amplification, index overhead, retention, partition/archive strategy và query pattern trên audit table. Có thể index theo `(sale_id, changed_at)` hoặc `(changed_at)` tùy nhu cầu tra cứu, sau khi kiểm tra workload.

---

## 11. Script cleanup

> **Cảnh báo:** Chỉ chạy script này khi chắc chắn `trigger_lab` chỉ dùng cho tutorial.

```sql
DROP DATABASE IF EXISTS trigger_lab;
```

Nếu chỉ muốn xóa demo multiple triggers mà giữ lab phần 1–2:

```sql
DROP TRIGGER IF EXISTS trigger_lab.trg_bi_demo_precedes_base;
DROP TRIGGER IF EXISTS trigger_lab.trg_bi_demo_base;
DROP TRIGGER IF EXISTS trigger_lab.trg_bi_demo_follows_base;

DROP TABLE IF EXISTS trigger_lab.trigger_order_log;
DROP TABLE IF EXISTS trigger_lab.trigger_order_demo;
```

---

## 12. Tóm tắt

- MySQL cho phép nhiều trigger cùng table/event/timing.
- Default order là creation order; dùng `PRECEDES`/`FOLLOWS` để đặt thứ tự rõ ràng.
- Dùng `SHOW TRIGGERS`, `SHOW CREATE TRIGGER`, `INFORMATION_SCHEMA.TRIGGERS` để review metadata.
- Trigger chạy theo definer security context; privilege và definer lifecycle cần được quản lý.
- Trigger không dùng được trên view/temporary table, không quản lý transaction, không dynamic SQL và không trả result set.
- Trigger không được modify table đang được invoking statement dùng.
- Foreign key cascades và TRUNCATE không kích hoạt trigger theo cách DELETE thông thường.
- Trigger nên nhỏ, có một mục đích, được version-control, benchmark và test cả success/failure paths.
- Với write volume lớn, audit/summary trigger cần được đánh giá về storage, concurrency và retention.

---

## 13. Từ khóa chính

- Multiple triggers
- Trigger order
- FOLLOWS
- PRECEDES
- ACTION_ORDER
- SHOW TRIGGERS
- SHOW CREATE TRIGGER
- INFORMATION_SCHEMA.TRIGGERS
- DEFINER
- TRIGGER privilege
- USER()
- CURRENT_USER()
- Dynamic SQL restriction
- Transaction restriction
- Trigger cache
- Foreign key cascade
- TRUNCATE TABLE
- Audit retention
- Outbox pattern
- Migration
