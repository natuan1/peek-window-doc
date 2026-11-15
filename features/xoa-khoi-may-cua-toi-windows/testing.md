# Xoá khỏi Máy của tôi — kiểm thử

## Tự động

| Lớp | Đâu | Số đo |
|---|---|---|
| C# | `UnpairTests` (10), `SettingsProtocolTests`, `SettingsLayoutTests` (Hoàn tác) | trọn bộ 1166/1166 |
| Rust | `tests/huy_ghep.rs` (8, gồm một proxy đóng vai kẻ đứng giữa) | trọn bộ 262/262, clippy sạch |
| Interop | `interoperability/tests/huy-ghep-windows.sh` — peek ↔ `examples/interop-ghep` | 15/15, 0 tiến trình sót |

Đột biến đã thử: bỏ phép chứng minh `403`, bỏ kiểm "chỉ gỡ chính mình", bỏ thu hồi token
(Rust: mỗi cái 1 đỏ); gửi lời huỷ ghép trên kênh ghim thay vì bootstrap (C#: 1 đỏ).

📐 Review spec độc lập tìm ra lỗ: phép chứng minh `403` đọc SPKI từ một kênh **mới**, nên kẻ
đứng giữa trả `403` của một máy khác rồi nhường đường cho máy thật là cắt được cặp máy. Harness
15/15 vẫn xanh vì không có kẻ đứng giữa. Test proxy đỏ trước khi sửa, xanh sau (bài học 251 của
`peekvn`).

## Nghiệm thu bằng cử chỉ thật

`apps/windows/ci/check-xoa-may.ps1` trên bản publish AOT: menu khay thật, cửa sổ Cài đặt thật
(nút Xoá / Hoàn tác), máy B là rust-host quảng bá mDNS thật — nên "thấy lại B" là discovery thật.
Trust Store thật của máy dev được sao lưu và trả lại. Chạy ngoài gói MSIX, khi máy rảnh ≥ 60 giây.

📐 **15/11/2026, bản AOT của `66e19e9`, ĐẠT 5/5:**

- A. Menu khay "Thêm Mac của Hưng vào Máy của tôi" → B Đồng ý → `Added Mac của Hưng (dev_Y7cE…) to my devices`.
- B. Cài đặt → Thiết bị → Xoá: không hộp hỏi lại; dòng thành "Đã xoá · Hoàn tác trong vài giây"
  (ảnh chụp); bấm Hoàn tác sau 0,6 s → `User undid removing … - nothing changed on either machine`;
  quá 7 s không gỡ gì, B gửi 256 KB vẫn `state=ACCEPTED` ngay trong `201`.
- C. Xoá lần nữa → 5 s sau `Removed … - owe it an unpair notice`, **8 ms** sau `Unpair notice delivered`;
  B in `UNTRUSTED`; dòng biến mất khỏi trang Thiết bị; B gửi tới → hộp hỏi Nhận thật,
  `authenticated as a nearby device (not paired) - its offers will ask the user`.
- D. B2 (`--ben`) ghép rồi tắt → Xoá → `not delivered (Unreachable) - will retry next time it is seen`,
  `unpair-notices.json` còn nợ B2; bật B2 lên → mDNS thật nghe lại sau ~1 s →
  `is on the network again - sending the unpair notice it is owed` → `delivered` → B2 `UNTRUSTED`.
- E. Thoát sạch (`Clean shutdown complete`). Trust Store thật của máy dev trả lại nguyên vẹn.

Lượt đầu đỏ ở D vì **dụng cụ**: kịch bản mở menu trước khi discovery kịp nghe B2, rồi toast che icon
khay làm `Open-Menu` ném lỗi thay vì thử lại — sửa ở `f120e6a`.
