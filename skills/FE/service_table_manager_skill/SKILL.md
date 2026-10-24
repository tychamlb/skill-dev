---
name: service_table_manager_skill
description: "Đặc tả chuẩn mực cấu trúc bảng quản lý dịch vụ & quản trị (Service Manager Table): Card container bo góc 12px viền mờ bán trong suốt, cụm Tiêu đề chữ trắng đậm sắc nét + Badge số lượng đếm bo tròn pill mờ + Nút Phóng to/Thu gọn 4 góc khung viền [⛶] chuẩn kính mờ 26x26px, thanh Quick Tips Toolbar 3 badge, cơ chế Portal Dropdown định vị 4 hướng (Dưới > Phải > Trái > Trên) cách nút 1px cách mép 2px, cuộn nội bộ co giãn (220px ⇋ 350px), sắp xếp chu kỳ click 3 nấc kèm badge hủy và Inline Edit tại chỗ."
---

# SERVICE TABLE MANAGER SKILL (QUY CHUẨN THIẾT KẾ BẢNG DỊCH VỤ & QUẢN TRỊ)

Tài liệu này chuẩn hóa mô hình giao diện và logic tương tác cao cấp cho các bảng dữ liệu quản lý trong hệ thống React/Next.js/Vite.

---

## 1. MÔ TẢ TỔNG QUAN VÙNG CHỨA & GIAO DIỆN

### 1.1 Vùng Chứa (Card Container)
- **Bo góc & Hiệu ứng**: Khung Card bo góc `12px`, viền kính mờ bán trong suốt `rgba(255,255,255,0.08)`, đổ bóng êm `0 10px 25px -5px rgba(0,0,0,0.3)`.
- **Thanh cuộn bảng & Nút Mở Rộng 4 Góc Khung Viền**:
  - Mặc định chiều cao: `max-height: 220px` (hoặc cấu hình tùy view `220px` - `360px`).
  - Khi bấm nút icon 4 hướng (cạnh badge số lượng): phóng to lên `350px` (hoặc `520px`).
  - Hiệu ứng chuyển động mượt mà: `transition: max-height 0.25s ease-in-out`.
  - Sticky Header: Cố định hàng tiêu đề (`position: sticky`, `top: 0`, `z-index: 10`) khi cuộn bảng.

---

### 1.2 Cụm Tiêu Đề + Badge Đếm + Nút 4 Hướng (Header Title Group)
Chuẩn thiết kế pixel-perfect đối chiếu từ giao diện thực tế:
- **Tiêu đề bảng**: Chữ trắng đậm sắc nét, kích thước `15px`, `fontWeight: 800`, không kèm emoji/icon thừa (`Danh sách người dùng` hoặc `Danh sách Services`).
- **Badge số lượng đếm**:
  - Hình dạng: Bo tròn 50% hoàn hảo (`borderRadius: 50%`, `minWidth: 24px`, `height: 24px`).
  - Màu sắc: Phối màu Gradient bán trong suốt cao cấp `linear-gradient(135deg, rgba(99, 102, 241, 0.22) 0%, rgba(139, 92, 246, 0.26) 100%)`, viền phát sáng nhẹ `border: 1px solid rgba(129, 140, 248, 0.4)`, đổ bóng nhẹ `boxShadow: 0 2px 8px rgba(99, 102, 241, 0.25)`, chữ số `#a5b4fc` nổi bật và căn giữa hoàn hảo.
- **Nút Phóng to / Thu gọn 4 Hướng [ ⛶ ]**:
  - Hình dáng: Khung vuông bo tròn mềm `width: 26px`, `height: 26px`, `borderRadius: 7px`.
  - Viền & nền kính mờ: `background: rgba(255, 255, 255, 0.06)`, `border: 1px solid rgba(255, 255, 255, 0.12)`.
  - Tương tác Hover: Khi rê chuột vào, nền sáng lên `rgba(255, 255, 255, 0.12)`, viền sáng `0.25`, icon đổi từ xám `#94a3b8` sang trắng tinh `#ffffff`.
  - Icon SVG 4 góc khung viền (corners):
    - Trạng thái thu gọn (`220px`): Hiện icon 4 góc khung viền mở rộng `[ ⛶ ]`.
    - Trạng thái mở rộng (`350px`): Hiện icon 4 góc chụm vào trong thu gọn.

