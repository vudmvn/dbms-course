---
layout: page
title: "Physical and Logical Data Independence"
source: "https://www.geeksforgeeks.org/dbms/physical-and-logical-data-independence/"
---


# Bài giảng: Physical and Logical Data Independence trong DBMS

**Cập nhật lần cuối:** 15/07/2025

---

## 1. Mục tiêu bài giảng

Sau khi hoàn thành bài học này, người học có thể:

1. Giải thích được khái niệm **data independence** trong hệ quản trị cơ sở dữ liệu.
2. Trình bày được vai trò của data independence trong kiến trúc DBMS ba mức.
3. Phân biệt được hai loại độc lập dữ liệu:
   - **Physical Data Independence** - độc lập dữ liệu vật lý.
   - **Logical Data Independence** - độc lập dữ liệu logic.
4. Mô tả được mối quan hệ giữa ba mức lược đồ:
   - **Physical Level**.
   - **Logical Level**.
   - **View Level**.
5. Giải thích được cách đạt được độc lập dữ liệu vật lý thông qua ánh xạ **PL-LL mapping**.
6. Giải thích được cách đạt được độc lập dữ liệu logic thông qua ánh xạ **VL-LL mapping**.
7. Phân tích được vì sao data independence giúp cơ sở dữ liệu linh hoạt, dễ bảo trì và bền vững lâu dài.

---

## 2. Data Independence là gì?

**Data Independence** hay **độc lập dữ liệu** là khả năng thay đổi lược đồ (*schema*) ở một mức trong kiến trúc DBMS mà không làm ảnh hưởng đến lược đồ ở các mức khác.

Nói cách khác, khi cách dữ liệu được lưu trữ hoặc tổ chức thay đổi, các chương trình ứng dụng và người dùng vẫn có thể tiếp tục truy cập dữ liệu mà không cần thay đổi lớn.

Ví dụ, nếu quản trị viên cơ sở dữ liệu tạo thêm một chỉ mục để tăng tốc truy vấn, người dùng vẫn có thể sử dụng câu lệnh SQL như cũ. Người dùng không cần biết dữ liệu đã được tổ chức lại bên trong như thế nào.

Data independence giúp:

- Giảm phụ thuộc giữa các tầng trong hệ thống cơ sở dữ liệu.
- Cho phép thay đổi ở tầng thấp mà không phá vỡ tầng cao.
- Giúp cơ sở dữ liệu dễ bảo trì và mở rộng hơn.
- Hạn chế việc phải sửa chương trình ứng dụng khi database thay đổi.
- Hỗ trợ tối ưu hiệu năng mà không làm thay đổi cách người dùng làm việc với dữ liệu.

---

## 3. Kiến trúc ba mức trong DBMS

Để hiểu data independence, cần nắm được kiến trúc ba mức của DBMS.

Ba mức này được sắp xếp từ thấp đến cao như sau:


![Database Management System](images/Databse-Management-System.webp)
*Hình: Minh họa liên quan tới DBMS.*

Trong đó:

1. **Physical Level** là mức thấp nhất.
2. **Logical Level** là mức trung gian.
3. **View Level** là mức cao nhất, gần với người dùng nhất.

---

## 4. Physical Level - Mức vật lý

**Physical Level** là mức mô tả cách dữ liệu thực sự được lưu trữ trong hệ thống.

Mức này quan tâm đến các chi tiết như:

- Dữ liệu được lưu trong file nào.
- Dữ liệu được đặt ở vị trí nào trên thiết bị lưu trữ.
- Cấu trúc chỉ mục được dùng là gì.
- Dữ liệu được nén hay không.
- Dữ liệu được phân mảnh hay sao chép ra sao.
- Không gian đĩa được phân bổ như thế nào.
- Phương pháp truy cập dữ liệu được sử dụng là gì.

Ví dụ, DBMS có thể lưu bảng `Employees` trong nhiều file vật lý khác nhau hoặc tạo index trên cột `employee_id` để tăng tốc tìm kiếm.

Người dùng cuối thường không cần biết các chi tiết này.

---

## 5. Logical Level - Mức logic

**Logical Level** hay **Conceptual Level** mô tả cấu trúc logic của toàn bộ cơ sở dữ liệu.

Mức này trả lời các câu hỏi như:

