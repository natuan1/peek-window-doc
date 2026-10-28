# Trích xuất tệp thật/ảo → TempDrops — kiểm thử

## Test tự động — `dotnet test apps/windows/Snappy.slnx`

594 test xanh (537 trước ticket). Nhóm mới:

| Lớp test | Khẳng định |
|---|---|
| `TempDropsTests` | một `{Guid}` mỗi Mục, đúng byte đúng tên; hai `image.png` không đè nhau; tên thù địch (`..\..\x`, `\x`, rỗng, `".. "`, `NUL`) không ra khỏi Mục; ADS bị gọt; nguồn hỏng giữa chừng không để tệp dở; Release; LRU; trần; Mục vừa thả lớn hơn trần thì giữ; Clear; tệp đang mở không kéo lượt dọn đổ theo; WipeAll; PurgeLeftovers |
| `DropIntakeTests` | tệp thật giữ đường dẫn gốc; thư mục thật; tệp ảo đúng byte + MIME theo byte; HDROP thắng tệp ảo; cây zip thành một Mục; một tệp hỏng không làm mất tệp khác; cỡ khai sai thì tin đĩa; đường dẫn đã mất; **luồng vô tận bị chặn ở trần** |
| `ShelfTests` | bỏ Mục trích xuất xoá tệp; bỏ Mục tệp thật **không** đụng tệp gốc; Mục đang mở ở app khác thì ở lại; trần → Mục cũ rời Ngăn; Touch; Clear; Mục gửi xong vẫn nằm lại |
| `MimeTypesTests` | PNG thắng đuôi `.jpg`; vỏ zip → docx theo đuôi; registry rác không được tin; mọi kết quả đúng hình dạng `WireFormat.IsMimeType` |
| `ShelfNoticesTests` | chữ của ba bong bóng; hai lý do "không nhận được" cho hai lời khuyên khác nhau |
| `ShellFileInfoTests` | tên loại cho tệp/thư mục; `HICON` thật và trả lại được |
| `TrayMenuTests` | mục "Xoá sạch Shelf (n mục)" chỉ hiện khi có Mục, không nằm sát "Thoát" |

**Đột biến đã ép đỏ** (sao lưu bằng `cp`, không `git checkout`): bỏ `keep` của trần;
bỏ đổi `\` → `/`; bỏ xoá tệp dở; bỏ luật "vỏ zip"; ưu tiên tệp ảo; bỏ gọt `:`; bỏ trần
luồng; bỏ phép giữ Mục khi xoá hỏng — cả tám đều đỏ.

## Nghiệm thu bằng cử chỉ thật — `apps\windows\ci\check-drop-extract.ps1`

Kéo bằng `SendInput` (nguồn kéo tự gọi `DoDragDrop`), đối chiếu **byte trên đĩa** —
tên, dung lượng, SHA-256 — với nguồn. 📐 01/10/2026, bản AOT:

| Kịch bản | Kết quả |
|---|---|
| A. Explorer, tệp thật | Mục trỏ tệp gốc 300 000 byte; TempDrops không thêm gì |
| B. Explorer, tệp trong zip | 200 000 byte, SHA-256 trùng |
| C. Chrome, ảnh `localhost` | `anh-thu.png` 547 714 byte, SHA-256 trùng |
| D. Edge, cùng ảnh | như C |
| E. RAM sau bốn cú thả | 24,31 · 24,32 · 24,36 · 24,40 MB (KPI 25) |
| F. chuột phải khay → "Xoá sạch Shelf (4 mục)" | TempDrops 3 → 0 thư mục |
| G. thả lại rồi thoát | `Wiped TempDrops on exit (1 drops)`, thư mục biến mất |

`check-mi-drag.ps1` của Ticket 10 vẫn đạt cả năm kịch bản. Bộ cài 10,88 MB (KPI 15 MB).
Khởi động sau khi cấy một thư mục rác: `Purged 1 leftover entries from a previous run`.

📐 **28/10/2026 chạy lại trên `main`** (publish của HEAD, 972 test xanh): `KẾT QUẢ`
xanh cả bốn nguồn đúng tên, đúng byte; F xoá sạch TempDrops 3 → 0; G
`Wiped TempDrops on exit (1 drops)`. RAM theo thước
[ADR-0016](../../adr/0016-kpi-ram-nen-do-bang-private-working-set.md): sau bốn cú thả
private **6,33 MB** / tổng 26,53 MB; sau menu khay + thả lại private **6,50 MB** /
tổng 27,18 MB (KPI private < 25, red line tổng 30).

## Outlook — đã đo 2026-10-28, kết cục không nằm ở phía nhận

- **Outlook Desktop** — máy không cài; đường `IStream` của nó được chứng minh bằng
  tệp trong zip. Không chặn ticket nào.
- **Outlook mới (olk)** — đã đăng nhập, mở `.eml` tự tạo (tệp đính kèm 200 000 byte),
  kéo thật bằng `SendInput` qua màn che, thả đúng ô Shelf trên Mí. Cả hai dạng nguồn
  (chip `tep-dinh-kem-olk.bin` lẫn ảnh nội dòng) Mí ghi nhận cùng một bộ định dạng:

  ```
  Drag offers: DragContext[4], DragImageBits[1], chromium/x-renderer-taint[1],
               chromium/x-drag-id[1], Chromium Web Custom MIME Data Format[1]
  ```

  Không `CF_HDROP`, không `FileGroupDescriptorW`/`FileContents` → `TempDrops` 0.
  Giao diện thư của Outlook mới là web renderer: cú kéo của nó không mang định dạng
  tệp OLE, nên **không receiver nào** trích xuất được — hành vi của nguồn, không phải
  lỗ hổng của Snappy.
- ~~RAM sau khi mở menu khay 25,5 MB~~ — đã sửa 2026-10-01 ([ADR-0016](../../adr/0016-kpi-ram-nen-do-bang-private-working-set.md)).
  Mốc "sau menu + thả lại" của `check-drop-extract.ps1` giờ là hàng rào: 📐 private
  6,0 MB, tổng 25,9 MB (KPI private < 25, red line tổng 30).
