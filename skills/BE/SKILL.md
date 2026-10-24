# Skill Kỹ thuật Backend cho FlixGate (BE Architecture & Guidelines)

Tài liệu danh mục chuẩn kỹ thuật kiến trúc máy chủ, bảo mật, xác thực và lưu trữ dữ liệu của FlixGate. Mỗi hạng mục kỹ thuật chuyên sâu được chia thành từng sub-skill độc lập trong thư mục `BE/`.

---

## 1. Bảo mật & Quản lý Bí mật Động (Vault Encryption AES-256-GCM)
- Tuyệt đối không hardcode API Key, Token Google Drive, URI MongoDB phụ, Hugging Face Token vào file cấu hình tĩnh hoặc yêu cầu sửa file `.env` mỗi lần thêm tài nguyên.
- Sử dụng mã hóa đối xứng **AES-256-GCM** kết hợp **PBKDF2** (100,000 vòng lặp) với mật khẩu `password` trong `.env`.
- Chuẩn hóa format lưu trữ: `salt:iv:tag:cipherHex`. Sai pass là văng lỗi ngay lập tức, không thể giải mã.
- **Chi tiết kỹ thuật & mã nguồn mẫu**: [.agents/skills/BE/vault_encryption_skill/SKILL.md](file:///d:/ICDingHoc/Project-2026/FlixGate/.agents/skills/BE/vault_encryption_skill/SKILL.md)

---

## 2. Quản Lý Cụm Đám Mây Đa Tầng (Multi-Cloud Vault: MongoDB, Buckets & Google Drive)
- Điều phối tập trung 3 trụ cột tài nguyên: MongoDB Clusters, Hugging Face Buckets/S3 và Google Drive Multi-Accounts.
- **Cơ chế Keep-Alive định kỳ 4 phút**: Chèn và xóa vi mô 1 bản ghi trên MongoDB để các cụm Free Tier không bị đóng băng/xóa vì không hoạt động; gửi heartbeat tới Buckets.
- **Giới hạn Quota & Rollover Chỉ Đọc**: Đặt hạn mức lưu trữ (ví dụ 100MB). Khi chạm trần, tự động khóa ghi chuyển sang `readonly` và kích hoạt node chờ (`standby`) tiếp theo để ghi mới.
- **Sao lưu song song & Bảo hiểm thảm họa 15 ngày**: Google Drive lưu trữ song song bản sao lưu của MongoDB và Buckets; tự động phát hiện lệch dữ liệu hoặc khôi phục khi cụm chính bị ngắt kết nối/xóa đột ngột.
- **Chi tiết kỹ thuật & mã nguồn mẫu**: [.agents/skills/BE/multi_cloud_vault_skill/SKILL.md](file:///d:/ICDingHoc/Project-2026/FlixGate/.agents/skills/BE/multi_cloud_vault_skill/SKILL.md)

---

## 3. Quản lý Phiên Đăng Nhập & Sliding Session (Token Lifecycle & Anti-Replay)
- Xác thực qua HttpOnly Cookie an toàn (`flixgate_session`).
- Sinh Token Key mật mã học qua `crypto.randomUUID()`, gắn liền với từng phiên đăng nhập (`UserLoginSession`).
- Duy trì thời hạn hoạt động (Sliding Window): 6 giờ cho mỗi lần tương tác, chặn tối đa 24 giờ.
- Cơ chế Anti-Replay: Phát hiện dùng lại Token cũ và tự động cấm IP & thiết bị 24 giờ nếu vi phạm 3 lần.
- Hợp nhất phiên môi trường Cục bộ (Localhost Deduplication): Chuẩn hóa `local_host_${userId}` cho `::1`, `127.0.0.1` để không đếm trùng thành nhiều phiên.
- Bảo vệ an toàn chống tự đăng xuất (Self-Logout Protection) cho Admin trên cả Frontend và Backend.
- **Chi tiết kỹ thuật & mã nguồn mẫu**: [.agents/skills/BE/session_token_lifecycle_skill/SKILL.md](file:///d:/ICDingHoc/Project-2026/FlixGate/.agents/skills/BE/session_token_lifecycle_skill/SKILL.md)

---

## 4. Quản lý GeoIP & Cache 2 Lớp (Hybrid Geo Session & Egress Probe)
- **Lớp 1 (0ms)**: Nhận diện IP nội bộ (`127.0.0.1`, `::1`, Private LAN), dùng cơ chế Host Egress IP để phân giải chuẩn Quốc gia & Lá cờ thật của máy chủ chủ trì.
- **Lớp 2 (<0.2ms)**: Tra cứu cache DB `IpGeoCache` trước khi gọi dịch vụ bên ngoài, ghi vĩnh viễn với đầy đủ quận/huyện, thành phố, zip, múi giờ chi tiết.
- **Lá cờ chuẩn**: Thuật toán Unicode Regional Indicator + Vector SVG flag icon đồng bộ với bộ chọn ngôn ngữ.
- **Chi tiết kỹ thuật & mã nguồn mẫu**: [.agents/skills/BE/geoip_session_resolution_skill/SKILL.md](file:///d:/ICDingHoc/Project-2026/FlixGate/.agents/skills/BE/geoip_session_resolution_skill/SKILL.md)

---

## 5. Xử lý Upload & Truyền tải Dữ liệu (Media & Streaming Pipeline)
- Sử dụng stream buffer, hạn chế ghi file rác xuống ổ đĩa cục bộ.
- Convert và tối ưu hình ảnh sang WebP trước khi đồng bộ lên các bộ nhớ đám mây (Hugging Face Datasets / Google Drive).
