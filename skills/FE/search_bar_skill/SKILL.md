# Kỹ Năng Thiết Kế Thanh Tìm Kiếm Đa Thiết Bị: Breakpoint 520px & Mobile Search Full-Header

Tài liệu nguyên lý thiết kế, cấu trúc giao diện responsive và mã nguồn mẫu độc lập (Zero-Dependency) cho thanh tìm kiếm đa nền tảng (Desktop / Tablet / Mobile). Tái sử dụng được cho mọi dự án React, Next.js, Vue hoặc HTML/CSS thuần.

---

## 1. Nguyên Lý Thiết Kế & Trải Nghiệm Người Dùng (UI/UX)

### A. Quy Chuẩn Breakpoint Responsive: Mốc 520px
- **Màn hình lớn (`> 520px` - Desktop & Tablet)**:
  - Thanh tìm kiếm dài nằm ở trung tâm Header (`.topbar__center`).
  - Thiết kế dạng viên thuốc (`border-radius: 15px` hoặc `var(--radius-full)`).
  - Tích hợp hiệu ứng micro-interaction: Khi người dùng click/focus vào input, chữ thụt lề mượt mà (`padding-left: 32px`) và icon địa cầu/kính lúp trượt hiện ra (`scale(1)`).
  - **Nút kính lúp Mobile (`.mobile-search-btn`) BẮT BUỘC ẩn hoàn toàn** (`display: none !important;`).
- **Màn hình nhỏ (`≤ 520px` - Mobile Phones)**:
  - Thanh tìm kiếm dài mặc định **ẩn hoàn toàn** (`display: none !important;`) để tránh chật chội Header.
  - Hiện một nút kính lúp hình tròn nhỏ (`.mobile-search-btn` kích thước 34px - 40px) ở cụm tác vụ bên phải (`.topbar__right`).

---

### B. Chế Độ Tìm Kiếm Mobile Tràn Toàn Bộ Header (Full-Header Search)
Khi người dùng chạm vào nút kính lúp trên Mobile (`≤ 520px`):
1. **Chiếm trọn không gian Header (`width: 100%`)**:
   - Khung chứa tìm kiếm mở rộng tràn viền Header (`flex: 1 1 100%`, `padding: 0`).
2. **Ẩn hoàn toàn các phần tử khác để tập trung tìm kiếm**:
   - Ẩn toàn bộ cụm bên trái: Nút Hamburger Menu (`☰`), Logo thương hiệu, Tiêu đề.
   - Ẩn toàn bộ cụm bên phải: Nút cờ ngôn ngữ, nút đổi Theme sáng/tối, nút Đăng nhập / Đăng ký.
3. **Nút Quay Về (Back / Hủy tìm kiếm) ở đầu bên trái**:
   - Đặt icon mũi tên quay về (`←`) ở vị trí đầu tiên bên trái thanh tìm kiếm.
   - Khi bấm nút quay về (hoặc nhấn Escape): Đóng thanh tìm kiếm dài, phục hồi lại đầy đủ Menu Hamburger, Logo và toàn bộ cụm nút bên phải.
4. **Tự động Focus**:
   - Khi vừa mở thanh tìm kiếm Mobile, input tự động nhận con trỏ nhập liệu (`input.focus()`).

---

## 2. Mã Nguồn Mẫu Hoàn Chỉnh (React / Next.js)

