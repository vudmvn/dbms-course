---
title: "Lab: Stored Procedures cơ bản trong MySQL với classicmodels"
author: "Tên giảng viên"
duration: "150m"
difficulty: "Intermediate"
prerequisites:
  - "Đã biết SELECT, WHERE, JOIN, GROUP BY và các hàm tổng hợp cơ bản"
  - "Đã biết sử dụng MySQL Workbench hoặc MySQL command-line client"
  - "Có quyền CREATE ROUTINE và EXECUTE trên cơ sở dữ liệu thực hành"
summary: "Thực hành stored procedure cơ bản trong MySQL: delimiter, CREATE PROCEDURE, DROP PROCEDURE, biến cục bộ, tham số IN/OUT/INOUT, thay đổi và liệt kê procedure trên classicmodels."
---

# Lab: Stored Procedures cơ bản trong MySQL với `classicmodels`

## Link tham khảo

- [MySQL Tutorial — Stored Procedures](https://www.mysqltutorial.org/mysql-stored-procedure/)
- [MySQL Tutorial — Introduction to Stored Procedures](https://www.mysqltutorial.org/mysql-stored-procedure/introduction-to-sql-stored-procedures/)
- [MySQL Tutorial — Delimiters](https://www.mysqltutorial.org/mysql-stored-procedure/mysql-delimiter/)
- [MySQL Tutorial — Create Procedure](https://www.mysqltutorial.org/mysql-stored-procedure/mysql-create-procedure/)
- [MySQL Tutorial — Variables in Stored Procedures](https://www.mysqltutorial.org/mysql-stored-procedure/mysql-stored-procedure-variables/)
- [MySQL Tutorial — Parameters](https://www.mysqltutorial.org/mysql-stored-procedure/mysql-stored-procedure-parameters/)


## 1. Mục tiêu học tập

Sau khi hoàn thành lab này, người học có thể:

1. Giải thích được stored procedure là gì và khi nào nên sử dụng.
2. Nêu được các ưu điểm và hạn chế chính của stored procedure.
3. Thay đổi delimiter để tạo procedure gồm nhiều câu lệnh.
4. Tạo và gọi procedure bằng `CREATE PROCEDURE` và `CALL`.
5. Xóa procedure bằng `DROP PROCEDURE`.
6. Khai báo và sử dụng biến cục bộ bằng `DECLARE`.
7. Phân biệt tham số `IN`, `OUT` và `INOUT`.
8. Viết procedure đọc dữ liệu từ các bảng trong `classicmodels`.
9. Thay đổi thân procedure bằng quy trình `DROP PROCEDURE` rồi `CREATE PROCEDURE`.
10. Liệt kê, kiểm tra định nghĩa và dọn dẹp các procedure đã tạo.

---

## 2. Stored procedure là gì?

**Stored procedure** là một chương trình SQL được lưu trong database server dưới một tên xác định. Sau khi tạo, procedure có thể được gọi lại bằng câu lệnh `CALL`.

Ví dụ, thay vì nhiều lần viết truy vấn:

```sql
SELECT customerNumber, customerName, city, country, creditLimit
FROM customers
WHERE country = 'USA'
ORDER BY customerName;
```

ta có thể tạo procedure:

```sql
CALL sp_lab_list_customers_by_country('USA');
```

Procedure sẽ chứa truy vấn bên trong database.

### 2.1. Một số đặc điểm

| Đặc điểm | Ý nghĩa |
|---|---|
| Có tên | Procedure được lưu và gọi bằng tên |
| Có thể có tham số | Có thể nhận dữ liệu đầu vào và trả kết quả ra ngoài |
| Có thể chứa nhiều câu lệnh | Ví dụ `SELECT`, `INSERT`, `UPDATE`, `DELETE`, điều kiện, vòng lặp |
| Chạy tại server | Logic SQL được thực thi ở MySQL Server |
| Không tự trả về một giá trị như function | Procedure có thể trả result set hoặc dùng tham số `OUT`/`INOUT` |

> Stored procedure khác với stored function. Function thường được gọi trong biểu thức SQL và phải trả về một giá trị; procedure được gọi bằng `CALL` và có thể thực hiện nhiều thao tác.

### 2.2. Ưu điểm

| Ưu điểm | Diễn giải |
|---|---|
| Tái sử dụng | Cùng một logic SQL có thể được gọi nhiều lần |
| Tập trung logic dữ liệu | Một phần nghiệp vụ được lưu gần dữ liệu |
| Giảm lặp lại SQL ở ứng dụng | Ứng dụng chỉ cần gọi procedure với tham số |
| Kiểm soát quyền | Có thể cấp quyền thực thi procedure thay vì cấp quyền trực tiếp trên bảng trong một số thiết kế |
| Có thể giảm số lần gửi câu lệnh từ ứng dụng đến server | Một procedure có thể gom nhiều câu lệnh SQL |

### 2.3. Hạn chế

| Hạn chế | Diễn giải |
|---|---|
| Khó kiểm thử và quản lý phiên bản hơn mã ứng dụng | Cần quản lý script tạo procedure cẩn thận |
| Phụ thuộc DBMS | Cú pháp MySQL có thể khác SQL Server, PostgreSQL hoặc Oracle |
| Logic phức tạp khó bảo trì | Procedure dài có thể khó đọc, khó debug |
| Có thể tạo phụ thuộc ẩn | Ứng dụng phụ thuộc vào tên và hành vi của procedure |
| Cần quyền phù hợp | Người dùng có thể không được phép tạo routine |

### Bài tập thực hành

**Bài 2.1.**

Giải thích stored procedure là gì bằng một hoặc hai câu.

**Bài 2.2.**

Nêu hai ưu điểm của stored procedure.

**Bài 2.3.**

Nêu hai hạn chế của stored procedure.

**Bài 2.4.**

Stored procedure và stored function khác nhau thế nào về cách gọi?

**Bài 2.5.**

Trong hệ thống bán hàng, hãy nêu một nghiệp vụ có thể phù hợp để đóng gói thành stored procedure.

---

## 3. Chuẩn bị môi trường

Chọn cơ sở dữ liệu:

```sql
USE classicmodels;
```

Kiểm tra các bảng thường dùng trong lab:

```sql
SHOW TABLES;
```

Xem nhanh dữ liệu mẫu:

```sql
SELECT customerNumber, customerName, city, country, creditLimit
FROM customers
ORDER BY customerNumber
LIMIT 10;
```

```sql
SELECT orderNumber, orderDate, status, customerNumber
FROM orders
ORDER BY orderNumber
LIMIT 10;
```

```sql
SELECT orderNumber, productCode, quantityOrdered, priceEach
FROM orderdetails
ORDER BY orderNumber, productCode
LIMIT 10;
```

Kiểm tra quyền tạo procedure nếu cần:

```sql
SHOW GRANTS;
```

Trong lab này, mọi procedure được đặt tên với tiền tố:

```text
sp_lab_
```

Ví dụ:

```text
sp_lab_list_customers_by_country
sp_lab_count_orders_by_customer
sp_lab_order_summary
```

Tiền tố giúp phân biệt procedure thực hành với procedure có sẵn của hệ thống.

### Bài tập thực hành

**Bài 3.1.**

Viết lệnh chọn cơ sở dữ liệu `classicmodels`.

**Bài 3.2.**

Viết lệnh xem danh sách bảng trong database hiện tại.

**Bài 3.3.**

Viết truy vấn xem 10 khách hàng đầu tiên trong bảng `customers`.

**Bài 3.4.**

Viết truy vấn xem 10 đơn hàng đầu tiên trong bảng `orders`.

**Bài 3.5.**

Viết lệnh kiểm tra các quyền hiện có của tài khoản đang đăng nhập.

---

## 4. Thay đổi delimiter

### 4.1. Vì sao cần delimiter?

Mặc định, MySQL client xem dấu chấm phẩy `;` là ký hiệu kết thúc một câu lệnh SQL.

Một procedure thường có nhiều câu lệnh bên trong `BEGIN ... END`, mỗi câu lệnh lại kết thúc bằng `;`.

Ví dụ:

```sql
BEGIN
    SELECT ...;
    SELECT ...;
END;
```

Nếu không thay delimiter, client có thể kết thúc lệnh `CREATE PROCEDURE` quá sớm tại dấu `;` đầu tiên bên trong procedure.

Vì vậy, khi tạo procedure nhiều câu lệnh, ta tạm đổi delimiter.

### 4.2. Mẫu thay đổi delimiter

```sql
DELIMITER $$

CREATE PROCEDURE procedure_name()
BEGIN
    SELECT 'Hello';
END$$

DELIMITER ;
```

Ý nghĩa:

| Dòng lệnh | Vai trò |
|---|---|
| `DELIMITER $$` | Yêu cầu client dùng `$$` để kết thúc lệnh tạm thời |
| `END$$` | Kết thúc toàn bộ câu `CREATE PROCEDURE` |
| `DELIMITER ;` | Đưa delimiter trở lại dấu `;` thông thường |

> `DELIMITER` là lệnh của client như MySQL command-line client hoặc MySQL Workbench, không phải một câu lệnh SQL được server MySQL lưu trong procedure.

### 4.3. Procedure chỉ có một câu lệnh

Với procedure có đúng một câu lệnh, có thể tạo không cần `BEGIN ... END`.

```sql
DELIMITER $$

CREATE PROCEDURE sp_lab_list_offices()
SELECT officeCode, city, country, territory
FROM offices
ORDER BY officeCode$$

DELIMITER ;
```

Tuy nhiên, trong lab này nên dùng `BEGIN ... END` khi học để dễ mở rộng procedure sau này.

### 4.4. Procedure nhiều câu lệnh

```sql
DELIMITER $$

CREATE PROCEDURE sp_lab_show_database_info()
BEGIN
    SELECT DATABASE() AS currentDatabase;
    SELECT COUNT(*) AS totalCustomers
    FROM customers;
END$$

DELIMITER ;
```

Gọi procedure:

```sql
CALL sp_lab_show_database_info();
```

Procedure trên trả về hai result set.

### Bài tập thực hành

**Bài 4.1.**

Viết các dòng lệnh thay delimiter sang `$$` rồi đổi lại thành `;`.

**Bài 4.2.**

Tạo procedure `sp_lab_hello` chỉ trả về chuỗi `Hello stored procedure`.

**Bài 4.3.**

Tạo procedure `sp_lab_show_offices` hiển thị `officeCode`, `city`, `country` từ bảng `offices`.

**Bài 4.4.**

Tạo procedure `sp_lab_show_counts` trả về hai result set: số khách hàng và số đơn hàng.

**Bài 4.5.**

Giải thích vì sao dấu `;` trong `BEGIN ... END` không được dùng trực tiếp làm dấu kết thúc của toàn bộ lệnh `CREATE PROCEDURE`.

---

## 5. Tạo và gọi stored procedure

### 5.1. Cú pháp `CREATE PROCEDURE`

Cú pháp đơn giản:

```sql
CREATE PROCEDURE procedure_name()
BEGIN
    statement_1;
    statement_2;
END;
```

Cú pháp có tham số:

```sql
CREATE PROCEDURE procedure_name(
    parameter_definition
)
BEGIN
    statement_1;
END;
```

### 5.2. Tạo procedure không có tham số

Trước khi tạo, xóa procedure cũ cùng tên nếu có:

```sql
DROP PROCEDURE IF EXISTS sp_lab_list_product_lines;
```

Tạo procedure:

```sql
DELIMITER $$

CREATE PROCEDURE sp_lab_list_product_lines()
BEGIN
    SELECT productLine, textDescription
    FROM productlines
    ORDER BY productLine;
END$$

DELIMITER ;
```

Gọi procedure:

```sql
CALL sp_lab_list_product_lines();
```

### 5.3. Tạo procedure trả về danh sách đơn hàng

```sql
DROP PROCEDURE IF EXISTS sp_lab_list_shipped_orders;

DELIMITER $$

CREATE PROCEDURE sp_lab_list_shipped_orders()
BEGIN
    SELECT orderNumber, orderDate, shippedDate, status, customerNumber
    FROM orders
    WHERE status = 'Shipped'
    ORDER BY shippedDate DESC, orderNumber DESC;
END$$

DELIMITER ;
```

Gọi procedure:

```sql
CALL sp_lab_list_shipped_orders();
```

### 5.4. Tạo procedure có truy vấn tổng hợp

```sql
DROP PROCEDURE IF EXISTS sp_lab_count_customers_by_country;

DELIMITER $$

CREATE PROCEDURE sp_lab_count_customers_by_country()
BEGIN
    SELECT country, COUNT(*) AS totalCustomers
    FROM customers
    GROUP BY country
    ORDER BY totalCustomers DESC, country;
END$$

DELIMITER ;
```

Gọi procedure:

```sql
CALL sp_lab_count_customers_by_country();
```

### 5.5. `CALL`

Cú pháp:

```sql
CALL procedure_name(argument_list);
```

Ví dụ procedure không có tham số:

```sql
CALL sp_lab_list_product_lines();
```

Ví dụ procedure có tham số sẽ được trình bày ở phần sau:

```sql
CALL sp_lab_list_customers_by_country('USA');
```

### Bài tập thực hành

**Bài 5.1.**

Tạo procedure `sp_lab_list_offices` hiển thị toàn bộ văn phòng, sắp xếp theo `officeCode`.

**Bài 5.2.**

Tạo procedure `sp_lab_list_motorcycles` hiển thị `productCode`, `productName`, `buyPrice`, `MSRP` của các sản phẩm thuộc dòng `Motorcycles`.

**Bài 5.3.**

Tạo procedure `sp_lab_list_cancelled_orders` hiển thị các đơn hàng có trạng thái `Cancelled`.

**Bài 5.4.**

Tạo procedure `sp_lab_count_orders_by_status` trả về mỗi `status` và số lượng đơn hàng tương ứng.

**Bài 5.5.**

Gọi toàn bộ procedure đã tạo ở Bài 5.1 đến Bài 5.4.

---

## 6. Xóa stored procedure bằng `DROP PROCEDURE`

### 6.1. Cú pháp

```sql
DROP PROCEDURE procedure_name;
```

Ví dụ:

```sql
DROP PROCEDURE sp_lab_list_product_lines;
```

Nếu procedure không tồn tại, MySQL báo lỗi.

### 6.2. Dùng `IF EXISTS`

Để tránh lỗi khi procedure có thể chưa tồn tại:

```sql
DROP PROCEDURE IF EXISTS sp_lab_list_product_lines;
```

Câu lệnh này đặc biệt hữu ích trong script triển khai, vì script có thể chạy lại nhiều lần.

### 6.3. Kiểm tra trước và sau khi xóa

Liệt kê procedure:

```sql
SHOW PROCEDURE STATUS
WHERE Db = 'classicmodels'
  AND Name = 'sp_lab_list_product_lines';
```

Xóa:

```sql
DROP PROCEDURE IF EXISTS sp_lab_list_product_lines;
```

Kiểm tra lại:

```sql
SHOW PROCEDURE STATUS
WHERE Db = 'classicmodels'
  AND Name = 'sp_lab_list_product_lines';
```

### 6.4. Lưu ý

- `DROP PROCEDURE` chỉ xóa definition của procedure, không xóa bảng hoặc dữ liệu.
- Các ứng dụng hoặc procedure khác gọi procedure đã xóa sẽ lỗi.
- Nên lưu script `CREATE PROCEDURE` trong file quản lý phiên bản để có thể tạo lại khi cần.

### Bài tập thực hành

**Bài 6.1.**

Xóa procedure `sp_lab_list_offices` nếu procedure này tồn tại.

**Bài 6.2.**

Tạo procedure `sp_lab_temp_procedure` chỉ trả về `CURRENT_DATE()`.

**Bài 6.3.**

Gọi `sp_lab_temp_procedure`.

**Bài 6.4.**

Xóa `sp_lab_temp_procedure` bằng `DROP PROCEDURE IF EXISTS`.

**Bài 6.5.**

Kiểm tra bằng `SHOW PROCEDURE STATUS` rằng `sp_lab_temp_procedure` không còn tồn tại.

---

## 7. Biến cục bộ trong stored procedure

### 7.1. Khai báo biến bằng `DECLARE`

Biến cục bộ chỉ tồn tại trong lúc procedure chạy.

Cú pháp:

```sql
DECLARE variable_name data_type [DEFAULT default_value];
```

Ví dụ:

```sql
DECLARE v_total_orders INT DEFAULT 0;
```

```sql
DECLARE v_total_amount DECIMAL(12,2) DEFAULT 0.00;
```

### 7.2. Quy tắc vị trí của `DECLARE`

Trong một block `BEGIN ... END`, các lệnh `DECLARE` phải nằm ở đầu block, trước các câu lệnh xử lý như `SELECT`, `SET`, `IF`, `INSERT`, `UPDATE`.

Sai:

```sql
BEGIN
    SELECT COUNT(*)
    FROM orders;

    DECLARE v_total_orders INT;
END
```

Đúng:

```sql
BEGIN
    DECLARE v_total_orders INT DEFAULT 0;

    SELECT COUNT(*)
    INTO v_total_orders
    FROM orders;
END
```

### 7.3. Gán giá trị bằng `SET`

```sql
SET v_total_orders = 0;
```

Ví dụ:

```sql
DROP PROCEDURE IF EXISTS sp_lab_local_variable_demo;

DELIMITER $$

CREATE PROCEDURE sp_lab_local_variable_demo()
BEGIN
    DECLARE v_message VARCHAR(100) DEFAULT 'Stored procedure is running';

    SELECT v_message AS message;
END$$

DELIMITER ;
```

Gọi procedure:

```sql
CALL sp_lab_local_variable_demo();
```

### 7.4. Gán kết quả truy vấn vào biến bằng `SELECT ... INTO`

```sql
SELECT COUNT(*)
INTO v_total_orders
FROM orders;
```

Ví dụ: thống kê số đơn hàng và tổng số dòng chi tiết của một đơn hàng.

```sql
DROP PROCEDURE IF EXISTS sp_lab_order_summary;

DELIMITER $$

CREATE PROCEDURE sp_lab_order_summary()
BEGIN
    DECLARE v_order_number INT DEFAULT 10100;
    DECLARE v_total_lines INT DEFAULT 0;
    DECLARE v_total_quantity INT DEFAULT 0;

    SELECT COUNT(*), COALESCE(SUM(quantityOrdered), 0)
    INTO v_total_lines, v_total_quantity
    FROM orderdetails
    WHERE orderNumber = v_order_number;

    SELECT v_order_number AS orderNumber,
           v_total_lines AS totalLines,
           v_total_quantity AS totalQuantity;
END$$

DELIMITER ;
```

Gọi procedure:

```sql
CALL sp_lab_order_summary();
```

### 7.5. Biến cục bộ và session variable

| Loại biến | Ví dụ | Phạm vi |
|---|---|---|
| Biến cục bộ | `v_total_orders` | Chỉ bên trong procedure/block |
| Session variable | `@order_count` | Trong phiên kết nối hiện tại |
| Tham số procedure | `p_customer_number` | Trong lúc procedure được gọi |

> Tên biến cục bộ trong lab bắt đầu bằng `v_`.  
> Tên tham số trong lab bắt đầu bằng `p_`.  
> Session variable có tiền tố `@` và thường dùng để nhận giá trị từ tham số `OUT` hoặc `INOUT`.

### Bài tập thực hành

**Bài 7.1.**

Tạo procedure `sp_lab_show_message` dùng biến cục bộ `v_message` để trả về chuỗi `Welcome to MySQL stored procedures`.

**Bài 7.2.**

Tạo procedure `sp_lab_total_customers` khai báo biến `v_total_customers`, đếm số khách hàng trong `customers` bằng `SELECT ... INTO`, sau đó hiển thị biến này.

**Bài 7.3.**

Tạo procedure `sp_lab_total_products` khai báo biến `v_total_products`, đếm số sản phẩm trong `products`, sau đó hiển thị kết quả.

**Bài 7.4.**

Tạo procedure `sp_lab_order_10100_quantity` dùng biến cục bộ để tính tổng `quantityOrdered` của đơn hàng `10100`.

**Bài 7.5.**

Giải thích vì sao mọi lệnh `DECLARE` phải đặt trước `SELECT`, `SET` hoặc các lệnh xử lý khác trong cùng một block.

---

## 8. Tham số `IN`, `OUT` và `INOUT`

Stored procedure có thể nhận hoặc trả dữ liệu qua tham số.

| Loại tham số | Vai trò |
|---|---|
| `IN` | Nhận dữ liệu đầu vào; procedure đọc giá trị này |
| `OUT` | Trả giá trị ra ngoài cho bên gọi |
| `INOUT` | Nhận một giá trị đầu vào và có thể thay đổi, trả lại giá trị mới |

### 8.1. Tham số `IN`

`IN` là mặc định nếu không ghi rõ mode.

Cú pháp:

```sql
IN parameter_name data_type
```

Ví dụ: liệt kê khách hàng theo quốc gia.

```sql
DROP PROCEDURE IF EXISTS sp_lab_list_customers_by_country;

DELIMITER $$

CREATE PROCEDURE sp_lab_list_customers_by_country(
    IN p_country VARCHAR(50)
)
BEGIN
    SELECT customerNumber, customerName, city, country, creditLimit
    FROM customers
    WHERE country = p_country
    ORDER BY customerName;
END$$

DELIMITER ;
```

Gọi procedure:

```sql
CALL sp_lab_list_customers_by_country('USA');
```

```sql
CALL sp_lab_list_customers_by_country('France');
```

### 8.2. Tham số `OUT`

`OUT` trả giá trị cho bên gọi.

Cú pháp:

```sql
OUT parameter_name data_type
```

Ví dụ: đếm số đơn hàng của một khách hàng.

```sql
DROP PROCEDURE IF EXISTS sp_lab_count_orders_by_customer;

DELIMITER $$

CREATE PROCEDURE sp_lab_count_orders_by_customer(
    IN p_customer_number INT,
    OUT p_order_count INT
)
BEGIN
    SELECT COUNT(*)
    INTO p_order_count
    FROM orders
    WHERE customerNumber = p_customer_number;
END$$

DELIMITER ;
```

Gọi procedure:

```sql
CALL sp_lab_count_orders_by_customer(103, @order_count);

SELECT @order_count AS totalOrders;
```

Trong ví dụ này:

- `103` là giá trị đầu vào cho `p_customer_number`.
- `@order_count` là session variable nhận giá trị từ `p_order_count`.

### 8.3. Tham số `INOUT`

`INOUT` vừa nhận giá trị đầu vào, vừa trả lại giá trị đã được thay đổi.

Cú pháp:

```sql
INOUT parameter_name data_type
```

Ví dụ tính giá trị sau khi cộng thuế:

```sql
DROP PROCEDURE IF EXISTS sp_lab_add_tax;

DELIMITER $$

CREATE PROCEDURE sp_lab_add_tax(
    INOUT p_amount DECIMAL(10,2),
    IN p_tax_rate DECIMAL(5,2)
)
BEGIN
    SET p_amount = p_amount * (1 + p_tax_rate / 100);
END$$

DELIMITER ;
```

Gọi procedure:

```sql
SET @amount = 100.00;

CALL sp_lab_add_tax(@amount, 10.00);

SELECT @amount AS amountAfterTax;
```

Kết quả mong đợi:

```text
110.00
```

### 8.4. Ví dụ kết hợp `IN` và `OUT`

```sql
DROP PROCEDURE IF EXISTS sp_lab_customer_credit_limit;

DELIMITER $$

CREATE PROCEDURE sp_lab_customer_credit_limit(
    IN p_customer_number INT,
    OUT p_credit_limit DECIMAL(10,2)
)
BEGIN
    SELECT creditLimit
    INTO p_credit_limit
    FROM customers
    WHERE customerNumber = p_customer_number;
END$$

DELIMITER ;
```

Gọi:

```sql
CALL sp_lab_customer_credit_limit(103, @credit_limit);

SELECT @credit_limit AS creditLimit;
```

### Bài tập thực hành

**Bài 8.1.**

Tạo procedure `sp_lab_list_orders_by_status` với tham số `IN p_status VARCHAR(20)`; procedure hiển thị các đơn hàng có trạng thái được truyền vào.

**Bài 8.2.**

Tạo procedure `sp_lab_count_payments_by_customer` với `IN p_customer_number INT` và `OUT p_payment_count INT`; procedure trả về số khoản thanh toán của khách hàng.

**Bài 8.3.**

Gọi `sp_lab_count_payments_by_customer` cho khách hàng `103` và hiển thị session variable nhận kết quả.

**Bài 8.4.**

Tạo procedure `sp_lab_apply_discount` với `INOUT p_price DECIMAL(10,2)` và `IN p_discount_percent DECIMAL(5,2)`; procedure giảm giá theo phần trăm.

**Bài 8.5.**

Gọi `sp_lab_apply_discount` với giá ban đầu `200.00` và phần trăm giảm `15.00`, sau đó hiển thị giá mới.

---

## 9. Thay đổi stored procedure

### 9.1. Không dùng `ALTER PROCEDURE` để sửa thân procedure

Trong MySQL, `ALTER PROCEDURE` không dùng để thay đổi trực tiếp phần thân SQL của procedure. Nó chỉ thay đổi một số đặc tính, chẳng hạn `COMMENT` hoặc `SQL SECURITY`.

Ví dụ:

```sql
ALTER PROCEDURE sp_lab_list_product_lines
COMMENT 'List all product lines';
```

Để thay đổi các câu lệnh bên trong procedure, quy trình thông dụng là:

1. Xem definition hiện có.
2. `DROP PROCEDURE`.
3. `CREATE PROCEDURE` lại với definition mới.
4. Gọi procedure để kiểm tra.

### 9.2. Quy trình sửa procedure

Giả sử procedure cũ:

```sql
DROP PROCEDURE IF EXISTS sp_lab_list_customers_by_country;

DELIMITER $$

CREATE PROCEDURE sp_lab_list_customers_by_country(
    IN p_country VARCHAR(50)
)
BEGIN
    SELECT customerNumber, customerName, city, country
    FROM customers
    WHERE country = p_country
    ORDER BY customerName;
END$$

DELIMITER ;
```

Muốn thêm `creditLimit` vào kết quả:

```sql
DROP PROCEDURE IF EXISTS sp_lab_list_customers_by_country;

DELIMITER $$

CREATE PROCEDURE sp_lab_list_customers_by_country(
    IN p_country VARCHAR(50)
)
BEGIN
    SELECT customerNumber, customerName, city, country, creditLimit
    FROM customers
    WHERE country = p_country
    ORDER BY creditLimit DESC, customerName;
END$$

DELIMITER ;
```

Kiểm tra:

```sql
CALL sp_lab_list_customers_by_country('USA');
```

### 9.3. Dùng `SHOW CREATE PROCEDURE` trước khi sửa

```sql
SHOW CREATE PROCEDURE sp_lab_list_customers_by_country;
```

Lệnh này hiển thị câu lệnh tạo procedure hiện tại, giúp sao lưu và chỉnh sửa definition chính xác.

### 9.4. Lưu ý quan trọng

Không chạy `DROP PROCEDURE` nếu chưa có script tạo lại procedure hoặc chưa sao lưu definition cần thiết.

### Bài tập thực hành

**Bài 9.1.**

Tạo procedure `sp_lab_list_usa_customers` hiển thị `customerNumber`, `customerName`, `city` của khách hàng ở `USA`.

**Bài 9.2.**

Dùng `SHOW CREATE PROCEDURE` để xem definition của `sp_lab_list_usa_customers`.

**Bài 9.3.**

Xóa `sp_lab_list_usa_customers`, sau đó tạo lại procedure để hiển thị thêm cột `creditLimit`.

**Bài 9.4.**

Tạo procedure `sp_lab_list_products_by_line` có tham số `IN p_product_line VARCHAR(50)`.

**Bài 9.5.**

Sửa procedure `sp_lab_list_products_by_line` để kết quả được sắp xếp theo `MSRP DESC`, bằng quy trình `DROP PROCEDURE` rồi `CREATE PROCEDURE`.

---

## 10. Liệt kê và kiểm tra stored procedure

### 10.1. `SHOW PROCEDURE STATUS`

Liệt kê procedure mà tài khoản hiện tại có quyền xem:

```sql
SHOW PROCEDURE STATUS;
```

Lọc theo database:

```sql
SHOW PROCEDURE STATUS
WHERE Db = 'classicmodels';
```

Lọc theo tên:

```sql
SHOW PROCEDURE STATUS
WHERE Db = 'classicmodels'
  AND Name LIKE 'sp_lab%';
```

### 10.2. `INFORMATION_SCHEMA.ROUTINES`

Có thể truy vấn metadata từ `INFORMATION_SCHEMA.ROUTINES`.

```sql
SELECT ROUTINE_SCHEMA,
       ROUTINE_NAME,
       ROUTINE_TYPE,
       CREATED,
       LAST_ALTERED
FROM INFORMATION_SCHEMA.ROUTINES
WHERE ROUTINE_SCHEMA = 'classicmodels'
  AND ROUTINE_TYPE = 'PROCEDURE'
ORDER BY ROUTINE_NAME;
```

Chỉ liệt kê procedure lab:

```sql
SELECT ROUTINE_NAME,
       CREATED,
       LAST_ALTERED
FROM INFORMATION_SCHEMA.ROUTINES
WHERE ROUTINE_SCHEMA = 'classicmodels'
  AND ROUTINE_TYPE = 'PROCEDURE'
  AND ROUTINE_NAME LIKE 'sp_lab%'
ORDER BY ROUTINE_NAME;
```

### 10.3. `SHOW CREATE PROCEDURE`

Xem câu lệnh tạo một procedure cụ thể:

```sql
SHOW CREATE PROCEDURE sp_lab_list_customers_by_country;
```

### 10.4. Khi nào dùng từng cách?

| Cách | Khi phù hợp |
|---|---|
| `SHOW PROCEDURE STATUS` | Muốn xem nhanh danh sách procedure |
| `INFORMATION_SCHEMA.ROUTINES` | Muốn lọc, chọn cột metadata, sắp xếp hoặc kết hợp với truy vấn khác |
| `SHOW CREATE PROCEDURE` | Muốn xem definition của một procedure cụ thể |

### Bài tập thực hành

**Bài 10.1.**

Liệt kê tất cả procedure trong database `classicmodels`.

**Bài 10.2.**

Liệt kê các procedure có tên bắt đầu bằng `sp_lab`.

**Bài 10.3.**

Dùng `INFORMATION_SCHEMA.ROUTINES` để hiển thị tên, ngày tạo và ngày thay đổi gần nhất của các procedure lab.

**Bài 10.4.**

Dùng `SHOW CREATE PROCEDURE` để xem definition của `sp_lab_list_customers_by_country`.

**Bài 10.5.**

Viết truy vấn từ `INFORMATION_SCHEMA.ROUTINES` chỉ hiển thị các routine có `ROUTINE_TYPE = 'PROCEDURE'`.

---

## 11. Quy trình thực hành hoàn chỉnh

Phần này minh họa một quy trình làm việc đầy đủ: kiểm tra, xóa procedure cũ, tạo procedure mới, gọi procedure và kiểm tra metadata.

### 11.1. Procedure: thống kê đơn hàng của một khách hàng

```sql
USE classicmodels;

DROP PROCEDURE IF EXISTS sp_lab_customer_order_summary;

DELIMITER $$

CREATE PROCEDURE sp_lab_customer_order_summary(
    IN p_customer_number INT,
    OUT p_order_count INT,
    OUT p_total_order_value DECIMAL(14,2)
)
BEGIN
    SELECT COUNT(*)
    INTO p_order_count
    FROM orders
    WHERE customerNumber = p_customer_number;

    SELECT COALESCE(SUM(od.quantityOrdered * od.priceEach), 0)
    INTO p_total_order_value
    FROM orders AS o
    JOIN orderdetails AS od
        ON od.orderNumber = o.orderNumber
    WHERE o.customerNumber = p_customer_number;
END$$

DELIMITER ;
```

Gọi procedure:

```sql
CALL sp_lab_customer_order_summary(103, @order_count, @total_order_value);

SELECT @order_count AS totalOrders,
       @total_order_value AS totalOrderValue;
```

Kiểm tra definition:

```sql
SHOW CREATE PROCEDURE sp_lab_customer_order_summary;
```

Kiểm tra metadata:

```sql
SELECT ROUTINE_NAME, CREATED, LAST_ALTERED
FROM INFORMATION_SCHEMA.ROUTINES
WHERE ROUTINE_SCHEMA = 'classicmodels'
  AND ROUTINE_NAME = 'sp_lab_customer_order_summary';
```

### 11.2. Dọn dẹp sau lab

Khi kết thúc lab, có thể xóa các procedure đã tạo:

```sql
DROP PROCEDURE IF EXISTS sp_lab_hello;
DROP PROCEDURE IF EXISTS sp_lab_show_offices;
DROP PROCEDURE IF EXISTS sp_lab_show_counts;
DROP PROCEDURE IF EXISTS sp_lab_list_product_lines;
DROP PROCEDURE IF EXISTS sp_lab_list_shipped_orders;
DROP PROCEDURE IF EXISTS sp_lab_count_customers_by_country;
DROP PROCEDURE IF EXISTS sp_lab_local_variable_demo;
DROP PROCEDURE IF EXISTS sp_lab_order_summary;
DROP PROCEDURE IF EXISTS sp_lab_list_customers_by_country;
DROP PROCEDURE IF EXISTS sp_lab_count_orders_by_customer;
DROP PROCEDURE IF EXISTS sp_lab_add_tax;
DROP PROCEDURE IF EXISTS sp_lab_customer_credit_limit;
DROP PROCEDURE IF EXISTS sp_lab_customer_order_summary;
```

### Bài tập thực hành

**Bài 11.1.**

Tạo procedure `sp_lab_customer_order_summary` theo ví dụ.

**Bài 11.2.**

Gọi procedure cho khách hàng `103`.

**Bài 11.3.**

Gọi procedure cho khách hàng `112`.

**Bài 11.4.**

Dùng `SHOW CREATE PROCEDURE` để xem definition của procedure.

**Bài 11.5.**

Xóa procedure `sp_lab_customer_order_summary` sau khi hoàn thành kiểm tra.

---

## 12. Một số lỗi thường gặp

### Lỗi 1. Quên đổi delimiter

Sai:

```sql
CREATE PROCEDURE sp_lab_bad_example()
BEGIN
    SELECT 'Hello';
    SELECT 'World';
END;
```

Client có thể hiểu dấu `;` sau `SELECT 'Hello';` là kết thúc lệnh `CREATE PROCEDURE`.

Cách khắc phục:

```sql
DELIMITER $$

CREATE PROCEDURE sp_lab_good_example()
BEGIN
    SELECT 'Hello';
    SELECT 'World';
END$$

DELIMITER ;
```

---

### Lỗi 2. Tạo procedure trùng tên

Nếu procedure đã tồn tại, `CREATE PROCEDURE` báo lỗi.

Cách khắc phục:

```sql
DROP PROCEDURE IF EXISTS sp_lab_good_example;
```

Sau đó mới tạo lại procedure.

---

### Lỗi 3. Đặt `DECLARE` sau câu lệnh xử lý

Sai:

```sql
BEGIN
    SELECT COUNT(*) FROM customers;
    DECLARE v_total INT;
END
```

Đúng:

```sql
BEGIN
    DECLARE v_total INT DEFAULT 0;

    SELECT COUNT(*)
    INTO v_total
    FROM customers;
END
```

---

### Lỗi 4. Dùng biến cục bộ để nhận `OUT` khi gọi procedure

Sai:

```sql
CALL sp_lab_count_orders_by_customer(103, v_order_count);
```

`v_order_count` chỉ tồn tại bên trong stored program, không tồn tại trong session gọi procedure.

Đúng:

```sql
CALL sp_lab_count_orders_by_customer(103, @order_count);

SELECT @order_count;
```

---

### Lỗi 5. Không dùng đúng kiểu dữ liệu cho tham số

Ví dụ tham số nhận mã quốc gia nên dùng kiểu chuỗi:

```sql
IN p_country VARCHAR(50)
```

Không nên khai báo:

```sql
IN p_country INT
```

---

### Lỗi 6. Xóa procedure trước khi lưu definition cần thiết

Trước khi `DROP PROCEDURE`, nên dùng:

```sql
SHOW CREATE PROCEDURE procedure_name;
```

hoặc lưu script `CREATE PROCEDURE` trong file `.sql`.

### Bài tập thực hành

**Bài 12.1.**

Sửa procedure bị lỗi do không thay delimiter.

**Bài 12.2.**

Viết câu lệnh để xóa procedure nếu procedure có thể đã tồn tại.

**Bài 12.3.**

Sửa procedure đặt `DECLARE` sau câu `SELECT`.

**Bài 12.4.**

Sửa lời gọi procedure dùng sai biến nhận giá trị `OUT`.

**Bài 12.5.**

Nêu hai cách bảo đảm có thể tạo lại procedure sau khi đã xóa.

---

## 13. Bài tập tổng hợp

### Bài 13.1. Procedure không tham số

Tạo procedure `sp_lab_list_high_credit_customers`:

1. Không có tham số.
2. Hiển thị `customerNumber`, `customerName`, `country`, `creditLimit`.
3. Chỉ lấy khách hàng có `creditLimit >= 100000`.
4. Sắp xếp `creditLimit` giảm dần.
5. Gọi procedure để kiểm tra.

---

### Bài 13.2. Procedure có tham số `IN`

Tạo procedure `sp_lab_list_orders_by_customer`:

1. Có tham số `IN p_customer_number INT`.
2. Hiển thị `orderNumber`, `orderDate`, `requiredDate`, `status`.
3. Chỉ lấy đơn hàng của khách hàng nhận từ tham số.
4. Sắp xếp theo `orderDate DESC`.
5. Gọi procedure cho khách hàng `103`.

---

### Bài 13.3. Procedure có tham số `OUT`

Tạo procedure `sp_lab_count_customer_payments`:

1. Có tham số `IN p_customer_number INT`.
2. Có tham số `OUT p_payment_count INT`.
3. Đếm số khoản thanh toán của khách hàng.
4. Gọi procedure cho khách hàng `103`.
5. Hiển thị kết quả bằng session variable.

---

### Bài 13.4. Procedure dùng biến cục bộ

Tạo procedure `sp_lab_product_statistics`:

1. Khai báo hai biến cục bộ.
2. Đếm tổng số sản phẩm.
3. Tính giá bán đề xuất trung bình `AVG(MSRP)`.
4. Hiển thị hai giá trị bằng một câu `SELECT`.
5. Gọi procedure để kiểm tra.

---

### Bài 13.5. Quy trình sửa procedure

Với procedure `sp_lab_list_orders_by_customer`:

1. Dùng `SHOW CREATE PROCEDURE` để xem definition.
2. Xóa procedure bằng `DROP PROCEDURE IF EXISTS`.
3. Tạo lại procedure để hiển thị thêm `comments`.
4. Gọi procedure cho khách hàng `103`.
5. Dùng `INFORMATION_SCHEMA.ROUTINES` kiểm tra procedure vừa được tạo.

---

## 14. Đáp án gợi ý cho một số bài tập

### Bài 4.2

```sql
DROP PROCEDURE IF EXISTS sp_lab_hello;

DELIMITER $$

CREATE PROCEDURE sp_lab_hello()
BEGIN
    SELECT 'Hello stored procedure' AS message;
END$$

DELIMITER ;

CALL sp_lab_hello();
```

### Bài 5.2

```sql
DROP PROCEDURE IF EXISTS sp_lab_list_motorcycles;

DELIMITER $$

CREATE PROCEDURE sp_lab_list_motorcycles()
BEGIN
    SELECT productCode, productName, buyPrice, MSRP
    FROM products
    WHERE productLine = 'Motorcycles'
    ORDER BY productName;
END$$

DELIMITER ;

CALL sp_lab_list_motorcycles();
```

### Bài 6.2 đến Bài 6.4

```sql
DROP PROCEDURE IF EXISTS sp_lab_temp_procedure;

DELIMITER $$

CREATE PROCEDURE sp_lab_temp_procedure()
BEGIN
    SELECT CURRENT_DATE() AS currentDate;
END$$

DELIMITER ;

CALL sp_lab_temp_procedure();

DROP PROCEDURE IF EXISTS sp_lab_temp_procedure;
```

### Bài 7.2

```sql
DROP PROCEDURE IF EXISTS sp_lab_total_customers;

DELIMITER $$

CREATE PROCEDURE sp_lab_total_customers()
BEGIN
    DECLARE v_total_customers INT DEFAULT 0;

    SELECT COUNT(*)
    INTO v_total_customers
    FROM customers;

    SELECT v_total_customers AS totalCustomers;
END$$

DELIMITER ;

CALL sp_lab_total_customers();
```

### Bài 8.1

```sql
DROP PROCEDURE IF EXISTS sp_lab_list_orders_by_status;

DELIMITER $$

CREATE PROCEDURE sp_lab_list_orders_by_status(
    IN p_status VARCHAR(20)
)
BEGIN
    SELECT orderNumber, orderDate, requiredDate, shippedDate,
           status, customerNumber
    FROM orders
    WHERE status = p_status
    ORDER BY orderDate DESC, orderNumber DESC;
END$$

DELIMITER ;

CALL sp_lab_list_orders_by_status('Shipped');
```

### Bài 8.2

```sql
DROP PROCEDURE IF EXISTS sp_lab_count_payments_by_customer;

DELIMITER $$

CREATE PROCEDURE sp_lab_count_payments_by_customer(
    IN p_customer_number INT,
    OUT p_payment_count INT
)
BEGIN
    SELECT COUNT(*)
    INTO p_payment_count
    FROM payments
    WHERE customerNumber = p_customer_number;
END$$

DELIMITER ;

CALL sp_lab_count_payments_by_customer(103, @payment_count);

SELECT @payment_count AS totalPayments;
```

### Bài 8.4

```sql
DROP PROCEDURE IF EXISTS sp_lab_apply_discount;

DELIMITER $$

CREATE PROCEDURE sp_lab_apply_discount(
    INOUT p_price DECIMAL(10,2),
    IN p_discount_percent DECIMAL(5,2)
)
BEGIN
    SET p_price = p_price * (1 - p_discount_percent / 100);
END$$

DELIMITER ;

SET @price = 200.00;

CALL sp_lab_apply_discount(@price, 15.00);

SELECT @price AS discountedPrice;
```

### Bài 10.3

```sql
SELECT ROUTINE_NAME,
       CREATED,
       LAST_ALTERED
FROM INFORMATION_SCHEMA.ROUTINES
WHERE ROUTINE_SCHEMA = 'classicmodels'
  AND ROUTINE_TYPE = 'PROCEDURE'
  AND ROUTINE_NAME LIKE 'sp_lab%'
ORDER BY ROUTINE_NAME;
```

### Bài 13.1

```sql
DROP PROCEDURE IF EXISTS sp_lab_list_high_credit_customers;

DELIMITER $$

CREATE PROCEDURE sp_lab_list_high_credit_customers()
BEGIN
    SELECT customerNumber, customerName, country, creditLimit
    FROM customers
    WHERE creditLimit >= 100000
    ORDER BY creditLimit DESC, customerName;
END$$

DELIMITER ;

CALL sp_lab_list_high_credit_customers();
```

### Bài 13.2

```sql
DROP PROCEDURE IF EXISTS sp_lab_list_orders_by_customer;

DELIMITER $$

CREATE PROCEDURE sp_lab_list_orders_by_customer(
    IN p_customer_number INT
)
BEGIN
    SELECT orderNumber, orderDate, requiredDate, status
    FROM orders
    WHERE customerNumber = p_customer_number
    ORDER BY orderDate DESC, orderNumber DESC;
END$$

DELIMITER ;

CALL sp_lab_list_orders_by_customer(103);
```

### Bài 13.3

```sql
DROP PROCEDURE IF EXISTS sp_lab_count_customer_payments;

DELIMITER $$

CREATE PROCEDURE sp_lab_count_customer_payments(
    IN p_customer_number INT,
    OUT p_payment_count INT
)
BEGIN
    SELECT COUNT(*)
    INTO p_payment_count
    FROM payments
    WHERE customerNumber = p_customer_number;
END$$

DELIMITER ;

CALL sp_lab_count_customer_payments(103, @payment_count);

SELECT @payment_count AS totalPayments;
```

### Bài 13.4

```sql
DROP PROCEDURE IF EXISTS sp_lab_product_statistics;

DELIMITER $$

CREATE PROCEDURE sp_lab_product_statistics()
BEGIN
    DECLARE v_total_products INT DEFAULT 0;
    DECLARE v_average_msrp DECIMAL(10,2) DEFAULT 0.00;

    SELECT COUNT(*), COALESCE(AVG(MSRP), 0)
    INTO v_total_products, v_average_msrp
    FROM products;

    SELECT v_total_products AS totalProducts,
           v_average_msrp AS averageMSRP;
END$$

DELIMITER ;

CALL sp_lab_product_statistics();
```

---

## 15. Tóm tắt

Các kiến thức chính trong lab:

- Stored procedure là chương trình SQL được lưu trong database và gọi bằng `CALL`.
- Stored procedure có thể nhận tham số, trả result set và trả giá trị qua `OUT` hoặc `INOUT`.
- `DELIMITER` giúp client phân biệt dấu `;` bên trong procedure với dấu kết thúc của câu `CREATE PROCEDURE`.
- `CREATE PROCEDURE` tạo procedure; `DROP PROCEDURE` xóa procedure.
- Biến cục bộ được khai báo bằng `DECLARE` và chỉ tồn tại khi procedure chạy.
- `DECLARE` phải nằm ở đầu block `BEGIN ... END`, trước các câu lệnh xử lý.
- `IN` nhận đầu vào; `OUT` trả đầu ra; `INOUT` vừa nhận vừa trả dữ liệu.
- Khi gọi procedure có `OUT` hoặc `INOUT`, session variable như `@order_count` thường được dùng để nhận kết quả.
- Muốn thay đổi thân procedure, thông thường cần `DROP PROCEDURE` rồi `CREATE PROCEDURE` lại.
- Có thể liệt kê procedure bằng `SHOW PROCEDURE STATUS` hoặc `INFORMATION_SCHEMA.ROUTINES`.
- Dùng `SHOW CREATE PROCEDURE` để xem và sao lưu definition trước khi sửa hoặc xóa procedure.

---

## 16. Từ khóa chính

- Stored Procedure
- CREATE PROCEDURE
- CALL
- DROP PROCEDURE
- DELIMITER
- BEGIN
- END
- DECLARE
- Local Variable
- Session Variable
- IN Parameter
- OUT Parameter
- INOUT Parameter
- SELECT ... INTO
- SHOW PROCEDURE STATUS
- SHOW CREATE PROCEDURE
- INFORMATION_SCHEMA.ROUTINES
- classicmodels
