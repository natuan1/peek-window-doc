# Thêm vào Máy của tôi bằng một lần Đồng ý — #262

**Trạng thái:** đang làm trên nhánh `claude/262-ghep-mot-lan-dong-y` của `peekvn`
([#262](https://github.com/natuan1/peekvn/issues/262), epic [#258](https://github.com/natuan1/peekvn/issues/258)).
Quyết định: [ADR-0034 của `peekvn`](https://github.com/natuan1/peekvn/blob/main/docs/adr/0034-ghep-mot-lan-dong-y.md).

## Làm được gì

Spec peek 1.0, Luồng 2: *"Không QR, không so số."*

| Bước | Máy | Thấy gì / làm gì |
|---|---|---|
| 1 | B | Bấm "Hiện với máy xung quanh" |
| 2 | A | Thấy B trong Máy xung quanh, bấm **Thêm B vào Máy của tôi** |
| 3 | B | Hỏi **"[A] muốn thêm vào Máy của tôi"** — "Hai máy sẽ gửi cho nhau không cần hỏi." — Từ chối · Đồng ý |
| 4 | Cả hai | Lên Máy của tôi của nhau; **B tự tắt chế độ hiện** |

B bấm Từ chối: A thấy **"Chưa được đồng ý"**, không gì thay đổi. B đang ẩn: A thấy
"B đang ẩn", B không hiện lời hỏi nào.

Windows làm được **cả hai vai**: người bấm Thêm (A) và người được hỏi (B). Hai máy
**nhận ra nhau bằng khoá** — đổi tên, đổi mạng vẫn là Máy của tôi, và A nhớ tên mới.

## User stories

- Là **Tuấn** có laptop mới, tôi bấm Hiện trên laptop, bấm Thêm trên máy bàn, bấm
  Đồng ý trên laptop — xong, không đọc số nào.
- Là **Lan** ngồi cạnh, nếu Tuấn lỡ bấm Thêm máy tôi, tôi bấm Từ chối và Tuấn biết
  là chưa được đồng ý.

## Ngoài phạm vi

- Nút + trên ô máy ở cửa sổ peek — #267. Hôm nay cử chỉ là menu khay "Thêm … vào Máy của tôi".
- Lời hỏi trên Mí — #270. Hôm nay là hộp thoại tạm.
- Xoá khỏi Máy của tôi ở hai phía — #264.
- iPhone (#293) và Android vẫn so số tới khi có ticket của chúng — dây không đổi nên
  vẫn ghép được với peek Windows.
- Hai bản **Windows** gửi tệp cho nhau: Windows chưa có vai push-sender (#252), nên
  "gửi qua lại" đo với rust-host.

## Rủi ro đã chấp nhận

Kẻ đứng giữa trong LAN **ngay lúc ghép** thành Máy của tôi (chủ dự án chốt
13/11/2026). Giảm bằng: chỉ nhận khi đang hiện, B tự ẩn sau Đồng ý, lời xin chỉ từ
một cú bấm thật.
