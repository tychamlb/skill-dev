---
name: ergonomic_button_controls_skill
description: Quy chuẩn thiết kế nút bấm công thái học (Ergonomic Buttons & Controls), quy tắc nút icon đơn thuần (Icon-Only Buttons), prefix icon cho bộ lọc và cơ chế tooltip mô tả đa ngôn ngữ.
---

# KỸ NĂNG THIẾT KẾ NÚT BẤM CÔNG THÁI HỌC & ICON CONTROLS (ERGONOMIC BUTTON CONTROLS SKILL)

## 1. NGUYÊN LÝ THIẾT KẾ CỐT LÕI (CORE PRINCIPLES)

1. **Nhận Diện Trực Quan Qua Icon (Visual Self-Explanation)**:
   - Các hành động phổ biến và có tính quy ước toàn cầu (như Làm mới / Tải lại `↻`, Tìm kiếm `🔍`, Xóa `✕`, Cài đặt `⚙️`, Đóng `✕`) tự bản thân biểu tượng đã thể hiện trọn vẹn ngữ nghĩa của hành động.
   - **Quy tắc Icon-Only**: Đối với nút làm mới (`btn-reload-action`) hay các nút công cụ phụ trợ, **chỉ giữ lại biểu tượng icon**, loại bỏ văn bản text đi kèm để thanh công cụ không bị chật chội, quá tải thị giác hoặc gây vỡ dòng trên màn hình hẹp.

2. **Prefix Icon Nhận Diện Cho Nhóm Bộ Lọc (Select Dropdowns Prefix Icons)**:
   - Mọi ô chọn lọc (`select`) trong thanh điều khiển `.main_controls` hoặc nhóm công cụ cần được trang bị **Prefix Icon** tương ứng ở bên trái:
     - Nhóm vai trò / thành viên: Icon nhóm người dùng (`👥` hoặc SVG users).
     - Nhóm trạng thái / hoạt động: Icon mục tiêu / radar (`🎯` hoặc SVG target circle).
   - Prefix icon giúp người dùng nhận diện ngay tính chất bộ lọc trong 0.2 giây mà không cần phải đọc toàn bộ chữ trong option.

3. **Cơ Chế Tooltip Mô Tả Đa Ngôn Ngữ Bắt Buộc (Rich Accessible Tooltips)**:
   - Khi nút chỉ hiển thị icon (Icon-Only), **bắt buộc phải có thuộc tính `title` và `aria-label`** được truyền qua hàm dịch `t('key')`.
   - Tooltip phải giải thích đầy đủ, rõ ràng hành động (ví dụ: `title={t('admin_btn_reload_title')}`, `aria-label={t('admin_btn_reload')}`).
   - Tuân thủ 100% nguyên tắc cô lập ngôn ngữ thuần khiết: Mỗi ngôn ngữ hiển thị độc lập, không trộn lẫn song ngữ.

4. **Nút Thao Tác Thường Dùng Trên Header (Header Contextual Action Icons)**:
   - Khi phát triển một chức năng/view mới có nút hành động hay dùng và đứng riêng lẻ (ví dụ: "Tạo / Tải mới", "Thêm nhanh"...):
     - **BẮT BUỘC** bổ sung thêm một nút icon tròn chuẩn `.btn-icon-action` (hoặc `.btn-header-*`) lên thanh Header, đặt cạnh các nút tiện ích hệ thống (ngôn ngữ, theme sáng/tối).
     - **Quy tắc hiển thị ngữ cảnh**: Nút này **CHỈ HIỂN THỊ KHI ĐANG Ở ĐÚNG TRANG/VIEW ĐÓ** (`currentView === '...'`) và tự động ẩn hoàn toàn khi chuyển sang trang khác.
     - Sử dụng đúng icon đồng bộ với nút thao tác chính bên dưới, trang bị `title` và `aria-label` đa ngôn ngữ qua `t('key')`.

---

## 2. QUY CHUẨN KÍCH THƯỚC ĐA THIẾT BỊ (RESPONSIVE BUTTON TIERS)

Không sử dụng kích thước cố định cứng cho toàn bộ màn hình. Bắt buộc phân bậc linh hoạt theo 3 vùng độ phân giải:

| Vùng Màn Hình (Breakpoints) | Chiều Cao Nút / Control | Kích Thước Nút Icon-Only | Icon Size | Font Chữ |
| :--- | :--- | :--- | :--- | :--- |
| **Màn Hình Lớn (`> 1500px`)** | `34px - 36px` | `34px × 34px` | `15px - 16px` | `13px` |
| **Màn Chuẩn (`520px - 1500px`)** | `30px - 32px` | `32px × 32px` | `13px - 14px` | `12px` |
| **Mobile Nhỏ (`< 520px`)** | `24px - 26px` | `24px × 24px` | `11px - 12px` | `11px` |

---

## 3. HIỆU ỨNG HÀO QUANG PHÁT SÁNG & HOVER ĐỒNG BỘ (AURA GLOW & HOVER SYNERGY)

