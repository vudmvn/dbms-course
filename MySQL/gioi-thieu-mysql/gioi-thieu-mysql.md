# Bài giảng: Giới thiệu về MySQL

**Cập nhật lần cuối:** 13/03/2026

---

## 1. Mục tiêu bài giảng

Sau khi hoàn thành bài học này, người học có thể:

1. Giải thích được khái niệm **MySQL** và vai trò của MySQL trong hệ sinh thái cơ sở dữ liệu.
2. Trình bày được cách MySQL xử lý yêu cầu từ client và thực thi truy vấn SQL.
3. Mô tả được các đặc điểm nổi bật của MySQL: mã nguồn mở, hiệu năng cao, ACID, storage engine, replication và security.
4. Nhận diện được các nhóm người dùng và lĩnh vực ứng dụng phổ biến của MySQL.
5. Phân tích được vai trò của cloud đối với sự phát triển hiện đại của MySQL.
6. Phân biệt được **MySQL** và **SQL**.
7. Vận dụng kiến thức để lựa chọn MySQL cho các tình huống ứng dụng phù hợp.

---

## 2. MySQL là gì?

**MySQL** là một hệ quản trị cơ sở dữ liệu quan hệ mã nguồn mở, tiếng Anh gọi là **Open-source Relational Database Management System (RDBMS)**.

MySQL sử dụng **SQL** (*Structured Query Language*) để:

- Lưu trữ dữ liệu.
- Quản lý dữ liệu.
- Truy vấn dữ liệu.
- Cập nhật dữ liệu.
- Xóa dữ liệu.
- Tổ chức dữ liệu theo mô hình quan hệ.

MySQL là một trong những hệ quản trị cơ sở dữ liệu được sử dụng rộng rãi trong các ứng dụng web nhờ tốc độ xử lý tốt, độ tin cậy cao, dễ sử dụng và có cộng đồng hỗ trợ lớn.

Một số đặc điểm tổng quan:

- Được phát triển và duy trì bởi **Oracle Corporation**.
- Hỗ trợ nhiều nền tảng như Windows, Linux và macOS.
- Thường được sử dụng với PHP, Java, Python và nhiều ngôn ngữ lập trình khác.
- Phù hợp để xây dựng các ứng dụng động cần lưu trữ và truy xuất dữ liệu thường xuyên.

![MySQL là gì](images/what_is_mysql.png)

---

### Quiz: Khái niệm MySQL

**Câu 1.** MySQL là gì?

A. Một hệ quản trị cơ sở dữ liệu quan hệ mã nguồn mở  
B. Một ngôn ngữ lập trình thay thế Python  
C. Một hệ điều hành máy chủ  
D. Một phần mềm chỉnh sửa ảnh  

**Câu 2.** MySQL sử dụng ngôn ngữ nào để thao tác với dữ liệu?

A. SQL  
B. HTML  
C. CSS  
D. LaTeX  

**Câu 3.** MySQL thường được dùng nhiều trong loại ứng dụng nào?

A. Ứng dụng web và ứng dụng cần dữ liệu có cấu trúc  
B. Chỉ phần mềm chỉnh sửa ảnh  
C. Chỉ trình phát nhạc  
D. Chỉ hệ điều hành  

---

## 3. Vai trò của MySQL trong hệ thống phần mềm

Trong một hệ thống phần mềm, MySQL thường đóng vai trò là **backend database**, tức là nơi lưu trữ và quản lý dữ liệu phía sau ứng dụng.

Ví dụ trong một website thương mại điện tử, MySQL có thể lưu trữ:

- Danh sách sản phẩm.
- Thông tin khách hàng.
- Đơn hàng.
- Lịch sử giao dịch.
- Giỏ hàng.
- Trạng thái thanh toán.
- Thông tin vận chuyển.

Trong một hệ thống quản lý sinh viên, MySQL có thể lưu trữ:

- Hồ sơ sinh viên.
- Danh sách lớp học.
- Môn học.
- Điểm số.
- Học phí.
- Lịch học.

MySQL đặc biệt phù hợp với các hệ thống có dữ liệu **có cấu trúc**, tức là dữ liệu có thể tổ chức thành các bảng, hàng và cột.

---

## 4. Cách MySQL hoạt động

MySQL xử lý các yêu cầu từ client và tương tác với dữ liệu được lưu trữ để thực thi các truy vấn SQL một cách hiệu quả. Quy trình hoạt động có thể mô tả qua các bước sau.

### 4.1. Client Request - Gửi yêu cầu từ client

