# ADR: Bản quyền Pro — verify Ed25519 tại máy bằng mã tự viết, khoá biên dịch sẵn, mạng và hộp thoại trong tiến trình con

Date: 2026-10-04
Status: Accepted

## Context

Ticket 14 ([natuan1/peekvn#145](https://github.com/natuan1/peekvn/issues/145)) đòi các điều sau:

- Pro không cần tài khoản.
- Key `SNPY-`, tối đa 5 Slot.
- Token ký Ed25519, lưu Credential Manager.
- Offline vĩnh viễn, re-validate thầm lặng khi có mạng.

Máy chủ (Cloudflare Worker) là hạ tầng ngoài spec #131 và chưa có.

Ba ràng buộc đã có từ trước:

- **RAM nền.** Một `HttpClient` tốn ~3 MB (bài học 172 của `peekvn`). Luồng UI đã tắt IME để menu khay không kéo TSF vào ([ADR-0016](0016-kpi-ram-nen-do-bang-private-working-set.md)), nên một ô gõ Key ở tiến trình nền phá đúng hàng rào ấy.
- **Kích thước.** KPI bộ cài 15 MB, hôm nay đang là 11,06 MB.
- **Nền tảng.** .NET 10 không có Ed25519 trong BCL. CNG của Windows chỉ có X25519 cho ECDH, không có EdDSA.

## Decision

1. **Verify tại máy, không hỏi máy chủ để có Pro.** Lúc khởi động, Pro = token trong Credential Manager qua được chữ ký Ed25519 bằng khoá công khai **biên dịch vào exe**, và có `mid` trùng Machine ID của máy. Mạng chỉ dùng để re-validate.
2. **Ed25519 tự viết, chỉ verify** (RFC 8032 §5.1.7, `BigInteger`).
   - Đối chiếu bằng vector RFC và bằng chữ ký của **BouncyCastle**. BouncyCastle chỉ có mặt ở test và máy chủ giả, không vào exe.
   - Không thời gian hằng định: mọi đầu vào đều công khai.
3. **Khoá không đổi được lúc chạy, địa chỉ máy chủ thì được.**
   - Một khoá đọc từ biến môi trường là một Pro miễn phí.
   - Địa chỉ (`SNAPPY_LICENSE_SERVER`) không quyết định được gì, vì token nào cũng phải qua khoá.
4. **Hộp thoại nhập Key và mọi lời gọi mạng chạy trong tiến trình Snappy con** (`--activate-pro`, `--check-license`), cùng mẫu với [ADR-0017](0017-tinh-nang-nang-phan-shell-chay-trong-tien-trinh-con.md).
   - Con trao token qua stdout.
   - Cha kiểm lại rồi mới ghi Credential Manager, nên cha là người ghi duy nhất.
   - Hộp thoại dùng điều khiển Win32 chuẩn. Visual style bật bằng activation context (manifest 124 của `shell32.dll`) chỉ trong con.
5. **Chỉ một câu trả lời tường minh mới lấy lại Pro**: `410 slot-released` hay `410 key-revoked`. Mất mạng, 5xx, hay `410` với mã lạ thì giữ Pro.
6. **Hôm nay tin khoá DEV.** Nửa bí mật của khoá này công khai trong `tools/Snappy.LicenseServer`. `ci/pack.ps1 -RequireSigning` từ chối phát hành chừng nào dấu `DEV-ONLY-LICENSE-KEY` còn trong `LicenseAuthority.cs`.

## Consequences

- ✅ Tiến trình nền không có `HttpClient`, không có ô gõ chữ, không thêm DLL nào. 📐 Private 5,28 → 6,69 MB sau một lượt kích hoạt trọn. Exe +186 KB, bộ cài +0,10 MB.
- ✅ Pro sống qua mất mạng, máy chủ chết, proxy chặn — đúng "offline vĩnh viễn".
- ⚠️ Chủ Key gỡ Slot ở portal thì máy cũ **chỉ** mất Pro khi nó lên mạng. Máy không bao giờ lên mạng giữ Pro mãi. Đây là cái giá của "offline vĩnh viễn", và là chủ ý.
- ⚠️ Lấy lại Pro không "thầm lặng": người dùng **phải** được báo vì sao Pro biến mất và đường quay lại (AGENTS.md §7.1). Ticket chỉ nói "re-validate thầm lặng". Lượt kiểm thì thầm lặng, còn kết cục xấu thì không.
- ⚠️ Đổi khoá ký ở máy chủ là vô hiệu mọi token đã phát. Muốn xoay khoá thì phải phát hành một bản app tin **cả hai** khoá trước.
- ⚠️ Machine ID là một giao kèo: đổi hằng số băm là mọi máy tốn thêm một Slot.
- ⚠️ Máy chủ **MUST** cho kích hoạt idempotent theo (Key, Machine ID) — xem [hợp đồng](../features/ban-quyen-windows/api.md). Câu báo "kích hoạt lại không tốn thêm máy" dựa vào điều này.
- 🔒 Treo tới khi có máy chủ thật: khoá thật, địa chỉ thật, portal.

## Related

- [Bản quyền Pro trên Windows](../features/ban-quyen-windows/overview.md) · [Hợp đồng API](../features/ban-quyen-windows/api.md)
- [ADR-0016](0016-kpi-ram-nen-do-bang-private-working-set.md) · [ADR-0017](0017-tinh-nang-nang-phan-shell-chay-trong-tien-trinh-con.md)
