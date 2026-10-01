# Trích xuất tệp thật/ảo → TempDrops — Ticket 11

**Trạng thái:** implement xong 2026-10-01 trên nhánh `feat/142-tempdrops`
([#142](https://github.com/natuan1/peekvn/issues/142)).

**Treo:** demo tệp đính kèm **Outlook Desktop** — máy dev không cài Outlook. Đường
`FileGroupDescriptorW` + `IStream` đã đo bằng tệp kéo từ trong một thư mục zip của
Explorer, cùng định dạng, nhưng không thay được nguồn thật.

## Làm được gì

Người dùng kéo một tệp lên Mí và thả. Tệp vào **Shelf** (một Ngăn):

```
kéo tệp  →  Mí nở ra  →  thả  →  Mục vào Ngăn
```

| Nguồn kéo | Snappy nhận gì | Ghi ra đĩa |
|---|---|---|
| Explorer, tệp hay thư mục thường | đường dẫn gốc (`CF_HDROP`) | **không** — Mục trỏ thẳng tệp của người dùng |
| Explorer, tệp **trong zip** | tệp ảo (`IStream`) | `TempDrops\{Guid}\<tên gốc>` |
| Chrome, Edge — ảnh trên trang | tệp ảo (`HGLOBAL`) | như trên |
| Outlook Desktop — tệp đính kèm | tệp ảo (`IStream`) | như trên — **chưa đo** |

Mỗi Mục mang: tên, dung lượng, MIME type (từ byte đầu tệp, không chỉ từ đuôi).

## Bộ dọn — đúng quy tắc của [CONTEXT.md §TempDrops](../../CONTEXT.md)

| Khi nào | Làm gì |
|---|---|
| Mục bị bỏ khỏi Ngăn | xoá thư mục `{Guid}` của nó |
| TempDrops vượt **2 GiB** | bỏ Mục lâu không dùng nhất, bong bóng **"Shelf đã đầy"** gọi tên tệp bị bỏ |
| Người dùng chọn **"Xoá sạch Shelf (n mục)"** ở menu khay | bỏ mọi Mục, xoá mọi bản trích xuất |
| Thoát app | xoá cả thư mục `TempDrops\` |
| Khởi động (lần trước bị taskkill) | dọn rác còn sót |

Tệp đang mở ở app khác thì không xoá được: Mục **ở lại** Ngăn, và lượt dọn lúc thoát
thử lại.

## Người dùng thấy gì — và chưa thấy gì

- Con trỏ báo **Copy** khi kéo tệp lên Mí; báo "không thả được" với chữ, liên kết.
- Thả xong Mí về Nghỉ. **Chưa có khay thẻ** để thấy Mục — đó là
  [Ticket 12](https://github.com/natuan1/peekvn/issues/143). Hôm nay chỗ duy nhất thấy
  được là con số trong mục menu khay "Xoá sạch Shelf (n mục)".
- Nguồn không trao được nội dung → bong bóng "Không giữ được tệp vừa thả", kèm lối ra.

## Các quyết định lệch kế hoạch

- **Lúc thả không gọi `SHGetFileInfoW`.** Nó tốn ~2,3 MB RAM và đẩy app qua KPI 25 MB.
  Tên loại và icon dời tới khay thẻ. Xem [ADR-0015](../../adr/0015-tempdrops-mot-thu-muc-moi-muc-khong-hoi-shell-luc-tha.md).
- **Mí nhận cú thả ngay từ ticket này**, không đợi Ticket 13: không có cú thả thật thì
  không có cách nghiệm thu "kéo từ Explorer/Chrome/Edge" bằng cử chỉ người dùng.
- **Trần 2 GiB** là con số chọn ở ticket này; spec chỉ nói "trần dung lượng".

Xem thêm: [workflow](workflow.md) · [implementation](implementation.md) · [testing](testing.md)
