# Khay thẻ Shelf — implementation

Repo `peekvn`, nhánh `feat/143-shelf-tray`.

| Tệp | Vai |
|---|---|
| `apps/windows/src/Snappy.Core/Shelf.cs` | chủ của Ngăn. `ShelfPlan` (Free = 1 Ngăn, không ghim), `ShelfCompartment`, `Active`, `TotalCount`, sự kiện `Changed`; trần LRU trải mọi Ngăn, Mục đã ghim không bị đẩy; `TryAddCompartment`/`TrySetActive`/`TrySetPinned` |
| `apps/windows/src/Snappy.Core/ShelfTrayLayout.cs` | bố cục thuần, test được: cao 124 DIP, thẻ 156×70, khe 8, tay cầm 132, nút 20; `HitTest`, cuộn bị kẹp |
| `apps/windows/src/Snappy.Core/ShelfWindow.cs` | cửa sổ khay **ở tiến trình con**: GDI double-buffer, hover, `DragDetect` → `DragOut`, ✕, mở thư mục, đóng khi mất tiêu điểm |
| `apps/windows/src/Snappy.Core/ShelfGlyphs.cs` | glyph Segoe MDL2 theo MIME — thay `SHGetFileInfoW` (📐 +7,5 MB) |
| `apps/windows/src/Snappy.Core/ShelfTrayProcess.cs` | cha: mở con, đọc sự kiện trên luồng nền, `Write`, `CloseRunning`. Con: `RunChild` — `OleInitialize`, vòng thông điệp, luồng đọc stdin gom ảnh chụp và kiểm tệp trên đĩa |
| `apps/windows/src/Snappy.Core/ShelfTrayProtocol.cs` | giao kèo từng dòng UTF-8, trường ngăn TAB: ảnh chụp `item …` tới `end`; sự kiện `remove`/`dragging`/`dragged`/`folder-missing`/`log`; dòng rác trả `null`, không đoán |
| `apps/windows/src/Snappy.Core/ChildProcess.cs` | phần chung với hộp chọn tệp: ba ống, stderr bất đồng bộ, `Close` không ném, `ExitWhenParentGone` |
| `apps/windows/src/Snappy.Interop/FileDragSource.cs` | `IDataObject` chỉ `CF_HDROP` (khối `DROPFILES` 20 byte, UTF-16) + `IDropSource`, qua `[GeneratedComInterface]`; `DoDragDrop` chỉ Copy; tách tệp đã mất trước khi kéo |
| `apps/windows/src/Snappy.Core/AppHost.cs` | nối dây: `OpenShelf`, `SendShelfSnapshot`, `HandleShelfTrayEvents` trên luồng UI, `ReportDragOut`, `ReportShelfTrayExit` |
| `apps/windows/src/Snappy.Core/TrayMenu.cs` | "Mở Shelf" (id 7) luôn có; "(n mục)" đếm **cả Shelf** |
| `apps/windows/src/Snappy.Core/MiWindow.cs` | `Suppressed`: Mí ngó lơ lượt kéo bắt đầu từ khay |
| `apps/windows/src/Snappy.Core/Monitors.cs` | màn hình dưới con trỏ (dùng chung Mí và khay) |

## Vì sao không dùng nguồn kéo của shell

`SHCreateDataObject` + `SHDoDragDrop` "giống Explorer hơn", nhưng 📐 02/10/2026:
- nó nạp chừng ấy DLL kéo-thả;
- với PIDL tuyệt đối làm con của Desktop, Explorer từ chối cú thả (con trỏ ⊘).

`IDataObject` tự viết với một định dạng là đủ. Explorer, ô thả tệp của Chromium và
`IFileOperation.CopyItems` đều nhận `CF_HDROP`.

## Mức nhật ký đi qua ống

Con không mở tệp nhật ký. Mỗi dòng của nó đi về cha dưới dạng sự kiện `log` kèm
`ShelfTrayLogLevel` (Trace/Info/Warn/Error). Cha ghi đúng mức đó, nên Trace vẫn tôn trọng
công tắc nhật ký chi tiết (AGENTS.md §6.4). `FileDragSource.Diagnostic` ghi mọi định
dạng đích thả hỏi, ở mức Trace.

Mức log dạng số (`log\t2\t…`) bị coi là rác. `Enum.TryParse` nhận cả "2", và một dòng
rác không được đọc thành Warn.
