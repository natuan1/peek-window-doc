# Nền móng app Windows — quy trình

## Người dùng đi qua những gì

| Người dùng làm | Họ thấy gì | Nếu không làm gì thì sao |
|---|---|---|
| Chạy `Snappy.exe` | Không cửa sổ nào mở ra; một icon xuất hiện ở khay hệ thống | App ngồi im ở khay, ~0% CPU |
| Bấm trái vào icon khay | Bảng trạng thái nhỏ hiện ở góc dưới-phải màn hình đang có con trỏ | Bảng đứng đó cho tới khi bấm ra ngoài |
| Bấm ra ngoài bảng | Bảng ẩn đi | — |
| Bấm phải vào icon khay | Menu hai mục: *Mở bảng trạng thái*, *Thoát Snappy* | Menu tự đóng khi bấm ra ngoài |
| Chọn *Thoát Snappy* | Icon biến mất khỏi khay, tiến trình kết thúc | — |
| Chạy `Snappy.exe` lần thứ hai | Bảng trạng thái của bản đang chạy hiện lên; không có icon khay thứ hai | — |

Bảng trạng thái nói đúng ba điều: tên sản phẩm, số phiên bản, và *"Đang chạy nền, sẵn sàng."* — cộng một câu chỉ đường thoát, vì app không có cửa sổ chính nào để bấm dấu X.

## Hệ thống đi qua những gì

### Khởi động

1. Windows đọc `app.manifest` → chốt chế độ DPI **Per-Monitor V2** cho tiến trình. Việc này xong trước dòng đầu tiên của `Main`.
2. Dựng `%LocalAppData%\Snappy\` và `logs\` rồi mở nhật ký. Hỏng bước này không dừng app — lỗi đi ra `OutputDebugStringW` và app vẫn chạy, vì icon khay không cần đĩa.
3. Giành khoá `Local\Snappy.SingleInstance`. Không giành được → tìm cửa sổ của bản đang chạy, gửi thông điệp mở bảng trạng thái, rồi thoát với mã 0.
4. Dựng cửa sổ ẩn nhận thông điệp (top-level, không `WS_VISIBLE`, 1×1).
5. Đăng ký hai thông điệp theo tên: `TaskbarCreated` và `SnappyShowStatusPanel`.
6. Dựng bảng trạng thái (ẩn sẵn, sống suốt đời tiến trình).
7. Nạp icon đúng cỡ theo DPI hệ thống, thêm vào khay, đặt `NOTIFYICON_VERSION_4`.
8. Vào vòng lặp thông điệp.

### Icon khay không thêm được

Xảy ra thật khi Snappy khởi động cùng lúc đăng nhập, lúc Explorer còn đang dựng. Không có icon khay thì **với người dùng app không tồn tại** — không bảng trạng thái, không đường thoát nào ngoài Task Manager.

Hai lớp đỡ, cần cả hai:

- `TaskbarCreated` — Explorer chết rồi sống lại thì phát thông điệp này; app thêm lại icon. Nhưng nếu Explorer đã phát xong **trước khi** cửa sổ của ta tồn tại thì nó không phát lại lần nào nữa.
- Hẹn giờ thử lại: 5 lượt, mỗi lượt cách 2 giây. Hết lượt thì ghi nhật ký mức Error — app vẫn chạy nhưng người dùng không có đường vào, và đó là thứ phải đọc được khi họ báo lỗi.

### Thoát

Lệnh thoát tới từ bốn nguồn không hẹn nhau: menu khay, `WM_CLOSE`, `WM_DESTROY`, và `WM_ENDSESSION` lúc Windows tắt máy. `AppLifecycle.BeginShutdown()` trả `true` cho đúng **một** người gọi trong đời tiến trình; ai nhận `true` là người dọn.

Thứ tự có chủ ý: **gỡ icon khay trước**, rồi mới `PostQuitMessage`. Sau `PostQuitMessage` không còn ai bơm thông điệp để shell kịp nhận lệnh gỡ — và một icon ma nằm lại ở khay sau khi app đã chết là mâu thuẫn người dùng không giải thích được.

### Đổi DPI

Bảng trạng thái nhận `WM_DPICHANGED` → dựng lại font theo DPI mới và đặt lại vị trí theo hình chữ nhật Windows đề nghị (nó đã tính cả phần bảng nằm đè lên ranh giới hai màn hình). Bảng luôn mở ở màn hình **đang có con trỏ**, không phải màn hình chính — người dùng vừa bấm vào icon khay nên con trỏ đang ở đúng màn hình họ nhìn.
