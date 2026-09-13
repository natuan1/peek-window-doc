# CLAUDE.md

Configuration and conventions for AI agent work on this project.

## Overview

This is a two-repository project:
- **`app/`**: Application source code
- **`docs/`**: Documentation, requirements, and issue tracking

The `docs` repository serves as the **single source of truth** for business logic, technical decisions, and project state. Implementation in `app/` must always align with `docs/`.

## Agent Skills

### Issue Tracker

Issues are tracked in **GitHub Issues** across two repositories: `app` (implementation) and `docs` (specifications and decisions). See `docs/agents/issue-tracker.md`.

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

1. **Feature specification**: Issues in the `docs` repo describe what needs to be built
2. **Implementation**: Code in the `app` repo implements the feature
3. **Documentation sync**: After the feature is done, `docs/CONTEXT.md` and/or ADRs are updated
4. **Review**: Verify that documentation accurately reflects the implementation
5. **Mark complete**: Issue is closed only after docs are updated

## Project Status

This is the initial setup. The two repositories (`app/` and `docs/`) exist locally. To fully initialize:

1. Create two GitHub repositories: `<username>/app` and `<username>/docs`
2. Push `app/` to `<username>/app`
3. Push `docs/` to `<username>/docs`
4. Create the Vietnamese triage labels in both repos (defined in `docs/agents/triage-labels.md`)
5. Update `docs/agents/issue-tracker.md` with the actual GitHub repository URLs

## Folder Structure

```
.
├── app/                          # Application source code
│   ├── src/
│   ├── tests/
│   └── [implementation files]
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

- `docs/agents/issue-tracker.md` — Issue tracking configuration
- `docs/agents/triage-labels.md` — Triage label vocabulary (Vietnamese)
- `docs/agents/domain.md` — Domain documentation layout and rules
- `docs/CONTEXT.md` — Main context (to be written)
- `docs/lessons-learned.md` — Captured lessons when results surprise expectations
- `docs/adr/` — Architecture decisions (to be written)
