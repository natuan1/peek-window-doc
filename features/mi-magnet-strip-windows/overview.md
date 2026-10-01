# Mí magnet strip trên Windows — Ticket 10

**Trạng thái:** implement xong 2026-09-30 trên nhánh `feat/141-mi-magnet-strip`
([#141](https://github.com/natuan1/peekvn/issues/141)).

**Chưa nghiệm thu:** đa màn hình, DPI 125–200% và fullscreen exclusive. Máy dev chỉ
có một màn hình 1920×1080 ở 100%.

**RAM:** đã về dưới KPI sau khi bỏ Composition
([#181](https://github.com/natuan1/peekvn/issues/181),
[ADR-0014](../../adr/0014-mi-ve-bang-layered-window-khong-composition.md)).

## Làm được gì

Người dùng kéo một tệp. Mí hiện ra ở cạnh trên màn hình, và nở ra khi tệp dừng
lại trên nó:

```
kéo tệp ở đâu đó  →  vạch Gợi ý hiện ở cạnh trên  →  rê tệp lên và dừng ≥ 80 ms  →  khay nở ra
```

Mí rộng 30% màn hình và bắt đầu từ 5% bên trái, chừa khoảng giữa cho Snap Layouts
và góc phải cho nút đóng cửa sổ.

**Hình:** **trắng đục 0,75**, gần với điểm sáng của Task View. Không viền, bo cong nhẹ
ở hai góc dưới. Vạch Gợi ý, khay nở ra và pill dùng cùng một màu. Chủ dự án chốt
ngày 2026-09-30, sau hai lần thử bản thật (bản đầu là đen 0,8).

## Năm trạng thái

| | Trạng thái | Người dùng thấy |
|---|---|---|
| 0 | Nghỉ | không có gì — cửa sổ ẩn hẳn |
| 1 | Gợi ý | vạch trắng đục 10 DIP |
| 2 | Sẵn sàng nhận | khay trắng đục cao 90 DIP, nở trong 160 ms |
| 3 | Nhắm đích | khay trắng đặc hơn (0,9), không viền |
| 4 | Đang chạy | pill 180×32 DIP |

## Chưa làm, có chủ ý

- ~~Mí chưa nhận cú thả nào.~~ **Từ Ticket 11 (2026-10-01) Mí nhận cú thả tệp** vào
  Shelf một Ngăn — xem [Trích xuất tệp thật/ảo → TempDrops](../tempdrops-windows/overview.md).
  Đích thả riêng (thiết bị, ô Shelf) vẫn là
  [Ticket 13](https://github.com/natuan1/peekvn/issues/144).
- Trạng thái 3 và 4 có trong máy trạng thái và có test, nhưng **chưa tới được
  bằng tay**. Cả hai cần Đích thả và Phiên truyền thật của Ticket 13.
- Khay đã nở chưa có chữ hay đích nào bên trong.

## Các quyết định lệch kế hoạch

- **Nghỉ là ẩn hẳn, xác nhận tệp đi qua `DragEnter`.** Clipboard đã được đo và
  không thấy lượt kéo. Xem [ADR-0013](../../adr/0013-mi-an-khi-nghi-dung-luoi-xac-nhan-tep-qua-dragenter.md).
- **Bỏ Composition, vẽ bằng layered window.** Xem [ADR-0014](../../adr/0014-mi-ve-bang-layered-window-khong-composition.md).

Xem thêm: [workflow](workflow.md) · [implementation](implementation.md) · [testing](testing.md)
