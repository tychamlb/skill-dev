# Kỹ Năng Thiết Kế Sidebar Trái: Thanh Cuộn Vô Hình & Thu Phóng Công Thái Học

Tài liệu nguyên lý thiết kế, kỹ thuật CSS chuyên sâu và code mẫu chuẩn công thái học độc lập (Zero-Dependency) tái sử dụng cho mọi dự án (React / Next.js / Vite / Tailwind / CSS thuần).

---

## 1. Nguyên Lý & Giải Pháp Kỹ Thuật Cốt Lõi

### A. Ẩn thanh cuộn nhưng vẫn cuộn mượt mà (Invisible / Ghost Scrollbar)
Để thanh cuộn biến mất 100% nhưng người dùng vẫn lăn chuột, vuốt trackpad hoặc cảm ứng mượt mà:
- **Chrome / Safari / Edge / Webkit**: `::-webkit-scrollbar { display: none; }`
- **Firefox**: `scrollbar-width: none;`
- **IE / Edge cũ**: `-ms-overflow-style: none;`
- **Chống giật trang**: Thêm `overflow-y: auto;` và `overscroll-behavior-y: contain;` để tránh hiện tượng bounce/scroll lan ra layout cha khi chạm đáy.

```css
/* Class tiện ích tái sử dụng */
.hide-scrollbar {
  overflow-y: auto;
  overscroll-behavior-y: contain;
  scrollbar-width: none; /* Firefox */
  -ms-overflow-style: none; /* IE / Edge cũ */
}
.hide-scrollbar::-webkit-scrollbar {
  display: none; /* Chrome, Safari, Edge Webkit */
}
```

---

### B. Nút chuyển đổi thu/phóng & Nút tác vụ Sidebar (Aura Glow & Ergonomic Controls)
- **Vị trí công thái học**:
  - *Floating Edge Tab*: Nằm nổi ở đúng mép phải Sidebar (`position: fixed; top: 50%; left: width; transform: translateY(-50%);`), chạy mượt theo mép sidebar.
  - *Nút Đăng xuất / Footer Action*: Nằm ở chân sidebar (`.sidebar-logout-btn`), phân tầng kích thước đa thiết bị (`<520px: 24px`, `520px-1500px: 32px`, `>1500px: 34px`).
- **Ngôn ngữ phát sáng đồng bộ (Aura Glow & Gold Hover)**:
  - *Trạng thái thường*: Bao quanh bởi vầng sáng hào quang huyền bí `filter: drop-shadow(0 0 6px rgba(255, 68, 0, 0.4))`.
  - *Hiệu ứng rê chuột (Hover)*: Phóng to nhẹ `scale(1.08)`, nền kính mờ đỏ `rgba(239, 68, 68, 0.15)`, viền đỏ `#ef4444`, bừng sáng vầng vàng tươi `filter: drop-shadow(0 0 12px rgba(234, 179, 8, 0.85))` và bóng đổ `box-shadow: 0 4px 14px rgba(239, 68, 68, 0.3)`.
  - *Hiệu ứng bấm giữ (Active)*: Co nhẹ `scale(0.92)` êm ái.
  - *Icon SVG bên trong*: Khóa tương tác `pointer-events: none` để chống nhiễu click.
- **Hiệu ứng xoay mượt thay vì đổi icon**:
  - Dùng `transform: rotate(180deg); transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1);` để chuyển động êm ái, tránh cảm giác giật frame UI.

---

### C. Cân bằng & Căn giữa khi thu gọn (Icon-only mode)
- **Chiều rộng chuẩn UX**: 
  - Màn hình Desktop: Thu gọn: **80px** | Mở rộng: **256px**.
  - Màn hình Tablet / Mobile lớn (`520px – 900px`): **Giảm 20px chiều rộng** (từ `270px` xuống **`250px`**) để vừa vặn, thanh thoát.
  - **Màn hình Điện Thoại Nhỏ (`< 520px`)**: **Giảm 50px chiều rộng** (từ `270px` xuống **`220px`**) để không che khuất toàn bộ màn hình, tối ưu không gian cho ngón cái thao tác và người dùng vẫn thấy được một phần bối cảnh trang phía sau backdrop.
