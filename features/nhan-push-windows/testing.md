# Nhận push — đo bằng gì

Bốn lớp bằng chứng, cộng một bảng đột biến. Không lớp nào thay được lớp khác, và
bảng đột biến là thứ nói được rằng bốn lớp kia có đo gì không.

## 1. Đột biến — "những hàng rào này có bắt được gì không"

📐 20/09/2026, `apps/windows/spikes/mutate-07.py`: **25 đột biến, 25 lần bị bắt.**
Mỗi đột biến gỡ đúng một hàng rào rồi chạy đúng bộ test canh nó.

Bốn dòng đáng đọc hơn con số:

| Đột biến | Kết quả |
|---|---|
| `Path.GetFullPath` thay cho `ResolveLinkTarget` | ❌ 2 test đỏ — junction chỉ thấy được khi **hỏi hệ tệp** |
| Bỏ phép kiểm chứa ở **đoạn cuối** | ❌ 4 test đỏ — đây là lối thoát `NUL`, xem dưới |
| `.partial` tự nối chuỗi thay vì đi qua hàng rào | ❌ **1/50** test đỏ, và đúng cái test 32 luồng ghi song song |
| Trần 60 request/phút áp cho cả request đã xác thực | ❌ test "thư mục 70 tệp" đỏ, test đối chứng "người lạ vẫn bị chặn" xanh |

Ba đột biến **sống sót** ở lượt đầu, và cả ba đều đổi mã hoặc đổi test:

- **Cộng bão hoà cho tổng `size`.** Test cũ dùng **hai** item sát trần 2^53−1 —
  tổng ấy chưa tràn `long`, nên nó không phân biệt được hai giả thuyết. Đúng bài
  học 12: một phép đo nhỏ hơn quy mô thật không cho câu trả lời kém chính xác
  hơn, nó cho câu trả lời **sai**. Sửa: 1100 item, vì `long.MaxValue / (2^53−1)`
  ≈ 1024.
- **Phép cắt "gửi nhiều byte hơn `size`" ở mỗi chunk.** Nó **không đột biến được
  vì nó là mã chết**: `Content-Length` đã bị chặn trước byte đầu tiên, và
  `Http1BodyStream` không bao giờ trả quá `Content-Length`. Đúng hình dạng
  SECURITY.md §5.17 gọi là "hai hàng rào chồng nhau che mất nhau". Sửa: **bỏ**
  phép kiểm chết, và thêm một test đo đúng tính chất cấu trúc còn lại — client
  ghi thừa 5000 byte thì tệp vẫn đúng `size`.
- **Gỡ `SKIPPED` khỏi tập "mục đã ngã ngũ".** Sống sót vì nó **vô hại**: không
  test nào sinh ra `SKIPPED`, nên gỡ nó không đổi hành vi nào. Một đột biến sống
  sót vì nó không làm gì thì không nói gì về bộ test — viết lại thành gỡ
  `FAILED`, và nó bị bắt ngay.

## 2. Đường dẫn ở tầng hệ tệp — "lối thoát của Windows có bị chặn không"

`PathSafetyTests`, 50 test. Hai tầng, đúng như SECURITY.md §5.2 nói: schema chặn
được các dạng viết thẳng, còn junction, không gian tên thiết bị và những cái tên
Win32 tự sửa thì chỉ tầng hệ tệp thấy.

- 15 vector traversal bị chặn (`../`, `..\`, absolute, `C:\…`, `file://`, UNC,
  đoạn rỗng, ký tự điều khiển…);
- **9 tên hợp lệ đi qua được** — nhóm đối chứng, vì một hàng rào chặn mọi thứ
  không chứng minh được gì. Gồm `Ảnh sinh nhật.jpg`, `日本語.txt`, `📷 Photo.mov`;
- 3 junction: giữa đường, ở đoạn cuối, và **một cái trỏ vào trong root vẫn hợp lệ**
  (S11 nói *"resolve vào bên trong"*, không nói *"không được là junction"*);
- 32 luồng ghi song song cùng một tên → 32 tệp, 32 nội dung, không cái nào bị nuốt;
- **4 biến thể `NUL`** bị chặn, cộng **7 tên trông giống thiết bị mà không phải**
  (`CON`, `CON.txt`, `PRN.jpg`, `COM1.bin`, `aux.txt`, `LPT1`, `NUL.txt`) đi qua
  được — xem dưới.

