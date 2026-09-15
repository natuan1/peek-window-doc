# ADR-0004: Cấu trúc app Windows — bốn project, `Snappy.Interop` là biên Win32 duy nhất

Date: 2026-09-15
Status: Accepted

## Context

[Spec #1](https://github.com/natuan1/peek-window-doc/issues/1) chốt bốn project (`Snappy.Core` / `Snappy.Protocol` / `Snappy.Interop` / `Snappy.Shared`) nhưng không nói **luật nào giữ ranh giới giữa chúng**. Khi implement [Ticket 01](https://github.com/natuan1/peek-window-doc/issues/2), ba câu hỏi phải trả lời trước khi viết dòng đầu tiên, vì cả ba đều rẻ lúc này và rất đắt ở ticket sau:

1. P/Invoke được phép nằm ở đâu? App này sẽ đụng vào Win32, COM shell (`IDropTarget`, `IDataObject`, `IStream`), GSMTC, Bluetooth GATT, CNG, Credential Manager. Rải `[LibraryImport]` theo nhu cầu là cách tự nhiên nhất, và cũng là cách làm cho câu hỏi "app đụng vào những gì của hệ điều hành" trở nên không trả lời được.
2. DPI khai ở đâu? Windows chốt chế độ DPI cho tiến trình **trước** dòng đầu tiên của `Main`.
3. Vẽ bằng gì? [ADR-0002](0002-ui-stack-aot-spike-pass.md) đã chứng minh Windows.UI.Composition sống được dưới Native AOT, nhưng không nói mọi cửa sổ đều phải dùng nó.

## Decision

**Bốn project, phụ thuộc một chiều**: `Shared` ← `Interop` ← `Protocol` ← `Core`. `Shared` không tham chiếu project nào — thứ gì cần `Interop` thì không phải "shared".

**Mọi khai báo `[LibraryImport]`/`[DllImport]` nằm trong `Snappy.Interop`, không nơi nào khác.** Bề mặt tiếp xúc với hệ điều hành phải đếm được bằng một lần đọc thư mục. Dùng `[LibraryImport]` chứ không `[DllImport]`: nó sinh mã marshalling lúc biên dịch nên AOT không phải dựng stub lúc chạy.

**`Snappy.Core` là project duy nhất sinh exe**, kiểu `WinExe` — hệ quả: không có console, nên chẩn đoán đi qua `AppLog` ghi file, cộng `OutputDebugStringW` cho những chỗ chính `AppLog` không còn hoạt động.

**Per-Monitor V2 khai trong `app.manifest`, không gọi `SetProcessDpiAwarenessContext` lúc chạy.** Gọi lúc chạy là muộn: hệ điều hành đã chốt chế độ DPI trước khi `Main` chạy, nên cửa sổ đầu tiên vẫn có thể bị kéo dãn mờ. Manifest là chỗ duy nhất nói kịp.

**Cửa sổ tĩnh vẽ bằng GDI; Composition dành riêng cho Mí.** Bảng trạng thái là bốn dòng chữ không animation — kéo cả `DispatcherQueue` vào để tô nó là trả giá bằng RAM nền, thứ đang bị KPI 25MB soi.

**Không khai `activeCodePage` UTF-8 trong manifest**: nó chỉ có từ Windows 10 1903, còn floor là 1809 — trên 1809 nó bị bỏ qua lặng lẽ. App gọi toàn API `*W` nên không cần tới nó.

## Consequences

**Được:**
- Mọi rủi ro tương thích Windows tập trung ở một project. Khi Ticket 11 thêm COM shell, chỗ phải đọc lại vẫn là một.
- `Snappy.Shared` và `Snappy.Protocol` không đụng Win32 → test được trên bất kỳ máy nào, và về sau chạy được trong interop suite mà không cần một màn hình.
- Manifest thay vì API lúc chạy: không có cửa sổ nào có thể sinh ra trước khi chế độ DPI đúng.

**Mất:**
- Mỗi lần thêm một API Win32 là thêm một file `Interop`, kể cả khi chỉ một chỗ gọi. Đây là giá cố ý trả.
- GDI cho cửa sổ tĩnh nghĩa là app có **hai** đường vẽ (GDI + Composition) chứ không một. Ranh giới phải rõ, nếu không sẽ có người vẽ Mí bằng GDI cho nhanh.
- `WinExe` khiến mọi lỗi lúc khởi động biểu hiện giống hệt nhau — "bấm vào không thấy gì". Nhật ký không phải tuỳ chọn ở đây, nó là thứ duy nhất phân biệt được.

## Related

- [ADR-0002](0002-ui-stack-aot-spike-pass.md) — spike xác nhận Native AOT + Win32 + Composition
- [Spec #1](https://github.com/natuan1/peek-window-doc/issues/1), [Ticket 01 (#2)](https://github.com/natuan1/peek-window-doc/issues/2)
- [Nền móng app Windows](../features/nen-mong-app-windows/implementation.md)
