# Onboarding 3 bước — triển khai

Tất cả trong `peekvn/apps/windows/src/Snappy.Core/` trừ khi ghi khác.

| Phần | Tệp | Ghi chú |
|---|---|---|
| Cài đặt bền | `AppSettings.cs` (`AppSettings`, `ShelfHotkey`, `OnboardingState`) | `%LocalAppData%\Snappy\settings.ini`, `key=value`; đọc chịu BOM, CRLF, `#`; dòng lạ chỉ dòng ấy về mặc định; ghi qua `.tmp` rồi `File.Move` |
| Đường dẫn | `Snappy.Shared/SnappyPaths.cs` | `Settings`, `Onboarding` |
| Bên của Mí | `MiLayout.cs` (`MiSide`, `MiSides.Wire/TryParse`) | bên phải = gương từng pixel của bên trái; `MiWindow.Side`, `CapsuleLayout.Home(…, side)`, `ShelfTrayLayout(…, side)` |
| Bước, câu chữ | `OnboardingFlow.cs` | thuần; `Next/Back/Skip/Apply`; `DownloadUrl` |
| Mã QR | `QrCode.cs` | byte mode UTF-8, mức M, phiên bản 1–40, mặt nạ theo N1/N2/N4 |
| Cửa sổ | `OnboardingWindow.cs` | GDI, thanh tiêu đề thường; ghi toạ độ màn hình của từng nút một lần mỗi bố cục (`Onboarding step N layout: …`) cho kịch bản nghiệm thu |
| Ống cha ↔ con | `OnboardingProtocol.cs`, `OnboardingProcess.cs` | dòng UTF-8 ngăn TAB; dòng nhật ký dùng định dạng của `ShelfTrayProtocol` |
| Nối dây | `AppHost.Onboarding.cs` | `LoadSettings`, `ApplyMiSide`, `RegisterShelfHotkey` (id 2), `OpenOnboarding`, `NoteSampleInShelf`, `HandleOnboardingEvents` |
| Menu khay | `TrayMenu.cs` | `OpenOnboarding` (id 10) đứng **trên** "Kích hoạt Snappy Pro…" — `ci/check-license.ps1` chọn mục Pro bằng ba lần ↑ |
| Capsule | `MediaCapsule.cs`, `MediaCapsuleProcess.cs`, `MediaCapsuleWindow.cs` | dòng `side left|right` trên ống; gửi lại mỗi lần con khởi động |

## Giao kèo trên ống

Cha → con: `settings <left|right> <hotkey-id> <registered|taken|off>`, `sample-taken`, `show`.
Con → cha: `side …`, `hotkey …`, `step N`, `finished <done|skipped> N`, `log <Level> <message>`.

## Số đo (📐 2026-10-04, bản AOT)

| | `main` | nhánh |
|---|---|---|
| Bộ cài | 11,18 MB | 11,23 MB |
| exe | 10 395 KB | 10 496 KB |
| Tiến trình nền: menu khay rồi huỷ → mở onboarding rồi bỏ qua | — | 5,34 / 23,95 → 5,42 / 24,18 MB (private / tổng), 0 DLL mới |
