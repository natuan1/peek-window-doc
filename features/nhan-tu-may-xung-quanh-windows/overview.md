# Nhận từ máy xung quanh — #261

**Trạng thái:** PR [natuan1/peekvn#285](https://github.com/natuan1/peekvn/pull/285) mở 2026-11-14, CI Jenkins #208 đạt, chưa merge
([#261](https://github.com/natuan1/peekvn/issues/261), epic [#258](https://github.com/natuan1/peekvn/issues/258))

## Làm được gì

Spec peek 1.0 chia máy làm hai nhóm. Ticket này làm phần giao thức cho cả hai:

| | Máy của tôi | Máy xung quanh |
|---|---|---|
| Khi gửi tới máy Windows | Tự nhận, không hỏi | Hỏi Nhận / Từ chối |
| Không ai bấm | — | Sau 1 phút tự từ chối |
| Người gửi thấy | nhận ngay | "Đã từ chối" hoặc "Không phản hồi" |
| Máy nhận nhớ gì | khoá đã ghim (Trust Store) | không gì cả |

Trước ticket này máy chưa ghép bị chặn ở cửa (`403 DEVICE_NOT_TRUSTED`), và máy đã
ghép vẫn bị hỏi từng lượt trừ khi bật công tắc Tự nhận cho riêng nó.

## User stories

- Là **Hưng**, khi Lan (chưa ghép) gửi tệp, tôi thấy "Lan's Mac (máy xung quanh,
  chưa ghép) muốn gửi tệp …", bấm Nhận là tệp vào `Downloads`.
- Là **Lan**, nếu Hưng bấm Từ chối tôi biết là bị từ chối; nếu Hưng không ở
  máy, sau một phút tôi biết là không có phản hồi — không treo mãi.
- Là **người dùng có hai máy của chính mình**, tệp gửi qua lại vào thẳng, không
  có hộp nào.

## Ngoài phạm vi

- Nút "Hiện với máy xung quanh" và việc máy đang ẩn từ chối máy lạ — #263. Tới
  khi đó máy Windows luôn nhận lời hỏi từ máy lạ.
- Lời hỏi trên Mí (đếm ngược, Chặn máy này) — #270. Hiện dùng hộp thoại tạm.
- Windows **gửi** tới máy xung quanh — #266/#267.
- iOS/Android gửi như máy lạ — chưa có ticket.
- Đánh dấu "tải từ mạng" cho tệp nhận từ máy xung quanh.

## Quyết định

Lý do và rủi ro đã chấp nhận ở [ADR-0029 của `peekvn`](https://github.com/natuan1/peekvn/blob/main/docs/adr/0029-may-la-da-xac-thuc.md)
(dãy ADR của repo mã nguồn, không phải ADR-0029 của repo này).

Xem thêm: [workflow](workflow.md) · [implementation](implementation.md) · [testing](testing.md)
