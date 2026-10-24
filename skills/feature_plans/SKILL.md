# QUY TRÌNH QUẢN LÝ KẾ HOẠCH THEO CHỨC NĂNG (FEATURE PLANS WORKFLOW)

## 1. NGUYÊN TẮC KÍCH HOẠT & QUY TẮC ĐÁNH SỐ THỨ TỰ (PLAN NUMBERING & CASCADE SHIFT)
- Khi người dùng yêu cầu phát triển một tính năng mới, hoặc khi sử dụng chức năng `/plan`, hoặc khi bắt đầu một Sprint mới:
  - **BẮT BUỘC tuân thủ quy trình lập kế hoạch 2 cấp độ**:
    1. **Cấp 1 - Kế hoạch tổng quát (`PROJECT_PLAN.md`)**: Cập nhật trạng thái module và danh sách chức năng trong lộ trình toàn dự án.
    2. **Cấp 2 - Kế hoạch chức năng chi tiết (`plans/`)**: Bắt buộc tạo hoặc cập nhật file plan riêng cho module đó (ví dụ: `plans/01_auth_rbac.md`, `plans/02_user_management.md`...).
- **QUY TẮC ĐẨY LÙI KẾ HOẠCH (CASCADE SHIFT FOR PENDING PLANS)**:
  - **Tuyệt đối không để 2 plan trùng số thứ tự** (ví dụ: không được tồn tại đồng thời `03_...` và `03_...`).
  - Khi một kế hoạch đã lập sẵn (chưa triển khai/chưa làm) mà người dùng yêu cầu lập kế hoạch mới để ưu tiên làm trước:
    - **Kế hoạch mới chuẩn bị làm** sẽ nhận đúng số thứ tự hiện tại (ví dụ: `03_multi_cloud_vault.md`).
    - **Toàn bộ kế hoạch cũ chưa làm** đứng phía sau phải **tự động đẩy lùi số thứ tự ra sau** (ví dụ: đổi `03_poster_hf_buckets.md` thành `04_poster_hf_buckets.md`), đồng thời cập nhật lại số Sprint và số Module tương ứng trong `PROJECT_PLAN.md` và tiêu đề file plan.


## 2. NỘI DUNG BẮT BUỘC TRONG FILE PLAN CHI TIẾT (`plans/XX_feature.md`)
Mỗi file plan chức năng trong thư mục `plans/` phải có đầy đủ các mục:
1. **Mục tiêu & Yêu cầu cốt lõi**: Nêu rõ bài toán cần giải quyết, vai trò của chức năng.
2. **DANH MỤC SKILL HỆ THỐNG BẮT BUỘC ÁP DỤNG (MANDATORY APPLIED SKILLS)**:
   - Liệt kê cụ thể từng file SKILL liên quan từ `.agents/skills/` (Ví dụ: `FE/popover_modal_skill`, `FE/scrollbar_ultrathin_skill`, `FE/sidebar_controls_skill`, `FE/search_bar_skill`, `bilingual_i18n`, `BE/vault_encryption_skill`...).
   - **Cam kết không tự ý làm kém hơn hoặc bỏ qua chuẩn kỹ thuật trong skill**: Khi lập kế hoạch và khi code, giao diện và logic phải kế thừa 100% chuẩn kỹ thuật tốt nhất đã được quy định trong các file SKILL (Modal Popover Native chuẩn top-layer, thanh cuộn 5px, nút bo tròn, hiệu ứng hover xoay đỏ 90 độ, đa ngôn ngữ 6 thứ tiếng...).
3. **Cơ sở dữ liệu (Database Schemas & Models)**: Mongoose schemas, indexes, cấu trúc bảng liên quan.
4. **Đặc tả API & Dịch vụ**: Endpoints (Method, Path, Request Body, Response mẫu, Validation, Error codes).
5. **Đặc tả Kỹ thuật Chuyên sâu (Technical Architecture)**:
   - Các thuật toán tính toán, quy trình xử lý dữ liệu.
   - Cơ chế cache, cơ chế bảo mật phiên hoặc token.
   - Thư viện hoặc API bên thứ ba sử dụng.
6. **Giao diện Người dùng (UI/UX & Components)**: Form, layout, modal, responsive behavior, hiệu ứng bo tròn.
7. **Bảng Từ Điển Đa Ngôn Ngữ**: Khai báo đầy đủ các key i18n cho cả 6 thứ tiếng (`vi`, `en`, `zh`, `ja`, `ko`, `hmn`).
8. **Tiêu Chí Kiểm Thử & Nghiệm Thu (Acceptance Criteria)**: Các kịch bản kiểm thử cụ thể để xác nhận chức năng đã hoàn thành không lỗi.

## 3. QUY TRÌNH KHI HOÀN THÀNH CHỨC NĂNG
1. Chạy kiểm thử tự động xác nhận đạt 100% tiêu chí nghiệm thu.
2. Cập nhật trạng thái sang `🟢 ĐÃ HOÀN THÀNH` trong file plan chi tiết và `PROJECT_PLAN.md`.
3. Khởi động lại dịch vụ để áp dụng mã nguồn mới nhất (Quy tắc 6).
4. Thực hiện `git commit` lưu lại mốc hoàn thành trước khi chuyển sang chức năng mới (Quy tắc 9).
