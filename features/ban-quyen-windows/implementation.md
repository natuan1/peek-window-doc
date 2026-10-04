# Cài đặt bản quyền

Mã: `peekvn/apps/windows/src/Snappy.Core/Licensing/` và hai lớp Win32 ở `Snappy.Interop`.

| Tệp | Vai |
|---|---|
| `Ed25519.cs` | verify Ed25519 theo RFC 8032 §5.1.7 (xem dưới) |
| `LicenseKey.cs` | đọc Key `SNPY-` (Crockford), dạng chuẩn, bản che `SNPY-****-****-****-XXXX` |
| `MachineId.cs` | Machine ID; đọc bảng SMBIOS là hàm thuần, kiểm được bằng byte |
| `LicenseToken.cs` | tách token, kiểm chữ ký **trước** khi đọc JSON, kiểm `v` và `mid` |
| `LicenseAuthority.cs` | khoá công khai biên dịch sẵn, địa chỉ máy chủ, dấu `DEV-ONLY-LICENSE-KEY` |
| `ProLicense.cs` | trạng thái bản quyền, `Entitlements`, `RequirePro`, `CredentialVault` |
| `LicenseClient.cs` | HTTP tới máy chủ. **Chỉ chạy trong tiến trình con** |
| `LicenseProcess.cs` | `--activate-pro` / `--check-license`, giao kèo stdout `LicenseLine` |
| `ActivationDialog.cs` | hộp thoại Win32 (`EDIT` + `BUTTON` chuẩn), trong con |
| `LicenseNotices.cs` | mọi câu tiếng Việt cho người dùng |
| `AppHost.License.cs` | nửa phía tiến trình nền: Load, menu, timer re-validate, đọc dòng của con |
| `Snappy.Interop/CredentialStore.cs` | `CredReadW` / `CredWriteW` / `CredDeleteW`, `CRED_TYPE_GENERIC`, `CRED_PERSIST_LOCAL_MACHINE` |
| `Snappy.Interop/HardwareIds.cs` | `GetSystemFirmwareTable('RSMB')`, CPUID |

## Machine ID

```
SHA-256( "snappy/machine-id/v1/smbios" ‖ 0x00 ‖ UUID(16 byte, SMBIOS type 1 offset 0x08) ‖ 0x00 ‖ "vendor|signature|brand" )
```

Kết quả là 64 ký tự hex thường. Một phép băm một chiều, nên không ai lần ngược ra số hiệu phần cứng. CPU không phải nguồn định danh: hai máy cùng đời chip có cùng chữ ký. Nó chỉ làm ID đổi khi bo mạch bị cấy sang máy khác.

Bo mạch OEM chưa ghi UUID trả toàn `00`, toàn `FF`, hoặc mẫu `03000200-0400-0500-0006-000700080009`. Khi ấy dùng `HKLM\SOFTWARE\Microsoft\Cryptography\MachineGuid`, với một miền băm khác. ID này đổi khi cài lại Windows, nên người dùng ấy tốn thêm một Slot.

⚠️ **Đổi bất kỳ hằng số nào ở trên là đổi ID của mọi máy đã kích hoạt.**

📐 Máy dev, 2026-10-04: nguồn `Motherboard`.

## Ed25519 tự viết, chỉ verify

.NET 10 không có Ed25519 trong BCL. CNG của Windows chỉ có X25519 cho ECDH. Mang cả một thư viện crypto vào exe có KPI bộ cài 15 MB để dùng đúng một phép thì đắt. Nên tự viết:

- `BigInteger`, toạ độ mở rộng, nhân đôi-cộng.
- Kiểm kiểu không-nhân-cofactor như ref10/libsodium: tính `[S]B − [k]A`, so **mã hoá** với `R`.
- Từ chối `S ≥ L` và khoá công khai không giải được thành điểm.

Không cần thời gian hằng định vì mọi đầu vào đều công khai. Không có hàm ký.

## Vì sao mạng và hộp thoại ở tiến trình con

- Một `HttpClient` tốn ~3 MB RAM ([bài học 172 của peekvn](https://github.com/natuan1/peekvn/blob/main/docs/bai-hoc.md)).
- Một ô gõ chữ kéo TSF vào luồng UI vốn đã tắt IME ([ADR-0016](../../adr/0016-kpi-ram-nen-do-bang-private-working-set.md)).
- Cùng mẫu với hộp chọn tệp và khay Shelf ([ADR-0017](../../adr/0017-tinh-nang-nang-phan-shell-chay-trong-tien-trinh-con.md)).
- Hộp thoại bật visual style bằng activation context lấy từ manifest tài nguyên 124 của `shell32.dll`, chỉ trong con. `app.manifest` chung không đổi.

Toàn bộ lý do ở [ADR-0019](../../adr/0019-ban-quyen-verify-tai-may-mang-trong-tien-trinh-con.md).

## Số đo (📐 2026-10-04, bản AOT, `ci/check-license.ps1`)

| | |
|---|---|
| Exe | 10 159 → **10 345 KB** (+186 KB) |
| Bộ cài | 11,06 → **11,16 MB** (KPI < 15 MB) |
| Tiến trình nền lúc nghỉ, Free | private 5,09 MB, tổng 21,5 MB |
| Sau một lần mở menu khay | 5,28 / 23,16 MB |
| Sau kích hoạt trọn (con chạy, token lưu, bong bóng) | **6,69 / 24,77 MB**, không DLL mới |
| Lúc nghỉ, **bản Pro** (verify token lúc khởi động) | **6,48 / 23,07 MB** — +1,33 MB private so với Free, trả ở **mỗi** lần mở: heap managed của verify `BigInteger`, không DLL. Chỗ đầu tiên để cắt nếu biên RAM hẹp lại |

## Các bẫy đã gặp

`OLEAUT32` hiện ra "sau kích hoạt" là do **cú bấm chuột vào mục menu khay**, không do bản quyền:

- Bấm chuột vào mục nào của menu khay (cả "Mở bảng trạng thái") cũng làm tiến trình nền nạp nó, kể cả khi toạ độ lấy thuần Win32, không UIA. Nên người dùng thật cũng trả giá ấy. Đó là chuyện của menu khay, chưa rõ đường nạp.
- Chọn mục bằng bàn phím thì không nạp.

Kịch bản kích hoạt bằng `↑↑↑ Enter` để số đo so được giữa hai bản build. Xem bài học 213 của `peekvn` (cả phần đính chính).
