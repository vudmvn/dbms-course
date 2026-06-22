---
title: DBMS
---

<style>
  .language-switcher {
    display: flex;
    gap: 0.5rem;
    flex-wrap: wrap;
    margin: 1rem 0 1.5rem;
  }

  .language-switcher button {
    border: 1px solid #d0d7de;
    background: #f6f8fa;
    color: #24292f;
    border-radius: 6px;
    cursor: pointer;
    font: inherit;
    padding: 0.45rem 0.8rem;
  }

  .language-switcher button[aria-pressed="true"] {
    background: #0969da;
    border-color: #0969da;
    color: #ffffff;
  }

  .language-panel[hidden] {
    display: none;
  }

  .book-cover {
    margin: 0.5rem 0;
  }

  .book-cover img {
    max-width: 180px;
    height: auto;
  }

  .language-panel table {
    width: 100%;
    border-collapse: collapse;
    margin: 1rem 0 2rem;
  }

  .language-panel table th,
  .language-panel table td {
    border: 1px solid #d0d7de;
    padding: 0.55rem 0.7rem;
    vertical-align: top;
  }

  .language-panel table th {
    background: #f6f8fa;
    text-align: left;
  }

  .language-panel table td:nth-child(1) {
    width: 34%;
  }

  .language-panel table td:nth-child(2),
  .language-panel table td:nth-child(3) {
    width: 22%;
  }

  .missing {
    color: #6e7781;
  }
</style>

# DBMS

<div class="language-switcher" aria-label="Language switcher">
  <button type="button" id="lang-vi" aria-pressed="true">Tiếng Việt</button>
  <button type="button" id="lang-en" aria-pressed="false">English</button>
</div>

<section id="content-vi" class="language-panel" lang="vi" markdown="1">

Tài liệu học tập về hệ quản trị cơ sở dữ liệu và MySQL.

## Đề cương môn học

- [Syllabus Database VNUIS](assets/files/Syllabus_Database_VNUIS.pdf)

## Sách tham khảo

