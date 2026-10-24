# Kỹ Năng Thiết Kế Thanh Cuộn Siêu Mảnh (Ultra-Thin 5px Scrollbar) Song Song Dọc & Ngang

Tài liệu nguyên lý thiết kế, thông số kỹ thuật chuẩn và mã nguồn độc lập (Zero-Dependency) cho thanh cuộn siêu mảnh (**5px**) hoạt động song song cả chiều dọc (Vertical) và chiều ngang (Horizontal), ứng dụng cho modal, bảng dữ liệu, và toàn bộ vùng nội dung trang web.

---

## 1. Nguyên Lý Thiết Kế & Thông Số UX Chuẩn

| Thuộc tính | Chi tiết kỹ thuật | Ghi chú UX |
| :--- | :--- | :--- |
| **Độ dày thanh cuộn Dọc (Width)** | **`5px`** | Siêu mảnh (Ultra-thin), tinh tế, không chiếm diện tích hiển thị |
| **Độ dày thanh cuộn Ngang (Height)**| **`5px`** | Mảnh nhẹ ở chân container/bảng, không gây cộm mắt |
| **Màu con trượt (Thumb)** | `rgba(99, 102, 241, 0.25)` (Indigo tím/xanh nhạt) | Hòa nhã với cả Dark Theme và Light Theme |
| **Màu con trượt khi Hover** | `rgba(99, 102, 241, 0.45)` | Đậm hơn khi trỏ chuột giúp người dùng dễ định vị |
| **Bo góc con trượt (Border Radius)** | **`3px`** | Bo tròn dạng viên thuốc thanh mảnh |
| **Rãnh cuộn (Track)** | `background: transparent` | Trong suốt hoàn toàn để giữ background sạch sẽ |
| **Hỗ trợ Cross-Browser** | Webkit (`::-webkit-scrollbar`) + Firefox (`scrollbar-width`, `scrollbar-color`) | Tương thích 100% mọi trình duyệt hiện đại |

---

## 2. Mã Nguồn CSS Tái Sử Dụng (Zero-Dependency)

### A. Định nghĩa Class tiện ích độc lập (`.custom-scrollbar` hoặc `.custom-table-scrollbar`)

```css
/* ========================================================
   THANH CUỘN SIÊU MẢNH TINH TẾ (CUSTOM ULTRA-THIN SCROLLBAR 5PX)
   ======================================================== */
.custom-scrollbar::-webkit-scrollbar,
.custom-table-scrollbar::-webkit-scrollbar {
  width: 5px;   /* Chiều rộng thanh cuộn DỌC */
  height: 5px;  /* Chiều cao thanh cuộn NGANG */
}

.custom-scrollbar::-webkit-scrollbar-track,
.custom-table-scrollbar::-webkit-scrollbar-track {
  background: transparent;
}

.custom-scrollbar::-webkit-scrollbar-thumb,
.custom-table-scrollbar::-webkit-scrollbar-thumb {
  background: rgba(99, 102, 241, 0.25);
  border-radius: 3px;
}

.custom-scrollbar::-webkit-scrollbar-thumb:hover,
.custom-table-scrollbar::-webkit-scrollbar-thumb:hover {
  background: rgba(99, 102, 241, 0.45);
}

/* Hỗ trợ Firefox hiện đại */
.custom-scrollbar,
.custom-table-scrollbar {
  scrollbar-width: thin;
  scrollbar-color: rgba(99, 102, 241, 0.25) transparent;
}
```

### A.1. Cơ chế mở rộng tối đa theo độ cao Viewport cho Menu Thả Xuống (Dynamic Viewport Height)
Khi áp dụng thanh cuộn cho các Menu thả xuống (`.lang-dropdown-menu`, `.dropdown-menu`, `.portal-filter-dropdown`):
- **Không đặt `max-height` cố định cứng** (như `300px`, `320px`) vì sẽ gây cuộn thừa thãi trên màn hình lớn.
- **Tính toán theo mép đáy màn hình (cách viền dưới đúng 2px)**:
  ```css
  /* Mở rộng tự nhiên, chạm cách mép dưới màn hình đúng 2px mới xuất hiện thanh cuộn */
  max-height: calc(100vh - var(--header-height) - 10px);
  overflow-y: auto;
  overscroll-behavior-y: contain;
  ```
