---
title: "Tutorial: Quản lý Database trong MySQL"
author: "Tên giảng viên"
duration: "150m"
difficulty: "Beginner–Intermediate"
prerequisites:
  - "Đã kết nối được MySQL bằng mysql client hoặc MySQL Workbench"
  - "Đã biết SELECT, CREATE TABLE và khái niệm table cơ bản"
  - "Có quyền CREATE/DROP phù hợp trên môi trường lab"
summary: "Thực hành chọn database bằng USE, tạo database bằng CREATE DATABASE và xóa database bằng DROP DATABASE; kèm charset, collation, privilege, metadata và backup checklist."
---

# Tutorial: Quản lý Database trong MySQL

## Link tham khảo

### MySQL Tutorial

- [MySQL Administration](https://www.mysqltutorial.org/mysql-administration/)
- [MySQL Create Database](https://www.mysqltutorial.org/mysql-administration/mysql-create-database/)
- [MySQL Drop Database](https://www.mysqltutorial.org/mysql-administration/mysql-drop-database/)

### MySQL Reference Manual

- [Creating and Using a Database](https://dev.mysql.com/doc/refman/8.4/en/database-use.html)
- [USE Statement](https://dev.mysql.com/doc/refman/8.4/en/use.html)
- [CREATE DATABASE Statement](https://dev.mysql.com/doc/refman/8.4/en/create-database.html)
- [DROP DATABASE Statement](https://dev.mysql.com/doc/refman/8.4/en/drop-database.html)
- [SHOW DATABASES Statement](https://dev.mysql.com/doc/refman/8.4/en/show-databases.html)
- [SHOW CREATE DATABASE Statement](https://dev.mysql.com/doc/refman/8.4/en/show-create-database.html)
- [INFORMATION_SCHEMA.SCHEMATA](https://dev.mysql.com/doc/refman/8.4/en/information-schema-schemata-table.html)
- [Character Sets and Collations](https://dev.mysql.com/doc/refman/8.4/en/charset.html)
- [mysqldump](https://dev.mysql.com/doc/refman/8.4/en/mysqldump.html)

> **Phạm vi lab:** Bài này chỉ tạo/xóa schema `database_management_lab`. Không chạy `DROP DATABASE` trên production database, database dùng chung, hoặc system schemas như `mysql`, `information_schema`, `performance_schema`, `sys`.

---

## 1. Mục tiêu học tập

Sau khi học xong, người học có thể:

1. Giải thích database/schema, default database và fully qualified object name.
2. Dùng `SHOW DATABASES`, `DATABASE()` và `INFORMATION_SCHEMA.SCHEMATA` để kiểm tra metadata.
3. Dùng `USE` để chọn database cho session hiện tại.
4. Tạo database bằng `CREATE DATABASE` hoặc `CREATE SCHEMA`.
5. Khai báo default character set và collation cho database.
6. Dùng `IF NOT EXISTS` đúng mục đích.
7. Kiểm tra definition bằng `SHOW CREATE DATABASE`.
8. Xóa database bằng `DROP DATABASE` hoặc `DROP DATABASE IF EXISTS` một cách có kiểm soát.
9. Phân biệt reset lab với destructive operation trên production.
10. Lập backup/restore checklist trước khi xóa database.

---

## 2. Database, schema và default database

### 2.1. Database và schema trong MySQL

Trong MySQL, hai từ khóa sau là synonyms trong phần lớn DDL:

```sql
CREATE DATABASE inventory_lab;
```

```sql
CREATE SCHEMA inventory_lab;
```

Tương tự:

```sql
DROP DATABASE inventory_lab;
```

```sql
DROP SCHEMA inventory_lab;
```

Trong tài liệu này dùng từ **database** để nhất quán, nhưng `SCHEMA` là cú pháp hợp lệ.

### 2.2. Database là namespace chứa object

Một database có thể chứa:

```text
tables
views
stored procedures
stored functions
triggers
events
```

Ví dụ:

```text
classicmodels.customers
database_management_lab.lab_notes
inventory_lab.products
```

Hai database có thể cùng có table tên `orders`:

```text
sales_lab.orders
archive_lab.orders
```

### 2.3. Default database

Default database là database MySQL tự dùng cho object name không có schema prefix.

```sql
USE database_management_lab;

CREATE TABLE notes (
    note_id INT PRIMARY KEY,
    note_text VARCHAR(100)
);
```

Câu trên tương đương:

```sql
CREATE TABLE database_management_lab.notes (
    note_id INT PRIMARY KEY,
    note_text VARCHAR(100)
);
```

Nếu chưa chọn database và không viết schema prefix, bạn có thể gặp lỗi:

```text
No database selected
```

### 2.4. Không nhầm database với MySQL Server

| Khái niệm | Ví dụ | Ý nghĩa |
|---|---|---|
| MySQL Server | `mysqld` | Server process quản lý nhiều database |
| Database/schema | `classicmodels` | Namespace chứa tables/objects |
| Table | `classicmodels.customers` | Object chứa rows/columns |
| Session | Một tab Workbench | Có default database riêng |
| User account | `'student'@'localhost'` | Account có privileges |

### Bài tập thực hành

**Bài 2.1.** Phân biệt database và table.

**Bài 2.2.** Phân biệt MySQL Server và database.

**Bài 2.3.** Viết hai lệnh tương đương dùng `CREATE DATABASE` và `CREATE SCHEMA`.

**Bài 2.4.** Giải thích default database là gì.

**Bài 2.5.** Viết fully qualified name của table `orders` trong database `sales_lab`.

---

## 3. Liệt kê database và kiểm tra metadata

### 3.1. SHOW DATABASES

```sql
SHOW DATABASES;
```

Danh sách thực tế phụ thuộc server và account privileges. Một số schemas hệ thống thường gặp:

| Schema | Vai trò khái quát |
|---|---|
| `mysql` | Metadata/account/privilege do MySQL quản lý |
| `information_schema` | Metadata queryable về objects |
| `performance_schema` | Performance instrumentation |
| `sys` | Views/procedures hỗ trợ quan sát performance |
| `classicmodels` | Sample schema nếu đã import |

Không được dùng như exercise cleanup:

```sql
DROP DATABASE mysql;
DROP DATABASE information_schema;
DROP DATABASE performance_schema;
DROP DATABASE sys;
```

### 3.2. Lọc database names

```sql
SHOW DATABASES LIKE '%lab%';
```

```sql
SHOW DATABASES LIKE 'database_management%';
```

### 3.3. DATABASE()

```sql
SELECT DATABASE() AS current_database;
```

Nếu session chưa chọn default database, kết quả là `NULL`.

### 3.4. INFORMATION_SCHEMA.SCHEMATA

```sql
SELECT SCHEMA_NAME,
       DEFAULT_CHARACTER_SET_NAME,
       DEFAULT_COLLATION_NAME
FROM INFORMATION_SCHEMA.SCHEMATA
ORDER BY SCHEMA_NAME;
```

Chỉ thấy metadata mà account có quyền nhìn thấy.

### 3.5. SHOW CREATE DATABASE

```sql
SHOW CREATE DATABASE database_management_lab;
```

Lệnh này giúp kiểm tra database name, default character set và collation.

### Bài tập thực hành

**Bài 3.1.** Chạy `SHOW DATABASES;`.

**Bài 3.2.** Dùng `SHOW DATABASES LIKE '%lab%';`.

**Bài 3.3.** Chạy `SELECT DATABASE();` trước khi dùng `USE`.

**Bài 3.4.** Viết query `INFORMATION_SCHEMA.SCHEMATA` lấy schema có tên chứa `lab`.

**Bài 3.5.** Nêu mục đích của `information_schema`.

---

# Phần A. Select a Database

## 4. USE

### 4.1. Cú pháp

```sql
USE database_name;
```

Ví dụ:

```sql
USE classicmodels;
```

Sau đó:

```sql
SELECT COUNT(*) AS total_customers
FROM customers;
```

được hiểu là:

```sql
SELECT COUNT(*) AS total_customers
FROM classicmodels.customers;
```

### 4.2. USE thay đổi gì?

`USE db_name`:

- Đặt `db_name` làm default/current database cho **session hiện tại**.
- Có hiệu lực đến khi session đóng hoặc một `USE` khác chạy.
- Không thay đổi MySQL user account.
- Không tự cấp thêm privilege.
- Không ngăn truy cập object ở database khác nếu account có quyền.

```sql
USE db1;

SELECT *
FROM author;

SELECT *
FROM db2.editor;
```

### 4.3. Kiểm tra sau USE

```sql
USE database_management_lab;

SELECT DATABASE() AS current_database;
```

### 4.4. Fully qualified names

Cú pháp:

```sql
database_name.table_name
```

Ví dụ:

```sql
SELECT *
FROM classicmodels.customers;
```

Nếu tên có ký tự đặc biệt, dùng backticks:

```sql
SELECT *
FROM `sales-report`.orders;
```

Tuy nhiên, nên dùng database names đơn giản:

```text
lowercase
snake_case
không space
không dấu tiếng Việt
không ký tự đặc biệt
```

### 4.5. USE và Workbench

Trong Workbench, double-click schema trong panel **SCHEMAS** thường đặt schema đó làm default cho SQL tab. Luôn kiểm tra schema default trước DDL/DML.

Trong mysql client, có thể chọn database khi kết nối:

```bash
mysql -u user_name -p database_management_lab
```

### Bài tập thực hành

**Bài 4.1.** Chạy `USE classicmodels;` nếu database này tồn tại và bạn có quyền.

**Bài 4.2.** Chạy `SELECT DATABASE();`.

**Bài 4.3.** Sau `USE classicmodels`, đếm số rows của `customers`.

**Bài 4.4.** Viết cùng query bằng `classicmodels.customers` mà không dùng `USE`.

**Bài 4.5.** Giải thích vì sao `USE db_name` không cấp privilege mới.

---

## 5. Truy cập nhiều database trong một session

### 5.1. Cross-database query

```sql
SELECT c.customerName,
       a.customer_id,
       a.archived_at
FROM classicmodels.customers AS c
JOIN archive_lab.customer_archive AS a
    ON a.customer_number = c.customerNumber;
```

Không cần chuyển default database giữa các tables nếu sử dụng full names.

### 5.2. Tạo object ở schema khác

Sau khi:

```sql
USE database_management_lab;
```

bạn vẫn có thể tạo object ở schema khác nếu có quyền:

```sql
CREATE TABLE another_lab.sample_table (
    id INT PRIMARY KEY
);
```

Trong script, chọn một cách rõ ràng:

- `USE target_schema;` ở đầu file; hoặc
- Fully qualify object names cho script multi-schema.

Không trộn hai kiểu khó theo dõi.

### 5.3. Default database không phải security boundary

Default database chỉ là context tiện lợi. Security boundary là grants:

```text
SELECT
INSERT
UPDATE
DELETE
CREATE
DROP
ALTER
EXECUTE
...
```

### Bài tập thực hành

**Bài 5.1.** Viết fully qualified name cho `products` trong `inventory_lab`.

**Bài 5.2.** Viết query đọc `classicmodels.customers` mà không dùng `USE classicmodels`.

**Bài 5.3.** Giải thích vì sao default database không phải security boundary.

**Bài 5.4.** Nêu hai cách script đảm bảo table được tạo đúng schema.

**Bài 5.5.** Viết một JOIN minh họa hai schemas bằng aliases.

---

# Phần B. Create Databases

## 6. CREATE DATABASE

### 6.1. Cú pháp cơ bản

```sql
CREATE DATABASE database_name;
```

Ví dụ:

```sql
CREATE DATABASE database_management_lab;
```

`CREATE SCHEMA` là synonym:

```sql
CREATE SCHEMA database_management_lab;
```

### 6.2. CREATE privilege

`CREATE DATABASE` cần `CREATE` privilege phù hợp.

Kiểm tra grants:

```sql
SHOW GRANTS;
```

Không tự cấp global privileges trên server dùng chung. Dùng local MySQL instance hoặc database do giảng viên/DBA cấp.

### 6.3. IF NOT EXISTS

```sql
CREATE DATABASE IF NOT EXISTS database_management_lab;
```

`IF NOT EXISTS` tránh error khi database đã tồn tại. Nó **không** đảm bảo:

```text
Database đang trống.
Database có charset/collation đúng.
Tables/routines/views đã đúng definition.
```

Kiểm tra bằng:

```sql
SHOW CREATE DATABASE database_management_lab;
```

### 6.4. Character set và collation

Ví dụ cho lab dùng Unicode:

```sql
CREATE DATABASE database_management_lab
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_0900_ai_ci;
```

- `utf8mb4` phù hợp Unicode đa ngôn ngữ.
- Collation ảnh hưởng comparison/sorting default ở database level.
- Table/column có thể override khi cần.

### 6.5. Xem charset/collation có sẵn

```sql
SHOW CHARACTER SET;
```

```sql
SHOW COLLATION LIKE 'utf8mb4%';
```

### 6.6. Không tạo database bằng file system

Không tạo folder thủ công trong MySQL data directory:

```text
mkdir /var/lib/mysql/new_database
```

Dùng `CREATE DATABASE`; server phải quản lý data directory/files.

### Bài tập thực hành

**Bài 6.1.** Tạo `database_management_lab` với utf8mb4/collation phù hợp.

**Bài 6.2.** Chạy `SHOW CREATE DATABASE database_management_lab;`.

**Bài 6.3.** Chạy lại CREATE với `IF NOT EXISTS`.

**Bài 6.4.** Dùng `SHOW CHARACTER SET` tìm `utf8mb4`.

**Bài 6.5.** Giải thích vì sao `CREATE DATABASE IF NOT EXISTS` không phải migration tool.

---

## 7. Database initialization script

### 7.1. Mẫu setup cho lab

```sql
DROP DATABASE IF EXISTS bookstore_lab;

CREATE DATABASE bookstore_lab
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_0900_ai_ci;

USE bookstore_lab;
```

> `DROP DATABASE IF EXISTS` phù hợp isolated lab/reset environment. Không đặt nó ở đầu production migration script nếu chưa có backup, review và confirmation.

### 7.2. Dev, test và production

Nên tách environment:

```text
course_registration_dev
course_registration_test
course_registration_stage
course_registration_prod
```

Trong lớp học:

```text
group01_course_registration_lab
group02_course_registration_lab
```

Lợi ích:

- Reset test data không ảnh hưởng production.
- Test migration/backup/restore trước.
- Phân quyền theo environment.
- Tránh chạy nhầm destructive command.

### 7.3. Một database hay nhiều database?

Đánh giá theo:

- Domain/application boundary.
- Team ownership.
- Security/privilege boundary.
- Backup/restore lifecycle.
- Deployment/migration workflow.
- Cross-database query/dependency complexity.

Không tách database chỉ vì muốn chia tables theo alphabet hoặc theo từng màn hình UI.

### Bài tập thực hành

**Bài 7.1.** Viết script tạo `inventory_project_lab`.

**Bài 7.2.** Đề xuất database names cho dev/test/prod của course registration project.

**Bài 7.3.** Giải thích vì sao không DROP database ở đầu mọi production migration script.

**Bài 7.4.** Nêu ba yếu tố khi quyết định tách database.

**Bài 7.5.** Viết setup script lab gồm DROP IF EXISTS, CREATE và USE.

---

## 8. Metadata của database

### 8.1. INFORMATION_SCHEMA.SCHEMATA

```sql
SELECT SCHEMA_NAME,
       DEFAULT_CHARACTER_SET_NAME,
       DEFAULT_COLLATION_NAME
FROM INFORMATION_SCHEMA.SCHEMATA
WHERE SCHEMA_NAME = 'database_management_lab';
```

### 8.2. Tạo table test

```sql
USE database_management_lab;

CREATE TABLE lab_notes (
    note_id INT NOT NULL AUTO_INCREMENT,
    note_text VARCHAR(255) NOT NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (note_id)
) ENGINE = InnoDB;
```

```sql
SHOW TABLES;
```

### 8.3. Session identity và current database

```sql
SELECT DATABASE() AS current_database,
       CURRENT_USER() AS privilege_account,
       USER() AS login_account;
```

### Bài tập thực hành

**Bài 8.1.** Query metadata của `database_management_lab`.

**Bài 8.2.** Tạo `lab_notes`.

**Bài 8.3.** Chạy `SHOW TABLES`.

**Bài 8.4.** Chạy `SELECT DATABASE(), CURRENT_USER(), USER();`.

**Bài 8.5.** Giải thích vì sao không tạo database bằng folder trong data directory.

---

# Phần C. Drop Databases

## 9. DROP DATABASE

### 9.1. Cú pháp

```sql
DROP DATABASE database_name;
```

Hoặc:

```sql
DROP SCHEMA database_name;
```

Cleanup-safe cho lab:

```sql
DROP DATABASE IF EXISTS database_management_lab;
```

### 9.2. DROP DATABASE làm gì?

`DROP DATABASE` xóa database và tables/data của nó. Nó không phải lệnh để:

| Mục tiêu | Lệnh phù hợp hơn |
|---|---|
| Xóa một số rows | `DELETE FROM ... WHERE ...` |
| Làm rỗng staging table | `TRUNCATE TABLE ...` sau dependency review |
| Xóa một table | `DROP TABLE ...` |
| Xóa một column | `ALTER TABLE ... DROP COLUMN ...` |
| Chọn schema khác | `USE ...` |

### 9.3. DROP privilege

Account cần `DROP` privilege trên database. User chỉ có `SELECT`/`INSERT` không mặc định được drop schema.

### 9.4. IF EXISTS

```sql
DROP DATABASE IF EXISTS database_management_lab;
```

`IF EXISTS` chỉ tránh error khi database không tồn tại. Nó không tạo backup hoặc confirmation.

### 9.5. Drop default database

```sql
USE database_management_lab;

DROP DATABASE database_management_lab;

SELECT DATABASE() AS current_database;
```

Nếu database bị drop là default database, `DATABASE()` trả `NULL`.

### 9.6. Grants/accounts cần review riêng

Nếu từng grant quyền riêng cho schema lab:

```sql
GRANT SELECT
ON database_management_lab.*
TO 'lab_reader'@'localhost';
```

sau khi drop database vẫn cần review account/grants theo policy. Không coi DROP DATABASE là complete security cleanup.

### 9.7. Temporary tables

Temporary tables tồn tại tới khi session tạo chúng kết thúc. Không dựa vào `DROP DATABASE` để cleanup temporary tables của các sessions khác.

### Bài tập thực hành

**Bài 9.1.** Viết `DROP DATABASE IF EXISTS` cho schema `inventory_project_lab`.

**Bài 9.2.** Nêu ba thứ có thể mất khi drop database.

**Bài 9.3.** Phân biệt DROP DATABASE và DROP TABLE.

**Bài 9.4.** Dự đoán `DATABASE()` trả gì sau khi drop default database.

**Bài 9.5.** Giải thích vì sao DROP DATABASE không thay privilege cleanup.

---

## 10. Checklist an toàn trước DROP DATABASE

Trước khi xóa database không phải lab disposable:

```text
[ ] Đúng MySQL Server và đúng environment?
[ ] Đúng database name?
[ ] Có backup gần nhất?
[ ] Đã test restore?
[ ] Có active application connections, jobs, events hay ETL không?
[ ] Có replication/reporting/dependency không?
[ ] Có users/roles/grants cần review không?
[ ] Có retention/compliance requirement không?
[ ] Có migration/recreate script hoặc rollback plan không?
[ ] Có approval/maintenance window nếu quy trình yêu cầu không?
```

### 10.1. Backup trước destructive operation

Ví dụ terminal command:

```bash
mysqldump -u root -p --routines --triggers --events \
  database_management_lab > database_management_lab_backup.sql
```

Password nên được hỏi interactive. Không hard-code password thật vào SQL script, report hoặc repository.

Restore test vào environment/schema phù hợp, không ghi đè source database:

```bash
mysql -u root -p < database_management_lab_backup.sql
```

### 10.2. Không dùng DROP DATABASE như reset button production

Trong lab:

```sql
DROP DATABASE IF EXISTS database_management_lab;
CREATE DATABASE database_management_lab ...;
```

có thể hợp lý.

Trong production, reset thường cần backup, retention review, migration/change process và approval.

### Bài tập thực hành

**Bài 10.1.** Liệt kê năm câu hỏi trước DROP production database.

**Bài 10.2.** Viết mysqldump command backup database kèm routines/triggers/events.

**Bài 10.3.** Giải thích vì sao backup file không đủ nếu chưa test restore.

**Bài 10.4.** Nêu một tình huống DROP DATABASE hợp lý trong lab.

**Bài 10.5.** Nêu một tình huống không nên dùng DROP DATABASE.

---

## 11. Thực hành controlled cleanup

### 11.1. Kiểm tra trước cleanup

```sql
SHOW DATABASES LIKE 'database_management_lab';

SHOW CREATE DATABASE database_management_lab;

USE database_management_lab;

SHOW TABLES;

SELECT COUNT(*) AS total_notes
FROM lab_notes;
```

### 11.2. Backup nếu cần giữ lab results

```bash
mysqldump -u root -p --routines --triggers --events \
  database_management_lab > database_management_lab_backup.sql
```

### 11.3. Drop lab database

```sql
DROP DATABASE IF EXISTS database_management_lab;
```

### 11.4. Xác minh

```sql
SHOW DATABASES LIKE 'database_management_lab';

SELECT DATABASE() AS current_database;
```

### 11.5. Recreate lab

```sql
CREATE DATABASE database_management_lab
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_0900_ai_ci;

USE database_management_lab;
```

### Bài tập thực hành

**Bài 11.1.** Kiểm tra schema/tables trước cleanup.

**Bài 11.2.** Tạo backup logic nếu có mysqldump.

**Bài 11.3.** Drop schema lab bằng IF EXISTS.

**Bài 11.4.** Xác minh schema không còn trong SHOW DATABASES.

**Bài 11.5.** Recreate schema lab.

---

## 12. Privileges và database lifecycle

### 12.1. Least privilege

MySQL access được quyết định bởi privileges, không phải việc một user đã `USE` schema nào.

Ví dụ local lab user chỉ đọc:

```sql
CREATE USER IF NOT EXISTS 'lab_reader'@'localhost'
IDENTIFIED BY 'ChangeThisLocalLabPassword!';

GRANT SELECT
ON database_management_lab.*
TO 'lab_reader'@'localhost';

SHOW GRANTS FOR 'lab_reader'@'localhost';
```

> Chỉ chạy trên local/lab khi có quyền quản trị. Không dùng password mẫu này cho môi trường thật.

### 12.2. CREATE database khác grant access

```sql
CREATE DATABASE course_registration_lab;
```

không tự cho mọi account access database đó.

```sql
GRANT SELECT, INSERT, UPDATE, DELETE
ON course_registration_lab.*
TO 'app_user'@'localhost';
```

### 12.3. SHOW DATABASES và metadata visibility

Không thấy database trong `SHOW DATABASES` không chắc database không tồn tại; account có thể không có quyền thấy/access nó.

### 12.4. Không cấp ALL PRIVILEGES mặc định

Không dùng như default lab command:

```sql
GRANT ALL PRIVILEGES ON *.*
TO 'student'@'%';
```

Ưu tiên scope nhỏ, quyền tối thiểu, host cụ thể và role rõ mục đích.

### Bài tập thực hành

**Bài 12.1.** Giải thích least privilege.

**Bài 12.2.** Viết GRANT SELECT cho user reporter trên schema lab.

**Bài 12.3.** Dùng SHOW GRANTS kiểm tra.

**Bài 12.4.** Phân biệt CREATE DATABASE và GRANT access.

**Bài 12.5.** Nêu lý do không cấp ALL PRIVILEGES ON *.* cho lab user.

---

## 13. Naming, charset và collation policy

### 13.1. Naming policy

Dùng:

```text
lowercase_snake_case
```

Ví dụ:

```text
course_registration_lab
inventory_management
sales_reporting
student_portal_dev
```

Tránh:

```text
Course Registration
CSDL_Đăng_Ký
final-final-v3!
test1
```

### 13.2. Charset policy

```sql
CREATE DATABASE student_portal_lab
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_0900_ai_ci;
```

Database default charset/collation là base setting; table/column có thể override khi needed.

### 13.3. Hierarchy

```text
Server default
  -> Database default
      -> Table default
          -> Column definition
              -> Expression/connection context
```

### Bài tập thực hành

**Bài 13.1.** Đề xuất database name cho hệ thống thư viện.

**Bài 13.2.** Tạo database lab dùng utf8mb4.

**Bài 13.3.** Kiểm tra charset/collation bằng SHOW CREATE DATABASE.

**Bài 13.4.** Giải thích vì sao database default charset không luôn là charset của mọi column.

**Bài 13.5.** Nêu hai lợi ích naming policy.

---

## 14. Bài tập tổng hợp

### Bài 14.1. Database lifecycle lab

1. Kiểm tra `library_management_lab` bằng `SHOW DATABASES LIKE`.
2. Tạo database dùng utf8mb4.
3. Chọn bằng `USE`.
4. Tạo table `books`.
5. Kiểm tra `DATABASE()`, `SHOW TABLES`, `SHOW CREATE DATABASE`.
6. Viết mysqldump backup command.
7. Drop lab database bằng `DROP DATABASE IF EXISTS`.

### Bài 14.2. Default database versus full name

1. Tạo `sales_lab` và `archive_lab`.
2. Trong mỗi schema tạo table `orders` tối giản.
3. Insert data khác nhau.
4. `USE sales_lab`, query `orders`.
5. Query `archive_lab.orders` mà không đổi default database.
6. Giải thích output.
7. Cleanup hai schemas.

### Bài 14.3. CREATE DATABASE safety

Cho script:

```sql
CREATE DATABASE IF NOT EXISTS inventory_system;
USE inventory_system;
CREATE TABLE products (...);
```

Trả lời:

1. Script làm gì khi database chưa tồn tại?
2. Script làm gì khi database đã tồn tại?
3. Có bảo đảm `products` đúng definition không?
4. Lệnh kiểm tra database definition là gì?
5. Lệnh kiểm tra table definition là gì?
6. Rủi ro nếu chạy nhầm production database?
7. Cách cải thiện cho lab và production migration?

### Bài 14.4. DROP DATABASE incident review

Lập checklist 10 bước trước:

```sql
DROP DATABASE course_registration;
```

Bao gồm environment, backup, restore test, active dependency, grants, retention, approval, rollback/recreate script và post-drop validation.

### Bài 14.5. Privilege design

Thiết kế local-lab accounts/roles:

1. `course_reporter`: chỉ SELECT.
2. `course_operator`: SELECT/INSERT/UPDATE nhưng không DROP database.
3. `course_schema_manager`: quyền migration phù hợp cho lab.
4. Viết SHOW GRANTS commands.
5. Giải thích lợi ích role.

---

## 15. Đáp án gợi ý

### Bài 14.1

```sql
SHOW DATABASES LIKE 'library_management_lab';

CREATE DATABASE IF NOT EXISTS library_management_lab
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_0900_ai_ci;

USE library_management_lab;

CREATE TABLE books (
    book_id INT UNSIGNED NOT NULL AUTO_INCREMENT,
    title VARCHAR(255) NOT NULL,
    published_year SMALLINT NULL,
    PRIMARY KEY (book_id)
) ENGINE = InnoDB;

SELECT DATABASE() AS current_database;

SHOW TABLES;

SHOW CREATE DATABASE library_management_lab;
```

Cleanup:

```sql
DROP DATABASE IF EXISTS library_management_lab;
```

### Bài 14.2

```sql
CREATE DATABASE IF NOT EXISTS sales_lab;
CREATE DATABASE IF NOT EXISTS archive_lab;

CREATE TABLE sales_lab.orders (
    order_id INT PRIMARY KEY,
    note VARCHAR(100)
);

CREATE TABLE archive_lab.orders (
    order_id INT PRIMARY KEY,
    note VARCHAR(100)
);

INSERT INTO sales_lab.orders VALUES (1, 'Current sales order');
INSERT INTO archive_lab.orders VALUES (1, 'Archived order');

USE sales_lab;

SELECT * FROM orders;

SELECT * FROM archive_lab.orders;
```

Cleanup:

```sql
DROP DATABASE IF EXISTS sales_lab;
DROP DATABASE IF EXISTS archive_lab;
```

### Bài 14.5

```sql
CREATE ROLE IF NOT EXISTS
    'role_course_reporter',
    'role_course_operator';

GRANT SELECT
ON course_registration_lab.*
TO 'role_course_reporter';

GRANT SELECT, INSERT, UPDATE
ON course_registration_lab.*
TO 'role_course_operator';

CREATE USER IF NOT EXISTS 'course_reporter'@'localhost'
IDENTIFIED BY 'ChangeThisLocalLabPassword!';

GRANT 'role_course_reporter'
TO 'course_reporter'@'localhost';

SET DEFAULT ROLE 'role_course_reporter'
TO 'course_reporter'@'localhost';

SHOW GRANTS FOR 'course_reporter'@'localhost';
```

---

## 16. Lỗi thường gặp

### Lỗi 1. Quên USE rồi tạo table

```sql
CREATE TABLE products (...);
```

có thể báo:

```text
No database selected
```

Khắc phục:

```sql
USE database_management_lab;
```

hoặc:

```sql
CREATE TABLE database_management_lab.products (...);
```

### Lỗi 2. Dùng IF NOT EXISTS và nghĩ schema đã đúng

`IF NOT EXISTS` không alter existing definition.

Khắc phục:

```sql
SHOW CREATE DATABASE database_name;
```

và kiểm tra tables/objects riêng.

### Lỗi 3. Drop nhầm environment

Trước destructive statement:

```sql
SELECT DATABASE();
SHOW DATABASES LIKE 'target_name';
SHOW TABLES;
```

Đồng thời kiểm tra connection profile/server host trong Workbench.

### Lỗi 4. Drop database để xóa một table

Dùng:

```sql
DROP TABLE table_name;
```

nếu chỉ cần xóa một table.

### Lỗi 5. Coi default database là privilege

`USE` chỉ đổi context. Privileges thực tế do GRANT quyết định.

### Lỗi 6. Hard-code password

Không commit password thật vào SQL script, report, Git repository hoặc screenshot.

### Lỗi 7. Drop schema nhưng quên review grants/accounts

Database-specific grants và account lifecycle cần review riêng.

### Bài tập thực hành

**Bài 16.1.** Sửa CREATE TABLE lỗi `No database selected`.

**Bài 16.2.** Nêu lệnh kiểm tra definition database.

**Bài 16.3.** Nêu lệnh kiểm tra default database hiện tại.

**Bài 16.4.** Nêu một việc cần làm trước DROP DATABASE ngoài backup.

**Bài 16.5.** Nêu lý do không hard-code password trong script.

---

## 17. Tóm tắt

- Database và schema là synonyms trong phần lớn MySQL DDL.
- `USE db_name` chọn default database cho session; không cấp privilege mới.
- `SHOW DATABASES`, `DATABASE()`, `SHOW CREATE DATABASE` và `INFORMATION_SCHEMA.SCHEMATA` giúp kiểm tra metadata.
- `CREATE DATABASE` cần CREATE privilege; `CREATE SCHEMA` là synonym.
- `CREATE DATABASE IF NOT EXISTS` chỉ tránh error nếu database đã tồn tại; không migrate definition.
- Character set/collation cần được chọn có chủ đích; `utf8mb4` thường phù hợp data Unicode.
- `DROP DATABASE` xóa database và dữ liệu; cần DROP privilege, backup/review và cẩn trọng.
- Drop default database khiến current/default database bị unset và `DATABASE()` trả NULL.
- Review accounts/grants riêng sau lifecycle change.
- Dùng schema lab riêng cho learning/dev/test; không dùng DROP DATABASE như production reset button.

---

## 18. Từ khóa chính

- Database
- Schema
- Default database
- USE
- DATABASE()
- SHOW DATABASES
- SHOW CREATE DATABASE
- INFORMATION_SCHEMA.SCHEMATA
- CREATE DATABASE
- CREATE SCHEMA
- IF NOT EXISTS
- Character set
- Collation
- utf8mb4
- CREATE privilege
- DROP DATABASE
- DROP SCHEMA
- DROP privilege
- Fully qualified name
- Least privilege
- mysqldump
- Backup
- Restore test
- database_management_lab