- Cơ sở dữ liệu có những bảng nào?
- Mỗi bảng có những thuộc tính nào?
- Khóa chính của mỗi bảng là gì?
- Khóa ngoại liên kết các bảng như thế nào?
- Có những ràng buộc dữ liệu nào?
- Các bảng có quan hệ với nhau ra sao?

Ví dụ, trong hệ thống quản lý nhân sự, mức logic có thể gồm các bảng:

- `Employees(employee_id, full_name, department_id, salary)`
- `Departments(department_id, department_name)`
- `Projects(project_id, project_name)`
- `Assignments(employee_id, project_id, role)`

Mức logic không quan tâm dữ liệu được lưu ở file nào hoặc block nào trên ổ đĩa.

---

## 6. View Level - Mức khung nhìn

**View Level** hay **External Level** là mức mô tả cách dữ liệu được hiển thị cho từng người dùng hoặc từng nhóm người dùng.

Một cơ sở dữ liệu có thể có nhiều view khác nhau.

Ví dụ trong hệ thống quản lý nhân sự:

- Nhân viên chỉ xem thông tin cá nhân của mình.
- Trưởng phòng xem danh sách nhân viên thuộc phòng ban.
- Phòng nhân sự xem hồ sơ nhân viên.
- Phòng kế toán xem thông tin lương.
- Quản trị viên có thể xem và quản lý toàn bộ dữ liệu.

Mức view giúp:

- Đơn giản hóa dữ liệu cho người dùng.
- Giới hạn dữ liệu theo vai trò.
- Tăng bảo mật.
- Che giấu cấu trúc logic phức tạp của database.

---

### Quiz: Kiến trúc ba mức DBMS

**Câu 1.** Mức nào là mức thấp nhất trong kiến trúc DBMS?

A. View Level  
B. Logical Level  
C. Physical Level  
D. Application Level  

**Câu 2.** Mức nào mô tả bảng, thuộc tính và quan hệ giữa dữ liệu?

A. Physical Level  
B. Logical Level  
C. View Level  
D. Hardware Level  

**Câu 3.** Mức nào gần người dùng cuối nhất?

A. Physical Level  
B. Logical Level  
C. View Level  
D. Storage Level  

---

## 7. Hai loại Data Independence

Có hai loại data independence chính:

1. **Physical Data Independence**
2. **Logical Data Independence**

Hai loại này tương ứng với hai kiểu thay đổi khác nhau trong kiến trúc DBMS.

| Loại độc lập dữ liệu | Thay đổi ở mức nào? | Không nên ảnh hưởng đến mức nào? |
|---|---|---|
| Physical Data Independence | Physical Level | Logical Level và View Level |
| Logical Data Independence | Logical Level | View Level và application programs |

---

## 8. Physical Data Independence

### 8.1. Khái niệm

**Physical Data Independence** là khả năng thay đổi cấu trúc ở mức vật lý thấp nhất của DBMS mà không làm ảnh hưởng đến các lược đồ ở mức cao hơn.

Nói cách khác, thay đổi ở **Physical Level** không nên làm thay đổi:

- Logical schema.
- View schema.
- Chương trình ứng dụng.
- Cách người dùng truy vấn dữ liệu.

Ví dụ, nếu DBA tạo thêm index hoặc chuyển dữ liệu sang ổ đĩa khác, ứng dụng vẫn có thể hoạt động bình thường.

---

### 8.2. Đặc điểm chính

Physical Data Independence có các đặc điểm sau:

1. **Thay đổi lưu trữ vật lý không ảnh hưởng đến tầng logic**

   Ví dụ, tạo file mới, thêm index hoặc đổi vị trí lưu trữ dữ liệu không làm thay đổi cấu trúc bảng.

2. **Hỗ trợ tối ưu hiệu năng**

   DBA có thể tối ưu cách dữ liệu được lưu trữ hoặc truy xuất mà không ảnh hưởng đến ứng dụng.

3. **Che giấu chi tiết lưu trữ**

   Người dùng và lập trình viên không cần biết dữ liệu được lưu trên ổ đĩa như thế nào.

4. **Dễ đạt được hơn Logical Data Independence**

   Vì thay đổi vật lý thường được DBMS che giấu tốt hơn so với thay đổi logic.

---

### 8.3. Ví dụ Physical Data Independence

Giả sử hệ thống có bảng `Employees`.

Ban đầu, bảng này chưa có index trên cột `employee_id`. Khi dữ liệu tăng lên, truy vấn theo `employee_id` chậm hơn.

DBA tạo index:

