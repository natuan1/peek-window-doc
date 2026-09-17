# ✅ Setup Complete!

> ⚠️ **Ảnh chụp lịch sử (2026-09-13), không phải hiện trạng.** Giữ lại để biết
> dự án đã dựng ra sao. Hai điểm trong đây **nay đã sai**:
>
> - Issue **không** còn ở `peek-window-doc`; mọi issue đã chuyển sang
>   `natuan1/peekvn` (2026-09-17) — xem `agents/issue-tracker.md`.
> - Thư mục workspace `peek-window` đã bị bỏ; giờ chỉ còn hai repo
>   `D:\code\peekvn` và `D:\code\peek-docs` — xem `CLAUDE.md`.


Tất cả đã được cấu hình. Dưới đây là tóm tắt.

## 🚀 Hai GitHub Repos Của Bạn

1. **peek-window** (Source Code)
   - URL: https://github.com/natuan1/peek-window
   - Chứa: Mã ứng dụng

2. **peek-window-doc** (Documentation + Issues)
   - URL: https://github.com/natuan1/peek-window-doc
   - Chứa: CONTEXT.md, ADRs, lessons-learned.md, issues tracking
   - **Đây là "Source of Truth"**

---

## 🎯 Workflow Mới: `/implement` → `/update-doc` → Done

### Trước đây
```
/implement [spec] → Done
```
❌ Docs không được cập nhật

### Bây giờ
```
/implement [spec] → /update-doc → /code-review → ✓ Done
```
✅ Code + Docs luôn đồng bộ

---

## 📋 Skill Mới: `/update-doc`

Sau khi `/implement` hoàn thành code, chạy:

```bash
/update-doc
```

Skill này sẽ tự động:
1. Đọc code changes (git diff)
2. Cập nhật `CONTEXT.md` (phản ánh thực tế project)
3. Tạo/cập nhật ADRs (nếu có architectural decisions)
4. Ghi bài học (nếu kết quả khác kỳ vọng)
5. Commit & push lên `peek-window-doc` GitHub

**Xem chi tiết**: `docs/WORKFLOW.md`

---

## 🌍 Vietnamese Language Rules

Tất cả agents phải nói **Tiếng Việt**:
- Responses từ agents ← Tiếng Việt
- Code comments ← Tiếng Việt
- Issues/PRs ← Tiếng Việt
- CONTEXT.md ← Tiếng Việt
- ADRs ← Tiếng Việt
- lessons-learned.md ← Tiếng Việt

---

## 📚 Lessons Learned Protocol

Khi code được implement **đúng theo kế hoạch** nhưng **kết quả khác kỳ vọng**:

```markdown
## [2026-09-13] Feature X - Unexpected Result

**Kế hoạch**: Improve metric by 30%
**Kết quả**: Only 5% improvement
**Bài học**: Users don't use feature because [reason]
**Hành động tiếp theo**: Research user behavior
```

Ghi vào: `docs/lessons-learned.md`

---

## 📋 Danh Sách Việc Cần Làm

### [ ] Bước 1: Tạo Triage Labels (Vietnamese)

Trong cả 2 GitHub repos:
1. Vào Settings → Labels
2. Tạo 5 labels này:
   - `cần-phân-loại` (Color: #0075ca)
   - `cần-thông-tin` (Color: #d876e3)
   - `sẵn-sàng-cho-agent` (Color: #0e8a16)
   - `sẵn-sàng-cho-người` (Color: #bfd4f2)
   - `không-sửa` (Color: #eaeaea)

### [ ] Bước 2: Viết CONTEXT.md

Trong `peek-window-doc` repo, tạo `CONTEXT.md` với:
- **Project Overview**: Ứng dụng này làm gì?
- **Core Business Logic**: Khái niệm miền chính?
- **Technical Architecture**: Tech stack, modules?
- **Critical Decisions**: Tại sao chọn công nghệ này?
- **Known Constraints**: Ràng buộc nào?

### [ ] Bước 3: Tạo Issue Đầu Tiên

Trong `peek-window-doc` repo:
1. Tạo issue (GitHub Issues)
2. Mô tả tính năng đầu tiên
3. Gắn label `sẵn-sàng-cho-agent`

### [ ] Bước 4: Implement & Update Docs

```bash
/implement [issue-number]
# ... code ...
/update-doc
/code-review
# ✓ Done
```

---

## 🔗 Key Files

- **CLAUDE.md** ← Cấu hình chính (rules, workflow)
- **WORKFLOW.md** ← Hướng dẫn chi tiết workflow
- **CONTEXT.md** ← Project context (TO BE WRITTEN)
- **adr/** ← Architecture decisions
- **lessons-learned.md** ← Bài học từ surprises
- **.claude/skills/update-doc/** ← Skill để update docs

---

## 🎓 The Pattern: Andrej Karpathy's LLM Wiki

Bạn đang implement mô hình:
- **Raw sources** = code trong `peek-window` app
- **The wiki** = documentation trong `peek-window-doc` repo
- **The schema** = CLAUDE.md + agents/ configs

**Lợi ích:**
- Docs luôn up-to-date (không bao giờ stale)
- Agents có "source of truth" để tham khảo
- Học được từ những surprise outcomes
- Project history được ghi lại (via ADRs)

---

## 🚀 Ready to Start!

Bây giờ bạn có:

✅ 2 GitHub repos (peek-window + peek-window-doc)
✅ Workflow mới (implement → update-doc → code-review)
✅ `/update-doc` skill (tự động cập nhật tài liệu)
✅ Vietnamese language enforcement
✅ Lessons learned tracking

**Bước tiếp theo:**
1. Tạo triage labels
2. Viết CONTEXT.md
3. Tạo issue đầu tiên
4. Chạy: `/implement` → `/update-doc` → `/code-review`

Let's build! 🚀
