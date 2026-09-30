# Mí — cài đặt

Mã ở `peekvn/apps/windows/`. P/Invoke và COM chỉ nằm trong `Snappy.Interop`
([ADR-0004](../../adr/0004-cau-truc-app-windows-bon-project.md)).

| Tệp | Vai |
|---|---|
| `src/Snappy.Core/MiStateMachine.cs` | Năm trạng thái, logic thuần. Thời gian là tham số; `NextDeadline` + `Tick` thay cho đồng hồ. Lệnh vẽ suy ra ở một chỗ duy nhất (`CommandsFor`) |
| `src/Snappy.Core/MiLayout.cs` | Khung cửa sổ theo màn hình + DPI: 30% / 5%, 60 DIP (Gợi ý), 90 DIP (nở), pill 180×32 |
| `src/Snappy.Core/FullscreenRule.cs` | QUNS hoặc cửa sổ không viền phủ kín màn hình |
| `src/Snappy.Core/MiWindow.cs` | Nối dây: hook + polling + OLE → máy trạng thái → `SetWindowPos` + `MiView` |
| `src/Snappy.Core/MiView.cs` | Composition: một `ShapeVisual`, một hình chữ nhật bo góc đặt lệch `-bán kính` để chỉ hai góc dưới tròn |
| `src/Snappy.Interop/WinEventHook.cs` | `SetWinEventHook` out-of-context, callback `[UnmanagedCallersOnly]` |
| `src/Snappy.Interop/DropTarget.cs` | `IDropTarget` qua `[GeneratedComInterface]` + `StrategyBasedComWrappers`; `IDataObject::QueryGetData` qua vtable tay |
| `src/Snappy.Interop/CompositionHost.cs` | DispatcherQueue native + `ICompositorDesktopInterop` theo khuôn spike ([ADR-0002](../../adr/0002-ui-stack-aot-spike-pass.md)) |
| `src/Snappy.Interop/Win32/MiNative.cs` | Khai báo Win32/COM của Mí |

## Cửa sổ

`WS_POPUP` với `WS_EX_TOPMOST | WS_EX_NOACTIVATE | WS_EX_TOOLWINDOW | WS_EX_NOREDIRECTIONBITMAP`.
📐 Đọc lại từ cửa sổ thật được `0x08200088`.

Chỗ không có visual thì trong suốt thật mà không cần layered window. Cửa sổ vẫn
bắt con trỏ, nên nó **không** click-through. `WM_MOUSEACTIVATE` trả `MA_NOACTIVATE`.

## Số đo (📐 30/09/2026, AOT, 1920×1080 @ 96 dpi)

| | Giá trị |
|---|---|
| Bộ cài | 10,8 → **11,41 MB** (KPI < 15) |
| exe | 9,37 → 10,70 MB (+1,33 MB, CsWinRT) |
| Working set nghỉ, chưa kéo lần nào | 20,33 → **21,56 MB** (đo xen kẽ với `main`) |
| Working set nghỉ, **sau** lần kéo đầu tiên | ⚠️ **26,7 MB**, private 8,0 MB. Vượt KPI 25, xem [#181](https://github.com/natuan1/peekvn/issues/181) |
| Vào vùng → nở | 87–108 ms (dwell 80 ms + nhịp timer ~15,6 ms) |
| Animation nở, tới lúc DWM báo xong | 172–187 ms qua 6 lượt, đặt 160 ms |

**Thời lượng animation phải đo tới đầu kia.** Đặt 200 ms thì DWM báo xong ở 219 ms.
Đặt 180 ms thì được 187 · 188 · 203 ms, vì độ trễ báo-xong dao động cả một khung hình.
