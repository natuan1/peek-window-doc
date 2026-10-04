# Lịch sử clipboard — kiểm thử

## Test đơn vị (`Snappy.Tests`)

`ClipboardHistoryTests` (trần Free/Pro, trùng lên đầu, rỗng, tạm dừng, xoá sạch, đổi gói hai chiều, quá lớn, `ToString` không lộ chữ), `ClipboardPrivacyTests` (từng dấu, bộ dấu KeePassXC thật, `CanInclude=1` được giữ, dấu chặn thắng dấu cho phép), `ClipboardPanelLayoutTests` (trong vùng làm việc ở 4 DPI, màn hình âm, hit-test, cuộn theo dòng), `TrayMenuTests` (mục, nhãn phím tắt). 📐 896/896 xanh.

## Nghiệm thu bằng cử chỉ người dùng (bản AOT, ngoài gói MSIX)

| Kịch bản | 📐 04/10/2026 |
|---|---|
| `ci/check-clipboard.ps1` | Ctrl+C vào lịch sử sau 17–25 ms; Free dừng ở 5/5; 5 biến thể dấu bị bỏ không đọc chữ, đối chứng `CanInclude=1` giữ; `Win+Alt+V` → ↓ Enter dán đúng mục dù con trỏ đứng yên trên dòng cuối; tạm dừng/tiếp tục/xoá sạch/menu khay; không chữ nào trong nhật ký hay dưới `%LOCALAPPDATA%\Snappy` |
| `ci/check-clipboard-keepassxc.ps1` | KeePassXC 2.7.12 thật, nút "Copy Password" → bỏ qua (`…FromMonitorProcessing`); `keepassxc-cli clip` không dấu → giữ |
| `ci/check-license.ps1` | Free `keeping 5`; kích hoạt → không giới hạn; Pro giữ 7; gỡ Slot → `keeps 5 entries; dropped the 2 oldest` |

RAM tiến trình nền: nghỉ 5,12 / 21,62 MB → sau bảng 5,44 / 23,39 MB (private / tổng), dưới KPI 25 MB private và red line 30 MB tổng.

## Treo

1Password, Bitwarden — cần tài khoản.
