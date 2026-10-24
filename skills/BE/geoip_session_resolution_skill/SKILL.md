---
name: geoip_session_resolution_skill
description: Quy chuẩn kiến trúc & kỹ thuật nhận diện GeoIP phiên đăng nhập, Egress Host IP cho Local/Intranet, Chuyển đổi mã ISO-2 sang Flag Icon chuẩn, và Phân cấp địa chỉ chi tiết.
---

# Kỹ thuật Định vị GeoIP & Quản trị Phiên Thiết bị (GeoIP Session Resolution & Flag Architecture)

Tài liệu chuẩn hóa kiến trúc giải quyết bài toán:
1. **Localhost / Intranet Egress Detection**: Nhận diện quốc gia & hiển thị cờ thật ngay cả khi người dùng đang chạy trên `::1`, `127.0.0.1` hoặc mạng LAN nội bộ.
2. **Kỹ thuật Chuyển đổi ISO-2 sang Lá cờ (Flag)**: Thuật toán Unicode Regional Indicator Symbols kết hợp Vector SVG Icon cao cấp đồng bộ với nút chọn ngôn ngữ.
3. **Phân cấp Định vị Địa chỉ (GeoIP ISP vs Client Geolocation GPS/Wi-Fi)**: Bản chất kỹ thuật giữa địa chỉ mạng cấp ISP (Quận/Huyện, Tỉnh/TP, Quốc gia, Mã bưu chính) và địa chỉ thực tế (Số nhà, Tên đường).
4. **2-Tier Hybrid Cache (0ms)**: Tối ưu hiệu năng, giảm thiểu 100% chi phí gọi API ngoài lặp lại.

---

## 1. Bản chất Kỹ thuật: GeoIP vs Định vị Thiết bị (GPS / Wi-Fi)

| Đặc tính | GeoIP (Phía Máy chủ / Server-side) | Client-side Geolocation (Trình duyệt / GPS) |
| :--- | :--- | :--- |
| **Cơ chế thu thập** | Đọc IP từ Header HTTP (`x-forwarded-for`, `req.ip`, TCP Socket). | Gọi API HTML5 `navigator.geolocation.getCurrentPosition()`. |
| **Độ chính xác** | **Cấp Quốc gia, Tỉnh/TP, Quận/Huyện, Mã bưu cục (Zip), Nhà mạng (ISP)**. | **Cấp Tọa độ vi mô (Số nhà, tên ngõ, tên đường) sai số 5m - 20m**. |
| **Yêu cầu kết nối mạng** | **Bắt buộc có kết nối Internet** (để tra cứu định tuyến IP ASN hoặc tải CSDL GeoIP MaxMind/IP2Location). | **Bắt buộc có kết nối mạng / GPS** (để tam giác hóa trạm BTS di động, Wi-Fi BSSID và Reverse Geocoding qua bản đồ). |
| **Quyền riêng tư (Permission)** | **Không cần người dùng cấp quyền** (Backend tự động xử lý ngầm trong 0.2ms). | **Bắt buộc người dùng bấm "Allow / Cho phép"** trên pop-up quyền của trình duyệt. |
| **Ứng dụng phù hợp** | Ghi log bảo mật phiên đăng nhập, phát hiện IP lạ truy cập trái phép, phân quyền vùng miền. | Đặt đồ ăn, giao hàng, bản đồ chỉ đường, chấm công theo tọa độ thực tế. |

> [!IMPORTANT]
> **Kết luận quan trọng:**
> - Địa chỉ định vị qua IP của các nhà mạng (VNPT, Viettel, FPT, Comcast...) được gán theo **DSLAM / Trạm trung chuyển / Subnet của ISP**, do đó **chỉ xác định chính xác đến Quận/Huyện, Tỉnh/Thành phố, Quốc gia, Mã Bưu điện**.
> - Không thể có "số nhà, tên đường" từ mỗi địa chỉ IP vì ISP cấp IP động (Dynamic DHCP) theo cụm dân cư để bảo vệ quyền riêng tư người dùng. Muốn có "số nhà, tên đường" bắt buộc phải kết hợp `navigator.geolocation` + Reverse Geocoding API (Google Maps / OpenStreetMap Nominatim).

---

## 2. Giải thuật Egress Public IP cho Môi trường Local / Dev (`::1`, `127.0.0.1`)

Khi lập trình viên hoặc máy chủ chạy local/mạng LAN, `req.ip` nhận được là `::1` hoặc `127.0.0.1`. Nếu chỉ trả về `Localhost` chung chung sẽ làm mất thông tin quốc gia và lá cờ.