```tsx
{/* ── Cụm Tiêu đề + Badge Số lượng Bo Tròn 50% + Nút 4 Hướng Thích Ứng Dark/Light ── */}
<div style={{ display: 'inline-flex', alignItems: 'center', gap: 8, flexWrap: 'wrap' }}>
  {/* Tiêu đề (Đổi màu linh hoạt: Dark mode sáng rõ ràng #f0f3f8, Light mode tối rõ ràng #151922) */}
  <span className="table-header-title">
    Danh sách Services
  </span>

  {/* Badge số lượng đếm (Bo tròn 50%, gradient màu mờ phát sáng nhẹ theo theme) */}
  <span className="table-count-badge">
    {entries.length}
  </span>

  {/* Nút Phóng to / Thu gọn (Thích ứng theme Dark/Light, icon 4 góc khung viền [ ⛶ ]) */}
  <button
    type="button"
    className="table-expand-btn"
    onClick={() => setIsExpanded(prev => !prev)}
    title={isExpanded ? 'Thu gọn danh sách (220px)' : 'Mở rộng thêm chiều cao (350px)'}
  >
    <svg
      width="14"
      height="14"
      viewBox="0 0 24 24"
      fill="none"
      stroke="currentColor"
      strokeWidth="2.4"
      strokeLinecap="round"
      strokeLinejoin="round"
    >
      {isExpanded ? (
        /* Icon Thu gọn: 4 góc hướng vào trong */
        <>
          <polyline points="4 9 9 9 9 4" />
          <polyline points="20 9 15 9 15 4" />
          <polyline points="4 15 9 15 9 20" />
          <polyline points="20 15 15 15 15 20" />
        </>
      ) : (
        /* Icon Phóng to chuẩn: 4 góc khung viền [ ⛶ ] */
        <>
          <polyline points="9 4 4 4 4 9" />
          <polyline points="15 4 20 4 20 9" />
          <polyline points="9 20 4 20 4 15" />
          <polyline points="15 20 20 20 20 15" />
        </>
      )}
    </svg>
  </button>
</div>
```

```css
/* CSS Thích ứng Đổi Màu Linh Hoạt Giữa Chế Độ Tối (Dark) & Chế Độ Sáng (Light) */
.table-header-title {
  font-size: 15px;
  font-weight: 800;
  letter-spacing: -0.2px;
  color: var(--text-primary);
  transition: color 0.2s ease;
}

.table-count-badge {
  min-width: 24px;
  height: 24px;
  padding: 0 7px;
  border-radius: 50%;
  font-size: 12px;
  font-weight: 800;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  line-height: 1;
  user-select: none;
  transition: all 0.2s ease;
}

[data-theme="dark"] .table-count-badge {
  background: linear-gradient(135deg, rgba(99, 102, 241, 0.24) 0%, rgba(139, 92, 246, 0.3) 100%);
  color: #c7d2fe;
  border: 1px solid rgba(129, 140, 248, 0.45);
  box-shadow: 0 2px 10px rgba(99, 102, 241, 0.3);
}

[data-theme="dark"] .table-count-badge {
  background: linear-gradient(135deg, rgba(99, 102, 241, 0.35) 0%, rgba(139, 92, 246, 0.42) 100%);
  color: #e0e7ff;
  border: 1px solid rgba(165, 180, 252, 0.55);
  box-shadow: 0 2px 12px rgba(99, 102, 241, 0.45);
}

[data-theme="light"] .table-count-badge {
  background: linear-gradient(135deg, #4f46e5 0%, #7c3aed 100%);
  color: #ffffff;
  border: 1px solid rgba(79, 70, 229, 0.7);
  box-shadow: 0 2px 10px rgba(99, 102, 241, 0.4), 0 0 0 2px rgba(99,102,241,0.08);
}

.table-expand-btn {
  width: 28px;
  height: 28px;
  padding: 0;
  border-radius: 7px;
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s cubic-bezier(0.16, 1, 0.3, 1);
}

/* Nút ở chế độ Tối (Dark): nền mờ đen kính viền ánh sáng tinh tế */
[data-theme="dark"] .table-expand-btn {
  background: rgba(255, 255, 255, 0.08);
  border: 1px solid rgba(255, 255, 255, 0.16);
  color: #cbd5e1;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.3);
}

[data-theme="dark"] .table-expand-btn:hover {
  background: rgba(99, 102, 241, 0.25);
  border-color: rgba(129, 140, 248, 0.6);
  color: #ffffff;
  box-shadow: 0 3px 10px rgba(99, 102, 241, 0.35);
  transform: translateY(-1px);
}

/* Nút ở chế độ Sáng (Light): nền sáng tương phản nổi bật rõ ràng, không bị chìm */
[data-theme="light"] .table-expand-btn {
  background: #ffffff;
  border: 1px solid #c7d2fe;
  color: #4338ca;
  box-shadow: 0 1px 4px rgba(99,102,241,0.12);
}

[data-theme="light"] .table-expand-btn:hover {
  background: linear-gradient(135deg, #eef2ff 0%, #ede9fe 100%);
  border-color: #818cf8;
  color: #4f46e5;
  box-shadow: 0 3px 12px rgba(99, 102, 241, 0.25);
  transform: translateY(-1px);
}

/* Nút đóng X của Dropdown Portal Lọc chuẩn popover_modal_skill (Compact 22x22px, xoay 90° đỏ khi hover) */
.filter-portal-close-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 22px;
  height: 22px;
  padding: 0;
  font-size: 12px;
  line-height: 1;
  color: var(--text-muted, #94a3b8);
  background: var(--bg-tertiary, #202532);
  border: 1px solid var(--border-color, #2a2e39);
  border-radius: 50%;
  cursor: pointer;
  outline: none;
  margin-left: auto;
  flex-shrink: 0;
  transition: all 0.25s cubic-bezier(0.16, 1, 0.3, 1);
}

.filter-portal-close-btn:hover {
  color: #ff0000;
  border-color: #ef4444;
  background-color: rgba(239, 68, 68, 0.12);
  transform: rotate(90deg);
  box-shadow: 0 2px 8px rgba(239, 68, 68, 0.25);
}

.filter-portal-close-btn:active {
  transform: scale(0.9) rotate(90deg);
}
```

