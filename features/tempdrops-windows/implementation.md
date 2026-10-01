# Trích xuất tệp thật/ảo → TempDrops — implementation

Repo `peekvn`, nhánh `feat/142-tempdrops`.

| Tệp | Vai |
|---|---|
| `apps/windows/src/Snappy.Interop/DropPayload.cs` | `IDropPayload` + bản thật trên `IDataObject`: `GetData` (slot 3), `EnumFormatEtc` (slot 8) qua vtable tay; `DragQueryFileW`; đọc `FILEDESCRIPTORW` bằng offset với phép kiểm biên cạnh phép đọc (trần 10 000 mục); `IStream::Seek/Read` hoặc `HGLOBAL`, khối 64 KB |
| `apps/windows/src/Snappy.Interop/DropTarget.cs` | trao payload cho sink ở `DragEnter`/`Drop`, đóng nó sau lời gọi; `effect &=` hiệu ứng nguồn cho phép |
| `apps/windows/src/Snappy.Interop/ShellFileInfo.cs` | `SHGetFileInfoW`: tên loại, icon (`ShellIcon : IDisposable`). **Chưa có ai gọi trong app** — dành cho khay thẻ Ticket 12 |
| `apps/windows/src/Snappy.Interop/Win32/DropNative.cs` | P/Invoke + `STGMEDIUM`, offset `FILEDESCRIPTORW` (592 byte), `SHFILEINFOW` |
| `apps/windows/src/Snappy.Core/DropIntake.cs` | payload → `ShelfItem`; tệp thật thắng tệp ảo; gom cây zip; trần luồng `max(2 GiB, cỡ khai)` để một nguồn đổ mãi không làm đầy đĩa |
| `apps/windows/src/Snappy.Core/TempDrops.cs` | một thư mục `{Guid}` mỗi Mục; ghi qua `RelativePath` + `DestinationRoot` (S11/S16); LRU bằng đồng hồ logic; `Release`/`EnforceCap`/`Clear`/`WipeAll`/`PurgeLeftovers` |
| `apps/windows/src/Snappy.Core/Shelf.cs` | Ngăn tối thiểu: `Add` (kèm trần), `Remove`, `Touch`, `Clear` — chủ của quy tắc dọn |
| `apps/windows/src/Snappy.Core/MimeTypes.cs` | chữ ký byte đầu tệp; vỏ zip/OLE/PE thì đuôi tệp quyết định; bảng đuôi cố định, rồi registry, rồi `application/octet-stream` |
| `apps/windows/src/Snappy.Core/ShelfNotices.cs` | chữ của ba bong bóng: đầy, không nhận được (hai lý do, hai lối ra), xoá chưa hết |
| `apps/windows/src/Snappy.Core/MiWindow.cs`, `AppHost.cs`, `TrayMenu.cs` | nối dây: `DROPEFFECT_COPY` khi có tệp, `TakeDrop`, mục khay `ClearShelf` (id 6), dọn lúc thoát/khởi động |

## Tên tệp ảo là dữ liệu không đáng tin

Tên trong `FILEDESCRIPTORW` do tiến trình nguồn đặt — với ảnh từ trình duyệt là do
**một trang web** đặt. Nó đi qua đúng hàng rào của tệp nhận qua mạng
(`peekvn` SECURITY.md S11/S16): `..`, đường tuyệt đối, `NUL` (bài học 176 của `peekvn`),
ADS `a.txt:x`, junction. Khác đường mạng ở một chỗ: `:` được gọt **trước** bước kiểm cú
pháp, để ảnh tên "Cuộc họp: 9h.png" không bị vứt — gọt xong `C:\x` chỉ là thư mục `C_`
bên trong Mục.

## Chưa làm

- `IStorage` (kéo cả một thư Outlook `.msg`) — bị từ chối có log và bong bóng.
- Dung lượng của **thư mục thật** không cộng lúc thả (`Size = null`): cộng cả cây trên
  thread thông điệp là treo Mí với một thư mục lớn.
- `Shelf.Touch` chưa có ai gọi — LRU hôm nay trùng thứ tự thả. Kéo ra (Ticket 12) và gửi
  (Ticket 13) là chỗ gọi nó.
