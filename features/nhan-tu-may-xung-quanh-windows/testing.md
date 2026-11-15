# Nhận từ máy xung quanh — kiểm thử

## Tự động

| Bộ | Đo gì |
|---|---|
| C# `AuthFlowTests` (5 test mới) | máy lạ xác thực được, không vào Trust Store; client cũ vẫn `403`; mang `deviceId` của máy đã ghép → `401`; khoá không dẫn ra `deviceId` → `400`; tên hỏng → `400` |
| C# `StrangerOfferTests` (8 test) | kẹp 60 giây; hết hạn đúng lúc và đánh thức long-poll; phát hiện lười cũng đánh thức; bộ hẹn giờ nổ sớm thì hẹn lại; bấm Nhận/Từ chối trước hạn; trần 8 lời hỏi; Máy của tôi giữ 24 giờ và tự nhận |
| Rust `tests/sec4_may_la.rs` (5 test) | cùng các câu trên phía oracle |
| Harness `transfer-windows-may-la` | **17/17**: hai tiến trình, Rust ↔ Snappy — đủ bốn tình huống nghiệm thu; "Không phản hồi" tới sau 59 giây |

Tổng sau rebase: C# 1119/1119, Rust 29/29 bộ, clippy sạch, CI Jenkins #208 đạt
(vàng chỉ vì chưa ký số).

Đột biến đã chạy: bỏ phép dẫn xuất `deviceId` → đỏ; bỏ đánh thức long-poll khi hết
hạn → đỏ hai test.

## Trên app AOT thật (14/11/2026)

Rust đóng vai "Lan's Mac" gửi tới Snappy đang chạy ở cổng 8443:

- không bấm: hộp hiện tên Lan, tự đóng sau 60 giây, người gửi `EXPIRED` ở giây 60;
- bấm Nhận: tệp 300 000 byte tới `Downloads\Snappy`, SHA-256 khớp;
- bấm Từ chối: `REJECTED`, không có tệp.

Nút được bấm bằng `BM_CLICK` vào HWND của nút, không phải chuột thật.

## Dấu "tải từ mạng" (#277, 15/11/2026)

- C# `MarkOfTheWebTests` (5 test): một tệp, cả thư mục, đường resumable mang
  `ZoneId=3` kèm đúng một dòng `Marked`; Máy của tôi không luồng và có dòng Trace
  riêng. Đột biến: tắt gắn dấu → 3 đỏ; gắn cho mọi máy → 1 đỏ.
- `ci/check-zone-mark.ps1` trên bản AOT, ngoài gói MSIX: bấm Nhận thật → luồng
  `[ZoneTransfer] | ZoneId=3`; mở qua Explorer → SmartScreen *"Windows protected
  your PC"*; Máy của tôi không luồng; không DLL nặng mới; private 8,6 MB. Trên bản
  `main` kịch bản đỏ đúng ở bước có dấu.

## Chưa đo

- Máy đã ghép gửi tới **trên app thật** (cần ghép qua hộp SAS) — đã đo qua harness.
- iPhone/Android gửi như máy lạ — app chưa làm.
