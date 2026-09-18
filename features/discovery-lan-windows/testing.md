# Discovery LAN trên Windows — kiểm thử

## Nguyên tắc

Đo một responder mDNS **bằng chính stack của nó là vô nghĩa**:
`DnsServiceBrowse` thấy đăng ký của chính máy nó qua cache nội bộ, kể cả khi
không byte nào ra dây. Nên mọi khẳng định "ra dây được" ở đây đều đi qua một
implementation khác.

## Bốn tầng, mỗi tầng trả lời một câu khác nhau

| Tầng | Câu hỏi | Dụng cụ |
|---|---|---|
| Đơn vị | TXT có đúng ba khoá không? tên instance có quá 63 byte không? ghép bản ghi có đúng không? bảng có đánh dấu offline đúng lúc không? | `dotnet test apps/windows/Snappy.slnx` — `DiscoveryNamingTests`, `BrowseAssemblerTests`, `PeerTableTests` |
| Liên ngôn ngữ | `deviceId` có khớp cách Rust tính không? | `DeviceIdentityTests` đọc thẳng `interoperability/fixtures/ultp-v1-vectors.json` — so **với bản Rust**, không so với hằng số chép tay |
| Liên tiến trình | Bản ghi có thật sự ra dây và resolve được từ stack khác không? | `cargo run --manifest-path reference/rust-host/Cargo.toml --example mdns-browse` |
| Người dùng thật | iPhone có thấy Snappy, và Snappy có thấy iPhone không? | `peekvn/interoperability/manual-ios.md` §"iPhone ↔ Snappy (Windows)" |

## Phép đối chứng là bắt buộc

Mọi lượt đo mạng phải chạy kèm một lượt `_airplay._tcp` (thiết bị Apple thật trên
LAN). Không có nó thì một kết quả rỗng đọc thành *"Snappy hỏng"* trong khi thủ
phạm có thể là cái thước — chuyện đã xảy ra thật hai lần trong lúc làm ticket
này.

Cùng loại: phép đo timeout cần **hai** lượt — một lượt để nó nổ (peer tắt thật),
một lượt để nó **không** nổ (peer sống suốt quá 120 giây). Thiếu lượt thứ hai thì
một cài đặt "luôn báo offline" cũng xanh.

## Đã đo (2026-09-18, máy dev Windows 11)

- Snappy → dây: `mdns-browse` resolve đủ SRV `natuan1.local:8443`, TXT
  `pv=1`/`id=dev_hHDZ…`/`port=8443`, IPv4 + 4 địa chỉ IPv6.
- Dây → Snappy: nhật ký `Nghe thấy natuan1 tại 192.168.1.35:58545`, và bảng trạng
  thái hiện đúng dòng ấy (đã chụp màn hình).
- Peer tắt → 120 giây sau có dòng `Không còn nghe thấy natuan1`; peer sống 144
  giây thì **không** có dòng ấy.
- Snappy thoát → biến khỏi kết quả duyệt bên kia trong vài giây (rút bản ghi).
- AOT: `publish.cmd` + `ci/smoke-test.ps1` — exe 8,04 MB, working set 17,84 MB.

## Chưa đo

**iPhone thật.** Cần thiết bị trong tay; sáu bước đã viết sẵn. Bằng chứng hiện có
nói Snappy đúng với **một** stack độc lập (Rust), không nói nó đúng với Bonjour
của Apple.
