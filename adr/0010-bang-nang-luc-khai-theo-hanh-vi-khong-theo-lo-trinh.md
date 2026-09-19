# ADR-0010: Bảng năng lực khai theo **hành vi hôm nay**, không theo lộ trình

Date: 2026-09-19
Status: Accepted

> ⚠️ Kho `peekvn` có một dãy ADR **độc lập** với dãy này.

## Context

`GET /v1/info` của Snappy trả một khối `capabilities` (SPEC §7.1). SPEC §7 nói
thẳng bên kia dùng đúng khối ấy để quyết định gửi gì:

```swift
if peer.platform == .windows { ... }      // SAI
if peer.capabilities.pullReceiver { ... } // ĐÚNG
```

Tiêu chí nghiệm thu của [Ticket 05 (#136)](https://github.com/natuan1/peekvn/issues/136)
viết: *"`GET /v1/info` trả JSON đúng schema capabilities (`pullSender`,
`pullReceiver`, …, `resumableUpload: ["httpbis-interop-6"]`)"*.

Nhưng Ticket 05 dựng **khung**. Router có đúng một đường: `GET /v1/info`. Không
có `/v1/transfers`, không có `/v1/pairings`. Khai `pushReceiver: true` lúc này
nghĩa là mời iPhone gửi file tới một địa chỉ trả `404`.

Bản Rust đã đi qua đúng chỗ này với `folder`, và để lại một hàng rào:
`reference/rust-host/tests/capability_truth.rs` —

> *"một capability khai sai nặng hơn vẻ ngoài của nó. `folder: true` trong khi
> `create_transfer` từ chối mọi `kind` khác `file` không làm test nào đỏ và
> không làm ai crash — nó chỉ biến một `400` thành thứ Sender **không có cách
> nào đoán trước**, vì nó đã hỏi và đã được trả lời là 'được'."*

## Decision

**Bảng năng lực khai đúng thứ Snappy làm được hôm nay.** Ở Ticket 05, đó là:

```json
"capabilities": {
  "pushSender": false, "pushReceiver": false,
  "pullSender": false, "pullReceiver": false,
  "file": false, "folder": false, "text": false, "url": false,
  "httpRange": false, "resumableUpload": [], "bundledItems": false,
  "sha256": true
}
```

Dòng `true` duy nhất là `sha256`, và nó đúng: SHA-256 chạy được và đã đo **liên
tiến trình** với Rust ở Ticket 03. SPEC §21 buộc implementation chính thức khai
`true`.

**Đây là chỗ cố ý lệch với chữ trong tiêu chí nghiệm thu**, và lệch có lý do:
SPEC §31 cho phép **thêm** capability sau mà không breaking, nên khai đúng sự
thật hôm nay không mất gì; còn khai thừa thì mất một lượt gửi của người dùng
thật. `capabilities.schema.json` cũng nói rõ `resumableUpload` rỗng *"vẫn hợp
lệ, xem fallback ở SPEC §17.4"*.

**Và lời khai được đối chiếu bằng mã, không bằng lời dặn.**
`CapabilityTruthTests` (nửa Windows của `capability_truth.rs`) giữ một hợp đồng
mười hai dòng — đúng bằng số khoá của khối `capabilities` — mỗi dòng nói năng
lực ấy cần gì mới là có thật:

| Điều kiện | Nghĩa | Năng lực |
|---|---|---|
| `TransferRoutes` | cần route dưới `/v1/transfers` | `pushReceiver` · `pullSender` · `httpRange` · `file` · `folder` · `text` · `url` · `resumableUpload` |
| `UltpClient` | cần một client ULTP (chưa có) | `pushSender` · `pullReceiver` |
| `Nothing` | đúng được ngay | `sha256` |
| `Never` | SPEC §7.1 "luôn false" | `bundledItems` |

Hàng rào đỏ theo **cả hai** chiều: lật một cờ mà quên route thì nó chặn; thêm
route mà quên lật cờ thì nó nhắc. Và một test thứ hai đối chiếu **số dòng** của
hợp đồng với số thuộc tính của kiểu `Capabilities` bằng reflection — không có
nó thì hợp đồng là một danh sách viết tay, và một danh sách viết tay mù đúng ở
chỗ vừa được thêm vào (bài học 158 của `peekvn`).

## Consequences

- iPhone/Android đọc `/v1/info` của Snappy hôm nay sẽ kết luận **chưa gửi được
  gì cho máy này** — và đó là sự thật. Tốt hơn hẳn một lượt gửi chạy tới `404`
  hoặc timeout.
- Ticket 07 (#138), 08 (#139), 09 (#140) **phải** lật cờ cùng lúc với route của
  nó; nếu quên, hàng rào nhắc.
- Tiêu chí nghiệm thu #2 của #136 cần được sửa lại ở issue, không chỉ được biện
  minh trong comment mã — nếu không, người làm ticket sau sẽ lật cờ theo chữ
  trong issue. Đã trả lời vào #136.
- Chi phí: hợp đồng mười hai dòng phải được sửa mỗi lần thêm capability. Đó
  đúng là cái giá muốn trả — nó là chỗ duy nhất bắt được một capability mới
  thêm mà chưa ai đối chiếu.

## Related

- [ADR-0003](0003-resumable-upload-104-h1-pass.md) — `resumableUpload:
  ["httpbis-interop-6"]` là đích của Ticket 08, không phải của Ticket 05.
- [Server ULTP trên Windows](../features/server-ultp-windows/overview.md)
- `peekvn/protocol/SPEC.md` §7, §7.1, §21, §31 ·
  `peekvn/protocol/schemas/capabilities.schema.json` ·
  `peekvn/reference/rust-host/tests/capability_truth.rs`
