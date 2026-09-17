> ⚠️ **ĐÓNG BĂNG — KHÔNG SỬA FILE NÀY.**
> Tài liệu này là bản gốc mang tính lịch sử. Source of truth hiện tại là `protocol/SPEC.md`,
> và mọi chỗ lệch khỏi bản gốc được ghi lại trong `docs/adr/`.
> Đọc file này để hiểu ý định ban đầu, không dùng nó để implement.

# Technical Requirements Specification
## Universal Local File Transfer Protocol & iOS Application v1

**Trạng thái:** Implementation Ready  
**Ưu tiên triển khai:** iOS trước  
**Protocol:** Cross-platform ngay từ v1  
**Target platforms cuối cùng:** iOS, Android, Windows, macOS  
**Minimum iOS:** iOS 17+  
**Ngày đặc tả:** 21/08/2026

---

# 1. Mục tiêu tài liệu

Tài liệu này là **source of truth** cho việc thiết kế và triển khai hệ thống truyền dữ liệu local giữa các thiết bị.

AI coding agent, developer và reviewer phải tuân thủ tài liệu này.

Nếu implementation và tài liệu mâu thuẫn:

> **Tài liệu này được ưu tiên, trừ khi có ADR mới thay đổi quyết định.**

Mục tiêu giai đoạn đầu:

```text
             iOS
          ↙   ↓   ↘
     Windows Android macOS
```

iOS phải:

- gửi file tới Windows;
- gửi file tới Android;
- gửi file tới macOS;
- nhận file từ Windows;
- nhận file từ Android;
- nhận file từ macOS.

Giai đoạn sau:

```text
iOS ↔ Android
iOS ↔ Windows
iOS ↔ macOS

Android ↔ Windows
Android ↔ macOS
Windows ↔ macOS

Android ↔ Android
Windows ↔ Windows
macOS ↔ macOS
iOS ↔ iOS
```

Protocol phải hỗ trợ mô hình này **ngay từ thiết kế v1**, kể cả khi implementation ban đầu chưa hoàn thành toàn bộ matrix.

---

# 2. Bối cảnh nghiệp vụ

Ứng dụng mobile là endpoint của một hệ sinh thái utility desktop/mobile.

Mục tiêu quan trọng nhất:

> Người dùng có thể truyền file giữa các thiết bị trong cùng mạng local nhanh, riêng tư và gần như không cần cấu hình.

Các quyết định sản phẩm đã có:

- mobile miễn phí;
- không Pro;
- không quảng cáo;
- không account;
- không cloud trong v1;
- file transfer miễn phí và không áp giới hạn dung lượng/số file ở tầng sản phẩm;
- mobile không chứa các utility module của desktop;
- Share Sheet là một entry point quan trọng;
- trusted device phải cho phép nhận file thuận tiện hơn sau lần pairing đầu tiên.

Lõi truyền file phải được thiết kế độc lập với UI và có khả năng được implement trên Windows, Android, iOS và macOS.

---

# 3. Product principles

## 3.1 Local-first

Transfer v1:

```text
Device A
   │
   │ LAN
   ▼
Device B
```

Không:

```text
Device A
   │
   ▼
Cloud
   │
   ▼
Device B
```

Không có:

- file relay server;
- cloud storage;
- login server;
- account;
- OAuth;
- cloud pairing;
- cloud clipboard.

---

## 3.2 No configuration

Happy path không được yêu cầu người dùng nhập:

- IP;
- hostname;
- port;
- token;
- network interface.

Thiết bị phải tự discovery.

Manual IP/QR chỉ là fallback.

---

## 3.3 Protocol first

Không thiết kế:

> iOS protocol

hoặc:

> Windows protocol.

Phải thiết kế:

> **Universal Local Transfer Protocol — ULTP**

Tên `ULTP` chỉ là working name và có thể thay khi chốt tên sản phẩm.

---

# 4. RFC terminology

Trong tài liệu:

**MUST** — bắt buộc.

**MUST NOT** — tuyệt đối không được.

**SHOULD** — nên thực hiện, chỉ bỏ khi có lý do kỹ thuật rõ ràng.

**SHOULD NOT** — không nên thực hiện.

**MAY** — optional.

---

# 5. Phạm vi v1

## P0 — bắt buộc

- Device discovery.
- Device identity.
- Pairing.
- Trusted device.
- TLS encryption.
- Capability negotiation.
- File transfer.
- Multiple files.
- Folder transfer.
- Upload.
- Download.
- PUSH transfer.
- PULL transfer.
- Progress.
- Cancel.
- Retry.
- Resume.
- Background iOS transfer.
- SHA-256 integrity.
- Transfer history.
- Share Extension.
- Notifications.
- UTF-8 filename.
- Cross-platform safe path.
- Network interruption recovery.

## P1 — làm sau P0 nhưng vẫn thuộc mobile 1.x

- Text transfer.
- URL transfer.
- Partial file acceptance.
- Transfer preview.
- Quick Send.
- QR pairing fallback.
- iOS Control Center control.
- Android Quick Settings.
- Adaptive parallel transfer.

## Không thuộc v1

- Internet transfer.
- TURN/STUN.
- WebRTC.
- Bluetooth file transport.
- Wi-Fi Direct transport.
- Cloud relay.
- Account.
- Login.
- Cloud sync.
- Clipboard background sync.
- File compression protocol.
- Delta transfer.
- Deduplication.
- Merkle tree.
- End-to-end encrypted cloud storage.
- Browser-based transfer.

---

# 6. Platform strategy

## 6.1 Production implementation đầu tiên

```text
iOS
Swift
SwiftUI
```

## 6.2 Reference implementations

Để test iOS phải đồng thời có:

```text
Windows Reference Host
macOS Reference Host
Android Reference Host
```

Các reference host chưa cần UI production.

Mục tiêu của chúng:

> chứng minh rằng protocol không phụ thuộc iOS.

---

# 7. Công nghệ iOS

MUST sử dụng native Apple stack:

```text
Swift
SwiftUI
Foundation
URLSession
Network.framework
CryptoKit
Security.framework
UniformTypeIdentifiers
UserNotifications
Photos
App Groups
Share Extension
OSLog
```

