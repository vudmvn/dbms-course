---
title: "Tutorial 1: Khám phá MySQL Server — Kiến trúc và mysqld"
author: "Tên giảng viên"
duration: "120m"
difficulty: "Beginner–Intermediate"
prerequisites:
  - "Đã biết database, table và câu lệnh SQL cơ bản"
  - "Có thể kết nối tới một MySQL Server bằng mysql client hoặc MySQL Workbench"
summary: "Giải thích kiến trúc MySQL ở mức vận hành, vai trò của mysqld, luồng xử lý truy vấn, storage engines và các lệnh kiểm tra server an toàn."
---

# Tutorial 1: Khám phá MySQL Server — Kiến trúc và `mysqld`

## Link tham khảo

- [MySQL Tutorial — MySQL Administration](https://www.mysqltutorial.org/mysql-administration/)
- [MySQL Tutorial — MySQL Architecture](https://www.mysqltutorial.org/mysql-administration/mysql-architecture/)
- [MySQL Tutorial — mysqld: The MySQL Server](https://www.mysqltutorial.org/mysql-administration/mysqld/)


> Bài này mô tả kiến trúc ở mức thực hành quản trị. Một MySQL Server thực tế có thêm nhiều thành phần như replication, binary logging, plugins, Performance Schema, security subsystem, background threads và cache/buffer của storage engine.

---

## 1. Mục tiêu học tập

Sau khi học xong, người học có thể:

1. Phân biệt MySQL Server, `mysqld`, MySQL client và MySQL Workbench.
2. Mô tả luồng cơ bản từ client gửi SQL đến storage engine trả kết quả.
3. Giải thích các lớp chính: connection/authentication, SQL layer, storage engine và data/log files.
4. Nêu vai trò của `mysqld`.
5. Phân biệt server option, system variable và status variable.
6. Dùng các truy vấn an toàn để kiểm tra version, connection, engine và thông tin server.
7. Giải thích tại sao InnoDB là storage engine thường được chọn cho ứng dụng giao dịch.
8. Nhận biết những tác vụ cần thận trọng: restart service, đổi config, xóa file data directory.

---

## 2. Các thành phần trong một hệ thống MySQL

Một hệ thống cơ bản có thể hình dung:

```text
+-------------------+
| Application       |
| Python / Java /   |
| PHP / Node.js     |
+---------+---------+
          |
          | MySQL protocol / TCP or local socket
          v
+-------------------+
| Client program    |
| mysql / Workbench |
+---------+---------+
          |
          v
+--------------------------------------------------+
| mysqld: MySQL Server                             |
|                                                  |
|  Connection + authentication + authorization     |
|  SQL parser / optimizer / executor               |
|  Storage-engine interface                        |
|      |                    |                      |
|    InnoDB               MyISAM                ...|
+------|-------------------------------------------+
       |
       v
+--------------------------------------------------+
| Data directory, tablespaces, redo logs, logs,    |
| system schema and other server-managed files     |
+--------------------------------------------------+
```

### 2.1. MySQL Server

**MySQL Server** là hệ quản trị cơ sở dữ liệu chạy ở phía server. Nó:

- Nhận kết nối từ client.
- Xác thực account.
- Kiểm tra quyền truy cập.
- Phân tích và tối ưu câu SQL.
- Gọi storage engine để đọc/ghi dữ liệu.
- Trả result set hoặc trạng thái thực thi cho client.

### 2.2. `mysqld`

`mysqld` là executable/process chính của MySQL Server.

Trên Windows, `mysqld.exe` thường chạy như một Windows Service.  
Trên Linux, `mysqld` hoặc service package tương ứng thường được systemd quản lý.

Không nên nhầm:

| Thành phần | Vai trò |
|---|---|
| `mysqld` | Database server process |
| `mysql` | Command-line client để kết nối server |
| MySQL Workbench | GUI client/tool quản trị |
| `mysqldump` | Utility logical backup |
| `mysqladmin` | Utility kiểm tra/quản trị server cơ bản |

### 2.3. Client không phải server

Một máy có MySQL Workbench chưa chắc đang chạy `mysqld` cục bộ. Workbench có thể kết nối đến server ở máy khác.

Tương tự, cài `mysql` client không có nghĩa máy đó đang chứa MySQL Server.

### Bài tập thực hành

**Bài 2.1.** Giải thích khác nhau giữa `mysqld` và `mysql`.

**Bài 2.2.** MySQL Workbench là server hay client? Giải thích.

**Bài 2.3.** Nêu ba nhiệm vụ của MySQL Server sau khi client gửi một câu `SELECT`.

**Bài 2.4.** Viết tên một công cụ dùng để backup logical database MySQL.

**Bài 2.5.** Giải thích vì sao một máy có MySQL client chưa chắc có MySQL Server đang chạy.

---

## 3. Kiến trúc MySQL ở mức khái niệm

### 3.1. Connection, authentication và authorization

Khi client kết nối:

1. Client kết nối bằng TCP/IP, Unix socket hoặc cơ chế local phù hợp.
2. Server nhận kết nối.
3. Server xác thực account, ví dụ `'user'@'host'`.
4. Server xác định privilege của account.
5. Client gửi statement SQL trong session đó.

Ví dụ kiểm tra account:

```sql
SELECT USER() AS loginUser,
       CURRENT_USER() AS privilegeAccount;
```

`USER()` mô tả user/host từ kết nối client; `CURRENT_USER()` là account MySQL được dùng khi kiểm tra privilege.

### 3.2. SQL layer

Ở mức khái niệm, SQL layer xử lý:

- Parse cú pháp SQL.
- Kiểm tra tên database/table/column.
- Kiểm tra quyền.
- Tối ưu hóa query plan.
- Điều phối thực thi.
- Trả result set hoặc thông báo lỗi.

Ví dụ, khi chạy:

```sql
SELECT customerName, country
FROM customers
WHERE country = 'USA'
ORDER BY customerName;
```

server phải:

1. Phân tích câu lệnh.
2. Xác định database/table/cột.
3. Kiểm tra `SELECT` privilege.
4. Chọn kế hoạch thực thi.
5. Yêu cầu storage engine đọc dữ liệu.
6. Sắp xếp/trả result set cho client nếu cần.

### 3.3. Storage-engine layer

Storage engine chịu trách nhiệm lưu trữ và truy xuất dữ liệu ở tầng thấp hơn.

MySQL có kiến trúc pluggable storage engine: cùng một SQL layer có thể làm việc với nhiều engine.

Kiểm tra engines hiện có:

```sql
SHOW ENGINES;
```

Một số engine thường gặp:

| Engine | Đặc điểm khái quát |
|---|---|
| InnoDB | Hỗ trợ transaction, crash recovery, foreign key và row-level locking |
| MyISAM | Engine cũ; không hỗ trợ transaction theo cách InnoDB hỗ trợ |
| MEMORY | Dữ liệu trong RAM; cần hiểu rõ tính không bền vững trước khi dùng |
| CSV | Lưu dữ liệu dạng CSV, phù hợp trường hợp đặc thù |

Trong hầu hết ứng dụng giao dịch hiện đại, InnoDB là lựa chọn mặc định/cần xem xét đầu tiên.

### 3.4. Data directory, tablespace và logs

`mysqld` quản lý dữ liệu thông qua data directory và các file server-managed. Các thành phần có thể gồm:

- Database directories.
- MySQL system schema.
- InnoDB tablespaces và log files.
- Error log, binary log hoặc các log được cấu hình.
- Metadata và data dictionary.

Không tự ý xóa, copy, đổi tên hoặc chỉnh sửa file trong data directory khi server còn chạy.

### Bài tập thực hành

**Bài 3.1.** Viết truy vấn kiểm tra `USER()` và `CURRENT_USER()`.

**Bài 3.2.** Chạy `SHOW ENGINES;` và tìm dòng `InnoDB`.

**Bài 3.3.** Nêu hai nhiệm vụ của SQL layer.

**Bài 3.4.** Nêu hai nhiệm vụ của storage engine.

**Bài 3.5.** Giải thích vì sao không được tự ý xóa file trong MySQL data directory.

---

## 4. Luồng xử lý một câu truy vấn

Xét truy vấn:

```sql
USE classicmodels;

SELECT o.orderNumber,
       o.orderDate,
       c.customerName
FROM orders AS o
JOIN customers AS c
    ON c.customerNumber = o.customerNumber
WHERE o.status = 'Shipped'
ORDER BY o.orderDate DESC;
```

Luồng xử lý khái quát:

```text
Client
  |
  | 1. Gửi SQL
  v
mysqld connection/session layer
  |
  | 2. Authentication + authorization
  v
SQL parser and optimizer
  |
  | 3. Tạo execution plan
  v
Storage-engine interface
  |
  | 4. Đọc index / pages / rows
  v
InnoDB or another engine
  |
  | 5. Trả row data
  v
SQL executor
  |
  | 6. Join, filter, sort, create result set
  v
Client receives result
```

### 4.1. Tại sao index và optimizer quan trọng?

Storage engine có thể phải đọc nhiều dữ liệu nếu query không có index phù hợp hoặc điều kiện không chọn lọc.

Optimizer cố chọn kế hoạch phù hợp dựa trên:

- Cấu trúc query.
- Available indexes.
- Statistics.
- Join order.
- Estimated cost.

Dùng `EXPLAIN` để quan sát kế hoạch query:

```sql
EXPLAIN
SELECT o.orderNumber,
       o.orderDate,
       c.customerName
FROM orders AS o
JOIN customers AS c
    ON c.customerNumber = o.customerNumber
WHERE o.status = 'Shipped';
```

> Không kết luận query tốt/xấu chỉ từ một cột của `EXPLAIN`. Cần đọc toàn bộ kế hoạch, dữ liệu, index và workload.

### 4.2. Query result không nhất thiết được cache như người học hình dung

Đừng giả định mọi `SELECT` đều được cache ở một "query cache" chung. Cách MySQL xử lý cache/buffer phụ thuộc version, configuration và storage engine. Với InnoDB, buffer pool là thành phần quan trọng cho việc cache data/index pages.

### Bài tập thực hành

**Bài 4.1.** Vẽ lại luồng xử lý từ client đến storage engine cho một câu `SELECT`.

**Bài 4.2.** Chạy `EXPLAIN` cho một query join giữa `orders` và `customers`.

**Bài 4.3.** Nêu vai trò của optimizer.

**Bài 4.4.** Nêu lý do storage engine cần index.

**Bài 4.5.** Giải thích vì sao không nên đánh giá query chỉ dựa vào việc nó trả kết quả nhanh trên bảng dữ liệu nhỏ.

---

## 5. `mysqld`: server process và server options

### 5.1. `mysqld` đọc options từ đâu?

`mysqld` có thể nhận options:

- Từ option file.
- Từ command line.
- Trong một số trường hợp từ môi trường/service manager.

Các options cần ổn định qua nhiều lần restart thường nên được đặt trong option file thay vì chỉ truyền thủ công trên command line.

`mysqld` đọc các option groups như:

```text
[mysqld]
[server]
```

Ví dụ tối giản trong option file:

```ini
[mysqld]
port=3306
max_connections=150
```

Không sao chép nguyên config mẫu vào production mà không đánh giá workload, RAM, security và cách cài đặt.

### 5.2. Xem help của `mysqld`

Trong terminal có PATH phù hợp:

```bash
mysqld --help
```

Xem danh sách chi tiết hơn:

```bash
mysqld --verbose --help
```

Lệnh này hữu ích để:

- Xem options server hỗ trợ.
- Xem compiled defaults.
- Xem danh sách option files được tìm theo thứ tự trên máy hiện tại.

### 5.3. Server option, system variable và status variable

| Khái niệm | Ví dụ | Ý nghĩa |
|---|---|---|
| Server option | `--port=3306` | Option khi khởi động `mysqld` |
| System variable | `max_connections`, `sql_mode` | Thiết lập ảnh hưởng server hoặc session |
| Status variable | `Threads_connected`, `Questions` | Số liệu trạng thái quan sát được |

Ví dụ:

```sql
SHOW VARIABLES LIKE 'port';
```

```sql
SHOW VARIABLES LIKE 'max_connections';
```

```sql
SHOW STATUS LIKE 'Threads_connected';
```

```sql
SHOW STATUS LIKE 'Uptime';
```

Một số system variable có thể dynamic, một số chỉ thay đổi khi startup. Cần xem tài liệu đúng version trước khi thay đổi.

### 5.4. Kiểm tra server còn phản hồi

Nếu `mysqladmin` có sẵn:

```bash
mysqladmin -u root -p version
```

Hoặc:

```bash
mysqladmin -u root -p ping
```

Từ SQL client:

```sql
SELECT VERSION() AS version,
       NOW() AS serverTime,
       CONNECTION_ID() AS connectionId;
```

### Bài tập thực hành

**Bài 5.1.** Chạy `mysqld --help` hoặc xác định lý do máy của bạn không tìm được lệnh này.

**Bài 5.2.** Dùng `SHOW VARIABLES LIKE 'port';` để xem port server.

**Bài 5.3.** Dùng `SHOW VARIABLES LIKE 'max_connections';`.

**Bài 5.4.** Dùng `SHOW STATUS LIKE 'Threads_connected';` và giải thích ý nghĩa.

**Bài 5.5.** Dùng `mysqladmin ... ping` hoặc `SELECT VERSION()` để xác minh server phản hồi.

---

## 6. Những thao tác nào thuộc về client, service manager và mysqld?

| Nhu cầu | Công cụ phù hợp |
|---|---|
| Gửi SQL query | `mysql`, Workbench, application connector |
| Xem metadata | SQL `SHOW ...`, `INFORMATION_SCHEMA` |
| Start/stop/restart service | Windows Service Control Manager, `systemctl`, package/service manager |
| Xem server options | option file, `mysqld --verbose --help`, SQL variables |
| Backup logical | `mysqldump` |
| Kiểm tra server phản hồi | `mysqladmin ping`, `mysqladmin version`, SQL connection test |

Ví dụ, `SHOW DATABASES;` không start server. Nó chỉ chạy được sau khi server đang hoạt động và client đã kết nối.

### Bài tập thực hành

**Bài 6.1.** Ghép mỗi nhiệm vụ với một công cụ phù hợp: backup, query, restart service, check connection.

**Bài 6.2.** Giải thích vì sao `SHOW DATABASES` không thể dùng khi `mysqld` đã dừng.

**Bài 6.3.** Nêu một tình huống nên dùng `mysqladmin ping`.

**Bài 6.4.** Nêu một tình huống nên dùng `mysqldump`.

**Bài 6.5.** Nêu một lý do không nên chạy `mysqld` trực tiếp bằng command line trên máy đã được package/service manager quản lý, trừ khi hiểu rõ cách cài đặt.

---

## 7. Lỗi nhận thức thường gặp

### Lỗi 1. Nhầm Workbench là database server

Workbench là GUI client/tool. `mysqld` mới là server process.

### Lỗi 2. Nhầm `mysql` client với `mysqld`

`mysql` gửi lệnh đến server; `mysqld` nhận và thực thi.

### Lỗi 3. Nghĩ mọi database là một file `.sql`

Database đang chạy không phải là file `.sql`. `.sql` thường là script/dump; data directory chứa các file nội bộ do server quản lý.

### Lỗi 4. Tự xóa file data để “reset database”

Không xóa file trực tiếp. Dùng SQL `DROP DATABASE` trong lab hoặc quy trình backup/restore/migration được phê duyệt.

### Lỗi 5. Đổi random server variable để “tăng tốc”

Mỗi option có trade-off về memory, concurrency, durability và workload. Không copy tuning values từ Internet sang production.

### Bài tập thực hành

**Bài 7.1.** Sửa phát biểu sai: “MySQL Workbench là process quản lý data directory.”

**Bài 7.2.** Sửa phát biểu sai: “mysql client chính là MySQL Server.”

**Bài 7.3.** Nêu cách an toàn để xóa một database lab.

**Bài 7.4.** Nêu hai yếu tố cần đánh giá trước khi đổi `max_connections`.

**Bài 7.5.** Giải thích vì sao data directory không được xem như một thư mục tài liệu thông thường.

---

## 8. Bài tập tổng hợp

### Bài 8.1. Kiểm tra server cơ bản

Viết các câu SQL để:

1. Xem MySQL version.
2. Xem user và privilege account hiện tại.
3. Xem port.
4. Xem `Threads_connected`.
5. Xem available storage engines.

### Bài 8.2. Luồng xử lý query

Với query join giữa `orders` và `customers`:

1. Mô tả bước client gửi query.
2. Nêu vai trò parser.
3. Nêu vai trò optimizer.
4. Nêu vai trò storage engine.
5. Nêu vai trò executor khi trả result set.

### Bài 8.3. Phân biệt công cụ

Phân loại `mysqld`, `mysql`, MySQL Workbench, `mysqldump`, `mysqladmin` theo: server, client GUI, client CLI, backup utility, admin utility.

### Bài 8.4. Kiểm tra system/status variables

Chạy và giải thích:

```sql
SHOW VARIABLES LIKE 'datadir';
SHOW VARIABLES LIKE 'port';
SHOW STATUS LIKE 'Uptime';
SHOW STATUS LIKE 'Threads_connected';
SHOW ENGINES;
```

### Bài 8.5. Đánh giá một yêu cầu thay đổi

Một bạn đề xuất: “Tăng `max_connections` từ 151 lên 5000 để hết lỗi connection.”

1. Nêu vì sao chưa thể đồng ý ngay.
2. Nêu ba số liệu hoặc thông tin cần thu thập.
3. Nêu một tác động có thể có về RAM.
4. Nêu một cách kiểm tra hiện tượng connection hiện tại.
5. Nêu một bước an toàn trước khi áp dụng đổi config.

---

## 9. Đáp án gợi ý

### Bài 8.1

```sql
SELECT VERSION() AS mysqlVersion;

SELECT USER() AS loginUser,
       CURRENT_USER() AS privilegeAccount;

SHOW VARIABLES LIKE 'port';

SHOW STATUS LIKE 'Threads_connected';

SHOW ENGINES;
```

### Bài 8.4

```sql
SHOW VARIABLES LIKE 'datadir';
SHOW VARIABLES LIKE 'port';
SHOW STATUS LIKE 'Uptime';
SHOW STATUS LIKE 'Threads_connected';
SHOW ENGINES;
```

### Bài 8.5 — hướng trả lời

Không tăng `max_connections` chỉ vì thấy lỗi connection. Cần xem số connection hiện có, peak workload, application pooling, error log, memory budget và nhu cầu thực. Việc tăng giới hạn có thể tăng memory pressure vì mỗi connection có overhead riêng. Trước khi đổi config, cần backup config, thử ở lab/staging, có kế hoạch restart/rollback và kiểm tra sau thay đổi.

---

## 10. Tóm tắt

- `mysqld` là MySQL Server process.
- `mysql` và Workbench là client; `mysqldump` là backup utility.
- Luồng query đi từ client → connection/authentication → SQL layer → storage engine → result set.
- Storage engines quản lý lưu trữ/truy xuất; InnoDB thường phù hợp cho workload giao dịch.
- Server options thường được giữ ổn định qua option file; system/status variables giúp quan sát/cấu hình runtime.
- Data directory chứa server-managed data và files; không tự thao tác file trực tiếp.
- Các hoạt động service lifecycle, configuration và data directory phải được tách khỏi thao tác SQL thông thường và thực hiện cẩn trọng.

---

## 11. Từ khóa chính

- MySQL Server
- mysqld
- mysql client
- MySQL Workbench
- SQL layer
- Query parser
- Optimizer
- Executor
- Storage engine
- InnoDB
- Data directory
- Server option
- System variable
- Status variable
- mysqladmin
- mysqldump
- Least privilege
