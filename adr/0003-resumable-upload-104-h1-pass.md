# ADR-0003: Resumable upload trên HTTP/1.1 — spike `104` PASS, advertise `resumableUpload`

Date: 2026-09-15
Status: Accepted

## Context

Server Windows đã chốt kiến trúc **h1-only tự viết** (TcpListener + SslStream, không Kestrel,
không h2 — chốt 2026-09-13). Trong khi đó toàn bộ phép đo resumable upload của hệ sinh thái
đều chạy trên đường **h2** (host đo ALPN ưu tiên h2, iOS tự chọn h2 — `peekvn` SPEC §17.3,
`reference/rust-host/tests/dataplane_104.rs`). Hyper còn từ chối có chủ đích mọi status 1xx
phía server, nên **chưa từng có phép đo nào về `104 (Upload Resumption Supported)` trên
HTTP/1.1 thuần** — đúng chỗ server Windows phải sống.

`104` là công tắc của cả tính năng (SPEC §17.3.2): không gửi được `104` thì không có
resumable upload dù mọi endpoint khác đúng. Vì vậy quyết định 2026-09-13 để tính năng này
**spike-gated**: chưa chứng minh thì không advertise.

## Decision

**Advertise `resumableUpload: ["httpbis-interop-6"]`.** Server Windows gửi `104` khi request
mang `Upload-Draft-Interop-Version: 6` (đúng draft-05), kèm `Location` trỏ upload resource;
fallback single-shot PUT vẫn giữ nguyên cho client không nói draft.

Cơ sở phép đo (spike `104-over-h1`, `peekvn` branch `prototype/104-over-h1`, commit `6059d18`):
iPhone thật (Safari/XHR, CFNetwork) ↔ PC qua Wi-Fi/LAN, plain HTTP/1.1, `104` flush **trước**
khi đọc body — 6/6 lượt nhận `201` trọn vẹn ở 1/10/50 MB (mode A có `104` so mode B control),
không một lần văng kết nối, tốc độ A≈B trong biên độ nhiễu (57,8 vs 60,5 MB/s ở 50 MB).

## Consequences

### Positive Consequences
- Upload iPhone→PC được CFNetwork lo đứt-kết-nối tự động (SPEC §17.5) — ô "upload × đứt kết
  nối" của ma trận resume được lấp mà client không cần code gì.
- Không cần phá kiến trúc h1-only, không cần thêm listener h2.
- Xử lý `104` measured ~0 chi phí hiệu năng.

### Negative Consequences (Trade-offs)
- **URLSession background (qua `nsurlsessiond`) chưa đo trực tiếp** — spike dùng Safari (CFNetwork
  in-process, cùng lõi parse). Kiểm chứng khi implement thật với app Peek.
- **Phát hiện phụ khi đo:** trên plain HTTP, client XHR không tự phát header `Upload-*` — CFNetwork
  có vẻ gate hành vi draft theo secure context (khác §17.5 đo trên host TLS). Production là TLS
  nên chuỗi khép (TLS → header tự phát → h1+104 xử lý đúng), nhưng server **chỉ nên gửi `104`
  trên kết nối TLS** — vừa khớp hành vi đã đo, vừa tránh kẻ nghe thường giả mạo upload resource.
- Server h1 phải tự viết parser xử lý interim response — không thư viện làm hộ (hyper cấm 1xx).

## Alternatives Considered
- **Single-shot only (không advertise)**: an toàn tuyệt đối đã đo (512 MB qua PUT thường), nhưng
  mất resume khi đứt Wi-Fi — with spike PASS, lựa chọn này không còn lý do.
- **Thêm listener h2 riêng như rust-host**: phá quyết định h1-only, ích lợi không đo được so với
  chi phí duy trì hai stack.

## Related
- [ADR-0002](0002-ui-stack-aot-spike-pass.md) — pattern spike trước đó (cùng workspace)
- `peekvn` [ADR-0010](https://github.com/natuan1/peekvn) — interop version 6
- `peekvn` `protocol/SPEC.md` §17.3–17.5 — nguồn sự thật giao thức
- Primary source: `peekvn` branch `prototype/104-over-h1`

## Decision Log
- 2026-09-15: Accepted — spike PASS trên iPhone thật cùng ngày.
