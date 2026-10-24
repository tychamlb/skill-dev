# Kỹ Năng Kỹ Thuật: Mã Hóa Bí Mật Động AES-256-GCM Vault (Dynamic Secret Vault Encryption)

Tài liệu nguyên lý kỹ thuật, cấu hình tham số chuẩn quân đội (Military-Grade Cryptography), quy cách đóng gói và mã nguồn module độc lập (Zero-Dependency) sử dụng chuẩn **AES-256-GCM** kết hợp **PBKDF2** cho hệ thống FlixGate.

---

## 1. Nguyên Lý & Yêu Cầu Cốt Lõi

### A. Vấn đề giải quyết
- Trong hệ thống streaming và lưu trữ đa đám mây (Google Drive nhiều tài khoản, MongoDB phụ, Hugging Face Token, Cloudflare Worker HMAC Key...), người quản trị cần liên tục thêm mới hoặc thay đổi thông tin kết nối.
- **Không thể mỗi lần thêm một tài khoản hay bucket mới lại phải SSH vào VPS để sửa file `.env` và restart server**.
- Toàn bộ bí mật cần được lưu trực tiếp vào Database dưới dạng mã hóa an toàn.
- **Nguyên tắc "Sai mật khẩu là KHÔNG THỂ giải mã"**: Nếu không có mật khẩu chính xác từ `.env` (biến `password`), dữ liệu hoàn toàn là chuỗi nhị phân vô nghĩa, không thể giải mã bằng bất kỳ hình thức nào.

### B. Tiêu chuẩn Cryptography Chuẩn Quân Đội (Military-Grade)
1. **Thuật toán cốt lõi: AES-256-GCM (Galois/Counter Mode)**:
   - Là thuật toán mã hóa đối xứng có xác thực toàn vẹn (**AEAD - Authenticated Encryption with Associated Data**).
   - Độ dài khóa: **256 bits** (32 bytes).
   - Sinh kèm một mã xác thực toàn vẹn (**Auth Tag - 16 bytes**). Nếu mật khẩu sai dù chỉ 1 ký tự, hoặc dữ liệu bị sửa đổi dù 1 bit, OpenSSL sẽ throw Exception ngay lập tức (`Unsupported state or unable to authenticate data`).
2. **Chống Brute-force và Rainbow Table: PBKDF2 với SHA-256**:
   - Sử dụng **100,000 vòng lặp (Iterations)**.
   - Kết hợp **Salt ngẫu nhiên 16 bytes** (`crypto.randomBytes(16)`) sinh mới cho mỗi lần mã hóa.
   - **Tính chất**: Cùng một văn bản và cùng một mật khẩu, mỗi lần mã hóa sẽ luôn sinh ra chuỗi kết quả hoàn toàn khác nhau.
3. **Initialization Vector (IV)**:
   - Dùng 12 bytes ngẫu nhiên chuẩn khuyến nghị của NIST cho GCM mode.

---

## 2. Quy Cách Đóng Gói Chuỗi Mã Hóa (Storage Format)

Dữ liệu được đóng gói thành chuỗi Hex duy nhất ngăn cách bằng dấu hai chấm `:`, có thể lưu trực tiếp vào các trường `String` của MongoDB:

```text
[saltHex]:[ivHex]:[tagHex]:[cipherHex]
```

- `saltHex`: 32 ký tự Hex (16 bytes Salt)
- `ivHex`: 24 ký tự Hex (12 bytes IV)
- `tagHex`: 32 ký tự Hex (16 bytes Auth Tag)
- `cipherHex`: Chuỗi nhị phân nội dung đã mã hóa (Ciphertext)

---

## 3. Mã Nguồn Module Chuẩn (`backend/src/utils/cryptoVault.js`)

