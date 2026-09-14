# ADR-0002: Xác nhận UI stack qua spike `aot-footprint` (Native AOT + Win32 + Composition)

Date: 2026-09-14
Status: Accepted

## Context

Quyết định 2026-09-13 chọn UI stack **C# Native AOT + Win32 + Windows.UI.Composition/Direct2D (CsWinRT)** nhưng đặt điều kiện: *spike AOT bắt buộc trước khi cam kết*, với KPI bộ cài **<15MB**, RAM mục tiêu **<25MB** (red line 30MB).

Spike `aot-footprint` được dựng trong `peekvn\apps\windows\spikes\` (commit `2e9e135`, branch `prototype/aot-footprint`): cửa sổ strip 1200×48 luôn-on-top (`WS_EX_TOPMOST | WS_EX_NOACTIVATE | WS_EX_TOOLWINDOW`), gắn Compositor qua `ICompositorDesktopInterop`, dựng SpriteVisual mô phỏng 4 ô Mí, đo sau 3s render + GC.

## Decision

**UI stack được XÁC NHẬN** — kết quả đo 2026-09-14 (.NET 10, win-x64, Release):

| Chỉ số | KPI | Đo được | Kết luận |
|---|---|---|---|
| Exe self-contained | <15MB | **3.05MB** | PASS (~20% ngân sách; Velopack nén còn nhỏ hơn) |
| Working set | <25MB (red line 30MB) | **14.62MB** | PASS (private working set chỉ 4.65MB) |
| Composition dưới AOT | phải chạy được | **OK** | Compositor + DesktopWindowTarget + SpriteVisual sống |

## Consequences

### Positive
- Ba rủi ro kiến trúc lớn nhất của Mí (kích thước, RAM, khả thi Composition-under-AOT) đều gỡ.
- Mô hình interop đã được chứng minh trong spike, tái dùng thẳng cho app thật.

### Bài học interop (chi tiết trong source spike)
1. **Compositor bắt buộc DispatcherQueue** trên thread — nhưng projection TFM 19041 không có `CreateOnCurrentThread`; phải gọi native `CreateDispatcherQueueController` (`coremessaging.dll`, `DQTYPE_THREAD_CURRENT` + `DQTAT_COM_ASTA`).
2. **IID phải tra SDK header** (`winrt/windows.ui.composition.interop.h`): `ICompositorDesktopInterop` = `29E691FA-4567-4DCA-B319-D0F207EB6807`. Ghi GUID từ trí nhớ sai → QI trả E_NOINTERFACE, mà .NET ánh xạ HRESULT này thành `InvalidCastException` "Specified cast is not valid" — gây hiểu nhầm là lỗi cast thay vì lỗi interface.
3. **`Marshal.QueryInterface`/`Release` không đáng tin dưới AOT** — gọi vtable tay qua `delegate* unmanaged`.
4. Build lưu ý: `vcvars64` đặt biến môi trường `PLATFORM=x64` làm output rơi vào `bin\x64\`; hoặc xoá biến hoặc `-p:IlcUseEnvironmentalTools=true` khi vswhere chưa sẵn sàng.

## Alternatives Considered

- Bỏ qua spike, cam kết luôn: rủi ro đã được loại trừ bằng đo đạc, không cần cân nhắc lại.
- (Giữ nguyên các phương án đã loại tại quyết định 2026-09-13: WPF, WinUI 3, Kestrel.)

## Related

- [ADR-0001](0001-brand-khac-service-mdns.md) — brand ≠ service name
- [CONTEXT.md](../CONTEXT.md) — quyết định UI stack 2026-09-13
- Primary source: `peekvn` branch **`prototype/aot-footprint`** (commit `2e9e135`), `apps/windows/spikes/aot-footprint/`
- Kế tiếp: spike resumable upload `104` qua HTTP/1.1 (gate `resumableUpload`)

## Decision Log
- 2026-09-14: Spike PASS, xác nhận UI stack
