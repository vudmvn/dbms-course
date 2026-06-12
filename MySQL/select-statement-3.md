---
title: "Lab: Thực hành các hàm SQL với classicmodels"
author: "Tên giảng viên"
duration: "120m"
difficulty: "Beginner - Intermediate"
prerequisites:
    - "Đã biết SELECT, FROM, WHERE, ORDER BY, LIMIT"
    - "Đã biết cách sử dụng cơ sở dữ liệu classicmodels"
    - "Khuyến nghị đã biết JOIN cơ bản"
summary: "Thực hành các nhóm hàm SQL thường dùng trong MySQL: hàm chuỗi, hàm số học, hàm ngày tháng, hàm xử lý NULL, hàm điều kiện và hàm tổng hợp trên cơ sở dữ liệu classicmodels."
---

# Lab: Thực hành các hàm SQL với `classicmodels`

## 1. Mục tiêu học tập

Sau khi hoàn thành lab này, người học có thể:

1. Hiểu vai trò của hàm trong câu lệnh SQL.
2. Sử dụng các hàm xử lý chuỗi như `CONCAT`, `UPPER`, `LOWER`, `LENGTH`, `SUBSTRING`, `TRIM`.
3. Sử dụng các hàm số học như `ROUND`, `CEIL`, `FLOOR`, `ABS`.
4. Sử dụng các hàm ngày tháng như `YEAR`, `MONTH`, `DAY`, `DATEDIFF`, `DATE_FORMAT`.
5. Xử lý giá trị rỗng bằng `IFNULL`, `COALESCE`, `NULLIF`.
6. Viết biểu thức điều kiện bằng `IF` và `CASE WHEN`.
7. Sử dụng các hàm tổng hợp cơ bản như `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`.
9. Viết được các truy vấn phục vụ phân tích dữ liệu đơn giản trên `classicmodels`.

---

## 2. Bối cảnh cơ sở dữ liệu mẫu

`classicmodels` là cơ sở dữ liệu mẫu mô phỏng hoạt động kinh doanh của một công ty bán các sản phẩm mô hình như xe cổ, xe máy, máy bay, tàu thuyền và các sản phẩm sưu tầm.

Một số bảng sẽ được dùng trong lab:

| Bảng | Ý nghĩa | Một số cột hay dùng |
|---|---|---|
| `products` | Sản phẩm | `productCode`, `productName`, `productLine`, `quantityInStock`, `buyPrice`, `MSRP` |
| `customers` | Khách hàng | `customerNumber`, `customerName`, `city`, `country`, `creditLimit`, `salesRepEmployeeNumber` |
| `employees` | Nhân viên | `employeeNumber`, `lastName`, `firstName`, `email`, `jobTitle`, `reportsTo` |
| `orders` | Đơn hàng | `orderNumber`, `orderDate`, `requiredDate`, `shippedDate`, `status`, `customerNumber` |
| `orderdetails` | Chi tiết đơn hàng | `orderNumber`, `productCode`, `quantityOrdered`, `priceEach` |
| `payments` | Thanh toán | `customerNumber`, `checkNumber`, `paymentDate`, `amount` |
| `offices` | Văn phòng | `officeCode`, `city`, `phone`, `country`, `territory` |

---

## 3. Chuẩn bị môi trường

Chọn cơ sở dữ liệu:

```sql
USE classicmodels;
```

Xem danh sách bảng:

```sql
SHOW TABLES;
```

Xem cấu trúc bảng sản phẩm:

```sql
DESC products;
```

Xem cấu trúc bảng đơn hàng:

```sql
DESC orders;
```

### Bài tập thực hành

**Bài 3.1.** Viết lệnh chọn cơ sở dữ liệu `classicmodels`.

**Bài 3.2.** Viết lệnh xem danh sách bảng trong cơ sở dữ liệu hiện tại.

**Bài 3.3.** Viết lệnh xem cấu trúc bảng `products`.

**Bài 3.4.** Viết lệnh xem cấu trúc bảng `orders`.

**Bài 3.5.** Viết lệnh xem cấu trúc bảng `customers`.

---

## 4. Hàm SQL là gì?

Hàm SQL là công cụ dùng để xử lý dữ liệu ngay trong câu truy vấn.

Hàm có thể nhận một hoặc nhiều giá trị đầu vào, sau đó trả về một giá trị kết quả.

