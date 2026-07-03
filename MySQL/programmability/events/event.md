---
title: "Tutorial: MySQL Events và Event Scheduler"
author: "Tên giảng viên"
duration: "180m"
difficulty: "Intermediate"
prerequisites:
  - "Đã biết INSERT, UPDATE, DELETE, CREATE TABLE và SELECT cơ bản"
  - "Đã biết stored procedure cơ bản, DELIMITER và BEGIN ... END"
  - "Có quyền EVENT trên schema lab; cần quyền quản trị phù hợp để bật/tắt Event Scheduler"
summary: "Thực hành MySQL Event Scheduler: kiểm tra và bật scheduler, CREATE EVENT, ALTER EVENT, SHOW EVENTS, INFORMATION_SCHEMA.EVENTS, DROP EVENT, event một lần và event định kỳ trong schema event_lab."
---

# Tutorial: MySQL Events và Event Scheduler

## Link tham khảo

### MySQL Tutorial

- [MySQL Events](https://www.mysqltutorial.org/mysql-events/)
- [Create Event](https://www.mysqltutorial.org/mysql-events/mysql-create-event/)
- [Alter Event](https://www.mysqltutorial.org/mysql-events/mysql-alter-event/)
- [Drop Event](https://www.mysqltutorial.org/mysql-events/mysql-drop-event/)
- [Show Events](https://www.mysqltutorial.org/mysql-events/mysql-show-events/)

### MySQL Reference Manual

- [Event Scheduler Overview](https://dev.mysql.com/doc/refman/8.4/en/events-overview.html)
- [Event Scheduler Configuration](https://dev.mysql.com/doc/refman/8.4/en/events-configuration.html)
- [CREATE EVENT Statement](https://dev.mysql.com/doc/refman/8.4/en/create-event.html)
- [ALTER EVENT Statement](https://dev.mysql.com/doc/refman/8.4/en/alter-event.html)
- [DROP EVENT Statement](https://dev.mysql.com/doc/refman/8.4/en/drop-event.html)
- [SHOW EVENTS Statement](https://dev.mysql.com/doc/refman/8.4/en/show-events.html)
- [SHOW CREATE EVENT Statement](https://dev.mysql.com/doc/refman/8.4/en/show-create-event.html)
- [INFORMATION_SCHEMA.EVENTS Table](https://dev.mysql.com/doc/refman/8.4/en/information-schema-events-table.html)
- [Event Scheduler and MySQL Privileges](https://dev.mysql.com/doc/refman/8.4/en/events-privileges.html)

> **Phạm vi lab:** Event là tác vụ tự chạy theo thời gian. Script tạo event trong bài này chỉ dùng schema `event_lab`; không tạo event trên production database hoặc database dùng chung. Chỉ bật Event Scheduler trên máy cá nhân/lab khi hiểu rõ những event nào đang tồn tại và có thể được chạy.

---

## 1. Mục tiêu học tập

Sau khi hoàn thành tutorial này, người học có thể:

1. Giải thích MySQL Event là gì và phân biệt event với trigger, procedure và cron/Task Scheduler.
2. Kiểm tra trạng thái `event_scheduler`.
3. Bật hoặc tắt Event Scheduler trong môi trường lab khi có quyền phù hợp.
4. Tạo one-time event bằng `CREATE EVENT ... ON SCHEDULE AT`.
5. Tạo recurring event bằng `CREATE EVENT ... ON SCHEDULE EVERY`.
6. Dùng `ON COMPLETION PRESERVE` để giữ one-time event sau khi nó chạy.
7. Dùng `ENABLE`, `DISABLE`, `DISABLE ON REPLICA`, `STARTS`, `ENDS`, `COMMENT` trong event definition.
8. Sửa schedule hoặc trạng thái event bằng `ALTER EVENT`.
9. Liệt kê và kiểm tra event bằng `SHOW EVENTS`, `SHOW CREATE EVENT` và `INFORMATION_SCHEMA.EVENTS`.
10. Xóa event bằng `DROP EVENT`.
11. Nhận biết các rủi ro về scheduler, definer, time zone, replication, audit và cleanup.
12. Thiết kế event theo nguyên tắc an toàn, idempotent và có thể kiểm thử.

---

## 2. MySQL Event là gì?

MySQL Event là một database object có tên, chứa một hoặc nhiều câu SQL được MySQL Event Scheduler thực thi theo một lịch định trước.

Có thể hình dung event như:

```text
cron job trong Unix/Linux
Task Scheduler trong Windows
scheduled job chạy bên trong MySQL Server
```

Ví dụ event có thể:

- Xóa dữ liệu hết hạn mỗi ngày.
- Cập nhật bảng summary vào đầu ngày.
- Archive dữ liệu cũ theo lịch.
- Đánh dấu row quá hạn.
- Chạy stored procedure bảo trì theo lịch.
- Ghi log định kỳ.

### 2.1. Event Scheduler

Event Scheduler là thành phần server quản lý và thực thi scheduled events.

Một event được tạo **không tự chạy** nếu Event Scheduler không hoạt động.

Kiểm tra:

```sql
SHOW VARIABLES LIKE 'event_scheduler';
```

Kết quả thường là một trong các giá trị:

| Giá trị | Ý nghĩa |
|---|---|
| `ON` | Scheduler đang chạy và có thể thực thi event enabled |
| `OFF` | Scheduler không chạy; event không được thực thi |
| `DISABLED` | Scheduler bị tắt ở mức server startup option và không thể bật lúc runtime |

### 2.2. Event khác trigger thế nào?

| Tiêu chí | Event | Trigger |
|---|---|---|
| Khi chạy | Theo thời gian/lịch | Khi `INSERT`, `UPDATE`, `DELETE` |
| Cần Event Scheduler | Có | Không |
| Subject object | Database event | Table |
| Kích hoạt bởi | Thời điểm hoặc interval | DML trên table |
| Ví dụ | Xóa log cũ mỗi ngày | Audit mỗi row update |
| Tần suất | Một lần hoặc định kỳ | Mỗi row DML |

### 2.3. Event khác procedure thế nào?

| Tiêu chí | Event | Stored Procedure |
|---|---|---|
| Có schedule | Có | Không |
| Cách chạy | Scheduler tự gọi | Client/application gọi `CALL` |
| Có thể gọi procedure? | Có | Có thể được gọi bởi event |
| Mục đích | Automation theo thời gian | Đóng gói logic có thể gọi lại |

Một event có thể gọi procedure:

```sql
DO CALL some_procedure();
```

hoặc trực tiếp thực thi statement/body `BEGIN ... END`.

### 2.4. Khi nào event phù hợp?

Event thường phù hợp cho task thuần SQL, chạy định kỳ, gần dữ liệu và không cần external side effects trực tiếp.

Ví dụ:

```text
DELETE expired session rows every day
UPDATE invoice status when overdue
refresh small reporting summary each night
archive rows older than a retention threshold
```

Event không phải lựa chọn đầu tiên khi cần:

- Gọi API ngoài hoặc gửi email trực tiếp.
- Workflow cần retry/backoff phức tạp.
- Distributed scheduling qua nhiều service.
- Observability phong phú và orchestration đa hệ thống.
- Business logic lớn cần code review/test ở application layer.

Trong các trường hợp đó, có thể dùng job scheduler của application, workflow platform hoặc worker queue; database event chỉ ghi một outbox row nếu cần.

### Bài tập thực hành

**Bài 2.1.** Giải thích MySQL Event là gì bằng một hoặc hai câu.

**Bài 2.2.** Nêu ba ví dụ task phù hợp với event.

**Bài 2.3.** Phân biệt event với trigger.

**Bài 2.4.** Phân biệt event với stored procedure.

**Bài 2.5.** Nêu một tình huống không nên dùng database event.

---

## 3. Quyền, scheduler status và an toàn trước khi thực hành

### 3.1. Quyền `EVENT`

Tạo, sửa hoặc xóa event trong một schema cần privilege:

```text
EVENT
```

Kiểm tra quyền của account hiện tại:

```sql
SHOW GRANTS;
```

Ví dụ cấp quyền trong lab bởi administrator:

```sql
GRANT EVENT
ON event_lab.*
TO 'lab_user'@'localhost';
```

Người học không nên tự chạy `GRANT` nếu không có quyền quản trị hoặc chưa được cấp phép.

### 3.2. Kiểm tra scheduler

```sql
SHOW VARIABLES LIKE 'event_scheduler';
```

Hoặc:

```sql
SELECT @@event_scheduler AS eventSchedulerStatus;
```

### 3.3. Bật scheduler trong lab

Nếu status là `OFF` và account có quyền thay đổi global system variable:

```sql
SET GLOBAL event_scheduler = ON;
```

Kiểm tra lại:

```sql
SHOW VARIABLES LIKE 'event_scheduler';
```

Tắt scheduler trong lab:

```sql
SET GLOBAL event_scheduler = OFF;
```

> `SET GLOBAL` ảnh hưởng server instance, không chỉ session hiện tại. Trên server có nhiều database hoặc nhiều ứng dụng, việc bật scheduler có thể làm **mọi event enabled** trên server bắt đầu chạy. Vì vậy không bật/tắt tùy tiện trên shared server hoặc production.

### 3.4. Scheduler bị `DISABLED`

Nếu giá trị là:

```text
DISABLED
```

thì server được khởi động với scheduler disabled theo configuration/startup option. `SET GLOBAL event_scheduler = ON` sẽ không giải quyết được.

Người quản trị cần xem server configuration, thường là option file của `mysqld`, ví dụ:

```ini
[mysqld]
event_scheduler=ON
```

Sau đó restart theo quy trình vận hành phù hợp.

### 3.5. Kiểm tra scheduler thread

Khi scheduler đang hoạt động, có thể quan sát process list:

```sql
SHOW PROCESSLIST;
```

Hoặc:

```sql
SELECT ID,
       USER,
       HOST,
       DB,
       COMMAND,
       TIME,
       STATE,
       INFO
FROM INFORMATION_SCHEMA.PROCESSLIST
WHERE USER = 'event_scheduler';
```

Không phải mọi account đều có quyền xem thread của user khác. Account có privilege phù hợp như `PROCESS` mới thấy toàn bộ process list.

### Bài tập thực hành

**Bài 3.1.** Chạy `SHOW GRANTS;`.

**Bài 3.2.** Kiểm tra `event_scheduler`.

**Bài 3.3.** Nêu điều gì xảy ra nếu event được tạo nhưng `event_scheduler = OFF`.

**Bài 3.4.** Nêu rủi ro của `SET GLOBAL event_scheduler = ON` trên shared server.

**Bài 3.5.** Giải thích khác nhau giữa `OFF` và `DISABLED`.

---

## 4. Chuẩn bị schema `event_lab`

> **Cảnh báo:** Script sau xóa toàn bộ schema `event_lab` nếu schema đã tồn tại.

```sql
DROP DATABASE IF EXISTS event_lab;

CREATE DATABASE event_lab
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_0900_ai_ci;

USE event_lab;
```

### 4.1. Bảng log để quan sát event

```sql
CREATE TABLE event_run_log (
    run_id BIGINT NOT NULL AUTO_INCREMENT,
    event_name VARCHAR(100) NOT NULL,
    executed_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    note VARCHAR(255) NOT NULL,
    scheduler_user VARCHAR(288) NULL,
    PRIMARY KEY (run_id),
    INDEX idx_event_run_log_event_time (
        event_name,
        executed_at
    )
) ENGINE = InnoDB;
```

### 4.2. Bảng notification có dữ liệu hết hạn

```sql
CREATE TABLE notifications (
    notification_id BIGINT NOT NULL AUTO_INCREMENT,
    recipient_code VARCHAR(30) NOT NULL,
    message_text VARCHAR(255) NOT NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    expires_at DATETIME NOT NULL,
    PRIMARY KEY (notification_id),
    INDEX idx_notifications_expires_at (expires_at)
) ENGINE = InnoDB;
```

Dữ liệu test:

```sql
INSERT INTO notifications (
    recipient_code,
    message_text,
    expires_at
)
VALUES
    (
        'U001',
        'Expired notification for cleanup demo',
        CURRENT_TIMESTAMP - INTERVAL 1 DAY
    ),
    (
        'U002',
        'Future notification for cleanup demo',
        CURRENT_TIMESTAMP + INTERVAL 7 DAY
    );
```

### 4.3. Bảng summary

```sql
CREATE TABLE notification_summary (
    summary_date DATE NOT NULL,
    active_notifications INT NOT NULL DEFAULT 0,
    refreshed_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (summary_date)
) ENGINE = InnoDB;
```

### 4.4. Kiểm tra dữ liệu

```sql
SELECT *
FROM notifications
ORDER BY notification_id;
```

```sql
SHOW CREATE TABLE event_run_log;
SHOW CREATE TABLE notifications;
SHOW CREATE TABLE notification_summary;
```

### Bài tập thực hành

**Bài 4.1.** Tạo schema `event_lab`.

**Bài 4.2.** Tạo table `event_run_log`.

**Bài 4.3.** Tạo table `notifications`.

**Bài 4.4.** Insert một notification đã hết hạn và một notification còn hiệu lực.

**Bài 4.5.** Kiểm tra index của `notifications` bằng `SHOW INDEX`.

---

## 5. Tạo one-time event bằng `CREATE EVENT`

### 5.1. Cú pháp cơ bản

```sql
CREATE EVENT event_name
ON SCHEDULE AT timestamp
DO event_body;
```

Ví dụ event chạy một lần sau 2 phút:

```sql
CREATE EVENT ev_lab_one_time_log
ON SCHEDULE AT CURRENT_TIMESTAMP + INTERVAL 2 MINUTE
ON COMPLETION PRESERVE
DISABLE
DO
    INSERT INTO event_run_log (
        event_name,
        note,
        scheduler_user
    )
    VALUES (
        'ev_lab_one_time_log',
        'One-time event executed',
        CURRENT_USER()
    );
```

### 5.2. Vì sao event được tạo với `DISABLE`?

Trong lab, event được tạo disabled để tránh tự chạy trước khi sinh viên kiểm tra definition và scheduler.

Kiểm tra event:

```sql
SHOW EVENTS FROM event_lab;
```

Khi sẵn sàng chạy, đặt lại lịch trong tương lai gần và enable:

```sql
ALTER EVENT ev_lab_one_time_log
ON SCHEDULE AT CURRENT_TIMESTAMP + INTERVAL 1 MINUTE
ENABLE;
```

### 5.3. `ON COMPLETION PRESERVE`

One-time event có thể bị drop sau khi chạy nếu không được preserve.

```sql
ON COMPLETION PRESERVE
```

giữ event definition sau khi event hoàn thành, giúp:

- Kiểm tra metadata.
- Xem `LAST_EXECUTED`.
- Audit/demo.
- Sửa và chạy lại sau đó.

Nếu dùng:

```sql
ON COMPLETION NOT PRESERVE
```

event hết hạn có thể bị server tự drop sau khi hoàn thành.

### 5.4. Kiểm tra kết quả sau khi event chạy

Đợi ít nhất thời gian đã schedule, sau đó chạy:

```sql
SELECT run_id,
       event_name,
       executed_at,
       note,
       scheduler_user
FROM event_run_log
ORDER BY run_id;
```

Kiểm tra event metadata:

```sql
SHOW EVENTS FROM event_lab
WHERE Name = 'ev_lab_one_time_log';
```

### 5.5. Sự khác nhau giữa one-time event và statement chạy trực tiếp

Nếu chạy:

```sql
INSERT INTO event_run_log (...);
```

statement chạy ngay.

Nếu tạo event:

```sql
CREATE EVENT ...
ON SCHEDULE AT ...
DO INSERT ...
```

statement được scheduler thực thi tại thời điểm schedule, với event definer context.

### Bài tập thực hành

**Bài 5.1.** Tạo `ev_lab_one_time_log` disabled.

**Bài 5.2.** Dùng `SHOW EVENTS` kiểm tra event vừa tạo.

**Bài 5.3.** Dùng `ALTER EVENT` để schedule event chạy sau 1 phút và enable.

**Bài 5.4.** Sau khi event chạy, kiểm tra `event_run_log`.

**Bài 5.5.** Giải thích vai trò của `ON COMPLETION PRESERVE`.

---

## 6. Tạo recurring event bằng `EVERY`

### 6.1. Cú pháp

```sql
CREATE EVENT event_name
ON SCHEDULE EVERY interval
[STARTS timestamp]
[ENDS timestamp]
DO event_body;
```

Ví dụ event cleanup notifications hết hạn:

```sql
CREATE EVENT ev_lab_cleanup_expired_notifications
ON SCHEDULE EVERY 1 DAY
STARTS CURRENT_TIMESTAMP + INTERVAL 1 DAY
ON COMPLETION PRESERVE
DISABLE
DO
    DELETE FROM notifications
    WHERE expires_at < CURRENT_TIMESTAMP;
```

Event này:

- Chạy mỗi ngày.
- Lần đầu chạy sau ít nhất một ngày.
- Ban đầu disabled trong lab.
- Nếu enabled, xóa notification hết hạn.

### 6.2. Test nhanh với interval ngắn

Chỉ trên lab, có thể dùng interval 1 minute:

```sql
CREATE EVENT ev_lab_heartbeat
ON SCHEDULE EVERY 1 MINUTE
STARTS CURRENT_TIMESTAMP + INTERVAL 1 MINUTE
ENDS CURRENT_TIMESTAMP + INTERVAL 5 MINUTE
ON COMPLETION PRESERVE
DISABLE
DO
    INSERT INTO event_run_log (
        event_name,
        note,
        scheduler_user
    )
    VALUES (
        'ev_lab_heartbeat',
        'Recurring heartbeat executed',
        CURRENT_USER()
    );
```

Khi sẵn sàng:

```sql
ALTER EVENT ev_lab_heartbeat
ENABLE;
```

Sau khoảng vài phút:

```sql
SELECT event_name,
       COUNT(*) AS totalRuns,
       MIN(executed_at) AS firstRun,
       MAX(executed_at) AS lastRun
FROM event_run_log
WHERE event_name = 'ev_lab_heartbeat'
GROUP BY event_name;
```

Sau demo, disable hoặc drop event:

```sql
ALTER EVENT ev_lab_heartbeat DISABLE;
```

hoặc:

```sql
DROP EVENT IF EXISTS ev_lab_heartbeat;
```

### 6.3. `STARTS` và `ENDS`

| Clause | Ý nghĩa |
|---|---|
| `STARTS` | Thời điểm event bắt đầu được chạy |
| `ENDS` | Thời điểm event ngừng chạy |
| Không có `ENDS` | Event định kỳ có thể chạy vô thời hạn nếu enabled |

Không tạo recurring event không có `ENDS` trong lab chỉ để thử cú pháp rồi quên dọn dẹp.

### 6.4. Event body nhiều statements

Nếu event có nhiều câu lệnh:

```sql
DELIMITER $$

CREATE EVENT event_name
ON SCHEDULE EVERY 1 DAY
DO
BEGIN
    statement_1;
    statement_2;
END$$

DELIMITER ;
```

Ví dụ summary update:

```sql
DELIMITER $$

CREATE EVENT ev_lab_refresh_notification_summary
ON SCHEDULE EVERY 1 DAY
STARTS CURRENT_TIMESTAMP + INTERVAL 1 DAY
ON COMPLETION PRESERVE
DISABLE
DO
BEGIN
    INSERT INTO notification_summary (
        summary_date,
        active_notifications,
        refreshed_at
    )
    SELECT CURRENT_DATE(),
           COUNT(*),
           CURRENT_TIMESTAMP
    FROM notifications
    WHERE expires_at >= CURRENT_TIMESTAMP
    ON DUPLICATE KEY UPDATE
        active_notifications = VALUES(active_notifications),
        refreshed_at = VALUES(refreshed_at);

    INSERT INTO event_run_log (
        event_name,
        note,
        scheduler_user
    )
    VALUES (
        'ev_lab_refresh_notification_summary',
        'Notification summary refreshed',
        CURRENT_USER()
    );
END$$

DELIMITER ;
```

> `VALUES(column_name)` in `ON DUPLICATE KEY UPDATE` remains supported in many MySQL versions but may generate deprecation warnings in newer versions. For production code targeting current MySQL, review version-specific syntax and consider aliases for `INSERT ... VALUES` where appropriate. The lab keeps this form because it is widely recognizable.

### Bài tập thực hành

**Bài 6.1.** Tạo event `ev_lab_cleanup_expired_notifications` disabled.

**Bài 6.2.** Tạo event `ev_lab_heartbeat` chạy mỗi 1 phút trong tối đa 5 phút.

**Bài 6.3.** Enable heartbeat event trong lab và kiểm tra số lần chạy.

**Bài 6.4.** Disable heartbeat event sau demo.

**Bài 6.5.** Tạo event multi-statement refresh `notification_summary`.

---

## 7. `ALTER EVENT`

### 7.1. Khi nào dùng `ALTER EVENT`?

`ALTER EVENT` dùng để sửa definition của event đã tồn tại, ví dụ:

- Thay đổi schedule.
- Đổi trạng thái enable/disable.
- Đổi `ON COMPLETION`.
- Đổi comment.
- Đổi event body.
- Đổi tên event.
- Đổi definer trong bối cảnh quản trị phù hợp.

MySQL không cần drop + create cho mọi thay đổi event; dùng `ALTER EVENT` khi câu lệnh hỗ trợ phần cần sửa.

### 7.2. Đổi schedule

Ví dụ đổi cleanup từ mỗi ngày sang mỗi giờ trong lab:

```sql
ALTER EVENT ev_lab_cleanup_expired_notifications
ON SCHEDULE EVERY 1 HOUR
STARTS CURRENT_TIMESTAMP + INTERVAL 1 HOUR;
```

Trên production, không đổi từ daily sang hourly chỉ để thử. Cần đánh giá workload, row volume, locking và retention policy.

### 7.3. Enable và disable event

Disable:

```sql
ALTER EVENT ev_lab_cleanup_expired_notifications
DISABLE;
```

Enable:

```sql
ALTER EVENT ev_lab_cleanup_expired_notifications
ENABLE;
```

`DISABLE ON REPLICA` có thể được dùng trong replication scenarios để event không chạy tại replica-side server theo semantics MySQL version đang dùng:

```sql
ALTER EVENT ev_lab_cleanup_expired_notifications
DISABLE ON REPLICA;
```

Dùng option này chỉ khi hiểu replication topology và version terminology của server.

### 7.4. Đổi completion behavior và comment

```sql
ALTER EVENT ev_lab_one_time_log
ON COMPLETION PRESERVE
COMMENT 'One-time lab event retained for inspection';
```

### 7.5. Đổi event body

Ví dụ sửa note:

```sql
ALTER EVENT ev_lab_one_time_log
DO
    INSERT INTO event_run_log (
        event_name,
        note,
        scheduler_user
    )
    VALUES (
        'ev_lab_one_time_log',
        'One-time event executed after alteration',
        CURRENT_USER()
    );
```

### 7.6. Đổi tên event

```sql
ALTER EVENT ev_lab_one_time_log
RENAME TO ev_lab_one_time_audit_log;
```

Sau rename, dùng tên mới trong `SHOW CREATE EVENT`, `ALTER EVENT` và `DROP EVENT`.

### 7.7. Alter không thay đổi mọi property nếu không nêu lại

`ALTER EVENT` chỉ cập nhật các options được chỉ rõ; các property khác vẫn giữ nguyên.

Tuy vậy, trước khi alter production event, luôn chạy:

```sql
SHOW CREATE EVENT schema_name.event_name;
```

để biết definition hiện có.

### Bài tập thực hành

**Bài 7.1.** Disable `ev_lab_cleanup_expired_notifications`.

**Bài 7.2.** Đổi schedule của event đó sang every 1 hour trong lab.

**Bài 7.3.** Đặt comment mô tả cho event.

**Bài 7.4.** Sửa body của `ev_lab_one_time_log`.

**Bài 7.5.** Rename `ev_lab_one_time_log` thành `ev_lab_one_time_audit_log`.

---

## 8. Liệt kê và kiểm tra event

### 8.1. `SHOW EVENTS`

Liệt kê event trong database hiện tại:

```sql
USE event_lab;

SHOW EVENTS;
```

Liệt kê event trong database chỉ rõ:

```sql
SHOW EVENTS FROM event_lab;
```

Lọc tên event:

```sql
SHOW EVENTS FROM event_lab
LIKE 'ev_lab%';
```

Hoặc:

```sql
SHOW EVENTS FROM event_lab
WHERE Name LIKE 'ev_lab%';
```

### 8.2. Các cột thường gặp

| Cột | Ý nghĩa |
|---|---|
| `Db` | Schema chứa event |
| `Name` | Tên event |
| `Definer` | Account tạo/định nghĩa event |
| `Time zone` | Time zone được event dùng |
| `Type` | `ONE TIME` hoặc `RECURRING` |
| `Execute at` | Thời điểm chạy với one-time event |
| `Interval value` / `Interval field` | Khoảng thời gian recurring |
| `Starts` / `Ends` | Giới hạn lịch recurring |
| `Status` | `ENABLED`, `DISABLED`, hoặc trạng thái replica-side |
| `Last executed` | Lần chạy gần nhất |
| `Event body` | Loại body, thường `SQL` |
| `Comment` | Mô tả event |

### 8.3. `SHOW CREATE EVENT`

```sql
SHOW CREATE EVENT event_lab.ev_lab_cleanup_expired_notifications;
```

Lệnh này rất quan trọng trước khi:

- Alter event.
- Drop event.
- Di chuyển event qua môi trường khác.
- Backup/review DDL.
- Kiểm tra `DEFINER`, schedule, status hoặc body.

### 8.4. `INFORMATION_SCHEMA.EVENTS`

```sql
SELECT EVENT_SCHEMA,
       EVENT_NAME,
       DEFINER,
       TIME_ZONE,
       EVENT_TYPE,
       EXECUTE_AT,
       INTERVAL_VALUE,
       INTERVAL_FIELD,
       STARTS,
       ENDS,
       STATUS,
       LAST_EXECUTED,
       ON_COMPLETION,
       EVENT_COMMENT,
       CREATED,
       LAST_ALTERED
FROM INFORMATION_SCHEMA.EVENTS
WHERE EVENT_SCHEMA = 'event_lab'
ORDER BY EVENT_NAME;
```

Lọc enabled events:

```sql
SELECT EVENT_NAME,
       EVENT_TYPE,
       STATUS,
       LAST_EXECUTED,
       STARTS,
       ENDS
FROM INFORMATION_SCHEMA.EVENTS
WHERE EVENT_SCHEMA = 'event_lab'
  AND STATUS = 'ENABLED'
ORDER BY EVENT_NAME;
```

### 8.5. Metadata và quyền

Khả năng xem metadata event phụ thuộc vào quyền. Nếu không thấy event mong đợi:

- Kiểm tra schema name.
- Kiểm tra account hiện tại.
- Kiểm tra `SHOW GRANTS`.
- Xác nhận event có bị drop hoặc rename không.
- Hỏi DBA về metadata visibility nếu cần.

### Bài tập thực hành

**Bài 8.1.** Dùng `SHOW EVENTS FROM event_lab`.

**Bài 8.2.** Lọc các event có tên bắt đầu bằng `ev_lab`.

**Bài 8.3.** Dùng `SHOW CREATE EVENT` xem definition event cleanup.

**Bài 8.4.** Viết query `INFORMATION_SCHEMA.EVENTS` chỉ liệt kê enabled events.

**Bài 8.5.** Giải thích ý nghĩa của `LAST_EXECUTED` và `STATUS`.

---

## 9. Xóa event bằng `DROP EVENT`

### 9.1. Cú pháp

```sql
DROP EVENT event_name;
```

Ví dụ:

```sql
DROP EVENT event_lab.ev_lab_heartbeat;
```

### 9.2. Dùng `IF EXISTS`

```sql
DROP EVENT IF EXISTS event_lab.ev_lab_heartbeat;
```

Dùng `IF EXISTS` giúp cleanup script chạy lại an toàn hơn khi event đã bị xóa.

### 9.3. Xóa nhiều event

```sql
DROP EVENT IF EXISTS event_lab.ev_lab_one_time_audit_log;
DROP EVENT IF EXISTS event_lab.ev_lab_cleanup_expired_notifications;
DROP EVENT IF EXISTS event_lab.ev_lab_refresh_notification_summary;
```

### 9.4. Những gì `DROP EVENT` không làm

`DROP EVENT` chỉ xóa event definition. Nó không:

- Xóa dữ liệu event đã tạo trước đó.
- Hoàn tác `DELETE`, `UPDATE`, `INSERT` mà event đã thực hiện.
- Xóa table được event sử dụng.
- Khôi phục dữ liệu đã bị cleanup.

Vì vậy, trước khi drop production event cần:

1. Xem `SHOW CREATE EVENT`.
2. Xác nhận impact.
3. Lưu migration/DDL.
4. Kiểm tra dependency.
5. Có rollback plan nếu cần tạo lại.

### 9.5. Event và `DROP DATABASE`

Nếu drop schema chứa event:

```sql
DROP DATABASE event_lab;
```

thì events trong schema đó bị drop theo schema.

Đây là lý do tutorial đặt mọi event vào schema lab riêng.

### Bài tập thực hành

**Bài 9.1.** Drop `ev_lab_heartbeat` bằng `DROP EVENT IF EXISTS`.

**Bài 9.2.** Dùng `SHOW EVENTS` xác nhận event không còn tồn tại.

**Bài 9.3.** Giải thích vì sao `DROP EVENT` không hoàn tác data changes cũ.

**Bài 9.4.** Nêu bốn kiểm tra trước khi drop event production.

**Bài 9.5.** Giải thích điều gì xảy ra với event khi drop schema chứa nó.

---

## 10. Event definer, time zone và replication

### 10.1. Definer

Event có `DEFINER`, thường là account đã chạy `CREATE EVENT` nếu không chỉ rõ.

Khi event chạy, privileges cho statements trong event body được đánh giá theo definer context.

Do đó, event deployment cần review:

- Definer account có tồn tại không?
- Definer có privilege tối thiểu nhưng đủ không?
- Definer có phải personal account dễ bị xóa không?
- Event body có thao tác table/object nào yêu cầu thêm quyền không?
- Backup/restore có giữ definer phù hợp không?

### 10.2. Time zone

Event có `TIME_ZONE` metadata.

Lịch chạy có thể chịu ảnh hưởng bởi:

- Time zone của server/session tại thời điểm tạo hoặc alter.
- Time zone khai báo cho event.
- Daylight saving time nếu dùng time zone khu vực có DST.
- Sự khác nhau giữa UTC và local business time.

Trong các hệ thống đa vùng hoặc có DST, cần thống nhất policy:

```text
Store and schedule in UTC
or
Use an explicit named time zone with tested DST behavior
```

Không chỉ giả định `CURRENT_TIMESTAMP` luôn nghĩa cùng một local time trên mọi server.

### 10.3. Replication

Trong replication topology, scheduled event có thể được replicated nhưng không nên mặc định cho event chạy ở cả source và replica.

Cần xem:

- Event status trên replica.
- `DISABLE ON REPLICA` / terminology của version đang dùng.
- Event scheduler có đang ON ở replica không.
- Business task có được phép chạy nhiều lần không.
- Idempotency của event body.

Không tự enable replicated events trên replica khi chưa hiểu topology.

### 10.4. Idempotency

Event body nên an toàn khi retry hoặc chạy lại theo cách có kiểm soát.

Ví dụ idempotent update:

```sql
UPDATE notifications
SET message_text = message_text
WHERE expires_at < CURRENT_TIMESTAMP;
```

Ví dụ tốt hơn cho cleanup:

```sql
DELETE FROM notifications
WHERE expires_at < CURRENT_TIMESTAMP;
```

Nếu chạy lại sau khi row đã bị xóa, statement không tạo duplicate side effect.

Ngược lại, event insert log không có unique guard có thể tạo duplicate rows nếu event bị chạy nhiều lần trong test/recovery scenario.

### Bài tập thực hành

**Bài 10.1.** Nêu hai lý do không nên dùng personal account làm event definer trong production.

**Bài 10.2.** Nêu hai vấn đề time zone có thể ảnh hưởng scheduled event.

**Bài 10.3.** Giải thích vì sao event không nên tự động chạy ở cả source và replica.

**Bài 10.4.** Giải thích idempotency.

**Bài 10.5.** Nêu một event cleanup có tính idempotent.

---

## 11. Best practices

### 11.1. Event phải có một mục đích rõ ràng

Tốt:

```text
ev_cleanup_expired_notifications
ev_refresh_daily_summary
ev_archive_old_audit_rows
```

Kém rõ ràng:

```text
ev_job1
ev_update_all
ev_misc_task
```

### 11.2. Giữ body ngắn, hoặc gọi stored procedure có kiểm soát

Nếu task có nhiều bước, body dài hoặc cần test riêng, cân nhắc:

```sql
CREATE EVENT ev_refresh_report
ON SCHEDULE EVERY 1 DAY
DO CALL sp_refresh_report_summary();
```

Stored procedure có thể được test bằng `CALL` trước khi schedule event.

### 11.3. Không thực hiện destructive task mà thiếu guard

Không viết event kiểu:

```sql
DELETE FROM logs;
```

Không có điều kiện.

Thay vào đó, có retention threshold rõ:

```sql
DELETE FROM logs
WHERE created_at < CURRENT_TIMESTAMP - INTERVAL 90 DAY;
```

Và cần:

- Backup/retention policy.
- Chỉ mục phù hợp trên cột thời gian.
- Batch delete nếu table lớn.
- Monitoring số rows affected.
- Test trên subset/lab.

### 11.4. Ghi log hoặc metric theo mức phù hợp

Với task quan trọng, cần có cách biết:

- Event có chạy không?
- Lần chạy cuối khi nào?
- Có bao nhiêu rows ảnh hưởng?
- Có lỗi không?
- Job có chạy lâu bất thường không?

Có thể dùng:

- `INFORMATION_SCHEMA.EVENTS.LAST_EXECUTED`.
- Event run log table.
- Error log.
- Monitoring/query metrics.
- Application/DBA alerting.

Không log quá mức cho high-frequency event, vì log table có thể trở thành write overhead.

### 11.5. Test manual trước schedule

Với body dự định chạy:

```sql
DELETE FROM notifications
WHERE expires_at < CURRENT_TIMESTAMP;
```

Hãy chạy hoặc mô phỏng trong transaction/lab trước để hiểu impact.

Sau khi đúng, tạo event disabled, review `SHOW CREATE EVENT`, rồi enable theo thời điểm đã chọn.

### 11.6. Không quên cleanup

Recurring event test phải được:

```sql
ALTER EVENT ... DISABLE;
```

hoặc:

```sql
DROP EVENT ...;
```

sau demo.

### Bài tập thực hành

**Bài 11.1.** Đặt tên rõ nghĩa cho event cleanup log sau 90 ngày.

**Bài 11.2.** Viết event body gọi procedure `sp_refresh_daily_summary()`.

**Bài 11.3.** Viết điều kiện delete audit rows cũ hơn 180 ngày.

**Bài 11.4.** Nêu bốn chỉ số/metadata cần theo dõi cho event quan trọng.

**Bài 11.5.** Viết hai lệnh cleanup để ngăn heartbeat event chạy tiếp sau lab.

---

## 12. Các lỗi thường gặp

### Lỗi 1. Tạo event nhưng scheduler OFF

Event tồn tại:

```sql
SHOW EVENTS FROM event_lab;
```

nhưng không chạy.

Kiểm tra:

```sql
SHOW VARIABLES LIKE 'event_scheduler';
```

### Lỗi 2. Tạo event disabled rồi quên enable

Event có status:

```text
DISABLED
```

Khắc phục:

```sql
ALTER EVENT event_name ENABLE;
```

### Lỗi 3. Tạo one-time event ở thời điểm đã qua

Nếu schedule time đã nằm trong quá khứ, event có thể không được giữ/chạy như mong đợi tùy completion behavior.

Trong lab, luôn schedule thời điểm tương lai:

```sql
AT CURRENT_TIMESTAMP + INTERVAL 1 MINUTE
```

### Lỗi 4. Quên `ON COMPLETION PRESERVE`

One-time event chạy xong có thể biến mất, gây nhầm rằng event chưa được tạo.

Nếu cần inspect sau chạy:

```sql
ON COMPLETION PRESERVE
```

### Lỗi 5. Quên delimiter cho multi-statement event

Sai khi body có nhiều statements nhưng không đổi delimiter.

Đúng:

```sql
DELIMITER $$

CREATE EVENT ...
DO
BEGIN
    statement_1;
    statement_2;
END$$

DELIMITER ;
```

### Lỗi 6. Không kiểm tra timezone

Event chạy “đúng giờ server” nhưng không đúng business expectation do UTC/local time mismatch.

### Lỗi 7. Dùng recurring event để test rồi quên cleanup

Event test tiếp tục chạy, làm log tăng hoặc xóa dữ liệu ngoài ý muốn.

### Lỗi 8. Hard-code logic destructive không có retention condition

Không tạo event delete/update toàn table chỉ vì “cleanup”.

### Bài tập thực hành

**Bài 12.1.** Nêu query kiểm tra scheduler status.

**Bài 12.2.** Sửa event disabled để enable.

**Bài 12.3.** Viết schedule one-time event an toàn chạy sau 2 phút.

**Bài 12.4.** Viết skeleton multi-statement event có delimiter đúng.

**Bài 12.5.** Nêu ba thao tác cleanup sau khi demo recurring event.

---

## 13. Bài tập tổng hợp

### Bài 13.1. One-time event audit

Tạo event `ev_lab_one_time_audit`:

1. Chạy sau 2 phút.
2. Ghi một row vào `event_run_log`.
3. Dùng `ON COMPLETION PRESERVE`.
4. Ban đầu `DISABLE`.
5. Dùng `ALTER EVENT` để enable và kiểm tra `LAST_EXECUTED`.

### Bài 13.2. Cleanup expired notifications

Tạo event `ev_lab_cleanup_expired_notifications`:

1. Chạy mỗi ngày.
2. Bắt đầu từ ngày mai.
3. Xóa notification đã hết hạn.
4. Có comment mô tả retention.
5. Ban đầu disabled để review trước.

### Bài 13.3. Recurring summary refresh

Tạo event `ev_lab_refresh_notification_summary`:

1. Chạy mỗi ngày lúc thời điểm phù hợp.
2. Refresh `notification_summary`.
3. Ghi run log.
4. Dùng body `BEGIN ... END`.
5. Sau khi test manual, enable event.

### Bài 13.4. Event inventory report

Viết query hiển thị:

1. Event name.
2. Type.
3. Status.
4. Execute time hoặc interval.
5. Starts/ends.
6. Last executed.
7. Definer.
8. Comment.
9. Last altered time.

### Bài 13.5. Event review checklist

Lập checklist trước khi deploy production event gồm:

1. Scheduler status.
2. Privilege/definer.
3. Time zone.
4. Event status.
5. Schedule.
6. Destructive impact.
7. Indexes/row volume.
8. Logging/monitoring.
9. Replication behavior.
10. Rollback/disable plan.

---

## 14. Đáp án gợi ý

### Bài 13.1

```sql
DROP EVENT IF EXISTS event_lab.ev_lab_one_time_audit;

CREATE EVENT event_lab.ev_lab_one_time_audit
ON SCHEDULE AT CURRENT_TIMESTAMP + INTERVAL 2 MINUTE
ON COMPLETION PRESERVE
DISABLE
COMMENT 'One-time audit event for lab'
DO
    INSERT INTO event_lab.event_run_log (
        event_name,
        note,
        scheduler_user
    )
    VALUES (
        'ev_lab_one_time_audit',
        'One-time audit event executed',
        CURRENT_USER()
    );
```

Enable lại với thời điểm tương lai:

```sql
ALTER EVENT event_lab.ev_lab_one_time_audit
ON SCHEDULE AT CURRENT_TIMESTAMP + INTERVAL 1 MINUTE
ENABLE;
```

### Bài 13.2

```sql
DROP EVENT IF EXISTS event_lab.ev_lab_cleanup_expired_notifications;

CREATE EVENT event_lab.ev_lab_cleanup_expired_notifications
ON SCHEDULE EVERY 1 DAY
STARTS CURRENT_TIMESTAMP + INTERVAL 1 DAY
ON COMPLETION PRESERVE
DISABLE
COMMENT 'Deletes notifications after their expiry time'
DO
    DELETE FROM event_lab.notifications
    WHERE expires_at < CURRENT_TIMESTAMP;
```

### Bài 13.3

```sql
DROP EVENT IF EXISTS event_lab.ev_lab_refresh_notification_summary;

DELIMITER $$

CREATE EVENT event_lab.ev_lab_refresh_notification_summary
ON SCHEDULE EVERY 1 DAY
STARTS CURRENT_TIMESTAMP + INTERVAL 1 DAY
ON COMPLETION PRESERVE
DISABLE
COMMENT 'Refreshes active-notification summary'
DO
BEGIN
    INSERT INTO event_lab.notification_summary (
        summary_date,
        active_notifications,
        refreshed_at
    )
    SELECT CURRENT_DATE(),
           COUNT(*),
           CURRENT_TIMESTAMP
    FROM event_lab.notifications
    WHERE expires_at >= CURRENT_TIMESTAMP
    ON DUPLICATE KEY UPDATE
        active_notifications = VALUES(active_notifications),
        refreshed_at = VALUES(refreshed_at);

    INSERT INTO event_lab.event_run_log (
        event_name,
        note,
        scheduler_user
    )
    VALUES (
        'ev_lab_refresh_notification_summary',
        'Daily notification summary refreshed',
        CURRENT_USER()
    );
END$$

DELIMITER ;
```

### Bài 13.4

```sql
SELECT EVENT_NAME,
       EVENT_TYPE,
       STATUS,
       EXECUTE_AT,
       INTERVAL_VALUE,
       INTERVAL_FIELD,
       STARTS,
       ENDS,
       LAST_EXECUTED,
       DEFINER,
       EVENT_COMMENT,
       LAST_ALTERED
FROM INFORMATION_SCHEMA.EVENTS
WHERE EVENT_SCHEMA = 'event_lab'
ORDER BY EVENT_NAME;
```

---

## 15. Script cleanup sau lab

> **Cảnh báo:** Chỉ chạy khi chắc chắn schema `event_lab` chỉ dùng cho bài thực hành.

```sql
ALTER EVENT IF EXISTS event_lab.ev_lab_heartbeat DISABLE;
ALTER EVENT IF EXISTS event_lab.ev_lab_cleanup_expired_notifications DISABLE;
ALTER EVENT IF EXISTS event_lab.ev_lab_refresh_notification_summary DISABLE;

DROP EVENT IF EXISTS event_lab.ev_lab_one_time_log;
DROP EVENT IF EXISTS event_lab.ev_lab_one_time_audit_log;
DROP EVENT IF EXISTS event_lab.ev_lab_one_time_audit;
DROP EVENT IF EXISTS event_lab.ev_lab_heartbeat;
DROP EVENT IF EXISTS event_lab.ev_lab_cleanup_expired_notifications;
DROP EVENT IF EXISTS event_lab.ev_lab_refresh_notification_summary;
```

Nếu muốn xóa toàn bộ lab:

```sql
DROP DATABASE IF EXISTS event_lab;
```

> Khi `DROP DATABASE event_lab`, các event trong schema đó cũng bị xóa.

---

## 16. Tóm tắt

Các kiến thức chính:

- MySQL Event là scheduled database task, tương tự cron job hoặc Windows Task Scheduler ở mức khái niệm.
- Event chỉ chạy khi `event_scheduler = ON` và event có status `ENABLED`.
- `CREATE EVENT ... ON SCHEDULE AT` tạo one-time event.
- `CREATE EVENT ... ON SCHEDULE EVERY` tạo recurring event.
- `STARTS`, `ENDS`, `ENABLE`, `DISABLE`, `ON COMPLETION PRESERVE` và `COMMENT` giúp kiểm soát lịch và vòng đời event.
- `ALTER EVENT` sửa schedule, status, body, name, comment hoặc completion behavior.
- `SHOW EVENTS`, `SHOW CREATE EVENT` và `INFORMATION_SCHEMA.EVENTS` giúp review metadata.
- `DROP EVENT` chỉ xóa definition; không hoàn tác data changes đã xảy ra.
- Event chạy theo definer security context; cần quản lý privilege và definer lifecycle.
- Cần kiểm tra time zone, replication topology, idempotency, logging và cleanup.
- Event body nên nhỏ, rõ nghĩa, có index/retention guard khi thao tác dữ liệu lớn.
- Không bật scheduler trên shared/prod server nếu chưa biết các event enabled nào có thể chạy.

---

## 17. Từ khóa chính

- MySQL Event
- Event Scheduler
- event_scheduler
- CREATE EVENT
- ALTER EVENT
- DROP EVENT
- SHOW EVENTS
- SHOW CREATE EVENT
- INFORMATION_SCHEMA.EVENTS
- One-time Event
- Recurring Event
- ON SCHEDULE AT
- ON SCHEDULE EVERY
- STARTS
- ENDS
- ON COMPLETION PRESERVE
- ENABLE
- DISABLE
- DISABLE ON REPLICA
- DEFINER
- Time Zone
- Idempotency
- Scheduled Job
- event_lab
