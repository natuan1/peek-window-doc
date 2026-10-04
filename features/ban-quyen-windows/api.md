# Hợp đồng API máy chủ cấp Key

[Ticket 14](https://github.com/natuan1/peekvn/issues/145), 2026-10-04. Phía app: `peekvn/apps/windows/src/Snappy.Core/Licensing/`. Phía máy chủ giả, nói đúng hợp đồng này: `peekvn/apps/windows/tools/Snappy.LicenseServer`.

Máy chủ thật là một **Cloudflare Worker** — hạ tầng riêng, ngoài spec app (#131), **chưa dựng**. Trang này là giao kèo nó phải giữ. Lệch trang này thì app hỏng ở máy người dùng, không ở máy chủ.

## Key

`SNPY-XXXX-XXXX-XXXX-XXXX`. Mười sáu ký tự sau tiền tố lấy từ bảng **base32 Crockford**: `0-9 A-Z`, bỏ `I L O U`.

- Máy chủ MUST sinh Key chỉ từ bảng ấy. Một Key chứa `U` bị app chặn ở "sai dạng" mà không gọi máy chủ.
- App đọc dễ dãi: chữ thường, khoảng trắng, bỏ gạch, gạch dài, `O→0`, `I/L→1`. Thứ gửi đi luôn là dạng chuẩn, chữ hoa, có gạch.

## Machine ID

64 ký tự hex thường. Máy chủ chỉ cần so bằng, không cần hiểu. Cách sinh: [implementation.md](implementation.md#machine-id).

## Token

```
base64url(payload) "." base64url(Ed25519(payload))
```

- base64url không đệm `=`.
- Payload là JSON UTF-8 `{"v":1,"key":"SNPY-…","mid":"<machine id>","slot":1..5,"iat":<unix giây>}`.
- Chữ ký phủ đúng **byte** payload, không phủ một dạng chuẩn hoá nào của JSON.
- Khoá công khai được biên dịch vào exe (`LicenseAuthority.PublicKey`).
- Payload với `v` khác 1 bị app từ chối.
- Tổng độ dài ≤ 2048 ký tự (trần blob của Credential Manager là 2560 byte).

## `POST /v1/license/activate`

```json
{"key":"SNPY-…","machineId":"…","app":"snappy-windows","version":"1.0.0"}
```

| Trả | Nghĩa | App làm gì |
|---|---|---|
| `200 {"token":"…"}` | máy được một Slot | kiểm chữ ký + `mid` rồi mới lưu |
| `404 {"code":"invalid-key"}` | Key không tồn tại | "Máy chủ không nhận ra Key này…" |
| `409 {"code":"no-free-slot"}` | Key đã đủ 5 Slot, máy này không trong số đó | "Key này đã kích hoạt trên đủ 5 máy…" |
| bất kỳ thứ gì khác | | "Máy chủ bản quyền đang trục trặc…" |

- **MUST idempotent theo (Key, Machine ID).** Máy đã giữ Slot kích hoạt lại thì nhận lại đúng Slot ấy, không tốn Slot mới. Chuyện này xảy ra khi cài lại Windows, khi Credential Manager bị xoá, hay khi app chết trước lúc lưu token. Câu báo "kích hoạt lại bằng cùng Key — không tốn thêm máy" dựa vào điều này.
- `404` hay `409` **không kèm đúng `code`** thì app coi là lỗi máy chủ, không coi là "Key sai". Lý do: một proxy công ty trả `404` HTML không được biến thành lời buộc tội người dùng gõ sai.

## `POST /v1/license/validate`

```json
{"token":"…"}
```

| Trả | App làm gì |
|---|---|
| `200 {"status":"active"}` | giữ Pro, hẹn kiểm lại sau 24 h |
| `410 {"code":"slot-released"}` | chủ Key đã gỡ máy này ở portal → về Free, xoá token, báo người dùng |
| `410 {"code":"key-revoked"}` | Key bị thu hồi (vd. hoàn tiền) → về Free, xoá token, báo "mua Key mới" |
| **mọi thứ khác** — mất mạng, 5xx, `410` mã lạ, HTML | **giữ Pro**, thử lại sau 1 h |

Chỉ hai mã `410` kia lấy lại được Pro. Một cổng Wi-Fi khách sạn trả `410` cho mọi request không được làm mất Pro của ai.

## Portal (ngoài app)

Chủ Key tự gỡ Slot bằng cặp (Key + email mua), không cần ai duyệt (CONTEXT.md). Gỡ một Slot **không dồn số** các Slot còn lại: token đã phát mang số Slot, và portal hiện "Máy 1…5".

Máy chủ giả có `POST /v1/license/release {"key":…,"slot":n}`, bỏ qua email. Route này chỉ để dựng cảnh "Slot bị gỡ" khi nghiệm thu, không thuộc hợp đồng của app.

## Địa chỉ

- Mặc định `https://license.snappy.vn` — **chưa tồn tại**.
- Biến môi trường `SNAPPY_LICENSE_SERVER` đổi được địa chỉ, vì nó không quyết định được gì: token nào cũng phải qua khoá biên dịch sẵn.
- HTTP thường chỉ nhận cho loopback (Key đi trong thân request).