Ví dụ:

```sql
SELECT 
    productName,
    UPPER(productName) AS upperProductName
FROM products;
```

Trong ví dụ trên:

- `UPPER(productName)` là hàm xử lý chuỗi.
- `productName` là giá trị đầu vào.
- `upperProductName` là bí danh của cột kết quả.

Hàm SQL có thể xuất hiện trong:

- Phần `SELECT` để tạo cột hiển thị mới.
- Phần `WHERE` để lọc dữ liệu theo giá trị đã xử lý.
- Phần `ORDER BY` để sắp xếp theo giá trị đã xử lý.

---

## 5. Hàm chuỗi: `CONCAT`

`CONCAT` dùng để nối nhiều chuỗi thành một chuỗi.

Ví dụ tạo họ tên đầy đủ của nhân viên:

```sql
SELECT 
    employeeNumber,
    CONCAT(firstName, ' ', lastName) AS fullName
FROM employees;
```

Ví dụ tạo tên người liên hệ của khách hàng:

```sql
SELECT 
    customerNumber,
    CONCAT(contactFirstName, ' ', contactLastName) AS contactName
FROM customers;
```

### Bài tập thực hành

**Bài 5.1.** Tạo cột `fullName` bằng cách nối `firstName` và `lastName` trong bảng `employees`.

**Bài 5.2.** Tạo cột `contactName` bằng cách nối `contactFirstName` và `contactLastName` trong bảng `customers`.

**Bài 5.3.** Tạo cột `addressInfo` từ `city` và `country` trong bảng `customers`.

**Bài 5.4.** Tạo cột `productInfo` từ `productCode`, `productName` và `productLine` trong bảng `products`.

**Bài 5.5.** Tạo cột `officeInfo` từ `city`, `country` và `territory` trong bảng `offices`.

---

## 6. Hàm chuỗi: `UPPER` và `LOWER`

`UPPER` chuyển chuỗi thành chữ in hoa.

`LOWER` chuyển chuỗi thành chữ thường.

Ví dụ:

```sql
SELECT 
    customerName,
    UPPER(customerName) AS upperName,
    LOWER(customerName) AS lowerName
FROM customers;
```

Ví dụ chuẩn hóa email nhân viên thành chữ thường:

```sql
SELECT 
    employeeNumber,
    email,
    LOWER(email) AS normalizedEmail
FROM employees;
```

### Bài tập thực hành

**Bài 6.1.** Hiển thị `customerName` và phiên bản in hoa của `customerName`.

**Bài 6.2.** Hiển thị `productName` và phiên bản chữ thường của `productName`.

**Bài 6.3.** Hiển thị `email` của nhân viên ở dạng chữ thường.

**Bài 6.4.** Hiển thị `country` trong bảng `customers` ở dạng chữ in hoa.

**Bài 6.5.** Hiển thị `productLine` trong bảng `products` ở dạng chữ thường và loại bỏ trùng lặp bằng `DISTINCT`.

---

## 7. Hàm chuỗi: `LENGTH` và `CHAR_LENGTH`

`LENGTH` trả về số byte của chuỗi.

`CHAR_LENGTH` trả về số ký tự của chuỗi.

Với dữ liệu tiếng Anh, hai giá trị này thường giống nhau. Với dữ liệu có ký tự Unicode, kết quả có thể khác nhau.

Ví dụ đo độ dài tên sản phẩm:

```sql
SELECT 
    productCode,
    productName,
    CHAR_LENGTH(productName) AS nameLength
FROM products;
```

Ví dụ tìm các sản phẩm có tên dài hơn 40 ký tự:

```sql
SELECT 
    productCode,
    productName,
    CHAR_LENGTH(productName) AS nameLength
FROM products
WHERE CHAR_LENGTH(productName) > 40;
```

### Bài tập thực hành

**Bài 7.1.** Hiển thị độ dài tên khách hàng bằng `CHAR_LENGTH(customerName)`.

**Bài 7.2.** Tìm các khách hàng có tên dài hơn 30 ký tự.

**Bài 7.3.** Hiển thị độ dài tên sản phẩm bằng `CHAR_LENGTH(productName)`.

**Bài 7.4.** Tìm các sản phẩm có tên dài hơn 35 ký tự.

**Bài 7.5.** Hiển thị độ dài email của từng nhân viên.

