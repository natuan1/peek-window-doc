# ADR-0015: TempDrops một thư mục mỗi Mục, cùng hàng rào với tệp nhận qua mạng; lúc thả không hỏi shell

Date: 2026-10-01
Status: Accepted

## Context

Ticket 11 ([natuan1/peekvn#142](https://github.com/natuan1/peekvn/issues/142)) phải
biến tệp ảo (tệp đính kèm Outlook, ảnh trên web) thành tệp thật trong TempDrops, kèm
bộ dọn theo quy tắc của [CONTEXT.md §TempDrops](../CONTEXT.md), và lấy *"metadata +
icon qua `SHGetFileInfoW`"*.

Ba lực kéo:

1. **Tên tệp ảo là dữ liệu không đáng tin.** `FILEDESCRIPTORW.cFileName` do tiến trình
   nguồn đặt; với ảnh kéo từ Chrome là do một trang web đặt.
2. **KPI RAM nền 25 MB** còn ~1,7 MB biên sau lần dùng Mí đầu tiên (Ticket 10).
3. Quy tắc dọn cần một **chủ**: "xóa khi Mục bị bỏ khỏi Ngăn" đòi một Ngăn, mà Shelf là
   Ticket 12.

📐 Đo 01/10/2026 trên bản AOT: một lời gọi `SHGetFileInfoW(SHGFI_TYPENAME |
SHGFI_USEFILEATTRIBUTES)` lúc thả nạp năm DLL không bao giờ nhả (`PROPSYS`, `OLEAUT32`,
`wintypes`, `Windows.System.Launcher`, `windows.staterepositorycore`). Sau bốn cú thả,
working set lên **26,5 MB**. Đối chứng chỉ thay lời gọi ấy bằng `null` cho **24,2 MB**.

## Decision

1. **Một Mục, một thư mục `TempDrops\{Guid}\`**, tên gốc giữ nguyên bên trong. Bỏ
   Mục là xoá đúng một thư mục. Hai ảnh `image.png` không bao giờ đè nhau.
2. **Mọi lần ghi đi qua `RelativePath` + `DestinationRoot`** — đúng hàng rào S11/S16
   của tệp nhận qua mạng (`..`, `NUL`, ADS, junction, không ghi đè). Chỉ khác ở một
   chỗ: `:` được gọt thành `_` trước bước kiểm cú pháp, vì ở đây chặn `:` là vứt mọi
   ảnh tên "Cuộc họp: 9h.png" mà không chặn thêm được gì.
3. **Lúc thả không gọi `SHGetFileInfoW`.** Mục mang tên, dung lượng, MIME (so byte đầu
   tệp, không qua `FindMimeFromData` của urlmon). `ShellFileInfo` (tên loại + icon) là
   API cho khay thẻ của Ticket 12.
4. **Trần 2 GiB**, LRU theo đồng hồ logic; Mục của cú thả vừa rồi không bao giờ bị bỏ,
   kể cả khi riêng nó vượt trần. Luồng tệp ảo có trần `max(2 GiB, cỡ khai)` ngay trong
   lúc chép, vì trần dung lượng chỉ chạy sau cú thả.
5. **Shelf một Ngăn tối thiểu** (`Add`/`Remove`/`Touch`/`Clear`) là chủ của quy tắc dọn.
   Ticket 12 dựng khay thẻ và tầng nhiều Ngăn trên nó.
6. **Mí nhận cú thả tệp từ ticket này** (`DROPEFFECT_COPY`, cả dải là một đích); Đích thả
   riêng vẫn là Ticket 13. Không có cú thả thật thì không nghiệm thu được bằng cử chỉ
   người dùng.

## Consequences

- RAM sau bốn cú thả **24,4 MB**, biên còn ~0,6 MB. Ticket 12 gọi `ShellFileInfo` sẽ trả
  ~2,3 MB ấy — **ticket ấy phải đo lại mốc "sau lần gọi shell đầu tiên"**, và có thể
  phải đưa câu hỏi lên chủ dự án như #181.
- Giữa Ticket 11 và Ticket 12, người dùng thả được tệp nhưng chỉ thấy nó qua con số
  trong mục khay "Xoá sạch Shelf (n mục)".
- LRU hôm nay trùng thứ tự thả: chưa có ai gọi `Touch` (kéo ra, gửi đi là của 12/13).
- Kéo cả một thư Outlook (`IStorage`) bị từ chối có log; tệp đính kèm Outlook **chưa
  đo** (máy dev không cài Outlook).
- Chép tệp ảo là đồng bộ trong `Drop` — OLE không hứa đối tượng dữ liệu sống lâu hơn.
  Một tệp lớn giữ Mí đứng yên trong lúc chép; nhật ký ghi thời gian.

## Related

- [ADR-0006](0006-dong-goi-velopack-cai-peruser.md) — TempDrops nằm dưới gốc cài, gỡ app là xoá sạch.
- [ADR-0013](0013-mi-an-khi-nghi-dung-luoi-xac-nhan-tep-qua-dragenter.md), [ADR-0014](0014-mi-ve-bang-layered-window-khong-composition.md) — Mí và mốc RAM sau lượt kéo.
- [Tính năng: Trích xuất tệp thật/ảo → TempDrops](../features/tempdrops-windows/overview.md)
- `peekvn` bài học 176 (`NUL`), 194 (đo sau lần dùng đầu), 196 (giá của `SHGetFileInfoW`), 197 (menu khay).
