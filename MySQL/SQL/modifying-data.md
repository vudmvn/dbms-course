---
title: "Tutorial: Thao tác dữ liệu nâng cao trong MySQL"
author: "Tên giảng viên"
duration: "180m"
difficulty: "Intermediate"
prerequisites:
    - "Đã biết CREATE TABLE, PRIMARY KEY, FOREIGN KEY và các kiểu dữ liệu cơ bản"
    - "Đã biết SELECT, WHERE, JOIN và subquery cơ bản"
    - "Đã cài MySQL và có cơ sở dữ liệu mẫu classicmodels"
summary: "Thực hành INSERT, INSERT multiple rows, INSERT INTO SELECT, INSERT IGNORE, DATE/DATETIME, UPDATE, UPDATE JOIN, DELETE, ON DELETE CASCADE, DELETE JOIN và REPLACE trong MySQL."
---

# Tutorial: Thao tác dữ liệu nâng cao trong MySQL

## Link tham khảo

- [MySQL Tutorial — MySQL Basics, Section 12: Modifying data in MySQL](https://www.mysqltutorial.org/mysql-basics/)
- [MySQL Reference Manual — INSERT Statement](https://dev.mysql.com/doc/refman/8.4/en/insert.html)
- [MySQL Reference Manual — INSERT ... SELECT](https://dev.mysql.com/doc/refman/8.4/en/insert-select.html)
- [MySQL Reference Manual — UPDATE Statement](https://dev.mysql.com/doc/refman/8.4/en/update.html)
- [MySQL Reference Manual — DELETE Statement](https://dev.mysql.com/doc/refman/8.4/en/delete.html)
- [MySQL Reference Manual — FOREIGN KEY Constraints](https://dev.mysql.com/doc/refman/8.4/en/create-table-foreign-keys.html)
- [MySQL Reference Manual — REPLACE Statement](https://dev.mysql.com/doc/refman/8.4/en/replace.html)

## 1. Mục tiêu học tập

Sau khi hoàn thành tutorial này, người học có thể:

1. Chèn một dòng hoặc nhiều dòng dữ liệu bằng `INSERT`.
2. Chèn dữ liệu từ kết quả truy vấn bằng `INSERT ... SELECT`.
3. Dùng `INSERT IGNORE` và hiểu rủi ro khi che giấu lỗi dữ liệu.
4. Chèn giá trị `DATE` và `DATETIME` đúng định dạng.
5. Cập nhật dữ liệu bằng `UPDATE`.
6. Cập nhật dữ liệu dựa trên bảng khác bằng `UPDATE ... JOIN`.
7. Xóa một hoặc nhiều dòng bằng `DELETE`.
8. Giải thích và kiểm tra tác động của `ON DELETE CASCADE`.
9. Xóa dữ liệu dựa trên điều kiện liên bảng bằng `DELETE ... JOIN`.
10. Phân biệt `REPLACE` với `INSERT` và `UPDATE`.
11. Sử dụng transaction để kiểm tra thay đổi trước khi lưu vĩnh viễn.
12. Thực hành an toàn trên một schema lab riêng và chỉ đọc dữ liệu từ `classicmodels`.

---

## 2. Tổng quan: các thao tác thay đổi dữ liệu

Nhóm câu lệnh DML (*Data Manipulation Language*) dùng để thêm, sửa và xóa dữ liệu.

| Câu lệnh | Mục đích |
|---|---|
| `INSERT` | Chèn một hoặc nhiều dòng mới |
| `INSERT ... SELECT` | Chèn dữ liệu lấy từ kết quả truy vấn |
| `INSERT IGNORE` | Chèn dữ liệu và chuyển một số lỗi thành warning |
| `UPDATE` | Cập nhật các dòng đã tồn tại |
| `UPDATE ... JOIN` | Cập nhật dựa trên dữ liệu của bảng liên quan |
| `DELETE` | Xóa các dòng thỏa điều kiện |
| `DELETE ... JOIN` | Xóa dựa trên điều kiện liên bảng |
| `REPLACE` | Thay thế dòng xung đột khóa bằng một dòng mới |

> **Nguyên tắc quan trọng:** Trước `UPDATE` và `DELETE`, hãy chạy `SELECT` với cùng điều kiện `WHERE` hoặc cùng điều kiện `JOIN` để kiểm tra các dòng sẽ bị ảnh hưởng.

### Bài tập thực hành

**Bài 2.1.** Câu lệnh nào dùng để thêm dữ liệu mới vào bảng?

**Bài 2.2.** Câu lệnh nào dùng để cập nhật dữ liệu đã tồn tại?

**Bài 2.3.** Câu lệnh nào dùng để xóa dữ liệu?

**Bài 2.4.** Câu lệnh nào dùng để cập nhật một bảng dựa trên dữ liệu của bảng khác?

**Bài 2.5.** Vì sao cần chạy `SELECT` trước khi thực hiện `UPDATE` hoặc `DELETE`?

---

## 3. Chuẩn bị môi trường thực hành

Tutorial sử dụng một schema lab riêng để tránh làm thay đổi dữ liệu gốc của `classicmodels`.

> Chạy toàn bộ phần thiết lập này **một lần** trước khi làm bài. Lệnh `DROP DATABASE` ở dưới chỉ áp dụng cho schema lab có tên `lab_modifying_data`; không chạy lệnh đó với `classicmodels`.

```sql
DROP DATABASE IF EXISTS lab_modifying_data;

CREATE DATABASE lab_modifying_data;

USE lab_modifying_data;
```

### 3.1. Tạo các bảng lab

```sql
CREATE TABLE lab_department (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(100) NOT NULL UNIQUE
) ENGINE=InnoDB;
```

```sql
CREATE TABLE lab_department_bonus (
    department_id INT PRIMARY KEY,
    bonus_rate DECIMAL(5,4) NOT NULL,
    CONSTRAINT fk_lab_bonus_department
        FOREIGN KEY (department_id)
        REFERENCES lab_department(department_id)
        ON DELETE CASCADE
) ENGINE=InnoDB;
```

```sql
CREATE TABLE lab_employee (
    employee_id INT PRIMARY KEY,
    department_id INT NULL,
    full_name VARCHAR(100) NOT NULL,
    email VARCHAR(120) NOT NULL UNIQUE,
    salary DECIMAL(12,2) NOT NULL DEFAULT 0.00,
    bonus_rate DECIMAL(5,4) NOT NULL DEFAULT 0.0000,
    hire_date DATE NOT NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_lab_employee_department
        FOREIGN KEY (department_id)
        REFERENCES lab_department(department_id)
        ON DELETE SET NULL
) ENGINE=InnoDB;
```

```sql
CREATE TABLE lab_event (
    event_id INT PRIMARY KEY,
    event_name VARCHAR(150) NOT NULL,
    event_date DATE NOT NULL,
    starts_at DATETIME NOT NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB;
```

```sql
CREATE TABLE lab_customer (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(150) NOT NULL,
    status ENUM('ACTIVE', 'INACTIVE') NOT NULL DEFAULT 'ACTIVE'
) ENGINE=InnoDB;
```

```sql
CREATE TABLE lab_sales_order (
    order_id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date DATE NOT NULL,
    order_status VARCHAR(30) NOT NULL,
    CONSTRAINT fk_lab_order_customer
        FOREIGN KEY (customer_id)
        REFERENCES lab_customer(customer_id)
        ON DELETE CASCADE
) ENGINE=InnoDB;
```

```sql
CREATE TABLE lab_sales_order_item (
    order_id INT NOT NULL,
    line_number INT NOT NULL,
    item_name VARCHAR(150) NOT NULL,
    quantity INT NOT NULL,
    PRIMARY KEY (order_id, line_number),
    CONSTRAINT fk_lab_order_item_order
        FOREIGN KEY (order_id)
        REFERENCES lab_sales_order(order_id)
        ON DELETE CASCADE
) ENGINE=InnoDB;
```

```sql
CREATE TABLE lab_customer_copy (
    customer_number INT PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL,
    country VARCHAR(50) NOT NULL,
    credit_limit DECIMAL(12,2) NULL
) ENGINE=InnoDB;
```

```sql
CREATE TABLE lab_product_stock (
    product_code VARCHAR(30) PRIMARY KEY,
    product_name VARCHAR(150) NOT NULL,
    quantity_in_stock INT NOT NULL DEFAULT 0,
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB;
```

```sql
CREATE TABLE lab_session (
    session_id INT PRIMARY KEY,
    user_name VARCHAR(100) NOT NULL,
    session_status VARCHAR(30) NOT NULL
) ENGINE=InnoDB;
```

```sql
CREATE TABLE lab_session_audit (
    audit_id INT PRIMARY KEY,
    session_id INT NOT NULL,
    action_name VARCHAR(100) NOT NULL
) ENGINE=InnoDB;
```

### 3.2. Chèn dữ liệu mẫu

```sql
INSERT INTO lab_department (department_id, department_name)
VALUES
    (1, 'Sales'),
    (2, 'Finance'),
    (3, 'Support');
```

```sql
INSERT INTO lab_department_bonus (department_id, bonus_rate)
VALUES
    (1, 0.1000),
    (2, 0.0500);
```

```sql
INSERT INTO lab_employee (
    employee_id,
    department_id,
    full_name,
    email,
    salary,
    bonus_rate,
    hire_date,
    created_at
)
VALUES
    (101, 1, 'Nguyen An', 'nguyen.an@example.com', 1500.00, 0.0000, '2024-01-15', '2024-01-15 08:30:00'),
    (102, 1, 'Tran Binh', 'tran.binh@example.com', 1800.00, 0.0000, '2024-03-10', '2024-03-10 09:15:00'),
    (103, 2, 'Le Chi', 'le.chi@example.com', 2200.00, 0.0000, '2023-08-20', '2023-08-20 10:00:00'),
    (104, 3, 'Pham Dung', 'pham.dung@example.com', 1300.00, 0.0000, '2025-02-01', '2025-02-01 08:00:00'),
    (105, NULL, 'Hoang Giang', 'hoang.giang@example.com', 1200.00, 0.0000, '2025-05-05', '2025-05-05 08:45:00');
```

```sql
INSERT INTO lab_customer (customer_id, customer_name, status)
VALUES
    (501, 'Alpha Trading', 'ACTIVE'),
    (502, 'Beta Retail', 'ACTIVE'),
    (503, 'Gamma Services', 'INACTIVE');
```

```sql
INSERT INTO lab_sales_order (order_id, customer_id, order_date, order_status)
VALUES
    (10001, 501, '2026-01-10', 'NEW'),
    (10002, 501, '2026-01-12', 'PAID'),
    (10003, 502, '2026-01-15', 'NEW'),
    (10004, 503, '2026-01-20', 'CANCELLED');
```

```sql
INSERT INTO lab_sales_order_item (order_id, line_number, item_name, quantity)
VALUES
    (10001, 1, 'Laptop Stand', 2),
    (10001, 2, 'Wireless Mouse', 3),
    (10002, 1, 'Mechanical Keyboard', 1),
    (10003, 1, 'USB-C Hub', 4),
    (10004, 1, 'Monitor Arm', 1);
```

```sql
INSERT INTO lab_product_stock (
    product_code,
    product_name,
    quantity_in_stock,
    updated_at
)
VALUES
    ('P100', 'USB-C Hub', 15, '2026-01-01 08:00:00'),
    ('P200', 'Wireless Mouse', 30, '2026-01-01 08:00:00');
```

```sql
INSERT INTO lab_session (session_id, user_name, session_status)
VALUES
    (1, 'minh', 'EXPIRED'),
    (2, 'an', 'ACTIVE'),
    (3, 'binh', 'EXPIRED');
```

```sql
INSERT INTO lab_session_audit (audit_id, session_id, action_name)
VALUES
    (1001, 1, 'login'),
    (1002, 1, 'view_profile'),
    (1003, 2, 'login'),
    (1004, 3, 'login');
```

### 3.3. Bắt đầu mỗi phần thực hành

Dùng schema lab:

```sql
USE lab_modifying_data;
```

Khi một ví dụ làm thay đổi dữ liệu, có thể bao quanh bằng transaction:

```sql
START TRANSACTION;

-- Câu lệnh INSERT, UPDATE, DELETE hoặc REPLACE

ROLLBACK;
```

`ROLLBACK` hủy các thay đổi chưa `COMMIT`. Trong giờ thực hành, dùng `ROLLBACK` để dữ liệu mẫu luôn giữ nguyên cho bài sau.

### Bài tập thực hành

**Bài 3.1.** Tạo schema `lab_modifying_data`.

**Bài 3.2.** Tạo bảng `lab_department` với khóa chính và unique constraint như phần thiết lập.

**Bài 3.3.** Chèn dữ liệu mẫu vào `lab_department`.

**Bài 3.4.** Viết truy vấn kiểm tra tất cả dòng của `lab_employee`.

**Bài 3.5.** Viết khung transaction gồm `START TRANSACTION` và `ROLLBACK`.

---

# Phần A. INSERT

## 4. INSERT: Chèn một dòng dữ liệu

`INSERT` dùng để thêm một dòng dữ liệu mới vào bảng.

Cú pháp tổng quát:

```sql
INSERT INTO table_name (
    column1,
    column2,
    ...
)
VALUES (
    value1,
    value2,
    ...
);
```

Ví dụ: chèn một employee mới.

```sql
START TRANSACTION;

INSERT INTO lab_employee (
    employee_id,
    department_id,
    full_name,
    email,
    salary,
    bonus_rate,
    hire_date,
    created_at
)
VALUES (
    106,
    3,
    'Vu Hai',
    'vu.hai@example.com',
    1400.00,
    0.0000,
    '2026-02-10',
    '2026-02-10 08:30:00'
);

SELECT *
FROM lab_employee
WHERE employee_id = 106;

ROLLBACK;
```

### Lưu ý

- Nên ghi rõ danh sách cột.
- Số giá trị trong `VALUES` phải khớp với số cột đã liệt kê.
- Thứ tự giá trị phải đúng với thứ tự cột.
- Chuỗi, ngày và thời điểm phải đặt trong dấu nháy đơn.
- `NULL` không đặt trong dấu nháy đơn.
- Sau DML, có thể dùng `SELECT ROW_COUNT();` ngay lập tức để kiểm tra số dòng ảnh hưởng.

### Bài tập thực hành

**Bài 4.1.** Chèn một employee có mã `106`, thuộc department `3`, tên `Vu Hai`, email `vu.hai@example.com`, lương `1400.00`, ngày vào làm `2026-02-10`.

**Bài 4.2.** Chèn một employee có mã `107`, không thuộc department nào, tên `Do Khang`, email `do.khang@example.com`, lương `1600.00`.

**Bài 4.3.** Chèn một event có mã `1`, tên `SQL Workshop`, `event_date = '2026-06-15'`, `starts_at = '2026-06-15 08:30:00'`.

**Bài 4.4.** Chèn một customer có mã `504`, tên `Delta Import`, trạng thái `ACTIVE`.

**Bài 4.5.** Chèn một sản phẩm vào `lab_product_stock` có mã `P300`, tên `Laptop Stand`, tồn kho `20`, thời điểm cập nhật `2026-02-01 09:00:00`.

---

## 5. INSERT Multiple Rows: Chèn nhiều dòng

Có thể chèn nhiều dòng bằng một câu lệnh `INSERT`.

Cú pháp:

```sql
INSERT INTO table_name (column1, column2, ...)
VALUES
    (value_row_1_column_1, value_row_1_column_2, ...),
    (value_row_2_column_1, value_row_2_column_2, ...),
    ...;
```

Ví dụ: chèn hai department.

```sql
START TRANSACTION;

INSERT INTO lab_department (department_id, department_name)
VALUES
    (4, 'Marketing'),
    (5, 'Operations');

SELECT *
FROM lab_department
WHERE department_id IN (4, 5);

ROLLBACK;
```

### Lưu ý

- Mỗi nhóm trong `VALUES (...)` là một dòng.
- Mọi dòng phải có số giá trị khớp với danh sách cột.
- Một lỗi có thể làm cả câu lệnh thất bại, tùy loại lỗi và chế độ SQL.
- Khi cần xử lý từng dòng độc lập hoặc cần bỏ qua lỗi có chủ đích, cân nhắc `INSERT IGNORE` nhưng phải kiểm tra warning.

### Bài tập thực hành

**Bài 5.1.** Dùng một `INSERT` để chèn department `(4, 'Marketing')` và `(5, 'Operations')`.

**Bài 5.2.** Dùng một `INSERT` để chèn ba event có mã `2`, `3`, `4` với ba ngày khác nhau.

**Bài 5.3.** Dùng một `INSERT` để chèn customer `505`, `506`, `507` với các trạng thái `ACTIVE`, `INACTIVE`, `ACTIVE`.

**Bài 5.4.** Dùng một `INSERT` để chèn sản phẩm `P301`, `P302`, `P303` vào `lab_product_stock`.

**Bài 5.5.** Dùng một `INSERT` để chèn hai employee mới thuộc department `1` và một employee mới thuộc department `2`.

---

## 6. INSERT INTO SELECT: Chèn từ kết quả truy vấn

`INSERT ... SELECT` chèn dữ liệu vào bảng đích từ result set của một câu truy vấn.

Cú pháp:

```sql
INSERT INTO target_table (target_column1, target_column2, ...)
SELECT source_expression1, source_expression2, ...
FROM source_table
WHERE condition;
```

Ví dụ: sao chép khách hàng ở France từ `classicmodels.customers` vào bảng lab.

```sql
START TRANSACTION;

INSERT INTO lab_customer_copy (
    customer_number,
    customer_name,
    country,
    credit_limit
)
SELECT
    customerNumber,
    customerName,
    country,
    creditLimit
FROM classicmodels.customers
WHERE country = 'France';

SELECT *
FROM lab_customer_copy
WHERE country = 'France';

ROLLBACK;
```

Ví dụ: chèn employee có lương cao vào bảng tạm được tạo trước.

```sql
CREATE TABLE IF NOT EXISTS lab_high_salary_employee (
    employee_id INT PRIMARY KEY,
    full_name VARCHAR(100) NOT NULL,
    salary DECIMAL(12,2) NOT NULL
) ENGINE=InnoDB;
```

```sql
START TRANSACTION;

INSERT INTO lab_high_salary_employee (
    employee_id,
    full_name,
    salary
)
SELECT
    employee_id,
    full_name,
    salary
FROM lab_employee
WHERE salary >= 1800.00;

SELECT *
FROM lab_high_salary_employee;

ROLLBACK;
```

### Lưu ý

- Số biểu thức trong phần `SELECT` phải khớp với số cột đích.
- Kiểu dữ liệu của biểu thức nguồn phải tương thích với cột đích.
- Câu lệnh có thể chọn dữ liệu từ một hoặc nhiều bảng.
- Khi đọc và ghi cùng một bảng trong `INSERT ... SELECT`, cần lưu ý các giới hạn và hành vi đặc thù của MySQL; với người mới, nên dùng bảng nguồn và bảng đích khác nhau.

### Bài tập thực hành

**Bài 6.1.** Chèn khách hàng ở `Germany` từ `classicmodels.customers` vào `lab_customer_copy`.

**Bài 6.2.** Chèn khách hàng có `creditLimit >= 100000` từ `classicmodels.customers` vào `lab_customer_copy`.

**Bài 6.3.** Chèn các employee có lương từ `1500.00` trở lên vào `lab_high_salary_employee`.

**Bài 6.4.** Chèn các employee thuộc department `1` vào `lab_high_salary_employee`.

**Bài 6.5.** Chèn các customer trong `classicmodels.customers` ở `USA` vào `lab_customer_copy`, chỉ lấy `customerNumber`, `customerName`, `country`, `creditLimit`.

---

## 7. INSERT IGNORE: Chèn và bỏ qua một số lỗi

`INSERT IGNORE` yêu cầu MySQL tiếp tục xử lý câu lệnh trong một số trường hợp lỗi thay vì dừng ngay. Các lỗi có thể được chuyển thành warning; tùy loại lỗi, một dòng có thể bị bỏ qua hoặc một giá trị có thể bị điều chỉnh.

Cú pháp:

```sql
INSERT IGNORE INTO table_name (column1, column2, ...)
VALUES (...), (...);
```

Ví dụ: department `1` đã tồn tại nên dòng đó bị bỏ qua, còn department `4` có thể được chèn.

```sql
START TRANSACTION;

INSERT IGNORE INTO lab_department (
    department_id,
    department_name
)
VALUES
    (1, 'Sales Duplicate'),
    (4, 'Marketing');

SELECT *
FROM lab_department
WHERE department_id IN (1, 4);

SHOW WARNINGS;

ROLLBACK;
```

### Lưu ý quan trọng

`INSERT IGNORE` hữu ích khi xử lý dữ liệu nhập hàng loạt mà việc bỏ qua xung đột đã được chấp nhận theo quy tắc nghiệp vụ.

Tuy nhiên, không nên dùng `INSERT IGNORE` chỉ để “làm biến mất lỗi”, vì:

- Dòng trùng có thể bị bỏ qua mà người dùng không nhận ra.
- Dữ liệu không hợp lệ có thể bị điều chỉnh ngoài ý muốn.
- Chất lượng dữ liệu có thể giảm.
- Cần luôn kiểm tra `SHOW WARNINGS` sau khi chạy.

### Bài tập thực hành

**Bài 7.1.** Dùng `INSERT IGNORE` để chèn lại department `(1, 'Sales Duplicate')` và chèn mới department `(4, 'Marketing')`.

**Bài 7.2.** Dùng `INSERT IGNORE` để chèn product `P100` với tên khác và product `P400` với tên `Monitor Arm`.

**Bài 7.3.** Dùng `INSERT IGNORE` để chèn event có mã `1` và event có mã `5` trong cùng câu lệnh.

**Bài 7.4.** Sau Bài 7.1, chạy `SHOW WARNINGS` và giải thích warning xuất hiện.

**Bài 7.5.** Nêu một tình huống thực tế mà `INSERT IGNORE` có thể phù hợp và một tình huống không nên dùng.

---

## 8. Chèn giá trị DATETIME

Kiểu `DATETIME` lưu cả ngày và thời điểm trong ngày.

Định dạng phổ biến:

```text
'YYYY-MM-DD HH:MM:SS'
```

Ví dụ:

```sql
START TRANSACTION;

INSERT INTO lab_event (
    event_id,
    event_name,
    event_date,
    starts_at
)
VALUES (
    10,
    'Database Design Seminar',
    '2026-07-10',
    '2026-07-10 14:30:00'
);

SELECT *
FROM lab_event
WHERE event_id = 10;

ROLLBACK;
```

Có thể dùng các hàm thời gian hiện tại:

```sql
INSERT INTO lab_event (
    event_id,
    event_name,
    event_date,
    starts_at
)
VALUES (
    11,
    'Current Time Demo',
    CURDATE(),
    NOW()
);
```

| Hàm | Kết quả |
|---|---|
| `CURDATE()` | Ngày hiện tại, kiểu DATE |
| `NOW()` | Ngày và giờ hiện tại, kiểu DATETIME |
| `CURRENT_TIMESTAMP` | Thường tương đương `NOW()` trong ngữ cảnh này |

### Bài tập thực hành

**Bài 8.1.** Chèn event `12`, tên `Morning SQL Lab`, ngày `2026-07-15`, bắt đầu lúc `2026-07-15 08:00:00`.

**Bài 8.2.** Chèn event `13`, tên `Afternoon SQL Lab`, ngày `2026-07-15`, bắt đầu lúc `2026-07-15 13:30:00`.

**Bài 8.3.** Chèn event `14`, tên `Current Datetime Event`, sử dụng `CURDATE()` cho `event_date` và `NOW()` cho `starts_at`.

**Bài 8.4.** Chèn event `15`, tên `Evening Workshop`, bắt đầu lúc `2026-12-31 23:59:59`.

**Bài 8.5.** Viết câu `INSERT` có giá trị DATETIME không hợp lệ và giải thích vì sao không nên dùng định dạng đó.

---

## 9. Chèn giá trị DATE

Kiểu `DATE` lưu ngày mà không lưu thông tin giờ, phút, giây.

Định dạng phổ biến:

```text
'YYYY-MM-DD'
```

Ví dụ:

```sql
START TRANSACTION;

INSERT INTO lab_employee (
    employee_id,
    department_id,
    full_name,
    email,
    salary,
    bonus_rate,
    hire_date,
    created_at
)
VALUES (
    108,
    2,
    'Bui Lan',
    'bui.lan@example.com',
    1700.00,
    0.0000,
    '2026-09-01',
    '2026-09-01 08:00:00'
);

SELECT employee_id, full_name, hire_date
FROM lab_employee
WHERE employee_id = 108;

ROLLBACK;
```

Ví dụ dùng `CURDATE()`:

```sql
INSERT INTO lab_customer (
    customer_id,
    customer_name,
    status
)
VALUES (
    504,
    'Delta Import',
    'ACTIVE'
);
```

> Bảng `lab_customer` không có cột DATE. Ví dụ này nhắc lại rằng chỉ dùng `CURDATE()` khi cột đích có kiểu DATE hoặc kiểu dữ liệu tương thích. Với `lab_event`, cột `event_date` phù hợp hơn.

```sql
INSERT INTO lab_event (
    event_id,
    event_name,
    event_date,
    starts_at
)
VALUES (
    16,
    'Today Event',
    CURDATE(),
    NOW()
);
```

### Bài tập thực hành

**Bài 9.1.** Chèn employee `109` với `hire_date = '2026-08-20'`.

**Bài 9.2.** Chèn employee `110` với `hire_date = CURDATE()`.

**Bài 9.3.** Chèn event `17` có `event_date = '2027-01-01'`.

**Bài 9.4.** Chèn event `18` có `event_date = CURDATE()`.

**Bài 9.5.** Giải thích sự khác nhau giữa `DATE` và `DATETIME`.

---

# Phần B. UPDATE

## 10. UPDATE: Cập nhật dữ liệu trong một bảng

`UPDATE` thay đổi giá trị của một hoặc nhiều cột trong các dòng đã tồn tại.

Cú pháp:

```sql
UPDATE table_name
SET
    column1 = expression1,
    column2 = expression2,
    ...
WHERE condition;
```

Ví dụ: tăng lương của employee `101`.

```sql
START TRANSACTION;

SELECT employee_id, full_name, salary
FROM lab_employee
WHERE employee_id = 101;

UPDATE lab_employee
SET salary = salary + 100.00
WHERE employee_id = 101;

SELECT employee_id, full_name, salary
FROM lab_employee
WHERE employee_id = 101;

ROLLBACK;
```

Ví dụ cập nhật nhiều cột:

```sql
UPDATE lab_employee
SET
    department_id = 2,
    salary = 2000.00,
    bonus_rate = 0.0500
WHERE employee_id = 102;
```

### Lưu ý

- Luôn dùng `WHERE` nếu không thực sự muốn thay đổi mọi dòng.
- Chạy `SELECT` trước với cùng điều kiện.
- `UPDATE` có thể ảnh hưởng một hoặc nhiều dòng.
- `ROW_COUNT()` nên được gọi ngay sau `UPDATE`.
- Cần cẩn thận khi thay đổi cột tham gia khóa chính, khóa ngoại hoặc unique constraint.

### Bài tập thực hành

**Bài 10.1.** Tăng lương employee `101` thêm `200.00`.

**Bài 10.2.** Cập nhật email employee `102` thành `tran.binh.new@example.com`.

**Bài 10.3.** Cập nhật employee `103` sang department `1` và đặt `bonus_rate = 0.1000`.

**Bài 10.4.** Cập nhật status customer `503` từ `INACTIVE` thành `ACTIVE`.

**Bài 10.5.** Cập nhật `order_status` của order `10001` thành `PAID`.

---

## 11. UPDATE JOIN: Cập nhật dựa trên bảng liên quan

`UPDATE ... JOIN` cho phép cập nhật một bảng dựa trên dữ liệu từ bảng khác.

### 11.1. UPDATE với INNER JOIN

Cú pháp:

```sql
UPDATE table_1 AS t1
INNER JOIN table_2 AS t2
    ON t1.join_column = t2.join_column
SET t1.column_to_update = expression
WHERE condition;
```

Ví dụ: cập nhật `bonus_rate` của employee theo bảng `lab_department_bonus`.

```sql
START TRANSACTION;

SELECT
    e.employee_id,
    e.full_name,
    e.department_id,
    e.bonus_rate,
    b.bonus_rate AS department_bonus_rate
FROM lab_employee AS e
JOIN lab_department_bonus AS b
    ON e.department_id = b.department_id;

UPDATE lab_employee AS e
INNER JOIN lab_department_bonus AS b
    ON e.department_id = b.department_id
SET e.bonus_rate = b.bonus_rate;

SELECT employee_id, full_name, department_id, bonus_rate
FROM lab_employee
ORDER BY employee_id;

ROLLBACK;
```

### 11.2. UPDATE với LEFT JOIN

`LEFT JOIN` hữu ích khi muốn cập nhật cả các dòng không có bản ghi phù hợp ở bảng bên phải.

Ví dụ: employee không có bonus rule sẽ nhận bonus rate `0`.

```sql
START TRANSACTION;

UPDATE lab_employee AS e
LEFT JOIN lab_department_bonus AS b
    ON e.department_id = b.department_id
SET e.bonus_rate = COALESCE(b.bonus_rate, 0.0000);

SELECT employee_id, department_id, bonus_rate
FROM lab_employee
ORDER BY employee_id;

ROLLBACK;
```

### Lưu ý

- Với multiple-table `UPDATE`, MySQL cập nhật mỗi dòng khớp tối đa một lần.
- Trong cú pháp multiple-table, không dùng `ORDER BY` hoặc `LIMIT`.
- Trước khi `UPDATE`, hãy chạy `SELECT` với cùng `JOIN` và `WHERE`.
- Cần phân biệt bảng nào là bảng dùng để đọc điều kiện và bảng nào thực sự bị cập nhật.

### Bài tập thực hành

**Bài 11.1.** Dùng `UPDATE ... INNER JOIN` để cập nhật `bonus_rate` cho employee theo `lab_department_bonus`.

**Bài 11.2.** Dùng `UPDATE ... LEFT JOIN` để gán `bonus_rate = 0.0000` cho employee không có dòng tương ứng trong `lab_department_bonus`.

**Bài 11.3.** Dùng `UPDATE ... INNER JOIN` để tăng lương employee thuộc department có `bonus_rate = 0.1000` thêm 5%.

**Bài 11.4.** Dùng `UPDATE ... LEFT JOIN` để gán `bonus_rate = 0.0000` cho employee có `department_id IS NULL`.

**Bài 11.5.** Viết câu `SELECT` tương ứng để xem employee nào sẽ bị ảnh hưởng trước khi thực hiện Bài 11.3.

---

# Phần C. DELETE

## 12. DELETE: Xóa dữ liệu trong một bảng

`DELETE` xóa các dòng thỏa điều kiện.

Cú pháp:

```sql
DELETE FROM table_name
WHERE condition;
```

Ví dụ: xóa một event.

```sql
START TRANSACTION;

SELECT *
FROM lab_event
WHERE event_id = 1;

DELETE FROM lab_event
WHERE event_id = 1;

SELECT *
FROM lab_event
WHERE event_id = 1;

ROLLBACK;
```

Ví dụ xóa nhiều event theo điều kiện:

```sql
DELETE FROM lab_event
WHERE event_date < '2026-01-01';
```

### Lưu ý

- `DELETE FROM table_name;` không có `WHERE` sẽ xóa mọi dòng trong bảng.
- Luôn kiểm tra bằng `SELECT` trước.
- `DELETE` không xóa cấu trúc bảng.
- Khóa ngoại có thể ngăn xóa nếu còn bảng con tham chiếu.
- Dùng transaction khi thử thao tác xóa.

### Bài tập thực hành

**Bài 12.1.** Xóa event có `event_id = 1`.

**Bài 12.2.** Xóa event có `event_id = 2`.

**Bài 12.3.** Xóa các event có `event_date` trước `2026-01-01`.

**Bài 12.4.** Xóa customer có `customer_id = 503` bằng `DELETE` đơn bảng và quan sát kết quả.

**Bài 12.5.** Viết câu `SELECT` cần chạy trước khi xóa employee có `employee_id = 105`.

---

## 13. ON DELETE CASCADE: Xóa tự động ở bảng con

`ON DELETE CASCADE` là một referential action của foreign key.

Khi một dòng ở bảng cha bị xóa, các dòng khớp ở bảng con cũng bị xóa tự động.

Trong schema lab:

```text
lab_customer
    └── lab_sales_order
            └── lab_sales_order_item
```

Các foreign key đã được khai báo:

```sql
FOREIGN KEY (customer_id)
REFERENCES lab_customer(customer_id)
ON DELETE CASCADE
```

và:

```sql
FOREIGN KEY (order_id)
REFERENCES lab_sales_order(order_id)
ON DELETE CASCADE
```

Ví dụ: xóa customer `501`.

```sql
START TRANSACTION;

SELECT *
FROM lab_customer
WHERE customer_id = 501;

SELECT *
FROM lab_sales_order
WHERE customer_id = 501;

SELECT i.*
FROM lab_sales_order_item AS i
JOIN lab_sales_order AS o
    ON i.order_id = o.order_id
WHERE o.customer_id = 501;

DELETE FROM lab_customer
WHERE customer_id = 501;

SELECT *
FROM lab_sales_order
WHERE customer_id = 501;

ROLLBACK;
```

Sau câu `DELETE`, các order của customer `501` và các order item thuộc các order đó sẽ bị xóa theo cascade.

### Lưu ý

- `ON DELETE CASCADE` chỉ nên dùng khi xóa dữ liệu con là đúng về mặt nghiệp vụ.
- Cascade giúp giảm thao tác xóa thủ công nhưng có thể xóa nhiều dữ liệu hơn dự kiến.
- Không nên xem cascade là cách thay thế cho việc hiểu quan hệ dữ liệu.
- Trước khi xóa parent, cần kiểm tra số lượng child rows bị ảnh hưởng.

### Bài tập thực hành

**Bài 13.1.** Kiểm tra các order và order item của customer `501` trước khi xóa.

**Bài 13.2.** Xóa customer `501` trong transaction và kiểm tra các order liên quan sau khi xóa.

**Bài 13.3.** Xóa customer `502` trong transaction và kiểm tra order item của order `10003`.

**Bài 13.4.** Giải thích vì sao xóa customer `501` có thể làm mất nhiều hơn một dòng.

**Bài 13.5.** Viết một câu `SELECT` đếm số order và một câu `SELECT` đếm số order item sẽ bị ảnh hưởng khi xóa customer `501`.

---

## 14. DELETE JOIN: Xóa dựa trên điều kiện liên bảng

MySQL hỗ trợ multiple-table `DELETE`, cho phép chọn bảng cần xóa và điều kiện liên kết.

### 14.1. Xóa một bảng dựa trên JOIN

Cú pháp:

```sql
DELETE target_alias
FROM target_table AS target_alias
JOIN related_table AS related_alias
    ON target_alias.join_column = related_alias.join_column
WHERE condition;
```

Ví dụ: xóa audit record của các session đã hết hạn.

```sql
START TRANSACTION;

SELECT a.*
FROM lab_session_audit AS a
JOIN lab_session AS s
    ON a.session_id = s.session_id
WHERE s.session_status = 'EXPIRED';

DELETE a
FROM lab_session_audit AS a
JOIN lab_session AS s
    ON a.session_id = s.session_id
WHERE s.session_status = 'EXPIRED';

SELECT *
FROM lab_session_audit
ORDER BY audit_id;

ROLLBACK;
```

### 14.2. Xóa từ nhiều bảng

Cú pháp tổng quát:

```sql
DELETE t1, t2
FROM table_1 AS t1
JOIN table_2 AS t2
    ON t1.join_column = t2.join_column
WHERE condition;
```

Ví dụ sau xóa cả session expired và audit record liên quan. Hai bảng lab này không khai báo foreign key để minh họa cú pháp multiple-table delete.

```sql
START TRANSACTION;

DELETE s, a
FROM lab_session AS s
LEFT JOIN lab_session_audit AS a
    ON a.session_id = s.session_id
WHERE s.session_status = 'EXPIRED';

SELECT *
FROM lab_session
ORDER BY session_id;

SELECT *
FROM lab_session_audit
ORDER BY audit_id;

ROLLBACK;
```

### Lưu ý

- Multiple-table `DELETE` có thể xóa từ một hoặc nhiều bảng.
- Trong multiple-table `DELETE`, không dùng `ORDER BY` hoặc `LIMIT`.
- Với quan hệ parent–child có foreign key, không nên tùy tiện xóa parent và child trong cùng multi-table DELETE; xóa child trước hoặc dùng cascade khi phù hợp.
- Luôn chạy `SELECT` tương ứng trước khi xóa.

### Bài tập thực hành

**Bài 14.1.** Dùng `DELETE ... JOIN` để xóa audit record của session có trạng thái `EXPIRED`.

**Bài 14.2.** Dùng `DELETE ... JOIN` để xóa audit record của session có `user_name = 'minh'`.

**Bài 14.3.** Dùng multiple-table `DELETE` để xóa cả session và audit record của user `binh`.

**Bài 14.4.** Viết câu `SELECT` tương ứng để xem các audit record sẽ bị xóa trong Bài 14.1.

**Bài 14.5.** Giải thích vì sao không nên dùng multiple-table `DELETE` để xóa parent và child có foreign key khi chưa hiểu thứ tự xử lý của DBMS.

---

# Phần D. REPLACE

## 15. REPLACE: Chèn mới hoặc thay thế dòng xung đột khóa

`REPLACE` kiểm tra xung đột với `PRIMARY KEY` hoặc `UNIQUE` key.

- Nếu không có dòng xung đột: MySQL chèn dòng mới.
- Nếu có dòng xung đột: MySQL xử lý theo ngữ nghĩa **xóa dòng xung đột rồi chèn dòng mới**.

Cú pháp:

```sql
REPLACE INTO table_name (
    column1,
    column2,
    ...
)
VALUES (
    value1,
    value2,
    ...
);
```

Ví dụ: product `P100` đã tồn tại.

```sql
START TRANSACTION;

SELECT *
FROM lab_product_stock
WHERE product_code = 'P100';

REPLACE INTO lab_product_stock (
    product_code,
    product_name,
    quantity_in_stock,
    updated_at
)
VALUES (
    'P100',
    'USB-C Hub Updated',
    25,
    '2026-06-01 10:00:00'
);

SELECT *
FROM lab_product_stock
WHERE product_code = 'P100';

ROLLBACK;
```

Ví dụ: product `P500` chưa tồn tại, nên `REPLACE` hoạt động như chèn mới.

```sql
REPLACE INTO lab_product_stock (
    product_code,
    product_name,
    quantity_in_stock,
    updated_at
)
VALUES (
    'P500',
    'Webcam',
    12,
    '2026-06-01 11:00:00'
);
```

### REPLACE khác UPDATE như thế nào?

| Tiêu chí | `UPDATE` | `REPLACE` |
|---|---|---|
| Dòng có tồn tại? | Cập nhật dòng hiện có | Có thể xóa rồi chèn lại |
| Cột không nêu trong lệnh | Giữ nguyên | Có thể nhận default/giá trị mới khi dòng được chèn lại |
| Phù hợp khi | Muốn sửa một vài cột | Muốn thay thế toàn bộ dòng theo khóa |
| Rủi ro | Update quá nhiều dòng nếu thiếu `WHERE` | Có thể mất các giá trị cũ không nêu trong `REPLACE` |

> Vì `REPLACE` mang ngữ nghĩa delete-then-insert khi có xung đột, không nên dùng nó như một thay thế mặc định cho `UPDATE`. Với dữ liệu có foreign key hoặc có cột cần giữ nguyên, cần đánh giá kỹ trước khi dùng.

### Bài tập thực hành

**Bài 15.1.** Dùng `REPLACE` để thay product `P100` thành tên `USB-C Hub Pro`, tồn kho `40`, thời điểm `2026-06-15 09:00:00`.

**Bài 15.2.** Dùng `REPLACE` để chèn product mới `P500`, tên `Webcam`, tồn kho `12`.

**Bài 15.3.** Dùng `REPLACE` để thay product `P200` bằng tên `Wireless Mouse Plus`, tồn kho `50`.

**Bài 15.4.** Giải thích vì sao `REPLACE` có thể nguy hiểm nếu bảng có nhiều cột mà câu lệnh không nêu đủ các cột cần bảo toàn.

**Bài 15.5.** Viết một câu `UPDATE` thay thế hợp lý cho Bài 15.1 nếu chỉ muốn đổi `quantity_in_stock` của `P100` mà không thay đổi các cột khác.

---

## 16. Kiểm tra số dòng ảnh hưởng và warning

Sau các câu lệnh DML, có thể kiểm tra số dòng ảnh hưởng bằng:

```sql
SELECT ROW_COUNT() AS affected_rows;
```

Ví dụ:

```sql
UPDATE lab_employee
SET salary = salary + 100.00
WHERE employee_id = 101;

SELECT ROW_COUNT() AS affected_rows;
```

Sau `INSERT IGNORE`, kiểm tra warning:

```sql
SHOW WARNINGS;
```

### Bài tập thực hành

**Bài 16.1.** Sau một lệnh `INSERT`, dùng `ROW_COUNT()` để kiểm tra số dòng được chèn.

**Bài 16.2.** Sau một lệnh `UPDATE`, dùng `ROW_COUNT()` để kiểm tra số dòng được cập nhật.

**Bài 16.3.** Sau một lệnh `DELETE`, dùng `ROW_COUNT()` để kiểm tra số dòng được xóa.

**Bài 16.4.** Sau một lệnh `INSERT IGNORE`, chạy `SHOW WARNINGS`.

**Bài 16.5.** Giải thích vì sao `ROW_COUNT()` nên được chạy ngay sau câu lệnh DML cần kiểm tra.

---

## 17. Những lỗi thường gặp

### Lỗi 1. Quên WHERE trong UPDATE

Sai:

```sql
UPDATE lab_employee
SET salary = 0.00;
```

Câu lệnh này thay đổi lương của toàn bộ employee.

---

### Lỗi 2. Quên WHERE trong DELETE

Sai:

```sql
DELETE FROM lab_event;
```

Câu lệnh này xóa tất cả event.

---

### Lỗi 3. Chèn trùng khóa chính

```sql
INSERT INTO lab_department (
    department_id,
    department_name
)
VALUES (
    1,
    'Another Sales'
);
```

Lỗi vì `department_id = 1` đã tồn tại.

---

### Lỗi 4. Chèn dữ liệu con khi parent chưa tồn tại

```sql
INSERT INTO lab_sales_order (
    order_id,
    customer_id,
    order_date,
    order_status
)
VALUES (
    10010,
    999,
    '2026-06-01',
    'NEW'
);
```

Lỗi vì customer `999` không tồn tại.

---

### Lỗi 5. Nhầm NULL với chuỗi 'NULL'

Sai:

```sql
UPDATE lab_employee
SET department_id = 'NULL'
WHERE employee_id = 105;
```

Đúng:

```sql
UPDATE lab_employee
SET department_id = NULL
WHERE employee_id = 105;
```

---

### Lỗi 6. Dùng REPLACE khi chỉ cần UPDATE

Nếu chỉ muốn đổi tồn kho của product `P100`, nên dùng:

```sql
UPDATE lab_product_stock
SET quantity_in_stock = 40
WHERE product_code = 'P100';
```

Thay vì `REPLACE`, vì `REPLACE` có thể thay toàn bộ dòng.

### Bài tập thực hành

**Bài 17.1.** Sửa câu `UPDATE` ở Lỗi 1 để chỉ cập nhật employee `101`.

**Bài 17.2.** Sửa câu `DELETE` ở Lỗi 2 để chỉ xóa event `1`.

**Bài 17.3.** Giải thích lỗi trong Lỗi 3.

**Bài 17.4.** Giải thích lỗi trong Lỗi 4.

**Bài 17.5.** Sửa câu ở Lỗi 5 để gán giá trị SQL `NULL`.

---

## 18. Bài tập tổng hợp

### Bài 18.1. INSERT và INSERT Multiple Rows

Trong một transaction:

1. Chèn department `4` tên `Marketing`.
2. Chèn department `5` tên `Operations`.
3. Chèn employee `106` thuộc department `4`.
4. Chèn employee `107` thuộc department `5`.
5. Kiểm tra dữ liệu rồi `ROLLBACK`.

---

### Bài 18.2. INSERT INTO SELECT và INSERT IGNORE

Trong một transaction:

1. Chèn khách hàng ở `France` từ `classicmodels.customers` vào `lab_customer_copy`.
2. Chạy lại câu lệnh bằng `INSERT IGNORE`.
3. Dùng `SHOW WARNINGS`.
4. Kiểm tra số dòng trong `lab_customer_copy`.
5. `ROLLBACK`.

---

### Bài 18.3. UPDATE và UPDATE JOIN

Trong một transaction:

1. Tăng lương employee department `1` thêm 5%.
2. Cập nhật bonus rate từ `lab_department_bonus`.
3. Dùng `LEFT JOIN` để gán bonus rate `0.0000` cho employee không có bonus rule.
4. Kiểm tra dữ liệu trước và sau cập nhật.
5. `ROLLBACK`.

---

### Bài 18.4. DELETE và ON DELETE CASCADE

Trong một transaction:

1. Đếm số order của customer `501`.
2. Đếm số order item thuộc các order của customer `501`.
3. Xóa customer `501`.
4. Kiểm tra lại order và order item.
5. `ROLLBACK`.

---

### Bài 18.5. DELETE JOIN và REPLACE

Trong một transaction:

1. Xóa audit record của session expired bằng `DELETE ... JOIN`.
2. Dùng `REPLACE` thay product `P100`.
3. Kiểm tra dữ liệu sau mỗi thao tác.
4. Dùng `ROW_COUNT()` sau từng DML.
5. `ROLLBACK`.

---

## 19. Đáp án gợi ý cho một số bài tập

### Bài 4.1

```sql
INSERT INTO lab_employee (
    employee_id,
    department_id,
    full_name,
    email,
    salary,
    bonus_rate,
    hire_date,
    created_at
)
VALUES (
    106,
    3,
    'Vu Hai',
    'vu.hai@example.com',
    1400.00,
    0.0000,
    '2026-02-10',
    '2026-02-10 08:30:00'
);
```

### Bài 5.1

```sql
INSERT INTO lab_department (
    department_id,
    department_name
)
VALUES
    (4, 'Marketing'),
    (5, 'Operations');
```

### Bài 6.1

```sql
INSERT INTO lab_customer_copy (
    customer_number,
    customer_name,
    country,
    credit_limit
)
SELECT
    customerNumber,
    customerName,
    country,
    creditLimit
FROM classicmodels.customers
WHERE country = 'Germany';
```

### Bài 7.1

```sql
INSERT IGNORE INTO lab_department (
    department_id,
    department_name
)
VALUES
    (1, 'Sales Duplicate'),
    (4, 'Marketing');
```

### Bài 8.1

```sql
INSERT INTO lab_event (
    event_id,
    event_name,
    event_date,
    starts_at
)
VALUES (
    12,
    'Morning SQL Lab',
    '2026-07-15',
    '2026-07-15 08:00:00'
);
```

### Bài 9.2

```sql
INSERT INTO lab_employee (
    employee_id,
    department_id,
    full_name,
    email,
    salary,
    bonus_rate,
    hire_date,
    created_at
)
VALUES (
    110,
    1,
    'Current Date Employee',
    'current.date.employee@example.com',
    1500.00,
    0.0000,
    CURDATE(),
    NOW()
);
```

### Bài 10.1

```sql
UPDATE lab_employee
SET salary = salary + 200.00
WHERE employee_id = 101;
```

### Bài 11.1

```sql
UPDATE lab_employee AS e
INNER JOIN lab_department_bonus AS b
    ON e.department_id = b.department_id
SET e.bonus_rate = b.bonus_rate;
```

### Bài 12.1

```sql
DELETE FROM lab_event
WHERE event_id = 1;
```

### Bài 13.5

```sql
SELECT COUNT(*) AS order_count
FROM lab_sales_order
WHERE customer_id = 501;
```

```sql
SELECT COUNT(*) AS order_item_count
FROM lab_sales_order_item AS i
JOIN lab_sales_order AS o
    ON i.order_id = o.order_id
WHERE o.customer_id = 501;
```

### Bài 14.1

```sql
DELETE a
FROM lab_session_audit AS a
JOIN lab_session AS s
    ON a.session_id = s.session_id
WHERE s.session_status = 'EXPIRED';
```

### Bài 15.1

```sql
REPLACE INTO lab_product_stock (
    product_code,
    product_name,
    quantity_in_stock,
    updated_at
)
VALUES (
    'P100',
    'USB-C Hub Pro',
    40,
    '2026-06-15 09:00:00'
);
```

### Bài 15.5

```sql
UPDATE lab_product_stock
SET quantity_in_stock = 40
WHERE product_code = 'P100';
```

---

## 20. Tóm tắt

Các kiến thức chính trong tutorial:

- `INSERT` chèn một dòng dữ liệu mới.
- `INSERT` có thể chèn nhiều dòng trong một câu lệnh.
- `INSERT ... SELECT` chèn dữ liệu từ kết quả truy vấn.
- `INSERT IGNORE` có thể tiếp tục xử lý khi một số lỗi xảy ra, nhưng cần kiểm tra warning để tránh che giấu lỗi dữ liệu.
- `DATE` lưu ngày; `DATETIME` lưu cả ngày và thời điểm.
- `UPDATE` thay đổi dữ liệu trong một bảng.
- `UPDATE ... JOIN` cập nhật một bảng dựa trên dữ liệu bảng liên quan.
- `DELETE` xóa các dòng thỏa điều kiện.
- `ON DELETE CASCADE` tự động xóa child rows khi parent row bị xóa, nếu foreign key được khai báo như vậy.
- `DELETE ... JOIN` xóa theo điều kiện liên bảng.
- `REPLACE` có thể chèn mới hoặc xử lý xung đột khóa bằng ngữ nghĩa xóa rồi chèn lại.
- Luôn kiểm tra bằng `SELECT`, dùng transaction khi thực hành và chỉ `COMMIT` khi chắc chắn cần lưu thay đổi.

---

## 21. Từ khóa chính

- SQL
- DML
- INSERT
- INSERT Multiple Rows
- INSERT INTO SELECT
- INSERT IGNORE
- DATE
- DATETIME
- CURDATE()
- NOW()
- UPDATE
- UPDATE JOIN
- INNER JOIN
- LEFT JOIN
- DELETE
- ON DELETE CASCADE
- DELETE JOIN
- REPLACE
- Foreign Key
- Transaction
- START TRANSACTION
- COMMIT
- ROLLBACK
- ROW_COUNT()
- SHOW WARNINGS
- classicmodels
