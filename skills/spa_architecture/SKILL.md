# QUY CHUẨN KIẾN TRÚC SPA (SPA ARCHITECTURE)

> **CHỈ THỊ CỐT LÕI**: Mọi tính năng hay màn hình SPA mới **BẮT BUỘC IMPORT VÀ ÁP DỤNG TRỌN VẸN CÁC SKILL TƯƠNG ỨNG** từ thư mục `.agents/skills/`. Tuyệt đối không tự ý code sơ sài, chắp vá làm chất lượng kém hơn chuẩn kỹ thuật đã có trong skill.

## 0. Checklist Import & Áp Dụng Skill Bắt Buộc Khi Thiết Kế Chức Năng Mới
Trước khi bắt đầu thiết kế/code bất kỳ chức năng hay view SPA mới nào, phải đối chiếu checklist:
- [ ] **Định tuyến 2 chiều (Deep Linking)**: Áp dụng mục 2 của `spa_architecture` (hỗ trợ mở trực tiếp bằng URL sạch, F5 không 404, Back/Forward popstate).
- [ ] **Modal / Hộp thoại**: Áp dụng `.agents/skills/FE/popover_modal_skill` (Native HTML5 Popover API trên Top Layer, nút đóng `✕` đối xứng xoay đỏ 90°, khử hoàn toàn thanh cuộn thừa).
- [ ] **Thanh cuộn bảng / danh sách**: Áp dụng `.agents/skills/FE/scrollbar_ultrathin_skill` (Thanh cuộn 5px mượt mà).
- [ ] **Thanh tìm kiếm & Thanh bên**: Áp dụng `FE/search_bar_skill` và `FE/sidebar_controls_skill`.
- [ ] **Đa ngôn ngữ Zero Hardcoded**: Áp dụng `bilingual_i18n` (Bổ sung đủ 6 ngôn ngữ `vi`, `en`, `zh`, `ja`, `ko`, `hmn`).
- [ ] **Bảo mật dữ liệu nhạy cảm**: Áp dụng `BE/vault_encryption_skill` (Mã hóa AES-256-GCM trước khi lưu).

## 1. Phân cấp Thư Mục & Cấu Trúc Module Đa Tầng (Multi-Tier Feature Hierarchy)
- **Nguyên lý Phân tách Ranh giới Đặc quyền (Admin vs User Separation of Concerns):**
  - **Thư mục Admin (`components/admin/` & `routes/admin/`)**:
    - **CHỈ DÀNH RIÊNG CHO QUẢN TRỊ VIÊN**.
    - Tuyệt đối không cho phép user thường truy cập, import hoặc dùng chung bất kỳ module nào trong thư mục này.
    - Toàn bộ backend routes thuộc `routes/admin/` bắt buộc được bảo vệ bởi middleware `requireAdmin`.
    - Toàn bộ view FE thuộc `components/admin/` bắt buộc kiểm tra `currentUser?.role === 'admin'`.
  - **Thư mục Người dùng & Dùng chung (`components/user/`, `components/account/` & `routes/account.js`)**:
    - Chứa các chức năng mà người dùng thông thường được phép tự quản lý (Hồ sơ cá nhân, Đổi mật khẩu, Danh sách phiên đăng nhập của chính mình).
    - Admin có toàn quyền nên thừa kế và sử dụng các tính năng này như một người dùng cấp cao.
    - **User thường không bao giờ có thể tiếp cận hay tác động tới mã nguồn hoặc API của Admin**.

