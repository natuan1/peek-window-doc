# ADR-0009: TLS 1.3 ghim cứng — và nó thu hẹp sàn hệ điều hành *thực tế* của Snappy

Date: 2026-09-19
Status: Accepted

> ⚠️ Kho `peekvn` có một dãy ADR **độc lập** với dãy này. `ADR-0009` ở đây không
> phải `ADR-0009` của `peekvn`.

## Context

[Ticket 05 (#136)](https://github.com/natuan1/peekvn/issues/136) dựng khung
server ULTP cho Snappy: `TcpListener` + `SslStream`, router HTTP/1.1 tự viết,
`GET /v1/info`. Từ ticket này Snappy **nhận** kết nối, không chỉ quảng bá.

Ba nguồn sự thật nói về phiên bản TLS, và chúng không để lại chỗ cho lựa chọn:

- `protocol/SECURITY.md` §2: *"MUST NOT dùng TLS 1.2 hoặc thấp hơn."* Kèm bảng
  kênh: *"TLS 1.3, chỉ các cipher suite AEAD chuẩn của 1.3"*.
- `protocol/SPEC.md` §11: *"Implementation chính thức MUST dùng HTTPS với
  TLS 1.3."*
- `AGENTS.md` §4.1: `SECURITY.md` **đè tất cả** về crypto.

Cùng lúc đó, [CONTEXT.md](../CONTEXT.md) chốt từ 2026-09-13: *"Floor hệ điều
hành: Windows 10 1809 (gồm LTSC 2019)"*, và `Directory.Build.props` ghim
`SupportedOSPlatformVersion = 10.0.17763.0`.

Hai câu ấy mâu thuẫn nhau ở một chỗ không ai để ý lúc chốt: **.NET trên Windows
không tự cài stack TLS, nó gọi SChannel của HĐH.** Microsoft tài liệu hoá TLS
1.3 trong SChannel là tính năng của **Windows 11 và Windows Server 2022**. Trên
Windows 10 — kể cả 22H2 — `SslProtocols.Tls13` không có gì để thoả thuận.

## Decision

**Ghim `EnabledSslProtocols = SslProtocols.Tls13`, không fallback.**

Không phải `Tls12 | Tls13`. Để mở là để một client cũ kéo cả kênh xuống 1.2 mà
không ai thấy, và lúc ấy câu MUST của `SECURITY.md` thành một câu về ý định chứ
không phải một thuộc tính của hệ thống. Kèm một phép khẳng định **sau**
handshake (`ssl.SslProtocol != Tls13` ⇒ đóng kết nối kèm dòng nhật ký): một
thiết lập là một lời hứa, một phép kiểm mới là hàng rào.

**Và ghi nhận hệ quả thay vì giấu nó:** sàn hệ điều hành *thực tế* cho **vai
server** của Snappy là **Windows 11 / Server 2022**, không phải Windows 10 1809.

Sàn *sản phẩm* không đổi — 1809 vẫn là con số CONTEXT.md chốt, và
`SupportedOSPlatformVersion` vẫn ghim 10.0.17763.0. Trên Windows 10, Snappy vẫn
cài được, vẫn chạy nền, vẫn quảng bá và vẫn **thấy** thiết bị lân cận; chỉ
không ai gõ cửa được nó.

## Consequences

**Cái mất.** Trên Windows 10, Snappy rơi vào đúng hình dạng xấu nhất của
discovery mà [ADR-0008](0008-discovery-qua-responder-in-box-windows.md) và bài
học 55 của `peekvn` mô tả: máy hiện trong danh sách của iPhone, bấm vào thì
timeout. Khác biệt quan trọng là lần này ta **biết trước**, nên nó là một câu
chữ phải viết ra màn hình chứ không phải một lỗi phải đi tìm.

**Cái được.** Không có đường nào để kênh rơi xuống 1.2, kể cả do một dòng cấu
hình sửa nhầm sau này.

**Chưa nghiệm thu được, và nói rõ là chưa:** không có máy Windows 10 sạch để
đo. Câu "TLS 1.3 không có trên Windows 10" ở đây đến từ **tài liệu Microsoft**,
không từ một phép đo của dự án — nên nó là giả thuyết mạnh, không phải bằng
chứng. Đúng theo quy tắc của CONTEXT.md §"Ưu tiên Windows 11": ghi ra, để treo,
không đánh dấu xanh và cũng không im lặng bỏ qua.

**Việc phải làm trước 1.0**, và nó không thuộc Ticket 05:

1. Đo thật trên một máy Windows 10 1809 hoặc 22H2.
2. Nếu đúng như tài liệu, [Ticket 17 — Onboarding](https://github.com/natuan1/peekvn/issues/148)
   phải phát hiện và nói thẳng với người dùng rằng máy này chỉ **gửi** được,
   chưa **nhận** được — bằng tiếng Việt, kèm việc phải làm. Một câu ở onboarding
   rẻ hơn nhiều so với để họ tự phát hiện bằng một lượt gửi timeout.
3. Hoặc chủ repo quyết định nâng sàn sản phẩm lên Windows 11. Đó là một quyết
   định về thị trường, không phải về mã — nên nó thuộc về chủ repo, và ADR này
   chỉ đặt nó lên bàn.

## Đã đo được (2026-09-19, bản AOT thật, máy dev Windows 11)

Dụng cụ là `openssl s_client`, tức **một stack TLS ngoài .NET** — cùng lý do
ADR-0008 đo mDNS bằng `mdns-browse` của Rust: hai stack .NET sẽ đồng ý với nhau
kể cả ở chỗ cả hai cùng sai.

| Phép đo | Kết quả |
|---|---|
| Kênh thoả thuận ra | `TLSv1.3` · `TLS_AES_256_GCM_SHA384` · `ecdsa_secp256r1_sha256` |
| Client chỉ nói TLS 1.2 | alert **70** (`protocol_version`) |
| Client khai `h2,http/1.1` | server chọn `http/1.1` |
| Client khai **chỉ** `h2` | alert **120** (`no_application_protocol`) |

Cipher suite và loại chữ ký khớp `SECURITY.md` §2: AEAD chuẩn của 1.3, và
P-256 — *"MUST NOT dùng curve khác P-256 ở v1"*.

## Một quyết định nhỏ hơn đi kèm: ALPN là **tuỳ chọn của client**

Bản đầu của server đòi `NegotiatedApplicationProtocol == http/1.1` sau
handshake. 📐 Nó chặn ngay chính client của Snappy: `SocketsHttpHandler`
**không gửi ALPN** khi request ghim HTTP/1.1, nên trường ấy rỗng và kết nối bị
đóng với một `SocketException` không nhắc một chữ nào tới ALPN.

RFC 7301 nói ALPN là tuỳ chọn. Luật đúng là: chấp nhận `http/1.1` **hoặc không
có gì**, từ chối phần còn lại. Trường hợp "client khai h2 thôi" không đi qua
phép kiểm ấy — SChannel tự đóng handshake bằng alert 120, vì server chỉ khai
`http/1.1`.

Bài học chung: **một phép khẳng định chặt hơn giao thức là một lỗi, không phải
một hàng rào.** Nó đã đỏ ở test trước khi ra tới LAN, nhưng chỉ vì có một test
dùng chính client thật thay vì một client dựng riêng cho test.

## Related

- [ADR-0008](0008-discovery-qua-responder-in-box-windows.md) — discovery qua
  responder in-box; chỗ ràng buộc "server phải nghe dual-stack" sinh ra.
- [ADR-0006](0006-dong-goi-velopack-cai-peruser.md) — cài PerUser, không admin.
- [Server ULTP trên Windows](../features/server-ultp-windows/overview.md)
- `peekvn/protocol/SECURITY.md` §2, §4.3 · `peekvn/protocol/SPEC.md` §11, §25