---

## 8. Hàm chuỗi: `SUBSTRING`, `LEFT`, `RIGHT`

`SUBSTRING` dùng để lấy một phần chuỗi.

Cú pháp:

```sql
SUBSTRING(text, start_position, length)
```

Trong MySQL, vị trí bắt đầu tính từ 1.

Ví dụ lấy 3 ký tự đầu của mã sản phẩm:

```sql
SELECT 
    productCode,
    SUBSTRING(productCode, 1, 3) AS prefix
FROM products;
```

`LEFT(text, n)` lấy `n` ký tự bên trái.

`RIGHT(text, n)` lấy `n` ký tự bên phải.

```sql
SELECT 
    productCode,
    LEFT(productCode, 3) AS leftPart,
    RIGHT(productCode, 4) AS rightPart
FROM products;
```

### Bài tập thực hành

**Bài 8.1.** Lấy 3 ký tự đầu của `productCode` trong bảng `products`.

**Bài 8.2.** Lấy 4 ký tự cuối của `productCode` trong bảng `products`.

**Bài 8.3.** Lấy 5 ký tự đầu của `customerName` trong bảng `customers`.

**Bài 8.4.** Lấy phần mở rộng điện thoại `extension` trong bảng `employees` và hiển thị 2 ký tự cuối.

**Bài 8.5.** Dùng `SUBSTRING` để lấy 4 ký tự đầu của `productScale` trong bảng `products`.

---

## 9. Hàm chuỗi: `TRIM`, `LTRIM`, `RTRIM`

`TRIM` loại bỏ khoảng trắng ở đầu và cuối chuỗi.

`LTRIM` loại bỏ khoảng trắng bên trái.

`RTRIM` loại bỏ khoảng trắng bên phải.

Ví dụ:

```sql
SELECT 
    customerNumber,
    customerName,
    country,
    TRIM(country) AS trimmedCountry
FROM customers;
```

Một số dữ liệu trong thực tế có thể bị dư khoảng trắng, đặc biệt khi nhập liệu thủ công.

### Bài tập thực hành

**Bài 9.1.** Hiển thị `country` và `TRIM(country)` từ bảng `customers`.

**Bài 9.2.** Hiển thị `customerName` và `TRIM(customerName)` từ bảng `customers`.

**Bài 9.3.** Tìm khách hàng có `TRIM(country) = 'Norway'`.

**Bài 9.4.** Hiển thị `city` và `TRIM(city)` từ bảng `customers`.

**Bài 9.5.** Hiển thị `postalCode` và `TRIM(postalCode)` từ bảng `customers`.

---

## 10. Hàm số học: `ROUND`, `CEIL`, `FLOOR`

`ROUND(x, d)` làm tròn số `x` đến `d` chữ số thập phân.

`CEIL(x)` làm tròn lên.

`FLOOR(x)` làm tròn xuống.

Ví dụ làm tròn giá bán:

```sql
SELECT 
    productCode,
    productName,
    MSRP,
    ROUND(MSRP, 0) AS roundedMSRP,
    CEIL(MSRP) AS ceilMSRP,
    FLOOR(MSRP) AS floorMSRP
FROM products;
```

Ví dụ tính tỷ lệ giữa giá bán đề xuất và giá mua:

```sql
SELECT 
    productCode,
    productName,
    buyPrice,
    MSRP,
    ROUND(MSRP / buyPrice, 2) AS markupRatio
FROM products;
```

### Bài tập thực hành

**Bài 10.1.** Hiển thị `buyPrice`, `MSRP` và `ROUND(MSRP - buyPrice, 2)` cho từng sản phẩm.

**Bài 10.2.** Tính `markupRatio = MSRP / buyPrice` và làm tròn 2 chữ số thập phân.

**Bài 10.3.** Hiển thị `CEIL(MSRP)` cho từng sản phẩm.

**Bài 10.4.** Hiển thị `FLOOR(buyPrice)` cho từng sản phẩm.

**Bài 10.5.** Tính `lineTotal = quantityOrdered * priceEach` từ bảng `orderdetails` và làm tròn 2 chữ số thập phân.

---

## 11. Hàm số học: `ABS`

`ABS(x)` trả về giá trị tuyệt đối của `x`.

Ví dụ tính độ lệch giữa `MSRP` và `buyPrice`:

