# Quản lý nghiên cứu – TS.BS. Phạm Duy Đức

Hệ thống gồm 3 phần:

- Dự án nghiên cứu thầy làm chủ nhiệm hoặc tham gia
- Tiến độ nhóm học viên hướng dẫn: chuyển nguyên 7 học viên cùng các mốc từ Sổ tiến độ cũ
- API chỉ đọc, để web quản lý của khoa hoặc đơn vị khác truy xuất

| Thành phần | Công nghệ | Vai trò |
|---|---|---|
| Kho dữ liệu | Google Sheet (7 tab) | Lưu dự án, mốc, sản phẩm, học viên, báo cáo, khoá API |
| API | Google Apps Script Web App | Đọc/ghi có phân quyền theo khoá |
| Giao diện | GitHub Pages (`index.html`) | Trang quản lý cho thầy và học viên |

## 1. Triển khai backend (khoảng 10 phút)

1. Tạo Google Sheet mới, đặt tên `QLNC – Dữ liệu`.
2. Mở **Tiện ích mở rộng → Apps Script**.
3. Tạo 2 tệp và dán nội dung tương ứng:
   - `Code.gs`: thay toàn bộ nội dung mặc định
   - `SeedData.gs`: bấm dấu + → Tập lệnh
4. Chọn hàm `khoiTao` và bấm **Chạy**. Lần đầu Google sẽ hỏi cấp quyền: chọn tài khoản → Nâng cao → Chuyển đến dự án → Cho phép.
5. Mở **Nhật ký thực thi** và chép dòng `KHOÁ QUẢN TRỊ`. Đây là khoá đăng nhập của thầy, cần giữ bí mật.
6. Bấm **Triển khai → Tùy chọn triển khai mới → Ứng dụng web**, đặt:
   - Thực thi dưới dạng: **Tôi**
   - Người có quyền truy cập: **Bất kỳ ai**
   - Bấm Triển khai, chép **URL ứng dụng web** (dạng `https://script.google.com/macros/s/…/exec`).

> Chọn "Bất kỳ ai" là để web khoa gọi được API. Dữ liệu vẫn được bảo vệ bằng khoá: không có khoá hợp lệ thì API chỉ trả về lỗi.

Khi sửa `Code.gs` về sau: **Triển khai → Quản lý các lần triển khai → Chỉnh sửa → Phiên bản mới**. Làm như vậy URL được giữ nguyên.

## 2. Đưa giao diện lên GitHub Pages

1. Trong org `ctump-yhct`, tạo repo `quanly-nghiencuu`. Có thể để public: trang không chứa dữ liệu, dữ liệu chỉ nằm trong Google Sheet.
2. Mở `index.html`, tìm dòng `const DEFAULT_URL = "";` và dán URL Web App vào giữa hai dấu ngoặc kép. Bỏ qua bước này thì trang sẽ hỏi URL khi đăng nhập.
3. Tải `index.html` lên repo, rồi vào **Settings → Pages → Deploy from branch → main / root**.
4. Địa chỉ trang: `https://ctump-yhct.github.io/quanly-nghiencuu/`. Đăng nhập bằng khoá quản trị.

## 3. Việc cần làm sau khi đăng nhập

- **Dự án của tôi**: 4 dự án được nhập sẵn (AI-SP, Liubao Granules, tổng quan fascia–kinh cân, 2 bài báo từ luận án). Thầy rà soát vai trò và ngày hạn, sau đó tick **Công khai** cho dự án muốn web khoa thấy. Mặc định tất cả đang ở chế độ riêng tư.
- **Kết nối API → Cấp khoá mới**:
  - Khoá **Học viên** cho từng người. Học viên đăng nhập bằng khoá này để gửi báo cáo 2 tuần, sửa kế hoạch cá nhân và xem phản hồi của thầy.
  - Khoá **Chỉ đọc** cho web khoa.
- Sổ tiến độ cũ trên claude.ai có thể giữ lại để tham khảo. Từ nay thầy chỉ cập nhật trên hệ thống mới.

## 4. Phân quyền

