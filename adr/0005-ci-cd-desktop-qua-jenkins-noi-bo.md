# ADR-0005: CI/CD app desktop chạy qua Jenkins nội bộ, không phải GitHub Actions

Date: 2026-09-16
Status: Accepted

## Context

Sau [Ticket 01](https://github.com/natuan1/peekvn/issues/132), app Windows có hàng rào CI viết bằng GitHub Actions nhưng **chưa bao giờ chạy xanh một lần nào**: tài khoản Actions bị chặn vì lý do thanh toán từ 2026-09-15, mọi job kể cả Rust/Swift có sẵn đều không khởi động được. `ci.yml` đã phải chuyển sang `workflow_dispatch` để không đẻ ra một run đỏ vô nghĩa mỗi commit.

Lý do tiền bạc là lý do trước mắt. Lý do thật sâu hơn, và nó không biến mất khi thanh toán thông:

**Runner đám mây không có phiên đồ hoạ.** Snappy là app khay hệ thống — không cửa sổ chính, toàn bộ giao diện nằm ở khay và ở một bảng popup. Phép nghiệm thu duy nhất có sức nặng là *khởi động app thật rồi quan sát hành vi bên ngoài*: cửa sổ nhận thông điệp có tồn tại không, icon khay có thêm được không, RAM nền bao nhiêu, `WM_CLOSE` có dọn sạch tiến trình không. Trên `windows-latest` của GitHub không có Explorer, không có khay hệ thống, và không có desktop để cửa sổ sống trên đó.

Điều này không phải suy luận. Bản đầu tiên của Ticket 01 **publish sạch rồi chết ngay lúc khởi động** (mã 1400 — `DefWindowProcW` nhận HWND rỗng trong `WM_NCCREATE`). `dotnet build` xanh, `dotnet publish` xanh, và exe không chạy được. Một hàng rào chỉ biết biên dịch sẽ cho lỗi đó đi thẳng tới người dùng.

Ngoài ra Native AOT cần MSVC linker, và bộ cài Velopack ở Ticket 02 sẽ cần ký số — cả hai đều dễ chịu hơn trên một máy Windows thật mà mình kiểm soát.

## Decision

**CI của app desktop chạy trên Jenkins nội bộ** (`http://192.168.1.235:9096`, job `snappy-windows`), với **agent là máy dev Windows 11 thật** (`win11-snappy`, nhãn `windows dotnet-aot msvc`).

Pipeline sáu bước, khai báo trong `apps/windows/Jenkinsfile` (pipeline-as-code, không phải cấu hình trong UI):

1. Lấy mã
2. Biên dịch
3. Test — xuất JUnit để Jenkins báo cáo từng test, không chỉ "bước này đỏ"
4. Publish Native AOT — gọi đúng `publish.cmd` mà người thật gõ, không chép lại lệnh
5. Hàng rào KPI — một exe duy nhất, dưới trần dung lượng
6. **Chạy thử app thật** — bước không tồn tại được trên runner đám mây

**Chọn nhãn chứ không chọn tên máy** trong `agent { label ... }`: thêm máy build thứ hai chỉ cần gắn nhãn.

**Hai script hàng rào nằm trong repo** (`ci/check-artifacts.ps1`, `ci/smoke-test.ps1`) chứ không nhúng trong Jenkinsfile, nên chạy được y hệt trên máy dev. Cả bốn đường đỏ của chúng đã được ép cho đỏ một lần — một hàng rào chưa bao giờ đỏ là một hàng rào chưa được chứng minh.

**Cấp quyền đọc repo private bằng deploy key SSH chỉ-đọc** cho riêng `peekvn`, không dùng token cá nhân: phạm vi hẹp nhất có thể, thu hồi được bằng một cú bấm, và không có đường nào từ Jenkins ghi ngược vào repo.

**GitHub Actions giữ lại ở chế độ `workflow_dispatch`**, không xoá: nó là đường dự phòng nếu Jenkins nội bộ chết, và là nơi duy nhất kiểm được các job Rust/Swift trên Linux/macOS — hai nền tảng mà agent Windows không thay thế được.

## Consequences

**Được:**
- Nghiệm thu đi qua đúng thứ người dùng đi qua: app chạy thật, không chỉ biên dịch thật. Bước 6 bắt được đúng loại lỗi mà bốn bước trước để lọt.
- Một lượt ~65 giây, không hàng đợi, không hạn mức, không hoá đơn.
- Native AOT dùng đúng bộ MSVC đã kiểm trên máy dev; không phải đoán phiên bản của runner.

**Mất — và đây là phần phải nhìn thẳng:**
- **Máy build là máy dev.** Tắt máy là CI chết. Không có bản dựng nào tái lập được từ một image sạch, nên một phụ thuộc lặng lẽ cài trên máy này sẽ không ai phát hiện cho tới khi dựng trên máy khác.
- **Bước "chạy thử app thật" cần phiên đồ hoạ**, nên agent phải chạy trong phiên người dùng đã đăng nhập, không chạy được như dịch vụ session 0. Đổi lại đúng thứ khiến bước đó có giá trị.
- **Không có hàng rào cho Rust/Swift/protocol** cho tới khi Actions sống lại. Bảng lệnh chạy tay trong `peekvn/AGENTS.md` §1 là thứ duy nhất giữ chỗ đó.
- **Jenkins nội bộ không nhận được webhook** từ GitHub (không lộ ra Internet), nên phải hỏi SCM định kỳ — build chạy trễ vài phút sau khi push.
- Thêm một hệ thống phải bảo trì: plugin, credential, agent, và một mật khẩu admin nữa.

## Alternatives Considered

- **Chờ thanh toán GitHub Actions rồi quay lại**: giải quyết được tiền, không giải quyết được phiên đồ hoạ. Bước 6 vẫn không chạy được, tức tiêu chí quan trọng nhất vẫn không có hàng rào.
- **Self-hosted runner của GitHub Actions trên chính máy này**: có phiên đồ hoạ, giữ nguyên `ci.yml`. Nhưng vẫn cần tài khoản Actions sống, và self-hosted runner cho repo private là bề mặt rủi ro mà GitHub khuyến cáo. Đáng xem lại khi thanh toán thông.
- **Chỉ chạy tay trên máy dev**: rẻ nhất, và là thứ đang có trước ADR này. Nó hỏng vì cùng một lý do mọi kỷ luật thủ công đều hỏng — nó phụ thuộc vào người nhớ chạy.

## Related

- [ADR-0004](0004-cau-truc-app-windows-bon-project.md) — cấu trúc app Windows
- [Nền móng app Windows — testing](../features/nen-mong-app-windows/testing.md)
- [Spec #131](https://github.com/natuan1/peekvn/issues/131)
