# Mí magnet strip trên Windows — Ticket 10

**Trạng thái:** implement xong 2026-09-30 trên nhánh `feat/141-mi-magnet-strip`
([#141](https://github.com/natuan1/peekvn/issues/141)). **Chưa nghiệm thu** đa màn
hình, DPI 125–200% và fullscreen exclusive, vì máy dev chỉ có một màn hình
1920×1080 ở 100%. ⚠️ RAM sau lần kéo đầu tiên vượt KPI, xem
[#181](https://github.com/natuan1/peekvn/issues/181).

## Làm được gì

Người dùng kéo một tệp. Mí hiện ra ở cạnh trên màn hình, và nở ra khi tệp dừng
lại trên nó:

```
kéo tệp ở đâu đó  →  vạch Gợi ý hiện ở cạnh trên  →  rê tệp lên và dừng ≥ 80 ms  →  khay nở ra
```

Mí rộng 30% màn hình và bắt đầu từ 5% bên trái. Khoảng giữa để cho Snap Layouts,
góc phải để cho nút đóng cửa sổ.

## Năm trạng thái

| | Trạng thái | Người dùng thấy |
|---|---|---|
| 0 | Nghỉ | không có gì, cửa sổ ẩn hẳn |
| 1 | Gợi ý | vạch xanh mảnh 6 DIP |
| 2 | Sẵn sàng nhận | khay tối cao 90 DIP, viền xanh, nở trong 160 ms |
| 3 | Nhắm đích | khay viền đậm hơn |
| 4 | Đang chạy | pill 180×32 DIP |

## Chưa làm, có chủ ý

- **Mí chưa nhận cú thả nào.** Con trỏ hiện "không thả được", vì chưa có Shelf
  ([Ticket 12](https://github.com/natuan1/peekvn/issues/143)) hay Đích thả
  ([Ticket 13](https://github.com/natuan1/peekvn/issues/144)). Nói "Copy" với OLE
  là hứa một việc rồi lặng lẽ vứt nó.
- Trạng thái 3 và 4 có trong máy trạng thái và có test, nhưng **chưa tới được
  bằng tay**: cần Đích thả thật và Phiên truyền thật của Ticket 13.
- Khay đã nở chưa có chữ hay đích nào bên trong.

## Ba quyết định lệch kế hoạch

Xem [ADR-0013](../../adr/0013-mi-an-khi-nghi-dung-luoi-xac-nhan-tep-qua-dragenter.md).

- **Nghỉ là ẩn**, không phải vạch 3px.
- **Xác nhận tệp đi qua `DragEnter`**, không qua clipboard. Clipboard đã được đo
  và không thấy lượt kéo.
- **OLE và Composition dựng lười.**

Xem thêm: [workflow](workflow.md) · [implementation](implementation.md) · [testing](testing.md)