```sql
CREATE INDEX idx_employee_id
ON Employees(employee_id);
```

Sau khi tạo index:

- Tốc độ truy vấn có thể tăng.
- Bảng `Employees` vẫn giữ nguyên cấu trúc logic.
- Ứng dụng vẫn dùng câu lệnh SQL như cũ.
- Người dùng không cần biết index đã được tạo.

Đây là ví dụ của **Physical Data Independence**.

---

### 8.4. Physical Data Independence đạt được như thế nào?

Physical Data Independence đạt được bằng cách trừu tượng hóa ánh xạ giữa:

- **Physical Level**
- **Logical Level**

Ánh xạ này gọi là **PL-LL mapping** (*Physical Level - Logical Level mapping*).

Ý tưởng chính:

- Logical schema không phụ thuộc trực tiếp vào cách lưu trữ vật lý.
- DBMS chịu trách nhiệm ánh xạ từ cấu trúc logic sang cấu trúc vật lý.
- Khi physical layer thay đổi, DBMS cập nhật ánh xạ bên trong.
- Người dùng và ứng dụng không cần thay đổi.

Nhờ đó, hệ thống vẫn ổn định ở mức logic ngay cả khi áp dụng các kỹ thuật tối ưu hiệu năng ở mức vật lý.

---

### Quiz: Physical Data Independence

**Câu 1.** Physical Data Independence là gì?

A. Khả năng thay đổi mức vật lý mà không ảnh hưởng đến mức logic hoặc view  
B. Khả năng thay đổi giao diện người dùng  
C. Khả năng xóa toàn bộ dữ liệu  
D. Khả năng thay đổi tên ứng dụng  

**Câu 2.** Ví dụ nào thuộc Physical Data Independence?

A. Tạo index để tăng tốc truy vấn mà ứng dụng không cần sửa  
B. Thêm cột mới vào bảng và sửa giao diện  
C. Đổi màu nền website  
D. Đổi tên người dùng đăng nhập  

**Câu 3.** PL-LL mapping là ánh xạ giữa hai mức nào?

A. Physical Level và Logical Level  
B. View Level và Logical Level  
C. User Interface và CSS  
D. Application và Browser  

---

## 9. Logical Data Independence

### 9.1. Khái niệm

**Logical Data Independence** là khả năng thay đổi lược đồ logic của cơ sở dữ liệu mà không làm ảnh hưởng đến view schema hoặc chương trình ứng dụng.

Nói cách khác, thay đổi ở **Logical Level** không nên buộc người dùng hoặc ứng dụng phải thay đổi cách tương tác với dữ liệu.

Các thay đổi ở mức logic có thể gồm:

- Thêm thuộc tính mới vào bảng.
- Xóa thuộc tính không còn sử dụng.
- Tạo quan hệ mới giữa các bảng.
- Tách một bảng thành nhiều bảng.
- Gộp nhiều bảng.
- Thay đổi ràng buộc dữ liệu.
- Tái cấu trúc mô hình dữ liệu.

---

### 9.2. Đặc điểm chính

Logical Data Independence có các đặc điểm sau:

1. **Thay đổi logical schema không nên ảnh hưởng đến view**

   Ví dụ, thêm cột mới vào bảng không nên làm hỏng các view cũ nếu chúng không dùng cột mới.

2. **Hỗ trợ database phát triển theo thời gian**

   Khi yêu cầu nghiệp vụ thay đổi, schema có thể được mở rộng hoặc tái cấu trúc.

3. **Giảm ảnh hưởng đến application programs**

   Ứng dụng vẫn có thể chạy nếu các view hoặc interface cũ được duy trì.

4. **Khó đạt được hơn Physical Data Independence**

   Vì thay đổi logic thường gần với dữ liệu mà ứng dụng đang sử dụng.

---

### 9.3. Ví dụ Logical Data Independence

Giả sử bảng `Employees` ban đầu gồm:

```text
employee_id, full_name, department
```

Sau đó, doanh nghiệp muốn lưu thêm email của nhân viên.

Ta thêm cột mới:

```sql
ALTER TABLE Employees
ADD email VARCHAR(100);
```

Nếu các chương trình ứng dụng cũ chỉ truy vấn:

```sql
SELECT employee_id, full_name, department
FROM Employees;
```

thì chúng vẫn có thể hoạt động bình thường.

Đây là ví dụ của **Logical Data Independence**.

---