Mọi nút tác vụ dạng icon tròn hoặc tab điều khiển (Theme Sáng/Tối, Nút Ngôn ngữ, Nút Làm mới, Nút Header Contextual Action, Nút Đăng xuất Sidebar `.sidebar-logout-btn`, Tab quai bấm mép Sidebar `.btn-sidebar-edge-toggle`, Nút icon thu gọn `.nav-item a`) bắt buộc mang ngôn ngữ thị giác đồng nhất:
- **Trạng thái thường (Default with Aura)**:
  - Nền `var(--bg-tertiary)`, viền `var(--border-color)`, màu chữ/icon `var(--text-primary)` (hoặc `#3b82f6` cho nút hành động).
  - **Vầng hào quang phát sáng (Drop-shadow Aura)**: `filter: drop-shadow(0 0 6px rgba(255, 68, 0, 0.4))` tạo ánh hào quang đỏ-cam huyền bí, cao cấp bao quanh nút tròn hoặc tab quai bấm.
  - Icon bên trong gán `pointer-events: none` để chống cản trở click.
- **Hiệu ứng khi rê chuột (Hover with Gold Glow)**:
  - Nổi nhẹ `transform: translateY(-1px) scale(1.08)` (phóng to nhẹ 8%).
  - Nền đỏ kính mờ `rgba(239, 68, 68, 0.15)`, viền đỏ rực `#ef4444`, màu chữ/icon `#ef4444`.
  - **Chuyển hào quang sang vàng tươi (Gold Glow)**: `filter: drop-shadow(0 0 12px rgba(234, 179, 8, 0.85))` kết hợp bóng đổ `box-shadow: 0 4px 14px rgba(239, 68, 68, 0.3)`.
- **Hiệu ứng bấm giữ (Active)**: Co nhẹ `transform: scale(0.92)` cho cảm giác phản hồi xúc giác êm ái.

---

## 4. CẤU TRÚC CODE MẪU CHUẨN (REACT JSX + CSS)

### 4.1. React JSX

```jsx
{/* Nhóm Bộ Lọc Có Icon Nhận Diện Cả Bên Ngoài Và Trong Từng Option */}
<div className="control-select-wrap">
  <svg className="control-select-prefix-icon" width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2.2">
    <path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"></path>
    <circle cx="9" cy="7" r="4"></circle>
  </svg>
  <select
    className="control-custom-select"
    value={roleFilter}
    onChange={(e) => setRoleFilter(e.target.value)}
    title={t('admin_filter_by_role')}
  >
    <option value="all">👥 {t('admin_filter_role_all')}</option>
    <option value="admin">🛡️ {t('role_admin')}</option>
    <option value="user">👤 {t('role_user')}</option>
  </select>
</div>

{/* Nút Làm Mới Icon-Only Với Tooltip Chuẩn */}
<button
  type="button"
  className="btn-control-secondary btn-reload-action"
  onClick={onReload}
  disabled={loading}
  title={t('admin_btn_reload_title')}
  aria-label={t('admin_btn_reload')}
>
  <span className={`reload-icon ${loading ? 'spinning' : ''}`}>↻</span>
</button>
```

### 4.2. CSS Chuẩn

```css
/* Prefix Icon Định Vị Tuyệt Đối Trong Wrap */
.control-select-wrap {
  flex: 1 1 125px;
  min-width: 110px;
  position: relative;
  display: flex;
  align-items: center;
}

.control-select-prefix-icon {
  position: absolute;
  left: 10px;
  top: 50%;
  transform: translateY(-50%);
  color: var(--text-muted);
  pointer-events: none;
  z-index: 1;
}

.control-custom-select {
  width: 100%;
  height: 32px;
  padding: 4px 26px 4px 28px;
  border-radius: var(--radius-full);
  font-size: 12px;
}

/* Nút Icon-Only Bo Tròn Đồng Bộ Hiệu Ứng Hover Nút Theme Sáng/Tối */
.btn-reload-action {
  flex: 0 0 32px;
  width: 32px;
  height: 32px;
  padding: 0;
  border-radius: 50%;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.2s cubic-bezier(0.16, 1, 0.3, 1);
}

.btn-reload-action:hover {
  background: rgba(239, 68, 68, 0.15) !important;
  border-color: #ef4444 !important;
  color: #ef4444 !important;
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(239, 68, 68, 0.25);
}

.btn-reload-action:active {
  transform: scale(0.92);
}

/* Responsive Phân Bậc Đa Thiết Bị */
@media (min-width: 1501px) {
  .btn-reload-action {
    flex: 0 0 34px;
    width: 34px;
    height: 34px;
  }
  .control-custom-select {
    height: 34px;
    font-size: 13px;
  }
}

@media (max-width: 520px) {
  .btn-reload-action {
    flex: 0 0 24px;
    width: 24px;
    height: 24px;
  }
  .control-custom-select {
    height: 26px;
    font-size: 11px;
    padding: 2px 20px 2px 24px;
  }
}
```

