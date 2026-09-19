# Luồng — Server ULTP trên Windows

## Một kết nối đi qua những gì

```
peer mở TCP tới 8443
   │
   ├─ vòng accept: còn slot không?  (trần 64 — SECURITY.md §4.3)
   │     không → đóng ngay, ghi nhật ký.  KHÔNG xếp hàng: xếp hàng là giữ socket,
   │             mà socket đang giữ chính là thứ cần chặn.
   │
   ├─ bắt tay TLS (trần 10 giây)
   │     ├─ TLS 1.3?  không → đóng + ghi nhật ký
   │     └─ ALPN là `http/1.1` HOẶC rỗng?  khác → đóng + ghi nhật ký
   │
   └─ vòng request (keep-alive, im quá 60 giây thì đóng)
         ├─ đọc đầu request vào bộ đệm CỐ ĐỊNH 16 KB
         │     ├─ vượt 16 KB        → 431 RESOURCE_LIMIT, đóng
         │     ├─ không đúng h1     → 400 INVALID_REQUEST, đóng
         │     └─ bên kia đóng sạch → thoát im lặng (đây là kết cục BÌNH THƯỜNG
         │                             của keep-alive, không phải lỗi)
         │
         ├─ Content-Length > 1 MB  → 413 RESOURCE_LIMIT, đóng
         │                            (KHÔNG nhả thân trước: nhả 4 GB là ngồi đọc
         │                             4 GB của một kẻ chỉ muốn ta ngồi đọc)
         ├─ có thân ≤ 1 MB         → nhả qua bộ đệm 16 KB, cố định
         │
         ├─ quá 60 request/phút/địa chỉ → 429 RATE_LIMITED + Retry-After
         │
         └─ router:  GET|HEAD /v1/info → 200 JSON
                     đường dẫn lạ       → 404 INVALID_REQUEST
                     method sai         → 405 INVALID_REQUEST
```

Mọi đường hỏng trả **lỗi có cấu trúc** (SPEC §26) rồi mới đóng. Đóng câm cũng
không crash, nhưng nó để bên kia không phân biệt được *"mình gửi sai"* với
*"mạng đứt"* — và hai chuyện đó cần hai hành động khác nhau.

## Vì sao **bắt buộc** nghe dual-stack

Đây là ràng buộc đắt nhất của ticket, và nó tới từ một phép đo chứ không từ sở
thích.

Snappy quảng bá `_peek._tcp` qua responder in-box của Windows, và bản ghi A/AAAA
do **HĐH** giữ cho tên máy — app không chọn được tập địa chỉ ấy
([ADR-0008](../../adr/0008-discovery-qua-responder-in-box-windows.md), bài học
169 của `peekvn`).

📐 Đo 18/09/2026: một lượt quảng bá ra dây mang **1 địa chỉ IPv4 và 4 địa chỉ
IPv6** (gồm cả link-local).

Bind mỗi `0.0.0.0` nghĩa là mời peer gõ cửa năm địa chỉ mà chỉ nhận ở một — tức
tự tay dựng lại [#62](https://github.com/natuan1/peekvn/issues/62) và bài học
55: peer hiện trong danh sách, bấm vào thì timeout, người dùng đọc ra *"máy kia
hỏng"*.

Cách làm: **một** socket `IPv6Any` với `DualMode = true`. Hai listener riêng thì
hai vòng accept và hai đường dọn. 📐 Đã đo `200` qua cả `127.0.0.1`, `::1`, địa
chỉ IPv4 LAN và địa chỉ IPv6 toàn cục thật của máy.

## Luồng điền cột nền tảng

```
PeekBrowser nghe thấy một endpoint ĐỔI
   │  (chỉ lượt đổi, không phải mọi lượt nghe thấy — responder nhắc lại bản ghi
   │   vài lần mỗi giây, nối vào mọi lượt là gõ cửa peer vài lần mỗi giây)
   ▼
NeighborDiscovery.AskWhatItIs  →  Task.Run  (KHÔNG chạy đồng bộ trên callback
   │                                          của dnsapi — nó sẽ chặn cả lượt duyệt)
   ▼
PeerInfoProbe.AskAsync
   │  thử LẦN LƯỢT từng địa chỉ: tập địa chỉ do HĐH bên kia chọn nên nó chứa cả
   │  những địa chỉ không định tuyến tới được từ máy này
   │
   ├─ mỗi địa chỉ hỏng → một dòng nhật ký mức Error kèm địa chỉ đã thử
   │     (đây là dòng duy nhất trả lời được "đã thử những địa chỉ nào" khi người
   │      dùng hỏi sao bấm vào không ăn)
   │
   └─ trả lời được → kiểm hình dạng theo schema → PeerTable.Describe → bảng vẽ lại
```

**Kết quả ở đây KHÔNG phải một danh tính.** Certificate bên kia chưa được tin
(chưa ghép đôi), nên lượt hỏi chấp nhận certificate lạ đúng theo bootstrap của
SPEC §11.2 — và **chỉ** để đọc dữ liệu công khai. SPEC §8.1 là security
invariant: `kết quả discovery ≠ Trusted Peer`. Mọi trường trả về mang chữ
`Hint` vì lý do đó.

## Đường thoát

Thứ tự lúc app thoát có chủ ý:

1. Rút quảng bá mDNS (`DnsServiceDeRegister`).
2. **Rồi mới** đóng cổng 8443.

Ngược lại để lại một cửa sổ vài trăm mili-giây trong đó Snappy còn tự khai mình
có mặt mà đã không nhận kết nối nữa — đúng bài học 96, chỉ ngắn hơn.
