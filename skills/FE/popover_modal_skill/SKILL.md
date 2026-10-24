# Kỹ Năng Thiết Kế Modal Popover Native (HTML5 Popover API) & Tối Ưu Chiều Cao Không Tràn Thanh Cuộn

Tài liệu nguyên lý thiết kế, ưu nhược điểm kỹ thuật, quy chuẩn bố cục Flexbox công thái học và mã nguồn mẫu độc lập (Zero-Dependency) tích hợp Modal bằng **Native HTML5 Popover API**. Sử dụng được cho cả HTML tĩnh, Vanilla JS, React, Next.js và Vue.

---

## 1. Nguyên Lý & Kỹ Thuật Cốt Lõi

### A. Cơ chế hoạt động của Popover API
- **Không cần JavaScript để điều khiển mở/đóng cơ bản**:
  - Nút bấm mở modal gắn thuộc tính `popovertarget="modalLogin"`.
  - Khung modal được khai báo với thuộc tính `popover` (hoặc `popover="auto"`).
- **Thuộc tính tự động trên Top Layer**:
  - Phần tử modal tự động được đưa lên tầng hiển thị cao nhất của trình duyệt (**Top Layer**), không bao giờ bị đè bởi bất kỳ `z-index` nào khác trong trang.
- **Light Dismiss mặc định**:
  - Tự động đóng modal khi nhấn phím `ESC`.
  - Tự động đóng khi người dùng click chuột ra ngoài vùng modal (**click-outside / backdrop dismissal**).
- **Pseudo-element `::backdrop`**:
  - Dùng để làm mờ (blur) hoặc tối nền hậu cảnh phía sau modal bằng CSS thuần mà không cần tạo thêm thẻ div `.overlay` thủ công.

---

### B. Nguyên lý Khử Thanh Cuộn (Zero Vertical Scrollbar)
Để khung modal vừa khít nội dung đăng nhập mà **không bao giờ xuất hiện thanh cuộn bên**:
1. **Thiết lập chiều cao tự co giãn**: Sử dụng `height: auto; max-height: 94vh; overflow-y: auto;`.
2. **Tối ưu Padding & Spacing (Compact Spacing)**:
   - Padding khu vực nội dung `.modal-content-area`: Tinh chỉnh thành `16px 22px 20px`.
   - Khoảng cách các trường nhập liệu (`gap` trong `.auth-form`): Giữ ở mức `10px`, khoảng cách `label - input` là `4px`.
   - Padding của các ô nhập và nút bấm: Rút gọn xuống `8px - 9px`.
   - Khoảng cách đường kẻ phân cách OAuth: Giảm `margin` xuống `12px` và `gap` giữa các nút mạng xã hội là `7px`.

---

### C. Vị trí Nút Đóng `✕` (Flexbox Space-Between) & Hiệu Ứng Hover Đỏ Xoay 90°
- **Vị trí**:
  - Nằm bên trong thanh tiêu đề `.login-header` / `.modal-header-tabs` với cấu trúc `display: flex; justify-content: space-between; align-items: center;`.
  - Nút đóng tự động căn lề sang mép phải ngang hàng với tiêu đề tab (`margin-left: auto;`).
- **Ngoại hình ban đầu (Solid & Crisp)**:
  - Giữ nguyên màu nền card `background: var(--bg-tertiary)` và viền `1px solid var(--border-color)` (không dùng nền trong suốt để tránh mất nhận diện nút).
  - Kích thước: `30px - 32px`, bo tròn `50%`.
- **Hiệu ứng Hover & Active**:
  - Khi hover: Chữ đổi màu đỏ tươi `#ff0000`, viền đổi sang đỏ `#ef4444`, nền phớt đỏ mờ `rgba(239, 68, 68, 0.12)`, và xoay 90 độ mượt mà (`transform: rotate(90deg)`).
  - Khi active (click): Thu nhỏ nhẹ `transform: scale(0.9) rotate(90deg)` mang lại phản hồi cơ học chân thực.

---

## 2. Mã Nguồn Mẫu Hoàn Chỉnh

### A. Mã HTML Skeleton (Bố cục chuẩn Flexbox)

```html
<!-- 1. Nút kích hoạt mở modal -->
<button type="button" class="login-btn" popovertarget="modalLogin">
  Đăng Nhập
</button>

<!-- 2. Khung Modal Popover trên Top Layer -->
<div class="container_login">
  <div class="container-login modal-card" id="modalLogin" popover="auto">
    
    <!-- Tiêu đề Header Modal: Flexbox space-between -->
    <div class="login-header modal-header-tabs">
      <div class="modal-tabs-group">
        <h2 class="login-title">Đăng Nhập Tài Khoản</h2>
      </div>
      
      <!-- Nút đóng nhanh đối xứng góc phải -->
      <button 
        type="button" 
        class="popover-close-btn close-btn" 
        popovertarget="modalLogin" 
        popovertargetaction="hide"
        aria-label="Close"
      >
        ✕
      </button>
    </div>

    <!-- Thân Modal chứa form nhập liệu vừa vặn (Không cuộn) -->
    <div class="modal-content-area">
      <form class="auth-form">
        <div class="form-group">
          <label>Email</label>
          <input type="email" placeholder="name@example.com" required />
        </div>
        
        <div class="form-group">
          <label>Mật khẩu</label>
          <input type="password" placeholder="••••••••" required />
        </div>
        
        <button type="submit" class="btn-form-submit">Đăng Nhập</button>
      </form>

      <!-- Phân cách & OAuth -->
      <div class="oauth-divider"><span>Hoặc</span></div>
      <div class="oauth-btn-group">
        <button type="button" class="btn-oauth">Google</button>
      </div>
    </div>

  </div>
</div>
```