### 9.4. Ảnh hưởng đến chương trình ứng dụng

Lý tưởng nhất, các chương trình ứng dụng hoặc view đang truy cập bảng không cần thay đổi khi schema logic được mở rộng.

Ví dụ, khi thêm cột `email`, các báo cáo cũ không sử dụng cột này vẫn chạy như trước.

Tuy nhiên, nếu thay đổi logic làm đổi tên cột, xóa cột hoặc tách bảng, ứng dụng có thể bị ảnh hưởng nếu không có view hoặc ánh xạ phù hợp.

---

### 9.5. Logical Data Independence đạt được như thế nào?

Logical Data Independence đạt được bằng cách đảm bảo ánh xạ giữa:

- **View Level**
- **Logical Level**

Ánh xạ này gọi là **VL-LL mapping** (*View Level - Logical Level mapping*).

Ý tưởng chính:

- View của người dùng không phụ thuộc trực tiếp vào toàn bộ logical schema.
- DBMS cho phép định nghĩa view để che giấu thay đổi ở mức logic.
- Khi logical schema thay đổi, view có thể được duy trì hoặc điều chỉnh.
- Ứng dụng tiếp tục truy cập view như trước.

Ví dụ:

```sql
CREATE VIEW EmployeeBasicView AS
SELECT employee_id, full_name, department
FROM Employees;
```

Nếu bảng `Employees` có thêm cột `email`, view `EmployeeBasicView` vẫn có thể giữ nguyên cấu trúc cũ.

---

### Quiz: Logical Data Independence

**Câu 1.** Logical Data Independence là gì?

A. Khả năng thay đổi schema logic mà không ảnh hưởng đến view hoặc application programs  
B. Khả năng thay đổi vị trí tệp dữ liệu trên ổ đĩa  
C. Khả năng đổi màu giao diện người dùng  
D. Khả năng xóa toàn bộ bảng  

**Câu 2.** Ví dụ nào thuộc Logical Data Independence?

A. Thêm cột `email` vào bảng `Employees` mà ứng dụng cũ vẫn hoạt động  
B. Thêm index vật lý trên ổ đĩa  
C. Chuyển database sang ổ SSD  
D. Thay đổi cấu hình RAM của server  

**Câu 3.** VL-LL mapping là ánh xạ giữa hai mức nào?

A. View Level và Logical Level  
B. Physical Level và Logical Level  
C. User Interface và Database Disk  
D. Browser và Server  

---

## 10. So sánh Physical và Logical Data Independence

| Tiêu chí | Physical Data Independence | Logical Data Independence |
|---|---|---|
| Mức thay đổi | Physical Level | Logical Level |
| Mức không bị ảnh hưởng | Logical Level và View Level | View Level và application programs |
| Trọng tâm | Cách dữ liệu được lưu trữ vật lý | Cấu trúc và tổ chức dữ liệu |
| Schema liên quan | Internal schema | Conceptual schema |
| Ví dụ thay đổi | Thêm index, đổi vị trí lưu trữ, tạo file mới | Thêm cột, xóa cột, tạo quan hệ mới |
| Mục tiêu | Tối ưu lưu trữ và hiệu năng | Cho phép database design phát triển |
| Mức độ dễ đạt được | Dễ hơn | Khó hơn |
| Ví dụ cụ thể | Di chuyển data files hoặc thêm index | Thêm/xóa cột trong bảng |

---

## 11. Vì sao Data Independence quan trọng?

### 11.1. Linh hoạt trong thiết kế cơ sở dữ liệu

Data independence cho phép thay đổi ở một mức của cơ sở dữ liệu mà không làm ảnh hưởng đến các mức khác.

Nhờ đó, hệ thống trở nên linh hoạt hơn trong:

- Quản lý dữ liệu.
- Thiết kế database.
- Tối ưu lưu trữ.
- Mở rộng schema.
- Điều chỉnh cách dữ liệu được trình bày cho người dùng.

---

### 11.2. Giảm công sức bảo trì

Với data independence, DBA có thể thực hiện các thay đổi cần thiết để:

- Cải thiện hiệu năng.
- Thêm tính năng mới.
- Sửa lỗi lưu trữ.
- Tối ưu hệ thống.
- Mở rộng dữ liệu.

mà không nhất thiết phải cập nhật toàn bộ chương trình ứng dụng hoặc giao diện người dùng.

Điều này giúp giảm chi phí bảo trì khi database phát triển theo thời gian.

