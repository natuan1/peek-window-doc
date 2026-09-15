# Snappy — Ngữ cảnh miền (Domain Context)

Snappy là bộ tiện ích desktop Windows kèm app mobile (iOS/Android đã phát hành), lõi là truyền file local đa nền tảng qua giao thức ULTP. Tài liệu này là bảng thuật ngữ chuẩn của miền nghiệp vụ — mọi tài liệu và mã nguồn phải dùng đúng các từ ở đây.

## Language

### Sản phẩm & bản phát hành

**Snappy**:
Tên sản phẩm (chốt 2026-09-13, tái xác nhận 2026-09-13 sau khi phát hiện mobile chạy dưới tên cũ). Bộ tiện ích Windows + app mobile; mobile sẽ đổi label theo.
_Avoid_: peek-window (chỉ là tên repo), Peek (tên cũ trên mobile — sẽ thay; riêng service mDNS `_peek._tcp` giữ nguyên vĩnh viễn vì là định danh kỹ thuật ẩn với người dùng)

**Free**:
Bản miễn phí; truyền file không giới hạn vĩnh viễn là ràng buộc cứng.
_Avoid_: bản thường, basic

**Pro**:
Bản mua một lần trọn đời, mở đa ngăn Shelf, ghim, clipboard không giới hạn và các Module nâng cao.
_Avoid_: premium, VIP, subscription

**Bản lẻ**:
Bản build cùng một mã nguồn nhưng có ít Module đăng ký hơn.

### Bề mặt giao diện

**Mí**:
Dải giao diện mảnh neo ở cạnh trên màn hình; nơi kéo-thả để giữ tạm hoặc gửi đi.
_Avoid_: notch, panel, toolbar

**Trạng thái Mí**:
Một trong năm tình trạng: Nghỉ (0), Gợi ý (1), Sẵn sàng nhận (2), Nhắm đích (3), Đang chạy (4).

**Đích thả**:
Đích hiện trên Mí ở trạng thái Sẵn sàng nhận: ô Shelf, một thiết bị, hoặc nút mở danh sách thiết bị đầy đủ.
_Avoid_: target, destination

### Shelf

**Shelf**:
Module khay giữ tạm (M1) — nơi gom Mục để giữ lại hoặc gửi đi sau.
_Avoid_: clipboard (khi nói về khay file), tray

**Ngăn**:
Một danh sách Mục trong Shelf. Free có đúng 1 ngăn; Pro có nhiều ngăn kèm ghim. (chốt 2026-09-13)

**Mục**:
Một phần tử trong Ngăn: file, thư mục hoặc text được giữ tạm.

**File ảo**:
File chưa tồn tại trên đĩa tại lúc bị kéo (đính kèm Outlook, ảnh trên web); Snappy phải trích xuất nội dung của nó trước khi dùng được.

**TempDrops**:
Bộ đệm tạm nơi File ảo được trích xuất. Quy tắc dọn: xóa khi Mục bị bỏ khỏi Ngăn; trần dung lượng thì xóa Mục cũ nhất theo LRU kèm cảnh báo; xóa sạch khi người dùng yêu cầu. Mục sau khi gửi thành công vẫn nằm lại Ngăn. (chốt 2026-09-13)

### Module

**Module**:
Một tiện ích độc lập trong Snappy: M1 Shelf, M2 Truyền file, M3 Media & pin Bluetooth, M4 Clipboard history, M5 Độ sáng, M6 Nén, M7 Đổi định dạng. Mọi Module dùng được không qua Mí (phím tắt, khay hệ thống).

**Cầu text/link**:
Tính năng gửi đoạn văn bản hoặc URL sang thiết bị khác qua ULTP.
_Avoid_: cầu nối text và link (cũ), text bridge

### Truyền file

**ULTP**:
Universal Local Transfer Protocol — giao thức truyền file local đa nền tảng (mDNS + HTTPS/TLS + REST, tín hiệu sự kiện bằng long-poll `?wait=`). Mọi nền tảng cùng nói một giao thức; TRS v1 đã đóng băng, nguồn chuẩn hiện hành là `protocol/SPEC.md`.
_Avoid_: "iOS protocol", "Windows protocol", WebSocket (đã loại bằng quyết định 2026-09-13 — xem mục quyết định kỹ thuật bên dưới)

**Ghép đôi**:
Quy trình thiết lập quan hệ tin cậy giữa hai thiết bị, xác minh bằng mã SAS.
_Avoid_: connect, login

**SAS**:
Mã 6 chữ số mà hai thiết bị cùng hiển thị để người dùng xác nhận Ghép đôi.
_Avoid_: OTP, mã kích hoạt

**Thiết bị tin cậy**:
Thiết bị đã hoàn thành Ghép đôi và được pin định danh (khóa công khai + SPKI).

**Tự nhận**:
Nhận file ngay từ Thiết bị tin cậy mà không hiện hộp xác nhận. (chốt 2026-09-13)
_Avoid_: auto-download

**Phiên truyền**:
Một lần gửi gồm một hoặc nhiều Mục giữa hai thiết bị; có vòng đời OFFERED → ACCEPTED → … → COMPLETED theo ULTP.
_Avoid_: session (mơ hồ), job

### Bản quyền

**Key**:
Mã bản quyền trọn đời của Pro, định dạng `SNPY-XXXX-XXXX-XXXX-XXXX`; một Key sở hữu tối đa 5 Slot. (chốt 2026-09-13)
_Avoid_: serial, license code

**Slot**:
Một chỗ dành cho một máy trong Key. Chủ Key tự gỡ Slot qua portal bằng cặp (Key + email mua), không cần ai duyệt. (chốt 2026-09-13)

