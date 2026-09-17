# Seam interop cho Windows

## 📋 Mô tả

Windows gia nhập **interop suite** (`peekvn/interoperability/`) như implementation
thứ ba của ULTP, cạnh `reference/rust-host` (Rust) và `apps/ios/PeekKit` (Swift).

Suite này không so hai *thư viện* gọi lẫn nhau — nó so hai **tiến trình** độc
lập nói cùng giao thức. Đó là lý do nó là seam kiểm thử **chính** của app
Windows: một unit test in-process chỉ chứng minh mã nhất quán với chính nó, còn
cái phải chứng minh là Snappy nhất quán với `protocol/SPEC.md`.

## 🎯 Phạm vi

**Có:**

- Generator fixture xác định của SPEC §43 trong `Snappy.Protocol`.
- Một console exe `snappy-interop` trình ra đúng mặt cắt dòng lệnh mà harness
  gọi (`fixture-sha256`, `fixture-bytes`).
- Cặp `rust-host ↔ windows` chạy trong `interoperability/run.sh`.

**Không có (thuộc ticket sau):**

- mDNS, server ULTP, client — tức các cặp `transfer` và `tls-handshake` chưa có
  phía Windows. Seam này tồn tại để **những ticket đó** có sẵn vòng kiểm chứng.
- Cờ `--offset` trên CLI: hợp đồng dòng lệnh phải giống nhau ở cả ba phía, nên
  thêm một cờ chỉ Windows mới có là làm lệch chính cái mặt cắt đang được so.

## 👥 User stories

1. Là người viết một tính năng protocol cho Snappy, tôi muốn một lệnh chứng
   minh byte của Windows khớp byte của Rust, để không phải gỡ lỗi lệch byte
   giữa ba ngôn ngữ ở giai đoạn truyền file thật.
2. Là người đọc kết quả harness trên một máy chỉ có một phần toolchain, tôi
   muốn biết **cặp nào chưa đo**, để không nhầm "không chạy" thành "đã đạt".

## 🔗 Liên quan

- [ADR-0007 — Windows gia nhập interop suite](../../adr/0007-windows-gia-nhap-interop-suite.md)
- [ADR-0004 — Cấu trúc app Windows bốn project](../../adr/0004-cau-truc-app-windows-bon-project.md)
- Issue: [Ticket 03 (#134)](https://github.com/natuan1/peekvn/issues/134)

## Status

- Implementation: Xong 2026-09-17
- Documentation: Xong
- Cập nhật lần cuối: 2026-09-17