---

### 11.3. Tương thích với chương trình ứng dụng

Physical và Logical Data Independence giúp chương trình ứng dụng ít bị ảnh hưởng bởi thay đổi trong cấu trúc database.

Nhờ đó:

- Developer không phải sửa code quá thường xuyên.
- Người dùng không bị gián đoạn.
- Hệ thống vẫn duy trì tính ổn định.
- Các interface cũ có thể tiếp tục hoạt động.

---

### 11.4. Bền vững lâu dài

Khi tổ chức phát triển, nhu cầu dữ liệu cũng thay đổi. Database cần được mở rộng, tối ưu và tái cấu trúc theo thời gian.

Data independence giúp:

- Dễ mở rộng hệ thống.
- Dễ thay đổi schema.
- Dễ tối ưu lưu trữ.
- Giảm rủi ro gián đoạn hoạt động kinh doanh.
- Tăng tuổi thọ của hệ thống dữ liệu.

---

### Quiz: Vai trò của Data Independence

**Câu 1.** Vì sao Data Independence giúp giảm công sức bảo trì?

A. Vì thay đổi ở database không nhất thiết buộc sửa toàn bộ ứng dụng  
B. Vì nó xóa toàn bộ ứng dụng cũ  
C. Vì nó không cho phép thay đổi schema  
D. Vì nó chỉ thay đổi màu giao diện  

**Câu 2.** Data Independence hỗ trợ tính bền vững lâu dài như thế nào?

A. Giúp database dễ mở rộng, tối ưu và tái cấu trúc khi nhu cầu thay đổi  
B. Làm database không thể thay đổi  
C. Làm ứng dụng luôn phải viết lại từ đầu  
D. Làm mất dữ liệu sau mỗi lần cập nhật  

**Câu 3.** Logical Data Independence thường khó hơn Physical Data Independence vì sao?

A. Vì thay đổi logic thường gần với cấu trúc mà application đang sử dụng  
B. Vì thay đổi vật lý luôn ảnh hưởng trực tiếp đến người dùng  
C. Vì logical schema không tồn tại trong DBMS  
D. Vì view không liên quan đến logical schema  

---

## 12. Ví dụ tổng hợp

Xét hệ thống quản lý đơn hàng của một cửa hàng trực tuyến.

### 12.1. Thay đổi vật lý

DBA tạo index trên cột `customer_id` trong bảng `Orders`:

```sql
CREATE INDEX idx_orders_customer
ON Orders(customer_id);
```

Mục tiêu là tăng tốc truy vấn đơn hàng theo khách hàng.

Ứng dụng vẫn có thể dùng truy vấn cũ:

```sql
SELECT *
FROM Orders
WHERE customer_id = 101;
```

Đây là ví dụ của **Physical Data Independence**.

---

### 12.2. Thay đổi logic

Sau đó, hệ thống thêm cột `customer_email` vào bảng `Customers`:

```sql
ALTER TABLE Customers
ADD customer_email VARCHAR(100);
```

Các ứng dụng cũ chỉ dùng `customer_id` và `customer_name` vẫn hoạt động bình thường nếu không phụ thuộc vào cột mới.

Đây là ví dụ của **Logical Data Independence**.

---

### 12.3. Vai trò của view

Nếu ứng dụng cũ cần giao diện dữ liệu ổn định, có thể tạo view:

```sql
CREATE VIEW CustomerBasicView AS
SELECT customer_id, customer_name
FROM Customers;
```

Ứng dụng tiếp tục truy vấn view này mà không cần quan tâm bảng `Customers` đã có thêm cột mới.

---

## 13. Câu hỏi ôn tập

### 13.1. Câu hỏi trắc nghiệm

**Câu 1.** Data independence là gì?

A. Khả năng thay đổi schema ở một mức mà không ảnh hưởng đến các mức khác  
B. Khả năng xóa dữ liệu không cần kiểm soát  
C. Khả năng đổi màu giao diện  
D. Khả năng thay thế database bằng file ảnh  

---

**Câu 2.** Có mấy loại data independence chính?

A. 1  
B. 2  
C. 3  
D. 4  

---

**Câu 3.** Physical Data Independence liên quan chủ yếu đến thay đổi ở mức nào?

A. Physical Level  
B. View Level  
C. User Interface  
D. Application Screen  

---

**Câu 4.** Logical Data Independence liên quan chủ yếu đến thay đổi ở mức nào?

