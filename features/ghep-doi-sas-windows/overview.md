# Ghép đôi SAS trên Windows — hai màn hình, sáu chữ số, một người

**Trạng thái:** ✅ Implement xong 2026-09-19 ([Ticket 06 / #137](https://github.com/natuan1/peekvn/issues/137))
**Nghiệm thu còn treo:** demo với **iPhone thật**
**Quyết định kèm theo:** [ADR-0011](../../adr/0011-ghep-doi-khong-tu-confirm-responder-cho-nguoi-dung-cuc-bo.md) · [ADR-0012](../../adr/0012-cap-harness-do-hanh-vi-nen-tang-phai-chay-cho-tung-tls-stack.md)

## Tính năng này là gì

Từ ticket này, Snappy **nhớ được** một thiết bị. Trước đó nó thấy peer (Ticket
04) và trả lời được câu "máy này là ai" (Ticket 05), nhưng mọi kết nối đều là
lần đầu — không có khái niệm *thiết bị của tôi*.

Nghi thức để đi từ "một máy lạ trong LAN" sang "thiết bị của tôi" có đúng một
bước, và nó không phải mật khẩu:

> Hai màn hình hiện **cùng một con số sáu chữ số**. Người dùng nhìn, so, rồi
> bấm đồng ý ở **cả hai** máy.

## Vì sao sáu chữ số, và vì sao phải là người

`protocol/SPEC.md` §10.1 gọi SAS là *"thứ **duy nhất** chặn được MITM"*. Câu ấy
đúng theo nghĩa đen, và `protocol/SECURITY.md` §7.3 ghi lại đánh đổi:

> Toàn bộ chống MITM đứng trên một hành động của con người.

Khoá trao đổi qua ECDH thì kẻ đứng giữa không cần biết bí mật — nó chỉ cần
**chuyển tiếp trung thực**, và lúc ấy hai bên vẫn ra cùng shared secret. Thứ
phân biệt "không có ai ở giữa" với "có, và nó ngoan" là một giá trị mà kẻ ở giữa
**không thể làm cho khớp**: SPKI của certificate TLS mà mỗi bên thực sự nhìn
thấy.

Vì vậy sáu chữ số ấy là hàm băm của *cả cuộc trao đổi cộng danh tính của kênh*.
Hai máy nói chuyện thẳng với nhau ⇒ hai con số bằng nhau. Có kẻ ở giữa ⇒ hai con
số khác nhau, và con người là bộ phận duy nhất trong hệ thống so được chúng.

## Người dùng thấy gì

Một hộp thoại, hai nút:

```
Snappy — ghép đôi

iPhone của Tuấn muốn ghép đôi với máy này.

Mã xác thực:  034212

So mã này với mã đang hiện trên thiết bị kia. Chỉ bấm Có khi hai mã
GIỐNG HỆT nhau.

Hai mã khác nhau nghĩa là có thiết bị lạ đứng giữa — hãy bấm Không.

Thiết bị: dev_wukFlCjjSradc2mo2SW4MfHveyrkPSxAEvQSwvWyFW4

                                          [ Có ]   [ Không ]
```

Ba chi tiết là quyết định, không phải thẩm mỹ:

1. **Nút mặc định là "Không".** Gõ Enter theo phản xạ sẽ *từ chối*.
2. **Không có nút thứ ba.** SECURITY.md §4.1 nói MUST NOT có "tin tưởng dù sao"
   — nút đó tồn tại vì nó tiện rồi thành thứ người dùng bấm theo phản xạ.
3. **Hộp thoại chặn.** Nó phải ở trước mặt cho tới lúc có quyết định, không nằm
   trong một khung có thể bị che rồi quên.

📐 Con số `034212` ở trên là số đo thật, 19/09/2026, trên bản AOT đã publish. Nó
có **số 0 đứng đầu** — đúng trường hợp SECURITY.md §3.4 nói sẽ dạy hư người dùng
nếu bị cắt, vì hai máy hiện `34212` và `034212` cho cùng một giá trị.

## Sau khi ghép đôi

Snappy ghi thiết bị vào `%LocalAppData%\Snappy\trusted-peers.json` và **nhớ qua
các lần khởi động lại**. Từ đó mọi kết nối:

* **ghim SPKI** — khoá khác giá trị đã nhớ thì kênh không mở, và không có đường
  đi tiếp (`DEVICE_IDENTITY_CHANGED`, bất biến S14);
* **xác thực hai chiều** — cả hai bên ký một chuỗi byte chứa nonce của cả hai,
  `deviceId` của cả hai, và pin của kênh đang dùng. Hai vai ký hai chuỗi **khác
  nhau**, nếu không thì chữ ký lấy ở vai này dùng lại được ở vai kia.

`DELETE /v1/pairings/{deviceId}` gỡ tin cậy và thu hồi ngay mọi session token
của thiết bị ấy.

## Cái chưa có

* **Không có chỗ bấm "quên thiết bị này" trên Windows.** Đường gỡ hiện chỉ đi từ
  phía peer, vì `DELETE` đòi session token và một peer chỉ gỡ được chính nó. Với
  người dùng, đó nghĩa là họ chưa gỡ được một thiết bị đã mất.
* **Không có danh sách "đã ghép đôi" trên bảng trạng thái.** Bằng chứng duy nhất
  là tệp trên đĩa và dòng nhật ký lúc khởi động.
* **`autoAccept` và `lastSeenAt` được lưu nhưng chưa ai đọc.** §10.2 nói MUST
  lưu; chúng nằm đó và sống qua vòng ghi/đọc, không hơn.

Cả ba đều là việc của ticket sau, và cả ba được ghi ra vì một mục im lặng sẽ
được đọc thành mục đó không tồn tại.
