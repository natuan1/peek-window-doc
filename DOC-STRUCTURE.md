# Cấu Trúc Tài Liệu (Documentation Structure)

Tài liệu của project được tổ chức theo **Hybrid Approach**: master-detail + organized by concepts.

## 📊 Sơ đồ Cấu Trúc

```
docs/
│
├─ README.md                   👈 Entry point cho người mới
├─ INDEX.md                    👈 Master index (QUAN TRỌNG!)
├─ CONTEXT.md                  👈 High-level project overview
│
├─ concepts/                   🎯 Domain concepts & terminology
│  ├─ INDEX.md                (index of concepts)
│  ├─ user-authentication.md  (authentication flow, user roles)
│  ├─ payment-flow.md         (payment processing concepts)
│  ├─ notification-system.md  (notification types, channels)
│  └─ ... (thêm concepts khác)
│
├─ architecture/              🏗️ Technical design & decisions
│  ├─ INDEX.md               (index of architecture docs)
│  ├─ system-overview.md     (system components, data flow)
│  ├─ database-schema.md     (tables, relationships, constraints)
│  ├─ api-design.md          (API endpoints, formats)
│  ├─ technology-stack.md    (frameworks, libraries, tools)
│  └─ security.md            (auth, encryption, vulnerabilities)
│
├─ features/                  🚀 Feature documentation
│  ├─ INDEX.md               (index of features)
│  ├─ authentication/        (feature folder)
│  │  ├─ overview.md         (feature description, scope)
│  │  ├─ user-stories.md     (user stories, workflows)
│  │  ├─ implementation.md   (how it's built, code pointers)
│  │  └─ testing.md          (test strategy, test cases)
│  ├─ payments/
│  │  ├─ overview.md
│  │  ├─ user-stories.md
│  │  ├─ implementation.md
│  │  └─ testing.md
│  └─ ... (thêm features khác)
│
├─ adr/                       ⚖️ Architecture Decision Records
│  ├─ 0001-authentication-strategy.md
│  ├─ 0002-database-choice.md
│  └─ ... (decisions)
│
├─ guides/                     📖 How-to guides
│  ├─ contributing.md         (how to contribute)
│  ├─ running-locally.md      (setup & run instructions)
│  ├─ testing.md              (how to test)
│  └─ debugging.md            (common issues & solutions)
│
├─ lessons-learned.md         📝 Bài học từ surprises
└─ WORKFLOW.md                🔄 Development workflow
```

---

## 🎯 Chi Tiết Từng Thư Mục

### 1. **ROOT: README.md**
**Mục đích**: Entry point cho người mới  
**Độ dài**: ~100 dòng  
**Nội dung:**
```markdown
# [Project Name]

Mô tả ngắn project làm gì.

## 🚀 Quick Links
- [CONTEXT.md](CONTEXT.md) - Project overview
- [INDEX.md](INDEX.md) - Master index of all docs
- [Features](features/) - Feature documentation

## 📖 Learn More
- [Architecture](architecture/) - System design
- [Concepts](concepts/) - Domain concepts
- [Contributing](guides/contributing.md)
```

### 2. **ROOT: INDEX.md** ⭐ (IMPORTANT)
**Mục đích**: Master index, central navigation  
**Độ dài**: ~200 dòng  
**Nội dung:**
```markdown
# Documentation Index

## 🎯 Getting Started
1. [CONTEXT.md](CONTEXT.md) - Start here for project overview
2. [README.md](README.md) - Project description

## 📚 By Topic

### 🎯 Domain Concepts
- [User Authentication](concepts/user-authentication.md)
- [Payment Flow](concepts/payment-flow.md)
- [Notification System](concepts/notification-system.md)

### 🏗️ Architecture
- [System Overview](architecture/system-overview.md)
- [Database Schema](architecture/database-schema.md)
- [API Design](architecture/api-design.md)
- [Technology Stack](architecture/technology-stack.md)

### 🚀 Features
- [Authentication](features/authentication/overview.md)
- [Payments](features/payments/overview.md)

### ⚖️ Decisions
- [ADR-001: Authentication Strategy](adr/0001-authentication-strategy.md)
- [ADR-002: Database Choice](adr/0002-database-choice.md)

### 📖 Guides
- [Contributing](guides/contributing.md)
- [Running Locally](guides/running-locally.md)
- [Testing](guides/testing.md)

### 📝 Reflections
- [Lessons Learned](lessons-learned.md)
```

