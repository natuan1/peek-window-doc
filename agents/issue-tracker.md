# Issue tracker

**Mọi issue của dự án nằm ở một nơi duy nhất: `natuan1/peekvn`.**

Chốt 2026-09-17. Trước đó ticket Windows nằm ở `peek-window-doc` còn code và CI
ở `peekvn` — hai tracker cho một dòng công việc, và cái giá của nó đo được:

- PR [`peekvn#130`](https://github.com/natuan1/peekvn/pull/130) **không tự đóng
  được** Ticket 02. `Closes #N` không đi xuyên repo, phải đóng bằng tay.
- `gh issue view 3` khi đứng trong `peekvn` ra một issue hoàn toàn khác. Mọi lệnh
  đều phải kèm `--repo`, và quên một lần là đọc nhầm ticket.

19 issue đã được chuyển sang `peekvn` (số mới = số cũ + 130; Spec cũ `#1` giờ là
[`#131`](https://github.com/natuan1/peekvn/issues/131)). GitHub giữ chuyển hướng
từ URL cũ, nên link cũ vẫn tới đúng chỗ — nhưng **con số thì không**: `#3` viết
trong ngữ cảnh `peekvn` giờ trỏ vào một issue khác hẳn.

## Repo này giữ gì

`peek-window-doc` (tức `D:\code\peek-docs`) **không còn issue nào**. Nó giữ phần
tài liệu sống lâu hơn một ticket:

- `CONTEXT.md` — ngữ vựng miền và quyết định đã xác minh
- `adr/` — vì sao đã chọn thế
- `features/` — mỗi tính năng bốn file
- `plans/` — ba tài liệu kế hoạch gốc
- `lessons-learned.md` — bài học khi kết quả khác kỳ vọng

Spec sản phẩm giờ là một **issue** ở `peekvn`
([#131](https://github.com/natuan1/peekvn/issues/131)), không phải file ở đây.

## Lệnh

Trong `peekvn` thì không cần `--repo`; từ chỗ khác thì bắt buộc:

```bash
gh issue view <id> --comments
gh issue list --state open --label "sẵn-sàng-cho-agent"
gh issue create --title "..." --body "..."
gh issue comment <id> --body "..."
gh issue edit <id> --add-label "..." --remove-label "..."
gh issue close <id> --comment "..."
```

Body nhiều dòng thì dùng heredoc.

**Tra cứu động, luôn luôn.** Không tin danh sách issue chép trong bất kỳ file
markdown nào — kể cả file này. Danh sách chép tay là danh sách sẽ mục, và lần
chuyển repo vừa rồi vừa chứng minh điều đó với 18 tham chiếu phải sửa tay.

## Nhãn

Năm nhãn triage tiếng Việt, xem [`triage-labels.md`](triage-labels.md). `peekvn`
đã có sẵn cả năm nên lượt chuyển giữ nguyên được nhãn.

## PR

`peekvn` nhận PR; ticket và code giờ cùng một repo nên `Closes #<id>` chạy thật.
Đó chính là lý do của lần chuyển này.
