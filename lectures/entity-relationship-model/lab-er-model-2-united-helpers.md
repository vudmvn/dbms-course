---
title: "Lab: Xây dựng ERD cho United Helpers"
author: "Tên giảng viên"
duration: "60-90m"
difficulty: "Beginner"
prerequisites:
    - "Thực thể và thuộc tính"
    - "Khóa chính và khóa ngoại"
    - "Quan hệ một-nhiều và nhiều-nhiều"
    - "Thực thể liên kết"
    - "Ký hiệu Crow's Foot"
summary: "Đề bài và các bước chính để xây dựng ERD Crow's Foot và lược đồ quan hệ cho bài United Helpers."
---

# Lab: Xây dựng ERD cho United Helpers

## 1. Mục tiêu học tập

Sau khi hoàn thành lab này, người học có thể:

1. Đọc mô tả nghiệp vụ và xác định các thực thể chính.
2. Xác định thuộc tính và khóa chính cho từng thực thể.
3. Phân tích quan hệ một-nhiều và nhiều-nhiều.
4. Tạo thực thể liên kết cho quan hệ nhiều-nhiều có thuộc tính riêng.
5. Xác định bội số và sự tham gia bắt buộc/tùy chọn theo Crow's Foot.
6. Chuyển ERD sang mô hình quan hệ.

---

## 2. Đề bài

United Helpers là một tổ chức phi lợi nhuận hỗ trợ người dân sau thảm họa thiên nhiên. Hệ thống cần quản lý:

- Tình nguyện viên.
- Nhiệm vụ của tổ chức.
- Danh sách đóng gói dùng cho các nhiệm vụ đóng gói.
- Các gói hàng được tạo ra.
- Các mặt hàng thực tế được đưa vào từng gói hàng.

Một số yêu cầu quan trọng:

- Tình nguyện viên có thể được phân công cho nhiều nhiệm vụ.
- Nhiệm vụ có thể cần nhiều tình nguyện viên.
- Khi phân công, cần lưu thời gian bắt đầu và thời gian kết thúc.
- Mỗi nhiệm vụ có mã nhiệm vụ, mô tả, loại và trạng thái.
- Nhiệm vụ loại "đóng gói" được gắn với đúng một danh sách đóng gói.
- Một danh sách đóng gói có thể chưa được dùng hoặc được dùng bởi nhiều nhiệm vụ.
- Một nhiệm vụ có thể tạo ra nhiều gói hàng hoặc không tạo ra gói hàng nào.
- Mỗi gói hàng thuộc đúng một nhiệm vụ.
- Một gói hàng chứa ít nhất một mặt hàng.
- Một mặt hàng có thể xuất hiện trong nhiều gói hàng hoặc chưa từng xuất hiện trong gói nào.
- Cần lưu số lượng thực tế của từng mặt hàng trong từng gói hàng.

---

## 3. Sản phẩm cần nộp

Người học cần hoàn thành các sản phẩm sau:

1. Danh sách thực thể chính.
2. Danh sách thuộc tính và khóa chính của từng thực thể.
3. Danh sách các quan hệ, bội số và mức tham gia.
4. Các thực thể liên kết cần thiết cho quan hệ nhiều-nhiều.
5. ERD dùng ký hiệu Crow's Foot.
6. Lược đồ quan hệ sau khi chuyển từ ERD.
7. Bảng kiểm tra đối chiếu giữa yêu cầu đề bài và mô hình đã thiết kế.

---

## 4. Các bước chính để triển khai

### Bước 1: Xác định thực thể chính

Đọc đề bài và gạch chân các danh từ đại diện cho đối tượng cần lưu trữ lâu dài.

Cần trả lời:

- Hệ thống cần quản lý những loại đối tượng nào?
- Đối tượng nào có dữ liệu riêng cần lưu?
- Đối tượng nào chỉ là thuộc tính hoặc trạng thái của đối tượng khác?

### Bước 2: Xác định thuộc tính và khóa chính

Với mỗi thực thể, xác định:

- Thuộc tính mô tả.
- Thuộc tính định danh.
- Khóa chính.
- Thuộc tính nào có thể rỗng.
- Thuộc tính nào cần ràng buộc nghiệp vụ.

### Bước 3: Xác định quan hệ nhiều-nhiều

Tìm các cặp thực thể mà mỗi bên có thể liên quan đến nhiều bản ghi của bên còn lại.

Với mỗi quan hệ nhiều-nhiều, xác định:

- Có cần tạo thực thể liên kết không?
- Quan hệ có thuộc tính riêng không?
- Khóa chính của thực thể liên kết gồm những thuộc tính nào?

### Bước 4: Xác định quan hệ và bội số

Với từng quan hệ, xác định:

- Quan hệ là `1:1`, `1:N` hay `M:N`.
- Phía nào bắt buộc tham gia.
- Phía nào tùy chọn tham gia.
- Cách biểu diễn bằng Crow's Foot.

### Bước 5: Vẽ mô hình ERD

Vẽ ERD bằng ký hiệu Crow's Foot.

ERD cần thể hiện:

- Tên thực thể.
- Thuộc tính chính.
- Khóa chính.
- Khóa ngoại nếu đã xác định.
- Các quan hệ giữa thực thể.
- Bội số ở hai đầu quan hệ.

### Bước 6: Chuyển sang mô hình quan hệ

Từ ERD, chuyển sang lược đồ quan hệ.

Với mỗi bảng, cần ghi:

- Tên bảng.
- Danh sách cột.
- Khóa chính.
- Khóa ngoại.
- Ràng buộc quan trọng.

### Bước 7: Kiểm tra lại yêu cầu đề bài

Đối chiếu mô hình với từng yêu cầu trong đề bài.

Cần kiểm tra:

- Có lưu được đầy đủ thông tin tình nguyện viên không?
- Có lưu được nhiệm vụ, loại nhiệm vụ và trạng thái không?
- Có biểu diễn đúng việc phân công tình nguyện viên vào nhiệm vụ không?
- Có lưu được thời gian bắt đầu và kết thúc phân công không?
- Có xử lý đúng nhiệm vụ đóng gói và danh sách đóng gói không?
- Có biểu diễn đúng gói hàng và mặt hàng trong gói không?
- Có lưu được số lượng thực tế của từng mặt hàng trong từng gói không?

---

## 5. Câu hỏi tự kiểm tra

1. Vì sao quan hệ giữa tình nguyện viên và nhiệm vụ không nên biểu diễn bằng một khóa ngoại đơn giản?
2. Vì sao số lượng thực tế của mặt hàng trong gói không nên đặt trong bảng mặt hàng?
3. Nhiệm vụ không phải loại đóng gói thì thuộc tính liên quan đến danh sách đóng gói nên xử lý thế nào?
4. Một mặt hàng chưa từng xuất hiện trong gói nào có được lưu trong hệ thống không?
5. Làm thế nào để thể hiện yêu cầu mỗi gói hàng phải có ít nhất một mặt hàng?
