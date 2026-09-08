---
title: "Tutorial: Transaction trong MySQL"
author: "Tên giảng viên"
duration: "120m"
difficulty: "Intermediate"
prerequisites:
    - "Đã biết SELECT, INSERT, UPDATE, DELETE"
    - "Đã biết khóa chính, khóa ngoại và các bảng cơ bản trong classicmodels"
    - "Đã biết cách dùng MySQL Workbench hoặc MySQL client"
summary: "Thực hành transaction trong MySQL với START TRANSACTION, COMMIT, ROLLBACK, SAVEPOINT, autocommit và các lưu ý về ACID, storage engine, DDL và implicit commit."
---

# Tutorial: Transaction trong MySQL

## Link tham khảo

- [MySQL Tutorial — MySQL Transactions](https://www.mysqltutorial.org/mysql-stored-procedure/mysql-transactions/)

## 1. Mục tiêu học tập

Sau khi hoàn thành tutorial này, người học có thể:

1. Giải thích được khái niệm transaction trong MySQL.
2. Trình bày được bốn tính chất ACID.
3. Phân biệt được `COMMIT`, `ROLLBACK`, `ROLLBACK TO SAVEPOINT` và `RELEASE SAVEPOINT`.
4. Kiểm tra và điều chỉnh chế độ `autocommit` trong một session MySQL.
5. Thực hiện transaction bằng `START TRANSACTION` để nhóm nhiều câu lệnh DML thành một đơn vị công việc.
6. Hoàn tác toàn bộ hoặc một phần thay đổi bằng `ROLLBACK` và `SAVEPOINT`.
7. Nhận biết được điều kiện cần để rollback hoạt động, đặc biệt là storage engine có hỗ trợ transaction như InnoDB.
8. Giải thích được vì sao một số lệnh DDL có thể gây implicit commit và không thể rollback như DML thông thường.
9. Viết được các transaction cơ bản trên dữ liệu `classicmodels` mà không làm thay đổi dữ liệu gốc sau khi kết thúc lab.
10. Nhận biết các lỗi thực hành thường gặp liên quan đến transaction, khóa ngoại, session và thao tác đồng thời.

---

## 2. Transaction là gì?

Một **transaction** là một chuỗi một hoặc nhiều câu lệnh SQL được xử lý như **một đơn vị công việc**.

Ý tưởng chính:

- Hoặc tất cả thao tác trong transaction được lưu thành công.
- Hoặc các thao tác cần thiết được hoàn tác để cơ sở dữ liệu quay về trạng thái nhất quán trước transaction.

Ví dụ chuyển tiền giữa hai tài khoản:

```text
1. Trừ tiền ở tài khoản A.
2. Cộng tiền ở tài khoản B.
```

Hai bước này phải được xem là một công việc thống nhất. Không thể chấp nhận trạng thái:

```text
A đã bị trừ tiền, nhưng B chưa được cộng tiền.
```

Trong MySQL, ba câu lệnh nền tảng để quản lý transaction là:

| Câu lệnh | Ý nghĩa |
|---|---|
| `START TRANSACTION` | Bắt đầu transaction rõ ràng |
| `COMMIT` | Lưu vĩnh viễn thay đổi của transaction hiện tại |
| `ROLLBACK` | Hủy thay đổi chưa commit của transaction hiện tại |

Ví dụ khung transaction:

```sql
START TRANSACTION;

-- Một hoặc nhiều lệnh INSERT, UPDATE, DELETE

COMMIT;
```

Hoặc trong giờ thực hành:

```sql
START TRANSACTION;

-- Một hoặc nhiều lệnh INSERT, UPDATE, DELETE

ROLLBACK;
```

### Bài tập thực hành

**Bài 2.1.**

Viết định nghĩa transaction bằng lời của bạn.

**Bài 2.2.**

Trong nghiệp vụ chuyển tiền, vì sao thao tác trừ tiền và cộng tiền cần nằm trong cùng một transaction?

**Bài 2.3.**

Câu lệnh nào bắt đầu transaction rõ ràng trong MySQL?

**Bài 2.4.**

Câu lệnh nào lưu vĩnh viễn các thay đổi?

**Bài 2.5.**

Câu lệnh nào dùng để hủy các thay đổi chưa được commit?

---

## 3. Bốn tính chất ACID

Transaction thường được mô tả bằng bốn tính chất **ACID**.

| Tính chất | Ý nghĩa |
|---|---|
| **Atomicity** | Transaction hoặc hoàn thành toàn bộ, hoặc không để lại thay đổi dở dang cần được hủy |
| **Consistency** | Transaction đưa database từ một trạng thái hợp lệ sang một trạng thái hợp lệ khác, tôn trọng constraint và quy tắc nghiệp vụ |
| **Isolation** | Các transaction đồng thời không được nhìn thấy hoặc tác động lẫn nhau theo cách gây kết quả không nhất quán |
| **Durability** | Sau `COMMIT`, thay đổi đã được xác nhận sẽ tồn tại ngay cả khi có sự cố hệ thống sau đó |

### 3.1. Atomicity

Ví dụ đơn hàng mới cần có:

1. Một dòng trong `orders`.
2. Một hoặc nhiều dòng trong `orderdetails`.

Nếu chỉ tạo được đơn hàng nhưng không tạo được chi tiết đơn hàng, hệ thống có thể có dữ liệu dở dang. Transaction giúp ứng dụng chủ động rollback khi chuỗi thao tác không hoàn tất.

### 3.2. Consistency

Database phải tiếp tục thỏa các ràng buộc:

- Khóa chính không được trùng.
- Khóa ngoại phải tham chiếu đến dữ liệu tồn tại.
- Cột `NOT NULL` phải có giá trị.
- Các `CHECK`, `UNIQUE` và quy tắc nghiệp vụ phải được tôn trọng.

### 3.3. Isolation

Nếu hai người cùng thay đổi một bản ghi, MySQL cần điều phối thông qua isolation level và locking để tránh các hành vi không mong muốn.

Ví dụ:

```text
Nhân viên A đang cập nhật số lượng tồn kho.
Nhân viên B đồng thời đặt hàng từ cùng số lượng tồn kho đó.
```

Nếu không có kiểm soát phù hợp, dữ liệu có thể bị cập nhật sai.

### 3.4. Durability

Sau khi `COMMIT`, MySQL/InnoDB có cơ chế bảo đảm thay đổi đã xác nhận có thể được phục hồi sau sự cố theo các cơ chế bền vững của hệ thống.

### Bài tập thực hành

**Bài 3.1.**

Giải thích Atomicity bằng ví dụ chuyển tiền.

**Bài 3.2.**

Nêu một constraint trong database giúp đảm bảo Consistency.

**Bài 3.3.**

Hai transaction cùng thay đổi một dòng dữ liệu liên quan đến tính chất ACID nào?

**Bài 3.4.**

Sau khi `COMMIT`, tính chất nào bảo đảm thay đổi không bị mất khi hệ thống gặp sự cố?

**Bài 3.5.**

Một transaction tạo `orders` nhưng không tạo được `orderdetails` rồi dùng `ROLLBACK` minh họa tính chất nào?

---

## 4. Chuẩn bị môi trường thực hành

Chọn cơ sở dữ liệu mẫu:

```sql
USE classicmodels;
```

Kiểm tra các bảng thường dùng:

```sql
SHOW TABLES;
```

Trong tutorial này, các ví dụ chủ yếu tham chiếu đến:

| Bảng | Vai trò |
|---|---|
| `customers` | Khách hàng |
| `orders` | Đơn hàng |
| `orderdetails` | Các dòng sản phẩm của đơn hàng |
| `payments` | Khoản thanh toán |
| `products` | Sản phẩm |

Kiểm tra dữ liệu mẫu:

```sql
SELECT customerNumber, customerName, creditLimit
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

### 4.1. Kiểm tra storage engine

Transaction và rollback chỉ hoạt động đúng với bảng dùng storage engine hỗ trợ transaction. Với MySQL hiện đại, InnoDB là engine phổ biến cho các bảng giao dịch.

Kiểm tra engine của các bảng:

```sql
SELECT table_name, engine
FROM information_schema.tables
WHERE table_schema = 'classicmodels'
  AND table_name IN ('customers', 'orders', 'orderdetails', 'payments');
```

Hoặc:

```sql
SHOW TABLE STATUS LIKE 'customers';
```

> Nếu một bảng không dùng engine có hỗ trợ transaction, đừng kỳ vọng `ROLLBACK` sẽ hoàn tác các thay đổi trên bảng đó.

### Bài tập thực hành

**Bài 4.1.**

Viết lệnh chọn cơ sở dữ liệu `classicmodels`.

**Bài 4.2.**

Viết truy vấn xem engine của bảng `customers`.

**Bài 4.3.**

Viết truy vấn xem engine của bốn bảng `customers`, `orders`, `orderdetails`, `payments`.

**Bài 4.4.**

Viết truy vấn xem 10 khách hàng đầu tiên theo `customerNumber`.

**Bài 4.5.**

Giải thích vì sao cần kiểm tra storage engine trước khi thực hành `ROLLBACK`.

---

## 5. Autocommit trong MySQL

### 5.1. Autocommit là gì?

MySQL thường chạy với chế độ `autocommit` bật.

Khi `autocommit = 1`, mỗi câu lệnh SQL thành công được xem như một transaction riêng và được commit tự động.

Ví dụ:

```sql
UPDATE customers
SET creditLimit = creditLimit + 1000.00
WHERE customerNumber = 103;
```

Nếu autocommit bật và câu lệnh thành công, thay đổi được lưu ngay; chạy `ROLLBACK` sau đó sẽ không hoàn tác được thay đổi đã tự commit.

Kiểm tra giá trị hiện tại:

```sql
SELECT @@autocommit;
```

Kết quả thường là:

```text
1
```

### 5.2. Tắt autocommit trong session hiện tại

```sql
SET autocommit = 0;
```

Khi đó, các câu lệnh DML chưa được commit sẽ nằm trong transaction hiện tại cho đến khi:

```sql
COMMIT;
```

hoặc:

```sql
ROLLBACK;
```

Bật lại autocommit:

```sql
SET autocommit = 1;
```

> `SET autocommit = 0` ảnh hưởng đến session hiện tại. Khi học và khi viết ứng dụng, cách rõ ràng hơn thường là dùng `START TRANSACTION` cho mỗi đơn vị công việc.

### 5.3. Thực hành an toàn với `START TRANSACTION`

Không cần thay đổi autocommit để bắt đầu một transaction rõ ràng:

```sql
START TRANSACTION;

UPDATE customers
SET creditLimit = creditLimit + 1000.00
WHERE customerNumber = 103;

ROLLBACK;
```

### Bài tập thực hành

**Bài 5.1.**

Viết truy vấn kiểm tra giá trị `autocommit` của session hiện tại.

**Bài 5.2.**

Viết lệnh tắt autocommit trong session hiện tại.

**Bài 5.3.**

Viết lệnh bật lại autocommit.

**Bài 5.4.**

Khi `autocommit = 1`, điều gì xảy ra với một câu `UPDATE` thành công?

**Bài 5.5.**

Viết khung transaction rõ ràng bằng `START TRANSACTION` mà không cần dùng `SET autocommit = 0`.

---

## 6. `START TRANSACTION`, `COMMIT` và `ROLLBACK`

### 6.1. `START TRANSACTION`

Dùng để bắt đầu transaction một cách rõ ràng:

```sql
START TRANSACTION;
```

Sau lệnh này, các thay đổi DML phù hợp sẽ chờ `COMMIT` hoặc `ROLLBACK`.

### 6.2. `COMMIT`

Dùng để lưu vĩnh viễn thay đổi:

```sql
COMMIT;
```

Sau `COMMIT`, các thay đổi của transaction không thể bị hoàn tác bằng `ROLLBACK` thông thường.

### 6.3. `ROLLBACK`

Dùng để hoàn tác thay đổi chưa commit:

```sql
ROLLBACK;
```

Ví dụ an toàn trên bảng `customers`:

```sql
START TRANSACTION;

SELECT customerNumber, customerName, creditLimit
FROM customers
WHERE customerNumber = 103;
```

```sql
UPDATE customers
SET creditLimit = creditLimit + 1000.00
WHERE customerNumber = 103;
```

```sql
SELECT customerNumber, customerName, creditLimit
FROM customers
WHERE customerNumber = 103;
```

```sql
ROLLBACK;
```

Sau `ROLLBACK`, kiểm tra lại:

```sql
SELECT customerNumber, customerName, creditLimit
FROM customers
WHERE customerNumber = 103;
```

Giá trị `creditLimit` phải trở về như trước transaction nếu bảng hỗ trợ transaction và chưa có implicit commit.

### 6.4. Quy trình an toàn khi thực hành DML

```text
1. SELECT để kiểm tra dòng sẽ ảnh hưởng.
2. START TRANSACTION.
3. INSERT / UPDATE / DELETE.
4. SELECT để kiểm tra kết quả tạm thời.
5. ROLLBACK khi chỉ thực hành; COMMIT khi muốn lưu thật.
```

### Bài tập thực hành

**Bài 6.1.**

Dùng `START TRANSACTION` để bắt đầu một transaction.

**Bài 6.2.**

Trong transaction, tăng `creditLimit` của khách hàng `103` thêm `1000.00`.

**Bài 6.3.**

Trong transaction, dùng `SELECT` để kiểm tra giá trị `creditLimit` sau cập nhật.

**Bài 6.4.**

Dùng `ROLLBACK` và kiểm tra lại giá trị `creditLimit` của khách hàng `103`.

**Bài 6.5.**

Viết một transaction gồm hai câu `UPDATE` trên khách hàng `103` và `112`, rồi kết thúc bằng `ROLLBACK`.

---

## 7. Transaction với nhiều thao tác DML

Transaction đặc biệt hữu ích khi một nghiệp vụ gồm nhiều thao tác phụ thuộc nhau.

Ví dụ tạo đơn hàng thực hành gồm:

1. Chèn khách hàng mới.
2. Chèn đơn hàng cho khách hàng đó.
3. Chèn dòng chi tiết đơn hàng.
4. Chèn khoản thanh toán.
5. Kiểm tra toàn bộ dữ liệu.
6. Rollback để không làm thay đổi dữ liệu gốc.

> Các mã thực hành dưới đây là ví dụ. Nếu môi trường dùng chung đã có các mã này, hãy thay bằng mã riêng trước khi chạy.

```sql
START TRANSACTION;
```

```sql
INSERT INTO customers (
    customerNumber,
    customerName,
    contactLastName,
    contactFirstName,
    phone,
    addressLine1,
    city,
    country,
    creditLimit
)
VALUES (
    900001,
    'Transaction Lab Co.',
    'Nguyen',
    'An',
    '+84 24 1234 5678',
    '1 Dai Co Viet',
    'Hanoi',
    'Vietnam',
    50000.00
);
```

```sql
INSERT INTO orders (
    orderNumber,
    orderDate,
    requiredDate,
    shippedDate,
    status,
    comments,
    customerNumber
)
VALUES (
    990001,
    '2026-06-23',
    '2026-06-30',
    NULL,
    'In Process',
    'Transaction practice order',
    900001
);
```

```sql
INSERT INTO orderdetails (
    orderNumber,
    productCode,
    quantityOrdered,
    priceEach,
    orderLineNumber
)
SELECT
    990001,
    productCode,
    2,
    MSRP,
    1
FROM products
ORDER BY productCode
LIMIT 1;
```

```sql
INSERT INTO payments (
    customerNumber,
    checkNumber,
    paymentDate,
    amount
)
VALUES (
    900001,
    'TXN-900001-01',
    '2026-06-23',
    1500.00
);
```

Kiểm tra dữ liệu tạm thời:

```sql
SELECT customerNumber, customerName, creditLimit
FROM customers
WHERE customerNumber = 900001;
```

```sql
SELECT orderNumber, status, customerNumber
FROM orders
WHERE orderNumber = 990001;
```

```sql
SELECT orderNumber, productCode, quantityOrdered, priceEach
FROM orderdetails
WHERE orderNumber = 990001;
```

```sql
SELECT customerNumber, checkNumber, paymentDate, amount
FROM payments
WHERE customerNumber = 900001;
```

Kết thúc lab:

```sql
ROLLBACK;
```

Sau rollback, các dòng vừa chèn không còn tồn tại.

### Bài tập thực hành

**Bài 7.1.**

Trong một transaction, chèn khách hàng có mã `900010` vào bảng `customers`.

**Bài 7.2.**

Trong cùng transaction, chèn đơn hàng có mã `990010` cho khách hàng `900010`.

**Bài 7.3.**

Trong cùng transaction, dùng `INSERT ... SELECT` để thêm một dòng `orderdetails` cho đơn hàng `990010`.

**Bài 7.4.**

Trong cùng transaction, chèn một khoản thanh toán cho khách hàng `900010`.

**Bài 7.5.**

Viết các truy vấn kiểm tra bốn bảng sau khi chèn, rồi dùng `ROLLBACK` và kiểm tra lại để xác nhận dữ liệu đã biến mất.

---

## 8. SAVEPOINT: hoàn tác một phần transaction

### 8.1. Khái niệm

`SAVEPOINT` tạo một mốc bên trong transaction.

Sau đó, có thể:

- Rollback về mốc đó bằng `ROLLBACK TO SAVEPOINT`.
- Xóa mốc bằng `RELEASE SAVEPOINT`.

Cú pháp:

```sql
SAVEPOINT savepoint_name;
```

```sql
ROLLBACK TO SAVEPOINT savepoint_name;
```

```sql
RELEASE SAVEPOINT savepoint_name;
```

### 8.2. Ví dụ

```sql
START TRANSACTION;
```

```sql
UPDATE customers
SET creditLimit = creditLimit + 1000.00
WHERE customerNumber = 103;
```

```sql
SAVEPOINT after_customer_103;
```

```sql
UPDATE customers
SET creditLimit = creditLimit + 2000.00
WHERE customerNumber = 112;
```

Kiểm tra tạm thời:

```sql
SELECT customerNumber, customerName, creditLimit
FROM customers
WHERE customerNumber IN (103, 112);
```

Hoàn tác chỉ thay đổi sau savepoint:

```sql
ROLLBACK TO SAVEPOINT after_customer_103;
```

Sau đó:

- Cập nhật cho khách hàng `112` bị hoàn tác.
- Cập nhật cho khách hàng `103` vẫn còn trong transaction.

Kiểm tra:

```sql
SELECT customerNumber, customerName, creditLimit
FROM customers
WHERE customerNumber IN (103, 112);
```

Cuối cùng, hủy toàn bộ transaction thực hành:

```sql
ROLLBACK;
```

### 8.3. `RELEASE SAVEPOINT`

Nếu không còn cần một savepoint:

```sql
RELEASE SAVEPOINT after_customer_103;
```

Sau khi release, không thể `ROLLBACK TO SAVEPOINT after_customer_103` nữa.

### Bài tập thực hành

**Bài 8.1.**

Bắt đầu một transaction và tăng `creditLimit` của khách hàng `103` thêm `1000.00`.

**Bài 8.2.**

Tạo savepoint tên `after_first_update`.

**Bài 8.3.**

Tăng `creditLimit` của khách hàng `112` thêm `2000.00`.

**Bài 8.4.**

Dùng `ROLLBACK TO SAVEPOINT after_first_update` để hoàn tác thay đổi của khách hàng `112` nhưng giữ thay đổi của khách hàng `103`.

**Bài 8.5.**

Tạo một savepoint, dùng `RELEASE SAVEPOINT` để xóa nó, sau đó thử giải thích vì sao không thể rollback về savepoint đã release.

---

## 9. `COMMIT` và hậu quả của việc commit

`COMMIT` xác nhận transaction và lưu vĩnh viễn các thay đổi.

Ví dụ:

```sql
START TRANSACTION;

UPDATE customers
SET creditLimit = creditLimit + 1000.00
WHERE customerNumber = 103;

COMMIT;
```

Sau `COMMIT`:

```sql
ROLLBACK;
```

không thể hoàn tác cập nhật đã được commit bằng transaction hiện tại.

### Khi nào nên dùng `COMMIT`?

Chỉ commit khi:

- Dữ liệu đã được kiểm tra.
- Toàn bộ các bước cần thiết đã hoàn thành.
- Không có lỗi constraint hoặc lỗi nghiệp vụ.
- Người dùng/ứng dụng thực sự muốn lưu thay đổi.
- Có cơ chế backup hoặc khôi phục phù hợp nếu cần.

### Khi nào nên dùng `ROLLBACK`?

Dùng rollback khi:

- Có bước trong transaction bị lỗi.
- Kiểm tra dữ liệu cho thấy kết quả chưa đúng.
- Người dùng hủy nghiệp vụ.
- Chỉ đang thực hành hoặc thử nghiệm.

### Bài tập thực hành

**Bài 9.1.**

Giải thích vì sao không nên dùng `COMMIT` ngay sau câu `UPDATE` đầu tiên trong một nghiệp vụ gồm nhiều bước.

**Bài 9.2.**

Viết transaction gồm hai câu `UPDATE`, sau đó dùng `COMMIT`. Chỉ chạy trên database cá nhân hoặc schema lab riêng.

**Bài 9.3.**

Sau Bài 9.2, thử chạy `ROLLBACK` và giải thích vì sao các thay đổi đã commit không quay lại.

**Bài 9.4.**

Nêu một tình huống nghiệp vụ cần dùng `ROLLBACK` thay vì `COMMIT`.

**Bài 9.5.**

Nêu ba việc cần kiểm tra trước khi commit transaction tạo đơn hàng.

---

## 10. Transaction, lỗi SQL và xử lý lỗi

Một lỗi trong một câu lệnh không phải lúc nào cũng tự động hoàn tác toàn bộ các câu lệnh trước đó trong transaction.

Ví dụ:

```sql
START TRANSACTION;
```

```sql
UPDATE customers
SET creditLimit = creditLimit + 1000.00
WHERE customerNumber = 103;
```

```sql
INSERT INTO orders (
    orderNumber,
    orderDate,
    requiredDate,
    status,
    customerNumber
)
VALUES (
    990099,
    '2026-06-23',
    '2026-06-30',
    'In Process',
    9999999
);
```

Nếu khách hàng `9999999` không tồn tại, lệnh `INSERT` có thể lỗi do foreign key. Tuy nhiên, không nên giả định rằng mọi thay đổi trước đó đã tự động bị hủy.

Khi cần bảo đảm toàn bộ nghiệp vụ là atomic, ứng dụng hoặc người thực hiện cần chủ động:

```sql
ROLLBACK;
```

### Mẫu xử lý logic

```text
START TRANSACTION

Thực hiện bước 1
Nếu lỗi: ROLLBACK

Thực hiện bước 2
Nếu lỗi: ROLLBACK

...

Nếu mọi bước thành công: COMMIT
```

Trong code ứng dụng, logic này thường được đặt trong khối `try/catch` hoặc cơ chế tương đương.

### Bài tập thực hành

**Bài 10.1.**

Viết transaction gồm một `UPDATE` hợp lệ và một `INSERT` vào `orders` có `customerNumber` không tồn tại.

**Bài 10.2.**

Sau khi lệnh `INSERT` lỗi, hãy dùng `ROLLBACK` để hủy thay đổi `UPDATE` trước đó.

**Bài 10.3.**

Giải thích vì sao không nên giả định mọi lỗi SQL tự động rollback toàn bộ transaction.

**Bài 10.4.**

Nêu một ví dụ lỗi khóa chính có thể xảy ra trong transaction.

**Bài 10.5.**

Viết pseudocode xử lý transaction gồm `START TRANSACTION`, các bước thao tác, `COMMIT` khi thành công và `ROLLBACK` khi lỗi.

---

## 11. DDL, implicit commit và các câu lệnh không rollback như DML

### 11.1. DDL là gì?

DDL (*Data Definition Language*) là nhóm câu lệnh thay đổi cấu trúc database, ví dụ:

```sql
CREATE TABLE ...
ALTER TABLE ...
DROP TABLE ...
CREATE DATABASE ...
DROP DATABASE ...
```

### 11.2. Lưu ý quan trọng

Nhiều câu lệnh DDL trong MySQL có thể gây **implicit commit**.

Điều này có nghĩa là transaction hiện tại có thể bị commit ngầm trước hoặc sau khi chạy câu lệnh DDL, tùy loại lệnh.

Ví dụ nguy hiểm trong một transaction:

```sql
START TRANSACTION;

UPDATE customers
SET creditLimit = creditLimit + 1000.00
WHERE customerNumber = 103;

CREATE TABLE transaction_test (
    id INT PRIMARY KEY
);

ROLLBACK;
```

Không nên kỳ vọng `ROLLBACK` ở cuối sẽ hoàn tác thay đổi như khi chỉ dùng DML, vì lệnh DDL có thể đã làm transaction commit ngầm.

### 11.3. Quy tắc thực hành

- Không trộn DDL và DML trong một transaction nếu kỳ vọng rollback toàn bộ.
- Tách thay đổi cấu trúc (`CREATE`, `ALTER`, `DROP`) khỏi transaction nghiệp vụ (`INSERT`, `UPDATE`, `DELETE`).
- Trước khi chạy migration/schema change, cần backup và kế hoạch khôi phục riêng.

### 11.4. Một số lệnh thường cần thận trọng

| Nhóm lệnh | Ví dụ |
|---|---|
| Tạo/xóa database | `CREATE DATABASE`, `DROP DATABASE` |
| Tạo/xóa bảng | `CREATE TABLE`, `DROP TABLE` |
| Thay đổi bảng | `ALTER TABLE` |
| Một số thao tác quản trị | `LOCK TABLES`, `UNLOCK TABLES`, một số `FLUSH` |

> Luôn kiểm tra tài liệu của phiên bản MySQL đang dùng vì danh sách lệnh implicit commit có thể rộng hơn các ví dụ trong bảng.

### Bài tập thực hành

**Bài 11.1.**

Nêu ba ví dụ lệnh DDL.

**Bài 11.2.**

Giải thích implicit commit là gì.

**Bài 11.3.**

Vì sao không nên đặt `ALTER TABLE` giữa một transaction nghiệp vụ?

**Bài 11.4.**

Phân loại các lệnh sau thành DDL hoặc DML: `INSERT`, `CREATE TABLE`, `UPDATE`, `DROP TABLE`, `DELETE`.

**Bài 11.5.**

Viết một quy tắc ngắn cho team phát triển về việc tách database migration và transaction nghiệp vụ.

---

## 12. Isolation level: giới thiệu cơ bản

Khi nhiều transaction chạy đồng thời, database cần xác định mức độ mà một transaction có thể nhìn thấy thay đổi chưa commit hoặc thay đổi vừa commit của transaction khác.

Trong InnoDB, các isolation level phổ biến gồm:

| Isolation level | Ý nghĩa khái quát |
|---|---|
| `READ UNCOMMITTED` | Có thể cho phép đọc dữ liệu chưa commit trong một số tình huống |
| `READ COMMITTED` | Mỗi lần đọc nhất quán thường thấy dữ liệu đã commit tại thời điểm đọc |
| `REPEATABLE READ` | Các lần đọc nhất quán trong một transaction có xu hướng thấy cùng snapshot; đây là mặc định phổ biến của InnoDB |
| `SERIALIZABLE` | Mức cô lập cao hơn, hạn chế nhiều hiện tượng đồng thời hơn nhưng có thể tăng blocking |

Kiểm tra isolation level hiện tại:

```sql
SELECT @@transaction_isolation;
```

Đặt isolation level cho transaction kế tiếp:

```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

Sau đó:

```sql
START TRANSACTION;

-- Các câu lệnh trong transaction

COMMIT;
```

> Trong bài học nhập môn, cần nhớ hai ý: isolation giúp quản lý truy cập đồng thời; isolation cao hơn thường làm giảm một số rủi ro nhất quán nhưng có thể tăng mức chờ khóa hoặc giảm khả năng song song.

### Bài tập thực hành

**Bài 12.1.**

Viết truy vấn kiểm tra isolation level hiện tại của session.

**Bài 12.2.**

Nêu isolation level mặc định phổ biến của InnoDB.

**Bài 12.3.**

Viết lệnh đặt isolation level `READ COMMITTED` cho transaction kế tiếp.

**Bài 12.4.**

Giải thích tại sao mức isolation cao hơn có thể làm tăng blocking.

**Bài 12.5.**

Nêu một tình huống hai người dùng cùng sửa tồn kho có liên quan đến isolation và locking.

---

## 13. Transaction và khóa ngoại

Khóa ngoại giúp duy trì tính nhất quán giữa bảng cha và bảng con.

Ví dụ:

```text
customers
    └── orders
            └── orderdetails
```

Trong một transaction:

- Có thể chèn customer rồi chèn order tham chiếu customer đó.
- Có thể chèn order rồi chèn orderdetails tham chiếu order đó.
- Nếu rollback toàn bộ, các thay đổi liên quan cũng được hoàn tác cùng nhau nếu các bảng dùng InnoDB và không có implicit commit.

Ví dụ tạo chuỗi dữ liệu rồi rollback:

```sql
START TRANSACTION;
```

```sql
INSERT INTO customers (
    customerNumber,
    customerName,
    contactLastName,
    contactFirstName,
    phone,
    addressLine1,
    city,
    country,
    creditLimit
)
VALUES (
    900020,
    'Foreign Key Transaction Co.',
    'Tran',
    'Binh',
    '+84 24 9999 9999',
    '20 Nguyen Trai',
    'Hanoi',
    'Vietnam',
    30000.00
);
```

```sql
INSERT INTO orders (
    orderNumber,
    orderDate,
    requiredDate,
    shippedDate,
    status,
    customerNumber
)
VALUES (
    990020,
    '2026-06-23',
    '2026-06-30',
    NULL,
    'In Process',
    900020
);
```

```sql
ROLLBACK;
```

Sau rollback, customer `900020` và order `990020` đều không còn.

### Bài tập thực hành

**Bài 13.1.**

Trong một transaction, chèn customer `900020` rồi chèn order `990020` tham chiếu đến customer đó.

**Bài 13.2.**

Dùng `ROLLBACK` và kiểm tra customer `900020` không còn tồn tại.

**Bài 13.3.**

Giải thích vì sao không thể chèn order cho `customerNumber = 9999999` nếu customer đó không tồn tại.

**Bài 13.4.**

Viết thứ tự hợp lý khi chèn dữ liệu có quan hệ: `customers`, `orders`, `orderdetails`.

**Bài 13.5.**

Viết thứ tự hợp lý khi xóa dữ liệu có quan hệ: `customers`, `orders`, `orderdetails`, `payments`.

---

## 14. Một số lỗi thường gặp

### Lỗi 1. Quên `ROLLBACK` trong môi trường thực hành

Nếu bạn chạy các câu lệnh DML trong transaction nhưng quên rollback hoặc commit, session vẫn giữ transaction mở. Điều này có thể giữ lock lâu hơn cần thiết.

**Cách phòng tránh:** luôn kết thúc lab bằng `ROLLBACK` hoặc `COMMIT` có chủ đích.

---

### Lỗi 2. Dùng `COMMIT` quá sớm

Sai về quy trình:

```sql
START TRANSACTION;

INSERT INTO orders (...);

COMMIT;

INSERT INTO orderdetails (...);
```

Nếu bước thêm `orderdetails` lỗi, đơn hàng đã commit có thể trở thành dữ liệu chưa hoàn chỉnh.

**Cách phòng tránh:** chỉ commit sau khi mọi bước bắt buộc hoàn thành.

---

### Lỗi 3. Dùng `ROLLBACK` sau khi autocommit đã lưu thay đổi

```sql
UPDATE customers
SET creditLimit = creditLimit + 1000.00
WHERE customerNumber = 103;

ROLLBACK;
```

Nếu autocommit bật và không có `START TRANSACTION`, rollback không hoàn tác thay đổi đã tự commit.

**Cách phòng tránh:** dùng `START TRANSACTION` trước các thao tác thực hành.

---

### Lỗi 4. Trộn DDL vào transaction nghiệp vụ

```sql
START TRANSACTION;

UPDATE customers
SET creditLimit = creditLimit + 1000.00
WHERE customerNumber = 103;

ALTER TABLE customers ADD COLUMN test_col INT;

ROLLBACK;
```

Không nên kỳ vọng rollback hoạt động như với DML thuần túy vì `ALTER TABLE` có thể gây implicit commit.

---

### Lỗi 5. Không kiểm tra storage engine

Nếu bảng không hỗ trợ transaction, `ROLLBACK` có thể không hoàn tác thay đổi mong muốn.

---

### Lỗi 6. Transaction quá dài

Transaction mở lâu có thể:

- Giữ lock.
- Làm transaction khác chờ.
- Tăng nguy cơ deadlock hoặc lock wait.
- Tăng tài nguyên cần thiết cho undo log và quản lý phiên bản dữ liệu.

**Cách phòng tránh:** giữ transaction ngắn, chỉ chứa các thao tác cần thiết, tránh chờ thao tác người dùng trong khi transaction đang mở.

### Bài tập thực hành

**Bài 14.1.**

Giải thích vì sao `COMMIT` sau bước đầu tiên của quy trình tạo đơn hàng có thể gây dữ liệu dở dang.

**Bài 14.2.**

Giải thích vì sao `ROLLBACK` không giúp được sau một câu DML đã autocommit.

**Bài 14.3.**

Nêu hậu quả của một transaction mở quá lâu.

**Bài 14.4.**

Giải thích vì sao không nên trộn `ALTER TABLE` vào transaction nghiệp vụ.

**Bài 14.5.**

Viết checklist gồm bốn bước để thực hành DML an toàn trong transaction.

---

## 15. Bài tập tổng hợp

### Bài 15.1. Transaction cập nhật hai khách hàng

Trong một transaction:

1. Kiểm tra `creditLimit` của khách hàng `103` và `112`.
2. Tăng `creditLimit` của khách hàng `103` thêm `1000.00`.
3. Tăng `creditLimit` của khách hàng `112` thêm `2000.00`.
4. Kiểm tra lại hai dòng.
5. Dùng `ROLLBACK`.
6. Kiểm tra lại để xác nhận dữ liệu quay về ban đầu.

---

### Bài 15.2. Transaction với savepoint

Trong một transaction:

1. Tăng `creditLimit` của khách hàng `103` thêm `500.00`.
2. Tạo savepoint `after_103`.
3. Tăng `creditLimit` của khách hàng `112` thêm `1000.00`.
4. Rollback về `after_103`.
5. Kiểm tra hai khách hàng.
6. Rollback toàn bộ transaction.

---

### Bài 15.3. Transaction tạo đơn hàng thực hành

Trong một transaction:

1. Chèn customer `900030`.
2. Chèn order `990030` cho customer `900030`.
3. Chèn một `orderdetails` bằng `INSERT ... SELECT` từ `products`.
4. Chèn payment cho customer `900030`.
5. Kiểm tra toàn bộ dữ liệu.
6. Rollback.

---

### Bài 15.4. Phân tích implicit commit

Cho transaction sau:

```sql
START TRANSACTION;

UPDATE customers
SET creditLimit = creditLimit + 1000.00
WHERE customerNumber = 103;

CREATE TABLE temp_transaction_example (
    id INT PRIMARY KEY
);

ROLLBACK;
```

Hãy trả lời:

1. Lệnh nào là DML?
2. Lệnh nào là DDL?
3. Vì sao rollback ở cuối có thể không hoạt động như mong muốn?
4. Cần thay đổi quy trình như thế nào?

---

### Bài 15.5. Thiết kế quy trình transaction cho thanh toán đơn hàng

Một hệ thống nhận thanh toán cần:

1. Thêm payment.
2. Cập nhật trạng thái order thành `Paid` hoặc trạng thái nghiệp vụ tương ứng.
3. Ghi log thanh toán.
4. Nếu bất kỳ bước nào lỗi, không được lưu bước nào.

Hãy viết pseudocode transaction và giải thích vị trí đặt `COMMIT`, `ROLLBACK`.

---

## 16. Đáp án gợi ý cho một số bài tập

### Bài 5.1

```sql
SELECT @@autocommit;
```

### Bài 5.2

```sql
SET autocommit = 0;
```

### Bài 5.3

```sql
SET autocommit = 1;
```

### Bài 6.5

```sql
START TRANSACTION;

UPDATE customers
SET creditLimit = creditLimit + 1000.00
WHERE customerNumber = 103;

UPDATE customers
SET creditLimit = creditLimit + 2000.00
WHERE customerNumber = 112;

ROLLBACK;
```

### Bài 8.4

```sql
START TRANSACTION;

UPDATE customers
SET creditLimit = creditLimit + 1000.00
WHERE customerNumber = 103;

SAVEPOINT after_first_update;

UPDATE customers
SET creditLimit = creditLimit + 2000.00
WHERE customerNumber = 112;

ROLLBACK TO SAVEPOINT after_first_update;

ROLLBACK;
```

### Bài 10.5

```text
START TRANSACTION

TRY
    Thực hiện bước 1
    Thực hiện bước 2
    Thực hiện bước 3
    COMMIT
CATCH lỗi
    ROLLBACK
END
```

### Bài 12.1

```sql
SELECT @@transaction_isolation;
```

### Bài 12.3

```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

### Bài 13.4

```text
customers → orders → orderdetails
```

### Bài 13.5

```text
payments → orderdetails → orders → customers
```

### Bài 15.1

```sql
START TRANSACTION;

SELECT customerNumber, customerName, creditLimit
FROM customers
WHERE customerNumber IN (103, 112);

UPDATE customers
SET creditLimit = creditLimit + 1000.00
WHERE customerNumber = 103;

UPDATE customers
SET creditLimit = creditLimit + 2000.00
WHERE customerNumber = 112;

SELECT customerNumber, customerName, creditLimit
FROM customers
WHERE customerNumber IN (103, 112);

ROLLBACK;

SELECT customerNumber, customerName, creditLimit
FROM customers
WHERE customerNumber IN (103, 112);
```

---

## 17. Tóm tắt

Các kiến thức chính trong tutorial:

- Transaction là một đơn vị công việc gồm một hoặc nhiều câu lệnh SQL.
- ACID gồm Atomicity, Consistency, Isolation và Durability.
- `START TRANSACTION` bắt đầu transaction; `COMMIT` lưu thay đổi; `ROLLBACK` hủy thay đổi chưa commit.
- MySQL thường chạy với autocommit bật; vì vậy cần dùng transaction rõ ràng khi muốn quản lý nhiều câu lệnh cùng nhau.
- `SAVEPOINT` cho phép rollback một phần transaction.
- Các bảng cần dùng storage engine hỗ trợ transaction, thường là InnoDB, để rollback hoạt động như mong muốn.
- Không nên trộn DDL như `CREATE TABLE`, `ALTER TABLE`, `DROP TABLE` với transaction nghiệp vụ nếu kỳ vọng rollback toàn bộ, vì một số lệnh có thể gây implicit commit.
- Transaction nên ngắn, có mục tiêu rõ ràng và luôn được kết thúc bằng `COMMIT` hoặc `ROLLBACK`.
- Khi thực hành trên `classicmodels`, nên dùng `ROLLBACK` để không làm thay đổi dữ liệu gốc.

---

## 18. Từ khóa chính

- Transaction
- ACID
- Atomicity
- Consistency
- Isolation
- Durability
- START TRANSACTION
- COMMIT
- ROLLBACK
- SAVEPOINT
- ROLLBACK TO SAVEPOINT
- RELEASE SAVEPOINT
- autocommit
- InnoDB
- Storage Engine
- Implicit Commit
- DDL
- DML
- Isolation Level
- Lock
- Foreign Key
- classicmodels
