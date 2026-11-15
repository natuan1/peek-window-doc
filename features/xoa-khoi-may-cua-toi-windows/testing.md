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
15/15 vẫn xanh vì không có kẻ đứng giữa. Test proxy đỏ trước khi sửa, xanh sau (bài học 250 của
`peekvn`).

## Nghiệm thu bằng cử chỉ thật

`apps/windows/ci/check-xoa-may.ps1` trên bản publish AOT: menu khay thật, cửa sổ Cài đặt thật
(nút Xoá / Hoàn tác), máy B là rust-host quảng bá mDNS thật — nên "thấy lại B" là discovery thật.
Trust Store thật của máy dev được sao lưu và trả lại. Chạy ngoài gói MSIX, khi máy rảnh ≥ 60 giây.
