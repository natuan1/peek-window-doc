# Discovery LAN trên Windows (`_peek._tcp`)

## 📋 Mô tả

Mặt phẳng discovery của Snappy: máy tính **tự quảng bá** mình trên Wi-Fi và
**tự nghe** xem có thiết bị Peek nào khác quanh đó. Người dùng mở app trên điện
thoại cùng mạng là hai đầu thấy nhau — không nhập IP, không nhập cổng, không quét
QR.

Snappy dùng **responder mDNS có sẵn của Windows** (`dnsapi.dll`), không tự dựng
responder — xem [ADR-0008](../../adr/0008-discovery-qua-responder-in-box-windows.md)
cho lý do và phép đo.

## 🎯 Phạm vi

**Có:**

- Quảng bá `_peek._tcp` với TXT record đúng SPEC §8: `pv`, `id`, `port` — và
  không gì khác.
- Bảng **Thiết bị lân cận**: tên, địa chỉ, lần cuối nghe thấy, còn online không.
- Bảng ấy hiện trên **bảng trạng thái khay** — nếu người dùng không nhìn thấy thì
  tính năng chưa tồn tại với họ.
- Danh tính thiết bị P-256 bền qua các lần khởi động (SPEC §9.1), lưu trong kho
  khoá CNG của người dùng.

**Không có (thuộc ticket sau):**

- **Nối được**: server ULTP là [Ticket 05](https://github.com/natuan1/peekvn/issues/136).
  Cổng 8443 đang được quảng bá trước khi có ai nghe ở đó, nên tiêu chí của ticket
  này dừng ở *"hai đầu **thấy** nhau"*.
- **Ghép đôi, tin cậy**: [Ticket 06](https://github.com/natuan1/peekvn/issues/137).
  Kết quả discovery là **gợi ý không đáng tin** (SPEC §8.1) — nó không tạo ra
  quyền gì.
- **Cột nền tảng** (iPhone/Android/Windows): discovery không trả lời được, vì TXT
  của SPEC §8 đóng ở ba khoá. Câu trả lời ở `GET /v1/info`, tức Ticket 05.

## 👥 User stories

- Là người dùng Windows, tôi muốn **thấy điện thoại của mình xuất hiện** ngay khi
  mở app mobile cùng mạng, để không phải cấu hình IP hay cổng gì cả
  (story 15 của spec #131).
- Là người dùng, tôi muốn **biết khi một máy không còn đó nữa**, để không bấm vào
  một cái tên đã chết.

## ⚠️ Điều người dùng sẽ gặp

- **Máy tắt hẳn còn nằm trong danh sách tới 120 giây**, rồi chuyển sang
  *"không thấy nữa"*. Đó là TTL của bản ghi mDNS, không phải app treo.
- **Snappy thoát thì biến mất ngay** ở đầu bên kia — app rút bản ghi lúc thoát.
- Mạng bật **client isolation** (nhiều Wi-Fi khách sạn, văn phòng) thì không đầu
  nào thấy đầu nào, và không có gì trong app sửa được điều đó.

## 🔗 Liên quan

- [Quy trình](workflow.md) · [Cài đặt](implementation.md) · [Kiểm thử](testing.md)
- [ADR-0008](../../adr/0008-discovery-qua-responder-in-box-windows.md)
- Ticket [#135](https://github.com/natuan1/peekvn/issues/135) · PR [#152](https://github.com/natuan1/peekvn/pull/152)
