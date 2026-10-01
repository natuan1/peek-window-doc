# ADR: KPI RAM nền của Snappy đo bằng private working set; luồng UI tắt IME

Date: 2026-10-01
Status: Accepted

## Context

KPI "RAM mục tiêu < 25 MB, red line 30 MB" (chốt 2026-09-13, xem [ADR-0002](0002-ui-stack-aot-spike-pass.md)) chưa bao giờ nói **thước nào**. Mọi hàng rào của `peekvn/apps/windows/ci` đo `WorkingSet64`, tức working set tổng.

Hai phép đo ngày 2026-10-01 (bài học 197 và 199 của `peekvn`):

1. **Mở menu khay một lần: 21,0 → 25,5 MB**, trên cả `main`. `SetForegroundWindow` trước menu, thứ bắt buộc để menu tự đóng khi bấm ra ngoài, kích hoạt Text Services Framework cho luồng UI. Bảy DLL nạp thêm và không bao giờ nhả.
2. Sửa xong (1), người đã **kéo thả và mở menu** vẫn ở **25,1–25,8 MB**: OLE drag-drop cộng 3,2 MB, menu cộng 1,6 MB. Cùng mốc ấy, **private working set chỉ 5,5 MB**. Phần còn lại là trang của DLL hệ thống dùng chung với mọi tiến trình. Không cách viết mã trung thực nào bớt được nó.

| Mốc | working set tổng | private working set |
|---|---|---|
| khởi động | 20,6 MB | 4,7 MB |
| sau kéo thả | 23,8 MB | 5,3 MB |
| sau kéo thả + menu (`main`) | 28,4 MB | 5,9 MB |
| sau kéo thả + menu (đã sửa) | 25,1 MB | 5,5 MB |

## Decision

1. **KPI = private working set < 25 MB.** Đây là đúng cột "Memory" mặc định của Task Manager, thứ người dùng thật nhìn thấy. Working set tổng vẫn được in ra và vẫn chặn ở **red line 30 MB**, vì một DLL nặng mới nạp sẽ lộ ra ở đó. Chủ dự án chọn phương án này giữa bốn phương án (2026-10-01). Phương án *cắt working set* bằng `SetProcessWorkingSetSize` bị loại, vì nó làm con số đẹp mà không đổi gì.
2. Định nghĩa nằm ở **một** chỗ: `peekvn/apps/windows/ci/ram.ps1`. Mọi kịch bản RAM dot-source nó, kể cả `smoke-test.ps1` trong Jenkins.
3. **Luồng UI gọi `ImmDisableIME(0)` trước cửa sổ đầu tiên** (`Snappy.Interop/ThreadIme.cs`). Luồng ấy không có ô gõ chữ nào. **Hộp chọn tệp chạy luồng STA riêng**, không có chủ, nên vẫn gõ được tiếng Việt bằng bộ gõ của Windows. 📐 Sau menu: 22,3 MB tổng, so với 25,5 MB trên `main`.

## Consequences

- Biên RAM của Ticket 12 (khay thẻ, icon qua `SHGetFileInfoW` ~2,3 MB tổng) không còn là ~0,6 MB. Theo private nó rộng gần 20 MB, theo red line tổng khoảng 4 MB.
- **Ràng buộc mới cho luồng UI:** mọi ô gõ chữ sau này (đổi tên Mục, ô tìm kiếm…) phải chạy trên luồng khác, hoặc phải xem lại quyết định này. Một ô gõ chữ đặt trên luồng UI sẽ không có IME.
- Trần 25 MB theo private rất lỏng so với 5,5 MB hôm nay. Nó chặn rò bộ đệm lớn, không chặn từng MB. Từng MB vẫn lộ ra ở red line tổng.
- Lỗi có sẵn lộ ra cùng lượt: hộp chọn tệp để lại ~63 MB tổng (private ~11 MB), vượt red line. Tách thành việc riêng.
- `ci/check-tray-menu.ps1` là hàng rào mới. Nó mở menu bằng cú chuột phải thật, kiểm menu tự đóng khi bấm ra ngoài, kiểm luồng UI không có IME, kiểm hộp chọn tệp ở luồng khác và còn IME. Nó đỏ trên `main`, và đỏ khi bỏ `SetForegroundWindow`.

## Related

- [ADR-0002](0002-ui-stack-aot-spike-pass.md): KPI gốc. Spike đã ghi cả hai thước (14,62 MB tổng, private 4,65 MB).
- [ADR-0014](0014-mi-ve-bang-layered-window-khong-composition.md), [ADR-0015](0015-tempdrops-mot-thu-muc-moi-muc-khong-hoi-shell-luc-tha.md): hai quyết định trước đó đều do biên RAM theo thước tổng.
- `peekvn/docs/bai-hoc.md` §197, §199.
