# Onboarding 3 bước — kiểm thử

## Test đơn vị (`peekvn/apps/windows/tests/Snappy.Tests`)

| Lớp test | Kiểm gì |
|---|---|
| `QrCodeTests` | **ZXing** (bộ giải mã độc lập, chỉ ở project test) đọc lại đúng chuỗi — URL tải app, chuỗi tiếng Việt, chuỗi ≥ phiên bản 7; URL vừa phiên bản 2 (25×25); chuỗi quá dài bị từ chối |
| `AppSettingsTests` | mặc định khi chưa có tệp; ghi/đọc một vòng; tệp Notepad có BOM + CRLF; ghi không BOM; dòng lạ chỉ dòng ấy về mặc định; không để lại `.tmp` |
| `OnboardingFlowTests` | ba bước, Xong ở bước cuối, bỏ qua ở mọi bước, quay lại, câu theo bên của Mí, câu phím đăng ký/bị giữ/tắt |
| `OnboardingProtocolTests` | mọi sự kiện/lệnh đi một vòng; dòng rác không đoán |
| `MiLayoutTests` | bên phải là gương 65–95%, cả màn hình phụ toạ độ âm; pill giữa dải |
| `MediaCapsuleTests`, `ShelfTrayLayoutTests`, `ShelfTrayProtocolTests` | capsule và khay Shelf theo bên; đối số `--shelf-tray right` |
| `TrayMenuTests` | "Mở Shelf (2 mục)\tWin+Alt+S"; "Hướng dẫn bắt đầu" luôn có; ba mục cuối bản Free vẫn là Pro → nhật ký → Thoát |

## Nghiệm thu — `peekvn/apps/windows/ci/check-onboarding.ps1`

Chạy từ **bộ cài thật**, ngoài gói MSIX của app Claude, chiếm chuột/bàn phím (màn che + `Release-Over`), chờ máy rảnh ≥ 60 s. Sao lưu bản cài của người dùng, xoá để Setup thấy máy mới, trả lại bằng `robocopy /MIR` ở cuối. Không nằm trong Jenkinsfile.

A. Setup → tự hiện · B. kéo tệp mẫu vào ô Shelf · C. bên phải; phím bị giữ (kịch bản giữ `Ctrl+Alt+S`) → 1409 → nhả → đăng ký được → bấm mở khay bên phải · D. ZXing đọc **ảnh chụp màn hình** ra `https://snappy.vn/tai` · E. Xong → `onboarding=done` · F. mở lại → không tự hiện, cài đặt còn · G. menu khay mở lại, kéo lên Mí bên phải, ✕ = bỏ qua.

📐 2026-10-04: PASS hai lượt liền. Ảnh và nhật ký ở `%TEMP%\snappy-onboarding-evidence`.

## Chưa đo

- Mốc "60 giây" của story 37 với người dùng thật.
- Đa màn hình / DPI 125–200% (thiếu máy hai màn hình — cùng treo với Ticket 10).
