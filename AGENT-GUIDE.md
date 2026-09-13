# 🤖 Guide for AI Agents

Instructions for AI agents on how to work with documentation in this project.

---

## 📖 Read This First

Before implementing any feature, you must read:
1. [CONTEXT.md](CONTEXT.md) — Project overview & business logic
2. [INDEX.md](INDEX.md) — Documentation navigation
3. Relevant feature/architecture docs

This gives you the context needed to make good decisions.

---

## 🔄 Documentation Maintenance Workflow

Your job is not just to write code. **You must keep documentation accurate and up-to-date.**

### The Golden Rule

> **Documentation is NOT optional.**  
> After implementing a feature, documentation MUST be updated.  
> If docs and code diverge, the feature is incomplete.

---

## 📋 Your Responsibilities

### 1. Before Implementing

- [ ] Read [CONTEXT.md](CONTEXT.md) to understand project
- [ ] Read relevant [INDEX.md](INDEX.md) sections
- [ ] Check if this introduces new concepts → Review [concepts/](concepts/)
- [ ] Check if this changes architecture → Review [architecture/](architecture/)

### 2. While Implementing

- [ ] Note what new concepts you're introducing
- [ ] Note what architectural decisions you're making
- [ ] Keep track of file changes (for documentation)

### 3. After Implementing (IMPORTANT!)

When code is done, run `/update-doc` to:

**Update CONTEXT.md:**
- Add any new domain concepts
- Update architecture description
- Document new workflows

**Create/Update Feature Documentation:**
- Create `docs/features/[feature-name]/` folder with:
  - `overview.md` — What is this feature?
  - `workflow.md` — How does it work?
  - `implementation.md` — How was it built?
  - `testing.md` — How to test it?

**Create ADRs (if applicable):**
- Important architectural decisions → Create `docs/adr/NNNN-*.md`
- Explain: Context → Decision → Consequences → Alternatives

**Update INDEX Files:**
- Folder INDEX.md (e.g., `concepts/INDEX.md`)
- Root [INDEX.md](INDEX.md)
- Make sure all links work!

**Record Lessons (if applicable):**
- If results were surprising → Update [lessons-learned.md](lessons-learned.md)

---

## 🎯 Documentation Quality Standards

When maintaining docs, ensure:

### ✅ Accuracy
- [ ] Docs reflect actual code
- [ ] Examples are from real codebase
- [ ] No stale/outdated claims

### ✅ Completeness
- [ ] Concepts are explained
- [ ] Workflows are step-by-step
- [ ] Implementation details are clear
- [ ] Tests are documented

### ✅ Navigability
- [ ] [INDEX.md](INDEX.md) is up-to-date
- [ ] Folder INDEX files are maintained
- [ ] Cross-links work (no broken links)
- [ ] Related docs are linked

### ✅ Language
- [ ] All Vietnamese (Tiếng Việt)
- [ ] Clear and concise
- [ ] Professional tone

---

## 📁 Folder Structure

```
docs/
├─ INDEX.md                 ← Master index (update this!)
├─ CONTEXT.md              ← Project overview (update this!)
│
├─ concepts/               ← Domain concepts
│  └─ INDEX.md            ← Update when adding concepts
│
├─ architecture/          ← Technical design
│  └─ INDEX.md           ← Update when adding architecture docs
│
├─ features/              ← Feature documentation
│  ├─ [feature-name]/
│  │  ├─ overview.md
│  │  ├─ workflow.md
│  │  ├─ implementation.md
│  │  └─ testing.md
│  └─ INDEX.md           ← Update when adding features
│
├─ adr/                   ← Architecture decisions
│  ├─ NNNN-*.md
│  └─ INDEX.md           ← Update when adding ADRs
│
└─ guides/               ← How-to guides
   └─ INDEX.md          ← Update when adding guides
```

---

## 🔗 Key Files to Update

When you create a new document:

### If adding a Concept
1. Create: `concepts/[concept-name].md`
2. Update: `concepts/INDEX.md` (add entry)
3. Update: [INDEX.md](INDEX.md) (add entry to Concepts section)

### If adding Architecture Doc
1. Create: `architecture/[topic].md`
2. Update: `architecture/INDEX.md`
3. Update: [INDEX.md](INDEX.md)

