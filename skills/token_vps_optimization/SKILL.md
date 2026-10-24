# QUY TẮC TỐI ƯU HÓA TOKEN & THAO TÁC VPS (TOKEN VPS OPTIMIZATION)

## 1. Chỉnh sửa file nhỏ qua Patch
- BẮT BUỘC sử dụng các thao tác sửa file nhỏ (`replace_file_content` / `multi_replace_file_content`) để sửa đúng dòng code cần thiết.
- Tuyệt đối KHÔNG ghi đè toàn bộ file lớn để tiết kiệm tối đa Token.

## 2. Sử dụng Script tự động trên VPS
- Ưu tiên sử dụng các shell script có sẵn trên VPS (`./start_flask.sh`, `systemctl restart proxy-flask.service`) thay vì chạy thủ công chuỗi lệnh rườm rà.
