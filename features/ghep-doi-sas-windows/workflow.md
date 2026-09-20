# Ghép đôi SAS — luồng đi qua những đâu

## Toàn cảnh: hai bước trên dây, ba quyết định của con người

```
iPhone (initiator)                         Snappy (responder)
──────────────────                         ──────────────────
mở TLS 1.3, bootstrap  ───────────────►    trình certificate
   (chấp nhận cert chưa tin, nhưng
    SChannel/URLSession VẪN verify
    chữ ký handshake — S17)

quan sát SPKI của kênh
sinh khoá ECDH ephemeral

POST /v1/pairings      ───────────────►    sinh khoá ephemeral của mình
  deviceId, publicKey,                     dựng transcript (SPKI của CHÍNH NÓ)
  ephemeralPublicKey                       tính SAS
                       ◄───────────────    201: pairingId, publicKey,
                                                ephemeralPublicKey, expiresAt

dựng transcript                            ┌──────────────────────────┐
  (SPKI QUAN SÁT ĐƯỢC)                      │ hiện hộp thoại sáu chữ số │
tính SAS                                    └──────────────────────────┘
┌──────────────────────┐
│ hiện sáu chữ số      │
└──────────────────────┘
                    👤 NGƯỜI DÙNG SO HAI MÀN HÌNH 👤

người dùng bấm đồng ý                       người dùng bấm đồng ý
POST …/confirm         ───────────────►    (request bị GIỮ MỞ ở đây,
  { confirmed: true }                       chờ nút bấm phía Windows)
                       ◄───────────────    200: deviceId, pairedAt
ghim SPKI quan sát được                     ghim SPKI của chính nó
```

Chỗ duy nhất hai transcript có thể lệch nhau là **SPKI của kênh**. Mọi giá trị
khác đều đi qua dây và được chuyển tiếp nguyên văn kể cả khi có kẻ đứng giữa.

## Ba quyết định của con người, không phải hai

Dễ đếm nhầm thành hai (một người bấm ở mỗi máy). Thực ra là ba, và cái thứ ba là
cái quan trọng nhất:

1. Người dùng iPhone bấm đồng ý.
2. Người dùng Windows bấm đồng ý.
3. **Trước cả hai: một người đã so hai con số.** Không có bước này thì hai bước
   trên chỉ là hai cú bấm.

Giao diện phải làm bước 3 thành thứ khó bỏ qua. Đó là lý do hộp thoại mang cả
sáu chữ số lẫn câu *"Chỉ bấm Có khi hai mã GIỐNG HỆT nhau"*, và lý do nút mặc
định là "Không".

## Sau ghép đôi: mọi kết nối đi qua ba cửa

```
1. TLS 1.3 + ghim SPKI     ← khác pin ⇒ DEVICE_IDENTITY_CHANGED, hết đường (S14)
2. POST /v1/auth/challenge ← chưa ghép đôi ⇒ DEVICE_NOT_TRUSTED
3. POST /v1/auth/verify    ← hai chiều: client ký, server verify bằng khoá ĐÃ GHIM,
                             rồi server ký một chuỗi byte KHÁC để client verify ngược
```

Cửa thứ ba trả về một session token ràng buộc với `(deviceId, pin của kênh,
clientNonce)`. Trình nó trên một kênh có SPKI khác thì bị từ chối.

## Huỷ ghép đôi

`DELETE /v1/pairings/{deviceId}` — cần session token, và một peer chỉ gỡ được
**chính nó**. Gỡ xong thì mọi session token của thiết bị ấy bị thu hồi ngay;
để chúng sống tiếp nghĩa là peer vừa bị gỡ vẫn gọi được API trong tối đa 30 phút.

⚠️ **Người dùng Windows chưa có chỗ bấm để quên một thiết bị.** Đường gỡ hiện chỉ
đi từ phía peer. Với một chiếc điện thoại đã mất, đó là một lỗ hổng thật trong
trải nghiệm — ghi ở đây vì một mục im lặng sẽ được đọc thành mục đó không tồn
tại. Việc của ticket sau.

## Ba đường hỏng, và cái người dùng thấy

| Hoàn cảnh | Trên dây | Người dùng Windows thấy |
|---|---|---|
| Hai mã khác nhau, ai đó bấm Không | `403 PAIRING_REJECTED` | hộp thoại: *"một trong hai bên đã từ chối… nếu hai mã KHÁC nhau, đừng thử lại ngay"* |
| Bên kia bỏ đi giữa chừng | kênh đứt ⇒ phiên bị huỷ | hộp thoại: *"phiên ghép đôi đã hết hạn"* |
| Không ai bấm trong 120 giây | `403` | như trên |

Hai kết cục cuối trông giống nhau trên dây nhưng **khác nhau với người dùng**, và
đó là lý do chúng có hai câu chữ khác nhau: một cái nói "thiết bị kia đã ngắt kết
nối", cái kia nói "không xác nhận kịp".

Kết cục **thành công** cố ý *không* có hộp thoại: thiết bị xuất hiện trong danh
sách đã ghép đôi là bằng chứng người dùng nhìn thấy được, và một hộp thoại nữa ở
đó là một cú bấm thừa.
