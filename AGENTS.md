# NGUYÊN TẮC ƯU TIÊN
- MẶC ĐỊNH luôn sử dụng quy tắc trong file này (`.agents/AGENTS.md`).
- LUÔN thực hiện Quy tắc 1 trước tiên.

# CHỈ THỊ HÀNH VI VÀ NGUYÊN TẮC PHÁT TRIỂN (SYSTEM RULES)

## 1. QUY TRÌNH TIẾP NHẬN & PHÂN TÍCH YÊU CẦU
- **Phân tích & Làm rõ Yêu cầu:** Ngay khi nhận được prompt, bắt buộc phải:
  1. Xác định rõ mục tiêu và yêu cầu cốt lõi của người dùng.
  2. Làm rõ chức năng/vấn đề cụ thể cần giải quyết.
  3. **BẮT BUỘC RÀ SOÁT QUY TẮC & SKILL LIÊN QUAN TRƯỚC KHI CODE:** Chủ động tra cứu danh sách skill trong `.agents/skills/` tương ứng với tác vụ (ví dụ: SPA frontend tra `spa_architecture`, `FE/`; Backend tra `BE/`; Kế hoạch tra `feature_plans`). **Tuyệt đối không bỏ qua quy tắc và không tự ý code theo thói quen/cảm tính**.
  4. **CHỐNG THIÊN KIẾN DỒN CODE VÀO 1 FILE (ANTI-MONOLITH BIAS) & PHÂN CẤP THƯ MỤC CHỨC NĂNG ĐA TẦNG (MULTI-TIER FEATURE FOLDERS):**
     - **Nguyên lý Phân tách Ranh giới Đặc quyền (Admin vs User Separation of Concerns):**
       - **Đặc quyền Admin (Admin Scope)**: Thư mục `components/admin/` (FE) và `routes/admin/` (BE) **CHỈ DÀNH RIÊNG CHO ADMIN**. User thường tuyệt đối KHÔNG ĐƯỢC phép truy cập, import, dùng chung tài nguyên hay gọi bất kỳ API nào bên trong thư mục này. **TUYỆT ĐỐI CẤM DÙNG API ADMIN CHO USER** dưới mọi hình thức. Bất kỳ chức năng nào chỉ Admin mới có quyền thao tác (quản trị toàn bộ danh sách users, can thiệp phiên người khác, quản lý tài nguyên cụm đám mây, phân quyền RBAC...) **bắt buộc phải nằm cô lập hoàn toàn trong thư mục `admin/`** kèm middleware `requireAdmin` và xác thực role trên FE.
       - **Chức năng Người dùng & Dùng chung (User & Common Scope)**: Các chức năng mà người dùng cá nhân sử dụng (xem/đổi hồ sơ cá nhân, đổi mật khẩu, xem/thu hồi phiên đăng nhập của CHÍNH MÌNH) phải thuộc phạm vi người dùng (`components/user/` hoặc `components/account/` trên FE; `routes/account.js` trên BE). Admin có toàn quyền (thừa kế) nên đương nhiên sử dụng được các chức năng của user, nhưng **ngược lại User thường không bao giờ được chạm vào code hay gọi API của Admin**. Nếu cùng một nhóm nghiệp vụ (như xem phiên đăng nhập, đổi thông tin) nhưng khác cấp độ phân quyền ➔ **BẮT BUỘC PHÂN TÁCH 2 API ĐỘC LẬP**: API User (`/api/account/...`) chỉ xử lý dữ liệu của chính mình qua token phiên, còn API Admin (`/api/admin/...`) xử lý diện rộng và được bảo vệ nghiêm ngặt bằng `requireAdmin`.
       - **Quy tắc phân cấp theo Domain (Áp dụng cho cả FE & BE):**
         - **BẮT BUỘC TẠO THƯ MỤC CHỨC NĂNG RIÊNG CHO MỌI NÚT Ở HEADER HOẶC SIDEBAR**:
           - Bất kỳ chức năng, nghiệp vụ nào đã được viết riêng thành một nút bấm/tab ở Header (ví dụ: Modal Đăng nhập/Đăng ký `auth/`, Cài đặt/Hồ sơ `account/`) hoặc ở Sidebar (ví dụ: Trang chủ `home/`, Quản lý tài khoản `account/`, Quản lý người dùng `admin/users/`, Quản lý cụm đám mây `admin/cloud-vault/`) **BẮT BUỘC PHẢI TẠO THƯ MỤC CHỨC NĂNG RIÊNG**.
           - **Tuyệt đối không viết dồn mã nguồn trực tiếp vào file gom nối `App.jsx`**. Toàn bộ mã JSX, form xử lý, logic sự kiện, modal của chức năng đó phải được viết bên trong thư mục chức năng tương ứng.
           - Nếu chức năng có nhiều thành phần con, sub-tabs hoặc modal phức tạp (như modal đăng nhập/đăng ký, form hồ sơ, form đổi mật khẩu, bảng phiên đăng nhập, card tài nguyên) ➔ **TIẾP TỤC TẠO THƯ MỤC CON BÊN TRONG** (ví dụ: `components/account/profile/`, `components/account/security/`, `components/account/sessions/`, `components/auth/`).
           - File trung tâm (`App.jsx`, `server.js`) **CHỈ GIỮ VAI TRÒ Router & Layout gom nối (Hub)**, kích thước tối ưu, không ôm đồm logic.
  5. **Tái sử dụng mã cho Thông báo, Hộp thoại xác nhận, Modal:** Khi nhận yêu cầu liên quan đến thông báo (alerts, toasts), thông báo xác nhận (confirm dialogs) hoặc modal, **BẮT BUỘC áp dụng quy tắc tái sử dụng mã nguồn** (tra cứu các component modal/confirm/SweetAlert2 có sẵn trong hệ thống thay vì dùng `confirm()` mặc định của trình duyệt). Nếu cần giải pháp tốt hơn, chuyên nghiệp và xịn hơn cái hiện tại thì chủ động code mới xịn hơn nhưng vẫn giữ tính đồng bộ, thẩm mỹ và hỗ trợ song ngữ.
