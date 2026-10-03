# Đích thả và gửi từ Mí — Ticket 13

**Trạng thái:** implement xong 2026-10-03 trên nhánh `feat/144-mi-drop-targets`
([#144](https://github.com/natuan1/peekvn/issues/144)). Đứng trên Ticket 09 (gửi bằng pull),
10 (Mí), 11 (TempDrops) và 12 (khay thẻ Shelf).

## Làm được gì

"Kéo tệp bay sang điện thoại" — không menu, không hộp chọn tệp:

```
kéo tệp lên cạnh trên  →  Mí nở ra, hiện Đích thả  →  nhắm một avatar (sáng xanh)  →  thả
                                                      →  Mí co thành pill: Đang mời → Đang gửi · 42% → Đã gửi
```

| Đích thả | Thả vào thì |
|---|---|
| **Ô Shelf** (đầu trái) | Mục vào Ngăn, như cả dải từ Ticket 11 |
| **Avatar** từng Thiết bị tin cậy | Mục vào Ngăn **trước**, rồi mời thiết bị ấy kéo về (pull, Ticket 09); Mí co thành pill |
| **"Tất cả"** (mép phải) | Mục vào Ngăn, rồi menu mọi thiết bị hiện ngay chỗ thả: chọn một máy là gửi, "Chỉ giữ ở Shelf" hay Esc là thôi |
| Khoảng trống giữa hai nhóm | như ô Shelf — một cú thả không trúng đích nào vẫn không mất tệp |

Năm trạng thái Mí (CONTEXT.md): Sẵn sàng nhận (2) vẽ Đích thả; con trỏ trên một đích là
Nhắm đích (3); thả lên avatar là Đang chạy (4).

**Thư mục gửi được.** Thả một thư mục là mời mọi tệp của cây, `relativePath` giữ đường từ tên
thư mục trở xuống (SPEC §18.3: không zip). 📐 iPhone dựng lại đúng `Thu muc C/con/hai.txt`,
SHA-256 trùng. Áp cho cả "Gửi tệp tới…" ở menu khay, vì cùng `OfferFiles`.

## Pill nói gì

| Pha | Chữ | Khi nào |
|---|---|---|
| Đang chờ | "Đang mời X…" → "Chờ X nhận…" | lời mời đã tạo; chữ đổi khi X đã kéo lời mời về |
| Đang gửi | "X đang bắt đầu nhận…" → "Đang gửi tới X · 42%" + thanh | byte thật đã rời máy; chưa có byte thì không vẽ "0%" |
| Xác nhận | "Đã gửi tới X" (xanh lá) | mọi mục xong; đứng 2 s |
| Hỏng | đỏ, đứng 5 s | xem bảng dưới |
| Thôi theo dõi | "X chưa trả lời — lời mời giữ 24 giờ" | X đã thấy lời mời mà 2 phút chưa ai bấm |

Bấm vào pill là ẩn nó; lượt gửi chạy tiếp ở server và kết cục tới bằng bong bóng. Một lượt kéo
mới cũng đẩy pill đi — Mí phải sẵn để thả.

## Hỏng thì sao

Mục **không bao giờ** rời Ngăn vì một lượt gửi hỏng — chúng vào Ngăn trước khi lời mời tồn tại.

| Tình huống | Pill | Sau đó |
|---|---|---|
| X không hỏi lời mời lần nào trong 45 s (app đóng, máy ngoài mạng) | "Không thấy X — mở app trên máy ấy để nhận" | bong bóng "Đã mời X… mở app trên X để nhận — lời mời giữ 24 giờ" |
| Byte dừng 20 s giữa chừng | "Mất kết nối với X — tệp vẫn ở Shelf" | bong bóng "Lượt gửi tới X dừng giữa chừng" |
| X tự tạm dừng | "X tạm dừng nhận" — **không** tính là đứt | |
| Bản ghép đôi cũ chưa có bảng năng lực | "Cần ghép đôi lại với X" | hộp thoại hướng dẫn ghép lại (sau khi cú thả xong) |
| X từ chối / huỷ / hết hạn | câu riêng cho từng cái | |

## Không làm (theo ticket hoặc cố ý)

- **Capsule media** ở ticket này chưa tồn tại — từ Ticket 15 nó có, xem [Capsule media](../capsule-media-windows/overview.md). Ticket này chỉ đòi hợp đồng: `MiWindow.FootprintChanged` báo khung
  Mí đang chiếm (hoặc `null` khi ẩn), một lần mỗi trạng thái đích. M3 nghe nó và trượt ra khỏi
  khung ấy. Chưa có ai đăng ký, nên chưa có phép đo nào cho nó.
- **Thư mục rỗng** không đi qua dây: P0 chưa gửi mục `directory` (SPEC §18.2). Thư mục không có
  tệp nào thì pill nói "Không có tệp nào để gửi".
- Avatar không có chấm "trực tuyến": iPhone chỉ quảng bá khi app mở, nên một chấm xám đọc như
  "không gửi được" trong khi lời mời vẫn sống 24 giờ. Phép đo thật là 45 s chờ ở pill.
- Pill ẩn giữa lúc Đang gửi thì không còn ai canh đứt; kết cục cuối vẫn tới bằng bong bóng
  "Đã gửi…" khi Phiên terminal.

## Đọc tiếp

- [workflow.md](workflow.md) — luồng từ cú thả tới byte cuối
- [implementation.md](implementation.md) — tệp, giao kèo, quyết định
- [testing.md](testing.md) — test tự động, nghiệm thu thật, số đo
