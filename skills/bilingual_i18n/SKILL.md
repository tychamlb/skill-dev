# QUY CHUẨN ĐA NGÔN NGỮ THÔNG MINH (MULTI-LANGUAGE I18N WORKFLOW)

## 1. NGUYÊN TẮC BẮT BUỘC (ZERO TECH DEBT)
- **Tự động áp dụng:** Khi nhận bất kỳ yêu cầu nào liên quan đến phát triển tính năng mới, chỉnh sửa giao diện, làm modal, tạo bảng, thêm nút hoặc cải tiến Front-End (FE), **BẮT BUỘC áp dụng ngay kỹ năng này mà không cần người dùng phải nhắc nhở**.
- **Tuyệt đối không hardcode text đơn lẻ:** Không viết cứng ngôn ngữ nào với ý định "sau này hoàn thiện lại".
- **Cặp ngôn ngữ chuẩn luôn song song:**
  - **Tiếng Việt (`vi`)**
  - **Tiếng Anh (`en`)** - *Ngôn ngữ chuẩn fallback mặc định toàn cầu khi thiết bị người dùng không khớp bất kỳ ngôn ngữ nào khác*
  - Hỗ trợ mở rộng: **Tiếng H'Mông (`hmn` - Ưu tiên Top #1)**, **Tiếng Lào (`lo`)**, **Tiếng Thái Lan (`th`)**, **Tiếng Malaysia (`ms`)**, **Tiếng Trung (`zh`)**, **Tiếng Nhật (`ja`)**, **Tiếng Hàn (`ko`)**, **Tiếng Nga (`ru`)**, **Tiếng Pháp (`fr`)**, **Tiếng Tây Ban Nha (`es`)**.

## 1.1. NGUYÊN TẮC CÔ LẬP NGÔN NGỮ THUẦN KHIẾT (STRICT PURE LANGUAGE ISOLATION - CHỐNG TRỘN LẪN SONG NGỮ)
- **Tuyệt đối không pha trộn nhiều ngôn ngữ trong cùng một chuỗi hiển thị:**
  - **CẤM:** Viết chuỗi hiển thị hoặc tooltip theo dạng song ngữ kèm ngoặc như: `"Lọc theo vai trò (Filter Role)"`, `"Trạng thái (Status)"`, `"Quản trị viên (Admin)"` hoặc `"Xác nhận (Confirm)"`.
  - **ĐÚNG:** Mỗi ngôn ngữ chỉ hiển thị duy nhất nội dung của chính ngôn ngữ đó:
    - Khi đang chọn `vi`: Chỉ hiển thị `"Lọc theo vai trò"`.
    - Khi đang chọn `en`: Chỉ hiển thị `"Filter by role"`.
    - Khi đang chọn `zh`: Chỉ hiển thị `"按角色筛选"`.
- **Nguyên tắc "Ngôn ngữ nào ra ngôn ngữ đó":**
  - Mọi chuỗi hiển thị trên giao diện (nhãn nút, tiêu đề, placeholder, tooltip `title`, thông báo toast/alert) bắt buộc phải đi qua hàm dịch `t('key')` độc lập.
  - Nếu một ngôn ngữ nào đó (như `zh`, `ja`, `ko`, `hmn`) chưa có khóa dịch hoặc khóa dịch bị rỗng (`null`/`undefined`/`""`): **Hệ thống tự động lấy bản dịch Tiếng Anh (`en`) làm fallback duy nhất**, tuyệt đối không được ghép/nối chuỗi Tiếng Việt vào chung.

## 2. CƠ CHẾ FALLBACK TUYỆT ĐỐI SANG TIẾNG ANH (EN)
- Khi người dùng chọn bất kỳ ngôn ngữ nào (kể cả ZH, JA, KO, HMN) mà một từ khóa/chuỗi văn bản chưa có bản dịch hoặc khó dịch:
  - **BẮT BUỘC hiển thị bản dịch Tiếng Anh (`en`) thay thế**.
  - Tuyệt đối không để lộ văn bản Tiếng Việt sang màn hình người dùng quốc tế.
  - Tuyệt đối không tự ý nối chuỗi Tiếng Việt vào cạnh Tiếng Anh để "giải thích thêm".