- **Xác định phạm vi & Nguyên tắc Bất Khả Xâm Phạm (Strict Scope Inviolability):**
  - **CHỈ CHỈNH SỬA ĐÚNG PHẠM VI YÊU CẦU:** Khi người dùng yêu cầu tinh chỉnh phần này (ví dụ: thanh tìm kiếm + nhóm nút bấm), **TUYỆT ĐỐI KHÔNG ĐƯỢC CHẠM VÀO, KHÔNG ĐƯỢC XÓA BỎ, KHÔNG ĐƯỢC LÀM SAI LỆCH** các phần khác (như bảng dữ liệu, cột phiên hoạt động, modal chi tiết phiên...).
  - Trước khi sửa bất kỳ file nào, phải xác định tọa độ dòng bắt đầu và kết thúc của vùng cần can thiệp. Bất kỳ hành vi xóa hoặc ghi đè mất mã nguồn hiện có không nằm trong yêu cầu đều bị coi là **VI PHẠM NGHIÊM TRỌNG**.
- **Xác thực môi trường/đối tượng:** Trước khi thực hiện kết nối hoặc tác động lên bất kỳ máy chủ, thư mục, hoặc dịch vụ nào, phải kiểm tra sự tồn tại chính xác của đối tượng đó. Nếu đối tượng không tồn tại hoặc sai lệch tên gọi, **bắt buộc phải báo cáo và dừng lại xin ý kiến**, tuyệt đối không tự ý suy đoán, ánh xạ.


## 2. NGUYÊN TẮC TÁI SỬ DỤNG MÃ NGUỒN
- Xem hướng dẫn chi tiết tại skill `reuse_codes` (`.agents/skills/reuse_codes/SKILL.md`) khi cần tìm kiếm và tái sử dụng mã nguồn.

## 3. BẢO VỆ MÃ NGUỒN HIỆN TẠI & CHỐNG REGRESSION DO PHẠM VI CAN THIỆP
- Xem hướng dẫn chi tiết tại skill `source_protection` (`.agents/skills/source_protection/SKILL.md`).
- **Nguyên tắc "Mổ nội soi" (Surgical Precision Edit):**
  - Khi thực hiện sửa đổi nhỏ (hotfix/tinh chỉnh UI), chỉ thay thế chính xác block code liên quan.
  - Không đọc lan man sang hàng loạt file không liên quan để rồi mất ngữ cảnh (context drift).
  - Không tự tiện dọn dẹp hoặc gỡ bỏ các đoạn mã của tính năng liền kề khi không có lệnh trực tiếp từ người dùng.

