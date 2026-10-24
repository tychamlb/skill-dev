# QUY TRÌNH UNDO & REDO TỰ ĐỘNG (UNDO REDO WORKFLOW)

## 1. Lưu mốc Git trước khi hoàn tác
- Khi nhận lệnh `/undo` hoặc yêu cầu hoàn tác, AI tự động lưu mốc mã nguồn hiện tại vào Git Reflog.

## 2. Tự động xử lý lệnh /redo
- Khi nhận lệnh `/redo` hoặc `redo`, AI tự động kiểm tra `git reflog` và thực thi `git reset --hard <commit-id>` để quay lại ngay lập tức mà không cần hỏi lại.
