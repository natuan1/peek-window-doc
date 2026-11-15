# Thêm vào Máy của tôi — quy trình

## Người dùng

Xem bảng ở [overview](overview.md). Kết cục ở máy A (người bấm Thêm):

| Kết cục | A thấy |
|---|---|
| B Đồng ý | bong bóng "Đã thêm B vào Máy của tôi"; menu có "Gửi tệp tới B…" |
| B Từ chối | hộp "Chưa được đồng ý — B đã bấm Từ chối. Không có gì thay đổi." |
| B đang ẩn | hộp "B đang ẩn — nhờ họ bấm Hiện với máy xung quanh" |
| Không ai bấm 2 phút | hộp "B chưa trả lời" |
| Không tới được | hộp "Không tới được B — kiểm tra cùng Wi-Fi" |

Máy B (được hỏi): Đồng ý thì không hộp nào nữa (máy kia hiện trong Máy của tôi);
Từ chối là cú bấm của chính họ; Đồng ý muộn sau khi phiên hết hạn thì có hộp "đã hết hạn".

## Hệ thống (dây ULTP v1, không đổi)

```
A: POST /v1/pairings            (từ cú bấm Thêm — không bao giờ tự phát)
B: 403 DEVICE_HIDDEN            nếu đang ẩn — không lời hỏi
B: 201 + hỏi người dùng
A: POST …/confirm {confirmed:true} ngay, chờ tới 135 s
B: Đồng ý → 200, ghim A, tự ẩn  |  Từ chối → 403 PAIRING_REJECTED "Chưa được đồng ý."  |  120 s → 410
A: 200 → ghim B (SPKI quan sát được + bảng năng lực của B)
```

SAS vẫn được tính trên dây nhưng **không** lên màn hình peek.

Gửi qua lại sau đó: A → B và B → A đều xác thực hai chiều bằng khoá đã ghim. Máy
trả lời (B) không có pin TLS của server A, nên nó mở kênh như lúc ghép rồi verify
chữ ký server bằng **khoá thiết bị** của A.