## 4. QUY CHUẨN THIẾT KẾ & PHÁT TRIỂN CHỨC NĂNG MỚI (SPA)
- Xem hướng dẫn chi tiết tại skill `spa_architecture` (`.agents/skills/spa_architecture/SKILL.md`) khi thiết kế/phát triển chức năng SPA mới.
- **BẮT BUỘC IMPORT & ÁP DỤNG DANH MỤC SKILL (MANDATORY SKILL IMPORT & INTEGRATION):**
  - Trước khi viết mã cho bất kỳ chức năng hay view mới nào, **BẮT BUỘC phải import/liệt kê danh mục các SKILL liên quan** từ `.agents/skills/` và tuân thủ 100% chuẩn kỹ thuật tốt nhất đã có trong skill:
    1. **Định tuyến URL 2 chiều & Deep Linking**: Bắt buộc hỗ trợ truy cập trực tiếp qua đường dẫn sạch (`/admin/users`, `/admin/cloud-vault`, `/admin/assets`...) theo mục 2 của `spa_architecture`.
    2. **Modal / Popover**: Bắt buộc áp dụng `FE/popover_modal_skill` (Native HTML5 Popover API trên Top Layer `popover="auto"`, nút đóng `✕` đối xứng xoay 90° đỏ, khử hoàn toàn thanh cuộn dọc thừa).
    3. **Thanh cuộn**: Bắt buộc áp dụng `FE/scrollbar_ultrathin_skill` (Thanh cuộn siêu mảnh 5px cho bảng và danh sách).
    4. **Thanh tìm kiếm & Điều hướng**: Áp dụng `FE/search_bar_skill` và `FE/sidebar_controls_skill`.
    5. **Bảo mật & Mã hóa**: Áp dụng `BE/vault_encryption_skill` (AES-256-GCM qua PBKDF2).
    6. **Quản trị Phiên & Vòng đời Token Key**: Áp dụng `BE/session_token_lifecycle_skill` (Sliding Session TTL 6h trần cứng 24h, Anti-Replay 24h Ban, Localhost Deduplication gộp phiên local, Self-Logout Protection).
    7. **Định vị GeoIP & Quản trị Thiết bị**: Áp dụng `BE/geoip_session_resolution_skill` (2-Tier Cache 0ms, Egress Probe cờ thật cho localhost, Vector SVG flags).
    8. **Đa ngôn ngữ i18n**: Áp dụng `bilingual_i18n` (6 thứ tiếng, Zero hardcoded strings).
    9. **Nút Thao Tác Thường Dùng & Nút Icon Công Thái Học**: Áp dụng `FE/ergonomic_button_controls_skill` (Icon-only, Aura glow, nút contextual action trên Header chỉ hiện khi vào đúng trang/view).
    10. **Thanh Công Cụ / Bộ Lọc Ghim Cuộn & Co Giãn 100%**: Áp dụng `FE/sticky_elastic_controls_skill` (Chạm dừng cách header 0px, chỉ bị đẩy khi section dưới trồi lên, đàn hồi lấp trọn 100% không gian).
    11. **Quản Trị Cụm Đám Mây Đa Tầng**: Áp dụng `BE/multi_cloud_vault_skill` (Keep-Alive 4 phút MongoDB, Quota & Rollover Readonly, Backup & Disaster Recovery 15 ngày Google Drive).
    12. **Bảng Quản Lý Dịch Vụ & Lưới Dữ Liệu**: Áp dụng `FE/service_table_manager_skill` (Lọc Portal 4 hướng, Sắp xếp chu kỳ, Inline Quick-Edit tại chỗ).
    13. **Bộ Lọc Thời Gian Phân Đoạn**: Áp dụng `FE/time_filter_segmented_skill` (Phân đoạn Tất cả - Năm - Tháng - Ngày, Dynamic Range Picker).
    14. **Polling Chạy Nền & Khử Giật Nháy Màn Hình**: Áp dụng FE/background_polling_silent_sync_skill (Chu kỳ setInterval(..., 15000), silent background update isSilent=true, spinner chỉ hiện khi dữ liệu ban đầu rỗng, cô lập mute lỗi ngầm).
  - **Quy tắc Chống Thoái Lui (No Regression Rule):** Tuyệt đối không tự ý code sơ sài hoặc làm ra giải pháp kém hơn chuẩn đã được đóng gói trong các file SKILL.

## 5. QUY TẮC DỌN DẸP & ĐÓNG GÓI
- Xem hướng dẫn chi tiết tại skill `build_and_cleanup` (`.agents/skills/build_and_cleanup/SKILL.md`) khi cần thao tác venv, build hoặc dọn dẹp cache.

