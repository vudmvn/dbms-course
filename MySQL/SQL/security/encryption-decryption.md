---
title: "Lab MySQL: Mã hóa dữ liệu, Hashing và Bảo vệ thông tin nhạy cảm"
author: "Tên giảng viên"
duration: "210m"
difficulty: "Intermediate"
prerequisites:
  - "Đã biết CREATE TABLE, INSERT, SELECT, UPDATE và transaction cơ bản"
  - "Đã hiểu VARCHAR, VARBINARY/BLOB và constraints ở mức cơ bản"
  - "MySQL 8.0+; phần KDF cần MySQL 8.0.30+"
summary: "Thực hành phân biệt encoding, hashing và encryption; dùng HEX/Base64, SHA2, AES_ENCRYPT/AES_DECRYPT, KDF + salt, thiết kế bảng lưu ciphertext và nhận biết giới hạn bảo mật của SQL-level encryption."
---

# Lab MySQL: Mã hóa dữ liệu, Hashing và Bảo vệ thông tin nhạy cảm

## Tài liệu tham khảo

### MySQL Reference Manual

- [Encryption and Compression Functions](https://dev.mysql.com/doc/refman/8.4/en/encryption-functions.html)
- [Security Guidelines](https://dev.mysql.com/doc/refman/8.4/en/security-guidelines.html)
- [Encrypted Connections](https://dev.mysql.com/doc/refman/8.4/en/encrypted-connection-protocols-ciphers.html)
- [InnoDB Data-at-Rest Encryption](https://dev.mysql.com/doc/refman/8.4/en/innodb-data-encryption.html)
- [ALTER TABLE: ENCRYPTION Clause](https://dev.mysql.com/doc/refman/8.4/en/alter-table.html)

### Security guidance

- [OWASP Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
- [OWASP Cryptographic Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html)

> **Cảnh báo an toàn**
>
> - Chỉ dùng dữ liệu giả và key giả trong lab. Không đưa key thật, password thật, PII thật, token thật hay dữ liệu production vào SQL client, SQL history, logs, screenshots hoặc Git.
> - `AES_ENCRYPT()`/`AES_DECRYPT()` hữu ích để học SQL crypto, nhưng plaintext và key material được truyền trong statement có thể lộ qua network/logs nếu kiến trúc không được bảo vệ.
> - Không dùng AES reversible encryption hoặc fast hash như `SHA2()` để lưu mật khẩu người dùng của ứng dụng. Hãy dùng Argon2id/bcrypt/scrypt/PBKDF2 qua application/authentication framework.

---


---

# 0. Nền tảng: chọn đúng hàm theo đúng mục đích

Trước khi chạy các hàm như `TO_BASE64()`, `SHA2()`, `AES_ENCRYPT()` hoặc `AES_DECRYPT()`, cần trả lời:

```text
Ta đang cần điều gì từ dữ liệu?
```

Không có một hàm duy nhất giải quyết mọi vấn đề. Mỗi nhóm hàm phục vụ một mục tiêu khác nhau.

## 0.1. Năm câu hỏi trước khi chọn kỹ thuật

| Câu hỏi nghiệp vụ/kỹ thuật | Kỹ thuật thường phù hợp | Không nên dùng |
|---|---|---|
| Cần gửi binary data trong JSON/text? | Encoding bằng Base64 hoặc HEX | Gọi Base64 là encryption |
| Cần kiểm tra dữ liệu có thay đổi không? | Hash/checksum, ví dụ SHA-256 | AES chỉ để so sánh integrity |
| Cần đọc lại original value khi có quyền? | Encryption, ví dụ AES | Hash vì hash không đảo được |
| Cần xác thực password người dùng? | Adaptive password hashing trong application | AES hoặc SHA-256 nhanh |
| Cần tìm exact match của encrypted field? | Thiết kế lookup token/blind index có threat model | So sánh ciphertext ngẫu nhiên có salt riêng |

Ví dụ:

```text
Email phải hiển thị lại cho workflow được cấp quyền
-> encryption.

File cần kiểm tra có bị thay đổi trong lúc truyền/lưu
-> hash/checksum.

Ảnh hoặc ciphertext cần đưa vào JSON
-> Base64/HEX representation.

Password đăng nhập
-> password hashing, không phải AES/SHA2.
```

---

## 0.2. Các mục tiêu bảo mật cơ bản

| Mục tiêu | Câu hỏi cần trả lời | Ví dụ kỹ thuật liên quan |
|---|---|---|
| Confidentiality - tính bí mật | Ai có thể đọc plaintext? | Encryption, key management, TLS, least privilege |
| Integrity - tính toàn vẹn | Dữ liệu có bị thay đổi không? | Hash, HMAC, digital signature, audit |
| Authenticity - tính xác thực nguồn gốc | Dữ liệu đến từ ai? | HMAC, digital signature, access control |
| Availability - tính sẵn sàng | Khi cần, dữ liệu có đọc/khôi phục được không? | Backup, restore test, key recovery |
| Password verification | User có cung cấp đúng password không? | Argon2id/bcrypt/scrypt/PBKDF2 |

> Encryption chủ yếu hỗ trợ **confidentiality**. Nó không tự tạo access control, không tự audit truy cập, không chứng minh ai là người tạo dữ liệu, và không thay thế backup/restore.

---

## 0.3. Từ vựng tối thiểu

| Thuật ngữ | Nghĩa ngắn gọn | Ví dụ trong lab |
|---|---|---|
| Plaintext | Dữ liệu gốc có thể đọc được | `an.nguyen@example.edu.vn` |
| Ciphertext | Dữ liệu sau khi encrypt, thường là binary bytes | output của `AES_ENCRYPT()` |
| Encryption key / key material | Secret dùng để encrypt/decrypt | passphrase/key do hệ thống quản lý |
| Passphrase | Chuỗi đầu vào có thể dùng để dẫn xuất key | `LAB_ONLY_KDF_PASSPHRASE` trong lab |
| Hash digest | Fingerprint có độ dài cố định | SHA-256 digest 32 bytes |
| Salt | Random bytes không cần giữ bí mật, dùng cùng KDF/hash policy | `RANDOM_BYTES(16)` |
| IV / nonce | Giá trị bổ sung cho encryption mode; quy tắc unique/random phụ thuộc mode | argument `init_vector` của AES functions |
| KDF | Key Derivation Function: dẫn xuất key từ passphrase + salt | `pbkdf2_hmac` |
| Encoding | Cách biểu diễn dữ liệu thành format khác | HEX, Base64 |
| Binary data | Dãy bytes không nhất thiết là text UTF-8 | ciphertext, salt, raw digest |

### Luồng cơ bản của encryption

```text
Plaintext
    + key material / passphrase
    + salt / KDF parameters / IV khi áp dụng
        -> AES_ENCRYPT(...)
        -> ciphertext (binary)

ciphertext
    + cùng key material
    + cùng KDF parameters / salt / IV khi áp dụng
        -> AES_DECRYPT(...)
        -> plaintext bytes
        -> CONVERT(... USING utf8mb4) nếu plaintext gốc là text UTF-8
```

### Luồng cơ bản của hashing

```text
Original content
    -> SHA2(content, 256)
    -> fixed-length digest

Later:
Current content
    -> SHA2(current content, 256)
    -> compare with stored digest
```

Nếu content thay đổi, digest gần như chắc chắn thay đổi hoàn toàn. Đây là tính chất “avalanche effect” ở mức khái niệm: thay đổi nhỏ ở input tạo output hash rất khác.

---

## 0.4. Bảng mục đích của các hàm SQL trong lab

| Hàm / biểu thức | Input | Output | Mục đích chính | Không dùng để làm gì |
|---|---|---|---|---|
| `TO_BASE64(value)` | text hoặc binary | text Base64 | Đưa data sang text-safe representation | Che giấu dữ liệu |
| `FROM_BASE64(value)` | text Base64 | binary bytes | Decode Base64 | Decrypt AES |
| `HEX(value)` | binary hoặc text | text hexadecimal | Hiển thị/debug binary data | Encrypt dữ liệu |
| `UNHEX(value)` | text hexadecimal | binary bytes | Đổi hex text về raw bytes | Hash dữ liệu |
| `SHA2(value, 256)` | text/binary input | 64-character hex digest | Integrity/fingerprint | Recover original plaintext |
| `UNHEX(SHA2(value, 256))` | input | 32-byte binary digest | Lưu SHA-256 compact trong `BINARY(32)` | Password hashing hiện đại |
| `RANDOM_BYTES(n)` | số bytes | random binary bytes | Tạo salt / random lab value | Tạo business ID có thứ tự |
| `AES_ENCRYPT(...)` | plaintext + key params | binary ciphertext | Giữ bí mật dữ liệu có thể cần đọc lại | Verify password |
| `AES_DECRYPT(...)` | ciphertext + đúng key params | binary plaintext bytes hoặc `NULL` | Phục hồi original data khi được phép | Chứng minh ciphertext authentic |
| `CONVERT(value USING utf8mb4)` | bytes | text theo charset | Diễn giải plaintext bytes thành text | Decode mọi binary tùy ý thành text |

---

## 0.5. Vì sao phải dùng `CONVERT(... USING utf8mb4)` sau AES_DECRYPT?

`AES_DECRYPT()` trả về **bytes**. Nếu plaintext ban đầu là text UTF-8, cần nói rõ với MySQL cách diễn giải bytes đó:

```sql
SELECT CONVERT(
           AES_DECRYPT(
               email_ciphertext,
               'LAB_ONLY_KDF_PASSPHRASE',
               '',
               'pbkdf2_hmac',
               kdf_salt,
               2000
           )
           USING utf8mb4
       ) AS email_plaintext
FROM customer_private;
```

Từng phần:

1. `AES_DECRYPT(...)` phục hồi bytes gốc.
2. `CONVERT(... USING utf8mb4)` hiển thị các bytes đó như text UTF-8.
3. Alias `email_plaintext` chỉ đặt tên cột output.

Không cần `CONVERT(... USING utf8mb4)` khi mục tiêu thật sự là binary data, ví dụ file bytes hoặc key bytes.

---

## 0.6. Vì sao `SHA2()` thường đi cùng `UNHEX()` trong lab?

Ví dụ:

```sql
UNHEX(
    SHA2(
        document_content,
        256
    )
)
```

Từng bước:

1. `SHA2(document_content, 256)` tạo SHA-256 digest ở dạng **64 ký tự hex**.
2. `UNHEX(...)` biến 64 ký tự hex đó thành **32 bytes binary**.
3. Table dùng `BINARY(32)` nên lưu raw digest gọn hơn.

| Cách | Kiểu cột phù hợp | Dung lượng SHA-256 |
|---|---|---:|
| Lưu text hex | `CHAR(64)` | 64 characters |
| Lưu binary digest | `BINARY(32)` | 32 bytes |

> `UNHEX(SHA2(passphrase, 512))` trong AES demo chỉ minh họa cách chuyển hex digest thành binary key material. Nó **không** thay thế KDF password hashing hoặc production key management. Khi MySQL version hỗ trợ, phần `pbkdf2_hmac` phía sau có ý nghĩa tốt hơn cho demo dẫn xuất key từ passphrase + salt.

---

## 0.7. Salt, IV và key: ba thứ không giống nhau

| Thành phần | Có cần bí mật? | Có thể lưu cùng ciphertext? | Vai trò |
|---|---:|---:|---|
| Encryption key / master key | Có | Không nên | Bí mật cốt lõi để decrypt |
| Passphrase/key material | Có | Không nên | Input để tạo/lấy key |
| Salt | Không | Có | Làm key derivation khác nhau giữa records |
| IV/nonce | Thường không cần bí mật | Thường có | Phụ thuộc encryption mode; giúp encryption an toàn hơn khi dùng đúng policy |
| `key_version` | Không | Có | Hỗ trợ rotation/migration |

```text
Salt không phải encryption key.
Salt bị lộ không đồng nghĩa ciphertext tự decrypt được.
Key bị lộ làm confidentiality suy giảm nghiêm trọng.
```

Cùng passphrase nhưng salt khác:

```text
row 1: passphrase + salt A -> derived key A
row 2: passphrase + salt B -> derived key B
```

Vì vậy cùng plaintext email có thể tạo ciphertext khác, nên equality search trực tiếp trên ciphertext không đơn giản.

---

## 0.8. Data types trước khi gọi hàm

| Loại dữ liệu | Kiểu dữ liệu thường dùng | Ví dụ |
|---|---|---|
| Text có thể đọc | `VARCHAR`, `TEXT` | `document_content`, `product_name` |
| Binary ciphertext | `VARBINARY`, `BLOB` | `email_ciphertext` |
| Binary hash cố định 32 bytes | `BINARY(32)` | SHA-256 raw digest |
| Salt binary ngắn | `VARBINARY(16)` / `VARBINARY(32)` | KDF salt |
| Hex/Base64 để transport/debug | `CHAR`, `VARCHAR`, JSON string | `HEX(ciphertext)` result |

Không lưu ciphertext vào `VARCHAR` chỉ vì dễ nhìn. Không ép binary data qua charset conversion trừ khi thực sự làm transport/representation và hiểu format.

---

## 0.9. Decision tree: nên dùng gì?

```text
Có cần lấy lại dữ liệu gốc?
|
+-- Không
|   |
|   +-- Cần xác thực password? -> Password hashing trong application.
|   |
|   +-- Cần kiểm tra dữ liệu thay đổi? -> SHA-256 / HMAC / signature theo threat model.
|
+-- Có
    |
    +-- Có cần giữ bí mật? -> Encryption + key management.
    |
    +-- Chỉ cần đổi binary thành text để transport? -> Base64/HEX, không phải encryption.
```

---

## 0.10. Mini-check trước khi viết SQL crypto

```text
[ ] Tôi cần confidentiality, integrity, authenticity, password verification hay chỉ representation?
[ ] Plaintext có thực sự cần quay lại không?
[ ] Key/passphrase có bị đưa vào client history/log/source code không?
[ ] Output là text hay binary?
[ ] Cần lưu salt/KDF parameters/key_version nào để decrypt về sau?
[ ] Query này có khiến database decrypt toàn bảng để tìm kiếm không?
[ ] Có role nào không cần thấy plaintext?
[ ] Backup/restore có thể khôi phục key dependency không?
```

---

## 1. Mục tiêu học tập

Sau lab này, sinh viên có thể:

1. Phân biệt encoding, hashing, encryption và password hashing.
2. Giải thích vì sao Base64/HEX không phải mã hóa bảo mật.
3. Dùng `HEX()`, `UNHEX()`, `TO_BASE64()` và `FROM_BASE64()`.
4. Dùng `SHA2()` để kiểm tra toàn vẹn dữ liệu.
5. Lưu SHA-256 dưới dạng `BINARY(32)` thay vì chỉ giữ 64 ký tự hex khi phù hợp.
6. Dùng `AES_ENCRYPT()` và `AES_DECRYPT()` trong môi trường lab.
7. Thiết kế bảng lưu ciphertext, KDF salt, scheme và key version.
8. Dùng KDF `pbkdf2_hmac` trong MySQL 8.0.30+.
9. Nêu trade-off giữa encryption và khả năng tìm kiếm/indexing.
10. Nêu vai trò của TLS, key management, least privilege, backup/restore và encryption at rest.

---

## 2. Bốn khái niệm không được nhầm lẫn

| Khái niệm | Có thể phục hồi giá trị gốc? | Mục tiêu chính | Ví dụ |
|---|---:|---|---|
| Encoding | Có, không cần secret | Đại diện/transport data | Base64, HEX, UTF-8 |
| Hashing | Không theo thiết kế | Fingerprint, integrity | SHA-256 |
| Encryption | Có, cần đúng key | Giữ bí mật dữ liệu có thể đọc lại | AES |
| Password hashing | Không; intentionally slow | Xác thực password | Argon2id, bcrypt, PBKDF2 |

### 2.1. Encoding không phải encryption

```sql
SELECT TO_BASE64('secret@example.edu.vn') AS base64_value;
```

Decode ngay được:

```sql
SELECT CONVERT(
           FROM_BASE64(
               TO_BASE64('secret@example.edu.vn')
           )
           USING utf8mb4
       ) AS decoded_value;
```

**Kết quả:** `decoded_value` quay trở lại `secret@example.edu.vn`.

**Kết luận:** Base64 chỉ đổi representation. Bất kỳ ai có value đều có thể decode.

### 2.2. Hashing không phải encryption

```sql
SELECT SHA2('secret@example.edu.vn', 256) AS sha256_hex;
```

- `SHA2()` tạo fingerprint 256-bit, hiển thị dạng 64 hex characters.
- Không có hàm `SHA2_DECRYPT()`.
- Hash phù hợp để biết data có thay đổi hay không; không phù hợp khi cần hiển thị plaintext trở lại.

### 2.3. AES encryption có thể decrypt

```sql
SELECT HEX(
           AES_ENCRYPT(
               'secret@example.edu.vn',
               UNHEX(
                   SHA2(
                       'LAB_ONLY_DEMO_KEY_MATERIAL',
                       512
                   )
               )
           )
       ) AS ciphertext_hex;
```

Decrypt bằng cùng key material:

```sql
SELECT CONVERT(
           AES_DECRYPT(
               AES_ENCRYPT(
                   'secret@example.edu.vn',
                   UNHEX(
                       SHA2(
                           'LAB_ONLY_DEMO_KEY_MATERIAL',
                           512
                       )
                   )
               ),
               UNHEX(
                   SHA2(
                       'LAB_ONLY_DEMO_KEY_MATERIAL',
                       512
                   )
               )
           )
           USING utf8mb4
       ) AS decrypted_value;
```

**Kết quả:** `decrypted_value = secret@example.edu.vn`.

> Đây chỉ là demo vòng đời encryption/decryption. Phần KDF bên dưới là hướng nên học với MySQL 8.0.30+.

### 2.4. Passwords là use case riêng

```sql
-- KHÔNG dùng cho ứng dụng thật:
CREATE TABLE app_users_bad (
    user_id BIGINT PRIMARY KEY,
    username VARCHAR(100) NOT NULL,
    password_sha256 CHAR(64) NOT NULL
);
```

Không dùng AES reversible encryption cho passwords. Không dùng SHA-256 nhanh cho password storage hiện đại. Password hashing nên thực hiện trong application/authentication framework bằng Argon2id, bcrypt, scrypt hoặc PBKDF2 với salt riêng cho từng password.

### Bài tập 2

1. Phân biệt Base64 với encryption.
2. Nêu use case phù hợp của SHA-256.
3. Nêu use case phù hợp của AES encryption.
4. Vì sao không dùng SHA2 nhanh để lưu password người dùng?
5. Chọn kỹ thuật phù hợp cho: file checksum, email cần hiển thị lại, password login, binary data đưa vào JSON.

---

## 3. Chuẩn bị schema `crypto_lab`

> **Cảnh báo:** Script sau xóa toàn bộ schema `crypto_lab` nếu đã tồn tại. Chỉ dùng cho local/lab.

```sql
DROP DATABASE IF EXISTS crypto_lab;

CREATE DATABASE crypto_lab
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_0900_ai_ci;

USE crypto_lab;
```

### 3.1. Kiểm tra version và encryption mode

```sql
SELECT VERSION() AS mysql_version;
```

```sql
SHOW VARIABLES LIKE 'block_encryption_mode';
```

Không đổi global encryption mode trên shared server chỉ để làm lab.

### 3.2. Bảng integrity hash

```sql
CREATE TABLE document_integrity (
    document_id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    document_name VARCHAR(150) NOT NULL,
    document_content TEXT NOT NULL,
    content_sha256 BINARY(32) NOT NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (document_id),
    CONSTRAINT uq_document_name UNIQUE (document_name)
) ENGINE = InnoDB;
```

### 3.3. Bảng ciphertext

```sql
CREATE TABLE customer_private (
    customer_id BIGINT UNSIGNED NOT NULL,
    email_ciphertext VARBINARY(512) NOT NULL,
    tax_id_ciphertext VARBINARY(512) NULL,
    kdf_salt VARBINARY(32) NOT NULL,
    encryption_scheme VARCHAR(50) NOT NULL,
    key_version SMALLINT UNSIGNED NOT NULL DEFAULT 1,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (customer_id),
    CONSTRAINT ck_customer_private_key_version
        CHECK (key_version > 0)
) ENGINE = InnoDB;
```

### 3.4. Giải thích thiết kế

| Column | Vai trò |
|---|---|
| `customer_id` | Identifier dùng để join/audit; giữ plaintext nếu không nhạy cảm |
| `email_ciphertext` | Email giả đã encrypt, lưu binary |
| `tax_id_ciphertext` | Tax ID giả đã encrypt, lưu binary |
| `kdf_salt` | Salt ngẫu nhiên cần dùng lại để decrypt record |
| `encryption_scheme` | Hỗ trợ migration/đọc đúng scheme |
| `key_version` | Cho biết key/policy version, không phải secret |

Không lưu trong table:

```text
- Raw encryption key.
- Production passphrase.
- Password người dùng.
- Master key/KMS secret.
```

### Bài tập 3

1. Tạo schema `crypto_lab`.
2. Kiểm tra version MySQL.
3. Kiểm tra `block_encryption_mode`.
4. Tạo hai tables trên.
5. Giải thích vì sao ciphertext dùng `VARBINARY`/`BLOB` thay vì `VARCHAR`.

---

# Phần A. Encoding và representation

## 4. HEX, UNHEX và Base64

### 4.1. HEX để hiển thị binary ciphertext

`AES_ENCRYPT()` trả binary string. Dùng `HEX()` để nhìn ciphertext mà không giả vờ nó là UTF-8 text.

```sql
SELECT HEX(
           AES_ENCRYPT(
               'hello',
               UNHEX(
                   SHA2(
                       'LAB_ONLY_DEMO_KEY_MATERIAL',
                       512
                   )
               )
           )
       ) AS cipher_hex;
```

`cipher_hex` là representation để debug/display; nó không phải plaintext.

### 4.2. UNHEX

```sql
SELECT CONVERT(
           UNHEX('48656C6C6F')
           USING utf8mb4
       ) AS plaintext;
```

**Kết quả:** `Hello`.

### 4.3. Base64 cho transport text

```sql
SELECT TO_BASE64('Xin chào SQL') AS base64_value;
```

```sql
SELECT CONVERT(
           FROM_BASE64(
               TO_BASE64('Xin chào SQL')
           )
           USING utf8mb4
       ) AS decoded_text;
```

### 4.4. Binary vs HEX text vs Base64

| Cách lưu | Ưu điểm | Nhược điểm |
|---|---|---|
| `VARBINARY`/`BLOB` | Compact, đúng kiểu ciphertext | Khó đọc bằng mắt |
| HEX trong `CHAR`/`VARCHAR` | Dễ copy/debug | Tốn khoảng gấp đôi bytes |
| Base64 trong `VARCHAR` | Tiện đưa qua JSON/text | Dài hơn binary, không bảo mật |

**Khuyến nghị lab:** lưu ciphertext dạng binary; chỉ dùng HEX/Base64 ở boundary display/debug/transport.

### Bài tập 4

1. Hiển thị output AES bằng `HEX()`.
2. Decode `48656C6C6F`.
3. Encode/decode một chuỗi tiếng Việt bằng Base64.
4. Giải thích vì sao Base64 không bảo mật.
5. Chọn kiểu lưu phù hợp cho ciphertext và nêu lý do.

---

# Phần B. Hashing để kiểm tra toàn vẹn

## 5. SHA2 và integrity check

### 5.1. Lưu SHA-256 dạng binary

```sql
INSERT INTO document_integrity (
    document_name,
    document_content,
    content_sha256
)
VALUES (
    'database_policy.txt',
    'Database Systems - Version 1',
    UNHEX(
        SHA2(
            'Database Systems - Version 1',
            256
        )
    )
);
```

Kiểm tra stored value:

```sql
SELECT document_id,
       document_name,
       document_content,
       HEX(content_sha256) AS stored_sha256_hex
FROM document_integrity;
```

### 5.2. Integrity check

```sql
SELECT document_id,
       document_name,
       UNHEX(
           SHA2(
               document_content,
               256
           )
       ) = content_sha256 AS integrity_ok
FROM document_integrity;
```

**Kết quả kỳ vọng:** `integrity_ok = 1`.

### 5.3. Mô phỏng dữ liệu bị thay đổi

```sql
UPDATE document_integrity
SET document_content = 'Database Systems - Version 2'
WHERE document_name = 'database_policy.txt';
```

Chạy lại query integrity. Kết quả kỳ vọng:

```text
integrity_ok = 0
```

Lý do: content đã là Version 2 nhưng hash vẫn là Version 1.

### 5.4. Cập nhật hash trong transaction khi change hợp lệ

```sql
START TRANSACTION;

UPDATE document_integrity
SET content_sha256 = UNHEX(
        SHA2(
            document_content,
            256
        )
    )
WHERE document_name = 'database_policy.txt';

COMMIT;
```

Kiểm tra lại; `integrity_ok` trở lại `1`.

### 5.5. Hash không tự xác thực nguồn gốc

Nếu một attacker có quyền sửa cả `document_content` và `content_sha256`, họ có thể thay cả hai. Hash một mình phù hợp phát hiện accidental change hoặc đối chiếu với fingerprint tin cậy; nó không thay access control, HMAC/digital signature, audit hoặc immutable storage.

### Bài tập 5

1. Insert một document khác cùng hash binary.
2. Hiển thị stored hash dạng hex.
3. Sửa content nhưng không sửa hash, rồi tìm records lỗi integrity.
4. Khôi phục hash trong transaction.
5. Giải thích tại sao hash không tự chứng minh ai tạo document.

---

# Phần C. AES encryption/decryption

## 6. AES demo cơ bản

### 6.1. Encrypt

```sql
SELECT HEX(
           AES_ENCRYPT(
               'demo.email@example.edu.vn',
               UNHEX(
                   SHA2(
                       'LAB_ONLY_DEMO_KEY_MATERIAL',
                       512
                   )
               )
           )
       ) AS ciphertext_hex;
```

### 6.2. Decrypt đúng key

```sql
SELECT CONVERT(
           AES_DECRYPT(
               AES_ENCRYPT(
                   'demo.email@example.edu.vn',
                   UNHEX(
                       SHA2(
                           'LAB_ONLY_DEMO_KEY_MATERIAL',
                           512
                       )
                   )
               ),
               UNHEX(
                   SHA2(
                       'LAB_ONLY_DEMO_KEY_MATERIAL',
                       512
                   )
               )
           )
           USING utf8mb4
       ) AS decrypted_email;
```

**Kết quả:** `demo.email@example.edu.vn`.

### 6.3. Vì sao dùng `UNHEX(SHA2(...,512))` trong demo?

Khi không dùng KDF, không nên truyền passphrase thô trực tiếp làm `key_str`. `SHA2(...,512)` tạo hex digest; `UNHEX(...)` đổi nó thành binary key material. Đây là demo/compatibility pattern; KDF là phần cần ưu tiên học ở MySQL 8.0.30+.

### 6.4. Decrypt sai key

```sql
SELECT CONVERT(
           AES_DECRYPT(
               AES_ENCRYPT(
                   'demo.email@example.edu.vn',
                   UNHEX(
                       SHA2(
                           'LAB_ONLY_DEMO_KEY_MATERIAL',
                           512
                       )
                   )
               ),
               UNHEX(
                   SHA2(
                       'WRONG_LAB_KEY',
                       512
                   )
               )
           )
           USING utf8mb4
       ) AS wrong_key_result;
```

Kết quả thường là `NULL`, nhưng không dựa vào `NULL` như authentication/integrity guarantee. Input hoặc key không hợp lệ đôi khi có thể tạo binary garbage không phải plaintext mong muốn.

### 6.5. Không hard-code production keys

Lab dùng literal key để tự chạy. Trong hệ thống thật, không:

```text
- Lưu key cùng table ciphertext.
- Commit key vào source control.
- Đặt key trong view/routine readable bởi nhiều users.
- Dán key vào Workbench history/screenshot.
- Truyền key qua connection không TLS.
```

### Bài tập 6

1. Encrypt một chuỗi giả và hiển thị HEX.
2. Decrypt với đúng key.
3. Thử wrong key.
4. Nêu bốn nơi key có thể bị lộ.
5. Giải thích vì sao output decrypt sai không đủ để xác thực tính toàn vẹn.

---

## 7. AES với KDF và salt — MySQL 8.0.30+

> Phần này cần MySQL 8.0.30+ do KDF arguments. Nếu server thấp hơn, học khái niệm và chỉ thực hành Section 6; không triển khai crypto production bằng workaround SQL.

### 7.1. KDF và salt

MySQL 8.0.30+ hỗ trợ KDF `hkdf` và `pbkdf2_hmac` trong `AES_ENCRYPT()`/`AES_DECRYPT()`.

Trong lab dùng `pbkdf2_hmac` để thấy rõ:

```text
passphrase/key material
+ salt ngẫu nhiên
+ iterations
-> derived encryption key
```

Salt:

- Nên random.
- Có thể lưu cùng ciphertext.
- Không phải secret.
- Phải dùng lại khi decrypt record tương ứng.

### 7.2. Insert record với salt mới

Query này tạo **một salt ngẫu nhiên**, dùng đúng salt đó cho cả hai encrypted fields của customer 1, rồi lưu salt.

```sql
INSERT INTO customer_private (
    customer_id,
    email_ciphertext,
    tax_id_ciphertext,
    kdf_salt,
    encryption_scheme,
    key_version
)
SELECT
    1,
    AES_ENCRYPT(
        'an.nguyen@example.edu.vn',
        'LAB_ONLY_KDF_PASSPHRASE',
        '',
        'pbkdf2_hmac',
        s.kdf_salt,
        2000
    ),
    AES_ENCRYPT(
        '000123456789',
        'LAB_ONLY_KDF_PASSPHRASE',
        '',
        'pbkdf2_hmac',
        s.kdf_salt,
        2000
    ),
    s.kdf_salt,
    'AES_PBKDF2_HMAC_LAB',
    1
FROM (
    SELECT RANDOM_BYTES(16) AS kdf_salt
) AS s;
```

> `2000` chỉ minh họa syntax theo tài liệu MySQL. Không coi đây là work factor production; policy thật cần threat model, benchmark, version và security review.

### 7.3. Xem ciphertext/salt dạng HEX

```sql
SELECT customer_id,
       HEX(email_ciphertext) AS email_ciphertext_hex,
       HEX(tax_id_ciphertext) AS tax_id_ciphertext_hex,
       HEX(kdf_salt) AS kdf_salt_hex,
       encryption_scheme,
       key_version
FROM customer_private;
```

Không có plaintext trong output.

### 7.4. Decrypt đúng toàn bộ parameters

```sql
SELECT customer_id,
       CONVERT(
           AES_DECRYPT(
               email_ciphertext,
               'LAB_ONLY_KDF_PASSPHRASE',
               '',
               'pbkdf2_hmac',
               kdf_salt,
               2000
           )
           USING utf8mb4
       ) AS email_plaintext,
       CONVERT(
           AES_DECRYPT(
               tax_id_ciphertext,
               'LAB_ONLY_KDF_PASSPHRASE',
               '',
               'pbkdf2_hmac',
               kdf_salt,
               2000
           )
           USING utf8mb4
       ) AS tax_id_plaintext
FROM customer_private
WHERE customer_id = 1;
```

**Kết quả kỳ vọng:**

| customer_id | email_plaintext | tax_id_plaintext |
|---:|---|---|
| 1 | an.nguyen@example.edu.vn | 000123456789 |

### 7.5. Cùng plaintext, salt mới

```sql
INSERT INTO customer_private (
    customer_id,
    email_ciphertext,
    tax_id_ciphertext,
    kdf_salt,
    encryption_scheme,
    key_version
)
SELECT
    2,
    AES_ENCRYPT(
        'an.nguyen@example.edu.vn',
        'LAB_ONLY_KDF_PASSPHRASE',
        '',
        'pbkdf2_hmac',
        s.kdf_salt,
        2000
    ),
    AES_ENCRYPT(
        '999888777666',
        'LAB_ONLY_KDF_PASSPHRASE',
        '',
        'pbkdf2_hmac',
        s.kdf_salt,
        2000
    ),
    s.kdf_salt,
    'AES_PBKDF2_HMAC_LAB',
    1
FROM (
    SELECT RANDOM_BYTES(16) AS kdf_salt
) AS s;
```

So sánh:

```sql
SELECT customer_id,
       HEX(email_ciphertext) AS email_ciphertext_hex,
       HEX(kdf_salt) AS kdf_salt_hex
FROM customer_private
ORDER BY customer_id;
```

Dù cùng plaintext email, salt khác làm key derivation/ciphertext không phù hợp làm stable lookup value.

### 7.6. Parameters phải khớp khi decrypt

```text
- ciphertext
- key material/passphrase
- init vector argument (lab dùng empty string)
- KDF name
- salt
- iterations
- encryption mode/environment policy
```

### Bài tập 7

1. Kiểm tra MySQL 8.0.30+.
2. Insert customer ID 3 với dữ liệu giả và salt mới.
3. Decrypt customer 3.
4. So sánh ciphertext email của customer 1/2/3.
5. Liệt kê các parameters phải khớp để decrypt.

---

# Phần D. Query design và access control

## 8. Encryption làm search/indexing khó hơn

### 8.1. Vì sao equality lookup theo ciphertext khó?

Khi email plaintext:

```sql
SELECT customer_id
FROM customers
WHERE email = 'an.nguyen@example.edu.vn';
```

Khi email encrypt với per-row salt/KDF, không có một ciphertext fixed để so sánh equality đơn giản. Do đó không kỳ vọng query kiểu sau là thiết kế đúng:

```sql
-- Không phải production pattern:
SELECT customer_id
FROM customer_private
WHERE email_ciphertext = AES_ENCRYPT(
    'an.nguyen@example.edu.vn',
    'LAB_ONLY_KDF_PASSPHRASE',
    '',
    'pbkdf2_hmac',
    ?,
    2000
);
```

Mỗi row có salt riêng.

### 8.2. Không decrypt toàn table trong WHERE production

Demo có thể viết:

```sql
SELECT customer_id
FROM customer_private
WHERE CONVERT(
          AES_DECRYPT(
              email_ciphertext,
              'LAB_ONLY_KDF_PASSPHRASE',
              '',
              'pbkdf2_hmac',
              kdf_salt,
              2000
          )
          USING utf8mb4
      ) = 'an.nguyen@example.edu.vn';
```

Nhưng production query này thường kém vì:

```text
- Decrypt nhiều rows.
- Khó dùng index equality hiệu quả.
- Đưa key material vào query path.
- Dễ lộ plaintext ở logs/debug/result.
- Tăng CPU và attack surface.
```

### 8.3. Lookup token/blind index: chỉ ở mức concept

Một thiết kế có thể tách:

```text
email_ciphertext      : phục hồi email khi được phép
email_lookup_token    : token được thiết kế riêng để lookup equality
```

Không tự biến `SHA2(email)` thành production blind index rồi xem là đủ. Token design cần secret tách database, normalization policy, rotation, collision handling và threat model.

### 8.4. Least privilege

| Vai trò | Quyền/đối tượng nên có |
|---|---|
| Reporting user | Chỉ summary/masked view; không decrypt nếu không cần |
| Application service | DML/EXECUTE tối thiểu cho use case |
| Security workflow | Đường decrypt có approval/audit |
| DBA/platform | Quản lý hạ tầng; không nên mặc định có application secrets |

### Bài tập 8

1. Giải thích per-row salt làm equality search khó như thế nào.
2. Nêu ba lý do không decrypt toàn table trong WHERE production.
3. Phân biệt `customer_id` plaintext với encrypted email.
4. Nêu mục tiêu lookup token ở mức concept.
5. Thiết kế ba roles cho encrypted customer data.

---

## 9. Key management, TLS và encryption at rest

### 9.1. Key management vượt ra ngoài AES syntax

Một thiết kế thật cần:

```text
- Key generation.
- Key storage tách ciphertext.
- Access control.
- Key rotation/revocation.
- Backup/recovery.
- Audit.
- Environment separation.
- Incident response.
```

Nếu key mất, encrypted data có thể không decrypt được. Nếu key lộ, encryption không còn bảo vệ confidentiality như kỳ vọng.

### 9.2. Không lưu key cùng data

```sql
-- KHÔNG dùng:
CREATE TABLE crypto_keys_bad (
    key_id INT PRIMARY KEY,
    raw_key VARCHAR(500) NOT NULL
);
```

Production secrets nên nằm trong KMS/HSM/secret manager hoặc kiến trúc tương đương, tách application/database theo threat model.

### 9.3. TLS và logging hygiene

```sql
SHOW STATUS LIKE 'Ssl_cipher';
```

Nếu `Ssl_cipher` có value, session có TLS cipher active. TLS giúp bảo vệ data in transit, nhưng không tự bảo vệ dữ liệu khỏi over-privileged account, SQL injection, logging plaintext tại app/server, hoặc key leakage.

### 9.4. Data-at-rest encryption/TDE

InnoDB data-at-rest encryption bảo vệ tablespace storage layer. Nó không thay thế:

```text
- Application authorization.
- Column/application encryption.
- Secret management.
- Query/export controls.
- Backup policy.
```

TDE/keyring là nhiệm vụ DBA/platform team. Không tự bật `ENCRYPTION = 'Y'` trong lab/server dùng chung nếu chưa có keyring, backup và runbook.

### 9.5. Restore test cho encrypted data

Checklist:

```text
[ ] Backup ciphertext/table schema.
[ ] Kiểm soát keyring/KMS/key version.
[ ] Restore vào environment test.
[ ] Decrypt được record mẫu sau restore.
[ ] Xác nhận ai có quyền access key/decrypt path.
[ ] Kiểm tra policy key rotation với backup cũ.
```

### Bài tập 9

1. Nêu năm nhiệm vụ của key management.
2. Nêu ba nơi key có thể lộ ngoài database table.
3. TLS giải quyết và không giải quyết vấn đề gì?
4. So sánh TDE với column/application encryption.
5. Lập checklist restore test cho encrypted data.

---

# Phần E. Bài tập tổng hợp

## 10. Bài tập tổng hợp

### Bài 10.1. Encoding versus encryption

1. Encode một chuỗi giả bằng Base64.
2. Decode lại.
3. Encrypt cùng chuỗi bằng AES demo.
4. Hiển thị ciphertext qua HEX.
5. Decrypt bằng đúng key.
6. Kết luận: kỹ thuật nào che giấu dữ liệu, kỹ thuật nào chỉ đổi representation?

### Bài 10.2. Document integrity workflow

1. Insert ba documents với hash binary.
2. Viết query integrity check toàn bảng.
3. Sửa một document không sửa hash.
4. Lọc documents `integrity_ok = 0`.
5. Khôi phục hash hợp lệ trong transaction.
6. Đề xuất một control bổ sung ngoài hash.

### Bài 10.3. KDF encrypted customer data

1. Kiểm tra version.
2. Insert customer ID 10 với email/tax ID giả và fresh salt.
3. Query ciphertext/salt dạng HEX.
4. Decrypt đúng KDF params.
5. Cố decrypt bằng wrong passphrase.
6. Insert customer ID 11 với cùng email và fresh salt.
7. So sánh ciphertext.
8. Giải thích `key_version`.

### Bài 10.4. Encryption design review

Requirement:

```text
Lưu email, phone, tax ID;
application cần lookup customer theo email;
reporting team chỉ cần customer count theo month;
security workflow có thể xem plaintext email khi được phê duyệt.
```

Trả lời:

1. Field nào cần encryption?
2. Field nào có thể plaintext/pseudonymous identifier?
3. Vì sao ciphertext equality lookup khó?
4. Reporting team nên truy cập object nào?
5. Key nên nằm ở đâu ở mức kiến trúc?
6. Cần audit gì?
7. Nêu hai risks còn tồn tại khi table đã encrypt.

### Bài 10.5. Password storage review

Đánh giá:

```text
A. password_plaintext VARCHAR(255)
B. AES_ENCRYPT(password, key)
C. SHA2(password, 256)
```

1. Proposal nào không chấp nhận được?
2. Vì sao reversible encryption không phù hợp password storage thông thường?
3. Vì sao SHA-256 nhanh không đủ cho password storage hiện đại?
4. Đề xuất hướng đúng.
5. Salt là gì? Pepper là gì ở mức concept?
6. Password hashing nên thực hiện ở đâu?

---

## 11. Đáp án gợi ý

### Bài 10.1

```sql
SELECT TO_BASE64(
           'demo.email@example.edu.vn'
       ) AS base64_value;
```

```sql
SELECT CONVERT(
           FROM_BASE64(
               TO_BASE64(
                   'demo.email@example.edu.vn'
               )
           )
           USING utf8mb4
       ) AS decoded_value;
```

```sql
SELECT HEX(
           AES_ENCRYPT(
               'demo.email@example.edu.vn',
               UNHEX(
                   SHA2(
                       'LAB_ONLY_DEMO_KEY_MATERIAL',
                       512
                   )
               )
           )
       ) AS ciphertext_hex;
```

Kết luận:

```text
Base64 chỉ encoding; ai có value có thể decode.
AES encryption tạo ciphertext và cần đúng key để decrypt.
```

### Bài 10.2

```sql
SELECT document_id,
       document_name,
       UNHEX(
           SHA2(
               document_content,
               256
           )
       ) = content_sha256 AS integrity_ok
FROM document_integrity;
```

Lọc records bị thay đổi:

```sql
SELECT document_id,
       document_name
FROM document_integrity
WHERE UNHEX(
          SHA2(
              document_content,
              256
          )
      ) <> content_sha256;
```

### Bài 10.3

```sql
INSERT INTO customer_private (
    customer_id,
    email_ciphertext,
    tax_id_ciphertext,
    kdf_salt,
    encryption_scheme,
    key_version
)
SELECT
    10,
    AES_ENCRYPT(
        'student10@example.edu.vn',
        'LAB_ONLY_KDF_PASSPHRASE',
        '',
        'pbkdf2_hmac',
        s.kdf_salt,
        2000
    ),
    AES_ENCRYPT(
        '100000000010',
        'LAB_ONLY_KDF_PASSPHRASE',
        '',
        'pbkdf2_hmac',
        s.kdf_salt,
        2000
    ),
    s.kdf_salt,
    'AES_PBKDF2_HMAC_LAB',
    1
FROM (
    SELECT RANDOM_BYTES(16) AS kdf_salt
) AS s;
```

Decrypt:

```sql
SELECT customer_id,
       CONVERT(
           AES_DECRYPT(
               email_ciphertext,
               'LAB_ONLY_KDF_PASSPHRASE',
               '',
               'pbkdf2_hmac',
               kdf_salt,
               2000
           )
           USING utf8mb4
       ) AS email_plaintext
FROM customer_private
WHERE customer_id = 10;
```

### Bài 10.5

| Proposal | Đánh giá |
|---|---|
| A. Plaintext | Không chấp nhận được |
| B. AES reversible encryption | Không phù hợp; key compromise có thể khôi phục passwords |
| C. SHA-256 | Không phù hợp cho modern password storage vì fast hash dễ brute-force |

Hướng đúng:

```text
Application/authentication framework
-> Argon2id (ưu tiên) hoặc bcrypt/scrypt/PBKDF2
-> unique per-password salt
-> work factor phù hợp
-> optional pepper được giữ tách password database
```

---

## 12. Checklist trước khi nộp lab

| # | Mục kiểm tra | Đạt? |
|---:|---|:---:|
| 1 | Phân biệt encoding, hashing, encryption và password hashing | ☐ |
| 2 | Không gọi Base64 là encryption | ☐ |
| 3 | Dùng SHA-256 cho integrity/fingerprint, không gọi là reversible encryption | ☐ |
| 4 | Không đề xuất SHA2/AES cho password user production | ☐ |
| 5 | Ciphertext lưu VARBINARY/BLOB | ☐ |
| 6 | KDF salt được lưu cùng encrypted record | ☐ |
| 7 | Có `encryption_scheme` và `key_version` | ☐ |
| 8 | Có test decrypt đúng parameters | ☐ |
| 9 | Có test wrong key, không coi NULL là authentication guarantee | ☐ |
| 10 | Có integrity test trước/sau update | ☐ |
| 11 | Không có secret/password/PII thật | ☐ |
| 12 | Có giải thích limitation về search/indexing ciphertext | ☐ |
| 13 | Có đề cập TLS, key management, backup/restore, least privilege | ☐ |
| 14 | Cleanup chỉ nhắm `crypto_lab` | ☐ |

---

## 13. Cleanup

> Chỉ chạy khi chắc chắn đây là schema lab.

```sql
DROP DATABASE IF EXISTS crypto_lab;
```

---

## 14. Tóm tắt

- Base64 và HEX là encoding/representation, không tạo confidentiality.
- `SHA2()` là one-way hash; phù hợp integrity check nhưng không phải modern password hashing.
- `AES_ENCRYPT()` tạo binary ciphertext; `AES_DECRYPT()` cần đúng key material và parameters.
- Lưu ciphertext bằng `VARBINARY`/`BLOB`; dùng HEX/Base64 chỉ ở display/transport boundary.
- MySQL 8.0.30+ hỗ trợ `hkdf` và `pbkdf2_hmac` cho AES functions; salt không phải secret nhưng phải lưu để decrypt.
- Encryption làm equality search/sort/indexing khó hơn; không decrypt toàn table trong WHERE production.
- Key management, TLS, logs, access control, backup/restore và audit quan trọng không kém AES syntax.
- Passwords phải dùng modern adaptive password hashing ở application layer.
- InnoDB data-at-rest encryption/TDE là một tầng bảo vệ khác và không thay thế application/column encryption hoặc authorization.

---

## 15. Từ khóa chính

- Encoding
- Base64
- HEX
- UNHEX
- Hashing
- SHA2
- SHA-256
- Integrity
- Encryption
- AES_ENCRYPT
- AES_DECRYPT
- Plaintext
- Ciphertext
- VARBINARY
- BLOB
- KDF
- PBKDF2
- HKDF
- Salt
- Key Material
- Key Version
- TLS
- Key Management
- TDE
- Data-at-Rest Encryption
- Password Hashing
- Argon2id
- bcrypt
- Least Privilege
- crypto_lab
