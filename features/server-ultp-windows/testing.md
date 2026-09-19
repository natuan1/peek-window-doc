# Kiểm thử — Server ULTP trên Windows

## Biên mới: raw-TCP trên loopback

Ticket 05 mở một biên test mà `tests/Snappy.Tests` chưa từng có:
`UltpServerTests` và `UltpServerLimitsTests` dựng một **server ULTP thật** trên
cổng tạm của loopback rồi gõ cửa bằng **byte thô** qua `SslStream`.

**Không `HttpClient`, và đó là cả điểm của nó.** Một HTTP client sửa hộ mọi thứ
nó thấy sai — thêm `Host`, gộp header, không cho gửi byte rác — nên nó không bao
giờ hỏi được câu *"server làm gì khi bên kia gửi bậy"*. Mà đó đúng là nửa sau
của tiêu chí nghiệm thu #3. Prior art là `reference/rust-host/tests/`.

Chúng vẫn in-process, nên chúng **không** thay được interop suite. Seam chính
của Windows vẫn là `interoperability/run.sh`.

## Cái gì kiểm ở đâu

| Phần | Kiểm bằng |
|---|---|
| `/v1/info` đúng sáu trường `required` của schema | `UltpServerTests` |
| Không lọt SPKI pin hay secret nào ra `/v1/info` | `UltpServerTests` |
| Byte ra dây không escape tiếng Việt và dấu `+` | `UltpServerTests` |
| TLS 1.3 + ALPN `http/1.1` trên kênh thật | `UltpServerTests` |
| Client **không khai ALPN** vẫn được phục vụ | `UltpServerTests` |
| Keep-alive nhiều request một kết nối | `UltpServerTests` |
| `HEAD` trả đúng `Content-Length` của `GET`, không thân | `UltpServerTests` |
| Dual-stack: `127.0.0.1` **và** `::1` | `UltpServerTests` |
| 8 dạng request rác → `400` có cấu trúc rồi đóng | `UltpServerTests` (`[Theory]`) |
| Thiếu `Host` → `400` | `UltpServerTests` |
| Rác không làm chết server cho người kế tiếp | `UltpServerTests` |
| Đầu request > 16 KB → `431` | `UltpServerTests` |
| `404` / `405` vẫn là lỗi **có cấu trúc** | `UltpServerTests` |
| Thân khai 4 GB → `413` **trước khi đọc byte nào** | `UltpServerTests` |
| Bộ nhớ **không tỉ lệ** với kích thước thân | `UltpServerTests` |
| Timeout idle 60 giây (đồng hồ rút còn 300 ms) | `UltpServerLimitsTests` |
| …và **không** đóng oan một kết nối đang bận | `UltpServerLimitsTests` |
| Trần 64 connection đồng thời, và slot được trả lại | `UltpServerLimitsTests` |
| Trần 64 header | `UltpServerLimitsTests` |
| Bắt tay TLS quá hạn thì bị đóng (Slowloris) | `UltpServerLimitsTests` |
| 60 request/phút → `429` + `Retry-After` hợp lệ | `UltpServerLimitsTests` |
| Cửa sổ trượt: mở lại đúng lúc, không mở lại cho kẻ bắn liên tục | `SlidingRateLimiterTests` |
| Bảng đếm không lớn vô hạn theo số địa chỉ đã gặp | `SlidingRateLimiterTests` |
| TLS Identity bền qua khởi động lại | `TlsIdentityTests` |
| Mất tệp `.der` thì dựng lại được từ khoá | `TlsIdentityTests` |
| Khoá TLS ≠ khoá danh tính | `TlsIdentityTests` |
| Lời khai capability khớp hành vi, **cả 12 dòng** | `CapabilityTruthTests` |
| Cột nền tảng: hỏi được, thử địa chỉ sau, trả `null` khi không tới được | `PeerInfoProbeTests` |
| Lời khai sai hình dạng của peer bị bỏ, không ra màn hình | `PeerInfoProbeTests` |
| `Uri` nuốt zone IPv6 nên zone phải đi đường khác | `PeerInfoProbeTests` |
| Cột nền tảng trong bảng: không bị lượt nghe thấy sau xoá mất | `PeerTableDescribeTests` |