> 📐 **Một lỗ hổng thật, tìm ra bằng phép đo chứ không bằng đọc mã.**
> `relativePath: "NUL"` (mọi cách viết hoa thường, và cả `Thu muc/NUL`) mở ra
> thiết bị null của Windows — **ngoài** Destination Root. Byte biến mất, còn
> Snappy trả `201` kèm một biên nhận có hash **khớp**, vì hash tính trên byte của
> dây chứ không đọc lại từ đĩa.
>
> Hàng rào cũ đi từng đoạn **thư mục** rồi tin rằng `CreateNew` lo đoạn cuối.
> Nhưng `CreateNew` trả lời câu *"tên này đã có ai chưa"*, không trả lời *"tên này
> dẫn đi đâu"* — và chỉ câu thứ hai là S11. Nay phép kiểm chứa hỏi lại **sau khi
> mở**, một câu tổng quát thay vì một danh sách tên thiết bị: cùng lượt đo ấy cho
> thấy `CON`, `PRN`, `AUX`, `COM1`, `LPT1`, `CONIN$`, `CLOCK$` đều tạo ra tệp
> **thật trong root**, nên một blocklist vừa thừa vừa thiếu (bài học 158).

## 3. Bộ nhớ — "có streaming thật không"

Đọc mã rồi khẳng định *"nó streaming mà"* **không phải một phép đo**: một
`MemoryStream` tích luỹ trong vòng lặp trông y hệt mã streaming ở mức nhìn lướt.

📐 Đo bằng `GC.GetTotalAllocatedBytes`, khẳng định là **tỉ lệ** chứ không phải một
con số tuyệt đối: ghi thêm 60 MiB làm cấp phát tăng dưới 1/8 con số ấy.

> ⚠️ Hai phép đo cấp phát trong repo này đọc một bộ đếm của **cả tiến trình**, và
> xUnit chạy các lớp test song song. 📐 Phép đo 64 MiB của ticket này chạy cùng
> lúc với phép đo nhả thân của Ticket 05 làm cái sau đỏ với *"cấp phát tăng thêm
> 524 KB"*, trong khi chạy riêng nó xanh 5/5 lần. Cả hai nay nằm chung một
> collection `DisableParallelization`. Dụng cụ đo tự nó làm hỏng phép đo — và ở
> đây thứ làm hỏng nó là **một phép đo khác**.

## 4. Hình dạng trên dây — "byte thật có khớp bản máy-đọc-được không"

`WireShapeTests`, 6 test. Nó đọc `required` và các `pattern` **từ chính tệp
schema** rồi đối chiếu với JSON thật đi qua TLS.

Đây không phải mối lo giả định: bài học 51 của `peekvn` là đúng lớp lỗi ấy, đã
xảy ra. Reference Host ghi `identitySpki`/`tlsSpki` trong khi schema gọi chúng là
`identityPublicKey`/`tlsSPKIHash`, và cái thứ hai lệch cả nội dung — không test
nào đỏ, vì không test nào đối chiếu với bản máy-đọc-được.

Giới hạn, ghi ra để không ai đọc nhầm nó thành một validator: nó bắt *thiếu
trường* và *sai hình dạng giá trị*, không bắt kiểu sai (`"3"` thay vì `3`).

## 5. Bản AOT thật — "thứ người dùng tải về có chạy không"

Bộ test chạy trên runtime thường, nơi `JsonSerializer` còn đường phản chiếu. Bản
publish AOT thì không, và nó **không đỏ lúc build** — nó ném
`NotSupportedException` lúc chạy, tức chỉ lộ ra trên máy người dùng. Ticket này
thêm năm kiểu vào `UltpJson`.

```
snappy-interop push --host 127.0.0.1:8443 --file "anh-mua-he.bin" --len 3145728
```

📐 20/09/2026, một tiến trình thật gõ cửa `Snappy.exe` đã publish:

