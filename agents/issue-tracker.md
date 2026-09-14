# Issue Tracker Configuration

## Summary

Issues are tracked in **GitHub Issues**:
- **`peek-window-doc` repository**: documentation, business requirements, and technical specifications (source of truth)
- **`peek-window` repository**: workspace-level planning and tracking issues
- **`peekvn` repository** (`natuan1/peekvn`): implementation source code (Snappy Windows lives in `apps/windows/`) and day-to-day implementation issues

Issues created by skills (like `to-tickets` and `triage`) target the appropriate repo based on issue type. The `peek-window-doc` repository serves as the **source of truth** for requirements, architecture decisions, and project state.

## Workflow Philosophy

This follows Andrej Karpathy's **LLM Wiki** pattern, adapted for a two-repo structure:

- **Documentation repo** (`docs`): the persistent, maintained wiki that reflects the true state of the application. All business logic, technical decisions, and requirements live here and are kept in sync with actual implementation.
- **Implementation code**: lives in the `peekvn` monorepo (`apps/windows/`), kept aligned with the documentation. The local `app/` folder is a placeholder only.
- **After completing a feature**, the documentation is always reviewed and updated to reflect the new reality.

This ensures that the documentation is never stale and always serves as a reliable reference for both humans and AI agents.

## Repository Names

- **`peek-window`** — Workspace-level planning and tracking
- **`peek-window-doc`** — Documentation, requirements, and issue tracking (source of truth)
- **`peekvn`** — Implementation source code (`apps/windows/` for Snappy Windows)

## Setup

To fully initialize this configuration:

1. Create two GitHub repositories:
   - `<username>/peek-window` — for source code
   - `<username>/peek-window-doc` — for documentation, requirements, and tracking

2. Implementation code lives in the separate `peekvn` monorepo (`github.com/natuan1/peekvn`), not in this workspace; the local `app/` folder stays a placeholder.

3. In the `docs/` folder:
   ```bash
   git remote add origin https://github.com/<username>/peek-window-doc.git
   git branch -M main
   git push -u origin main
   ```

4. Create the Vietnamese triage labels in both repos (defined in `docs/agents/triage-labels.md`)

## Issue Types by Repo

### `peek-window-doc` repo (source of truth)
- Feature specifications (PRD-style, requirements, user stories)
- Architecture decisions (ADRs)
- Technical design documents
- Documentation updates
- Integration issues or cross-repo concerns

### `peek-window` repo (workspace tracking)
- Planning-level issues spanning docs and implementation
- Spike/prototype tracking for decisions under investigation

### `peekvn` repo (implementation)
- Bug reports in application code
- Implementation tasks derived from specs in `peek-window-doc`
- Code refactoring tasks
- Performance improvements
- Build and test infrastructure issues

## CLI Integration

Skills that interact with this tracker use the `gh` CLI:

```bash
# Skills read issues from and write issues to:
gh issue list --repo <username>/peek-window
gh issue list --repo <username>/peek-window-doc
gh issue create --repo <username>/peek-window --title "..." --body "..."
gh issue create --repo <username>/peek-window-doc --title "..." --body "..."
```

## Note on PRs

Pull requests are **not** surfaced as a separate request surface to the triage queue. PRs are code review artifacts; issues are the source of work. If you want external PRs to be triaged, that can be configured separately.