**Kích hoạt**:
Gán máy hiện tại vào một Slot trống của Key; xong thì máy dùng Pro offline vĩnh viễn, chỉ re-validate thầm lặng khi có mạng. (chốt 2026-09-13)
_Avoid_: đăng nhập, đăng ký tài khoản

### Đo lường & riêng tư

**Heartbeat ẩn danh**:
Tín hiệu check-update định kỳ dùng để ước lượng số bản cài còn hoạt động (suy ra tỷ lệ gỡ); không kèm dữ liệu nhận dạng cá nhân. (chốt 2026-09-13)
_Avoid_: telemetry, tracking

## Quyết định kỹ thuật đã xác minh (2026-09-13)

- **Server Windows: HTTP/1.1-only tự viết trên TcpListener + SslStream** — không Kestrel, không WebSocket (dùng long-poll `?wait=` như cả hệ sinh thái); không client nào cần h2; mô hình đã được Android chứng minh sống thật. (chốt 2026-09-13)
- **Resumable upload phone→PC: spike-gated** — spike tự gửi `104` qua HTTP/1.1 đo CFNetwork; thất bại thì không vỡ gì (tự rơi về single-shot, đúng hành vi đo được trên iPhone thật); **chưa spike thành công thì không advertise** `resumableUpload`. (chốt 2026-09-13) **Spike PASS 2026-09-15**: CFNetwork xử lý nổi `104` giữa luồng h1 thuần đến 50MB, 6/6 lượt `201` — advertise `resumableUpload: ["httpbis-interop-6"]`, server chỉ gửi `104` trên TLS — xem [ADR-0003](adr/0003-resumable-upload-104-h1-pass.md).
- **UI stack: C# Native AOT + Win32 + Windows.UI.Composition/Direct2D (CsWinRT)** — WPF không AOT được; WinUI 3 phá rào dung lượng (runtime 116MB); Kestrel bỏ. KPI bộ cài quay về **<15MB**; RAM mục tiêu <25MB, red line 30MB. (chốt 2026-09-13) **Spike PASS 2026-09-14**: exe 3.05MB, working set 14.62MB (private 4.65MB), Composition sống dưới AOT — xem [ADR-0002](adr/0002-ui-stack-aot-spike-pass.md).
- **Floor hệ điều hành: Windows 10 1809** (gồm LTSC 2019); test trên Win10 21H2/22H2 + LTSC 2019 + Win11. (chốt 2026-09-13)
- **Thương hiệu thống nhất "Snappy"**, mobile đổi label; service mDNS giữ `_peek._tcp` — xem [ADR-0001](adr/0001-brand-khac-service-mdns.md). (chốt 2026-09-13)
- **Windows app code nằm trong monorepo `peekvn\apps\windows\`** — dùng chung schemas, interop suite, Rust oracle; `peek-window` là workspace kế hoạch/tài liệu. (chốt 2026-09-13)
- **Trạng thái Mí 1 "Gợi ý" GIỮ NGUYÊN** — không cần phương án dự phòng trong kế hoạch. Cơ chế khả thi không cần mouse hook: `SetWinEventHook` out-of-context nghe cửa sổ drag-image (`SysDragImage`) làm tín hiệu chính; xác nhận "đang kéo FILE" bằng một lần `OleGetClipboard` + `CFSTR_INDRAGLOOP` + `CF_HDROP`/`FileGroupDescriptorW`; polling 2 tầng làm fallback; strip mỏng luôn là drop target thật (magnet strip, không click-through, `WS_EX_NOACTIVATE`). Prior art đã ship: yeet, DropCast, Bytover, ShelfLife.
- **Phạm vi mobile đi cùng Windows 1.0**: đổi thương hiệu + store-readiness (Apple trả phí $99/năm, App Group, dọn applicationId) + auto-accept toggle; text/url mobile để 1.1. (chốt 2026-09-13)

## Thực trạng app Windows (2026-09-15)

Nền móng đã chạy được — [Ticket 01](https://github.com/natuan1/peek-window-doc/issues/2), nhánh `ticket/01-skeleton-aot` của `peekvn`. Chi tiết: [Nền móng app Windows](features/nen-mong-app-windows/overview.md).

- **Bốn project** trong `peekvn/apps/windows/`: `Snappy.Shared` (DTO, đường dẫn), `Snappy.Interop` (biên Win32 duy nhất), `Snappy.Protocol` (ULTP — còn rỗng, do Ticket 04/05 đắp vào), `Snappy.Core` (project duy nhất sinh exe) — xem [ADR-0004](adr/0004-cau-truc-app-windows-bon-project.md).
- **Chạy được**: app chạy nền, icon khay hệ thống, bảng trạng thái nhỏ, menu Thoát dọn sạch tiến trình, chỉ một bản chạy mỗi người dùng.
- **Số đo thật** (publish Native AOT, máy dev): exe **1,69MB**, working set lúc nghỉ **12,51MB**, thư mục publish chỉ có `Snappy.exe` + symbol native. Cả ba đều dưới KPI và đều có hàng rào CI.
- **Chưa có**: mDNS, server ULTP, Mí, Shelf, ghép đôi, bản quyền, bộ cài Velopack — theo đúng thứ tự ticket.
- **Chưa nghiệm thu được**: chạy trên Windows 10 1809 sạch (không có máy); CI xanh (tài khoản GitHub Actions đang bị chặn vì thanh toán — mọi job, kể cả Rust/Swift có sẵn, đều không khởi động được từ 2026-09-15).
