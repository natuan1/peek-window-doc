# CLAUDE.md — repo tài liệu (`peek-docs`)

Quy tắc cho agent khi làm việc **trong repo này**. Mã nguồn có `CLAUDE.md` riêng
(`D:\code\peekvn\CLAUDE.md`) — đừng trộn hai bộ luật.

## Repo này là gì

`peek-docs` = clone của `github.com/natuan1/peek-window-doc`, nhánh **`master`**
(không phải `main` — `git push origin main` sẽ đỏ).

Đây là **nguồn sự thật** cho nghiệp vụ, quyết định kiến trúc và trạng thái dự án
của **Snappy**. Mã nguồn không nằm ở đây và sẽ không bao giờ nằm ở đây.

## Hai repo, hai vai — hết

```
D:\code\
├── peekvn\      mã nguồn  → github.com/natuan1/peekvn        (nhánh main)
│                app Windows ở apps\windows\, dùng chung schemas +
│                interop suite + Rust oracle với iOS/Android
└── peek-docs\   tài liệu  → github.com/natuan1/peek-window-doc (nhánh master)
```

Cơ cấu lại 2026-09-17. Trước đó có một thư mục thứ ba (`peek-window`) vừa là
workspace vừa chứa clone tài liệu bên trong, cộng một repo `peek-window` gần như
rỗng — và hệ quả đo được là **hai file `CLAUDE.md` lệch nhau 89 dòng**, trong đó
bản được git theo dõi lại là bản mang quy trình đã bị thay. Một bản sao cũ của
chính tài liệu quy trình nguy hiểm hơn là không có tài liệu.

**Skill dùng chung** giờ nằm ở mức người dùng (`C:\Users\tuana\.claude\skills\`),
một bản duy nhất, dùng được từ cả hai repo. Đừng chép chúng vào repo nào — đó
đúng là cái bẫy vừa gỡ.

## Ngôn ngữ

**Mọi thứ bằng tiếng Việt**: trả lời trong chat, tài liệu, tiêu đề ADR, mô tả
issue và PR, comment trong mã.

## Bài học có hai nửa — nửa ĐỌC đi trước nửa GHI

### Nửa 1 — ĐỌC, trước khi bắt tay (bắt buộc)

- `peekvn/docs/bai-hoc.md` — đọc §**Luật thường trực** (5 họ lỗi lặp lại), rồi
  `git grep -n "^## " docs/bai-hoc.md` để quét tiêu đề tìm vùng sắp chạm.
- `lessons-learned.md` của repo này — ngắn, đọc hết.

Một bài học khớp mà vẫn cố tình làm khác thì **nói ra lý do trong báo cáo**.

**Vì sao luật này tồn tại:** trước 2026-09-16, mọi luật về bài học đều chỉ nói
*ghi*. Hệ quả đo được: bài học 162 của `peekvn` phát minh lại y nguyên dấu hiệu
nhận biết của bài học 2, sau 160 mục. Một file bài học chỉ-ghi là một file vô dụng.

### Nửa 2 — GHI, khi kết quả khác kỳ vọng

Ghi vào `lessons-learned.md` khi kế hoạch được làm **đúng** nhưng kết quả **khác**
dự tính. Không phải chỗ ghi bug — chỗ ghi bất ngờ. Mỗi mục phải nói rõ: *đã tin
gì*, *cái gì lộ ra*, *dấu hiệu nhận biết lần sau*. Thiếu dấu hiệu thì đó là một
câu chuyện, không phải bài học.

## Golden Rule: tài liệu phải khớp sự thật của mã

Xong một tính năng hay đổi kiến trúc là cập nhật `CONTEXT.md` + ADR **ngay**.
Tài liệu lệch mã thì **tài liệu sai**, và tính năng chưa xong.

Theo mẫu **LLM Wiki** của Andrej Karpathy: tài liệu không phải sản phẩm phụ của
việc phát triển, nó là một hiện vật cốt lõi, giữ với cùng mức nghiêm khắc như mã.

## Cấu trúc

| Đường dẫn | Chứa gì |
|---|---|
| `CONTEXT.md` | Ngữ vựng miền + quyết định đã xác minh + thực trạng app Windows |
| `INDEX.md` | **Mục lục chính — điều hướng qua đây** |
| `adr/` | Architecture Decision Records, `NNNN-kebab-case.md` |
| `features/<ten>/` | Bốn file: `overview` · `workflow` · `implementation` · `testing` |
| `plans/` | Ba tài liệu kế hoạch gốc (Master Plan, Kế hoạch chi tiết, TRS) |
| `agents/` | Cấu hình cho agent: issue tracker, nhãn triage, quy tắc domain |
| `lessons-learned.md` | Bài học khi kết quả khác kỳ vọng |
| `concepts/` `architecture/` `guides/` | Chưa dùng tới; thuật ngữ đang nằm gọn trong `CONTEXT.md` |

Thêm tài liệu mới thì phải cập nhật **cả** INDEX của thư mục **lẫn** `INDEX.md`
gốc, và kiểm không có link gãy.

## Issue

Spec và quyết định: `natuan1/peek-window-doc` (repo này).
Issue triển khai hằng ngày: `natuan1/peekvn`.

Tra cứu **động** bằng `gh issue view <id> --comments`. **Không** tin danh sách
issue chép trong bất kỳ file markdown nào — kể cả file này.

Nhãn triage bằng tiếng Việt, xem `agents/triage-labels.md`.

## Quy trình một ticket

Chuỗi đầy đủ nằm ở `peekvn/AGENTS.md` §2 (SOP 6 bước) và ở skill `/implement`.
Phần thuộc repo này là bước 6:

```
/update-doc  →  CONTEXT.md + ADR + features/ + lessons-learned  →  push master
```

Tính năng chỉ xong khi tài liệu đã cập nhật **và đã push**.

## Bí mật

`.gitignore` chặn `.env`, `*.token`, `*secret*`. Repo này không có build, không
có test, nên **không có gì chặn một file lạc vào ngoài chính `.gitignore`** —
đừng nới nó ra.

Credential Jenkins nằm ở biến môi trường người dùng (`JENKINS_USER`,
`JENKINS_TOKEN`), không ở file nào trong repo.