---

### 1.3 Thanh Hướng Dẫn Nhanh Gọn (Quick Tips Toolbar)
Nằm ở header toolbar cạnh hoặc đối xứng cụm tiêu đề bảng, gồm 3 badge trực quan:
- `↕️ Sắp xếp cột`: Click vào tiêu đề các cột để đảo chiều sắp xếp.
- `🌐 Lọc thông minh`: Nhấn nút lọc hoặc click đúp header để lọc Dropdown Portal.
- `⚡ Click đúp sửa nhanh`: Click đúp vào ô nội dung của bất kỳ dòng nào để nhập trực tiếp (Inline Edit) không cần mở modal.

---

### 1.4 Hệ Thống Icon SVG Đồng Bộ
| Vị trí | Icon SVG | Ý nghĩa |
| :--- | :---: | :--- |
| **Tiêu đề bảng** | Văn bản đậm | Chữ trắng/đen đậm sắc nét không icon thừa |
| **Nút mở rộng** | `[ ⛶ ]` SVG 4 góc khung viền | 4 góc bung ra ngoài (Mở rộng) ⇋ 4 góc chụm vào trong (Thu gọn) |
| **Vai trò: Admin** | Shield SVG (`M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z`) | Khiên bảo vệ, quyền quản trị tối cao |
| **Vai trò: Thành viên** | User SVG (`M20 21v-2a4 4... circle cx=12 cy=7 r=4`) | Người dùng thành viên hệ thống |
| **Header Cột** | 🏷️ / 👥 / 🔑 / 🎯 | Nhãn cột kèm mũi tên sort `↑ (A-Z)` / `↓ (Z-A)` / `↕` |
| **Nút Lọc Cột** | Filter SVG / Shield SVG / User SVG | Nút trigger lọc nổi trên Header cột kèm mũi tên ▾ |
| **Nút X Đóng Modal Lọc** | `✕` xoay 90° đỏ (`filter-portal-close-btn`) | Chuẩn `popover_modal_skill`, viền đỏ, chữ `#ff0000`, xoay 90 độ |
| **Active Sort / Filter** | Badge kèm SVG icon tương ứng | Hiển thị trạng thái đang lọc/sort kèm nút hủy `×` |
| **Thao tác Action** | SVG Lock / Delete | Icon hành động kèm mở rộng nhãn chữ (hover expand label) |