| Bìa sách | Tài liệu | Tác giả | Nhà xuất bản | ISBN | Liên kết |
|---|---|---|---|---|---|
| <img src="assets/images/database-systems-14th-cover.jpg" alt="Bìa sách Database Systems: Design, Implementation, and Management, 14th Edition" width="90"> | **Database Systems: Design, Implementation, & Management, 14th Edition** | Carlos Coronel, Steven Morris | Cengage, 2023 | 9780357673034 | [Thông tin sách](https://www.cengageasia.com/title/default/detail?isbn=9780357673034) |
| <img src="assets/images/database-systems-pragmatic-approach-3rd-cover.jpg" alt="Bìa sách Database Systems: A Pragmatic Approach, 3rd edition" width="90"> | **Database Systems: A Pragmatic Approach, 3rd edition** | Elvis C. Foster, Shripad Godbole | CRC Press / Taylor & Francis, 2022 | 9781032202020 | [Thông tin sách](https://www.routledge.com/Database-Systems-A-Pragmatic-Approach-3rd-edition/Foster-Godbole/p/book/9781032202020) |

## DBMS Basic

| Bài học | Bài giảng Markdown | Slides PDF | Ghi chú |
|---|---|---|---|
| Giới thiệu cơ sở dữ liệu | [Markdown](DBMS_Basic/gioi_thieu_csdl_vi/gioi_thieu_csdl_vi.md) | [VI](DBMS_Basic/gioi_thieu_csdl_vi/gioi_thieu_csdl_vi_beamer.pdf) / [EN](DBMS_Basic/gioi_thieu_csdl_vi/gioi_thieu_csdl_en_beamer.pdf) | Có slides song ngữ |
| Giới thiệu DBMS | [Markdown](DBMS_Basic/gioi_thieu_dbms_vi/gioi_thieu_dbms_vi.md) | <span class="missing">—</span> | |
| Nhu cầu sử dụng DBMS | [Markdown](DBMS_Basic/nhu_cau_su_dung_dbms_vi/nhu_cau_su_dung_dbms_vi.md) | <span class="missing">—</span> | |
| Kiến trúc DBMS | [Markdown](DBMS_Basic/kien_truc_dbms_vi/kien_truc_dbms_vi.md) | [VI](DBMS_Basic/kien_truc_dbms_vi/kien_truc_dbms_vi.pdf) / [EN](DBMS_Basic/kien_truc_dbms_vi/kien_truc_dbms_en.pdf) | Có slides song ngữ |
| Trừu tượng hóa dữ liệu | [Markdown](DBMS_Basic/data-abstraction/data-abstraction.md) | <span class="missing">—</span> | |
| Độc lập dữ liệu | [Markdown](DBMS_Basic/data-independence/data-independence.md) | [VI](DBMS_Basic/data-independence/data_independence_beamer_pdflatex.pdf) / [EN](DBMS_Basic/data-independence/data_independence_beamer_english_pdflatex.pdf) | Có slides song ngữ |
| Lược đồ cơ sở dữ liệu | [Markdown](DBMS_Basic/database-schema/database-schema.md) | <span class="missing">—</span> | |
| Cách lựa chọn DBMS phù hợp | [Markdown](DBMS_Basic/choose-right-dbms/choose-right-dbms.md) | [VI](DBMS_Basic/choose-right-dbms/choose-right-dbms-vi.pdf) / [EN](DBMS_Basic/choose-right-dbms/choose-right-dbms-en.pdf) | Có slides song ngữ |

<!-- | Độc lập vật lý và độc lập logic | [Markdown](DBMS_Basic/physical-logical-independence/physical-logical-independence.md) | <span class="missing">—</span> | | -->

## Data Modeling

| Bài học | Bài giảng Markdown | Slides PDF | Ghi chú |
|---|---|---|---|
| Mô hình hóa dữ liệu trong DBMS | [Markdown](Data-Modeling/data-modeling/data-modeling.md) | <span class="missing">—</span> | |
| Relational Schema trong DBMS | [Markdown](Data-Modeling/relational-schema/relational-schema.md) | <span class="missing">—</span> | |
| Mapping từ ER Model sang Relational Model | [Markdown](Data-Modeling/er-2-relational/er-2-relational.md) | <span class="missing">—</span> | |
| Giới thiệu ER Model trong DBMS | [Markdown](Data-Modeling/er-model/er-model.md) | [VI](Data-Modeling/er-model/er-model-vi.pdf) / [EN](Data-Modeling/er-model/er-model-en.pdf) | Có slides song ngữ |
| Quan hệ đệ quy trong sơ đồ ER | [Markdown](Data-Modeling/recursive-relationship/recursive-relationship.md) | <span class="missing">—</span> | |
| Mô hình Enhanced ER trong DBMS | [Markdown](Data-Modeling/enhanced-er-model/enhanced-er-model.md) | [VI](Data-Modeling/enhanced-er-model/enhance-er-model-beamer-vi.pdf) | |
| Generalization, Specialization và Aggregation trong ER Model | [Markdown](Data-Modeling/generalization-specialization-aggregation/generalization-specialization-aggregation.md) | <span class="missing">—</span> | Có ví dụ và bài tập vận dụng |
| Các loại khóa trong mô hình quan hệ | [Markdown](Data-Modeling/key/key.md) | <span class="missing">—</span> | Super, candidate, primary, foreign và các loại khóa khác |
| Functional Dependency trong DBMS | [Markdown](Data-Modeling/functional-dependency/functional-dependency.md) | <span class="missing">—</span> | Phụ thuộc hàm, determinant và dependent attribute |
| Các loại Functional Dependency trong DBMS | [Markdown](Data-Modeling/functional-dependency-types/functional-dependency-types.md) | <span class="missing">—</span> | Trivial, non-trivial, multivalued, transitive, fully và partial dependency |
| Attribute Closure trong DBMS | [Markdown](Data-Modeling/attribute-closure/attribute-closure.md) | <span class="missing">—</span> | Đang cập nhật nội dung bài học |
| Normal Forms trong DBMS | [Markdown](Data-Modeling/normal-forms/normal-forms.md) | <span class="missing">—</span> | Tổng quan chuẩn hóa và các dạng chuẩn |
| First Normal Form (1NF) trong DBMS | [Markdown](Data-Modeling/1st-normal-form/1st-normal-form.md) | <span class="missing">—</span> | Loại bỏ nhóm lặp và đảm bảo giá trị nguyên tử |
| Second Normal Form (2NF) trong DBMS | [Markdown](Data-Modeling/2nd-normal-form/2nd-normal-form.md) | <span class="missing">—</span> | Phụ thuộc đầy đủ vào khóa chính |
| Third Normal Form (3NF) trong DBMS | [Markdown](Data-Modeling/3rd-normal-form/3rd-normal-form.md) | <span class="missing">—</span> | Loại bỏ phụ thuộc bắc cầu |
| Fourth Normal Form (4NF) trong DBMS | [Markdown](Data-Modeling/4th-normal-form/4th-normal-form.md) | <span class="missing">—</span> | Xử lý phụ thuộc đa trị |
| Schema Design trong DBMS | [Markdown](Data-Modeling/schema-design/schema-design.md) | <span class="missing">—</span> | Đang cập nhật nội dung bài học |

## Labs

| Lab | Markdown | Thời gian | Độ khó |
|---|---|---:|---|
| Lab 1: ER model basics | [Markdown](Data-Modeling/lab-er-model-1.md) / [Solution](Data-Modeling/lab-er-model-1-hot-water-solution/) | 60m | Beginner |

## MySQL Server

| Bài học | Bài giảng Markdown | Slides PDF | Ghi chú |
|---|---|---|---|
| Giới thiệu MySQL | [Markdown](MySQL/gioi-thieu-mysql/gioi-thieu-mysql.md) | <span class="missing">—</span> | |
| Cài đặt MySQL trên Windows | [Markdown](MySQL/huong_dan_cai_dat_mysql_windows/huong_dan_cai_dat_mysql_windows.md) | <span class="missing">—</span> | |
| Cài đặt MySQL Workbench trên Windows | [Markdown](MySQL/huong_dan_cai_dat_mysql_workbench_windows/huong_dan_cai_dat_mysql_workbench_windows.md) | <span class="missing">—</span> | |
| Kết nối MySQL bằng command options | [Markdown](MySQL/huong_dan_ket_noi_mysql_command_options/huong_dan_ket_noi_mysql_command_options.md) | <span class="missing">—</span> | |
| Kết nối MySQL trong VS Code | [Markdown](MySQL/huong_dan_ket_noi_mysql_vscode/huong_dan_ket_noi_mysql_vscode.md) | <span class="missing">—</span> | |
| Khởi động và dừng MySQL | [Markdown](MySQL/start-stop-MySQL/start-stop-MySQL.md) | <span class="missing">—</span> | |
| MySQL Sample Database: classicmodels | [Markdown](MySQL/mysql-sample-database/mysql-sample-database.md) | <span class="missing">—</span> | |
| Nạp MySQL sample database vào server | [Markdown](MySQL/load-sample-database/load-sample-database.md) | <span class="missing">—</span> | |
| MySQL Storage Engines | [Markdown](MySQL/storage-engine/storage-engine.md) | <span class="missing">—</span> | |
| Chọn cơ sở dữ liệu trong MySQL | [Markdown](MySQL/select-database/select-database.md) | <span class="missing">—</span> | |
| Tạo cơ sở dữ liệu trong MySQL | [Markdown](MySQL/create-database/create-database.md) | <span class="missing">—</span> | |
| Xóa cơ sở dữ liệu trong MySQL | [Markdown](MySQL/drop-database/drop-database.md) | <span class="missing">—</span> | |

## Các câu lệnh SQL trong MySQL

| Bài học | Bài giảng Markdown | Slides PDF | Ghi chú |
|---|---|---|---|
| Kiểu dữ liệu SQL | [Markdown](MySQL/sql-data-types/sql-data-types.md) | <span class="missing">—</span> | |
| Tạo bảng bằng SQL | [Markdown](MySQL/sql-create-table/sql-create-table.md) | <span class="missing">—</span> | |
| Câu lệnh SELECT cơ bản trong MySQL | [Markdown](MySQL/select-statement-1.md) | <span class="missing">—</span> | Sử dụng CSDL mẫu `classicmodels` |
| SELECT và JOIN trong MySQL | [Markdown](MySQL/select-statement-2.md) | <span class="missing">—</span> | Sử dụng CSDL mẫu `classicmodels` |
| Hàm SQL trong MySQL | [Markdown](MySQL/select-statement-3.md) | <span class="missing">—</span> | Sử dụng CSDL mẫu `classicmodels`|
| GROUP BY, HAVING và truy vấn tổng hợp trong MySQL | [Markdown](MySQL/select-statement-4.md) | <span class="missing">—</span> | Sử dụng CSDL mẫu `classicmodels` |
| Subquery trong MySQL | [Markdown](MySQL/select-statement-5.md) | <span class="missing">—</span> | Sử dụng CSDL mẫu `classicmodels` |
| CTE và WITH trong MySQL | [Markdown](MySQL/select-statement-6.md) | <span class="missing">—</span> | Sử dụng CSDL mẫu `classicmodels` |
| Subquery với EXISTS, NOT EXISTS, ALL và ANY trong MySQL | [Markdown](MySQL/select-statement-7.md) | <span class="missing">—</span> | Sử dụng CSDL mẫu `classicmodels` |
| INSERT, UPDATE và DELETE trong MySQL | [Markdown](MySQL/insert-update-delete.md) | <span class="missing">—</span> | Sử dụng CSDL mẫu `classicmodels` |
| Lab: Kết nối nhiều bảng bằng WHERE và JOIN ON trong MySQL | [Markdown](MySQL/select-statement-note-1.md) | <span class="missing">—</span> | Ghi chú bổ sung về implicit join, explicit join và điều kiện `ON` |
| Tutorial: CTE vs Subquery trong MySQL | [Markdown](MySQL/select-statement-note-2.md) | <span class="missing">—</span> | Ghi chú bổ sung so sánh Subquery và CTE |

## Tham khảo

- GeeksforGeeks: [Database Management System Tutorial](https://www.geeksforgeeks.org/dbms/dbms/)
- GeeksforGeeks: [SQL Tutorial](https://www.geeksforgeeks.org/sql/sql-tutorial/)
- MySQL Tutorial: [MySQL Tutorial](https://www.mysqltutorial.org/)
- GeeksforGeeks: [30 Days of SQL - From Basic to Advanced Level](https://www.geeksforgeeks.org/sql/30-days-of-sql-from-basic-to-advanced-level/)
- Roadmap.sh: [SQL Roadmap](https://roadmap.sh/sql?fl=1)

</section>

<section id="content-en" class="language-panel" lang="en" hidden markdown="1">

Learning materials for database management systems and MySQL.

## Course Syllabus

- [Syllabus Database VNUIS](assets/files/Syllabus_Database_VNUIS.pdf)

## Reference Books

| Cover | Reference | Authors | Publisher | ISBN | Link |
|---|---|---|---|---|---|
| <img src="assets/images/database-systems-14th-cover.jpg" alt="Cover of Database Systems: Design, Implementation, and Management, 14th Edition" width="90"> | **Database Systems: Design, Implementation, & Management, 14th Edition** | Carlos Coronel, Steven Morris | Cengage, 2023 | 9780357673034 | [Book information](https://www.cengageasia.com/title/default/detail?isbn=9780357673034) |
| <img src="assets/images/database-systems-pragmatic-approach-3rd-cover.jpg" alt="Cover of Database Systems: A Pragmatic Approach, 3rd edition" width="90"> | **Database Systems: A Pragmatic Approach, 3rd edition** | Elvis C. Foster, Shripad Godbole | CRC Press / Taylor & Francis, 2022 | 9781032202020 | [Book information](https://www.routledge.com/Database-Systems-A-Pragmatic-Approach-3rd-edition/Foster-Godbole/p/book/9781032202020) |

## DBMS Basic

| Lesson | Markdown Lecture | Slides PDF | Notes |
|---|---|---|---|
| Introduction to Databases | [Markdown](DBMS_Basic/gioi_thieu_csdl_vi/gioi_thieu_csdl_vi.md) | [VI](DBMS_Basic/gioi_thieu_csdl_vi/gioi_thieu_csdl_vi_beamer.pdf) / [EN](DBMS_Basic/gioi_thieu_csdl_vi/gioi_thieu_csdl_en_beamer.pdf) | Bilingual slides |
| Introduction to DBMS | [Markdown](DBMS_Basic/gioi_thieu_dbms_vi/gioi_thieu_dbms_vi.md) | <span class="missing">—</span> | |
| Why Use a DBMS? | [Markdown](DBMS_Basic/nhu_cau_su_dung_dbms_vi/nhu_cau_su_dung_dbms_vi.md) | <span class="missing">—</span> | |
| DBMS Architecture | [Markdown](DBMS_Basic/kien_truc_dbms_vi/kien_truc_dbms_vi.md) | [VI](DBMS_Basic/kien_truc_dbms_vi/kien_truc_dbms_vi.pdf) / [EN](DBMS_Basic/kien_truc_dbms_vi/kien_truc_dbms_en.pdf) | Bilingual slides |
| Data Abstraction | [Markdown](DBMS_Basic/data-abstraction/data-abstraction.md) | <span class="missing">—</span> | |
| Data Independence | [Markdown](DBMS_Basic/data-independence/data-independence.md) | [VI](DBMS_Basic/data-independence/data_independence_beamer_pdflatex.pdf) / [EN](DBMS_Basic/data-independence/data_independence_beamer_english_pdflatex.pdf) | Bilingual slides |
| Physical and Logical Independence | [Markdown](DBMS_Basic/physical-logical-independence/physical-logical-independence.md) | <span class="missing">—</span> | |
| Database Schema | [Markdown](DBMS_Basic/database-schema/database-schema.md) | <span class="missing">—</span> | |
| How to Choose the Right DBMS | [Markdown](DBMS_Basic/choose-right-dbms/choose-right-dbms.md) | [VI](DBMS_Basic/choose-right-dbms/choose-right-dbms-vi.pdf) / [EN](DBMS_Basic/choose-right-dbms/choose-right-dbms-en.pdf) | Bilingual slides |

## Data Modeling

| Lesson | Markdown Lecture | Slides PDF | Notes |
|---|---|---|---|
| Data Modeling in DBMS | [Markdown](Data-Modeling/data-modeling/data-modeling.md) | <span class="missing">—</span> | |
| Relational Schema in DBMS | [Markdown](Data-Modeling/relational-schema/relational-schema.md) | <span class="missing">—</span> | |
| Mapping from ER Model to Relational Model | [Markdown](Data-Modeling/er-2-relational/er-2-relational.md) | <span class="missing">—</span> | |
| Introduction to ER Model in DBMS | [Markdown](Data-Modeling/er-model/er-model.md) | [VI](Data-Modeling/er-model/er-model-vi.pdf) / [EN](Data-Modeling/er-model/er-model-en.pdf) | Bilingual slides |
| Recursive Relationships in ER Diagrams | [Markdown](Data-Modeling/recursive-relationship/recursive-relationship.md) | <span class="missing">—</span> | |
| Enhanced ER Model | [Markdown](Data-Modeling/enhanced-er-model/enhanced-er-model.md) | [VI](Data-Modeling/enhanced-er-model/enhance-er-model-beamer-vi.pdf) | |
| Generalization, Specialization, and Aggregation in the ER Model | [Markdown](Data-Modeling/generalization-specialization-aggregation/generalization-specialization-aggregation.md) | <span class="missing">—</span> | Includes examples and exercises |
| Types of Keys in the Relational Model | [Markdown](Data-Modeling/key/key.md) | <span class="missing">—</span> | Super, candidate, primary, foreign, and other key types |
| Functional Dependency in DBMS | [Markdown](Data-Modeling/functional-dependency/functional-dependency.md) | <span class="missing">—</span> | Functional dependency, determinant, and dependent attribute |
| Types of Functional Dependencies in DBMS | [Markdown](Data-Modeling/functional-dependency-types/functional-dependency-types.md) | <span class="missing">—</span> | Trivial, non-trivial, multivalued, transitive, fully, and partial dependency |
| Attribute Closure in DBMS | [Markdown](Data-Modeling/attribute-closure/attribute-closure.md) | <span class="missing">—</span> | Lesson content in progress |
| Normal Forms in DBMS | [Markdown](Data-Modeling/normal-forms/normal-forms.md) | <span class="missing">—</span> | Overview of normalization and normal forms |
| First Normal Form (1NF) in DBMS | [Markdown](Data-Modeling/1st-normal-form/1st-normal-form.md) | <span class="missing">—</span> | Removes repeating groups and keeps values atomic |
| Second Normal Form (2NF) in DBMS | [Markdown](Data-Modeling/2nd-normal-form/2nd-normal-form.md) | <span class="missing">—</span> | Full dependency on the primary key |
| Third Normal Form (3NF) in DBMS | [Markdown](Data-Modeling/3rd-normal-form/3rd-normal-form.md) | <span class="missing">—</span> | Removes transitive dependencies |
| Fourth Normal Form (4NF) in DBMS | [Markdown](Data-Modeling/4th-normal-form/4th-normal-form.md) | <span class="missing">—</span> | Handles multivalued dependencies |
| Schema Design in DBMS | [Markdown](Data-Modeling/schema-design/schema-design.md) | <span class="missing">—</span> | Lesson content in progress |

## Labs

| Lab | Markdown | Duration | Difficulty |
|---|---|---:|---|
| Lab 1: ER model basics | [Markdown](Data-Modeling/lab-er-model-1.md) / [Solution](Data-Modeling/lab-er-model-1-hot-water-solution/) | 60m | Beginner |

**Short description:** Hands-on practice building an ER model: entities, attributes, primary keys, relationships, and mapping to relational schema.

## MySQL Server

| Lesson | Markdown Lecture | Slides PDF | Notes |
|---|---|---|---|
| Introduction to MySQL | [Markdown](MySQL/gioi-thieu-mysql/gioi-thieu-mysql.md) | <span class="missing">—</span> | |
| Install MySQL on Windows | [Markdown](MySQL/huong_dan_cai_dat_mysql_windows/huong_dan_cai_dat_mysql_windows.md) | <span class="missing">—</span> | |
| Install MySQL Workbench on Windows | [Markdown](MySQL/huong_dan_cai_dat_mysql_workbench_windows/huong_dan_cai_dat_mysql_workbench_windows.md) | <span class="missing">—</span> | |
| Connect to MySQL with Command Options | [Markdown](MySQL/huong_dan_ket_noi_mysql_command_options/huong_dan_ket_noi_mysql_command_options.md) | <span class="missing">—</span> | |
| Connect to MySQL in VS Code | [Markdown](MySQL/huong_dan_ket_noi_mysql_vscode/huong_dan_ket_noi_mysql_vscode.md) | <span class="missing">—</span> | |
| MySQL Sample Database: classicmodels | [Markdown](MySQL/mysql-sample-database/mysql-sample-database.md) | <span class="missing">—</span> | |
| Load MySQL Sample Database into Server | [Markdown](MySQL/load-sample-database/load-sample-database.md) | <span class="missing">—</span> | |
| MySQL Storage Engines | [Markdown](MySQL/storage-engine/storage-engine.md) | <span class="missing">—</span> | |
| Start and Stop MySQL | [Markdown](MySQL/start-stop-MySQL/start-stop-MySQL.md) | <span class="missing">—</span> | |
| Selecting a MySQL Database | [Markdown](MySQL/select-database/select-database.md) | <span class="missing">—</span> | |
| Creating a MySQL Database | [Markdown](MySQL/create-database/create-database.md) | <span class="missing">—</span> | |
| Dropping a MySQL Database | [Markdown](MySQL/drop-database/drop-database.md) | <span class="missing">—</span> | |

## SQL Statements in MySQL

| Lesson | Markdown Lecture | Slides PDF | Notes |
|---|---|---|---|
| SQL Data Types | [Markdown](MySQL/sql-data-types/sql-data-types.md) | <span class="missing">—</span> | |
| Creating Tables with SQL | [Markdown](MySQL/sql-create-table/sql-create-table.md) | <span class="missing">—</span> | |
| Basic MySQL SELECT Statement | [Markdown](MySQL/select-statement-1.md) | <span class="missing">—</span> | Uses the `classicmodels` sample database |
| MySQL SELECT and JOIN | [Markdown](MySQL/select-statement-2.md) | <span class="missing">—</span> | Uses the `classicmodels` sample database |
| MySQL SQL Functions | [Markdown](MySQL/select-statement-3.md) | <span class="missing">—</span> | Excludes `GROUP BY`/`HAVING` |
| MySQL GROUP BY, HAVING, and Aggregate Queries | [Markdown](MySQL/select-statement-4.md) | <span class="missing">—</span> | Uses the `classicmodels` sample database |
| MySQL Subqueries | [Markdown](MySQL/select-statement-5.md) | <span class="missing">—</span> | Uses the `classicmodels` sample database |
| MySQL CTE and WITH | [Markdown](MySQL/select-statement-6.md) | <span class="missing">—</span> | Uses the `classicmodels` sample database |
| MySQL Subqueries with EXISTS, NOT EXISTS, ALL, and ANY | [Markdown](MySQL/select-statement-7.md) | <span class="missing">—</span> | Uses the `classicmodels` sample database |
| MySQL INSERT, UPDATE, and DELETE | [Markdown](MySQL/insert-update-delete.md) | <span class="missing">—</span> | Uses the `classicmodels` sample database |
| Lab: Joining Tables with WHERE and JOIN ON in MySQL | [Markdown](MySQL/select-statement-note-1.md) | <span class="missing">—</span> | Supplementary notes on implicit joins, explicit joins, and `ON` conditions |
| Tutorial: CTE vs Subquery in MySQL | [Markdown](MySQL/select-statement-note-2.md) | <span class="missing">—</span> | Supplementary notes comparing Subquery and CTE |

## References

- GeeksforGeeks: [Database Management System Tutorial](https://www.geeksforgeeks.org/dbms/dbms/)
- GeeksforGeeks: [SQL Tutorial](https://www.geeksforgeeks.org/sql/sql-tutorial/)
- MySQL Tutorial: [MySQL Tutorial](https://www.mysqltutorial.org/)
- GeeksforGeeks: [30 Days of SQL - From Basic to Advanced Level](https://www.geeksforgeeks.org/sql/30-days-of-sql-from-basic-to-advanced-level/)
- Roadmap.sh: [SQL Roadmap](https://roadmap.sh/sql?fl=1)

</section>

<script>
  const viButton = document.getElementById("lang-vi");
  const enButton = document.getElementById("lang-en");
  const viContent = document.getElementById("content-vi");
  const enContent = document.getElementById("content-en");

  function setLanguage(language) {
    const isVietnamese = language === "vi";

    viContent.hidden = !isVietnamese;
    enContent.hidden = isVietnamese;
    viButton.setAttribute("aria-pressed", String(isVietnamese));
    enButton.setAttribute("aria-pressed", String(!isVietnamese));
    document.documentElement.lang = language;
    localStorage.setItem("dbms-index-language", language);
  }

  viButton.addEventListener("click", () => setLanguage("vi"));
  enButton.addEventListener("click", () => setLanguage("en"));

  setLanguage(localStorage.getItem("dbms-index-language") || "vi");
</script>
