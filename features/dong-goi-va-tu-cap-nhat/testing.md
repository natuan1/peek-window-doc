# Đóng gói & tự cập nhật — kiểm thử

## Chia việc: cái gì kiểm được ở đâu

| Câu cần chứng minh | Kiểm ở đâu | Vì sao ở đó |
|---|---|---|
| Nhịp 4 giờ, máy trạng thái, chữ cho người dùng | xUnit, `UpdateCoordinatorTests` | Đồng hồ giả tua được; ngồi chờ bốn tiếng thì không ai làm |
| Bộ cài < 15MB và không đòi UAC | `ci/check-installer.ps1` | Đọc thẳng manifest nhúng — UAC do đúng một dòng quyết định |
| Gói vá chỉ mang phần đã đổi | `ci/check-delta.ps1` | Tự dựng phần đã đổi rồi đo |
| Chữ ký hợp lệ | `ci/verify-signature.ps1` | 🕓 Treo — chưa có chữ ký thật |
| Cài thật, cập nhật thật, gỡ thật | **Bằng tay** | Cần một máy có màn hình và một người bấm |

## Biên phụ: 71 test xUnit (34 test mới)

```sh
dotnet test apps/windows/Snappy.slnx
```

`UpdateCoordinatorTests` kiểm hành vi bên ngoài của vòng cập nhật, với một `IUpdateSource` giả:

