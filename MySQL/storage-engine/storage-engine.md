# Bài giảng: MySQL Storage Engines

## 1. Tóm tắt bài học

Trong bài học này, người học sẽ tìm hiểu về các **storage engine** khác nhau trong MySQL. Việc hiểu đặc điểm của từng storage engine là rất quan trọng vì nó giúp lựa chọn cơ chế lưu trữ phù hợp, từ đó cải thiện hiệu năng, độ tin cậy và khả năng quản lý dữ liệu của cơ sở dữ liệu.

---

## 2. Mục tiêu học tập

Sau khi hoàn thành bài học này, người học có thể:

1. Giải thích được khái niệm **storage engine** trong MySQL.
2. Liệt kê được các storage engine phổ biến trong MySQL.
3. Sử dụng câu lệnh SQL để kiểm tra các storage engine có sẵn trên MySQL Server.
4. Phân biệt được các đặc điểm chính của InnoDB, MyISAM, MEMORY, ARCHIVE, CSV, BLACKHOLE, MERGE và FEDERATED.
5. Biết cách chỉ định storage engine khi tạo bảng.
6. Lựa chọn storage engine phù hợp với từng tình huống sử dụng.

---

## 3. Khái niệm MySQL Storage Engine

Trong MySQL, **storage engine** là thành phần phần mềm chịu trách nhiệm quản lý cách dữ liệu được:

- Lưu trữ.
- Truy xuất.
- Cập nhật.
- Xóa.
- Quản lý bên trong các bảng.

Nói cách khác, storage engine quyết định cách MySQL tổ chức dữ liệu ở tầng lưu trữ vật lý và hỗ trợ các tính năng khác nhau cho bảng.

Ví dụ, một storage engine có thể hỗ trợ:

- Giao dịch.
- Khóa ngoại.
- Khôi phục sau sự cố.
- Tìm kiếm toàn văn.
- Lưu trữ dữ liệu trong bộ nhớ.
- Nén dữ liệu.
- Truy cập dữ liệu từ máy chủ từ xa.

---

## 4. Vì sao cần hiểu Storage Engine?

Không phải tất cả các bảng trong MySQL đều được lưu trữ theo cùng một cách. Mỗi storage engine có ưu điểm và hạn chế riêng.

Ví dụ:

- Nếu cần giao dịch, rollback và khóa ngoại, nên dùng `InnoDB`.
- Nếu cần bảng tạm trong bộ nhớ, có thể dùng `MEMORY`.
- Nếu cần lưu trữ dữ liệu lịch sử với dung lượng lớn, có thể dùng `ARCHIVE`.
- Nếu cần lưu bảng dạng CSV để trao đổi dữ liệu, có thể dùng `CSV`.

Do đó, lựa chọn storage engine phù hợp giúp:

- Tăng hiệu năng truy vấn.
- Đảm bảo tính toàn vẹn dữ liệu.
- Giảm dung lượng lưu trữ.
- Phù hợp hơn với mục đích sử dụng của ứng dụng.

---

## 5. Kiểm tra các Storage Engine có sẵn trong MySQL

MySQL hỗ trợ nhiều storage engine. Để xem các storage engine hiện có trên MySQL Server, có thể dùng truy vấn sau:

```sql
SELECT 
  engine, 
  support 
FROM 
  information_schema.engines 
ORDER BY 
  engine;
```

Ví dụ kết quả:

```text
+--------------------+---------+
| engine             | support |
+--------------------+---------+
| ARCHIVE            | YES     |
| BLACKHOLE          | YES     |
| CSV                | YES     |
| FEDERATED          | NO      |
| InnoDB             | DEFAULT |
| MEMORY             | YES     |
| MRG_MYISAM         | YES     |
| MyISAM             | YES     |
| ndbcluster         | NO      |
| ndbinfo            | NO      |
| PERFORMANCE_SCHEMA | YES     |
+--------------------+---------+
```

### Ý nghĩa của cột `support`

| Giá trị | Ý nghĩa |
|---|---|
| `YES` | Storage engine được hỗ trợ |
| `NO` | Storage engine không được hỗ trợ |
| `DEFAULT` | Storage engine được hỗ trợ và đang là mặc định |