---

### 1.5 Quy Chuẩn Bố Cục Cột & Khoảng Cách (Column Layout, Padding & Spacing)
Để bảng hiển thị tập trung, không bị giãn khoảng cách thênh thang giữa các cột:
1. **Khoảng đệm (Padding) chuẩn gọn gàng**:
   - `th`: `padding: 10px 10px;` (chống tràn, chữ đậm `12px`, `white-space: nowrap`).
   - `td`: `padding: 10px 10px;` (`vertical-align: middle; white-space: nowrap`).
2. **Quy định tỷ lệ cột thông minh qua `<colgroup>`**:
   - Cột thông tin chính (Tên, Email, Tài nguyên): `width: auto` để chiếm trọn không gian dư thừa.
   - Các cột thuộc tính (Vai trò, Trạng thái, Phiên, Ngày, Thao tác): gán độ rộng cố định vừa vặn (`100px - 150px`).
3. **Badge Vai trò Đa Năng (Role Badge - Hover Expand & Double-Click)**:
   - Bình thường: Chỉ hiển thị SVG Icon tinh gọn (`width: 13px`), tiết kiệm diện tích.
   - Khi hover: Trượt mở rộng mượt mà (`max-width: 100px; opacity: 1; transition: 0.24s cubic-bezier(0.16, 1, 0.3, 1)`), hiện rõ tên vai trò.
   - Khi Double-click: Kích hoạt đổi vai trò trực tiếp (nếu được phép), loại bỏ hoàn toàn việc phải thêm một cột "Phân quyền" riêng biệt rườm rà.


## 2. LOGIC XỬ LÝ (CORE BUSINESS LOGIC)

### 2.1 Pipeline Dữ Liệu (Filtering ➔ Sorting)
Dữ liệu gốc đi qua `useMemo` gồm các bước:
1. **Lọc Danh Mục / Vai Trò / Thuộc Tính**: So sánh giá trị với state lọc đang chọn.
2. **Lọc Trạng Thái**: Phân loại theo `all`, `active`, `locked`, hoặc mã cụ thể.
3. **Sắp Xếp (Sorting)**:
   - Chuỗi ký tự (Tên, Email): `localeCompare`.
   - Số liệu (Phiên, Tốc độ ms): `order === 'asc' ? a - b : b - a`.
   - Trạng thái / Boolean: Đưa giá trị ưu tiên lên trước.

### 2.2 Thuật Toán Định Vị Modal Thông Minh 4 Hướng (`calculateSmartPosition`)
Khi bấm nút lọc:
- **Thứ tự ưu tiên không gian**: `Dưới (bottom) > Phải (right) > Trái (left) > Trên (top)`.
- **Khoảng cách tiêu chuẩn**:
  - Luôn cách nút bấm đúng **`1px`**.
  - Luôn cách 4 mép màn hình tối thiểu **`2px`**.
  - Chiều cao tối thiểu `minHeight: 200px` (có thanh cuộn `overflowY: auto`).
- **Portal vào `document.body`**: Dropdown không nằm trong DOM của bảng mà render trực tiếp ra ngoài qua `createPortal`, loại bỏ hoàn toàn lỗi bị che khuất bởi `overflow: auto/hidden`.
- **Tự động bám dính**: Đăng ký sự kiện `scroll` (với `capture: true` để bắt cả cuộn bảng và cuộn trang) + sự kiện `resize` của trình duyệt.

