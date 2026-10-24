# QUY TẮC COMMIT TỰ ĐỘNG VÀ QUẢN LÝ THỜI ĐIỂM COMMIT (COMMIT WORKFLOW)

## 1. Tự động Commit khi Chuyển Chức Năng Mới
- Khi đang làm Chức năng 1 (hoặc nhiệm vụ A) mà người dùng yêu cầu chuyển sang Chức năng 2 (hoặc nhiệm vụ B) ➔ Tự động hiểu rằng Chức năng 1 đã hoàn tất ổn định không lỗi.
- **BẮT BUỘC phải thực hiện `git add` và `git commit`** để lưu mốc Chức năng 1 trước khi bắt đầu triển khai Chức năng 2.

## 2. Giới hạn khi đang Sửa lỗi (Debugging / Fixing)
- Khi đang trong quá trình sửa lỗi hoặc chỉnh sửa tính năng cũ: **Tuyệt đối KHÔNG tự ý `git commit`** trừ khi người dùng có yêu cầu commit rõ ràng.

## 3. Thực thi Quy tắc 6 Sau khi Hoàn thành Yêu cầu
- Mỗi khi làm xong bất kỳ yêu cầu nào, **bắt buộc thực hiện Quy tắc 6** (khởi động/chạy lại ứng dụng/Flask server tự động) để áp dụng ngay phiên bản mã nguồn mới nhất.