Trong ví dụ trên, `InnoDB` có giá trị `DEFAULT`, nghĩa là MySQL đang dùng `InnoDB` làm storage engine mặc định.

---

## 6. Sử dụng lệnh SHOW ENGINES

Ngoài cách truy vấn bảng `information_schema.engines`, ta cũng có thể dùng câu lệnh:

```sql
SHOW ENGINES;
```

Ví dụ kết quả:

```text
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| Engine             | Support | Comment                                                        | Transactions | XA   | Savepoints |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |
| CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |
| FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |
| MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |
| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
| ndbinfo            | NO      | MySQL Cluster system information storage engine                | NULL         | NULL | NULL       |
| BLACKHOLE          | YES     | /dev/null storage engine                                       | NO           | NO   | NO         |
| ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |
| ndbcluster         | NO      | Clustered, fault-tolerant tables                               | NULL         | NULL | NULL       |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
```

Lệnh `SHOW ENGINES` cung cấp nhiều thông tin hơn, bao gồm:

- Tên storage engine.
- Trạng thái hỗ trợ.
- Mô tả ngắn.
- Có hỗ trợ giao dịch hay không.
- Có hỗ trợ XA transaction hay không.
- Có hỗ trợ savepoint hay không.

---

## 7. Storage Engine mặc định trong MySQL

Từ MySQL 5.5 trở đi, `InnoDB` là storage engine mặc định.

Điều này có nghĩa là nếu khi tạo bảng không chỉ định storage engine, MySQL sẽ tự động dùng `InnoDB`.

Ví dụ:

```sql
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    student_name VARCHAR(100)
);
```

Trong trường hợp này, nếu storage engine mặc định là `InnoDB`, bảng `students` sẽ được tạo bằng `InnoDB`.

---

## 8. Chỉ định Storage Engine khi tạo bảng

Để chỉ định storage engine khi tạo bảng, sử dụng mệnh đề `ENGINE` trong câu lệnh `CREATE TABLE`.

Cú pháp tổng quát:

```sql
CREATE TABLE table_name (
    column_list
) ENGINE = engine_name;
```

Ví dụ tạo bảng dùng `InnoDB`:

```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    phone VARCHAR(20)
) ENGINE = InnoDB;
```

Ví dụ tạo bảng dùng `MEMORY`:

```sql
CREATE TABLE temp_results (
    id INT PRIMARY KEY,
    result_value VARCHAR(100)
) ENGINE = MEMORY;
```

Nếu bỏ qua mệnh đề `ENGINE`, MySQL sẽ dùng storage engine mặc định.

---

## 9. So sánh các Storage Engine trong MySQL

Bảng sau tóm tắt một số đặc điểm quan trọng của các storage engine phổ biến.

| Tính năng | MyISAM | MEMORY | CSV | ARCHIVE | BLACKHOLE | MERGE | FEDERATED | InnoDB |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| Hỗ trợ giao dịch | Không | Không | Không | Không | Không | Không | Không | Có |
| Tuân thủ ACID | Không | Không | Không | Không | Không | Không | Không | Có |
| Khóa mức bảng | Có | Không | Có | Có | Có | Không | Có | Có |
| Tìm kiếm toàn văn | Có | Không | Không | Không | Không | Không | Không | Có |
| Khóa ngoại | Không | Không | Không | Không | Không | Không | Không | Có |
| Khôi phục sau sự cố | Không | Không | Không | Không | Không | Không | Không | Có |
| Truy cập dữ liệu ngoài | Không | Không | Có | Có | Không | Không | Có | Không |
| Bảng tạm | Có | Có | Không | Không | Không | Không | Không | Có |
| Storage engine mặc định | Không | Không | Không | Không | Không | Không | Không | Có |

---

## 10. InnoDB

`InnoDB` là storage engine mặc định và được sử dụng phổ biến nhất trong MySQL hiện nay.

### Đặc điểm chính

`InnoDB` hỗ trợ đầy đủ:

