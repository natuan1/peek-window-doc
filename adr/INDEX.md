# ⚖️ Architecture Decision Records

Record of important architectural decisions — why we chose certain approaches.

## Overview

Architecture Decision Records (ADRs) document significant decisions and their rationale. They help:
- Understand WHY decisions were made
- Prevent repeating past discussions
- Track decision evolution over time
- Onboard new team members

## Decisions

| ADR | Title | Status | Date |
|-----|-------|--------|------|
| [0001](0001-brand-khac-service-mdns.md) | Thương hiệu ≠ tên service mDNS (Snappy giữ `_peek._tcp`) | Accepted | 2026-09-13 |
| [0002](0002-ui-stack-aot-spike-pass.md) | Xác nhận UI stack qua spike `aot-footprint` (3.05MB / 14.62MB) | Accepted | 2026-09-14 |
| [0003](0003-resumable-upload-104-h1-pass.md) | Resumable upload trên h1 — spike `104` PASS, advertise `resumableUpload` | Accepted | 2026-09-15 |
| [0004](0004-cau-truc-app-windows-bon-project.md) | Cấu trúc app Windows — bốn project, `Snappy.Interop` là biên Win32 duy nhất | Accepted | 2026-09-15 |
| [0005](0005-ci-cd-desktop-qua-jenkins-noi-bo.md) | CI/CD app desktop qua Jenkins nội bộ, không phải GitHub Actions | Accepted | 2026-09-16 |
| [0006](0006-dong-goi-velopack-cai-peruser.md) | Đóng gói Velopack — cài `PerUser`, gốc cài trùng gốc dữ liệu, KPI 15MB chuyển sang bộ cài | Accepted | 2026-09-16 |
| [0007](0007-windows-gia-nhap-interop-suite.md) | Windows gia nhập interop suite — Rust là oracle, phía vắng mặt phải nói ra | Accepted | 2026-09-17 |
| [0008](0008-discovery-qua-responder-in-box-windows.md) | Discovery đi qua responder in-box của Windows, và hostname phải là tên máy thật | Accepted | 2026-09-18 |
| [0009](0009-tls-1-3-ghim-cung-thu-hep-san-he-dieu-hanh-thuc-te.md) | TLS 1.3 ghim cứng — và sàn hệ điều hành nâng lên Windows 11 | Accepted | 2026-09-19 |
| [0010](0010-bang-nang-luc-khai-theo-hanh-vi-khong-theo-lo-trinh.md) | Bảng năng lực khai theo **hành vi hôm nay**, không theo lộ trình | Accepted | 2026-09-19 |
| [0011](0011-ghep-doi-khong-tu-confirm-responder-cho-nguoi-dung-cuc-bo.md) | Ghép đôi **không tự confirm** — responder giữ request mở chờ người dùng cục bộ | Accepted | 2026-09-19 |
| [0012](0012-cap-harness-do-hanh-vi-nen-tang-phai-chay-cho-tung-tls-stack.md) | Cặp harness đo **hành vi nền tảng** phải chạy cho từng TLS stack | Accepted | 2026-09-19 |
| [0013](0013-mi-an-khi-nghi-dung-luoi-xac-nhan-tep-qua-dragenter.md) | Mí **ẩn hẳn** khi Nghỉ, dựng OLE lười, xác nhận tệp qua `DragEnter` chứ không qua clipboard (phần Composition → 0014) | Accepted | 2026-09-30 |
| [0014](0014-mi-ve-bang-layered-window-khong-composition.md) | Mí vẽ bằng **layered window**, không Composition. Đen 0,8, không viền; vùng thả alpha 1 | Accepted | 2026-09-30 |
| [0015](0015-tempdrops-mot-thu-muc-moi-muc-khong-hoi-shell-luc-tha.md) | TempDrops **một thư mục `{Guid}` mỗi Mục**, ghi qua hàng rào S11/S16; trần 2 GiB + LRU; lúc thả **không** gọi `SHGetFileInfoW` (📐 2,3 MB RAM) | Accepted | 2026-10-01 |
| [0016](0016-kpi-ram-nen-do-bang-private-working-set.md) | KPI RAM nền = **private working set < 25 MB** (red line 30 MB theo working set tổng); luồng UI tắt IME, hộp chọn tệp chạy luồng riêng (📐 menu khay 25,5 → 22,3 MB) | Accepted | 2026-10-01 |
| [0017](0017-tinh-nang-nang-phan-shell-chay-trong-tien-trinh-con.md) | Tính năng nặng phần shell (**hộp chọn tệp**, **khay thẻ Shelf**) chạy trong **tiến trình Snappy con**; `ChildProcess` chung; icon thẻ là glyph (📐 hộp chọn tệp 63 → 23 MB) | Accepted | 2026-10-02 |

---

## ADR Format

**Filename:** `NNNN-kebab-case-title.md`

Example: `0001-authentication-strategy.md`, `0002-database-choice.md`

**Template:**
```markdown
# ADR-NNNN: [Title]

Date: YYYY-MM-DD
Status: Accepted | Proposed | Superseded | Deprecated

## Context
Why did we need to make this decision?
What was the problem or question?

## Decision
What did we decide to do?
Be concise and clear.

## Consequences
What are the implications?

### Positive Consequences
- Better performance
- Easier to maintain
- Etc.

### Negative Consequences (Trade-offs)
- More complex setup
- Higher costs
- Etc.

## Alternatives Considered
Why didn't we choose these?
- Option A: Why not? (trade-off analysis)
- Option B: Why not? (trade-off analysis)

## Related
- [Link to related ADR if any]
- [Link to relevant feature/architecture doc]

## Decision Log
- 2026-09-13: Accepted
- 2026-09-15: Clarified consequences
```

---

## How to Add an ADR

When making an important architectural decision:

1. Create file: `NNNN-kebab-case.md` (increment NNNN)
2. Fill in: Context → Decision → Consequences → Alternatives
3. Get team consensus (Status: Accepted)
4. Add entry to this INDEX.md
5. Update [CONTEXT.md](../CONTEXT.md) if it affects project overview
6. Link from relevant feature/architecture docs

---

## Decision Lifecycle

```
Proposed
   ↓
   ├─→ Accepted ← (team consensus)
   │      ↓
   │   Implemented
   │      ↓
   └─→ Superseded (by newer ADR)
   │      ↓
   └─→ Deprecated (no longer used)
```

---

## Navigation

- Back to [Main INDEX](../INDEX.md)
- See [Architecture](../architecture/)
- See [Features](../features/)
