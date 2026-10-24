# QUY TẮC BẢO VỆ MÃ NGUỒN (SOURCE PROTECTION)

## 1. Ranh giới phạm vi tác động
- Chỉ chỉnh sửa đúng phạm vi file và hàm được yêu cầu. Tuyệt đối không refactor hoặc thay đổi các đoạn code không liên quan.
- Giữ nguyên kiến trúc hiện tại, các router, helper và API contract gốc của dự án.

## 2. Xử lý code thừa & môi trường
- Không tự ý xóa code cũ. Muốn comment out hoặc loại bỏ code thừa phải giải thích lý do và xác nhận.
- Tuyệt đối không tự ý tác động hoặc cấu hình thiết bị/môi trường ngoài yêu cầu.

## 3. Nguyên tắc "Mổ nội soi" (Surgical Precision Editing)
- Khi nhận yêu cầu tinh chỉnh một thành phần cụ thể (ví dụ: chỉ sửa thanh tìm kiếm hoặc các nút bấm):
  - **Bắt buộc** cô lập chính xác tọa độ dòng của khối code đó.
  - **Tuyệt đối không đụng chạm đến các khối code liền kề** (như bảng dữ liệu, cột đếm, modal chi tiết, các handler sự kiện bên dưới).
  - Nghiêm cấm hành vi ghi đè toàn bộ file hoặc xóa bỏ các component con của tính năng mà người dùng không yêu cầu sửa.

## 4. Kiểm Soát Đọc Mã (Context Focus & Anti-Drift)
- Khi giải quyết một lỗi hoặc yêu cầu nhỏ, chỉ đọc các file và khối hàm thực sự liên quan.
- Tránh đọc lan man toàn bộ dự án khiến phân tán ngữ cảnh (context drift), dẫn đến việc suy diễn sai lệch và sửa nhầm các chức năng đang hoạt động bình thường.
