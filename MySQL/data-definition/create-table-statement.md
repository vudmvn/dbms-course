---
title: "Tutorial: Ràng buộc trong CREATE TABLE với MySQL"
author: "Tên giảng viên"
duration: "150m"
difficulty: "Beginner–Intermediate"
prerequisites:
    - "Đã biết kiểu dữ liệu cơ bản như INT, VARCHAR, DECIMAL, DATE, DATETIME"
    - "Đã biết câu lệnh CREATE DATABASE, USE và CREATE TABLE cơ bản"
    - "Đã biết cách mở MySQL client hoặc MySQL Workbench"
summary: "Thực hành khai báo PRIMARY KEY, FOREIGN KEY, NOT NULL, UNIQUE, CHECK, DEFAULT và quản lý kiểm tra khóa ngoại trong MySQL."
---

# Tutorial: Ràng buộc trong `CREATE TABLE` với MySQL

## Link tham khảo

- [MySQL constraints – MySQL Tutorial](https://www.mysqltutorial.org/mysql-basics/)
- [MySQL PRIMARY KEY](https://www.mysqltutorial.org/mysql-basics/mysql-primary-key/)
- [MySQL FOREIGN KEY](https://www.mysqltutorial.org/mysql-basics/mysql-foreign-key/)
- [Disable foreign key checks](https://www.mysqltutorial.org/mysql-basics/mysql-disable-foreign-key-checks/)
- [MySQL NOT NULL constraint](https://www.mysqltutorial.org/mysql-basics/mysql-not-null-constraint/)
- [MySQL UNIQUE constraint](https://www.mysqltutorial.org/mysql-basics/mysql-unique-constraint/)
- [MySQL CHECK constraint](https://www.mysqltutorial.org/mysql-basics/mysql-check-constraint/)
- [MySQL DEFAULT](https://www.mysqltutorial.org/mysql-basics/mysql-default/)
- [MySQL CREATE TABLE statement](https://dev.mysql.com/doc/refman/8.4/en/create-table.html)

## 1. Mục tiêu học tập

Sau khi hoàn thành tutorial này, người học có thể:

1. Giải thích được vai trò của constraint trong việc bảo đảm toàn vẹn dữ liệu.
2. Tạo khóa chính đơn và khóa chính ghép bằng `PRIMARY KEY`.
3. Tạo quan hệ tham chiếu bằng `FOREIGN KEY ... REFERENCES ...`.
4. Hiểu khi nào có thể tạm thời tắt `FOREIGN_KEY_CHECKS` và các rủi ro của thao tác này.
5. Khai báo cột bắt buộc có dữ liệu bằng `NOT NULL`.
6. Bảo đảm giá trị không trùng lặp bằng `UNIQUE`.
7. Áp dụng quy tắc nghiệp vụ bằng `CHECK`.
8. Gán giá trị mặc định bằng `DEFAULT`.
9. Đặt tên constraint theo quy ước để dễ đọc và bảo trì.
10. Kiểm tra cấu trúc bảng và constraint bằng `SHOW CREATE TABLE`.

---

## 2. Constraint là gì?

**Constraint** là ràng buộc do DBMS kiểm tra nhằm bảo đảm dữ liệu được lưu vào bảng tuân thủ các quy tắc đã xác định.

Ví dụ:

- Mỗi sinh viên phải có một mã duy nhất.
- Tên đăng nhập không được trùng nhau.
- Điểm phải nằm trong khoảng từ `0` đến `10`.
- Một đơn hàng chỉ được thuộc về khách hàng đã tồn tại.
- Khi chưa nhập trạng thái đơn hàng, hệ thống tự gán trạng thái `Pending`.

Các constraint chính trong bài này:

| Constraint | Vai trò chính |
|---|---|
| `PRIMARY KEY` | Nhận diện duy nhất mỗi dòng |
| `FOREIGN KEY` | Bảo đảm tham chiếu giữa các bảng hợp lệ |
| `NOT NULL` | Bắt buộc một cột phải có giá trị |
| `UNIQUE` | Ngăn giá trị trùng lặp |
| `CHECK` | Ép dữ liệu thỏa điều kiện logic |
| `DEFAULT` | Cấp giá trị mặc định khi không được cung cấp |

Một bảng có thể có nhiều constraint cùng lúc. Ví dụ:

```sql
CREATE TABLE students (
    student_id INT,
    full_name VARCHAR(100) NOT NULL,
    email VARCHAR(150) NOT NULL UNIQUE,
    average_score DECIMAL(4,2) DEFAULT 0.00,
    CONSTRAINT pk_students PRIMARY KEY (student_id),
    CONSTRAINT chk_students_score
        CHECK (average_score BETWEEN 0 AND 10)
);
```

Trong ví dụ trên:

- `student_id` là khóa chính.
- `full_name` và `email` không được để trống.
- `email` không được trùng.
- `average_score` mặc định bằng `0.00`.
- `average_score` phải thuộc đoạn từ `0` đến `10`.

---

## 3. Chuẩn bị môi trường thực hành

Tutorial sử dụng một schema riêng tên là `constraints_lab`. Không tạo bảng thực hành trực tiếp trong `classicmodels` để tránh ảnh hưởng đến dữ liệu mẫu.

```sql
CREATE DATABASE IF NOT EXISTS constraints_lab;
USE constraints_lab;
```

Xem các bảng hiện có:

```sql
SHOW TABLES;
```

Xem câu lệnh đã dùng để tạo một bảng:

```sql
SHOW CREATE TABLE table_name;
```

Ví dụ:

```sql
SHOW CREATE TABLE students;
```

Xóa bảng khi cần thực hiện lại một ví dụ:

```sql
DROP TABLE IF EXISTS students;
```

> **Lưu ý:** Khi nhiều bảng có khóa ngoại, hãy xóa bảng con trước bảng cha, hoặc làm theo hướng dẫn trong phần `FOREIGN_KEY_CHECKS`.

---

## 4. Quy ước đặt tên constraint

MySQL có thể tự tạo tên cho một số constraint. Tuy nhiên, nên đặt tên tường minh để dễ đọc và dễ xóa/sửa về sau.

| Loại constraint | Quy ước gợi ý | Ví dụ |
|---|---|---|
| Primary key | `pk_<table>` | `pk_students` |
| Foreign key | `fk_<child>_<parent>` | `fk_orders_customers` |
| Unique | `uq_<table>_<column>` | `uq_users_email` |
| Check | `chk_<table>_<rule>` | `chk_products_price` |

Ví dụ:

```sql
CREATE TABLE users (
    user_id INT,
    email VARCHAR(150) NOT NULL,
    age INT,
    CONSTRAINT pk_users PRIMARY KEY (user_id),
    CONSTRAINT uq_users_email UNIQUE (email),
    CONSTRAINT chk_users_age CHECK (age >= 18)
);
```

---

# Phần A. `PRIMARY KEY`

## 5. Khóa chính đơn

### 5.1. Vai trò của `PRIMARY KEY`

`PRIMARY KEY` là một cột hoặc một nhóm cột dùng để nhận diện duy nhất từng dòng trong bảng.

Một primary key có các đặc điểm:

- Giá trị phải duy nhất.
- Không được là `NULL`.
- Mỗi bảng chỉ có tối đa một primary key.
- Primary key có thể gồm một cột hoặc nhiều cột.

### 5.2. Cú pháp khóa chính một cột

Có thể khai báo trực tiếp trên cột:

```sql
CREATE TABLE departments (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(100) NOT NULL
);
```

Hoặc khai báo ở cuối danh sách cột:

```sql
CREATE TABLE departments (
    department_id INT,
    department_name VARCHAR(100) NOT NULL,
    CONSTRAINT pk_departments PRIMARY KEY (department_id)
);
```

Cách thứ hai thường phù hợp hơn khi muốn đặt tên constraint rõ ràng.

### 5.3. Ví dụ hoàn chỉnh

```sql
DROP TABLE IF EXISTS departments;

CREATE TABLE departments (
    department_id INT,
    department_name VARCHAR(100) NOT NULL,
    office_phone VARCHAR(20),
    CONSTRAINT pk_departments PRIMARY KEY (department_id)
) ENGINE = InnoDB;
```

Chèn dữ liệu hợp lệ:

```sql
INSERT INTO departments (department_id, department_name, office_phone)
VALUES (1, 'Information Systems', '024-1234-5678');
```

Chèn dữ liệu trùng primary key sẽ gây lỗi:

```sql
INSERT INTO departments (department_id, department_name, office_phone)
VALUES (1, 'Computer Science', '024-8765-4321');
```

Chèn `NULL` vào primary key cũng gây lỗi:

```sql
INSERT INTO departments (department_id, department_name)
VALUES (NULL, 'Data Science');
```

Kiểm tra cấu trúc bảng:

```sql
SHOW CREATE TABLE departments;
```

### Bài tập thực hành

**Bài 5.1.**

Tạo bảng `courses` gồm các cột `course_id INT`, `course_name VARCHAR(150)`, `credits INT`. Đặt `course_id` là primary key và đặt tên constraint là `pk_courses`.

**Bài 5.2.**

Tạo bảng `classrooms` gồm `room_id INT`, `building_name VARCHAR(100)`, `capacity INT`. Đặt `room_id` là primary key.

**Bài 5.3.**

Tạo bảng `books` gồm `book_id INT`, `title VARCHAR(200)`, `publisher VARCHAR(100)`. Đặt `book_id` là primary key trực tiếp tại định nghĩa cột.

**Bài 5.4.**

Tạo bảng `employees_basic` gồm `employee_id INT`, `full_name VARCHAR(100)`, `hire_date DATE`. Đặt `employee_id` là primary key bằng table constraint và đặt tên `pk_employees_basic`.

**Bài 5.5.**

Dùng `SHOW CREATE TABLE` để kiểm tra primary key của bảng `courses`, `classrooms`, `books` và `employees_basic`.

---

## 6. Khóa chính ghép

### 6.1. Khái niệm

**Composite primary key** là primary key gồm từ hai cột trở lên.

Mỗi cột riêng lẻ có thể trùng lặp, nhưng tổ hợp giá trị của tất cả các cột trong khóa phải là duy nhất.

Ví dụ, một sinh viên có thể học nhiều học phần và một học phần có nhiều sinh viên:

```text
ENROLLMENT(student_id, course_id, semester)
```

Một cách xác định duy nhất bản ghi đăng ký là:

```text
(student_id, course_id, semester)
```

### 6.2. Cú pháp

```sql
CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    semester VARCHAR(20),
    enrolled_at DATE,
    CONSTRAINT pk_enrollments
        PRIMARY KEY (student_id, course_id, semester)
);
```

Không thể khai báo composite primary key hoàn chỉnh theo dạng inline ở từng cột. Cần dùng table constraint.

### 6.3. Ví dụ

```sql
DROP TABLE IF EXISTS enrollments;

CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    semester VARCHAR(20),
    enrolled_at DATE,
    CONSTRAINT pk_enrollments
        PRIMARY KEY (student_id, course_id, semester)
) ENGINE = InnoDB;
```

Các dòng sau hợp lệ vì tổ hợp khóa khác nhau:

```sql
INSERT INTO enrollments (student_id, course_id, semester, enrolled_at)
VALUES
    (1, 101, '2026A', '2026-01-10'),
    (1, 102, '2026A', '2026-01-10'),
    (1, 101, '2026B', '2026-08-10');
```

Dòng sau không hợp lệ vì trùng cả ba thành phần của khóa:

```sql
INSERT INTO enrollments (student_id, course_id, semester, enrolled_at)
VALUES (1, 101, '2026A', '2026-01-12');
```

### Bài tập thực hành

**Bài 6.1.**

Tạo bảng `student_courses` gồm `student_id INT`, `course_id INT`, `semester VARCHAR(20)`, `final_score DECIMAL(4,2)`. Đặt primary key ghép là `(student_id, course_id, semester)`.

**Bài 6.2.**

Tạo bảng `order_lines` gồm `order_id INT`, `product_id INT`, `quantity INT`, `price_each DECIMAL(10,2)`. Đặt primary key ghép là `(order_id, product_id)`.

**Bài 6.3.**

Tạo bảng `employee_projects` gồm `employee_id INT`, `project_id INT`, `assigned_date DATE`, `role_name VARCHAR(100)`. Đặt primary key ghép là `(employee_id, project_id)`.

**Bài 6.4.**

Tạo bảng `daily_attendance` gồm `student_id INT`, `attendance_date DATE`, `status VARCHAR(20)`. Đặt primary key ghép là `(student_id, attendance_date)`.

**Bài 6.5.**

Chèn ba dòng vào `order_lines` sao cho `order_id` có thể trùng nhưng tổ hợp `(order_id, product_id)` không trùng. Sau đó thử chèn một dòng trùng tổ hợp khóa để quan sát lỗi.

---

# Phần B. `FOREIGN KEY`

## 7. Khóa ngoại

### 7.1. Vai trò của `FOREIGN KEY`

**Foreign key** là một cột hoặc nhóm cột trong bảng con tham chiếu đến primary key hoặc unique key ở bảng cha.

Foreign key giúp bảo đảm **referential integrity**:

- Không thể chèn dữ liệu con tham chiếu tới dữ liệu cha không tồn tại.
- Không thể xóa hoặc sửa dữ liệu cha nếu hành động đó làm mất tính hợp lệ của dữ liệu con, trừ khi đã khai báo một hành động tham chiếu phù hợp.

Ví dụ:

```text
DEPARTMENTS (parent table)
    department_id

EMPLOYEES (child table)
    department_id → DEPARTMENTS.department_id
```

### 7.2. Cú pháp cơ bản

```sql
CONSTRAINT constraint_name
FOREIGN KEY (child_column)
REFERENCES parent_table(parent_column)
```

Ví dụ:

```sql
DROP TABLE IF EXISTS employees;
DROP TABLE IF EXISTS departments;

CREATE TABLE departments (
    department_id INT,
    department_name VARCHAR(100) NOT NULL,
    CONSTRAINT pk_departments PRIMARY KEY (department_id)
) ENGINE = InnoDB;

CREATE TABLE employees (
    employee_id INT,
    full_name VARCHAR(100) NOT NULL,
    department_id INT,
    CONSTRAINT pk_employees PRIMARY KEY (employee_id),
    CONSTRAINT fk_employees_departments
        FOREIGN KEY (department_id)
        REFERENCES departments(department_id)
) ENGINE = InnoDB;
```

> Với InnoDB, cột tham chiếu ở bảng cha nên là primary key hoặc unique key. Kiểu dữ liệu giữa cột con và cột cha cần tương thích.

### 7.3. Ví dụ chèn dữ liệu

Chèn bảng cha trước:

```sql
INSERT INTO departments (department_id, department_name)
VALUES (10, 'Information Systems');
```

Sau đó mới chèn bảng con:

```sql
INSERT INTO employees (employee_id, full_name, department_id)
VALUES (1001, 'Nguyen An', 10);
```

Câu lệnh sau không hợp lệ nếu department `999` chưa tồn tại:

```sql
INSERT INTO employees (employee_id, full_name, department_id)
VALUES (1002, 'Tran Binh', 999);
```

### 7.4. `ON DELETE` và `ON UPDATE`

Có thể quy định hành động khi dữ liệu cha bị xóa hoặc thay đổi.

| Tùy chọn | Ý nghĩa |
|---|---|
| `RESTRICT` | Từ chối xóa/cập nhật dữ liệu cha nếu có dữ liệu con tham chiếu |
| `CASCADE` | Tự động xóa/cập nhật các dòng con liên quan |
| `SET NULL` | Đặt foreign key ở bảng con thành `NULL` |
| `NO ACTION` | Trong MySQL/InnoDB thường có hành vi tương tự `RESTRICT` |

Ví dụ xóa một department thì xóa các employee liên quan:

```sql
CREATE TABLE employees_cascade (
    employee_id INT,
    full_name VARCHAR(100) NOT NULL,
    department_id INT,
    CONSTRAINT pk_employees_cascade PRIMARY KEY (employee_id),
    CONSTRAINT fk_employees_cascade_departments
        FOREIGN KEY (department_id)
        REFERENCES departments(department_id)
        ON DELETE CASCADE
        ON UPDATE CASCADE
) ENGINE = InnoDB;
```

> Không nên dùng `CASCADE` chỉ để “cho tiện”. Hãy dùng nó khi quy tắc nghiệp vụ thực sự cho phép xóa/cập nhật tự động dữ liệu con.

### Bài tập thực hành

**Bài 7.1.**

Tạo bảng `faculties` gồm `faculty_id INT`, `faculty_name VARCHAR(100)`. Đặt `faculty_id` là primary key.

**Bài 7.2.**

Tạo bảng `students_fk` gồm `student_id INT`, `full_name VARCHAR(100)`, `faculty_id INT`. Đặt `student_id` là primary key và `faculty_id` là foreign key tham chiếu `faculties(faculty_id)`.

**Bài 7.3.**

Tạo bảng `categories` gồm `category_id INT`, `category_name VARCHAR(100)`; sau đó tạo bảng `products_fk` gồm `product_id INT`, `product_name VARCHAR(150)`, `category_id INT`, trong đó `category_id` tham chiếu `categories(category_id)`.

**Bài 7.4.**

Tạo bảng `orders_fk` gồm `order_id INT`, `customer_name VARCHAR(100)`; sau đó tạo bảng `order_items_fk` gồm `order_item_id INT`, `order_id INT`, `product_name VARCHAR(150)`. Khai báo foreign key từ `order_items_fk.order_id` đến `orders_fk.order_id` với `ON DELETE CASCADE`.

**Bài 7.5.**

Chèn một faculty có mã `1` vào `faculties`, sau đó chèn một sinh viên thuộc faculty `1`. Cuối cùng, thử chèn một sinh viên có `faculty_id = 999` và giải thích lỗi.

---

## 8. Xóa và thêm foreign key bằng `ALTER TABLE`

Foreign key cũng có thể được thêm sau khi bảng đã tạo.

### 8.1. Thêm foreign key

```sql
ALTER TABLE students_fk
ADD CONSTRAINT fk_students_fk_faculties
FOREIGN KEY (faculty_id)
REFERENCES faculties(faculty_id);
```

### 8.2. Xóa foreign key

Muốn xóa foreign key, cần biết tên constraint:

```sql
ALTER TABLE students_fk
DROP FOREIGN KEY fk_students_fk_faculties;
```

Có thể dùng:

```sql
SHOW CREATE TABLE students_fk;
```

để xem tên foreign key hiện có.

### 8.3. Lưu ý

- Nếu bảng đã có dữ liệu không hợp lệ, lệnh thêm foreign key có thể thất bại.
- Khi xóa foreign key, MySQL có thể vẫn giữ index liên quan; cần kiểm tra cấu trúc bảng nếu muốn quản lý index riêng.

### Bài tập thực hành

**Bài 8.1.**

Tạo bảng `suppliers` gồm `supplier_id INT` là primary key và `supplier_name VARCHAR(100)`.

**Bài 8.2.**

Tạo bảng `products_alter_fk` gồm `product_id INT` là primary key, `product_name VARCHAR(150)`, `supplier_id INT`, nhưng chưa tạo foreign key.

**Bài 8.3.**

Dùng `ALTER TABLE` để thêm foreign key từ `products_alter_fk.supplier_id` đến `suppliers.supplier_id`. Đặt tên constraint là `fk_products_alter_fk_suppliers`.

**Bài 8.4.**

Dùng `SHOW CREATE TABLE products_alter_fk` để xác định foreign key vừa tạo.

**Bài 8.5.**

Dùng `ALTER TABLE` để xóa foreign key `fk_products_alter_fk_suppliers`.

---

## 9. Tắt kiểm tra foreign key: `FOREIGN_KEY_CHECKS`

### 9.1. Cú pháp

Tắt kiểm tra foreign key trong session hiện tại:

```sql
SET FOREIGN_KEY_CHECKS = 0;
```

Bật lại:

```sql
SET FOREIGN_KEY_CHECKS = 1;
```

Kiểm tra giá trị hiện tại:

```sql
SELECT @@FOREIGN_KEY_CHECKS;
```

### 9.2. Khi nào có thể dùng?

Một số tình huống có thể cần tạm thời tắt kiểm tra:

- Import dữ liệu trong môi trường kiểm soát được.
- Nạp dữ liệu cha và con không theo thứ tự.
- Xóa hoặc tái tạo một nhóm bảng có relationship phức tạp trong môi trường lab.

### 9.3. Rủi ro quan trọng

Khi `FOREIGN_KEY_CHECKS = 0`, MySQL có thể cho phép tạo dữ liệu không nhất quán.

Ví dụ:

```sql
SET FOREIGN_KEY_CHECKS = 0;

INSERT INTO students_fk (student_id, full_name, faculty_id)
VALUES (9001, 'Invalid Reference', 999);

SET FOREIGN_KEY_CHECKS = 1;
```

Sau khi bật lại, MySQL **không tự động rà soát và sửa/xóa** các dòng đã được chèn sai trong lúc kiểm tra bị tắt. Tuy nhiên, các lệnh chèn hoặc cập nhật sau đó vẫn phải tuân thủ foreign key.

> Chỉ nên tắt foreign key checks trong môi trường lab, import có kiểm soát hoặc quy trình bảo trì đã được phê duyệt. Không dùng thao tác này để “né” lỗi thiết kế dữ liệu.

### Bài tập thực hành

**Bài 9.1.**

Dùng `SELECT @@FOREIGN_KEY_CHECKS;` để kiểm tra trạng thái foreign key checks hiện tại.

**Bài 9.2.**

Tắt foreign key checks bằng `SET FOREIGN_KEY_CHECKS = 0;`, sau đó kiểm tra lại giá trị bằng `SELECT @@FOREIGN_KEY_CHECKS;`.

**Bài 9.3.**

Bật lại foreign key checks bằng `SET FOREIGN_KEY_CHECKS = 1;`, sau đó kiểm tra lại giá trị.

**Bài 9.4.**

Tạo hai bảng `parent_demo` và `child_demo` có foreign key. Trong một session lab riêng, tắt foreign key checks và thử chèn một child tham chiếu đến parent không tồn tại. Sau đó bật lại checks và kiểm tra dữ liệu đã chèn.

**Bài 9.5.**

Viết ngắn gọn ba tình huống có thể dùng `FOREIGN_KEY_CHECKS = 0` và ba rủi ro của thao tác này.

---

# Phần C. `NOT NULL`

## 10. Ràng buộc `NOT NULL`

### 10.1. Vai trò

`NOT NULL` yêu cầu một cột phải luôn có giá trị. Không thể chèn hoặc cập nhật cột này thành `NULL`.

Cú pháp:

```sql
column_name data_type NOT NULL
```

Ví dụ:

```sql
CREATE TABLE users_not_null (
    user_id INT,
    full_name VARCHAR(100) NOT NULL,
    email VARCHAR(150) NOT NULL,
    CONSTRAINT pk_users_not_null PRIMARY KEY (user_id)
);
```

### 10.2. Ví dụ

```sql
DROP TABLE IF EXISTS users_not_null;

CREATE TABLE users_not_null (
    user_id INT,
    full_name VARCHAR(100) NOT NULL,
    email VARCHAR(150) NOT NULL,
    created_at DATETIME NOT NULL,
    CONSTRAINT pk_users_not_null PRIMARY KEY (user_id)
) ENGINE = InnoDB;
```

Chèn hợp lệ:

```sql
INSERT INTO users_not_null (user_id, full_name, email, created_at)
VALUES (1, 'Nguyen An', 'an@example.com', '2026-06-21 08:00:00');
```

Chèn không hợp lệ vì `full_name` là `NULL`:

```sql
INSERT INTO users_not_null (user_id, full_name, email, created_at)
VALUES (2, NULL, 'binh@example.com', '2026-06-21 08:00:00');
```

### 10.3. `NOT NULL` và chuỗi rỗng

`NULL` và chuỗi rỗng `''` không giống nhau.

- `NULL`: không có giá trị/không biết giá trị.
- `''`: có giá trị là chuỗi rỗng.

Một cột `VARCHAR(...) NOT NULL` vẫn có thể nhận `''` nếu không có thêm `CHECK` hoặc quy tắc ứng dụng ngăn điều đó.

### Bài tập thực hành

**Bài 10.1.**

Tạo bảng `teachers_not_null` gồm `teacher_id INT`, `full_name VARCHAR(100)`, `email VARCHAR(150)`, `hire_date DATE`. Đặt cả `full_name`, `email`, `hire_date` là `NOT NULL`.

**Bài 10.2.**

Tạo bảng `products_not_null` gồm `product_id INT`, `product_name VARCHAR(150)`, `unit_price DECIMAL(10,2)`. Đặt `product_name` và `unit_price` là `NOT NULL`.

**Bài 10.3.**

Tạo bảng `class_sessions` gồm `session_id INT`, `course_name VARCHAR(150)`, `session_date DATE`, `room_name VARCHAR(100)`. Đặt `course_name` và `session_date` là `NOT NULL`.

**Bài 10.4.**

Chèn một dòng hợp lệ vào `teachers_not_null`, sau đó thử chèn một dòng có `email = NULL` để quan sát lỗi.

**Bài 10.5.**

Giải thích sự khác nhau giữa `NULL` và `''` đối với một cột `VARCHAR(100) NOT NULL`.

---

# Phần D. `UNIQUE`

## 11. Ràng buộc `UNIQUE`

### 11.1. Vai trò

`UNIQUE` bảo đảm giá trị của một cột hoặc một tổ hợp cột không trùng lặp giữa các dòng.

Khác với primary key:

| Tiêu chí | `PRIMARY KEY` | `UNIQUE` |
|---|---|---|
| Số lượng mỗi bảng | Tối đa một | Có thể nhiều |
| Giá trị `NULL` | Không được phép | Có thể cho phép tùy định nghĩa cột |
| Mục đích | Nhận diện chính | Bảo đảm tính duy nhất của dữ liệu nghiệp vụ |

Trong MySQL, một unique index trên cột có thể nhận `NULL` cho phép nhiều giá trị `NULL`. Nếu dữ liệu bắt buộc phải có giá trị và không trùng, thường kết hợp `NOT NULL UNIQUE`.

### 11.2. Cú pháp trên một cột

```sql
CREATE TABLE users_unique (
    user_id INT,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(150) NOT NULL,
    CONSTRAINT pk_users_unique PRIMARY KEY (user_id),
    CONSTRAINT uq_users_unique_email UNIQUE (email)
);
```

### 11.3. Unique ghép

Một unique constraint có thể áp dụng trên nhiều cột.

Ví dụ, một phòng chỉ được đặt một lần cho cùng ngày và cùng ca:

```sql
CREATE TABLE room_bookings (
    booking_id INT,
    room_id INT NOT NULL,
    booking_date DATE NOT NULL,
    time_slot VARCHAR(20) NOT NULL,
    CONSTRAINT pk_room_bookings PRIMARY KEY (booking_id),
    CONSTRAINT uq_room_bookings_slot
        UNIQUE (room_id, booking_date, time_slot)
);
```

### Bài tập thực hành

**Bài 11.1.**

Tạo bảng `users_unique` gồm `user_id INT`, `username VARCHAR(50)`, `email VARCHAR(150)`. Đặt `user_id` là primary key, `username` là `NOT NULL UNIQUE`, và `email` là `NOT NULL UNIQUE`.

**Bài 11.2.**

Tạo bảng `students_unique` gồm `student_id INT`, `student_code VARCHAR(20)`, `email VARCHAR(150)`. Đặt `student_id` là primary key; tạo unique constraint cho `student_code` và một unique constraint khác cho `email`.

**Bài 11.3.**

Tạo bảng `room_bookings` với unique constraint ghép `(room_id, booking_date, time_slot)`.

**Bài 11.4.**

Chèn hai user có username khác nhau vào `users_unique`. Sau đó thử chèn user thứ ba có username trùng với user đầu tiên.

**Bài 11.5.**

Chèn hai dòng vào `room_bookings` có cùng `room_id` và `booking_date` nhưng khác `time_slot`. Sau đó thử chèn một dòng trùng cả ba giá trị để quan sát lỗi.

---

# Phần E. `CHECK`

## 12. Ràng buộc `CHECK`

### 12.1. Vai trò

`CHECK` buộc dữ liệu phải thỏa một biểu thức Boolean.

Cú pháp:

```sql
CONSTRAINT constraint_name CHECK (condition)
```

Ví dụ:

```sql
CREATE TABLE products_check (
    product_id INT,
    product_name VARCHAR(150) NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    quantity_in_stock INT NOT NULL,
    CONSTRAINT pk_products_check PRIMARY KEY (product_id),
    CONSTRAINT chk_products_check_price CHECK (unit_price >= 0),
    CONSTRAINT chk_products_check_quantity CHECK (quantity_in_stock >= 0)
);
```

### 12.2. Lưu ý về phiên bản MySQL

MySQL thực thi `CHECK` từ phiên bản **8.0.16** trở lên. Nếu dùng phiên bản cũ hơn, `CHECK` có thể không được thực thi như mong đợi.

Kiểm tra phiên bản MySQL:

```sql
SELECT VERSION();
```

### 12.3. `CHECK` và `NULL`

Một biểu thức `CHECK` cần được thiết kế cùng `NOT NULL` nếu cột bắt buộc có dữ liệu.

Ví dụ:

```sql
age INT CHECK (age >= 18)
```

Nếu `age` cho phép `NULL`, giá trị `NULL` không nhất thiết bị constraint này loại bỏ. Nếu tuổi là bắt buộc, nên viết:

```sql
age INT NOT NULL CHECK (age >= 18)
```

### 12.4. Ví dụ kiểm tra nhiều cột

```sql
CREATE TABLE promotions (
    promotion_id INT,
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    discount_percent DECIMAL(5,2) NOT NULL,
    CONSTRAINT pk_promotions PRIMARY KEY (promotion_id),
    CONSTRAINT chk_promotions_dates
        CHECK (end_date >= start_date),
    CONSTRAINT chk_promotions_discount
        CHECK (discount_percent BETWEEN 0 AND 100)
);
```

### Bài tập thực hành

**Bài 12.1.**

Tạo bảng `products_check` gồm `product_id INT`, `product_name VARCHAR(150)`, `unit_price DECIMAL(10,2)`, `quantity_in_stock INT`. Đặt `unit_price >= 0` và `quantity_in_stock >= 0` bằng `CHECK`.

**Bài 12.2.**

Tạo bảng `students_check` gồm `student_id INT`, `full_name VARCHAR(100)`, `average_score DECIMAL(4,2)`. Đặt constraint bảo đảm `average_score BETWEEN 0 AND 10`.

**Bài 12.3.**

Tạo bảng `employees_check` gồm `employee_id INT`, `full_name VARCHAR(100)`, `salary DECIMAL(12,2)`, `age INT`. Đặt constraint `salary > 0` và `age >= 18`.

**Bài 12.4.**

Tạo bảng `promotions` gồm `promotion_id INT`, `start_date DATE`, `end_date DATE`, `discount_percent DECIMAL(5,2)`. Đặt constraint `end_date >= start_date` và `discount_percent BETWEEN 0 AND 100`.

**Bài 12.5.**

Chèn một dòng hợp lệ và một dòng không hợp lệ vào `products_check`: dòng không hợp lệ có `unit_price = -10.00` hoặc `quantity_in_stock = -1`. Ghi lại lỗi nhận được.

---

# Phần F. `DEFAULT`

## 13. Ràng buộc `DEFAULT`

### 13.1. Vai trò

`DEFAULT` cung cấp giá trị tự động cho một cột khi câu lệnh `INSERT` không chỉ định giá trị cho cột đó.

Cú pháp:

```sql
column_name data_type DEFAULT default_value
```

Ví dụ:

```sql
CREATE TABLE tasks_default (
    task_id INT,
    task_name VARCHAR(150) NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'Pending',
    priority_level INT NOT NULL DEFAULT 3,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT pk_tasks_default PRIMARY KEY (task_id)
);
```

### 13.2. Ví dụ

```sql
DROP TABLE IF EXISTS tasks_default;

CREATE TABLE tasks_default (
    task_id INT,
    task_name VARCHAR(150) NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'Pending',
    priority_level INT NOT NULL DEFAULT 3,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT pk_tasks_default PRIMARY KEY (task_id)
) ENGINE = InnoDB;
```

Nếu chỉ chèn `task_id` và `task_name`:

```sql
INSERT INTO tasks_default (task_id, task_name)
VALUES (1, 'Prepare database lab');
```

Kết quả mong đợi:

```text
status         = 'Pending'
priority_level = 3
created_at     = thời điểm chèn dữ liệu
```

Nếu chỉ định giá trị cụ thể, giá trị đó được ưu tiên hơn default:

```sql
INSERT INTO tasks_default (task_id, task_name, status, priority_level)
VALUES (2, 'Review assignment', 'In Progress', 1);
```

### 13.3. Lưu ý

- `DEFAULT` không thay thế `NOT NULL`; hai constraint thường được dùng cùng nhau.
- `DEFAULT` chỉ được dùng khi cột bị bỏ qua trong `INSERT` hoặc dùng từ khóa `DEFAULT`.
- Default phải phù hợp với kiểu dữ liệu.
- Với cột thời gian, `CURRENT_TIMESTAMP` là lựa chọn phổ biến.

### Bài tập thực hành

**Bài 13.1.**

Tạo bảng `tasks_default` gồm `task_id INT`, `task_name VARCHAR(150)`, `status VARCHAR(20)`, `priority_level INT`. Đặt `status` mặc định là `'Pending'` và `priority_level` mặc định là `3`.

**Bài 13.2.**

Tạo bảng `products_default` gồm `product_id INT`, `product_name VARCHAR(150)`, `quantity_in_stock INT`, `is_active TINYINT`. Đặt `quantity_in_stock` mặc định là `0` và `is_active` mặc định là `1`.

**Bài 13.3.**

Tạo bảng `orders_default` gồm `order_id INT`, `customer_name VARCHAR(100)`, `status VARCHAR(20)`, `created_at DATETIME`. Đặt `status` mặc định là `'New'` và `created_at` mặc định là `CURRENT_TIMESTAMP`.

**Bài 13.4.**

Chèn một dòng vào `tasks_default` chỉ với `task_id` và `task_name`. Sau đó truy vấn để kiểm tra các giá trị default.

**Bài 13.5.**

Chèn một dòng vào `products_default` với `quantity_in_stock = 25` và `is_active = 0`. Giải thích vì sao các giá trị này không dùng giá trị default.

---

## 14. Kết hợp nhiều constraint trong một bảng

Trong thực tế, các constraint thường được dùng cùng nhau.

Ví dụ bảng `course_registrations`:

```sql
DROP TABLE IF EXISTS course_registrations;
DROP TABLE IF EXISTS students_constraints;
DROP TABLE IF EXISTS courses_constraints;

CREATE TABLE students_constraints (
    student_id INT,
    student_code VARCHAR(20) NOT NULL,
    full_name VARCHAR(100) NOT NULL,
    email VARCHAR(150) NOT NULL,
    CONSTRAINT pk_students_constraints PRIMARY KEY (student_id),
    CONSTRAINT uq_students_constraints_code UNIQUE (student_code),
    CONSTRAINT uq_students_constraints_email UNIQUE (email)
) ENGINE = InnoDB;

CREATE TABLE courses_constraints (
    course_id INT,
    course_name VARCHAR(150) NOT NULL,
    credits INT NOT NULL DEFAULT 3,
    CONSTRAINT pk_courses_constraints PRIMARY KEY (course_id),
    CONSTRAINT chk_courses_constraints_credits
        CHECK (credits BETWEEN 1 AND 6)
) ENGINE = InnoDB;

CREATE TABLE course_registrations (
    student_id INT,
    course_id INT,
    semester VARCHAR(20) NOT NULL DEFAULT '2026A',
    final_score DECIMAL(4,2),
    registered_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT pk_course_registrations
        PRIMARY KEY (student_id, course_id, semester),
    CONSTRAINT fk_course_registrations_student
        FOREIGN KEY (student_id)
        REFERENCES students_constraints(student_id),
    CONSTRAINT fk_course_registrations_course
        FOREIGN KEY (course_id)
        REFERENCES courses_constraints(course_id),
    CONSTRAINT chk_course_registrations_score
        CHECK (final_score BETWEEN 0 AND 10)
) ENGINE = InnoDB;
```

Bảng trên minh họa:

| Thành phần | Constraint được dùng |
|---|---|
| `students_constraints.student_id` | `PRIMARY KEY` |
| `student_code`, `email` | `NOT NULL`, `UNIQUE` |
| `courses_constraints.credits` | `NOT NULL`, `DEFAULT`, `CHECK` |
| `course_registrations` | composite `PRIMARY KEY` |
| `student_id`, `course_id` | `FOREIGN KEY` |
| `semester`, `registered_at` | `DEFAULT` |
| `final_score` | `CHECK` |

### Bài tập thực hành

**Bài 14.1.**

Tạo bảng `authors_constraints` có `author_id` là primary key, `full_name` là `NOT NULL`, `email` là `NOT NULL UNIQUE`.

**Bài 14.2.**

Tạo bảng `books_constraints` có `book_id` là primary key, `author_id` là foreign key tham chiếu `authors_constraints(author_id)`, `title` là `NOT NULL`, `price` có `CHECK (price > 0)`, `stock` có `DEFAULT 0`.

**Bài 14.3.**

Tạo bảng `borrowings_constraints` có primary key ghép `(student_id, book_id, borrowed_date)`, trong đó `borrowed_date` có default là ngày hiện tại phù hợp với MySQL của bạn.

**Bài 14.4.**

Dùng `SHOW CREATE TABLE` để xác định tất cả constraint trong `books_constraints`.

**Bài 14.5.**

Chỉ ra constraint nào trong `books_constraints` ngăn mỗi tình huống sau: tiêu đề rỗng (`NULL`), giá âm, author không tồn tại, book_id trùng.

---

## 15. Một số lỗi thường gặp

### Lỗi 1: Nhầm `PRIMARY KEY` và `UNIQUE`

Sai nhận định:

> Một bảng có thể có nhiều primary key.

Đúng:

- Mỗi bảng chỉ có tối đa một primary key.
- Primary key có thể là khóa ghép.
- Một bảng có thể có nhiều unique constraints.

---

### Lỗi 2: Tạo bảng con trước bảng cha khi có foreign key

Sai thứ tự:

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    department_id INT,
    FOREIGN KEY (department_id)
        REFERENCES departments(department_id)
);
```

nếu bảng `departments` chưa tồn tại.

Đúng:

1. Tạo bảng cha `departments` trước.
2. Tạo bảng con `employees` sau.

---

### Lỗi 3: Dùng `SET NULL` trên cột không cho phép `NULL`

Ví dụ không phù hợp:

```sql
department_id INT NOT NULL,
FOREIGN KEY (department_id)
    REFERENCES departments(department_id)
    ON DELETE SET NULL
```

Khi xóa department, DBMS không thể đặt `department_id` thành `NULL` vì cột này là `NOT NULL`.

---

### Lỗi 4: Nghĩ rằng `UNIQUE` luôn chỉ cho phép một `NULL`

Trong MySQL, unique index có thể cho phép nhiều giá trị `NULL` nếu cột cho phép `NULL`.

Nếu cần email vừa bắt buộc vừa không trùng:

```sql
email VARCHAR(150) NOT NULL UNIQUE
```

---

### Lỗi 5: Dùng `CHECK` nhưng không kiểm tra phiên bản MySQL

Với MySQL cũ hơn 8.0.16, `CHECK` có thể không được thực thi theo kỳ vọng.

Kiểm tra phiên bản:

```sql
SELECT VERSION();
```

---

### Lỗi 6: Tắt `FOREIGN_KEY_CHECKS` và quên bật lại

Sai:

```sql
SET FOREIGN_KEY_CHECKS = 0;
-- thao tác dữ liệu
-- quên SET FOREIGN_KEY_CHECKS = 1;
```

Đúng:

```sql
SET FOREIGN_KEY_CHECKS = 0;
-- chỉ thực hiện thao tác cần thiết
SET FOREIGN_KEY_CHECKS = 1;
```

---

## 16. Đáp án gợi ý cho một số bài tập

### Bài 5.1

```sql
CREATE TABLE courses (
    course_id INT,
    course_name VARCHAR(150),
    credits INT,
    CONSTRAINT pk_courses PRIMARY KEY (course_id)
) ENGINE = InnoDB;
```

### Bài 6.2

```sql
CREATE TABLE order_lines (
    order_id INT,
    product_id INT,
    quantity INT,
    price_each DECIMAL(10,2),
    CONSTRAINT pk_order_lines PRIMARY KEY (order_id, product_id)
) ENGINE = InnoDB;
```

### Bài 7.2

```sql
CREATE TABLE students_fk (
    student_id INT,
    full_name VARCHAR(100),
    faculty_id INT,
    CONSTRAINT pk_students_fk PRIMARY KEY (student_id),
    CONSTRAINT fk_students_fk_faculties
        FOREIGN KEY (faculty_id)
        REFERENCES faculties(faculty_id)
) ENGINE = InnoDB;
```

### Bài 8.3

```sql
ALTER TABLE products_alter_fk
ADD CONSTRAINT fk_products_alter_fk_suppliers
FOREIGN KEY (supplier_id)
REFERENCES suppliers(supplier_id);
```

### Bài 10.1

```sql
CREATE TABLE teachers_not_null (
    teacher_id INT,
    full_name VARCHAR(100) NOT NULL,
    email VARCHAR(150) NOT NULL,
    hire_date DATE NOT NULL,
    CONSTRAINT pk_teachers_not_null PRIMARY KEY (teacher_id)
) ENGINE = InnoDB;
```

### Bài 11.3

```sql
CREATE TABLE room_bookings (
    booking_id INT,
    room_id INT NOT NULL,
    booking_date DATE NOT NULL,
    time_slot VARCHAR(20) NOT NULL,
    CONSTRAINT pk_room_bookings PRIMARY KEY (booking_id),
    CONSTRAINT uq_room_bookings_slot
        UNIQUE (room_id, booking_date, time_slot)
) ENGINE = InnoDB;
```

### Bài 12.4

```sql
CREATE TABLE promotions (
    promotion_id INT,
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    discount_percent DECIMAL(5,2) NOT NULL,
    CONSTRAINT pk_promotions PRIMARY KEY (promotion_id),
    CONSTRAINT chk_promotions_dates
        CHECK (end_date >= start_date),
    CONSTRAINT chk_promotions_discount
        CHECK (discount_percent BETWEEN 0 AND 100)
) ENGINE = InnoDB;
```

### Bài 13.3

```sql
CREATE TABLE orders_default (
    order_id INT,
    customer_name VARCHAR(100),
    status VARCHAR(20) DEFAULT 'New',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT pk_orders_default PRIMARY KEY (order_id)
) ENGINE = InnoDB;
```

### Bài 14.2

```sql
CREATE TABLE books_constraints (
    book_id INT,
    author_id INT,
    title VARCHAR(200) NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    stock INT NOT NULL DEFAULT 0,
    CONSTRAINT pk_books_constraints PRIMARY KEY (book_id),
    CONSTRAINT fk_books_constraints_authors
        FOREIGN KEY (author_id)
        REFERENCES authors_constraints(author_id),
    CONSTRAINT chk_books_constraints_price
        CHECK (price > 0)
) ENGINE = InnoDB;
```

---

## 17. Tóm tắt

Các kiến thức chính trong tutorial:

- `PRIMARY KEY` nhận diện duy nhất mỗi dòng và không cho phép `NULL`.
- Composite primary key dùng tổ hợp nhiều cột để nhận diện một dòng.
- `FOREIGN KEY` bảo đảm dữ liệu tham chiếu giữa bảng cha và bảng con hợp lệ.
- `FOREIGN_KEY_CHECKS` chỉ nên tắt tạm thời trong môi trường kiểm soát được.
- `NOT NULL` yêu cầu cột phải có giá trị.
- `UNIQUE` ngăn các giá trị trùng lặp; thường kết hợp với `NOT NULL` cho dữ liệu như username hoặc email.
- `CHECK` kiểm tra một quy tắc logic; với MySQL nên dùng phiên bản 8.0.16 trở lên để constraint được thực thi.
- `DEFAULT` tự cấp giá trị khi cột bị bỏ qua trong `INSERT`.
- Nên đặt tên constraint tường minh và dùng `SHOW CREATE TABLE` để kiểm tra cấu trúc bảng.
- Nhiều constraint thường được kết hợp để phản ánh đầy đủ quy tắc nghiệp vụ.

---

## 18. Từ khóa chính

- SQL
- MySQL
- CREATE TABLE
- Constraint
- PRIMARY KEY
- Composite Primary Key
- FOREIGN KEY
- Referential Integrity
- FOREIGN_KEY_CHECKS
- NOT NULL
- UNIQUE
- CHECK
- DEFAULT
- ON DELETE CASCADE
- ON UPDATE CASCADE
- SHOW CREATE TABLE
- ALTER TABLE
- InnoDB
