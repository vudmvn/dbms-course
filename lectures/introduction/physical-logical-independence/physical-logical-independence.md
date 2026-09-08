---
title: "Bài giảng: Data Abstraction trong DBMS"
author: "Tên giảng viên"
duration: "120m"
difficulty: "Beginner"
prerequisites:
  - "Đã học khái niệm database và DBMS cơ bản"
summary: "Giải thích data abstraction trong DBMS, ba mức Internal–Conceptual–External, vai trò người dùng ở mỗi mức và mối liên hệ với bảo mật, khả năng sử dụng và bảo trì."
---

# Bài giảng: Data Abstraction trong DBMS

## Tài liệu tham khảo

- GeeksforGeeks – *What is Data Abstraction in DBMS?*
- Các khái niệm kiến trúc ba mức của DBMS (Internal, Conceptual, External)

> Bài này tập trung vào **người dùng hoặc ứng dụng nhìn thấy dữ liệu ở mức nào**. Bài tiếp theo về **Data Independence** tập trung vào việc thay đổi ở một mức có thể ít ảnh hưởng đến mức cao hơn như thế nào.

---

## 1. Mục tiêu học tập

Sau bài học, người học có thể:

1. Giải thích Data Abstraction trong DBMS bằng ví dụ thực tế.
2. Phân biệt ba mức: Internal, Conceptual và External.
3. Nêu được câu hỏi trọng tâm mà mỗi mức trả lời.
4. Xác định nhóm người dùng thường làm việc tại mỗi mức.
5. Giải thích vì sao trừu tượng hóa giúp hệ thống dễ dùng hơn.
6. Phân biệt Data Abstraction với Data Independence.
7. Liên hệ External Level với view, report, form, dashboard và API response.
8. Phân tích một hệ thống đơn giản theo ba mức trừu tượng.

---

## 2. Data Abstraction là gì?

**Data Abstraction** hay **trừu tượng hóa dữ liệu** là cách DBMS che giấu các chi tiết không cần thiết đối với một người dùng hoặc một ứng dụng, đồng thời chỉ cung cấp phần thông tin phù hợp với nhiệm vụ của họ.

Người dùng đặt hàng trực tuyến thường cần biết:

```text
Tên sản phẩm
Giá bán
Màu sắc
Kích thước
Tình trạng còn hàng
```

Người dùng thường không cần biết:

```text
Dữ liệu được lưu ở file vật lý nào
DBMS dùng index nào để tìm sản phẩm
Dữ liệu nằm ở page/block nào
Có bao nhiêu bản sao dữ liệu
Chiến lược tối ưu truy vấn đang được dùng
```

![Data Abstraction che giấu chi tiết lưu trữ](images/data-abstraction-overview.png)

### 2.1. Mục đích

Data Abstraction giúp:

- Đơn giản hóa việc sử dụng dữ liệu.
- Che giấu phần kỹ thuật phức tạp không cần thiết.
- Hỗ trợ thiết kế giao diện dữ liệu khác nhau cho các nhóm vai trò.
- Hạn chế việc hiển thị dữ liệu không liên quan.
- Cho phép DBA và kỹ sư tối ưu hệ thống phía sau mà người dùng không phải biết chi tiết.

> Data Abstraction **hỗ trợ** bảo mật vì có thể giới hạn thông tin hiển thị. Tuy nhiên, bảo mật thực tế vẫn cần xác thực, phân quyền, policy, audit và các cơ chế kiểm soát khác.

---

## 3. Ba mức trừu tượng dữ liệu

DBMS thường mô tả dữ liệu theo ba mức:

```text
View / External Level
        ↑
Logical / Conceptual Level
        ↑
Physical / Internal Level
```

| Mức | Câu hỏi trọng tâm | Ai thường làm việc? |
|---|---|---|
| View / External | Mỗi nhóm người dùng cần nhìn thấy dữ liệu nào? | End users, application developers, analysts |
| Logical / Conceptual | Toàn bộ dữ liệu được tổ chức logic như thế nào? | Database designers, developers, analysts, DBA |
| Physical / Internal | Dữ liệu thực sự được lưu và truy cập như thế nào? | DBA, system engineers, DBMS engineers |

![Ba mức trừu tượng dữ liệu trong DBMS](images/dbms-three-level-abstraction.png)

---

## 4. Physical / Internal Level

### 4.1. Khái niệm

