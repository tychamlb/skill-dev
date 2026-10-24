# Kỹ Năng Thiết Kế Bộ Lọc Thời Gian Phân Đoạn: Tất Cả - Năm - Tháng - Ngày (Segmented Time Filter & Dynamic Pickers)

Tài liệu nguyên lý thiết kế, cấu trúc giao diện responsive, khả năng hỗ trợ đa ngôn ngữ (VI / EN / Đa ngữ) và mã nguồn mẫu độc lập (Zero-Dependency) cho bộ lọc thời gian: **Tất cả - Năm - Tháng - Ngày (All / Year / Month / Day)** kết hợp ô chọn động (Dynamic Pickers). Tái sử dụng được cho mọi dự án React, Next.js, Vue hoặc Vanilla JS / HTML / CSS thuần.

---

## 1. Mô Tả & Nguyên Lý Hoạt Động (UI/UX)

Bộ lọc thời gian Tất cả / Năm / Tháng / Ngày là một thành phần giao diện người dùng dạng **Segmented Control / Tab Switcher** kết hợp **Dynamic Picker Inputs**:
- **Mục đích**: Cho phép người dùng chuyển đổi linh hoạt phạm vi thời gian hiển thị dữ liệu (thống kê doanh thu, biểu đồ lượt xem, lịch sử đăng nhập/phiên, log hệ thống, danh sách kiểm toán backup...).
- **4 Mức độ chi tiết (Granularity)**:
  1. **Tất cả (All)**: Không lọc theo mốc thời gian cụ thể, hiển thị toàn bộ dữ liệu từ trước đến nay. (`{ period: 'all', value: null }`).
  2. **Năm (Year)**: Hiển thị danh sách ô chọn Năm (Ví dụ: `2024`, `2025`, `2026`). (`{ period: 'year', value: '2026' }`).
  3. **Tháng (Month)**: Hiển thị ô chọn Tháng & Năm theo định dạng `YYYY-MM` (`<input type="month">`). (`{ period: 'month', value: '2026-10' }`).
  4. **Ngày (Day)**: Hiển thị ô chọn Ngày cụ thể theo định dạng `YYYY-MM-DD` (`<input type="date">`). (`{ period: 'day', value: '2026-10-17' }`).

---

## 2. Mã Nguồn Mẫu Hoàn Chỉnh (HTML + CSS + Vanilla JS)

### A. Mã HTML (Markup)
```html
<div class="time-filter-wrapper">
  <!-- Segmented Tabs Selector -->
  <div class="filter-tabs" role="tablist" aria-label="Bộ lọc thời gian">
    <button type="button" class="tab-btn active" data-period="all" role="tab" aria-selected="true">
      <span class="lang-vi">Tất cả</span>
      <span class="lang-en">All</span>
    </button>
    <button type="button" class="tab-btn" data-period="year" role="tab" aria-selected="false">
      <span class="lang-vi">Năm</span>
      <span class="lang-en">Year</span>
    </button>
    <button type="button" class="tab-btn" data-period="month" role="tab" aria-selected="false">
      <span class="lang-vi">Tháng</span>
      <span class="lang-en">Month</span>
    </button>
    <button type="button" class="tab-btn" data-period="day" role="tab" aria-selected="false">
      <span class="lang-vi">Ngày</span>
      <span class="lang-en">Day</span>
    </button>
  </div>

  <!-- Dynamic Picker Inputs -->
  <div class="filter-inputs">
    <!-- Picker cho Năm -->
    <div class="picker-group hidden" id="picker-year">
      <select id="select-year" class="form-input">
        <option value="2024">2024</option>
        <option value="2025">2025</option>
        <option value="2026" selected>2026</option>
      </select>
    </div>

    <!-- Picker cho Tháng -->
    <div class="picker-group hidden" id="picker-month">
      <input type="month" id="select-month" class="form-input" value="2026-10">
    </div>

    <!-- Picker cho Ngày -->
    <div class="picker-group hidden" id="picker-day">
      <input type="date" id="select-date" class="form-input" value="2026-10-17">
    </div>
  </div>
</div>
```

---

