---
name: session_token_lifecycle_skill
description: Quy chuẩn kiến trúc sinh Token Key, cơ chế trượt Sliding Session Window (TTL 6h, Hard Max 24h), bảo vệ chống tái phát lặp Token cũ (Anti-Replay Attack & 24h Ban), hợp nhất phiên cục bộ Localhost Deduplication, và phòng thủ chống tự đăng xuất Self-Logout Protection.
---

# Quản Trị Vòng Đời Phiên & Khóa Token Mật Mã (Session & Token Key Lifecycle Architecture)

Tài liệu chuẩn hóa kiến trúc backend và frontend cho hệ thống xác thực của FlixGate, giải quyết trọn vẹn:
1. **Cơ chế Sinh & Bảo vệ Token Key**: `crypto.randomUUID()` kết hợp JWT HttpOnly Cookie bảo vệ XSS.
2. **Cơ chế Gia hạn trượt Sliding Session Window**: Kéo dài thêm 6 giờ mỗi khi người dùng tương tác, giới hạn trần cứng tối đa 24 giờ.
3. **Mô hình "1 Phiên Mới, 1 Phiên Cũ" & Chống phát lặp Key (Anti-Replay Protection)**: Lưu trữ vết `previousSessionToken`, tự động cấm IP & thiết bị trong 24 giờ nếu vi phạm 3 lần.
4. **Hợp nhất phiên môi trường Cục bộ (Localhost Deduplication)**: Chuẩn hóa `local_host_${userId}` cho `::1`, `127.0.0.1`, LAN để không phát sinh phiên ảo trùng lặp.
5. **Cơ chế Đăng xuất Dứt điểm & Phòng thủ Tự đăng xuất (Self-Logout Protection)**: Đăng xuất xóa sạch cookie và cập nhật DB, chặn nghiêm ngặt Admin tự đăng xuất hoặc tự thu hồi phiên đang hoạt động của chính mình.

---

## 1. Cơ Chế Sinh & Truyền Tải Token Key

### 1.1. Sinh Token
Hệ thống sử dụng đồng thời 2 lớp khóa:
- **`sessionToken` (Session Key định danh trong DB)**: Sinh bằng mật mã học ngẫu nhiên cao cấp:
  ```javascript
  const crypto = require('crypto');
  const sessionToken = crypto.randomUUID ? crypto.randomUUID() : crypto.randomBytes(24).toString('hex');
  ```
- **`JWT Token` (Xác thực HTTP Request)**: Ký điện tử bằng `jsonwebtoken` với `JWT_SECRET`:
  ```javascript
  function generateToken(user, initialSessionStart = Date.now()) {
    return jwt.sign(
      {
        id: user._id,
        email: user.email,
        role: user.role,
        fullName: user.fullName,
        sessionStart: initialSessionStart // Ghi nhớ mốc bắt đầu phiên gốc ban đầu
      },
      JWT_SECRET,
      { expiresIn: SESSION_TTL_SECONDS } // 6 giờ
    );
  }
  ```

### 1.2. Vận chuyển an toàn qua HttpOnly Cookie
```javascript
function setAuthCookie(res, token) {
  res.cookie('flixgate_session', token, {
    httpOnly: true,                               // Chống 100% việc đọc qua JS/XSS
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'lax',
    maxAge: SESSION_TTL_SECONDS * 1000            // 6 giờ
  });
}
```

---

## 2. Sliding Session Window: Thời Hạn & Gia Hạn Tự Động

Mỗi khi người dùng gửi request hợp lệ (lướt web, tương tác, xem phim), middleware xác thực thực hiện thuật toán **Sliding Window**:

```text
Request đến ──► Verify JWT ──► Tính sessionAge = Date.now() - sessionStart
                                       │
            ┌──────────────────────────┴──────────────────────────┐
            ▼                                                     ▼
   sessionAge < 24 giờ                                  sessionAge >= 24 giờ
            │                                                     │
   [Cấp lại Token mới + 6h]                             [Đánh dấu status = 'expired']
   • Cookie gia hạn 6h                                  • Xóa Cookie
   • DB: expiresAt = now + 6h                           • Trả về 401 Session Max Lifetime
   • DB: lastActive = now                               • Bắt buộc đăng nhập lại
   • DB: refreshCount += 1
```

### Mã nguồn chuẩn hóa trong middleware (`auth.js`):
```javascript
const SESSION_TTL_SECONDS = 6 * 60 * 60;          // TTL: 6 giờ
const MAX_SESSION_LIFETIME_MS = 24 * 60 * 60 * 1000; // Trần tối đa: 24 giờ

const now = Date.now();
const sessionStart = decoded.sessionStart || (decoded.iat ? decoded.iat * 1000 : now);
const sessionAge = now - sessionStart;

if (sessionAge < MAX_SESSION_LIFETIME_MS) {
  // Còn trong hạn 24h -> Kéo dài thêm 6h nữa
  const refreshedToken = generateToken(userDoc, sessionStart);
  setAuthCookie(res, refreshedToken);
  res.setHeader('X-Refreshed-Token', refreshedToken);

  if (currentSession) {
    currentSession.lastActive = new Date();
    currentSession.lastRefreshedAt = new Date();
    currentSession.refreshCount = (currentSession.refreshCount || 0) + 1;
    currentSession.expiresAt = new Date(now + SESSION_TTL_SECONDS * 1000);
    await currentSession.save();
  }
} else {
  // Đã vượt quá 24h vòng đời tối đa -> Bắt buộc hết hạn
  if (currentSession) {
    currentSession.status = 'expired';
    await currentSession.save();
  }
  clearAuthCookie(res);
  return res.status(401).json({
    success: false,
    errorKey: 'session_max_lifetime_exceeded',
    message: 'Phiên đăng nhập đã chạm giới hạn 24 giờ. Vui lòng đăng nhập lại.'
  });
}
```