Physical Level hoặc Internal Level là mức thấp nhất. Nó mô tả cách dữ liệu được lưu trữ và truy cập trong hệ thống.

Ví dụ các chi tiết ở mức này:

```text
Data files
Pages hoặc blocks
Indexes
Access paths
B+ tree hoặc hash structures
Nén dữ liệu
Phân vùng dữ liệu
Sao chép dữ liệu
Vị trí lưu trữ
```

### 4.2. Ví dụ

Khi một người tìm sinh viên theo mã số, họ chỉ yêu cầu hệ thống trả về thông tin sinh viên. DBMS có thể sử dụng index hoặc đọc một số page cụ thể để trả kết quả.

Người dùng không cần biết:

```text
Index có tồn tại hay không
Index dùng cấu trúc nào
Dữ liệu nằm trong file nào
Cần đọc bao nhiêu page
```

### 4.3. Ai quan tâm đến mức này?

- DBA tối ưu hiệu năng và quản lý lưu trữ.
- Kỹ sư hệ thống giám sát tài nguyên.
- DBMS engineers phát triển cơ chế lưu trữ.
- Người dùng cuối thường không làm việc trực tiếp ở mức này.

---

## 5. Logical / Conceptual Level

### 5.1. Khái niệm

Logical Level hoặc Conceptual Level mô tả cấu trúc logic của toàn bộ database.

Nó trả lời các câu hỏi:

```text
Có những thực thể hoặc bảng nào?
Mỗi thực thể có dữ liệu gì?
Các thực thể có quan hệ gì?
Có quy tắc dữ liệu nào?
Thông tin nào cần được lưu?
```

Ví dụ trong hệ thống quản lý sinh viên:

```text
Sinh viên
Môn học
Lớp học phần
Đăng ký học
Điểm
Khoa
Giảng viên
```

Ở mức khái niệm, nhà thiết kế quan tâm đến ý nghĩa và cấu trúc dữ liệu; họ không cần biết bảng đang được lưu trong file nào.

### 5.2. Vai trò

- Là nền tảng để thiết kế database.
- Giúp thống nhất cách hiểu dữ liệu giữa nghiệp vụ và kỹ thuật.
- Cung cấp cấu trúc chung cho nhiều external views.
- Là mức gần nhất với các mô hình ERD và relational schema trong các bài học tiếp theo.

---

## 6. View / External Level

### 6.1. Khái niệm

View Level hoặc External Level là mức gần người dùng nhất. Nó mô tả dữ liệu nào được hiển thị cho một người dùng, một nhóm người dùng hoặc một ứng dụng cụ thể.

Trong một trường đại học:

| Nhóm | Thông tin thường cần |
|---|---|
| Sinh viên | Hồ sơ cá nhân, lịch học, điểm của chính mình |
| Giảng viên | Danh sách lớp, lịch giảng, điểm của lớp phụ trách |
| Phòng đào tạo | Dữ liệu học vụ rộng hơn |
| Phòng tài chính | Học phí, thanh toán, hỗ trợ tài chính |
| DBA | Metadata, quyền, cấu hình và giám sát hệ thống |

### 6.2. External Level không chỉ là SQL VIEW

External Level có thể được hiện thực bằng:

```text
SQL view
Form
Report
Dashboard
API response
Trang web hoặc mobile screen
Export file
Role-based interface
```

SQL `VIEW` là một cơ chế phổ biến, nhưng không phải là toàn bộ ý nghĩa của External Level.

![Các external views cho nhiều vai trò](images/external-views-by-role.png)

---

## 7. Ví dụ xuyên suốt: hệ thống quản lý sinh viên

### 7.1. Physical / Internal

```text
Dữ liệu được lưu trong data files.
DBMS có thể dùng index để tìm nhanh thông tin.
Có thể có backup, replication hoặc partition.
```

### 7.2. Logical / Conceptual

```text
Có các nhóm dữ liệu: sinh viên, môn học, lớp học phần, đăng ký, điểm.
Có các liên hệ và quy tắc nghiệp vụ giữa các nhóm dữ liệu.
```

### 7.3. View / External

```text
Sinh viên xem dữ liệu cá nhân.
Giảng viên xem lớp mình phụ trách.
Phòng đào tạo xem thông tin học vụ.
Phòng tài chính xem dữ liệu học phí.
```

![Ví dụ ba mức trong hệ thống quản lý sinh viên](images/student-system-three-abstraction-levels.png)

