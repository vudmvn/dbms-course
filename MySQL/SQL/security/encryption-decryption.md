---
title: "Lab nhập môn: Các hàm Hash và Mã hóa trong MySQL"
author: "Tên giảng viên"
duration: "120m"
difficulty: "Beginner–Intermediate"
prerequisites:
  - "Đã biết SELECT, INSERT, UPDATE, CREATE DATABASE và CREATE TABLE"
  - "MySQL 8.0+"
summary: "Giới thiệu các hàm HEX, UNHEX, TO_BASE64, FROM_BASE64, MD5, SHA1, SHA2, AES_ENCRYPT và AES_DECRYPT; mục đích, ví dụ ngắn, bài tập và đáp án gợi ý."
---

# Lab nhập môn: Các hàm Hash và Mã hóa trong MySQL

## Tài liệu tham khảo

- [MySQL Tutorial - Encryption and Compression Functions](https://www.mysqltutorial.org/mysql-administration/mysql-encryption-functions/)
- [MySQL Reference Manual - Encryption and Compression Functions](https://dev.mysql.com/doc/refman/8.4/en/encryption-functions.html)
- [OWASP Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)

> **Lưu ý:** Toàn bộ dữ liệu và key trong lab là giả. Không đưa password thật, dữ liệu cá nhân thật hoặc secret production vào SQL editor, screenshot, file `.sql` hay Git repository.

---

## 1. Mục tiêu

Sau lab này, sinh viên có thể:

1. Phân biệt **encoding**, **hashing** và **encryption**.
2. Biết mục đích của `HEX()`, `UNHEX()`, `TO_BASE64()` và `FROM_BASE64()`.
3. Biết sự khác nhau cơ bản giữa `MD5()`, `SHA1()` và `SHA2()`.
4. Dùng `SHA2(..., 256)` để kiểm tra dữ liệu có bị thay đổi không.
5. Dùng `AES_ENCRYPT()` để mã hóa dữ liệu cần đọc lại.
6. Dùng `AES_DECRYPT()` để giải mã bằng đúng key.
7. Biết ciphertext là binary data và nên xem qua `HEX()`.
8. Không nhầm Base64 với encryption, và không dùng MD5/SHA1/SHA2/AES để lưu password người dùng trong ứng dụng thật.

---

## 2. Ba mục đích khác nhau

Trước khi gọi một hàm, hãy hỏi:

```text
Tôi cần chuyển cách biểu diễn dữ liệu?
Tôi cần kiểm tra dữ liệu có thay đổi không?
Hay tôi cần che giấu dữ liệu và đọc lại khi được phép?
```

| Mục tiêu | Kỹ thuật phù hợp | Hàm MySQL trong lab | Có thể lấy lại dữ liệu gốc? |
|---|---|---|---:|
| Biểu diễn binary thành text | Encoding | `HEX`, `UNHEX`, `TO_BASE64`, `FROM_BASE64` | Có |
| Kiểm tra nội dung thay đổi | Hashing | `MD5`, `SHA1`, `SHA2` | Không |
| Che giấu dữ liệu nhưng cần đọc lại | Encryption | `AES_ENCRYPT`, `AES_DECRYPT` | Có, cần đúng key |

### 2.1. Encoding không phải encryption

Ví dụ:

```sql
SELECT TO_BASE64('hello') AS base64_value;
```

Kết quả là text Base64. Có thể decode ngay:

```sql
SELECT CONVERT(
           FROM_BASE64(
               TO_BASE64('hello')
           )
           USING utf8mb4
       ) AS original_text;
```

Kết quả:

```text
hello
```

**Mục đích của Base64:** đưa binary/text sang dạng text để truyền trong JSON, XML, email hoặc API.

**Không dùng Base64 để bảo vệ bí mật.** Ai có giá trị Base64 đều có thể decode.

### 2.2. Hashing không thể giải mã ngược

Ví dụ:

```sql
SELECT SHA2('hello', 256) AS sha256_value;
```

`SHA2()` tạo một fingerprint có độ dài cố định. Không có hàm:

```sql
SHA2_DECRYPT(...)
```

Hash phù hợp cho:

```text
- Kiểm tra dữ liệu có bị thay đổi.
- So sánh hai input có giống nhau không.
- Lưu checksum/fingerprint.
```

### 2.3. Encryption có thể giải mã

Ví dụ:

```sql
SELECT CONVERT(
           AES_DECRYPT(
               AES_ENCRYPT(
                   'hello',
                   'LAB_KEY_2026'
               ),
               'LAB_KEY_2026'
           )
           USING utf8mb4
       ) AS original_text;
```

Kết quả:

```text
hello
```

**Mục đích của AES:** bảo vệ dữ liệu cần đọc lại, ví dụ ghi chú bí mật giả trong lab hoặc dữ liệu nhạy cảm cần hiển thị cho workflow được cấp quyền.

### 2.4. Không dùng các hàm trong lab để lưu password người dùng

Không dùng:

```text
MD5(password)
SHA1(password)
SHA2(password, 256)
AES_ENCRYPT(password, key)
```

để lưu password của user trong ứng dụng thật.

Password phải dùng password hashing chuyên dụng, thường do application/authentication framework xử lý, như Argon2id, bcrypt, scrypt hoặc PBKDF2. Lab này chỉ giới thiệu các hàm MySQL cơ bản để hiểu mục đích của chúng.

### Bài tập 2

**Bài 2.1.** Base64 là encoding hay encryption?

**Bài 2.2.** Hashing có thể lấy lại original value không?

**Bài 2.3.** Khi nào nên dùng AES encryption?

**Bài 2.4.** Chọn kỹ thuật phù hợp cho: file checksum, dữ liệu cần đưa vào JSON, email cần đọc lại.

**Bài 2.5.** Vì sao không nên lưu password bằng MD5 hoặc SHA2?

---

## 3. Chuẩn bị schema lab

> **Cảnh báo:** Chỉ chạy trên môi trường thực hành. Lệnh dưới đây xóa schema `crypto_intro_lab` nếu schema đã tồn tại.

```sql
DROP DATABASE IF EXISTS crypto_intro_lab;

CREATE DATABASE crypto_intro_lab
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_0900_ai_ci;

USE crypto_intro_lab;
```

Tạo bảng kiểm tra hash:

```sql
CREATE TABLE documents (
    document_id INT UNSIGNED NOT NULL AUTO_INCREMENT,
    document_name VARCHAR(100) NOT NULL,
    document_content TEXT NOT NULL,
    content_sha256 CHAR(64) NOT NULL,
    PRIMARY KEY (document_id),
    CONSTRAINT uq_documents_name UNIQUE (document_name)
) ENGINE = InnoDB;
```

Tạo bảng lưu ciphertext:

```sql
CREATE TABLE secret_notes (
    note_id INT UNSIGNED NOT NULL AUTO_INCREMENT,
    note_title VARCHAR(100) NOT NULL,
    note_ciphertext VARBINARY(500) NOT NULL,
    PRIMARY KEY (note_id)
) ENGINE = InnoDB;
```

### Vì sao hai kiểu dữ liệu khác nhau?

| Column | Kiểu | Lý do |
|---|---|---|
| `document_content` | `TEXT` | Nội dung gốc có thể đọc |
| `content_sha256` | `CHAR(64)` | SHA-256 ở dạng hex gồm 64 ký tự |
| `note_ciphertext` | `VARBINARY(500)` | `AES_ENCRYPT()` trả binary ciphertext |

### Bài tập 3

**Bài 3.1.** Tạo schema `crypto_intro_lab`.

**Bài 3.2.** Tạo table `documents`.

**Bài 3.3.** Tạo table `secret_notes`.

**Bài 3.4.** Vì sao `note_ciphertext` dùng `VARBINARY` thay vì `VARCHAR`?

**Bài 3.5.** Vì sao SHA-256 hex dùng `CHAR(64)` trong lab?

---

# Phần A. Các hàm encoding

## 4. HEX() và UNHEX()

### 4.1. HEX()

`HEX()` chuyển value thành dạng hexadecimal để dễ quan sát.

```sql
SELECT HEX('Hello') AS hello_hex;
```

Kết quả:

```text
48656C6C6F
```

Mỗi cặp hex biểu diễn một byte.

### 4.2. UNHEX()

`UNHEX()` chuyển hex text trở về binary bytes:

```sql
SELECT CONVERT(
           UNHEX('48656C6C6F')
           USING utf8mb4
       ) AS decoded_text;
```

Kết quả:

```text
Hello
```

### 4.3. Mục đích trong lab

`HEX()` rất hữu ích khi xem ciphertext của AES. Ciphertext là binary data, nên không nên hiển thị trực tiếp như text.

Ví dụ:

```sql
SELECT HEX(
           AES_ENCRYPT(
               'secret note',
               'LAB_KEY_2026'
           )
       ) AS ciphertext_hex;
```

Kết quả là một chuỗi hex khác `secret note`.

### Bài tập 4

**Bài 4.1.** Dùng `HEX('Database')`.

**Bài 4.2.** Dùng `UNHEX()` để đổi `48656C6C6F` thành text.

**Bài 4.3.** Dùng `HEX()` hiển thị ciphertext của `'hello'`.

**Bài 4.4.** `HEX()` có làm dữ liệu bí mật hơn không? Giải thích.

**Bài 4.5.** Khi nào nên dùng `HEX()` trong kết quả AES?

---

## 5. TO_BASE64() và FROM_BASE64()

### 5.1. TO_BASE64()

```sql
SELECT TO_BASE64('Database Systems') AS base64_value;
```

Kết quả là text Base64, ví dụ có dạng:

```text
RGF0YWJhc2UgU3lzdGVtcw==
```

### 5.2. FROM_BASE64()

```sql
SELECT CONVERT(
           FROM_BASE64(
               'RGF0YWJhc2UgU3lzdGVtcw=='
           )
           USING utf8mb4
       ) AS decoded_value;
```

Kết quả:

```text
Database Systems
```

### 5.3. So sánh HEX và Base64

| Đặc điểm | HEX | Base64 |
|---|---|---|
| Mục đích | Hiển thị/debug binary | Đưa binary vào text transport |
| Kích thước | Thường dài hơn binary khoảng 2 lần | Thường dài hơn binary khoảng 1/3 |
| Có bảo mật không? | Không | Không |
| Có thể decode? | Có | Có |

### 5.4. Ví dụ Base64 cho ciphertext

```sql
SELECT TO_BASE64(
           AES_ENCRYPT(
               'secret note',
               'LAB_KEY_2026'
           )
       ) AS ciphertext_base64;
```

Dạng này có thể hữu ích khi phải đưa ciphertext vào JSON/text payload. Trong database, lab vẫn lưu ciphertext dạng binary.

### Bài tập 5

**Bài 5.1.** Encode tên của bạn bằng `TO_BASE64()`.

**Bài 5.2.** Decode lại bằng `FROM_BASE64()`.

**Bài 5.3.** So sánh độ dài của HEX và Base64 cho cùng ciphertext.

**Bài 5.4.** Base64 có phải encryption không?

**Bài 5.5.** Nêu một use case phù hợp cho Base64.

---

# Phần B. Các hàm hash

## 6. MD5(), SHA1() và SHA2()

### 6.1. So sánh các hàm

```sql
SELECT MD5('hello') AS md5_hash,
       SHA1('hello') AS sha1_hash,
       SHA2('hello', 256) AS sha256_hash,
       SHA2('hello', 512) AS sha512_hash;
```

| Hàm | Độ dài output hex | Mục đích học trong lab |
|---|---:|---|
| `MD5()` | 32 ký tự | Ví dụ thuật toán hash cũ |
| `SHA1()` | 40 ký tự | Ví dụ thuật toán hash cũ |
| `SHA2(value, 256)` | 64 ký tự | Hash hiện đại hơn để minh họa integrity |
| `SHA2(value, 512)` | 128 ký tự | SHA-512 digest lớn hơn |

### 6.2. Cảnh báo về MD5 và SHA1

`MD5()` và `SHA1()` vẫn có thể xuất hiện trong hệ thống cũ hoặc checksum không nhạy cảm, nhưng không nên dùng cho mục tiêu bảo mật mới như password storage, digital signature mới hoặc integrity chống đối thủ chủ động.

Trong lab, ưu tiên:

```sql
SHA2(value, 256)
```

để minh họa hash.

### 6.3. Cùng input, cùng hash

```sql
SELECT SHA2('Database', 256) AS hash_1,
       SHA2('Database', 256) AS hash_2;
```

Kỳ vọng:

```text
hash_1 = hash_2
```

### 6.4. Thay đổi nhỏ, hash thay đổi mạnh

```sql
SELECT SHA2('Database', 256) AS hash_1,
       SHA2('database', 256) AS hash_2;
```

Chỉ đổi chữ `D` thành `d`, nhưng hai hash khác nhau hoàn toàn.

### 6.5. Hash không phải mã hóa

Sai ý tưởng:

```text
SHA2 -> encrypt
SHA2 -> decrypt
```

Đúng:

```text
SHA2 -> fingerprint
Không có SHA2_DECRYPT
```

### Bài tập 6

**Bài 6.1.** So sánh output của MD5, SHA1, SHA2-256 cho cùng text.

**Bài 6.2.** Tính SHA2-256 cho tên của bạn.

**Bài 6.3.** Đổi một ký tự trong tên và so sánh hash.

**Bài 6.4.** Hàm nào trong lab nên ưu tiên khi minh họa integrity: MD5, SHA1 hay SHA2-256?

**Bài 6.5.** Giải thích vì sao hash không dùng để đọc lại original content.

---

## 7. Dùng SHA2 để kiểm tra dữ liệu bị thay đổi

### 7.1. Insert document và hash

```sql
INSERT INTO documents (
    document_name,
    document_content,
    content_sha256
)
VALUES (
    'course_policy.txt',
    'Students must submit the database assignment before Friday.',
    SHA2(
        'Students must submit the database assignment before Friday.',
        256
    )
);
```

### 7.2. Xem document và hash

```sql
SELECT document_id,
       document_name,
       document_content,
       content_sha256
FROM documents;
```

### 7.3. Kiểm tra integrity

```sql
SELECT document_id,
       document_name,
       SHA2(
           document_content,
           256
       ) = content_sha256 AS integrity_ok
FROM documents;
```

**Giải thích:**

- MySQL hash lại `document_content` hiện tại.
- So sánh hash mới với `content_sha256` đã lưu.
- Nếu giống nhau, `integrity_ok = 1`.
- Nếu khác nhau, `integrity_ok = 0`.

### 7.4. Mô phỏng document bị thay đổi

```sql
UPDATE documents
SET document_content =
    'Students must submit the database assignment before Monday.'
WHERE document_name = 'course_policy.txt';
```

Chạy lại query integrity:

```sql
SELECT document_id,
       document_name,
       SHA2(
           document_content,
           256
       ) = content_sha256 AS integrity_ok
FROM documents;
```

Kỳ vọng:

```text
integrity_ok = 0
```

vì content đã thay đổi nhưng hash cũ chưa cập nhật.

### 7.5. Cập nhật hash khi nội dung thay đổi hợp lệ

```sql
UPDATE documents
SET content_sha256 = SHA2(
        document_content,
        256
    )
WHERE document_name = 'course_policy.txt';
```

Kiểm tra lại:

```sql
SELECT document_id,
       document_name,
       SHA2(
           document_content,
           256
       ) = content_sha256 AS integrity_ok
FROM documents;
```

Kỳ vọng:

```text
integrity_ok = 1
```

### Bài tập 7

**Bài 7.1.** Insert một document khác cùng SHA2-256 hash.

**Bài 7.2.** Viết query kiểm tra integrity cho toàn bộ documents.

**Bài 7.3.** Thay đổi một document mà không update hash.

**Bài 7.4.** Lọc các documents có `integrity_ok = 0`. Gợi ý: lặp lại biểu thức SHA2 trong WHERE.

**Bài 7.5.** Update hash sau khi content thay đổi hợp lệ.

---

# Phần C. AES encryption/decryption

## 8. AES_ENCRYPT() và AES_DECRYPT()

### 8.1. Mục đích

`AES_ENCRYPT()` dùng để tạo ciphertext từ plaintext.

```text
Plaintext + key
-> AES_ENCRYPT()
-> ciphertext
```

`AES_DECRYPT()` dùng để lấy plaintext từ ciphertext khi có đúng key.

```text
Ciphertext + same key
-> AES_DECRYPT()
-> plaintext
```

### 8.2. Encrypt một ghi chú giả

```sql
INSERT INTO secret_notes (
    note_title,
    note_ciphertext
)
VALUES (
    'Demo note',
    AES_ENCRYPT(
        'This is a private note for lab only.',
        'LAB_KEY_2026'
    )
);
```

### 8.3. Xem ciphertext

```sql
SELECT note_id,
       note_title,
       HEX(note_ciphertext) AS ciphertext_hex
FROM secret_notes;
```

`ciphertext_hex` không thể đọc như plaintext note.

### 8.4. Decrypt bằng đúng key

```sql
SELECT note_id,
       note_title,
       CONVERT(
           AES_DECRYPT(
               note_ciphertext,
               'LAB_KEY_2026'
           )
           USING utf8mb4
       ) AS decrypted_note
FROM secret_notes;
```

Kỳ vọng:

```text
This is a private note for lab only.
```

### 8.5. Giải thích `CONVERT(... USING utf8mb4)`

`AES_DECRYPT()` trả binary bytes. Vì note gốc là text UTF-8, dùng:

```sql
CONVERT(... USING utf8mb4)
```

để hiển thị bytes đó như text.

### 8.6. Decrypt với key khác

```sql
SELECT note_id,
       CONVERT(
           AES_DECRYPT(
               note_ciphertext,
               'WRONG_LAB_KEY'
           )
           USING utf8mb4
       ) AS wrong_key_result
FROM secret_notes;
```

Không kỳ vọng đọc được plaintext đúng. Kết quả có thể là `NULL` hoặc output không hợp lệ tùy input/key/environment.

### 8.7. Key trong lab và key trong thực tế

Trong lab:

```sql
'LAB_KEY_2026'
```

được viết thẳng để người học dễ chạy ví dụ.

Trong thực tế, không:

```text
- Hard-code key vào SQL file.
- Lưu key cùng ciphertext trong table.
- Đưa key thật vào Git, screenshot hoặc client history.
- Đưa key qua connection không được bảo vệ.
```

### Bài tập 8

**Bài 8.1.** Insert một note khác bằng AES_ENCRYPT.

**Bài 8.2.** Hiển thị ciphertext bằng HEX.

**Bài 8.3.** Decrypt note bằng đúng key.

**Bài 8.4.** Thử decrypt bằng wrong key.

**Bài 8.5.** Nêu ba nơi không nên lưu production encryption key.

---

## 9. Mục đích của từng hàm trong lab

| Hàm | Ví dụ ngắn | Dùng khi nào? | Không dùng khi nào? |
|---|---|---|---|
| `HEX()` | `HEX(AES_ENCRYPT(...))` | Cần xem binary/ciphertext dạng text | Muốn che giấu dữ liệu |
| `UNHEX()` | `UNHEX('48656C6C6F')` | Cần đổi hex về bytes | Muốn hash hoặc decrypt |
| `TO_BASE64()` | `TO_BASE64(binary_data)` | Cần đưa binary vào JSON/text | Muốn encryption |
| `FROM_BASE64()` | `FROM_BASE64(...)` | Cần decode Base64 | Muốn decrypt AES |
| `MD5()` | `MD5('hello')` | Quan sát hash cũ/legacy demo | Security mới hoặc passwords |
| `SHA1()` | `SHA1('hello')` | Quan sát hash cũ/legacy demo | Security mới hoặc passwords |
| `SHA2(x, 256)` | `SHA2(content, 256)` | Integrity/fingerprint demo | Đọc lại plaintext |
| `AES_ENCRYPT()` | `AES_ENCRYPT(note, key)` | Bảo vệ data cần đọc lại trong lab | Password hashing |
| `AES_DECRYPT()` | `AES_DECRYPT(cipher, key)` | Phục hồi data bằng đúng key | Kiểm tra integrity |

### Bài tập 9

**Bài 9.1.** Chọn hàm phù hợp để hiển thị ciphertext.

**Bài 9.2.** Chọn hàm phù hợp để kiểm tra document thay đổi.

**Bài 9.3.** Chọn hàm phù hợp để đưa binary ciphertext vào JSON.

**Bài 9.4.** Chọn hàm phù hợp để đọc lại note bí mật giả.

**Bài 9.5.** Nêu hàm/nhóm kỹ thuật phù hợp cho password user trong ứng dụng thật.

---

## 10. Bài tập tổng hợp

### Bài 10.1. Encoding practice

1. Chuyển `'SQL Lab'` sang HEX.
2. Chuyển HEX đó trở lại text.
3. Chuyển `'SQL Lab'` sang Base64.
4. Decode Base64.
5. Giải thích vì sao HEX và Base64 không phải encryption.

### Bài 10.2. Hash practice

1. Tính MD5, SHA1, SHA2-256 cho `'MySQL'`.
2. Tính SHA2-256 cho `'mysql'`.
3. So sánh hash của `'MySQL'` và `'mysql'`.
4. Insert một row mới vào `documents`.
5. Viết integrity query.

### Bài 10.3. Encryption practice

1. Insert hai secret notes.
2. Xem ciphertext bằng HEX.
3. Decrypt bằng đúng lab key.
4. Thử decrypt bằng wrong key.
5. Giải thích vì sao table lưu `VARBINARY`.

### Bài 10.4. Chọn kỹ thuật

Điền kỹ thuật phù hợp:

| Dữ liệu / yêu cầu | Kỹ thuật |
|---|---|
| Chuyển image bytes vào JSON | ? |
| Kiểm tra file syllabus có thay đổi | ? |
| Lưu note cần mở lại trong admin workflow | ? |
| Lưu password người dùng | ? |
| Hiển thị ciphertext dạng có thể copy vào report | ? |

### Bài 10.5. Review an toàn

Nêu ít nhất năm điều không nên làm khi thực hành hàm crypto trong MySQL.

---

## 11. Đáp án gợi ý

### Bài 10.1

```sql
SELECT HEX('SQL Lab') AS hex_value;
```

```sql
SELECT CONVERT(
           UNHEX(
               HEX('SQL Lab')
           )
           USING utf8mb4
       ) AS original_text;
```

```sql
SELECT TO_BASE64('SQL Lab') AS base64_value;
```

```sql
SELECT CONVERT(
           FROM_BASE64(
               TO_BASE64('SQL Lab')
           )
           USING utf8mb4
       ) AS original_text;
```

Kết luận:

```text
HEX và Base64 chỉ đổi representation.
Chúng không yêu cầu secret để decode.
```

### Bài 10.2

```sql
SELECT MD5('MySQL') AS md5_hash,
       SHA1('MySQL') AS sha1_hash,
       SHA2('MySQL', 256) AS sha256_hash,
       SHA2('mysql', 256) AS sha256_lowercase;
```

### Bài 10.3

```sql
INSERT INTO secret_notes (
    note_title,
    note_ciphertext
)
VALUES
(
    'Note A',
    AES_ENCRYPT(
        'First secret note',
        'LAB_KEY_2026'
    )
),
(
    'Note B',
    AES_ENCRYPT(
        'Second secret note',
        'LAB_KEY_2026'
    )
);
```

```sql
SELECT note_id,
       note_title,
       HEX(note_ciphertext) AS ciphertext_hex,
       CONVERT(
           AES_DECRYPT(
               note_ciphertext,
               'LAB_KEY_2026'
           )
           USING utf8mb4
       ) AS decrypted_note
FROM secret_notes;
```

### Bài 10.4

| Dữ liệu / yêu cầu | Kỹ thuật |
|---|---|
| Chuyển image bytes vào JSON | Base64 |
| Kiểm tra file syllabus có thay đổi | SHA2-256 |
| Lưu note cần mở lại trong admin workflow | AES encryption |
| Lưu password người dùng | Argon2id/bcrypt/scrypt/PBKDF2 trong application |
| Hiển thị ciphertext để copy vào report | HEX |

### Bài 10.5

Không:

```text
1. Dùng Base64 như encryption.
2. Lưu plaintext password.
3. Dùng MD5/SHA1/SHA2 nhanh để lưu password application.
4. Hard-code production key vào SQL/Git/screenshot.
5. Lưu encryption key cùng ciphertext.
6. Dùng dữ liệu PII thật trong lab.
7. Chạy DROP DATABASE nhầm production schema.
```

---

## 12. Cleanup script

> Chỉ chạy trong lab.

```sql
DROP DATABASE IF EXISTS crypto_intro_lab;
```

---

## 13. Tóm tắt

- `HEX()` và `UNHEX()` dùng để chuyển giữa binary bytes và hexadecimal representation.
- `TO_BASE64()` và `FROM_BASE64()` dùng để chuyển dữ liệu sang/từ Base64; Base64 không bảo mật.
- `MD5()` và `SHA1()` là hash functions cũ; trong lab có thể quan sát output, nhưng không dùng cho security mới.
- `SHA2(value, 256)` là ví dụ phù hợp hơn để tạo fingerprint/checksum kiểm tra integrity.
- Hash không thể decrypt để lấy original value.
- `AES_ENCRYPT()` tạo ciphertext binary; `AES_DECRYPT()` cần đúng key để phục hồi plaintext.
- Ciphertext nên lưu bằng `VARBINARY`/`BLOB`; dùng `HEX()` khi cần hiển thị.
- Password user cần password hashing chuyên dụng ở application layer, không dùng các ví dụ AES/SHA trong lab.
- Key thật không được hard-code, commit, log hoặc lưu cùng ciphertext.

---

## 14. Từ khóa

- Encoding
- Encryption
- Decryption
- Hashing
- Integrity
- Plaintext
- Ciphertext
- HEX
- UNHEX
- Base64
- TO_BASE64
- FROM_BASE64
- MD5
- SHA1
- SHA2
- SHA-256
- AES_ENCRYPT
- AES_DECRYPT
- VARBINARY
- Password Hashing
- Argon2id
- bcrypt
- crypto_intro_lab