## 6. CHẠY CHƯƠNG TRÌNH & THỰC THI TỰ ĐỘNG
- **Ưu tiên:** Chạy chính xác lệnh người dùng yêu cầu.
- **Tự động:** Script khởi động (`start_flask.sh`, `run.sh`) -> `npm start` -> `node server.js` -> `npm run dev`.
- **Tự động áp dụng sau mỗi yêu cầu:** BẮT BUỘC thực hiện Quy tắc 6 (chạy lại dịch vụ/Flask server) ngay sau khi làm xong mỗi yêu cầu để áp dụng ngay phiên bản mã nguồn mới nhất.

## 7. TỐI ƯU HÓA TOKEN & THAO TÁC VPS
- Xem hướng dẫn chi tiết tại skill `token_vps_optimization` (`.agents/skills/token_vps_optimization/SKILL.md`) để áp dụng patch sửa đổi nhỏ và sử dụng script tự động hóa trên VPS.

## 8. QUY TRÌNH UNDO & REDO TỰ ĐỘNG
- Xem hướng dẫn chi tiết tại skill `undo_redo_workflow` (`.agents/skills/undo_redo_workflow/SKILL.md`) khi người dùng yêu cầu `/undo` hoặc `/redo`.

## 9. QUY TẮC COMMIT TỰ ĐỘNG VÀ QUẢN LÝ THỜI ĐIỂM COMMIT
- Xem hướng dẫn chi tiết tại skill `commit_workflow` (`.agents/skills/commit_workflow/SKILL.md`).
- **Tự động commit khi chuyển tính năng:** Đang làm chức năng 1 xong mà được yêu cầu làm chức năng 2 ➔ Hiểu là chức năng 1 đã xong không lỗi, **BẮT BUỘC `git commit` chức năng 1 rồi mới làm chức năng 2**.
- **Không tự ý commit khi đang sửa lỗi:** Đang sửa/fix chức năng thì KHÔNG được tự ý commit khi không có yêu cầu commit từ người dùng.

## 10. QUY TẮC ĐA NGÔN NGỮ BẮT BUỘC (MULTI-LANGUAGE I18N WORKFLOW)
- Xem hướng dẫn chi tiết tại skill `bilingual_i18n` (`.agents/skills/bilingual_i18n/SKILL.md`).
- **Nguyên tắc cốt lõi:** Bắt buộc áp dụng ngay khi code (Zero Tech Debt), không hardcode bất kỳ ngôn ngữ nào.
- **Nguyên tắc "Ngôn ngữ nào ra ngôn ngữ đó":** Tuyệt đối KHÔNG ghép chuỗi trộn lẫn song ngữ (ví dụ: cấm `Tiếng Việt (English)`, `Lọc theo vai trò (Filter Role)`). Mỗi ngôn ngữ hiển thị độc lập 100% qua `t('key')`. Nếu thiếu bản dịch, **BẮT BUỘC fallback sang Tiếng Anh (`en`)**, không để lộ Tiếng Việt sang màn hình người dùng quốc tế.
- Luôn có song song cặp ngôn ngữ chuẩn Tiếng Việt (`vi`) và Tiếng Anh (`en` - Mặc định fallback toàn diện khi từ khóa thiếu hoặc khó dịch), hỗ trợ mở rộng `zh`, `ja`, `ko`, `hmn`.
- Bảo mật thông tin hệ thống: Không công khai chi tiết kiến trúc hạ tầng nội bộ ra giao diện người xem.

## 11. QUY TRÌNH QUẢN LÝ KẾ HOẠCH THEO CHỨC NĂNG (FEATURE PLANS WORKFLOW)
- Xem hướng dẫn chi tiết tại skill `feature_plans` (`.agents/skills/feature_plans/SKILL.md`).
- **Nguyên tắc kích hoạt:** Áp dụng khi người dùng có yêu cầu lập plan, hoặc sử dụng lệnh `/plan`, hoặc bắt đầu một chức năng/Sprint mới.
- **Quy trình 2 cấp độ:** Bắt buộc cập nhật kế hoạch ở lộ trình tổng quát (`PROJECT_PLAN.md`) trước, sau đó lập file kế hoạch chi tiết cho chức năng đó trong thư mục `plans/` (ví dụ: `plans/01_auth_rbac.md`, `plans/02_poster_hf_buckets.md`...) rồi mới bắt tay vào viết code.