- Giao dịch.
- Tính chất ACID.
- Khóa ngoại.
- `COMMIT`.
- `ROLLBACK`.
- Khôi phục sau sự cố.
- Khóa mức dòng.

### Ưu điểm

- Phù hợp với các hệ thống cần tính toàn vẹn dữ liệu cao.
- Hỗ trợ ràng buộc khóa ngoại.
- Có khả năng phục hồi sau lỗi.
- Thích hợp cho ứng dụng giao dịch như ngân hàng, bán hàng, quản lý đơn hàng.

### Ví dụ sử dụng

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    total_amount DECIMAL(12, 2)
) ENGINE = InnoDB;
```

### Khi nào nên dùng InnoDB?

Nên dùng `InnoDB` khi hệ thống cần:

- Giao dịch.
- Khóa ngoại.
- Dữ liệu nhất quán.
- Khả năng rollback.
- Ứng dụng có nhiều thao tác thêm, sửa, xóa dữ liệu.

---

## 11. MyISAM

`MyISAM` là storage engine cũ, từng là storage engine mặc định trong MySQL trước phiên bản 5.5.

### Đặc điểm chính

- Tối ưu cho tốc độ đọc.
- Hỗ trợ nén bảng thành bảng chỉ đọc.
- Có thể di chuyển giữa các nền tảng và hệ điều hành.
- Không hỗ trợ giao dịch.
- Không hỗ trợ khóa ngoại.

### Ưu điểm

- Truy vấn đọc có thể nhanh trong một số tình huống.
- Phù hợp với dữ liệu ít thay đổi.
- Có thể dùng cho các bảng chủ yếu phục vụ đọc dữ liệu.

### Hạn chế

- Không an toàn về giao dịch.
- Không hỗ trợ rollback.
- Không hỗ trợ khóa ngoại.
- Khả năng khôi phục sau sự cố kém hơn InnoDB.

### Ví dụ sử dụng

```sql
CREATE TABLE article_archive (
    article_id INT PRIMARY KEY,
    title VARCHAR(255),
    content TEXT
) ENGINE = MyISAM;
```

---

## 12. MERGE

`MERGE`, còn gọi là `MRG_MyISAM`, là storage engine cho phép kết hợp nhiều bảng `MyISAM` có cùng cấu trúc thành một bảng ảo.

### Đặc điểm chính

- Bảng `MERGE` không tự lưu dữ liệu.
- Dữ liệu thật nằm trong các bảng `MyISAM` thành phần.
- Bảng `MERGE` sử dụng chỉ mục của các bảng thành phần.
- Khi dùng `DROP TABLE` với bảng `MERGE`, MySQL chỉ xóa bảng `MERGE`, không xóa các bảng bên dưới.

### Ứng dụng

`MERGE` có thể được dùng khi muốn chia dữ liệu thành nhiều bảng nhỏ hơn nhưng vẫn truy vấn như một bảng thống nhất.

Ví dụ:

- Bảng doanh thu tháng 1.
- Bảng doanh thu tháng 2.
- Bảng doanh thu tháng 3.

Sau đó tạo bảng `MERGE` để truy vấn tổng hợp nhiều tháng.

---

## 13. MEMORY

`MEMORY` là storage engine lưu toàn bộ dữ liệu trong bộ nhớ RAM.

### Đặc điểm chính

- Dữ liệu được lưu trong bộ nhớ.
- Tốc độ truy cập nhanh.
- Thường dùng chỉ mục băm.
- Dữ liệu sẽ mất khi MySQL Server dừng hoặc khởi động lại.
- Trước đây còn được gọi là `HEAP`.

### Ưu điểm

- Truy xuất rất nhanh.
- Phù hợp với bảng tạm, dữ liệu trung gian, kết quả xử lý tạm thời.

### Hạn chế

- Không phù hợp để lưu dữ liệu lâu dài.
- Dữ liệu phụ thuộc vào thời gian hoạt động của máy chủ.
- Bị giới hạn bởi dung lượng RAM.

### Ví dụ sử dụng

```sql
CREATE TABLE session_cache (
    session_id VARCHAR(100) PRIMARY KEY,
    user_id INT,
    created_at DATETIME
) ENGINE = MEMORY;
```

---

## 14. ARCHIVE

`ARCHIVE` là storage engine dùng để lưu trữ số lượng lớn bản ghi ở dạng nén.

### Đặc điểm chính

- Dữ liệu được nén khi ghi.
- Dữ liệu được giải nén khi đọc.
- Phù hợp với dữ liệu lưu trữ lịch sử.
- Chủ yếu hỗ trợ `INSERT` và `SELECT`.
- Không hỗ trợ chỉ mục như các storage engine thông thường.
- Khi đọc thường phải quét toàn bảng.

### Ưu điểm

- Tiết kiệm dung lượng đĩa.
- Phù hợp với dữ liệu ít khi cập nhật.
- Thích hợp cho dữ liệu log, lịch sử, lưu trữ dài hạn.

### Hạn chế

- Không phù hợp với truy vấn cần hiệu năng cao.
- Không phù hợp với dữ liệu thường xuyên cập nhật.
- Việc đọc dữ liệu có thể chậm do phải quét toàn bảng.

### Ví dụ sử dụng

```sql
CREATE TABLE system_logs (
    log_id INT,
    log_message TEXT,
    created_at DATETIME
) ENGINE = ARCHIVE;
```

---

## 15. CSV

`CSV` là storage engine lưu dữ liệu dưới dạng tệp CSV, tức là dữ liệu được phân tách bằng dấu phẩy.

### Đặc điểm chính

- Dữ liệu được lưu dưới dạng văn bản CSV.
- Dễ trao đổi với các ứng dụng ngoài SQL.
- Có thể mở bằng phần mềm bảng tính.
- Không hỗ trợ kiểu dữ liệu `NULL`.
- Khi đọc dữ liệu thường cần quét toàn bảng.

### Ứng dụng

`CSV` phù hợp khi cần:

- Trao đổi dữ liệu với phần mềm bảng tính.
- Xuất nhập dữ liệu đơn giản.
- Lưu dữ liệu theo định dạng dễ đọc bởi con người.

### Ví dụ sử dụng

```sql
CREATE TABLE export_customers (
    customer_id INT NOT NULL,
    customer_name VARCHAR(100) NOT NULL,
    phone VARCHAR(20) NOT NULL
) ENGINE = CSV;
```

---

## 16. BLACKHOLE

`BLACKHOLE` là storage engine đặc biệt không lưu dữ liệu.

Điều này có nghĩa là mọi dữ liệu được ghi vào bảng `BLACKHOLE` sẽ bị loại bỏ.

### Đặc điểm chính

- Có thể ghi dữ liệu vào bảng.
- Dữ liệu không được lưu lại.
- Truy vấn đọc không trả về dữ liệu thực.
- Có thể hữu ích trong một số tình huống sao chép dữ liệu.

### Ứng dụng

`BLACKHOLE` có thể được dùng trong kịch bản replication, khi muốn ghi nhận thay đổi dữ liệu trên máy chủ chính nhưng không muốn lưu dữ liệu đó trên máy chủ cục bộ.

### Ví dụ sử dụng

```sql
CREATE TABLE event_sink (
    event_id INT,
    event_data TEXT
) ENGINE = BLACKHOLE;
```

---

## 17. FEDERATED

`FEDERATED` là storage engine cho phép quản lý dữ liệu từ một MySQL Server từ xa mà không cần dùng công nghệ cluster hoặc replication.

### Đặc điểm chính

- Bảng cục bộ không lưu dữ liệu thật.
- Khi truy vấn bảng cục bộ, dữ liệu được lấy từ bảng từ xa.
- Cho phép truy cập dữ liệu nằm trên MySQL Server khác.

### Ứng dụng

`FEDERATED` phù hợp khi cần:

- Truy vấn dữ liệu từ xa.
- Kết nối dữ liệu phân tán.
- Truy cập bảng ở máy chủ khác mà không sao chép dữ liệu về máy cục bộ.

### Hạn chế

- Phụ thuộc vào kết nối mạng.
- Hiệu năng có thể thấp hơn bảng cục bộ.
- Cấu hình phức tạp hơn các storage engine thông thường.

---

## 18. Ví dụ thực hành tổng hợp

### 18.1. Tạo bảng mặc định

```sql
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    price DECIMAL(10, 2)
);
```

Nếu không chỉ định `ENGINE`, MySQL sẽ dùng storage engine mặc định.

---

### 18.2. Tạo bảng InnoDB có khóa ngoại

```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100)
) ENGINE = InnoDB;

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    CONSTRAINT fk_orders_customers
        FOREIGN KEY (customer_id)
        REFERENCES customers(customer_id)
) ENGINE = InnoDB;
```

Bảng `orders` dùng khóa ngoại tham chiếu đến bảng `customers`, vì vậy nên dùng `InnoDB`.

---

### 18.3. Tạo bảng MEMORY cho dữ liệu tạm

```sql
CREATE TABLE temp_scores (
    student_id INT PRIMARY KEY,
    score DECIMAL(5, 2)
) ENGINE = MEMORY;
```

Bảng này phù hợp để lưu dữ liệu tạm trong quá trình xử lý.

---

### 18.4. Tạo bảng ARCHIVE cho dữ liệu log

```sql
CREATE TABLE audit_logs (
    log_id INT,
    action_name VARCHAR(100),
    action_time DATETIME
) ENGINE = ARCHIVE;
```

Bảng này phù hợp để lưu dữ liệu lịch sử với dung lượng lớn.

---

## 19. Hướng dẫn lựa chọn Storage Engine

| Nhu cầu sử dụng | Storage engine phù hợp |
|---|---|
| Cần giao dịch, rollback, khóa ngoại | `InnoDB` |
| Dữ liệu chủ yếu đọc, ít cập nhật | `MyISAM` |
| Dữ liệu tạm, cần tốc độ cao | `MEMORY` |
| Lưu trữ log, dữ liệu lịch sử | `ARCHIVE` |
| Trao đổi dữ liệu dạng CSV | `CSV` |
| Không cần lưu dữ liệu thật | `BLACKHOLE` |
| Truy cập dữ liệu từ MySQL Server khác | `FEDERATED` |
| Gộp nhiều bảng MyISAM cùng cấu trúc | `MERGE` |

---

## 20. Một số lưu ý quan trọng

1. `InnoDB` là lựa chọn mặc định và an toàn cho hầu hết các ứng dụng hiện đại.
2. Không nên dùng `MyISAM` nếu hệ thống cần giao dịch hoặc khóa ngoại.
3. `MEMORY` nhanh nhưng dữ liệu không bền vững sau khi máy chủ dừng.
4. `ARCHIVE` phù hợp với lưu trữ lâu dài nhưng không phù hợp với cập nhật thường xuyên.
5. `CSV` thuận tiện cho trao đổi dữ liệu nhưng hạn chế về hiệu năng và kiểu dữ liệu.
6. `BLACKHOLE` là storage engine đặc biệt, không lưu dữ liệu thật.
7. `FEDERATED` hữu ích cho dữ liệu từ xa nhưng cần chú ý hiệu năng mạng.

---

## 21. Câu hỏi ôn tập

### Câu hỏi 1

Storage engine trong MySQL dùng để làm gì?

### Câu hỏi 2

Câu lệnh nào dùng để liệt kê các storage engine đang có trong MySQL?

### Câu hỏi 3

Giá trị `DEFAULT` trong cột `Support` của lệnh `SHOW ENGINES` có ý nghĩa gì?

### Câu hỏi 4

Storage engine nào hỗ trợ giao dịch và khóa ngoại?

### Câu hỏi 5

Vì sao `MEMORY` không phù hợp để lưu dữ liệu quan trọng lâu dài?

### Câu hỏi 6

Storage engine nào phù hợp để lưu dữ liệu lịch sử với dung lượng lớn?

### Câu hỏi 7

Vì sao quan hệ khóa ngoại thường nên dùng với `InnoDB`?

---

## 22. Bài tập thực hành

### Bài tập 1: Kiểm tra storage engine

Viết câu lệnh SQL để liệt kê tất cả các storage engine có trong MySQL Server.

---

### Bài tập 2: Tạo bảng InnoDB

Tạo bảng `students` gồm các cột:

- `student_id`
- `student_name`
- `email`

Yêu cầu bảng sử dụng storage engine `InnoDB`.

---

### Bài tập 3: Tạo bảng MEMORY

Tạo bảng `temp_report` gồm các cột:

- `report_id`
- `report_name`
- `created_at`

Yêu cầu bảng sử dụng storage engine `MEMORY`.

---

### Bài tập 4: Phân tích tình huống

Một hệ thống bán hàng cần lưu thông tin khách hàng, đơn hàng và chi tiết đơn hàng. Hệ thống cần đảm bảo:

- Có thể rollback khi giao dịch lỗi.
- Đơn hàng phải tham chiếu đến khách hàng hợp lệ.
- Dữ liệu không bị mất khi máy chủ khởi động lại.

Hãy chọn storage engine phù hợp và giải thích lý do.

---

### Bài tập 5: So sánh storage engine

So sánh `InnoDB`, `MyISAM` và `MEMORY` theo các tiêu chí:

- Có hỗ trợ giao dịch không?
- Có hỗ trợ khóa ngoại không?
- Dữ liệu có bền vững không?
- Phù hợp với tình huống nào?

---

## 23. Đáp án gợi ý

<details>
<summary>Bấm để xem đáp án gợi ý</summary>

### Đáp án bài tập 1

```sql
SHOW ENGINES;
```

Hoặc:

```sql
SELECT 
  engine, 
  support 
