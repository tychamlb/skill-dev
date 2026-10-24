# Skill Kỹ Thuật: Quản Lý Cụm Đám Mây Đa Tầng (Multi-Cloud Vault Management)

Tài liệu quy chuẩn kỹ thuật cho hệ thống điều phối 3 trụ cột tài nguyên đám mây: **MongoDB Clusters**, **Hugging Face S3/Buckets Storage**, và **Google Drive Multi-Accounts**.

---

## 1. Nguyên Lý Cốt Lõi

### A. Quản lý bí mật động & Mã hóa Zero-Leak
- Toàn bộ chuỗi kết nối (`mongodb+srv://...`), token (`hf_...`), và token Google Drive đều được mã hóa bằng chuẩn **AES-256-GCM** thông qua `backend/src/utils/cryptoVault.js`.
- Không hardcode vào code hay file tĩnh, không phụ thuộc vào việc phải SSH lên VPS sửa `.env` mỗi khi mở rộng thêm tài nguyên.

### B. Giới Hạn Quota & Chuyển Đổi Tự Động (Read-Only Rollover)
1. **Kiểm tra ngưỡng lưu trữ**:
   - Mỗi MongoDB hoặc Bucket có thể đặt hạn mức (Quota), ví dụ `100MB`.
   - Khi dung lượng sử dụng đạt hoặc vượt ngưỡng (`usedBytes >= quotaBytes`):
     - Đặt trạng thái của node thành `readonly`.
     - Chuyển `isCurrentWriteTarget = false`.
     - Kích hoạt node kế tiếp trong danh sách `standby` thành `active` để tiếp tục ghi dữ liệu mới.
2. **Nguyên tắc đọc (Read Routing)**:
   - Các node ở chế độ `readonly` vẫn duy trì kết nối bình thường để phục vụ các yêu cầu đọc/truy vấn dữ liệu cũ.

### C. Cơ Chế Chống Ngủ Đông / Bị Xóa (Keep-Alive Micro-Ping Mỗi 4 Phút)
- Các dịch vụ Free Tier (như MongoDB Atlas M0, Hugging Face Spaces/Buckets) thường bị tạm ngưng (pause) hoặc cảnh báo xóa nếu không có truy vấn trong một khoảng thời gian.
- **Giải pháp**:
  - Tự động chạy nền mỗi **4 phút**.
  - **MongoDB**: Thực hiện chèn 1 bản ghi vi mô vào collection `_keep_alive_pings` rồi xóa ngay lập tức (`insertOne` -> `deleteOne`). Việc này phát sinh transaction ghi thực tế mà không làm tốn dung lượng hay ảnh hưởng dữ liệu thật.
  - **Buckets/Storage**: Gửi truy vấn HEAD/GET nhẹ để đo độ trễ và giữ kết nối sẵn sàng.

### D. Sao Lưu Song Song & Phục Hồi Thảm Họa (Disaster Recovery 15 Ngày)
- **Google Drive làm Kho Lưu Trữ Song Song Dự Phòng (Cold/Warm Vault)**:
  - Khi lưu trữ dữ liệu quan trọng vào MongoDB hoặc đẩy ảnh vào Buckets, hệ thống gửi song song 1 bản sao lưu mã hóa sang Google Drive.
  - Tích hợp endpoint cấp `access_token` ngắn hạn 1 giờ (`https://drive.mrlht.store/api/oauth/token/short-lived?account=...`) và tự động tái cấp.
- **Bảo hiểm dữ liệu 15 ngày**:
  - Nếu trong vòng 15 ngày phát hiện MongoDB phụ hoặc Bucket bị lỗi kết nối, dữ liệu trả về rỗng (null), hoặc tài nguyên bị xóa đột ngột:
    - Báo động đỏ trên bảng quản trị.
    - Cung cấp cơ chế tự động hoặc kích hoạt 1-click để đồng bộ lại dữ liệu từ Google Drive sang cụm mới thay thế.

---

## 2. Cấu Trúc Bảng Dữ Liệu (`CloudResource`)

```javascript
const cloudResourceSchema = new mongoose.Schema({
  resourceId: { type: String, unique: true, index: true }, // cr_mongo_01, cr_bucket_01, cr_drive_01
  type: { type: String, enum: ['mongodb', 'bucket', 'drive'], required: true, index: true },
  name: { type: String, required: true },
  encryptedConfig: { type: String, required: true }, // AES-256-GCM cipher
  quotaBytes: { type: Number, default: 104857600 },   // Mặc định 100MB
  usedBytes: { type: Number, default: 0 },
  maxSpeedMbps: { type: Number, default: 0 },         // Giới hạn tốc độ (0 = không giới hạn)
  status: { type: String, enum: ['active', 'readonly', 'standby', 'error'], default: 'active' },
  isCurrentWriteTarget: { type: Boolean, default: false },
  keepAlive: {
    enabled: { type: Boolean, default: true },
    intervalMinutes: { type: Number, default: 4 },
    lastPingAt: { type: Date },
    lastPingStatus: { type: String, enum: ['success', 'failed', 'pending'], default: 'pending' },
    lastPingMessage: { type: String }
  },
  createdAt: { type: Date, default: Date.now }
});
```

---

## 3. Quy Trình API Chuẩn Quản Trị (Admin Endpoints)

- `GET /api/admin/cloud-resources`: Lấy danh sách tài nguyên và trạng thái dung lượng.
- `POST /api/admin/cloud-resources`: Thêm tài nguyên mới (tự động mã hóa AES-256-GCM trước khi lưu).
- `PUT /api/admin/cloud-resources/:resourceId`: Điều chỉnh quota, tốc độ tối đa hoặc chuyển trạng thái sang `readonly` / `active`.
- `POST /api/admin/cloud-resources/:resourceId/ping`: Thực hiện ping và ghi/xóa vi mô kiểm tra tức thì.
- `POST /api/admin/cloud-resources/restore-from-drive`: Khôi phục dữ liệu từ bản sao lưu Google Drive.
