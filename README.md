# skill-dev

Kho lưu trữ cấu hình quy chuẩn Agent, hệ thống chỉ thị hành vi và danh mục kỹ năng (Skills).

---

## 🚀 Hướng dẫn Clone vào dự án

Khi bạn đang ở thư mục gốc của một dự án (như FlixGate hoặc bất kỳ project nào) và muốn clone về để thư mục đích có tên là `.agents`, chỉ cần thêm tham số `.agents` vào cuối lệnh:

```bash
git clone https://github.com/tychamlb/skill-dev.git .agents
```

### 💡 Lưu ý quan trọng:
- Nếu chỉ chạy `git clone https://github.com/tychamlb/skill-dev.git`, mặc định Git sẽ tạo thư mục `skill-dev`.
- Thêm `.agents` ở cuối sẽ yêu cầu Git đặt tên thư mục là `.agents` đúng chuẩn hệ thống Agentic coding.

---

## 🔄 Cập nhật Skills mới nhất khi đã có sẵn thư mục:
Nếu trong project đã clone `.agents` rồi và muốn kéo các cập nhật mới nhất về:
```bash
git -C .agents pull origin main
```
