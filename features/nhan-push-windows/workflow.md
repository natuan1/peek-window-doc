# Nhận push — luồng đi qua những đâu

## Toàn cảnh: bốn chặng, một quyết định của con người

```
iPhone (Sender, client)                     Snappy (Receiver, server)
───────────────────────                     ─────────────────────────
TLS 1.3, ghim SPKI đã lưu   ───────────►
POST /v1/auth/challenge     ───────────►
POST /v1/auth/verify        ◄──────────►    session token (30 phút)

POST /v1/transfers          ───────────►    validate TỪNG item:
  mode: push                                  kind · name · relativePath
  items: [ {name, relativePath,                size · mimeType · sha256
            size, sha256} ]                  kiểm dung lượng trống (§23)
                            ◄───────────    201: transferId, itemId, OFFERED
                                             (KHÔNG có token — chưa ai đồng ý)

                                            ┌──────────────────────────────┐
GET /v1/transfers/{id}      ───────────►    │ hộp thoại: ai, bao nhiêu, ở  │
  ?wait=30                                  │ đâu — mặc định "Không"        │
  (request treo ở đây)                      └──────────────────────────────┘
                                                      👤 NGƯỜI DÙNG BẤM 👤

                            ◄───────────    200: ACCEPTED, mỗi item một token
PUT …/items/{itemId}        ───────────►    .partial → hash → rename nguyên tử
  X-ULTP-Item-Token: …                      token của item chết ngay khi xong
                            ◄───────────    201: {itemId, receivedBytes, sha256}
```

Người dùng bấm **một** lần cho cả lượt, không một lần cho mỗi tệp: họ chọn gửi
một lượt, nên câu hỏi phải mang cùng hạt với thứ họ khởi động.

## `?wait=` — Snappy là bản đầu tiên thật sự giữ response lại

iOS đã gửi `?wait=55` từ lâu (`ULTPClient.swift`). Tới hôm nay **chưa server nào
implement nó** — Reference Host bỏ qua tham số này, và SPEC §16.2 cho phép: tham
số lạ MUST được bỏ qua chứ MUST NOT làm fail. Nên fallback nằm sẵn trong một MUST
đã có, và Snappy là bản đầu tiên biến một vòng 240 request thành 1–2 request.

§14.1 định nghĩa "đổi trạng thái" **hẹp**, và MUST chỉ gồm hai thứ:

1. `state` khác lúc bắt đầu chờ, **hoặc**
2. một item bất kỳ **vừa** có `token` — tức bên kia đã accept.

Hết giờ thì **200** với biểu diễn hiện tại, không phải `408`: *"chưa có gì đổi"*
là một câu trả lời đúng, không phải một lỗi.

> ⚠️ **Một hệ quả đã biết, và nó không phải lỗi ở đây.** Client gọi `?wait=` *sau*
> khi đã cầm token sẽ chờ trọn cửa sổ — thay đổi nó đang hỏi đã xảy ra **trước**
> khi request được gửi. Bài học 135 của `peekvn` đo đúng hình dạng ấy hai lần:
> 55 giây đứng im, **không phụ thuộc kích thước tệp**. Chỗ sửa nằm ở client, và
> luật là *chỉ đọc token ở nhánh nối lại*.

## Đường ghi: `.partial` → verify → đổi tên

```
.anh.bin.partial  →  nhận byte, băm trong cùng lượt  →  so size, so hash  →  anh.bin
```

Ba điều kiện, cả ba đều từ đột biến chứ không từ suy luận (SECURITY.md §5.11):

- **`.partial` đi qua đúng hàng rào của tên cuối.** Tự nối chuỗi thì nó nằm ngoài
  phép kiểm chứa, ngoài `CREATE_NEW`, ngoài chống trùng tên. Trên bản Rust, lôi
  nó ra khỏi hàng rào làm **12/13** test vẫn xanh.
- **Tên cuối không xuất hiện trước khi hash khớp** — kể cả dưới dạng tệp 0 byte.
  Trên Windows không có cả bước giữ chỗ: `MoveFileW` **không** có
  `MOVEFILE_REPLACE_EXISTING` vừa tạo tên cuối vừa hỏng nếu tên ấy đã có, trong
  một lời gọi nguyên tử.
- **Dở dang thì bị dọn.** SPEC §22.1 đã đo được 269,8 MB nằm lại sau ba lần huỷ.

## Trên Windows, lối thoát rẻ hơn trên POSIX

📐 Đo 20/09/2026, Windows 11 26200, .NET 10 — ba hành vi mà một luật hình dạng
POSIX không phủ:

| Đo được | Hệ quả |
|---|---|
| `Directory.CreateSymbolicLink` ném *"A required privilege is not held"*, nhưng **`mklink /J` chạy** không cần quyền gì | Junction là lối thoát một người dùng thường dựng được. Ghi xuyên nó thì tệp nằm **ngoài** root, và `Path.GetFullPath` không thấy gì — nó thuần từ vựng |
| `FileMode.CreateNew` trên `a.txt:evil` **thành công** khi `a.txt` đang tồn tại, nội dung nhìn thấy được không đổi | Alternate Data Stream: byte của người lạ nằm **bên trong** một tệp có sẵn, và `O_EXCL` không thấy gì. SECURITY.md §7.6 chỉ soi `:` ở **đoạn đầu** |
| `CreateNew` cho `"ten. "` tạo ra tệp tên `ten` | Win32 cắt đuôi chấm/khoảng trắng im lặng ⇒ đoạn `".. "` đi lọt phép kiểm cú pháp rồi **thành `..`** lúc mở tệp |

Nên hàng rào ở đây là hai bước, hai chuỗi, và thứ tự là bắt buộc:

1. **Kiểm cú pháp trên chuỗi đúng như nhận được** (§19.1 — MUST NOT chuẩn hoá
   trước bước này: NFKC biến `．．／` fullwidth thành đúng `../`).
2. **Gọt cho vừa NTFS** (§19.2 — MUST, vì một tên hợp lệ trên POSIX MUST NOT làm
   fail cả Transfer nếu gọt được an toàn), rồi **kiểm lại** kết quả: bước gọt tự
   nó đẻ được ra `..`.

Từ đó trở đi chỉ chuỗi đã gọt được dùng, nên phép kiểm chứa và phép mở tệp nhìn
**cùng một chuỗi** — §19.1 nói chuẩn hoá ở giữa hai bước là cách tạo ra lỗ hổng
từ hai đoạn mã đều đúng.

Và phép kiểm chứa hỏi **hệ tệp** (`ResolveLinkTarget`), ở **từng đoạn**: một
`relativePath` không hề có `..` vẫn ghi ra ngoài nếu một đoạn giữa đường là
junction, và kiểm một lần ở cuối thì thư mục trung gian đã kịp mọc ra rồi.

## Tự nhận — SECURITY.md §4.2

*"`autoAccept` bỏ qua bước hỏi người dùng, và **chỉ** bước đó."*

Mọi thứ khác chạy y nguyên, và chúng chạy **trước** chỗ rẽ nhánh: kiểm dung lượng,
validate đường dẫn, gọt tên đều đã xảy ra lúc tạo Offer; chống trùng tên và verify
hash nằm ở đường ghi. Nên không có nhánh nào để quên chúng — đó là cấu trúc,
không phải kỷ luật.

Hai quyết định về giao diện, cả hai đến từ §4.1 (*"nút đó tồn tại vì nó tiện, rồi
trở thành thứ người dùng bấm theo phản xạ"*):

- **Không** có nút "luôn nhận từ thiết bị này" trên hộp thoại nhận. Đường bật nó
  nằm ở menu khay, tách hẳn khỏi lúc đang vội.
- Mục ấy **không** nằm sát "Thoát Snappy" — có một vạch ngăn, và có một test
  khẳng định về bố cục ấy.

## Ba đường hỏng, và cái Sender thấy

| Hoàn cảnh | Trả về | Vì sao không phải mã khác |
|---|---|---|
| Thiếu byte so với `size` đã khai | `409 SIZE_MISMATCH` | §21: MUST NOT báo `HASH_MISMATCH` — trên Offer khai `sha256: null` chưa có phép so hash nào chạy, và Sender sẽ đi kiểm lại tệp nguồn thay vì kiểm đường truyền |
| Nội dung sai | `409 HASH_MISMATCH` | không có tệp đích, và cũng không còn `.partial` |
| Thân đứt giữa chừng | `400 NETWORK_INTERRUPTED`, `retryable: true` | đáng thử lại, khác hẳn hai cái trên |
| Lỗi ghi đĩa | `500 INTERNAL_ERROR` | MUST NOT đoán "hết dung lượng" từ một lỗi IO bất kỳ — permission, rename hỏng và đĩa đầy là ba chuyện |

Một mục hỏng **MUST NOT** giết cả Transfer (§16.1): mục ấy vào `FAILED`, các mục
khác vẫn chạy tiếp, và Transfer không đóng.