```tsx
import React, { useState, useRef } from "react";

export function ResponsiveHeaderSearch() {
  const [isMobileSearchOpen, setIsMobileSearchOpen] = useState(false);
  const [searchQuery, setSearchQuery] = useState("");
  const inputRef = useRef<HTMLInputElement>(null);

  const handleOpenMobileSearch = () => {
    setIsMobileSearchOpen(true);
    setTimeout(() => {
      inputRef.current?.focus();
    }, 100);
  };

  const handleCloseMobileSearch = () => {
    setIsMobileSearchOpen(false);
    setSearchQuery("");
  };

  return (
    <header className="main-header">
      {/* 1. CỤM TRÁI: Hamburger + Logo (Ẩn hoàn toàn khi đang bật tìm kiếm Mobile) */}
      {!isMobileSearchOpen && (
        <div className="topbar__left">
          <button className="btn-hamburger" aria-label="Menu">☰</button>
          <a href="/" className="brand-link">
            <span className="brand-logo">FG</span>
            <span className="brand-name">FlixGate</span>
          </a>
        </div>
      )}

      {/* 2. CỤM GIỮA: Thanh tìm kiếm (Bung full 100% Header khi bật trên Mobile) */}
      <div className={`topbar__center ${isMobileSearchOpen ? "mobile-search-full" : ""}`}>
        <div className={`search-container ${isMobileSearchOpen ? "show" : ""}`}>
          {/* Nút Quay Về (Hủy tìm kiếm) ở đầu bên trái khi mở Mobile */}
          {isMobileSearchOpen && (
            <button
              type="button"
              className="search-back-btn"
              onClick={handleCloseMobileSearch}
              title="Quay lại"
              aria-label="Back"
            >
              <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2.2" strokeLinecap="round" strokeLinejoin="round">
                <line x1="19" y1="12" x2="5" y2="12"></line>
                <polyline points="12 19 5 12 12 5"></polyline>
              </svg>
            </button>
          )}

          <div className="search-input-wrapper">
            {/* Icon tương tác khi focus (Desktop) */}
            {!isMobileSearchOpen && (
              <span className="search-focus-icon">
                <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round">
                  <circle cx="12" cy="12" r="10"></circle>
                  <line x1="2" y1="12" x2="22" y2="12"></line>
                  <path d="M12 2a15.3 15.3 0 0 1 4 10 15.3 15.3 0 0 1-4 10 15.3 15.3 0 0 1-4-10 15.3 15.3 0 0 1 4-10z"></path>
                </svg>
              </span>
            )}

            <input
              ref={inputRef}
              type="text"
              value={searchQuery}
              onChange={(e) => setSearchQuery(e.target.value)}
              placeholder="Tìm kiếm phim, video..."
              className="search-input"
            />
          </div>

          {/* Nút Kính lúp kích hoạt tìm kiếm */}
          <button type="submit" className="search-btn" aria-label="Search">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2.2" strokeLinecap="round" strokeLinejoin="round">
              <circle cx="11" cy="11" r="8"></circle>
              <line x1="21" y1="21" x2="16.65" y2="16.65"></line>
            </svg>
          </button>
        </div>
      </div>

      {/* 3. CỤM PHẢI: Nút kính lúp Mobile & Các nút tác vụ (Ẩn hoàn toàn khi đang bật tìm kiếm Mobile) */}
      {!isMobileSearchOpen && (
        <div className="topbar__right">
          {/* Nút kính lúp Mobile: Chỉ hiển thị khi <= 520px */}
          <button
            type="button"
            className="btn-icon-action mobile-search-btn"
            onClick={handleOpenMobileSearch}
            title="Tìm kiếm"
            aria-label="Toggle search"
          >
            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round">
              <circle cx="11" cy="11" r="8"></circle>
              <line x1="21" y1="21" x2="16.65" y2="16.65"></line>
            </svg>
          </button>

          {/* Nút Ngôn ngữ, Theme, Đăng nhập... */}
          <button className="btn-icon-action">🌐</button>
          <button className="btn-primary">Đăng nhập</button>
        </div>
      )}
    </header>
  );
}
```

---

## 3. Mã Nguồn CSS Chuyên Sâu (Zero-Dependency)

