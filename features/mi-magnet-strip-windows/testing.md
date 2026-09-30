# Mí — kiểm thử

## Unit test (`peekvn/apps/windows/tests/Snappy.Tests`)

Có 60 test (tính cả từng ca của `[Theory]`).

- **`MiStateMachineTests` (26 test).** Mỗi mép thời gian được kiểm một mili-giây trước và đúng lúc: dwell 80 ms, trễ rời vùng 250 ms, ân hạn 300 ms. Ngoài ra còn các ca sau:
  - lướt nhanh qua vùng;
  - lượt kéo không mang tệp;
  - ảnh kéo bị dựng lại giữa lượt;
  - fullscreen trước khi kéo và trong khi kéo;
  - nhả chuột thì Mí về Nghỉ ngay;
  - lượt kéo bắt đầu trong lúc Đang chạy.
- **`MiPainterTests` và `MiAnimationTests`.** Kiểm các tính chất sau:
  - pixel đen 0,8;
  - không viền: mép trái, mép phải và hàng trên cùng cùng một màu với giữa khay;
  - hai góc dưới bo tròn, hai góc trên vuông;
  - mép cong có pixel chuyển tiếp (khử răng cưa);
  - không pixel nào của vùng Gợi ý trong suốt hẳn (alpha ≥ 1);
  - bất biến premultiplied của màu Nhắm đích;
  - cubic ease-out đúng công thức, và thời lượng chừa biên dưới 200 ms.
- **`MiLayoutTests` và `FullscreenRuleTests`.** Bao phủ 30% / 5% của màn hình, DPI 96–192, màn hình toạ độ âm, và pill. Về fullscreen: có 3 mã QUNS, cửa sổ phóng to còn thanh tiêu đề, màn hình nền, và game ở màn hình khác.

Mỗi dòng giữ một bất biến trong máy trạng thái đều có một phép đột biến gỡ đúng dòng đó. Máy trạng thái: 6/6 đột biến bị giết. Hai đột biến ban đầu sống sót do guard kép, nên thêm hai test.

Bộ tô pixel: có 6 đột biến. Đột biến "không premultiply" ban đầu sống, vì màu đen có RGB bằng 0 nên nhân với alpha vẫn ra 0; đã thêm test dùng màu Nhắm đích để bắt nó. Đột biến "bỏ nhánh `dy ≤ 0`" cũng sống; nhánh đó là mã chết nên đã xoá.

## Nghiệm thu bằng lượt kéo thật: `ci/check-mi-drag.ps1`

Script kéo một tệp **thật** trong Explorer bằng `SendInput` và đọc kết quả từ nhật ký của app. Nó chiếm chuột khoảng 20 giây nên **không** nằm trong Jenkins: agent Jenkins chạy trên chính máy dev.

| Kịch bản | Khẳng định |
|---|---|
| Cửa sổ | Có TOPMOST, TOOLWINDOW, NOACTIVATE, LAYERED; không có TRANSPARENT |
| A. Lướt nhanh | Gợi ý hiện; `WindowFromPoint` ngay dưới vạch (vùng alpha 1) trả về Mí; Mí **không** nở; nhả chuột thì ẩn ngay |
| B. Dừng trong vùng | Nở sau ≥ 80 ms; animation ≤ 200 ms; rời < 250 ms thì không co; tiêu điểm vẫn ở Explorer |
| D. Giữ chuột mà không có ảnh kéo | Polling thấy, Gợi ý hiện; nhả chuột thì ẩn ngay |
| E. Như D, nhưng trong cửa sổ không viền phủ kín màn hình | Nhận ra là fullscreen; Mí **không** hiện |

D là đối chứng của E: cùng một cử chỉ, chỉ khác đúng một thứ.

Script in working set trước và sau lượt kéo đầu tiên, và **đỏ** nếu con số sau ≥ 25 MB.

📐 Đột biến `ZoneAlpha = 0` trên app thật làm đỏ năm khẳng định cùng lúc. `WindowFromPoint` trả về cửa sổ Chrome bên dưới, và không có `DragEnter` nào tới Mí.

Ảnh chụp đi qua `BitBlt` + `CAPTUREBLT`. Thiếu cờ này thì ảnh không có layered window, và một ảnh "không thấy Mí" sẽ trông y hệt lúc Mí không hiện thật.

## Treo

| Chưa kiểm | Thiếu gì |
|---|---|
| Cắm/rút màn hình (`WM_DISPLAYCHANGE`); DPI 125–200% (`WM_DPICHANGED`) | Màn hình thứ hai |
| Fullscreen exclusive (Direct3D) | Một game D3D exclusive; hiện mới có unit test |
| Trạng thái 3/4 bằng tay | Ticket 13 |
