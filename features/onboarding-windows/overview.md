# Onboarding 3 bước trên Windows (Ticket 17)

Ticket: [natuan1/peekvn#148](https://github.com/natuan1/peekvn/issues/148) · Spec cha: [#131](https://github.com/natuan1/peekvn/issues/131) story 37, 38 · Quyết định: [ADR-0021](../../adr/0021-onboarding-tien-trinh-con-hen-bang-co-lan-chay-dau.md) · Mã: `peekvn/apps/windows` (README mục "Onboarding 3 bước — Ticket 17")

**Trạng thái:** ✅ nghiệm thu 2026-10-04 từ bộ cài thật (`ci/check-onboarding.ps1` PASS A→G), nhánh `feat/148-onboarding`.

## User story

> As a người dùng lần đầu, I want to được hướng dẫn 3 bước ngắn (kéo thử file mẫu, chọn vị trí Mí, quét QR tải app mobile), so that trong 60 giây tôi tự làm được thao tác cốt lõi. — Spec #131, story 37

> As a người dùng Windows, I want to mở bảng đầy đủ bằng phím tắt khi Mí đang ẩn, so that mọi tính năng dùng được không qua Mí. — Spec #131, story 38

## Người dùng được gì

Cài xong lần đầu, cửa sổ "Snappy — Bắt đầu" tự hiện:

1. **Thử kéo một tệp lên Mí** — một thẻ "Snappy - tệp mẫu.txt" ngay trong cửa sổ; kéo nó lên cạnh trên, Mí nở, thả vào ô Shelf → "Xong! Tệp mẫu đã nằm trong Shelf".
2. **Mí ở bên nào, mở Shelf bằng phím nào** — Bên trái / Bên phải; phím `Win+Alt+S`, `Ctrl+Alt+S` hoặc không dùng. Phím bị app khác giữ thì cửa sổ nói thẳng và bảo chọn phím khác.
3. **Cài Snappy trên điện thoại** — mã QR tới `https://snappy.vn/tai`, và **Ghim Snappy ra thanh tác vụ** (Windows 11 giấu icon mới sau dấu ^).

"Bỏ qua hướng dẫn", ✕ hay Esc ở bước nào cũng được. Cửa sổ không modal, không topmost: Mí, khay Shelf, menu khay vẫn chạy khi nó mở. Menu khay **"Hướng dẫn bắt đầu"** mở lại nó.

## Quyết định của chủ dự án (2026-10-04)

| Câu hỏi | Chốt |
|---|---|
| "Phím tắt mở panel" là panel nào | **Khay Shelf** — đúng story 38; chọn trong vài tổ hợp đặt sẵn |
| QR trỏ đâu | `https://snappy.vn/tai` — trang ấy (chuyển App Store/Play) thuộc spec landing page |
| Mí bên phải nằm đâu | **Gương** của bên trái: 65%–95%, chừa 5% mép phải |

## Lệch kế hoạch (đã đo)

| Kế hoạch | Thực tế | Vì sao |
|---|---|---|
| Phím tắt "tùy biến" | ba lựa chọn đặt sẵn | 📐 `RegisterHotKey`: `Win+Alt+S`, `Ctrl+Alt+S` nhận; `Win+Shift+S`, `Ctrl+Alt+Space` trả 1409 — một ô ghi phím tuỳ ý mời chọn đúng phím đã có chủ |
| Ghim icon là "mẹo" | một mục có tiêu đề riêng ở bước 3 | lessons-learned 2026-09-15: app vừa cài mà khay trống là app chưa tồn tại |

## Phạm vi

**Trong:** cửa sổ ba bước, cài đặt bền `settings.ini`, Mí/capsule/khay Shelf theo bên, phím tắt mở Shelf, mục menu khay mở lại.

**Ngoài:**
- Trang `snappy.vn/tai` và nội dung của nó (spec landing page).
- Đổi phím `Win+Alt+V` của lịch sử clipboard — vẫn cố định (ADR-0020).
- Đo "60 giây": chưa có phép đo thời gian người dùng thật.
