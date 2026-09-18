# ADR-0008: Discovery trên Windows đi qua responder in-box của HĐH, và hostname phải là tên máy thật

Date: 2026-09-18
Status: Accepted

> ⚠️ Kho `peekvn` có một dãy ADR **độc lập** với dãy này. `ADR-0008` ở đây không
> phải `ADR-0008` của `peekvn`.

## Context

[Ticket 04 (#135)](https://github.com/natuan1/peekvn/issues/135) mở mặt phẳng
discovery cho Snappy: quảng bá `_peek._tcp` và nghe LAN, để người dùng mở app
trên điện thoại cùng mạng là hai đầu thấy nhau, không cấu hình gì.

Hai đường đi, và chúng khác nhau tới mức không thể chọn bằng thẩm mỹ:

1. **Responder in-box của Windows** — `dnsapi.dll`
   (`DnsServiceRegister`/`DnsServiceBrowse`), dịch vụ `dnscache` làm việc thật.
2. **Tự dựng responder** trên socket UDP 5353, đúng cách `reference/rust-host`
   làm với crate `mdns-sd`.

`protocol/SPEC.md` §39 ghi chú Phase 2 rằng *"Windows không có mDNS responder sẵn
như macOS, nên cần một responder thuần Rust"* — câu ấy đẩy mặc định về hướng (2).
Nó đúng cho bản Rust và **không còn đúng** như một câu về nền tảng:
`DnsServiceRegister` có từ Windows 10 1703, dưới cả sàn 1809 của Snappy.

Hai ràng buộc riêng của Snappy làm cán cân lệch hẳn:

- **Bộ cài là PerUser, người dùng không có quyền admin** ([ADR-0006](0006-dong-goi-velopack-cai-peruser.md)).
  Một socket 5353 thuộc về `Snappy.exe` sẽ kéo theo hộp thoại Windows Firewall ở
  lần chạy đầu, và tạo rule inbound thì cần admin. App bị chặn inbound sẽ im lặng
  không thấy ai — đúng loại hỏng không ai chẩn đoán được.
- **Bản ghi A phải theo kịp khi máy đổi IP.** Bản Rust đã trả giá cho chuyện này
  ([#62](https://github.com/natuan1/peekvn/issues/62)): nó phải tự liệt kê địa chỉ
  2 giây một lần và dựng lại daemon khi tập địa chỉ đổi.

## Decision

**1. Dùng responder in-box qua `dnsapi.dll`.** Không thư viện mới, không socket
của riêng app, không hộp thoại firewall. Bề mặt P/Invoke nằm gọn trong
`Snappy.Interop/Win32/DnsSd*.cs` theo [ADR-0004](0004-cau-truc-app-windows-bon-project.md).

**2. Hostname trong bản ghi SRV phải là tên mDNS thật của máy
(`<tên máy>.local`), không phải một tên do app đặt.**

📐 Đây là kết quả đo ngày 2026-09-18, không phải lựa chọn thẩm mỹ. Quan sát viên
là một implementation **khác hẳn** — `reference/rust-host --example mdns-browse`,
responder thuần Rust nghe thẳng multicast — chạy cùng lúc với phép đối chứng
`_airplay._tcp` (một thiết bị Apple thật trên LAN) để biết cái thước còn tốt:

| Hostname trong SRV | Truyền IPv4 tường minh? | Bên kia thấy |
|---|---|---|
| `peek-<dấu Device>.local` (app tự đặt, như bản Rust) | không | PTR ra dây, **không resolve nổi** |
| `peek-<dấu Device>.local` | **có** | PTR ra dây, **vẫn không resolve nổi** |
| `<tên máy>.local` | không | **đủ**: SRV, TXT `pv`/`id`/`port`, IPv4 + IPv6 |

Responder của Windows chỉ giữ bản ghi A cho tên máy của chính nó; tham số
`pIp4` của `DnsServiceConstructInstance` được **nhận** mà không quảng bá.

**3. Dấu Device chuyển hết sang instance name.** Vì hostname không còn mang dấu
Device, chỗ duy nhất phân biệt hai máy trùng tên là tên instance
(`<tên máy> [<deviceId bỏ dev_>]`) — đúng hình dạng Rust sinh ra và iOS cắt ra.

**4. Peer rời bảng bằng timeout 120 giây, không bằng bản ghi goodbye.** SPEC §8
không quy định con số này; 120 giây là TTL mặc định của bản ghi DNS-SD
(RFC 6762 §10) và trùng con số Android đang dùng. Đường goodbye **không** được
cài vì chưa ai đo được dnsapi có chuyển tín hiệu ấy vào callback duyệt hay không.

**5. Device Identity (P-256, SPEC §9.1) nằm trong kho khoá CNG của người dùng**
(Microsoft Software Key Storage Provider), không phải một tệp `.key` cạnh nhật
ký. Đó là chỗ tương đương Keychain mà SPEC đòi: khoá được DPAPI bảo vệ theo hồ sơ
người dùng, không export được, và mã của app không bao giờ cầm byte khoá riêng.

## Consequences

**Được:**

- Không hộp thoại firewall, không quyền admin, không thư viện thứ hai (KPI bộ cài
  giữ nguyên 10,18 MB).
- Bản ghi A do HĐH giữ ⇒ đổi IP là HĐH tự cập nhật. Snappy **không** cần vòng canh
  địa chỉ mà bản Rust phải có.

**Mất, và phải nhớ:**

- **Tập địa chỉ quảng bá do HĐH chọn, gồm cả IPv6.** Ta không lọc được. Vì vậy
  server ULTP của [Ticket 05 (#136)](https://github.com/natuan1/peekvn/issues/136)
  **bắt buộc nghe dual-stack**: bind mỗi `0.0.0.0` là quảng bá bốn địa chỉ mà chỉ
  nhận ở một, tức tự dựng lại #62 bằng tay.
- **Hai máy Windows trùng tên** trỏ về cùng một hostname; phân biệt chỉ còn ở
  instance name. Conflict resolution là việc của responder HĐH.
- Không kiểm soát được nhịp truy vấn và TTL — chúng là chính sách của dnscache.
- Một máy tắt hẳn còn nằm trong bảng tới 120 giây (hệ quả của quyết định 4).

**Nghiệm thu:** hai chiều đã đo với `reference/rust-host` làm implementation thứ
hai. Chiều iPhone thật **chưa đo** — quy trình ở
`peekvn/interoperability/manual-ios.md` §"iPhone ↔ Snappy (Windows)".

## Related

- [ADR-0001](0001-brand-khac-service-mdns.md) — service `_peek._tcp` bất biến vĩnh viễn
- [ADR-0004](0004-cau-truc-app-windows-bon-project.md) — mọi P/Invoke nằm trong `Snappy.Interop`
- [ADR-0006](0006-dong-goi-velopack-cai-peruser.md) — cài PerUser, không admin
- `peekvn/docs/bai-hoc.md` **169** (bản ghi A của Windows) và **170** (nghe thấy ≠ đổi)
- Ba vòng đo giữ ở nhánh `prototype/mdns-dnsapi` của `peekvn`
