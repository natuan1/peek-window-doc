# CLAUDE.md

Configuration and conventions for AI agent work on this project.

## Overview

This workspace is the **planning & documentation workspace** for Snappy (Windows desktop app). The application source code does **not** live here — per the decision of 2026-09-13, Snappy's Windows code lives in the `peekvn` monorepo at `D:\code\peekvn\apps\windows\` (https://github.com/natuan1/peekvn), sharing schemas, the interop suite, and the Rust protocol oracle with iOS/Android.

- **`docs/`**: Documentation, requirements, and issue tracking — itself a git repo, synced to `github.com/natuan1/peek-window-doc`. Serves as the **single source of truth** for business logic, technical decisions, and project state.
- **`app/`**: Placeholder only. Do not put source code here.

Implementation in `peekvn\apps\windows\` must always align with `docs/`.

## Agent Skills

### Issue Tracker

Issues are tracked in **GitHub Issues**: `natuan1/peek-window-doc` (specifications and decisions, source of truth), `natuan1/peek-window` (workspace-level tracking), and `natuan1/peekvn` (implementation code and day-to-day implementation issues). See `docs/agents/issue-tracker.md`.

### Triage Labels

Triage workflow uses five Vietnamese labels to organize work. See `docs/agents/triage-labels.md`.

### Domain Docs

All domain documentation lives in `docs/CONTEXT.md` (main context) and `docs/adr/` (architecture decisions). This is single-context layout. See `docs/agents/domain.md`.

## Agent Communication Rules

### Language
**Agents must communicate in Vietnamese (Tiếng Việt).**

All responses, explanations, documentation updates, and communications from agents should be in Vietnamese. This applies to:
- Chat responses and explanations
- Comments in code or documentation
- Issue descriptions and PR titles
- ADR titles and content
- CONTEXT.md updates

### Lessons Learned Protocol

**Bài học có hai nửa. Nửa đọc đi trước nửa ghi.**

#### Nửa 1 — ĐỌC, trước khi viết dòng mã đầu tiên (bắt buộc)

Trước mỗi lần `/implement`, mở **cả hai** file bài học và đọc phần đầu:

- `peekvn/docs/bai-hoc.md` — §Luật thường trực (5 họ lỗi lặp lại), rồi `git grep -n "^## " docs/bai-hoc.md` để quét tiêu đề tìm vùng sắp chạm.
- `docs/lessons-learned.md` — ngắn, đọc hết.

Một bài học khớp mà vẫn cố tình làm khác thì **phải nói ra lý do trong báo cáo**, không im lặng đi qua.

**Vì sao luật này tồn tại:** trước 2026-09-16, mọi luật về bài học trong repo đều chỉ nói *ghi*. Hệ quả đo được: bài học 162 của `peekvn` phát minh lại y nguyên dấu hiệu nhận biết của bài học 2, sau 160 mục. Một file bài học chỉ-ghi là một file vô dụng — bài học là để rút kinh nghiệm cho lần sau, không phải học xong để đó.

#### Nửa 2 — GHI, khi kết quả khác kỳ vọng

**When a planned approach succeeds according to the plan, but the result doesn't match expectations, record the lesson in `docs/lessons-learned.md`.**

This documents not failures, but surprising outcomes where:
- The implementation was correct
- The plan was followed properly
- But the actual outcome differed from anticipated results

These are valuable insights for improvement. Example:
```
## [2026-09-13] Feature X completed but metrics showed Y

Plan: Implement feature X which should improve metric by 30%
Result: Feature works correctly, but metrics only improved by 5%
Lesson: Our assumption about user behavior was incomplete. Users don't use feature X because [reason]. 
Next: Need to research why users avoid this feature.
```

This is different from fixing bugs or course-correcting during development. It's about capturing surprises when everything worked as designed but wasn't as impactful as expected.

## Golden Rule: Keep Documentation in Sync

**After completing any feature or making architectural changes, update `docs/CONTEXT.md` and relevant ADRs immediately.**

The documentation must always reflect the true state of the application. If the docs and code diverge, the docs are out of date and must be fixed before the work is considered complete.

This follows Andrej Karpathy's **LLM Wiki** pattern: documentation is not a side effect of development; it is a core artifact that is maintained with the same rigor as code.

## Development Workflow

### Feature Implementation Flow

**Một lệnh `/implement [ticket]` phải chạy hết chuỗi này.** Người dùng không phải gõ lệnh cho từng bước — agent tự đi tiếp, và chỉ dừng lại ở đúng hai chỗ có rào vật lý (§The Discipline).

```
/implement #N
   │
   ├─ 0. Đọc bài học ────────── peekvn/docs/bai-hoc.md §Luật thường trực + grep tiêu đề
   │                            docs/lessons-learned.md
   ├─ 1. Đọc spec ───────────── gh issue view N  (đừng tin danh sách chép trong markdown)
   ├─ 2. Baseline xanh ──────── chạy test TRƯỚC khi sửa gì
   ├─ 3. Implement ─────────── /tdd ở các seam đã thống nhất
   ├─ 4. Nghiệm thu thật ───── chạy app/bộ cài thật, không chỉ test xanh (AGENTS.md §7.3)
   ├─ 5. /code-review ──────── sửa phát hiện, hoặc nói rõ vì sao không sửa
   ├─ 6. /update-doc ───────── CONTEXT.md + ADR + feature + bài học → push peek-window-doc
   ├─ 7. commit + push nhánh ─ push là điều kiện để CI chạy, KHÔNG phải mở PR
   ├─ 8. CI đạt ───────────── `apps\windows\ci\ci-run.ps1` — tự kích hoạt, tự
   │                            theo dõi, tự đối chiếu commit. Không bấm tay.
   └─ 9. Mở PR ────────────── chỉ khi 8 đạt
