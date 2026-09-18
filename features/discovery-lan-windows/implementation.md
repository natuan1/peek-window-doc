# Discovery LAN trên Windows — cài đặt

## Các mảnh và chỗ ở của chúng

| Tệp (`peekvn/apps/windows/`) | Việc |
|---|---|
| `src/Snappy.Interop/Win32/DnsSd.cs` | P/Invoke `dnsapi.dll` + đọc chuỗi `DNS_RECORD` ra kiểu quản lý |
| `src/Snappy.Interop/Win32/DnsSdSession.cs` | Một lượt đăng ký và một lượt duyệt, kèm vòng đời bộ nhớ native |
| `src/Snappy.Protocol/Discovery/UltpDiscovery.cs` | Hằng số SPEC §8: service type, `pv`, cổng, ngưỡng offline |
| `src/Snappy.Protocol/Discovery/TxtRecord.cs` | Dựng TXT (tập khoá **đóng**) và đọc TXT của bên kia |
| `src/Snappy.Protocol/Discovery/ServiceNaming.cs` | Tên instance + dấu Device, cắt 63 byte theo ranh giới ký tự |
| `src/Snappy.Protocol/Discovery/BrowseAssembler.cs` | Ghép PTR/SRV/TXT/A rời rạc thành endpoint |
| `src/Snappy.Protocol/Discovery/PeerTable.cs` | Bảng lân cận, `LastSeen`, offline, lọc chính mình |
| `src/Snappy.Protocol/Discovery/PeekAdvertiser.cs` · `PeekBrowser.cs` | Dây nối mỏng tới dnsapi |
| `src/Snappy.Protocol/Identity/DeviceIdentity.cs` | P-256 trong kho khoá CNG, `deviceId` theo SPEC §9.1 |
| `src/Snappy.Core/NeighborDiscovery.cs` | Gom danh tính + quảng bá + duyệt + bảng thành một vòng đời |
| `src/Snappy.Core/StatusPanelWindow.cs` | Mục "Thiết bị lân cận" trên bảng trạng thái |

## Ba chỗ dễ làm sai, và vì sao chúng được làm thế này

**1. Bản ghi địa chỉ mang tên *hostname*, không mang tên instance.** Một cách
ghép "theo tên bản ghi" sẽ không bao giờ nối được địa chỉ vào đúng thiết bị, và
mọi endpoint ra đời không có đường đi tới. `BrowseAssembler` vì vậy giữ hai bảng
riêng: instance → (host, port, txt) và host → địa chỉ.

**2. Một mẻ bản ghi là *toàn bộ* sự thật về những host nó nhắc tới.** Tập địa chỉ
mới **thay thế** tập cũ chứ không cộng dồn — cộng dồn là
[#62](https://github.com/natuan1/peekvn/issues/62): peer còn trong danh sách với
một địa chỉ đã chết trong khi giao diện vẫn nói "đang ở cùng mạng".

**3. "Nghe thấy" và "có gì đổi" là hai khái niệm.** Mọi lượt nghe thấy đều làm
mới `LastSeen`; chỉ thay đổi nhìn thấy được mới vẽ lại màn hình và ghi nhật ký.
Gộp hai khái niệm làm một thì một máy đang sống mà không đổi gì bị đánh dấu
offline sau đúng 120 giây (bài học 170 của `peekvn`).

## Danh tính thiết bị

`deviceId = "dev_" + base64url_nopad(SHA256(SPKI DER))` — SPEC §9.1, luôn 47 ký
tự. Khoá riêng P-256 nằm trong **kho khoá CNG** của người dùng
(`Snappy.DeviceIdentity`, Microsoft Software KSP), không export được, và app chỉ
cầm handle chứ không cầm byte khoá.

Hệ quả phải nhớ: gỡ app bằng Velopack xoá `%LocalAppData%\Snappy` nhưng **không**
xoá khoá. Dọn nó là việc của Ticket 18.

## Ràng buộc để lại cho Ticket 05

HĐH quảng bá **mọi** địa chỉ của tên máy, gồm cả IPv6 toàn cục và link-local, và
app không chọn được tập ấy. Server ULTP phải **nghe dual-stack** — bind mỗi
`0.0.0.0` là quảng bá bốn địa chỉ mà chỉ nhận ở một.
