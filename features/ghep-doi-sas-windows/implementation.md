# Ghép đôi SAS — implementation

Mã ở `apps/windows/src/Snappy.Protocol/`, ba thư mục mới: `Pairing/`, `Auth/`,
`Trust/`, cộng `Client/`.

## Nguồn sự thật là SECURITY.md §3.4, không phải công thức của SPEC §10.1

SPEC §10.1 viết công thức SAS rồi nói thẳng về chính nó:

> Công thức trên là **hình dạng**, không phải đặc tả, và hai implementation lệch
> nhau ở đây sẽ không bao giờ pairing được với nhau.

Hai chỗ một bản "implement theo công thức" sẽ lệch, và **cả hai đều chạy đúng,
đều ra sáu chữ số, đều không có test nào của riêng mình bắt được**:

| Chỗ lệch | Cả hai cách đều hợp lệ về mật mã |
|---|---|
| `transcriptHash` là **salt** của HKDF-Extract, không phải `info` của Expand | ✅ và cho kết quả **khác nhau** |
| Thứ tự sáu trường theo **vai**, mỗi trường có tiền tố độ dài `uint32` big-endian | ✅ nối trần cũng ra một hash |

## Transcript

```
transcript =
      "ULTP-v1 pairing"
    ‖ LP(initiatorIdentityPublicKey)      91 byte, SPKI DER
    ‖ LP(responderIdentityPublicKey)      91 byte
    ‖ LP(initiatorEphemeralPublicKey)     91 byte
    ‖ LP(responderEphemeralPublicKey)     91 byte
    ‖ LP(responderTLSSPKI)                91 byte   ← ràng buộc kênh
    ‖ LP(pairingId)                       26 byte ASCII
```

⚠️ **Năm giá trị đều dài đúng 91 byte**, nên chúng thay thế được cho nhau mà
trình biên dịch không kêu một câu. SECURITY.md §5.6 đo lớp lỗi ấy trên bản Rust:
đổi transcript của initiator sang dùng identity SPKI của responder thì **6/7**
test vẫn xanh — kể cả test mang tên *"initiator dùng SPKI quan sát được"*, vì nó
chỉ khẳng định initiator **đọc** đúng chứ không khẳng định nó **dùng** đúng.

Vì vậy `CryptoVectorTests` khẳng định trên **chuỗi byte** của `ToBytes()`, không
trên object kết quả, và so với vector do **bản Rust** sinh ra
(`interoperability/fixtures/ultp-v1-vectors.json`).

## `responderTLSSPKI` — trường chống MITM

Mỗi bên điền theo góc nhìn của chính mình:

| Vai | Điền gì | Ở đâu trong mã |
|---|---|---|
| initiator | SPKI nó **quan sát được** trong handshake | `UltpChannel.ObservedSpkiDer` |
| responder | SPKI **của chính nó** | `TlsIdentity.SpkiDer` |

Không có tấn công thì hai giá trị bằng nhau. Có MITM thì lệch, SAS lệch, người
dùng dừng lại.

## `SpkiPin` là một kiểu, không phải `byte[]`

SPKI DER (91 byte) và pin của nó (32 byte) đều là `byte[]`. Nhầm chúng là **bài
học 148** của `peekvn`: so một hash với một DER thì không bao giờ khớp, và thứ lộ
ra lỗi không phải một test đỏ mà là *một dòng log*.

```csharp
SpkiPin.FromDer(spki)  // ném nếu nhận 32 byte — đó là độ dài của một pin
SpkiPin.FromRaw(pin)   // ném nếu nhận 91 byte — đó là độ dài của một DER
```

