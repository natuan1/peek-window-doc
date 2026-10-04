# Lịch sử clipboard trên Windows (M4, Ticket 16)

Ticket: [natuan1/peekvn#147](https://github.com/natuan1/peekvn/issues/147) · Quyết định: [ADR-0020](../../adr/0020-lich-su-clipboard-loc-theo-dau-o-tien-trinh-nen.md) · Mã: `peekvn/apps/windows` (README mục "Lịch sử clipboard — Ticket 16")

## Người dùng được gì

- Copy chữ ở bất kỳ app nào → Snappy giữ nó trong **RAM của phiên**. Bản Free giữ **5 mục gần nhất**, Pro không giới hạn (cổng gate `Entitlements.ClipboardHistoryLimit`, Ticket 14). Đổi gói giữa phiên thì cắt ngay, giữ mục mới nhất.
- **`Win+Alt+V`** (hoặc menu khay "Lịch sử clipboard") mở **bảng dán nhanh**: mục mới nhất ở trên, mỗi dòng một dòng xem trước + "N ký tự · 3 phút trước". ↑↓ Enter hay bấm chuột là dán mục ấy vào cửa sổ vừa dùng; mục được dán lên đầu lịch sử.
- Hai nút trên bảng: **Tạm dừng / Tiếp tục** theo dõi (tạm dừng là không đọc clipboard), **Xoá sạch**.
- Mật khẩu từ trình quản lý mật khẩu **có tự đánh dấu** không bao giờ vào lịch sử — chữ của nó không được đọc ra.
- Thoát Snappy là quên hết. Không gì xuống đĩa; nhật ký chỉ ghi độ dài và exe nguồn.

## Lệch kế hoạch (đã đo)

| Kế hoạch / ticket | Thực tế | Vì sao |
|---|---|---|
| Phím `Win+Shift+V` | `Win+Alt+V` | 📐 `RegisterHotKey` trả 1409 — Windows giữ `Win+Shift+V` và `Win+V` |
| Dấu `ExcludeClipboardContentFromMonitor` | kiểm thêm `ExcludeClipboardContentFromMonitorProcessing` | 📐 tên có tài liệu, và là tên KeePassXC 2.7.12 đặt thật |
| "Mật khẩu KHÔNG BAO GIỜ vào lịch sử" | chỉ với app tự đánh dấu | 📐 `keepassxc-cli clip` không đặt dấu nào; như `Win+V` của Windows, Snappy giữ nó |
| Phím tắt "hoặc tùy biến" | chưa tùy biến được | Onboarding (Ticket 17) là chỗ đặt phím tắt |

## Treo

- **1Password, Bitwarden**: chưa thử — cả hai cần tài khoản. KeePassXC là trình quản lý mật khẩu thật duy nhất đã đo.