---

## 3. Nhận Diện Thiết Bị & Hợp Nhất Phiên Cục Bộ (Localhost Deduplication)

### 3.1. Phân biệt thiết bị thật
- Mỗi thiết bị thật được gắn chữ ký:
  ```javascript
  function generateDeviceId(ipAddress, deviceName, browser) {
    return crypto.createHash('md5').update(`${ipAddress}|${deviceName}|${browser}`).digest('hex');
  }
  ```
- **Quy tắc**: Bất kỳ sự khác biệt nào về **IP**, hoặc **Tên thiết bị**, hoặc **Trình duyệt** đều được coi là một phiên / thiết bị riêng biệt.

### 3.2. Quy tắc môi trường Cục bộ (Loopback & LAN)
- Khi chạy trên `::1`, `127.0.0.1`, `localhost` hoặc dải Private IP, **BẮT BUỘC quy chuẩn về 1 định danh duy nhất**:
  ```javascript
  const isLocal = isPrivateOrLocalIp(clientIp);
  const deviceId = isLocal ? `local_host_${user._id}` : generateDeviceId(clientIp, deviceName, browser);
  ```
- **Tự động quét dọn phiên trùng lặp (Local Deduplication)**: Trước khi trả về danh sách phiên cho Admin, tự động quét và duy trì duy nhất 1 phiên mới nhất trên local, xóa hẳn các bản ghi rác/trùng để badge số lượng luôn là `● 1 / 1`.

---

## 4. Mô Hình "1 Phiên Mới, 1 Phiên Cũ" & Chống Tái Sử Dụng Key (Anti-Replay 24h Ban)

Khi cùng một thiết bị đăng nhập lại:
1. Thông tin phiên hiện tại được lưu lại vào vết cũ:
   ```javascript
   existingSession.previousSessionToken = existingSession.sessionToken;
   existingSession.previousSession = {
     sessionToken: existingSession.sessionToken,
     lastActive: existingSession.lastActive,
     expiresAt: existingSession.expiresAt,
     status: existingSession.status
   };
   ```
2. Gán `sessionToken` mới và gia hạn thời gian.
3. **Phát hiện tái sử dụng Token cũ (Old Key Replay)**:
   - Nếu client gửi request mang token trùng với `previousSessionToken`:
   - Tăng `replayViolationCount`.
   - Nếu vi phạm **từ 3 lần trở lên** ➔ Ghi bản ghi vào `BlockedDeviceIp`, **CHẶN TOÀN BỘ IP VÀ DEVICE TRONG 24H** (`403 device_ip_blocked_24h`).

---

## 5. Quy Chuẩn Đăng Xuất Dứt Điểm & Phòng Thủ Tự Đăng Xuất (Self-Logout Protection)

### 5.1. Đăng xuất triệt để (`POST /api/auth/logout`)
Không chỉ xóa cookie, hệ thống cập nhật dứt điểm bản ghi DB:
```javascript
await UserLoginSession.updateMany(
  {
    userId: decoded.id,
    $or: [
      { sessionToken: token },
      { deviceId }
    ]
  },
  { $set: { status: 'logged_out', lastActive: new Date() } }
);
clearAuthCookie(res);
```

### 5.2. Phòng thủ tự đăng xuất chính mình (Self-Logout Protection)
- **Ở Frontend**: Ẩn nút "Đăng xuất phiên này" đối với thẻ phiên hiện tại (`isCurrent`).
- **Ở Backend (`DELETE /api/admin/users/:userId/sessions/:sessionId`)**:
  ```javascript
  const isSelfSession = (
    (currentAdminToken && session.sessionToken === currentAdminToken) ||
    (req.user && req.user.id === userId && (session.deviceId === `local_host_${userId}` || session.deviceId === 'local_host_' + req.user.id))
  );

  if (isSelfSession) {
    return res.status(400).json({
      success: false,
      errorKey: 'cannot_revoke_own_session',
      message: 'Không thể tự đăng xuất chính phiên đang hoạt động của bạn. Vui lòng dùng nút Đăng xuất trên thanh điều hướng.'
    });
  }
  ```

---

## 6. Tiêu Chuẩn Nghiệm Thu & Checklist Kiểm Tra

| Tiêu chuẩn | Điều kiện kiểm thử | Kết quả mong đợi |
| :--- | :--- | :--- |
| **Gia hạn trượt (Sliding)** | Thực hiện request sau 10 phút | `refreshCount` tăng +1, `expiresAt` tự lùi thêm 6h, token mới được cấp qua cookie. |
| **Trần cứng 24h** | Sửa `sessionStart` lùi quá 24h | Bị chặn `401 session_max_lifetime_exceeded`, cookie bị xóa. |
| **Local Deduplication** | Đăng nhập trên `localhost` nhiều lần | Cột phiên hiển thị `● 1 / 1`, không bao giờ đếm trùng thành `2 / 2`. |
| **Bảo vệ tự đăng xuất** | Mở modal phiên của chính Admin | Thẻ phiên hiện tại có huy hiệu `Thiết bị hiện tại`, không có nút Đăng xuất. |
| **Chống lặp key (Replay)** | Dùng Postman spam token cũ 3 lần | IP & Thiết bị bị khóa 24 giờ với mã lỗi `403`. |
