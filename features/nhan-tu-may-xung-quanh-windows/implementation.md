# Nhận từ máy xung quanh — implementation

Tất cả trong repo `peekvn`, nhánh của PR [#285](https://github.com/natuan1/peekvn/pull/285).

## Giao thức (nguồn sự thật)

| Tệp | Nội dung |
|---|---|
| `protocol/SECURITY.md` | S1, S6 đổi lời; §4.2 bỏ công tắc theo từng peer; **§4.4 Máy lạ đã xác thực** mới; ghi chú S8 ở §5.9 |
| `protocol/SPEC.md` | §10 bảng hai lớp; §10.2 `autoAccept` không còn quyết định gì; §12 nhánh máy lạ; §16.1 hạn 60 giây và hai kết cục |
| `protocol/openapi/ultp-v1.yaml` | `AuthVerifyRequest.publicKey`/`displayName`; challenge hết `403` |
| `docs/adr/0029-may-la-da-xac-thuc.md` | quyết định, rủi ro đã chấp nhận, phụ thuộc iOS/Android |

## Windows (`apps/windows`)

| Chỗ | Việc |
|---|---|
| `UltpRouter.AuthVerify` | tra Trust Store **trước** (S8); không có thì nhánh máy lạ: kiểm dẫn xuất, kiểm tên, verify bằng khoá gửi kèm; tên giữ trong RAM (`NearbyName`, trần 256) |
| `UltpRouter.CreateTransfer` | máy lạ: kẹp hạn 60 giây, trần 8 lời hỏi chờ cùng lúc |
| `UltpRouter.AnnounceOffer` | máy đã ghép → tự nhận (không đọc `AutoAccept`); máy lạ → hẹn giờ + sự kiện "cần hỏi" |
| `UltpRouter.AnswerTimedOut` | hết hạn → `EXPIRED`; nổ sớm vài ms thì hẹn lại |
| `TransferStore.ExpireIfDue` | **mọi** lần chuyển sang `EXPIRED` đều đánh thức long-poll (bài học 245 của `peekvn`) |
| `RouteAsync` | máy lạ không được tính "đang ở gần" ở tab Thiết bị |
| `ConfirmationDialog.ShowUntil` | TaskDialog có hạn (`TDF_CALLBACK_TIMER`), trả `null` khi hết giờ |
| `AppHost.AskUserToAcceptTransfer` | hiện tên + "(máy xung quanh, chưa ghép)", hết giờ không gọi `RejectLocally` |
| `UltpRouter.FinishItem` | (#277) hỏi Trust Store lúc tệp đóng lại; máy lạ → `ItemSink.TryFinish(markFromInternet: true)` + dòng `Marked …` |
| `Files.ZoneMark` | (#277) ghi `[ZoneTransfer]`/`ZoneId=3` vào luồng `Zone.Identifier`; hỏng thì trả lý do, không ném |
| `SettingsLayout` | trang Thiết bị chỉ còn nút Quên |

Đường `auto-accept` trong `SettingsWindow.Input.cs`/`SettingsProtocol.cs` còn đó
nhưng không ai gọi — dọn ở #269.

## Oracle Rust (`reference/rust-host`)

- `server.rs`: nhánh máy lạ ở `auth_verify`, kẹp 60 giây ở `create_transfer`,
  `nearby_name`. `GET` không giữ `wait`, nên không có bộ hẹn giờ.
- `client.rs`: `look_up_nearby`, `begin_auth_as_stranger`, `authenticate_as_stranger`.
- `examples/interop-nearby.rs`: vai "Lan's Mac" cho harness; `--pair` đi đường Máy của tôi.