```sql
SELECT 
    productCode,
    productName,
    buyPrice,
    MSRP,
    ABS(MSRP - buyPrice) AS priceGap
FROM products;
```

Trong ví dụ trên, hiệu số vốn dĩ dương, nhưng `ABS` vẫn hữu ích khi không chắc chiều trừ.

### Bài tập thực hành

**Bài 11.1.** Tính `ABS(MSRP - buyPrice)` cho từng sản phẩm.

**Bài 11.2.** Tính `ABS(buyPrice - MSRP)` cho từng sản phẩm và so sánh với bài 11.1.

**Bài 11.3.** Tính độ lệch giữa `requiredDate` và `shippedDate` bằng `DATEDIFF`, sau đó lấy trị tuyệt đối bằng `ABS`.

**Bài 11.4.** Tính `ABS(amount - 50000)` cho từng khoản thanh toán.

**Bài 11.5.** Sắp xếp các khoản thanh toán theo khoảng cách đến 50000 tăng dần.

---

## 12. Hàm ngày tháng: `YEAR`, `MONTH`, `DAY`

`YEAR(date)` lấy năm.

`MONTH(date)` lấy tháng.

`DAY(date)` lấy ngày trong tháng.

Ví dụ tách ngày đặt hàng:

```sql
SELECT 
    orderNumber,
    orderDate,
    YEAR(orderDate) AS orderYear,
    MONTH(orderDate) AS orderMonth,
    DAY(orderDate) AS orderDay
FROM orders;
```

Ví dụ lấy các đơn hàng trong năm 2005:

```sql
SELECT 
    orderNumber,
    orderDate,
    status
FROM orders
WHERE YEAR(orderDate) = 2005;
```

### Bài tập thực hành

**Bài 12.1.** Hiển thị `orderDate`, năm, tháng và ngày của từng đơn hàng.

**Bài 12.2.** Lấy các đơn hàng có `orderDate` trong năm 2004.

**Bài 12.3.** Lấy các khoản thanh toán có `paymentDate` trong năm 2005.

**Bài 12.4.** Lấy các đơn hàng có `orderDate` trong tháng 11.

**Bài 12.5.** Lấy các khoản thanh toán có `paymentDate` trong tháng 12 năm 2004.

---

## 13. Hàm ngày tháng: `DATEDIFF`

`DATEDIFF(date1, date2)` trả về số ngày giữa hai ngày:

```sql
DATEDIFF(date1, date2) = date1 - date2
```

Ví dụ tính số ngày từ lúc đặt hàng đến hạn yêu cầu:

```sql
SELECT 
    orderNumber,
    orderDate,
    requiredDate,
    DATEDIFF(requiredDate, orderDate) AS daysAllowed
FROM orders;
```

Ví dụ tính số ngày giao hàng thực tế:

```sql
SELECT 
    orderNumber,
    orderDate,
    shippedDate,
    DATEDIFF(shippedDate, orderDate) AS daysToShip
FROM orders
WHERE shippedDate IS NOT NULL;
```

Ví dụ kiểm tra đơn hàng giao sau ngày yêu cầu:

```sql
SELECT 
    orderNumber,
    requiredDate,
    shippedDate,
    DATEDIFF(shippedDate, requiredDate) AS delayDays
FROM orders
WHERE shippedDate IS NOT NULL
  AND DATEDIFF(shippedDate, requiredDate) > 0;
```

### Bài tập thực hành

**Bài 13.1.** Tính số ngày từ `orderDate` đến `requiredDate` cho từng đơn hàng.

**Bài 13.2.** Tính số ngày từ `orderDate` đến `shippedDate` cho các đơn hàng đã giao.

**Bài 13.3.** Tìm các đơn hàng có `shippedDate` muộn hơn `requiredDate`.

**Bài 13.4.** Tính số ngày chênh lệch giữa `requiredDate` và `shippedDate`.

**Bài 13.5.** Lấy 10 đơn hàng có thời gian giao hàng thực tế lâu nhất.

---

## 14. Hàm ngày tháng: `DATE_FORMAT`

`DATE_FORMAT` dùng để định dạng ngày tháng khi hiển thị.

Ví dụ:

```sql
SELECT 
    orderNumber,
    orderDate,
    DATE_FORMAT(orderDate, '%d/%m/%Y') AS formattedOrderDate
FROM orders;
```