### Giải pháp "Egress IP Probe" (Bộ dò IP cổng ra):
1. Nhận diện các IP Loopback (`::1`, `127.0.0.1`) hoặc dải Private (`10.x.x.x`, `192.168.x.x`, `172.16-31.x.x`).
2. Gửi request siêu nhẹ (Timeout 2000ms) qua cổng ra Internet của máy chủ chủ trì (Host Egress): `https://api.ipify.org?format=json`.
3. Lấy IP Public thật của máy chủ chủ trì để phân giải Quốc gia (`Vietnam`), Mã quốc gia (`VN`), Lá cờ (`🇻🇳`), Thành phố (`Hanoi`), Khu vực (`Hanoi`).
4. Đóng nhãn minh bạch `(Local Host)` hoặc `(Mạng cục bộ)` để quản trị viên phân biệt rõ phiên thật và phiên nội bộ.
5. Lưu bộ nhớ RAM Cache trong 1 giờ để không tốn tài nguyên gọi mạng lặp lại.

```javascript
// backend/src/utils/geoSession.js
let localHostGeoCache = null;

async function getLocalHostEgressGeo() {
  if (localHostGeoCache && (Date.now() - localHostGeoCache.timestamp < 3600000)) {
    return localHostGeoCache.data;
  }
  try {
    const ipRes = await fetch('https://api.ipify.org?format=json', { signal: AbortSignal.timeout(2000) });
    if (ipRes.ok) {
      const { ip } = await ipRes.json();
      if (ip && !isPrivateOrLocalIp(ip)) {
        const geo = await resolveGeoIp(ip);
        localHostGeoCache = { timestamp: Date.now(), data: geo };
        return geo;
      }
    }
  } catch (err) {
    // Trường hợp offline hoàn toàn
  }
  return null;
}
```

---

## 3. Kiến trúc 2-Tier Hybrid Cache GeoIP (< 0.2ms)

Mọi yêu cầu đăng nhập/đăng ký đều phải được xử lý tức thì, không làm trễ thời gian phản hồi của người dùng:
1. **Tier 1 - DB Cache (`IpGeoCache`)**: Tra cứu IP trong MongoDB với `index: true`. Thời gian phản hồi < 0.2ms. Nếu đã tồn tại ➔ trả về ngay lập tức.
2. **Tier 2 - Single Remote Fetch & Permanent Upsert**: Nếu là IP mới, gọi API tra cứu (ví dụ `ip-api.com` với `fields=status,country,countryCode,regionName,city,district,zip,lat,lon,timezone,isp,org`). Ghi vĩnh viễn vào MongoDB để tất cả các phiên tương lai truy vấn 0ms.

---

## 4. Kỹ thuật Chuyển đổi Mã Quốc gia ISO-2 thành Lá Cờ

### 4.1. Chuyển đổi mã Unicode Emoji phía Backend
Sử dụng chuẩn Unicode Regional Indicator Symbols (Offset `127397 + ASCII`):
```javascript
function countryCodeToFlag(countryCode) {
  if (!countryCode || countryCode.length !== 2) return '🌐';
  const code = countryCode.toUpperCase();
  const codePoints = code.split('').map(char => 127397 + char.charCodeAt(0));
  return String.fromCodePoint(...codePoints);
}
// Ví dụ: 'VN' -> 🇻🇳, 'US' -> 🇺🇸, 'JP' -> 🇯🇵
```

### 4.2. Khắc phục nhược điểm Emoji Windows (Hiển thị Vector SVG Icon ở Frontend)
Trên một số phiên bản Windows, emoji cờ hiển thị thành chữ cái 2 ký tự (ví dụ `[V][N]`). Do đó phía Frontend chuẩn hóa bộ từ điển Vector SVG sắc nét (tương tự nút chọn ngôn ngữ):
```jsx
// Sử dụng bộ vector SVG chính xác
<span className="opt-flag" style={{ display: 'inline-flex', alignItems: 'center', width: '22px', height: '15px' }}>
  {COUNTRY_FLAGS_SVG[s.countryCode] || <span style={{ fontSize: '15px' }}>{s.flag || '🌐'}</span>}
</span>
```

---

## 5. Quy chuẩn Trình bày Giao diện Phiên (UI Separation Rule)

Để thông tin mạch lạc, dễ quản trị, thông tin định danh mạng phải được chia thành **3 dòng độc lập**:

- **Dòng 1: IP & Loại kết nối**: `🌐 IP: 192.168.1.100` hoặc `14.232.xxx.xxx`.
- **Dòng 2: Quốc gia & Lá cờ**: `[Icon Flag SVG] Quốc gia: Việt Nam (VN)`.
- **Dòng 3: Địa chỉ chi tiết**: `📍 Địa chỉ: Cầu Giấy, Hà Nội, Hanoi, Việt Nam (100000)`.
- **Dòng 4 (Bổ trợ)**: `🏢 Nhà mạng: Viettel Group • 🕒 Múi giờ: Asia/Ho_Chi_Minh (UTC+07:00 • Giờ Việt Nam / Đông Dương)`.
