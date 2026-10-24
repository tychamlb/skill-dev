---
name: background_polling_silent_sync_skill
description: Kỹ thuật tích hợp cơ chế Polling chạy nền (setInterval 15000ms), khử hoàn toàn giật nháy màn hình (Zero-Flicker Background Silent Sync), tối ưu trải nghiệm người dùng và cô lập thông báo lỗi ngầm.
---

# KỸ NĂNG TÍCH HỢP POLLING CHẠY NỀN & KHỬ GIẬT NHÁY GIAO DIỆN (SILENT BACKGROUND POLLING & ZERO-FLICKER SYNC)

## 1. NGUYÊN LÝ THIẾT KẾ CỐT LÕI (CORE PRINCIPLES)

1. **Chu Kỳ Polling Tiêu Chuẩn 15 Giây (`15000ms`)**:
   - Khoảng thời gian lý tưởng để cập nhật liên tục trạng thái tài nguyên, hạn mức, tiến trình nền mà không gây nghẽn mạng hay quá tải CPU.
   - Luôn sử dụng `setInterval` trong hook `useEffect` và **BẮT BUỘC có hàm dọn dẹp `clearInterval`** để chống rò rỉ bộ nhớ (Memory Leak).

2. **Nguyên Lý "Cập Nhật Âm Thầm" (Silent Update Mode - Zero Spinner Flicker)**:
   - **Vấn đề cốt lõi gây phiền**: Khi polling ngầm, nếu kích hoạt spinner toàn màn hình ("Đang xử lý...") hoặc xóa trắng danh sách dữ liệu cũ, giao diện người dùng sẽ bị chớp nháy liên tục mỗi chu kỳ 15 giây, gây ức chế và làm gián đoạn tương tác chuột/bàn phím.
   - **Quy tắc giải quyết**: Phân tách rõ ràng giữa **Lần tải đầu tiên (Initial Load)** và **Đồng bộ ngầm (Silent Polling)** qua tham số boolean `isSilent = false`.

3. **Chỉ Hiện Spinner Ở Lần Tải Đầu Tiên (Initial Load Guard)**:
   - Điều kiện render spinner: **CHỈ HIỂN THỊ KHI ĐANG LOADING VÀ DỮ LIỆU HIỆN TẠI ĐANG RỖNG**.
   - Cấm dùng: `{loading ? <Spinner /> : <DataList />}` ➔ Gây chớp nháy mỗi 15s.
   - Bắt buộc dùng: `{loading && dataList.length === 0 ? <Spinner /> : <DataList />}` ➔ Giữ nguyên vẹn danh sách trên màn hình trong lúc dữ liệu mới đang được fetch ngầm.

4. **Cô Lập Thông Báo Lỗi Ngầm (Mute Silent Error Toasts)**:
   - Nếu trong một chu kỳ polling mạng bị chập chờn 1-2 giây, **tuyệt đối không bật thông báo đỏ popup/toast** làm phiền người dùng.
   - Chỉ hiển thị Toast lỗi khi người dùng chủ động bấm nút thao tác tay (`isSilent === false`).

---

## 2. KIẾN TRÚC CODE MẪU CHUẨN (STANDARD REACT PATTERN)

### Bước 1: Thiết kế Hàm Fetch Hỗ Trợ `isSilent`

```javascript
// useCallback để tránh recreate function gây re-render thừa
const loadResources = useCallback(async (isSilent = false) => {
  // Chỉ bật loading spinner nếu là lần tải chủ động hoặc chưa có dữ liệu
  if (!isSilent) {
    setLoading(true);
  }

  try {
    const res = await fetch('/api/admin/resources', { credentials: 'include' });
    const data = await res.json();
    if (res.ok) {
      setResources(data.resources || []);
    } else {
      // Chỉ hiện toast khi người dùng bấm tay, không hiện khi đang polling ngầm
      if (!isSilent) {
        showToast(data.message || t('status_error'), 'error');
      }
    }
  } catch (err) {
    // Không làm phiền màn hình khi polling ngầm gặp lỗi mạng thoáng qua
    if (!isSilent) {
      showToast(t('err_network'), 'error');
    }
  } finally {
    if (!isSilent) {
      setLoading(false);
    }
  }
}, [t, showToast]);
```

### Bước 2: Thiết lập useEffect Tải Lần Đầu & Polling 15 Giây

```javascript
// 1. Tải lần đầu tiên khi vào trang (isSilent = false để hiện spinner ban đầu)
useEffect(() => {
  loadResources(false);
}, [loadResources]);

// 2. Cơ chế Polling tự động chạy nền mỗi 15s (isSilent = true để đồng bộ êm ái)
useEffect(() => {
  const pollingTimer = setInterval(() => {
    loadResources(true);
  }, 15000);

  // Dọn dẹp an toàn khi chuyển trang hoặc đóng component
  return () => clearInterval(pollingTimer);
}, [loadResources]);
```

### Bước 3: Render Giao Diện Khử Hoàn Toàn Giật Nháy (Anti-Flicker Guard)

```jsx
{/* Lưới / Bảng dữ liệu */}
{loading && resources.length === 0 ? (
  <div className="loading-state-box">
    <div className="loading-spinner" />
    <span>{t('status_loading')}</span>
  </div>
) : (
  <div className="resources-grid">
    {resources.map((item) => (
      <ResourceCard key={item.id} item={item} />
    ))}
  </div>
)}
```

---

## 3. VÙNG ÁP DỤNG BẮT BUỘC (MANDATORY USE CASES)

Bất kỳ chức năng FE nào dưới đây **BẮT BUỘC** phải áp dụng skill này:
1. **Quản lý Cụm Đám Mây Đa Tầng (`CloudVaultManager`)**: Đồng bộ dung lượng lưu trữ, lượt đọc/ghi D1 SQL, hạn mức và trạng thái tài nguyên.
2. **Bảng Điều Khiển Giám Sát (`DashboardView`, `SystemMonitor`)**: Cập nhật chỉ số CPU, RAM, GPU, hạn mức Quota Kaggle/Cloud theo thời gian thực.
3. **Tiến Trình Sinh Giọng Nói / AI Worker (`VoiceStudioView`, `AiPlaygroundView`)**: Polling trạng thái kernel/job Kaggle từ `running` ➔ `complete` mà không giật nháy form nhập liệu.
4. **Bảng Quản Lý Dữ Liệu & Mô Hình (`DatasetManagerView`, `ModelHubView`)**: Đồng bộ danh sách file và phiên bản tự động.
