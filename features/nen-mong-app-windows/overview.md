# Nền móng app Windows — tổng quan

**Trạng thái:** đã implement và **merge vào `main`** của `peekvn` (2026-09-15) · [Ticket 01 (#132)](https://github.com/natuan1/peekvn/issues/132)

## Tính năng này là gì

Viên đạn tracer cho toàn bộ chuỗi ticket Windows: một app chạy được thật, cài đặt bằng một file, sống ở khay hệ thống, và thoát sạch. Mọi ticket sau (mDNS, server ULTP, Mí, Shelf) dựng lên trên nó.

Ở mức người dùng, nó làm đúng bốn việc:

1. Chạy nền, không cửa sổ chính, không chớp console lúc khởi động.
2. Hiện icon ở khay hệ thống.
3. Bấm vào icon mở một bảng trạng thái nhỏ, nói app đang chạy và chỉ đường thoát.
4. Bấm Thoát là tiến trình biến mất, không sót icon ma ở khay.

## User story

> As a người dùng Windows, I want to app chạy nền tiêu thụ ~0% CPU khi tôi không tương tác, so that nó không làm máy tôi chậm. — Spec #1, story 36

Kèm hai lời hứa nền, không phải story nhưng là ràng buộc cứng của sản phẩm:

- Bộ cài < 15MB, RAM nền < 25MB (red line 30MB).
- Cài user-space, không UAC, không vào Program Files.

## Phạm vi

**Trong:** solution bốn project, publish Native AOT ra một exe, icon khay, bảng trạng thái, Per-Monitor V2 DPI, khoá một-bản-đang-chạy, nhật ký chẩn đoán, hàng rào CI.

**Ngoài:** bộ cài Velopack và auto-update (Ticket 02), mDNS (04), server ULTP (05), Mí (10/11), Shelf (12/13), ghép đôi (06), bản quyền (14), onboarding (17).

## Nghiệm thu

| Tiêu chí | Trạng thái | Bằng chứng |
|---|---|---|
| `dotnet publish` ra một exe duy nhất | ✅ | 1,69MB; thư mục publish chỉ có `Snappy.exe` + symbol native |
| Icon khay mở bảng trạng thái; Thoát không sót tiến trình | ✅ | `Shell_NotifyIcon` trả thành công; `WM_CLOSE` → 0 tiến trình sót |
| Per-Monitor V2 DPI | ✅ | `AreDpiAwarenessContextsEqual(…, PER_MONITOR_AWARE_V2)` = true |
| Working set < 25MB | ✅ | 12,51MB lúc nghỉ; 17,23MB sau khi mở bảng trạng thái |
| Thêm lại icon khay sau khi Explorer sống lại | ✅ | Gửi thông điệp `TaskbarCreated` → app sống, ghi đúng nhật ký, bảng vẫn mở được |
| Chạy trên Windows 10 1809 sạch | 🕓 **Treo** | Chưa có máy 1809. Quyết định 2026-09-15: ưu tiên Windows 11 trước, thêm phép thử này khi có máy. Floor sản phẩm **không đổi** — `SupportedOSPlatformVersion` vẫn ghim 10.0.17763.0 để chặn API mới hơn bò vào mã |
| CI xanh | ✅ | Jenkins nội bộ, job `snappy-windows` build #4 SUCCESS trên `main` 2026-09-16 (~65 giây): 37 test, publish AOT 1,65MB, chạy thử app thật 11,19MB, thoát sạch. Không phải GitHub Actions — [ADR-0005](../../adr/0005-ci-cd-desktop-qua-jenkins-noi-bo.md) |

## Việc còn phải kiểm bằng tay (trên Windows 11)

Jenkins lo phần chạy được; ba thứ dưới đây cần mắt người:

1. Nhìn thấy icon ở khay và bấm được vào nó — **xem [bài học 2026-09-15](../../lessons-learned.md) về việc Windows 11 giấu icon mới**.
2. Đưa con trỏ sang màn hình có mức scale khác rồi bấm icon khay: bảng phải hiện sắc nét ở đúng tỉ lệ màn hình đó.
3. Kết thúc `explorer.exe` thật trong Task Manager rồi chạy lại — icon phải tự quay lại khay. (Nhánh xử lý đã kiểm bằng thông điệp; phần chưa kiểm là Explorer thật có phát đúng lúc không.)

**Treo tới khi có máy:** chạy trên Windows 10 1809 sạch, không cài .NET runtime.

## Tài liệu liên quan

- [Workflow](workflow.md) — người dùng và hệ thống đi qua những bước nào
- [Implementation](implementation.md) — bốn project, file chính, cái bẫy đã gặp
- [Testing](testing.md) — seam nào tự động, seam nào bằng tay
- [ADR-0004](../../adr/0004-cau-truc-app-windows-bon-project.md) — vì sao bốn project và vì sao Interop là biên duy nhất
