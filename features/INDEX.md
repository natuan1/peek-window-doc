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
| [Discovery LAN trên Windows](discovery-lan-windows/overview.md) | Quảng bá và duyệt `_peek._tcp` qua responder in-box của Windows; bảng thiết bị lân cận trên bảng trạng thái | ✅ Implement xong 2026-09-18, **chờ demo iPhone thật** ([#135](https://github.com/natuan1/peekvn/issues/135)) |
| [Server ULTP trên Windows](server-ultp-windows/overview.md) | Khung server h1-only: TLS 1.3 trên 8443 nghe dual-stack, router HTTP/1.1 tự viết, `GET /v1/info`, cột nền tảng của bảng thiết bị | ✅ Implement xong 2026-09-19, **chờ demo iPhone thật** + nghiệm thu Windows 10 ([#136](https://github.com/natuan1/peekvn/issues/136)) |
| [Ghép đôi SAS trên Windows](ghep-doi-sas-windows/overview.md) | Hai màn hình hiện cùng sáu chữ số, người thật so rồi bấm; Trust Store bền qua khởi động lại; mọi kết nối sau ghim SPKI và xác thực hai chiều | ✅ Implement xong 2026-09-19, **chờ demo iPhone thật** ([#137](https://github.com/natuan1/peekvn/issues/137)) |
| [Nhận push từ iPhone trên Windows](nhan-push-windows/overview.md) | iPhone đẩy tệp sang PC: offer/accept, token theo từng mục, PUT chảy thẳng xuống đĩa, chống path traversal của Windows, tự nhận theo từng thiết bị | ✅ Implement xong 2026-09-20, **chờ demo iPhone thật** ([#138](https://github.com/natuan1/peekvn/issues/138)) |
| [Mí magnet strip trên Windows](mi-magnet-strip-windows/overview.md) | Dải 30% bề rộng neo cạnh trên màn hình, năm trạng thái, phát hiện kéo qua `SysDragImage` + polling, nở 160 ms, vẽ bằng layered window (trắng đục 0,75, không viền), ẩn khi fullscreen; nhận cú thả tệp từ Ticket 11 | ✅ Implement xong 2026-09-30 ([#141](https://github.com/natuan1/peekvn/issues/141)) — **chưa nghiệm thu** đa màn hình/DPI; RAM sau lần kéo đầu 23,3 MB, dưới KPI, sau khi bỏ Composition ([#181](https://github.com/natuan1/peekvn/issues/181)) |
| [Trích xuất tệp thật/ảo → TempDrops](tempdrops-windows/overview.md) | Thả tệp lên Mí vào Shelf một Ngăn: tệp thật giữ đường dẫn gốc, tệp ảo (zip, Chrome, Edge, Outlook) ra `TempDrops\{Guid}\` qua hàng rào S11/S16; trần 2 GiB + LRU, xoá sạch qua menu khay, dọn lúc thoát và lúc khởi động | ✅ Implement xong 2026-10-01 ([#142](https://github.com/natuan1/peekvn/issues/142)) — **treo** demo Outlook (máy không cài); RAM sau bốn cú thả 24,4 MB |
| [Khay thẻ Shelf một Ngăn](shelf-tray-windows/overview.md) | Khay thẻ ngang neo trên Mí, mở từ menu khay; kéo một thẻ hay cả Ngăn ra Explorer/trình duyệt (`CF_HDROP`, Copy), Mục ở lại và được Touch; mở thư mục, ✕, báo "Không còn ở chỗ cũ"; khay sống trong tiến trình con | ✅ Implement xong 2026-10-02 ([#143](https://github.com/natuan1/peekvn/issues/143)) — RAM tổng 28,3 MB sau mọi thao tác (biên ~1,7 MB); chưa nghiệm thu đa màn hình/DPI |
| [Đích thả và gửi từ Mí](mi-dich-tha-windows/overview.md) | Khay Sẵn sàng nhận vẽ ô Shelf, avatar từng Thiết bị tin cậy, nút "Tất cả"; nhắm → Nhắm đích; thả avatar → Mục vào Ngăn rồi mời (pull), pill tiến trình thật; gửi thư mục; báo "Không thấy" / "Mất kết nối" | ✅ Implement xong 2026-10-03 ([#144](https://github.com/natuan1/peekvn/issues/144)) — nghiệm thu với iOS Simulator; ⚠️ RAM tổng vượt red line ở hai lượt (phần của ticket +0,9 MB); iPhone thật treo |
| [Capsule media trên Windows](capsule-media-windows/overview.md) | Nhạc đang phát (GSMTC) → capsule giữa dải Mí: bìa, "Bài — Nghệ sĩ", ⏸/▶/⏭ điều khiển app thật, % pin tai nghe GATT (`-`, không bao giờ `0%`); trượt sang bên khi Mí hiện; ẩn khi tạm dừng 30 s, khi fullscreen, khi không có phiên; tiến trình con thường trực, WinRT bằng vtable tay | ✅ Implement xong 2026-10-03 ([#146](https://github.com/natuan1/peekvn/issues/146)) — nghiệm thu với Chrome + Windows Media Player; **treo** pin tai nghe BLE thật (máy không có Bluetooth), Spotify |
| [Bản quyền Pro trên Windows](ban-quyen-windows/overview.md) | Menu khay "Kích hoạt Snappy Pro…" → hộp thoại (tiến trình con) dán Key `SNPY-`; câu rõ cho sai dạng / Key lạ / đủ 5 máy / mất mạng; token Ed25519 (verify tự viết) trong Credential Manager; Pro offline vĩnh viễn, re-validate 2 phút sau khởi động rồi mỗi 24 h; `Entitlements` gate Free/Pro (1 Ngăn, 5 mục clipboard); [hợp đồng API](ban-quyen-windows/api.md) | ✅ Implement xong 2026-10-04 ([#145](https://github.com/natuan1/peekvn/issues/145)) — nghiệm thu A→D với máy chủ giả; **treo** máy chủ thật + khoá ký thật (chặn phát hành ở `pack.ps1`) |

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