Một số định dạng thường dùng:

| Mẫu | Ý nghĩa | Ví dụ |
|---|---|---|
| `%Y` | Năm 4 chữ số | `2005` |
| `%y` | Năm 2 chữ số | `05` |
| `%m` | Tháng 2 chữ số | `01` |
| `%d` | Ngày 2 chữ số | `09` |
| `%M` | Tên tháng đầy đủ | `January` |
| `%b` | Tên tháng viết tắt | `Jan` |

### Bài tập thực hành

**Bài 14.1.** Hiển thị `orderDate` theo định dạng `dd/mm/yyyy`.

**Bài 14.2.** Hiển thị `paymentDate` theo định dạng `dd/mm/yyyy`.

**Bài 14.3.** Hiển thị `orderDate` theo định dạng `Month Year`, ví dụ `January 2005`.

**Bài 14.4.** Hiển thị `requiredDate` theo định dạng `yyyy-mm`.

**Bài 14.5.** Hiển thị `shippedDate` theo định dạng `dd/mm/yyyy`, chỉ với các đơn hàng đã giao.

---

## 15. Hàm xử lý `NULL`: `IFNULL`

`IFNULL(value, replacement)` trả về `replacement` nếu `value` là `NULL`.

Ví dụ hiển thị trạng thái nhân viên bán hàng phụ trách khách hàng:

```sql
SELECT 
    customerNumber,
    customerName,
    IFNULL(salesRepEmployeeNumber, 'No Sales Rep') AS salesRep
FROM customers;
```

Ví dụ thay `state` bị rỗng bằng `Unknown`:

```sql
SELECT 
    customerNumber,
    customerName,
    city,
    IFNULL(state, 'Unknown') AS stateName,
    country
FROM customers;
```

### Bài tập thực hành

**Bài 15.1.** Dùng `IFNULL` để thay `state` bằng `Unknown` trong bảng `customers`.

**Bài 15.2.** Dùng `IFNULL` để thay `addressLine2` bằng `No address line 2` trong bảng `customers`.

**Bài 15.3.** Dùng `IFNULL` để hiển thị `salesRepEmployeeNumber`; nếu rỗng thì ghi `No Sales Rep`.

**Bài 15.4.** Dùng `IFNULL` để hiển thị `shippedDate`; nếu rỗng thì ghi `Not shipped yet`.

**Bài 15.5.** Dùng `IFNULL` để hiển thị `comments`; nếu rỗng thì ghi `No comment` trong bảng `orders`.

---

## 16. Hàm xử lý `NULL`: `COALESCE`

`COALESCE(value1, value2, ..., valueN)` trả về giá trị đầu tiên không phải `NULL`.

Ví dụ tạo địa chỉ phụ từ `addressLine2` hoặc `addressLine1`:

```sql
SELECT 
    customerNumber,
    customerName,
    COALESCE(addressLine2, addressLine1) AS availableAddress
FROM customers;
```

Ví dụ lấy thông tin địa phương ưu tiên `state`, nếu không có thì dùng `city`, nếu vẫn không có thì dùng `country`:

```sql
SELECT 
    customerNumber,
    customerName,
    COALESCE(state, city, country) AS locationInfo
FROM customers;
```

### Bài tập thực hành

**Bài 16.1.** Dùng `COALESCE(state, city, country)` để tạo cột `locationInfo` trong bảng `customers`.

**Bài 16.2.** Dùng `COALESCE(addressLine2, addressLine1)` để tạo cột `availableAddress`.

**Bài 16.3.** Dùng `COALESCE(comments, 'No comment')` trong bảng `orders`.

**Bài 16.4.** Dùng `COALESCE(shippedDate, requiredDate)` để tạo cột `referenceDate` trong bảng `orders`.

**Bài 16.5.** So sánh kết quả của `IFNULL(state, 'Unknown')` và `COALESCE(state, 'Unknown')`.

---

## 17. Hàm điều kiện: `IF`

`IF(condition, value_if_true, value_if_false)` trả về một trong hai giá trị tùy theo điều kiện.

Ví dụ phân loại sản phẩm theo giá bán đề xuất:

```sql
SELECT 
    productCode,
    productName,
    MSRP,
    IF(MSRP >= 100, 'High price', 'Normal price') AS priceGroup
FROM products;
```

