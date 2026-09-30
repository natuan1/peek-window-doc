# Mí — kiểm thử

## Unit test (`peekvn/apps/windows/tests/Snappy.Tests`)

Có 37 test.

- **`MiStateMachineTests` (26 test).** Mỗi mép thời gian được kiểm một mili-giây trước và đúng lúc: dwell 80 ms, trễ rời vùng 250 ms, ân hạn 300 ms. Ngoài ra còn các ca sau:
  - lướt nhanh qua vùng;
  - lượt kéo không mang tệp;
  - ảnh kéo bị dựng lại giữa lượt;
  - fullscreen trước khi kéo và trong khi kéo;
  - nhả chuột thì Mí về Nghỉ ngay;
  - lượt kéo bắt đầu trong lúc Đang chạy.
- **`MiLayoutTests` và `FullscreenRuleTests` (11 test).** Bao phủ 30% / 5% của màn hình, DPI 96–192, màn hình toạ độ âm, và pill. Về fullscreen: có 3 mã QUNS, cửa sổ phóng to còn thanh tiêu đề, màn hình nền, và game ở màn hình khác.

Mỗi dòng giữ một bất biến trong máy trạng thái đều có một phép đột biến gỡ đúng dòng đó. Cả 6 đột biến đều bị giết. Hai đột biến ban đầu sống sót do guard kép, nên thêm hai test.

## Nghiệm thu bằng lượt kéo thật: `ci/check-mi-drag.ps1`

Script kéo một tệp **thật** trong Explorer bằng `SendInput` và đọc kết quả từ nhật ký của app. Nó chiếm chuột khoảng 20 giây nên **không** nằm trong Jenkins: agent Jenkins chạy trên chính máy dev.

| Kịch bản | Khẳng định |
|---|---|
| Cửa sổ | Có TOPMOST, TOOLWINDOW, NOACTIVATE; không có TRANSPARENT, LAYERED |
| A. Lướt nhanh | Gợi ý hiện; Mí **không** nở; nhả chuột thì ẩn ngay |
| B. Dừng trong vùng | Nở sau ≥ 80 ms; animation ≤ 200 ms; rời < 250 ms thì không co; tiêu điểm vẫn ở Explorer |
| D. Giữ chuột mà không có ảnh kéo | Polling thấy, Gợi ý hiện; nhả chuột thì ẩn ngay |
| E. Như D, nhưng trong cửa sổ không viền phủ kín màn hình | Nhận ra là fullscreen; Mí **không** hiện |

D là đối chứng của E: cùng một cử chỉ, chỉ khác đúng một thứ. Script cũng in working set trước và sau lượt kéo đầu tiên.

## Treo

| Chưa kiểm | Thiếu gì |
|---|---|
| Cắm/rút màn hình (`WM_DISPLAYCHANGE`); DPI 125–200% (`WM_DPICHANGED`) | Màn hình thứ hai |
| Fullscreen exclusive (Direct3D) | Một game D3D exclusive; hiện mới có unit test |
| Trạng thái 3/4 bằng tay | Ticket 13 |
