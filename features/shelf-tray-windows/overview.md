# Khay thẻ Shelf một Ngăn — Ticket 12

**Trạng thái:** implement xong 2026-10-02 trên nhánh `feat/143-shelf-tray`
([#143](https://github.com/natuan1/peekvn/issues/143)). Đứng trên bản sửa hộp chọn tệp
(nhánh `fix/file-picker-process`, PR [#185](https://github.com/natuan1/peekvn/pull/185)).

## Làm được gì

Người dùng thả tệp lên Mí (Ticket 11), rồi mở **khay thẻ** để xem và lấy lại:

```
menu khay "Mở Shelf (n mục)"  →  khay thẻ ngang neo trên Mí  →  kéo thẻ ra Explorer / trình duyệt / app khác
```

| Cử chỉ | Kết quả |
|---|---|
| Kéo **một thẻ** ra app khác | app nhận đúng tệp (`CF_HDROP`), **Copy** — tệp gốc ở nguyên chỗ |
| Kéo **tay cầm** ở góc khay | cả Ngăn đi ra trong một lượt kéo |
| Bấm nút thư mục trên thẻ | Explorer mở thư mục chứa, chọn sẵn tệp |
| Bấm ✕ | bỏ Mục khỏi Ngăn; bản trích xuất trong TempDrops bị xoá |
| Esc, hoặc bấm ra ngoài | khay đóng |
| Lăn chuột | cuộn ngang khi Ngăn dài hơn khay |

Kéo ra **không** bỏ Mục khỏi Ngăn (chốt 2026-09-13). Mục thành "vừa dùng" trong LRU của
trần 2 GiB — như sau khi gửi đi.

Mỗi thẻ: glyph theo loại (thư mục, ảnh, video, âm thanh, nén, tài liệu), tên, dung lượng.
Tệp gốc đã bị xoá, đổi tên hay nằm trên ổ đã rút thì thẻ ghi **"Không còn ở chỗ cũ"**,
không đi cùng lượt kéo, và bong bóng **"Chỉ kéo được X trong Y mục"** nói ra điều đó.

## Không làm (theo ticket)

- Lối vào khay từ một **ô Shelf trên Mí**: Đích thả riêng là Ticket 13. Hôm nay khay mở
  từ menu khay hệ thống, và mọi Module dùng được không qua Mí (CONTEXT.md §Module).
- **Nhiều Ngăn và ghim** là của gói Pro. Tầng dữ liệu đã sẵn (`ShelfPlan`,
  `ShelfCompartment`, `TrySetPinned`); giao diện chỉ vẽ một Ngăn.
- Icon shell thật — xem [ADR-0017](../../adr/0017-tinh-nang-nang-phan-shell-chay-trong-tien-trinh-con.md).

## Đọc tiếp

- [workflow.md](workflow.md) — luồng giữa tiến trình nền và tiến trình khay
- [implementation.md](implementation.md) — tệp, giao kèo, quyết định
- [testing.md](testing.md) — test tự động, nghiệm thu thật, số đo RAM
