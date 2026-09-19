# Implementation — Server ULTP trên Windows

Mã nằm ở `peekvn/apps/windows/`. Tất cả trong `Snappy.Protocol` trừ phần nối
vào vòng đời app.

## File chính

| File | Vai trò |
|---|---|
| `src/Snappy.Protocol/Identity/TlsIdentity.cs` | Certificate self-signed P-256, bền qua khởi động lại (SPEC §9.2) |
| `src/Snappy.Protocol/Server/UltpServer.cs` | Vòng accept dual-stack, bắt tay TLS, keep-alive, giới hạn |
| `src/Snappy.Protocol/Server/Http1Reader.cs` | Phân tích đầu request, bộ đệm cố định 16 KB |
| `src/Snappy.Protocol/Server/Http1Request.cs` | Kiểu request đã phân tích + kết cục của một lượt đọc |
| `src/Snappy.Protocol/Server/Http1Response.cs` | Ghi response trong **một** lượt `WriteAsync` |
| `src/Snappy.Protocol/Server/UltpRouter.cs` | Bảng đường dẫn — hôm nay đúng một đường |
| `src/Snappy.Protocol/Server/UltpLimits.cs` | SPEC §25 / SECURITY.md §4.3 |
| `src/Snappy.Protocol/Server/SlidingRateLimiter.cs` | Cửa sổ trượt 60 giây theo địa chỉ nguồn |
| `src/Snappy.Protocol/Wire/DeviceInfo.cs` | DTO của `/v1/info` và envelope lỗi |
| `src/Snappy.Protocol/Wire/SnappyCapabilities.cs` | Nguồn **duy nhất** của bảng năng lực |
| `src/Snappy.Protocol/Wire/UltpJson.cs` | Bối cảnh serializer **sinh lúc biên dịch** |
| `src/Snappy.Protocol/Discovery/PeerInfoProbe.cs` | Lượt hỏi `/v1/info` điền cột nền tảng |
| `src/Snappy.Core/AppHost.cs` | Mở/đóng server theo vòng đời app |
| `src/Snappy.Core/NeighborDiscovery.cs` | Dựng thân `/v1/info`, gọi probe |

## Ba quyết định đáng đọc trước khi sửa

### 1. Khoá TLS phải do CNG giữ, không được là khoá tạm

SChannel tìm khoá riêng qua thuộc tính `CERT_KEY_PROV_INFO` của certificate,
**không** qua đối tượng .NET đang cầm nó. Một certificate dựng từ
`ECDsa.Create()` mang khoá *ephemeral*: ký được trong tiến trình này, nhưng
không có mục nào trong kho để thuộc tính kia trỏ tới — và
`AuthenticateAsServerAsync` ném *"The credentials supplied to the package were
not recognized"*, một câu lỗi không nhắc một chữ nào tới khoá.

Vì vậy khoá **luôn** được tạo trong kho CNG của người dùng trước, rồi
certificate mới được dựng quanh nó. Cùng chỗ tương đương Keychain của iOS mà
SPEC gọi là "secure storage", cùng cách `DeviceIdentity` đã làm — và cố ý dùng
**tên khoá khác**, vì hai tên trùng nhau là cách hai cặp khoá âm thầm trở thành
một.

Mất tệp `tls-cert.der` thì dựng lại được từ khoá; mất khoá thì phải ghép đôi
lại. Cấp lại vì tệp hỏng **có ghi nhật ký** — cấp lại im lặng làm mọi peer đã
ghim báo `DEVICE_IDENTITY_CHANGED` mà không để lại dấu vết nào để bắt đầu tìm.

### 2. JSON phải đi qua bối cảnh sinh lúc biên dịch

`Snappy.Core` publish Native AOT. Đường phản chiếu của `JsonSerializer` bị
ILCompiler cắt mất — nó **không** đỏ lúc build, nó ném `NotSupportedException`
lúc chạy, tức chỉ lộ ra trên máy người dùng.

Bối cảnh `UltpJson.Wire` khác `UltpJson.Default` đúng hai thứ:

