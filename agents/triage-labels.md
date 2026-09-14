# Triage Label Mapping

The `triage` skill uses five canonical role labels to organize work. This file maps those roles to Vietnamese label names used in GitHub Issues.

## Label Mapping

| Canonical Role | Vietnamese Label | Use Case |
|---|---|---|
| `needs-triage` | `cần-phân-loại` | Issue is new and needs initial review to understand scope and category |
| `needs-info` | `cần-thông-tin` | Issue needs more details from the reporter before it can be acted on |
| `ready-for-agent` | `sẵn-sàng-cho-agent` | Issue is well-defined and ready for an AI agent to work on implementation/analysis |
| `ready-for-human` | `cần-người-làm` | Issue needs human judgment, decision-making, or manual action |
| `wontfix` | `không-xử-lý` | Issue is acknowledged but will not be fixed (by-design, low priority, duplicate, etc.) |

## Triage Workflow

When an issue is created:

1. **Triage** (`cần-phân-loại`): Initial label. The triage agent reviews it and moves to one of:
   - `cần-thông-tin` if more details are needed
   - `sẵn-sàng-cho-agent` if it's ready for work
   - `cần-người-làm` if it requires human input
   - `không-xử-lý` if it should not be worked on

2. **Information gathering** (`cần-thông-tin`): Once the reporter provides details, re-triage to `cần-phân-loại` or move directly to action labels.

3. **Ready states**: Issues with `sẵn-sàng-cho-agent` or `cần-người-làm` are ready for work.

4. **Rejected** (`không-xử-lý`): No further action on this issue.

## Implementation

These labels exist across the repositories with identical names and descriptions:
- `peek-window` and `peek-window-doc`: workspace and documentation issues
- `peekvn` (implementation monorepo): the vocabulary here is **aligned with the labels already in use on `peekvn`** — do not introduce variant spellings

Use Vietnamese label names consistently across all repos to maintain a unified workflow.