A. Physical Level  
B. Logical Level  
C. Disk Level  
D. Hardware Level  

---

**Câu 5.** Ví dụ nào là Physical Data Independence?

A. Tạo index mới mà không làm thay đổi ứng dụng  
B. Thêm trường email vào form giao diện  
C. Đổi màu trang web  
D. Xóa một nút trên màn hình  

---

**Câu 6.** Ví dụ nào là Logical Data Independence?

A. Thêm cột mới vào bảng mà ứng dụng cũ vẫn hoạt động  
B. Chuyển dữ liệu sang ổ SSD  
C. Tạo file vật lý mới  
D. Thay đổi block size trên đĩa  

---

**Câu 7.** PL-LL mapping là ánh xạ giữa mức nào?

A. Physical Level và Logical Level  
B. View Level và Logical Level  
C. Physical Level và View Level trực tiếp  
D. User Interface và Network Layer  

---

**Câu 8.** VL-LL mapping là ánh xạ giữa mức nào?

A. View Level và Logical Level  
B. Physical Level và Logical Level  
C. Disk Level và File Level  
D. Browser và HTML  

---

**Câu 9.** Data independence giúp ứng dụng như thế nào?

A. Giảm khả năng bị ảnh hưởng bởi thay đổi trong database  
B. Bắt buộc viết lại ứng dụng sau mọi thay đổi  
C. Không cho phép thay đổi schema  
D. Xóa toàn bộ view  

---

**Câu 10.** Loại data independence nào thường khó đạt được hơn?

A. Logical Data Independence  
B. Physical Data Independence  
C. Không loại nào  
D. Cả hai luôn dễ như nhau  

---

### 13.2. Câu hỏi tự luận ngắn

**Câu 1.** Giải thích khái niệm data independence trong DBMS.

---

**Câu 2.** Phân biệt Physical Data Independence và Logical Data Independence.

---

**Câu 3.** Vì sao Physical Data Independence giúp tối ưu hiệu năng mà không ảnh hưởng đến ứng dụng?

---

**Câu 4.** Vì sao Logical Data Independence thường khó đạt được hơn Physical Data Independence?

---

**Câu 5.** Hãy giải thích vai trò của view trong việc hỗ trợ Logical Data Independence.

---

## 14. Bài tập vận dụng

### Bài tập 1

Một DBA chuyển dữ liệu từ ổ HDD sang SSD và tạo thêm index để tăng tốc truy vấn. Ứng dụng không cần thay đổi.

**Yêu cầu:**  
Hãy xác định đây là loại data independence nào và giải thích.

---

### Bài tập 2

Một hệ thống quản lý nhân sự thêm cột `email` vào bảng `Employees`. Các báo cáo cũ vẫn chỉ hiển thị mã nhân viên, tên và phòng ban.

**Yêu cầu:**  
Hãy xác định đây là loại data independence nào và giải thích.

---

### Bài tập 3

Một hệ thống bán hàng muốn thay đổi cấu trúc bảng `Customers`, nhưng vẫn muốn ứng dụng cũ tiếp tục truy vấn `customer_id` và `customer_name`.

**Yêu cầu:**  
Hãy đề xuất cách dùng view để giảm ảnh hưởng đến ứng dụng.

---

### Bài tập 4

Hãy cho một ví dụ trong hệ thống quản lý sinh viên minh họa cả Physical Data Independence và Logical Data Independence.

---

## 15. Tóm tắt bài học

- Data independence là khả năng thay đổi schema ở một mức mà không ảnh hưởng đến các mức khác trong kiến trúc DBMS.
- Có hai loại độc lập dữ liệu chính: Physical Data Independence và Logical Data Independence.
- Physical Data Independence cho phép thay đổi cách lưu trữ vật lý mà không ảnh hưởng đến logical schema hoặc view.
- Logical Data Independence cho phép thay đổi logical schema mà không ảnh hưởng đến view hoặc application programs.
- Physical Data Independence thường dễ đạt được hơn Logical Data Independence.
- PL-LL mapping giúp che giấu thay đổi vật lý khỏi mức logic.
- VL-LL mapping giúp che giấu thay đổi logic khỏi mức view.
- Data independence giúp database linh hoạt hơn, giảm công sức bảo trì, tăng khả năng tương thích với ứng dụng và hỗ trợ phát triển lâu dài.

---

## 16. Từ khóa chính

