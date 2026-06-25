---
title: "Lab: Stored Functions trong MySQL với classicmodels"
author: "Tên giảng viên"
duration: "150m"
difficulty: "Intermediate"
prerequisites:
  - "Đã hoàn thành lab Stored Procedures cơ bản"
  - "Đã biết DELIMITER, CREATE PROCEDURE, biến cục bộ, IF và SELECT ... INTO"
  - "Đã biết SELECT, JOIN, GROUP BY, hàm tổng hợp và biểu thức SQL"
  - "Có quyền CREATE ROUTINE và EXECUTE trên cơ sở dữ liệu thực hành"
summary: "Thực hành tạo, sử dụng, xóa và liệt kê stored functions trong MySQL bằng cơ sở dữ liệu classicmodels."
---

# Lab: Stored Functions trong MySQL với `classicmodels`

## Link tham khảo

- [MySQL Tutorial — Stored Procedures and Stored Functions](https://www.mysqltutorial.org/mysql-stored-procedure/)
- [MySQL Tutorial — Creating Stored Functions](https://www.mysqltutorial.org/mysql-stored-procedure/mysql-stored-function/)
- [MySQL Tutorial — DROP FUNCTION](https://www.mysqltutorial.org/mysql-stored-procedure/mysql-drop-function/)
- [MySQL Tutorial — Listing Stored Functions](https://www.mysqltutorial.org/mysql-stored-procedure/mysql-show-functions/)


## 1. Mục tiêu học tập

Sau khi hoàn thành lab này, người học có thể:

1. Giải thích stored function là gì và phân biệt được stored function với stored procedure.
2. Tạo một stored function bằng `CREATE FUNCTION`.
3. Khai báo đúng `RETURNS data_type` và dùng `RETURN` để trả về một giá trị.
4. Dùng stored function trong biểu thức SQL, `SELECT`, `WHERE`, `ORDER BY` hoặc `HAVING` khi phù hợp.
5. Chọn và khai báo các characteristics như `DETERMINISTIC`, `NO SQL`, `READS SQL DATA`.
6. Viết stored function dùng biến cục bộ, `IF` và `SELECT ... INTO`.
7. Xóa stored function bằng `DROP FUNCTION`.
8. Liệt kê stored function bằng `SHOW FUNCTION STATUS` và `INFORMATION_SCHEMA.ROUTINES`.
9. Xem definition bằng `SHOW CREATE FUNCTION`.
10. Nhận biết các lỗi thường gặp và các hạn chế cơ bản của stored function.

---

## 2. Stored function là gì?

**Stored function** là một routine SQL được lưu trên MySQL Server, nhận các giá trị đầu vào và trả về **một giá trị duy nhất**.

Ví dụ, ta có thể tạo function tính tổng giá trị của một đơn hàng:

```sql
SELECT fn_lab_order_total(10100) AS totalOrderValue;
```

Stored function được gọi trong một **biểu thức SQL**, tương tự các hàm dựng sẵn như:

```sql
ROUND(...)
COALESCE(...)
CONCAT(...)
NOW()
```

### 2.1. Stored function và stored procedure

| Tiêu chí | Stored Function | Stored Procedure |
|---|---|---|
| Cách gọi | Gọi trong biểu thức SQL | Gọi bằng `CALL` |
| Giá trị trả về | Bắt buộc trả về một giá trị | Có thể trả result set hoặc dùng `OUT`/`INOUT` |
| `RETURNS` | Bắt buộc | Không dùng |
| `RETURN` | Dùng để trả giá trị | Không dùng để trả kết quả |
| Tham số | Chỉ là input | Có thể là `IN`, `OUT`, `INOUT` |
| Dùng trong `SELECT` | Có | Không trực tiếp |
| Mục tiêu phổ biến | Tính một giá trị có thể tái sử dụng | Thực hiện một chuỗi thao tác hoặc trả result set |

Ví dụ gọi procedure:

```sql
CALL sp_lab_list_customers_by_country('USA');
```

Ví dụ gọi function:

```sql
SELECT fn_lab_order_total(10100);
```

### 2.2. Khi nào nên dùng stored function?

Stored function phù hợp khi cần:

- Tính một giá trị đơn lẻ.
- Dùng lại cùng một công thức ở nhiều truy vấn.
- Chuẩn hóa logic tính toán đơn giản.
- Tạo một giá trị có thể được gọi trực tiếp trong `SELECT`.

Ví dụ:

- Tổng giá trị một đơn hàng.
- Chênh lệch giữa MSRP và buy price.
- Phân loại một mức giá.
- Tính giá sau chiết khấu.
- Chuyển một trạng thái thành nhãn mô tả.

### 2.3. Khi nào nên dùng procedure thay vì function?

Dùng procedure khi cần:

- Trả về nhiều dòng hoặc nhiều result set.
- Thực hiện quy trình gồm nhiều bước.
- Có tham số `OUT` hoặc `INOUT`.
- Thực hiện thao tác DML phức tạp theo quy trình.
- Điều khiển transaction hoặc xử lý nghiệp vụ phức tạp.

> Quy tắc đơn giản:  
> **Function trả một giá trị để dùng trong biểu thức. Procedure thực hiện một tác vụ hoặc quy trình.**

### Bài tập thực hành

**Bài 2.1.**

Giải thích stored function là gì bằng một hoặc hai câu.

**Bài 2.2.**

Nêu hai khác biệt giữa stored function và stored procedure.

**Bài 2.3.**

Cho biết trường hợp nào phù hợp hơn với stored function: trả về tổng giá trị của một order, hay trả về danh sách toàn bộ order của một customer. Giải thích.

**Bài 2.4.**

Viết một ví dụ cách gọi stored function trong `SELECT`.

**Bài 2.5.**

Nêu hai nghiệp vụ trong `classicmodels` có thể đóng gói thành stored function.

---

## 3. Chuẩn bị môi trường

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
SELECT orderNumber, productCode, quantityOrdered, priceEach
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

Kiểm tra các quyền hiện có khi cần:

```sql
SHOW GRANTS;
```

Trong lab này, stored function được đặt tên với tiền tố:

```text
fn_lab_
```

Ví dụ:

```text
fn_lab_order_total
fn_lab_product_margin
fn_lab_discounted_price
fn_lab_credit_category
```

> Tiền tố `fn_` giúp phân biệt stored function với procedure `sp_`, view `vw_` và bảng thông thường.

### Bài tập thực hành

**Bài 3.1.**

Viết lệnh chọn database `classicmodels`.

**Bài 3.2.**

Viết truy vấn xem các dòng chi tiết của đơn hàng `10100`.

**Bài 3.3.**

Viết truy vấn xem `productCode`, `productName`, `buyPrice`, `MSRP` của năm sản phẩm đầu tiên.

**Bài 3.4.**

Viết lệnh kiểm tra quyền của tài khoản đang đăng nhập.

**Bài 3.5.**

Giải thích ý nghĩa của tiền tố `fn_lab_`.

---

## 4. Tạo stored function bằng `CREATE FUNCTION`

### 4.1. Cú pháp cơ bản

```sql
CREATE FUNCTION function_name (
    parameter_1 data_type,
    parameter_2 data_type
)
RETURNS return_data_type
[characteristic ...]
routine_body;
```

Các phần quan trọng:

| Thành phần | Ý nghĩa |
|---|---|
| `CREATE FUNCTION` | Tạo stored function mới |
| `function_name` | Tên function |
| `(parameter ...)` | Danh sách tham số đầu vào |
| `RETURNS data_type` | Kiểu dữ liệu của giá trị function trả về |
| `characteristic` | Đặc tính như `DETERMINISTIC`, `NO SQL`, `READS SQL DATA` |
| `routine_body` | Thân function, thường có `RETURN` |

Ví dụ function đơn giản tính giá sau giảm giá:

```sql
DROP FUNCTION IF EXISTS fn_lab_discounted_price;

DELIMITER $$

CREATE FUNCTION fn_lab_discounted_price(
    p_price DECIMAL(10,2),
    p_discount_percent DECIMAL(5,2)
)
RETURNS DECIMAL(10,2)
DETERMINISTIC
NO SQL
BEGIN
    RETURN p_price * (1 - p_discount_percent / 100);
END$$

DELIMITER ;
```

Gọi function:

```sql
SELECT fn_lab_discounted_price(200.00, 15.00) AS discountedPrice;
```

Kết quả mong đợi:

```text
170.00
```

### 4.2. Function không có tham số

Danh sách dấu ngoặc vẫn bắt buộc, dù không có tham số.

```sql
DROP FUNCTION IF EXISTS fn_lab_current_year;

DELIMITER $$

CREATE FUNCTION fn_lab_current_year()
RETURNS INT
NOT DETERMINISTIC
NO SQL
BEGIN
    RETURN YEAR(CURRENT_DATE());
END$$

DELIMITER ;
```

Gọi function:

```sql
SELECT fn_lab_current_year() AS currentYear;
```

### 4.3. Vì sao cần thay delimiter?

Nếu function có block `BEGIN ... END` và nhiều câu lệnh, phải đổi delimiter để client không kết thúc `CREATE FUNCTION` tại dấu `;` bên trong function.

Mẫu chung:

```sql
DELIMITER $$

CREATE FUNCTION function_name(...)
RETURNS data_type
BEGIN
    -- statements;
    RETURN value;
END$$

DELIMITER ;
```

### 4.4. Function phải có `RETURNS`

Khác với procedure, function phải khai báo kiểu dữ liệu trả về.

Sai:

```sql
CREATE FUNCTION fn_lab_bad_example(
    p_value INT
)
BEGIN
    RETURN p_value * 2;
END;
```

Đúng:

```sql
CREATE FUNCTION fn_lab_good_example(
    p_value INT
)
RETURNS INT
DETERMINISTIC
NO SQL
BEGIN
    RETURN p_value * 2;
END;
```

### Bài tập thực hành

**Bài 4.1.**

Tạo function `fn_lab_double_number` nhận một số nguyên và trả về gấp đôi số đó.

**Bài 4.2.**

Tạo function `fn_lab_add_numbers` nhận hai số nguyên và trả về tổng của chúng.

**Bài 4.3.**

Tạo function `fn_lab_price_after_tax` nhận giá và phần trăm thuế, trả về giá sau thuế.

**Bài 4.4.**

Tạo function `fn_lab_greeting` nhận tên `VARCHAR(50)` và trả về chuỗi `Hello, <name>`.

**Bài 4.5.**

Tạo function không có tham số trả về ngày hiện tại dưới dạng `DATE`.

---

## 5. `RETURN`, biến cục bộ và tham số trong stored function

### 5.1. `RETURN`

Stored function phải trả về một giá trị bằng `RETURN`.

Ví dụ:

```sql
RETURN p_price * (1 - p_discount_percent / 100);
```

Có thể trả về:

- Hằng số.
- Biểu thức.
- Biến cục bộ.
- Giá trị được tính từ truy vấn.

### 5.2. Tham số function chỉ là input

Stored function không dùng `IN`, `OUT`, `INOUT`.

Cú pháp function parameter:

```sql
parameter_name data_type
```

Ví dụ:

```sql
CREATE FUNCTION fn_lab_square(
    p_number INT
)
RETURNS INT
DETERMINISTIC
NO SQL
RETURN p_number * p_number;
```

Không viết:

```sql
CREATE FUNCTION fn_lab_bad_square(
    IN p_number INT
)
RETURNS INT
...
```

### 5.3. Dùng biến cục bộ

Ví dụ tính phần trăm chênh lệch giá:

```sql
DROP FUNCTION IF EXISTS fn_lab_margin_percent;

DELIMITER $$

CREATE FUNCTION fn_lab_margin_percent(
    p_buy_price DECIMAL(10,2),
    p_msrp DECIMAL(10,2)
)
RETURNS DECIMAL(8,2)
DETERMINISTIC
NO SQL
BEGIN
    DECLARE v_margin_percent DECIMAL(8,2) DEFAULT 0.00;

    IF p_buy_price IS NULL
       OR p_buy_price <= 0
       OR p_msrp IS NULL THEN
        RETURN NULL;
    END IF;

    SET v_margin_percent =
        (p_msrp - p_buy_price) / p_buy_price * 100;

    RETURN v_margin_percent;
END$$

DELIMITER ;
```

Gọi function:

```sql
SELECT fn_lab_margin_percent(50.00, 75.00) AS marginPercent;
```

### 5.4. Lưu ý về `NULL`

Nếu input có thể là `NULL`, cần xác định quy tắc rõ ràng:

- Trả `NULL`.
- Trả giá trị mặc định.
- Trả mã lỗi hoặc nhãn đặc biệt.

Ví dụ dùng `COALESCE` để gán giá trị thay thế:

```sql
RETURN COALESCE(p_value, 0);
```

Tuy nhiên, chỉ nên thay `NULL` bằng `0` khi điều đó hợp lý về mặt nghiệp vụ.

### Bài tập thực hành

**Bài 5.1.**

Tạo function `fn_lab_absolute_value` nhận một số nguyên và trả về giá trị tuyệt đối của nó bằng `IF`.

**Bài 5.2.**

Tạo function `fn_lab_price_difference` nhận `buyPrice` và `MSRP`, trả về `MSRP - buyPrice`.

**Bài 5.3.**

Tạo function `fn_lab_safe_divide` nhận tử số và mẫu số. Nếu mẫu số bằng `0`, trả về `NULL`; ngược lại trả về thương.

**Bài 5.4.**

Tạo function `fn_lab_customer_label` nhận `customerNumber` và trả về chuỗi `Customer-<customerNumber>`.

**Bài 5.5.**

Giải thích vì sao function parameter không có `OUT` hoặc `INOUT`.

---

## 6. Function characteristics

Khi tạo function, có thể khai báo các đặc tính mô tả hành vi của function.

### 6.1. `DETERMINISTIC` và `NOT DETERMINISTIC`

```sql
DETERMINISTIC
```

nghĩa là function được khai báo là luôn trả về cùng kết quả khi input giống nhau.

Ví dụ:

```sql
CREATE FUNCTION fn_lab_square(
    p_number INT
)
RETURNS INT
DETERMINISTIC
NO SQL
RETURN p_number * p_number;
```

Với `p_number = 5`, kết quả luôn là `25`.

```sql
NOT DETERMINISTIC
```

dùng khi kết quả có thể thay đổi dù input không đổi.

Ví dụ:

```sql
CREATE FUNCTION fn_lab_current_timestamp_text()
RETURNS VARCHAR(30)
NOT DETERMINISTIC
NO SQL
RETURN DATE_FORMAT(NOW(), '%Y-%m-%d %H:%i:%s');
```

Kết quả phụ thuộc vào thời điểm gọi function.

> `DETERMINISTIC` là một khai báo đặc tính; người viết function phải bảo đảm khai báo phản ánh đúng logic. MySQL không tự chứng minh function có thực sự deterministic hay không.

### 6.2. `NO SQL`, `READS SQL DATA`, `MODIFIES SQL DATA`, `CONTAINS SQL`

| Characteristic | Ý nghĩa |
|---|---|
| `NO SQL` | Function không chứa câu SQL truy cập dữ liệu |
| `READS SQL DATA` | Function đọc dữ liệu bằng SQL |
| `MODIFIES SQL DATA` | Function có thao tác thay đổi dữ liệu |
| `CONTAINS SQL` | Function có SQL nhưng không được khai báo cụ thể là đọc hay thay đổi dữ liệu |

Ví dụ function chỉ tính toán:

```sql
CREATE FUNCTION fn_lab_square(
    p_number INT
)
RETURNS INT
DETERMINISTIC
NO SQL
RETURN p_number * p_number;
```

Ví dụ function đọc dữ liệu từ `orderdetails`:

```sql
CREATE FUNCTION fn_lab_order_total(
    p_order_number INT
)
RETURNS DECIMAL(14,2)
READS SQL DATA
...
```

### 6.3. Chọn characteristic phù hợp

| Function | Characteristic phù hợp |
|---|---|
| Tính bình phương | `DETERMINISTIC NO SQL` |
| Tính giá sau giảm giá | `DETERMINISTIC NO SQL` |
| Lấy tổng giá trị order từ bảng | `READS SQL DATA` |
| Trả năm hiện tại | `NOT DETERMINISTIC NO SQL` |
| Trả thời điểm hiện tại | `NOT DETERMINISTIC NO SQL` |

### 6.4. Lưu ý về môi trường server

Việc tạo function yêu cầu quyền `CREATE ROUTINE`. Trên một số server có binary logging, chính sách cấu hình và quyền có thể đặt thêm yêu cầu đối với việc tạo stored function.

Nếu `CREATE FUNCTION` bị từ chối dù cú pháp đúng:

1. Kiểm tra `SHOW GRANTS`.
2. Hỏi quản trị viên database.
3. Không tự thay đổi cấu hình server trên môi trường dùng chung hoặc production.

### Bài tập thực hành

**Bài 6.1.**

Phân loại function `fn_lab_double_number` nên dùng `DETERMINISTIC` hay `NOT DETERMINISTIC`.

**Bài 6.2.**

Phân loại function lấy `NOW()` nên dùng `DETERMINISTIC` hay `NOT DETERMINISTIC`.

**Bài 6.3.**

Chọn characteristic dữ liệu phù hợp cho function chỉ thực hiện phép tính trên tham số.

**Bài 6.4.**

Chọn characteristic dữ liệu phù hợp cho function đọc `creditLimit` từ bảng `customers`.

**Bài 6.5.**

Giải thích vì sao không nên khai báo `DETERMINISTIC` cho function trả về `RAND()`.

---

## 7. Tạo function đọc dữ liệu từ `classicmodels`

### 7.1. Function tính tổng giá trị đơn hàng

Function dưới đây nhận mã order và trả về:

```text
SUM(quantityOrdered * priceEach)
```

của các dòng `orderdetails`.

```sql
DROP FUNCTION IF EXISTS fn_lab_order_total;

DELIMITER $$

CREATE FUNCTION fn_lab_order_total(
    p_order_number INT
)
RETURNS DECIMAL(14,2)
READS SQL DATA
BEGIN
    DECLARE v_total DECIMAL(14,2) DEFAULT 0.00;

    SELECT COALESCE(SUM(quantityOrdered * priceEach), 0)
    INTO v_total
    FROM orderdetails
    WHERE orderNumber = p_order_number;

    RETURN v_total;
END$$

DELIMITER ;
```

Gọi function:

```sql
SELECT fn_lab_order_total(10100) AS totalOrderValue;
```

### 7.2. Dùng function trong truy vấn

Có thể dùng function trong danh sách cột:

```sql
SELECT orderNumber,
       orderDate,
       status,
       fn_lab_order_total(orderNumber) AS totalOrderValue
FROM orders
WHERE customerNumber = 103
ORDER BY totalOrderValue DESC;
```

### 7.3. So sánh với truy vấn không dùng function

Không dùng function:

```sql
SELECT o.orderNumber,
       o.orderDate,
       o.status,
       COALESCE(SUM(od.quantityOrdered * od.priceEach), 0) AS totalOrderValue
FROM orders AS o
LEFT JOIN orderdetails AS od
    ON od.orderNumber = o.orderNumber
WHERE o.customerNumber = 103
GROUP BY o.orderNumber, o.orderDate, o.status
ORDER BY totalOrderValue DESC;
```

Dùng function có thể làm câu truy vấn phía ngoài ngắn hơn, nhưng không phải lúc nào cũng có hiệu năng tốt hơn.

> Nếu gọi function đọc bảng cho rất nhiều dòng của một query, database có thể phải thực hiện nhiều lần logic bên trong function. Với báo cáo lớn, truy vấn set-based bằng `JOIN` và `GROUP BY` thường là lựa chọn cần cân nhắc trước.

### 7.4. Function trả về hạn mức tín dụng

```sql
DROP FUNCTION IF EXISTS fn_lab_customer_credit_limit;

DELIMITER $$

CREATE FUNCTION fn_lab_customer_credit_limit(
    p_customer_number INT
)
RETURNS DECIMAL(10,2)
READS SQL DATA
BEGIN
    DECLARE v_credit_limit DECIMAL(10,2) DEFAULT NULL;

    SELECT MAX(creditLimit)
    INTO v_credit_limit
    FROM customers
    WHERE customerNumber = p_customer_number;

    RETURN v_credit_limit;
END$$

DELIMITER ;
```

Gọi function:

```sql
SELECT fn_lab_customer_credit_limit(103) AS creditLimit;
```

Nếu customer không tồn tại, `MAX(creditLimit)` trả về `NULL`.

### Bài tập thực hành

**Bài 7.1.**

Tạo function `fn_lab_order_total` theo ví dụ.

**Bài 7.2.**

Gọi `fn_lab_order_total` cho các đơn hàng `10100`, `10101`, `10102`.

**Bài 7.3.**

Dùng `fn_lab_order_total(orderNumber)` để hiển thị tổng giá trị của các order thuộc customer `103`.

**Bài 7.4.**

Tạo function `fn_lab_order_line_count` nhận `orderNumber` và trả về số dòng trong `orderdetails`.

**Bài 7.5.**

Tạo function `fn_lab_product_margin` nhận `productCode` và trả về `MSRP - buyPrice` của sản phẩm đó.

---

## 8. Dùng `IF` và `CASE` trong stored function

### 8.1. Function phân loại hạn mức tín dụng

```sql
DROP FUNCTION IF EXISTS fn_lab_credit_category;

DELIMITER $$

CREATE FUNCTION fn_lab_credit_category(
    p_customer_number INT
)
RETURNS VARCHAR(30)
READS SQL DATA
BEGIN
    DECLARE v_credit_limit DECIMAL(10,2) DEFAULT NULL;
    DECLARE v_category VARCHAR(30) DEFAULT '';

    SELECT MAX(creditLimit)
    INTO v_credit_limit
    FROM customers
    WHERE customerNumber = p_customer_number;

    IF v_credit_limit IS NULL THEN
        SET v_category = 'Customer not found';
    ELSEIF v_credit_limit >= 100000.00 THEN
        SET v_category = 'High';
    ELSEIF v_credit_limit >= 50000.00 THEN
        SET v_category = 'Medium';
    ELSE
        SET v_category = 'Low';
    END IF;

    RETURN v_category;
END$$

DELIMITER ;
```

Gọi function:

```sql
SELECT customerNumber,
       customerName,
       creditLimit,
       fn_lab_credit_category(customerNumber) AS creditCategory
FROM customers
WHERE customerNumber IN (103, 112, 114);
```

### 8.2. Function phân loại giá sản phẩm bằng CASE

```sql
DROP FUNCTION IF EXISTS fn_lab_product_price_category;

DELIMITER $$

CREATE FUNCTION fn_lab_product_price_category(
    p_msrp DECIMAL(10,2)
)
RETURNS VARCHAR(30)
DETERMINISTIC
NO SQL
BEGIN
    RETURN CASE
        WHEN p_msrp IS NULL THEN 'Price not available'
        WHEN p_msrp >= 150.00 THEN 'Premium'
        WHEN p_msrp >= 100.00 THEN 'Standard'
        ELSE 'Budget'
    END;
END$$

DELIMITER ;
```

Gọi function:

```sql
SELECT productCode,
       productName,
       MSRP,
       fn_lab_product_price_category(MSRP) AS priceCategory
FROM products
ORDER BY MSRP DESC
LIMIT 10;
```

### 8.3. Function có thể dùng trong `WHERE`

Ví dụ:

```sql
SELECT productCode, productName, MSRP
FROM products
WHERE fn_lab_product_price_category(MSRP) = 'Premium'
ORDER BY MSRP DESC;
```

Tuy nhiên, nên thận trọng khi đặt function lên cột trong `WHERE` trên bảng lớn, vì điều đó có thể khiến optimizer khó tận dụng index trực tiếp hơn một điều kiện đơn giản.

Nếu điều kiện thực chất chỉ là:

```sql
MSRP >= 150.00
```

thì viết điều kiện trực tiếp thường rõ ràng hơn:

```sql
SELECT productCode, productName, MSRP
FROM products
WHERE MSRP >= 150.00
ORDER BY MSRP DESC;
```

### Bài tập thực hành

**Bài 8.1.**

Tạo function `fn_lab_credit_category` theo ví dụ.

**Bài 8.2.**

Tạo function `fn_lab_stock_category` nhận `quantityInStock INT` và trả về `Low`, `Normal`, `High` theo các ngưỡng do bạn chọn.

**Bài 8.3.**

Tạo function `fn_lab_order_status_label` nhận `status VARCHAR(20)` và dùng `CASE` để trả về nhãn tiếng Anh dễ hiểu.

**Bài 8.4.**

Dùng `fn_lab_product_price_category(MSRP)` để hiển thị category của 20 sản phẩm đầu tiên.

**Bài 8.5.**

Viết truy vấn không dùng function để lọc sản phẩm có MSRP từ `150.00` trở lên. Giải thích vì sao câu này có thể phù hợp hơn khi chỉ cần một điều kiện đơn giản.

---

## 9. Gọi stored function trong các ngữ cảnh SQL

Stored function có thể được gọi trong các biểu thức SQL khi giá trị trả về phù hợp kiểu dữ liệu và ngữ cảnh.

### 9.1. Trong `SELECT`

```sql
SELECT productCode,
       productName,
       buyPrice,
       MSRP,
       fn_lab_margin_percent(buyPrice, MSRP) AS marginPercent
FROM products
ORDER BY marginPercent DESC
LIMIT 10;
```

### 9.2. Trong biểu thức tính toán

```sql
SELECT 200.00 AS originalPrice,
       fn_lab_discounted_price(200.00, 15.00) AS discountedPrice;
```

### 9.3. Trong `ORDER BY`

```sql
SELECT productCode,
       productName,
       buyPrice,
       MSRP
FROM products
ORDER BY fn_lab_margin_percent(buyPrice, MSRP) DESC
LIMIT 10;
```

### 9.4. Trong `HAVING`

```sql
SELECT o.customerNumber,
       COUNT(*) AS totalOrders,
       SUM(fn_lab_order_total(o.orderNumber)) AS totalOrderValue
FROM orders AS o
GROUP BY o.customerNumber
HAVING totalOrderValue >= 100000.00
ORDER BY totalOrderValue DESC;
```

> Ví dụ trên nhằm minh họa cú pháp. Với tổng hợp theo nhiều order trên tập dữ liệu lớn, cách `JOIN orders` với `orderdetails` và dùng `SUM()` trực tiếp thường hiệu quả và dễ tối ưu hơn.

### 9.5. Không gọi function bằng `CALL`

Sai:

```sql
CALL fn_lab_order_total(10100);
```

Đúng:

```sql
SELECT fn_lab_order_total(10100);
```

### Bài tập thực hành

**Bài 9.1.**

Dùng `fn_lab_discounted_price` trong một `SELECT` để tính giá sau giảm 10% cho ba giá trị bất kỳ.

**Bài 9.2.**

Dùng `fn_lab_margin_percent` để hiển thị margin của các sản phẩm thuộc `Motorcycles`.

**Bài 9.3.**

Dùng `fn_lab_credit_category(customerNumber)` để hiển thị category của khách hàng ở `USA`.

**Bài 9.4.**

Sắp xếp 10 sản phẩm có MSRP cao nhất bằng `ORDER BY fn_lab_product_price_category(MSRP)` và `MSRP DESC`.

**Bài 9.5.**

Sửa câu lệnh sai sau để gọi function đúng cách:

```sql
CALL fn_lab_product_price_category(120.00);
```

---

## 10. Xóa stored function bằng `DROP FUNCTION`

### 10.1. Cú pháp

```sql
DROP FUNCTION function_name;
```

Ví dụ:

```sql
DROP FUNCTION fn_lab_discounted_price;
```

Nếu function không tồn tại, MySQL báo lỗi.

### 10.2. Dùng `IF EXISTS`

Để tránh lỗi khi function có thể chưa tồn tại:

```sql
DROP FUNCTION IF EXISTS fn_lab_discounted_price;
```

`IF EXISTS` giúp script có thể chạy lặp lại. Nếu function không tồn tại, MySQL tạo warning thay vì lỗi.

### 10.3. Quy trình thay đổi function

MySQL không dùng `ALTER FUNCTION` để thay đổi trực tiếp thân function.

Quy trình thông dụng:

1. Dùng `SHOW CREATE FUNCTION` để xem definition hiện có.
2. Lưu hoặc sao chép definition cần thiết.
3. `DROP FUNCTION IF EXISTS function_name`.
4. `CREATE FUNCTION` lại với logic mới.
5. Gọi function để kiểm tra.

Ví dụ:

```sql
SHOW CREATE FUNCTION fn_lab_discounted_price;
```

```sql
DROP FUNCTION IF EXISTS fn_lab_discounted_price;
```

Sau đó tạo lại function với definition mới.

### 10.4. Lưu ý

- `DROP FUNCTION` xóa definition của stored function, không xóa bảng hoặc dữ liệu.
- `DROP FUNCTION` trong phần này dùng để xóa **stored function**. MySQL cũng có một ngữ cảnh khác của `DROP FUNCTION` dành cho loadable function; trong lab này chỉ làm việc với stored function tạo bằng `CREATE FUNCTION ... RETURNS`.
- Nên có file script lưu các câu `CREATE FUNCTION` để tái tạo function khi cần.

### Bài tập thực hành

**Bài 10.1.**

Tạo function `fn_lab_temp_function` trả về chuỗi `Temporary function`.

**Bài 10.2.**

Gọi `fn_lab_temp_function` bằng `SELECT`.

**Bài 10.3.**

Xóa `fn_lab_temp_function` bằng `DROP FUNCTION IF EXISTS`.

**Bài 10.4.**

Chạy lại `DROP FUNCTION IF EXISTS fn_lab_temp_function` và kiểm tra warning nếu có.

**Bài 10.5.**

Nêu các bước an toàn để thay đổi logic của `fn_lab_discounted_price`.

---

## 11. Liệt kê và kiểm tra stored function

### 11.1. `SHOW FUNCTION STATUS`

Liệt kê function mà tài khoản hiện tại có quyền xem:

```sql
SHOW FUNCTION STATUS;
```

Lọc theo database:

```sql
SHOW FUNCTION STATUS
WHERE Db = 'classicmodels';
```

Lọc theo tên:

```sql
SHOW FUNCTION STATUS
WHERE Db = 'classicmodels'
  AND Name LIKE 'fn_lab%';
```

Thông tin thường có:

| Cột | Ý nghĩa |
|---|---|
| `Db` | Database chứa function |
| `Name` | Tên function |
| `Type` | `FUNCTION` |
| `Definer` | Tài khoản định nghĩa routine |
| `Created` | Thời điểm tạo |
| `Modified` | Thời điểm sửa gần nhất |
| `Security_type` | Ngữ cảnh bảo mật khi thực thi |

### 11.2. `SHOW CREATE FUNCTION`

Xem statement dùng để tạo một function cụ thể:

```sql
SHOW CREATE FUNCTION fn_lab_order_total;
```

Lệnh này hữu ích khi:

- Muốn sao lưu definition.
- Muốn xem characteristic đã khai báo.
- Muốn sửa function bằng quy trình drop và create lại.
- Muốn kiểm tra `DEFINER`, `SQL SECURITY`, return type hoặc logic hiện tại.

### 11.3. `INFORMATION_SCHEMA.ROUTINES`

Có thể truy vấn metadata của stored functions:

```sql
SELECT ROUTINE_SCHEMA,
       ROUTINE_NAME,
       ROUTINE_TYPE,
       DATA_TYPE,
       DTD_IDENTIFIER,
       IS_DETERMINISTIC,
       SQL_DATA_ACCESS,
       CREATED,
       LAST_ALTERED
FROM INFORMATION_SCHEMA.ROUTINES
WHERE ROUTINE_SCHEMA = 'classicmodels'
  AND ROUTINE_TYPE = 'FUNCTION'
ORDER BY ROUTINE_NAME;
```

Chỉ liệt kê function lab:

```sql
SELECT ROUTINE_NAME,
       DATA_TYPE,
       DTD_IDENTIFIER,
       IS_DETERMINISTIC,
       SQL_DATA_ACCESS,
       CREATED,
       LAST_ALTERED
FROM INFORMATION_SCHEMA.ROUTINES
WHERE ROUTINE_SCHEMA = 'classicmodels'
  AND ROUTINE_TYPE = 'FUNCTION'
  AND ROUTINE_NAME LIKE 'fn_lab%'
ORDER BY ROUTINE_NAME;
```

### 11.4. Quyền xem metadata

Kết quả `SHOW FUNCTION STATUS`, `SHOW CREATE FUNCTION` và `INFORMATION_SCHEMA.ROUTINES` phụ thuộc vào quyền của tài khoản.

Nếu không xem được function hoặc không xem được definition đầy đủ:

1. Kiểm tra `SHOW GRANTS`.
2. Kiểm tra function có phải do tài khoản hiện tại tạo ra không.
3. Hỏi quản trị viên về quyền liên quan đến routine metadata.

### Bài tập thực hành

**Bài 11.1.**

Dùng `SHOW FUNCTION STATUS` để liệt kê function trong `classicmodels`.

**Bài 11.2.**

Dùng `SHOW FUNCTION STATUS` để chỉ liệt kê các function có tên bắt đầu bằng `fn_lab`.

**Bài 11.3.**

Dùng `SHOW CREATE FUNCTION` để xem definition của `fn_lab_order_total`.

**Bài 11.4.**

Dùng `INFORMATION_SCHEMA.ROUTINES` để hiển thị tên function, kiểu trả về và `IS_DETERMINISTIC` của các function lab.

**Bài 11.5.**

Dùng `INFORMATION_SCHEMA.ROUTINES` để liệt kê các function có `SQL_DATA_ACCESS = 'READS SQL DATA'`.

---

## 12. Hạn chế và lưu ý thiết kế

### 12.1. Function trả về một giá trị, không trả result set

Không viết function kiểu sau:

```sql
CREATE FUNCTION fn_lab_bad_list_orders(
    p_customer_number INT
)
RETURNS INT
BEGIN
    SELECT orderNumber, orderDate
    FROM orders
    WHERE customerNumber = p_customer_number;
END;
```

Function không dùng để trả result set. Nếu cần danh sách order, dùng procedure hoặc một truy vấn `SELECT` trực tiếp.

### 12.2. Không dùng `CALL` cho function

Sai:

```sql
CALL fn_lab_order_total(10100);
```

Đúng:

```sql
SELECT fn_lab_order_total(10100);
```

### 12.3. Không dùng `OUT` hoặc `INOUT` cho function

Sai:

```sql
CREATE FUNCTION fn_lab_bad_function(
    IN p_value INT,
    OUT p_result INT
)
RETURNS INT
...
```

Đúng:

```sql
CREATE FUNCTION fn_lab_good_function(
    p_value INT
)
RETURNS INT
DETERMINISTIC
NO SQL
RETURN p_value * 2;
```

### 12.4. Không đặt tên trùng với built-in function

Tránh đặt tên như:

```text
COUNT
SUM
ROUND
CONCAT
NOW
```

Tên function tự tạo nên rõ nghĩa và có prefix.

Ví dụ tốt:

```text
fn_lab_order_total
fn_sales_customer_tier
fn_reporting_month_label
```

### 12.5. Cẩn trọng khi gọi function theo từng dòng

Câu truy vấn:

```sql
SELECT orderNumber,
       fn_lab_order_total(orderNumber)
FROM orders;
```

có thể gọi function nhiều lần, một lần cho mỗi order. Nếu function bên trong đọc bảng, điều này có thể gây chi phí đáng kể trên dữ liệu lớn.

Khi cần tổng hợp trên nhiều dòng, thường nên so sánh với truy vấn set-based:

```sql
SELECT o.orderNumber,
       COALESCE(SUM(od.quantityOrdered * od.priceEach), 0) AS totalOrderValue
FROM orders AS o
LEFT JOIN orderdetails AS od
    ON od.orderNumber = o.orderNumber
GROUP BY o.orderNumber;
```

### 12.6. Hạn chế thao tác dữ liệu trong function

Trong thiết kế ứng dụng thông thường, nên ưu tiên function mang tính tính toán hoặc đọc dữ liệu. Tránh dùng function để tạo side effect khó dự đoán.

Đặc biệt, MySQL có các hạn chế đối với stored function trong một số ngữ cảnh, chẳng hạn không nên cố thay đổi một bảng đã được dùng trong statement đang gọi function đó.

### Bài tập thực hành

**Bài 12.1.**

Giải thích vì sao function không phù hợp để trả về danh sách tất cả customer ở `USA`.

**Bài 12.2.**

Sửa câu gọi function sai dùng `CALL`.

**Bài 12.3.**

Sửa definition function sai có `OUT` parameter.

**Bài 12.4.**

Đề xuất tên function tốt hơn cho một function người học đặt là `SUM`.

**Bài 12.5.**

Viết truy vấn set-based thay cho yêu cầu “hiển thị tổng giá trị của mọi order”.

---

## 13. Ví dụ tổng hợp: Các function hỗ trợ báo cáo đơn hàng

Phần này tạo ba function có thể dùng chung trong báo cáo.

### 13.1. Function tính tổng đơn hàng

```sql
DROP FUNCTION IF EXISTS fn_lab_order_total;

DELIMITER $$

CREATE FUNCTION fn_lab_order_total(
    p_order_number INT
)
RETURNS DECIMAL(14,2)
READS SQL DATA
BEGIN
    DECLARE v_total DECIMAL(14,2) DEFAULT 0.00;

    SELECT COALESCE(SUM(quantityOrdered * priceEach), 0)
    INTO v_total
    FROM orderdetails
    WHERE orderNumber = p_order_number;

    RETURN v_total;
END$$

DELIMITER ;
```

### 13.2. Function phân loại tổng đơn hàng

```sql
DROP FUNCTION IF EXISTS fn_lab_order_value_category;

DELIMITER $$

CREATE FUNCTION fn_lab_order_value_category(
    p_order_total DECIMAL(14,2)
)
RETURNS VARCHAR(20)
DETERMINISTIC
NO SQL
BEGIN
    RETURN CASE
        WHEN p_order_total IS NULL THEN 'Unknown'
        WHEN p_order_total >= 100000.00 THEN 'Large'
        WHEN p_order_total >= 25000.00 THEN 'Medium'
        ELSE 'Small'
    END;
END$$

DELIMITER ;
```

### 13.3. Function diễn giải status

```sql
DROP FUNCTION IF EXISTS fn_lab_order_status_label;

DELIMITER $$

CREATE FUNCTION fn_lab_order_status_label(
    p_status VARCHAR(20)
)
RETURNS VARCHAR(50)
DETERMINISTIC
NO SQL
BEGIN
    RETURN CASE p_status
        WHEN 'Shipped' THEN 'Order shipped'
        WHEN 'In Process' THEN 'Order in process'
        WHEN 'On Hold' THEN 'Order on hold'
        WHEN 'Cancelled' THEN 'Order cancelled'
        WHEN 'Disputed' THEN 'Order disputed'
        WHEN 'Resolved' THEN 'Order resolved'
        ELSE 'Other or unknown status'
    END;
END$$

DELIMITER ;
```

### 13.4. Dùng các function trong báo cáo

```sql
SELECT o.orderNumber,
       o.orderDate,
       o.status,
       fn_lab_order_status_label(o.status) AS statusLabel,
       fn_lab_order_total(o.orderNumber) AS totalOrderValue,
       fn_lab_order_value_category(
           fn_lab_order_total(o.orderNumber)
       ) AS valueCategory
FROM orders AS o
WHERE o.customerNumber = 103
ORDER BY totalOrderValue DESC;
```

> Truy vấn trên gọi `fn_lab_order_total` hai lần cho cùng một order. Với báo cáo lớn, nên viết `JOIN`/`GROUP BY` hoặc derived table/CTE để chỉ tính một lần. Ví dụ này chủ yếu minh họa cách kết hợp stored functions.

### Bài tập thực hành

**Bài 13.1.**

Tạo ba function trong ví dụ.

**Bài 13.2.**

Dùng `fn_lab_order_total` để báo cáo các order của customer `103`.

**Bài 13.3.**

Dùng `fn_lab_order_status_label` để hiển thị nhãn cho tất cả trạng thái đơn hàng khác nhau.

**Bài 13.4.**

Dùng `fn_lab_order_value_category` để phân loại tổng giá trị của các order thuộc customer `112`.

**Bài 13.5.**

Viết một truy vấn `JOIN` + `GROUP BY` thay thế báo cáo ở mục 13.4, không gọi `fn_lab_order_total` lặp lại.

---

## 14. Một số lỗi thường gặp

### Lỗi 1. Quên `RETURNS`

Sai:

```sql
CREATE FUNCTION fn_lab_bad_double(
    p_number INT
)
RETURN p_number * 2;
```

Đúng:

```sql
CREATE FUNCTION fn_lab_good_double(
    p_number INT
)
RETURNS INT
DETERMINISTIC
NO SQL
RETURN p_number * 2;
```

---

### Lỗi 2. Không dùng `RETURN`

Sai:

```sql
CREATE FUNCTION fn_lab_bad_square(
    p_number INT
)
RETURNS INT
DETERMINISTIC
NO SQL
BEGIN
    SET p_number = p_number * p_number;
END;
```

Đúng:

```sql
CREATE FUNCTION fn_lab_good_square(
    p_number INT
)
RETURNS INT
DETERMINISTIC
NO SQL
BEGIN
    RETURN p_number * p_number;
END;
```

---

### Lỗi 3. Dùng `CALL` cho function

Sai:

```sql
CALL fn_lab_order_total(10100);
```

Đúng:

```sql
SELECT fn_lab_order_total(10100);
```

---

### Lỗi 4. Tạo lại function khi function cũ còn tồn tại

Sai:

```sql
CREATE FUNCTION fn_lab_discounted_price(...)
...
```

nếu function cùng tên đã tồn tại.

Khắc phục:

```sql
DROP FUNCTION IF EXISTS fn_lab_discounted_price;
```

sau đó tạo lại function.

---

### Lỗi 5. Quên khôi phục delimiter

Sau khi tạo function nhiều câu lệnh, cần trả lại:

```sql
DELIMITER ;
```

Nếu không, những câu lệnh tiếp theo trong client có thể không được kết thúc như mong muốn.

---

### Lỗi 6. Đặt characteristic không phù hợp

Không nên khai báo:

```sql
DETERMINISTIC
```

cho function gọi `NOW()` hoặc `RAND()`.

Không nên khai báo:

```sql
NO SQL
```

cho function có `SELECT ... FROM customers`.

### Bài tập thực hành

**Bài 14.1.**

Sửa function bị thiếu `RETURNS INT`.

**Bài 14.2.**

Sửa function có tính toán nhưng không có `RETURN`.

**Bài 14.3.**

Sửa câu gọi function dùng `CALL`.

**Bài 14.4.**

Viết các câu lệnh cần có để tạo lại `fn_lab_discounted_price` an toàn.

**Bài 14.5.**

Chọn characteristic đúng cho một function dùng `SELECT MAX(creditLimit) FROM customers`.

---

## 15. Bài tập tổng hợp

### Bài 15.1. Function tính giá trị dòng order

Tạo function `fn_lab_order_line_value`:

1. Nhận `p_quantity INT` và `p_price_each DECIMAL(10,2)`.
2. Trả về `p_quantity * p_price_each`.
3. Khai báo characteristic phù hợp.
4. Gọi function với ít nhất ba bộ giá trị.
5. Dùng function trong `SELECT` trên `orderdetails` của order `10100`.

---

### Bài 15.2. Function tính margin của product

Tạo function `fn_lab_product_margin`:

1. Nhận `p_product_code VARCHAR(15)`.
2. Đọc `buyPrice` và `MSRP` từ bảng `products`.
3. Trả về `MSRP - buyPrice`.
4. Nếu product không tồn tại, trả về `NULL`.
5. Dùng function để hiển thị margin của các sản phẩm thuộc `Motorcycles`.

---

### Bài 15.3. Function phân loại customer

Tạo function `fn_lab_customer_tier`:

1. Nhận `p_customer_number INT`.
2. Đọc `creditLimit` từ bảng `customers`.
3. Trả về `Gold`, `Silver`, `Bronze` theo ngưỡng tự chọn.
4. Trả về `Customer not found` nếu mã không tồn tại.
5. Dùng function để hiển thị tier của 10 khách hàng đầu tiên.

---

### Bài 15.4. Liệt kê metadata function

1. Liệt kê mọi function trong database `classicmodels`.
2. Chỉ lọc function có prefix `fn_lab`.
3. Hiển thị return type của các function đó.
4. Hiển thị `IS_DETERMINISTIC` và `SQL_DATA_ACCESS`.
5. Dùng `SHOW CREATE FUNCTION` để xem definition của một function tự tạo.

---

### Bài 15.5. Dọn dẹp

1. Liệt kê các function lab cần xóa.
2. Viết `DROP FUNCTION IF EXISTS` cho từng function.
3. Xác nhận bằng `SHOW FUNCTION STATUS`.
4. Giải thích vì sao dọn dẹp function lab là cần thiết trên database dùng chung.
5. Lưu lại script tạo function trước khi xóa.

---

## 16. Đáp án gợi ý cho một số bài tập

### Bài 4.1

```sql
DROP FUNCTION IF EXISTS fn_lab_double_number;

DELIMITER $$

CREATE FUNCTION fn_lab_double_number(
    p_number INT
)
RETURNS INT
DETERMINISTIC
NO SQL
BEGIN
    RETURN p_number * 2;
END$$

DELIMITER ;

SELECT fn_lab_double_number(12) AS doubledValue;
```

### Bài 4.3

```sql
DROP FUNCTION IF EXISTS fn_lab_price_after_tax;

DELIMITER $$

CREATE FUNCTION fn_lab_price_after_tax(
    p_price DECIMAL(10,2),
    p_tax_percent DECIMAL(5,2)
)
RETURNS DECIMAL(10,2)
DETERMINISTIC
NO SQL
BEGIN
    RETURN p_price * (1 + p_tax_percent / 100);
END$$

DELIMITER ;

SELECT fn_lab_price_after_tax(100.00, 10.00) AS priceAfterTax;
```

### Bài 5.1

```sql
DROP FUNCTION IF EXISTS fn_lab_absolute_value;

DELIMITER $$

CREATE FUNCTION fn_lab_absolute_value(
    p_number INT
)
RETURNS INT
DETERMINISTIC
NO SQL
BEGIN
    IF p_number < 0 THEN
        RETURN -p_number;
    END IF;

    RETURN p_number;
END$$

DELIMITER ;
```

### Bài 5.3

```sql
DROP FUNCTION IF EXISTS fn_lab_safe_divide;

DELIMITER $$

CREATE FUNCTION fn_lab_safe_divide(
    p_numerator DECIMAL(14,2),
    p_denominator DECIMAL(14,2)
)
RETURNS DECIMAL(14,4)
DETERMINISTIC
NO SQL
BEGIN
    IF p_denominator IS NULL
       OR p_denominator = 0 THEN
        RETURN NULL;
    END IF;

    RETURN p_numerator / p_denominator;
END$$

DELIMITER ;
```

### Bài 7.4

```sql
DROP FUNCTION IF EXISTS fn_lab_order_line_count;

DELIMITER $$

CREATE FUNCTION fn_lab_order_line_count(
    p_order_number INT
)
RETURNS INT
READS SQL DATA
BEGIN
    DECLARE v_line_count INT DEFAULT 0;

    SELECT COUNT(*)
    INTO v_line_count
    FROM orderdetails
    WHERE orderNumber = p_order_number;

    RETURN v_line_count;
END$$

DELIMITER ;
```

### Bài 7.5

```sql
DROP FUNCTION IF EXISTS fn_lab_product_margin;

DELIMITER $$

CREATE FUNCTION fn_lab_product_margin(
    p_product_code VARCHAR(15)
)
RETURNS DECIMAL(10,2)
READS SQL DATA
BEGIN
    DECLARE v_margin DECIMAL(10,2) DEFAULT NULL;

    SELECT MAX(MSRP - buyPrice)
    INTO v_margin
    FROM products
    WHERE productCode = p_product_code;

    RETURN v_margin;
END$$

DELIMITER ;
```

### Bài 8.2

```sql
DROP FUNCTION IF EXISTS fn_lab_stock_category;

DELIMITER $$

CREATE FUNCTION fn_lab_stock_category(
    p_quantity_in_stock INT
)
RETURNS VARCHAR(20)
DETERMINISTIC
NO SQL
BEGIN
    RETURN CASE
        WHEN p_quantity_in_stock IS NULL THEN 'Unknown'
        WHEN p_quantity_in_stock < 1000 THEN 'Low'
        WHEN p_quantity_in_stock < 5000 THEN 'Normal'
        ELSE 'High'
    END;
END$$

DELIMITER ;
```

### Bài 8.3

```sql
DROP FUNCTION IF EXISTS fn_lab_order_status_label;

DELIMITER $$

CREATE FUNCTION fn_lab_order_status_label(
    p_status VARCHAR(20)
)
RETURNS VARCHAR(50)
DETERMINISTIC
NO SQL
BEGIN
    RETURN CASE p_status
        WHEN 'Shipped' THEN 'Order shipped'
        WHEN 'In Process' THEN 'Order in process'
        WHEN 'On Hold' THEN 'Order on hold'
        WHEN 'Cancelled' THEN 'Order cancelled'
        WHEN 'Disputed' THEN 'Order disputed'
        ELSE 'Other or unknown status'
    END;
END$$

DELIMITER ;
```

### Bài 11.4

```sql
SELECT ROUTINE_NAME,
       DATA_TYPE,
       DTD_IDENTIFIER,
       IS_DETERMINISTIC,
       SQL_DATA_ACCESS,
       CREATED,
       LAST_ALTERED
FROM INFORMATION_SCHEMA.ROUTINES
WHERE ROUTINE_SCHEMA = 'classicmodels'
  AND ROUTINE_TYPE = 'FUNCTION'
  AND ROUTINE_NAME LIKE 'fn_lab%'
ORDER BY ROUTINE_NAME;
```

### Bài 15.1

```sql
DROP FUNCTION IF EXISTS fn_lab_order_line_value;

DELIMITER $$

CREATE FUNCTION fn_lab_order_line_value(
    p_quantity INT,
    p_price_each DECIMAL(10,2)
)
RETURNS DECIMAL(14,2)
DETERMINISTIC
NO SQL
BEGIN
    RETURN p_quantity * p_price_each;
END$$

DELIMITER ;

SELECT orderNumber,
       productCode,
       quantityOrdered,
       priceEach,
       fn_lab_order_line_value(
           quantityOrdered,
           priceEach
       ) AS lineValue
FROM orderdetails
WHERE orderNumber = 10100
ORDER BY orderLineNumber;
```

### Bài 15.2

```sql
DROP FUNCTION IF EXISTS fn_lab_product_margin;

DELIMITER $$

CREATE FUNCTION fn_lab_product_margin(
    p_product_code VARCHAR(15)
)
RETURNS DECIMAL(10,2)
READS SQL DATA
BEGIN
    DECLARE v_margin DECIMAL(10,2) DEFAULT NULL;

    SELECT MAX(MSRP - buyPrice)
    INTO v_margin
    FROM products
    WHERE productCode = p_product_code;

    RETURN v_margin;
END$$

DELIMITER ;

SELECT productCode,
       productName,
       fn_lab_product_margin(productCode) AS marginValue
FROM products
WHERE productLine = 'Motorcycles'
ORDER BY marginValue DESC;
```

### Bài 15.3

```sql
DROP FUNCTION IF EXISTS fn_lab_customer_tier;

DELIMITER $$

CREATE FUNCTION fn_lab_customer_tier(
    p_customer_number INT
)
RETURNS VARCHAR(30)
READS SQL DATA
BEGIN
    DECLARE v_credit_limit DECIMAL(10,2) DEFAULT NULL;

    SELECT MAX(creditLimit)
    INTO v_credit_limit
    FROM customers
    WHERE customerNumber = p_customer_number;

    IF v_credit_limit IS NULL THEN
        RETURN 'Customer not found';
    ELSEIF v_credit_limit >= 100000.00 THEN
        RETURN 'Gold';
    ELSEIF v_credit_limit >= 50000.00 THEN
        RETURN 'Silver';
    ELSE
        RETURN 'Bronze';
    END IF;
END$$

DELIMITER ;

SELECT customerNumber,
       customerName,
       creditLimit,
       fn_lab_customer_tier(customerNumber) AS customerTier
FROM customers
ORDER BY customerNumber
LIMIT 10;
```

---

## 17. Script dọn dẹp sau lab

Sau khi hoàn thành lab, có thể xóa các function thực hành:

```sql
DROP FUNCTION IF EXISTS fn_lab_double_number;
DROP FUNCTION IF EXISTS fn_lab_add_numbers;
DROP FUNCTION IF EXISTS fn_lab_price_after_tax;
DROP FUNCTION IF EXISTS fn_lab_greeting;
DROP FUNCTION IF EXISTS fn_lab_current_year;
DROP FUNCTION IF EXISTS fn_lab_absolute_value;
DROP FUNCTION IF EXISTS fn_lab_price_difference;
DROP FUNCTION IF EXISTS fn_lab_safe_divide;
DROP FUNCTION IF EXISTS fn_lab_customer_label;
DROP FUNCTION IF EXISTS fn_lab_discounted_price;
DROP FUNCTION IF EXISTS fn_lab_margin_percent;
DROP FUNCTION IF EXISTS fn_lab_order_total;
DROP FUNCTION IF EXISTS fn_lab_customer_credit_limit;
DROP FUNCTION IF EXISTS fn_lab_order_line_count;
DROP FUNCTION IF EXISTS fn_lab_product_margin;
DROP FUNCTION IF EXISTS fn_lab_credit_category;
DROP FUNCTION IF EXISTS fn_lab_product_price_category;
DROP FUNCTION IF EXISTS fn_lab_stock_category;
DROP FUNCTION IF EXISTS fn_lab_order_status_label;
DROP FUNCTION IF EXISTS fn_lab_order_value_category;
DROP FUNCTION IF EXISTS fn_lab_order_line_value;
DROP FUNCTION IF EXISTS fn_lab_customer_tier;
DROP FUNCTION IF EXISTS fn_lab_current_timestamp_text;
```

Kiểm tra sau khi dọn dẹp:

```sql
SHOW FUNCTION STATUS
WHERE Db = 'classicmodels'
  AND Name LIKE 'fn_lab%';
```

---

## 18. Tóm tắt

Các kiến thức chính trong lab:

- Stored function là routine trả về một giá trị và được gọi trong biểu thức SQL.
- Function khác procedure ở cách gọi, khả năng trả result set và loại tham số.
- `CREATE FUNCTION` yêu cầu `RETURNS data_type`.
- Function dùng `RETURN` để trả giá trị.
- Function parameter chỉ là input; không dùng `IN`, `OUT`, `INOUT`.
- `DETERMINISTIC` và `NOT DETERMINISTIC` mô tả tính ổn định của kết quả theo input.
- `NO SQL` phù hợp với function chỉ tính toán; `READS SQL DATA` phù hợp với function đọc bảng.
- Có thể dùng function trong `SELECT`, biểu thức, `ORDER BY`, `WHERE` và nhiều ngữ cảnh SQL khác khi phù hợp.
- Dùng `DROP FUNCTION [IF EXISTS]` để xóa stored function.
- Dùng `SHOW FUNCTION STATUS`, `SHOW CREATE FUNCTION` và `INFORMATION_SCHEMA.ROUTINES` để liệt kê và kiểm tra function.
- Không nên dùng function để thay thế một truy vấn set-based đơn giản trên dữ liệu lớn.
- Trước khi xóa hoặc sửa function, nên lưu lại definition bằng `SHOW CREATE FUNCTION` hoặc script `.sql`.

---

## 19. Từ khóa chính

- Stored Function
- CREATE FUNCTION
- RETURNS
- RETURN
- Function Parameter
- DETERMINISTIC
- NOT DETERMINISTIC
- NO SQL
- READS SQL DATA
- SQL DATA ACCESS
- DROP FUNCTION
- SHOW FUNCTION STATUS
- SHOW CREATE FUNCTION
- INFORMATION_SCHEMA.ROUTINES
- Routine Metadata
- Scalar Value
- Set-based SQL
- classicmodels
