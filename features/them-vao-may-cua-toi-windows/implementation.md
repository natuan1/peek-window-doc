# Thêm vào Máy của tôi — implementation

## Windows (`apps/windows`)

| Tệp | Vai |
|---|---|
| `Snappy.Protocol/Client/UltpClient.cs` | `AddToMyDevicesAsync` — vai A: mở phiên, `confirm` ngay, chờ `AnswerPatience` (120 + 15 s, dài hơn hạn của B — bài học 206), đọc kết cục thành `AddOutcome` |
| `Snappy.Protocol/Client/PairingInProgress.cs` | `SendAsync` trả câu trả lời thô; ghim bảng năng lực B khai trong `201` |
| `Snappy.Protocol/Server/UltpRouter.cs` | vai B: Đồng ý → `VisibilityWindow.Hide("just added … to my devices")`; Từ chối → "Chưa được đồng ý."; trusted peer gửi `displayName` trong verify → nhớ tên mới |
| `Snappy.Protocol/Pairing/PairingCoordinator.cs` | sự kiện `RequestPresented` (trước là `SasPresented`) |
| `Snappy.Core/AppHost.cs` | hộp hỏi `AskUserToAddDevice`; lệnh khay `AddToMyDevices` (một lần một máy, thử từng địa chỉ, kết cục về luồng UI) |
| `Snappy.Core/TrayMenu.cs` | nhóm "Thêm … vào Máy của tôi" (id 2000+), lọc máy ẩn và máy đã là Máy của tôi |
| `Snappy.Core/PromptStyle.cs` | `PairingPromptText` (không mã số, không deviceId), `AddToMyDevicesText` |

## rust-host (oracle)

- `HostState::hoi_truoc_khi_ghep` / `dong_y_ghep` — chế độ hỏi như app; mặc định oracle vẫn tự đồng ý.
- Tự ẩn sau ghim khi đang trong Visibility Window; kiểm `deviceId` dẫn xuất từ `publicKey`.
- `client::authenticate_by_key`, `begin_pairing_with`, `HostState::client_identity` / `doi_ten`.
- ⚠️ Giới hạn: `h2serve` không huỷ handler khi client đứt kênh, nên oracle không huỷ phiên đang hỏi khi bên xin bỏ đi.

## Giao thức

SPEC §10.0 mới, SECURITY §4.6 mới, S3/S6 sửa, §7.3 thành rủi ro chấp nhận; openapi
mô tả lại `/pairings` và `displayName` của verify.