| Nhóm | Kiểm cái gì |
|---|---|
| Nhịp | Lượt đầu sau `FirstCheckDelay`, rồi **đúng 4 giờ** một lượt — `FakeTimeProvider` tua đồng hồ |
| Dừng đúng lúc | Có bản vá chờ sẵn → không kiểm nữa; chạy từ `publish\` → không hỏi mạng lần nào |
| Đường hỏng | Mất mạng không đổ vỡ app và không hứa bản nào; hỏng một lần rồi vẫn kiểm lại được |
| Áp bản vá | Thoát khi có bản vá → hẹn áp; không có gì chờ → không hẹn gì; bấm "khởi động lại ngay" → áp ngay; không có bản vá → `TryApplyNow()` trả `false` |
| Huỷ | Bấm Thoát giữa hai lượt kiểm không để `OperationCanceledException` lọt ra ngoài |
| Chữ cho người dùng | Mọi trạng thái nói được thành tiếng Việt; **không** lọt `Exception`/`HTTP`/`null` ra UI (AGENTS.md §7.2); câu "có bản vá" phải nói rõ phải làm gì; câu "kiểm hỏng" phải nói rõ **không cần làm gì** và **không đổ lỗi cho mạng**; tooltip ≤ 127 ký tự (shell cắt ở đó) |

`TrayMenuTests` mở rộng: mục "Khởi động lại để cập nhật" chỉ xuất hiện khi có bản vá; "Thoát" vẫn là mục cuối; **id của một lệnh không đổi khi menu dài ngắn khác nhau** — id đánh theo vị trí thì một hôm nào đó cú bấm "Thoát" rơi vào lệnh khác.

**Không** test in-process phần Velopack thật (tải gói, vá, khởi động lại): nó là adapter ra thế giới bên ngoài.

## Hàng rào CI — Jenkins, chín bước

Job `snappy-windows`, agent Windows 11 thật ([ADR-0005](../../adr/0005-ci-cd-desktop-qua-jenkins-noi-bo.md)). Bốn bước cuối là mới:

6. **Đóng gói Velopack** — `ci/pack.ps1`. Credential ký số chỉ được nạp khi build chạy với tham số `PHAT_HANH`.
7. **Hàng rào bộ cài** — `ci/check-installer.ps1`: KPI 15MB + manifest `asInvoker`.
8. **Gói vá delta** — `ci/check-delta.ps1`.
9. **Kiểm chữ ký** — `ci/verify-signature.ps1`.

Mọi script chạy được y hệt trên máy dev:

```
powershell -File apps\windows\ci\pack.ps1
powershell -File apps\windows\ci\check-installer.ps1
powershell -File apps\windows\ci\check-delta.ps1
powershell -File apps\windows\ci\verify-signature.ps1
```

### Bước `Kiểm chữ ký` có ba màu, không hai

| Mã thoát | Nghĩa | Jenkins |
|---|---|---|
| 0 | `signtool verify /pa /v` đã chạy và pass | 🟢 xanh |
| 2 | Chưa ký — signtool **chưa hề được gọi** | 🟡 vàng |
| khác | Chữ ký hỏng | 🔴 đỏ |

Gộp "chưa ký" vào màu xanh là nói dối đúng chỗ dễ tin nhất: một bước xanh ở đó sẽ đọc thành "chữ ký đã kiểm và hợp lệ". Ở lượt `PHAT_HANH` thì `-Require` bật, chưa ký là đỏ, không còn cửa vàng.

### `check-delta.ps1` tự dựng phần "đã đổi"

Script chép thư mục publish ra chỗ khác, **nhét thêm 1 MB dữ liệu ngẫu nhiên**, rồi đóng gói thành 1.0.1 và đo gói vá. Kết quả: gói vá **1,01 MB = 15 %** gói đầy đủ, có chặn dưới (phải mang nổi ít nhất nửa phần đã đổi) và chặn trên (25 %).

**Bản đầu của hàng rào này sai.** Nó đóng gói 1.0.1 từ **đúng** bộ nhị phân 1.0.0, nên đo gói vá của *không có gì thay đổi*: 11,5 KB, 0,2 %. Con số ấy đúng nhưng vô nghĩa — nó cũng đúng với một cơ chế delta đã hỏng hoàn toàn, và cái trần 25 % không bao giờ đỏ được. Chính con số 11,5 KB cũ là thứ chặn dưới mới bắt được. Ghi ở [lessons-learned](../../lessons-learned.md).

Script **gọi lại `pack.ps1`** thay vì chép 11 cờ `vpk` sang: `--packId`, `--instLocation PerUser`, `--shortcuts` là những thứ spec #1 ràng buộc, và bản chép thứ hai sẽ lệch lặng lẽ đúng vào ngày có người sửa bản gốc.

### Hàng rào "không UAC" đo cái gì — và không đo cái gì

`check-installer.ps1` có **hai** phép thử, và chỉ hai:

1. Bộ cài < 15MB.
2. Manifest nhúng là `asInvoker`, không `requireAdministrator`/`highestAvailable`.

**"Cài vào user-space" cố ý KHÔNG có phép thử riêng.** `--instLocation` không để lại dấu vết nào trong artifact để soi (nuspec chỉ ghi `shortcutLocations`), nên mọi thứ viết ra cho câu đó sẽ là một dòng luôn xanh — và một hàng rào luôn xanh tệ hơn không có hàng rào. Phép thử 2 thay thế nó: một bộ cài `asInvoker` không ghi nổi vào Program Files, nên không-UAC và user-space là cùng một sự thật nhìn từ hai phía.

## Mọi đường đỏ đã được ép cho đỏ

Một hàng rào chưa bao giờ đỏ là một hàng rào chưa được chứng minh.

| Hàng rào | Ép đỏ bằng | Kết quả |
|---|---|---|
| `check-installer` | Bộ cài đòi UAC (`sdbinst.exe` đổi tên) | ✅ đỏ |
| `check-installer` | Bộ cài không nhúng manifest (`dism.exe`) | ✅ đỏ |
| `check-installer` | Vỡ trần dung lượng (`-MaxSetupMB 5`) | ✅ đỏ |
| `check-installer` | Thiếu bộ cài | ✅ đỏ |
| `check-delta` | Gói vá vượt trần tỉ lệ (`-MaxDeltaPercent 10`) | ✅ đỏ |
| `check-delta` | Gói vá không mang nổi phần đã đổi | ✅ chặn dưới bắt được 11,5 KB của bản cũ |
| `pack` | Đóng gói bản build 1.0.0 thành gói 1.0.1 | ✅ đỏ |
| `verify-signature` | `-Require` trên bộ cài chưa ký | ✅ đỏ (mã 1) |
| `verify-signature` | Không `-Require`, chưa ký | ✅ vàng (mã 2) |

## Còn phải kiểm bằng tay

Lượt cài + cập nhật + gỡ đầy đủ **đã** đi bằng tay 2026-09-16 — chi tiết và bằng chứng ở [Workflow](workflow.md).

Ba bề mặt chưa đi bằng mắt, vì CI không có Explorer:

1. Dòng cập nhật trên bảng trạng thái, và nó có sáng lên khi có bản vá không.
2. Tooltip khay khi rê chuột qua icon lúc có bản vá chờ.
3. Mục menu "Khởi động lại để cập nhật" — có hiện đúng lúc, và bấm vào có vá thật không.

**Treo tới khi có tài khoản Azure Trusted Signing:** `signtool verify /pa /v` pass, và SmartScreen không cảnh báo trên một máy Windows sạch.