Ví dụ kiểm tra khách hàng có hạn mức tín dụng hay không:

```sql
SELECT 
    customerNumber,
    customerName,
    creditLimit,
    IF(creditLimit > 0, 'Has credit', 'No credit') AS creditStatus
FROM customers;
```

### Bài tập thực hành

**Bài 17.1.** Phân loại sản phẩm thành `High price` nếu `MSRP >= 100`, ngược lại là `Normal price`.

**Bài 17.2.** Phân loại khách hàng thành `High credit` nếu `creditLimit >= 100000`, ngược lại là `Normal credit`.

**Bài 17.3.** Phân loại đơn hàng thành `Completed` nếu `status = 'Shipped'`, ngược lại là `Not completed`.

**Bài 17.4.** Phân loại sản phẩm thành `Low stock` nếu `quantityInStock < 1000`, ngược lại là `Enough stock`.

**Bài 17.5.** Phân loại khoản thanh toán thành `Large payment` nếu `amount >= 50000`, ngược lại là `Normal payment`.

---

## 18. Hàm điều kiện: `CASE WHEN`

`CASE WHEN` dùng khi có nhiều nhánh điều kiện.

Ví dụ phân loại sản phẩm theo mức giá:

```sql
SELECT 
    productCode,
    productName,
    MSRP,
    CASE
        WHEN MSRP >= 150 THEN 'Expensive'
        WHEN MSRP >= 80 THEN 'Medium'
        ELSE 'Low'
    END AS priceLevel
FROM products;
```

Ví dụ phân loại trạng thái đơn hàng:

```sql
SELECT 
    orderNumber,
    status,
    CASE
        WHEN status = 'Shipped' THEN 'Completed'
        WHEN status IN ('Cancelled', 'Disputed') THEN 'Problem'
        ELSE 'In progress'
    END AS statusGroup
FROM orders;
```

### Bài tập thực hành

**Bài 18.1.** Phân loại sản phẩm theo `MSRP`: từ 150 trở lên là `Expensive`, từ 80 đến dưới 150 là `Medium`, còn lại là `Low`.

**Bài 18.2.** Phân loại khách hàng theo `creditLimit`: từ 100000 trở lên là `High`, từ 50000 đến dưới 100000 là `Medium`, còn lại là `Low`.

**Bài 18.3.** Phân loại đơn hàng theo `status`: `Shipped` là `Completed`, `Cancelled` hoặc `Disputed` là `Problem`, còn lại là `Processing`.

**Bài 18.4.** Phân loại tồn kho sản phẩm: dưới 1000 là `Low`, từ 1000 đến dưới 5000 là `Medium`, còn lại là `High`.

**Bài 18.5.** Phân loại thanh toán theo `amount`: từ 50000 trở lên là `Large`, từ 10000 đến dưới 50000 là `Medium`, còn lại là `Small`.

---

## 19. Hàm tổng hợp: `COUNT`

`COUNT` dùng để đếm số dòng hoặc số giá trị không rỗng.

Đếm tổng số sản phẩm:

```sql
SELECT COUNT(*) AS totalProducts
FROM products;
```

Đếm số khách hàng có nhân viên bán hàng phụ trách:

```sql
SELECT COUNT(salesRepEmployeeNumber) AS customersWithSalesRep
FROM customers;
```

Đếm số quốc gia khác nhau có khách hàng:

```sql
SELECT COUNT(DISTINCT country) AS numberOfCountries
FROM customers;
```

### Bài tập thực hành

**Bài 19.1.** Đếm tổng số sản phẩm trong bảng `products`.

**Bài 19.2.** Đếm tổng số khách hàng trong bảng `customers`.

**Bài 19.3.** Đếm số đơn hàng trong bảng `orders`.

**Bài 19.4.** Đếm số khách hàng có `salesRepEmployeeNumber` khác `NULL`.

**Bài 19.5.** Đếm số quốc gia khác nhau trong bảng `customers`.

---

## 20. Hàm tổng hợp: `SUM`, `AVG`, `MIN`, `MAX`

`SUM` tính tổng.

`AVG` tính trung bình.

`MIN` lấy giá trị nhỏ nhất.

`MAX` lấy giá trị lớn nhất.

Ví dụ thống kê giá sản phẩm:

