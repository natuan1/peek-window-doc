# Capsule media — kiểm thử

## Test tự động — `dotnet test apps/windows/Snappy.slnx`

765 test xanh (706 trước ticket).

| Lớp test | Khẳng định |
|---|---|
| `MediaCapsuleTests` | **Hiện/ẩn:** ẩn khi không có phiên; đang phát thì hiện mãi; tạm dừng nán đúng 30 s; sự kiện lặp không kéo dài thời gian nán; **phiên đã dừng từ trước không hiện**, kể cả khi đổi sang nó. **Chữ:** dòng chữ bỏ phần thiếu và rơi về tên app. **Ô pin:** không tai nghe → không ô; 1–100 → `n%`; `null`/0/101/255 → `-`, **không bao giờ `0%`**; hai bên → số nhỏ nhất; chỉ nhóm `Audio` là tai nghe (đối chứng: chuột không phải). **Chỗ đứng:** nhà giữa dải Mí ở 96/144/192 dpi; Mí ở mọi trạng thái × 4 DPI thì capsule không chạm khung Mí; Mí ở màn hình khác thì capsule đứng yên; thiếu chỗ thì sang trái, thiếu cả hai thì ẩn; các phần không đè nhau; hit-test. **Giao kèo:** dòng `footprint` khứ hồi, dòng rác không đọc thành khung. **Win32 thật:** `DevGetObjectProperties` trên container "máy này" (`Computer.*`, Connected) |
| `WinRtTests` | `HSTRING` khứ hồi tiếng Việt; delegate dựng tay trả lời đúng IID của nó, `IUnknown`, `IAgileObject`, và từ chối IID khác; `Invoke` gọi callback, và đối tượng còn sống qua `Dispose` khi nguồn sự kiện còn giữ tham chiếu; tra pin trên máy không có Bluetooth thì không ném |

**Đỏ trước khi xanh:** hai test "phiên đã dừng từ trước" được viết sau lượt dò thật đầu tiên, và đỏ trên bản cũ.

## Nghiệm thu thật — `ci/check-media-capsule.ps1`

Chạy trên bản AOT, Snappy **ngoài gói MSIX** (bài học 201), có màn che, chuột thật (`SendInput`). Trước mọi cú bấm, kịch bản kiểm `WindowFromPoint` là capsule.

Hai nguồn nhạc:

- Một trang Chrome, hồ sơ tạm **không** ẩn danh, phát WAV câm và khai `navigator.mediaSession`. Ở chế độ ẩn danh, Chrome báo metadata là "A site is playing media".
- Windows Media Player, mở bằng liên kết tệp mặc định.

⚠️ Kịch bản chiếm chuột ~80 s và không nằm trong Jenkinsfile.

📐 03/10/2026, lượt cuối:

| Kịch bản | Kết quả |
|---|---|
| A. Khởi động, máy có phiên Riot Client đã dừng | `Capsule hidden (Paused)`; tiến trình con `--media-capsule` có mặt |
| B. Chrome phát | `Capsule shown (home) x=234..534`; `Track is "Nơi này có anh — Sơn Tùng M-TP"`; bìa giải mã ở 24 px; `WindowFromPoint` = `SnappyMediaCapsule` |
| C. Bấm ⏸ rồi ▶ trên capsule | `User pressed pause … Chrome accepted`; tiêu đề Chrome đổi `paused` rồi `playing` |
| D. Bấm ⏭ | capsule sang bài 2, Chrome báo `#1` |
| E. Kéo tệp từ Explorer lên Mí | `Strip occupies x=96..672` → `Capsule shown (beside the strip) x=680..980`, trượt mất 162–174 ms. Dưới con trỏ là `SnappyMiStrip`, và capsule vẫn bắt chuột ở chỗ mới. Esc → capsule về nhà |
| F. Cửa sổ không viền phủ kín màn hình | capsule ẩn ngay (`quns=2`); đóng cửa sổ → hiện lại sau 1,5 s |
| G. Tạm dừng | ẩn sau **30 021 ms**; chỗ cũ của capsule là cửa sổ bên dưới |
| H. Đóng Chrome | phiên rơi về Riot Client đã dừng; capsule **không** hiện lại |
| K. Windows Media Player (nhấp đúp tệp `.wav`) | `Media session Microsoft.ZuneMusic…: Playing`; capsule hiện tên tệp, nốt nhạc thay bìa, nút ⏭ mờ (`next off`). Bấm ⏸ → `… ZuneMusic… accepted` |
| I. Pin | `Headset battery: no connected Bluetooth headset with a GATT Battery Service`, `battery none` |
| J. Tắt Snappy | tiến trình con tự thoát theo, vì stdin đóng |

Kịch bản này từng xanh giả một lần, và chuyện ấy được giữ lại làm bài học. Bước F khớp nhầm dòng `Capsule shown (home)` của kịch bản E, trong khi capsule thật ra còn ẩn thêm 4,5 s. Giờ mốc đo được lấy **sau** khi đóng cửa sổ toàn màn hình.

## Số đo

| | `main` | nhánh |
|---|---|---|
| Bộ cài | 11,01 MB | **11,06 MB** |
| exe | 10 042 KB | 10 175 KB |
| Tiến trình nền lúc nghỉ (private / tổng) | 4,82 / 20,8 MB | 5,05 / 21,9 MB, không DLL mới |
| Tiến trình nền sau cả kịch bản A–K | — | 5,82 / 25,5 MB |
| Tiến trình capsule lúc nghỉ / đang phát | — | 4,3 / 22,1 MB · 5,3 / 27,8 MB |
| `smoke-test.ps1` | | đạt, 0 tiến trình sót sau `WM_CLOSE` |

## Không đo được

- **Pin với tai nghe BLE thật**: máy không có Bluetooth adapter.
- **Spotify**: máy không cài.
- **Đa màn hình, DPI ≠ 100%**: máy chỉ có một màn hình ở 100%. Bố cục có test ở 96–192 dpi.
- **`check-mi-drag.ps1`** đỏ bốn khẳng định ở A/B trên **cả `main`** (worktree riêng, dùng exe và kịch bản của `main`). Đây là lỗi có sẵn, đã tách thành việc riêng. Kịch bản E (fullscreen) của nó vẫn chạy sau khi tách `FullscreenRule.Measure`.
