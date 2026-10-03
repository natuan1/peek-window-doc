# Lessons Learned

Tài liệu này ghi lại những bài học khi kế hoạch được thực hiện đúng nhưng kết quả không như mong muốn.

## Mục đích

Khi một tính năng được triển khai **đúng theo kế hoạch** nhưng **kết quả khác biệt với kỳ vọng**, chúng ta ghi lại tại sao điều này xảy ra.

Đây không phải là các lỗi cần sửa, mà là những **phát hiện bất ngờ** giúp cải thiện hiểu biết về dự án.

## Cấu trúc

Mỗi mục ghi bài học theo định dạng:

```markdown
## [YYYY-MM-DD] Tiêu đề tính năng / bài học

**Kế hoạch**: Điều gì được dự định sẽ xảy ra?

**Kết quả thực tế**: Điều gì thực sự xảy ra?

**Bài học**: Tại sao có sự khác biệt? Giả định nào của chúng ta sai?

**Hành động tiếp theo**: Cần làm gì để cải thiện?
```

## Ví dụ

```markdown
## [2026-09-13] Tính năng X hoàn thành nhưng chỉ số không cải thiện như mong đợi

**Kế hoạch**: Triển khai tính năng X, dự kiến sẽ cải thiện metric Y thêm 30%

**Kết quả thực tế**: Tính năng hoạt động đúng, nhưng metric Y chỉ tăng 5%

**Bài học**: Giả định của chúng ta về hành vi người dùng không chính xác. 
Người dùng không sử dụng tính năng X vì [lý do]. 
Chúng ta cần hiểu rõ hơn nhu cầu thực tế của họ.

**Hành động tiếp theo**: Nghiên cứu vì sao người dùng không dùng tính năng này
```

---

*Được cập nhật từ: CLAUDE.md → Agent Communication Rules → Lessons Learned Protocol*

---

## [2026-09-15] Icon khay chạy đúng, nhưng người dùng Windows 11 không thấy nó

**Kế hoạch**: Ticket 01 hứa "tray icon: mở bảng trạng thái nhỏ". Giả định ngầm là icon xuất hiện ở khay thì người dùng thấy và bấm được — đó là toàn bộ giao diện của app ở giai đoạn này.

**Kết quả thực tế**: Mã chạy đúng — `Shell_NotifyIcon(NIM_ADD)` trả thành công, icon tồn tại thật. Nhưng khi chụp lại vùng khay hệ thống để kiểm chứng thì **không thấy icon Snappy đâu**: Windows 11 mặc định giấu mọi icon khay mới vào phần tràn sau dấu `^`, và người dùng phải tự bấm mở rồi kéo nó ra ngoài.

**Bài học**: "API trả thành công" và "người dùng thấy" là hai chuyện khác nhau, và khoảng cách giữa chúng nằm ở một mặc định của hệ điều hành mà không có API nào vượt qua được (Microsoft cố ý không cho app tự bỏ ẩn). Với người dùng, app vừa cài mà khay không có gì nghĩa là **app chưa tồn tại** — đúng câu hỏi số 1 của AGENTS.md §7.1: "người dùng tới đây bằng cách nào?". Nếu không có phép kiểm bằng mắt, lỗi này sẽ đi hết Ticket 01→16 mà không ai phát hiện, vì mọi test tự động đều xanh.

