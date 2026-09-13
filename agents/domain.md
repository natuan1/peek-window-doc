# Domain Docs Configuration

## Layout

This project uses a **single-context** domain documentation layout:

- **Location**: All domain documentation lives in the `docs/` repository
- **Root file**: `docs/CONTEXT.md` contains the central overview
- **ADR directory**: `docs/adr/` contains Architecture Decision Records
- **Source of truth**: The `docs` repository is the authoritative reference for all business logic, technical decisions, and application state

## Files

### `docs/CONTEXT.md`

The main context document for this project. Contains:

- **Project overview**: what the application does, its purpose
- **Core business logic**: key domain concepts, entities, workflows
- **Technical architecture**: high-level system design, technology choices
- **Critical decisions**: why certain approaches were chosen
- **Known constraints**: limitations, dependencies, assumptions

This file is read by agents (like `to-spec`, `implement`, `code-review`) to understand the domain before working on features or reviews.

**Responsibility**: Keep this file in sync with the actual application state. After completing any significant feature, review and update CONTEXT.md to reflect the new reality.

### `docs/adr/`

Architecture Decision Records — one file per significant technical decision.

Format (suggested):
```markdown
# ADR: [short title]

Date: YYYY-MM-DD
Status: Accepted | Proposed | Superseded

## Context
Why was this decision needed?

## Decision
What did we decide?

## Consequences
What are the trade-offs and implications?

## Related
Links to related ADRs or documentation.
```

Store ADR files as `docs/adr/NNNN-slug.md` (e.g., `0001-two-repo-structure.md`).

## Consumer Rules

### For skill `to-spec`
When writing feature specifications, read `CONTEXT.md` to understand existing domain language and architectural constraints.

### For skill `implement`
When implementing a feature, read `CONTEXT.md` to understand what the application does and how components interact. Refer to relevant ADRs for architectural decisions.

### For skill `code-review`
When reviewing code, compare changes against `CONTEXT.md` and ADRs to verify the code matches documented intentions.

### For skill `triage`
When triaging issues, read `CONTEXT.md` to understand scope and use proper terminology for the domain.

### For LLM agents generally
Always read `CONTEXT.md` first to ground any work in the correct domain model. Treat it as the single source of truth for what the application is and how it works.

## Workflow: Keeping Docs in Sync

**The Golden Rule**: After completing a feature or making architectural changes, **update CONTEXT.md and/or relevant ADRs immediately**.

This is not optional. It is as important as the feature itself. The documentation must reflect reality, or it becomes worse than useless.

1. **Feature complete**: code is implemented, tested, merged
2. **Update docs**: modify `CONTEXT.md` or create/update ADRs to reflect the new state
3. **Review**: verify that the docs accurately describe what was built
4. **Mark issue as done**: once docs are updated, the issue is truly complete

This cycle ensures the wiki stays current and remains a reliable source of truth.

## Monorepo Considerations

This project currently uses a two-repo structure (app + docs) rather than a monorepo. The documentation lives in the `docs` repo and is the single source of truth for both repos. If the project later expands to multiple app repos, all documentation continues to live in the `docs` repo, and each app repo references it as needed.