```sql
SELECT 
    MIN(MSRP) AS minMSRP,
    MAX(MSRP) AS maxMSRP,
    AVG(MSRP) AS avgMSRP
FROM products;
```

Ví dụ tính tổng tiền thanh toán:

```sql
SELECT SUM(amount) AS totalPayment
FROM payments;
```

Ví dụ tính tổng giá trị của một dòng chi tiết đơn hàng:

```sql
SELECT 
    orderNumber,
    productCode,
    quantityOrdered,
    priceEach,
    quantityOrdered * priceEach AS lineTotal
FROM orderdetails;
```

Ví dụ tính tổng doanh thu theo toàn bộ chi tiết đơn hàng:

```sql
SELECT 
    SUM(quantityOrdered * priceEach) AS totalSales
FROM orderdetails;
```

### Bài tập thực hành

**Bài 20.1.** Tính giá `MSRP` nhỏ nhất, lớn nhất và trung bình trong bảng `products`.

**Bài 20.2.** Tính tổng số lượng tồn kho `quantityInStock` của tất cả sản phẩm.

**Bài 20.3.** Tính tổng số tiền thanh toán trong bảng `payments`.

**Bài 20.4.** Tính khoản thanh toán nhỏ nhất, lớn nhất và trung bình.

**Bài 20.5.** Tính tổng doanh thu theo công thức `quantityOrdered * priceEach` trong bảng `orderdetails`.

---

## 21. Một số lỗi thường gặp

### Lỗi 1: Quên đặt bí danh cho cột tính toán

Vẫn chạy được nhưng kết quả khó đọc:

```sql
SELECT MSRP - buyPrice
FROM products;
```

Nên viết:

```sql
SELECT MSRP - buyPrice AS profitEstimate
FROM products;
```

---

### Lỗi 2: Dùng `=` để so sánh với `NULL`

Sai:

```sql
SELECT customerName, state
FROM customers
WHERE state = NULL;
```

Đúng:

```sql
SELECT customerName, state
FROM customers
WHERE state IS NULL;
```

---

### Lỗi 3: Không xử lý `NULL` trước khi hiển thị

Kết quả có thể khó đọc:

```sql
SELECT customerName, state
FROM customers;
```

Có thể cải thiện:

```sql
SELECT 
    customerName,
    IFNULL(state, 'Unknown') AS stateName
FROM customers;
```

---

## 22. Đáp án gợi ý cho một số bài tập

### Bài 5.1

```sql
SELECT 
    employeeNumber,
    CONCAT(firstName, ' ', lastName) AS fullName
FROM employees;
```

### Bài 6.1

```sql
SELECT 
    customerName,
    UPPER(customerName) AS upperCustomerName
FROM customers;
```

### Bài 7.2

```sql
SELECT 
    customerNumber,
    customerName,
    CHAR_LENGTH(customerName) AS nameLength
FROM customers
WHERE CHAR_LENGTH(customerName) > 30;
```

### Bài 8.1

```sql
SELECT 
    productCode,
    LEFT(productCode, 3) AS productPrefix
FROM products;
```

### Bài 10.2

```sql
SELECT 
    productCode,
    productName,
    buyPrice,
    MSRP,
    ROUND(MSRP / buyPrice, 2) AS markupRatio
FROM products;
```

### Bài 12.2

```sql
SELECT 
    orderNumber,
    orderDate,
    status
FROM orders
WHERE YEAR(orderDate) = 2004;
```

### Bài 13.3

```sql
SELECT 
    orderNumber,
    requiredDate,
    shippedDate,
    DATEDIFF(shippedDate, requiredDate) AS delayDays
FROM orders
WHERE shippedDate IS NOT NULL
  AND DATEDIFF(shippedDate, requiredDate) > 0;
```

### Bài 15.1

```sql
SELECT 
    customerNumber,
    customerName,
    IFNULL(state, 'Unknown') AS stateName
FROM customers;
```

### Bài 17.1

```sql
SELECT 
    productCode,
    productName,
    MSRP,
    IF(MSRP >= 100, 'High price', 'Normal price') AS priceGroup
FROM products;
```

### Bài 18.1

```sql
SELECT 
    productCode,
    productName,
    MSRP,
    CASE
        WHEN MSRP >= 150 THEN 'Expensive'
        WHEN MSRP >= 80 THEN 'Medium'
        ELSE 'Low'
    END AS priceLevel
FROM products;
```

