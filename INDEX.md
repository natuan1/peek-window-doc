# 📑 Documentation Index

Master index of all documentation. Start here to navigate!

## 🚀 Getting Started

**New to this project?** Follow this path:
1. [README.md](README.md) — What is this project?
2. [CONTEXT.md](CONTEXT.md) — Project overview & business logic
3. [DOC-STRUCTURE.md](DOC-STRUCTURE.md) — How docs are organized (this!)

---

## 📚 By Topic

### 🎯 **Concepts** (Domain Knowledge)
Understanding the domain — what, not how.

- [Concepts Index](concepts/INDEX.md)
  - *(Concepts will be added here as project grows)*
  - Example: User authentication flow, Payment processing, etc.

**💡 Use this when:**
- Learning domain terminology
- Understanding business logic
- Reading about workflows

---

### 🏗️ **Architecture** (Technical Design)
How the system is built — the technical decisions.

- [Architecture Index](architecture/INDEX.md)
  - *(Architecture docs will be added here)*
  - Examples: System overview, Database schema, API design, etc.

**💡 Use this when:**
- Understanding system design
- Learning about modules & components
- Implementing new features
- Reviewing code

---

### 🚀 **Features** (Feature Documentation)
Feature-by-feature breakdown.

- [Features Index](features/INDEX.md)
  - *(Features will be added as they're implemented)*
  - Example structure per feature:
    - Overview (scope, user stories)
    - Workflow (step-by-step)
    - Implementation (code, components)
    - Testing (test strategy)

**💡 Use this when:**
- Implementing a specific feature
- Understanding user stories
- Learning how a feature works

---

### ⚖️ **Architecture Decision Records** (Decisions)
Why we made important decisions.

- [0001-brand-khac-service-mdns.md](adr/0001-brand-khac-service-mdns.md) — Thương hiệu ≠ tên service mDNS (Snappy giữ `_peek._tcp`)
- [0002-ui-stack-aot-spike-pass.md](adr/0002-ui-stack-aot-spike-pass.md) — Xác nhận UI stack qua spike `aot-footprint` (3.05MB / 14.62MB)
- [0003-resumable-upload-104-h1-pass.md](adr/0003-resumable-upload-104-h1-pass.md) — Resumable upload trên h1 — spike `104` PASS, advertise `resumableUpload` (2026-09-15)

**Format:** `NNNN-kebab-case-title.md`  
**Content:** Context → Decision → Consequences

**💡 Use this when:**
- Understanding why we chose technology X
- Understanding trade-offs & constraints
- Making similar decisions

---

### 📖 **Guides** (How-To)
Practical guides for working with the project.

- [Contributing](guides/contributing.md) — How to contribute
- [Running Locally](guides/running-locally.md) — Setup & run
- [Testing](guides/testing.md) — How to test
- [Debugging](guides/debugging.md) — Common issues & solutions

**💡 Use this when:**
- Setting up local development
- Running tests
- Troubleshooting issues

---

### 📝 **Reflections**
Learning & improvements.

- [lessons-learned.md](lessons-learned.md) — Surprising outcomes & lessons
- [WORKFLOW.md](WORKFLOW.md) — Development workflow (implement → update-docs → review)

---

## 🔄 Workflow

```
/implement [spec]  →  /update-doc  →  /code-review  →  ✓ Done

After each step:
1. Code changes in peek-window repo
2. Docs update in peek-window-doc repo
3. Everything stays in sync
```

See [WORKFLOW.md](WORKFLOW.md) for details.

---

## 📊 Document Relationships

```
                        CONTEXT.md
                      (High-level overview)
                            ↓
        ┌───────────────────┼───────────────────┐
        ↓                   ↓                   ↓
    Concepts/          Architecture/        Features/
   (Why & What?)      (How & Design?)     (Feature-specific)
        ↓                   ↓                   ↓
   Domain terms        Components         User stories
   Business logic      Data flow          Workflows
   Workflows           Technology         Implementation
        ↓                   ↓                   ↓
        └───────────────────┼───────────────────┘
                      (All link to)
                       ADR/ (Why?)
                    Guides/ (How-to?)
```

---

## 🎯 Quick Navigation

### I want to...

- **Understand what this project does**
  → [CONTEXT.md](CONTEXT.md)

- **Learn domain concepts & terminology**
  → [Concepts/](concepts/)

- **Understand system architecture**
  → [Architecture/](architecture/)

- **Implement a feature**
  → [Features/](features/) + [Workflow](WORKFLOW.md)

- **Understand why a decision was made**
  → [ADR/](adr/)

- **Set up & run locally**
  → [Guides/running-locally.md](guides/running-locally.md)

- **Fix an issue**
  → [Guides/debugging.md](guides/debugging.md)

- **Contribute**
  → [Guides/contributing.md](guides/contributing.md)

---

## 📈 Documentation Status

| Category | Status | Last Updated |
|----------|--------|--------------|
| CONTEXT.md | ✅ Exists | See git log |
| Concepts | ⏳ To be created | - |
| Architecture | ⏳ To be created | - |
| Features | ⏳ To be created | - |
| ADRs | ⏳ To be created | - |
| Guides | ⏳ To be created | - |

**Note:** Most sections will be populated as features are implemented via `/implement` + `/update-doc` workflow.

---

## 🔗 Related Files

- [DOC-STRUCTURE.md](DOC-STRUCTURE.md) — Detailed explanation of doc structure
- [WORKFLOW.md](WORKFLOW.md) — How to implement & update docs
- [CLAUDE.md](CLAUDE.md) — Project configuration & rules
- [lessons-learned.md](lessons-learned.md) — Learning from surprises
- [SETUP-COMPLETE.md](SETUP-COMPLETE.md) — Initial setup checklist

---

## 💡 For AI Agents

If you're an AI agent reading this:

1. **First visit**: Read [CONTEXT.md](CONTEXT.md) to understand the project
2. **Before implementing**: Check [Features/](features/) for scope
3. **While implementing**: Reference [Architecture/](architecture/) & [Concepts/](concepts/)
4. **After implementing**: Update relevant docs (see [WORKFLOW.md](WORKFLOW.md))
5. **When making decisions**: Create or link to ADRs in [adr/](adr/)

All documentation is in **Tiếng Việt** (Vietnamese).

---

**Last updated**: See git history  
**Structure**: See [DOC-STRUCTURE.md](DOC-STRUCTURE.md)