Không sử dụng Flutter.

Không sử dụng React Native.

Không sử dụng .NET MAUI.

Không đưa Rust networking vào iOS v1.

### Lý do

Khó khăn lớn nhất của ứng dụng nằm ở:

- Share Extension;
- URLSession background;
- App Group;
- Local Network permission;
- Keychain;
- Bonjour;
- iOS lifecycle.

Native APIs phải được kiểm soát trực tiếp.

---

# 8. Minimum iOS

MUST support:

```text
iOS 17+
```

Một lý do quan trọng là URLSession từ iOS 17 hỗ trợ resumable HTTP uploads nếu server hỗ trợ protocol tương ứng.

Không lấy iOS 26 làm minimum chỉ để sử dụng API mới hơn.

---

# 9. Core architectural principle

Hệ thống phải phân biệt:

```text
DISCOVERY PLANE
CONTROL PLANE
DATA PLANE
```

Kiến trúc:

```text
┌────────────────────────────────────┐
│ DISCOVERY PLANE                    │
│                                    │
│ Bonjour / DNS-SD / mDNS            │
│ QR fallback                        │
└────────────────┬───────────────────┘
                 │
                 ▼
┌────────────────────────────────────┐
│ CONTROL PLANE                      │
│                                    │
│ HTTPS REST                         │
│ WebSocket                          │
│ Pairing                            │
│ Authentication                     │
│ Transfer negotiation               │
└────────────────┬───────────────────┘
                 │
                 ▼
┌────────────────────────────────────┐
│ DATA PLANE                         │
│                                    │
│ HTTPS                              │
│ Background URLSession              │
│ HTTP Range                         │
│ Resumable HTTP Upload              │
└────────────────────────────────────┘
```

Không truyền file lớn qua WebSocket.

---

# 10. Universal PUSH/PULL model

Protocol MUST hỗ trợ hai mode.

## 10.1 PUSH

Receiver chạy HTTP server.

```text
Sender                         Receiver

HTTP Client                    HTTP Server

     ─────── upload ─────────►
```

Ví dụ:

```text
iPhone → Windows
iPhone → Android
iPhone → macOS
```

---

## 10.2 PULL

Sender chạy HTTP server.

```text
Sender                         Receiver

HTTP Server                    HTTP Client

     ◄──────── GET ───────────
```

Ví dụ:

```text
Windows → iPhone
Android → iPhone
macOS → iPhone
```

---

# 11. iOS role v1

iOS v1 MUST ưu tiên vai trò:

```text
HTTP Client
```

Khả năng:

```text
pushSender       = true
pullReceiver     = true

backgroundUpload   = true
backgroundDownload = true
```

iOS v1 không được phụ thuộc vào persistent HTTP server để hoạt động.

Có thể bổ sung foreground-only control listener trong tương lai.

---

# 12. Desktop/Android role

Windows, macOS và Android official implementations SHOULD hỗ trợ:

```text
pushSender
pushReceiver

pullSender
pullReceiver

httpClient
httpServer
```

Nhờ vậy:

```text
Windows ↔ Android
Windows ↔ macOS
Android ↔ macOS
```

có thể negotiate mode tối ưu.

---

# 13. Capability-driven architecture

Code MUST NOT làm:

```swift
if peer.platform == .windows {
    ...
}
```

cho quyết định transport.

Phải làm:

```swift
if peer.capabilities.pullReceiver {
    ...
}
```

hoặc:

```swift
if peer.capabilities.pushReceiver {
    ...
}
```

Platform chỉ phục vụ:

- UI;
- analytics;
- compatibility diagnostics.

Không dùng platform làm transport decision.

---

# 14. Capability object

Ví dụ:

```json
{
  "protocol": {
    "min": 1,
    "max": 1
  },
  "capabilities": {
    "pushSender": true,
    "pushReceiver": true,
    "pullSender": true,
    "pullReceiver": true,

    "file": true,
    "folder": true,
    "text": true,
    "url": true,

    "httpRange": true,
    "resumableUpload": true,

    "sha256": true
  }
}
```

Capabilities MUST có khả năng mở rộng mà không phá protocol cũ.

Unknown capability MUST được ignore.

---

# 15. Discovery

Primary discovery:

```text
DNS-SD + mDNS / Bonjour
```

Không dùng UDP multicast protocol tự chế làm primary mechanism trên iOS.

Apple yêu cầu Local Network permission cho Bonjour register/browse/resolve. App phải khai báo `NSLocalNetworkUsageDescription` và danh sách Bonjour service trong `NSBonjourServices`.

Working service type:

```text
_ultp._tcp.local
```

Tên này MUST được thay theo tên sản phẩm trước release production.

---

# 16. Bonjour metadata

TXT record phải cực nhỏ.

Ví dụ:

```text
pv=1
id=7cd14...
port=43120
```

MAY có:

```text
caps=ab817...
```

là hash capability state.

MUST NOT broadcast:

- user email;
- username;
- filenames;
- auth token;
- secret;
- private key;
- transfer history.

Discovery chỉ dùng để biết:

> Có một endpoint khả dĩ ở đây.

---

# 17. Discovery không phải authentication

Đây là security invariant.

```text
Bonjour result
      ≠
Trusted Device
```

Không được trust dựa trên:

- IP;
- device name;
- hostname;
- mDNS record;
- MAC address;
- random fingerprint.

LocalSend từng có critical MITM advisory do discovery không được authenticate đầy đủ, cho phép attacker trong cùng LAN impersonate thiết bị.

Protocol của chúng ta MUST coi discovery là **untrusted hint**.

---

# 18. Device identity

Mỗi installation MUST tạo một permanent identity key pair.

Algorithm:

```text
P-256
ECDSA
```

Private key MUST được lưu:

iOS:

```text
Keychain
```

Có thể sử dụng Secure Enclave khi phù hợp.

Public key được phép export.

Logical Device ID:

```text
deviceId =
base64url(
    SHA256(identityPublicKey)
)
```

Device ID:

- không phụ thuộc IP;
- không phụ thuộc hostname;
- không thay khi đổi Wi-Fi.

---

# 19. Device information

Endpoint:

```http
GET /v1/info
```

Example:

```json
{
  "deviceId": "dev_yE4...",
  "displayName": "Tuan's MacBook",
  "platform": "macos",

  "protocol": {
    "min": 1,
    "max": 1
  },

  "publicKey": "...",

  "capabilities": {
    "pushReceiver": true,
    "pullSender": true,
    "httpRange": true,
    "resumableUpload": true
  }
}
```

Không chứa secret.

---

# 20. Pairing requirement

Transfer giữa hai device MUST có authenticated relationship.

Flow lần đầu:

```text
Discover
   ↓
Connect
   ↓
Pair Request
   ↓
Ephemeral Key Exchange
   ↓
SAS
   ↓
User verifies
   ↓
Trusted Device
```

---

# 21. SAS pairing

Hai thiết bị sinh ephemeral P-256 ECDH keys.

```text
A privateEphemeral
A publicEphemeral

B privateEphemeral
B publicEphemeral
```

Shared secret:

```text
ECDH(
    privateA,
    publicB
)
```

Key derivation:

```text
pairKey =
HKDF-SHA256(
    sharedSecret,
    transcriptHash
)
```

SAS:

```text
HMAC-SHA256(
    pairKey,
    "ULTP-PAIRING-SAS"
)
```

convert thành 6 chữ số:

```text
482 913
```

Hai thiết bị hiển thị cùng giá trị.

Người dùng xác nhận:

```text
Numbers match
[Pair]
```

Nếu khác:

```text
[Cancel]
```

---

# 22. Trusted device storage

Một trusted peer phải lưu tối thiểu:

```text
deviceId
displayName
identityPublicKey
tlsSPKIHash
pairedAt
lastSeenAt
autoAccept
```

Secret MUST nằm trong secure storage khi cần.

---

# 23. TLS

Data MUST NOT được truyền plaintext.

Official implementations MUST sử dụng:

```text
HTTPS
TLS 1.3
```

Mỗi device có khả năng chạy server MUST có local TLS identity.

Certificate có thể self-signed.

Sau pairing:

```text
SPKI / public key
```

phải được pin.

Certificate name/IP validation thông thường không đủ cho local dynamic IP environment.

---

# 24. Pairing và TLS bootstrap

Lần kết nối đầu:

```text
TLS certificate chưa trusted
```

Client MAY tạm chấp nhận certificate trong **pairing session duy nhất** để lấy SPKI/public identity.

Không được đánh dấu trusted trước SAS verification.

Sau SAS confirmation:

```text
SPKI fingerprint
+
device public identity key
```

được pin.

Các connection sau:

```text
presented SPKI
MUST ==
trusted SPKI
```

Nếu khác:

```text
DEVICE_IDENTITY_CHANGED
```

Không silent accept.

---

# 25. Application authentication

Sau pairing, connection MUST thực hiện challenge-response.

Concept:

```text
Server:
serverNonce

Client:
clientNonce
signature(
    serverNonce
    + clientNonce
    + deviceId
    + protocolVersion
)
```

Server verify signature bằng trusted peer public key.

Server cũng phải chứng minh identity tương tự.

Sau mutual authentication server MAY cấp:

```text
short-lived session token
```

Ví dụ TTL:

```text
30 minutes
```

Token chỉ tồn tại trong memory hoặc secure transient storage.

---

# 26. Authorization

Mỗi transfer MUST có:

```text
transferId
```

Mỗi item MUST có:

```text
itemId
```

Mỗi file endpoint SHOULD sử dụng scoped authorization.

Không được dựa duy nhất vào:

```text
GET /file?id=123
```

Per-file access token MUST:

- random;
- unguessable;
- scoped vào transfer/file;
- expire;
- invalidated khi transfer kết thúc hoặc cancel.

---

# 27. Control API

Base path:

```text
/v1
```

Minimum endpoints:

```text
GET    /v1/info

POST   /v1/pairings
POST   /v1/pairings/{pairingId}/confirm
DELETE /v1/pairings/{deviceId}

POST   /v1/auth/challenge
POST   /v1/auth/verify

POST   /v1/transfers
GET    /v1/transfers/{transferId}

POST   /v1/transfers/{transferId}/accept
POST   /v1/transfers/{transferId}/reject
POST   /v1/transfers/{transferId}/cancel

GET    /v1/transfers/{transferId}/items/{itemId}

GET    /v1/events
```

`/v1/events` là WebSocket endpoint khi platform hỗ trợ.

---

# 28. WebSocket

WebSocket chỉ dùng:

```text
peer online/offline
transfer offer
transfer accepted
transfer rejected
transfer cancelled
transfer state change
```

MUST NOT dùng WebSocket gửi binary file lớn.

Khi iOS app foreground, iOS SHOULD duy trì authenticated WebSocket tới các trusted peers đang online và cần giao tiếp.

Đây là cơ chế để:

```text
Windows → iPhone
```

gửi transfer offer trong khi iPhone vẫn đóng vai client.

---

# 29. iOS presence model

iOS v1 không phụ thuộc vào inbound server.

Khi app mở:

```text
iOS
 ↓
Bonjour browse
 ↓
find trusted peers
 ↓
HTTPS authenticate
 ↓
WebSocket connect
```

Peer biết:

```text
iPhone = ONLINE
```

Khi iPhone không có active control connection:

```text
iPhone = OFFLINE / NOT READY
```

Desktop UX phải hiển thị:

> Open the app on your iPhone to receive.

Không được giả vờ rằng iPhone luôn có thể bị đánh thức bằng LAN.

---

# 30. Background limitation của iOS

Background `URLSession` có thể tiếp tục HTTP/HTTPS transfer khi app bị suspend hoặc bị hệ thống terminate. Apple thực hiện transfer bằng process riêng.

Nhưng nếu người dùng manually force-quit app:

> background URLSession transfers bị cancel và iOS không tự relaunch app cho đến khi người dùng mở app lại.

Đây phải được xem là platform constraint, không phải bug.

---

# 31. Transfer object

Schema logic:

```json
{
  "transferId": "01J...",
  "protocolVersion": 1,

  "senderDeviceId": "dev_...",
  "receiverDeviceId": "dev_...",

  "createdAt": "2026-08-21T12:00:00Z",

  "items": []
}
```