**Hành động tiếp theo**: [Ticket 17 (Onboarding)](https://github.com/natuan1/peekvn/issues/148) phải coi "dạy người dùng ghim icon Snappy ra khay" là một bước bắt buộc, không phải mẹo phụ — kèm ảnh chỉ đúng dấu `^`. Và mọi ticket có bề mặt nhìn thấy được từ đây về sau phải nghiệm thu bằng ảnh chụp thật, không bằng mã trả về.

---

## [2026-09-16] Hàng rào gói vá delta xanh suốt — vì nó đo một thứ không thay đổi

**Kế hoạch**: Ticket 02 hứa *"Cập nhật 1.0.0 → 1.0.1 chỉ tải gói vá nhỏ"*. Kế hoạch là dựng một hàng rào CI tự động chứng minh câu đó: đóng gói bản 1.0.1 trên nền thư mục phát hành đã có 1.0.0, rồi kiểm gói vá có tồn tại, có được kênh phát hành liệt kê, và có nhỏ hơn một tỉ lệ trần so với gói đầy đủ.

**Kết quả thực tế**: Hàng rào chạy đúng từng bước của kế hoạch và xanh ngay lần đầu, với một con số đẹp tới mức đáng ngờ — gói vá **11,5 KB = 0,2 %** gói đầy đủ. Mã đúng, kế hoạch đúng, kết quả xanh. Nhưng con số ấy không chứng minh gì cả: script đóng gói 1.0.1 từ **đúng** bộ nhị phân của 1.0.0, nên thứ nó đo là gói vá của *không có gì thay đổi*. Một cơ chế delta đã hỏng hoàn toàn cũng cho đúng con số đó, và cái trần 25 % thì không bao giờ đỏ được.

**Bài học**: Giả định sai nằm ở chỗ tưởng rằng "gói vá nhỏ" là một câu về **gói vá**. Nó là một câu về **quan hệ** giữa gói vá và phần đã thay đổi — mà một phép đo không có phần thay đổi nào thì không có quan hệ nào để đo. Dạng chung: một hàng rào chỉ có **chặn trên** sẽ xanh cả khi đại lượng nó đo bằng không, và "bằng không" thường trùng khít với "phép đo chưa xảy ra". Đây là bản CI của bài học "Icon khay chạy đúng, nhưng người dùng Windows 11 không thấy nó" ở ngay trên: API trả thành công ≠ người dùng thấy; hàng rào xanh ≠ hàng rào đã đo.

Điều đáng chú ý là hàng rào này đã được ép đỏ trước khi tin — nhưng ép đỏ bằng cách hạ trần xuống 10 %, tức vẫn chỉ chứng minh cái trần hoạt động, không chứng minh phép đo có ý nghĩa.

**Hành động tiếp theo**: `ci/check-delta.ps1` giờ **tự dựng phần đã thay đổi** (chép thư mục publish ra chỗ khác, nhét thêm 1 MB dữ liệu ngẫu nhiên) và có thêm **chặn dưới**: gói vá phải mang nổi ít nhất một nửa phần đã đổi. Đo lại: 1,01 MB = 15 % gói đầy đủ. Chính con số 11,5 KB cũ là thứ chặn dưới mới bắt được.

Luật rút ra cho mọi hàng rào sau: **mỗi hàng rào phải trả lời được câu "nếu thứ tôi đo chưa hề xảy ra thì tôi màu gì?"** Nếu câu trả lời là "xanh", nó cần một chặn dưới trước khi được tin.

---

## [2026-09-17] Harness "chạy được ở mọi nơi" — và nó chưa từng chạy trên Windows

**Kế hoạch**: Ticket 03 chỉ cần hai thứ: một CLI trình ra đúng mặt cắt dòng
lệnh, và vài dòng trong `run.sh` để gọi nó. `interoperability/README.md` viết
sẵn rằng cặp `fixture` "chạy được ở mọi nơi", nên phần harness trông như một
việc nửa giờ.

**Kết quả thực tế**: phần CLI đúng là nửa giờ và khớp byte với Rust ngay lần
đầu. Phần harness thì không — vì chưa ai từng chạy `run.sh` trên Windows, và ba
thứ khác nhau đổ ra cùng lúc:

1. `run.sh` dựng **cả hai** phía vô điều kiện, nên trên Windows nó chết ở
   `swift build` trước khi tới cặp nào.
2. Cặp `vectors` đỏ cả 43 dòng vì `core.autocrlf` — không byte nội dung nào
   lệch, chỉ là `\r\n` gặp `\n`.
3. Lỗi đỏ ấy lộ tiếp một lỗi thứ ba đã ngủ từ lâu: ghi chú nhiều dòng làm
   `run.sh` đếm một FAIL thành bảy.

Và sau khi tất cả đã xanh, CI vẫn đỏ — vì một `mode` symlink sai trong index từ
một commit tài liệu hôm trước, thứ không dính dáng gì tới ticket này.

**Bài học**: giả định sai nằm ở chỗ đọc câu "chạy được ở mọi nơi" trong tài liệu
như một **phép đo**, trong khi nó là một **dự định**. Một câu trong README nói
về các nền tảng chưa ai thử là một lời hứa, không phải một kết quả — và lời hứa
ấy càng dễ tin khi chính người viết nó cũng tin.

Dạng chung: mỗi lần một dự án chạy lần đầu trên một nền tảng mới, cái đổ vỡ
không phải tính năng đang làm mà là **hạ tầng quanh nó** — dòng lệnh, ký tự
xuống dòng, phân biệt hoa thường, quyền tệp. Ba lỗi ở trên đều thuộc loại đó, và
không lỗi nào nằm trong phạm vi ticket.

**Hành động tiếp theo**: `.gitattributes` ghim `eol=lf` cho mọi tệp được so theo
byte giữa hai công cụ; `run.sh` dò toolchain và in `BỎQUA` kèm lý do thay vì
giả định; và ba bài học (166, 167, 168) trong `peekvn/docs/bai-hoc.md`. Ticket
sau nào đưa Snappy vào các cặp `transfer`/`tls-handshake` nên tính trước một
khoản cho tầng hạ tầng này, không chỉ cho mã giao thức.

---

## [2026-09-18] Kế hoạch là "làm giống bản Rust" — và đúng chỗ giống nhất là chỗ hỏng

**Kế hoạch**: Ticket 04 mở discovery cho Windows. Bản Rust đã làm việc này rồi và
làm kỹ — instance name mang dấu Device, hostname riêng `peek-<dấu>.local`, tự
canh địa chỉ khi máy đổi IP. Kế hoạch là bê nguyên hình dạng ấy sang Windows,
chỉ đổi thư viện mDNS bên dưới.

**Kết quả thực tế**: phần bê nguyên chạy đúng như kế hoạch — và **bên kia không
resolve nổi**. Bản ghi PTR ra dây đầy đủ, tên hiện lên trong mọi công cụ duyệt,
SRV có host có cổng, nhưng không có bản ghi A nào cho `peek-<dấu>.local`, nên
không ai gõ cửa được. Truyền IPv4 tường minh cho API cũng không đổi gì: responder
của Windows **chỉ** giữ bản ghi A cho tên máy của chính nó. Đổi sang
`<tên máy>.local` là resolve đủ ngay lượt đầu, kèm cả IPv6.

**Bài học**: giả định sai nằm ở chỗ coi "cùng giao thức" là "cùng cách làm". mDNS
là một giao thức, nhưng *ai giữ bản ghi nào* là chính sách của từng
implementation — và ở đây ta không tự giữ bản ghi, ta đi nhờ HĐH giữ hộ. Một
tham số API nhận vào (`pIp4`) mà không quảng bá ra là hoàn toàn hợp lệ với người
viết nó, chỉ vô lý với người đọc chữ ký hàm như một lời hứa.

Dạng chung, và nó rộng hơn mDNS: **khi chuyển một thiết kế sang nền tảng mới, thứ
đổ vỡ không phải phần khó — nó là phần đã làm xong ở nền tảng cũ**, vì đó đúng là
phần không ai nghĩ phải đo lại. Ticket 03 đã nói gần y hệt về hạ tầng quanh mã;
lần này là về chính mã.

Một hệ quả nhỏ mà đắt: vì hostname không còn mang dấu Device, chỗ duy nhất phân
biệt hai máy trùng tên là instance name — và vì tập địa chỉ do HĐH chọn (gồm cả
IPv6), server của Ticket 05 buộc phải nghe dual-stack. Cả hai ràng buộc ấy sinh
ra từ đúng một phép đo.

**Hành động tiếp theo**: [ADR-0008](adr/0008-discovery-qua-responder-in-box-windows.md)
ghi quyết định và bảng đo; ba vòng đo giữ ở nhánh `prototype/mdns-dnsapi` của
`peekvn`; bài học 169 của `peekvn/docs/bai-hoc.md` ghi dấu hiệu nhận biết. Và một
luật cho mọi phép đo mạng về sau: **luôn chạy một phép đối chứng cùng lúc** —
trong ticket này, hai lần một kết quả rỗng suýt bị đọc thành "mã hỏng" trong khi
thủ phạm là cái thước.

---

## [2026-09-19] Kế hoạch là "dựng khung server" — và hai thứ tốn nhất không phải khung

**Kế hoạch**: Ticket 05 dựng khung server ULTP cho Snappy: `TcpListener` +
`SslStream`, router HTTP/1.1 tự viết, `GET /v1/info`. Phần khó dự kiến nằm ở bộ
phân tích HTTP tự viết — chỗ duy nhất phải đúng theo RFC mà không có thư viện
đỡ.

**Kết quả thực tế**: bộ phân tích đúng là phần dễ kiểm nhất — mỗi luật RFC là
một dòng `[InlineData]`, và nó xanh gần như ngay. Hai thứ tốn nhất nằm ngoài
tầm nhìn của kế hoạch, và cả hai đều là **một tầng bên dưới thứ đang viết**:

1. **SChannel không nhận certificate mang khoá tạm.** `CreateSelfSigned` trả về
   một certificate ký được ngay trong tiến trình, nên mọi phép thử ở tầng mã đều
   nói nó tốt. Nhưng SChannel tìm khoá riêng qua thuộc tính
   `CERT_KEY_PROV_INFO` của certificate, không qua đối tượng .NET đang cầm nó —
   và câu lỗi nó ném ra (*"The credentials supplied to the package were not
   recognized"*) không nhắc một chữ nào tới khoá.

2. **`System.Uri` nuốt zone của IPv6.** 📐 `new Uri("https://[fe80::1%7]:…")`
   cho ra host `fe80::1`. Không ngoại lệ, không cờ nào bảo nó giữ. Mà 4 trong 5
   địa chỉ Snappy quảng bá là IPv6, và máy dev có ba giao diện mạng.

Và một thứ thứ ba, không phải lỗi mà là một cái giá không ai tính: **một dòng
`new HttpClient()` trong constructor ăn 3,0 MB working set** — hơn một nửa biên
còn lại của KPI 25 MB, tiêu cho một máy chưa nghe thấy hàng xóm nào.

**Bài học**: giả định sai nằm ở chỗ đo "độ khó" bằng **lượng mã phải viết**.
Bộ phân tích HTTP là 280 dòng mã của ta, nên nó *trông* khó — và chính vì nó là
mã của ta nên nó cũng là phần duy nhất ta kiểm được trọn vẹn. Ba chỗ đắt ở trên
đều là **một dòng gọi vào thứ người khác viết**, nơi chữ ký hàm là tất cả những
gì ta thấy, và hành vi thật chỉ lộ ra lúc chạy.

Dạng chung, và nó nối thẳng với bài học của Ticket 04 (*"thứ đổ vỡ không phải
phần khó — nó là phần đã làm xong ở nền tảng cũ"*): **rủi ro không tỉ lệ với
lượng mã, nó tỉ lệ với số biên giới đi qua.** Mỗi lời gọi ra ngoài là một chỗ
lời hứa (chữ ký hàm, tài liệu) có thể lệch khỏi hành vi, và không phép đọc mã
nào phát hiện được — chỉ một phép chạy thật.

Một hệ quả nhỏ mà đáng giữ: cả ba đều bị bắt bởi **cùng một loại phép thử** —
chạy thứ thật, đo bằng công cụ bên ngoài. Certificate lộ ra ở lần
`AuthenticateAsServerAsync` đầu tiên trong test; zone IPv6 lộ ra vì có một
assertion viết ra để *khẳng định nó còn nguyên* (và nó đỏ ngay); 3 MB lộ ra vì
`Get-Process` trên bản AOT thật, không trên `dotnet run`.

**Hành động tiếp theo**: [ADR-0009](adr/0009-tls-1-3-ghim-cung-thu-hep-san-he-dieu-hanh-thuc-te.md)
ghi quyết định TLS 1.3 ghim cứng và hệ quả về sàn HĐH;
[ADR-0010](adr/0010-bang-nang-luc-khai-theo-hanh-vi-khong-theo-lo-trinh.md) ghi
luật khai capability theo hành vi; bài học **171** và **172** của
`peekvn/docs/bai-hoc.md` ghi dấu hiệu nhận biết của hai chỗ trên. Và một luật
cho các ticket sau: **mỗi thư viện hoặc API hệ thống mới đưa vào phải kèm một
con số RAM đo trên bản AOT thật**, ghi vào bảng số đo của
`apps/windows/README.md` — biên tới KPI giờ chỉ còn ~3 MB sau khi có kết nối.

---

## [2026-09-30] Mí làm đúng kế hoạch — và đúng câu cơ chế trong kế hoạch là câu không chạy

**Kế hoạch**: Ticket 10 dựng Mí đúng như `CONTEXT.md` đã chốt: `SysDragImage` làm tín hiệu chính, xác nhận "đang kéo FILE" bằng `OleGetClipboard` + `CFSTR_INDRAGLOOP`, Composition để vẽ. Phần rủi ro lớn nhất được coi là đã gỡ, vì spike AOT (ADR-0002) đã chứng minh Composition sống được dưới Native AOT và vừa KPI RAM.

**Kết quả thực tế**: trước dòng mã đầu tiên, một đầu dò kéo thật một tệp từ Explorer. `SysDragImage` đúng như tài liệu viết. **Clipboard thì mù**: `DV_E_FORMATETC` cho mọi định dạng, y hệt lúc không kéo gì. Mí xây xong, đạt năm kịch bản kéo thật. Nhưng sau lượt kéo đầu tiên, RAM nền lên **26,7 MB**, vượt KPI 25 MB mà spike từng cho là còn rộng.

**Bài học**: giả định sai nằm ở chỗ coi hai thứ đã "chốt" là hai phép đo.

- Câu `OleGetClipboard` trong `CONTEXT.md` chưa ai chạy. Nó là một ý tưởng nghe hợp lý, được ghi cạnh những quyết định đã đo nên mượn luôn độ tin của chúng.
- Spike ADR-0002 đo Composition trên một **exe trống** (14,6 MB). Nó không đo Composition **cộng** chín ticket đã đắp lên nền. Một spike chứng minh *khả thi*, không chứng minh *vừa ngân sách lúc tới lượt*.

Và hàng rào RAM của CI đo lúc khởi động, tức trước khi thứ đắt nhất của ticket kịp xảy ra, nên nó xanh suốt.

**Hành động tiếp theo**:

- Xác nhận tệp chuyển sang `IDropTarget::DragEnter` ([ADR-0013](adr/0013-mi-an-khi-nghi-dung-luoi-xac-nhan-tep-qua-dragenter.md)).
- `CONTEXT.md` gạch câu sai và ghi số đo cạnh nó.
- Xung đột KPI ↔ Composition giao cho chủ dự án ở [natuan1/peekvn#181](https://github.com/natuan1/peekvn/issues/181).
- Ba bài học 192–194 ở `peekvn/docs/bai-hoc.md`.
- Luật cho các ticket sau: mọi câu **cơ chế** trong kế hoạch phải kèm một số đo, hoặc bị đánh dấu là chưa đo. Mọi phép "hoãn tới lần dùng đầu" phải có một phép đo **sau** lần dùng đầu.

**Kết cục (cùng ngày)**: chủ dự án chọn bỏ Composition cho Mí và vẽ bằng layered window ([ADR-0014](adr/0014-mi-ve-bang-layered-window-khong-composition.md)).

- Sau lượt kéo đầu tiên RAM còn 23,3 MB, dưới KPI.
- Bộ cài quay về 10,84 MB.
- Có một bẫy mới: pixel alpha 0 của layered window là click-through, và OLE không coi chỗ đó là đích thả. Hàng rào cũ khẳng định "không có `WS_EX_LAYERED`", tức là canh một cờ chứ không canh hành vi. Nó phải được thay bằng một phép `WindowFromPoint` (bài học 195 của `peekvn`).

---

## [2026-10-01] Metadata qua `SHGetFileInfoW` đúng như ticket ghi — và nó là thứ đắt nhất của cả ticket

**Kế hoạch**: Ticket 11 ([natuan1/peekvn#142](https://github.com/natuan1/peekvn/issues/142)) ghi *"Metadata + icon qua `SHGetFileInfoW`: tên, dung lượng, MIME type chính xác"*. Bản đầu làm đúng thế: mỗi Mục hỏi shell tên loại ngay lúc thả. Lời gọi không mở tệp, chỉ tra theo đuôi — tưởng là rẻ.

**Kết quả thực tế**: Mọi tệp vào đúng tên, đúng byte (SHA-256 trùng cho Explorer, zip, Chrome, Edge), nhưng sau bốn cú thả RAM nền **26,5 MB**, vượt KPI 25 MB. Đối chứng chỉ bỏ đúng lời gọi shell: **24,2 MB**. Lời gọi ấy nạp năm DLL của kho AppX và không bao giờ nhả chúng.

**Bài học**: Một API "chỉ tra registry" trên Windows 11 có thể kéo theo cả một tầng WinRT. Kế hoạch gom "metadata" và "icon" vào một câu như thể chúng cùng giá và cùng lúc cần, trong khi thứ người dùng *cần ngay lúc thả* chỉ là tên, dung lượng, MIME. Tên loại và icon chỉ có người xem khi khay thẻ vẽ chúng.

**Hành động tiếp theo**: Lúc thả không hỏi shell ([ADR-0015](adr/0015-tempdrops-mot-thu-muc-moi-muc-khong-hoi-shell-luc-tha.md)); `ShellFileInfo` để dành cho Ticket 12. **Ticket 12 phải đo RAM sau lần gọi shell đầu tiên trước khi coi xong**, vì biên còn ~0,6 MB. Kèm theo: menu khay tự nó đã đẩy RAM lên 25,5 MB trên `main` — chưa hàng rào nào đo sau cử chỉ ấy.

---

## [2026-10-01] KPI RAM "< 25 MB" chưa từng nói thước nào — và thước tổng đếm cả DLL của Windows

**Kế hoạch**: sửa lỗi menu khay đẩy RAM lên 25,5 MB, rồi mới làm Ticket 12. Tin rằng đó là lỗi cuối cùng giữa app và KPI.

**Kết quả thực tế**: sửa được bằng `ImmDisableIME` cho luồng UI (25,5 → 22,3 MB). Nhưng người đã kéo thả **và** mở menu vẫn ở 25,1–25,8 MB, vì OLE và menu cộng dồn. Đo thêm thước private ở cùng các mốc: chỉ 4,7–5,5 MB. Khoảng 20 MB của KPI là trang DLL hệ thống dùng chung.

**Bài học**: một KPI hiệu năng phải ghi rõ thước đo ngay từ ngày đặt ra. Sáu ticket liền đếm biên từng 0,1 MB theo một thước đếm cả thứ không phải của app. Hai quyết định thiết kế (bỏ Composition, dời `SHGetFileInfoW`) được chốt theo thước ấy. Spike 14/09 đã ghi cả hai con số, chỉ là không ai chọn.

**Hành động tiếp theo**: chủ dự án chốt private working set < 25 MB, red line 30 MB theo thước tổng ([ADR-0016](adr/0016-kpi-ram-nen-do-bang-private-working-set.md)). Một định nghĩa duy nhất ở `peekvn/apps/windows/ci/ram.ps1`. Hai quyết định cũ vẫn đúng theo red line, không cần mở lại.


## [2026-10-02] Shelf kéo ra Explorer đúng byte ngay lượt đầu — và lượt kéo ấy là 17 MB không bao giờ nhả

**Kế hoạch**: khay thẻ Ticket 12 là một cửa sổ nữa trong tiến trình nền, như Mí. Nguồn kéo là `IDataObject` tự viết với `CF_HDROP`, nhẹ hơn nguồn kéo của shell.

**Kết quả thực tế**: kéo ra đúng byte, nhưng lượt kéo đầu tiên đẩy working set tổng 25,3 → 42,8 MB. Windows 11 nạp 18 DLL kéo-thả hiện đại (`d3d11`, `dcomp`, `datatransfer`…) cho **mọi** nguồn kéo, kể cả `DoDragDrop` thuần của `ole32`. Định chuyển riêng `DoDragDrop` sang tiến trình con thì con đứng chờ mãi: OLE chỉ bắt chuột cho cú nhấn rơi vào cửa sổ của chính luồng gọi nó. Cùng tuần, hộp chọn tệp có sẵn từ Ticket 09 cũng lộ ra 63 MB.

**Bài học**: với Windows, "API nhẹ" không có nghĩa là "DLL nhẹ". Giá RAM của một tính năng đọc ở bản publish sau lần dùng đầu, không đọc ở tên API. Và một DLL đã nạp thì chỉ có một cách bỏ đi: để tiến trình chứa nó chết.

**Hành động tiếp theo**: tính năng nặng phần shell chạy trong tiến trình Snappy con. Cả khay (vì ràng buộc chuột của OLE), cả hộp chọn tệp, dùng chung `ChildProcess` ([ADR-0017](adr/0017-tinh-nang-nang-phan-shell-chay-trong-tien-trinh-con.md)). Biên theo red line tổng chỉ còn ~1,7 MB, nên ticket sau phải đo RAM sau lần dùng đầu ngay từ bản publish đầu tiên.

---

## [2026-10-03] Đích thả làm đúng kế hoạch — và ba phép đo của chính tôi đo sai thứ trước khi đo đúng

**Kế hoạch**: Ticket 13 ([natuan1/peekvn#144](https://github.com/natuan1/peekvn/issues/144)) — Mí vẽ Đích thả, thả lên avatar là gửi qua đường pull của Ticket 09, pill báo tiến trình, đường hỏng có câu rõ ràng. Phần khó dự kiến là vẽ chữ trên layered window (GDI không biết alpha).

**Kết quả thực tế**: phần dự kiến khó xong trong một buổi — chữ đi qua mặt nạ độ phủ, kiểm bằng ảnh dựng thật trước khi đụng app. Cái tốn thời gian là **dụng cụ đo**, ba lần:

1. Trust Store đọc từ shell của agent là bản ảo hoá MSIX cũ (một thiết bị) — Snappy thật có hai, rồi ba.
2. Điểm thả "giữa dải" của hai kịch bản cũ — "vào Shelf" từ Ticket 11 — trở thành avatar thứ ba khi Trust Store có ba máy. Kịch bản sẽ gửi tệp thử tới điện thoại.
3. "Đóng app iPhone giữa chừng" không cắt được gì: tải về sống trong `nsurlsessiond`. Pill nói đúng "Đang gửi · 26%", còn kịch bản đỏ vì đòi một cú đứt chưa hề xảy ra.

Và một phát hiện ngoài phạm vi: bấm "Có" ở hộp thoại ghép đôi chậm 30 giây làm Windows tin iPhone trong khi iPhone không tin Windows.

**Bài học**: khi tính năng nằm giữa hai máy và dữ liệu của người dùng, **ý nghĩa** của một phép đo đổi theo thứ nằm ngoài mã — Trust Store thật, số thiết bị, cách iOS chạy tải về. Cả ba lần, câu hỏi cứu được là "phía bên kia ghi gì?": nhật ký thật của Snappy, dòng `Drop targets:` với toạ độ thật, dòng `interrupted` của server.

**Hành động tiếp theo**: bài học 204–207 của `peekvn/docs/bai-hoc.md`; kịch bản đọc toạ độ đích từ nhật ký thay vì tự tính; red line RAM tổng (+0,9 MB của ticket, mốc nền trôi 2,5 MB trong mười phút) và bề rộng pill là hai câu hỏi cho chủ dự án; lỗi ghép đôi nửa vời tách thành việc riêng.

---

## [2026-10-03] Capsule media chạy trong tiến trình con — mà tiến trình nền vẫn trả 1,1 MB cho nó

**Kế hoạch**: Ticket 15 đặt toàn bộ capsule (GSMTC, WIC, GATT) trong một tiến trình Snappy con, đúng như ADR-0017. Tiến trình nền không gọi một dòng WinRT nào, nên RAM của nó không được đổi. TFM đã có sẵn projection CsWinRT, nên viết `using Windows.Media.Control` là xong phần interop.

**Kết quả thực tế**: Capsule chạy đúng, nhưng đối chứng xen kẽ với `main` cho tiến trình nền **+1,1 MB private, +2,6 MB tổng**, kèm một DLL mới là `oleaut32`. Một bản thăm dò không khởi động con cũng cho đúng con số ấy. Nguyên nhân là `WinRT.Runtime` và projection SDK mang `[ModuleInitializer]`, và Native AOT chạy mọi module initializer của ảnh exe lúc khởi động. Cùng bản ấy còn ném `InvalidCastException` khi truyền `List<string>` vào WinRT. Viết lại GSMTC/WIC bằng vtable tay và pin bằng Win32 thì tiến trình nền về sát `main` (không DLL mới), exe nhỏ lại 1,6 MB, và con cũng nhẹ hơn (34,9 → 27,8 MB tổng khi phát).

**Bài học**: "Tách sang tiến trình khác" chỉ cô lập được thứ **chạy** ở tiến trình ấy. Nó không cô lập được thứ **có mặt** trong exe dùng chung. Với một exe AOT dùng cho cả cha lẫn con, cái giá của một thư viện tính theo sự có mặt, không theo lời gọi. Câu hỏi đúng không phải "ai gọi nó", mà là "nó có mang code chạy lúc nạp không".

**Hành động tiếp theo**: [ADR-0018](adr/0018-capsule-media-tien-trinh-con-winrt-bang-vtable-tay.md) cấm projection CsWinRT trong mọi project sinh ra `Snappy.exe`. Mọi tính năng chạy trong tiến trình con từ nay đối chứng RAM **tiến trình nền** với `main`, kèm danh sách DLL, không chỉ đo tiến trình con. Ghi ở `peekvn/docs/bai-hoc.md` §209.
