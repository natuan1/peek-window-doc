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

1. **Feature specification**: Issues in the `docs` repo describe what needs to be built
2. **Implement**: Use `/implement` skill to code the feature in `peekvn\apps\windows\` (monorepo `peekvn`)
   ```
   /implement [spec/ticket]
   ```
3. **Update Documentation**: After code is done, automatically sync docs using `/update-doc` skill
   ```
   /update-doc
   ```
   This skill will:
   - Read code changes from git diff
   - Update `docs/CONTEXT.md` to reflect new state
   - Create/update ADRs (Architecture Decision Records) if needed
   - Record lessons learned if results surprised expectations
   - Commit and push to `peek-window-doc` repo on GitHub

4. **Code Review**: Use `/code-review` to verify implementation matches spec
   ```
   /code-review
   ```
5. **Mark complete**: Issue is closed only after:
   - Code is implemented ✓
   - Documentation is updated ✓
   - Code review passes ✓

### The Discipline

**After completing any feature, you MUST run `/update-doc` before considering it done.**

The documentation must always reflect the true state of the application. If docs and code diverge, the feature is incomplete.

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
3. **`/to-spec` → `/to-tickets`** from the detailed plan. ⚠️ Plan §2.3 still says "Kestrel server" — superseded by the 2026-09-13 decision (self-written HTTP/1.1-only server, long-poll `?wait=`, no WebSocket); fix when spec-ing.

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
