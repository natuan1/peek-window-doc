# ADR-0001: Thương hiệu ≠ tên service mDNS (Snappy giữ `_peek._tcp`)

Date: 2026-09-13 (quyết định chốt; ADR ghi nhận 2026-09-14)
Status: Accepted

## Context

Ngày 2026-09-13 thương hiệu được chốt là **Snappy** (tái xác nhận cùng ngày sau khi phát hiện mobile đang chạy dưới tên cũ "Peek"); app mobile sẽ đổi label theo. Tuy nhiên, service mDNS mà toàn bộ hệ sinh thái đang phát sóng và khám phá là **`_peek._tcp`** (port 8443, TLS 1.3):

- iOS ("Peek") và Android đang advertise `_peek._tcp` — đã phát hành tới người dùng.
- `protocol/SPEC.md` (nguồn chuẩn hiện hành của ULTP) và interop suite/fixtures của monorepo `peekvn` đều neo vào tên service này.
- Tên service là **định danh kỹ thuật ẩn với người dùng cuối**: người dùng không bao giờ thấy `_peek._tcp` trên màn hình — họ thấy thương hiệu Snappy.

Đặt ra câu hỏi: đổi thương hiệu thì có đổi luôn tên service cho "đồng bộ" không?

## Decision

1. **Thương hiệu người dùng thấy: Snappy** — trên Windows và label app mobile.
2. **Service mDNS giữ nguyên `_peek._tcp` vĩnh viễn**; port 8443/TLS 1.3 không đổi.
3. Nguyên tắc tổng quát: **brand ≠ định danh giao thức**. Định danh kỹ thuật của giao thức (tên service mDNS, port, path REST) chỉ đổi khi có phiên bản giao thức chính thức kèm lộ trình tương thích, không đổi theo marketing.

## Consequences

### Positive Consequences
- Không phá tương thích với iOS/Android đã phát hành; không phải chạy đợt chuyển tiếp dual-advertise trên 3 nền tảng + spec + interop fixtures.
- Không có mâu thuẫn thương hiệu trong thực tế: service name ẩn với người dùng, họ chỉ thấy Snappy.
- Bản Windows (Snappy) discover thiết bị cũ bằng đúng `_peek._tcp` như mọi client hiện hữu — zero-migration.

### Negative Consequences (Trade-offs)
- Tên kỹ thuật "peek" sống âm ỉ vĩnh viễn trong giao thức, log, mã nguồn — khi debug sẽ thấy `_peek._tcp` cạnh brand Snappy.
- Cần kỷ luật tài liệu: mọi tài liệu/issue phải nói rõ *brand = Snappy, service = `_peek._tcp`* để agent không "sửa giúp" cho khớp tên và làm vỡ tương thích.

## Alternatives Considered

- **Đổi sang `_snappy._tcp`**: phá tương thích với mobile đã phát hành, phải dual-advertise giai đoạn chuyển, đồng bộ đạc trên 3 nền tảng + spec + fixtures — trong khi lợi ích thương hiệu bằng 0 vì service ẩn với người dùng. ❌
- **Dual-advertise cả hai tên vĩnh viễn**: phức tạp hoá discovery vô ích, hai nguồn sự thật cho cùng một thiết bị. ❌

## Related

- [CONTEXT.md](../CONTEXT.md) — định nghĩa **Snappy**, **ULTP** (nguồn chuẩn hiện hành: `protocol/SPEC.md`)
- Monorepo `peekvn`: `protocol/SPEC.md`, `interoperability/`, `apps/ios/`, `apps/android/`

## Decision Log
- 2026-09-13: Chốt quyết định trong phiên làm việc kế hoạch
- 2026-09-14: Ghi nhận thành ADR-0001