Người dùng hoặc ứng dụng gửi một câu lệnh SQL đến MySQL Server thông qua ứng dụng, API, công cụ dòng lệnh hoặc công cụ quản trị như MySQL Workbench, phpMyAdmin hoặc DBeaver.

Ví dụ:

```sql
SELECT * FROM students;
```

### 4.2. Connection - Thiết lập kết nối

MySQL Server thiết lập kết nối với client, tạo một **session** để client có thể gửi truy vấn và nhận kết quả. Ở bước này, MySQL kiểm tra tên người dùng, mật khẩu, host và quyền truy cập.

### 4.3. SQL Parsing - Phân tích câu lệnh SQL

MySQL phân tích câu lệnh SQL để kiểm tra cú pháp, tên bảng, tên cột và quyền thực hiện thao tác.

Ví dụ câu lệnh sau sai cú pháp:

```sql
SELEC * FROM students;
```

Do viết sai từ khóa `SELECT`, MySQL sẽ báo lỗi.

### 4.4. Query Optimization - Tối ưu hóa truy vấn

Sau khi câu lệnh hợp lệ, MySQL tìm cách thực thi truy vấn hiệu quả nhất. Trình tối ưu hóa có thể quyết định:

- Có dùng index hay không.
- Nên đọc bảng nào trước.
- Nên dùng phương án join nào.
- Cách giảm số lượng bản ghi cần quét.

Ví dụ nếu bảng `students` có index trên `student_id`, truy vấn sau có thể nhanh hơn:

```sql
SELECT * FROM students
WHERE student_id = 'S001';
```

### 4.5. Execution - Thực thi truy vấn

MySQL thực thi truy vấn để lấy, thêm, cập nhật hoặc xóa dữ liệu.

```sql
INSERT INTO students(student_id, full_name, major)
VALUES ('S001', 'Nguyen Van A', 'Data Science');
```

### 4.6. Storage Engine - Bộ máy lưu trữ

MySQL sử dụng **storage engine** để quản lý cách dữ liệu được lưu trữ, truy xuất và duy trì trên đĩa.

Một số storage engine phổ biến:

- **InnoDB**.
- **MyISAM**.
- **Memory**.
- **CSV**.
- **Archive**.

Trong thực tế, **InnoDB** là storage engine mặc định và phổ biến nhất vì hỗ trợ transaction, ACID, foreign key, row-level locking và crash recovery.

### 4.7. Result Generation và Response

Sau khi thực thi truy vấn, MySQL tạo kết quả trả về. Kết quả có thể là bảng dữ liệu, số dòng bị ảnh hưởng, thông báo thành công hoặc thông báo lỗi. Sau đó, MySQL Server gửi kết quả về ứng dụng client để hiển thị cho người dùng.

### 4.8. Transaction Management

MySQL quản lý transaction để đảm bảo nhiều thao tác được thực hiện đáng tin cậy và nhất quán.

Ví dụ chuyển tiền:

1. Trừ tiền tài khoản A.
2. Cộng tiền vào tài khoản B.

Hai thao tác này phải thành công cùng nhau. Nếu một thao tác thất bại, toàn bộ giao dịch phải được hủy.

### 4.9. Logging, Recovery, Replication và Backup

MySQL ghi log để hỗ trợ phục hồi dữ liệu khi có lỗi. MySQL cũng hỗ trợ replication và backup để:

- Tăng khả năng sẵn sàng.
- Giảm rủi ro mất dữ liệu.
- Phân tán tải đọc.
- Tạo bản sao dữ liệu trên nhiều server.
- Hỗ trợ phục hồi sau sự cố.

---

### Quiz: Cách MySQL hoạt động

**Câu 1.** Bước nào kiểm tra cú pháp của câu lệnh SQL?

A. SQL Parsing  
B. Backup  
C. Replication  
D. Result Display  

**Câu 2.** Storage engine trong MySQL dùng để làm gì?

A. Quản lý cách dữ liệu được lưu trữ và truy xuất  
B. Thiết kế giao diện người dùng  
C. Biên dịch chương trình Java  
D. Tạo slide trình chiếu  

**Câu 3.** Storage engine phổ biến và mặc định hiện nay của MySQL là gì?

A. InnoDB  
B. MyISAM  
C. CSV  
D. Memory  

---

## 5. Các đặc điểm nổi bật của MySQL

MySQL là lựa chọn phổ biến cho nhiều hệ thống cơ sở dữ liệu quan hệ nhờ các đặc điểm sau.

### 5.1. Mã nguồn mở

MySQL là phần mềm mã nguồn mở, cho phép sử dụng, triển khai và phân phối với chi phí thấp. Điều này giúp MySQL phù hợp với sinh viên, người mới học, doanh nghiệp nhỏ và vừa.