```typescript
const calculateSmartPosition = (triggerEl: HTMLElement | null): React.CSSProperties => {
  if (!triggerEl) return { display: 'none' };
  const rect = triggerEl.getBoundingClientRect();
  const vw = window.innerWidth;
  const vh = window.innerHeight;
  const pad = 2; // Cách mép màn hình 2px
  const gap = 1; // Cách nút 1px
  const targetW = 240;
  const minH = 200;

  const spaceBottom = Math.max(0, vh - rect.bottom - gap - pad);
  const spaceTop = Math.max(0, rect.top - gap - pad);
  const spaceRight = Math.max(0, vw - rect.right - gap - pad);
  const spaceLeft = Math.max(0, rect.left - gap - pad);

  // Ưu tiên: Dưới > Phải > Trái > Trên
  let placement: 'bottom' | 'right' | 'left' | 'top' = 'bottom';
  if (spaceBottom >= minH) {
    placement = 'bottom';
  } else if (spaceRight >= targetW && (vh - 2 * pad) >= minH) {
    placement = 'right';
  } else if (spaceLeft >= targetW && (vh - 2 * pad) >= minH) {
    placement = 'left';
  } else if (spaceTop >= minH) {
    placement = 'top';
  } else {
    const maxVert = Math.max(spaceBottom, spaceTop);
    const maxHoriz = Math.max(spaceRight, spaceLeft);
    if (spaceBottom >= spaceTop && spaceBottom >= maxHoriz) placement = 'bottom';
    else if (spaceRight >= maxVert && spaceRight >= spaceLeft) placement = 'right';
    else if (spaceLeft >= maxVert) placement = 'left';
    else placement = 'top';
  }

  let top = 0, left = 0, maxH = minH;

  if (placement === 'bottom') {
    top = rect.bottom + gap;
    maxH = Math.max(minH, spaceBottom);
    left = Math.min(Math.max(pad, rect.left), vw - pad - targetW);
  } else if (placement === 'top') {
    maxH = Math.max(minH, spaceTop);
    top = Math.max(pad, rect.top - gap - maxH);
    left = Math.min(Math.max(pad, rect.left), vw - pad - targetW);
  } else if (placement === 'right') {
    left = rect.right + gap;
    maxH = Math.min(320, Math.max(minH, vh - 2 * pad));
    top = Math.min(Math.max(pad, rect.top), vh - pad - maxH);
  } else {
    left = Math.max(pad, rect.left - gap - targetW);
    maxH = Math.min(320, Math.max(minH, vh - 2 * pad));
    top = Math.min(Math.max(pad, rect.top), vh - pad - maxH);
  }

  return {
    position: 'fixed',
    top: Math.round(top),
    left: Math.round(left),
    width: targetW,
    maxHeight: Math.round(maxH),
    minHeight: minH,
    zIndex: 99999,
  };
};
```

### 2.3 Sửa Nhanh Tại Chỗ (Inline Editing)
- Sự kiện `onDoubleClick` tại ô dữ liệu kích hoạt chế độ sửa.
- Tự động `focus()` và `select()` toàn bộ text.
- Nhấn `Enter` hoặc click ra ngoài (`onBlur`): Gửi payload cập nhật về Server / State cha (Optimistic Update).
- Nhấn `Escape`: Hủy thao tác sửa, giữ nguyên giá trị ban đầu.

---

### 2.4 Kiểm Soát Quyền & Vô Hiệu Thao Tác (Action Permission & Safety Guards)
Mọi bảng quản trị hoặc danh sách dữ liệu có tương tác nguy hiểm (phân quyền, khóa, xóa) bắt buộc phải tích hợp lớp bảo vệ logic 2 tầng (Frontend + Backend):

1. **Ràng buộc Tài Khoản Bản Thân (`isSelf`)**:
   - `isSelf = user._id === currentUser.id`.
   - **Phân quyền**: Vô hiệu hóa nút đổi vai trò (`disabled`), nhãn hiển thị `(Bạn)` kèm `title="Không thể tự thay đổi vai trò của chính mình"`.
   - **Khóa tài khoản**: Vô hiệu hóa nút khóa (`disabled`), tooltip giải thích rõ ràng.
   - **Xóa tài khoản**: Vô hiệu hóa nút xóa (`disabled`), tooltip giải thích rõ ràng.

2. **Ràng buộc Quản Trị Viên Duy Nhất (`isLastAdmin`)**:
   - Tính toán `totalAdmins = users.filter(u => u.role === 'admin').length`.
   - `isLastAdmin = user.role === 'admin' && totalAdmins <= 1`.
   - **Hạ quyền**: Vô hiệu hóa nút hạ quyền khi `isLastAdmin` (`title="Không thể hạ quyền Quản trị viên duy nhất trong hệ thống"`).
   - **Khóa tài khoản**: Vô hiệu hóa nút khóa khi `isLastAdmin` (`title="Không thể khóa Quản trị viên duy nhất trong hệ thống"`).
   - **Xóa tài khoản**: Vô hiệu hóa nút xóa khi `isLastAdmin` (`title="Không thể xóa Quản trị viên duy nhất trong hệ thống"`).