---

### B. Mã CSS Hoàn Chỉnh (Zero Vertical Scrollbar & Red Glow 90deg Hover)

```css
/* ========================================================
   KHUNG MODAL POPOVER CHUẨN TOP LAYER & KHỬ THANH CUỘN
   ======================================================== */
.modal-card,
.container-login {
  border-radius: 14px;
  width: 100%;
  max-width: clamp(340px, 92vw, 420px);
  height: auto;          /* Tự động co giãn theo nội dung, khử thanh cuộn */
  max-height: 94vh;
  overflow-y: auto;
  padding: 0;
  border: 1px solid var(--border-color, #2a2e39);
  background-color: var(--bg-secondary, #161922);
  color: var(--text-primary, #ffffff);
  box-shadow: 0 24px 60px rgba(0, 0, 0, 0.45);
  margin: auto;
  position: fixed;
  inset: 0;
}

/* Kích hoạt hiển thị khi Popover mở */
.modal-card:popover-open,
.container-login:popover-open {
  display: flex;
  flex-direction: column;
  animation: popoverFadeIn 0.25s cubic-bezier(0.16, 1, 0.3, 1);
}

/* Lớp phủ Backdrop làm mờ mượt mà phía sau */
.modal-card::backdrop,
.container-login::backdrop {
  background-color: rgba(0, 0, 0, 0.78);
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
}

/* ========================================================
   HEADER MODAL: FLEXBOX SPACE-BETWEEN
   ======================================================== */
.modal-header-tabs,
.login-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-bottom: 1px solid var(--border-color, #2a2e39);
  padding: 10px 14px 10px 20px;
}

.login-title {
  margin: 0;
  font-size: 16px;
  font-weight: 700;
}

/* ========================================================
   NÚT ĐÓNG: NỀN SOLID + HIỆU ỨNG HOVER ĐỎ XOAY 90 ĐỘ
   ======================================================== */
.popover-close-btn,
.close-btn,
.modal-close-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 30px;
  height: 30px;
  padding: 0;
  font-size: 14px;
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

/* Hover: Đổi chữ đỏ tươi, viền đỏ, nền phớt đỏ, xoay 90 độ */
.popover-close-btn:hover,
.close-btn:hover,
.modal-close-btn:hover {
  color: #ff0000;
  border-color: #ef4444;
  background-color: rgba(239, 68, 68, 0.12);
  transform: rotate(90deg);
  box-shadow: 0 2px 10px rgba(239, 68, 68, 0.25);
}

/* Click Active: Thu nhỏ nhẹ */
.popover-close-btn:active,
.close-btn:active,
.modal-close-btn:active {
  transform: scale(0.9) rotate(90deg);
}

/* ========================================================
   NỘI DUNG FORM NHẬP LIỆU GỌN GÀNG (KHÔNG TRÀN THANH CUỘN)
   ======================================================== */
.modal-content-area {
  padding: 16px 22px 20px;
}

.auth-form {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.form-group label {
  font-size: 11px;
  font-weight: 600;
  color: var(--text-secondary, #94a3b8);
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.form-group input {
  background: var(--bg-tertiary, #202532);
  border: 1px solid var(--border-color, #2a2e39);
  padding: 8px 12px;
  border-radius: 8px;
  color: #fff;
  font-size: 13px;
  outline: none;
  transition: border-color 0.2s;
}

.form-group input:focus {
  border-color: #ef4444;
}

.btn-form-submit {
  margin-top: 2px;
  padding: 9px 14px;
  background: var(--accent-color, #e50914);
  color: #fff;
  border: none;
  border-radius: 8px;
  font-size: 13px;
  font-weight: 700;
  cursor: pointer;
  transition: opacity 0.2s;
}

.oauth-divider {
  display: flex;
  align-items: center;
  margin: 12px 0 10px;
  gap: 10px;
}

.oauth-divider::before,
.oauth-divider::after {
  content: "";
  flex: 1;
  height: 1px;
  background: var(--border-color, #2a2e39);
}

.oauth-divider span {
  font-size: 11px;
  color: var(--text-muted, #94a3b8);
}

.oauth-btn-group {
  display: flex;
  flex-direction: column;
  gap: 7px;
}

.btn-oauth {
  padding: 8px 12px;
  background: var(--bg-tertiary, #202532);
  border: 1px solid var(--border-color, #2a2e39);
  border-radius: 8px;
  color: #fff;
  font-size: 13px;
  cursor: pointer;
}

@keyframes popoverFadeIn {
  from {
    opacity: 0;
    transform: scale(0.96) translateY(-6px);
  }
  to {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
}
```

---

## 3. Checklist Nghiệm Thu & Tái Sử Dụng

| Tiêu chuẩn | Điều kiện kiểm thử | Kết quả mong đợi |
| :--- | :--- | :--- |
| **Không thanh cuộn** | Mở modal trên màn hình laptop / PC | Modal tự co giãn chiều cao ôm trọn form đăng nhập, không có thanh cuộn thừa bên cạnh. |
| **Vị trí nút đóng** | Quan sát thanh Header của Modal | Nút đóng nằm đối xứng ở mép phải ngang hàng với tiêu đề nhờ `justify-content: space-between`. |
| **Nền & Viền nút đóng** | Trạng thái bình thường | Nút có màu nền card và viền bo tròn sắc nét, không bị chìm hay trong suốt. |
| **Hiệu ứng Hover** | Rê chuột vào nút đóng `✕` | Chữ đổi đỏ `#ff0000`, viền đổi đỏ, có bóng mờ và xoay mượt 90 độ. |
| **Light Dismiss** | Nhấn phím `ESC` hoặc click ra ngoài | Trình duyệt tự động đóng modal trên Top Layer mượt mà. |
