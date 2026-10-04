# Onboarding 3 bước — quy trình

## Lần đầu cài

1. Người dùng chạy `Snappy-win-Setup.exe`. Velopack cài vào `%LocalAppData%\Snappy`, chạy hook `--veloapp-install`, rồi mở app với `VELOPACK_FIRSTRUN`.
2. `Program` thấy lần-chạy-đầu → `AppHost.FirstRunAfterInstall`. `LoadSettings` ghi `onboarding=pending` vào `settings.ini`.
3. Cuối `AppHost.Run`, hẹn còn → tạo tệp mẫu (`Onboarding\Snappy - tệp mẫu.txt`) → mở tiến trình con `Snappy.exe --onboarding <tệp mẫu>`, gửi cài đặt hiện tại qua stdin.
4. Con dựng cửa sổ giữa vùng làm việc của màn hình có con trỏ, xin foreground (📐 `foreground=True` sau Setup thật).

## Bước 1 — kéo tệp mẫu

1. Người dùng nhấn giữ thẻ → `DragDetect` → `DoDragDrop` ngay trong tiến trình con (nguồn kéo nạp tầng kéo-thả 17 MB — chỉ ở con).
2. Mí của tiến trình nền thấy lượt kéo, nở, hiện Đích thả. Thả vào ô Shelf (hay lên một thiết bị — khi ấy tệp mẫu được gửi đi).
3. `TakeDropItems` thấy đường dẫn tệp mẫu trong Mục vừa vào → `sample-taken` xuống con → "Xong!".
4. Chưa thả vẫn bấm "Tiếp" được.

## Bước 2 — bên của Mí, phím tắt

1. Bấm "Bên phải" → con gửi `side right` → cha đặt `MiWindow.Side`, gửi `side right` cho capsule, lưu `settings.ini`, gửi lại cài đặt.
2. Bấm một phím → cha nhả phím cũ, `RegisterHotKey` phím mới → `Registered` hoặc `Taken` (mã thật trong nhật ký), lưu, gửi lại. Cửa sổ vẽ câu theo kết quả cha gửi, không theo cú bấm.
3. Khay Shelf mở sau đó nhận bên qua đối số dòng lệnh `--shelf-tray right`.

## Bước 3 — QR, ghim icon

QR vẽ từ `QrCode.Encode(OnboardingFlow.DownloadUrl)`, mỗi ô một `FillRect`, lề yên tĩnh 4 ô. Bên dưới: tiêu đề "Ghim Snappy ra thanh tác vụ" và dấu ^ vẽ bằng glyph ChevronUp.

## Kết thúc

- "Xong" ở bước 3 → `finished done 3`; bỏ qua/✕/Esc → `finished skipped N`. Cha ghi `onboarding=done`; con thoát mã 0.
- Thoát Snappy giữa chừng → hẹn còn → lần mở sau hiện lại.
- Mở từ menu Start sau đó: không có lần-chạy-đầu, hẹn đã gỡ → không tự hiện.

## Phím tắt Shelf

Đăng ký mỗi lần khởi động theo `settings.ini` (mặc định `Win+Alt+S`). Bấm → `OpenShelf()` như mục menu "Mở Shelf". Menu khay ghi phím sau dấu tab chỉ khi đăng ký được.
