# Khay thẻ Shelf — kiểm thử

## Test tự động — `dotnet test apps/windows/Snappy.slnx`

654 test xanh (607 trên nhánh hộp chọn tệp). Nhóm mới:

| Lớp test | Khẳng định |
|---|---|
| `ShelfTests` (+6) | Free không thêm được Ngăn thứ hai, không ghim; cờ ghim lậu bị gọt; Mục ghim không bị trần đẩy ra; trần trải mọi Ngăn; Clear xoá mọi Ngăn |
| `ShelfTrayLayoutTests` | khay neo đúng chỗ Mí, thẻ không tràn, HitTest từng phần, cuộn bị kẹp, DPI |
| `ShelfDragOutTests` | khối `DROPFILES` đối chiếu bằng chính `DragQueryFileW`; `QueryContinueDrag`; mọi tệp đã mất thì không kéo; glyph theo MIME |
| `ShelfDragOutShellTests` | `IFileOperation.CopyItems` — đúng bộ chép Explorer dùng sau cú thả — nhận `IDataObject` của ta và chép đúng byte |
| `ShelfTrayProtocolTests` | ảnh chụp đi qua ống nguyên vẹn (tên tiếng Việt, TempDropId thật); mọi sự kiện khứ hồi; TAB/xuống dòng trong log không làm vỡ dòng; dòng rác → `null`; `FindMissing` |
| `TrayMenuTests` (+2), `ShelfNoticesTests` (+3) | "Mở Shelf (n mục)" luôn có và nằm trên "Xoá sạch"; chữ của hai bong bóng mới |

**Đột biến đã ép đỏ** bằng Edit:
- bỏ chặn mức log dạng số: test cũ với "7" **không** đỏ, vì `IsDefined` đã chặn sẵn. Test sửa sang "2" thì đỏ.
- nới ngưỡng `0x10000` của hộp chọn tệp: đỏ.

## Nghiệm thu thật — `ci/check-shelf-tray.ps1`

Chạy **ngoài gói MSIX**, bằng chuột thật. Không nằm trong Jenkins, vì nó chiếm chuột và
màn hình khoảng 2 phút.

| Cổng | Cử chỉ | 📐 02/10/2026 |
|---|---|---|
| A | thả 2 tệp thật một lượt + 1 tệp trong zip lên Mí | 3 Mục, 1 thư mục TempDrops |
| B | menu khay → "Mở Shelf (3 mục)" | khay ở tiến trình **con**, đúng hình học |
| C | kéo thẻ tệp ảo ra Explorer | SHA-256 trùng nguồn, Touch được gọi |
| D | kéo tay cầm (cả nhóm) ra Explorer | 3/3 trùng byte; tệp gốc còn nguyên (Copy); tiến trình nền không nạp `d3d11`/`dcomp`/`datatransfer` |
| E | nút thư mục | Explorer mở đúng thư mục nguồn |
| F | ✕ trên thẻ tệp ảo | Mục rời Ngăn, TempDrops 1 → 0 |
| G | Esc; bấm ra ngoài | khay đóng, tiến trình khay thoát, nhật ký ghi mã 0 cho mỗi lần đóng |
| H | xoá tệp gốc B, mở lại khay | "1 items no longer exist" |
| J | kéo cả nhóm vào trang web trong **Edge InPrivate** | trang nhận A đúng tên, cỡ, SHA-256; B (đã mất) không đi cùng; bong bóng "Chỉ kéo được 1 trong 2 mục" |
| I | RAM sau tất cả | private **8,03 MB**, tổng **28,3 MB** (red line 30) |

⚠️ Biên theo red line tổng chỉ còn ~1,7 MB. Lượt trước, chưa có cổng J nên không có bong
bóng, đo được 26,19 MB tổng.

### An toàn khi kịch bản cầm chuột

`ci/drop-guard.ps1`:

- **Màn che** phủ vùng làm việc. Không phủ toàn màn hình, vì như thế Mí sẽ coi là fullscreen và ẩn đi.
- **`Release-Over`**: chỉ nhả chuột khi cửa sổ gốc dưới con trỏ đúng là đích; sai thì Esc rồi mới nhả.

`check-drop-extract.ps1` cũng dùng nó. Lý do: ngày 2026-10-02, một cú thả chệch đã rơi vào
cửa sổ chat Chrome của người dùng.

## Chưa nghiệm thu

- Đa màn hình, DPI 125–200%: máy dev có một màn hình ở 100%.
- Thả lên Mí **trong lúc khay đang mở**: khay nằm đè vùng Mí và không là đích thả. Thực
  tế người dùng bấm vào cửa sổ nguồn để bắt đầu kéo, nên khay mất tiêu điểm và đóng
  trước. Chưa có cổng nào kiểm điều này.