- Data Independence
- Physical Data Independence
- Logical Data Independence
- Physical Level
- Logical Level
- View Level
- Internal Level
- Conceptual Level
- External Level
- Schema
- PL-LL Mapping
- VL-LL Mapping
- Database Administrator
- Index
- View
- Application Program
- Data Abstraction
- Data Independence

---

## 17. Đáp án và gợi ý trả lời

### Quiz: Kiến trúc ba mức DBMS

- **Câu 1.** C
- **Câu 2.** B
- **Câu 3.** C

### Quiz: Physical Data Independence

- **Câu 1.** A
- **Câu 2.** A
- **Câu 3.** A

### Quiz: Logical Data Independence

- **Câu 1.** A
- **Câu 2.** A
- **Câu 3.** A

### Quiz: Vai trò của Data Independence

- **Câu 1.** A
- **Câu 2.** A
- **Câu 3.** A

### Câu hỏi ôn tập - Trắc nghiệm

- **Câu 1.** A
- **Câu 2.** B
- **Câu 3.** A
- **Câu 4.** B
- **Câu 5.** A
- **Câu 6.** A
- **Câu 7.** A
- **Câu 8.** A
- **Câu 9.** A
- **Câu 10.** A

### Câu hỏi ôn tập - Tự luận ngắn

#### Câu 1

**Gợi ý trả lời:**

Data independence là khả năng thay đổi schema ở một mức trong kiến trúc DBMS mà không làm ảnh hưởng đến schema ở các mức khác hoặc chương trình ứng dụng đang sử dụng cơ sở dữ liệu.

#### Câu 2

**Gợi ý trả lời:**

Physical Data Independence liên quan đến thay đổi ở mức vật lý như thêm index, đổi vị trí lưu trữ, đổi access path mà không ảnh hưởng đến mức logic hoặc view. Logical Data Independence liên quan đến thay đổi ở mức logic như thêm cột, xóa cột, tạo quan hệ mới mà không ảnh hưởng đến view hoặc ứng dụng.

#### Câu 3

**Gợi ý trả lời:**

Physical Data Independence giúp DBA có thể tối ưu hiệu năng bằng cách thay đổi cách lưu trữ, thêm index hoặc điều chỉnh access path. Các thay đổi này được DBMS che giấu thông qua ánh xạ PL-LL nên ứng dụng vẫn dùng schema logic như cũ.

#### Câu 4

**Gợi ý trả lời:**

Logical Data Independence khó hơn vì logical schema thường gần với dữ liệu mà ứng dụng và view đang sử dụng. Khi thay đổi bảng, thuộc tính hoặc quan hệ, ứng dụng có thể bị ảnh hưởng nếu không có view hoặc ánh xạ phù hợp.

#### Câu 5

**Gợi ý trả lời:**

View giúp che giấu thay đổi ở logical schema khỏi người dùng và ứng dụng. Nếu bảng gốc thay đổi, ta có thể duy trì view với cấu trúc cũ để ứng dụng tiếp tục truy vấn mà không cần sửa code.

### Bài tập vận dụng

#### Bài tập 1

**Gợi ý trả lời:**

Đây là Physical Data Independence vì thay đổi xảy ra ở mức vật lý: chuyển dữ liệu sang SSD và tạo thêm index. Ứng dụng không cần thay đổi vì logical schema và view vẫn giữ nguyên.

#### Bài tập 2

**Gợi ý trả lời:**

Đây là Logical Data Independence vì thay đổi xảy ra ở mức logic: thêm cột `email` vào bảng `Employees`. Các báo cáo cũ vẫn hoạt động vì chúng không phụ thuộc vào cột mới.

#### Bài tập 3

**Gợi ý trả lời:**

Có thể tạo view giữ cấu trúc dữ liệu cũ:

```sql
CREATE VIEW CustomerBasicView AS
SELECT customer_id, customer_name
FROM Customers;
```

Ứng dụng cũ truy vấn `CustomerBasicView` thay vì truy vấn trực tiếp bảng `Customers`.

#### Bài tập 4

**Gợi ý trả lời:**

Trong hệ thống quản lý sinh viên, Physical Data Independence có thể là DBA tạo index trên `student_id` để truy vấn nhanh hơn mà ứng dụng không đổi. Logical Data Independence có thể là thêm cột `email` vào bảng `Students` trong khi các màn hình cũ vẫn chỉ hiển thị mã sinh viên, họ tên và lớp.
