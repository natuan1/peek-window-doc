# Nhận push từ iPhone trên Windows — Ticket 07

**Trạng thái:** implement xong 2026-09-20, **chờ demo iPhone thật**
([#138](https://github.com/natuan1/peekvn/issues/138))

## Làm được gì

iPhone đẩy tệp sang PC, tệp xuất hiện trong `Downloads\Snappy`. Từ đầu tới cuối:

```
iPhone mời  →  Snappy hỏi người dùng  →  người bấm Nhận  →  byte chảy xuống đĩa
```

Đây là **làn đẩy đầu tiên end-to-end** của Snappy. Trước ticket này Snappy mở
được kênh, ghép đôi được và xác thực được — nhưng chưa có một byte dữ liệu nào
đi qua nó.

## Ba câu quyết định hình dạng của cả ticket

**1. Ở chiều `push`, Receiver là server.** SPEC §6 nói `mode` quyết định *ai chạy
HTTP server*, và điều đó **trực giao** với ai là Sender. Nên iPhone — bên gửi —
là *client*, còn Snappy — bên nhận — là *server*. Hệ quả cụ thể, và nó đi ngược
trực giác: `POST /v1/transfers` do **bên gửi** gọi, còn `accept` là quyết định
của người ngồi trước máy này, không của một request nào.

**2. Byte không bao giờ nằm trọn trong RAM.** SPEC §17.1: *"mức tiêu thụ memory
MUST NOT tỉ lệ với kích thước file"*. Thân request đi qua một `Stream` chặn đúng
`Content-Length`, đọc từng khúc 64 KB mượn từ `ArrayPool`, băm SHA-256 ngay lúc
ghi.

**3. Tên tệp và đường dẫn là dữ liệu của người lạ.** SPEC §19.1 gọi path
traversal là *"regression test bắt buộc"* vì LocalSend từng có lỗ hổng mức cao ở
đúng chỗ này. Trên Windows lối thoát ấy **rẻ hơn trên POSIX** — xem
[workflow.md](workflow.md).

## Người dùng thấy gì

| Lúc | Thấy |
|---|---|
| Có lời mời | Hộp thoại: ai gửi, bao nhiêu mục, tổng dung lượng, **lưu vào đâu** — hai nút, mặc định *Không* |
| Đang nhận | Nhật ký ghi từng tệp; tiến độ đọc được qua `GET /v1/transfers/{id}` |
| Nhận xong | Tệp nằm trong `Downloads\Snappy`; menu khay có mục **Mở thư mục nhận** |
| Không muốn bị hỏi nữa | Menu khay → **Tự nhận từ &lt;thiết bị&gt;**, có dấu tích, hỏi lại một lần khi bật |

`Downloads\Snappy` chứ không phải thẳng `Downloads`, và hỏi hệ điều hành đường
dẫn thật thay vì nối `%UserProfile%\Downloads` — Downloads là thư mục người dùng
**dời được** sang ổ khác. Lý do thứ ba mới là lý do chính: *"bên trong Destination
Root"* là một lời hứa hẹp hơn hẳn khi root là thư mục của riêng Snappy.

## Còn thiếu, nói thẳng

- **Không có màn hình tiến độ.** Tiến độ có thật và đọc được qua API, nhưng bảng
  trạng thái chưa hiện nó. Với một tệp 4 GB, thứ người dùng thấy là một hộp thoại
  rồi im lặng cho tới lúc tệp xuất hiện.
- **Không có nút huỷ ở phía Windows.** `POST …/cancel` có và cả hai bên gọi được;
  người ngồi trước máy này thì chưa có chỗ bấm.
- **Chưa có implementation thứ hai đẩy tới.** Bằng chứng liên tiến trình của
  ticket này là bản AOT thật nhận từ một đầu dò — cùng codebase. Phép đo
  `rust-host ↔ windows` là tiêu chí của Ticket 08.
- **Không resume.** Đứt giữa chừng là mất phần đã nhận và phải gửi lại từ đầu.
  Đó là Ticket 08 ([#139](https://github.com/natuan1/peekvn/issues/139)) — và
  SPEC §17.5 nói thẳng rằng ô "upload × đứt kết nối" chỉ có lời giải nếu
  **Receiver** nói được protocol ấy, tức toàn bộ điều kiện nằm ở phía này.