```

Issue chỉ được đóng khi **đủ cả năm**:

- Code đã implement ✓
- Đã nghiệm thu bằng cử chỉ người dùng thật ✓
- Code review đã qua ✓
- Tài liệu đã cập nhật và push ✓
- **CI đạt trên nhánh đó** ✓ — và PR đã merge

### The Discipline

**"Xong" là hết bước 9, không phải hết bước 3.** Dừng ở `git commit` rồi báo xong là báo sai.

**CI "đạt", không phải "xanh".** `ci-run.ps1` trả `0` SUCCESS, `2` UNSTABLE, `1` FAILURE. Mã 2 đi tiếp được **chỉ khi** xác minh được vàng là do bước treo đã biết (chưa ký số) — script tự đọc console để phân biệt với vàng-do-test-đỏ. Đòi xanh là chặn mọi ticket về sau vì một lý do không liên quan tới ticket nào: pipeline này không thể xanh cho tới khi có tài khoản Azure Trusted Signing.

Chỉ có **hai lý do** được phép dừng lại hỏi người dùng:

1. **Thiếu credential lần đầu** — `JENKINS_USER` / `JENKINS_TOKEN` chưa đặt trên máy. Một lần duy nhất, sau đó CI hoàn toàn tự động.
2. **Tiêu chí treo vì thiếu phần cứng/tài khoản** — nói rõ treo cái gì, thiếu cái gì, và vì sao nó không chặn đường các ticket sau.

**Không bao giờ bảo người dùng đi bấm một nút mà mình gọi được bằng API.** Gặp `401`/`403`, câu hỏi đúng là *"lấy khoá kiểu gì"*, không phải *"nhờ ai mở hộ"* — một mã lỗi xác thực là câu hỏi, không phải câu trả lời (bài học 165 của `peekvn`).

Mọi bước còn lại agent tự đi. Nếu CI đỏ thì sửa rồi push lại — không mở PR trên một nhánh đang đỏ.

Tài liệu phải luôn phản ánh đúng thực trạng mã. Docs lệch mã thì tính năng chưa xong.

## Project Status

✅ **Setup Complete:**
- **peek-window**: https://github.com/natuan1/peek-window — workspace repo
- **peek-window-doc**: https://github.com/natuan1/peek-window-doc — documentation & issues (`docs/` here is its local clone)
- `docs/CONTEXT.md` written — domain language + verified decisions (2026-09-13)
- Vietnamese triage labels created (2026-09-14), vocabulary aligned with the `peekvn` repo
- [ADR-0001](docs/adr/0001-brand-khac-service-mdns.md) — brand ≠ mDNS service name (2026-09-14)

### Next Steps (order matters)

1. ~~Spike Native AOT + Win32/Composition~~ ✅ **PASS 2026-09-14** — exe 3.05MB, working set 14.62MB, Composition OK under AOT ([ADR-0002](docs/adr/0002-ui-stack-aot-spike-pass.md); primary source: `peekvn` branch `prototype/aot-footprint`).
2. ~~Spike resumable upload (`104` over HTTP/1.1, real iPhone)~~ ✅ **PASS 2026-09-15** — CFNetwork xử lý nổi `104` giữa luồng h1 đến 50MB, 6/6 lượt `201`; advertise `resumableUpload: ["httpbis-interop-6"]`, server chỉ gửi `104` trên TLS ([ADR-0003](docs/adr/0003-resumable-upload-104-h1-pass.md); primary source: `peekvn` branch `prototype/104-over-h1`).
3. ~~`/to-spec` → `/to-tickets` từ bản kế hoạch chi tiết~~ ✅ **Xong 2026-09-15** — [Spec #1](https://github.com/natuan1/peek-window-doc/issues/1) (đã sửa 5 điểm kế hoạch lệch sự thật: Kestrel→h1-only, `_ultp._tcp`→`_peek._tcp`, draft-12→interop-6, 3 thiết bị→5 Slot, WPF/WinUI 3→Win32+Composition) + **18 ticket tracer-bullet** [#2–#19](https://github.com/natuan1/peek-window-doc/issues/2), label `sẵn-sàng-cho-agent`.
4. ~~Ticket 01 — skeleton Native AOT~~ ✅ **Xong và đã merge vào `main` của `peekvn` 2026-09-15** — 4 project, tray + bảng trạng thái + Per-Monitor V2, exe 1,69MB, RAM nền 12,51MB, 37 test xanh. Tài liệu: [ADR-0004](docs/adr/0004-cau-truc-app-windows-bon-project.md) + [feature](docs/features/nen-mong-app-windows/overview.md). **CI đã xanh** từ 2026-09-16 nhưng trên **Jenkins nội bộ**, không phải GitHub Actions ([ADR-0005](docs/adr/0005-ci-cd-desktop-qua-jenkins-noi-bo.md)) — runner đám mây không có phiên đồ hoạ để chạy thử app khay. **Một tiêu chí còn treo có chủ ý**: nghiệm thu Windows 10 1809 hoãn tới khi có máy, ưu tiên Windows 11 trước; floor sản phẩm và `SupportedOSPlatformVersion` **không đổi**.
5. ~~Ticket 02 — bộ cài Velopack + ký số + auto-update delta~~ ✅ **Xong và đã merge vào `main` của `peekvn` 2026-09-17** ([PR #130](https://github.com/natuan1/peekvn/pull/130), CI build #14 trên `2beb4ef`) — bộ cài **10,02MB** cài `PerUser` không UAC, vòng kiểm cập nhật 4 giờ, gói vá delta 15% gói đầy đủ, 71 test. Tài liệu: [ADR-0006](docs/adr/0006-dong-goi-velopack-cai-peruser.md) + [feature](docs/features/dong-goi-va-tu-cap-nhat/overview.md). **Ba tiêu chí treo có chủ ý**: ký số Azure Trusted Signing, `signtool verify /pa /v`, SmartScreen trên máy sạch — cùng một lý do là chưa có **tài khoản Azure Trusted Signing**; đường ống đã dựng xong và đã ép đỏ ở nhánh "chưa ký", nên khi có tài khoản chỉ cần cắm secret rồi `ci-run.ps1 -Release`. **KPI hẹp lại**: Velopack ăn ~6MB exe và ~2,5MB RAM nền (exe 1,69→7,71MB, RAM 12,51→15,04MB) — đọc bảng số đo trong `peekvn/apps/windows/README.md` trước khi thêm thư viện thứ hai.
6. Kế tiếp: **implement theo frontier** — Ticket 03 (#4, mở seam interop) và 04/05 (#5/#6, mDNS + server h1) chạy song song trên nền Ticket 01.
7. Nợ vận hành: GitHub Actions vẫn tắt, nên **Rust/Swift/protocol không có hàng rào tự động nào** — chỉ app Windows có. Khôi phục khi thanh toán thông: bỏ chú thích hai khối `push`/`pull_request` ở đầu `peekvn/.github/workflows/ci.yml`.

## Folder Structure

```
.
├── app/                          # Placeholder — NO source code here.
│                                 # Snappy Windows code lives in peekvn\apps\windows\
│
├── docs/                         # Documentation (source of truth)
│   ├── CONTEXT.md               # Main context document
│   ├── lessons-learned.md       # Captured lessons when results differ from plan
│   ├── agents/                  # Agent skill configuration
│   │   ├── issue-tracker.md    # Where issues are tracked
│   │   ├── triage-labels.md    # Vietnamese label definitions
│   │   └── domain.md           # Domain docs consumer rules
│   └── adr/                     # Architecture Decision Records
│       └── [decisions]
│
└── CLAUDE.md                    # This file
```

## Related Files

### Configuration
- `docs/agents/issue-tracker.md` — Issue tracking configuration
- `docs/agents/triage-labels.md` — Triage label vocabulary (Vietnamese)
- `docs/agents/domain.md` — Domain documentation layout and rules

### Documentation
- `docs/README.md` — Entry point
- **`docs/INDEX.md`** — Master index (IMPORTANT - navigate via this!)
- `docs/CONTEXT.md` — Main project context (single source of truth)
- `docs/DOC-STRUCTURE.md` — Documentation structure & organization
- `docs/WORKFLOW.md` — Development workflow

### Documentation Tree
- `docs/concepts/` — Domain concepts & terminology
- `docs/architecture/` — Technical architecture & design
- `docs/features/` — Feature documentation (one folder per feature)
- `docs/adr/` — Architecture Decision Records
- `docs/guides/` — How-to guides & practical documentation
- `docs/lessons-learned.md` — Captured lessons from surprises
