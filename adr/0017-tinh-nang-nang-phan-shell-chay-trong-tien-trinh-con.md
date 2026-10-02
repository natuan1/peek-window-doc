# ADR: Tính năng nặng phần shell chạy trong tiến trình Snappy con — hộp chọn tệp, khay thẻ Shelf

Date: 2026-10-02
Status: Accepted

## Context

Một DLL đã nạp vào tiến trình nền thì ở lại tới khi tiến trình chết. Đóng cửa sổ không nhả được gì. Hai tính năng của Ticket 09 và Ticket 12 nạp những DLL rất nặng, và cả hai đều đẩy Snappy qua red line 30 MB (working set tổng) của [ADR-0016](0016-kpi-ram-nen-do-bang-private-working-set.md):

| Tính năng | Nạp gì | 📐 Working set tổng |
|---|---|---|
| Hộp chọn tệp (`GetOpenFileNameW`), "Gửi tệp tới …" | bộ khung Explorer: `ExplorerFrame`, `Windows.Storage`, thanh tìm kiếm, mọi shell extension của máy | **63 MB** sau một lần mở (private ~11 MB) |
| Kéo Mục từ Shelf ra app khác (`DoDragDrop`) | 18 DLL của tầng kéo-thả hiện đại: `windows.applicationmodel.datatransfer`, `d3d11`, `dxgi`, `dcomp`, `d2d1`, `Windows.UI`… | 25,3 → **42,8 MB** sau lượt kéo đầu |

Đo ngày 2026-10-01 và 2026-10-02 trên bản publish AOT, ngoài mọi gói MSIX.

Phần kéo-thả không né được bằng cách viết khác:

- `DoDragDrop` thuần của `ole32` với một `IDataObject` tự viết vẫn nạp đủ 18 DLL ấy. Windows 11 dựng tầng hiện đại cho mọi **nguồn** kéo.
- `SHCreateDataObject` + `SHDoDragDrop` cũng nạp chừng ấy. Thêm nữa, với PIDL tuyệt đối làm con của Desktop, Explorer từ chối cú thả (con trỏ ⊘).
- Chỉ đưa riêng `DoDragDrop` sang một tiến trình con thì không chạy. OLE chỉ bắt chuột khi cú nhấn rơi vào cửa sổ của chính luồng gọi nó, nên con nhận cú nhấn của cha thì đứng chờ mãi.

Cắt working set (`SetProcessWorkingSetSize`, `EmptyWorkingSet`) đã bị loại từ ADR-0016, vì nó làm con số đẹp mà không đổi gì.

## Decision

1. **Tính năng nào nạp phần nặng của shell thì chạy trong một tiến trình Snappy con**, cùng một exe, đánh dấu bằng cờ đứng đầu dòng lệnh:
   - `Snappy.exe --pick-files "<tiêu đề>"`: đúng một hộp chọn tệp, rồi thoát.
   - `Snappy.exe --shelf-tray`: cả khay thẻ Shelf, sống đúng lúc khay mở. Cú nhấn, cú kéo, và mọi DLL đều thuộc về con.

   Con chết thì hệ điều hành thu hồi trọn những gì nó đã nạp.

2. **Con đứng trước mọi thứ trong `Main`**, trước cả Velopack. Nó không phải một lượt "mở app", không đụng khoá một-bản-đang-chạy, và không mở tệp nhật ký.

3. **Cha vẫn là chủ duy nhất của dữ liệu.** Shelf, TempDrops, `Touch`, bong bóng và nhật ký đều ở cha. Hai bên nói với nhau bằng từng dòng UTF-8 qua stdin/stdout, các trường ngăn bằng TAB. Windows cấm ký tự 1–31 trong đường dẫn, nên dòng và TAB là ranh giới an toàn. Nhật ký của con đi về cha dạng sự kiện `log`, giữ nguyên mức; Trace vẫn tôn trọng công tắc nhật ký chi tiết.

4. **Một lớp chung, `ChildProcess`**, vì hai bản viết tay đã sai ở cùng ba chỗ:
   - Redirect cả ba ống. stderr gom bất đồng bộ, vì hai ống cùng đọc đồng bộ là đứng cả hai.
   - `Close()` không ném. Nó nằm giữa đường thoát của app.
   - Con tự thoát khi stdin đóng, tức khi cha chết, kể cả bị `taskkill`. Không để một hộp thoại mồ côi trên màn hình.

5. **Kết cục của con phải phân biệt được.**
   - Hộp chọn tệp có `PickOutcome`: chọn / huỷ / lỗi hộp thoại (mã `CommDlgExtendedError`, luôn dưới `0x10000`) / con hỏng (mã khác, kèm stderr) / cha đóng vì Snappy thoát (không báo lỗi cho người dùng).
   - Khay Shelf: mã 0 là đóng êm, cha giết lúc thoát là Info, còn lại là Error kèm stderr.

## Consequences

- 📐 RAM nền sau hộp chọn tệp: private 5,1 MB, tổng **23,0 MB** (trước: 63 MB).
- 📐 Sau toàn bộ kịch bản Shelf (thả, mở khay, kéo ra Explorer và Edge, bỏ Mục, đóng khay, bong bóng): private 8,0 MB, tổng **28,3 MB**. Tiến trình nền không nạp `d3d11`/`dcomp`/`datatransfer`. Biên theo red line tổng chỉ còn **~1,7 MB**, và phần lớn mức tăng là bong bóng của khay hệ thống.
- Mỗi lần mở mất thêm khoảng 0,1 s để khởi động con. Không người dùng nào thấy.
- Luồng chính của con chưa tắt IME, nên **ô tên tệp của hộp chọn tệp vẫn gõ được tiếng Việt**. Ràng buộc "không có ô gõ chữ trên luồng UI" của ADR-0016 được giải luôn cho hộp chọn tệp.
- **Icon thẻ Shelf là glyph Segoe MDL2, không phải icon shell.** 📐 `SHGetFileInfoW` cho icon cộng ~7,5 MB tổng. ADR-0015 từng dời việc tra icon sang khay thẻ; ADR này chốt là khay thẻ cũng không tra.
- Tính năng nặng sau này (xem trước tệp, thumbnail, đổi định dạng…) có sẵn đường đi: thêm một cờ con, dùng `ChildProcess`.
- Hàng rào: `ci/check-tray-menu.ps1` (A–F) và `ci/check-shelf-tray.ps1` (A–J). Cả hai đỏ nếu hộp chọn tệp hay khay chạy ngay trong tiến trình nền, nếu con không thoát sau khi đóng, hoặc nếu RAM sau tính năng vượt red line.

## Related

- [ADR-0016](0016-kpi-ram-nen-do-bang-private-working-set.md): thước RAM và red line 30 MB.
- [ADR-0015](0015-tempdrops-mot-thu-muc-moi-muc-khong-hoi-shell-luc-tha.md): `SHGetFileInfoW` không gọi lúc thả.
- [Khay thẻ Shelf trên Windows](../features/shelf-tray-windows/overview.md).
- `peekvn/docs/bai-hoc.md` §200–§203.