### If adding a Feature
1. Create: `features/[feature-name]/overview.md`
2. Create: `features/[feature-name]/workflow.md`
3. Create: `features/[feature-name]/implementation.md`
4. Create: `features/[feature-name]/testing.md`
5. Update: `features/INDEX.md`
6. Update: [INDEX.md](INDEX.md)

### If creating an ADR
1. Create: `adr/NNNN-kebab-case.md` (increment NNNN)
2. Update: `adr/INDEX.md`
3. Update: [INDEX.md](INDEX.md)

---

## ✅ Checklist for Every Feature

After implementing a feature, verify:

- [ ] Feature folder created with 4 files
- [ ] CONTEXT.md updated with new concepts
- [ ] Architecture docs updated if needed
- [ ] ADR created if important decision made
- [ ] All INDEX.md files updated
- [ ] No broken links in INDEX.md
- [ ] All documentation in Vietnamese
- [ ] Docs accurately reflect code
- [ ] Commit message: "Docs: Update for [feature name]"
- [ ] Changes pushed to GitHub

---

## 🔍 How to Check Documentation Quality

Before committing, verify:

```bash
# 1. Check all INDEX.md files exist
ls docs/*/INDEX.md

# 2. Verify no broken links (manual check)
# Read through INDEX.md and click/check all links

# 3. Confirm docs match code
# Compare code changes with what was documented

# 4. Check language
# All text should be Vietnamese (Tiếng Việt)
```

---

## 💡 Common Documentation Scenarios

### Scenario 1: Simple Feature
1. Create feature folder: `features/[name]/`
2. Write 4 files (overview, workflow, implementation, testing)
3. Update: `features/INDEX.md` + root `INDEX.md`
4. Update: `CONTEXT.md` if introduces new concepts

### Scenario 2: Complex Feature (Multi-component)
1. Create feature folder: `features/[name]/`
2. Write 4 files + additional architecture notes
3. Create architecture doc: `architecture/[component].md`
4. Create ADR: `adr/NNNN-[decision].md`
5. Update: Multiple INDEX.md files

### Scenario 3: Bug Fix
1. Update relevant feature doc: `features/[name]/testing.md`
2. Update: `lessons-learned.md` if interesting learning
3. No new feature folder needed
4. Update: root `INDEX.md` if major insights

### Scenario 4: Refactoring
1. Update: `architecture/` docs if structure changes
2. Update: `CONTEXT.md` if high-level changes
3. Update: Feature docs if workflows affected
4. Create ADR if major architectural change

---

## 🚨 Red Flags

Watch for these and fix immediately:

🔴 Broken links in INDEX.md  
🔴 Outdated information in CONTEXT.md  
🔴 Feature implemented but no docs created  
🔴 Architectural change but no ADR  
🔴 Documentation in English (should be Vietnamese)  
🔴 Docs don't match actual implementation  

---

## 📚 Documentation Examples

### Good CONTEXT.md
- Clear project overview
- Business logic explained
- Tech stack & why
- Critical decisions noted
- Links to architecture & features

### Good Feature Documentation
- Clear problem statement
- Step-by-step workflows
- Code component mapping
- Test strategy explained
- Links to related concepts & decisions

### Good ADR
- Clear context & problem
- Specific decision made
- Trade-offs listed
- Alternatives considered
- Related decisions linked

---

## 🤝 Team Collaboration

Your documentation helps:
- ✅ Other agents understand project
- ✅ Future maintainers learn codebase
- ✅ New team members onboard faster
- ✅ Track project history & decisions

**Don't treat docs as an afterthought — they're as important as the code.**

---

## 🆘 When in Doubt

If unsure about documentation:
1. Check [DOC-STRUCTURE.md](DOC-STRUCTURE.md) for structure details
2. Look at existing docs for examples
3. Err on the side of more documentation (not less)
4. Ask human: "Is this documentation sufficient?"

---

## Summary

**Before & During**: Read existing docs to understand project  
**After Implementation**: Update docs via `/update-doc`  
**Always**: Keep docs accurate, complete, navigable, in Vietnamese  
**Remember**: Docs are NOT optional — they're part of the feature

You're not just building code. You're building and maintaining knowledge about the project.

Make it count! 🚀