### 3. **ROOT: CONTEXT.md**
**Mục đích**: High-level project overview (nó là "single source of truth")  
**Độ dài**: ~300-500 dòng  
**Nội dung:**
```markdown
# Project Context

## 📋 What is [Project]?
Brief description of what the project does.

## 🎯 Core Business Logic
- Key concepts & entities
- Main workflows
- User roles & permissions

## 🏗️ Technical Architecture
- Main components
- Data flow
- Technology choices (why?)

## 💡 Critical Decisions
- Why we chose [tech]
- Why we structure [feature] this way
- Known trade-offs

## 🚫 Known Constraints
- Limitations
- Assumptions
- Dependencies

## 📖 Quick Navigation
- [See Concepts →](concepts/)
- [See Architecture →](architecture/)
- [See Features →](features/)
- [See ADRs →](adr/)
```

### 4. **concepts/** - Domain Concepts
**Mục đích**: Terminology & domain concepts  
**Files:**
- `INDEX.md` - List of all concepts
- `user-authentication.md` - What is authentication in this project?
- `payment-flow.md` - How payments work conceptually
- `notification-system.md` - Notification types & channels

**Mỗi file:**
```markdown
# [Concept Name]

## Definition
What is this concept?

## Why It Matters
Why is this important for the project?

## Key Terms
- Term 1: Definition
- Term 2: Definition

## Related Concepts
- [Link to other concept]
- [Link to architecture doc]

## Example
Real-world example from project
```

### 5. **architecture/** - Technical Design
**Mục đích**: How the system is built  
**Files:**
- `INDEX.md` - Architecture overview
- `system-overview.md` - Components & data flow
- `database-schema.md` - Tables, relationships
- `api-design.md` - REST endpoints, formats
- `technology-stack.md` - Frameworks, libraries, tools
- `security.md` - Auth, encryption, security considerations

**Mỗi file:**
```markdown
# [Architecture Topic]

## Overview
High-level description

## Components
List of components involved

## Data Flow
How data flows through the system

## Implementation Details
Key implementation points

## Related Decisions
- [Link to ADR if relevant]
```

### 6. **features/** - Feature Documentation
**Mục đích**: Feature-specific documentation  
**Structure per feature:**
```
features/
├─ INDEX.md (list of all features)
├─ authentication/
│  ├─ overview.md (what, scope, user stories)
│  ├─ user-stories.md (detailed user stories)
│  ├─ workflow.md (step-by-step workflows)
│  ├─ implementation.md (how it's coded, components)
│  └─ testing.md (test strategy, test cases)
└─ payments/
   └─ (same structure as authentication)
```

**features/[feature]/overview.md:**
```markdown
# [Feature Name]

## 📋 Description
What does this feature do?

## 🎯 Scope
What's included and excluded?

## 👥 User Stories
1. As a [user], I want [action], so that [benefit]
2. ...

## 🔗 Related Concepts
- [Link to concept doc]
- [Link to architecture doc]

## 📍 Implementation Status
- Status: In development / Ready / Complete
- Last updated: YYYY-MM-DD
- Owner: [Who maintains this?]
```

**features/[feature]/workflow.md:**
```markdown
# [Feature] - Workflow

## User Journey
Step-by-step how user interacts with feature

## System Flow
Step-by-step technical flow

## Edge Cases
Special cases to handle
```

**features/[feature]/implementation.md:**
```markdown
# [Feature] - Implementation

## Architecture
Components involved, how they interact

## Key Files
- src/auth/login.ts (handles login logic)
- src/auth/session.ts (manages sessions)

## Database
Tables used, schema

## API
Endpoints exposed
```