```css
/* ========================================================
   HEADER & TOPBAR LAYOUT
   ======================================================== */
.main-header {
  height: 60px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 16px;
  background: var(--bg-secondary, #1a1d24);
  border-bottom: 1px solid var(--border-color, #2a2e39);
  position: sticky;
  top: 0;
  z-index: 100;
}

.topbar__left,
.topbar__right {
  display: flex;
  align-items: center;
  gap: 12px;
  flex-shrink: 0;
}

.topbar__center {
  flex: 1 1 auto;
  display: flex;
  justify-content: center;
  padding: 0 16px;
}

/* ========================================================
   THANH TÌM KIẾM DẠNG VIÊN THUỐC (DESKTOP)
   Tự động mở rộng linh hoạt theo kích thước Laptop / PC
   ======================================================== */
.search-container {
  display: flex;
  align-items: center;
  border-radius: 15px;
  border: 1px solid var(--border-color, #333846);
  background: var(--bg-tertiary, #222632);
  position: relative;
  overflow: hidden;
  transition: all 0.3s ease;
  width: 100%;
  max-width: 350px; /* Giới hạn tối đa 350px cân đối, vừa vặn trên laptop & PC */
}

.search-input-wrapper {
  position: relative;
  display: flex;
  align-items: center;
  flex: 1 1 auto;
  min-width: 0;
}

/* Micro-Interaction: Icon trượt mượt mà khi focus */
.search-focus-icon {
  position: absolute;
  left: 9px;
  top: 50%;
  transform: translateY(-50%) scale(0.6);
  opacity: 0;
  pointer-events: none;
  color: var(--accent-color, #00d26a);
  transition: opacity 0.35s ease, transform 0.35s cubic-bezier(0.4, 0, 0.2, 1);
  display: flex;
  align-items: center;
}

.search-input {
  width: 100%;
  min-width: 180px;
  padding: 7px 14px;
  background: transparent;
  border: none;
  outline: none;
  color: #fff;
  font-size: 14px;
  transition: padding-left 0.4s cubic-bezier(0.4, 0, 0.2, 1), width 0.3s ease;
}

.search-container:focus-within {
  border-color: var(--accent-color, #00d26a);
  box-shadow: 0 0 0 2px rgba(0, 210, 106, 0.2);
}

.search-container:focus-within .search-focus-icon {
  opacity: 1;
  transform: translateY(-50%) scale(1);
}

.search-container:focus-within .search-input {
  padding-left: 32px; /* Dịch chữ sang phải chừa chỗ cho icon */
}

.search-btn {
  border: none;
  border-left: 1px solid var(--border-color, #333846);
  padding: 6px 12px;
  background: transparent;
  color: var(--text-secondary, #9da4b0);
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background 0.2s, color 0.2s;
}

.search-btn:hover {
  background: rgba(255, 255, 255, 0.08);
  color: #fff;
}

/* Nút quay về (Back) bên trái */
.search-back-btn {
  background: transparent;
  border: none;
  color: var(--text-secondary, #9da4b0);
  padding: 6px 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  border-radius: 50%;
  margin-left: 4px;
  transition: color 0.2s, background 0.2s;
}

.search-back-btn:hover {
  color: #fff;
  background: rgba(255, 255, 255, 0.1);
}

/* ========================================================
   QUY CHUẨN ẨN / HIỆN THEO BREAKPOINT 520PX
   ======================================================== */

/* Mặc định trên Desktop & Màn hình > 520px: ẨN TUYỆT ĐỐI nút tìm kiếm mobile */
.mobile-search-btn {
  display: none !important;
}

@media (min-width: 521px) {
  .mobile-search-btn {
    display: none !important;
  }
}

/* Khi màn hình <= 520px: BẬT CHẾ ĐỘ MOBILE SEARCH */
@media (max-width: 520px) {
  /* 1. Mặc định ẩn thanh tìm kiếm dài */
  .search-container:not(.show) {
    display: none !important;
  }

  /* 2. Hiện nút kính lúp tròn ở góc phải */
  .mobile-search-btn {
    display: flex !important;
  }

  /* 3. Khi click mở tìm kiếm: Bung dài toàn bộ Header 100% */
  .topbar__center.mobile-search-full {
    flex: 1 1 100% !important;
    width: 100% !important;
    max-width: 100% !important;
    padding: 0 !important;
    margin: 0 !important;
  }

  .topbar__center.mobile-search-full .search-container.show {
    display: flex !important;
    width: 100% !important;
    max-width: 100% !important;
    border-radius: 9999px;
    z-index: 100;
    box-shadow: 0 4px 14px rgba(0, 0, 0, 0.25);
    animation: scaleUp 0.2s cubic-bezier(0.4, 0, 0.2, 1) forwards;
  }

  .topbar__center.mobile-search-full .search-input-wrapper {
    flex: 1 1 auto;
    width: 100%;
  }

  .topbar__center.mobile-search-full .search-input {
    width: 100% !important;
    max-width: 100% !important;
    padding-left: 8px !important;
  }
}

@keyframes scaleUp {
  from {
    opacity: 0;
    transform: scale(0.97);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}
```

---

## 4. Bảng Kiểm Tra Chất Lượng (Checklist Tái Sử Dụng)

| Tiêu chuẩn | Điều kiện kiểm thử | Kết quả mong đợi |
| :--- | :--- | :--- |
| **Màn hình > 520px** | Kéo resize trình duyệt > 520px | Nút kính lúp tròn mobile biến mất hoàn toàn (`display: none !important;`). Thanh tìm kiếm desktop ở giữa hiển thị chuẩn. |
| **Màn hình ≤ 520px** | Kéo resize trình duyệt ≤ 520px | Thanh input dài mặc định ẩn. Hiện nút icon kính lúp tròn trong cụm tác vụ bên phải. |
| **Click nút tìm kiếm Mobile** | Bấm vào icon kính lúp mobile | Logo, menu Hamburger và toàn bộ nút tác vụ bên phải ẩn hết. Thanh tìm kiếm bung dài 100% Header và input tự focus. |
| **Click nút mũi tên quay về** | Bấm vào icon `←` ở đầu bên trái thanh tìm kiếm | Đóng thanh tìm kiếm dài, phục hồi lại nguyên trạng Menu, Logo và cụm nút bên phải. |
| **Đa ngôn ngữ (i18n)** | Chuyển đổi ngôn ngữ hệ thống | Nút quay về hiển thị tooltip/aria-label tương ứng theo ngôn ngữ (`vi`, `en`, `zh`, `ja`, `ko`, `hmn`). |
