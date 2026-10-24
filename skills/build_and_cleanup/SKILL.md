# QUY TẮC DỌN DẸP & ĐÓNG GÓI (BUILD AND CLEANUP)

## 1. Quản lý Môi trường Virtualenv (venv)
- Mọi thao tác cài đặt thư viện Python phải được thực hiện trong môi trường ảo `venv` (nếu có).

## 2. Dọn dẹp Cache & Artifacts
- Tự động dọn dẹp các thư mục cache `__pycache__`, `.pytest_cache` trước và sau khi build hoặc khởi động lại ứng dụng.
- Đảm bảo các file đóng gói đầu ra (binary, bundle) nằm đúng thư mục đích (`public/dist/`).