### Bài 19.1

```sql
SELECT COUNT(*) AS totalProducts
FROM products;
```

### Bài 20.5

```sql
SELECT 
    SUM(quantityOrdered * priceEach) AS totalSales
FROM orderdetails;
```

## 23. Bài tập tổng hợp cuối lab

### Bài tổng hợp 1

Từ bảng `products`, hiển thị:

- `productCode`
- `productName`
- `productLine`
- `buyPrice`
- `MSRP`
- `profitEstimate = MSRP - buyPrice`
- `priceLevel` theo quy tắc:
  - `MSRP >= 150`: `Expensive`
  - `MSRP >= 80`: `Medium`
  - còn lại: `Low`

Sắp xếp theo `profitEstimate` giảm dần.

---

### Bài tổng hợp 2

Từ bảng `orders`, hiển thị:

- `orderNumber`
- `orderDate`
- `requiredDate`
- `shippedDate`
- số ngày từ đặt hàng đến hạn giao
- số ngày từ đặt hàng đến ngày giao thực tế
- trạng thái giao hàng:
  - nếu `shippedDate IS NULL`: `Not shipped`
  - nếu `shippedDate <= requiredDate`: `On time`
  - ngược lại: `Late`

---

### Bài tổng hợp 3

Từ bảng `payments`, hiển thị:

- `customerNumber`
- `checkNumber`
- `paymentDate`
- `amount`
- năm thanh toán
- tháng thanh toán
- phân loại thanh toán:
  - `amount >= 100000`: `Large payment`
  - `amount >= 50000`: `Medium payment`
  - còn lại: `Small payment`

Sắp xếp theo `amount` giảm dần và lấy 10 dòng đầu tiên.

---

### Bài tổng hợp 4

Từ bảng `orderdetails`, hiển thị:

- `orderNumber`
- `productCode`
- `quantityOrdered`
- `priceEach`
- `lineTotal = quantityOrdered * priceEach`
- phân loại dòng đơn hàng:
  - `lineTotal >= 10000`: `High value line`
  - `lineTotal >= 5000`: `Medium value line`
  - còn lại: `Low value line`

Sắp xếp theo `lineTotal` giảm dần và lấy 20 dòng đầu tiên.

---

## 24. Tóm tắt

Các nhóm hàm chính trong lab:

| Nhóm hàm | Ví dụ | Công dụng |
|---|---|---|
| Hàm chuỗi | `CONCAT`, `UPPER`, `LOWER`, `CHAR_LENGTH`, `SUBSTRING`, `TRIM` | Xử lý văn bản |
| Hàm số học | `ROUND`, `CEIL`, `FLOOR`, `ABS` | Tính toán và làm tròn số |
| Hàm ngày tháng | `YEAR`, `MONTH`, `DAY`, `DATEDIFF`, `DATE_FORMAT` | Xử lý dữ liệu ngày tháng |
| Hàm xử lý `NULL` | `IFNULL`, `COALESCE`, `NULLIF` | Thay thế hoặc kiểm tra giá trị rỗng |
| Hàm điều kiện | `IF`, `CASE WHEN` | Phân loại dữ liệu theo điều kiện |
| Hàm tổng hợp | `COUNT`, `SUM`, `AVG`, `MIN`, `MAX` | Tóm tắt dữ liệu |

Thứ tự thường gặp khi dùng hàm trong truy vấn phân tích:

```sql
SELECT columns_or_functions
FROM table_or_joined_tables
WHERE row_conditions
ORDER BY sorting_columns
LIMIT number;
```

---

## 25. Từ khóa chính

- SQL functions
- MySQL functions
- String functions
- Numeric functions
- Date functions
- Aggregate functions
- `CONCAT`
- `UPPER`
- `LOWER`
- `CHAR_LENGTH`
- `SUBSTRING`
- `TRIM`
- `ROUND`
- `CEIL`
- `FLOOR`
- `ABS`
- `YEAR`
- `MONTH`
- `DAY`
- `DATEDIFF`
- `DATE_FORMAT`
- `IFNULL`
- `COALESCE`
- `IF`
- `CASE WHEN`
- `COUNT`
- `SUM`
- `AVG`
- `MIN`
- `MAX`
- `classicmodels`