- **Quy tắc phân cấp theo Domain (Áp dụng đồng bộ cho cả Frontend & Backend):**
  - **BẮT BUỘC TẠO THƯ MỤC CHỨC NĂNG RIÊNG CHO MỌI NÚT Ở HEADER HOẶC SIDEBAR**:
    - Bất kỳ chức năng, nghiệp vụ nào đã được viết riêng thành một nút bấm/tab ở Header (ví dụ: Modal Đăng nhập/Đăng ký `components/auth/`, Hồ sơ/Tài khoản `components/account/`) hoặc ở Sidebar (ví dụ: Trang chủ `components/home/`, Quản lý tài khoản `components/account/`, Quản lý người dùng `components/admin/users/`, Quản lý cụm đám mây `components/admin/cloud-vault/`) **BẮT BUỘC PHẢI CÓ THƯ MỤC CHỨC NĂNG RIÊNG BIỆT**.
    - **Cấm viết trực tiếp logic hay JSX của chức năng vào `App.jsx`**.
  1. **Tầng 1 (Chức năng lớn / Top-Level Features)**: Mọi chức năng tương ứng với Tab chính, Sidebar item, hoặc cụm nghiệp vụ lớn **bắt buộc** có một thư mục riêng biệt:
     - Phân hệ Admin: `frontend/src/components/admin/users/`, `frontend/src/components/admin/cloud-vault/`, `backend/src/routes/admin/users/`, `backend/src/routes/admin/cloud-vault/`
     - Phân hệ User & Chung: `frontend/src/components/auth/`, `frontend/src/components/account/`, `backend/src/routes/auth.js`, `backend/src/routes/account.js`
  2. **Tầng 2 (Chức năng con / Sub-Features có độ phức tạp cao)**: Nếu trong một chức năng lớn có các khối chức năng con độc lập hoặc modal quản lý chuyên sâu:
     - Tiếp tục phân cấp xuống thư mục con (ví dụ: `frontend/src/components/account/profile/`, `frontend/src/components/account/security/`, `frontend/src/components/account/sessions/`, `frontend/src/components/admin/users/sessions/UserSessionsModal.jsx`, `frontend/src/components/admin/cloud-vault/cards/CloudResourceCard.jsx`, `frontend/src/components/admin/cloud-vault/modals/AddCloudResourceModal.jsx`).
  3. **File trung tâm (`App.jsx`, `server.js`)**: Chỉ đóng vai trò Router, Context Provider và Layout Wrapper; cấm viết các hàm xử lý dữ liệu phức tạp hoặc JSX chi tiết của từng màn hình tại đây.

## 2. Định tuyến Zero Reload & Fallback Routing (URL Routing & Deep Linking)
- **Tuyệt đối không sử dụng Hash routing (`#`)**:
  - Toàn bộ các module chức năng (Admin, User, Cài đặt, Cụm đám mây, v.v.) **BẮT BUỘC hỗ trợ truy cập trực tiếp qua đường dẫn sạch (URL Deep Linking)**, ví dụ:
    - `/` -> Trang chủ
    - `/admin/users` -> Quản lý người dùng
    - `/admin/cloud-vault` -> Quản lý Cụm đám mây đa tầng
    - `/admin/assets` -> Quản lý Poster & Hugging Face Bucket
    - `/account/profile`, `/account/security`, `/account/sessions` -> Cài đặt tài khoản
- **Cơ chế đồng bộ 2 chiều (Bidirectional URL Synchronization)**:
  1. **Khi tải trang / F5 (Initial Route)**: Hàm `getInitialRoutingFromURL()` đọc `window.location.pathname` và `searchParams` để khởi tạo đúng `currentView`, `subTab`, `search`, bộ lọc ngay khi vừa mở trang.
  2. **Khi chuyển đổi View / Tab trên UI**: Hàm `syncRouteToURL(view, ...)` dùng `window.history.pushState()` để cập nhật thanh địa chỉ của trình duyệt mà không reload trang.
  3. **Khi bấm nút Back / Forward trên trình duyệt**: Bắt sự kiện `popstate` để cập nhật lại state View tương ứng một cách mượt mà.
- **Backend Fallback Route (Zero 404 on F5)**:
  - Backend Express cấu hình wildcard route `app.get('*', ...)` trả về file `index.html` của SPA cho mọi request không phải `/api/...` để người dùng F5 hoặc chia sẻ link không bị lỗi 404.

## 3. Cô lập lỗi (Fault Isolation) & Global Scope
- Bọc `try-catch` (FE) và `try-except` (BE) cấp hàm. Lỗi của chức năng phụ không được làm gián đoạn chức năng chính.
- Tránh trùng tên hàm ở Global Scope.

