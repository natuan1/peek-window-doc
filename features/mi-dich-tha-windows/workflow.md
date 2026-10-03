# Đích thả — luồng

## Từ lượt kéo tới cú thả

```
SysDragImage / polling          IDropTarget (OLE)                 MiStateMachine
──────────────────────          ─────────────────                 ──────────────
lượt kéo bắt đầu  ───────────►                                    Nghỉ → Gợi ý (vạch)
                                DragEnter (có tệp)  ─────────────►  ở 80 ms → Sẵn sàng nhận
                                                                    └ LayTargets(): hỏi Trust Store,
                                                                      ghi "Drop targets: shelf x=…, device … x=…, all x=…"
                                DragOver(x, y) ── HitTest ───────►  vào một đích → Nhắm đích
                                                                    ra khoảng trống → Sẵn sàng nhận
                                Drop(x, y) ── HitTest:
                                  avatar   → IMiHost.SendTo      →  RunStarted → Đang chạy (pill)
                                  "Tất cả" → IMiHost.ChooseDevice →  Dropped → Nghỉ; menu sau khi Drop trả về
                                  còn lại  → IMiHost.TakeIntoShelf →  Dropped → Nghỉ
```

Không lời gọi nào trong `Drop` mở hộp thoại: nguồn kéo (Explorer) đang chờ `Drop` trả về, và một
hộp thoại ở đó treo luôn Explorer. Menu "Tất cả" và hộp thoại "Cần ghép đôi lại" đi qua
`PostMessageW` rồi chạy sau.

## Gửi

```
SendTo(device, payload)
  TakeDropItems(payload)        Mục vào Ngăn (DropIntake + Shelf.Add) — TRƯỚC lời mời
  OfferShelfItems(device, items)
    UltpRouter.OfferFiles(deviceId, paths)      Ticket 09; thư mục bung thành từng tệp
      Offered     → SendPill.Offered(transferId)
      còn lại     → SendPill.Refused(result) [+ hộp thoại nếu cần người làm gì đó]
```

## Pill theo dõi Phiên

Bộ đếm 250 ms trên luồng UI:

```
TransferStore.Read(id, me, PillProgress.Of)     dưới khoá: state, lần bị hỏi, byte đã đi, mục xong
  → SendPill.Update(progress, now)              logic thuần
  → đổi pha thì ghi "Pill Waiting -> Sending"
  → Over (hết thời gian giữ kết cục) → IMiHost.PillEnded → RunFinished → Nghỉ (hoặc Gợi ý nếu đang kéo)
```

"Bên kia có mở app không" ở `pull` chỉ đo được bằng một thứ: bên kia có kéo danh sách lời mời
về không (`Transfer.TimesSeenByReceiver`). iPhone hỏi mỗi 5 giây khi app mở, nhưng 📐 có khe tới 21 s trên máy thật; 45 s không thấy hỏi
là "Không thấy X".

## Bong bóng

| Kết cục pill | Bong bóng |
|---|---|
| Không thấy X, hoặc thôi theo dõi, hoặc bị ẩn lúc Đang chờ | `SendNotices.ForOffer(Offered)` — "mở app trên X để nhận, lời mời giữ 24 giờ" |
| Đứt giữa chừng | `SendNotices.ForStalled` |
| Xong / terminal | không thêm; `TellUserTransferFinished` vẫn báo "Đã gửi…" như đường menu khay |