Hai hàm chặn hai chiều của cùng một lỗi. Và `TrustedPeer.NewlyPaired` nhận
**DER** rồi tự băm, nên bất biến **S5** (*"SPKI ghim vào Trust Store MUST là SPKI
đã đi vào transcript"*) đọc được từ chữ ký hàm thay vì từ một câu dặn.

## Không tự confirm

`PairingCoordinator.ConfirmAsync` nhận `confirmed` từ dây rồi **chờ tiếp** một
lời gọi `ConfirmLocally` từ giao diện. Xem [ADR-0011](../../adr/0011-ghep-doi-khong-tu-confirm-responder-cho-nguoi-dung-cuc-bo.md)
cho lý do và cái giá.

Hết hạn tính là **từ chối** — chiều duy nhất an toàn, vì một phiên không ai ngồi
trước máy là đúng hoàn cảnh kẻ tấn công cần.

`ConnectionScope` gắn phiên vào kênh mở nó, và `UltpServer` huỷ chúng ở `finally`
— SECURITY.md §3.4 nói phiên MUST bị huỷ khi TLS đứt, và đường đi qua `finally`
bằng một ngoại lệ đúng là đường câu MUST ấy nói tới. Trước khi có nó, `Cancel`
tồn tại mà **không chỗ gọi nào trong sản phẩm** — chỉ test gọi.

## Transcript auth — mọi trường cố định độ dài, nên không có `LP()`

```
authTranscript(role) =
      "ULTP-v1 auth " ‖ role      role ∈ { "client", "server" }
    ‖ serverNonce                 32 byte
    ‖ clientNonce                 32 byte
    ‖ clientDeviceIdRaw           32 byte
    ‖ serverDeviceIdRaw           32 byte
    ‖ serverTLSSPKIPin            32 byte   ← ràng buộc kênh, và đây là **pin**
    ‖ uint32_be(protocolVersion)   4 byte
```

Chú ý: transcript pairing dùng SPKI **DER**, transcript auth dùng **pin**. Hai
dạng khác nhau của cùng một khái niệm, ở hai chỗ cách nhau vài dòng — và đó đúng
là lý do `SpkiPin` tồn tại như một kiểu.

Chữ ký là ECDSA P-256 dạng raw `r‖s` **64 byte**, không phải DER: độ dài DER thay
đổi theo giá trị (70–72 byte), và đó là kiểu lỗi chạy đúng hàng trăm lần rồi hỏng
ở khoảng 1/128 số chữ ký — triệu chứng là *"thỉnh thoảng auth fail, thử lại thì
được"*.

## Trust Store

`%LocalAppData%\Snappy\trusted-peers.json`, bảy trường của SPEC §10.2, tên trường
theo `protocol/schemas/device-info.schema.json`:

| Trường schema | Hình dạng | Ghi chú |
|---|---|---|
| `identityPublicKey` | 124 ký tự base64 | SPKI DER |
| `tlsSPKIHash` | 44 ký tự base64 | **pin**, không phải DER |

⚠️ Bản Rust ghi `identitySpki`/`tlsSpki` và cái thứ hai lệch cả nội dung — nó ghi
nguyên DER vào một trường tên `*Hash*`. Ghi chú trong `trust.rs` của nó nhận
khoảng lệch ấy và nói *"đóng nó là việc riêng"*. Snappy không đi theo.

Ghi qua tệp tạm rồi thay chỗ. Tệp **hỏng** thì ném chứ không trả store rỗng: nuốt
lỗi ở đó biến "tệp hỏng" thành "không tin ai", im lặng. Nhưng một **BOM** không
phải tệp hỏng — xem bài học 175 của `peekvn`.

## Giới hạn tần suất — SECURITY.md §4.3

| Dòng §4.3 | Snappy | Ghi chú |
|---|---|---|
| `POST /v1/pairings` 5/phút/nguồn | ✅ | endpoint đắt nhất người lạ gọi được |
| Auth thất bại 10/phút/`deviceId` | ✅ | và **quên khi thành công** |
| Phiên pairing chờ cùng lúc | ✅ 8 | trần riêng, không có trong §4.3 |
| Nonce đang sống | ✅ 256 | đầy thì `429`, **không** đuổi nonce người khác |

Dòng thứ hai có một câu MUST dễ bỏ: *"Một lần auth thành công MUST xoá lịch sử
thất bại của `deviceId` đó."* Không có nó, bộ đếm quay ra chĩa vào người dùng —
`deviceId` là thứ **kẻ tấn công chọn được**, nên mười chữ ký rác mang id của một
máy thật là chặn máy ấy, lặp lại mãi.