- **Icon Container cố định (`24x24px`)**:
  - Bọc icon trong container cố định kích thước `width: 24px; height: 24px; display: flex; align-items: center; justify-content: center; flex-shrink: 0;`.
  - Giúp icon luôn nằm chính xác ở tâm nút, không bao giờ bị méo hoặc giật vị trí khi label text bị ẩn.
- **Kích thước nút khi thu gọn (`44x44px` và `margin: 0 auto`)**:
  - Khi mở rộng: Nút dạng thanh dài ngang full width (`width: 100%; justify-content: flex-start;`).
  - Khi thu gọn: Chuyển thành khối vuông hoặc tròn chuẩn ngón tay bấm (`width: 44px; height: 44px; margin: 0 auto; justify-content: center;`).

### D. Khung Nút Dạng Viên Thuốc, Nền Vàng Kem Nhạt & Chữ Đỏ Khi Hover (Pill Frame & Warm Hover Synergy)
- **Cấu trúc Khung Viền Bao Quanh (`li.nav-item`)**:
  - `border-radius: 48px 0 0 48px;` tạo vòm bo tròn ở mép trái bao quanh nút.
  - Bình thường nền trong suốt `background: transparent;`.
  - **Khi Hover**: Chuyển sang nền vàng kem nhạt `background: var(--light-warning)` (`#FFF2C6` trên Light theme, và lớp vàng kem kính mờ `rgba(255, 242, 198, 0.12)` trên Dark theme).
- **Phản Hồi Nút Bấm (`li a`)**:
  - Nút con `a` bo tròn dạng viên thuốc `border-radius: 48px;` (hoặc `radius-full`).
  - **Khi Hover**: Chữ và icon chuyển sang màu đỏ rực (`color: #e50914` / `color: red;`), đồng thời dịch nhẹ sang phải `transform: translateX(4px);` với `transition: all 0.3s ease;` tạo cảm giác tương tác phản hồi xúc giác tinh tế và cao cấp.
- **Khi Thu Gọn (Collapsed Mode)**:
  - Khung `li` và thẻ `a` chuyển thành hình tròn hoàn hảo `border-radius: 50%; width: 44px; height: 44px; margin: 0 auto;`.
  - Giữ nguyên vầng hào quang Aura Glow và bung sáng vàng tươi `drop-shadow(0 0 12px rgba(234, 179, 8, 0.85))` khi hover.

---

## 2. Mã Nguồn Mẫu Hoàn Chỉnh (Độc Lập / Tái Sử Dụng)

Dưới đây là component mẫu `AppSidebar.tsx` viết bằng React / TypeScript với Inline CSS độc lập:

