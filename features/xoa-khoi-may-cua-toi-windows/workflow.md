# Xoá khỏi Máy của tôi — quy trình

## Người dùng (máy xoá, A)

| Lúc | A thấy | Hai máy |
|---|---|---|
| Bấm Xoá | dòng "Đã xoá · Hoàn tác trong vài giây", nút Hoàn tác | chưa gì đổi — vẫn tự nhận nhau |
| Bấm Hoàn tác (< 5 s) | dòng về như cũ | không gì đổi |
| Hết 5 s | dòng biến mất | A thôi tin B; lời huỷ ghép đi nếu B đang ở mạng |
| Thoát app giữa 5 s | — | coi như hết 5 s: A gỡ B, lời nợ đi ở lần chạy sau |

Máy bị xoá (B) không hỏi gì: B chỉ thấy A chuyển xuống Máy xung quanh.

## Hệ thống

```
A: hết 5 s → gỡ B khỏi Trust Store, thu hồi token → ghi lời nợ (unpair-notices.json)
A: thấy B (discovery, hoặc ngay lúc này) → bootstrap TLS → auth bằng khoá đã ghim của B
A: DELETE /v1/pairings/{deviceId của A}
B: 204 — gỡ A, thu hồi token   |  404 / 403 DEVICE_NOT_TRUSTED ở verify — đã gỡ từ trước
A: xoá lời nợ khi 204/404/403; mọi kết cục khác giữ cho lần gặp sau
```

Lời nợ không tới — B gửi tới A:

```
B → A: POST /v1/auth/verify (nhánh Máy của tôi) → 403 DEVICE_NOT_TRUSTED
  kênh ghim SPKI của A → B gỡ A ngay
  kênh bootstrap (bản ghi phía responder) → B xác thực lại nhánh máy lạ, ghim ĐÚNG SPKI
     đã trả 403, verify chữ ký bằng khoá đã ghim của A → qua thì gỡ; A đang ẩn thì không gỡ
B → A: gửi như máy xung quanh → A hỏi Nhận
```