---

# 32. Transfer item

```json
{
  "itemId": "01J...",

  "kind": "file",

  "name": "video.mov",

  "relativePath": "Holiday/2026/video.mov",

  "size": 8753482937,

  "mimeType": "video/quicktime",

  "sha256": null
}
```

`sha256` MAY chưa có khi offer được tạo.

Không bắt sender hash 100 GB trước khi transfer được bắt đầu.

---

# 33. Payload kinds

Protocol:

```text
file
directory
text
url
```

P0:

```text
file
directory
```

P1:

```text
text
url
```

---

# 34. Folder representation

Không zip folder mặc định.

Ví dụ:

```text
Project/
├── README.md
├── Images/
│   └── a.png
└── Videos/
    └── demo.mov
```

Manifest chứa:

```text
Project/README.md
Project/Images/a.png
Project/Videos/demo.mov
```

Receiver reconstruct tree.

---

# 35. Path rules

Protocol path separator:

```text
/
```

Không phụ thuộc OS.

MUST reject:

```text
../
..\

absolute paths

C:\...

/Users/...

\\server\share

file://...
```

Sau normalize:

```text
destinationRoot + relativePath
```

resolved path MUST nằm bên trong `destinationRoot`.

Nếu không:

```text
INVALID_PATH
```

LocalSend từng có high-severity path traversal vulnerability cho phép malicious peer ghi file ngoài destination directory. Đây là regression test bắt buộc cho project.

---

# 36. Filename handling

Metadata encoding:

```text
UTF-8
```

MUST support:

```text
Ảnh sinh nhật.jpg
日本語.txt
한국어.pdf
📷 Photo.mov
```

Receiver phải sanitize filename theo filesystem của mình.

Original filename MAY được lưu metadata riêng.

Một filename không hợp lệ trên Windows không được làm fail cả transfer nếu có thể sanitize an toàn.

---

# 37. Duplicate filenames

Receiver MUST có deterministic conflict strategy.

Default:

```text
photo.jpg
photo (1).jpg
photo (2).jpg
```

MAY hỗ trợ:

```text
replace
skip
rename
```

nhưng auto-receive MUST default sang safe rename.

Không silently overwrite.

---

# 38. Manifest scalability

Protocol MUST NOT có product-level file-count limit.

Không được hard-code:

```text
maxFiles = 100
```

Manifest lớn phải hỗ trợ paging/streaming.

Implementation MAY có resource protection để chống DoS.

Ví dụ:

```text
pageSize <= 1000
```

nhưng tổng item count không có business cap cố định.

---

# 39. Transfer negotiation

Sender tạo transfer offer.

Receiver trả một trong:

```text
ACCEPT
REJECT
```

Nếu accept:

```text
selectedTransferMode
```

được negotiate.

Preferred order:

```text
PULL
then PUSH
```

nếu cả hai đều hỗ trợ.

---

# 40. PUSH flow

Ví dụ:

```text
iOS → Windows
```

```text
iPhone                     Windows

discover
   │
   ├──────────────────────►
   │
authenticate
   │
   ├──────────────────────►
   │
POST transfer offer
   │
   ├──────────────────────►
   │
   │◄──── accepted ────────
   │
background upload
   │
   ├══════════════════════►
   │
   │◄──── verified ────────
```

---

# 41. PULL flow

Ví dụ:

```text
Windows → iOS
```

iOS app đang online qua control connection.

```text
Windows                    iPhone

transfer.offer
   │
   ├──────────────────────►
   │
   │◄──── accept ──────────
   │
expose files
   │
   │◄════ HTTPS GET ═══════
   │
verify
   │
   ├──────────────────────►
```

Một khi background download bắt đầu, user có thể chuyển app sang background/lock screen và URLSession chịu trách nhiệm lifecycle theo giới hạn của iOS.

---

# 42. Download endpoint

PULL endpoint:

```http
GET /v1/transfers/{transferId}/items/{itemId}
```

Server MUST support:

```http
Accept-Ranges: bytes
```

Client MAY request:

```http
Range: bytes=8388608-
```

Server:

```http
206 Partial Content
```

Downloads MUST support resume.

---

# 43. Upload

Official receiver implementations SHOULD hỗ trợ resumable HTTP upload để iOS có thể tận dụng native URLSession resume.

Apple hỗ trợ resumable upload từ iOS 17 khi server triển khai HTTP resumable-upload protocol phù hợp.

Tại thời điểm tài liệu này, IETF `draft-ietf-httpbis-resumable-upload-12` được phát hành ngày 06/07/2026 và vẫn là Internet-Draft, chưa phải RFC cuối cùng.

Vì vậy capability MUST version hóa:

```json
{
  "resumableUpload": [
    "httpbis-draft-12"
  ]
}
```

Không hard-code giả định rằng draft-12 sẽ tồn tại vĩnh viễn.

Future implementation có thể advertise:

```text
httpbis-draft-13
httpbis-rfc-xxxx
```

---

# 44. Upload fallback

Nếu receiver không hỗ trợ resumable upload:

```text
single HTTP upload
```

MUST vẫn hoạt động.

UI MAY hiển thị reduced reliability cho file rất lớn.

Official Windows/macOS/Android implementations của project SHOULD hỗ trợ resumable upload trước production release.

---

# 45. Streaming requirement

MUST NOT:

```text
load entire file
→ RAM
→ upload
```

Files có thể:

```text
1 KB
1 GB
20 GB
100 GB+
```

Memory consumption không được tỷ lệ thuận với file size.

Uploads trong iOS background MUST dùng file-backed task.

Apple background URLSession chỉ hỗ trợ background upload từ file; data/stream upload không tiếp tục đúng cách sau khi app process thoát.

---

# 46. Share Extension

Share Sheet là feature P0.

Flow:

```text
Photos / Files / Browser
          ↓
        Share
          ↓
   Application Extension
          ↓
      select peer
          ↓
       transfer
```

Share Extension MUST xử lý:

```text
NSItemProvider
```

Không load file lớn thành `Data`.

---

# 47. App Group

