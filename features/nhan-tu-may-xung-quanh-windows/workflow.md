# Nhận từ máy xung quanh — quy trình

## Máy xung quanh gửi tới (Luồng 4 của spec)

```
Lan's Mac                                   máy Windows của Hưng
─────────                                   ───────────────────
GET /v1/info          ─────────────────────▶  deviceId + publicKey
  kiểm deviceId dẫn xuất từ publicKey
POST /v1/auth/challenge ───────────────────▶  phát nonce (cho mọi deviceId)
POST /v1/auth/verify  ─────────────────────▶  không có trong Trust Store
  + publicKey, displayName                      → kiểm deviceId dẫn xuất từ publicKey
                                                → verify chữ ký bằng khoá ấy
                      ◀─────────────────────  token + chữ ký server (không lưu gì)
POST /v1/transfers (xin 24 giờ) ───────────▶  kẹp hạn còn 60 giây, state OFFERED
                                                → hộp "nhận tệp" hiện tên Lan
GET …?wait=55  (đứng chờ)
```

Ba kết cục:

| Người nhận | Trạng thái | Người gửi thấy | Nhật ký Windows |
|---|---|---|---|
| bấm **Nhận** | `ACCEPTED` → byte chảy | tiến độ, rồi xong | `User accepted N items from …` |
| bấm **Từ chối** | `REJECTED` | "Đã từ chối" | `User rejected transfer …` |
| không bấm 60 giây | `EXPIRED` | "Không phản hồi" | `No answer within 60 s - declined offer … automatically (sender sees EXPIRED)` |

Người gửi tự rút lời mời (#256) là `CANCELLED` — kết cục thứ tư, tách riêng.

Hết 60 giây: bộ hẹn giờ của router chuyển Offer sang `EXPIRED` **đúng lúc** và
đánh thức người gửi đang chờ; hộp thoại tự đóng cùng lúc (`[Prompt] ReceiveFile
prompt closed: no answer within 60000 ms`). Hết giờ **không** tính là bấm Từ chối.

## Máy của tôi gửi tới

Như cũ tới bước verify — verify bằng khoá đã ghim, `publicKey` gửi kèm (nếu có) bị
bỏ qua. Offer được nhận ngay trong response của `POST /v1/transfers`
(`Auto-accepted N items from … - paired device, never asks`), hạn 24 giờ giữ nguyên.

## Client cũ, máy chưa ghép

Không gửi `publicKey` → `403 DEVICE_NOT_TRUSTED`, như trước — chỉ là ở bước verify
thay vì challenge.

## Giới hạn

- Tối đa **8** lời hỏi từ máy lạ chờ cùng lúc; lời mời thứ 9 nhận `429`. Phần còn
  lại của 64 chỗ chung dành cho Máy của tôi.
- Hộp dự phòng (`MessageBoxW`, khi TaskDialog không dựng được) không tự đóng; router
  vẫn đóng Offer đúng lúc nên bấm muộn không làm gì.
