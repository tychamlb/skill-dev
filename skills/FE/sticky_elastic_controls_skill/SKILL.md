# Kỹ Năng Thanh Điều Khiển Đa Năng Co Giãn & Dính Cuộn (.main_controls / Sticky Elastic Control Bar)

Tài liệu nguyên lý thiết kế, cấu trúc giao diện co giãn theo thiết bị (Flexbox Wrap + Elastic Expand), hiệu ứng cuộn dính công thái học (`position: sticky`), nút điều khiển đa dạng (`.menu_btn`) và thanh tìm kiếm tự động giãn đều (`.search_bar`). Tái sử dụng độc lập (Zero-Dependency) cho mọi dự án React, Vue hoặc HTML/CSS thuần.

---

### 1. Nguyên Lý & Giải Pháp Kỹ Thuật Cốt Lõi

#### A. Tự động to/nhỏ phù hợp thiết bị & lấp đầy trọn vẹn 100% không gian (Elastic Full-Span)
- Sử dụng `display: flex; flex-wrap: wrap; align-items: stretch; width: 100%; box-sizing: border-box; gap: 8px;`.
- **Cơ chế co giãn chiếm trọn không gian**:
  - Mỗi `.menu_btn` được thiết lập `flex: 1 1 160px; text-align: center; justify-content: center;`.
  - Trên màn hình rộng (Desktop): Tất cả các nút tự động dãn đều theo tỷ lệ bằng nhau để **chiếm trọn 100% bề ngang thanh điều khiển**, tuyệt đối không bị thừa khoảng trống vô nghĩa bên phải.
  - Khi thu hẹp (Tablet/Mobile): Các nút tự động chia cột cân đối (ví dụ 2x2 hoặc 3x2) và mỗi nút vẫn dãn đều lấp kín dòng chứa nó, trên mobile nhỏ thì chiếm full dòng (`flex: 1 1 100%`), không bao giờ bị tình trạng lệch "2 ngang 3 dọc" xấu xí.

#### B. Cơ chế cuộn dính đẩy dạt (Sticky Scroll Push & Overflow Gotcha)
1. **Nguyên lý hoạt động của `position: sticky; top: ...`**:
   - `position: sticky` hoạt động như một hybrid giữa `relative` và `fixed`. Khi phần tử nằm trong khung nhìn bình thường thì nó hành xử như `relative`. Khi mép trên chạm mốc `top`, nó sẽ "dính chặt" vào viewport trong suốt phạm vi của **container cha trực tiếp**.
   - **Khi nào nó bị đẩy mất đi?**: Nó **chỉ trượt đi** khi toàn bộ khối cha bao quanh nó bị cuộn hết qua, hoặc **khối section kế tiếp ở dưới cuộn lên đụng vào nó**.

2. **CẠM BẪY CHẾT NGƯỜI (Khiến Sticky bị trôi mất dạng dù chưa có section dưới đẩy lên)**:
   - Theo chuẩn W3C CSS Specification: **Bất kỳ phần tử tổ tiên nào (parent/ancestor) có thuộc tính `overflow: hidden`, `overflow: auto`, `overflow: scroll` hoặc `overflow-x: hidden` sẽ vô hiệu hóa hoàn toàn cơ chế `position: sticky` của các phần tử con bên trong!**
   - Trình duyệt sẽ coi phần tử tổ tiên có `overflow` đó là scroll context riêng biệt (chứ không phải window), dẫn đến sticky không nhận được tín hiệu cuộn của trang web và trôi tuột theo dòng cuộn như `position: static`.
   - **Biện pháp khắc phục**: Đảm bảo toàn bộ các thẻ cha bao bọc (`html`, `body`, `.app-layout`, `.app-body-container`, `.main-wrapper`) **không chứa bất kỳ khai báo `overflow: hidden` hoặc `overflow-x: hidden` thừa nào** cắt ngang ngữ cảnh cuộn của trang.

