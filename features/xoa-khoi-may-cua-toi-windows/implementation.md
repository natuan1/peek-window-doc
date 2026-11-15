# Xoá khỏi Máy của tôi — implementation

## Windows (`apps/windows`)

| Tệp | Vai |
|---|---|
| `Snappy.Protocol/Trust/PendingRemovals.cs` | khoảng Hoàn tác 5 giây (`TimeProvider`, có test với đồng hồ giả) |
| `Snappy.Protocol/Trust/UnpairNotices.cs` | lời huỷ ghép còn nợ, bền trên đĩa — cùng định dạng tệp với Trust Store, `unpair-notices.json` |
| `Snappy.Protocol/Client/UnpairCourier.cs` | `Remove` gỡ cục bộ + ghi nợ; `DeliverAsync` gửi bù, bỏ nợ khi đã ghép lại |
| `Snappy.Protocol/Client/UltpClient.cs` | `SendUnpairAsync` (bootstrap + khoá đã ghim, `DELETE /v1/pairings/{id}`); `AuthenticateAsync` tự hạ khi kênh ghim nhận `403` |
| `Snappy.Core/AppHost.Unpair.cs` | nút Xoá → Hoàn tác → gỡ; gửi bù mỗi khi bảng lân cận đổi; gỡ hết lúc thoát |
| `Snappy.Core/SettingsLayout.cs`, `SettingsWindow.Input.cs`, `SettingsProtocol.cs` | nút "Xoá" không hỏi lại, dòng "Đã xoá · Hoàn tác", sự kiện `undo-forget` |
| `Snappy.Protocol/Server/UltpRouter.cs` | phía nhận `DELETE /v1/pairings/{deviceId}` — đã có từ trước, không đổi |

## rust-host (oracle)

- Route `DELETE /v1/pairings/{device_id}` (chỉ gỡ chính người gọi, thu hồi session).
- `HostState::xoa_khoi_may_cua_toi`, `gui_bu_huy_ghep`, `ha_xuong_may_xung_quanh`, lời nợ bền cạnh Trust Store.
- `client::bao_huy_ghep`; `ClientError::NoLongerTrusted` từ `authenticate` (kênh ghim) và
  `authenticate_by_key` (sau phép chứng minh ghim đúng SPKI đã trả `403`).
- `examples/interop-ghep`: lệnh `xoa`, `hoan-tac`, `gap`, cờ `--ben`.

## Giao thức

SPEC §10.3 mới, SECURITY §4.7 mới, openapi mô tả lại `DELETE /pairings/{deviceId}` và thêm
`404 DEVICE_NOT_TRUSTED` vào bảng ánh xạ (Windows đã trả mã ấy từ trước mà bảng chưa ghi).