```tsx
"use client";

import React, { useState } from "react";

export interface MenuItem {
  id: string;
  label: string;
  icon: React.ReactNode;
  badge?: string | number;
}

interface AppSidebarProps {
  items?: MenuItem[];
}

export const AppSidebar: React.FC<AppSidebarProps> = ({ items = sampleItems }) => {
  const [isCollapsed, setIsCollapsed] = useState(false);
  const [activeId, setActiveId] = useState("home");

  const sidebarWidth = isCollapsed ? 72 : 256;

  return (
    <aside
      style={{
        width: sidebarWidth,
        height: "100vh",
        backgroundColor: "#0f172a",
        color: "#f8fafc",
        display: "flex",
        flexDirection: "column",
        transition: "width 0.3s cubic-bezier(0.4, 0, 0.2, 1)",
        borderRight: "1px solid rgba(255, 255, 255, 0.08)",
        userSelect: "none",
        position: "relative",
      }}
    >
      {/* 1. Header Sidebar */}
      <div
        style={{
          height: 64,
          display: "flex",
          alignItems: "center",
          justifyContent: isCollapsed ? "center" : "space-between",
          padding: isCollapsed ? "0" : "0 20px",
          borderBottom: "1px solid rgba(255, 255, 255, 0.06)",
        }}
      >
        {!isCollapsed ? (
          <span style={{ fontWeight: 700, fontSize: 18, letterSpacing: -0.5 }}>
            MY APP
          </span>
        ) : (
          <div
            style={{
              width: 36,
              height: 36,
              borderRadius: 8,
              background: "linear-gradient(135deg, #e50914, #ff4d58)",
              display: "flex",
              alignItems: "center",
              justifyContent: "center",
              fontWeight: 800,
            }}
          >
            M
          </div>
        )}
      </div>

      {/* 2. Menu Navigation (Tự cuộn với thanh cuộn vô hình) */}
      <nav
        className="hide-scrollbar"
        style={{
          flex: 1,
          overflowY: "auto",
          overscrollBehaviorY: "contain",
          padding: isCollapsed ? "12px 0" : "12px 10px",
          display: "flex",
          flexDirection: "column",
          gap: 6,
          scrollbarWidth: "none",
          msOverflowStyle: "none",
        }}
      >
        {items.map((item) => {
          const isActive = activeId === item.id;
          return (
            <button
              key={item.id}
              onClick={() => setActiveId(item.id)}
              title={isCollapsed ? item.label : undefined}
              style={{
                display: "flex",
                alignItems: "center",
                justifyContent: isCollapsed ? "center" : "flex-start",
                width: isCollapsed ? 44 : "100%",
                height: 44,
                margin: isCollapsed ? "0 auto" : 0,
                padding: isCollapsed ? 0 : "0 12px",
                borderRadius: 10,
                border: "none",
                background: isActive ? "rgba(229, 9, 20, 0.15)" : "transparent",
                color: isActive ? "#ff4d58" : "#94a3b8",
                cursor: "pointer",
                transition: "all 0.2s ease",
                position: "relative",
              }}
              onMouseEnter={(e) => {
                if (!isActive) e.currentTarget.style.backgroundColor = "rgba(255, 255, 255, 0.05)";
              }}
              onMouseLeave={(e) => {
                if (!isActive) e.currentTarget.style.backgroundColor = "transparent";
              }}
            >
              {/* Vạch Active bên trái khi thu gọn */}
              {isCollapsed && isActive && (
                <div
                  style={{
                    position: "absolute",
                    left: 0,
                    width: 3,
                    height: 20,
                    backgroundColor: "#e50914",
                    borderRadius: "0 4px 4px 0",
                  }}
                />
              )}

              {/* Icon Container 24x24: Đảm bảo không lệch tâm */}
              <div
                style={{
                  width: 24,
                  height: 24,
                  display: "flex",
                  alignItems: "center",
                  justifyContent: "center",
                  flexShrink: 0,
                }}
              >
                {item.icon}
              </div>

              {/* Label: Ẩn khi thu gọn */}
              {!isCollapsed && (
                <span
                  style={{
                    marginLeft: 12,
                    fontSize: 14,
                    fontWeight: 500,
                    whiteSpace: "nowrap",
                    overflow: "hidden",
                    textOverflow: "ellipsis",
                  }}
                >
                  {item.label}
                </span>
              )}
            </button>
          );
        })}
      </nav>

      {/* 3. Nút Toggle Thu/Phóng Xoay 180 Độ ở Footer */}
      <div
        style={{
          padding: 12,
          borderTop: "1px solid rgba(255, 255, 255, 0.06)",
          display: "flex",
          justifyContent: "center",
        }}
      >
        <button
          onClick={() => setIsCollapsed(!isCollapsed)}
          title={isCollapsed ? "Mở rộng menu" : "Thu gọn menu"}
          style={{
            width: isCollapsed ? 44 : "100%",
            height: 40,
            display: "flex",
            alignItems: "center",
            justifyContent: "center",
            gap: 8,
            backgroundColor: "rgba(255, 255, 255, 0.05)",
            border: "1px solid rgba(255, 255, 255, 0.08)",
            borderRadius: 8,
            color: "#94a3b8",
            cursor: "pointer",
            transition: "all 0.2s ease",
          }}
          onMouseEnter={(e) => {
            e.currentTarget.style.backgroundColor = "rgba(255, 255, 255, 0.1)";
            e.currentTarget.style.color = "#ffffff";
          }}
          onMouseLeave={(e) => {
            e.currentTarget.style.backgroundColor = "rgba(255, 255, 255, 0.05)";
            e.currentTarget.style.color = "#94a3b8";
          }}
        >
          {/* Mũi tên SVG xoay 180 độ mượt mà */}
          <svg
            width="18"
            height="18"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            strokeWidth="2"
            strokeLinecap="round"
            strokeLinejoin="round"
            style={{
              transform: isCollapsed ? "rotate(180deg)" : "rotate(0deg)",
              transition: "transform 0.3s cubic-bezier(0.4, 0, 0.2, 1)",
            }}
          >
            <path d="M15 18l-6-6 6-6" />
          </svg>

          {!isCollapsed && (
            <span style={{ fontSize: 13, fontWeight: 500 }}>Thu gọn</span>
          )}
        </button>
      </div>

      <style jsx>{`
        .hide-scrollbar::-webkit-scrollbar {
          display: none;
        }
      `}</style>
    </aside>
  );
};

const sampleItems: MenuItem[] = [
  { id: "home", label: "Trang chủ", icon: <span>🏠</span> },
  { id: "movies", label: "Phim lẻ", icon: <span>🎬</span> },
  { id: "series", label: "Phim bộ", icon: <span>📺</span> },
  { id: "trending", label: "Thịnh hành", icon: <span>🔥</span> },
];
```

