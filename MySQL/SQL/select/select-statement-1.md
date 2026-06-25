---
title: "Tutorial: Sử dụng câu lệnh SELECT cơ bản với classicmodels"
author: "Tên giảng viên"
duration: "90m"
difficulty: "Beginner"
prerequisites:
    - "Đã biết khái niệm bảng, cột, dòng trong CSDL quan hệ"
    - "Đã biết cách mở MySQL client hoặc MySQL Workbench"
summary: "Thực hành truy vấn dữ liệu một bảng bằng SELECT trên cơ sở dữ liệu mẫu classicmodels."
---

# Tutorial: Sử dụng câu lệnh `SELECT` cơ bản với `classicmodels`

## Link tham khảo

- [MySQL SELECT FROM](https://www.mysqltutorial.org/mysql-basics/mysql-select-from/)
- [MySQL SELECT](https://www.mysqltutorial.org/mysql-basics/mysql-select/)
- [MySQL DISTINCT](https://www.mysqltutorial.org/mysql-basics/mysql-distinct/)
- [MySQL WHERE](https://www.mysqltutorial.org/mysql-basics/mysql-where/)

## 1. Mục tiêu học tập

Sau khi hoàn thành tutorial này, người học có thể:

1. Hiểu vai trò của câu lệnh `SELECT` trong SQL.
2. Truy vấn toàn bộ dữ liệu hoặc một số cột cụ thể từ một bảng.
3. Đặt bí danh cho cột bằng `AS`.
4. Lọc dữ liệu bằng `WHERE`.
5. Sử dụng các toán tử so sánh như `=`, `<>`, `>`, `<`, `>=`, `<=`.
6. Kết hợp điều kiện bằng `AND`, `OR`, `NOT`.
7. Tìm kiếm mẫu chuỗi bằng `LIKE`.
8. Lọc dữ liệu bằng `IN`, `BETWEEN`, `IS NULL`, `IS NOT NULL`.
9. Sắp xếp kết quả bằng `ORDER BY`.
10. Giới hạn số dòng kết quả bằng `LIMIT`.
11. Loại bỏ giá trị trùng lặp bằng `DISTINCT`.
12. Viết được các truy vấn một bảng đơn giản trên cơ sở dữ liệu `classicmodels`.

---

## 2. Giới thiệu cơ sở dữ liệu mẫu `classicmodels`

`classicmodels` là một cơ sở dữ liệu mẫu dùng để thực hành SQL. Cơ sở dữ liệu mô phỏng hoạt động của một công ty bán các mô hình xe, tàu, máy bay và một số sản phẩm sưu tầm.

Một số bảng chính trong cơ sở dữ liệu:

| Bảng | Ý nghĩa |
|---|---|
| `productlines` | Nhóm hoặc dòng sản phẩm |
| `products` | Thông tin sản phẩm |
| `offices` | Văn phòng của công ty |
| `employees` | Nhân viên |
| `customers` | Khách hàng |
| `payments` | Thanh toán |
| `orders` | Đơn hàng |
| `orderdetails` | Chi tiết đơn hàng |

Trong tutorial này, ta chỉ thực hành các truy vấn **một bảng**. Mỗi ví dụ chỉ đọc dữ liệu từ một bảng tại một thời điểm.

### Bài tập thực hành

**Bài 2.1.**

Xem danh sách cac bang chính trong `classicmodels` va ghi lại vai trò của từng bảng: `productlines`, `products`, `offices`, `employees`, `customers`, `payments`, `orders`, `orderdetails`.

**Bài 2.2.**

Dựa trên schema, xác định bảng nào lưu thông tin sản phẩm va liệt kê ít nhất 5 cot của bảng đó.

**Bài 2.3.**

Dựa trên schema, xác định bảng nào lưu thông tin khách hàng va liệt kê các cột liên quan đến địa chỉ.

**Bài 2.4.**

Dựa trên schema, xác định bảng nào lưu thông tin đơn hàng va các cột ngày tháng trong bang do.

**Bài 2.5.**

Dựa trên schema, xác định bảng nào có khóa chính gồm nhiều cột va nêu các cột tạo thành khóa chính.

---

## 3. Chuẩn bị môi trường

Trước khi thực hành, cần chắc chắn rằng cơ sở dữ liệu đã được tạo và được chọn bằng lệnh:

```sql
USE classicmodels;
```

Để xem danh sách các bảng đang có trong cơ sở dữ liệu:

```sql
SHOW TABLES;
```

Để xem cấu trúc của một bảng:

```sql
DESC products;
```

Ví dụ xem cấu trúc bảng khách hàng:

```sql
DESC customers;
```

### Bài tập thực hành

**Bài 3.1.**

Viết lệnh chọn cơ sở dữ liệu `classicmodels` để bắt đầu thực hành.

**Bài 3.2.**

Viết lệnh xem danh sách tất cả bảng trong cơ sở dữ liệu hiện tại.

**Bài 3.3.**

Viết lệnh xem cấu trúc bảng `products`.

**Bài 3.4.**

Viết lệnh xem cấu trúc bảng `customers`.

**Bài 3.5.**

Viết lệnh xem cấu trúc bảng `orders` và xác định cột nào có thể nhận giá trị `NULL`.

---

## 4. Cú pháp cơ bản của `SELECT`

Câu lệnh `SELECT` dùng để lấy dữ liệu từ bảng.

Cú pháp tổng quát:

```sql
SELECT column1, column2, ...
FROM table_name;
```

Trong đó:

| Thành phần | Ý nghĩa |
|---|---|
| `SELECT` | Chỉ định các cột muốn hiển thị |
| `FROM` | Chỉ định bảng chứa dữ liệu |
| `column1, column2, ...` | Danh sách các cột cần lấy |
| `table_name` | Tên bảng cần truy vấn |

### Bài tập thực hành

**Bài 4.1.**

Viết khung truy vấn `SELECT ... FROM ...` để lấy dữ liệu từ bảng `employees`.

**Bài 4.2.**

Viết truy vấn lấy cột `customerName` từ bảng `customers`.

**Bài 4.3.**

Viết truy vấn lấy cột `orderNumber` từ bảng `orders`.

**Bài 4.4.**

Viết truy vấn lấy cột `productLine` từ bảng `productlines`.

**Bài 4.5.**

Viết truy vấn lấy cột `amount` từ bảng `payments`.

---

## 5. Lấy toàn bộ dữ liệu từ một bảng

Ký hiệu `*` nghĩa là lấy tất cả các cột trong bảng.

Ví dụ lấy toàn bộ dữ liệu từ bảng `productlines`:

```sql
SELECT *
FROM productlines;
```

Ví dụ lấy toàn bộ dữ liệu từ bảng `offices`:

```sql
SELECT *
FROM offices;
```

### Lưu ý

Trong thực tế, không nên dùng `SELECT *` quá nhiều nếu bảng có nhiều cột hoặc dữ liệu lớn. Khi học SQL, `SELECT *` hữu ích để xem nhanh dữ liệu trong bảng.

### Bài tập thực hành

**Bài 5.1.**

Viết truy vấn lấy toàn bộ dữ liệu từ bảng `offices`.

**Bài 5.2.**

Viết truy vấn lấy toàn bộ dữ liệu từ bảng `productlines`.

**Bài 5.3.**

Viết truy vấn lấy toàn bộ dữ liệu từ bảng `employees`.

**Bài 5.4.**

Viết truy vấn lấy toàn bộ dữ liệu từ bảng `orders`.

**Bài 5.5.**

Viết truy vấn lấy toàn bộ dữ liệu từ bảng `payments`.

---

## 6. Lấy một số cột cụ thể

Thông thường, ta chỉ lấy các cột cần thiết.

Ví dụ lấy mã sản phẩm, tên sản phẩm và dòng sản phẩm:

```sql
SELECT productCode, productName, productLine
FROM products;
```

Ví dụ lấy tên khách hàng, thành phố và quốc gia:

```sql
SELECT customerName, city, country
FROM customers;
```

Ví dụ lấy họ, tên và chức danh nhân viên:

```sql
SELECT lastName, firstName, jobTitle
FROM employees;
```

### Bài tập thực hành

**Bài 6.1.**

Viết truy vấn lấy `productCode`, `productName`, `productLine`, `MSRP` từ bảng `products`.

**Bài 6.2.**

Viết truy vấn lấy `customerNumber`, `customerName`, `city`, `country` từ bảng `customers`.

**Bài 6.3.**

Viết truy vấn lấy `employeeNumber`, `lastName`, `firstName`, `jobTitle` từ bảng `employees`.

**Bài 6.4.**

Viết truy vấn lấy `orderNumber`, `orderDate`, `status`, `customerNumber` từ bảng `orders`.

**Bài 6.5.**

Viết truy vấn lấy `checkNumber`, `paymentDate`, `amount` từ bảng `payments`.

---

## 7. Đặt bí danh cho cột bằng `AS`

Bí danh giúp tên cột hiển thị dễ đọc hơn.

Ví dụ:

```sql
SELECT 
    productCode AS MaSanPham,
    productName AS TenSanPham,
    productLine AS DongSanPham
FROM products;
```

Ví dụ với bảng `customers`:

```sql
SELECT 
    customerName AS TenKhachHang,
    phone AS SoDienThoai,
    country AS QuocGia
FROM customers;
```

Có thể bỏ từ khóa `AS`, nhưng nên dùng `AS` khi mới học để câu lệnh rõ ràng hơn:

```sql
SELECT 
    customerName TenKhachHang,
    country QuocGia
FROM customers;
```

### Bài tập thực hành

**Bài 7.1.**

Viết truy vấn lấy `customerName`, `city`, `country` từ bảng `customers` và đặt bí danh tiếng Việt không dấu cho các cột.

**Bài 7.2.**

Viết truy vấn lấy `productCode`, `productName`, `MSRP` từ bảng `products` và đặt bí danh `MaSanPham`, `TenSanPham`, `GiaBanDeXuat`.

**Bài 7.3.**

Viết truy vấn lấy `employeeNumber`, `lastName`, `firstName`, `email` từ bảng `employees` và đặt bí danh dễ đọc cho từng cột.

**Bài 7.4.**

Viết truy vấn lấy `orderNumber`, `orderDate`, `requiredDate`, `status` từ bảng `orders` và đặt bí danh dễ đọc.

**Bài 7.5.**

Viết truy vấn lấy `officeCode`, `city`, `phone`, `country` từ bảng `offices` và đặt bí danh cho kết quả.

---

## 8. Tạo cột tính toán đơn giản trong `SELECT`

Trong phần `SELECT`, ta có thể viết biểu thức tính toán đơn giản.

Ví dụ tính chênh lệch giữa giá bán đề xuất và giá mua:

```sql
SELECT 
    productCode,
    productName,
    buyPrice,
    MSRP,
    MSRP - buyPrice AS priceDifference
FROM products;
```

Ví dụ tính giá trị tồn kho theo giá mua:

```sql
SELECT 
    productCode,
    productName,
    quantityInStock,
    buyPrice,
    quantityInStock * buyPrice AS stockValue
FROM products;
```

Các cột tính toán này chỉ xuất hiện trong kết quả truy vấn, không tự động tạo thêm cột mới trong bảng.

### Bài tập thực hành

**Bài 8.1.**

Viết truy vấn lấy `productCode`, `productName`, `buyPrice`, `MSRP` và tạo cột `profitEstimate = MSRP - buyPrice` từ bảng `products`.

**Bài 8.2.**

Viết truy vấn lấy `productCode`, `productName`, `quantityInStock`, `buyPrice` và tạo cột `inventoryValue = quantityInStock * buyPrice`.

**Bài 8.3.**

Viết truy vấn lấy `orderNumber`, `productCode`, `quantityOrdered`, `priceEach` và tạo cột `lineTotal = quantityOrdered * priceEach` từ bảng `orderdetails`.

**Bài 8.4.**

Viết truy vấn lấy `customerNumber`, `checkNumber`, `amount` và tạo cột `amountInThousands = amount / 1000` từ bảng `payments`.

**Bài 8.5.**

Viết truy vấn lấy `productCode`, `productName`, `MSRP` và tạo cột `discountPrice = MSRP * 0.9` từ bảng `products`.

---

## 9. Lọc dữ liệu bằng `WHERE`

Mệnh đề `WHERE` dùng để chỉ lấy các dòng thỏa mãn điều kiện.

Cú pháp:

```sql
SELECT column1, column2
FROM table_name
WHERE condition;
```

Ví dụ lấy các sản phẩm thuộc dòng `Motorcycles`:

```sql
SELECT productCode, productName, productLine
FROM products
WHERE productLine = 'Motorcycles';
```

Ví dụ lấy khách hàng ở `USA`:

```sql
SELECT customerNumber, customerName, city, country
FROM customers
WHERE country = 'USA';
```

Ví dụ lấy các đơn hàng có trạng thái `Cancelled`:

```sql
SELECT orderNumber, orderDate, status, customerNumber
FROM orders
WHERE status = 'Cancelled';
```

### Bài tập thực hành

**Bài 9.1.**

Viết truy vấn lấy các sản phẩm thuộc dòng `Motorcycles`.

**Bài 9.2.**

Viết truy vấn lấy các khách hàng ở `USA`.

**Bài 9.3.**

Viết truy vấn lấy các đơn hàng có trạng thái `Shipped`.

**Bài 9.4.**

Viết truy vấn lấy các nhân viên có `jobTitle` là `Sales Rep`.

**Bài 9.5.**

Viết truy vấn lấy các văn phòng có `country` là `USA`.

---

## 10. Các toán tử so sánh thường dùng

| Toán tử | Ý nghĩa | Ví dụ |
|---|---|---|
| `=` | Bằng | `country = 'USA'` |
| `<>` | Khác | `country <> 'USA'` |
| `>` | Lớn hơn | `MSRP > 100` |
| `<` | Nhỏ hơn | `quantityInStock < 1000` |
| `>=` | Lớn hơn hoặc bằng | `creditLimit >= 100000` |
| `<=` | Nhỏ hơn hoặc bằng | `buyPrice <= 50` |

Ví dụ lấy sản phẩm có giá bán đề xuất lớn hơn 150:

```sql
SELECT productCode, productName, MSRP
FROM products
WHERE MSRP > 150;
```

Ví dụ lấy sản phẩm có số lượng tồn kho dưới 1000:

```sql
SELECT productCode, productName, quantityInStock
FROM products
WHERE quantityInStock < 1000;
```

Ví dụ lấy khách hàng có hạn mức tín dụng từ 100000 trở lên:

```sql
SELECT customerNumber, customerName, creditLimit
FROM customers
WHERE creditLimit >= 100000;
```

### Bài tập thực hành

**Bài 10.1.**

Viết truy vấn lấy các sản phẩm có `MSRP` lớn hơn 150.

**Bài 10.2.**

Viết truy vấn lấy các sản phẩm có `quantityInStock` nhỏ hơn 1000.

**Bài 10.3.**

Viết truy vấn lấy các khách hàng có `creditLimit` lớn hơn hoặc bằng 100000.

**Bài 10.4.**

Viết truy vấn lấy các khoản thanh toán có `amount` lớn hơn 50000.

**Bài 10.5.**

Viết truy vấn lấy các dòng trong `orderdetails` có `quantityOrdered` từ 50 tro len.

---

## 11. Kết hợp điều kiện bằng `AND`

`AND` yêu cầu tất cả điều kiện đều đúng.

Ví dụ lấy sản phẩm thuộc dòng `Classic Cars` và có giá bán đề xuất lớn hơn 100:

```sql
SELECT productCode, productName, productLine, MSRP
FROM products
WHERE productLine = 'Classic Cars'
  AND MSRP > 100;
```

Ví dụ lấy khách hàng ở `USA` và có hạn mức tín dụng lớn hơn 100000:

```sql
SELECT customerNumber, customerName, country, creditLimit
FROM customers
WHERE country = 'USA'
  AND creditLimit > 100000;
```

### Bài tập thực hành

**Bài 11.1.**

Viết truy vấn lấy các khách hàng ở `USA` và có `creditLimit` lớn hơn 100000.

**Bài 11.2.**

Viết truy vấn lấy các sản phẩm thuộc `Classic Cars` va có `MSRP` lớn hơn 100.

**Bài 11.3.**

Viết truy vấn lấy các đơn hàng co `status = 'Shipped'` va `shippedDate` sau ngày `2005-01-01`.

**Bài 11.4.**

Viết truy vấn lấy các sản phẩm co `buyPrice` lớn hơn 50 va `quantityInStock` lớn hơn 5000.

**Bài 11.5.**

Viết truy vấn lấy các khách hàng ở `France` và có `salesRepEmployeeNumber` khác `NULL`.

---

## 12. Kết hợp điều kiện bằng `OR`

`OR` yêu cầu ít nhất một điều kiện đúng.

Ví dụ lấy khách hàng ở `USA` hoặc `France`:

```sql
SELECT customerNumber, customerName, country
FROM customers
WHERE country = 'USA'
   OR country = 'France';
```

Ví dụ lấy sản phẩm thuộc dòng `Motorcycles` hoặc `Planes`:

```sql
SELECT productCode, productName, productLine
FROM products
WHERE productLine = 'Motorcycles'
   OR productLine = 'Planes';
```

### Bài tập thực hành

**Bài 12.1.**

Viết truy vấn lấy các khách hàng ở `France` hoặc `Germany`.

**Bài 12.2.**

Viết truy vấn lấy các sản phẩm thuộc dòng `Planes` hoặc `Ships`.

**Bài 12.3.**

Viết truy vấn lấy các đơn hàng có trạng thái `Cancelled` hoặc `Disputed`.

**Bài 12.4.**

Viết truy vấn lấy các văn phòng ở `USA` hoặc `France`.

**Bài 12.5.**

Viết truy vấn lấy các nhân viên có `jobTitle` là `Sales Rep` hoặc `VP Sales`.

---

## 13. Phủ định điều kiện bằng `NOT`

`NOT` dùng để lấy các dòng không thỏa mãn điều kiện.

Ví dụ lấy khách hàng không ở `USA`:

```sql
SELECT customerNumber, customerName, country
FROM customers
WHERE NOT country = 'USA';
```

Có thể viết tương đương bằng toán tử `<>`:

```sql
SELECT customerNumber, customerName, country
FROM customers
WHERE country <> 'USA';
```

Ví dụ lấy đơn hàng không có trạng thái `Shipped`:

```sql
SELECT orderNumber, orderDate, status
FROM orders
WHERE status <> 'Shipped';
```

### Bài tập thực hành

**Bài 13.1.**

Viết truy vấn lấy các khách hàng không ở `USA`.

**Bài 13.2.**

Viết truy vấn lấy các đơn hàng không có trạng thái `Shipped`.

**Bài 13.3.**

Viết truy vấn lấy các sản phẩm không thuộc dòng `Classic Cars`.

**Bài 13.4.**

Viết truy vấn lấy các văn phòng không thuộc `USA`.

**Bài 13.5.**

Viết truy vấn lấy các khách hàng không có `creditLimit` lớn hơn 100000.

---

## 14. Lọc theo nhiều giá trị bằng `IN`

`IN` giúp viết gọn khi cần so sánh với nhiều giá trị.

Ví dụ:

```sql
SELECT customerNumber, customerName, country
FROM customers
WHERE country IN ('USA', 'France', 'Germany');
```

Truy vấn trên tương đương với:

```sql
SELECT customerNumber, customerName, country
FROM customers
WHERE country = 'USA'
   OR country = 'France'
   OR country = 'Germany';
```

Ví dụ lấy các đơn hàng có trạng thái thuộc một số nhóm cần theo dõi:

```sql
SELECT orderNumber, orderDate, status
FROM orders
WHERE status IN ('Cancelled', 'On Hold', 'Disputed');
```

### Bài tập thực hành

**Bài 14.1.**

Viết truy vấn lấy các đơn hàng có trạng thái la `Cancelled`, `On Hold` hoặc `Disputed`.

**Bài 14.2.**

Viết truy vấn lấy khách hàng ở các quốc gia `USA`, `France`, `Germany`.

**Bài 14.3.**

Viết truy vấn lấy sản phẩm thuộc các dòng `Classic Cars`, `Motorcycles`, `Planes`.

**Bài 14.4.**

Viết truy vấn lấy các văn phòng co `territory` thuộc `NA`, `EMEA`.

**Bài 14.5.**

Viết truy vấn lấy nhan vien co `jobTitle` thuộc `Sales Rep`, `Sales Manager (NA)`, `VP Sales`.

---

## 15. Lọc theo khoảng bằng `BETWEEN`

`BETWEEN` dùng để kiểm tra giá trị nằm trong một khoảng.

Ví dụ lấy sản phẩm có giá bán đề xuất từ 50 đến 100:

```sql
SELECT productCode, productName, MSRP
FROM products
WHERE MSRP BETWEEN 50 AND 100;
```

Ví dụ lấy khách hàng có hạn mức tín dụng từ 50000 đến 100000:

```sql
SELECT customerNumber, customerName, creditLimit
FROM customers
WHERE creditLimit BETWEEN 50000 AND 100000;
```

Ví dụ lấy đơn hàng trong tháng 1 năm 2005:

```sql
SELECT orderNumber, orderDate, status
FROM orders
WHERE orderDate BETWEEN '2005-01-01' AND '2005-01-31';
```

### Bài tập thực hành

**Bài 15.1.**

Viết truy vấn lấy các sản phẩm co `buyPrice` từ 50 đến 80.

**Bài 15.2.**

Viết truy vấn lấy các sản phẩm co `MSRP` từ 100 đến 150.

**Bài 15.3.**

Viết truy vấn lấy các khoản thanh toán co `paymentDate` trong năm 2004.

**Bài 15.4.**

Viết truy vấn lấy các đơn hàng co `orderDate` từ `2005-01-01` den `2005-03-31`.

**Bài 15.5.**

Viết truy vấn lấy các dòng `orderdetails` có `quantityOrdered` từ 20 den 40.

---

## 16. Tìm kiếm chuỗi bằng `LIKE`

`LIKE` dùng để tìm kiếm theo mẫu chuỗi.

Một số ký hiệu thường dùng:

| Ký hiệu | Ý nghĩa |
|---|---|
| `%` | Đại diện cho chuỗi bất kỳ, có thể dài 0 hoặc nhiều ký tự |
| `_` | Đại diện cho đúng một ký tự |

Ví dụ lấy khách hàng có tên bắt đầu bằng `Mini`:

```sql
SELECT customerNumber, customerName
FROM customers
WHERE customerName LIKE 'Mini%';
```

Ví dụ lấy sản phẩm có tên chứa từ `Ford`:

```sql
SELECT productCode, productName
FROM products
WHERE productName LIKE '%Ford%';
```

Ví dụ lấy sản phẩm có tên kết thúc bằng `Coupe`:

```sql
SELECT productCode, productName
FROM products
WHERE productName LIKE '%Coupe';
```

Ví dụ lấy nhân viên có họ bắt đầu bằng chữ `P`:

```sql
SELECT employeeNumber, lastName, firstName
FROM employees
WHERE lastName LIKE 'P%';
```

### Bài tập thực hành

**Bài 16.1.**

Viết truy vấn lấy các sản phẩm có tên chứa từ `Ford`.

**Bài 16.2.**

Viết truy vấn lấy khách hàng co `customerName` bắt đầu bằng `Mini`.

**Bài 16.3.**

Viết truy vấn lấy nhan vien co `email` kết thúc bằng `classicmodelcars.com`.

**Bài 16.4.**

Viết truy vấn lấy sản phẩm co `productName` chứa từ `Harley`.

**Bài 16.5.**

Viết truy vấn lấy van phong co `phone` chứa ký tự `+1`.

---

## 17. Kiểm tra giá trị rỗng bằng `IS NULL`

Trong SQL, giá trị rỗng được biểu diễn bằng `NULL`.

Không nên kiểm tra `NULL` bằng dấu `=`.

Không nên viết:

```sql
SELECT customerNumber, customerName, salesRepEmployeeNumber
FROM customers
WHERE salesRepEmployeeNumber = NULL;
```

Nên viết:

```sql
SELECT customerNumber, customerName, salesRepEmployeeNumber
FROM customers
WHERE salesRepEmployeeNumber IS NULL;
```

Ví dụ lấy khách hàng chưa có nhân viên bán hàng phụ trách:

```sql
SELECT customerNumber, customerName, salesRepEmployeeNumber
FROM customers
WHERE salesRepEmployeeNumber IS NULL;
```

Ví dụ lấy khách hàng đã có nhân viên bán hàng phụ trách:

```sql
SELECT customerNumber, customerName, salesRepEmployeeNumber
FROM customers
WHERE salesRepEmployeeNumber IS NOT NULL;
```

Ví dụ lấy đơn hàng chưa có ngày giao hàng:

```sql
SELECT orderNumber, orderDate, shippedDate, status
FROM orders
WHERE shippedDate IS NULL;
```

### Bài tập thực hành

**Bài 17.1.**

Viết truy vấn lấy các khách hàng chưa có nhân viên bán hàng phụ trách.

**Bài 17.2.**

Viết truy vấn lấy các đơn hàng chưa có `shippedDate`.

**Bài 17.3.**

Viết truy vấn lấy các khách hàng chưa có `state`.

**Bài 17.4.**

Viết truy vấn lấy các văn phòng chưa có `addressLine2`.

**Bài 17.5.**

Viết truy vấn lấy các nhân viên không có quản lý trực tiếp (`reportsTo IS NULL`).

---

## 18. Loại bỏ giá trị trùng lặp bằng `DISTINCT`

`DISTINCT` dùng để chỉ lấy các giá trị khác nhau.

Ví dụ xem các quốc gia có khách hàng:

```sql
SELECT DISTINCT country
FROM customers;
```

Ví dụ xem các trạng thái đơn hàng:

```sql
SELECT DISTINCT status
FROM orders;
```

Ví dụ xem các dòng sản phẩm:

```sql
SELECT DISTINCT productLine
FROM products;
```

Nếu chọn nhiều cột, `DISTINCT` xét sự trùng lặp theo cả tổ hợp các cột.

Ví dụ:

```sql
SELECT DISTINCT city, country
FROM customers;
```

### Bài tập thực hành

**Bài 18.1.**

Viết truy vấn xem các quốc gia khác nhau trong bang `customers`.

**Bài 18.2.**

Viết truy vấn xem các trạng thái đơn hàng khác nhau trong bang `orders`.

**Bài 18.3.**

Viết truy vấn xem các dòng sản phẩm khác nhau trong bang `products`.

**Bài 18.4.**

Viết truy vấn xem các chức danh khác nhau trong bang `employees`.

**Bài 18.5.**

Viết truy vấn xem các `territory` khác nhau trong bang `offices`.

---

## 19. Sắp xếp kết quả bằng `ORDER BY`

`ORDER BY` dùng để sắp xếp kết quả truy vấn.

Cú pháp:

```sql
SELECT column1, column2
FROM table_name
ORDER BY column_name ASC;
```

Trong đó:

| Từ khóa | Ý nghĩa |
|---|---|
| `ASC` | Sắp xếp tăng dần |
| `DESC` | Sắp xếp giảm dần |

Ví dụ sắp xếp sản phẩm theo giá bán đề xuất tăng dần:

```sql
SELECT productCode, productName, MSRP
FROM products
ORDER BY MSRP ASC;
```

Ví dụ sắp xếp sản phẩm theo giá bán đề xuất giảm dần:

```sql
SELECT productCode, productName, MSRP
FROM products
ORDER BY MSRP DESC;
```

Ví dụ sắp xếp khách hàng theo quốc gia, sau đó theo thành phố:

```sql
SELECT customerNumber, customerName, city, country
FROM customers
ORDER BY country ASC, city ASC;
```

### Bài tập thực hành

**Bài 19.1.**

Viết truy vấn lấy `productCode`, `productName`, `MSRP`, sắp xếp theo `MSRP` giảm dần.

**Bài 19.2.**

Viết truy vấn lấy `customerName`, `country`, `city`, sắp xếp theo `country` tăng dần rồi `city` tăng dần.

**Bài 19.3.**

Viết truy vấn lấy `orderNumber`, `orderDate`, `status`, sắp xếp theo `orderDate` mới nhất trước.

**Bài 19.4.**

Viết truy vấn lấy `customerNumber`, `paymentDate`, `amount`, sắp xếp theo `amount` giảm dần.

**Bài 19.5.**

Viết truy vấn lấy `lastName`, `firstName`, `jobTitle`, sắp xếp theo `lastName` rồi `firstName`.

---

## 20. Giới hạn số dòng bằng `LIMIT`

`LIMIT` dùng để giới hạn số dòng kết quả trả về.

Ví dụ lấy 10 sản phẩm đầu tiên:

```sql
SELECT productCode, productName, MSRP
FROM products
LIMIT 10;
```

Ví dụ lấy 5 sản phẩm có giá bán đề xuất cao nhất:

```sql
SELECT productCode, productName, MSRP
FROM products
ORDER BY MSRP DESC
LIMIT 5;
```

Ví dụ lấy 5 khách hàng có hạn mức tín dụng cao nhất:

```sql
SELECT customerNumber, customerName, creditLimit
FROM customers
ORDER BY creditLimit DESC
LIMIT 5;
```

### Bài tập thực hành

**Bài 20.1.**

Viết truy vấn lấy 10 khách hàng co `creditLimit` cao nhất.

**Bài 20.2.**

Viết truy vấn lấy 5 sản phẩm co `MSRP` cao nhất.

**Bài 20.3.**

Viết truy vấn lấy 5 khoan thanh toan co `amount` lớn nhất.

**Bài 20.4.**

Viết truy vấn lấy 3 đơn hàng moi nhat theo `orderDate`.

**Bài 20.5.**

Viết truy vấn lấy 10 sản phẩm co `quantityInStock` thấp nhất.

---

## 21. Bỏ qua một số dòng bằng `LIMIT offset, row_count`

Trong MySQL, có thể dùng dạng:

```sql
LIMIT offset, row_count;
```

Trong đó:

| Thành phần | Ý nghĩa |
|---|---|
| `offset` | Số dòng bỏ qua |
| `row_count` | Số dòng cần lấy |

Ví dụ bỏ qua 10 sản phẩm đầu tiên và lấy 10 sản phẩm tiếp theo:

```sql
SELECT productCode, productName, MSRP
FROM products
ORDER BY productCode ASC
LIMIT 10, 10;
```

Cách viết khác:

```sql
SELECT productCode, productName, MSRP
FROM products
ORDER BY productCode ASC
LIMIT 10 OFFSET 10;
```

### Bài tập thực hành

**Bài 21.1.**

Viết truy vấn lấy 10 sản phẩm đầu tiên sau khi sắp xếp theo `productCode`.

**Bài 21.2.**

Viết truy vấn bỏ qua 10 sản phẩm đầu tiên va lấy 10 sản phẩm tiếp theo theo `productCode`.

**Bài 21.3.**

Viết truy vấn lấy trang thứ 3 cua danh sách khách hàng, mỗi trang 20 khách hàng, sắp xếp theo `customerNumber`.

**Bài 21.4.**

Viết truy vấn bỏ qua 5 khoan thanh toan lớn nhất va lấy 5 khoan tiếp theo theo `amount` giảm dần.

**Bài 21.5.**

Viết truy vấn bỏ qua 20 đơn hàng moi nhat va lấy 10 đơn hàng tiếp theo theo `orderDate` giảm dần.

---

## 22. Thứ tự viết các mệnh đề trong một truy vấn đơn giản

Một truy vấn một bảng thường có thứ tự như sau:

```sql
SELECT column1, column2
FROM table_name
WHERE condition
ORDER BY column_name
LIMIT number;
```

Ví dụ hoàn chỉnh:

```sql
SELECT productCode, productName, productLine, MSRP
FROM products
WHERE productLine = 'Classic Cars'
  AND MSRP > 100
ORDER BY MSRP DESC
LIMIT 10;
```

Ý nghĩa:

1. Lấy dữ liệu từ bảng `products`.
2. Chỉ lấy các sản phẩm thuộc dòng `Classic Cars`.
3. Chỉ lấy sản phẩm có `MSRP > 100`.
4. Sắp xếp theo `MSRP` giảm dần.
5. Chỉ hiển thị 10 dòng đầu tiên.

---

### Bài tập thực hành

**Bài 22.1.**

Viết truy vấn lấy 5 sản phẩm thuộc dòng `Classic Cars`, có `MSRP` lớn hơn 100, sắp xếp theo `MSRP` giảm dần.

**Bài 22.2.**

Viết truy vấn lấy 10 khách hàng ở `USA` co `creditLimit` lớn hơn 50000, sắp xếp theo `creditLimit` giảm dần.

**Bài 22.3.**

Viết truy vấn lấy 5 đơn hàng có trạng thái `Shipped` trong năm 2004, sắp xếp theo `orderDate` giảm dần.

**Bài 22.4.**

Viết truy vấn lấy 10 sản phẩm thuộc `Motorcycles` hoặc `Planes`, co `quantityInStock` lớn hơn 1000, sắp xếp theo `productName` tăng dần.

**Bài 22.5.**

Viết truy vấn lấy 5 khoan thanh toan trong năm 2004 có `amount` lớn hơn 30000, sắp xếp theo `amount` giảm dần.

---

## 23. Một số lỗi thường gặp

### Lỗi 1: Quên dấu nháy đơn cho dữ liệu chuỗi

Sai:

```sql
SELECT customerName, country
FROM customers
WHERE country = USA;
```

Đúng:

```sql
SELECT customerName, country
FROM customers
WHERE country = 'USA';
```

---

### Lỗi 2: Dùng `=` để so sánh với `NULL`

Sai:

```sql
SELECT customerName, salesRepEmployeeNumber
FROM customers
WHERE salesRepEmployeeNumber = NULL;
```

Đúng:

```sql
SELECT customerName, salesRepEmployeeNumber
FROM customers
WHERE salesRepEmployeeNumber IS NULL;
```

---

### Lỗi 3: Đặt `ORDER BY` trước `WHERE`

Sai:

```sql
SELECT productCode, productName, MSRP
FROM products
ORDER BY MSRP DESC
WHERE productLine = 'Classic Cars';
```

Đúng:

```sql
SELECT productCode, productName, MSRP
FROM products
WHERE productLine = 'Classic Cars'
ORDER BY MSRP DESC;
```

---

### Lỗi 4: Quên dấu phẩy giữa các cột

Sai:

```sql
SELECT productCode productName MSRP
FROM products;
```

Đúng:

```sql
SELECT productCode, productName, MSRP
FROM products;
```

### Bài tập thực hành

**Bài 23.1.**

Cho câu sai `SELECT customerName country FROM customers;`. Hay sửa lại để lấy hai cột `customerName` va `country`.

**Bài 23.2.**

Cho câu sai `SELECT * FROM customers WHERE country = USA;`. Hay sửa lỗi chuỗi ký tự.

**Bài 23.3.**

Cho câu sai `SELECT customerName FROM customers WHERE salesRepEmployeeNumber = NULL;`. Hay sửa lỗi kiểm tra `NULL`.

**Bài 23.4.**

Cho câu sai `SELECT productCode, productName FROM products ORDER BY MSRP DESC WHERE productLine = 'Classic Cars';`. Hay sửa thứ tự mệnh đề.

**Bài 23.5.**

Cho câu sai `SELECT productCode productName MSRP FROM products;`. Hay sửa lỗi thiếu dấu phẩy.

---

## 24. Đáp án gợi ý cho một số bài tập

### Bai 5.1
```sql
SELECT *
FROM offices;
```

### Bai 6.1
```sql
SELECT productCode, productName, productLine, MSRP
FROM products;
```

### Bai 7.1
```sql
SELECT 
    customerName AS TenKhachHang,
    city AS ThanhPho,
    country AS QuocGia
FROM customers;
```

### Bai 9.1
```sql
SELECT productCode, productName, productLine
FROM products
WHERE productLine = 'Motorcycles';
```

### Bai 10.1
```sql
SELECT productCode, productName, MSRP
FROM products
WHERE MSRP > 150;
```

### Bai 11.1
```sql
SELECT customerNumber, customerName, country, creditLimit
FROM customers
WHERE country = 'USA'
  AND creditLimit > 100000;
```

### Bai 12.1
```sql
SELECT customerNumber, customerName, country
FROM customers
WHERE country = 'France'
   OR country = 'Germany';
```

### Bai 14.1
```sql
SELECT orderNumber, orderDate, status
FROM orders
WHERE status IN ('Cancelled', 'On Hold', 'Disputed');
```

### Bai 15.1
```sql
SELECT productCode, productName, buyPrice
FROM products
WHERE buyPrice BETWEEN 50 AND 80;
```

### Bai 16.1
```sql
SELECT productCode, productName
FROM products
WHERE productName LIKE '%Ford%';
```

### Bai 17.1
```sql
SELECT customerNumber, customerName, salesRepEmployeeNumber
FROM customers
WHERE salesRepEmployeeNumber IS NULL;
```

### Bai 18.1
```sql
SELECT DISTINCT country
FROM customers;
```

### Bai 19.1
```sql
SELECT productCode, productName, MSRP
FROM products
ORDER BY MSRP DESC;
```

### Bai 20.1
```sql
SELECT customerNumber, customerName, creditLimit
FROM customers
ORDER BY creditLimit DESC
LIMIT 10;
```

### Bai 22.1
```sql
SELECT productCode, productName, productLine, MSRP
FROM products
WHERE productLine = 'Classic Cars'
  AND MSRP > 100
ORDER BY MSRP DESC
LIMIT 5;
```

---

## 25. Tóm tắt

Các kiến thức chính trong tutorial:

- `SELECT` dùng để truy vấn dữ liệu.
- `FROM` chỉ định bảng cần đọc dữ liệu.
- Có thể chọn tất cả cột bằng `*` hoặc chọn từng cột cụ thể.
- `AS` dùng để đặt bí danh cho cột.
- `WHERE` dùng để lọc dòng theo điều kiện.
- `AND`, `OR`, `NOT` dùng để kết hợp hoặc phủ định điều kiện.
- `IN` dùng để lọc theo nhiều giá trị.
- `BETWEEN` dùng để lọc theo khoảng.
- `LIKE` dùng để tìm kiếm theo mẫu chuỗi.
- `IS NULL` và `IS NOT NULL` dùng để kiểm tra giá trị rỗng.
- `DISTINCT` dùng để loại bỏ kết quả trùng lặp.
- `ORDER BY` dùng để sắp xếp kết quả.
- `LIMIT` dùng để giới hạn số dòng kết quả.

---

## 26. Từ khóa chính

- SQL
- SELECT
- FROM
- WHERE
- DISTINCT
- ORDER BY
- LIMIT
- OFFSET
- LIKE
- IN
- BETWEEN
- IS NULL
- IS NOT NULL
- AND
- OR
- NOT
- Alias
- classicmodels