### B. Mã CSS (Styling & Dark Mode)
```css
:root {
  --primary-color: #3b82f6;
  --primary-hover: #2563eb;
  --bg-tab-container: #f1f5f9;
  --bg-card: #ffffff;
  --text-main: #1e293b;
  --text-muted: #64748b;
  --border-color: #cbd5e1;
  --radius: 8px;
  --transition: all 0.2s ease-in-out;
}

/* Dark mode support */
[data-theme="dark"] {
  --bg-tab-container: #1e293b;
  --bg-card: #0f172a;
  --text-main: #f8fafc;
  --text-muted: #94a3b8;
  --border-color: #334155;
}

.time-filter-wrapper {
  display: inline-flex;
  align-items: center;
  gap: 12px;
  font-family: inherit;
  flex-wrap: wrap;
}

/* Tab Container (Pill / Segmented control) */
.filter-tabs {
  display: inline-flex;
  background-color: var(--bg-tab-container);
  padding: 4px;
  border-radius: var(--radius);
  border: 1px solid var(--border-color);
}

.tab-btn {
  padding: 6px 14px;
  font-size: 13px;
  font-weight: 500;
  border: none;
  background: transparent;
  color: var(--text-muted);
  border-radius: calc(var(--radius) - 2px);
  cursor: pointer;
  transition: var(--transition);
  user-select: none;
}

.tab-btn:hover {
  color: var(--text-main);
}

.tab-btn.active {
  background-color: var(--bg-card);
  color: var(--primary-color);
  font-weight: 600;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

/* Input / Pickers */
.filter-inputs {
  display: inline-flex;
  align-items: center;
}

.picker-group {
  display: flex;
  align-items: center;
}

.picker-group.hidden {
  display: none !important;
}

.form-input {
  padding: 6px 10px;
  font-size: 13px;
  border: 1px solid var(--border-color);
  border-radius: var(--radius);
  background-color: var(--bg-card);
  color: var(--text-main);
  outline: none;
  transition: var(--transition);
}

.form-input:focus {
  border-color: var(--primary-color);
  box-shadow: 0 0 0 2px rgba(59, 130, 246, 0.2);
}
```

---

### C. Mã JavaScript (Behavior & Logic Class)
```javascript
class TimeFilter {
  constructor(container, onFilterChange) {
    this.container = typeof container === 'string' ? document.querySelector(container) : container;
    this.onFilterChange = onFilterChange;
    this.currentPeriod = 'all';

    this.tabButtons = this.container.querySelectorAll('.tab-btn');
    this.pickerGroups = {
      year: this.container.querySelector('#picker-year'),
      month: this.container.querySelector('#picker-month'),
      day: this.container.querySelector('#picker-day')
    };

    this.init();
  }

  init() {
    // 1. Lắng nghe click chuyển tab
    this.tabButtons.forEach(btn => {
      btn.addEventListener('click', (e) => {
        const period = btn.dataset.period;
        this.setPeriod(period);
      });
    });

    // 2. Lắng nghe thay đổi giá trị của các input picker
    this.container.querySelector('#select-year')?.addEventListener('change', () => this.triggerChange());
    this.container.querySelector('#select-month')?.addEventListener('change', () => this.triggerChange());
    this.container.querySelector('#select-date')?.addEventListener('change', () => this.triggerChange());
  }

  setPeriod(period) {
    this.currentPeriod = period;

    // Cập nhật trạng thái active của tab
    this.tabButtons.forEach(btn => {
      const isActive = btn.dataset.period === period;
      btn.classList.toggle('active', isActive);
      btn.setAttribute('aria-selected', isActive ? 'true' : 'false');
    });

    // Ẩn/Hiện picker tương ứng
    Object.keys(this.pickerGroups).forEach(key => {
      const group = this.pickerGroups[key];
      if (group) {
        group.classList.toggle('hidden', key !== period);
      }
    });

    this.triggerChange();
  }

  getValue() {
    let value = null;
    if (this.currentPeriod === 'year') {
      value = this.container.querySelector('#select-year')?.value;
    } else if (this.currentPeriod === 'month') {
      value = this.container.querySelector('#select-month')?.value; // "YYYY-MM"
    } else if (this.currentPeriod === 'day') {
      value = this.container.querySelector('#select-date')?.value;  // "YYYY-MM-DD"
    }

    return {
      period: this.currentPeriod, // 'all' | 'year' | 'month' | 'day'
      value: value
    };
  }

  triggerChange() {
    const filterData = this.getValue();
    if (typeof this.onFilterChange === 'function') {
      this.onFilterChange(filterData);
    }
  }
}

// Cách khởi tạo & sử dụng:
document.addEventListener('DOMContentLoaded', () => {
  const filter = new TimeFilter('.time-filter-wrapper', (data) => {
    console.log('Bộ lọc đã thay đổi:', data);
    // Ví dụ output:
    // { period: 'all', value: null }
    // { period: 'year', value: '2026' }
    // { period: 'month', value: '2026-10' }
    // { period: 'day', value: '2026-10-17' }

    // Gọi API nạp dữ liệu tương ứng:
    // fetchStats(data.period, data.value);
  });
});
```

---

## 3. Bản Chuyển Đổi Sang React Component Chuẩn SPA FlixGate

Để tái sử dụng ngay trong các view Admin (`UserManagement`, `CloudVaultManager`, `AuditLogs`, `PosterStudio`...):