Tổng: **170 test** xanh (trước ticket này là 117).

## Hai phép đo không phải test

### 1. Bằng công cụ ngoài .NET

Cùng lý do Ticket 04 đo mDNS bằng `mdns-browse` của Rust: hai stack .NET sẽ
đồng ý với nhau kể cả ở chỗ cả hai cùng sai.

| Câu hỏi | Lệnh | Kết quả (2026-09-19) |
|---|---|---|
| Kênh là gì? | `openssl s_client -connect <ip>:8443 -alpn http/1.1 -brief` | `TLSv1.3` · `TLS_AES_256_GCM_SHA384` · `ecdsa_secp256r1_sha256` |
| TLS 1.2 bị từ chối? | `… -tls1_2` | alert **70** |
| Khai **chỉ** h2? | `… -alpn h2` | alert **120** |
| IPv6 thật? | `curl -k https://[<ipv6>]:8443/v1/info` | `200` |
| Thiếu `Host`? | `printf 'GET /v1/info HTTP/1.1\r\n\r\n' \| openssl s_client -quiet …` | `400 Bad Request` |

### 2. Với một implementation khác, trên LAN thật

```sh
cargo run --manifest-path reference/rust-host/Cargo.toml --example host -- <thư mục>
```

📐 19/09/2026, nhật ký Snappy:

```
NeighborDiscovery  Nghe thấy natuan1 tại 192.168.1.35:51292
NeighborDiscovery  natuan1 khai mình là Mac
NeighborDiscovery  Không hỏi được https://192.168.1.35:51276/v1/info: TaskCanceledException
```

Ba dòng, hai kết cục khác nhau, và cặp ấy mới là bằng chứng: dòng thứ hai là
`rust-host` đang sống; dòng thứ ba là **xác quảng bá** của một tiến trình
`rust-host` đã bị giết trước đó, bản ghi còn trên dây mà không ai mở cửa. Trước
Ticket 05 hai máy ấy trông giống hệt nhau trên bảng.

## Hàng rào đã được ép đỏ

Một hàng rào chưa bao giờ đỏ là một hàng rào chưa được chứng minh
(`lessons-learned.md`, 2026-09-16).

- **"Bộ nhớ không tỉ lệ với thân"** — đổi bộ đệm nhả thân thành
  `new byte[remaining]`: 📐 cấp phát nhảy từ **161 KB → 1017 KB** và test đỏ với
  đúng câu giải thích. Phép đo này so **hai mốc cách nhau 16 lần** chứ không so
  với một trần tuyệt đối: một trần tuyệt đối đi qua được cả khi phép đo chưa hề
  xảy ra. Và nó có **chặn dưới** — khẳng định client nhận được `404`, tức thân
  đã thật sự được nhả.
- **Timeout idle** có hai lượt: một lượt để nó nổ (kết nối im lặng bị đóng) và
  một lượt để nó **không** nổ (kết nối bận suốt 2,5 lần hạn mà không rơi). Thiếu
  lượt thứ hai thì một bản cài đặt *"luôn đóng"* cũng xanh — bài học 170.
- **Hợp đồng capability** có hàng rào của riêng nó: một test đối chiếu số dòng
  hợp đồng với số thuộc tính của kiểu `Capabilities` bằng reflection. Không có
  nó thì hợp đồng là một danh sách viết tay, và danh sách viết tay mù đúng ở chỗ
  vừa được thêm vào (bài học 158).

## Còn treo

- **iPhone thật** — quy trình ở `peekvn/interoperability/manual-ios.md`. Bằng
  chứng hiện có nói Snappy đúng với một stack độc lập (Rust), **không** nói nó
  đúng với CFNetwork của Apple.
- **Windows 10** — xem
  [ADR-0009](../../adr/0009-tls-1-3-ghim-cung-thu-hep-san-he-dieu-hanh-thuc-te.md):
  SChannel của Windows 10 không có TLS 1.3 theo tài liệu Microsoft, và chưa có
  máy để đo. Đây là giả thuyết mạnh, không phải bằng chứng.