```
PUSH sas=834995 peer=dev_hHDZJdu4lums5uk-SJt985_NQdvSM1ecLpR6lakBiJE name=NATUAN1
PUSH paired
PUSH authenticated
PUSH offered transfer=01M2YQBHWS99XHRBPVEDVS2RB3 state=OFFERED bytes=3145728
PUSH ok bytes=3145728 sha256=08929205719022ce269762f22fc1c49727e5568a33577029144e8bb7eb244360
```

Nhật ký của Snappy cùng lượt ấy:

```
Đầu dò Snappy muốn gửi 1 mục (3145728 byte) — đang chờ người dùng trả lời
Người dùng đồng ý nhận 1 mục từ dev_kLpUNlQ4…
Đã nhận `anh-mua-he.bin` (3145728 byte) vào C:\Users\tuana\Downloads\Snappy
```

`certutil -hashfile` trên tệp đã ghi cho đúng con số ở trên.

⚠️ Hai hộp thoại được bấm bằng một script (`spikes/bam-co.ps1`). Đó là **công cụ
đo**, không phải nghiệm thu: thứ nó đo là AOT và dây, và cả hai không liên quan gì
tới cử chỉ của con người. Cử chỉ ấy là bài demo iPhone.

## Bảng tổng

| Lớp | Số test | Câu nó trả lời |
|---|---|---|
| `TransferFlowTests` | 23 | tệp có nằm đúng chỗ, đúng từng byte, qua một kết nối thật không |
| `PathSafetyTests` | 50 | lối thoát của Windows có bị chặn không, và tên hợp lệ có đi qua không |
| `TransferControlTests` | 10 | `?wait=`, lọc `direction`/`state`, `EXPIRED` khác `FAILED` |
| `TransferLimitsTests` | 21 | trần tài nguyên, và chúng có chĩa vào người dùng không |
| `WireShapeTests` | 6 | byte thật có khớp `protocol/schemas/` không |
| `MemoryFootprintTests` | 2 | bộ nhớ có đi theo kích thước tệp không |

Tổng bộ test Windows: **341**, xanh 6/6 lượt chạy liên tiếp.

## Số đo bộ nhớ trên bản AOT — và một câu hỏi cho KPI

📐 20/09/2026, `Snappy.exe` publish, đẩy 256 MB ba lượt liên tiếp:

| Lúc | Working set | Private bytes |
|---|---|---|
| Nghỉ | **19,4 MB** | 6,3 MB |
| Sau 8 MiB | 27,6 MB | 8,4 MB |
| Sau 256 MB | 30,6 MB | 11,5 MB |

Bản **trước** khi bộ đệm 64 KB chuyển sang `ArrayPool` cho 30,1 → 34,0 → 34,1 MB
qua ba lượt 256 MB liên tiếp: **không rò** (nó chững ở lượt hai) nhưng vượt red
line 30 MB. KPI *"working set lúc nghỉ < 25 MB"* vẫn đạt và không đổi so với
Ticket 06 (19,1 MB).

Khoảng cách giữa hai cột nói vì sao con số dưới tải sát red line: ~19 MB là
**trang có tệp nền** mà hệ điều hành tính vào working set trong lúc ta ghi xuống
đĩa, không phải bộ nhớ Snappy sở hữu. Private bytes — thứ Snappy thật sự chiếm —
là 11,5 MB, và lúc nghỉ chỉ 6,3 MB.

Ghi ra thành một câu hỏi thay vì một con số: nếu dự án muốn một KPI *"working set
dưới tải"*, nó cần một định nghĩa phân biệt private với file-backed. Không thì
con số ấy đo chính sách cache của Windows, không đo Snappy.

## Cái không đo được từ đây

**Chưa implementation nào khác đẩy tới Snappy.** Đầu dò ở lớp 5 dùng chung
codebase, nên nó chứng minh AOT và dây, không chứng minh interop. Phép đo
`rust-host ↔ windows` là tiêu chí của Ticket 08 ([#139]).

Và không phép đo nào chạm tới câu **người dùng có tìm thấy tệp không**. Bài đo là
`interoperability/manual-ios.md`, mục *"iPhone → Snappy — đẩy một tệp và một thư
mục, Ticket 07"*.

[#139]: https://github.com/natuan1/peekvn/issues/139
