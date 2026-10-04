# Lịch sử clipboard — luồng

## Một lượt copy

1. App khác đặt dữ liệu lên clipboard → Windows gửi `WM_CLIPBOARDUPDATE` tới cửa sổ ẩn của tiến trình nền (`AddClipboardFormatListener`).
2. Đang tạm dừng → dừng, không chạm clipboard (`Clipboard changed while paused - not read`).
3. Không có `CF_UNICODETEXT` → bỏ (ảnh, tệp).
4. Mở clipboard, hỏi **dấu** trước: `…FromMonitorProcessing`, `…FromMonitor`, `Clipboard Viewer Ignore` (có mặt là chặn); `CanIncludeInClipboardHistory` (0 hay không đọc được là chặn, 1 là cho phép). Có dấu chặn bằng sự có mặt thì không hỏi giá trị nào nữa. Bị chặn → `Skipped a copy from <exe>: it carries <dấu> - nothing read, nothing kept`.
5. Đọc chữ — quá 1 000 000 ký tự thì bỏ mà không chép vào RAM.
6. Rỗng → bỏ; đã có trong lịch sử → lên đầu; còn lại → thêm, cắt theo trần của gói.

## Dán lại

1. `Win+Alt+V` → nhớ cửa sổ gốc đang foreground (đích dán). Là Snappy (mở từ menu khay) → không có đích, chọn mục chỉ chép.
2. Bảng hiện ở giữa bề ngang, một phần tư chiều cao màn hình có con trỏ, lấy tiêu điểm. Windows không cho lên trước → ẩn ngay + bong bóng.
3. Chọn mục → ẩn bảng → đặt chữ lên clipboard (lượt này đưa mục lên đầu) → trả foreground về đích → sau 60 ms, chỉ khi đích **thật sự** đang foreground và không phím bổ trợ nào còn giữ, `SendInput` Ctrl+V. Không thoả sau ~1,5 s → không gõ gì, bong bóng "bấm Ctrl+V để dán".
4. Bấm ra ngoài hay Esc → đóng. Bấm phím tắt lần nữa → đóng.
