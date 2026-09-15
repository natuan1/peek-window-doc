# Lessons Learned

Tài liệu này ghi lại những bài học khi kế hoạch được thực hiện đúng nhưng kết quả không như mong muốn.

## Mục đích

Khi một tính năng được triển khai **đúng theo kế hoạch** nhưng **kết quả khác biệt với kỳ vọng**, chúng ta ghi lại tại sao điều này xảy ra.

Đây không phải là các lỗi cần sửa, mà là những **phát hiện bất ngờ** giúp cải thiện hiểu biết về dự án.

## Cấu trúc

Mỗi mục ghi bài học theo định dạng:

```markdown
## [YYYY-MM-DD] Tiêu đề tính năng / bài học

**Kế hoạch**: Điều gì được dự định sẽ xảy ra?

**Kết quả thực tế**: Điều gì thực sự xảy ra?

**Bài học**: Tại sao có sự khác biệt? Giả định nào của chúng ta sai?

**Hành động tiếp theo**: Cần làm gì để cải thiện?
```

## Ví dụ

```markdown
## [2026-09-13] Tính năng X hoàn thành nhưng chỉ số không cải thiện như mong đợi

**Kế hoạch**: Triển khai tính năng X, dự kiến sẽ cải thiện metric Y thêm 30%

**Kết quả thực tế**: Tính năng hoạt động đúng, nhưng metric Y chỉ tăng 5%

**Bài học**: Giả định của chúng ta về hành vi người dùng không chính xác. 
Người dùng không sử dụng tính năng X vì [lý do]. 
Chúng ta cần hiểu rõ hơn nhu cầu thực tế của họ.

**Hành động tiếp theo**: Nghiên cứu vì sao người dùng không dùng tính năng này
```

---

*Được cập nhật từ: CLAUDE.md → Agent Communication Rules → Lessons Learned Protocol*

---

## [2026-09-15] Icon khay chạy đúng, nhưng người dùng Windows 11 không thấy nó

**Kế hoạch**: Ticket 01 hứa "tray icon: mở bảng trạng thái nhỏ". Giả định ngầm là icon xuất hiện ở khay thì người dùng thấy và bấm được — đó là toàn bộ giao diện của app ở giai đoạn này.

**Kết quả thực tế**: Mã chạy đúng — `Shell_NotifyIcon(NIM_ADD)` trả thành công, icon tồn tại thật. Nhưng khi chụp lại vùng khay hệ thống để kiểm chứng thì **không thấy icon Snappy đâu**: Windows 11 mặc định giấu mọi icon khay mới vào phần tràn sau dấu `^`, và người dùng phải tự bấm mở rồi kéo nó ra ngoài.

**Bài học**: "API trả thành công" và "người dùng thấy" là hai chuyện khác nhau, và khoảng cách giữa chúng nằm ở một mặc định của hệ điều hành mà không có API nào vượt qua được (Microsoft cố ý không cho app tự bỏ ẩn). Với người dùng, app vừa cài mà khay không có gì nghĩa là **app chưa tồn tại** — đúng câu hỏi số 1 của AGENTS.md §7.1: "người dùng tới đây bằng cách nào?". Nếu không có phép kiểm bằng mắt, lỗi này sẽ đi hết Ticket 01→16 mà không ai phát hiện, vì mọi test tự động đều xanh.

**Hành động tiếp theo**: [Ticket 17 (Onboarding)](https://github.com/natuan1/peek-window-doc/issues/18) phải coi "dạy người dùng ghim icon Snappy ra khay" là một bước bắt buộc, không phải mẹo phụ — kèm ảnh chỉ đúng dấu `^`. Và mọi ticket có bề mặt nhìn thấy được từ đây về sau phải nghiệm thu bằng ảnh chụp thật, không bằng mã trả về.
