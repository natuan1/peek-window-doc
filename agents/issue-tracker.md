# Issue Tracker Configuration

## Summary

Issues are tracked in **GitHub Issues** across two separate GitHub repositories:
- **`app` repository**: source code and implementation issues
- **`docs` repository**: documentation, business requirements, and technical specifications

Issues created by skills (like `to-tickets` and `triage`) target the appropriate repo based on issue type. The `docs` repository serves as the **source of truth** for requirements, architecture decisions, and project state.

## Workflow Philosophy

This follows Andrej Karpathy's **LLM Wiki** pattern, adapted for a two-repo structure:

- **Documentation repo** (`docs`): the persistent, maintained wiki that reflects the true state of the application. All business logic, technical decisions, and requirements live here and are kept in sync with actual implementation.
- **Application repo** (`app`): implementation code that is kept aligned with the documentation.
- **After completing a feature**, the documentation is always reviewed and updated to reflect the new reality.

This ensures that the documentation is never stale and always serves as a reliable reference for both humans and AI agents.

## Setup

To fully initialize this configuration:

1. Create two GitHub repositories:
   - `<username>/app` — for source code
   - `<username>/docs` — for documentation, requirements, and tracking

2. Push the local `app/` folder to the `<username>/app` repo
3. Push the local `docs/` folder to the `<username>/docs` repo

4. Update this file with the actual GitHub URLs once repos are created.

## Issue Types by Repo

### `docs` repo (source of truth)
- Feature specifications (PRD-style, requirements, user stories)
- Architecture decisions (ADRs)
- Technical design documents
- Documentation updates
- Integration issues or cross-repo concerns

### `app` repo (implementation)
- Bug reports in application code
- Implementation tasks derived from specs in `docs`
- Code refactoring tasks
- Performance improvements
- Build and test infrastructure issues

## CLI Integration

Skills that interact with this tracker use the `gh` CLI:

```bash
# Skills read issues from and write issues to:
gh issue list --repo <username>/app
gh issue list --repo <username>/docs
gh issue create --repo <username>/app --title "..." --body "..."
gh issue create --repo <username>/docs --title "..." --body "..."
```

## Note on PRs

Pull requests are **not** surfaced as a separate request surface to the triage queue. PRs are code review artifacts; issues are the source of work. If you want external PRs to be triaged, that can be configured separately.