3. **Quy Chuẩn CSS Cho Trạng Thái Vô Hiệu Hóa (`.disabled` / `:disabled`)**:
   - `opacity: 0.38` kết hợp `filter: grayscale(0.8)`.
   - Con trỏ `cursor: not-allowed !important`.
   - **Bắt buộc giữ `pointer-events: auto`**: Cho phép người dùng rê chuột vào nút bị khóa để đọc tooltip (`title`) giải thích lý do bị cấm thao tác.
   - Khử toàn bộ hiệu ứng `hover`, `active`, không nở rộng nhãn `.action-label` và không có bóng phát sáng.

---

### 2.5 Huy Hiệu Tương Tác Vai Trò & Trạng Thái (Interactive Role & Status Badges)
Để giữ bảng gọn gàng và không làm căng giãn các cột, áp dụng pattern tương tác công thái học:
1. **Header Cột**:
   - Giữ ngắn gọn với biểu tượng SVG lọc và mũi tên nhỏ `▾` (Không chèn chuỗi text dài như "Lọc", "Tất cả trạng thái").
   - **Tuyệt đối không pha trộn nhiều ngôn ngữ trong tooltip** (ví dụ: cấm viết dạng `Lọc theo vai trò (Filter Role)`).
   - Bắt buộc dùng `t('admin_filter_by_role')` và `t('admin_filter_by_status')` độc lập cho từng ngôn ngữ. Khi người dùng chọn ngôn ngữ nào thì tooltip chỉ hiển thị duy nhất ngôn ngữ đó; nếu ngôn ngữ đó thiếu khóa dịch thì tự động fallback chuẩn về Tiếng Anh (`en`).
2. **Huy hiệu trong dòng (`.user-role-badge`, `.status-badge`)**:
   - **Mặc định**: Chỉ hiển thị icon hoặc chấm trạng thái tròn (`status-dot`), nhãn text ẩn (`max-width: 0`, `opacity: 0`).
   - **Rê chuột (Hover)**: Nở mượt nhãn text (`max-width: 100px`, `opacity: 1`), hiển thị Tiếng Việt rõ ràng.
   - **Nhấp đúp (Double-click)**:
     - Kích hoạt thao tác nhanh đổi vai trò (`onChangeRole`) hoặc đổi trạng thái khóa/mở (`onChangeStatus`).
     - Kiểm tra đầy đủ quyền an toàn (`isSelf`, `isLastAdmin`). Nếu không được phép, thêm class `.readonly`, chặn `onDoubleClick` và hiển thị tooltip tiếng Việt giải thích lý do cấm.

---

### 2.6 Quy Chuẩn Căn Giữa Các Cột Ngắn & Thao Tác (Short Columns & Actions Center Alignment)
Đối với các bảng quản trị dữ liệu (như bảng người dùng, bảng tài nguyên, bảng phiên hoạt động):
1. **Các cột văn bản dài hoặc nhận diện chính** (Tên, Email, Mô tả, Đường dẫn): Căn lề trái (`text-align: left`) để thuận mắt đọc tự nhiên.
2. **Các cột dữ liệu ngắn, trạng thái, huy hiệu và nút hành động** (Vai trò `Role`, Trạng thái `Status`, Số phiên `Sessions`, Hành động `Actions`):
   - **Bắt buộc căn giữa đồng bộ cả `<th>` và `<td>`**:
     - `th style={{ textAlign: 'center' }}` kết hợp container flex nội dung `justifyContent: 'center'`.
     - `td style={{ textAlign: 'center' }}` để huy hiệu hoặc nút bấm nằm ngay trục trung tâm của cột.
3. **Chỉ Báo Sắp Xếp Ở Tiêu Đề Cột (Sort Indicator)**:
   - **Tuyệt đối không chèn text cứng** như `(Ít nhất / Nhiều nhất)`, `(Cũ nhất / Mới nhất)`, `(A-Z / Z-A)` làm tràn dòng và gây lỗi đa ngôn ngữ.
   - **Chỉ hiển thị ký hiệu gọn gàng**: `↑` (tăng dần), `↓` (giảm dần) hoặc `↕` (chưa sắp xếp).
   - Chi tiết ngữ nghĩa sắp xếp được chuyển vào tooltip `title` của cột và truyền qua hàm dịch `t('...')` đa ngôn ngữ đầy đủ.

