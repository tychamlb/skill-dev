# skill-dev

Kho lưu trữ cấu hình quy chuẩn Agent, hệ thống chỉ thị hành vi và danh mục kỹ năng (Skills).

---

## 🚀 Hướng dẫn Clone vào dự án

### Trường hợp 1: Dự án chưa có thư mục `.agents` (Clone mới)
Khi bạn đang ở thư mục gốc của một dự án (như FlixGate, proxy-system...) và muốn clone về để thư mục đích có tên là `.agents`:

```bash
git clone https://github.com/tychamlb/skill-dev.git .agents
```

> **💡 Lưu ý:** Bắt buộc có chữ `.agents` ở cuối lệnh để Git đặt đúng tên thư mục, không bị tạo thành thư mục `skill-dev`.

---

### Trường hợp 2: Dự án ĐÃ CÓ sẵn thư mục `.agents` (Đổi liên kết và ghi đè nội dung mới)
Nếu dự án đã có sẵn thư mục `.agents` (khiến lệnh clone báo lỗi `destination path '.agents' already exists and is not an empty directory`), bạn dùng lệnh sau để liên kết sang repo này và ép ghi đè đồng bộ hoàn toàn:

```bash
cd .agents
git init
git remote remove origin 2>/dev/null || true
git remote add origin https://github.com/tychamlb/skill-dev.git
git fetch origin
git branch -M main
git reset --hard origin/main
cd ..
```

---

## 🔄 Cập nhật Skills mới nhất (Sau khi đã liên kết xong)
Bất kỳ khi nào muốn cập nhật nội dung mới nhất từ repo về thư mục `.agents` của dự án:
```bash
git -C .agents pull origin main
```
