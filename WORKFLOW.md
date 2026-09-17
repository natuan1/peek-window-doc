# Workflow: Implement + Update Docs

Quy trình để implement một tính năng và tự động cập nhật tài liệu.

## Workflow Chính

```
Issue/Spec → /implement → /update-doc → /code-review → ✓ Done
```

## Chi Tiết Từng Bước

### 1. Issue/Spec trong `peekvn` repo

> ⚠️ **Đổi từ 2026-09-17:** issue **không** còn nằm ở repo này. Toàn bộ đã
> chuyển sang `peekvn` (commit `7972e7a`), nên repo tài liệu hiện có 0 issue mở.
> Tracker là một chỗ duy nhất: **`peekvn`**. Repo này giữ *vì sao*
> (`CONTEXT.md`, `adr/`, `lessons-learned.md`), không giữ *việc cần làm*.

Tạo issue hoặc spec trong repo https://github.com/natuan1/peekvn với:
- Mô tả tính năng
- User stories
- Implementation decisions
- Testing decisions

Gắn label `sẵn-sàng-cho-agent` (ready-for-agent)

### 2. Implement Code

```bash
/implement [issue-number hoặc spec]
```

Skill này sẽ:
- Tạo branch mới
- Implement code theo spec
- Chạy tests
- Commit work vào repo mã nguồn (`peekvn`, nhánh `main`)

**Output**: Code hoàn thành trong `peekvn` repo

### 3. Update Documentation (NEW!)

```bash
/update-doc
```

🎯 **Đây là bước mới và RẤT QUAN TRỌNG**

Skill này sẽ tự động:

1. **Đọc code changes**
   - Xem `git diff` để hiểu gì được thay đổi
   - Xem issue/spec để hiểu intent

2. **Cập nhật `CONTEXT.md`**
   - Thêm/cập nhật domain concepts
   - Cập nhật technical architecture
   - Ghi lại các quyết định mới
   - Thêm ví dụ/use cases mới

3. **Tạo/Cập nhật ADRs (Architecture Decision Records)**
   - Nếu có architectural decision mới, tạo file `adr/NNNN-slug.md`
   - Format: Context → Decision → Consequences

4. **Ghi Bài Học (Lessons Learned)**
   - Nếu kết quả thực tế khác kỳ vọng, ghi vào `lessons-learned.md`

5. **Commit & Push**
   ```bash
   git commit -m "Docs: Update for [feature name]"
   git push origin master
   ```

**Output**: Documentation được cập nhật trong `peek-window-doc` repo

### 4. Code Review

```bash
/code-review
```

Verify implementation matches spec và code quality

### 5. Mark Complete

✓ Issue được close khi:
- Code implemented ✓
- **Documentation updated** ✓ (Critical!)
- Code reviewed ✓

---

## Key Rule: Documentation is Not Optional

> **Golden Rule**: After completing ANY feature, you MUST run `/update-doc` before marking it done.

Documentation phải luôn phản ánh thực trạng hiện tại của code.

Nếu:
- Code: ✓ Working
- Docs: ✗ Out of date

→ Feature chưa xong. Phải cập nhật docs trước.

---

## Ví Dụ Workflow Hoàn Chỉnh

### Issue được tạo
```
Issue #5: Thêm authentication flow
Repo: peekvn
```

### Implement
```bash
/implement #5
```
- Branch: `feature/auth`
- Code: Implement login, logout, session management
- Tests: ✓ Pass
- Commit: `app: Add authentication flow`

### Update Docs
```bash
/update-doc
```
- Reads: git diff, issue #5, current CONTEXT.md
- Updates: CONTEXT.md
  - Add "Authentication" section
  - Describe login flow, session management
  - List new modules/components
- Creates: ADR `0001-authentication-strategy.md`
  - Decision: JWT tokens in HTTP-only cookies
  - Why: Secure against XSS
  - Trade-offs: Can't use CSRF tokens
- Commits: "Docs: Add authentication flow documentation"
- Pushes: To `peek-window-doc` master

### Code Review
```bash
/code-review
```
- Verifies code matches spec
- Checks implementation follows ADRs
- Reviews against CONTEXT.md

### Mark Complete
✓ Issue #5 closed

---

## Files Updated by `/update-doc`

Skill này có thể update:
- `CONTEXT.md` — Main project context (luôn update)
- `adr/*.md` — Architecture decisions (khi cần)
- `lessons-learned.md` — Unexpected outcomes (khi có)

Tất cả các file này nằm trong `peek-window-doc` repo.

---

## Commit Message Convention

```
Docs: Update for [feature name]
Docs: Add ADR for [decision name]
Docs: Record lesson learned - [what was surprising]
```

---

## Checking the Results

Sau khi `/update-doc` chạy xong:

1. Check `CONTEXT.md` trong `peek-window-doc`:
   ```bash
   git log --oneline -5  # Thấy "Docs: Update for..."
   cat CONTEXT.md        # Xem cập nhật
   ```

2. Check ADRs:
   ```bash
   ls -la adr/           # Xem ADRs mới
   ```

3. Check GitHub:
   ```
   https://github.com/natuan1/peek-window-doc
   → Commits tab
   → Should see new commits from /update-doc
   ```

---

## Troubleshooting

### `/update-doc` doesn't find CONTEXT.md
- Make sure `docs/CONTEXT.md` exists
- If not, create it first:
  ```bash
  echo "# Project Context" > CONTEXT.md
  git add CONTEXT.md && git commit -m "Initial CONTEXT.md"
  ```

### Can't push to GitHub
- Check git remote:
  ```bash
  git remote -v
  # Should show https://github.com/natuan1/peek-window-doc.git
  ```
- Check GitHub credentials:
  ```bash
  gh auth status
  ```

### What if I forgot to run `/update-doc`?
- No problem, just run it now:
  ```bash
  /update-doc
  ```
- It will read recent commits and update docs

---

## That's It! 🎉

Workflow:
```
/implement → /update-doc → /code-review → ✓
```

Tài liệu luôn đồng bộ, luôn phản ánh thực tế. Project luôn có "single source of truth".
