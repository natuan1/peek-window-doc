# Đóng gói & tự cập nhật — tổng quan

**Trạng thái:** đã implement trên nhánh `ticket/02-velopack-ky-so` của `peekvn` (2026-09-16) · [Ticket 02 (#3)](https://github.com/natuan1/peek-window-doc/issues/3)

## Tính năng này là gì

Hai đoạn đời của app mà người dùng gặp trước cả khi dùng được nó: **cài vào máy** và **luôn ở bản mới**. Cả hai phải im lặng tới mức không đáng nhớ.

Ở mức người dùng, nó làm bốn việc:

1. Bấm vào bộ cài là cài xong trong vài giây — không hỏi UAC, không hỏi cài ở đâu, không vào Program Files.
2. App tự tìm bản mới ở nền, mỗi 4 giờ, không hỏi gì.
3. Có bản mới thì tải **gói vá** chứ không tải lại cả bộ cài.
4. Bản vá áp lúc mở lại app. Muốn ngay thì có mục "Khởi động lại để cập nhật" ở menu khay; không thì cứ thoát như mọi khi, lần mở sau đã là bản mới.

## User story

> As a người dùng Windows, I want to app tự cập nhật ngầm theo gói vá nhỏ (delta), so that tôi luôn ở bản mới mà không phải tải lại bộ cài. — Spec #1, story 34

> As a người dùng Windows, I want to cài app không hiện popup UAC, không cài vào Program Files, so that cài/gỡ sạch sẽ trong vài giây. — Spec #1, story 35

## Phạm vi

**Trong:** bộ cài Velopack cài `PerUser`, `VelopackApp` ở entry point, vòng kiểm cập nhật nền 4 giờ, tải gói vá delta, áp bản vá lúc thoát hoặc theo yêu cầu, đường ký số Azure Trusted Signing trong CI, và bốn hàng rào CI mới.

**Ngoài:**

- **Hạ tầng phát hành thật** (tên miền, CDN, nơi chứa `releases.win.json`) — spec #1 đã đẩy hạ tầng web ra ngoài phạm vi. Hiện `VelopackUpdateSource.DefaultFeedUrl` trỏ vào repo `peekvn` và đè được bằng biến môi trường `SNAPPY_UPDATE_FEED`.
- Onboarding sau khi cài (Ticket 17), gồm cả việc dạy người dùng ghim icon ra khỏi phần tràn của khay.
- Tự khởi động cùng Windows.

## Nghiệm thu

| Tiêu chí (#3) | Trạng thái | Bằng chứng |
|---|---|---|
| Bộ cài Velopack user-space, không popup UAC, < 15MB | ✅ | `Snappy-win-Setup.exe` = **10,02 MB**; manifest nhúng là `asInvoker`, kiểm máy móc bằng `ci/check-installer.ps1`; cài thật bằng `--silent` không một popup |
| `VelopackApp` chạy ở entry point | ✅ | Dòng đầu tiên của `Program.Main`, trước cả việc dựng thư mục dữ liệu — xem [ADR-0006 §2](../../adr/0006-dong-goi-velopack-cai-peruser.md) |
| Background kiểm tra cập nhật mỗi 4 giờ | ✅ | `UpdateCoordinator.CheckInterval`; kiểm bằng `FakeTimeProvider` tua đồng hồ, không phải chờ thật. Lượt đầu sau 30 giây — đo thật: nhật ký 22:19:38 → 22:20:08 |
| 1.0.0 → 1.0.1 chỉ tải gói vá nhỏ | ✅ | `ci/check-delta.ps1`: 1 MB đổi → gói vá **1,01 MB = 15 %** gói đầy đủ. Đường delta cũng đã đi thật trên máy: `Downloading delta 1.0.1` → `Applying 1 patches` |
| Áp dụng khi khởi động lại app | ✅ | Cài 1.0.0 → cập nhật → `WM_CLOSE` → mở lại thì `sq.version` ghi 1.0.1 và hook `--veloapp-updated` chạy qua |
| Ký số Azure Trusted Signing trong CI | 🕓 **Treo** | Đường ống dựng xong (`ci/pack.ps1` + `vpk --azureTrustedSignFile`, sáu secret, tham số `PHAT_HANH` ở Jenkins). Thiếu **tài khoản Azure Trusted Signing** — thiếu credential, không thiếu mã |
| `signtool verify /pa /v` pass | 🕓 **Treo** | `ci/verify-signature.ps1` đã viết và đã ép đỏ ở nhánh "chưa ký". Chưa chạy được lượt xanh nào vì chưa có chữ ký thật |
| Cài trên máy sạch SmartScreen không cảnh báo | 🕓 **Treo** | Phụ thuộc cả chữ ký lẫn danh tiếng tích luỹ của chứng chỉ; cần một máy Windows sạch và một bộ cài đã ký |

Ba tiêu chí treo đều treo vì **một lý do duy nhất**: chưa có tài khoản Azure Trusted Signing. Không đánh dấu xanh, và cũng không coi là nợ chặn đường của các ticket sau.

## Giá phải trả

Velopack là thư viện **đầu tiên** của cả bốn project, và nó không rẻ:

| | Ticket 01 | Ticket 02 | KPI |
|---|---|---|---|
| exe | 1,69 MB | **7,71 MB** | — |
| working set lúc nghỉ | 12,51 MB | **15,04 MB** | < 25 MB |
| bộ cài | — | **10,02 MB** | < 15 MB |

Cả hai vẫn dưới KPI, nhưng biên đã hẹp đi thật. Mọi ticket sau nên đọc bảng này trước khi thêm thư viện thứ hai.

## Việc còn phải kiểm bằng tay

Lượt cài + cập nhật đầy đủ **đã** đi bằng tay trên máy dev 2026-09-16 (xem [Workflow](workflow.md)). Phần chưa đi bằng mắt:

1. Dòng cập nhật trên bảng trạng thái.
2. Tooltip khay khi có bản vá chờ.
3. Mục menu "Khởi động lại để cập nhật".

Treo tới khi có tài khoản: `signtool verify /pa /v` và SmartScreen trên máy sạch.

## Tài liệu liên quan

- [Workflow](workflow.md) — người dùng và hệ thống đi qua những bước nào
- [Implementation](implementation.md) — file chính, seam, cái bẫy đã gặp
- [Testing](testing.md) — hàng rào nào tự động, cái gì còn bằng tay
- [ADR-0006](../../adr/0006-dong-goi-velopack-cai-peruser.md) — vì sao `PerUser`, vì sao gốc cài trùng gốc dữ liệu, vì sao KPI chuyển sang bộ cài
- [Nền móng app Windows](../nen-mong-app-windows/overview.md) — Ticket 01, nền mà tính năng này dựng lên