```jsx
import React, { useState } from 'react';

export function TimeFilterControl({ t, onFilterChange, initialPeriod = 'all' }) {
  const [period, setPeriod] = useState(initialPeriod);
  const [yearVal, setYearVal] = useState('2026');
  const [monthVal, setMonthVal] = useState('2026-10');
  const [dayVal, setDayVal] = useState('2026-10-17');

  const handlePeriodClick = (p) => {
    setPeriod(p);
    let val = null;
    if (p === 'year') val = yearVal;
    if (p === 'month') val = monthVal;
    if (p === 'day') val = dayVal;
    if (onFilterChange) onFilterChange({ period: p, value: val });
  };

  const handleValueChange = (p, val) => {
    if (p === 'year') setYearVal(val);
    if (p === 'month') setMonthVal(val);
    if (p === 'day') setDayVal(val);
    if (onFilterChange) onFilterChange({ period: p, value: val });
  };

  return (
    <div className="time-filter-wrapper">
      <div className="filter-tabs" role="tablist">
        <button
          type="button"
          className={`tab-btn ${period === 'all' ? 'active' : ''}`}
          onClick={() => handlePeriodClick('all')}
        >
          {t ? t('filter_time_all') : 'Tất cả'}
        </button>
        <button
          type="button"
          className={`tab-btn ${period === 'year' ? 'active' : ''}`}
          onClick={() => handlePeriodClick('year')}
        >
          {t ? t('filter_time_year') : 'Năm'}
        </button>
        <button
          type="button"
          className={`tab-btn ${period === 'month' ? 'active' : ''}`}
          onClick={() => handlePeriodClick('month')}
        >
          {t ? t('filter_time_month') : 'Tháng'}
        </button>
        <button
          type="button"
          className={`tab-btn ${period === 'day' ? 'active' : ''}`}
          onClick={() => handlePeriodClick('day')}
        >
          {t ? t('filter_time_day') : 'Ngày'}
        </button>
      </div>

      <div className="filter-inputs">
        {period === 'year' && (
          <div className="picker-group">
            <select
              className="form-input"
              value={yearVal}
              onChange={(e) => handleValueChange('year', e.target.value)}
            >
              <option value="2024">2024</option>
              <option value="2025">2025</option>
              <option value="2026">2026</option>
            </select>
          </div>
        )}

        {period === 'month' && (
          <div className="picker-group">
            <input
              type="month"
              className="form-input"
              value={monthVal}
              onChange={(e) => handleValueChange('month', e.target.value)}
            />
          </div>
        )}

        {period === 'day' && (
          <div className="picker-group">
            <input
              type="date"
              className="form-input"
              value={dayVal}
              onChange={(e) => handleValueChange('day', e.target.value)}
            />
          </div>
        )}
      </div>
    </div>
  );
}
```

---

## 4. Xử Lý Truy Vấn Backend & MongoDB Pipeline

Khi nhận tham số từ bộ lọc `{ period, value }`, phía Backend xử lý query date range chuẩn:

```javascript
function buildDateFilter(period, value, dateField = 'createdAt') {
  if (!period || period === 'all' || !value) {
    return {}; // Không giới hạn thời gian
  }

  if (period === 'year') {
    const year = parseInt(value);
    const start = new Date(`${year}-01-01T00:00:00.000Z`);
    const end = new Date(`${year + 1}-01-01T00:00:00.000Z`);
    return { [dateField]: { $gte: start, $lt: end } };
  }

  if (period === 'month') {
    const [year, month] = value.split('-').map(Number);
    const start = new Date(Date.UTC(year, month - 1, 1));
    const end = new Date(Date.UTC(year, month, 1));
    return { [dateField]: { $gte: start, $lt: end } };
  }

  if (period === 'day') {
    const start = new Date(`${value}T00:00:00.000Z`);
    const end = new Date(`${value}T23:59:59.999Z`);
    return { [dateField]: { $gte: start, $lte: end } };
  }

  return {};
}
```

---

## 5. Checklist Tái Sử Dụng & Nghiệm Thu

| Tiêu chuẩn | Kiểm tra | Kết quả mong đợi |
| :--- | :--- | :--- |
| **Segmented Pill Tabs** | Click chuyển tab | Hiệu ứng highlight tab chuyển động mượt mà, đổi màu text và shadow nổi bật. |
| **Ẩn/Hiện Picker** | Chọn "Năm" / "Tháng" / "Ngày" | Chỉ hiện đúng picker tương ứng, ẩn các picker còn lại. Khi chọn "Tất cả" ẩn toàn bộ picker. |
| **Đa ngôn ngữ** | Chuyển đổi ngôn ngữ hệ thống | Nhãn tab hiển thị chuẩn ngữ cảnh (VI / EN / ZH / JA / KO / HMN). |
| **Dark Theme** | Chuyển sang Dark Theme | Màu nền `--bg-tab-container` và input tương thích hoàn hảo không bị lóa mắt. |
