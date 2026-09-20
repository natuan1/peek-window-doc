# Nhận push — implementation

Mã ở `apps/windows/src/Snappy.Protocol/`, hai thư mục mới — `Files/` và
`Transfers/` — cộng một nửa mới của router.

## Chỗ đắt nhất: thân request không còn là `byte[]`

Ticket 05 dựng `Http1Reader` với đúng hai lựa chọn cho thân: **đọc trọn vào một
`byte[]`**, hoặc **nhả đi**. Một tệp 4 GB không đi được bằng cả hai.

```
Http1BodyStream : Stream      ← chặn đúng Content-Length, đọc từ bộ đệm rồi mới chạm socket
UltpRouter.PlanFor(path, method) → Drain | Control | Manifest | Stream
```

`PlanFor` đi qua **đúng** hàm `Match` mà việc định tuyến dùng. Một bản sao luật
khớp đường dẫn ở tầng server là chỗ để hai tầng hiểu khác nhau về cùng một URL —
mà "hai cách hiểu về cùng chuỗi byte" đúng là định nghĩa của request smuggling.

`Http1BodyStream` tự chặn ở `Remaining` thay vì đọc tới EOF, và lý do không phải
sự cẩn thận: trên keep-alive **không có EOF**, byte ngay sau thân là dòng đầu của
request kế tiếp.

## Ba trần thân, không phải một

| Đường | Trần | Vì sao |
|---|---|---|
| Control plane (pairing, auth) | 16 KB | thân lớn nhất là hai SPKI base64, ~600 byte |
| `POST /v1/transfers` | **1 MB** | SECURITY.md §4.3 "Body metadata ≤ 1 MB" |
| `PUT …/items/{id}` | `size` của Offer | chỉ router biết con số ấy |
| còn lại | 1 MB, nhả đi | |

Manifest **phải** rộng hơn control plane: SPEC §18.4 cấm hard-code `maxFiles`, và
16 KB là khoảng **một trăm** item — tức đúng một `maxFiles`, chỉ viết bằng byte
thay vì bằng số. ⚠️ 1 MB vẫn là một trần (~7000 item), và lời giải đúng của
protocol là gửi manifest theo trang — mà §18.4 chỉ định nghĩa cho chiều *đọc*.

## "Lingering close" — một câu trả lời đúng biến thành một lỗi mạng

Server từ chối sớm (`403` token sai, `413` vượt `size`) rồi đóng ngay thì bên kia
đang ở giữa lời ghi: nó nhận `ECONNABORTED` **trước khi** kịp đọc câu trả lời, và
màn hình iPhone nói *"mất kết nối"* cho một chuyện là *"không có quyền"*. Đúng bài
học 36 của `peekvn`: một câu trôi chảy nhưng sai chuyện vừa xảy ra thì khó bị báo
hơn một mã lỗi thô.

Nên: **trả lời trước, rồi đọc bỏ có trần** (2 MB / 5 giây) mới đóng. Trần là cả
nửa còn lại của lời hứa — nhả *hết* một thân 4 GB là ngồi đọc 4 GB của một kẻ vừa
bị từ chối, đúng thứ §25 cấm. Con số 2 MB = gấp đôi trần manifest, vì khi thân bị
từ chối *vì vượt trần* thì thứ chưa đọc là **cả** thân.

## Giới hạn quay ra chĩa vào người dùng — SECURITY.md §4.3

Bản Ticket 05 đếm **mọi** request ở tầng server, nơi chưa ai biết người gọi là
ai. Dòng §4.3 nói *"Request **chưa authenticate**: 60 / phút"*, và chữ *chưa
authenticate* là cả nội dung của nó.

Hậu quả chỉ lộ ra từ ticket này: một thư mục 70 tệp là 1 `POST` + 70 `PUT`, tức
hạn 60 chặn đứng một **người dùng thật** ở giữa lượt truyền. §4.3 gọi đúng tên
chuyện này ở hai gạch đầu dòng cuối. Phép đếm nay nằm trong router, sau khi
session token đã được tra — và có một test đối chứng khẳng định người lạ **vẫn**
bị chặn ở 60.

## `RelativePath` — hai bước, hai chuỗi, thứ tự bắt buộc

```csharp
TryValidate(raw)      // §19.1 — trên chuỗi ĐÚNG NHƯ NHẬN ĐƯỢC
  → TrySanitize(segs) // §19.2 — gọt cho vừa NTFS, rồi kiểm LẠI
  → segments          // từ đây chỉ chuỗi đã gọt được dùng
```

