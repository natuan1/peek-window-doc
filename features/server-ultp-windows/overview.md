# Server ULTP trên Windows — khung h1-only

**Trạng thái:** ✅ Implement xong 2026-09-19 ([Ticket 05 / #136](https://github.com/natuan1/peekvn/issues/136))
**Nghiệm thu còn treo:** demo với **iPhone thật**
**Kéo theo:** sàn hệ điều hành đã nâng lên **Windows 11** ([ADR-0009](../../adr/0009-tls-1-3-ghim-cung-thu-hep-san-he-dieu-hanh-thuc-te.md))

## Tính năng này là gì

Từ ticket này, Snappy **nhận** kết nối chứ không chỉ quảng bá. Nó mở cổng
`8443`, nói TLS 1.3, và trả lời đúng một câu hỏi: *"máy này là ai và làm được
gì"* (`GET /v1/info`, SPEC §9.3).

Nghe nhỏ, nhưng nó đóng một món nợ có chủ ý của Ticket 04: cổng 8443 đã được
quảng bá ra LAN từ 2026-09-18 trong khi **chưa ai nghe ở đó** — bài học 96 của
`peekvn` ở dạng cố ý và có hạn, *quảng bá sống lâu hơn server*. Vì vậy tiêu chí
của Ticket 04 chỉ dám dừng ở *"hai đầu **thấy** nhau"*. Từ đây lời quảng bá ấy
thành thật.

## Người dùng thấy gì

Hai thứ, và cả hai đều trên bảng trạng thái khay:

1. **Cột nền tảng của bảng thiết bị lân cận.** Trước đây một dòng chỉ có tên và
   địa chỉ; giờ nó nói máy kia là iPhone, Android, Mac hay Windows. Thông tin ấy
   **không** tới từ mDNS — TXT record của SPEC §8 đóng ở ba khoá `pv`/`id`/`port`
   và không khoá nào nói nền tảng — mà từ một lượt hỏi `GET /v1/info` sau khi đã
   nối được.

2. **Và chỗ cột ấy trống là một thông tin thật.** mDNS trả lời được *"có nghe
   thấy không"* mà không trả lời được *"có tới được không"* (bài học 55 và 169
   của `peekvn`). Một dòng ghi `· chưa trả lời` nghĩa là máy này nghe thấy peer
   kia mà gõ cửa không ai mở — trạng thái xấu nhất của discovery, và trước
   Ticket 05 nó trông giống hệt một máy đang khoẻ.

📐 Đo 19/09/2026, hai dòng cùng lúc trên bảng:

```
Thiết bị lân cận
• natuan1 · 192.168.1.35 · chưa trả lời      ← xác quảng bá của một tiến trình đã bị giết
• natuan1 · Mac · 192.168.1.35               ← reference/rust-host đang sống
```

## Phạm vi

**Có:**

- TLS 1.3 trên cổng 8443, ALPN chỉ `http/1.1`, certificate self-signed P-256
  **bền qua khởi động lại** (SPEC §9.2).
- Nghe **dual-stack** (IPv4 + IPv6) — ràng buộc bắt buộc, xem `workflow.md`.
- Router HTTP/1.1 **tự viết**: không Kestrel, không h2, không WebSocket.
- `GET /v1/info` đúng schema `device-info.schema.json`.
- Request rác / header xấu → lỗi có cấu trúc rồi đóng kết nối sạch, không crash.
- Giới hạn tài nguyên của SECURITY.md §4.3, gồm **hai dòng Reference Host còn
  ghi ❌** (timeout idle, trần connection đồng thời).
- `PeerInfoProbe` — lượt hỏi điền cột nền tảng.

**Chưa có, và cố ý:**

- Ghép đôi (Ticket 06), nhận push (07), resumable (08), pull (09).
- Vì vậy **bảng năng lực khai gần như toàn `false`** — xem
  [ADR-0010](../../adr/0010-bang-nang-luc-khai-theo-hanh-vi-khong-theo-lo-trinh.md).
  Đây là chỗ cố ý lệch với chữ trong tiêu chí nghiệm thu của #136.

## User story

> Là người dùng, tôi mở Snappy trên PC và mở Peek trên điện thoại cùng mạng.
> Bảng trạng thái của Snappy nói rõ mỗi máy hàng xóm là **loại máy gì**, và nói
> rõ máy nào **chưa gõ cửa được** — để tôi không ngồi chờ một lượt gửi vào một
> máy vốn không nhận được.

## Đọc tiếp

- [workflow.md](workflow.md) — luồng một kết nối đi qua, và vì sao dual-stack
- [implementation.md](implementation.md) — file, lớp, giới hạn
- [testing.md](testing.md) — biên raw-TCP, và những phép đo bằng công cụ ngoài .NET
- [ADR-0009](../../adr/0009-tls-1-3-ghim-cung-thu-hep-san-he-dieu-hanh-thuc-te.md) — TLS 1.3 ghim cứng và hệ quả về sàn HĐH
- [ADR-0010](../../adr/0010-bang-nang-luc-khai-theo-hanh-vi-khong-theo-lo-trinh.md) — bảng năng lực khai theo hành vi