Containing app và Share Extension MUST có:

```text
App Group
```

Working identifier:

```text
group.<bundle-id>.shared
```

Background URLSession từ app extension MUST set:

```swift
configuration.sharedContainerIdentifier
```

sang App Group tương ứng.

Apple ghi rõ background session tạo từ app extension sẽ invalid nếu không có valid shared container identifier.

---

# 48. Share Extension staging

Flow:

```text
NSItemProvider
      ↓
file representation
      ↓
App Group staging
      ↓
background upload
```

Temporary/staged data MUST được cleanup:

- success;
- cancel;
- permanent failure;
- stale cleanup job.

Không được để App Group tăng dung lượng vô hạn.

---

# 49. Background sessions

Tối đa một số rất nhỏ session identifier ổn định.

Không tạo:

```text
one URLSession per file
```

Apple khuyến nghị sử dụng ít background sessions và có thể chạy nhiều task trong cùng session; hàng nghìn task riêng có overhead đáng kể.

Suggested:

```text
com.product.transfer.background.main
com.product.transfer.background.extension
```

---

# 50. Transfer state machine

```text
CREATED
   │
   ▼
DISCOVERING
   │
   ▼
CONNECTING
   │
   ▼
AUTHENTICATING
   │
   ▼
OFFERED
   │
   ├────────────► REJECTED
   │
   ▼
ACCEPTED
   │
   ▼
PREPARING
   │
   ▼
TRANSFERRING
   │
   ├────────────► PAUSED
   │                  │
   │                  ▼
   │             TRANSFERRING
   │
   ▼
VERIFYING
   │
   ▼
COMPLETED
```

Global terminal states:

```text
COMPLETED
REJECTED
CANCELLED
FAILED
```

---

# 51. Item state

Mỗi TransferItem:

```text
PENDING
PREPARING
TRANSFERRING
PAUSED
VERIFYING
COMPLETED
SKIPPED
FAILED
CANCELLED
```

Một transfer nhiều file không được chỉ lưu một progress number.

---

# 52. Progress calculation

Overall progress:

```text
sum(transferredBytes)
────────────────────────
sum(totalBytes)
```

Không tính:

```text
completedFiles / totalFiles
```

vì size file không đồng đều.

---

# 53. Integrity

Official implementations MUST support:

```text
SHA-256
```

Hash SHOULD được compute streaming cùng quá trình đọc/ghi nếu có thể.

Không bắt người dùng chờ hash toàn bộ file trước khi transfer bắt đầu.

Transfer flow:

```text
TRANSFERRING
     ↓
VERIFYING
     ↓
COMPLETED
```

Nếu mismatch:

```text
INTEGRITY_MISMATCH
```

Không đánh dấu completed.

---

# 54. Atomic file handling

Receiver không được write trực tiếp vào final path trong quá trình transfer.

Flow:

```text
.filename.partial
       ↓
download/upload
       ↓
verify
       ↓
atomic rename
       ↓
filename
```

Nếu transfer fail:

```text
.partial
```

có thể giữ để resume hoặc cleanup theo policy.

---

# 55. iOS file destination

Mặc định:

### Generic files

```text
Files
└── <App Name>
    └── Received
```

### Folder

```text
Files
└── <App Name>
    └── Received
        └── OriginalFolder
```

### Photos/videos

User setting MAY cho:

```text
Save received media to Photos
```

Chỉ request Photos permission khi thực sự cần.

Không request ngay onboarding nếu user chưa dùng feature.

---

# 56. Local Network permission UX

Trước system prompt phải có pre-permission screen:

```text
Find your devices

This app uses your local network to find your
computers and transfer files directly.

Your files are not uploaded to the cloud.

[Continue]
```

Sau đó mới trigger Local Network operation.

Nếu denied:

```text
Local Network Access Required

[Open Settings]
```

Apple local network permission có trạng thái undetermined/allowed/denied và người dùng có thể đổi trong Settings.

---

# 57. Main iOS navigation

MVP chỉ cần:

```text
Devices
Transfers
Settings
```

## Devices

```text
Ready to transfer

Nearby

💻 Tuan-PC
   Trusted · Online

💻 MacBook
   Trusted · Online

📱 Pixel
   Trusted · Online
```

Primary button:

```text
Send
```

---

# 58. Send flow

```text
Send
 ↓
Choose Files / Folder / Photos
 ↓
Choose Device
 ↓
Review
 ↓
Send
```

Hoặc preferred mobile flow:

```text
Photos
 ↓
Share
 ↓
Our App
 ↓
Choose Device
```

---

# 59. Receive flow

Trusted auto-accept disabled:

```text
Tuan-PC wants to send

3 files
4.8 GB

[Decline] [Accept]
```

Trusted auto-accept enabled:

```text
Receiving from Tuan-PC
```

Auto accept MUST chỉ áp dụng cho cryptographically trusted peer.

---

# 60. History

Transfer history MUST lưu:

```text
transferId
direction
peer
startedAt
completedAt
status
totalBytes
itemCount
```

MAY lưu:

```text
file names
```

nhưng phải có:

```text
Clear History
```

Không upload history lên server/cloud.

---

# 61. Persistence

Domain entities:

```text
TrustedPeer
Transfer
TransferItem
TransferProgress
AppSetting
StagedShare
```

Secrets:

```text
Keychain
```

Application data có thể dùng:

```text
SQLite
```

hoặc một native persistence layer được abstraction.

Domain layer MUST không phụ thuộc persistence implementation.

---

# 62. Clean Architecture

iOS project:

```text
iOS/
│
├── App/
│
├── Presentation/
│   ├── Devices/
│   ├── Transfers/
│   ├── Pairing/
│   └── Settings/
│
├── Application/
│   ├── DiscoverPeers/
│   ├── PairPeer/
│   ├── SendTransfer/
│   ├── ReceiveTransfer/
│   └── ResumeTransfer/
│
├── Domain/
│   ├── Device/
│   ├── Transfer/
│   ├── Pairing/
│   └── Protocol/
│
├── Infrastructure/
│   ├── Discovery/
│   ├── Networking/
│   ├── Security/
│   ├── Persistence/
│   ├── Files/
│   └── Notifications/
│
└── ShareExtension/
```