Bước 1 chạy trước nên bước 2 **không thể** đẻ ra traversal: mọi đoạn tới tay nó
đã không phải `.` hay `..`, và nó chỉ *thay* hoặc *cắt* ký tự chứ không nối đoạn.
Bước 2 vẫn kiểm lại, vì phép cắt đuôi chấm/khoảng trắng tự nó biến `".. "` thành
`".."` — 📐 đo được, Win32 làm đúng thế và im lặng.

`:` bị gọt ở **mọi** đoạn, không chỉ đoạn đầu: SECURITY.md §7.6 chỉ soi đoạn đầu
vì trên POSIX `a:b.txt` là một tên tệp bình thường. Trên NTFS nó không phải tên
tệp — nó là một câu lệnh (Alternate Data Stream).

## `DestinationRoot` — hỏi hệ tệp, ở từng đoạn

```csharp
Directory.ResolveLinkTarget(candidate, returnFinalTarget: true)   // theo hết junction
  → Contains(real)                                                // S11, mỗi đoạn một lần
  → new FileStream(path, FileMode.CreateNew, …)                   // S16, hệ tệp trả lời "đã có chưa"
  → File.Move(partial, final, overwrite: false)                   // nguyên tử, không giữ chỗ 0 byte
```

`CreateNew` quan trọng vì lý do **đúng**, không vì lý do nghe hợp lý: SPEC §19.3
nói trùng tên MUST được phát hiện bởi filesystem, MUST NOT bằng cách so chuỗi.
NTFS không phân biệt hoa thường, nên `Photo.JPG` và `photo.jpg` là **cùng một
tệp**; một receiver tự so chuỗi sẽ kết luận "hai tên khác nhau" rồi ghi đè im
lặng — kịch bản một peer độc hại gửi hai item trong cùng Transfer để xoá dữ liệu,
và nó không trông giống một lỗ hổng.

`File.Move` **không** `overwrite` thay cho "mở tên cuối để xí chỗ rồi rename đè":
cách sau cũng giữ S16 nhưng nó đẻ ra một tệp rỗng mang tên cuối và một cửa sổ
giữa lúc đóng handle với lúc rename.

## Vì sao không có hàng đợi verify bền

SPEC §20.2 nói `VERIFYING` MUST sống sót qua việc process bị kill. Câu MUST ấy có
một hoàn cảnh cụ thể: **nền tảng ghi tệp hộ** (background download của iOS), nên
app chỉ được đánh thức một lúc rất ngắn và hash 4 GB trong callback thì bị giết
giữa chừng.

Ở đây Snappy **cầm từng byte** và băm ngay lúc ghi, nên `VERIFYING` không có độ
dài — nó là một phép so 32 byte. Ghi ra vì một mục im lặng sẽ được đọc thành "đã
quên".

## `TransferStore` chỉ nằm trong bộ nhớ

Bản Rust bền hoá trạng thái xuống đĩa vì Host của nó tạo Offer `pull` sống 24 giờ,
và một lần khởi động lại sẽ quên hết lời mời. Chiều đang làm là `push`: Offer do
iPhone tạo và iPhone giữ, nên Snappy quên sau khi khởi động lại chỉ làm iPhone
phải gửi lại — một cú bấm, không phải mất dữ liệu. Ticket 09 dựng chiều `pull` và
mang theo đúng câu hỏi ấy.

## Bảng khai năng lực: điều kiện cũ thôi phân biệt được

`pushReceiver`, `file`, `folder` lật lên `true`. Nhưng điều kiện cũ của
`CapabilityTruthTests` là *"có route nào dưới `/v1/transfers` không"* — một proxy
đủ tốt khi **chưa có** route nào. Từ lúc `PUT` có mặt mà `GET` thì chưa, nó trả
`true` cho cả sáu dòng, tức đòi khai `pullSender` và `httpRange` là `true` trong
khi hai thứ ấy không tồn tại.

Bài học 158 ở dạng tinh vi hơn: không phải một danh sách bỏ sót một dòng, mà một
**điều kiện** thôi phân biệt được ngay lúc nó bắt đầu quan trọng. Hàng rào vẫn đỏ
đúng lúc — và câu trả lời đúng là **siết điều kiện**, không phải lật cờ cho hết
đỏ. Nay mỗi dòng đo đúng thứ của nó: `pushReceiver` ↔ `PUT` trên đường item,
`pullSender`/`httpRange` ↔ `GET`, `file`/`folder`/`text`/`url` ↔
`UltpRouter.SupportedItemKinds` (cùng **một** object mà bộ validate manifest đọc,
không phải một danh sách thứ hai).
