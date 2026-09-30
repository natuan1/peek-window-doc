# Mí — luồng người dùng và hệ thống

## Kéo một tệp từ Explorer lên Mí

| Người dùng | Hệ thống | Trạng thái |
|---|---|---|
| nhấn chuột lên một tệp rồi rê đi | Explorer gọi `DoDragDrop` và dựng cửa sổ `SysDragImage`; hook out-of-context của Snappy thấy nó (📐 0,7 s sau khi nhấn) | Nghỉ → **Gợi ý** |
| | Trước lệnh hiện: đặt Mí lên màn hình có con trỏ và hỏi fullscreen. Lần hiện đầu tiên còn dựng OLE và bề mặt vẽ layered | |
| rê tệp vào vùng Mí (60 DIP) | OLE gọi `IDropTarget::DragEnter`; `QueryGetData(CF_HDROP)` trả lời là tệp; hẹn 80 ms | Gợi ý |
| dừng ≥ 80 ms | hết hạn → lệnh `Expand`, cửa sổ lên 90 DIP, app tự vẽ ~11 khung trong 160 ms, cubic ease-out | → **Sẵn sàng nhận** |
| rê ra, quay lại trong < 250 ms | `DragLeave` hẹn 250 ms; `DragEnter` huỷ hẹn | giữ nguyên |
| rê ra hẳn | hết 250 ms → `Collapse` | → **Gợi ý** |
| nhả chuột ở đâu đó | ảnh kéo biến mất, không còn nút nào giữ → `DragEnded` | → **Nghỉ**, ngay |
| nhấn Esc (chuột còn giữ) | ảnh kéo biến mất, nút vẫn giữ → ân hạn 300 ms; lúc nhả, polling thấy cạnh nhả → `DragEnded` | → **Nghỉ** |

## Lướt nhanh qua

Tệp nằm trong vùng dưới 80 ms: `DragLeave` huỷ hẹn nở. Mí giữ vạch Gợi ý, không nở.

## Kéo không có ảnh kéo (dự phòng)

Có nguồn kéo không dựng `SysDragImage`. Polling hai tầng lo phần này:

- Tầng chậm, mỗi 500 ms: có nút chuột nào đang giữ không?
- Tầng nhanh, mỗi 50 ms: con trỏ đã vào vùng Mí chưa?

Hai trường hợp bị bỏ qua: cú nhấn **bắt đầu** trong vùng Mí (giữ tab trình duyệt), và
lúc đang kéo một cửa sổ (`EVENT_SYSTEM_MOVESIZESTART`).

## Fullscreen

Mí coi là fullscreen trong hai trường hợp:

- `SHQueryUserNotificationState` báo `BUSY`, `D3D_FULL_SCREEN` hoặc `PRESENTATION_MODE`;
- hoặc cửa sổ foreground không có `WS_CAPTION` và phủ kín màn hình.

Việc kiểm chạy **trước** lệnh hiện, và mỗi lần foreground đổi. Máy trạng thái vẫn đi
tiếp nhưng `Visible` là `false`, nên không có pixel nào lên trên game.

Một cửa sổ phóng to bình thường (còn thanh tiêu đề) **không** tính là fullscreen, kể cả
khi taskbar tự ẩn.