Dependency direction:

```text
Presentation
     ↓
Application
     ↓
Domain

Infrastructure
     ↓
Domain/Application interfaces
```

Domain MUST NOT import SwiftUI.

---

# 63. Core interfaces

Minimum abstractions:

```text
PeerDiscovery
PeerRepository

PairingService
AuthenticationService
TrustStore

TransferNegotiator
TransferRepository

UploadTransport
DownloadTransport

FileStore
HashingService

BackgroundTransferManager
NotificationService
```

Protocol implementation phải mock được cho unit test.

---

# 64. Reference host architecture

Repository MUST chứa reference host riêng.

Recommended:

```text
reference-host/
└── rust/
```

Rust reference host phải build được cho ít nhất:

```text
Windows
macOS
```

MAY support Linux phục vụ development.

Responsibilities:

```text
HTTP/HTTPS server
Bonjour advertise
TLS
pairing
authentication
transfer API
upload
download
range
resumable upload
SHA-256
filesystem
```

UI không bắt buộc.

CLI đủ:

```text
ULTP Reference Host

Device: Tuan-PC
Listening: 43120
Status: Ready
```

---

# 65. Android reference implementation

Android reference app chưa cần product UI.

Minimum:

```text
Start Receiver
Stop Receiver
Nearby peers
Select test file
Transfer log
```

Mục tiêu:

```text
iOS → Android
Android → iOS
```

phải pass interoperability suite.

---

# 66. LocalSend usage

LocalSend Protocol được dùng làm **reference**, không phải dependency bắt buộc.

LocalSend v2.2 dùng REST, local transfer không external server, `prepare-upload`, session/file token và reverse download API; họ cũng chỉ yêu cầu một bên có HTTP server. Đây là reference hữu ích cho protocol design.

Không implement LocalSend compatibility trong v1.

Future MAY có:

```text
LocalSendCompatibilityAdapter
```

---

# 67. Security lessons from LocalSend

Project MUST có regression tests cho ít nhất:

### Device impersonation / MITM

Không trust discovery metadata. LocalSend từng có critical advisory cho unauthenticated discovery impersonation.

### Path traversal

Không cho remote path thoát destination root.

### Filename rendering

Filename là untrusted user-controlled data.

Nếu sau này có HTML/web interface:

```text
filename
```

MUST được escape như text, tuyệt đối không chèn raw HTML. LocalSend từng có stored XSS advisory liên quan filename trong web share interface.

---

# 68. Input validation

Mọi remote input là untrusted.

Phải validate:

```text
JSON
device name
filename
relativePath
mimeType
transferId
itemId
file size
hash
capabilities
URL
headers
range
token
```

Malformed input MUST return structured protocol error.

Không crash process.

---

# 69. Resource exhaustion protection

"No user-facing file limit" không có nghĩa attacker được phép tiêu tốn vô hạn memory.

Implementation MUST:

- stream manifest lớn;
- giới hạn HTTP request body metadata;
- giới hạn nesting;
- timeout idle connection;
- rate-limit pairing;
- rate-limit authentication failure;
- rate-limit unauthenticated requests;
- tránh allocate dựa trực tiếp trên remote declared file size.

File size MUST dùng 64-bit unsigned representation.

---

# 70. Error response

Common shape:

```json
{
  "error": {
    "code": "DEVICE_NOT_TRUSTED",
    "message": "The peer is not trusted.",
    "retryable": false
  }
}
```

Required error codes:

```text
PROTOCOL_UNSUPPORTED
CAPABILITY_UNSUPPORTED

PAIRING_REQUIRED
PAIRING_REJECTED
PAIRING_EXPIRED

AUTH_FAILED
DEVICE_IDENTITY_CHANGED

TRANSFER_NOT_FOUND
TRANSFER_REJECTED
TRANSFER_CANCELLED

ITEM_NOT_FOUND
INVALID_PATH
INVALID_FILENAME

INSUFFICIENT_STORAGE
PERMISSION_DENIED

NETWORK_INTERRUPTED
PEER_OFFLINE

HASH_MISMATCH

RATE_LIMITED
RESOURCE_LIMIT

INTERNAL_ERROR
```

UI không hiển thị raw internal exception.

---

# 71. Logging

Sử dụng structured logging.

iOS:

```text
OSLog
```

Categories:

```text
discovery
pairing
auth
control
transfer
background
storage
security
```

MUST NOT log:

- private keys;
- auth tokens;
- full file content;
- sensitive text payload;
- Photos content.

Debug logging MAY log sanitized filename.

---

# 72. Analytics/privacy

Không analytics SDK bắt buộc trong v1.

Nếu analytics được thêm:

MUST NOT collect:

```text
filename
file contents
text payload
URL payload
private IP
public key
auth token
clipboard
```

MAY collect opt-in anonymous aggregates:

```text
transferSuccess
transferFailure
platform
protocolVersion
sizeBucket
durationBucket
```

---

# 73. No-file-size assumption

Không code:

```text
Int32 fileSize
```

MUST support:

```text
UInt64
```

Tests phải có sparse/test files:

```text
> 4 GB
> 10 GB
```

Không yêu cầu test device phải có 100 GB thực tế nếu có thể dùng generated/sparse fixtures.

---

# 74. Interoperability matrix — Phase 1

Bắt buộc pass:

| Sender | Receiver | Required |
|---|---|---|
| iOS | Windows | ✅ |
| Windows | iOS | ✅ |
| iOS | macOS | ✅ |
| macOS | iOS | ✅ |
| iOS | Android | ✅ |
| Android | iOS | ✅ |

Full mesh sau:

| Sender | Receiver |
|---|---|
| Windows | macOS |
| macOS | Windows |
| Windows | Android |
| Android | Windows |
| macOS | Android |
| Android | macOS |

Protocol v1 không được cần thay đổi breaking để support matrix sau.

---

# 75. Functional test matrix

Mỗi interoperability pair phải test:

```text
1 byte
1 KB
1 MB
100 MB
1 GB
10 GB+
```

Và:

```text
1 file
10 files
100 files
1,000 files
folder
nested folders
empty file
empty folder
duplicate names
```