3. **Tọa độ dính chuẩn xác (Chạm dừng 0px dưới Header)**:
   - Khi có header cố định (Top Navbar `position: sticky; top: 0;`), các thanh công cụ, bộ lọc dính cuộn (như `.main_controls`, `.am-toolbar`) **BẮT BUỘC chạm dừng cách mép dưới Header đúng 0px**:
     `position: sticky; top: var(--header-height, 60px);`
   - **Cơ chế đẩy dạt tự nhiên**: Thanh công cụ giữ cố định trên viewport cho đến khi có một section tương tự, cùng cấp ở phía dưới trồi lên chạm vào và đẩy nó đi.
   - `z-index` bắt buộc nằm trong khoảng an toàn (ví dụ `z-index: 40 - 50`), thấp hơn Header (`z-index: 110`) và thấp hơn Modal/Toast (`z-index: 10000+`).
   - Sử dụng hiệu ứng `backdrop-filter: blur(16px)` với độ mờ nền 90%+ để nội dung cuộn bên dưới không bị lẫn thị giác.

---

### 2. Cấu Trúc CSS Tiêu Chuẩn

```css
/* ========================================================
   1. KHUNG CHÍNH: TỰ CO GIÃN THEO THIẾT BỊ & STICKY CUỘN
   ======================================================== */
.main_controls {
  display: flex;
  flex-wrap: wrap;             /* Tự động xuống dòng khi màn hình nhỏ */
  align-items: center;
  gap: 8px;                    /* Khoảng cách đều giữa các nút */
  width: 100%;
  margin: 0 0 16px 0;
  padding: 10px 14px;
  background-color: var(--bg-card, #1e293b);
  border: 1px solid var(--border-color, #334155);
  border-radius: var(--radius-md, 8px);
  box-shadow: 0px 4px 12px rgba(0, 0, 0, 0.15);
  box-sizing: border-box;

  /* DÍNH TRÊN ĐẦU VÀ CHỈ BỊ ĐẨY ĐI KHI SECTION DƯỚI ĐẨY LÊN */
  position: sticky;
  top: 0;                      /* Cách đỉnh hoặc header */
  z-index: 50;                 /* Luôn nổi trên nội dung cuộn bên dưới */
  backdrop-filter: blur(12px);
}

/* ========================================================
   2. CÁC NÚT ĐIỀU KHIỂN (.menu_btn) & HIỆU ỨNG HOVER
   ======================================================== */
.menu_btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  padding: 8px 14px;
  height: 38px;
  background-color: var(--bg-tertiary, #0f172a);
  color: var(--text-primary, #f8fafc);
  border: 1px solid var(--border-color, #334155);
  border-radius: var(--radius-sm, 6px);
  font-size: 13px;
  font-weight: 500;
  cursor: pointer;
  outline: none;
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  white-space: nowrap;
  box-sizing: border-box;
}

/* Hiệu ứng khi Hover nút */
.menu_btn:hover {
  background-color: rgba(59, 130, 246, 0.15);
  color: var(--accent-color, #3b82f6);
  border-color: var(--accent-color, #3b82f6);
  transform: translateY(-2px);              /* Nổi nhẹ lên */
  box-shadow: 0 4px 10px rgba(59, 130, 246, 0.2);
}

/* Trạng thái khi nút đang active/được chọn */
.menu_btn.active {
  background-color: var(--accent-color, #3b82f6);
  color: #ffffff;
  border-color: var(--accent-color, #3b82f6);
  box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.2);
}

.menu_btn.btn-cta-add {
  background: linear-gradient(135deg, #3b82f6 0%, #8b5cf6 100%);
  color: #ffffff;
  border: none;
  font-weight: 600;
  box-shadow: 0 4px 12px rgba(59, 130, 246, 0.3);
}

.menu_btn.btn-cta-add:hover {
  background: linear-gradient(135deg, #2563eb 0%, #7c3aed 100%);
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(59, 130, 246, 0.45);
}

.menu_btn:active {
  transform: translateY(0);
}

/* ========================================================
   3. THANH TÌM KIẾM (.search_bar) CO GIÃN THÔNG MINH
   ======================================================== */
.main_controls .search_bar {
  position: relative;
  flex-grow: 1;                 /* Tự động kéo dài chiếm hết khoảng trống còn lại */
  min-width: 180px;
  padding: 0;
  border-radius: 20px;
  overflow: hidden;
  height: 38px;
}

.main_controls .search_bar input {
  width: 100%;
  height: 100%;
  border: 1px solid var(--border-color, #334155);
  border-radius: 20px;
  padding: 8px 15px 8px 36px;   /* Chừa lề trái 36px cho icon kính lúp */
  font-size: 13px;
  outline: none;
  box-sizing: border-box;
  transition: all 0.3s ease;
  background-color: var(--bg-tertiary, #0f172a);
  color: var(--text-primary, #f8fafc);
}

.main_controls .search_bar input:hover {
  border-color: var(--accent-color, #3b82f6);
}

.main_controls .search_bar input:focus {
  border-color: var(--accent-color, #3b82f6);
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.25);
  background-color: var(--bg-card, #1e293b);
}

.main_controls .search_bar i,
.main_controls .search_bar span.search-icon {
  position: absolute;
  left: 12px;
  top: 50%;
  transform: translateY(-50%);
  color: var(--text-muted, #94a3b8);
  pointer-events: none;
  font-size: 14px;
  transition: color 0.3s ease;
}

.main_controls .search_bar:focus-within i,
.main_controls .search_bar:focus-within span.search-icon {
  color: var(--accent-color, #3b82f6);
}

/* ========================================================
   4. RESPONSIVE THEO KÍCH THƯỚC MÀN HÌNH (MOBILE / TABLET)
   ======================================================== */
@media (max-width: 600px) {
  .main_controls {
    gap: 6px;
    padding: 8px;
  }

  .menu_btn {
    padding: 6px 10px;
    height: 34px;
    font-size: 12px;
  }

  .main_controls .search_bar {
    order: 10;                /* Đưa thanh tìm kiếm xuống hàng cuối cùng */
    width: 100%;              /* Tràn đầy 100% chiều ngang màn hình điện thoại */
    min-width: 100%;
    margin-top: 4px;
  }
}
```

