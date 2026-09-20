# ADR-0012: Cặp harness đo **hành vi nền tảng** phải chạy cho từng TLS stack, không chỉ cho phía đã đặt câu hỏi

Date: 2026-09-19
Status: Accepted

> ⚠️ Kho `peekvn` có một dãy ADR **độc lập** với dãy này. ADR-0012/0013/0014 nhắc
> tới dưới đây là của `peekvn`.

## Context

`peekvn` ADR-0013 nói mọi verifier TLS MUST verify **chữ ký handshake**, kể cả
trong phiên pairing nơi chain/tên/hạn được phép bỏ qua. Lý do đo được ở
`protocol/SECURITY.md` §5.8: bỏ bước ấy thì kẻ tấn công trình certificate **của
chính nạn nhân**, gửi chữ ký rác, và vẫn hoàn tất handshake — lúc đó SPKI mà
initiator quan sát được *bằng đúng* SPKI thật, nên SAS khớp và toàn bộ ràng buộc
kênh của ADR-0012 vô hiệu.

`peekvn` ADR-0014 trả lời câu hỏi ấy cho Apple: `URLSession` **có** verify, nên
client iOS an toàn. Nhưng mục Hệ quả của nó viết thẳng:

> Khẳng định "nền tảng lo việc đó" MUST NOT được mang sang implementation ULTP
> trên nền tảng khác mà không đo lại. Đây là kết luận về **một** TLS stack, không
> phải về một lớp thư viện.

Cặp `interoperability/tests/tls-handshake.sh` là chỗ phép đo ấy sống. Và nó mở
đầu bằng:

```bash
require_side swift "toàn bộ cặp" || exit 0
```

Hợp lý khi viết ra — cặp ấy sinh ra để trả lời một câu hỏi về `URLSession`. Sai
kể từ khi Ticket 06 dựng một client ULTP thứ hai, trên `SslStream`/SChannel:

- trên máy dev Windows, `swift` không có ⇒ **cả cặp BỎQUA**;
- trong khi phía Windows dựng được, `stolen-cert.rs` chạy được, và câu hỏi *"SChannel
  có verify chữ ký handshake không"* **chưa ai trả lời**.

Bảng kết quả không nói dối: nó ghi rõ `BỎQUA`, và `run.sh` in "bỏ qua = CHƯA ĐO
trên máy này, không phải đã đạt". Nó chỉ không nói rằng có một phép đo *khác*, đo
được ngay hôm nay, mà script vừa từ chối chạy.

## Decision

**Một cặp đo hành vi của nền tảng phải lặp trên mọi phía dựng được, không dừng ở
phía đầu tiên vắng mặt.**

`require_side` chuyển từ **cổng đầu file** thành **một dòng khai báo cho từng
phía**, và thân cặp chạy trong một vòng lặp:

```bash
SIDES=()
[ -n "${SWIFT_INTEROP:-}" ] && SIDES+=("swift")
[ -n "${WINDOWS_INTEROP:-}" ] && SIDES+=("windows")

require_side swift   "toàn bộ nhánh Swift (ADR-0013 trên URLSession)" || true
require_side windows "toàn bộ nhánh Windows (ADR-0013 trên SChannel)" || true

for side in "${SIDES[@]}"; do … done
```

Mỗi dòng kết quả mang tiền tố `[swift]` hoặc `[windows]`, nên một bảng có bốn
PASS và một BỎQUA đọc được là "đo xong một stack, còn một stack chưa đo".

Kèm theo: `apps/windows/tools/Snappy.Harness` nhận lệnh
`probe-tls --host <addr:port> [--pin <hex>]`, đi qua **đúng đường sản phẩm**
(`UltpChannel.ConnectAsync`) chứ không qua một `SslStream` dựng riêng cho test —
một đầu dò có cấu hình khác client thật thì nó đo một thứ khác client thật.

## Consequences

**Câu hỏi của ADR-0013 nay có số đo cho Windows.** 📐 19/09/2026,
`SslStream`/SChannel, đối thủ là `reference/rust-host/examples/stolen-cert.rs`:

| Nhánh | Certificate | Khoá ký | Kết quả |
|---|---|---|---|
| `--honest` | của nạn nhân | của nạn nhân | handshake **hoàn tất** (đối chứng) |
| `--honest` + pin đúng | — | — | **hoàn tất** (đối chứng) |
| `--honest` + pin lệch 1 ký tự | — | — | `reason=deviceIdentityChanged` (S14) |
| `--stolen` | của nạn nhân | của kẻ tấn công | **hỏng** — và server cũng ghi `HANDSHAKE rejected` |

SChannel verify `CertificateVerify` dù `RemoteCertificateValidationCallback` trả
`true`; callback ấy chỉ quyết định chain/tên/hạn. Kết luận này ràng buộc **đúng
một stack**, y như ADR-0014 — nếu Snappy đổi sang một TLS stack khác, cặp này đỏ
hoặc phải đo lại.

**Cặp `tls-handshake` trở thành canary cho cả hai nền tảng.** Apple đổi hành vi,
hoặc Microsoft đổi, hoặc ai đó thay `SslStream` bằng thứ khác — nó đỏ ngay.

**Một lỗi tìm được nhờ chính nhóm đối chứng.** Bản đầu của đầu dò gộp "bắt tay
xong" với "đọc được `/v1/info`" vào một verdict, và nhánh `--honest` in
`PROBE rejected reason=JsonException` — đỏ trên một handshake đã thành công, chỉ
vì `stolen-cert.rs` trả `capabilities: {}` còn kiểu `Capabilities` của Snappy
khai `required` cho cả mười hai khoá. Nếu cùng lỗi ấy rơi vào nhánh `--stolen`,
cặp sẽ báo **PASS** — "cert ăn cắp bị từ chối" — trong khi thứ bị từ chối là một
dấu ngoặc nhọn. Verdict nay được chốt ngay tại bước phép đo nói về (kênh mở
được ⇒ `completed`); mọi thứ sau đó là thông tin thêm và không đổi màu.

**Luật rút ra, áp cho mọi cặp sau này:** `require_side <x> "toàn bộ cặp" || exit 0`
ở *đầu* một script là một khẳng định rằng cặp ấy chỉ có nghĩa với phía `x`. Với
một cặp đo hành vi nền tảng, khẳng định đó gần như luôn sai.

## Alternatives considered

**Viết một cặp riêng `tls-handshake-windows.sh`.** Bị loại: hai bản chép của cùng
một kịch bản sẽ lệch nhau ở lần sửa thứ nhất, và nhóm đối chứng — phần dễ quên
nhất — sẽ chỉ được sửa ở một bản.

**Suy luận rằng SChannel an toàn vì `CertificateVerify` nằm trong TLS stack.**
Đúng, và không đủ: `protocol/SPEC.md` §0.2 tách rõ *suy luận* khỏi *số đo*, và
ADR-0014 tồn tại chính vì cùng một suy luận ấy — đúng — vẫn cần được đo.