---

# 76. Filename test matrix

Test:

```text
hello.txt

ảnh gia đình.jpg

日本語.txt

한국어.txt

😀📷🎂.jpg

file with spaces.pdf

CON.txt
AUX.txt

a:b?.txt

very-long-name...
```

Receiver không được crash.

---

# 77. Network resilience tests

Bắt buộc:

```text
Wi-Fi disconnect
Wi-Fi reconnect

router restart

IP change

peer process restart

iPhone screen lock

iPhone background

system terminates app

user force-quits app

sender sleeps

receiver sleeps
```

Expected behavior phải deterministic.

---

# 78. Background iOS acceptance tests

## Download

1. Start 10 GB Windows → iPhone.
2. Transfer đạt >5%.
3. Lock iPhone.
4. Chờ transfer tiếp tục.
5. Unlock.
6. Transfer phải hoàn thành hoặc resume đúng.
7. SHA-256 match.

## Upload

1. Share video lớn từ Photos/Files.
2. Start iPhone → Windows.
3. Close Share Extension.
4. Background transfer tiếp tục.
5. Lock device.
6. Transfer hoàn thành.
7. SHA-256 match.

## Force quit

1. Start transfer.
2. User force-quits app.
3. Transfer có thể bị cancel theo lifecycle iOS.
4. App mở lại phải reconcile state và cho retry/resume nếu protocol/server state còn khả dụng.

Không xem bước 3 là bug.

---

# 79. Security test matrix

MUST có automated hoặc integration tests cho:

```text
fake Bonjour identity

changed TLS key

wrong pairing signature

SAS mismatch

expired token

stolen file token

path traversal

absolute path

Windows UNC path

malformed Range

oversized metadata

invalid UTF-8

duplicate IDs

replayed authentication message

replayed transfer token

hash mismatch

partial upload corruption

unauthenticated upload

unauthenticated download
```

---

# 80. Performance requirements

Priority:

```text
Reliability
>
Integrity
>
Memory
>
Speed
```

Target cho một large file trong LAN tốt:

```text
≥ 70% raw achievable TCP throughput
```

sau optimization.

Không được hy sinh integrity/security để đạt speed.

---

# 81. Memory requirement

Application MUST NOT giữ entire file trong memory.

Mục tiêu iOS app process:

```text
< 150 MB peak
```

khi truyền file lớn trong common scenario.

Share Extension MUST đặc biệt tránh memory-heavy operations.

Không copy file lớn nhiều lần nếu tránh được.

---

# 82. Storage requirement

Trước nhận file:

```text
requiredBytes
```

SHOULD được kiểm tra với available storage.

Nếu thiếu:

```text
INSUFFICIENT_STORAGE
```

Không bắt đầu nhận rồi đợi filesystem fail ở 99%.

---

# 83. Battery/network behavior

Không:

```text
busy-loop discovery
scan entire subnet continuously
poll /info every second
```

Bonjour event-driven.

WebSocket event-driven.

Background URLSession do system quản lý.

---

# 84. Repository structure

Recommended monorepo:

```text
product/
│
├── protocol/
│   ├── SPEC.md
│   ├── SECURITY.md
│   ├── DISCOVERY.md
│   ├── TRANSFER.md
│   │
│   ├── schemas/
│   │   ├── device-info.schema.json
│   │   ├── capabilities.schema.json
│   │   ├── transfer.schema.json
│   │   └── error.schema.json
│   │
│   └── openapi/
│       └── ultp-v1.yaml
│
├── apps/
│   ├── ios/
│   ├── android/
│   ├── windows/
│   └── macos/
│
├── reference/
│   └── rust-host/
│
├── interoperability/
│   ├── fixtures/
│   └── tests/
│
├── docs/
│   ├── ADR/
│   ├── architecture/
│   └── security/
│
└── changelog/
```

---

# 85. Protocol schemas

Protocol MUST có machine-readable schemas.

Recommended:

```text
JSON Schema
+
OpenAPI
```

Không để Swift/Kotlin/Rust tự định nghĩa DTO khác nhau bằng tay rồi drift.

CI phải validate examples against schemas.

---

# 86. Versioning

Version v1:

```text
1
```

Peer advertise:

```json
{
  "protocol": {
    "min": 1,
    "max": 1
  }
}
```

Negotiation:

```text
highest mutually supported version
```

Nếu không overlap:

```text
PROTOCOL_UNSUPPORTED
```

---

# 87. Backward compatibility rule

Trong cùng major protocol:

MAY:

- add optional field;
- add capability;
- add error code;
- add optional endpoint.

MUST NOT:

- đổi nghĩa existing field;
- remove required field;
- đổi cryptographic semantics;
- đổi transfer semantics mà không negotiate.

Breaking changes:

```text
protocol v2
```

---

# 88. AI coding rules

AI agent MUST:

1. Không tự thay kiến trúc PUSH/PULL.
2. Không thay HTTP/HTTPS bằng custom TCP protocol.
3. Không thêm cloud.
4. Không thêm account.
5. Không thêm WebRTC.
6. Không thêm Wi-Fi Direct.
7. Không bỏ pairing/security để làm demo nhanh.
8. Không hard-code platform transport logic.
9. Không load entire file vào RAM.
10. Không hard-code giới hạn 100 file.
11. Không bypass path validation.
12. Không trust discovery.
13. Không ignore hash mismatch.
14. Không implement UI trước core interoperability.
15. Mọi thay đổi ngoài spec phải có ADR.

---

# 89. Coding quality

Implementation phải follow:

```text
Clean Architecture
SOLID
KISS
YAGNI
```

Không tạo abstraction không có use case.

Không over-engineer distributed system.

Không microservice.

Không dependency injection framework nặng nếu constructor injection đủ.

---

# 90. Testing rules

Business/domain logic:

```text
unit tests
```

Protocol:

```text
contract tests
```

Network:

```text
integration tests
```

Cross-platform:

```text
interoperability tests
```

iOS UI:

```text
XCTest / UI tests
```

Critical security regressions MUST có automated tests.

---

# 91. Development milestone M0 — Protocol skeleton

Deliverables:

