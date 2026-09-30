# ADR-0014: Mí vẽ bằng layered window, không Composition

Date: 2026-09-30
Status: Accepted. Thay [ADR-0002](0002-ui-stack-aot-spike-pass.md) **cho riêng Mí**.

## Context

[ADR-0002](0002-ui-stack-aot-spike-pass.md) chọn Windows.UI.Composition để vẽ Mí, dựa trên một spike đo trên **exe trống**: 14,6 MB working set. Ticket 10 ([#141](https://github.com/natuan1/peekvn/issues/141)) làm đúng khuôn ấy. Composition và OLE được dựng lười ở lần Mí hiện đầu tiên ([ADR-0013](0013-mi-an-khi-nghi-dung-luoi-xac-nhan-tep-qua-dragenter.md)), vì vậy RAM lúc nghỉ đạt KPI: 21,56 MB, và `smoke-test.ps1` xanh.

📐 Đo 30/09/2026 **sau** lượt kéo tệp đầu tiên, lúc app đã về Nghỉ: **26,7 MB, vượt KPI 25 MB**. Lượt kéo ấy nạp sáu DLL hệ thống và không bao giờ nhả chúng. Bốn trong số đó là của Composition: `dcomp`, `coremessaging`, `twinapi.appcore`, `Microsoft.Internal.WarpPal`. Không có cách viết mã nào thoả được cả ADR-0002 lẫn KPI, nên câu hỏi được đưa lên chủ dự án ở [natuan1/peekvn#181](https://github.com/natuan1/peekvn/issues/181).

## Decision

**Chủ dự án chốt 2026-09-30: bỏ Composition cho Mí và vẽ bằng layered window.** Kèm theo là yêu cầu hình: *"màu đen đục, không viền, bo cong nhẹ, opacity 0.8"*.

**Cách vẽ:**

- Cửa sổ `WS_EX_LAYERED`, nội dung đưa lên bằng `UpdateLayeredWindow` với alpha từng pixel (`AC_SRC_ALPHA`).
- Pixel BGRA premultiplied được tô tay trong `MiPainter`, là logic thuần và có test pixel.
- Đọc yêu cầu hình như sau:
  - Đen, alpha 0xCC (0,8), áp đều cả thân.
  - Không viền.
  - Bo 10 DIP ở **hai góc dưới**, vì Mí dính cạnh trên màn hình. Vạch Gợi ý và pill thì bo tròn hẳn hai đầu dưới.
- Góc bo được khử răng cưa bằng độ phủ, tính từ khoảng cách tới tâm cung.
- Tô ở pixel thật của màn hình, nên sắc nét ở mọi DPI mà không có bước co giãn nào.
- App tự vẽ từng khung animation trên một `WM_TIMER` 10 ms (thực tế khoảng 15,6 ms), cubic ease-out 160 ms.

**Vùng thả vẽ alpha 1, không phải 0.** Pixel alpha 0 của một layered window vừa click-through vừa không là đích thả, vì OLE chọn đích bằng `WindowFromPoint`, và phép tra ấy đi xuyên qua pixel trong suốt hẳn. Alpha 1/255 thì không ai nhìn thấy, nhưng Windows vẫn tính là trúng Mí.

📐 Đột biến `ZoneAlpha = 0` trên app thật cho thấy rõ hệ quả: `WindowFromPoint` ngay dưới vạch Gợi ý trả về cửa sổ Chrome bên dưới, không có `DragEnter` nào tới, và Mí không nở được lần nào.

## Consequences

**Được** (📐 30/09/2026, AOT, đo xen kẽ với một bản `main` publish cùng lúc):

| | `main` | Composition | **Layered** |
|---|---|---|---|
| Working set nghỉ | 20,33 MB | 21,56 MB | **20,48 MB** |
| Working set sau lượt kéo đầu | — | 26,7 MB | **23,3 MB** (private 7,14) |
| DLL nạp thêm sau lượt kéo | — | 6 | 3, đều của OLE |
| exe | 9,37 MB | 10,70 MB | 9,46 MB |
| Bộ cài | 10,8 MB | 11,41 MB | 10,84 MB |
| Animation nở, tới khung cuối | — | 172–187 ms (DWM báo) | 172 ms / 11 khung |

Ba DLL còn lại (`clbcatq`, `dataexchange`, `twinapi.appcore`) là giá của OLE drag-drop. Không bỏ được, vì Mí phải là đích thả.

**Mất, và phải nói ra:**

- Biên tới KPI 25 MB **sau lần dùng Mí đầu tiên** chỉ còn **~1,7 MB** cho tám ticket còn lại, không phải 4,5 MB như con số "lúc nghỉ" gợi ra. `ci/check-mi-drag.ps1` giờ đỏ nếu mốc ấy chạm 25 MB.
- Animation chạy trên thread giao diện, không còn chạy trên thread của DWM. Một lời gọi chậm trên thread ấy (một `IDataObject` chậm của Outlook) sẽ làm rơi khung hình. Chuyện này chưa đo.
- Mỗi hiệu ứng mới (bóng đổ, blur, chữ trong khay) phải tự tô. Không có `SpriteVisual` hay brush nào để mượn.
- Vạch Gợi ý đen 0,8 cao 6 DIP rất khó thấy trên thanh tiêu đề tối. Đó là hệ quả của màu đã chốt, và là câu hỏi mở cho chủ dự án.

## Alternatives Considered

- **Giữ Composition và định nghĩa KPI theo private bytes.** Được đề xuất ở #181, chủ dự án không chọn.
- **Giữ Composition và chấp nhận vượt KPI** sau lần dùng đầu tiên. Không chọn.
- **Direct2D trên một HWND thường**: vẫn nạp `d2d1`/`dxgi`, chưa đo, và vẫn phải tự làm per-pixel alpha. Không có lý do đo khi layered window đã vừa ngân sách.
- **Vùng thả alpha 0 kèm `WM_NCHITTEST` trả `HTCLIENT`**: không cứu được, vì với layered window, `WindowFromPoint` bỏ qua pixel trong suốt trước khi hỏi `WM_NCHITTEST`. Đây là suy luận từ đột biến ở trên, chưa đo riêng.

## Related

- [ADR-0002](0002-ui-stack-aot-spike-pass.md): vẫn đúng cho câu *"Composition sống được dưới AOT"*, không còn là cách vẽ Mí
- [ADR-0013](0013-mi-an-khi-nghi-dung-luoi-xac-nhan-tep-qua-dragenter.md): Nghỉ ẩn hẳn, OLE dựng lười (phần Composition của nó hết hiệu lực)
- [natuan1/peekvn#181](https://github.com/natuan1/peekvn/issues/181): câu hỏi KPI và quyết định
- `peekvn/docs/bai-hoc.md` mục 194, 195
