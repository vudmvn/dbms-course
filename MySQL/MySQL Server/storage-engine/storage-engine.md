---
layout: page
title: "MySQL Storage Engines"
---

# MySQL Storage Engines

**Cập nhật lần cuối:** 04/06/2026

**Nguồn tham khảo:**  
- MySQL Tutorial: [MySQL Storage Engines](https://www.mysqltutorial.org/mysql-administration/mysql-storage-engines/)
- MySQL Reference Manual: [Alternative Storage Engines](https://dev.mysql.com/doc/refman/8.4/en/storage-engines.html)
- MySQL Reference Manual: [The InnoDB Storage Engine](https://dev.mysql.com/doc/refman/8.4/en/innodb-storage-engine.html)

---

## 1. Mục tiêu bài giảng

Sau khi hoàn thành bài học này, người học có thể:

1. Giải thích được khái niệm **storage engine** trong MySQL.
2. Trình bày được vai trò của storage engine trong việc lưu trữ và xử lý dữ liệu bảng.
3. Sử dụng được `SHOW ENGINES` và `information_schema.engines` để kiểm tra storage engine.
4. Chỉ định được storage engine khi tạo bảng bằng mệnh đề `ENGINE`.
5. Phân biệt được các storage engine phổ biến như `InnoDB`, `MyISAM`, `MEMORY`, `ARCHIVE`, `CSV`, `BLACKHOLE`, `MERGE` và `FEDERATED`.
6. Lựa chọn được storage engine phù hợp cho một số tình huống thực tế.

---

## 2. Giới thiệu tổng quan

Trong MySQL, mỗi bảng được quản lý bởi một **storage engine**. Storage engine quyết định cách dữ liệu của bảng được lưu trữ, đọc, ghi, khóa, phục hồi và bảo vệ.

Có thể hình dung MySQL Server gồm hai phần lớn:

- **Tầng SQL:** nhận câu lệnh SQL, phân tích cú pháp, tối ưu truy vấn và điều phối thực thi.
- **Tầng storage engine:** thực hiện việc lưu trữ và truy xuất dữ liệu vật lý cho từng bảng.

Việc hiểu storage engine rất quan trọng vì không phải engine nào cũng hỗ trợ cùng một tính năng. Ví dụ, `InnoDB` hỗ trợ giao dịch và khóa ngoại, còn `MEMORY` lưu dữ liệu trong RAM và không phù hợp để lưu dữ liệu lâu dài.

---

### Quiz nhanh: Giới thiệu tổng quan

**Câu 1.** Storage engine trong MySQL gắn trực tiếp với đối tượng nào?

A. User  
B. Table  
C. Database password  
D. SQL comment  

**Câu 2.** Vì sao cần hiểu storage engine?

A. Vì storage engine quyết định toàn bộ cú pháp SQL  
B. Vì mỗi storage engine có đặc điểm lưu trữ và tính năng khác nhau  
C. Vì MySQL chỉ có một storage engine  
D. Vì storage engine chỉ dùng để đổi tên bảng  

**Câu 3.** Tầng nào chịu trách nhiệm lưu trữ dữ liệu vật lý của bảng?

A. SQL parser  
B. Storage engine  
C. Query cache  
D. Client application  

---

## 3. Khái niệm cơ bản

### 3.1. Storage engine là gì?

**Storage engine** là thành phần phần mềm trong MySQL chịu trách nhiệm quản lý cách dữ liệu của bảng được lưu trữ và truy xuất.

Một storage engine có thể quyết định:

- Dữ liệu được lưu trên đĩa hay trong bộ nhớ.
- Bảng có hỗ trợ giao dịch hay không.
- Bảng có hỗ trợ khóa ngoại hay không.
- Cơ chế khóa là khóa dòng hay khóa bảng.
- Cách phục hồi dữ liệu sau sự cố.
- Cách tổ chức chỉ mục.

### 3.2. Engine mặc định

Từ MySQL 5.5 trở đi, `InnoDB` là storage engine mặc định. Nếu câu lệnh `CREATE TABLE` không chỉ định `ENGINE`, MySQL sẽ dùng engine mặc định của server.

Ví dụ:

```sql
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    student_name VARCHAR(100)
);
```

Nếu engine mặc định là `InnoDB`, bảng `students` sẽ được tạo bằng `InnoDB`.

### 3.3. Ý nghĩa của storage engine

Storage engine ảnh hưởng trực tiếp đến:

- Tính toàn vẹn dữ liệu.
- Hiệu năng đọc/ghi.
- Khả năng phục hồi sau lỗi.
- Khả năng hỗ trợ ràng buộc khóa ngoại.
- Tính phù hợp của bảng với từng loại ứng dụng.

---

### Quiz nhanh: Khái niệm cơ bản

**Câu 1.** Storage engine nào là mặc định trong các phiên bản MySQL hiện đại?

A. MyISAM  
B. MEMORY  
C. InnoDB  
D. CSV  

**Câu 2.** Tính năng nào thường gắn với `InnoDB`?

A. Không lưu dữ liệu thật  
B. Hỗ trợ giao dịch và khóa ngoại  
C. Chỉ lưu dữ liệu trong RAM  
D. Chỉ lưu dữ liệu dạng CSV  

**Câu 3.** Nếu không chỉ định `ENGINE` khi tạo bảng, MySQL sẽ làm gì?

A. Báo lỗi ngay lập tức  
B. Tạo bảng không có dữ liệu  
C. Dùng storage engine mặc định  
D. Tự động dùng `CSV`  

---

## 4. Cách MySQL làm việc với Storage Engine

Khi người dùng gửi một câu lệnh SQL liên quan đến bảng, MySQL xử lý theo luồng tổng quát sau:

1. **Nhận câu lệnh SQL:** Client gửi câu lệnh như `SELECT`, `INSERT`, `UPDATE`, `DELETE` hoặc `CREATE TABLE`.
2. **Phân tích và tối ưu:** MySQL Server kiểm tra cú pháp, quyền truy cập và lập kế hoạch thực thi.
3. **Gọi storage engine:** MySQL chuyển yêu cầu đọc/ghi xuống storage engine của bảng.
4. **Truy cập dữ liệu:** Storage engine đọc, ghi, khóa, phục hồi hoặc tổ chức dữ liệu theo cơ chế riêng.
5. **Trả kết quả:** MySQL Server nhận dữ liệu từ storage engine và trả về cho client.

Ví dụ khi tạo bảng có chỉ định engine:

```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    phone VARCHAR(20)
) ENGINE = InnoDB;
```

Trong ví dụ này, bảng `customers` được quản lý bởi `InnoDB`.

---

### Quiz nhanh: Cách hoạt động

**Câu 1.** Mệnh đề nào dùng để chỉ định storage engine khi tạo bảng?

A. `USING`  
B. `ENGINE`  
C. `STORAGE PATH`  
D. `TABLE TYPE ONLY`  

**Câu 2.** Lệnh nào tạo bảng dùng engine `MEMORY`?

A. `CREATE TABLE t (id INT) ENGINE = MEMORY;`  
B. `CREATE MEMORY t (id INT);`  
C. `CREATE TABLE t MEMORY(id INT);`  
D. `ENGINE MEMORY CREATE TABLE t;`  

**Câu 3.** Storage engine được MySQL gọi khi nào?

A. Khi cần đọc hoặc ghi dữ liệu bảng  
B. Chỉ khi đổi mật khẩu user  
C. Chỉ khi mở MySQL Workbench  
D. Chỉ khi sao lưu file ảnh  

---

## 5. Các thành phần chính khi làm việc với Storage Engine

### 5.1. Danh sách engine có sẵn

Dùng lệnh sau để xem các storage engine mà MySQL Server đang hỗ trợ:

```sql
SHOW ENGINES;
```

Hoặc truy vấn bảng hệ thống:

```sql
SELECT
    engine,
    support
FROM information_schema.engines
ORDER BY engine;
```

### 5.2. Trạng thái hỗ trợ

Cột `Support` hoặc `support` thường có các giá trị:

| Giá trị | Ý nghĩa |
|---|---|
| `YES` | Engine được hỗ trợ |
| `NO` | Engine không được hỗ trợ |
| `DEFAULT` | Engine được hỗ trợ và đang là mặc định |
| `DISABLED` | Engine có trong MySQL nhưng đang bị vô hiệu hóa |

### 5.3. Các khả năng quan trọng

Khi so sánh storage engine, cần chú ý:

- Có hỗ trợ giao dịch không.
- Có hỗ trợ khóa ngoại không.
- Có hỗ trợ phục hồi sau sự cố không.
- Dữ liệu có bền vững sau khi server dừng không.
- Engine phù hợp với đọc nhiều, ghi nhiều hay dữ liệu tạm.

### 5.4. Mệnh đề `ENGINE`

Cú pháp tổng quát:

```sql
CREATE TABLE table_name (
    column_list
) ENGINE = engine_name;
```

---

### Quiz nhanh: Các thành phần chính

**Câu 1.** Lệnh nào dùng để xem danh sách storage engine?

A. `SHOW TABLES;`  
B. `SHOW ENGINES;`  
C. `SHOW DATABASES;`  
D. `SHOW USERS;`  

**Câu 2.** Giá trị `DEFAULT` trong `SHOW ENGINES` nghĩa là gì?

A. Engine bị vô hiệu hóa  
B. Engine không tồn tại  
C. Engine đang là mặc định  
D. Engine chỉ dùng cho backup  

**Câu 3.** Bảng hệ thống nào có thể dùng để xem thông tin engine?

A. `information_schema.engines`  
B. `mysql.users`  
C. `performance_schema.passwords`  
D. `sys.tablespaces_only`  

---

## 6. Phân loại các Storage Engine chính

Một số storage engine thường gặp trong MySQL:

1. **InnoDB:** engine mặc định, hỗ trợ giao dịch, khóa ngoại và phục hồi sau sự cố.
2. **MyISAM:** engine cũ, tối ưu cho một số tình huống đọc nhiều, không hỗ trợ giao dịch.
3. **MEMORY:** lưu dữ liệu trong RAM, phù hợp cho dữ liệu tạm.
4. **ARCHIVE:** lưu trữ dữ liệu nén, phù hợp cho dữ liệu lịch sử hoặc log.
5. **CSV:** lưu dữ liệu dưới dạng file CSV.
6. **BLACKHOLE:** nhận dữ liệu ghi vào nhưng không lưu dữ liệu.
7. **MERGE/MRG_MyISAM:** gộp nhiều bảng `MyISAM` cùng cấu trúc thành một bảng logic.
8. **FEDERATED:** truy cập dữ liệu từ một MySQL Server từ xa.

---

## 7. InnoDB

### 7.1. Khái niệm

`InnoDB` là storage engine mặc định và phổ biến nhất trong MySQL hiện đại. Đây là lựa chọn an toàn cho phần lớn hệ thống nghiệp vụ vì hỗ trợ giao dịch, khóa ngoại, khóa mức dòng và phục hồi sau sự cố.

### 7.2. Đặc điểm chính

1. **Hỗ trợ giao dịch**

   `InnoDB` hỗ trợ `COMMIT`, `ROLLBACK` và tính chất ACID.

2. **Hỗ trợ khóa ngoại**

   Có thể khai báo ràng buộc `FOREIGN KEY` để đảm bảo dữ liệu tham chiếu hợp lệ.

3. **Khóa mức dòng**

   Khi nhiều người dùng thao tác đồng thời, `InnoDB` có thể khóa ở mức dòng để giảm xung đột so với khóa toàn bảng.

4. **Phục hồi sau sự cố**

   `InnoDB` có cơ chế ghi log và phục hồi để giảm nguy cơ hỏng dữ liệu khi server gặp lỗi.

### 7.3. Ví dụ

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date DATE,
    total_amount DECIMAL(12, 2)
) ENGINE = InnoDB;
```

Nên dùng `InnoDB` cho hệ thống bán hàng, ngân hàng, quản lý đơn hàng, quản lý người dùng và các ứng dụng cần tính toàn vẹn dữ liệu.

---

### Quiz nhanh: InnoDB

**Câu 1.** `InnoDB` phù hợp nhất với hệ thống nào?

A. Hệ thống cần giao dịch và khóa ngoại  
B. Bảng chỉ dùng để bỏ dữ liệu đi  
C. File CSV mở bằng Excel  
D. Bảng tạm mất dữ liệu khi server dừng  

**Câu 2.** `InnoDB` hỗ trợ cơ chế nào?

A. `ROLLBACK`  
B. Chỉ đọc file text  
C. Không lưu dữ liệu thật  
D. Chỉ lưu dữ liệu trong RAM  

**Câu 3.** Lý do quan trọng để dùng `InnoDB` cho bảng `orders` là gì?

A. Vì đơn hàng thường cần tính nhất quán và ràng buộc dữ liệu  
B. Vì `orders` không cần lưu dữ liệu  
C. Vì `orders` luôn là file CSV  
D. Vì `orders` không bao giờ thay đổi  

---

## 8. Các Storage Engine khác

### 8.1. MyISAM

`MyISAM` từng là storage engine mặc định trong MySQL trước phiên bản 5.5. Engine này có thể nhanh trong một số tình huống đọc nhiều, nhưng không hỗ trợ giao dịch và khóa ngoại.

Ví dụ:

```sql
CREATE TABLE article_archive (
    article_id INT PRIMARY KEY,
    title VARCHAR(255),
    content TEXT
) ENGINE = MyISAM;
```

### 8.2. MEMORY

`MEMORY` lưu dữ liệu trong RAM. Engine này phù hợp cho bảng tạm hoặc dữ liệu trung gian cần truy xuất nhanh.

```sql
CREATE TABLE session_cache (
    session_id VARCHAR(100) PRIMARY KEY,
    user_id INT,
    created_at DATETIME
) ENGINE = MEMORY;
```

Lưu ý: dữ liệu trong bảng `MEMORY` sẽ mất khi MySQL Server dừng hoặc khởi động lại.

### 8.3. ARCHIVE

`ARCHIVE` dùng để lưu lượng lớn dữ liệu lịch sử ở dạng nén. Engine này phù hợp với dữ liệu log hoặc dữ liệu ít thay đổi.

```sql
CREATE TABLE audit_logs (
    log_id INT,
    action_name VARCHAR(100),
    action_time DATETIME
) ENGINE = ARCHIVE;
```

### 8.4. CSV

`CSV` lưu dữ liệu dưới dạng file CSV. Engine này thuận tiện cho trao đổi dữ liệu nhưng hạn chế về hiệu năng và ràng buộc.

```sql
CREATE TABLE export_customers (
    customer_id INT NOT NULL,
    customer_name VARCHAR(100) NOT NULL,
    phone VARCHAR(20) NOT NULL
) ENGINE = CSV;
```

### 8.5. BLACKHOLE

`BLACKHOLE` nhận dữ liệu ghi vào nhưng không lưu dữ liệu. Nó thường dùng cho một số kịch bản kiểm thử hoặc replication đặc biệt.

```sql
CREATE TABLE event_sink (
    event_id INT,
    event_data TEXT
) ENGINE = BLACKHOLE;
```

### 8.6. MERGE

`MERGE`, còn gọi là `MRG_MyISAM`, cho phép kết hợp nhiều bảng `MyISAM` có cùng cấu trúc thành một bảng logic.

### 8.7. FEDERATED

`FEDERATED` cho phép truy cập dữ liệu từ một MySQL Server từ xa mà không lưu dữ liệu thật trong bảng cục bộ.

---

### Quiz nhanh: Các Storage Engine khác

**Câu 1.** Engine nào lưu dữ liệu trong RAM?

A. MEMORY  
B. CSV  
C. BLACKHOLE  
D. FEDERATED  

**Câu 2.** Engine nào nhận dữ liệu nhưng không lưu lại dữ liệu?

A. InnoDB  
B. BLACKHOLE  
C. MyISAM  
D. ARCHIVE  

**Câu 3.** Engine nào phù hợp hơn cho dữ liệu log lịch sử, ít cập nhật?

A. ARCHIVE  
B. MEMORY  
C. BLACKHOLE  
D. FEDERATED  

---

## 9. Nguyên lý và tiêu chí lựa chọn Storage Engine

### 9.1. Tính toàn vẹn dữ liệu

Nếu dữ liệu có quan hệ chặt chẽ, cần khóa ngoại hoặc cần rollback khi lỗi, nên ưu tiên `InnoDB`.

### 9.2. Độ bền dữ liệu

Nếu dữ liệu phải tồn tại sau khi server khởi động lại, không nên dùng `MEMORY` cho dữ liệu chính.

### 9.3. Mục tiêu hiệu năng

Nếu cần truy cập dữ liệu tạm rất nhanh, `MEMORY` có thể phù hợp. Nếu cần lưu dữ liệu giao dịch ổn định, `InnoDB` thường là lựa chọn mặc định.

### 9.4. Tính chất dữ liệu

Dữ liệu lịch sử ít thay đổi có thể phù hợp với `ARCHIVE`. Dữ liệu trao đổi với công cụ ngoài có thể cân nhắc `CSV`, nhưng không nên dùng cho bảng nghiệp vụ chính.

---

### Quiz nhanh: Nguyên lý lựa chọn

**Câu 1.** Nếu cần khóa ngoại, engine nào thường phù hợp nhất?

A. InnoDB  
B. MEMORY  
C. BLACKHOLE  
D. CSV  

**Câu 2.** Vì sao không nên dùng `MEMORY` cho dữ liệu nghiệp vụ quan trọng?

A. Vì dữ liệu có thể mất khi server dừng  
B. Vì nó luôn hỗ trợ khóa ngoại  
C. Vì nó chỉ dùng cho file ảnh  
D. Vì nó không thể tạo bảng  

**Câu 3.** Dữ liệu log lớn, ít cập nhật có thể cân nhắc engine nào?

A. ARCHIVE  
B. BLACKHOLE  
C. FEDERATED  
D. MEMORY  

---

## 10. Ứng dụng thực tế

1. **Hệ thống bán hàng**

   Dùng `InnoDB` cho bảng khách hàng, đơn hàng, chi tiết đơn hàng và thanh toán vì cần giao dịch và tính nhất quán.

2. **Bảng dữ liệu tạm**

   Dùng `MEMORY` cho bảng tạm trong quá trình tính toán hoặc tạo báo cáo ngắn hạn.

3. **Lưu trữ log**

   Dùng `ARCHIVE` khi cần lưu lượng lớn dữ liệu lịch sử, ít khi cập nhật.

4. **Trao đổi dữ liệu**

   Dùng `CSV` khi cần dữ liệu ở dạng dễ đọc bởi công cụ bảng tính hoặc quy trình import/export đơn giản.

5. **Kiểm thử hoặc replication đặc biệt**

   Dùng `BLACKHOLE` trong các tình huống muốn ghi nhận câu lệnh ghi nhưng không lưu dữ liệu thật.

---

### Quiz nhanh: Ứng dụng thực tế

**Câu 1.** Hệ thống bán hàng có đơn hàng và thanh toán nên ưu tiên engine nào?

A. InnoDB  
B. CSV  
C. BLACKHOLE  
D. MEMORY  

**Câu 2.** Bảng tạm để lưu kết quả trung gian có thể dùng engine nào?

A. MEMORY  
B. FEDERATED  
C. BLACKHOLE  
D. CSV  

**Câu 3.** Dữ liệu cần trao đổi dạng file văn bản phân tách bằng dấu phẩy phù hợp với engine nào?

A. CSV  
B. InnoDB  
C. MEMORY  
D. MERGE  

---

## 11. Vai trò trong quản trị cơ sở dữ liệu

### 11.1. Thiết kế cơ sở dữ liệu

Người thiết kế cần chọn storage engine phù hợp với tính chất bảng. Bảng nghiệp vụ chính thường dùng `InnoDB`.

### 11.2. Tối ưu hiệu năng

DBA cần hiểu engine để phân tích nguyên nhân chậm, xung đột khóa, khả năng ghi log và đặc điểm đọc/ghi.

### 11.3. Sao lưu và phục hồi

Mỗi engine có đặc điểm lưu trữ khác nhau, vì vậy chiến lược backup/restore cần phù hợp với engine đang dùng.

### 11.4. Vận hành hệ thống

Khi nâng cấp, di chuyển hoặc kiểm tra server, DBA cần biết engine nào đang được hỗ trợ và engine nào đang là mặc định.

---

### Quiz nhanh: Vai trò theo lĩnh vực

**Câu 1.** Trong thiết kế cơ sở dữ liệu nghiệp vụ, engine nào thường là lựa chọn mặc định an toàn?

A. InnoDB  
B. BLACKHOLE  
C. CSV  
D. MEMORY  

**Câu 2.** DBA dùng kiến thức storage engine để làm gì?

A. Phân tích hiệu năng và khả năng phục hồi dữ liệu  
B. Đổi màu giao diện MySQL Workbench  
C. Tạo mật khẩu hệ điều hành  
D. Viết HTML cho website  

**Câu 3.** Khi kiểm tra server, lệnh nào giúp biết engine đang được hỗ trợ?

A. `SHOW ENGINES;`  
B. `SHOW COLORS;`  
C. `SHOW FILES;`  
D. `SHOW CLIENTS;`  

---

## 12. Bảng so sánh

| Tiêu chí | InnoDB | MyISAM | MEMORY | ARCHIVE | CSV | BLACKHOLE | FEDERATED |
|---|---|---|---|---|---|---|---|
| Giao dịch | Có | Không | Không | Không | Không | Không | Không |
| Khóa ngoại | Có | Không | Không | Không | Không | Không | Không |
| Dữ liệu bền vững | Có | Có | Không | Có | Có | Không | Phụ thuộc server từ xa |
| Mục tiêu chính | Bảng nghiệp vụ | Đọc nhiều, hệ thống cũ | Bảng tạm | Lưu trữ lịch sử | Trao đổi CSV | Bỏ dữ liệu ghi vào | Truy cập dữ liệu từ xa |
| Phù hợp cho dữ liệu chính | Có | Hạn chế | Không | Hạn chế | Không | Không | Hạn chế |
| Ví dụ | Đơn hàng | Bài viết cũ | Cache tạm | Audit log | Export data | Replication đặc biệt | Bảng remote |

---

## 13. Câu hỏi ôn tập

### 13.1. Câu hỏi trắc nghiệm

**Câu 1.** Storage engine trong MySQL dùng để làm gì?

A. Quản lý cách bảng lưu trữ và truy xuất dữ liệu  
B. Chỉ quản lý màu giao diện  
C. Chỉ tạo user  
D. Chỉ đổi tên database  

---

**Câu 2.** Lệnh nào dùng để xem danh sách storage engine?

A. `SHOW ENGINES;`  
B. `SHOW TABLE TYPES ONLY;`  
C. `LIST STORAGE;`  
D. `SELECT ENGINE FROM users;`  

---

**Câu 3.** Engine nào là mặc định trong MySQL hiện đại?

A. CSV  
B. InnoDB  
C. BLACKHOLE  
D. MERGE  

---

**Câu 4.** Engine nào hỗ trợ giao dịch và khóa ngoại?

A. InnoDB  
B. MyISAM  
C. CSV  
D. MEMORY  

---

**Câu 5.** Engine nào lưu dữ liệu trong RAM?

A. MEMORY  
B. ARCHIVE  
C. FEDERATED  
D. MyISAM  

---

**Câu 6.** Engine nào phù hợp cho dữ liệu log lịch sử ở dạng nén?

A. ARCHIVE  
B. BLACKHOLE  
C. MEMORY  
D. CSV  

---

**Câu 7.** Engine nào không lưu dữ liệu thật?

A. BLACKHOLE  
B. InnoDB  
C. MyISAM  
D. CSV  

---

**Câu 8.** Câu lệnh nào chỉ định bảng dùng `InnoDB`?

A. `CREATE TABLE t (id INT) ENGINE = InnoDB;`  
B. `CREATE InnoDB TABLE t (id INT);`  
C. `CREATE TABLE t ENGINE ONLY;`  
D. `ENGINE t CREATE TABLE InnoDB;`  

---

**Câu 9.** Nếu bảng cần khóa ngoại, lựa chọn nào phù hợp nhất?

A. InnoDB  
B. MEMORY  
C. CSV  
D. BLACKHOLE  

---

**Câu 10.** Engine nào cho phép truy cập bảng từ MySQL Server từ xa?

A. FEDERATED  
B. MEMORY  
C. ARCHIVE  
D. CSV  

---

### 13.2. Câu hỏi tự luận ngắn

**Câu 1.** Trình bày khái niệm storage engine trong MySQL.

---

**Câu 2.** Vì sao `InnoDB` thường được khuyến nghị cho các bảng nghiệp vụ chính?

---

**Câu 3.** Phân biệt `InnoDB` và `MEMORY` theo độ bền dữ liệu.

---

**Câu 4.** Nêu một tình huống phù hợp để dùng `ARCHIVE`.

---

**Câu 5.** Vì sao không nên dùng `BLACKHOLE` cho dữ liệu cần lưu trữ?

---

## 14. Bài tập vận dụng

### Bài tập 1

Viết câu lệnh SQL để liệt kê tất cả storage engine có trên MySQL Server.

**Yêu cầu:**  
Sử dụng ít nhất một trong hai cách: `SHOW ENGINES` hoặc truy vấn `information_schema.engines`.

---

### Bài tập 2

Tạo bảng `students` gồm các cột:

- `student_id`
- `student_name`
- `email`

**Yêu cầu:**  
Bảng phải dùng storage engine `InnoDB`.

---

### Bài tập 3

Tạo bảng `temp_report` gồm các cột:

- `report_id`
- `report_name`
- `created_at`

**Yêu cầu:**  
Bảng phải dùng storage engine `MEMORY` và người học cần giải thích hạn chế của lựa chọn này.

---

### Bài tập 4

Một hệ thống bán hàng cần lưu khách hàng, đơn hàng và thanh toán. Hệ thống cần rollback khi giao dịch lỗi và cần đảm bảo đơn hàng luôn tham chiếu đến khách hàng hợp lệ.

**Yêu cầu:**  
Chọn storage engine phù hợp và giải thích lý do.

---

## 15. Tóm tắt bài học

- Storage engine quyết định cách MySQL lưu trữ và truy xuất dữ liệu trong bảng.
- `InnoDB` là engine mặc định và phù hợp với phần lớn ứng dụng hiện đại.
- `InnoDB` hỗ trợ giao dịch, khóa ngoại, khóa mức dòng và phục hồi sau sự cố.
- `MEMORY` nhanh nhưng không phù hợp để lưu dữ liệu lâu dài.
- `ARCHIVE` phù hợp cho dữ liệu lịch sử hoặc log lớn, ít cập nhật.
- `CSV`, `BLACKHOLE`, `MERGE` và `FEDERATED` phục vụ các nhu cầu đặc thù.
- Chọn storage engine đúng giúp cải thiện hiệu năng, độ tin cậy và tính toàn vẹn dữ liệu.

---

## 16. Từ khóa chính

- MySQL
- Storage engine
- InnoDB
- MyISAM
- MEMORY
- ARCHIVE
- CSV
- BLACKHOLE
- MERGE
- FEDERATED
- Transaction
- Foreign key
- `SHOW ENGINES`
- `information_schema.engines`
- `CREATE TABLE`
- `ENGINE`

---

## 17. Đáp án và gợi ý trả lời

<details markdown="1">
<summary>Bấm để xem đáp án và gợi ý trả lời</summary>

### Quiz nhanh: Giới thiệu tổng quan

- **Câu 1.** B
- **Câu 2.** B
- **Câu 3.** B

### Quiz nhanh: Khái niệm cơ bản

- **Câu 1.** C
- **Câu 2.** B
- **Câu 3.** C

### Quiz nhanh: Cách hoạt động

- **Câu 1.** B
- **Câu 2.** A
- **Câu 3.** A

### Quiz nhanh: Các thành phần chính

- **Câu 1.** B
- **Câu 2.** C
- **Câu 3.** A

### Quiz nhanh: InnoDB

- **Câu 1.** A
- **Câu 2.** A
- **Câu 3.** A

### Quiz nhanh: Các Storage Engine khác

- **Câu 1.** A
- **Câu 2.** B
- **Câu 3.** A

### Quiz nhanh: Nguyên lý lựa chọn

- **Câu 1.** A
- **Câu 2.** A
- **Câu 3.** A

### Quiz nhanh: Ứng dụng thực tế

- **Câu 1.** A
- **Câu 2.** A
- **Câu 3.** A

### Quiz nhanh: Vai trò theo lĩnh vực

- **Câu 1.** A
- **Câu 2.** A
- **Câu 3.** A

### Câu hỏi ôn tập - Trắc nghiệm

- **Câu 1.** A
- **Câu 2.** A
- **Câu 3.** B
- **Câu 4.** A
- **Câu 5.** A
- **Câu 6.** A
- **Câu 7.** A
- **Câu 8.** A
- **Câu 9.** A
- **Câu 10.** A

### Câu hỏi ôn tập - Tự luận ngắn

#### Câu 1

**Gợi ý trả lời:**  
Storage engine là thành phần trong MySQL quản lý cách dữ liệu của bảng được lưu trữ, truy xuất, khóa, cập nhật và phục hồi.

#### Câu 2

**Gợi ý trả lời:**  
`InnoDB` thường được khuyến nghị vì hỗ trợ giao dịch, khóa ngoại, tính nhất quán dữ liệu, khóa mức dòng và phục hồi sau sự cố.

#### Câu 3

**Gợi ý trả lời:**  
`InnoDB` lưu dữ liệu bền vững trên đĩa và có cơ chế phục hồi. `MEMORY` lưu dữ liệu trong RAM nên dữ liệu có thể mất khi MySQL Server dừng hoặc khởi động lại.

#### Câu 4

**Gợi ý trả lời:**  
`ARCHIVE` phù hợp để lưu audit log, dữ liệu lịch sử, dữ liệu ít cập nhật nhưng có dung lượng lớn.

#### Câu 5

**Gợi ý trả lời:**  
`BLACKHOLE` không lưu dữ liệu thật. Dữ liệu ghi vào bảng sẽ bị bỏ đi, nên không phù hợp cho dữ liệu cần lưu trữ.

### Bài tập vận dụng

#### Bài tập 1

**Gợi ý trả lời:**

```sql
SHOW ENGINES;
```

Hoặc:

```sql
SELECT
    engine,
    support
FROM information_schema.engines
ORDER BY engine;
```

#### Bài tập 2

**Gợi ý trả lời:**

```sql
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    student_name VARCHAR(100),
    email VARCHAR(100)
) ENGINE = InnoDB;
```

#### Bài tập 3

**Gợi ý trả lời:**

```sql
CREATE TABLE temp_report (
    report_id INT PRIMARY KEY,
    report_name VARCHAR(100),
    created_at DATETIME
) ENGINE = MEMORY;
```

Hạn chế: dữ liệu trong bảng `MEMORY` không phù hợp để lưu lâu dài vì có thể mất khi MySQL Server dừng hoặc khởi động lại.

#### Bài tập 4

**Gợi ý trả lời:**  
Nên chọn `InnoDB` vì hệ thống cần giao dịch, rollback, khóa ngoại và tính toàn vẹn dữ liệu giữa khách hàng, đơn hàng và thanh toán.

</details>

