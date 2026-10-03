# Đích thả — cài đặt

Mọi đường dẫn tính từ `peekvn/apps/windows/`.

| Tệp | Vai |
|---|---|
| `src/Snappy.Core/MiTargets.cs` | **thuần**: chỗ từng Đích thả trên khay đã nở, `HitTest`, chữ cái avatar |
| `src/Snappy.Core/SendPill.cs` | **thuần**: pha, chữ, tỉ lệ của pill; mọi hạn giờ (45 s / 20 s / 120 s / giữ 2–5 s) |
| `src/Snappy.Core/MiContent.cs` | **thuần**: tô hình tròn, chữ (qua mặt nạ độ phủ), thanh tiến trình lên bộ đệm premultiplied |
| `src/Snappy.Interop/TextRasterizer.cs` | GDI dựng một dòng chữ thành mặt nạ độ phủ |
| `src/Snappy.Core/MiWindow.cs` | nối dây: `Over`/`Drop` hit-test, `ShowPill`, bộ đếm pill, `FootprintChanged` |
| `src/Snappy.Core/AppHost.Mi.cs` | `IMiHost`: Shelf, Trust Store, `OfferFiles`, menu "Tất cả", bong bóng |
| `src/Snappy.Core/MiStateMachine.cs` | thêm `RunStarted` — pill mở khi Mí đã về Nghỉ (đường menu "Tất cả") |
| `src/Snappy.Protocol/Server/UltpRouter.Downloads.cs` | `OfferFiles` bung thư mục (`FilesUnder`) |
| `src/Snappy.Protocol/Transfers/TransferStore.cs` | `Read<T>` — đọc một Phiên dưới khoá |

## Quyết định

**Chữ trên layered window đi đường vòng.** GDI không biết alpha: `DrawText` thẳng lên DIB của Mí
làm byte alpha dưới nét chữ thành 0, tức chữ thủng thành lỗ click-through ngay dưới con trỏ (cùng
bẫy với `MiPainter.ZoneAlpha`, ADR-0014). Nên `TextRasterizer` vẽ trắng-trên-đen vào một DIB riêng
(`ANTIALIASED_QUALITY`, không ClearType — ClearType cho ba độ phủ ở ba kênh), đọc kênh G làm độ
phủ, và `MiContent.Blend` trộn "source over" — alpha ra luôn ≥ alpha nền. Không DLL mới: GDI và
Segoe UI đã nạp cho bảng trạng thái.

**Cột 92 DIP cố định**, không chia đều bề rộng: một avatar to ra khi chỉ có một thiết bị là đổi
chỗ đích dưới tay người dùng mỗi lần ghép thêm máy. 📐 76 DIP cắt "iPhone của Tuấn" thành
"iPhone của …" — phần bị cắt là tên người, thứ phân biệt hai iPhone trong một nhà. Ở 1920 px:
ô Shelf, 4 avatar, "Tất cả"; máy thứ năm trở đi chỉ qua menu "Tất cả".

**Pill 320 DIP** (Ticket 10 để 180). Câu dài nhất — "Không thấy iPhone của Tuấn — mở app trên máy
ấy để nhận" — cần chừng ấy ở 12 DIP. ⚠️ Hình pill là thứ chủ dự án chốt ở Ticket 10; bề rộng mới
cần chủ dự án duyệt lại.

**Đích chỉ vẽ khi thân đã đứng đúng hình.** Giữa lúc nở, chữ trôi theo một khung đang to ra trông
như vỡ. Nhắm/bỏ nhắm chỉ đổi màu, không đổi hình, nên ở đó đích có mặt mọi khung animation.

**Thư mục: cả lượt hoặc không gì.** Một thư mục con không liệt kê được thì không có lời mời nào
(`FileUnreadable`) — lời mời thiếu một nhánh cây là nói sai điều người dùng vừa thả. Không đi vào
reparse point: junction trỏ ngược lên cha là vòng lặp, symlink trỏ ra ngoài là mời bên kia đọc thứ
người dùng không kéo vào (SPEC §19.1, chiều đọc).

## Hợp đồng với capsule media (M3)

`MiWindow.FootprintChanged(PixelRect?)` — khung màn hình Mí đang chiếm, `null` khi ẩn. Bắn **một
lần cho mỗi trạng thái đích** (vạch Gợi ý, khay đã nở, pill), không cho từng khung animation. M3
nằm cùng cạnh trên; nó nghe sự kiện này và trượt ra khỏi khung. Mí không biết M3 tồn tại.

## Nhật ký (tag `MiStrip`, `AppHost`)

| Dòng | Nghĩa |
|---|---|
| `Drop targets: shelf x=106..198, device dev_… x=198..290, …, all x=570..662` | khay vừa nở; kịch bản nghiệm thu đọc toạ độ từ đây |
| `Aiming at device dev_…` / `the Shelf cell` / `the all-devices button` | Nhắm đích (Info); khoảng trống là Trace |
| `Dropped Files on the strip at device dev_…` | cú thả và đích của nó |
| `Strip send of N Shelf items to dev_…: Offered as 01…` | kết cục `OfferFiles` |
| `Pill shows a send to X: Waiting (01…)` · `Pill Waiting -> Sending` · `Pill Sending -> Done` · `Pill Waiting -> Failed (NotSeen)` | đời pill |
| `User chose dev_… from the device list` · `User chose to keep N dropped items in the Shelf` · `User closed the device list` | ba nhánh menu "Tất cả" |
| `User clicked the pill - hiding it, the send goes on` | |
