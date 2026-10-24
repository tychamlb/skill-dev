# QUY TẮC TÁI SỬ DỤNG MÃ NGUỒN (REUSE CODES)

## 1. Nguyên tắc cốt lõi
- Luôn ưu tiên tái sử dụng code/helper hiện có trong hệ thống thay vì viết lặp lại.
- Tra cứu tìm kiếm helper cục bộ trước, sau đó tra cứu toàn cục. Cấm viết trùng lặp logic đã có.

## 2. Quy trình kiểm tra
- Trước khi viết hàm mới: Dùng ripgrep (`grep_search`) tìm kiếm tên hàm/logic tương tự trong `core/`, `routes/`, hoặc `public/static/js/`.
- Nếu hàm đã tồn tại: Import hoặc gọi trực tiếp helper có sẵn.
