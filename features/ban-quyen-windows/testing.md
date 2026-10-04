# Kiểm thử bản quyền

## Hai implementation độc lập

`tools/Snappy.LicenseServer` là máy chủ giả, nói đúng [hợp đồng API](api.md).

- Nó ký token bằng **BouncyCastle** và không tham chiếu project nào của `src/`. Test đặt chữ ký của nó trước phép verify tự viết của app.
- Một verify được kiểm bằng chữ ký do chính nó sinh ra chỉ chứng minh mã nhất quán với chính nó.
- Hạt giống khoá DEV công khai trong repo. Test `Khoa_dev_trong_app_trung_khoa_cua_may_chu_gia` canh để hai nửa của khoá không lệch nhau.

## xUnit (`tests/Snappy.Tests`) — 86 test mới, cả bộ 851

| Lớp | Canh |
|---|---|
| `Ed25519Tests` | 3 vector RFC 8032 §7.1, 24 vòng khoá ngẫu nhiên của BouncyCastle (mỗi vòng thêm một bit lệch và một khoá khác), `S + L` bị từ chối, sai độ dài, khoá ngoài trường |
| `LicenseKeyTests` | các cách người ta chép Key, `O→0` `I/L→1`, `U` bị từ chối, gạch sai chỗ |
| `MachineIdTests` | đọc UUID qua một cấu trúc có vùng chuỗi, bảng cụt, ba UUID OEM rác, ID không chứa UUID trần, máy thật ổn định qua hai lần đọc |
| `LicenseTokenTests` | token sửa một ký tự, khoá khác, máy khác, payload ký đúng nhưng không phải JSON hay sai phiên bản |
| `LicenseClientTests` | HTTP thật trên loopback: trần 5 Slot, kích hoạt lại không tốn Slot, gỡ Slot không dồn số, 503, cổng đóng, token máy khác, proxy `404` HTML, cổng Wi-Fi `410` HTML |
| `ProLicenseTests` | khởi động lại khi máy chủ **đã tắt hẳn** vẫn Pro, kho không đọc/ghi được, gỡ Slot thì xoá token, Shelf theo gói, Credential Manager **thật** |
| `LicenseProcessTests` · `TrayMenuTests` | giao kèo stdout, câu báo không lộ mã HTTP, mục menu chỉ có ở bản Free |

Đột biến đã thử: gỡ phép kiểm `S ≥ L` thì `S_cong_them_L_thi_bi_tu_choi…` đỏ.

## Nghiệm thu bằng cử chỉ người dùng — `ci/check-license.ps1`

Chạy ngoài gói MSIX của app Claude (bài học 201 của `peekvn`), mất khoảng 8 phút. Máy chủ giả chạy trên `localhost:8790`, Snappy là bản AOT thật trỏ vào nó.

| Bước | Cử chỉ | Bằng chứng (📐 2026-10-04) |
|---|---|---|
| A | chuột phải icon khay, bàn phím `↑↑↑ Enter` vào "Kích hoạt Snappy Pro…", **dán** Key (Ctrl+V) | hộp thoại trong tiến trình con. Ba câu đỏ đúng: sai dạng / không nhận ra / đủ 5 máy. Rồi `Pro activated: key SNPY-****-****-****-0001, slot 1, saved to Credential Manager`. `CredReadW` thấy mục. Không tệp nào dưới `%LOCALAPPDATA%\Snappy` chứa `eyJ2Ijox`. Menu thôi mời kích hoạt |
| B | khởi động lại, có mạng | `Pro restored offline…`, rồi `License server confirmed Pro (200 active)` |
| C | khởi động lại, máy chủ `localhost:1` (đóng) | `Pro restored offline…`, rồi `License check inconclusive (… actively refused …) - keeping Pro` |
| D | portal giả gỡ Slot 1 → khởi động lại | `Pro revoked by the license server (slot-released) - running Free`. Bong bóng hiện. Kho trống. Menu mời kích hoạt lại |

Hàng rào phát hành: `pack.ps1 -RequireSigning` đỏ với *"LicenseAuthority.cs van tin khoa ky ban quyen DEV"* (đã chạy, 2026-10-04).

## Treo

- **Kích hoạt với máy chủ thật.** Treo vì thiếu tài khoản và hạ tầng: Cloudflare Worker và khoá ký thật chưa có. Không chặn ticket nào sau. Nó chặn **phát hành**, và hàng rào ở `pack.ps1` giữ chỗ ấy.
- **Re-validate ở máy nằm sau proxy công ty thật.** Treo vì thiếu môi trường. Đã có test với proxy `404`/`410` HTML giả.