```javascript
const crypto = require('crypto');

const ALGORITHM = 'aes-256-gcm';
const IV_LENGTH = 12;      // 12 bytes chuẩn NIST GCM
const SALT_LENGTH = 16;    // 16 bytes Salt
const ITERATIONS = 100000; // 100,000 vòng PBKDF2 chống GPU brute-force
const KEY_LENGTH = 32;     // 256 bits

function getMasterPassword(customPassword = null) {
  const masterKey = customPassword || process.env.password;
  if (!masterKey) {
    throw new Error('Chưa cấu hình mật khẩu mã hóa (process.env.password)');
  }
  return masterKey;
}

/**
 * Mã hóa chuỗi văn bản bằng Master Password
 */
function encryptVault(plainText, customPassword = null) {
  if (typeof plainText !== 'string') {
    throw new Error('Dữ liệu cần mã hóa phải là chuỗi string');
  }

  const masterPassword = getMasterPassword(customPassword);
  const salt = crypto.randomBytes(SALT_LENGTH);
  const iv = crypto.randomBytes(IV_LENGTH);

  const key = crypto.pbkdf2Sync(masterPassword, salt, ITERATIONS, KEY_LENGTH, 'sha256');

  const cipher = crypto.createCipheriv(ALGORITHM, key, iv);
  const encrypted = Buffer.concat([cipher.update(plainText, 'utf8'), cipher.final()]);
  const tag = cipher.getAuthTag();

  return [
    salt.toString('hex'),
    iv.toString('hex'),
    tag.toString('hex'),
    encrypted.toString('hex')
  ].join(':');
}

/**
 * Giải mã chuỗi - Ném lỗi ngay nếu SAI MẬT KHẨU
 */
function decryptVault(cipherPackage, customPassword = null) {
  if (!cipherPackage || typeof cipherPackage !== 'string') {
    throw new Error('Dữ liệu mã hóa không hợp lệ');
  }

  const parts = cipherPackage.split(':');
  if (parts.length !== 4) {
    throw new Error('Định dạng gói mã hóa không đúng chuẩn AES-256-GCM Vault');
  }

  const [saltHex, ivHex, tagHex, encryptedHex] = parts;
  const salt = Buffer.from(saltHex, 'hex');
  const iv = Buffer.from(ivHex, 'hex');
  const tag = Buffer.from(tagHex, 'hex');
  const encrypted = Buffer.from(encryptedHex, 'hex');

  const masterPassword = getMasterPassword(customPassword);
  const key = crypto.pbkdf2Sync(masterPassword, salt, ITERATIONS, KEY_LENGTH, 'sha256');

  const decipher = crypto.createDecipheriv(ALGORITHM, key, iv);
  decipher.setAuthTag(tag);

  try {
    const decrypted = Buffer.concat([decipher.update(encrypted), decipher.final()]);
    return decrypted.toString('utf8');
  } catch (err) {
    throw new Error('Xác thực thất bại: Mật khẩu sai hoặc dữ liệu mã hóa đã bị can thiệp/hư hại.');
  }
}

/**
 * Kiểm tra cấu trúc chuỗi có đúng format Vault
 */
function isVaultCipher(text) {
  if (!text || typeof text !== 'string') return false;
  const parts = text.split(':');
  if (parts.length !== 4) return false;
  return parts.every(part => /^[0-9a-fA-F]+$/.test(part) && part.length > 0);
}

module.exports = {
  encryptVault,
  decryptVault,
  isVaultCipher
};
```

---

## 4. Tích Hợp API Dành Cho Quản Trị Viên (Admin Routes)

### `POST /api/admin/vault/encrypt`
- **Mục đích**: Mã hóa các giá trị nhạy cảm trước khi lưu DB.
- **Body**: `{ "text": "secret_string", "customPassword": "optional_override" }`
- **Response**: `{ "success": true, "data": { "encrypted": "..." } }`

### `POST /api/admin/vault/decrypt`
- **Mục đích**: Kiểm tra chuỗi mã hóa và đọc lại giá trị khi cần xử lý kết nối.
- **Body**: `{ "cipher": "...", "customPassword": "optional_override" }`
- **Response**: `{ "success": true, "data": { "decrypted": "..." } }` (hoặc lỗi nếu sai mật khẩu).
