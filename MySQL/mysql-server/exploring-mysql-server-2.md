---
title: "Tutorial 2: Quản lý vòng đời MySQL Server — Start, Stop và Restart"
author: "Tên giảng viên"
duration: "120m"
difficulty: "Beginner–Intermediate"
prerequisites:
  - "Đã học kiến trúc MySQL cơ bản và vai trò của mysqld"
  - "Có quyền administrator trên máy lab hoặc quyền sudo/service management phù hợp"
summary: "Hướng dẫn kiểm tra, start, stop và restart MySQL Server trên Windows và Linux; xác minh server sau thay đổi; đọc log cơ bản và tránh thao tác nguy hiểm trên môi trường dùng chung."
---

# Tutorial 2: Quản lý vòng đời MySQL Server — Start, Stop và Restart

## Link tham khảo

- [MySQL Tutorial — Start MySQL Server](https://www.mysqltutorial.org/mysql-administration/start-mysql/)
- [MySQL Tutorial — Stop MySQL Server](https://www.mysqltutorial.org/mysql-administration/stop-mysql-server/)
- [MySQL Tutorial — Restart MySQL Server](https://www.mysqltutorial.org/mysql-administration/restart-mysql-server/)

> **Cảnh báo:** Start/stop/restart làm thay đổi trạng thái service và có thể ngắt ứng dụng, query hoặc transaction. Chỉ thực hành trên máy cá nhân/lab. Trên production, phải theo runbook, maintenance window và quy trình kiểm tra phù hợp.

---

## 1. Mục tiêu học tập

Sau khi hoàn thành tutorial này, người học có thể:

1. Kiểm tra MySQL Server có đang chạy hay không.
2. Phân biệt service name với display name trên Windows/Linux.
3. Start, stop và restart MySQL service bằng Windows Services, `sc`, PowerShell hoặc `systemctl`.
4. Xác minh server đáp ứng sau thao tác bằng SQL hoặc `mysqladmin`.
5. Đọc trạng thái service và error log cơ bản để chẩn đoán start failure.
6. Phân biệt graceful shutdown với việc ép process dừng.
7. Biết khi nào cần restart, khi nào không nên restart.
8. Áp dụng checklist an toàn trước/sau restart.

---

## 2. Khi nào cần start, stop hoặc restart?

| Thao tác | Mục đích phổ biến |
|---|---|
| Start | Server/service đang dừng và cần đưa vào hoạt động |
| Stop | Shutdown có kiểm soát để bảo trì, nâng cấp hoặc chuyển máy |
| Restart | Áp dụng config startup-only, khắc phục service bị kẹt theo runbook, hoặc sau maintenance |
| Không cần restart | Chỉ chạy SQL query, tạo database/table, hoặc thay đổi một dynamic variable phù hợp |

### 2.1. Restart không phải giải pháp mặc định

Không restart chỉ vì:

- Một query chậm.
- Một user không đăng nhập được.
- Một ứng dụng gặp lỗi không rõ nguyên nhân.
- Thấy `Sleep` connections.
- Muốn “làm mới database”.

Trước khi restart, cần xác định:

- Có bao nhiêu application đang kết nối?
- Có transaction quan trọng không?
- Có backup/migration đang chạy không?
- Error log có nói gì?
- Có thay đổi config nào cần áp dụng không?

### 2.2. Graceful shutdown

Khi stop qua service manager hoặc `mysqladmin shutdown` theo cách phù hợp, server có cơ hội đóng kết nối, flush/hoàn tất các bước shutdown nội bộ theo quy trình của MySQL.

Không dùng Task Manager `End task`, `kill -9`, hoặc tắt máy đột ngột trừ tình huống khẩn cấp đã có quy trình xử lý. Những cách này có thể dẫn đến crash recovery kéo dài sau khi server khởi động lại.

### Bài tập thực hành

**Bài 2.1.** Nêu một tình huống phù hợp để restart MySQL Server.

**Bài 2.2.** Nêu hai tình huống không nên restart ngay.

**Bài 2.3.** Giải thích vì sao graceful shutdown tốt hơn ép process dừng.

**Bài 2.4.** Nêu ba điều cần kiểm tra trước restart trên môi trường dùng chung.

**Bài 2.5.** Phân biệt start và restart.

---

## 3. Kiểm tra MySQL Server trước khi thao tác

### 3.1. Kiểm tra từ SQL client

Nếu đã kết nối được:

```sql
SELECT VERSION() AS mysqlVersion,
       NOW() AS serverTime,
       CONNECTION_ID() AS connectionId;
```

Nếu query trả kết quả, server đang phản hồi với session hiện tại.

### 3.2. Kiểm tra bằng `mysqladmin`

```bash
mysqladmin -u root -p ping
```

Hoặc:

```bash
mysqladmin -u root -p version
```

`mysqladmin version` có thể cung cấp version, uptime, connection information và một số counters.

### 3.3. Kiểm tra port

Windows PowerShell:

```powershell
Test-NetConnection -ComputerName localhost -Port 3306
```

Linux:

```bash
ss -ltnp | grep 3306
```

Port `3306` là mặc định phổ biến, nhưng server có thể dùng port khác. Kiểm tra từ SQL:

```sql
SHOW VARIABLES LIKE 'port';
```

### 3.4. Xác định service name

**Windows PowerShell:**

```powershell
Get-Service -Name "MySQL*"
```

Hoặc:

```powershell
Get-Service | Where-Object {
    $_.DisplayName -match "MySQL"
}
```

**Windows CMD:**

```cmd
sc query state= all | findstr /I MySQL
```

**Linux systemd:**

```bash
systemctl list-unit-files | grep -E 'mysql|mysqld'
```

Tên unit thường gặp:

```text
mysql
mysqld
```

Tên thực tế phụ thuộc distribution và cách cài đặt.

### Bài tập thực hành

**Bài 3.1.** Kiểm tra server phản hồi bằng `SELECT VERSION()`.

**Bài 3.2.** Chạy `mysqladmin -u root -p ping` nếu utility sẵn có.

**Bài 3.3.** Kiểm tra port từ SQL.

**Bài 3.4.** Dùng PowerShell hoặc CMD để tìm service MySQL trên Windows.

**Bài 3.5.** Dùng `systemctl list-unit-files` để tìm unit MySQL trên Linux.

---

# Phần A. Windows

## 4. Start/Stop/Restart MySQL trên Windows

### 4.1. Dùng Services console

1. Nhấn `Win + R`.
2. Nhập:

```text
services.msc
```

3. Tìm service có tên hoặc display name chứa MySQL, ví dụ `MySQL80`.
4. Kiểm tra trạng thái `Running` hoặc `Stopped`.
5. Chuột phải và chọn `Start`, `Stop` hoặc `Restart`.

> Không giả định mọi máy đều dùng service name `MySQL80`. Cần xác định tên service thực tế.

### 4.2. Dùng PowerShell

Liệt kê service MySQL:

```powershell
Get-Service -Name "MySQL*"
```

Kiểm tra chi tiết:

```powershell
Get-Service -Name MySQL80
```

Start:

```powershell
Start-Service -Name MySQL80
```

Stop:

```powershell
Stop-Service -Name MySQL80
```

Restart:

```powershell
Restart-Service -Name MySQL80
```

Các lệnh này cần chạy PowerShell với quyền phù hợp. Nếu service name không phải `MySQL80`, thay bằng name đã xác định bằng `Get-Service`.

### 4.3. Dùng Command Prompt với `sc`

Kiểm tra service:

```cmd
sc query MySQL80
```

Start:

```cmd
sc start MySQL80
```

Stop:

```cmd
sc stop MySQL80
```

Sau stop/start, kiểm tra lại:

```cmd
sc query MySQL80
```

### 4.4. Dùng `net start` / `net stop`

Một số môi trường cho phép:

```cmd
net start MySQL80
```

```cmd
net stop MySQL80
```

`sc` thường hữu ích hơn khi cần đọc state chi tiết.

### 4.5. Kiểm tra sau restart

PowerShell:

```powershell
Get-Service -Name MySQL80
```

SQL:

```sql
SELECT VERSION(),
       NOW(),
       @@port;
```

Hoặc:

```powershell
mysqladmin -u root -p ping
```

### Bài tập thực hành

**Bài 4.1.** Mở `services.msc` và xác định display name của MySQL service.

**Bài 4.2.** Dùng `Get-Service -Name "MySQL*"` để tìm service name.

**Bài 4.3.** Dùng `sc query <service_name>` để kiểm tra trạng thái service.

**Bài 4.4.** Trên máy lab, restart service bằng PowerShell hoặc Services console.

**Bài 4.5.** Sau restart, xác minh server bằng `SELECT VERSION()` hoặc `mysqladmin ping`.

---

## 5. Khi MySQL không start được trên Windows

### 5.1. Kiểm tra service status

```powershell
Get-Service -Name MySQL80
```

Hoặc:

```cmd
sc query MySQL80
```

### 5.2. Kiểm tra Windows Event Viewer và error log

Event Viewer:

1. Nhấn `Win + R`.
2. Nhập `eventvwr.msc`.
3. Xem **Windows Logs** → **Application**.
4. Tìm entries liên quan MySQL/service.

Đồng thời, kiểm tra error log path từ MySQL nếu server có thể start hoặc từ configuration/service setup.

Từ SQL, khi server đang chạy:

```sql
SHOW VARIABLES LIKE 'log_error';
```

### 5.3. Kiểm tra port conflict

Nếu MySQL không bind được port vì port đang bị process khác dùng, cần xác định port và process.

PowerShell:

```powershell
Get-NetTCPConnection -LocalPort 3306 -ErrorAction SilentlyContinue
```

CMD:

```cmd
netstat -ano | findstr :3306
```

Không tự dừng process khác chỉ vì dùng cùng port. Xác định process, ứng dụng và cấu hình trước.

### 5.4. Các nguyên nhân thường gặp

| Triệu chứng | Khả năng cần kiểm tra |
|---|---|
| Service stopped ngay sau start | Error log, config syntax, permission, data directory |
| Port bind failure | Port conflict hoặc bind-address/network config |
| Access denied to files | Windows service account hoặc NTFS permissions |
| Server failed after config edit | Sai option name/value hoặc sai group trong `my.ini` |
| Không kết nối được nhưng service running | Port, firewall, bind address, authentication/account |

### Bài tập thực hành

**Bài 5.1.** Dùng `sc query` để xem state của MySQL service.

**Bài 5.2.** Mở Event Viewer và xác định vị trí xem Application logs.

**Bài 5.3.** Chạy `netstat -ano | findstr :3306` trên Windows.

**Bài 5.4.** Nêu hai nguyên nhân service có thể dừng ngay sau start.

**Bài 5.5.** Nêu hai thông tin cần thu thập trước khi đổi port MySQL.

---

# Phần B. Linux

## 6. Start/Stop/Restart MySQL trên Linux dùng systemd

Nhiều Linux distributions hiện dùng systemd. Tên service thường là:

- `mysql` trên Debian/Ubuntu package.
- `mysqld` trên RPM-based package.

Không đoán tên unit. Kiểm tra trước:

```bash
systemctl list-unit-files | grep -E 'mysql|mysqld'
```

### 6.1. Kiểm tra trạng thái

Debian/Ubuntu thường gặp:

```bash
sudo systemctl status mysql
```

RPM-based thường gặp:

```bash
sudo systemctl status mysqld
```

### 6.2. Start service

```bash
sudo systemctl start mysql
```

hoặc:

```bash
sudo systemctl start mysqld
```

### 6.3. Stop service

```bash
sudo systemctl stop mysql
```

hoặc:

```bash
sudo systemctl stop mysqld
```

### 6.4. Restart service

```bash
sudo systemctl restart mysql
```

hoặc:

```bash
sudo systemctl restart mysqld
```

### 6.5. Enable/disable startup at boot

Enable:

```bash
sudo systemctl enable mysql
```

Disable:

```bash
sudo systemctl disable mysql
```

Chỉ thay đổi boot behavior khi hiểu yêu cầu vận hành của máy. Trên server production, đây thường là quyết định thuộc runbook/deployment policy.

### 6.6. Đọc log với journalctl

Debian/Ubuntu unit `mysql`:

```bash
sudo journalctl -u mysql -n 100 --no-pager
```

RPM-based unit `mysqld`:

```bash
sudo journalctl -u mysqld -n 100 --no-pager
```

Theo dõi log realtime:

```bash
sudo journalctl -u mysql -f
```

### Bài tập thực hành

**Bài 6.1.** Dùng `systemctl list-unit-files` để xác định unit name.

**Bài 6.2.** Dùng `systemctl status` để xem MySQL service.

**Bài 6.3.** Trên máy lab, restart unit đúng tên.

**Bài 6.4.** Dùng `journalctl` xem 100 dòng log gần nhất của unit.

**Bài 6.5.** Giải thích khác nhau giữa `systemctl restart` và `systemctl enable`.

---

## 7. Xác minh sau start/restart trên Linux

### 7.1. Kiểm tra unit

```bash
sudo systemctl is-active mysql
```

Hoặc:

```bash
sudo systemctl is-active mysqld
```

Nếu output là:

```text
active
```

service đang active theo systemd.

### 7.2. Kiểm tra server response

```bash
mysqladmin -u root -p ping
```

Hoặc:

```bash
mysql -u root -p -e "SELECT VERSION(), NOW();"
```

### 7.3. Kiểm tra port đang listen

```bash
sudo ss -ltnp | grep 3306
```

Nếu MySQL dùng port khác:

```sql
SHOW VARIABLES LIKE 'port';
```

### 7.4. Kiểm tra Uptime

```sql
SHOW STATUS LIKE 'Uptime';
```

Sau restart, `Uptime` sẽ gần với thời điểm service vừa lên.

### Bài tập thực hành

**Bài 7.1.** Dùng `systemctl is-active` để xác minh service.

**Bài 7.2.** Dùng `mysqladmin ping` để xác minh server đáp ứng.

**Bài 7.3.** Dùng `SHOW STATUS LIKE 'Uptime';`.

**Bài 7.4.** Dùng `ss` để tìm port MySQL đang listen.

**Bài 7.5.** Giải thích vì sao `systemctl is-active` và `mysqladmin ping` kiểm tra hai khía cạnh khác nhau.

---

## 8. Dừng server qua `mysqladmin shutdown`

Nếu account có quyền `SHUTDOWN`, có thể yêu cầu server shutdown qua client utility:

```bash
mysqladmin -u root -p shutdown
```

Sau đó service manager sẽ phản ánh trạng thái server đã dừng.

### 8.1. Khi dùng cách này?

Có thể phù hợp trong lab hoặc script quản trị khi:

- Có quyền phù hợp.
- Biết chính xác instance/server cần dừng.
- Có quy trình xác minh.
- Không làm gián đoạn hệ thống ngoài kế hoạch.

Trên máy đã được systemd/Windows Service quản lý, thao tác service qua service manager thường rõ ràng hơn cho start/stop/restart theo lifecycle hệ điều hành.

### 8.2. Không dùng force kill như thao tác bình thường

Không dùng:

```bash
kill -9 <pid>
```

như cách stop MySQL thường xuyên.

Nếu server không phản hồi, đây là tình huống incident cần đánh giá logs, process state, disk, memory và quy trình khôi phục. Sau forced termination, server có thể phải crash recovery lúc khởi động lại.

### Bài tập thực hành

**Bài 8.1.** Nêu quyền cần thiết cho `mysqladmin shutdown`.

**Bài 8.2.** Viết command yêu cầu shutdown qua `mysqladmin`.

**Bài 8.3.** Nêu một lý do system service manager thường phù hợp hơn để quản lý lifecycle.

**Bài 8.4.** Giải thích rủi ro của `kill -9` với database server.

**Bài 8.5.** Viết hai cách xác minh server đã dừng.

---

## 9. Checklist an toàn trước và sau restart

### 9.1. Trước restart

```text
[ ] Xác nhận đúng server instance/service name.
[ ] Xác nhận môi trường: lab, dev, staging hay production.
[ ] Kiểm tra ứng dụng/user bị ảnh hưởng.
[ ] Kiểm tra backup/migration/long-running jobs.
[ ] Đọc error log và service status.
[ ] Sao lưu option file nếu restart để áp dụng config mới.
[ ] Chuẩn bị rollback nếu server không khởi động lại.
```

### 9.2. Sau restart

```text
[ ] Service state là running/active.
[ ] mysqladmin ping hoặc SQL connection thành công.
[ ] Server version/port đúng như mong đợi.
[ ] Error log không có lỗi startup nghiêm trọng.
[ ] Application health checks thành công.
[ ] Uptime mới phản ánh restart.
```

### Bài tập thực hành

**Bài 9.1.** Nêu ba mục cần kiểm tra trước restart.

**Bài 9.2.** Nêu ba mục cần kiểm tra sau restart.

**Bài 9.3.** Viết một command kiểm tra service state trên Windows.

**Bài 9.4.** Viết một command kiểm tra service state trên Linux.

**Bài 9.5.** Giải thích vì sao cần xác định đúng instance khi một máy có nhiều MySQL instances.

---

## 10. Bài tập tổng hợp

### Bài 10.1. Windows lifecycle

Trên Windows:

1. Tìm service name MySQL.
2. Kiểm tra trạng thái.
3. Restart service trên máy lab.
4. Kiểm tra status sau restart.
5. Kết nối SQL và kiểm tra `Uptime`.

### Bài 10.2. Linux lifecycle

Trên Linux/systemd:

1. Xác định unit `mysql` hoặc `mysqld`.
2. Kiểm tra status.
3. Restart unit trên máy lab.
4. Xem journal log sau restart.
5. Kiểm tra server bằng `mysqladmin ping`.

### Bài 10.3. Chẩn đoán service không start

Một service MySQL vừa start rồi stopped:

1. Nêu ba nơi/lệnh cần kiểm tra.
2. Nêu một nguyên nhân liên quan config.
3. Nêu một nguyên nhân liên quan port.
4. Nêu một nguyên nhân liên quan data directory/permissions.
5. Nêu cách an toàn để rollback config vừa sửa.

### Bài 10.4. Đánh giá restart request

Application báo chậm. Một thành viên đề xuất restart MySQL ngay.

1. Nêu ba câu hỏi cần trả lời trước.
2. Nêu hai nguồn log/metric cần xem.
3. Nêu một giải pháp không phải restart có thể cần xem xét.
4. Nêu rủi ro của restart.
5. Nêu điều kiện khi restart có thể hợp lý.

### Bài 10.5. Health-check script logic

Thiết kế các bước:

1. Service is active/running.
2. TCP port accepts connections.
3. `mysqladmin ping` thành công.
4. `SELECT 1` thành công.
5. Error log không có lỗi mới nghiêm trọng.

---

## 11. Tóm tắt

- Start/stop/restart là thao tác vận hành service, không phải câu SQL thông thường.
- Trên Windows, dùng Services console, PowerShell hoặc `sc`; service name cần được xác định trước.
- Trên Linux, dùng `systemctl`; unit name thường là `mysql` hoặc `mysqld` tùy distribution.
- Xác minh sau thao tác bằng service state, `mysqladmin ping`, SQL connection, port listen và error log.
- Restart không phải “cách chữa chung”; cần đánh giá impact trước.
- Tránh force kill database process trừ tình huống khẩn cấp theo quy trình.
- Luôn giữ checklist và rollback plan khi restart để áp dụng configuration.

---

## 12. Từ khóa chính

- mysqld
- MySQL service
- Windows Services
- services.msc
- PowerShell
- sc query
- systemctl
- journalctl
- mysqladmin ping
- mysqladmin shutdown
- graceful shutdown
- restart
- service status
- port 3306
- error log
- uptime
- health check
