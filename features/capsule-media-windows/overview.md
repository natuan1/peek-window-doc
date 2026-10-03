# Capsule media trên Windows

[Ticket 15](https://github.com/natuan1/peekvn/issues/146), 2026-10-03, nhánh `feat/146-media-capsule`.

Khi nhạc đang phát, một viên capsule nhỏ hiện ở cạnh trên màn hình chính, giữa dải Mí. Nó có bìa album, dòng "Bài — Nghệ sĩ", nút phát/tạm dừng, nút bài kế, và % pin tai nghe Bluetooth nếu có. Kéo tệp lên cạnh trên thì Mí hiện, và capsule trượt sang bên nhường vùng thả.

## Người dùng thấy gì

| Lúc | Capsule |
|---|---|
| Có app đang **phát** (Spotify, Chrome, Windows Media Player…) | hiện ở giữa dải Mí, 300 × 32 DIP, trắng đục như Mí, bo hai góc dưới |
| Bấm ⏸ / ▶ / ⏭ | app phát làm theo. Nút mà app không cho bấm thì vẽ mờ, bấm vào không làm gì |
| **Tạm dừng** | nán lại 30 s để bấm phát lại, rồi ẩn |
| Phiên đã dừng **từ trước** (Riot Client, tab YouTube đã dừng…) | không hiện |
| Kéo tệp tới cạnh trên | Mí hiện, capsule trượt sang phải khung Mí trong ~165 ms. Thiếu chỗ thì sang trái, thiếu cả hai thì ẩn. Mí ẩn thì capsule về chỗ cũ |
| App toàn màn hình (game, video F11) | ẩn trong ≤ 1,5 s, hiện lại khi app ấy đi |
| Không có app media nào | ẩn hẳn, không còn cửa sổ nào bắt chuột ở đó |

## Pin tai nghe

| Tình huống | Ô pin |
|---|---|
| Không có tai nghe BLE nào đang kết nối có Battery Service `0x180F` | **không vẽ ô pin** |
| Có tai nghe, đọc được 1–100 | `🎧 85%` |
| Có tai nghe, đọc hỏng, hay số rác (0, > 100) | `🎧 -`. **Không bao giờ `0%`** |
| Tai nghe true-wireless có hai dịch vụ (trái, phải) | số **nhỏ nhất** đọc được |

Chỉ thiết bị mà Windows xếp vào nhóm `Audio` mới được coi là tai nghe, nên pin của chuột BLE không hiện. Pin được đọc lại mỗi 5 phút khi capsule đang hiện.

## Không làm (theo ticket hoặc cố ý)

- **Tai nghe Bluetooth Classic** (A2DP/HFP, phần lớn tai nghe hiện nay) không có GATT, nên không có ô pin. Pin của chúng đi đường HFP, ngoài phạm vi ticket ("qua GATT Battery Service 0x180F").
- **Chưa có công tắc bật/tắt.** Kế hoạch tổng nói module Media "chỉ kích hoạt khi bật trong Settings", nhưng Settings chưa tồn tại.
- **Capsule luôn ở màn hình chính.** Mí đi theo màn hình có con trỏ. Kéo tệp ở màn hình phụ thì không xung đột, nhưng capsule cũng không nằm "trên Mí" ấy.
- Bấm vào chữ hay bìa không làm gì, không mở app phát. Capsule nằm trên dải tab của trình duyệt đang phóng to, nên phần ấy nuốt cú bấm. Đó là lý do capsule chỉ hiện khi đang phát.
- Chỉ đọc **phiên hiện tại** của Windows (`GetCurrentSession`), không duyệt mọi phiên.

## Treo

- **Pin với tai nghe BLE thật**: máy dev không có Bluetooth adapter. Mới nghiệm thu nhánh "không có tai nghe".
- **Spotify**: máy dev không cài. Đã nghiệm thu với Chrome (`navigator.mediaSession`) và Windows Media Player.

## Đọc tiếp

- [workflow.md](workflow.md) — luồng từ GSMTC tới pixel
- [implementation.md](implementation.md) — tệp, quyết định, nhật ký
- [testing.md](testing.md) — test tự động, nghiệm thu thật, số đo
- [ADR-0018](../../adr/0018-capsule-media-tien-trinh-con-winrt-bang-vtable-tay.md)
