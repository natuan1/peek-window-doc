# Trích xuất tệp thật/ảo → TempDrops — quy trình

## Một cú thả

```
OLE DragEnter   → Mí nhớ "lượt này mang tệp" (CF_HDROP / FileGroupDescriptorW)
OLE DragOver    → trả DROPEFFECT_COPY (cắt theo hiệu ứng nguồn cho phép)
OLE Drop        → DropIntake.Take(payload)   ← đồng bộ, ngay trong lời gọi Drop
                    ├─ có CF_HDROP?          → Mục trỏ đường dẫn gốc
                    └─ không, có tệp ảo?     → mỗi phần tử cấp đầu một TempDrops\{Guid}\
                                               chép FileContents theo khối 64 KB
                → Shelf.Add(items)
                    └─ TempDrops.EnforceCap  → bỏ Mục LRU nếu vượt 2 GiB → bong bóng
                → Mí về Nghỉ
```

**Vì sao đồng bộ:** OLE chỉ hứa đối tượng dữ liệu sống trong lời gọi `Drop`; Outlook
thu nó lại ngay sau đó. Cái giá: một tệp ảo lớn giữ Mí đứng yên trong lúc chép. Dòng
`Extracted … in N ms` ghi lại cái giá ấy (📐 ảnh 547 KB từ Chrome: 3 ms).

**Tệp thật thắng tệp ảo.** 📐 Explorer trao *cả* `CF_HDROP` lẫn `FileGroupDescriptorW`
cho tệp thường; thư mục zip chỉ trao `FileGroupDescriptorW`; Chromium trao
`FileContents` bằng `HGLOBAL|IStream`.

**Thư mục trong zip là một Mục**, không phải một Mục cho mỗi tệp con: phần tử lồng
(`hop\a.txt`) đi vào thư mục TempDrop của tổ tiên cấp đầu.

## Xoá sạch theo yêu cầu

```
chuột phải icon khay (Windows 11: trong vùng tràn sau dấu ^)
  → "Xoá sạch Shelf (n mục)"   ← chỉ hiện khi Ngăn có Mục
  → Shelf.Clear → TempDrops.Clear
  → còn tệp đang mở ở app khác? → bong bóng "Chưa xoá hết Shelf — đóng chúng rồi chọn lại"
```

## Thoát và khởi động

- `RequestExit` → sau khi gỡ Mí → `TempDrops.WipeAll()` (dòng `Wiped TempDrops on exit (n drops)`).
- Khởi động → `TempDrops.PurgeLeftovers()` trước cú thả đầu tiên (dòng `Purged n leftover entries`).
  Danh sách Mục sống trong RAM, nên sau một lần taskkill không Mục nào còn trỏ tới rác ấy.

## Nhật ký

| Tag | Dòng | Mức |
|---|---|---|
| `MiStrip` | `Dropped Files/VirtualFiles on the strip`; `Drop offers: …` (định dạng + TYMED) | Info / Trace |
| `DropIntake` | `Took real/virtual file '<tên>' (<n> bytes, <mime>)`; `Extracted k of n … in N ms` | Info |
| `DropIntake` | `Virtual entry i not extracted: …`; `declared X bytes but the source gave Y`; `Dropped path no longer exists` | Warn |
| `TempDrops` | `Released`, `Cleared`, `Evicted … to stay under the cap`, `Wiped … on exit`, `Purged …` | Info / Warn |
| `Shelf` | `Added n items, holding m`; `Cleared n items on request, m left` | Info |
