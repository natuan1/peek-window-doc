# Thêm vào Máy của tôi — kiểm thử

## Tự động

| Lớp | Đâu | Số đo |
|---|---|---|
| C# | `AddToMyDevicesTests` (5), `TrayMenuAddToMyDevicesTests`, `AddToMyDevicesTextTests`, `PairingPromptTextTests` | trọn bộ 1153/1153 |
| Rust | `tests/ghep_mot_lan_dong_y.rs` (5) | trọn bộ 254/254, clippy sạch |
| Interop | `interoperability/tests/ghep-windows.sh` — peek ↔ `examples/interop-ghep` | 20/20; toàn bộ harness 89 pass, 5 bỏ qua (Swift) |

Đột biến đã thử: gỡ tự ẩn, gỡ cập nhật tên (C#: 2 đỏ); gỡ chờ Đồng ý, gỡ tự ẩn,
gỡ kiểm `deviceId` (Rust: 4 đỏ).

## Nghiệm thu bằng cử chỉ thật

`apps/windows/ci/check-them-may.ps1` trên bản publish AOT: menu khay thật, hộp
thoại thật, máy B là rust-host quảng bá mDNS thật. Chạy ngoài gói MSIX
(`%TEMP%\snappy-accept\outside.ps1`) và chỉ khi máy rảnh ≥ 60 giây.

📐 **15/11/2026, bản AOT `8149d08`, ĐẠT 7/7:**

- A. Menu khay "Thêm Mac của Hưng vào Máy của tôi" → B hỏi (`from=NATUAN1`) → Đồng ý → 0,8 s sau
  `Added Mac của Hưng (dev_belk…) to my devices`; B in `HIDDEN`; menu có "Gửi tệp tới Mac của Hưng…".
- B. B gửi 256 KB → `state=ACCEPTED` ngay trong `201`; `Auto-accepted 1 items from Mac của Hưng - paired device, never asks`.
- C. B đổi tên thành "Mác mới của Hưng" → vẫn tự nhận; `renamed itself from … - same key, still one of my devices`; menu đổi tên theo.
- D. Máy đang ẩn không có mục "Thêm" nào; máy đang hiện cạnh nó thì có (đối chứng).
- E. C Từ chối → hộp báo "Chưa được đồng ý"; `ended NotAgreed (status 403)`; C không ghim gì.
- F. D xin thêm lúc peek ẩn → `403 DEVICE_HIDDEN`, không hộp nào; bấm Hiện → hộp hỏi đúng chữ spec, không mã số → Đồng ý → D ghim peek; `Hidden from nearby devices - just added iPad của Hưng to my devices`.
- G. Thoát sạch.

CI Jenkins #211 đạt (UNSTABLE chỉ do bước ký số), commit khớp `8149d08`.

Chưa đo bằng cử chỉ thật: chiều A → B qua hộp chọn tệp (đã đo ở harness `[qua-lai]` cùng `OfferFiles`).
