# Xoá khỏi Máy của tôi ở một máy là đủ — #264

**Trạng thái:** PR vào `main` của `peekvn` ([#264](https://github.com/natuan1/peekvn/issues/264), epic
[#258](https://github.com/natuan1/peekvn/issues/258)).
Quyết định: [ADR-0035 của `peekvn`](https://github.com/natuan1/peekvn/blob/main/docs/adr/0035-huy-ghep-o-mot-may-la-du.md).
Giao thức: `protocol/SPEC.md` §10.3, `protocol/SECURITY.md` §4.7.

## Làm được gì

Spec peek 1.0, Luồng 2 — Xóa: *"Không hỏi lại. Hiện «Đã xóa · Hoàn tác» trong vài giây. Xóa ở
một máy là đủ. Hai máy đều thôi tự nhận của nhau, lần sau gửi sẽ bị hỏi như máy xung quanh."*

| Bước | Máy | Thấy gì / làm gì |
|---|---|---|
| 1 | A | Cài đặt → Thiết bị → **Xoá** trên dòng của B. Không hộp hỏi lại |
| 2 | A | Dòng thành **"Đã xoá · Hoàn tác trong vài giây"**, nút **Hoàn tác** |
| 3a | A | Bấm Hoàn tác trong 5 giây → không gì thay đổi ở cả hai máy |
| 3b | A | Không bấm → hết 5 giây A thôi tự nhận B, gửi **lời huỷ ghép** tới B; B thôi tự nhận A |
| 4 | Cả hai | Lần sau gửi cho nhau là bị hỏi như máy xung quanh |

B đang tắt lúc A xoá: A vẫn thôi tin B ngay; lời huỷ ghép **nằm chờ trên đĩa** và đi ở lần A
thấy lại B trên mạng. Lời huỷ ghép không bao giờ tới (iPhone không chạy server): B gửi tới A thì
A trả "không còn ghép", B tự hạ A xuống Máy xung quanh — chỉ khi câu trả lời ấy chứng minh được
là của đúng A.

## User stories

- Là **Tuấn** mất laptop, tôi xoá laptop ấy trên máy bàn. Từ đó laptop không gửi tự do vào máy
  bàn được nữa, kể cả khi nó đang tắt lúc tôi xoá.
- Là **Tuấn** bấm nhầm Xoá, tôi bấm Hoàn tác ngay — hai máy không biết gì đã xảy ra.

## Ngoài phạm vi

- Chuột phải ô máy "Xóa khỏi Máy của tôi" ở cửa sổ peek mới (#265 và các ticket giao diện sau).
  Hôm nay nút nằm ở trang Thiết bị của cửa sổ Cài đặt.
- Nửa client "hạ ở bước xác thực" trên iOS và Android — ticket riêng của từng nền tảng.

## Rủi ro đã chấp nhận

Người dùng ghép lại đúng lúc một lời huỷ ghép cũ đang trên đường đi thì lần ghép mới có thể bị
gỡ ở máy kia (cửa sổ một vòng xác thực). Hậu quả: hai máy về Máy xung quanh, thêm lại một lần.
