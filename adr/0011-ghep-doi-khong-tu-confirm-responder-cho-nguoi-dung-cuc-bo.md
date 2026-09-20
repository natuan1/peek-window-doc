# ADR-0011: Ghép đôi **không tự confirm** — responder giữ request mở chờ người dùng cục bộ

Date: 2026-09-19
Status: Accepted

> ⚠️ Kho `peekvn` có một dãy ADR **độc lập** với dãy này. ADR-0012/0013/0014 được
> nhắc tới dưới đây là của `peekvn`, không phải của kho này.

## Context

Tiêu chí nghiệm thu thứ tư của [Ticket 06 (#137)](https://github.com/natuan1/peekvn/issues/137)
viết: *"Quy trình ghép đôi KHÔNG tự confirm — người thật đọc số trên hai màn
hình."*

Hợp đồng trên dây (`protocol/openapi/ultp-v1.yaml`) có đúng hai bước:

```
POST /v1/pairings                      → 201, hai bên tự tính SAS
POST /v1/pairings/{pairingId}/confirm  → 200, body: { "confirmed": true }
```

Reference Host làm điều hiển nhiên: nhận `confirmed: true` thì ghim luôn. Nó là
một Host không có giao diện, nên đó là lựa chọn duy nhất nó có.

Với Snappy thì lựa chọn ấy **sai**, và sai ở chỗ không lộ ra trong bất kỳ phép
thử chức năng nào. `confirmed: true` là quyết định của người dùng **phía bên
kia**. Nó không nói gì về người đang ngồi trước máy Windows. Tin nó một mình
nghĩa là:

- kẻ tấn công mở `POST /v1/pairings` rồi tự gửi `confirm`, và Snappy ghim nó —
  **không có người nào so mã cả**;
- nghi thức sáu chữ số trở thành một thủ tục mà một bên tự hoàn tất được.

`protocol/SECURITY.md` §7.3 ghi thẳng giới hạn mà toàn bộ mô hình đứng lên:

> Toàn bộ chống MITM đứng trên một hành động của con người.

Bỏ hành động ấy ở một đầu thì đầu đó không còn được bảo vệ, bất kể đầu kia làm
gì đúng.

## Decision

**Một phiên chỉ thành Trusted Peer khi cả hai phía đã đồng ý.**

`PairingCoordinator.ConfirmAsync` nhận `confirmed` từ dây, rồi **chờ tiếp** một
lời gọi `ConfirmLocally` đến từ giao diện. Ba hệ quả được chốt cùng lúc:

1. **Request HTTP `confirm` bị giữ mở** cho tới khi người dùng bấm, tối đa bằng
   hạn của phiên (120 giây, SECURITY.md §3.4).
2. **Hết hạn tính là *từ chối***, không phải chấp nhận. Đây là chiều duy nhất an
   toàn: một phiên mà không ai ngồi trước máy để so mã là đúng hoàn cảnh kẻ tấn
   công cần.
3. **Kết nối TLS đứt thì phiên bị huỷ**, và mọi lời gọi đang chờ được đánh thức
   với kết quả từ chối.

Giao diện tạm là một `MessageBoxW` hai nút với `MB_DEFBUTTON2`. Hai chi tiết là
quyết định bảo mật chứ không phải thẩm mỹ:

- **Nút mặc định là "Không".** Người dùng gõ Enter theo phản xạ sẽ *từ chối*.
- **Không có nút thứ ba.** SECURITY.md §4.1 nói MUST NOT có "tin tưởng dù sao",
  vì nút đó tồn tại vì nó tiện rồi trở thành thứ người dùng bấm theo phản xạ.

## Consequences

**Giữ một request HTTP mở 120 giây là một cái giá thật**, và nó được trả có chủ
ý. Nó chiếm một trong 64 slot của trần connection đồng thời, và nó làm
`UltpRouter` phải `async` suốt chặng. Đổi lại: không có đường nào trong mã để
một phiên tự hoàn tất.

Cái giá ấy được chặn bằng hai trần: 120 giây cho một phiên, và tối đa **8** phiên
chờ cùng lúc (`PairingCoordinator.MaxPendingSessions`), trên nền rate-limit 5
phiên/phút/địa chỉ nguồn của SECURITY.md §4.3. Nếu không có hai trần ấy thì
"giữ request mở" chính là công thức của một cú từ chối dịch vụ rẻ tiền.

**Snappy nghiêm hơn Reference Host ở đây, và đó là đúng.** Reference Host là bản
tham chiếu về *wire format*; nó không có màn hình nên không thể mang nghĩa vụ
này. Khi hai bản khác nhau về hành vi mà không khác nhau về byte trên dây, chỗ
ghi lại khác biệt ấy là một ADR — nếu không, người đọc mã Rust sau này sẽ đọc ra
rằng Snappy đang làm thừa.

**Đo được.** `PairingFlowTests.Nguoi_dung_phia_Windows_tu_choi_thi_khong_ai_ghim_gi`
cho initiator gửi `confirmed: true` trong khi người dùng phía Windows bấm
"Không", và khẳng định **không bên nào** ghim gì. Trên bản AOT thật, 19/09/2026:
`snappy-interop pair` mở một phiên tới `Snappy.exe` đang chạy, cửa sổ
`Snappy — ghép đôi` hiện ra và đứng chờ, và không có `trusted-peers.json` nào
được tạo.

## Alternatives considered

**Tin `confirmed: true` như Reference Host.** Đơn giản hơn, không phải giữ
request, không phải `async`. Bị loại vì nó làm tiêu chí thứ tư của ticket thành
một câu không kiểm được, và làm SECURITY.md §7.3 sai với phía Windows.

**Trả `202 Accepted` ngay rồi báo kết quả qua một đường khác.** Đúng hơn về mặt
HTTP và không giữ connection. Bị loại vì openapi chốt hợp đồng là `200`/`403`, và
một đường thông báo thứ hai cần WebSocket (SPEC §15) mà Ticket 06 chưa dựng. Đây
là lựa chọn đáng xem lại nếu trần 8 phiên trở thành chỗ nghẽn thật.

**Hỏi người dùng *trước* khi tính SAS.** Bị loại vì lúc ấy chưa có gì để họ so —
sáu chữ số chỉ tồn tại sau khi hai bên đã trao khoá ephemeral.