| Quyền | Đọc | Ghi |
|---|---|---|
| Quản trị (`ad_…`) | Toàn bộ | Toàn bộ, cấp và thu hồi khoá |
| Học viên (`hv_…`) | Tiến độ, báo cáo, phản hồi của chính mình | Gửi báo cáo, sửa kế hoạch cá nhân |
| Chỉ đọc (`rd_…`) | Dữ liệu tổng hợp đã công khai | Không |

Không bao giờ xuất qua API chỉ đọc: kinh phí, ghi chú riêng, email học viên, nội dung báo cáo, phản hồi của thầy, kế hoạch cá nhân.

Thu hồi khoá: bỏ tick **Hoạt động** trong thẻ Kết nối API, hoặc đặt cột `hoatDong` = FALSE trong tab `ApiKeys`.

## 5. Tài liệu API cho web khoa

Gọi bằng `GET {URL}?action=<action>&key=<khoá chỉ đọc>`.

| action | Trả về |
|---|---|
| `summary` | Số liệu tổng hợp: dự án theo trạng thái/cấp/vai trò, học viên theo chương trình và đèn tín hiệu, sản phẩm theo loại/trạng thái, số mốc trễ hạn |
| `projects` | Dự án công khai: thông tin, `tienDo` (%), `den`, `lyDo`, `mocTiepTheo`, `moc[]`, `hocVienThamGia[]`, `sanPham[]`. Thêm `&id=` để lấy 1 dự án |
| `students` | Học viên: chương trình, giai đoạn, đề tài, `tienDo`, `den`, `mocTiepTheo`, `baoCaoGanNhat`, `moc[]`, `sanPham[]` |
| `outputs` | Sản phẩm công khai: loại, tên, nơi công bố, trạng thái, ngày, liên kết, dự án/học viên |
| `upcoming` | Mốc quá hạn (`treHan: true`) và mốc đến hạn trong 30 ngày tới |
| `public` | Tất cả các mục trên trong 1 lần gọi |
| `ping` | Kiểm tra kết nối (không cần khoá) |

Định dạng phản hồi:

```json
{ "ok": true, "generatedAt": "2026-10-08T01:22:14Z", "data": { } }
{ "ok": false, "error": "Khoá không hợp lệ hoặc đã bị thu hồi", "code": "unauthorized" }
```

Ý nghĩa `den`: `ok` đúng tiến độ · `warn` cần chú ý · `bad` cần can thiệp · `done` hoàn thành · `pause` tạm dừng.

Ví dụ gọi API:

```js
// Trình duyệt
const r = await fetch(API + "?action=projects&key=" + KEY);
const { ok, data } = await r.json();
```

```php
// PHP
$data = json_decode(file_get_contents($API . "?action=summary&key=" . $KEY), true);
```

```html
<!-- Trang tĩnh không dùng được fetch: JSONP -->
<script src="{URL}?action=summary&key={KEY}&callback=hienThi"></script>
```

Lưu ý khi tích hợp:

- Dữ liệu tổng hợp được lưu đệm 5 phút, và bộ đệm tự xoá mỗi khi thầy sửa dữ liệu. Web khoa không nên gọi quá 1 lần/phút.
- Khoá nằm trong địa chỉ URL. Nếu web khoa gọi từ máy chủ (PHP, Node) thì khoá không lộ ra trình duyệt của người xem; đây là cách nên dùng.

## 6. Cấu trúc Google Sheet

| Tab | Nội dung |
|---|---|
| `DuAn` | Dự án. Cột `hocVien` chứa id học viên, ngăn bằng dấu phẩy |
| `MocDuAn` | Mốc của dự án (`duAnId`) |
| `SanPham` | Sản phẩm đầu ra, gắn với dự án (`duAnId`) và/hoặc học viên (`hocVienId`) |
| `HocVien` | Học viên, kèm 4 cột kế hoạch cá nhân (`idp…`) |
| `MocHocVien` | Mốc của học viên |
| `BaoCao` | Báo cáo 2 tuần và phản hồi của thầy |
| `ApiKeys` | Khoá truy cập |

Thầy có thể sửa trực tiếp trong Sheet. Ngày ghi dạng `yyyy-MM-dd`. Không đổi tên cột ở dòng 1.
