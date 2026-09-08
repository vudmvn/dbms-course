---
title: "Tutorial 3: MySQL Configuration File và Data Directory"
author: "Tên giảng viên"
duration: "150m"
difficulty: "Intermediate"
prerequisites:
  - "Đã biết mysqld và start/stop/restart MySQL Server"
  - "Có quyền đọc configuration/service metadata trên máy lab"
  - "Có một môi trường lab hoặc test để thực hành thay đổi config"
summary: "Hướng dẫn tìm, đọc và chỉnh sửa an toàn MySQL option file; hiểu option groups, precedence, system variables và data directory; kiểm tra cấu hình/runtime trước và sau restart."
---

# Tutorial 3: MySQL Configuration File và Data Directory

## Link tham khảo

- [MySQL Tutorial — MySQL Configuration File](https://www.mysqltutorial.org/mysql-administration/mysql-configuration-file/)
- [MySQL Tutorial — MySQL Data Directory](https://www.mysqltutorial.org/mysql-administration/mysql-data-directory/)
- [MySQL 8.4 Reference Manual — Using Option Files](https://dev.mysql.com/doc/refman/8.4/en/option-files.html)
- [MySQL 8.4 Reference Manual — Server Command Options](https://dev.mysql.com/doc/refman/8.4/en/server-options.html)
- [MySQL 8.4 Reference Manual — The MySQL Data Directory](https://dev.mysql.com/doc/refman/8.4/en/data-directory.html)
- [MySQL 8.4 Reference Manual — Server Configuration Validation](https://dev.mysql.com/doc/refman/8.4/en/server-configuration-validation.html)
- [MySQL 8.4 Reference Manual — System Variables](https://dev.mysql.com/doc/refman/8.4/en/server-system-variables.html)

> **Cảnh báo:** Sai một option file có thể khiến MySQL không khởi động. Không chỉnh `my.ini`, `my.cnf`, `datadir`, permissions hoặc file trong data directory trên production nếu chưa có backup config, change plan, maintenance window và rollback plan.

---

## 1. Mục tiêu học tập

Sau khi hoàn thành tutorial này, người học có thể:

1. Phân biệt option file với SQL script.
2. Nêu được vai trò của `my.ini` và `my.cnf`.
3. Dùng `mysqld --verbose --help` để xem các option files mà server tìm trên máy hiện tại.
4. Hiểu option group như `[mysqld]`, `[server]`, `[client]`, `[mysqldump]`.
5. Hiểu nguyên tắc option xuất hiện sau có thể ghi đè option xuất hiện trước.
6. Đọc các cấu hình phổ biến: `port`, `bind-address`, `datadir`, `max_connections`, `max_allowed_packet`, `log_error`.
7. Kiểm tra giá trị runtime bằng `SHOW VARIABLES`.
8. Xác định MySQL data directory an toàn bằng SQL.
9. Giải thích nội dung khái quát trong data directory và lý do không thao tác file trực tiếp.
10. Lập quy trình an toàn khi thay đổi config hoặc chuẩn bị di chuyển data directory.

---

## 2. Option file là gì?

Option file là file cấu hình văn bản mà MySQL programs đọc khi khởi động. Tên file thường gặp:

```text
my.ini
my.cnf
```

Option file không phải SQL script.

| Loại file | Ví dụ | Dùng để |
|---|---|---|
| SQL script | `setup.sql` | Chạy `CREATE TABLE`, `INSERT`, `SELECT` |
| Option file | `my.ini`, `my.cnf` | Cấu hình `mysqld`, client tools và utility |
| Dump file | `backup.sql` | Logical backup/restore |
| Service unit | `.service` | Service manager quản lý process |

Ví dụ option file:

```ini
[mysqld]
port=3306
max_connections=150
```

Không chạy file trên bằng:

```sql
SOURCE my.ini;
```

### 2.1. Cấu trúc option group

Các options được đặt trong nhóm:

```ini
[mysqld]
port=3306
```

Nhóm cho biết program nào đọc options đó.

| Group | Program/ý nghĩa thường gặp |
|---|---|
| `[mysqld]` | MySQL Server process |
| `[server]` | Server programs phù hợp |
| `[client]` | Các standard client programs |
| `[mysql]` | mysql command-line client |
| `[mysqldump]` | mysqldump utility |

Ví dụ:

```ini
[client]
port=3306

[mysqld]
port=3306
max_connections=150

[mysqldump]
quick
```

### 2.2. Option form

Một option thường có dạng:

```ini
option_name=value
```

Ví dụ:

```ini
port=3306
max_allowed_packet=64M
```

Nhiều server command options có dạng `--option-name` trên command line nhưng dùng `option-name` hoặc `option_name` trong option file tùy option/cú pháp MySQL. Hãy dùng tên chính xác theo documentation của version đang chạy.

### Bài tập thực hành

**Bài 2.1.** Phân biệt `my.ini` với `setup.sql`.

**Bài 2.2.** Viết một option group `[mysqld]` có `port=3306`.

**Bài 2.3.** Nêu mục đích của group `[client]`.

**Bài 2.4.** Nêu mục đích của group `[mysqldump]`.

**Bài 2.5.** Giải thích vì sao không thể chạy `SOURCE my.cnf;` trong MySQL client để áp dụng config.

---

## 3. Tìm configuration file đang được dùng

### 3.1. Dùng `mysqld --verbose --help`

Trong terminal/Command Prompt, chạy:

```bash
mysqld --verbose --help
```

Tìm phần tương tự:

```text
Default options are read from the following files in the given order:
...
```

Đây là cách đáng tin cậy hơn việc chỉ ghi nhớ một path cố định, vì đường dẫn phụ thuộc:

- Hệ điều hành.
- Cách cài MySQL.
- Version/package.
- Service configuration.
- Command-line options như `--defaults-file`.

### 3.2. Vị trí phổ biến trên Windows

Với MySQL Installer trên Windows, một vị trí thường gặp là:

```text
C:\ProgramData\MySQL\MySQL Server 8.x\my.ini
```

Nhưng không giả định path này luôn đúng. Dùng:

```cmd
mysqld --verbose --help
```

hoặc xem service configuration:

```cmd
sc qc MySQL80
```

Nếu service name khác, thay `MySQL80` bằng name thực tế.

### 3.3. Vị trí phổ biến trên Linux

Tùy distribution/package, các file hay gặp gồm:

```text
/etc/my.cnf
/etc/mysql/my.cnf
/etc/mysql/mysql.conf.d/mysqld.cnf
```

Không phải mọi file đều được đọc trong mọi cách cài đặt. Cần xác minh bằng `mysqld --verbose --help` và service/package docs.

### 3.4. Service command có thể ghi đè file defaults

Một service manager có thể khởi động `mysqld` với command-line options hoặc environment settings. Options từ command line có thể ảnh hưởng giá trị cuối cùng.

Do đó, khi debug “đã sửa config nhưng không có hiệu lực”, cần kiểm tra:

1. Đúng file chưa?
2. Đúng group `[mysqld]` chưa?
3. Có file option khác đọc sau ghi đè không?
4. Service có command-line override không?
5. Option có cần restart không?
6. Option name/value có hợp lệ ở version hiện tại không?

### Bài tập thực hành

**Bài 3.1.** Chạy `mysqld --verbose --help` và tìm danh sách default option files.

**Bài 3.2.** Trên Windows, chạy `sc qc <service_name>` để xem service configuration.

**Bài 3.3.** Trên Linux, liệt kê các file cấu hình phổ biến trong `/etc/mysql/` nếu thư mục tồn tại.

**Bài 3.4.** Nêu ba lý do đã sửa file config nhưng runtime value chưa thay đổi.

**Bài 3.5.** Giải thích vì sao không nên chỉ dựa vào một path config “mặc định” từ Internet.

---

## 4. Precedence và option groups

### 4.1. Options được đọc theo thứ tự

MySQL programs có thể đọc nhiều option files/groups. Khi cùng một option được chỉ định nhiều lần, option đọc sau có thể override option đọc trước.

Ví dụ minh họa:

```ini
[client]
port=3306

[mysqldump]
port=3307
```

`mysqldump` có thể dùng option riêng trong `[mysqldump]` thay vì general setting trong `[client]`.

### 4.2. General group trước, specific group sau

Một cách tổ chức dễ hiểu:

```ini
[client]
port=3306

[mysqldump]
quick
```

Tương tự, trong file lớn nên:

- Giữ option groups rõ ràng.
- Không lặp option vô lý.
- Ghi comment cho thay đổi quan trọng.
- Chia file bằng include nếu policy cho phép.
- Không lưu password ở file có quyền đọc rộng.

### 4.3. `!include` và `!includedir`

Option file có thể include file khác:

```ini
!include /home/mydir/myopt.cnf
```

Hoặc include directory:

```ini
!includedir /home/mydir
```

Khi dùng include directory, cần biết thứ tự file có thể không nên được giả định một cách tùy tiện; đặt tên file rõ ràng và theo policy của package/distribution.

### 4.4. Password trong client option file

`[client]` có thể chứa connection options, nhưng file có password phải được bảo vệ để người dùng khác không đọc được.

Trong lab, không lưu password thật vào file share/public repository.

### Bài tập thực hành

**Bài 4.1.** Giải thích precedence khi cùng một option xuất hiện ở nhiều nơi.

**Bài 4.2.** Viết ví dụ option file có `[client]`, `[mysqld]`, `[mysqldump]`.

**Bài 4.3.** Nêu mục đích của `!include`.

**Bài 4.4.** Nêu rủi ro khi lưu password trong `[client]`.

**Bài 4.5.** Đề xuất một cách giảm nhầm lẫn khi một server dùng nhiều file config.

---

## 5. Các options thường gặp trong `[mysqld]`

### 5.1. `port`

```ini
[mysqld]
port=3306
```

Runtime check:

```sql
SHOW VARIABLES LIKE 'port';
```

`3306` là port mặc định phổ biến, không phải bắt buộc.

### 5.2. `bind-address`

Ví dụ:

```ini
[mysqld]
bind-address=127.0.0.1
```

Ý nghĩa khái quát: giới hạn địa chỉ network mà server lắng nghe.

Không đổi `bind-address` để mở remote access nếu chưa xem xét:

- Firewall.
- Host-based account grants.
- TLS/encrypted connections.
- Network segmentation.
- Exposure risk.

### 5.3. `max_connections`

```ini
[mysqld]
max_connections=150
```

Runtime check:

```sql
SHOW VARIABLES LIKE 'max_connections';
SHOW STATUS LIKE 'Threads_connected';
SHOW STATUS LIKE 'Max_used_connections';
```

Không tăng tùy tiện. Số connection lớn hơn có thể tăng memory demand và che giấu vấn đề connection leak/pooling.

### 5.4. `max_allowed_packet`

```ini
[mysqld]
max_allowed_packet=64M
```

Runtime check:

```sql
SHOW VARIABLES LIKE 'max_allowed_packet';
```

Option này ảnh hưởng kích thước packet/message MySQL được phép xử lý. Chỉ thay đổi khi hiểu nguyên nhân lỗi và tác động tới memory/workload.

### 5.5. `datadir`

```ini
[mysqld]
datadir=/path/to/mysql-data
```

`datadir` xác định data directory server. Đây là option nhạy cảm; không đổi tùy tiện.

Runtime check:

```sql
SHOW VARIABLES LIKE 'datadir';
```

### 5.6. Error log option

Ví dụ option có thể là:

```ini
[mysqld]
log_error=/path/to/mysql-error.log
```

Đường dẫn/phương thức log cụ thể phụ thuộc platform, package và MySQL version.

Khi server đang chạy, có thể thử:

```sql
SHOW VARIABLES LIKE 'log_error';
```

Nếu cần xác định chính xác log path trong môi trường của bạn, kết hợp:

- Runtime variables.
- Service config.
- OS logs/journal.
- Installer/package documentation.

### Bài tập thực hành

**Bài 5.1.** Dùng `SHOW VARIABLES` kiểm tra `port`.

**Bài 5.2.** Dùng `SHOW VARIABLES` kiểm tra `max_connections`.

**Bài 5.3.** Dùng `SHOW STATUS` kiểm tra `Max_used_connections`.

**Bài 5.4.** Dùng `SHOW VARIABLES` kiểm tra `datadir`.

**Bài 5.5.** Nêu hai rủi ro khi tăng `max_connections` mà không đo workload.

---

## 6. Quy trình chỉnh config an toàn

Không sửa config kiểu “mở file, đổi số, restart”.

### 6.1. Quy trình đề xuất

```text
1. Xác định yêu cầu và option cần thay đổi.
2. Xem documentation đúng version.
3. Kiểm tra runtime value hiện tại.
4. Xác định option dynamic hay startup-only.
5. Sao lưu option file.
6. Thay đổi tối thiểu, có comment/change record.
7. Validate config nếu tool/version hỗ trợ.
8. Restart có kế hoạch nếu cần.
9. Kiểm tra service, error log và runtime value.
10. Roll back file cũ nếu server không lên hoặc behavior không mong muốn.
```

### 6.2. Ví dụ thay đổi có kiểm soát

Giả sử lab cần đổi `max_allowed_packet`.

1. Kiểm tra hiện tại:

```sql
SHOW VARIABLES LIKE 'max_allowed_packet';
```

2. Sao lưu config:

Windows PowerShell:

```powershell
Copy-Item "C:\path\to\my.ini" "C:\path\to\my.ini.bak"
```

Linux:

```bash
sudo cp /etc/mysql/my.cnf /etc/mysql/my.cnf.bak
```

3. Sửa đúng `[mysqld]`:

```ini
[mysqld]
max_allowed_packet=64M
```

4. Restart đúng service.

5. Kiểm tra lại:

```sql
SHOW VARIABLES LIKE 'max_allowed_packet';
```

6. Đọc error log nếu server không start.

### 6.3. Không chỉnh nhiều option cùng lúc khi học

Nếu đổi cùng lúc port, datadir, bind-address, max connections và logging, khi lỗi xảy ra sẽ khó biết nguyên nhân.

Trong lab, đổi một option nhỏ, xác minh, sau đó mới sang option khác.

### Bài tập thực hành

**Bài 6.1.** Viết checklist sáu bước trước/sau thay đổi config.

**Bài 6.2.** Viết command backup file `my.ini` trên Windows PowerShell.

**Bài 6.3.** Viết command backup file config trên Linux.

**Bài 6.4.** Nêu lý do nên đổi một option tại một thời điểm trong lab.

**Bài 6.5.** Nêu hai cách kiểm tra config change đã có hiệu lực.

---

## 7. MySQL Data Directory

### 7.1. Data directory là gì?

Data directory là directory nơi MySQL Server quản lý nhiều thông tin quan trọng của instance.

Theo khái quát, nó có thể chứa:

- Subdirectories tương ứng database/schema.
- `mysql` system schema.
- `performance_schema` và `sys` liên quan metadata/runtime support.
- InnoDB tablespaces và log files.
- Server logs được cấu hình tại/ngoài directory.
- Data dictionary và các server-managed files khác.

`INFORMATION_SCHEMA` là schema chuẩn nhưng không có database directory tương ứng theo cùng cách như schema thông thường.

### 7.2. Tìm data directory từ SQL

```sql
SHOW VARIABLES LIKE 'datadir';
```

Hoặc:

```sql
SELECT @@datadir AS dataDirectory;
```

Ví dụ output trên Linux có thể là:

```text
/var/lib/mysql/
```

Ví dụ output trên Windows phụ thuộc installation/configuration.

Không ghi nhớ path mẫu như một sự thật tuyệt đối. Hãy hỏi server đang chạy.

### 7.3. Khám phá an toàn ở mức read-only

Sau khi lấy path từ SQL, có thể chỉ liệt kê để quan sát trên máy lab.

Linux:

```bash
sudo ls -lah /var/lib/mysql
```

Windows PowerShell, sau khi thay path đúng:

```powershell
Get-ChildItem -Force "C:\path\to\mysql-data"
```

Không:

- Xóa file.
- Đổi tên file.
- Copy file hot từ data directory để coi là backup.
- Sửa quyền tùy tiện.
- Dùng editor mở/chỉnh binary files.
- Chép data directory giữa server versions mà không có migration plan.

### 7.4. Database directory và schema

Trong data directory, các schema user-created thường có representation dưới dạng directories/files do server quản lý. Tuy nhiên, không dùng filesystem để quản lý database.

Để tạo/xóa database, dùng SQL:

```sql
CREATE DATABASE lab_db;
```

```sql
DROP DATABASE lab_db;
```

Không dùng:

```text
rm -rf /var/lib/mysql/lab_db
```

### Bài tập thực hành

**Bài 7.1.** Dùng `SELECT @@datadir;` để tìm data directory.

**Bài 7.2.** Dùng lệnh read-only của OS để liệt kê nội dung directory trên máy lab.

**Bài 7.3.** Nêu ba loại thành phần có thể tồn tại trong data directory.

**Bài 7.4.** Giải thích vì sao `INFORMATION_SCHEMA` khác các schema có directory vật lý theo cách thông thường.

**Bài 7.5.** Viết SQL tạo và xóa một database lab, không dùng thao tác filesystem.

---

## 8. Di chuyển data directory: nguyên tắc, không phải recipe one-size-fits-all

Di chuyển data directory là một thao tác quản trị nâng cao vì liên quan:

- Server shutdown.
- Consistent backup/copy.
- `datadir` option.
- Ownership và filesystem permissions.
- SELinux/AppArmor policy trên Linux.
- Windows service account permissions.
- InnoDB/system tablespace/log configuration.
- Server version và package behavior.
- Rollback plan.

### 8.1. Quy trình khái quát

```text
1. Đọc documentation đúng version/package.
2. Tạo backup đã kiểm tra restore.
3. Xác định data directory hiện tại bằng runtime variable.
4. Stop đúng MySQL service theo graceful shutdown.
5. Copy/migrate data theo phương pháp được hỗ trợ.
6. Cập nhật datadir và OS security/permissions.
7. Start server và đọc error log ngay.
8. Xác minh schema, table, data và application.
9. Giữ directory cũ cho đến khi rollback window kết thúc.
```

### 8.2. Remote mounts và systemd

Với systemd, cần đặc biệt cẩn thận nếu data directory là remote mount. Nếu mount biến mất và mount point trông như empty directory, cách service/package xử lý data directory cần được hiểu rõ trước khi start server.

Không dùng remote storage cho data directory chỉ vì “dễ copy file” mà không xem xét durability, latency, mounting, failure mode và recommendation của vendor.

### 8.3. Không coi copy file đang chạy là backup

Copy file trực tiếp trong data directory khi `mysqld` đang chạy không mặc định tạo backup nhất quán.

Dùng:

- `mysqldump` logical backup.
- Physical backup tool/quy trình được hỗ trợ.
- Storage snapshot được thiết kế nhất quán cho MySQL.
- Runbook backup/restore của tổ chức.

### Bài tập thực hành

**Bài 8.1.** Nêu năm yếu tố cần kiểm tra trước khi di chuyển data directory.

**Bài 8.2.** Giải thích vì sao cần graceful stop trước thao tác migrate file.

**Bài 8.3.** Nêu lý do OS permissions có thể khiến server không start sau khi đổi datadir.

**Bài 8.4.** Giải thích vì sao không nên copy data directory đang chạy để coi như backup.

**Bài 8.5.** Nêu một lý do remote mount làm data directory phức tạp hơn.

---

## 9. Kiểm tra runtime configuration qua SQL

Các câu lệnh chỉ đọc hữu ích:

```sql
SHOW VARIABLES LIKE 'basedir';
SHOW VARIABLES LIKE 'datadir';
SHOW VARIABLES LIKE 'port';
SHOW VARIABLES LIKE 'socket';
SHOW VARIABLES LIKE 'max_connections';
SHOW VARIABLES LIKE 'max_allowed_packet';
SHOW VARIABLES LIKE 'bind_address';
```

Một số variables có thể khác theo version/platform, vì vậy nếu một variable không tồn tại hoặc có tên khác, dùng:

```sql
SHOW VARIABLES LIKE '%dir%';
```

```sql
SHOW VARIABLES LIKE '%log%';
```

Xem giá trị bằng `SELECT`:

```sql
SELECT @@version AS version,
       @@port AS port,
       @@datadir AS dataDirectory,
       @@max_connections AS maxConnections,
       @@max_allowed_packet AS maxAllowedPacket;
```

### 9.1. Runtime value không tự nói lên nguồn config

Nếu `@@port = 3306`, giá trị đó có thể đến từ:

- Compiled default.
- Option file.
- Command line.
- Persisted configuration.
- Package/service default.

Khi cần biết “file nào đã áp dụng value này”, phải kiểm tra option files, service command và cách installation quản lý config.

### Bài tập thực hành

**Bài 9.1.** Chạy `SHOW VARIABLES LIKE 'datadir';`.

**Bài 9.2.** Chạy `SHOW VARIABLES LIKE '%dir%';`.

**Bài 9.3.** Viết `SELECT` trả về version, port, datadir và max connections.

**Bài 9.4.** Giải thích vì sao runtime value không luôn cho biết file config nguồn.

**Bài 9.5.** Nêu ba nguồn có thể thiết lập giá trị server option.

---

## 10. Một số lỗi thường gặp

### Lỗi 1. Sửa `[client]` thay vì `[mysqld]`

Nếu muốn đổi server port nhưng sửa:

```ini
[client]
port=3307
```

thì client có thể cố kết nối port 3307, còn server vẫn chạy port cũ.

Để đổi server port cần kiểm tra:

```ini
[mysqld]
port=3307
```

và update client/application config tương ứng.

### Lỗi 2. Sửa một file config không được server đọc

Không thấy hiệu lực sau restart có thể do sửa sai file. Dùng:

```bash
mysqld --verbose --help
```

và service configuration để xác minh.

### Lỗi 3. Đặt option sai group hoặc sai tên

Một option không hợp lệ có thể khiến server không start hoặc bị bỏ qua tùy context. Đọc error log sau restart.

### Lỗi 4. Đổi `datadir` nhưng quên permissions/security policy

Data directory mới phải có ownership/permissions và security labeling/policy phù hợp với service account và OS.

### Lỗi 5. Chỉnh trực tiếp file data

Không dùng text editor để chỉnh `.ibd`, redo log, dictionary file hoặc data files.

### Lỗi 6. Copy config từ Internet không theo version

Một option hợp lệ ở version cũ có thể deprecated, removed hoặc khác behavior ở version mới.

### Bài tập thực hành

**Bài 10.1.** Sửa ví dụ config đặt `port` nhầm trong `[client]`.

**Bài 10.2.** Nêu command giúp xác minh option files được tìm.

**Bài 10.3.** Nêu ba việc cần làm khi server không start sau config edit.

**Bài 10.4.** Giải thích vì sao đổi data directory cần kiểm tra permissions.

**Bài 10.5.** Nêu lý do không được copy tuning config từ version khác mà không đọc docs.

---

## 11. Bài tập tổng hợp

### Bài 11.1. Khám phá configuration

1. Xác định version MySQL.
2. Xem `port`, `datadir`, `max_connections`, `max_allowed_packet`.
3. Dùng `mysqld --verbose --help` để xác định option-file search path.
4. Xác định service manager đang dùng.
5. Đưa ra bảng gồm runtime value và cách kiểm tra nguồn config.

### Bài 11.2. Thay đổi config có kiểm soát

Giả sử cần đổi `max_allowed_packet` trong lab:

1. Kiểm tra giá trị hiện tại.
2. Backup config.
3. Nêu đúng option group cần sửa.
4. Nêu cách restart service theo Windows hoặc Linux.
5. Nêu SQL kiểm tra sau restart.

### Bài 11.3. Data directory audit

1. Lấy `@@datadir`.
2. Liệt kê read-only nội dung directory trên máy lab.
3. Nêu ba object/file category có thể thấy.
4. Nêu ba thao tác không được thực hiện trực tiếp.
5. Nêu hai phương pháp backup phù hợp hơn copy file nóng.

### Bài 11.4. Debug “config không có hiệu lực”

Một admin sửa `my.ini`, restart MySQL, nhưng `@@port` không đổi.

1. Nêu năm nguyên nhân có thể.
2. Viết hai lệnh cần chạy.
3. Nêu file/group cần kiểm tra.
4. Nêu log cần đọc.
5. Nêu rollback plan tối thiểu.

### Bài 11.5. Di chuyển data directory

Viết checklist 10 bước khái quát, từ backup đã test đến rollback window sau migration.

---

## 12. Đáp án gợi ý

### Bài 5.1 đến Bài 5.4

```sql
SHOW VARIABLES LIKE 'port';
SHOW VARIABLES LIKE 'max_connections';
SHOW STATUS LIKE 'Max_used_connections';
SHOW VARIABLES LIKE 'datadir';
```

### Bài 6.2 — Windows PowerShell

```powershell
Copy-Item "C:\path\to\my.ini" "C:\path\to\my.ini.bak"
```

### Bài 6.3 — Linux

```bash
sudo cp /etc/mysql/my.cnf /etc/mysql/my.cnf.bak
```

### Bài 7.1

```sql
SELECT @@datadir AS dataDirectory;
```

### Bài 7.5

```sql
CREATE DATABASE config_lab;

DROP DATABASE config_lab;
```

### Bài 9.3

```sql
SELECT @@version AS version,
       @@port AS port,
       @@datadir AS dataDirectory,
       @@max_connections AS maxConnections,
       @@max_allowed_packet AS maxAllowedPacket;
```

### Bài 11.2 — workflow

```sql
SHOW VARIABLES LIKE 'max_allowed_packet';
```

Sau khi backup config và thay đổi đúng `[mysqld]`, restart đúng service instance, rồi kiểm tra:

```sql
SHOW VARIABLES LIKE 'max_allowed_packet';
```

Nếu server không start, restore config backup và đọc error log trước khi thử thay đổi khác.

---

## 13. Tóm tắt

- `my.ini`/`my.cnf` là option file, không phải SQL script.
- Dùng `mysqld --verbose --help` để biết server tìm option files ở đâu trên máy hiện tại.
- `[mysqld]` áp dụng cho server; `[client]` áp dụng cho clients; `[mysqldump]` áp dụng cho mysqldump.
- Options đọc sau có thể override options đọc trước.
- Kiểm tra runtime bằng `SHOW VARIABLES` và `SELECT @@variable`.
- `datadir` là server-managed directory chứa nhiều thành phần quan trọng; không thao tác file trực tiếp.
- Di chuyển data directory là task nâng cao: cần backup đã kiểm tra, service stop, config, permissions, security policy, error-log review và rollback plan.
- Thay đổi config phải tối thiểu, có backup và có cách xác minh sau restart.

---

## 14. Từ khóa chính

- my.ini
- my.cnf
- Option file
- Option group
- [mysqld]
- [server]
- [client]
- [mysqldump]
- Option precedence
- !include
- !includedir
- port
- bind-address
- max_connections
- max_allowed_packet
- datadir
- basedir
- log_error
- SHOW VARIABLES
- Data directory
- InnoDB tablespace
- permissions
- SELinux
- AppArmor
- rollback plan
