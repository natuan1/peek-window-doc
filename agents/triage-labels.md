# Triage Label Mapping

The `triage` skill uses five canonical role labels to organize work. This file maps those roles to Vietnamese label names used in GitHub Issues.

## Label Mapping

| Canonical Role | Vietnamese Label | Use Case |
|---|---|---|
| `needs-triage` | `cần-phân-loại` | Issue is new and needs initial review to understand scope and category |
| `needs-info` | `cần-thông-tin` | Issue needs more details from the reporter before it can be acted on |
| `ready-for-agent` | `sẵn-sàng-cho-agent` | Issue is well-defined and ready for an AI agent to work on implementation/analysis |
| `ready-for-human` | `sẵn-sàng-cho-người` | Issue needs human judgment, decision-making, or manual action |
| `wontfix` | `không-sửa` | Issue is acknowledged but will not be fixed (by-design, low priority, duplicate, etc.) |

## Triage Workflow

When an issue is created:

1. **Triage** (`cần-phân-loại`): Initial label. The triage agent reviews it and moves to one of:
   - `cần-thông-tin` if more details are needed
   - `sẵn-sàng-cho-agent` if it's ready for work
   - `sẵn-sàng-cho-người` if it requires human input
   - `không-sửa` if it should not be worked on

2. **Information gathering** (`cần-thông-tin`): Once the reporter provides details, re-triage to `cần-phân-loại` or move directly to action labels.

3. **Ready states**: Issues with `sẵn-sàng-cho-agent` or `sẵn-sàng-cho-người` are ready for work.

4. **Rejected** (`không-sửa`): No further action on this issue.

## Implementation

These labels should be created in both repositories:
- `app` repo: for implementation and code issues
- `docs` repo: for documentation and specification issues

Use Vietnamese label names consistently across both repos to maintain a unified workflow.