---

### 3. Ví Dụ Tích Hợp Chuẩn 2 Khối Cùng Cấp (Sticky Push & Dynamic Count)

```jsx
{/* Khối Cùng Cấp 1 (Banner Dính Cuộn): Sẽ dính trước và bị khối controls dưới đẩy trượt lên khi cuộn */}
<div className="cloud-header-banner">
  <div>
    <div className="cloud-header-title">
      <span>☁️</span> {t('cloud_title')}
    </div>
    <div className="cloud-header-desc">{t('cloud_subtitle')}</div>
  </div>
</div>

{/* Khối Cùng Cấp 2 (.main_controls): 5 nút co giãn đàn hồi, cuộn lên chạm và đẩy khối 1 */}
<div className="main_controls">
  {/* Số lượng tổng (count) CHỈ hiển thị tại nút đang được chọn (active) */}
  <button
    type="button"
    className={`menu_btn ${filterType === 'all' ? 'active' : ''}`}
    onClick={() => setFilterType('all')}
  >
    🌐 {t('cloud_tab_all')} {filterType === 'all' && `(${totalCount})`}
  </button>
  <button
    type="button"
    className={`menu_btn ${filterType === 'mongodb' ? 'active' : ''}`}
    onClick={() => setFilterType('mongodb')}
  >
    🍃 {t('cloud_tab_mongo')} {filterType === 'mongodb' && `(${mongoCount})`}
  </button>
  <button
    type="button"
    className={`menu_btn ${filterType === 'bucket' ? 'active' : ''}`}
    onClick={() => setFilterType('bucket')}
  >
    🪣 {t('cloud_tab_bucket')} {filterType === 'bucket' && `(${bucketCount})`}
  </button>
  <button
    type="button"
    className={`menu_btn ${filterType === 'drive' ? 'active' : ''}`}
    onClick={() => setFilterType('drive')}
  >
    📁 {t('cloud_tab_drive')} {filterType === 'drive' && `(${driveCount})`}
  </button>

  {/* Nút chính Action Thêm Mới */}
  <button 
    type="button" 
    className="menu_btn btn-cta-add" 
    popovertarget="modalAddCloud"
    onClick={() => setIsAddModalOpen(true)}
  >
    ➕ {t('cloud_btn_add')}
  </button>
</div>
```