---

## 8. Data Abstraction và Data Independence

Hai khái niệm liên quan nhưng không giống nhau.

| Khái niệm | Câu hỏi trung tâm |
|---|---|
| Data Abstraction | Người dùng hoặc ứng dụng cần nhìn dữ liệu ở mức nào? |
| Data Independence | Khi thay đổi một mức dữ liệu, mức cao hơn hoặc ứng dụng có cần thay đổi đáng kể không? |

Ví dụ:

```text
Sinh viên chỉ xem điểm của mình
→ Data Abstraction / External Level.

DBA thêm index để truy vấn nhanh hơn mà portal sinh viên không cần sửa
→ Physical Data Independence.
```

---

## 9. Quiz

**Câu 9.1.** Một hệ thống hiển thị cho sinh viên lịch học và điểm của chính họ, nhưng không hiển thị học phí của sinh viên khác. Điều này phản ánh trực tiếp nhất điều gì?

A. Physical Level.  
B. External Level được thiết kế theo vai trò.  
C. Cách DBMS phân bổ page trên đĩa.  
D. Một thay đổi ở internal schema.

**Câu 9.2.** DBA thay đổi cấu trúc index để cải thiện tốc độ truy vấn. Chi tiết này thuộc mức nào?

A. View / External Level.  
B. Logical / Conceptual Level.  
C. Physical / Internal Level.  
D. Application presentation layer.

**Câu 9.3.** Phát biểu nào phân biệt đúng Data Abstraction và Data Independence?

A. Data Abstraction tập trung vào mức hiển thị thông tin; Data Independence tập trung vào mức ảnh hưởng của thay đổi schema.  
B. Cả hai chỉ nói về sao lưu dữ liệu.  
C. Data Independence chỉ áp dụng cho dashboard.  
D. Data Abstraction chỉ áp dụng cho index.

### Đáp án quiz

| Câu | Đáp án | Giải thích |
|---:|:---:|---|
| 9.1 | B | External Level xác định phần dữ liệu phù hợp với từng vai trò. |
| 9.2 | C | Index và access path thuộc chi tiết lưu trữ/truy cập vật lý. |
| 9.3 | A | Hai khái niệm liên quan nhưng trả lời hai câu hỏi khác nhau. |

---

## 10. Bài tập vận dụng

### Bài 10.1. Hệ thống bệnh viện

Một hệ thống bệnh viện có bác sĩ, bệnh nhân, kế toán và quản trị viên.

Hãy nêu:

1. Một external view cho bác sĩ.
2. Một external view cho bệnh nhân.
3. Một loại thông tin ở conceptual level.
4. Hai chi tiết thuộc physical level.

### Bài 10.2. Cổng thương mại điện tử

Đối với một website bán hàng, hãy phân loại các nội dung sau vào Physical, Conceptual hoặc External Level:

```text
Danh mục sản phẩm hiển thị cho khách hàng
Chiến lược index tìm kiếm sản phẩm
Thông tin sản phẩm và đơn hàng cần được quản lý
Dashboard doanh thu cho quản lý
Bản sao dữ liệu dự phòng
```

### Bài 10.3. Nhận xét thiết kế

Một nhóm nói: “Chúng tôi dùng dashboard nên đã có Data Independence.”

Nhận xét phát biểu này bằng 3–5 câu, đồng thời chỉ ra phần nào liên quan đến abstraction và phần nào liên quan đến independence.

---

## 11. Tóm tắt

- Data Abstraction che giấu chi tiết không cần thiết và chỉ hiển thị thông tin phù hợp với từng mục đích.
- DBMS thường có ba mức: Physical/Internal, Logical/Conceptual và View/External.
- Physical Level tập trung vào cách lưu trữ và truy cập dữ liệu.
- Logical Level tập trung vào cấu trúc logic tổng thể của dữ liệu.
- External Level tập trung vào phần dữ liệu từng nhóm người dùng hoặc ứng dụng cần thấy.
- External Level có thể xuất hiện dưới dạng views, forms, reports, dashboards hoặc API responses.
- Data Abstraction không đồng nghĩa hoàn toàn với Data Independence.

## 12. Từ khóa chính

- Data Abstraction
- Internal Level
- Physical Level
- Conceptual Level
- Logical Level
- External Level
- View Level
- External Schema
- Conceptual Schema
- Internal Schema
- Metadata
- Database View
- Role-based Access
- Data Independence