### 7. **adr/** - Architecture Decision Records
**Mục đích**: Record important decisions & their rationale  
**File naming:** `NNNN-kebab-case-title.md`  
**Example:** `0001-authentication-strategy.md`

**Template:**
```markdown
# ADR-001: [Decision Title]

Date: 2026-09-13
Status: Accepted | Proposed | Superseded

## Context
Why did we need to make this decision?
What was the problem or question?

## Decision
What did we decide to do?

## Consequences
- Positive consequences
- Negative consequences (trade-offs)
- Long-term implications

## Alternatives Considered
- Why we didn't choose option A
- Why we didn't choose option B

## Related
- [Link to related ADR]
- [Link to relevant feature doc]
```

### 8. **guides/** - How-to Documentation
**Mục đích**: Practical guides for developers  
**Files:**
- `contributing.md` - How to contribute
- `running-locally.md` - Setup & run locally
- `testing.md` - How to run tests
- `debugging.md` - Common issues & solutions

---

## 🔗 Cross-Linking Strategy

Documents should link to each other:

```markdown
# features/authentication/overview.md

Related concepts:
- [User Authentication](../../concepts/user-authentication.md)
- [API Design](../../architecture/api-design.md)

Implementation:
- See [Implementation Guide](./implementation.md)

Decisions:
- [ADR-001: Authentication Strategy](../../adr/0001-authentication-strategy.md)
```

---

## 📊 Navigation Flowchart

```
New person arrives
    ↓
Read README.md
    ↓
Read CONTEXT.md (high-level)
    ↓
   ┌─────────────────────┬─────────────────┬──────────────┐
   ↓                     ↓                 ↓              ↓
Learn WHAT           Learn HOW          Learn SPECIFIC  Find
(Concepts)          (Architecture)      FEATURES        HOW-TOs
   ↓                     ↓                 ↓              ↓
concepts/          architecture/        features/     guides/
   ↓                     ↓                 ↓              ↓
[concept docs]     [architecture      [feature        [how-to
                     docs]             docs]            guides]
   ↓                     ↓                 ↓              ↓
    └─────────────────────┴─────────────────┴──────────────┘
                         ↓
                   (All link to)
                      INDEX.md
```

---

## ✅ Rules for Maintenance

### When Adding a New Document
1. ✅ Add entry to appropriate folder's `INDEX.md`
2. ✅ Add entry to root `INDEX.md`
3. ✅ Link to related documents
4. ✅ Write in Tiếng Việt (Vietnamese)

### When Updating CONTEXT.md
1. ✅ Update after each `/implement` + `/update-doc`
2. ✅ Keep it as "single source of truth"
3. ✅ Make sure architecture docs also update
4. ✅ If major change, consider creating ADR

### When Creating New Feature Folder
1. ✅ Follow the structure: overview → user-stories → workflow → implementation → testing
2. ✅ Add to `features/INDEX.md`
3. ✅ Add to root `INDEX.md`
4. ✅ Link to related concepts & architecture

---

## 🎯 Why This Structure Works for Agents

✅ **INDEX.md**: Agents can read it first to understand doc organization  
✅ **CONTEXT.md**: Agents can read it to understand project intent  
✅ **concepts/**: Agents can learn domain terminology  
✅ **architecture/**: Agents can understand how to build  
✅ **features/**: Agents can implement specific features  
✅ **adr/**: Agents can understand decision context  
✅ **Cross-links**: Agents can navigate between related docs  

---

## 📝 Example INDEX.md Entry

When you add a new document:

```markdown
## 📚 New Section

### Feature: User Profiles
- [Overview](features/user-profiles/overview.md)
- [Workflow](features/user-profiles/workflow.md)
- [Implementation](features/user-profiles/implementation.md)

### Concept: User Roles
- [User Roles Definition](concepts/user-roles.md)
```

---

**Next step**: Create this folder structure and start populating with CONTEXT.md!
