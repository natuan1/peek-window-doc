# Capsule media — luồng

```
Snappy.exe (nền)                         Snappy.exe --media-capsule (con)
────────────────                         ─────────────────────────────────
AppHost.StartCapsule ── ChildProcess ──▶ MediaCapsuleWindow (luồng UI, layered window)
MiWindow.FootprintChanged                   │
   └─ "footprint\tL\tT\tR\tB" ── stdin ──▶  SetFootprint → Update → trượt
AppLog["Capsule"] ◀── stdout "log\t…" ──    │
                                            ├─ MediaSessions (luồng MTA riêng)
                                            │    RequestAsync → session manager
                                            │    sự kiện CurrentSession/Sessions/MediaProperties/PlaybackInfo
                                            │      → RequestRead (gom 50 ms) → ReadAsync → MediaNow
                                            │    ReadArtAsync → OpenReadAsync → WIC → uint[] PBGRA
                                            │    SendAsync → TryTogglePlayPause / TrySkipNext
                                            └─ HeadsetBatteryReader (thread pool, mỗi 5 phút khi đang hiện)
                                                 SetupDi 0x180F → container Audio + Connected → GATT 0x2A19
```

## Một lần đổi bài

1. App phát đổi metadata. GSMTC bắn `MediaPropertiesChanged` trên luồng của nó, và delegate dựng tay gọi `RequestRead`.
2. `RequestRead` gom mọi sự kiện trong 50 ms thành một lượt đọc. Lượt đọc chạy trên luồng media: phiên hiện tại, `PlaybackInfo`, `Controls`, rồi `TryGetMediaPropertiesAsync`.
3. `MediaNow` về luồng UI qua `Post` (một hàng đợi và một `PostMessage`). `CapsuleModel.Update` quyết định hiện hay ẩn, và tính thời gian nán khi tạm dừng.
4. Khoá bài (app + tên + nghệ sĩ) đổi thì nạp bìa. Cùng một bài mà bìa chưa có thì thử lại, tối đa 3 lần, vì nhiều app gửi bìa sau tên bài.
5. `Update` tính chỗ đứng (`CapsuleLayout.Place`), ghi một dòng `Capsule shown (…) x=…` hay `Capsule hidden (…)`, rồi vẽ.

## Một cú kéo tệp

1. Mí hiện vạch Gợi ý → `FootprintChanged(khung)` → cha ghi một dòng vào stdin của con.
2. Khung Mí chạm capsule, nên capsule trượt sang phải khung, cách một khe 8 DIP, trong 160 ms (cubic ease-out).
3. Khay nở 90 DIP thì khung đổi. Capsule đã ở ngoài nên đứng yên.
4. Thả hay Esc → Mí ẩn → `FootprintChanged(null)` → capsule trượt về nhà.

## Bấm nút

`WM_LBUTTONDOWN` → `CapsuleParts.HitTest` → kiểm app có cho lệnh ấy không. Không cho thì ghi `does not allow it` và dừng. Cho thì chạy `SendAsync` trên luồng media, ghi `accepted`/`refused`, rồi đọc lại phiên. Cửa sổ là `WS_EX_NOACTIVATE`: bấm không lấy tiêu điểm khỏi cửa sổ đang gõ.
