# Mí — cài đặt

Mã ở `peekvn/apps/windows/`. P/Invoke và COM chỉ nằm trong `Snappy.Interop`
([ADR-0004](../../adr/0004-cau-truc-app-windows-bon-project.md)). Vẽ bằng layered
window, không Composition ([ADR-0014](../../adr/0014-mi-ve-bang-layered-window-khong-composition.md)).

| Tệp | Vai |
|---|---|
| `src/Snappy.Core/MiStateMachine.cs` | Năm trạng thái, logic thuần. Thời gian là tham số: `NextDeadline` + `Tick` thay cho đồng hồ. Lệnh vẽ suy ra ở một chỗ duy nhất (`CommandsFor`). |
| `src/Snappy.Core/MiLayout.cs` | Khung cửa sổ và `MiShape` (thân, bán kính, màu, vùng bắt con trỏ) theo màn hình + DPI: 30% / 5%; Gợi ý 6 DIP trên vùng 60 DIP; khay 90 DIP; pill 180×32. |
| `src/Snappy.Core/MiPainter.cs` | Tô `MiShape` thành BGRA premultiplied. Đen 0,8, không viền, hai góc dưới khử răng cưa. Vùng thả ngoài thân vẽ alpha 1. |
| `src/Snappy.Core/MiAnimation.cs` | Cubic ease-out 160 ms giữa hai `MiShape`, thời gian là tham số. |
| `src/Snappy.Core/FullscreenRule.cs` | Fullscreen nếu QUNS báo, hoặc cửa sổ không viền phủ kín màn hình. |
| `src/Snappy.Core/MiWindow.cs` | Nối dây: hook + polling + OLE → máy trạng thái → `MiPainter` → `LayeredSurface`. Vẽ từng khung trên `WM_TIMER` 10 ms. |
| `src/Snappy.Interop/LayeredSurface.cs` | DIB section top-down giữ theo sức chứa, và `UpdateLayeredWindow` đặt vị trí, cỡ, pixel trong một lời gọi. |
| `src/Snappy.Interop/WinEventHook.cs` | `SetWinEventHook` out-of-context, callback `[UnmanagedCallersOnly]`. |
| `src/Snappy.Interop/DropTarget.cs` | `IDropTarget` qua `[GeneratedComInterface]` + `StrategyBasedComWrappers`. `IDataObject::QueryGetData` gọi qua vtable tay. |
| `src/Snappy.Interop/Win32/MiNative.cs` | Khai báo Win32/COM của Mí. |

## Cửa sổ

`WS_POPUP` với `WS_EX_TOPMOST | WS_EX_NOACTIVATE | WS_EX_TOOLWINDOW | WS_EX_LAYERED`.
📐 Đọc lại từ cửa sổ thật được `0x08080088`.

⚠️ Pixel alpha 0 của layered window là click-through và không là đích thả, vì OLE
chọn đích bằng `WindowFromPoint`. Vì vậy phần vùng thả dưới vạch Gợi ý vẽ alpha 1.
`WM_MOUSEACTIVATE` trả `MA_NOACTIVATE`.

## Số đo (📐 30/09/2026, AOT, 1920×1080 @ 96 dpi)

| | Giá trị |
|---|---|
| Bộ cài | 10,8 → **10,84 MB** (KPI < 15) |
| exe | 9,37 → 9,46 MB |
| Working set nghỉ, chưa kéo lần nào | 20,33 → **20,48 MB** (đo xen kẽ với `main`) |
| Working set nghỉ, **sau** lần kéo đầu tiên | **23,3 MB**, private 7,14 MB (bản Composition: 26,7 MB) |
| Vào vùng → nở | 87–108 ms (dwell 80 ms + nhịp timer ~15,6 ms) |
| Animation nở, tới khung cuối | 172 ms / 11 khung (~64 khung/giây) |

Biên tới KPI 25 MB **sau lần dùng Mí đầu tiên** còn ~1,7 MB. Đây là con số cần
nhìn trước Ticket 11.
