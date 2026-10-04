# Luồng bản quyền

## Khởi động

1. `MachineId.ForThisMachine()` lấy ID máy. Nhật ký ghi **nguồn** (`Motherboard` / `WindowsInstall`), không ghi con số.
2. `ProLicense.Load()` thực hiện:
   - đọc `Snappy/ProLicense` trong Credential Manager;
   - kiểm chữ ký Ed25519 bằng khoá biên dịch sẵn;
   - kiểm `mid` khớp máy này;
   - đạt cả hai thì Pro. **Không gọi mạng.**
3. Shelf nhận gói (`ApplyPlan`), bảng trạng thái nhận nhãn.
4. Đang là Pro thì hẹn lượt re-validate đầu sau **2 phút**. Để sớm hơn thì máy vừa mở nắp chưa có mạng, lượt ấy hỏng, và lượt thật bị đẩy ra một giờ sau.

## Kích hoạt

```
menu khay "Kích hoạt Snappy Pro…"
  → tiến trình nền khởi động  Snappy.exe --activate-pro <machineId>   (con)
      con: hộp thoại, người dùng dán Key, bấm Kích hoạt
      con: LicenseKey.TryParse — sai dạng thì báo ngay, không mạng
      con: POST /v1/license/activate  (HttpClient chỉ sống trong con)
      con: kiểm token (chữ ký + mid) TRƯỚC khi trao
      con → stdout:  "attempt invalid-key (404 invalid-key)" | … | "activated <token>"
  ← nền: ghi nhật ký từng dòng attempt (không bao giờ ghi token)
  ← nền: activated → ProLicense.Install: kiểm lại → CredWriteW → Pro → bong bóng
```

Cha là chỗ **duy nhất** ghi vào Credential Manager: con trao token, cha kiểm lại rồi mới cất. Hộp thoại đang mở mà bấm menu lần nữa thì không mở hộp thứ hai (có dòng nhật ký). Snappy thoát thì hộp thoại đóng theo.

## Re-validate

```
WM_TIMER  → Snappy.exe --check-license   (con đọc token từ kho, POST /v1/license/validate)
con → stdout: "check active 200 active" | "check revoked slot-released" | "check inconclusive …" | "check none"
nền:
  active        → hẹn lần sau sau 24 h
  revoked       → Revoke: xoá token, về Free, bong bóng theo lý do
  inconclusive  → giữ Pro, thử lại sau 1 h
  none          → kho trống mà RAM có Pro (lượt lưu hỏng): giữ Pro tới khi thoát
```

## Nhật ký (tag `License`, tiếng Anh)

- `Machine ID derived from Motherboard; license server …; trusting the DEVELOPMENT signing key`
- `No Pro token stored - running Free`, hoặc `Pro restored offline from Credential Manager: key SNPY-****-****-****-0001, slot 1`
- `Activation attempt failed: no-free-slot (409 no-free-slot)`
- `Pro activated: key SNPY-****-****-****-0001, slot 1, saved to Credential Manager`
- `License server confirmed Pro (200 active)` · `License check inconclusive (…) - keeping Pro, retrying in an hour` · `Pro revoked by the license server (slot-released) - running Free`

Key luôn được che, chỉ giữ nhóm cuối. Token không bao giờ ra nhật ký.
