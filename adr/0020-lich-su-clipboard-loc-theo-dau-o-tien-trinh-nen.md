# ADR-0020: Lịch sử clipboard lọc theo dấu, sống trong RAM của tiến trình nền

Date: 2026-10-04
Status: Accepted

## Context

M4 (Ticket 16, [natuan1/peekvn#147](https://github.com/natuan1/peekvn/issues/147)) hứa: lưu những gì người dùng copy trong phiên, dán lại bằng phím tắt, và mật khẩu từ trình quản lý mật khẩu không bao giờ vào lịch sử. Kế hoạch (mục 3.2) ghi sẵn cơ chế: `AddClipboardFormatListener`, phím `Win+Shift+V`, ba dấu `Clipboard Viewer Ignore` / `ExcludeClipboardContentFromMonitor` / `CanIncludeInClipboardHistory`.

Ba phép đo ngày 2026-10-04 làm lệch kế hoạch:

1. `RegisterHotKey` trả **1409** cho `Win+Shift+V` (và `Win+V`) — Windows giữ chúng.
2. "Copy Password" của **KeePassXC 2.7.12** đặt `ExcludeClipboardContentFromMonitorProcessing` (tên có tài liệu của Windows), `CanIncludeInClipboardHistory=0`, `CanUploadToCloudClipboard` — không có tên cụt của ticket.
3. `keepassxc-cli clip` không đặt dấu nào.

ADR-0017 cô lập tính năng **nặng phần shell** trong tiến trình con. Bảng dán nhanh không kéo-thả, không gọi shell.

## Decision

1. **Lọc theo dấu, đóng cửa khi không chắc.** Chặn nếu có `…FromMonitorProcessing`, `…FromMonitor` hoặc `Clipboard Viewer Ignore`, hoặc nếu `CanIncludeInClipboardHistory` = 0 hay không đọc được. Bằng 1 thì app cho phép, và lượt copy được giữ. Dấu được hỏi **trước** khi đọc chữ; một lượt bị chặn thì chữ không bao giờ vào RAM của Snappy. Không đăng ký được tên một dấu thì không đọc chữ.
2. **Chỉ RAM, chỉ phiên.** Không có đường xuống đĩa. Nhật ký chỉ ghi độ dài, exe nguồn và tên dấu. `ClipboardEntry.ToString()` không in chữ.
3. **Phím `Win+Alt+V`.** Đăng ký được, và không đụng phím của app phổ biến (`Ctrl+Shift+V` là "dán chữ thuần"; `Ctrl+Alt+V` là AltGr+V). Không đăng ký được thì bảng vẫn mở từ menu khay.
4. **Bảng dán nhanh ở tiến trình nền**, vẽ GDI như bảng trạng thái. Phím tắt cần bảng hiện tức thì. 📐 Giá đo được: nghỉ 5,12 → 5,44 MB private sau khi dùng bảng, không DLL nặng.
5. **Dán = đặt clipboard + `SendInput` Ctrl+V**, chỉ khi cửa sổ đích thật sự đang foreground. Không chắc đích thì không gõ, và bong bóng bảo người dùng bấm Ctrl+V.

## Consequences

- Lời hứa "mật khẩu không bao giờ vào lịch sử" chỉ giữ với app **tự đánh dấu**. `keepassxc-cli` và mọi app không đánh dấu thì bị giữ, như `Win+V` của Windows. Không có cách trung thực nào nhận ra mật khẩu từ nội dung.
- Bảng ở tiến trình nền nạp `MSCTF`, `TextShaping`, `wtdccm`, `OLEAUT32` sau lần dùng đầu tiên (+0,3 MB private, +1,8 MB tổng), cùng họ với menu khay. Nếu biên red line tổng hẹp lại, đây là ứng viên chuyển sang tiến trình con.
- Phím tắt chưa tùy biến được. Onboarding (Ticket 17) có bước "thiết lập phím tắt".
- 1Password và Bitwarden chưa được đo (cần tài khoản); chủ dự án đưa ra ngoài phạm vi Ticket 16 ngày 2026-10-04.

## Related

- [ADR-0016](0016-kpi-ram-nen-do-bang-private-working-set.md) — KPI RAM private
- [ADR-0017](0017-tinh-nang-nang-phan-shell-chay-trong-tien-trinh-con.md) — khi nào cần tiến trình con
- [ADR-0019](0019-ban-quyen-verify-tai-may-mang-trong-tien-trinh-con.md) — cổng gate `Entitlements`
- [Lịch sử clipboard trên Windows](../features/lich-su-clipboard-windows/overview.md)
- `peekvn/docs/bai-hoc.md` 215–218