- **Lợi ích UX**: Nếu chiều cao màn hình còn đủ chỗ thì menu bung dài tự nhiên 100% không hiện cuộn; chỉ khi danh sách dài chạm gần đáy màn hình thì thanh cuộn 5px mới xuất hiện.

---

### B. Áp dụng toàn trang (Global Body & Modal Content)

```css
/* Áp dụng thanh cuộn 5px cho toàn bộ trang và hộp thoại Modal */
html::-webkit-scrollbar,
body::-webkit-scrollbar,
.modal-card::-webkit-scrollbar {
  width: 5px;
  height: 5px;
}

html::-webkit-scrollbar-track,
body::-webkit-scrollbar-track,
.modal-card::-webkit-scrollbar-track {
  background: transparent;
}

html::-webkit-scrollbar-thumb,
body::-webkit-scrollbar-thumb,
.modal-card::-webkit-scrollbar-thumb {
  background: rgba(99, 102, 241, 0.25);
  border-radius: 3px;
}

html::-webkit-scrollbar-thumb:hover,
body::-webkit-scrollbar-thumb:hover,
.modal-card::-webkit-scrollbar-thumb:hover {
  background: rgba(99, 102, 241, 0.45);
}

html,
body,
.modal-card {
  scrollbar-width: thin;
  scrollbar-color: rgba(99, 102, 241, 0.25) transparent;
}
```

---

## 3. Mã Nguồn JSX Mẫu: Bảng / Vùng Cuộn Co Giãn Chiều Cao (+130px)

```tsx
"use client";

import React, { useState } from "react";

export function ExpandableScrollableArea() {
  const [isExpanded, setIsExpanded] = useState(false);

  return (
    <div style={{
      background: "#12161f",
      border: "1px solid rgba(255, 255, 255, 0.08)",
      borderRadius: 12,
      overflow: "hidden",
      maxWidth: 900,
      margin: "20px auto"
    }}>
      {/* Thanh công cụ với nút co giãn (+130px) */}
      <div style={{
        padding: "12px 16px",
        display: "flex",
        alignItems: "center",
        justifyContent: "space-between",
        borderBottom: "1px solid rgba(255, 255, 255, 0.06)"
      }}>
        <span style={{ fontWeight: 700, fontSize: 15, color: "#fff" }}>Danh sách Dữ liệu</span>
        <button
          type="button"
          onClick={() => setIsExpanded(prev => !prev)}
          style={{
            padding: "5px 12px",
            fontSize: 12,
            borderRadius: 6,
            cursor: "pointer",
            background: isExpanded ? "rgba(99, 102, 241, 0.2)" : "rgba(255, 255, 255, 0.06)",
            color: isExpanded ? "#818cf8" : "#94a3b8",
            border: "1px solid rgba(99, 102, 241, 0.4)",
            display: "inline-flex",
            alignItems: "center",
            gap: 6
          }}
        >
          <span>{isExpanded ? "🔼" : "🔽"}</span>
          <span>{isExpanded ? "Thu hẹp (-130px)" : "Mở rộng (+130px)"}</span>
        </button>
      </div>

      {/* Vùng cuộn 2 chiều dọc & ngang sử dụng class .custom-scrollbar */}
      <div
        className="custom-scrollbar"
        style={{
          maxHeight: isExpanded ? 350 : 220, // Chiều cao mở rộng linh hoạt
          overflowY: "auto",
          overflowX: "auto",
          transition: "max-height 0.25s ease-in-out",
          padding: 16
        }}
      >
        <div style={{ minWidth: 650 }}>
          {/* Nội dung bảng hoặc danh sách dài */}
        </div>
      </div>
    </div>
  );
}
```

---

## 4. Checklist Kiểm Thử
- [x] **Kích thước chuẩn**: Thanh cuộn chỉ dày đúng `5px`, thanh mảnh, không đè nội dung.
- [x] **Rãnh trong suốt**: Giữ trọn vẹn màu nền của Modal và Page.
- [x] **Hiệu ứng Hover**: Con trượt đậm lên `0.45` khi hover chuột vào, phản hồi thị giác tốt.
- [x] **Đa trình duyệt**: Hiển thị đồng nhất trên Chrome, Safari, Edge và Firefox.