---

## 3. Tóm Tắt Các Điểm Cốt Lõi Tái Sử Dụng
1. **Container icon cố định (`24x24px`)**: Chống méo hoặc nhảy vị trí icon khi text biến mất.
2. **Kích thước nút khi thu nhỏ (`44x44px` và `margin: 0 auto`)**: Biến thanh menu dài thành các ô vuông bo góc căn giữa cột.
3. **Mũi tên xoay 180° (`rotate(180deg)`) hoặc đảo hướng**: Tránh giật UI, tạo hiệu ứng đảo chiều mượt mà.
4. **Bộ ba thuộc tính CSS ẩn scrollbar**:
   ```css
   scrollbar-width: none;
   -ms-overflow-style: none;
   &::-webkit-scrollbar { display: none; }
   ```

---

## 4. Chuẩn Hóa Kiến Trúc Responsive: Desktop vs Mobile (< 520px)

1. **Trên Desktop / Tablet (> 520px)**:
   - Dùng nút **Tab Quai Bấm Chuẩn** dính mép phải Sidebar:
     ```css
     position: fixed;
     top: 50%;
     transform: translateY(-50%);
     left: isCollapsed ? 80px : 256px;
     z-index: 105;
     border-left: none;
     border-radius: 0 8px 8px 0;
     ```
2. **Trên Điện thoại (< 520px / < 900px)**:
   - **Ẩn nút quai bấm**: `display: none !important;` (chỉ sử dụng nút Hamburger Menu `☰` trên Header).
   - **Chế độ Drawer đè lên trên**: `position: fixed; z-index: 150 !important;` kèm lớp nền tối mờ Backdrop `z-index: 140;`.
   - **Không thu nhỏ icon-only trên Mobile**: Khi mở drawer trên Mobile, luôn hiển thị đầy đủ danh sách gồm **Icon + Tên chữ** (`width: 270px; .nav-item-text { display: inline-block !important; }`), hiển thị đầy đủ thông tin tài khoản và nút Đăng xuất ở chân Sidebar.