- **Bộ mã hoá.** Mặc định của `System.Text.Json` an toàn cho HTML nên nó escape
  cả `+` lẫn mọi ký tự có dấu. 📐 Đo trên bản AOT thật: `publicKey` ra dây thành
  `…Ib+Tq…`, và một `displayName` tiếng Việt thành sáu byte mỗi chữ. Hợp
  lệ với mọi bộ phân tích, nhưng nó làm cùng một object trông khác hẳn ở ba
  implementation — mà *"ba bên sinh ra cùng byte"* đúng là thứ
  `interoperability/run.sh` đi kiểm.
- **`MaxDepth = 32`** — SECURITY.md §4.3. Mặc định 64 không tràn stack, nhưng 64
  không phải con số ta hứa, và một trần của thư viện là thứ đổi được ở lần cập
  nhật sau mà không ai để ý.

### 3. `System.Uri` nuốt zone của IPv6

📐 `new Uri("https://[fe80::1%7]:8443/")` cho ra host `fe80::1` — mất `%7`,
không ngoại lệ, không cờ nào bảo nó giữ. Với link-local, zone là thứ duy nhất
nói địa chỉ ấy thuộc giao diện mạng nào, và máy dev có ba giao diện.

Vì 4 trong 5 địa chỉ quảng bá là IPv6, đây không phải một nhánh hiếm. Bản vá:
địa chỉ thật đi xuống socket dưới dạng `IPEndPoint` qua
`SocketsHttpHandler.ConnectCallback` — kiểu duy nhất còn giữ `ScopeId` — còn URI
chỉ làm phần việc còn lại của nó. Ghi ở bài học **171** của `peekvn`.

## Giới hạn tài nguyên

Bảng SECURITY.md §4.3 ghi ❌ cho Reference Host ở hai dòng thuộc tầng transport.
Cả hai nằm trong `UltpLimits` vì server này tự viết vòng accept.

| Dòng §4.3 | Snappy | Con số |
|---|---|---|
| Connection idle | ✅ | 60 giây |
| Trần connection đồng thời | ✅ | 64 |
| Request chưa authenticate | ✅ | 60/phút/địa chỉ nguồn |
| Body metadata | ✅ | 1 MB |
| Độ sâu JSON | ✅ | 32 |
| *(riêng của h1)* Đầu request | ✅ | 16 KB → `431` |
| *(riêng của h1)* Số header | ✅ | 64 |
| *(riêng của h1)* Bắt tay TLS | ✅ | 10 giây |

**Khoá rate-limit là địa chỉ đã quy về IPv4 nếu nó là IPv4-mapped.** Listener
chạy dual-stack nên cùng một máy tới bằng `127.0.0.1` hay `::ffff:127.0.0.1`
tuỳ đường; giữ hai dạng là hai khoá, tức **120** request/phút cho một máy. Cái
bẫy này chỉ có ở dual-stack và không lộ ra ở một test chạy toàn IPv4.

## Số đo (2026-09-19, bản AOT thật)

| Chỉ số | Ticket 04 | Ticket 05 | KPI |
|---|---|---|---|
| Bộ cài | 10,18 MB | **10,45 MB** | < 15 MB |
| exe | 8,04 MB | 8,59 MB | — |
| Working set lúc nghỉ | 17,84 MB | **19,39 MB** | < 25 MB |
| Working set sau một request | — | **21,87 MB** | < 25 MB |

**Con số 19,39 MB đã là con số sau một lần cắt.** Bản đầu dựng sẵn `HttpClient`
của `PeerInfoProbe` trong constructor và cho **22,38 MB** — `SocketsHttpHandler`
một mình **3,0 MB**, tiêu cho một máy chưa có hàng xóm nào để hỏi, tức trạng
thái thường trực của một máy để bàn. Hoãn tới lần hỏi đầu tiên trả lại đúng 3 MB
ấy. Ghi ở bài học **172** của `peekvn`.

Biên tới KPI 25 MB còn ~5,6 MB lúc nghỉ và ~3,1 MB sau khi có kết nối, cho mười
ba ticket nữa — trong đó Mí (Ticket 10) mang theo cả Composition.