## 3. BẢO MẬT THÔNG TIN HỆ THỐNG TRÊN GIAO DIỆN
- Nội dung văn bản hiển thị trên giao diện phải hướng tới trải nghiệm người xem (phim ảnh, rạp chiếu trực tuyến, tốc độ phát mượt mà).
- **Tuyệt đối không công khai chi tiết kiến trúc hạ tầng nội bộ** (như chi tiết token JWT, Bcrypt, cấu trúc database, logic cấp quyền admin...) ra màn hình người dùng.

## 4. CƠ CHẾ TỰ ĐỘNG XÁC ĐỊNH NGÔN NGỮ (SMART LOCALE DETECTION)
Thứ tự ưu tiên nhận diện:
1. **Lựa chọn đã lưu:** Kiểm tra `localStorage.getItem('flixgate_lang')` hoặc tùy chọn cá nhân trong tài khoản DB.
2. **Xác định qua IP (GeoIP):** Tra cứu mã quốc gia từ IP người dùng (VN -> `vi`, US -> `en`, GB -> `en-gb`, JP -> `ja`, KR -> `ko`, CN/TW/HK -> `zh`).
3. **Ngôn ngữ thiết bị:** Tra cứu `navigator.language` / `navigator.languages`.
4. **Fallback mặc định:** Nếu ngôn ngữ không nằm trong danh sách hỗ trợ, **mặc định hiển thị Tiếng Anh (`en`)**.

## 5. CƠ CHẾ DỊCH NỘI DUNG TỰ ĐỘNG (AI & FREE TRANSLATION ENGINE)
- **Nhập 1 chuyển tất cả:** Người dùng hoặc Admin có thể nhập tiêu đề/mô tả bằng 1 ngôn ngữ bất kỳ.
- **Hỗ trợ Dịch tự động:** Tích hợp nút **"Dịch tự động"** qua API dịch miễn phí (Google Translate Free API, MyMemory, Free LLM AI).
- **Chỉnh sửa thủ công (Manual Override):** Cung cấp các tab/ô nhập tương ứng cho từng ngôn ngữ (`vi`, `en`, `zh`, `ja`, `ko`, `hmn`) để người dùng có thể tự gõ hoặc chỉnh sửa lại cho sát nghĩa nhất.

## 6. PHẠM VI ÁP DỤNG TRÊN GIAO DIỆN
1. **Thành phần văn bản tĩnh & Động:** Tiêu đề trang, modal, tên thẻ, cột bảng, form label, placeholder, tooltip.
2. **Thành phần tương tác & Điều khiển:** Nút bấm, menu, tabs, badges, toasts, confirm dialogs.
3. **Player & Media:** Trình điều khiển video, tên phụ đề, nhãn chất lượng stream.

## 7. TỪ ĐIỂN CHUẨN THUẬT NGỮ ĐỊA PHƯƠNG HÓA TIẾNG H'MÔNG (HMN GLOSSARY)
Nhằm tránh dịch theo kiểu mượn từ nửa vời ("Load Dua", "Admin", v.v.), bắt buộc áp dụng từ vựng chuẩn bản địa H'Mông:
- **Tải lại / Làm mới (Reload / Refresh)**: `"Ntxiav tshiab"` (không dùng `"Load Dua"`).
- **Quản trị viên (Administrator)**: `"Tus Thawj Admin"` hoặc `"Tus Tswj Hwm"`.
- **Người dùng / Thành viên (Member / User)**: `"Tswv Cuab"` / `"Neeg Siv"`.
- **Đang hoạt động (Active)**: `"Ua Haujlwm"`.
- **Đã bị khóa (Locked)**: `"Xauv Lawm"`.
- **Lọc dữ liệu (Filter)**: `"Lim"`.
- **Sắp xếp (Sort)**: `"Txheeb"`.
- **Phiên hoạt động (Sessions)**: `"Kev Nkag Session"`.
- **Vị trí ưu tiên trên giao diện**: **Tiếng H'Mông (`hmn`)** luôn được xếp ở vị trí đầu tiên (`top #1`) trong danh sách chọn ngôn ngữ (`LANG_CONFIGS`), theo sau là Tiếng Việt (`vi`), Tiếng Anh (`en`) và các ngôn ngữ quốc tế khác.