```text
protocol/SPEC.md
protocol/SECURITY.md
protocol/openapi/ultp-v1.yaml
protocol/schemas/*
```

Must compile/validate schemas.

Không cần UI.

---

# 92. M1 — Reference Host

Rust reference host:

```text
Bonjour
/info
TLS
pairing
auth
upload
download
HTTP Range
SHA-256
```

Run được trên:

```text
Windows
macOS
```

---

# 93. M2 — iOS Discovery + Pairing

iOS:

```text
Local Network onboarding
Bonjour discovery
Device list
Pairing
SAS
Keychain trust store
```

Acceptance:

```text
iPhone ↔ Windows reference host
iPhone ↔ macOS reference host
```

pair thành công.

---

# 94. M3 — iOS → Windows

Support:

```text
one file
background upload
progress
cancel
integrity
```

Acceptance:

```text
10 GB transfer
SHA-256 identical
```

---

# 95. M4 — Windows → iOS

Support:

```text
control offer
PULL
background download
HTTP Range
resume
integrity
```

Acceptance:

```text
10 GB
screen locked
successful receive
```

---

# 96. M5 — Share Extension

Support:

```text
Photos → Share → app → Windows
Files → Share → app → Windows
```

Background transfer phải survive Share Extension closure.

---

# 97. M6 — Multi-file + folder

Support:

```text
multiple files
folders
nested folders
relative paths
duplicate names
```

No path traversal.

---

# 98. M7 — macOS interoperability

Pass:

```text
iOS → macOS
macOS → iOS
```

Không special-case macOS trong iOS business logic.

---

# 99. M8 — Android interoperability

Minimal Android host.

Pass:

```text
iOS → Android
Android → iOS
```

Không special-case Android trong iOS business logic.

---

# 100. M9 — Production UX

Sau khi interoperability hoàn thành mới tập trung:

```text
animations
onboarding polish
history
notifications
settings
auto accept
visual design
```

---

# 101. Phase 1 Definition of Done

Phase iOS-first chỉ được xem là hoàn thành khi:

### Protocol

- Protocol v1 documented.
- OpenAPI valid.
- Schemas valid.
- Capability negotiation hoạt động.
- Security model implemented.

### iOS

- Native Swift implementation.
- Local Network permission đúng.
- Bonjour discovery.
- Pairing.
- SAS verification.
- Trusted peers.
- Share Extension.
- Background uploads.
- Background downloads.
- Multi-file.
- Folder.
- Resume.
- Progress.
- Cancel.
- History.
- Notifications.

### Interoperability

Pass:

```text
iOS ↔ Windows
iOS ↔ macOS
iOS ↔ Android
```

### Reliability

- network disconnect test pass;
- resume test pass;
- background test pass;
- integrity test pass;
- no whole-file memory loading.

### Security

- TLS enabled;
- discovery not trusted;
- device identity pinned;
- path traversal blocked;
- unauthenticated file access blocked;
- replay protection;
- malformed remote data không crash.

### Quality

```text
0 compiler errors
0 warnings
all unit tests pass
all integration tests pass
all interoperability tests pass
CI green
```

---

# 102. Explicit non-goals cho AI agent

Nếu AI nghĩ:

> "Thêm cloud sẽ dễ hơn."

Không làm.

Nếu AI nghĩ:

> "Dùng Firebase để discovery."

Không làm.

Nếu AI nghĩ:

> "Dùng WebRTC cho nhanh."

Không làm.

Nếu AI nghĩ:

> "Chỉ làm iOS ↔ Windows trước rồi protocol tính sau."

Không làm.

Nếu AI nghĩ:

> "Để security sau."

Không làm.

Nếu AI nghĩ:

> "Load Data(contentsOf:) cho đơn giản."

Không làm với file transfer.

Nếu AI nghĩ:

> "ZIP folder trước."

Không làm mặc định.

---

# 103. Kiến trúc cuối cùng cần đạt

```text
                   UNIVERSAL TRANSFER PROTOCOL

                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
   Discovery             Security             Transfer
        │                    │                    │
     Bonjour             Identity             Manifest
     DNS-SD              Pairing              PUSH
     QR                  SAS                  PULL
                         TLS                  Resume
                         Trust                SHA-256
                                                │
                                                │
              ┌─────────────────────────────────┼──────────────┐
              │                                 │              │
              ▼                                 ▼              ▼

             iOS                            Windows         Android
        Swift / URLSession                 HTTP Host       HTTP Host
              │
              │
              └──────────────────────────────┐
                                             │
                                             ▼
                                           macOS
                                          HTTP Host
```

Sau này:

```text
Windows ↔ Android
Windows ↔ macOS
Android ↔ macOS
```

không tạo protocol mới.

Chỉ triển khai cùng protocol.

---

# 104. Core product contract

Sản phẩm phải cho cảm giác:

```text
Open
 ↓
Devices appear
 ↓
Select
 ↓
Send
```

Không:

```text
configure IP
 ↓
create account
 ↓
login
 ↓
upload cloud
 ↓
download
```

Đối với mobile:

```text
Photos / Files
       ↓
      Share
       ↓
     Device
```

phải là flow quan trọng nhất.

---

# 105. Final architectural decision

Quyết định cuối cùng của v1:

> **Chúng ta không xây một ứng dụng iOS có tính năng gửi file.**

Chúng ta xây:

> **một universal local-transfer protocol và iOS là production implementation đầu tiên của protocol đó.**

Transport baseline:

```text
Bonjour / DNS-SD
        +
HTTPS / TLS
        +
REST control
        +
WebSocket events
        +
PUSH / PULL negotiation
        +
URLSession
        +
HTTP Range
        +
resumable upload
        +
SHA-256
```

Security baseline:

```text
P-256 device identity
        +
ECDH pairing
        +
SAS verification
        +
public-key/SPKI pinning
        +
challenge-response authentication
```

iOS baseline:

```text
Swift + SwiftUI
URLSession
Network.framework
CryptoKit
Keychain
Share Extension
App Groups
```

Đây là nền móng phải được giữ ổn định để các implementation Windows, macOS và Android sau này có thể giao tiếp trực tiếp với nhau mà không cần thay đổi bản chất protocol.