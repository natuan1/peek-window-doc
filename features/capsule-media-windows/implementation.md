# Capsule media — cài đặt

Mọi đường dẫn tính từ `peekvn/apps/windows/`.

| Tệp | Vai |
|---|---|
| `src/Snappy.Core/MediaCapsule.cs` | **thuần**: `CapsuleModel` (hiện/ẩn, nán 30 s, dòng chữ, chữ ô pin), `CapsuleLayout` (nhà giữa dải Mí, trượt né khung Mí, các phần + hit-test), `MediaCapsuleProtocol` (dòng `footprint`) |
| `src/Snappy.Core/CapsuleContent.cs` | **thuần**: tô capsule lên bộ đệm premultiplied bằng `MiPainter`/`MiContent` |
| `src/Snappy.Core/MediaCapsuleWindow.cs` | cửa sổ layered của con: nối GSMTC, pin, fullscreen, chuột, trượt |
| `src/Snappy.Core/MediaCapsuleProcess.cs` | phía cha: khởi động con, chuyển khung Mí, ghi nhật ký của con, dựng lại ≤ 3 lần. Phía con: `RunChild` |
| `src/Snappy.Core/FullscreenRule.cs` | thêm `Measure`: Mí và capsule dùng chung một phép đo fullscreen |
| `src/Snappy.Interop/WinRt.cs` | WinRT bằng vtable tay: `HSTRING`, activation factory, chờ `IAsyncOperation`, `WinRtHandler` (delegate COM bốn slot) |
| `src/Snappy.Interop/MediaSessions.cs` | GSMTC + `ImageDecoder` (WIC). Mọi lời gọi đi qua một luồng MTA riêng |
| `src/Snappy.Interop/HeadsetBattery.cs` | SetupAPI + Bluetooth GATT Win32 |
| `src/Snappy.Interop/DeviceContainer.cs` | `DevGetObjectProperties`: `CategoryIds`, `Connected`, tên của một device container |
| `ci/check-media-capsule.ps1` | nghiệm thu thật A–K |

## Quyết định

**Tiến trình con thường trực, không projection CsWinRT.** Xem [ADR-0018](../../adr/0018-capsule-media-tien-trinh-con-winrt-bang-vtable-tay.md). 📐 Riêng việc projection có mặt trong exe đã làm tiến trình nền trả +1,1 MB private, vì module initializer chạy lúc khởi động dưới AOT.

**Chỉ nán 30 s sau một lần Đang phát → Tạm dừng của cùng app.** 📐 Riot Client giữ một phiên Paused suốt đời nó. Bản đầu đếm từ lúc *thấy* phiên, nên capsule hiện 30 s mỗi lần Snappy khởi động.

**Đo fullscreen theo nhịp 1,5 s khi có thứ để hiện.** Video bấm F11 thành toàn màn hình mà foreground không đổi, nên không có sự kiện nào báo. 📐 Ở kịch bản F, khi chỉ nghe sự kiện foreground, capsule đè lên một cửa sổ toàn màn hình suốt 4 s.

**Tai nghe = container thuộc nhóm `Audio`.** 📐 Trên máy dev, `System.Devices.CategoryIds` cho `Audio`, `Display.Monitor,Audio`, `Input.Keyboard,Input.Mouse`.

**`0` hiện là `-`.** Một tai nghe đang trả lời GATT thì không thể ở 0%, và nhiều firmware trả 0 khi chưa đo xong. Ticket nói "TUYỆT ĐỐI không hiện 0%".

**Chỗ đứng.** Nhà là giữa dải Mí (5–35% bề rộng màn hình). Khung Mí chạm capsule, kể cả trong một khe 8 DIP, thì capsule sang phải. Thiếu chỗ thì sang trái, thiếu cả hai thì ẩn. Mí ở màn hình khác thì không chạm, và capsule đứng yên.

## Nhật ký (tag `Capsule`, tiếng Anh)

| Dòng | Nghĩa |
|---|---|
| `Media capsule child N started` | cha đã khởi động con |
| `Listening to Windows media sessions` | GSMTC đã trả session manager |
| `Media session X: Playing (play off, pause on, next on)` | phiên hiện tại đổi app hay trạng thái |
| `Track changed on X: title N chars, artist M chars` · Trace `Track is "…"` | bài mới. Tên bài chỉ có ở mức Trace |
| `Capsule shown (home\|beside the strip) x=L..R y=T..B` · `Capsule hidden (Paused\|Idle\|fullscreen app\|no media session\|no room beside the strip)` | mỗi lần đổi chỗ hay đổi lý do. Kịch bản nghiệm thu đọc `x=` từ đây |
| `Capsule buttons: play x=…, next x=…, battery none\|shown` | toạ độ nút cho kịch bản bấm |
| `User pressed pause on the capsule - X accepted\|refused\|does not allow it` | ba nhánh của một cú bấm |
| `Album art decoded at Npx` · `The player gave no album art - drawing a note instead` | kết quả nạp bìa |
| `Headset battery: no connected Bluetooth headset with a GATT Battery Service` · `Headset battery shows "85%" from …` | kết quả tra pin. Trace có một dòng cho mỗi thiết bị bị bỏ qua, kèm lý do |
| `A fullscreen app took the screen (quns=…) - hiding the capsule` · `Fullscreen app gone` | vào và ra fullscreen |
| `Media capsule child N exited with code C - restarting in 5 s (k/3)` | con chết bất thường |
