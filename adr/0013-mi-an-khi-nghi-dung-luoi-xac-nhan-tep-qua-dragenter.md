# ADR-0013: Mí ẩn hẳn khi Nghỉ, dựng OLE/Composition lười, xác nhận tệp qua `DragEnter`

Date: 2026-09-30
Status: Accepted

## Context

[Ticket 10 (#141)](https://github.com/natuan1/peekvn/issues/141) dựng Mí. Kế hoạch 1.1/1.2 và `CONTEXT.md` đã chốt ba điều trước khi có dòng mã nào:

1. Nghỉ (0) là *"ẩn hoặc vệt 3px mờ"*.
2. "Đang kéo FILE" xác nhận bằng *"một lần `OleGetClipboard` + `CFSTR_INDRAGLOOP` + `CF_HDROP`/`FileGroupDescriptorW`"*.
3. UI stack là Composition ([ADR-0002](0002-ui-stack-aot-spike-pass.md)), và RAM nền phải dưới 25 MB.

Trước dòng mã đầu tiên, một đầu dò kéo thật một tệp từ Explorer bằng `SendInput` và đo (📐 30/09/2026). Kết quả phá điều 2 và buộc phải chọn lại điều 1 và 3.

## Decision

**1. Nghỉ là cửa sổ ẩn hẳn, không phải vạch 3px.** Cạnh trên màn hình là chỗ người dùng bấm tab trình duyệt. Một cửa sổ không click-through nằm ở đó lúc không ai kéo gì sẽ cướp đúng những cú bấm ấy. Mí chỉ hiện khi có một lượt kéo, tức lúc chuột đang bị nguồn kéo giữ.

**2. "Là tệp" đọc từ `IDataObject::QueryGetData` trong `IDropTarget::DragEnter`, không từ clipboard.** 📐 `OleGetClipboard` giữa một lượt kéo Explorer trả `DV_E_FORMATETC` cho cả `InShellDragLoop`, `CF_HDROP` và `FileGroupDescriptorW`, y hệt phép đối chứng hỏi trước khi kéo. Clipboard là một kênh khác. Đối tượng của lượt kéo đi thẳng từ `DoDragDrop` tới đích thả dưới con trỏ.

Hệ quả: Gợi ý (1) hiện cho **mọi** lượt kéo (cả chữ, liên kết, bôi chọn qua đường polling). Chỉ bước nở ra Sẵn sàng nhận (2) mới lọc tệp.

**3. OLE (`OleInitialize` + `RegisterDragDrop`) và Composition dựng ở lần Mí hiện đầu tiên, không lúc khởi động.** 📐 Đo xen kẽ ba lượt với một bản `main` publish cùng lúc: làm việc này lúc khởi động đưa working set nghỉ từ 20,3 lên 23,5 MB, trên một biên còn 4,2 MB. Mí ẩn thì không ai thả vào được, nên đích thả chỉ cần có mặt trước lúc nó hiện.

**4. Tắt tín hiệu kéo đi qua ân hạn 300 ms, trừ khi không còn nút chuột vật lý nào giữ.** 📐 Explorer huỷ rồi dựng lại `SysDragImage` giữa một lượt kéo (cái đầu sống 31 ms). Nhả nút thì về Nghỉ ngay, để cửa sổ 60 DIP không nuốt cú bấm kế tiếp.

## Consequences

**Được:**

- Lúc nghỉ, cạnh trên màn hình hoàn toàn là của người dùng.
- RAM nền của Ticket 10 chỉ +1,2 MB (20,33 → 21,56 MB).
- Cơ chế xác nhận tệp là thứ OLE bảo đảm, không phải một mẹo.

**Mất, và phải nói ra:**

- ⚠️ **Dựng lười chỉ dời lúc phải trả.** Sau lượt kéo đầu tiên, app đã về Nghỉ mà working set là **26,7 MB, vượt KPI 25 MB** (dưới red line 30). Private bytes chỉ 8,0 MB (+0,87). Phần còn lại là trang của sáu DLL hệ thống nạp thêm và không nhả: `dcomp`, `coremessaging`, `twinapi.appcore`, `WarpPal` (Composition), `dataexchange`, `clbcatq` (OLE).
- Đó là xung đột giữa điều 3 của Context và ADR-0002, không phải một lỗi mã, nên **quyết định để ở [natuan1/peekvn#181](https://github.com/natuan1/peekvn/issues/181)**. Ba lối ra: KPI theo private bytes, bỏ Composition cho Mí, hoặc chấp nhận vượt sau lần dùng đầu.
- `smoke-test.ps1` đo 6 giây sau khi khởi động, tức trước lượt kéo đầu tiên, nên nó xanh với vụ vượt này. Bài học 194 của `peekvn`.
- Mí không thể gợi ý **riêng** cho tệp trước khi con trỏ tới nó.
- Lần hiện đầu tiên trả thêm độ trễ dựng Composition: 📐 đo được 0–16 ms, ở một thời điểm mà `SysDragImage` đã tới sau 0,7 s kể từ lúc nhấn chuột.

## Alternatives Considered

- **Vạch 3px luôn hiện**: là đích thả sẵn, không cần hook. Bị loại vì nó cướp cú bấm ở cạnh trên màn hình suốt cả ngày.
- **Giữ `OleGetClipboard`**: đã đo, và nó không hoạt động.
- **Dựng OLE/Composition lúc khởi động**: đơn giản hơn. Bị loại vì 2 MB RAM nền cho một máy có thể không bao giờ kéo tệp.
- **Gọi `SetProcessWorkingSetSize` để ép working set xuống**: bị loại, vì đó là làm đẹp con số chứ không đổi bộ nhớ (bài học 162).

## Related

- [ADR-0002](0002-ui-stack-aot-spike-pass.md): UI stack Composition dưới AOT
- [ADR-0004](0004-cau-truc-app-windows-bon-project.md): P/Invoke và COM chỉ trong `Snappy.Interop`
- [Mí magnet strip trên Windows](../features/mi-magnet-strip-windows/overview.md)
- `peekvn/docs/bai-hoc.md` mục 192, 193, 194
