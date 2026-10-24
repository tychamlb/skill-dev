# Skill thiết kế giao diện Frontend cho FlixGate

## 1. Nút ưu tiên bo tròn
- Các nút hành động chính (Sign In, Sign Up, CTA Watch Now) và nút icon tròn (Theme toggle, Language Flag button) sử dụng bo tròn hoàn toàn `border-radius: var(--radius-full)`.

## 2. Nút chọn ngôn ngữ & Thuật toán Cờ Unicode Tự Động
- **Hiển thị trên nút chọn:** Chỉ hiển thị duy nhất icon lá cờ Emoji Unicode (ví dụ: `🇻🇳`, `🇬🇧`, `🇯🇵`).
- **Thuật toán chuyển đổi động:**
  - Nhận mã ISO-2 (quốc gia: `VN`, `GB`, `JP`, `CN`, `KR`... hoặc mã viết tắt ngôn ngữ: `VI`, `EN`, `JA`, `ZH`, `KO`).
  - Ánh xạ sang mã quốc gia và tính theo công thức độ bù Unicode: `127397 + ASCII`.
  - Không hardcode ảnh PNG/SVG tĩnh.
- **Ngoại lệ ngôn ngữ đặc thù:** Ngôn ngữ không thuộc ISO-2 quốc gia (ví dụ: Tiếng H'Mông `HMN`) được gán icon trực tiếp (`🌄`).
- **Khi click mở dropdown:** Hiển thị danh sách gồm: `[Lá cờ Emoji]` + `[Tên quốc gia/dân tộc]` (Ví dụ: `🇻🇳 Việt Nam`, `🇬🇧 United Kingdom / Global`, `🇯🇵 日本 (Japan)`...).

## 3. Modal & Popover
- Modal sử dụng glassmorphism với backdrop blur mờ ảo.
- Cho phép đóng bằng nút ✕ hoặc click ra vùng overlay bên ngoài (Light dismiss).
- Chi tiết kỹ thuật Native Popover API: [.agents/skills/FE/popover_modal_skill/SKILL.md](file:///d:/ICDingHoc/Project-2026/FlixGate/.agents/skills/FE/popover_modal_skill/SKILL.md)

## 4. Hiệu ứng & Animation
### Flow
- Button, link, icon có micro-animation hover và transform nhẹ nhàng.
- Chuyển đổi ngôn ngữ mượt mà không reload trang.
- Tự động fallback sang Tiếng Anh (`en`) nếu một cụm từ chưa có trong từ điển ngôn ngữ được chọn.


## 5. Sidebar trái (Scrollbar + Nút thu/phóng + Nút Hamburger Menu) 
- Chi tiết kỹ thuật & thiết kế chuẩn: [.agents/skills/FE/sidebar_controls_skill/SKILL.md](file:///d:/ICDingHoc/Project-2026/FlixGate/.agents/skills/FE/sidebar_controls_skill/SKILL.md)

## 6. Thanh tìm kiếm đa thiết bị (Breakpoint 520px & Mobile Search Full Header)
- Chi tiết kỹ thuật & thiết kế chuẩn: [.agents/skills/FE/search_bar_skill/SKILL.md](file:///d:/ICDingHoc/Project-2026/FlixGate/.agents/skills/FE/search_bar_skill/SKILL.md)

## 7. Modal Popover Native (HTML5 Popover API)
- Chi tiết kỹ thuật & thiết kế chuẩn: [.agents/skills/FE/popover_modal_skill/SKILL.md](file:///d:/ICDingHoc/Project-2026/FlixGate/.agents/skills/FE/popover_modal_skill/SKILL.md)

## 8. Thanh cuộn siêu mảnh 5px (Ultra-Thin 5px Scrollbar Song Song Dọc & Ngang)
- Chi tiết kỹ thuật & thiết kế chuẩn: [.agents/skills/FE/scrollbar_ultrathin_skill/SKILL.md](file:///d:/ICDingHoc/Project-2026/FlixGate/.agents/skills/FE/scrollbar_ultrathin_skill/SKILL.md)

## 9. Bộ lọc thời gian phân đoạn: Tất cả - Năm - Tháng - Ngày (Segmented Time Filter & Dynamic Pickers)
- Chi tiết kỹ thuật & thiết kế chuẩn: [.agents/skills/FE/time_filter_segmented_skill/SKILL.md](file:///d:/ICDingHoc/Project-2026/FlixGate/.agents/skills/FE/time_filter_segmented_skill/SKILL.md)

## 10. Thanh điều khiển co giãn & dính cuộn (.main_controls / Sticky Elastic Control Bar)
- Chi tiết kỹ thuật & thiết kế chuẩn: [.agents/skills/FE/sticky_elastic_controls_skill/SKILL.md](file:///d:/ICDingHoc/Project-2026/FlixGate/.agents/skills/FE/sticky_elastic_controls_skill/SKILL.md)

## 11. Bảng quản lý dịch vụ: Bộ lọc Portal 4 hướng, Sắp xếp chu kỳ & Sửa nhanh tại chỗ (Service Table Manager)
- Chi tiết kỹ thuật & thiết kế chuẩn: [.agents/skills/FE/service_table_manager_skill/SKILL.md](file:///d:/ICDingHoc/Project-2026/FlixGate/.agents/skills/FE/service_table_manager_skill/SKILL.md)

## 12. Nút bấm công thái học & Icon Controls (Ergonomic Buttons & Controls)
- Chi tiết kỹ thuật & thiết kế chuẩn: [.agents/skills/FE/ergonomic_button_controls_skill/SKILL.md](file:///d:/ICDingHoc/Project-2026/FlixGate/.agents/skills/FE/ergonomic_button_controls_skill/SKILL.md)
