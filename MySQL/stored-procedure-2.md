---
title: "Lab: Điều kiện và vòng lặp trong MySQL Stored Procedures với classicmodels"
author: "Tên giảng viên"
duration: "180m"
difficulty: "Intermediate"
prerequisites:
  - "Đã hoàn thành lab Stored Procedures cơ bản: DELIMITER, CREATE PROCEDURE, CALL, biến cục bộ và tham số"
  - "Đã biết SELECT, WHERE, JOIN, GROUP BY và các hàm tổng hợp cơ bản"
  - "Có quyền CREATE ROUTINE và EXECUTE trên cơ sở dữ liệu thực hành"
summary: "Thực hành IF, CASE, LOOP, WHILE, REPEAT và LEAVE trong MySQL stored procedures trên cơ sở dữ liệu classicmodels."
---

# Lab: Điều kiện và vòng lặp trong MySQL Stored Procedures với `classicmodels`

## Link tham khảo

- [MySQL Tutorial — Stored Procedures](https://www.mysqltutorial.org/mysql-stored-procedure/)
- [MySQL Tutorial — IF THEN Statement](https://www.mysqltutorial.org/mysql-stored-procedure/mysql-if-statement/)
- [MySQL Tutorial — CASE Statement](https://www.mysqltutorial.org/mysql-stored-procedure/mysql-case-statement/)
- [MySQL Tutorial — LOOP](https://www.mysqltutorial.org/mysql-stored-procedure/loop-in-stored-procedures/)
- [MySQL Tutorial — WHILE Loop](https://www.mysqltutorial.org/mysql-stored-procedure/mysql-while-loop/)
- [MySQL Tutorial — REPEAT Loop](https://www.mysqltutorial.org/mysql-stored-procedure/mysql-repeat-loop/)
- [MySQL Tutorial — LEAVE Statement](https://www.mysqltutorial.org/mysql-stored-procedure/mysql-leave/)

## 1. Mục tiêu học tập

Sau khi hoàn thành lab này, người học có thể:

1. Dùng `IF ... THEN`, `IF ... THEN ... ELSE`, và `IF ... THEN ... ELSEIF ... ELSE` trong stored procedure.
2. Phân biệt **IF statement** trong stored procedure với hàm `IF()` trong biểu thức SQL.
3. Dùng **simple CASE statement** để so sánh một giá trị với nhiều lựa chọn.
4. Dùng **searched CASE statement** để kiểm tra nhiều điều kiện logic.
5. Giải thích được sự khác nhau giữa `CASE statement` và `CASE expression`.
6. Dùng `LOOP` cùng `LEAVE` để tạo vòng lặp có điều kiện dừng.
7. Dùng `WHILE` khi cần kiểm tra điều kiện trước mỗi lần lặp.
8. Dùng `REPEAT` khi cần thực hiện thân vòng lặp ít nhất một lần.
9. Dùng `LEAVE` để thoát khỏi vòng lặp hoặc block có nhãn.
10. Viết và gọi các stored procedure áp dụng điều kiện, vòng lặp trên dữ liệu `classicmodels`.

---

## 2. Chuẩn bị môi trường

Chọn cơ sở dữ liệu:

```sql
USE classicmodels;
```

Kiểm tra dữ liệu sẽ dùng trong lab:

```sql
SELECT customerNumber, customerName, country, creditLimit
FROM customers
ORDER BY customerNumber
LIMIT 10;
```

```sql
SELECT orderNumber, orderDate, shippedDate, status, customerNumber
FROM orders
ORDER BY orderNumber
LIMIT 10;
```

```sql
SELECT productCode, productName, productLine, buyPrice, MSRP
FROM products
ORDER BY productCode
LIMIT 10;
```

```sql
SELECT orderNumber, productCode, quantityOrdered, priceEach, orderLineNumber
FROM orderdetails
WHERE orderNumber = 10100
ORDER BY orderLineNumber;
```

Trong lab này, procedure dùng tiền tố:

```text
sp_lab_
```

Ví dụ:

```text
sp_lab_classify_credit_limit
sp_lab_order_status_message
sp_lab_sum_with_loop
sp_lab_factorial_with_while
sp_lab_countdown_with_repeat
```

Các procedure minh họa chủ yếu chỉ đọc dữ liệu từ `classicmodels` hoặc thực hiện tính toán trên biến cục bộ. Do đó, việc gọi procedure không làm thay đổi dữ liệu gốc.

### Bài tập thực hành

**Bài 2.1.**

Viết lệnh chọn cơ sở dữ liệu `classicmodels`.

**Bài 2.2.**

Viết truy vấn xem `customerNumber`, `customerName`, `creditLimit` của khách hàng `103`.

**Bài 2.3.**

Viết truy vấn xem `orderNumber`, `status`, `customerNumber` của đơn hàng `10100`.

**Bài 2.4.**

Viết truy vấn xem `productCode`, `productName`, `MSRP` của năm sản phẩm đầu tiên.

**Bài 2.5.**

Giải thích vì sao các procedure trong lab này nên được đặt tên với tiền tố `sp_lab_`.

---

## 3. Kiến thức nền: block, biến cục bộ và label

Các cấu trúc điều kiện và vòng lặp được viết bên trong block:

```sql
BEGIN
    -- Khai báo biến
    -- Các câu lệnh IF, CASE, LOOP, WHILE, REPEAT
END
```

Ví dụ khai báo biến cục bộ:

```sql
DECLARE v_total INT DEFAULT 0;
DECLARE v_message VARCHAR(100) DEFAULT '';
```

Quy tắc quan trọng:

> Tất cả `DECLARE` phải xuất hiện ở đầu block `BEGIN ... END`, trước `SELECT`, `SET`, `IF`, `CASE`, vòng lặp và các câu lệnh khác.

Một số cấu trúc cần **label** để chỉ ra nơi cần thoát bằng `LEAVE`.

Ví dụ:

```sql
number_loop: LOOP
    -- statements

    IF condition THEN
        LEAVE number_loop;
    END IF;
END LOOP number_loop;
```

| Thành phần | Ý nghĩa |
|---|---|
| `number_loop:` | Nhãn bắt đầu của loop |
| `LEAVE number_loop` | Thoát khỏi loop có nhãn `number_loop` |
| `END LOOP number_loop` | Kết thúc loop; ghi lại nhãn để tăng tính rõ ràng |

### Bài tập thực hành

**Bài 3.1.**

Viết một block `BEGIN ... END` có khai báo biến `v_count INT DEFAULT 0`.

**Bài 3.2.**

Nêu lý do `DECLARE` phải đặt ở đầu block.

**Bài 3.3.**

Viết một loop label tên `test_loop`.

**Bài 3.4.**

Viết câu lệnh `LEAVE` để thoát khỏi loop `test_loop`.

**Bài 3.5.**

Nêu sự khác nhau giữa biến cục bộ `v_total` và session variable `@total`.

---

# Phần A. Điều kiện trong Stored Procedures

## 4. `IF` statement

### 4.1. `IF ... THEN`

`IF` statement cho phép procedure kiểm tra một điều kiện. Nếu điều kiện đúng, procedure thực hiện nhóm câu lệnh tương ứng.

Cú pháp:

```sql
IF search_condition THEN
    statement_list;
END IF;
```

Ví dụ: kiểm tra xem một khách hàng có hạn mức tín dụng cao hay không.

```sql
DROP PROCEDURE IF EXISTS sp_lab_check_high_credit_customer;

DELIMITER $$

CREATE PROCEDURE sp_lab_check_high_credit_customer(
    IN p_customer_number INT
)
BEGIN
    DECLARE v_credit_limit DECIMAL(10,2) DEFAULT 0.00;
    DECLARE v_customer_count INT DEFAULT 0;
    DECLARE v_message VARCHAR(150) DEFAULT '';

    SELECT COUNT(*), COALESCE(MAX(creditLimit), 0)
    INTO v_customer_count, v_credit_limit
    FROM customers
    WHERE customerNumber = p_customer_number;

    IF v_customer_count = 1
       AND v_credit_limit >= 100000.00 THEN
        SET v_message = 'Customer has a high credit limit';
    END IF;

    SELECT p_customer_number AS customerNumber,
           v_credit_limit AS creditLimit,
           v_message AS message;
END$$

DELIMITER ;
```

Gọi procedure:

```sql
CALL sp_lab_check_high_credit_customer(103);
```

Nếu khách hàng không thỏa điều kiện, cột `message` giữ chuỗi rỗng. Trong thực tế, dạng `IF ... ELSE` thường dễ đọc hơn vì luôn xử lý cả hai nhánh.

### 4.2. `IF ... THEN ... ELSE`

Cú pháp:

```sql
IF search_condition THEN
    statement_list_1;
ELSE
    statement_list_2;
END IF;
```

Ví dụ: phân loại khách hàng có hạn mức tín dụng từ `50000.00` trở lên hay thấp hơn.

```sql
DROP PROCEDURE IF EXISTS sp_lab_credit_limit_level;

DELIMITER $$

CREATE PROCEDURE sp_lab_credit_limit_level(
    IN p_customer_number INT
)
BEGIN
    DECLARE v_credit_limit DECIMAL(10,2) DEFAULT 0.00;
    DECLARE v_customer_count INT DEFAULT 0;
    DECLARE v_level VARCHAR(30) DEFAULT '';

    SELECT COUNT(*), COALESCE(MAX(creditLimit), 0)
    INTO v_customer_count, v_credit_limit
    FROM customers
    WHERE customerNumber = p_customer_number;

    IF v_customer_count = 0 THEN
        SET v_level = 'Customer not found';
    ELSEIF v_credit_limit >= 50000.00 THEN
        SET v_level = 'Standard or high credit';
    ELSE
        SET v_level = 'Low credit';
    END IF;

    SELECT p_customer_number AS customerNumber,
           v_credit_limit AS creditLimit,
           v_level AS creditLevel;
END$$

DELIMITER ;
```

Gọi procedure:

```sql
CALL sp_lab_credit_limit_level(103);
```

### 4.3. `IF ... THEN ... ELSEIF ... ELSE`

Cú pháp tổng quát:

```sql
IF search_condition_1 THEN
    statement_list_1;
ELSEIF search_condition_2 THEN
    statement_list_2;
ELSEIF search_condition_3 THEN
    statement_list_3;
ELSE
    statement_list_else;
END IF;
```

Ví dụ: phân loại đơn hàng theo trạng thái.

```sql
DROP PROCEDURE IF EXISTS sp_lab_order_status_if;

DELIMITER $$

CREATE PROCEDURE sp_lab_order_status_if(
    IN p_order_number INT
)
BEGIN
    DECLARE v_status VARCHAR(15) DEFAULT '';
    DECLARE v_order_count INT DEFAULT 0;
    DECLARE v_message VARCHAR(150) DEFAULT '';

    SELECT COUNT(*), COALESCE(MAX(status), '')
    INTO v_order_count, v_status
    FROM orders
    WHERE orderNumber = p_order_number;

    IF v_order_count = 0 THEN
        SET v_message = 'Order not found';
    ELSEIF v_status = 'Shipped' THEN
        SET v_message = 'Order has been shipped';
    ELSEIF v_status = 'In Process' THEN
        SET v_message = 'Order is being processed';
    ELSEIF v_status = 'On Hold' THEN
        SET v_message = 'Order is on hold';
    ELSEIF v_status = 'Cancelled' THEN
        SET v_message = 'Order was cancelled';
    ELSE
        SET v_message = 'Order has another status';
    END IF;

    SELECT p_order_number AS orderNumber,
           v_status AS status,
           v_message AS message;
END$$

DELIMITER ;
```

Gọi procedure:

```sql
CALL sp_lab_order_status_if(10100);
```

### 4.4. `IF` statement và hàm `IF()`

Đừng nhầm hai cấu trúc sau:

| Cấu trúc | Dùng ở đâu? | Ví dụ |
|---|---|---|
| `IF ... THEN ... END IF` | Bên trong stored procedure, trigger, event | Điều khiển luồng thực thi |
| `IF(condition, value_if_true, value_if_false)` | Trong biểu thức SQL | Tạo giá trị trong `SELECT` |

Ví dụ hàm `IF()` trong `SELECT`:

```sql
SELECT customerNumber,
       customerName,
       creditLimit,
       IF(creditLimit >= 50000.00, 'High', 'Low') AS creditLevel
FROM customers
WHERE customerNumber = 103;
```

Ví dụ trên không phải `IF statement` của stored procedure.

### Bài tập thực hành

**Bài 4.1.**

Tạo procedure `sp_lab_check_customer_country` có tham số `IN p_customer_number INT`. Nếu khách hàng thuộc `USA`, procedure trả về message `Customer is in USA`; nếu không thì trả về chuỗi rỗng.

**Bài 4.2.**

Tạo procedure `sp_lab_credit_category` phân loại khách hàng `103`: `High` nếu `creditLimit >= 100000.00`, ngược lại `Normal`.

**Bài 4.3.**

Tạo procedure `sp_lab_order_shipping_state` có tham số `IN p_order_number INT`. Nếu `shippedDate IS NULL`, trả về `Not shipped`; ngược lại trả về `Shipped`.

**Bài 4.4.**

Tạo procedure `sp_lab_product_stock_message` có tham số `IN p_product_code VARCHAR(15)`. Phân loại `quantityInStock`: `Low` nếu dưới `1000`, `Available` nếu từ `1000` trở lên.

**Bài 4.5.**

Tạo procedure `sp_lab_customer_exists` có tham số `IN p_customer_number INT`. Nếu khách hàng tồn tại, trả về `Found`; ngược lại trả về `Not found`.

---

## 5. `CASE` statement

`CASE statement` là một cấu trúc điều kiện thay thế cho nhiều nhánh `IF ... ELSEIF`.

MySQL có hai dạng:

1. **Simple CASE statement**: so sánh một giá trị với các giá trị cụ thể.
2. **Searched CASE statement**: kiểm tra nhiều điều kiện logic.

> `CASE statement` trong stored procedure khác với `CASE expression` dùng bên trong `SELECT`, `ORDER BY` hoặc biểu thức SQL.

---

## 6. Simple `CASE` statement

### 6.1. Cú pháp

```sql
CASE case_value
    WHEN when_value_1 THEN
        statement_list_1;
    WHEN when_value_2 THEN
        statement_list_2;
    ELSE
        statement_list_else;
END CASE;
```

Simple `CASE` phù hợp khi cần so sánh **một giá trị** với nhiều giá trị cụ thể.

### 6.2. Ví dụ: diễn giải trạng thái đơn hàng

```sql
DROP PROCEDURE IF EXISTS sp_lab_order_status_case;

DELIMITER $$

CREATE PROCEDURE sp_lab_order_status_case(
    IN p_order_number INT
)
BEGIN
    DECLARE v_status VARCHAR(15) DEFAULT '';
    DECLARE v_order_count INT DEFAULT 0;
    DECLARE v_message VARCHAR(150) DEFAULT '';

    SELECT COUNT(*), COALESCE(MAX(status), '')
    INTO v_order_count, v_status
    FROM orders
    WHERE orderNumber = p_order_number;

    CASE v_status
        WHEN 'Shipped' THEN
            SET v_message = 'Order has been shipped';
        WHEN 'In Process' THEN
            SET v_message = 'Order is being processed';
        WHEN 'On Hold' THEN
            SET v_message = 'Order is on hold';
        WHEN 'Cancelled' THEN
            SET v_message = 'Order was cancelled';
        WHEN 'Disputed' THEN
            SET v_message = 'Order is disputed';
        ELSE
            SET v_message = 'Order not found or status is not classified';
    END CASE;

    SELECT p_order_number AS orderNumber,
           v_status AS status,
           v_message AS message;
END$$

DELIMITER ;
```

Gọi procedure:

```sql
CALL sp_lab_order_status_case(10100);
```

### 6.3. Vì sao dùng simple CASE?

So sánh hai cách viết:

```sql
IF v_status = 'Shipped' THEN
    ...
ELSEIF v_status = 'In Process' THEN
    ...
ELSEIF v_status = 'On Hold' THEN
    ...
END IF;
```

và:

```sql
CASE v_status
    WHEN 'Shipped' THEN
        ...
    WHEN 'In Process' THEN
        ...
    WHEN 'On Hold' THEN
        ...
END CASE;
```

Khi tất cả nhánh đều kiểm tra cùng một biến `v_status`, simple `CASE` thường ngắn và dễ đọc hơn.

### 6.4. Lưu ý về `ELSE`

Nên có nhánh `ELSE` để xử lý các giá trị không dự kiến hoặc trường hợp không tìm thấy dữ liệu.

Với `CASE statement`, nếu không có nhánh phù hợp và không có `ELSE`, MySQL có thể phát sinh lỗi `Case not found`.

### Bài tập thực hành

**Bài 6.1.**

Tạo procedure `sp_lab_status_label` có tham số `IN p_order_number INT`. Dùng simple `CASE` để gán nhãn cho các trạng thái `Shipped`, `In Process`, `Cancelled`, `On Hold`.

**Bài 6.2.**

Tạo procedure `sp_lab_office_territory_message` có tham số `IN p_office_code VARCHAR(10)`. Dùng simple `CASE` với `territory` để trả về thông điệp phù hợp cho `NA`, `EMEA`, `APAC`, `Japan`.

**Bài 6.3.**

Tạo procedure `sp_lab_product_line_message` có tham số `IN p_product_code VARCHAR(15)`. Dùng simple `CASE` với `productLine` để trả về nhãn cho `Classic Cars`, `Motorcycles`, `Planes`; các trường hợp khác trả về `Other product line`.

**Bài 6.4.**

Tạo procedure `sp_lab_order_status_case` nhưng chỉ dùng `CASE`, không dùng `IF ... ELSEIF` để phân loại status.

**Bài 6.5.**

Giải thích vì sao simple `CASE` phù hợp hơn `IF ... ELSEIF` khi mọi điều kiện đều so sánh cùng một biến `v_status`.

---

## 7. Searched `CASE` statement

### 7.1. Cú pháp

```sql
CASE
    WHEN search_condition_1 THEN
        statement_list_1;
    WHEN search_condition_2 THEN
        statement_list_2;
    ELSE
        statement_list_else;
END CASE;
```

Searched `CASE` phù hợp khi các nhánh là các điều kiện khác nhau, chẳng hạn so sánh theo khoảng giá trị.

### 7.2. Ví dụ: phân loại sản phẩm theo MSRP

```sql
DROP PROCEDURE IF EXISTS sp_lab_product_price_category;

DELIMITER $$

CREATE PROCEDURE sp_lab_product_price_category(
    IN p_product_code VARCHAR(15)
)
BEGIN
    DECLARE v_msrp DECIMAL(10,2) DEFAULT 0.00;
    DECLARE v_product_count INT DEFAULT 0;
    DECLARE v_category VARCHAR(30) DEFAULT '';

    SELECT COUNT(*), COALESCE(MAX(MSRP), 0)
    INTO v_product_count, v_msrp
    FROM products
    WHERE productCode = p_product_code;

    CASE
        WHEN v_product_count = 0 THEN
            SET v_category = 'Product not found';
        WHEN v_msrp >= 150.00 THEN
            SET v_category = 'Premium';
        WHEN v_msrp >= 100.00 THEN
            SET v_category = 'Standard';
        WHEN v_msrp > 0 THEN
            SET v_category = 'Budget';
        ELSE
            SET v_category = 'Price is not available';
    END CASE;

    SELECT p_product_code AS productCode,
           v_msrp AS MSRP,
           v_category AS priceCategory;
END$$

DELIMITER ;
```

Gọi procedure:

```sql
CALL sp_lab_product_price_category('S10_1678');
```

### 7.3. Ví dụ: phân loại mức giảm giá

```sql
DROP PROCEDURE IF EXISTS sp_lab_discount_category;

DELIMITER $$

CREATE PROCEDURE sp_lab_discount_category(
    IN p_discount_percent DECIMAL(5,2)
)
BEGIN
    DECLARE v_message VARCHAR(100) DEFAULT '';

    CASE
        WHEN p_discount_percent < 0 THEN
            SET v_message = 'Invalid discount';
        WHEN p_discount_percent = 0 THEN
            SET v_message = 'No discount';
        WHEN p_discount_percent <= 10 THEN
            SET v_message = 'Small discount';
        WHEN p_discount_percent <= 30 THEN
            SET v_message = 'Medium discount';
        WHEN p_discount_percent <= 100 THEN
            SET v_message = 'Large discount';
        ELSE
            SET v_message = 'Invalid discount';
    END CASE;

    SELECT p_discount_percent AS discountPercent,
           v_message AS discountCategory;
END$$

DELIMITER ;
```

Gọi procedure:

```sql
CALL sp_lab_discount_category(15.00);
```

### 7.4. `CASE statement` và `CASE expression`

`CASE statement` dùng để thực hiện các câu lệnh:

```sql
CASE
    WHEN v_msrp >= 150 THEN
        SET v_category = 'Premium';
    ELSE
        SET v_category = 'Budget';
END CASE;
```

`CASE expression` dùng để tạo một giá trị trong câu SQL:

```sql
SELECT productCode,
       productName,
       MSRP,
       CASE
           WHEN MSRP >= 150 THEN 'Premium'
           WHEN MSRP >= 100 THEN 'Standard'
           ELSE 'Budget'
       END AS priceCategory
FROM products;
```

### Bài tập thực hành

**Bài 7.1.**

Tạo procedure `sp_lab_credit_limit_category` có tham số `IN p_customer_number INT`. Dùng searched `CASE` để phân loại `creditLimit`: `High` nếu từ `100000.00`, `Medium` nếu từ `50000.00`, còn lại `Low`.

**Bài 7.2.**

Tạo procedure `sp_lab_order_value_category` có tham số `IN p_order_number INT`. Tính tổng `quantityOrdered * priceEach` của đơn hàng, sau đó dùng searched `CASE` để phân loại `Large`, `Medium`, `Small`.

**Bài 7.3.**

Tạo procedure `sp_lab_product_margin_category` có tham số `IN p_product_code VARCHAR(15)`. Dựa vào `MSRP - buyPrice`, dùng searched `CASE` để phân loại `High margin`, `Medium margin`, `Low margin`.

**Bài 7.4.**

Tạo procedure `sp_lab_payment_amount_category` có tham số `IN p_amount DECIMAL(12,2)`. Dùng searched `CASE` phân loại `Invalid`, `Small`, `Medium`, `Large`.

**Bài 7.5.**

Viết một câu `SELECT` dùng `CASE expression` để phân loại mọi sản phẩm thành `Premium`, `Standard`, `Budget` theo `MSRP`.

---

# Phần B. Vòng lặp trong Stored Procedures

## 8. Tổng quan về vòng lặp

MySQL hỗ trợ ba cấu trúc vòng lặp chính trong stored programs:

| Vòng lặp | Kiểm tra điều kiện | Số lần chạy tối thiểu | Cách dừng thường dùng |
|---|---|---:|---|
| `LOOP` | Không có điều kiện tích hợp | Không xác định | `LEAVE` |
| `WHILE` | Kiểm tra trước mỗi lần lặp | 0 | Điều kiện thành false |
| `REPEAT` | Kiểm tra sau thân vòng lặp | 1 | Điều kiện sau `UNTIL` thành true |

### Khi nào dùng?

| Cấu trúc | Trường hợp phù hợp |
|---|---|
| `LOOP` | Muốn tự kiểm soát điều kiện dừng bằng `IF` + `LEAVE` |
| `WHILE` | Chỉ cần chạy khi điều kiện ban đầu là đúng |
| `REPEAT` | Cần chắc chắn thân vòng lặp chạy ít nhất một lần |
| `LEAVE` | Cần thoát ngay khỏi loop hoặc block có nhãn |

> Trong các ứng dụng cơ sở dữ liệu thực tế, vòng lặp thường được kết hợp với cursor để duyệt từng dòng. Cursor sẽ được học ở phần sau. Trong lab này, vòng lặp được minh họa bằng phép tính và quy tắc nghiệp vụ đơn giản để tập trung vào cú pháp và luồng điều khiển.

### Bài tập thực hành

**Bài 8.1.**

Nêu khác biệt giữa `WHILE` và `REPEAT` về số lần thực hiện tối thiểu.

**Bài 8.2.**

Nêu lý do `LOOP` thường cần `LEAVE`.

**Bài 8.3.**

Chọn vòng lặp phù hợp nếu thân vòng lặp phải chạy ít nhất một lần.

**Bài 8.4.**

Chọn vòng lặp phù hợp nếu cần kiểm tra điều kiện trước lần lặp đầu tiên.

**Bài 8.5.**

Nêu một tình huống trong stored procedure mà có thể cần vòng lặp.

---

## 9. `LOOP`

### 9.1. Cú pháp

```sql
[begin_label:] LOOP
    statement_list
END LOOP [end_label];
```

`LOOP` không có điều kiện dừng tích hợp. Vì vậy cần dùng một cơ chế dừng bên trong, thường là:

```sql
IF condition THEN
    LEAVE label;
END IF;
```

Nếu không có `LEAVE`, procedure có thể tạo vòng lặp vô hạn.

### 9.2. Ví dụ: tính tổng từ 1 đến n bằng `LOOP`

```sql
DROP PROCEDURE IF EXISTS sp_lab_sum_with_loop;

DELIMITER $$

CREATE PROCEDURE sp_lab_sum_with_loop(
    IN p_n INT
)
BEGIN
    DECLARE v_i INT DEFAULT 1;
    DECLARE v_total BIGINT DEFAULT 0;
    DECLARE v_message VARCHAR(100) DEFAULT '';

    IF p_n < 1 THEN
        SET v_message = 'p_n must be at least 1';
    ELSE
        sum_loop: LOOP
            SET v_total = v_total + v_i;

            IF v_i >= p_n THEN
                LEAVE sum_loop;
            END IF;

            SET v_i = v_i + 1;
        END LOOP sum_loop;

        SET v_message = 'Sum calculated successfully';
    END IF;

    SELECT p_n AS n,
           v_total AS total,
           v_message AS message;
END$$

DELIMITER ;
```

Gọi procedure:

```sql
CALL sp_lab_sum_with_loop(10);
```

Kết quả mong đợi:

```text
n = 10
total = 55
```

### 9.3. Phân tích vòng lặp

Với `p_n = 3`:

| Lần lặp | `v_i` trước khi cộng | `v_total` sau khi cộng | Điều kiện dừng |
|---:|---:|---:|---|
| 1 | 1 | 1 | `1 >= 3` là false |
| 2 | 2 | 3 | `2 >= 3` là false |
| 3 | 3 | 6 | `3 >= 3` là true → `LEAVE` |

### 9.4. Ví dụ: đếm số bước cần đạt ngưỡng

```sql
DROP PROCEDURE IF EXISTS sp_lab_steps_to_reach_target;

DELIMITER $$

CREATE PROCEDURE sp_lab_steps_to_reach_target(
    IN p_start_value INT,
    IN p_target_value INT,
    IN p_step INT
)
BEGIN
    DECLARE v_current INT DEFAULT 0;
    DECLARE v_steps INT DEFAULT 0;
    DECLARE v_message VARCHAR(100) DEFAULT '';

    SET v_current = p_start_value;

    IF p_step <= 0 THEN
        SET v_message = 'Step must be greater than 0';
    ELSEIF p_start_value >= p_target_value THEN
        SET v_message = 'Target already reached';
    ELSE
        target_loop: LOOP
            SET v_current = v_current + p_step;
            SET v_steps = v_steps + 1;

            IF v_current >= p_target_value THEN
                LEAVE target_loop;
            END IF;
        END LOOP target_loop;

        SET v_message = 'Target reached';
    END IF;

    SELECT p_start_value AS startValue,
           p_target_value AS targetValue,
           p_step AS stepValue,
           v_current AS finalValue,
           v_steps AS totalSteps,
           v_message AS message;
END$$

DELIMITER ;
```

Gọi procedure:

```sql
CALL sp_lab_steps_to_reach_target(10, 50, 7);
```

### Bài tập thực hành

**Bài 9.1.**

Tạo procedure `sp_lab_sum_even_numbers_loop` có tham số `IN p_n INT`; dùng `LOOP` để tính tổng các số chẵn từ 2 đến `p_n`.

**Bài 9.2.**

Tạo procedure `sp_lab_count_multiples_loop` có tham số `IN p_n INT`, `IN p_divisor INT`; dùng `LOOP` để đếm số bội của `p_divisor` từ 1 đến `p_n`.

**Bài 9.3.**

Tạo procedure `sp_lab_power_loop` có tham số `IN p_base INT`, `IN p_exponent INT`; dùng `LOOP` để tính `p_base^p_exponent`.

**Bài 9.4.**

Tạo procedure `sp_lab_find_first_multiple_loop` có tham số `IN p_start INT`, `IN p_divisor INT`; dùng `LOOP` để tìm số đầu tiên lớn hơn hoặc bằng `p_start` chia hết cho `p_divisor`.

**Bài 9.5.**

Giải thích điều gì xảy ra nếu một `LOOP` không có `LEAVE` hoặc một cơ chế kết thúc tương đương.

---

## 10. `WHILE` loop

### 10.1. Cú pháp

```sql
[begin_label:] WHILE search_condition DO
    statement_list
END WHILE [end_label];
```

`WHILE` kiểm tra điều kiện **trước** khi thực hiện thân vòng lặp.

Vì vậy, nếu điều kiện ngay từ đầu là false, thân vòng lặp không chạy lần nào.

### 10.2. Ví dụ: tính giai thừa bằng `WHILE`

```sql
DROP PROCEDURE IF EXISTS sp_lab_factorial_with_while;

DELIMITER $$

CREATE PROCEDURE sp_lab_factorial_with_while(
    IN p_n INT
)
BEGIN
    DECLARE v_i INT DEFAULT 1;
    DECLARE v_factorial DECIMAL(30,0) DEFAULT 1;
    DECLARE v_message VARCHAR(100) DEFAULT '';

    IF p_n < 0 THEN
        SET v_message = 'p_n must be nonnegative';
    ELSEIF p_n > 20 THEN
        SET v_message = 'p_n is too large for this demonstration';
    ELSE
        WHILE v_i <= p_n DO
            SET v_factorial = v_factorial * v_i;
            SET v_i = v_i + 1;
        END WHILE;

        SET v_message = 'Factorial calculated successfully';
    END IF;

    SELECT p_n AS n,
           v_factorial AS factorialValue,
           v_message AS message;
END$$

DELIMITER ;
```

Gọi procedure:

```sql
CALL sp_lab_factorial_with_while(5);
```

Kết quả mong đợi:

```text
5! = 120
```

### 10.3. Ví dụ: tính tổng số lượng sản phẩm theo dãy chỉ số

```sql
DROP PROCEDURE IF EXISTS sp_lab_sum_odd_numbers_while;

DELIMITER $$

CREATE PROCEDURE sp_lab_sum_odd_numbers_while(
    IN p_n INT
)
BEGIN
    DECLARE v_i INT DEFAULT 1;
    DECLARE v_total BIGINT DEFAULT 0;
    DECLARE v_message VARCHAR(100) DEFAULT '';

    IF p_n < 1 THEN
        SET v_message = 'p_n must be at least 1';
    ELSE
        WHILE v_i <= p_n DO
            SET v_total = v_total + v_i;
            SET v_i = v_i + 2;
        END WHILE;

        SET v_message = 'Odd-number sum calculated successfully';
    END IF;

    SELECT p_n AS upperBound,
           v_total AS oddNumberSum,
           v_message AS message;
END$$

DELIMITER ;
```

Gọi procedure:

```sql
CALL sp_lab_sum_odd_numbers_while(9);
```

### 10.4. Khi `WHILE` không chạy

```sql
WHILE v_i <= p_n DO
    ...
END WHILE;
```

Nếu `v_i = 1` và `p_n = 0`, điều kiện `1 <= 0` là false. Thân vòng lặp không thực hiện lần nào.

Đây là điểm khác biệt quan trọng với `REPEAT`.

### Bài tập thực hành

**Bài 10.1.**

Tạo procedure `sp_lab_sum_numbers_while` có tham số `IN p_n INT`; dùng `WHILE` để tính tổng từ 1 đến `p_n`.

**Bài 10.2.**

Tạo procedure `sp_lab_countdown_while` có tham số `IN p_start INT`; dùng `WHILE` để đếm số lần cần giảm từ `p_start` về 0 và trả về tổng số bước.

**Bài 10.3.**

Tạo procedure `sp_lab_compound_interest_while` có tham số `IN p_initial_amount DECIMAL(12,2)`, `IN p_rate DECIMAL(5,2)`, `IN p_periods INT`; dùng `WHILE` để tính giá trị cuối sau nhiều kỳ.

**Bài 10.4.**

Tạo procedure `sp_lab_product_line_count_while` có tham số `IN p_n INT`; dùng `WHILE` để tính tổng từ 1 đến `p_n`, sau đó trả về kết quả cùng số lượng product line thực tế của bảng `productlines`.

**Bài 10.5.**

Giải thích vì sao `WHILE` phù hợp khi yêu cầu là “chỉ lặp khi điều kiện ban đầu đúng”.

---

## 11. `REPEAT` loop

### 11.1. Cú pháp

```sql
[begin_label:] REPEAT
    statement_list
UNTIL search_condition
END REPEAT [end_label];
```

`REPEAT` thực hiện thân vòng lặp trước, sau đó mới kiểm tra điều kiện sau `UNTIL`.

Vì vậy, `REPEAT` luôn chạy **ít nhất một lần**.

### 11.2. Ví dụ: tính tổng từ 1 đến n bằng `REPEAT`

```sql
DROP PROCEDURE IF EXISTS sp_lab_sum_with_repeat;

DELIMITER $$

CREATE PROCEDURE sp_lab_sum_with_repeat(
    IN p_n INT
)
BEGIN
    DECLARE v_i INT DEFAULT 1;
    DECLARE v_total BIGINT DEFAULT 0;
    DECLARE v_message VARCHAR(100) DEFAULT '';

    IF p_n < 1 THEN
        SET v_message = 'p_n must be at least 1';
    ELSE
        REPEAT
            SET v_total = v_total + v_i;
            SET v_i = v_i + 1;
        UNTIL v_i > p_n
        END REPEAT;

        SET v_message = 'Sum calculated successfully';
    END IF;

    SELECT p_n AS n,
           v_total AS total,
           v_message AS message;
END$$

DELIMITER ;
```

Gọi procedure:

```sql
CALL sp_lab_sum_with_repeat(10);
```

### 11.3. Ví dụ: tìm số mũ nhỏ nhất đạt một ngưỡng

Ví dụ này tính số lần nhân đôi cần thiết để giá trị bắt đầu đạt hoặc vượt mục tiêu.

```sql
DROP PROCEDURE IF EXISTS sp_lab_doubling_steps_repeat;

DELIMITER $$

CREATE PROCEDURE sp_lab_doubling_steps_repeat(
    IN p_start_value DECIMAL(14,2),
    IN p_target_value DECIMAL(14,2)
)
BEGIN
    DECLARE v_current DECIMAL(14,2) DEFAULT 0.00;
    DECLARE v_steps INT DEFAULT 0;
    DECLARE v_message VARCHAR(100) DEFAULT '';

    SET v_current = p_start_value;

    IF p_start_value <= 0 OR p_target_value <= 0 THEN
        SET v_message = 'Values must be greater than 0';
    ELSEIF p_start_value >= p_target_value THEN
        SET v_message = 'Target already reached';
    ELSE
        REPEAT
            SET v_current = v_current * 2;
            SET v_steps = v_steps + 1;
        UNTIL v_current >= p_target_value
        END REPEAT;

        SET v_message = 'Target reached';
    END IF;

    SELECT p_start_value AS startValue,
           p_target_value AS targetValue,
           v_current AS finalValue,
           v_steps AS totalSteps,
           v_message AS message;
END$$

DELIMITER ;
```

Gọi procedure:

```sql
CALL sp_lab_doubling_steps_repeat(10.00, 100.00);
```

### 11.4. So sánh `WHILE` và `REPEAT`

| Tiêu chí | `WHILE` | `REPEAT` |
|---|---|---|
| Vị trí kiểm tra điều kiện | Trước thân vòng lặp | Sau thân vòng lặp |
| Số lần chạy tối thiểu | 0 | 1 |
| Điều kiện tiếp tục | `condition` là true | `UNTIL condition` chưa true |
| Phù hợp khi | Có thể không cần chạy lần nào | Cần chạy ít nhất một lần |

### Bài tập thực hành

**Bài 11.1.**

Tạo procedure `sp_lab_sum_even_numbers_repeat` có tham số `IN p_n INT`; dùng `REPEAT` để tính tổng các số chẵn từ 2 đến `p_n`.

**Bài 11.2.**

Tạo procedure `sp_lab_multiplication_repeat` có tham số `IN p_number INT`, `IN p_times INT`; dùng `REPEAT` để tính `p_number * p_times` bằng phép cộng lặp.

**Bài 11.3.**

Tạo procedure `sp_lab_reduce_value_repeat` có tham số `IN p_start INT`, `IN p_step INT`; dùng `REPEAT` để giảm giá trị về nhỏ hơn hoặc bằng 0, sau đó trả về số lần lặp.

**Bài 11.4.**

Tạo procedure `sp_lab_repeat_once_demo` có tham số `IN p_n INT`; cho thấy thân `REPEAT` chạy ít nhất một lần ngay cả khi `p_n = 0`.

**Bài 11.5.**

Giải thích sự khác nhau về vị trí kiểm tra điều kiện của `WHILE` và `REPEAT`.

---

## 12. `LEAVE` statement

### 12.1. Vai trò

`LEAVE` thoát ngay khỏi:

- Một vòng lặp có label: `LOOP`, `WHILE`, `REPEAT`.
- Một block `BEGIN ... END` có label.

Cú pháp:

```sql
LEAVE label;
```

`LEAVE` không tự biết cần thoát khỏi đâu. Vì vậy, cần ghi rõ label hợp lệ.

### 12.2. `LEAVE` trong `LOOP`

Ví dụ:

```sql
number_loop: LOOP
    SET v_i = v_i + 1;

    IF v_i > p_n THEN
        LEAVE number_loop;
    END IF;
END LOOP number_loop;
```

### 12.3. Ví dụ: tìm số đầu tiên chia hết

```sql
DROP PROCEDURE IF EXISTS sp_lab_first_divisible_number;

DELIMITER $$

CREATE PROCEDURE sp_lab_first_divisible_number(
    IN p_start_value INT,
    IN p_divisor INT
)
BEGIN
    DECLARE v_current INT DEFAULT 0;
    DECLARE v_message VARCHAR(100) DEFAULT '';

    SET v_current = p_start_value;

    IF p_divisor = 0 THEN
        SET v_message = 'Divisor cannot be zero';
    ELSE
        find_number: LOOP
            IF MOD(v_current, p_divisor) = 0 THEN
                LEAVE find_number;
            END IF;

            SET v_current = v_current + 1;
        END LOOP find_number;

        SET v_message = 'First divisible number found';
    END IF;

    SELECT p_start_value AS startValue,
           p_divisor AS divisorValue,
           v_current AS firstDivisibleValue,
           v_message AS message;
END$$

DELIMITER ;
```

Gọi procedure:

```sql
CALL sp_lab_first_divisible_number(17, 5);
```

Kết quả mong đợi:

```text
firstDivisibleValue = 20
```

### 12.4. `LEAVE` để thoát block ngoài cùng

Một block `BEGIN ... END` cũng có thể có label.

```sql
DROP PROCEDURE IF EXISTS sp_lab_leave_block_demo;

DELIMITER $$

CREATE PROCEDURE sp_lab_leave_block_demo(
    IN p_customer_number INT
)
main_block: BEGIN
    DECLARE v_customer_count INT DEFAULT 0;

    SELECT COUNT(*)
    INTO v_customer_count
    FROM customers
    WHERE customerNumber = p_customer_number;

    IF v_customer_count = 0 THEN
        SELECT 'Customer not found' AS message;
        LEAVE main_block;
    END IF;

    SELECT customerNumber, customerName, country, creditLimit
    FROM customers
    WHERE customerNumber = p_customer_number;
END$$

DELIMITER ;
```

Gọi procedure:

```sql
CALL sp_lab_leave_block_demo(103);
```

```sql
CALL sp_lab_leave_block_demo(999999);
```

### 12.5. `LEAVE` và `RETURN`

| Câu lệnh | Dùng cho |
|---|---|
| `LEAVE` | Procedure, function, trigger, event; thoát khỏi label |
| `RETURN` | Stored function; trả giá trị và kết thúc function |

Trong stored procedure, dùng `LEAVE` để thoát sớm khỏi block có nhãn; không dùng `RETURN`.

### Bài tập thực hành

**Bài 12.1.**

Tạo procedure `sp_lab_first_multiple_of_three` có tham số `IN p_start INT`; dùng `LOOP` và `LEAVE` để tìm số đầu tiên từ `p_start` trở đi chia hết cho 3.

**Bài 12.2.**

Tạo procedure `sp_lab_first_product_above_price` có tham số `IN p_start_price DECIMAL(10,2)`. Không cần cursor; procedure chỉ trả về message mô tả yêu cầu và giá trị input, sau đó giải thích vì sao muốn duyệt từng product row thực sự thì cần cursor ở phần sau.

**Bài 12.3.**

Tạo procedure `sp_lab_leave_block_if_customer_missing` có tham số `IN p_customer_number INT`; nếu không tìm thấy khách hàng thì `SELECT 'Not found'` và `LEAVE` block ngoài cùng.

**Bài 12.4.**

Sửa `sp_lab_first_divisible_number` để trả về message `Invalid divisor` và thoát block sớm nếu `p_divisor <= 0`.

**Bài 12.5.**

Giải thích vì sao `LEAVE some_loop` cần một label tồn tại và phù hợp.

---

# Phần C. So sánh và áp dụng

## 13. Chọn cấu trúc phù hợp

### 13.1. Bảng quyết định

| Yêu cầu | Cấu trúc phù hợp |
|---|---|
| Thực hiện câu lệnh khi điều kiện đúng | `IF ... THEN` |
| Chọn một nhánh theo đúng một giá trị | Simple `CASE` |
| Phân loại theo nhiều khoảng/điều kiện | Searched `CASE` |
| Lặp và tự kiểm soát điều kiện thoát | `LOOP` + `LEAVE` |
| Chỉ lặp nếu điều kiện đầu tiên đúng | `WHILE` |
| Bắt buộc thực hiện ít nhất một lần | `REPEAT` |
| Dừng ngay một loop hoặc block có nhãn | `LEAVE` |

### 13.2. Lựa chọn theo ví dụ

| Bài toán | Đề xuất |
|---|---|
| Phân loại status `Shipped`, `On Hold`, `Cancelled` | Simple `CASE` |
| Phân loại MSRP theo các mức giá | Searched `CASE` |
| Tính giai thừa khi `n >= 0` | `WHILE` |
| Lặp đến khi đạt mục tiêu | `LOOP` + `LEAVE` hoặc `REPEAT` |
| Bắt buộc chạy một thao tác xác nhận ít nhất một lần | `REPEAT` |
| Kết thúc procedure sớm khi customer không tồn tại | Labeled block + `LEAVE` |

### Bài tập thực hành

**Bài 13.1.**

Chọn cấu trúc phù hợp để phân loại trạng thái đơn hàng thành 5 nhãn cụ thể. Giải thích lựa chọn.

**Bài 13.2.**

Chọn cấu trúc phù hợp để phân loại hạn mức tín dụng theo ba khoảng. Giải thích lựa chọn.

**Bài 13.3.**

Chọn cấu trúc phù hợp để tính tổng từ 1 đến `n`, với yêu cầu không chạy khi `n < 1`.

**Bài 13.4.**

Chọn cấu trúc phù hợp để thực hiện một thao tác ít nhất một lần trước khi kiểm tra điều kiện dừng.

**Bài 13.5.**

Chọn cấu trúc phù hợp để thoát sớm khỏi procedure khi đầu vào không hợp lệ.

---

## 14. Ví dụ tổng hợp: đánh giá khách hàng

Procedure sau kết hợp:

- `IF` để kiểm tra khách hàng có tồn tại hay không.
- Searched `CASE` để phân loại hạn mức tín dụng.
- Truy vấn tổng hợp để đếm đơn hàng và thanh toán.

```sql
DROP PROCEDURE IF EXISTS sp_lab_customer_profile_summary;

DELIMITER $$

CREATE PROCEDURE sp_lab_customer_profile_summary(
    IN p_customer_number INT
)
main_block: BEGIN
    DECLARE v_customer_count INT DEFAULT 0;
    DECLARE v_customer_name VARCHAR(100) DEFAULT '';
    DECLARE v_credit_limit DECIMAL(10,2) DEFAULT 0.00;
    DECLARE v_order_count INT DEFAULT 0;
    DECLARE v_payment_count INT DEFAULT 0;
    DECLARE v_credit_category VARCHAR(30) DEFAULT '';

    SELECT COUNT(*),
           COALESCE(MAX(customerName), ''),
           COALESCE(MAX(creditLimit), 0)
    INTO v_customer_count, v_customer_name, v_credit_limit
    FROM customers
    WHERE customerNumber = p_customer_number;

    IF v_customer_count = 0 THEN
        SELECT p_customer_number AS customerNumber,
               'Customer not found' AS message;
        LEAVE main_block;
    END IF;

    SELECT COUNT(*)
    INTO v_order_count
    FROM orders
    WHERE customerNumber = p_customer_number;

    SELECT COUNT(*)
    INTO v_payment_count
    FROM payments
    WHERE customerNumber = p_customer_number;

    CASE
        WHEN v_credit_limit >= 100000.00 THEN
            SET v_credit_category = 'High';
        WHEN v_credit_limit >= 50000.00 THEN
            SET v_credit_category = 'Medium';
        ELSE
            SET v_credit_category = 'Low';
    END CASE;

    SELECT p_customer_number AS customerNumber,
           v_customer_name AS customerName,
           v_credit_limit AS creditLimit,
           v_credit_category AS creditCategory,
           v_order_count AS totalOrders,
           v_payment_count AS totalPayments;
END$$

DELIMITER ;
```

Gọi procedure:

```sql
CALL sp_lab_customer_profile_summary(103);
```

### Bài tập thực hành

**Bài 14.1.**

Tạo procedure `sp_lab_customer_profile_summary` theo ví dụ.

**Bài 14.2.**

Gọi procedure cho khách hàng `103`.

**Bài 14.3.**

Gọi procedure cho khách hàng `112`.

**Bài 14.4.**

Gọi procedure cho mã khách hàng không tồn tại, ví dụ `999999`.

**Bài 14.5.**

Sửa procedure để thêm một cột `customerActivity`: `Active` nếu có ít nhất một đơn hàng, ngược lại `No orders`.

---

## 15. Một số lỗi thường gặp

### Lỗi 1. Quên `THEN` hoặc `END IF`

Sai:

```sql
IF v_total > 0
    SET v_message = 'Positive';
END IF;
```

Đúng:

```sql
IF v_total > 0 THEN
    SET v_message = 'Positive';
END IF;
```

---

### Lỗi 2. Dùng `END` thay cho `END IF`

Sai:

```sql
IF v_total > 0 THEN
    SET v_message = 'Positive';
END;
```

Đúng:

```sql
IF v_total > 0 THEN
    SET v_message = 'Positive';
END IF;
```

---

### Lỗi 3. Quên `END CASE`

Sai:

```sql
CASE v_status
    WHEN 'Shipped' THEN
        SET v_message = 'Done';
END;
```

Đúng:

```sql
CASE v_status
    WHEN 'Shipped' THEN
        SET v_message = 'Done';
    ELSE
        SET v_message = 'Other';
END CASE;
```

---

### Lỗi 4. Tạo `LOOP` không có điều kiện dừng

Sai:

```sql
infinite_loop: LOOP
    SET v_i = v_i + 1;
END LOOP infinite_loop;
```

Câu lệnh trên có thể chạy vô hạn.

Đúng:

```sql
controlled_loop: LOOP
    SET v_i = v_i + 1;

    IF v_i >= 10 THEN
        LEAVE controlled_loop;
    END IF;
END LOOP controlled_loop;
```

---

### Lỗi 5. Nhầm điều kiện `UNTIL` trong `REPEAT`

Sai về logic nếu mục tiêu là tiếp tục đến khi `v_i > p_n`:

```sql
REPEAT
    SET v_i = v_i + 1;
UNTIL v_i <= p_n
END REPEAT;
```

Đúng:

```sql
REPEAT
    SET v_i = v_i + 1;
UNTIL v_i > p_n
END REPEAT;
```

---

### Lỗi 6. Dùng `RETURN` trong stored procedure

`RETURN` dùng trong stored function để trả giá trị. Trong stored procedure, muốn thoát sớm khỏi block cần dùng `LEAVE` với label phù hợp.

### Bài tập thực hành

**Bài 15.1.**

Sửa lỗi cú pháp thiếu `THEN` trong một câu `IF`.

**Bài 15.2.**

Sửa lỗi dùng `END` thay vì `END IF`.

**Bài 15.3.**

Sửa một simple `CASE` thiếu `ELSE` và `END CASE`.

**Bài 15.4.**

Sửa một `LOOP` để có điều kiện thoát bằng `LEAVE`.

**Bài 15.5.**

Giải thích vì sao không dùng `RETURN` để kết thúc stored procedure.

---

## 16. Bài tập tổng hợp

### Bài 16.1. Phân loại khách hàng

Tạo procedure `sp_lab_customer_credit_summary`:

1. Có tham số `IN p_customer_number INT`.
2. Kiểm tra khách hàng tồn tại bằng `IF`.
3. Nếu tồn tại, lấy `customerName` và `creditLimit`.
4. Dùng searched `CASE` để phân loại credit limit: `High`, `Medium`, `Low`.
5. Trả về mã, tên, hạn mức và nhãn phân loại.

---

### Bài 16.2. Phân loại sản phẩm

Tạo procedure `sp_lab_product_summary`:

1. Có tham số `IN p_product_code VARCHAR(15)`.
2. Kiểm tra sản phẩm tồn tại.
3. Lấy `productName`, `productLine`, `buyPrice`, `MSRP`.
4. Dùng simple `CASE` để phân loại nhóm sản phẩm cho ít nhất ba `productLine`.
5. Dùng searched `CASE` để phân loại biên lợi nhuận `MSRP - buyPrice`.

---

### Bài 16.3. Tính tổng bằng ba vòng lặp

Tạo ba procedure cùng tính tổng từ 1 đến `p_n`:

1. `sp_lab_sum_loop`.
2. `sp_lab_sum_while`.
3. `sp_lab_sum_repeat`.

Yêu cầu:

- Mỗi procedure có tham số `IN p_n INT`.
- Trả về `p_n`, tổng tính được và message.
- Xử lý trường hợp `p_n < 1`.
- So sánh hành vi khi `p_n = 0`.
- Giải thích cấu trúc nào phù hợp nhất với từng tình huống.

---

### Bài 16.4. Tìm bội số đầu tiên

Tạo procedure `sp_lab_first_multiple`:

1. Có tham số `IN p_start INT` và `IN p_divisor INT`.
2. Dùng `LOOP`.
3. Dùng `LEAVE` để thoát ngay khi tìm thấy số hợp lệ.
4. Xử lý `p_divisor <= 0` bằng `IF` và labeled block.
5. Trả về số tìm được và message.

---

### Bài 16.5. Tóm tắt đơn hàng

Tạo procedure `sp_lab_order_summary_with_case`:

1. Có tham số `IN p_order_number INT`.
2. Kiểm tra order tồn tại.
3. Lấy `status`.
4. Tính tổng giá trị đơn hàng bằng `SUM(quantityOrdered * priceEach)`.
5. Dùng simple `CASE` để diễn giải status và searched `CASE` để phân loại giá trị đơn hàng.

---

## 17. Đáp án gợi ý cho một số bài tập

### Bài 4.5

```sql
DROP PROCEDURE IF EXISTS sp_lab_customer_exists;

DELIMITER $$

CREATE PROCEDURE sp_lab_customer_exists(
    IN p_customer_number INT
)
BEGIN
    DECLARE v_customer_count INT DEFAULT 0;
    DECLARE v_message VARCHAR(50) DEFAULT '';

    SELECT COUNT(*)
    INTO v_customer_count
    FROM customers
    WHERE customerNumber = p_customer_number;

    IF v_customer_count = 1 THEN
        SET v_message = 'Found';
    ELSE
        SET v_message = 'Not found';
    END IF;

    SELECT p_customer_number AS customerNumber,
           v_message AS message;
END$$

DELIMITER ;

CALL sp_lab_customer_exists(103);
```

### Bài 6.1

```sql
DROP PROCEDURE IF EXISTS sp_lab_status_label;

DELIMITER $$

CREATE PROCEDURE sp_lab_status_label(
    IN p_order_number INT
)
BEGIN
    DECLARE v_status VARCHAR(15) DEFAULT '';
    DECLARE v_message VARCHAR(100) DEFAULT '';

    SELECT COALESCE(MAX(status), '')
    INTO v_status
    FROM orders
    WHERE orderNumber = p_order_number;

    CASE v_status
        WHEN 'Shipped' THEN
            SET v_message = 'Completed shipment';
        WHEN 'In Process' THEN
            SET v_message = 'Processing';
        WHEN 'Cancelled' THEN
            SET v_message = 'Cancelled';
        WHEN 'On Hold' THEN
            SET v_message = 'Waiting';
        ELSE
            SET v_message = 'Other or not found';
    END CASE;

    SELECT p_order_number AS orderNumber,
           v_status AS status,
           v_message AS statusLabel;
END$$

DELIMITER ;
```

### Bài 7.1

```sql
DROP PROCEDURE IF EXISTS sp_lab_credit_limit_category;

DELIMITER $$

CREATE PROCEDURE sp_lab_credit_limit_category(
    IN p_customer_number INT
)
BEGIN
    DECLARE v_credit_limit DECIMAL(10,2) DEFAULT 0.00;
    DECLARE v_category VARCHAR(30) DEFAULT '';

    SELECT COALESCE(MAX(creditLimit), 0)
    INTO v_credit_limit
    FROM customers
    WHERE customerNumber = p_customer_number;

    CASE
        WHEN v_credit_limit >= 100000.00 THEN
            SET v_category = 'High';
        WHEN v_credit_limit >= 50000.00 THEN
            SET v_category = 'Medium';
        ELSE
            SET v_category = 'Low';
    END CASE;

    SELECT p_customer_number AS customerNumber,
           v_credit_limit AS creditLimit,
           v_category AS creditCategory;
END$$

DELIMITER ;
```

### Bài 9.1

```sql
DROP PROCEDURE IF EXISTS sp_lab_sum_even_numbers_loop;

DELIMITER $$

CREATE PROCEDURE sp_lab_sum_even_numbers_loop(
    IN p_n INT
)
BEGIN
    DECLARE v_i INT DEFAULT 2;
    DECLARE v_total BIGINT DEFAULT 0;

    IF p_n >= 2 THEN
        even_loop: LOOP
            SET v_total = v_total + v_i;

            IF v_i + 2 > p_n THEN
                LEAVE even_loop;
            END IF;

            SET v_i = v_i + 2;
        END LOOP even_loop;
    END IF;

    SELECT p_n AS n,
           v_total AS evenNumberSum;
END$$

DELIMITER ;
```

### Bài 10.1

```sql
DROP PROCEDURE IF EXISTS sp_lab_sum_numbers_while;

DELIMITER $$

CREATE PROCEDURE sp_lab_sum_numbers_while(
    IN p_n INT
)
BEGIN
    DECLARE v_i INT DEFAULT 1;
    DECLARE v_total BIGINT DEFAULT 0;

    WHILE v_i <= p_n DO
        SET v_total = v_total + v_i;
        SET v_i = v_i + 1;
    END WHILE;

    SELECT p_n AS n,
           v_total AS total;
END$$

DELIMITER ;
```

### Bài 11.1

```sql
DROP PROCEDURE IF EXISTS sp_lab_sum_even_numbers_repeat;

DELIMITER $$

CREATE PROCEDURE sp_lab_sum_even_numbers_repeat(
    IN p_n INT
)
BEGIN
    DECLARE v_i INT DEFAULT 2;
    DECLARE v_total BIGINT DEFAULT 0;

    IF p_n >= 2 THEN
        REPEAT
            SET v_total = v_total + v_i;
            SET v_i = v_i + 2;
        UNTIL v_i > p_n
        END REPEAT;
    END IF;

    SELECT p_n AS n,
           v_total AS evenNumberSum;
END$$

DELIMITER ;
```

### Bài 12.1

```sql
DROP PROCEDURE IF EXISTS sp_lab_first_multiple_of_three;

DELIMITER $$

CREATE PROCEDURE sp_lab_first_multiple_of_three(
    IN p_start INT
)
BEGIN
    DECLARE v_current INT DEFAULT 0;

    SET v_current = p_start;

    multiple_loop: LOOP
        IF MOD(v_current, 3) = 0 THEN
            LEAVE multiple_loop;
        END IF;

        SET v_current = v_current + 1;
    END LOOP multiple_loop;

    SELECT p_start AS startValue,
           v_current AS firstMultipleOfThree;
END$$

DELIMITER ;
```

### Bài 16.5

```sql
DROP PROCEDURE IF EXISTS sp_lab_order_summary_with_case;

DELIMITER $$

CREATE PROCEDURE sp_lab_order_summary_with_case(
    IN p_order_number INT
)
main_block: BEGIN
    DECLARE v_order_count INT DEFAULT 0;
    DECLARE v_status VARCHAR(15) DEFAULT '';
    DECLARE v_total_value DECIMAL(14,2) DEFAULT 0.00;
    DECLARE v_status_message VARCHAR(100) DEFAULT '';
    DECLARE v_value_category VARCHAR(30) DEFAULT '';

    SELECT COUNT(*), COALESCE(MAX(status), '')
    INTO v_order_count, v_status
    FROM orders
    WHERE orderNumber = p_order_number;

    IF v_order_count = 0 THEN
        SELECT p_order_number AS orderNumber,
               'Order not found' AS message;
        LEAVE main_block;
    END IF;

    SELECT COALESCE(SUM(quantityOrdered * priceEach), 0)
    INTO v_total_value
    FROM orderdetails
    WHERE orderNumber = p_order_number;

    CASE v_status
        WHEN 'Shipped' THEN
            SET v_status_message = 'Order shipped';
        WHEN 'In Process' THEN
            SET v_status_message = 'Order in process';
        WHEN 'On Hold' THEN
            SET v_status_message = 'Order on hold';
        WHEN 'Cancelled' THEN
            SET v_status_message = 'Order cancelled';
        ELSE
            SET v_status_message = 'Other status';
    END CASE;

    CASE
        WHEN v_total_value >= 100000.00 THEN
            SET v_value_category = 'Large';
        WHEN v_total_value >= 25000.00 THEN
            SET v_value_category = 'Medium';
        ELSE
            SET v_value_category = 'Small';
    END CASE;

    SELECT p_order_number AS orderNumber,
           v_status AS status,
           v_status_message AS statusMessage,
           v_total_value AS totalValue,
           v_value_category AS valueCategory;
END$$

DELIMITER ;
```

---

## 18. Tóm tắt

Các kiến thức chính trong lab:

- `IF ... THEN ... END IF` thực hiện câu lệnh khi điều kiện đúng.
- `ELSEIF` và `ELSE` cho phép xử lý nhiều nhánh.
- Simple `CASE` so sánh một giá trị với nhiều giá trị cụ thể.
- Searched `CASE` kiểm tra nhiều điều kiện logic, đặc biệt phù hợp với các khoảng giá trị.
- `CASE statement` trong procedure khác với `CASE expression` trong câu `SELECT`.
- `LOOP` không tự có điều kiện dừng; thường dùng `IF` và `LEAVE`.
- `WHILE` kiểm tra điều kiện trước mỗi lần lặp nên có thể chạy 0 lần.
- `REPEAT` kiểm tra điều kiện sau thân vòng lặp nên luôn chạy ít nhất 1 lần.
- `LEAVE` thoát khỏi một loop hoặc block có nhãn; trong stored procedure không dùng `RETURN` để thoát sớm.
- Vòng lặp thường được kết hợp với cursor khi cần xử lý từng row của result set, nội dung này sẽ được học trong phần tiếp theo.

---

## 19. Từ khóa chính

- Stored Procedure
- IF statement
- ELSEIF
- ELSE
- CASE statement
- Simple CASE
- Searched CASE
- CASE expression
- LOOP
- WHILE
- REPEAT
- UNTIL
- LEAVE
- Label
- BEGIN
- END
- Local Variable
- DELIMITER
- classicmodels
