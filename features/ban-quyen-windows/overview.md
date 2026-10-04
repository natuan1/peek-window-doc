# Bản quyền Pro trên Windows

[Ticket 14](https://github.com/natuan1/peekvn/issues/145), 2026-10-04, nhánh `feat/145-licensing`.

Mua Pro không cần tài khoản. Người dùng dán Key nhận qua email, máy được gán vào một Slot của Key, và Pro chạy offline vĩnh viễn. Khi có mạng, app thầm lặng hỏi lại máy chủ xem Slot còn không.

## Người dùng thấy gì

| Lúc | Thấy |
|---|---|
| Bản Free | menu khay có **"Kích hoạt Snappy Pro…"**; bảng trạng thái ghi `Phiên bản 1.0.0 · Bản Free` |
| Bấm mục ấy | hộp thoại "Kích hoạt Snappy Pro": một ô Key, một dòng trạng thái, nút **Kích hoạt** / **Đóng** |
| Dán Key sai dạng | đỏ: "Key có dạng SNPY-XXXX-XXXX-XXXX-XXXX — 16 ký tự sau SNPY…" (không gọi mạng) |
| Key không tồn tại | đỏ: "Máy chủ không nhận ra Key này. Kiểm lại từng ký tự với email mua hàng — chép rồi dán là chắc nhất." |
| Key đã đủ 5 máy | đỏ: "Key này đã kích hoạt trên đủ 5 máy. Gỡ một máy cũ ở trang quản lý Key (cần Key và email mua) rồi bấm Kích hoạt lại." |
| Không có mạng | đỏ: "Chưa kết nối được máy chủ bản quyền. Kiểm tra mạng rồi bấm Kích hoạt lại." |
| Key đúng | xanh: "Đã kích hoạt Snappy Pro trên máy này…", nút Đóng thành **Xong**; bong bóng "Snappy Pro đã bật"; menu thôi mời kích hoạt; bảng trạng thái ghi `· Snappy Pro` |
| Khởi động lại, có hay không có mạng | vẫn Pro, không một câu hỏi nào |
| Chủ Key gỡ máy này ở portal | lượt kiểm sau (≤ 2 phút sau khởi động, rồi mỗi 24 h) về Free, bong bóng "Snappy Pro đã gỡ khỏi máy này…" chỉ đường kích hoạt lại |
| Key bị thu hồi | về Free, bong bóng "…Key Pro này đã bị thu hồi… mua Key mới…" |

Enter là nút Kích hoạt, Esc là Đóng. Ô Key tự viết hoa và nhận Key có gạch dài hay khoảng trắng do trình soạn thư chèn vào.

## Cổng gate Free/Pro

`Entitlements` là chỗ **duy nhất** ghi ranh giới:

| | Free | Pro |
|---|---|---|
| Ngăn Shelf | 1 | không trần |
| Ghim | không | có |
| Lịch sử clipboard (M4, Ticket 16 đọc) | 5 mục | không giới hạn |

`ProLicense.RequirePro(feature)` là API cho tính năng Pro hỏi trước khi chạy. Shelf đổi gói ngay giữa phiên (`Shelf.ApplyPlan`). Xuống Free không xoá Ngăn hay Mục nào, chỉ chặn tạo thêm.

## Không làm (theo ticket hoặc cố ý)

- **Máy chủ thật.** Cloudflare Worker là hạ tầng ngoài spec #131. App giữ đúng [hợp đồng API](api.md). Hôm nay app tin **khoá ký DEV** và gọi `https://license.snappy.vn`, một địa chỉ chưa tồn tại, nên người dùng thật luôn nhận câu "chưa kết nối được". `ci/pack.ps1 -RequireSigning` từ chối phát hành chừng nào khoá DEV còn trong mã.
- **Portal gỡ Slot** (Key + email) thuộc máy chủ.
- **Giao diện nhiều Ngăn và ghim.** Spec #131 nói v1.0 chỉ cần cấu trúc sẵn sàng, nên mua Pro hôm nay chưa đổi gì người dùng nhìn thấy ngoài nhãn "Snappy Pro".
- **Trang mua.** Hộp thoại không có đường mua Key, vì chưa có trang nào để trỏ tới.

## Xem thêm

- [Hợp đồng API](api.md) · [Luồng](workflow.md) · [Cài đặt](implementation.md) · [Kiểm thử](testing.md)
- [ADR-0019](../../adr/0019-ban-quyen-verify-tai-may-mang-trong-tien-trinh-con.md)
