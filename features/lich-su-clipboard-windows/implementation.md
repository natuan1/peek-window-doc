# Lịch sử clipboard — cài đặt

| Phần | Tệp (`peekvn/apps/windows/src`) | Ghi chú |
|---|---|---|
| Lịch sử | `Snappy.Core/ClipboardHistory.cs` — `ClipboardHistory` | thuần; mới nhất trước, trùng thì lên đầu, `ApplyLimit`, tạm dừng, `Changed` |
| Bộ lọc | cùng tệp — `ClipboardPrivacy.ReasonToSkip` | trả **tên dấu** (đi vào nhật ký), không bao giờ chữ |
| Mục | `ClipboardEntry(Text, CopiedAt)` | `ToString()` không in chữ |
| Đọc/ghi | `Snappy.Interop/ClipboardAccess.cs` | hỏi dấu trước; đăng ký tên dấu hỏng thì không đọc chữ (đóng cửa) |
| Win32 | `Snappy.Interop/Win32/ClipboardNative.cs` | chỉ `user32`/`kernel32` — không DLL mới cho tiến trình nền |
| Gõ Ctrl+V | `Snappy.Interop/PasteKeys.cs` | app chạy quyền quản trị nuốt phím (UIPI) mà `SendInput` không báo — nhật ký ghi "Sent", không "Pasted" |
| Bảng | `Snappy.Core/ClipboardPanelLayout.cs` (thuần) + `ClipboardPanelWindow.cs` (GDI) | dựng ở lần mở đầu; chuột chỉ đổi dòng đang chọn sau khi thật sự di chuyển |
| Nối dây | `Snappy.Core/AppHost.Clipboard.cs` | listener, `RegisterHotKey`, timer dán, `ApplyClipboardLimit` từ `ApplyEdition` |
| Menu khay | `TrayMenu.cs` — `OpenClipboardHistory` (id 9) | nhãn mang phím tắt sau `\t`; không đăng ký được thì không hứa phím |

Bảng ở **tiến trình nền**, không tiến trình con như khay Shelf — xem [ADR-0020](../../adr/0020-lich-su-clipboard-loc-theo-dau-o-tien-trinh-nen.md).