FROM 
  information_schema.engines 
ORDER BY 
  engine;
```

### Đáp án bài tập 2

```sql
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    student_name VARCHAR(100),
    email VARCHAR(100)
) ENGINE = InnoDB;
```

### Đáp án bài tập 3

```sql
CREATE TABLE temp_report (
    report_id INT PRIMARY KEY,
    report_name VARCHAR(100),
    created_at DATETIME
) ENGINE = MEMORY;
```

### Đáp án bài tập 4

Nên chọn `InnoDB` vì:

- Hỗ trợ giao dịch.
- Hỗ trợ rollback.
- Hỗ trợ khóa ngoại.
- Dữ liệu được lưu bền vững.
- Phù hợp với hệ thống bán hàng cần tính toàn vẹn dữ liệu.

### Đáp án bài tập 5

| Tiêu chí | InnoDB | MyISAM | MEMORY |
|---|---|---|---|
| Hỗ trợ giao dịch | Có | Không | Không |
| Hỗ trợ khóa ngoại | Có | Không | Không |
| Dữ liệu bền vững | Có | Có | Không, dữ liệu mất khi server dừng |
| Tình huống phù hợp | Hệ thống giao dịch | Dữ liệu đọc nhiều, ít cập nhật | Dữ liệu tạm, cache |

</details>

---

## 24. Tổng kết

Storage engine quyết định cách MySQL lưu trữ, truy xuất và xử lý dữ liệu trong bảng.

Các ý chính cần nhớ:

- `InnoDB` là storage engine mặc định và phù hợp với hầu hết ứng dụng hiện đại.
- `InnoDB` hỗ trợ giao dịch, khóa ngoại và khôi phục sau sự cố.
- `MyISAM` nhanh trong một số tình huống đọc dữ liệu nhưng không hỗ trợ giao dịch.
- `MEMORY` lưu dữ liệu trong RAM, phù hợp cho dữ liệu tạm.
- `ARCHIVE` phù hợp cho lưu trữ dữ liệu lịch sử dạng nén.
- `CSV` thuận tiện cho trao đổi dữ liệu với phần mềm ngoài SQL.
- `BLACKHOLE` không lưu dữ liệu thật.
- `FEDERATED` cho phép truy cập dữ liệu từ máy chủ MySQL từ xa.
- Việc lựa chọn đúng storage engine có ảnh hưởng trực tiếp đến hiệu năng, tính toàn vẹn và độ an toàn của hệ thống cơ sở dữ liệu.

