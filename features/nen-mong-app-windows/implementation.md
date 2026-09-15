# Nền móng app Windows — implementation

Mã nguồn: `peekvn/apps/windows/` ([README của module](https://github.com/natuan1/peekvn/blob/main/apps/windows/README.md)). Quyết định kiến trúc: [ADR-0004](../../adr/0004-cau-truc-app-windows-bon-project.md).

## Bốn project

| Project | Vai trò | Luật |
|---|---|---|
| `Snappy.Shared` | `AppInfo` (tên + phiên bản), `SnappyPaths` (gốc dữ liệu user-space) | Không tham chiếu project nào khác |
| `Snappy.Interop` | `NativeMethods` (toàn bộ P/Invoke), `TrayIcon`, `AppIcon`, `WindowClass`, `DisplayScale` | **Mọi** `[LibraryImport]` nằm ở đây |
| `Snappy.Protocol` | ULTP: mDNS (#5), server h1-only (#6), client | Còn rỗng ở ticket này — có chủ ý |
| `Snappy.Core` | `Program`, `AppHost`, `StatusPanelWindow`, `AppLifecycle`, `TrayMenu`, `SingleInstance`, `AppLog` | Project duy nhất sinh exe (`WinExe`) |

Phụ thuộc một chiều: `Shared` ← `Interop` ← `Protocol` ← `Core`.

## File đáng đọc trước

- `src/Snappy.Core/app.manifest` — Per-Monitor V2, `asInvoker`, long-path. Đây là chỗ DPI được quyết định, **không phải** trong mã.
- `src/Snappy.Core/AppLifecycle.cs` — cổng một chiều `Starting → Running → Stopping → Stopped`; `BeginShutdown()` trả `true` đúng một lần.
- `src/Snappy.Core/AppHost.cs` — cửa sổ ẩn, callback khay, menu, đường thoát duy nhất.
- `src/Snappy.Interop/Win32/NativeMethods.cs` — toàn bộ bề mặt hệ điều hành của app, đếm được bằng một lần đọc.
- `Directory.Build.props` — nơi **duy nhất** đặt `<Version>`.

## Số đo (publish Native AOT, máy dev 2026-09-15)

| Chỉ số | Đo được | KPI |
|---|---|---|
| Kích thước exe | 1,69MB | chặn trên của KPI bộ cài 15MB |
| Working set lúc nghỉ | 12,51MB | < 25MB (red line 30MB) |
| File trong thư mục publish | `Snappy.exe` + `Snappy.pdb` | một exe duy nhất |

Nhỏ hơn spike `aot-footprint` (3,05MB / 14,62MB — [ADR-0002](../../adr/0002-ui-stack-aot-spike-pass.md)) vì ticket này chưa kéo Windows.UI.Composition vào. Con số sẽ tăng lại ở Ticket 10 khi Mí dùng Composition thật; **KPI phải đo lại ở đó**, đừng coi 1,69MB là ngân sách còn dư.

## Bốn cái bẫy đã trả giá

**`DefWindowProcW` với HWND rỗng.** Cửa sổ nhận `WM_NCCREATE`/`WM_CREATE` **trước khi** `CreateWindowExW` trả về, tức lúc đó trường `_hwnd` vẫn là 0. Gọi `DefWindowProcW(0, …)` ở đó làm chính lời gọi tạo cửa sổ thất bại với mã 1400 (`ERROR_INVALID_WINDOW_HANDLE`) — một mã lỗi trỏ vào đúng chỗ không có lỗi. Luôn dùng `hwnd` do Windows truyền vào WndProc, không dùng trường đã lưu.

**Số phiên bản đọc nhầm assembly.** `AppInfo.Version` đọc thuộc tính của assembly chứa chính nó — tức `Snappy.Shared`. Đặt `<Version>` ở `Snappy.Core.csproj` thì Shared vẫn lấy mặc định 1.0.0, và bảng trạng thái sẽ ghi 1.0.0 trong khi gói Velopack tên 1.0.1. `<Version>` phải ở `Directory.Build.props` cho cả bốn assembly cùng số.

**Bấm icon khay khi bảng đang mở.** Cú bấm làm bảng mất tiêu điểm **trước khi** lệnh bấm tới nơi, nên tới lúc xử lý thì bảng đã tự ẩn — chỉ nhìn cờ `_visible` thì cú bấm mở lại bảng thay vì đóng nó, và người dùng không đóng được bảng bằng chính cái nút họ vừa mở nó ra. Phải nhớ mốc thời gian vừa ẩn (cửa sổ 300ms).

**Publish AOT không chạy được từ Git Bash.** ILCompiler định vị `vswhere.exe` qua biến `ProgramFiles(x86)`; tên biến có dấu ngoặc là tên không hợp lệ trong shell POSIX nên Git Bash lặng lẽ bỏ nó, và mọi tiến trình con sinh ra từ đó đều thiếu. Dùng `apps\windows\publish.cmd`, hoặc chạy từ cmd.exe/PowerShell.

## Asset

`assets/snappy.ico` sinh bằng mã (`tools/icongen/IconGen.cs`), không bằng một file thiết kế mà sau này không ai mở lại được. Chín cỡ, DIB cho ≤64px và PNG cho 128/256.
