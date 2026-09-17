# 🚀 Features Index

Feature documentation — what the application does.

## Overview
This folder contains documentation for each feature — user stories, workflows, implementation details, and testing strategies.

## Features List

| Feature | Description | Status |
|---------|-------------|--------|
| [Nền móng app Windows](nen-mong-app-windows/overview.md) | App chạy nền, icon khay, bảng trạng thái, thoát sạch; publish Native AOT ra một exe | ✅ Implement xong 2026-09-15 ([#132](https://github.com/natuan1/peekvn/issues/132)) |
| [Đóng gói & tự cập nhật](dong-goi-va-tu-cap-nhat/overview.md) | Bộ cài Velopack user-space không UAC; tự tìm bản mới mỗi 4 giờ, tải gói vá delta, áp lúc mở lại app | ✅ Merge vào `main` 2026-09-17 ([#133](https://github.com/natuan1/peekvn/issues/133)) — ký số còn treo vì chưa có tài khoản Azure |
| [Seam interop cho Windows](seam-interop-windows/overview.md) | Windows thành implementation ULTP thứ ba trong `interoperability/run.sh`: CLI fixture + cặp `rust-host ↔ windows` | ✅ Implement xong 2026-09-17 ([#134](https://github.com/natuan1/peekvn/issues/134)) |

---

## Feature Folder Structure

Each feature gets its own folder with consistent structure:

```
features/
├─ feature-name/
│  ├─ overview.md       (scope, description, user stories)
│  ├─ workflow.md       (step-by-step user & system flows)
│  ├─ implementation.md (architecture, key files, database)
│  └─ testing.md        (test strategy, test cases)
```

---

## How to Add a Feature

When documenting a new feature:

1. Create folder: `feature-name/`
2. Create files in order:
   - `overview.md` — What is this feature?
   - `workflow.md` — How does it work?
   - `implementation.md` — How is it built?
   - `testing.md` — How to test?
3. Add entry to this INDEX.md
4. Add entries to root [INDEX.md](../INDEX.md)
5. Update [CONTEXT.md](../CONTEXT.md) if it affects overall architecture

---

## Feature Document Templates

### overview.md Template
```markdown
# [Feature Name]

## 📋 Description
What does this feature do?

## 🎯 Scope
- What's included
- What's NOT included (out of scope)

## 👥 User Stories
1. As a [user], I want [action], so that [benefit]
2. ...

## 🔗 Related Concepts
- [Concept name](../concepts/...)
- [Architecture component](../architecture/...)

## Status
- Implementation: In progress / Complete
- Documentation: In progress / Complete
- Last updated: YYYY-MM-DD
```

### workflow.md Template
```markdown
# [Feature Name] - Workflow

## 👤 User Journey
Step-by-step what user does...

## 🖥️ System Flow
Step-by-step what system does...

## 📊 Diagram
(If helpful)

## 🔄 Edge Cases
- Case 1: What happens when...
- Case 2: What happens when...
```

### implementation.md Template
```markdown
# [Feature Name] - Implementation

## 🏗️ Architecture
Components involved:
- Component A: Does X
- Component B: Does Y

## 📁 Key Files
- `src/auth/login.ts` — Login logic
- `src/auth/session.ts` — Session management

## 💾 Database
Tables used:
- `users` — User data
- `sessions` — Session tokens

## 🔌 API
Endpoints exposed:
- `POST /auth/login`
- `GET /auth/profile`
```

### testing.md Template
```markdown
# [Feature Name] - Testing

## 📊 Test Strategy
What makes a good test?
- Only test external behavior
- Test happy path + edge cases
- Use existing test patterns

## 🧪 Unit Tests
- File: `src/auth/login.test.ts`
- Cases: Login success, invalid password, user not found

## 🔗 Integration Tests
- What flows do we test end-to-end?

## ✅ Test Checklist
- [ ] Happy path
- [ ] Error cases
- [ ] Edge cases
```

---

## Navigation

- Back to [Main INDEX](../INDEX.md)
- See [Concepts](../concepts/)
- See [Architecture](../architecture/)
- See [Guides](../guides/)
