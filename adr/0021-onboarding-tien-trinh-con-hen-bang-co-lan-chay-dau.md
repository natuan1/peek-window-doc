# ADR-0021: Onboarding chạy trong tiến trình con, hẹn bằng cờ lần-chạy-đầu của Velopack, cài đặt trong `settings.ini`

Date: 2026-10-04
Status: Accepted

## Context

Ticket 17 ([natuan1/peekvn#148](https://github.com/natuan1/peekvn/issues/148)) đòi ba bước hướng dẫn "chỉ hiện lần đầu cài đặt", "bỏ qua được ở mọi bước", "không chặn chức năng khác", cùng hai lựa chọn phải sống qua khởi động lại: bên của Mí và phím tắt mở panel. Trước ticket này Snappy **không có** chỗ lưu lựa chọn nào của người dùng.

Ba sự thật đo được ràng buộc cách làm:

1. Bước 1 cần một **nguồn kéo**. 📐 02/10/2026 (ADR-0017, bài học 200): nguồn kéo đầu tiên của một tiến trình nạp 18 DLL kéo-thả của Windows 11, không bao giờ nhả.
2. "Lần đầu cài đặt" là trạng thái máy mà chỉ bộ cài biết. Velopack đặt `VELOPACK_FIRSTRUN` khi Setup mở app (`OnFirstRun`). 📐 Setup trên máy **đã cài** hỏi ghi đè và chờ người bấm — đó là đường cài lại.
3. Chủ dự án chốt 04/10/2026: "panel" là khay Shelf (story 38), QR tới `https://snappy.vn/tai`, Mí phải đối xứng 65–95%.

## Decision

1. **Cửa sổ onboarding sống ở tiến trình con** `Snappy.exe --onboarding <tệp mẫu>`, cùng khuôn `ChildProcess` với khay Shelf. Cha là chủ cài đặt: con báo lựa chọn, cha áp, lưu, **thử** đăng ký phím, rồi gửi lại kết quả thật để con vẽ.
2. **Hẹn bằng cờ lần-chạy-đầu**: `OnFirstRun` → `settings.ini` ghi `onboarding=pending`; hẹn chỉ gỡ (`done`) khi người dùng bấm Xong hoặc bỏ qua. Snappy chạy không qua bộ cài (dev, CI) không bao giờ có hẹn. Bản cài từ trước Ticket 17 cập nhật lên cũng không — họ mở được từ menu khay "Hướng dẫn bắt đầu".
3. **`settings.ini` dạng `key=value`**, dưới gốc dữ liệu Snappy: đọc rộng lượng (BOM, CRLF, `#`; dòng lạ chỉ dòng ấy về mặc định — bài học 175), ghi qua tệp tạm. Không JSON: AOT cần source-gen, và ba khoá không đáng.
4. **Phím tắt Shelf đặt sẵn** `Win+Alt+S` (mặc định) / `Ctrl+Alt+S` / tắt, đăng ký ở tiến trình nền mỗi lần khởi động.
5. **Bên của Mí là một tham số của bố cục**, không phải trạng thái toàn cục: `MiLayout`, `CapsuleLayout.Home`, `ShelfTrayLayout` nhận `MiSide`; tiến trình con nhận nó qua ống (capsule) hay đối số (khay Shelf).
6. **Mã QR tự viết** (`QrCode.cs`) — không thêm thư viện vào exe (giá theo sự có mặt, bài học 209); chứng minh bằng ZXing ở project test và bằng ảnh chụp màn hình trong nghiệm thu.

## Consequences

- 📐 Tiến trình nền trả **+0,08 MB private, 0 DLL** cho một lượt mở-rồi-bỏ-qua onboarding; bộ cài +0,05 MB.
- `Win+Alt+S` được đăng ký cho mọi bản cài, kể cả người chưa qua onboarding. Một app khác cần đúng phím ấy sẽ thấy nó bị giữ; người dùng đổi hay tắt ở "Hướng dẫn bắt đầu".
- Phím đã chọn mà bị giữ vẫn được lưu: mỗi lần khởi động thử lại (app kia có thể đã nhả), và menu khay không hứa phím ấy. Cửa sổ onboarding là chỗ duy nhất nói câu "bị giữ".
- Khay Shelf đang mở không dời khi đổi bên; lần mở sau mới theo.
- Phím `Win+Alt+V` của lịch sử clipboard vẫn cố định — chưa có giao diện đổi nó.
- Mọi lựa chọn bền về sau (cài đặt khác) có chỗ để sống: thêm khoá vào `AppSettings`.

## Related

- [ADR-0017](0017-tinh-nang-nang-phan-shell-chay-trong-tien-trinh-con.md) — tiến trình con cho tính năng nặng phần shell
- [ADR-0006](0006-dong-goi-velopack-cai-peruser.md) — Velopack, `OnFirstRun`
- [ADR-0020](0020-lich-su-clipboard-loc-theo-dau-o-tien-trinh-nen.md) — phím `Win+Alt+V`
- [Onboarding 3 bước trên Windows](../features/onboarding-windows/overview.md)
- `peekvn/docs/bai-hoc.md` 219–221
