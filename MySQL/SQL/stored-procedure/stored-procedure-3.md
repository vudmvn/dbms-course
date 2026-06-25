---
title: "Lab: Cursors và Prepared Statements trong MySQL Stored Procedures với classicmodels"
author: "Tên giảng viên"
duration: "180m"
difficulty: "Intermediate–Advanced"
prerequisites:
  - "Đã hoàn thành lab Stored Procedures cơ bản và lab Conditional Statements & Loops"
  - "Đã biết CREATE PROCEDURE, DELIMITER, biến cục bộ, tham số, IF, CASE, LOOP, WHILE, REPEAT và LEAVE"
  - "Đã biết SELECT, JOIN, GROUP BY, ORDER BY, LIMIT và các hàm tổng hợp cơ bản"
  - "Có quyền CREATE ROUTINE và EXECUTE trên cơ sở dữ liệu thực hành"
summary: "Thực hành cursor để xử lý từng dòng trong result set và prepared statement để thực thi truy vấn tham số hóa hoặc dynamic SQL một cách có kiểm soát trong MySQL."
---

# Lab: Cursors và Prepared Statements trong MySQL Stored Procedures với `classicmodels`

## Link tham khảo

- [MySQL Tutorial — Stored Procedures](https://www.mysqltutorial.org/mysql-stored-procedure/)
- [MySQL Tutorial — Cursors](https://www.mysqltutorial.org/mysql-stored-procedure/sql-cursor-in-stored-procedures/)
- [MySQL Tutorial — Prepared Statements](https://www.mysqltutorial.org/mysql-stored-procedure/mysql-prepared-statement/)

## 1. Mục tiêu học tập

Sau khi hoàn thành lab này, người học có thể:

1. Giải thích được cursor là gì và khi nào nên hoặc không nên dùng cursor.
2. Thực hiện đúng vòng đời của cursor: `DECLARE` → `OPEN` → `FETCH` → `CLOSE`.
3. Khai báo biến, cursor và handler theo đúng thứ tự trong block `BEGIN ... END`.
4. Dùng `DECLARE CONTINUE HANDLER FOR NOT FOUND` để nhận biết khi cursor đã hết dòng.
5. Dùng `LOOP` và `LEAVE` để xử lý từng dòng do cursor trả về.
6. Viết procedure dùng cursor để tính toán hoặc tổng hợp dữ liệu theo từng dòng.
7. Giải thích prepared statement là gì và lợi ích của placeholder `?`.
8. Dùng `PREPARE`, `EXECUTE ... USING` và `DEALLOCATE PREPARE`.
9. Phân biệt dữ liệu có thể bind bằng `?` với identifier phải được kiểm soát bằng whitelist.
10. Dùng prepared statement trong stored procedure một cách an toàn và giải phóng statement sau khi dùng.

---

## 2. Chuẩn bị môi trường

Chọn cơ sở dữ liệu:

```sql
USE classicmodels;
```

Kiểm tra một số bảng dùng trong lab:

```sql
SELECT customerNumber, customerName, country, creditLimit
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
SELECT orderNumber, productCode, quantityOrdered, priceEach, orderLineNumber
FROM orderdetails
WHERE orderNumber = 10100
ORDER BY orderLineNumber;
```

```sql
SELECT productCode, productName, productLine, buyPrice, MSRP
FROM products
ORDER BY productCode
LIMIT 10;
```

Trong lab này, procedure được đặt tên bằng tiền tố:

```text
sp_lab_cursor_
sp_lab_prepared_
```

Ví dụ:

```text
sp_lab_cursor_order_total
sp_lab_cursor_product_margin
sp_lab_prepared_customers_by_country
sp_lab_prepared_safe_list
```

Trước khi tạo lại procedure, dùng:

```sql
DROP PROCEDURE IF EXISTS procedure_name;
```

### Bài tập thực hành

**Bài 2.1.**

Viết lệnh chọn cơ sở dữ liệu `classicmodels`.

**Bài 2.2.**

Viết truy vấn xem các dòng `orderdetails` của đơn hàng `10100`.

**Bài 2.3.**

Viết truy vấn xem mã, tên và giá bán đề xuất của sản phẩm thuộc dòng `Motorcycles`.

**Bài 2.4.**

Viết truy vấn xem các đơn hàng của khách hàng `103`.

**Bài 2.5.**

Nêu lý do nên dùng tiền tố `sp_lab_cursor_` và `sp_lab_prepared_` khi tạo procedure thực hành.

---

# Phần A. Cursors

## 3. Cursor là gì?

### 3.1. Khái niệm

Một **cursor** là cơ chế cho phép stored program duyệt result set của một câu `SELECT` **từng dòng một**.

Thông thường, SQL làm việc theo tư duy **set-based**: một câu lệnh xử lý một tập dòng cùng lúc.

Ví dụ, để tính tổng giá trị của toàn bộ dòng chi tiết trong đơn hàng `10100`, cách set-based rất ngắn:

```sql
SELECT COALESCE(SUM(quantityOrdered * priceEach), 0) AS totalOrderValue
FROM orderdetails
WHERE orderNumber = 10100;
```

Cursor cho phép thực hiện cùng kiểu bài toán theo từng dòng:

1. Lấy dòng thứ nhất.
2. Xử lý dòng đó.
3. Lấy dòng tiếp theo.
4. Lặp lại đến khi hết result set.

### 3.2. Đặc điểm của cursor trong MySQL

Cursor trong MySQL stored programs có các đặc điểm:

| Đặc điểm | Ý nghĩa |
|---|---|
| Read-only | Không cập nhật trực tiếp dòng hiện tại qua cursor |
| Nonscrollable | Chỉ duyệt theo một hướng từ đầu đến cuối; không nhảy tới dòng tùy ý |
| Asensitive | MySQL có thể dùng kết quả trung gian hoặc dữ liệu gốc; ứng dụng không nên giả định một cơ chế lưu trữ cụ thể |
| Gắn với `SELECT` | Cursor được khai báo cho một câu truy vấn `SELECT` |
| Dùng trong stored program | Thường được dùng trong procedure, function, trigger hoặc event theo các ràng buộc tương ứng |

### 3.3. Khi nào nên dùng cursor?

Cursor phù hợp khi logic thực sự cần xử lý tuần tự từng dòng, ví dụ:

- Áp dụng quy tắc nghiệp vụ phụ thuộc vào kết quả của dòng trước.
- Tạo báo cáo có tính toán tuần tự phức tạp.
- Duyệt từng dòng để gọi procedure hoặc xử lý nhiều bước.
- Học cơ chế xử lý theo từng record trong stored program.

### 3.4. Khi nào không nên dùng cursor?

Không nên dùng cursor khi một câu SQL set-based có thể giải quyết trực tiếp.

Ví dụ, thay vì dùng cursor để đếm số sản phẩm trong từng product line, thường nên dùng:

```sql
SELECT productLine, COUNT(*) AS totalProducts
FROM products
GROUP BY productLine;
```

Cursor thường dài hơn, phức tạp hơn và có thể kém hiệu quả hơn truy vấn set-based khi dữ liệu lớn.

> Quy tắc thực hành: **Ưu tiên `SELECT`, `JOIN`, `GROUP BY`, `UPDATE ... JOIN`, hoặc câu SQL set-based trước. Chỉ dùng cursor khi cần xử lý row-by-row có lý do rõ ràng.**

### Bài tập thực hành

**Bài 3.1.**

Giải thích cursor là gì bằng một hoặc hai câu.

**Bài 3.2.**

Nêu hai đặc điểm của cursor MySQL.

**Bài 3.3.**

Nêu một tình huống phù hợp để dùng cursor.

**Bài 3.4.**

Viết một truy vấn set-based tính tổng giá trị đơn hàng `10100` mà không dùng cursor.

**Bài 3.5.**

Giải thích vì sao không nên dùng cursor chỉ để tính `COUNT(*)` hoặc `SUM(...)` nếu một câu SQL tổng hợp đã đủ.

---

## 4. Vòng đời của cursor: DECLARE, OPEN, FETCH, CLOSE

Một cursor thường được dùng theo bốn bước:

```text
1. DECLARE cursor
2. OPEN cursor
3. FETCH từng dòng từ cursor
4. CLOSE cursor
```

### 4.1. Khai báo cursor

Cú pháp:

```sql
DECLARE cursor_name CURSOR FOR
    SELECT_statement;
```

Ví dụ:

```sql
DECLARE cur_order_details CURSOR FOR
    SELECT productCode, quantityOrdered, priceEach
    FROM orderdetails
    WHERE orderNumber = p_order_number
    ORDER BY orderLineNumber;
```

Cursor chỉ được **khai báo** ở bước này; query chưa bắt đầu được duyệt.

### 4.2. Mở cursor

```sql
OPEN cur_order_details;
```

Sau khi `OPEN`, cursor sẵn sàng cung cấp các dòng.

### 4.3. Lấy một dòng bằng FETCH

```sql
FETCH cur_order_details
INTO v_product_code, v_quantity_ordered, v_price_each;
```

Số lượng biến trong `INTO` phải đúng bằng số cột trong câu `SELECT` của cursor.

### 4.4. Đóng cursor

```sql
CLOSE cur_order_details;
```

Sau khi dùng xong, nên đóng cursor rõ ràng.

Nếu cursor không được đóng tường minh, MySQL sẽ đóng khi block `BEGIN ... END` kết thúc. Tuy vậy, trong procedure thực hành và production code, nên gọi `CLOSE` để vòng đời tài nguyên rõ ràng.

### 4.5. Khung cursor đầy đủ

```sql
BEGIN
    DECLARE v_done BOOLEAN DEFAULT FALSE;
    DECLARE v_value INT;

    DECLARE cur_example CURSOR FOR
        SELECT someColumn
        FROM someTable;

    DECLARE CONTINUE HANDLER FOR NOT FOUND
        SET v_done = TRUE;

    OPEN cur_example;

    read_loop: LOOP
        FETCH cur_example INTO v_value;

        IF v_done THEN
            LEAVE read_loop;
        END IF;

        -- Xử lý v_value tại đây
    END LOOP read_loop;

    CLOSE cur_example;
END
```

### Bài tập thực hành

**Bài 4.1.**

Viết cú pháp khai báo cursor `cur_products` lấy `productCode`, `productName`, `MSRP` từ bảng `products`.

**Bài 4.2.**

Viết lệnh mở cursor `cur_products`.

**Bài 4.3.**

Viết lệnh `FETCH` từ `cur_products` vào ba biến `v_product_code`, `v_product_name`, `v_msrp`.

**Bài 4.4.**

Viết lệnh đóng cursor `cur_products`.

**Bài 4.5.**

Sắp xếp đúng thứ tự bốn thao tác: `FETCH`, `CLOSE`, `DECLARE`, `OPEN`.

---

## 5. Thứ tự khai báo biến, cursor và handler

Trong một block `BEGIN ... END`, MySQL yêu cầu các khai báo có thứ tự rõ ràng:

```text
1. Local variables và conditions
2. Cursors
3. Handlers
4. Các câu lệnh xử lý
```

Mẫu đúng:

```sql
BEGIN
    DECLARE v_done BOOLEAN DEFAULT FALSE;
    DECLARE v_product_code VARCHAR(15);

    DECLARE cur_products CURSOR FOR
        SELECT productCode
        FROM products;

    DECLARE CONTINUE HANDLER FOR NOT FOUND
        SET v_done = TRUE;

    -- OPEN, FETCH, LOOP, CLOSE ...
END
```

### 5.1. Sai: khai báo cursor sau handler

```sql
BEGIN
    DECLARE v_done BOOLEAN DEFAULT FALSE;

    DECLARE CONTINUE HANDLER FOR NOT FOUND
        SET v_done = TRUE;

    DECLARE cur_products CURSOR FOR
        SELECT productCode
        FROM products;
END
```

Cursor phải được khai báo **trước** handler.

### 5.2. Sai: khai báo biến sau cursor

```sql
BEGIN
    DECLARE cur_products CURSOR FOR
        SELECT productCode
        FROM products;

    DECLARE v_product_code VARCHAR(15);
END
```

Biến phải được khai báo trước cursor.

### 5.3. Vì sao handler `NOT FOUND` cần thiết?

Khi `FETCH` cố lấy dòng sau dòng cuối cùng của cursor, MySQL phát sinh condition `NOT FOUND` với SQLSTATE `'02000'`.

Ta thường khai báo handler:

```sql
DECLARE CONTINUE HANDLER FOR NOT FOUND
    SET v_done = TRUE;
```

Sau `FETCH`, procedure kiểm tra `v_done`:

```sql
IF v_done THEN
    LEAVE read_loop;
END IF;
```

### 5.4. Lưu ý quan trọng: `NOT FOUND` không chỉ xuất hiện với FETCH

`NOT FOUND` cũng có thể xảy ra khi `SELECT ... INTO` không trả về dòng nào.

Vì vậy, sau khi khai báo `NOT FOUND` handler cho cursor:

- Không nên dùng `SELECT ... INTO` có khả năng không trả về dòng trong cùng block mà không kiểm soát kỹ.
- Hoặc dùng một inner block có handler riêng.
- Hoặc dùng aggregate như `COUNT(*)`, `MAX()`, `COALESCE()` khi phù hợp, vì aggregate thường trả về một dòng.

### Bài tập thực hành

**Bài 5.1.**

Viết đúng thứ tự khai báo: biến `v_done`, cursor `cur_orders`, handler `NOT FOUND`.

**Bài 5.2.**

Sửa đoạn code đặt handler trước cursor.

**Bài 5.3.**

Sửa đoạn code đặt biến `v_order_number` sau cursor.

**Bài 5.4.**

Viết handler `CONTINUE` đặt `v_done = TRUE` khi hết dữ liệu cursor.

**Bài 5.5.**

Giải thích vì sao không nên đặt `SELECT ... INTO` có thể không trả về dòng trong cùng block mà handler `NOT FOUND` đang được dùng để kiểm soát cursor.

---

## 6. Ví dụ 1: Dùng cursor tính tổng giá trị đơn hàng

Ví dụ dưới đây duyệt từng dòng `orderdetails` của một đơn hàng, sau đó tính:

- Số dòng chi tiết.
- Tổng số lượng sản phẩm đặt.
- Tổng giá trị đơn hàng.

> Với bài toán này, `SUM()` set-based thường ngắn hơn. Cursor được dùng ở đây để minh họa đúng quy trình khai báo, mở, fetch, kiểm tra hết dữ liệu và đóng cursor.

```sql
DROP PROCEDURE IF EXISTS sp_lab_cursor_order_total;

DELIMITER $$

CREATE PROCEDURE sp_lab_cursor_order_total(
    IN p_order_number INT
)
main_block: BEGIN
    DECLARE v_done BOOLEAN DEFAULT FALSE;
    DECLARE v_order_exists INT DEFAULT 0;

    DECLARE v_product_code VARCHAR(15);
    DECLARE v_quantity_ordered INT;
    DECLARE v_price_each DECIMAL(10,2);

    DECLARE v_total_lines INT DEFAULT 0;
    DECLARE v_total_quantity INT DEFAULT 0;
    DECLARE v_total_value DECIMAL(14,2) DEFAULT 0.00;

    DECLARE cur_order_details CURSOR FOR
        SELECT productCode, quantityOrdered, priceEach
        FROM orderdetails
        WHERE orderNumber = p_order_number
        ORDER BY orderLineNumber;

    DECLARE CONTINUE HANDLER FOR NOT FOUND
        SET v_done = TRUE;

    SELECT COUNT(*)
    INTO v_order_exists
    FROM orders
    WHERE orderNumber = p_order_number;

    IF v_order_exists = 0 THEN
        SELECT p_order_number AS orderNumber,
               'Order not found' AS message;
        LEAVE main_block;
    END IF;

    OPEN cur_order_details;

    read_loop: LOOP
        FETCH cur_order_details
        INTO v_product_code, v_quantity_ordered, v_price_each;

        IF v_done THEN
            LEAVE read_loop;
        END IF;

        SET v_total_lines = v_total_lines + 1;
        SET v_total_quantity = v_total_quantity + v_quantity_ordered;
        SET v_total_value =
            v_total_value + v_quantity_ordered * v_price_each;
    END LOOP read_loop;

    CLOSE cur_order_details;

    SELECT p_order_number AS orderNumber,
           v_total_lines AS totalLines,
           v_total_quantity AS totalQuantity,
           v_total_value AS totalOrderValue;
END$$

DELIMITER ;
```

Gọi procedure:

```sql
CALL sp_lab_cursor_order_total(10100);
```

### 6.1. Diễn giải

| Bước | Nội dung |
|---|---|
| Khai báo biến | Lưu từng dòng fetch và các biến tổng hợp |
| Khai báo cursor | Chọn các `orderdetails` của một order |
| Khai báo handler | Đánh dấu `v_done = TRUE` khi hết dòng |
| Kiểm tra order | Tránh trả kết quả gây nhầm cho order không tồn tại |
| `OPEN` | Mở cursor |
| `FETCH` | Lấy từng dòng vào các biến |
| `IF v_done` | Rời loop ngay khi fetch đã hết dữ liệu |
| Tính toán | Cộng dồn số dòng, số lượng và giá trị |
| `CLOSE` | Đóng cursor |
| `SELECT` | Trả kết quả cuối |

### Bài tập thực hành

**Bài 6.1.**

Tạo procedure `sp_lab_cursor_order_total` theo ví dụ.

**Bài 6.2.**

Gọi procedure cho đơn hàng `10100`.

**Bài 6.3.**

Gọi procedure cho đơn hàng `10101`.

**Bài 6.4.**

Gọi procedure cho mã đơn hàng không tồn tại, ví dụ `999999`.

**Bài 6.5.**

Viết truy vấn set-based dùng `COUNT(*)`, `SUM(quantityOrdered)` và `SUM(quantityOrdered * priceEach)` để kiểm tra kết quả của procedure.

---

## 7. Ví dụ 2: Dùng cursor tính thống kê lợi nhuận sản phẩm

Procedure sau duyệt từng sản phẩm trong một `productLine` và tính:

- Số sản phẩm.
- Tổng biên lợi nhuận ước tính: `MSRP - buyPrice`.
- Biên lợi nhuận lớn nhất.
- Số sản phẩm có `MSRP >= 100.00`.

```sql
DROP PROCEDURE IF EXISTS sp_lab_cursor_product_margin;

DELIMITER $$

CREATE PROCEDURE sp_lab_cursor_product_margin(
    IN p_product_line VARCHAR(50)
)
main_block: BEGIN
    DECLARE v_done BOOLEAN DEFAULT FALSE;
    DECLARE v_product_count_in_line INT DEFAULT 0;

    DECLARE v_product_code VARCHAR(15);
    DECLARE v_buy_price DECIMAL(10,2);
    DECLARE v_msrp DECIMAL(10,2);
    DECLARE v_margin DECIMAL(10,2);

    DECLARE v_total_products INT DEFAULT 0;
    DECLARE v_total_margin DECIMAL(14,2) DEFAULT 0.00;
    DECLARE v_max_margin DECIMAL(10,2) DEFAULT 0.00;
    DECLARE v_products_msrp_100 INT DEFAULT 0;

    DECLARE cur_products CURSOR FOR
        SELECT productCode, buyPrice, MSRP
        FROM products
        WHERE productLine = p_product_line
        ORDER BY productCode;

    DECLARE CONTINUE HANDLER FOR NOT FOUND
        SET v_done = TRUE;

    SELECT COUNT(*)
    INTO v_product_count_in_line
    FROM products
    WHERE productLine = p_product_line;

    IF v_product_count_in_line = 0 THEN
        SELECT p_product_line AS productLine,
               'No products found for this product line' AS message;
        LEAVE main_block;
    END IF;

    OPEN cur_products;

    product_loop: LOOP
        FETCH cur_products
        INTO v_product_code, v_buy_price, v_msrp;

        IF v_done THEN
            LEAVE product_loop;
        END IF;

        SET v_margin = v_msrp - v_buy_price;
        SET v_total_products = v_total_products + 1;
        SET v_total_margin = v_total_margin + v_margin;

        IF v_margin > v_max_margin THEN
            SET v_max_margin = v_margin;
        END IF;

        IF v_msrp >= 100.00 THEN
            SET v_products_msrp_100 = v_products_msrp_100 + 1;
        END IF;
    END LOOP product_loop;

    CLOSE cur_products;

    SELECT p_product_line AS productLine,
           v_total_products AS totalProducts,
           v_total_margin AS totalEstimatedMargin,
           v_max_margin AS maximumEstimatedMargin,
           v_products_msrp_100 AS productsWithMSRPAtLeast100;
END$$

DELIMITER ;
```

Gọi procedure:

```sql
CALL sp_lab_cursor_product_margin('Motorcycles');
```

### Bài tập thực hành

**Bài 7.1.**

Tạo procedure `sp_lab_cursor_product_margin` theo ví dụ.

**Bài 7.2.**

Gọi procedure cho `Motorcycles`.

**Bài 7.3.**

Gọi procedure cho `Classic Cars`.

**Bài 7.4.**

Sửa procedure để đếm thêm số sản phẩm có `quantityInStock < 1000`.

**Bài 7.5.**

Viết truy vấn set-based tương đương để tính `COUNT(*)`, `SUM(MSRP - buyPrice)`, `MAX(MSRP - buyPrice)` cho một product line.

---

## 8. Cursor và lỗi thường gặp

### 8.1. Lỗi: FETCH có số biến không đúng

Cursor trả về ba cột:

```sql
DECLARE cur_products CURSOR FOR
    SELECT productCode, productName, MSRP
    FROM products;
```

Sai:

```sql
FETCH cur_products INTO v_product_code, v_product_name;
```

Đúng:

```sql
FETCH cur_products
INTO v_product_code, v_product_name, v_msrp;
```

### 8.2. Lỗi: xử lý dữ liệu sau khi hết cursor

Sai:

```sql
FETCH cur_products
INTO v_product_code, v_product_name, v_msrp;

SET v_count = v_count + 1;

IF v_done THEN
    LEAVE product_loop;
END IF;
```

Khi `FETCH` không còn dòng, procedure vẫn tăng `v_count` một lần trước khi kiểm tra `v_done`.

Đúng:

```sql
FETCH cur_products
INTO v_product_code, v_product_name, v_msrp;

IF v_done THEN
    LEAVE product_loop;
END IF;

SET v_count = v_count + 1;
```

### 8.3. Lỗi: quên đóng cursor

Nên luôn có:

```sql
CLOSE cur_products;
```

trước khi procedure kết thúc.

### 8.4. Lỗi: không có handler `NOT FOUND`

Nếu không có handler phù hợp, `FETCH` cuối có thể phát sinh condition không được xử lý theo cách mong muốn.

### 8.5. Lỗi: dùng cursor khi SQL set-based đơn giản hơn

Ví dụ, việc tính tổng giá trị đơn hàng có thể viết bằng `SUM()` mà không cần cursor. Cursor chỉ nên được chọn khi cần xử lý tuần tự từng dòng.

### Bài tập thực hành

**Bài 8.1.**

Sửa đoạn `FETCH` thiếu một biến nhận dữ liệu.

**Bài 8.2.**

Sửa vòng lặp xử lý dữ liệu trước khi kiểm tra `v_done`.

**Bài 8.3.**

Bổ sung `CLOSE cursor_name` vào một procedure thiếu thao tác đóng cursor.

**Bài 8.4.**

Bổ sung `DECLARE CONTINUE HANDLER FOR NOT FOUND` vào procedure cursor chưa có handler.

**Bài 8.5.**

Cho bài toán tính tổng giá trị mỗi đơn hàng. Nêu lý do `GROUP BY` thường phù hợp hơn cursor.

---

# Phần B. Prepared Statements

## 9. Prepared statement là gì?

### 9.1. Khái niệm

Một **prepared statement** tách quá trình thực thi SQL thành ba bước:

```text
1. PREPARE: Chuẩn bị câu SQL
2. EXECUTE: Thực thi câu SQL, có thể gắn các giá trị tham số
3. DEALLOCATE PREPARE: Giải phóng prepared statement
```

Cú pháp tổng quát:

```sql
PREPARE statement_name
FROM preparable_statement;

EXECUTE statement_name
USING user_variable_list;

DEALLOCATE PREPARE statement_name;
```

Ví dụ:

```sql
SET @sql_text =
    'SELECT customerNumber, customerName, country
     FROM customers
     WHERE country = ?';

PREPARE stmt_customers_by_country
FROM @sql_text;

SET @country = 'USA';

EXECUTE stmt_customers_by_country
USING @country;

DEALLOCATE PREPARE stmt_customers_by_country;
```

### 9.2. Placeholder `?`

Dấu `?` là placeholder cho **giá trị dữ liệu**.

Ví dụ hợp lệ:

```sql
SELECT customerNumber, customerName
FROM customers
WHERE country = ?;
```

```sql
SELECT productCode, productName, MSRP
FROM products
WHERE MSRP >= ?;
```

Giá trị được truyền vào khi `EXECUTE`.

### 9.3. Vì sao prepared statement có ích?

| Lợi ích | Diễn giải |
|---|---|
| Tách dữ liệu khỏi cú pháp SQL | Giá trị được bind thay vì nối trực tiếp vào chuỗi SQL |
| Hỗ trợ an toàn hơn khi giá trị đến từ input | Giảm rủi ro SQL injection đối với phần dữ liệu được bind đúng cách |
| Có thể tái thực thi | Một statement đã prepare có thể execute nhiều lần với các giá trị khác nhau trong cùng session |
| Hỗ trợ dynamic SQL có kiểm soát | Có thể xây dựng câu SQL tại runtime, nhưng cần kiểm soát chặt chẽ phần được nối chuỗi |

> Prepared statement không tự động làm mọi dynamic SQL an toàn. Nếu tự nối tên bảng, tên cột hoặc mệnh đề SQL từ input không đáng tin, vẫn có nguy cơ SQL injection. Identifier cần whitelist hoặc được chọn từ tập giá trị hợp lệ.

### 9.4. Prepared statement trong stored procedure

MySQL cho phép `PREPARE`, `EXECUTE`, `DEALLOCATE PREPARE` trong **stored procedure**.

Các statement này không được dùng trong stored function hoặc trigger.

Khi dùng prepared statement trong procedure:

- Dùng session variable, ví dụ `@sql_text`, để chứa câu SQL.
- Dùng session variable, ví dụ `@country_param`, để bind giá trị khi `EXECUTE ... USING`.
- Gọi `DEALLOCATE PREPARE` sau khi dùng xong.

### Bài tập thực hành

**Bài 9.1.**

Nêu ba bước trong vòng đời của prepared statement.

**Bài 9.2.**

Dấu `?` trong prepared statement dùng để thay thế cho dữ liệu hay tên bảng?

**Bài 9.3.**

Nêu một lợi ích bảo mật của bind parameter.

**Bài 9.4.**

Giải thích vì sao prepared statement không tự bảo vệ nếu tên bảng được nối trực tiếp từ input không được kiểm tra.

**Bài 9.5.**

Nêu lý do cần dùng `DEALLOCATE PREPARE`.

---

## 10. Prepared statement cơ bản

### 10.1. Chuẩn bị và thực thi một truy vấn có tham số

Ví dụ tìm sản phẩm có MSRP không nhỏ hơn một mức giá.

```sql
SET @sql_text =
    'SELECT productCode, productName, productLine, MSRP
     FROM products
     WHERE MSRP >= ?
     ORDER BY MSRP DESC, productName';

PREPARE stmt_products_by_min_msrp
FROM @sql_text;

SET @min_msrp = 100.00;

EXECUTE stmt_products_by_min_msrp
USING @min_msrp;

DEALLOCATE PREPARE stmt_products_by_min_msrp;
```

### 10.2. Execute nhiều lần

Một prepared statement có thể thực thi nhiều lần với các giá trị khác nhau trước khi deallocate.

```sql
SET @sql_text =
    'SELECT customerNumber, customerName, city, country
     FROM customers
     WHERE country = ?
     ORDER BY customerName';

PREPARE stmt_customers_by_country
FROM @sql_text;

SET @country = 'USA';
EXECUTE stmt_customers_by_country
USING @country;

SET @country = 'France';
EXECUTE stmt_customers_by_country
USING @country;

SET @country = 'Germany';
EXECUTE stmt_customers_by_country
USING @country;

DEALLOCATE PREPARE stmt_customers_by_country;
```

### 10.3. Số placeholder và số biến `USING`

Số biến sau `USING` phải bằng số placeholder `?`.

Ví dụ:

```sql
SET @sql_text =
    'SELECT productCode, productName, MSRP
     FROM products
     WHERE productLine = ?
       AND MSRP >= ?';

PREPARE stmt_products_by_line_and_price
FROM @sql_text;

SET @product_line = 'Classic Cars';
SET @min_msrp = 100.00;

EXECUTE stmt_products_by_line_and_price
USING @product_line, @min_msrp;

DEALLOCATE PREPARE stmt_products_by_line_and_price;
```

### Bài tập thực hành

**Bài 10.1.**

Tạo prepared statement hiển thị khách hàng theo `country`, sau đó thực thi với giá trị `USA`.

**Bài 10.2.**

Dùng cùng prepared statement ở Bài 10.1 để thực thi tiếp với `France` và `Germany`.

**Bài 10.3.**

Tạo prepared statement hiển thị đơn hàng theo `status`.

**Bài 10.4.**

Tạo prepared statement hiển thị sản phẩm theo `productLine` và `MSRP` tối thiểu.

**Bài 10.5.**

Viết prepared statement có hai placeholder, sau đó bind đúng hai session variable trong `EXECUTE ... USING`.

---

## 11. Prepared statement trong stored procedure

Ví dụ procedure nhận quốc gia làm tham số, sau đó gọi prepared statement để truy vấn khách hàng.

```sql
DROP PROCEDURE IF EXISTS sp_lab_prepared_customers_by_country;

DELIMITER $$

CREATE PROCEDURE sp_lab_prepared_customers_by_country(
    IN p_country VARCHAR(50)
)
BEGIN
    SET @sql_text =
        'SELECT customerNumber, customerName, city, country, creditLimit
         FROM customers
         WHERE country = ?
         ORDER BY customerName';

    SET @country_param = p_country;

    PREPARE stmt_customers_by_country
    FROM @sql_text;

    EXECUTE stmt_customers_by_country
    USING @country_param;

    DEALLOCATE PREPARE stmt_customers_by_country;
END$$

DELIMITER ;
```

Gọi procedure:

```sql
CALL sp_lab_prepared_customers_by_country('USA');
```

### 11.1. Vì sao dùng session variable?

Trong SQL prepared statement của MySQL, local variable của stored procedure không được tham chiếu trực tiếp bên trong prepared statement đã được tạo.

Do đó, trong ví dụ:

```sql
SET @country_param = p_country;
```

rồi:

```sql
EXECUTE stmt_customers_by_country
USING @country_param;
```

`p_country` là procedure parameter; `@country_param` là session variable dùng để bind vào `?`.

### 11.2. Ví dụ: procedure lọc sản phẩm theo dòng và giá

```sql
DROP PROCEDURE IF EXISTS sp_lab_prepared_products_by_line_and_price;

DELIMITER $$

CREATE PROCEDURE sp_lab_prepared_products_by_line_and_price(
    IN p_product_line VARCHAR(50),
    IN p_min_msrp DECIMAL(10,2)
)
BEGIN
    SET @sql_text =
        'SELECT productCode, productName, productLine, buyPrice, MSRP
         FROM products
         WHERE productLine = ?
           AND MSRP >= ?
         ORDER BY MSRP DESC, productName';

    SET @product_line_param = p_product_line;
    SET @min_msrp_param = p_min_msrp;

    PREPARE stmt_products_by_line_and_price
    FROM @sql_text;

    EXECUTE stmt_products_by_line_and_price
    USING @product_line_param, @min_msrp_param;

    DEALLOCATE PREPARE stmt_products_by_line_and_price;
END$$

DELIMITER ;
```

Gọi procedure:

```sql
CALL sp_lab_prepared_products_by_line_and_price(
    'Motorcycles',
    80.00
);
```

### 11.3. Lưu ý về tên statement

Tên prepared statement như:

```text
stmt_customers_by_country
```

thuộc session hiện tại. Procedure nên deallocate ngay sau execute để:

- Giải phóng tài nguyên.
- Tránh xung đột khi lại prepare statement cùng tên trong cùng session.
- Tránh tiến gần giới hạn `max_prepared_stmt_count` khi nhiều statement không được giải phóng.

### Bài tập thực hành

**Bài 11.1.**

Tạo procedure `sp_lab_prepared_orders_by_status` có tham số `IN p_status VARCHAR(20)` và dùng prepared statement để hiển thị đơn hàng theo status.

**Bài 11.2.**

Tạo procedure `sp_lab_prepared_payments_by_customer` có tham số `IN p_customer_number INT` và dùng prepared statement để hiển thị các khoản thanh toán.

**Bài 11.3.**

Tạo procedure `sp_lab_prepared_products_by_price_range` có hai tham số `IN p_min_price DECIMAL(10,2)` và `IN p_max_price DECIMAL(10,2)`.

**Bài 11.4.**

Gọi procedure ở Bài 11.3 với khoảng giá từ `50.00` đến `100.00`.

**Bài 11.5.**

Giải thích vai trò của `SET @country_param = p_country` trong procedure prepared statement.

---

## 12. Dynamic SQL và whitelist identifier

### 12.1. Placeholder không thay thế được identifier

Dấu `?` chỉ bind được **data value**, không bind được:

- Tên bảng.
- Tên cột.
- Tên database.
- Từ khóa SQL như `ASC`, `DESC`.
- Toàn bộ mệnh đề SQL.

Sai về ý tưởng:

```sql
SET @sql_text = 'SELECT * FROM ?';
PREPARE stmt_bad FROM @sql_text;
```

Không thể dùng `?` để thay cho tên bảng.

### 12.2. Cách không an toàn

Không ghép trực tiếp input không được kiểm tra vào SQL:

```sql
SET @sql_text =
    CONCAT('SELECT * FROM ', @user_input);
```

Nếu `@user_input` đến từ nguồn không tin cậy, statement có thể bị thay đổi theo cách không mong muốn.

### 12.3. Cách an toàn hơn: whitelist

Chỉ cho phép một tập giá trị định trước.

Ví dụ procedure chỉ được phép chọn một trong hai nguồn dữ liệu:

- `customers`
- `products`

```sql
DROP PROCEDURE IF EXISTS sp_lab_prepared_safe_list;

DELIMITER $$

CREATE PROCEDURE sp_lab_prepared_safe_list(
    IN p_source VARCHAR(20)
)
main_block: BEGIN
    IF p_source = 'customers' THEN
        SET @sql_text =
            'SELECT customerNumber AS entityCode,
                    customerName AS entityName
             FROM customers
             ORDER BY customerNumber
             LIMIT 10';
    ELSEIF p_source = 'products' THEN
        SET @sql_text =
            'SELECT productCode AS entityCode,
                    productName AS entityName
             FROM products
             ORDER BY productCode
             LIMIT 10';
    ELSE
        SELECT 'Invalid source. Use customers or products.' AS message;
        LEAVE main_block;
    END IF;

    PREPARE stmt_safe_list
    FROM @sql_text;

    EXECUTE stmt_safe_list;

    DEALLOCATE PREPARE stmt_safe_list;
END$$

DELIMITER ;
```

Gọi procedure:

```sql
CALL sp_lab_prepared_safe_list('customers');
```

```sql
CALL sp_lab_prepared_safe_list('products');
```

```sql
CALL sp_lab_prepared_safe_list('orders');
```

### 12.4. Dynamic ORDER BY: ưu tiên cách không cần dynamic SQL

Nhiều trường hợp có thể dùng `CASE` thay vì nối trực tiếp tên cột vào `ORDER BY`.

Ví dụ đơn giản:

```sql
SELECT customerNumber, customerName, country, creditLimit
FROM customers
ORDER BY
    CASE WHEN 'NAME' = 'NAME' THEN customerName END ASC,
    CASE WHEN 'NAME' = 'CREDIT' THEN creditLimit END DESC;
```

Trong procedure, có thể thay chuỗi `'NAME'` bằng parameter và dùng `CASE` để giới hạn các lựa chọn hợp lệ.

Điều này thường an toàn hơn so với nối trực tiếp input vào chuỗi SQL.

### Bài tập thực hành

**Bài 12.1.**

Giải thích vì sao `SELECT * FROM ?` không thể dùng placeholder để thay tên bảng.

**Bài 12.2.**

Tạo procedure tương tự `sp_lab_prepared_safe_list`, chỉ chấp nhận `customers` hoặc `products`.

**Bài 12.3.**

Mở rộng procedure ở Bài 12.2 để chấp nhận thêm nguồn `offices`.

**Bài 12.4.**

Gọi procedure với giá trị không hợp lệ, ví dụ `orders`, và kiểm tra thông báo trả về.

**Bài 12.5.**

Giải thích vì sao whitelist an toàn hơn việc ghép trực tiếp `p_source` vào `CONCAT('SELECT ... FROM ', p_source)`.

---

## 13. Prepared statement và hiệu năng

Prepared statement có thể được execute nhiều lần trong cùng session với các giá trị khác nhau.

Ví dụ:

```sql
SET @sql_text =
    'SELECT customerNumber, customerName, country
     FROM customers
     WHERE country = ?';

PREPARE stmt_country
FROM @sql_text;

SET @country = 'USA';
EXECUTE stmt_country USING @country;

SET @country = 'France';
EXECUTE stmt_country USING @country;

SET @country = 'Germany';
EXECUTE stmt_country USING @country;

DEALLOCATE PREPARE stmt_country;
```

### 13.1. Khi prepared statement có thể hữu ích

Prepared statement thường phù hợp khi:

- Cùng cấu trúc SQL được chạy lặp lại nhiều lần.
- Chỉ giá trị input thay đổi.
- Ứng dụng hoặc procedure cần dynamic SQL có kiểm soát.
- Cần tách dữ liệu đầu vào khỏi phần cú pháp SQL.

### 13.2. Không nên khẳng định luôn nhanh hơn

Prepared statement không bảo đảm mọi truy vấn đều nhanh hơn.

Hiệu quả phụ thuộc vào:

- Số lần statement được thực thi lại.
- Chi phí parse/prepare so với chi phí truy xuất dữ liệu.
- Chỉ mục, kích thước bảng, kế hoạch thực thi.
- Driver hoặc API của ứng dụng.
- Môi trường server và cache.

Do đó, nên hiểu prepared statement chủ yếu là cơ chế **tham số hóa và tái sử dụng statement**; lợi ích hiệu năng cần được đo trong bối cảnh cụ thể.

### 13.3. Giải phóng prepared statement

Luôn deallocate sau khi dùng:

```sql
DEALLOCATE PREPARE stmt_country;
```

Nếu không deallocate, prepared statement tồn tại đến khi:

- Được deallocate tường minh.
- Hoặc session kết thúc.

Nhiều prepared statement không được giải phóng có thể dẫn đến lỗi khi đạt giới hạn `max_prepared_stmt_count`.

### Bài tập thực hành

**Bài 13.1.**

Chuẩn bị một statement tìm khách hàng theo quốc gia, sau đó execute ba lần với ba quốc gia khác nhau.

**Bài 13.2.**

Giải thích vì sao không nên deallocate statement giữa ba lần execute ở Bài 13.1.

**Bài 13.3.**

Viết lệnh deallocate statement `stmt_country`.

**Bài 13.4.**

Nêu hai yếu tố khiến prepared statement không chắc chắn luôn nhanh hơn một câu SQL thông thường.

**Bài 13.5.**

Giải thích rủi ro khi tạo nhiều prepared statement nhưng không deallocate.

---

## 14. So sánh cursor và prepared statement

| Tiêu chí | Cursor | Prepared Statement |
|---|---|---|
| Mục đích chính | Xử lý từng dòng trong result set | Chuẩn bị và thực thi câu SQL có thể tham số hóa |
| Cách làm việc | `DECLARE` → `OPEN` → `FETCH` → `CLOSE` | `PREPARE` → `EXECUTE` → `DEALLOCATE` |
| Có dùng result set? | Có, để duyệt từng row | Có thể có hoặc không |
| Tối ưu cho | Logic row-by-row | Tái sử dụng cùng statement với giá trị khác nhau |
| Bảo mật | Không trực tiếp giải quyết SQL injection | Placeholder bảo vệ phần dữ liệu được bind |
| Thay thế set-based SQL? | Thường không nên nếu có SQL set-based | Không thay thế SELECT/UPDATE thông thường; là cách thực thi tham số hóa |
| Dùng với dynamic identifier? | Không phải công cụ dành cho dynamic identifier | Cần whitelist, vì `?` không bind identifier |

### Bài tập thực hành

**Bài 14.1.**

Nêu một bài toán phù hợp với cursor.

**Bài 14.2.**

Nêu một bài toán phù hợp với prepared statement.

**Bài 14.3.**

Cho yêu cầu “tính tổng doanh thu theo country”. Nên dùng cursor hay `GROUP BY`? Giải thích.

**Bài 14.4.**

Cho yêu cầu “thực hiện cùng câu SELECT theo 10 country khác nhau”. Prepared statement có thể phù hợp không? Vì sao?

**Bài 14.5.**

Giải thích vì sao cursor và prepared statement phục vụ hai mục đích khác nhau.

---

## 15. Ví dụ tổng hợp

### 15.1. Procedure cursor: thống kê sản phẩm theo product line

Procedure dưới đây dùng cursor để tính tổng số sản phẩm và tổng giá trị MSRP của từng sản phẩm trong một product line.

```sql
DROP PROCEDURE IF EXISTS sp_lab_cursor_product_line_summary;

DELIMITER $$

CREATE PROCEDURE sp_lab_cursor_product_line_summary(
    IN p_product_line VARCHAR(50)
)
main_block: BEGIN
    DECLARE v_done BOOLEAN DEFAULT FALSE;
    DECLARE v_line_count INT DEFAULT 0;

    DECLARE v_product_code VARCHAR(15);
    DECLARE v_msrp DECIMAL(10,2);

    DECLARE v_product_count INT DEFAULT 0;
    DECLARE v_total_msrp DECIMAL(14,2) DEFAULT 0.00;
    DECLARE v_average_msrp DECIMAL(10,2) DEFAULT 0.00;

    DECLARE cur_product_line CURSOR FOR
        SELECT productCode, MSRP
        FROM products
        WHERE productLine = p_product_line
        ORDER BY productCode;

    DECLARE CONTINUE HANDLER FOR NOT FOUND
        SET v_done = TRUE;

    SELECT COUNT(*)
    INTO v_line_count
    FROM products
    WHERE productLine = p_product_line;

    IF v_line_count = 0 THEN
        SELECT p_product_line AS productLine,
               'Product line not found or has no products' AS message;
        LEAVE main_block;
    END IF;

    OPEN cur_product_line;

    product_loop: LOOP
        FETCH cur_product_line
        INTO v_product_code, v_msrp;

        IF v_done THEN
            LEAVE product_loop;
        END IF;

        SET v_product_count = v_product_count + 1;
        SET v_total_msrp = v_total_msrp + v_msrp;
    END LOOP product_loop;

    CLOSE cur_product_line;

    SET v_average_msrp = v_total_msrp / v_product_count;

    SELECT p_product_line AS productLine,
           v_product_count AS totalProducts,
           v_total_msrp AS totalMSRP,
           v_average_msrp AS averageMSRP;
END$$

DELIMITER ;
```

Gọi procedure:

```sql
CALL sp_lab_cursor_product_line_summary('Classic Cars');
```

### 15.2. Procedure prepared statement: lọc order theo status

```sql
DROP PROCEDURE IF EXISTS sp_lab_prepared_orders_by_status;

DELIMITER $$

CREATE PROCEDURE sp_lab_prepared_orders_by_status(
    IN p_status VARCHAR(20)
)
BEGIN
    SET @sql_text =
        'SELECT orderNumber, orderDate, requiredDate, shippedDate,
                status, customerNumber
         FROM orders
         WHERE status = ?
         ORDER BY orderDate DESC, orderNumber DESC';

    SET @status_param = p_status;

    PREPARE stmt_orders_by_status
    FROM @sql_text;

    EXECUTE stmt_orders_by_status
    USING @status_param;

    DEALLOCATE PREPARE stmt_orders_by_status;
END$$

DELIMITER ;
```

Gọi procedure:

```sql
CALL sp_lab_prepared_orders_by_status('Shipped');
```

### Bài tập thực hành

**Bài 15.1.**

Tạo `sp_lab_cursor_product_line_summary` theo ví dụ.

**Bài 15.2.**

Gọi procedure cho `Classic Cars` và `Planes`.

**Bài 15.3.**

Tạo `sp_lab_prepared_orders_by_status` theo ví dụ.

**Bài 15.4.**

Gọi procedure cho `Shipped`, `Cancelled` và `On Hold`.

**Bài 15.5.**

Viết truy vấn set-based thay thế cursor procedure ở phần 15.1 bằng `COUNT(*)`, `SUM(MSRP)` và `AVG(MSRP)`.

---

## 16. Một số lỗi thường gặp

### Lỗi 1. Khai báo handler trước cursor

Sai:

```sql
DECLARE CONTINUE HANDLER FOR NOT FOUND
    SET v_done = TRUE;

DECLARE cur_products CURSOR FOR
    SELECT productCode
    FROM products;
```

Đúng:

```sql
DECLARE cur_products CURSOR FOR
    SELECT productCode
    FROM products;

DECLARE CONTINUE HANDLER FOR NOT FOUND
    SET v_done = TRUE;
```

---

### Lỗi 2. Kiểm tra `v_done` sau khi xử lý dòng

Sai:

```sql
FETCH cur_products INTO v_product_code;

SET v_count = v_count + 1;

IF v_done THEN
    LEAVE product_loop;
END IF;
```

Đúng:

```sql
FETCH cur_products INTO v_product_code;

IF v_done THEN
    LEAVE product_loop;
END IF;

SET v_count = v_count + 1;
```

---

### Lỗi 3. Dùng số biến FETCH không khớp số cột cursor

Nếu cursor trả về ba cột, `FETCH ... INTO` cũng phải có ba biến.

---

### Lỗi 4. Quên `DEALLOCATE PREPARE`

Sai:

```sql
PREPARE stmt_country FROM @sql_text;
EXECUTE stmt_country USING @country;
```

Nên có:

```sql
DEALLOCATE PREPARE stmt_country;
```

---

### Lỗi 5. Dùng local variable trực tiếp trong prepared statement

Không nên giả định local variable có thể xuất hiện trực tiếp trong SQL string đã prepare.

Thay vì:

```sql
SET @sql_text =
    'SELECT * FROM customers WHERE country = p_country';
```

Hãy dùng placeholder và session variable:

```sql
SET @sql_text =
    'SELECT * FROM customers WHERE country = ?';

SET @country_param = p_country;

PREPARE stmt_country FROM @sql_text;
EXECUTE stmt_country USING @country_param;
DEALLOCATE PREPARE stmt_country;
```

---

### Lỗi 6. Dùng placeholder cho tên bảng hoặc tên cột

Sai:

```sql
SET @sql_text = 'SELECT * FROM ?';
```

Placeholder `?` không dùng cho identifier.

Cần whitelist khi tên bảng hoặc tên cột thay đổi theo input.

### Bài tập thực hành

**Bài 16.1.**

Sửa đoạn code có handler được khai báo trước cursor.

**Bài 16.2.**

Sửa vòng lặp xử lý một dòng sau khi cursor đã hết dữ liệu.

**Bài 16.3.**

Bổ sung lệnh `DEALLOCATE PREPARE` cho một prepared statement còn thiếu.

**Bài 16.4.**

Sửa prepared statement dùng local parameter trực tiếp trong SQL string thành statement dùng `?` và `USING`.

**Bài 16.5.**

Giải thích vì sao `SELECT * FROM ?` không phải là cách hợp lệ để chọn bảng động.

---

## 17. Bài tập tổng hợp

### Bài 17.1. Cursor tổng giá trị đơn hàng

Tạo procedure `sp_lab_cursor_order_statistics`:

1. Có tham số `IN p_order_number INT`.
2. Kiểm tra order tồn tại.
3. Khai báo cursor lấy `productCode`, `quantityOrdered`, `priceEach`.
4. Duyệt cursor để tính số dòng, tổng số lượng, tổng giá trị.
5. Trả về toàn bộ kết quả sau khi đóng cursor.

---

### Bài 17.2. Cursor phân loại biên lợi nhuận

Tạo procedure `sp_lab_cursor_margin_counts`:

1. Có tham số `IN p_product_line VARCHAR(50)`.
2. Duyệt các sản phẩm trong product line bằng cursor.
3. Tính `MSRP - buyPrice` cho từng sản phẩm.
4. Đếm số sản phẩm có biên lợi nhuận `High`, `Medium`, `Low` theo các ngưỡng do bạn chọn.
5. Trả về ba số lượng sau khi cursor kết thúc.

---

### Bài 17.3. Prepared statement lọc khách hàng

Tạo procedure `sp_lab_prepared_customers_by_credit`:

1. Có tham số `IN p_country VARCHAR(50)`.
2. Có tham số `IN p_min_credit DECIMAL(10,2)`.
3. Chuẩn bị một câu SQL có hai placeholder.
4. Hiển thị khách hàng theo quốc gia và hạn mức tối thiểu.
5. Deallocate prepared statement sau execute.

---

### Bài 17.4. Prepared statement chọn nguồn có kiểm soát

Tạo procedure `sp_lab_prepared_safe_list`:

1. Có tham số `IN p_source VARCHAR(20)`.
2. Chỉ chấp nhận `customers`, `products`, `offices`.
3. Dùng `IF ... ELSEIF` để chọn câu SQL cố định tương ứng.
4. Dùng `PREPARE`, `EXECUTE`, `DEALLOCATE PREPARE`.
5. Trả thông báo nếu `p_source` không hợp lệ.

---

### Bài 17.5. So sánh cursor với set-based SQL

Với yêu cầu:

> Tính tổng doanh thu của mọi order theo customer.

1. Viết một truy vấn set-based dùng `JOIN` và `GROUP BY`.
2. Giải thích vì sao set-based SQL là lựa chọn đầu tiên.
3. Nêu một thay đổi trong yêu cầu khiến cursor có thể cần thiết hơn.
4. Đề xuất các biến cần có nếu viết cursor.
5. Nêu cách phát hiện cursor đã hết dòng.

---

## 18. Đáp án gợi ý cho một số bài tập

### Bài 6.5

```sql
SELECT COUNT(*) AS totalLines,
       COALESCE(SUM(quantityOrdered), 0) AS totalQuantity,
       COALESCE(SUM(quantityOrdered * priceEach), 0) AS totalOrderValue
FROM orderdetails
WHERE orderNumber = 10100;
```

### Bài 7.5

```sql
SELECT productLine,
       COUNT(*) AS totalProducts,
       COALESCE(SUM(MSRP - buyPrice), 0) AS totalEstimatedMargin,
       COALESCE(MAX(MSRP - buyPrice), 0) AS maximumEstimatedMargin
FROM products
WHERE productLine = 'Motorcycles'
GROUP BY productLine;
```

### Bài 10.1

```sql
SET @sql_text =
    'SELECT customerNumber, customerName, city, country
     FROM customers
     WHERE country = ?
     ORDER BY customerName';

PREPARE stmt_customers_by_country
FROM @sql_text;

SET @country = 'USA';

EXECUTE stmt_customers_by_country
USING @country;

DEALLOCATE PREPARE stmt_customers_by_country;
```

### Bài 10.4

```sql
SET @sql_text =
    'SELECT productCode, productName, productLine, MSRP
     FROM products
     WHERE productLine = ?
       AND MSRP >= ?
     ORDER BY MSRP DESC';

PREPARE stmt_products_by_line_and_price
FROM @sql_text;

SET @product_line = 'Classic Cars';
SET @min_msrp = 100.00;

EXECUTE stmt_products_by_line_and_price
USING @product_line, @min_msrp;

DEALLOCATE PREPARE stmt_products_by_line_and_price;
```

### Bài 11.1

```sql
DROP PROCEDURE IF EXISTS sp_lab_prepared_orders_by_status;

DELIMITER $$

CREATE PROCEDURE sp_lab_prepared_orders_by_status(
    IN p_status VARCHAR(20)
)
BEGIN
    SET @sql_text =
        'SELECT orderNumber, orderDate, requiredDate, shippedDate,
                status, customerNumber
         FROM orders
         WHERE status = ?
         ORDER BY orderDate DESC, orderNumber DESC';

    SET @status_param = p_status;

    PREPARE stmt_orders_by_status
    FROM @sql_text;

    EXECUTE stmt_orders_by_status
    USING @status_param;

    DEALLOCATE PREPARE stmt_orders_by_status;
END$$

DELIMITER ;
```

### Bài 11.3

```sql
DROP PROCEDURE IF EXISTS sp_lab_prepared_products_by_price_range;

DELIMITER $$

CREATE PROCEDURE sp_lab_prepared_products_by_price_range(
    IN p_min_price DECIMAL(10,2),
    IN p_max_price DECIMAL(10,2)
)
BEGIN
    SET @sql_text =
        'SELECT productCode, productName, productLine, MSRP
         FROM products
         WHERE MSRP BETWEEN ? AND ?
         ORDER BY MSRP, productName';

    SET @min_price_param = p_min_price;
    SET @max_price_param = p_max_price;

    PREPARE stmt_products_by_price_range
    FROM @sql_text;

    EXECUTE stmt_products_by_price_range
    USING @min_price_param, @max_price_param;

    DEALLOCATE PREPARE stmt_products_by_price_range;
END$$

DELIMITER ;
```

### Bài 12.2

```sql
DROP PROCEDURE IF EXISTS sp_lab_prepared_safe_list;

DELIMITER $$

CREATE PROCEDURE sp_lab_prepared_safe_list(
    IN p_source VARCHAR(20)
)
main_block: BEGIN
    IF p_source = 'customers' THEN
        SET @sql_text =
            'SELECT customerNumber AS entityCode,
                    customerName AS entityName
             FROM customers
             ORDER BY customerNumber
             LIMIT 10';
    ELSEIF p_source = 'products' THEN
        SET @sql_text =
            'SELECT productCode AS entityCode,
                    productName AS entityName
             FROM products
             ORDER BY productCode
             LIMIT 10';
    ELSE
        SELECT 'Invalid source. Use customers or products.' AS message;
        LEAVE main_block;
    END IF;

    PREPARE stmt_safe_list
    FROM @sql_text;

    EXECUTE stmt_safe_list;

    DEALLOCATE PREPARE stmt_safe_list;
END$$

DELIMITER ;
```

### Bài 17.3

```sql
DROP PROCEDURE IF EXISTS sp_lab_prepared_customers_by_credit;

DELIMITER $$

CREATE PROCEDURE sp_lab_prepared_customers_by_credit(
    IN p_country VARCHAR(50),
    IN p_min_credit DECIMAL(10,2)
)
BEGIN
    SET @sql_text =
        'SELECT customerNumber, customerName, city, country, creditLimit
         FROM customers
         WHERE country = ?
           AND creditLimit >= ?
         ORDER BY creditLimit DESC, customerName';

    SET @country_param = p_country;
    SET @min_credit_param = p_min_credit;

    PREPARE stmt_customers_by_credit
    FROM @sql_text;

    EXECUTE stmt_customers_by_credit
    USING @country_param, @min_credit_param;

    DEALLOCATE PREPARE stmt_customers_by_credit;
END$$

DELIMITER ;
```

### Bài 17.5 — Truy vấn set-based

```sql
SELECT o.customerNumber,
       c.customerName,
       COALESCE(SUM(od.quantityOrdered * od.priceEach), 0) AS totalOrderValue
FROM customers AS c
JOIN orders AS o
    ON o.customerNumber = c.customerNumber
JOIN orderdetails AS od
    ON od.orderNumber = o.orderNumber
GROUP BY o.customerNumber, c.customerName
ORDER BY totalOrderValue DESC, o.customerNumber;
```

---

## 19. Tóm tắt

Các kiến thức chính trong lab:

- Cursor xử lý result set từng dòng một.
- Vòng đời cursor gồm `DECLARE`, `OPEN`, `FETCH`, `CLOSE`.
- Trong block, thứ tự khai báo là: biến/condition → cursor → handler.
- `DECLARE CONTINUE HANDLER FOR NOT FOUND` thường dùng để phát hiện khi `FETCH` không còn dòng.
- Sau `FETCH`, phải kiểm tra cờ hết dữ liệu trước khi xử lý biến nhận dòng.
- Cursor phù hợp với logic row-by-row; truy vấn set-based thường là lựa chọn ưu tiên cho tổng hợp và cập nhật hàng loạt.
- Prepared statement dùng ba bước: `PREPARE`, `EXECUTE`, `DEALLOCATE PREPARE`.
- Placeholder `?` bind dữ liệu, không bind identifier như tên bảng hoặc tên cột.
- Trong stored procedure, prepared statement thường dùng session variables `@...` để chứa SQL và bind các giá trị.
- Khi identifier cần thay đổi theo input, phải whitelist các giá trị hợp lệ.
- Prepared statement có thể được execute nhiều lần trong cùng session; cần deallocate sau khi hoàn tất.
- `PREPARE`, `EXECUTE`, `DEALLOCATE PREPARE` dùng được trong stored procedure, nhưng không dùng được trong stored function hoặc trigger.

---

## 20. Từ khóa chính

- Cursor
- DECLARE CURSOR
- OPEN
- FETCH
- CLOSE
- NOT FOUND
- CONTINUE HANDLER
- Read-only Cursor
- Nonscrollable Cursor
- Asensitive Cursor
- Row-by-row Processing
- Set-based SQL
- Prepared Statement
- PREPARE
- EXECUTE
- USING
- DEALLOCATE PREPARE
- Placeholder
- Session Variable
- Dynamic SQL
- Whitelist
- SQL Injection
- classicmodels
