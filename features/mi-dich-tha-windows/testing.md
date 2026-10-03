# Đích thả — kiểm thử

## Test tự động — `dotnet test apps/windows/Snappy.slnx`

706 test xanh (654 trước ticket). Nhóm mới:

| Lớp test | Khẳng định |
|---|---|
| `MiTargetsTests` | ô Shelf đầu trái, avatar kế tiếp, "Tất cả" dính mép phải; cột cao trọn khay; thiết bị thừa đi qua "Tất cả"; DPI; hit-test; mép trái + 40 px là ô Shelf ở mọi DPI × mọi số thiết bị, và **đối chứng**: x = 20% là avatar khi có ba máy; tô không bao giờ hạ alpha của thân; đích đang nhắm đổi màu; thanh tiến trình đúng tỉ lệ; chưa có byte thì không vẽ thanh |
| `SendPillTests` | mọi mép giờ ở đúng mili-giây (15 s không bị hỏi, 20 s byte đứng, 120 s chờ người, giữ 2/5 s); Paused không phải đứt và đếm lại từ đầu khi tiếp tục; không "0%"; xong một phần không nói "đã gửi"; không mã kỹ thuật nào lên pill |
| `PullTests` (+3) | thư mục thành từng tệp với `relativePath` giữ cây, đúng byte qua dây thật; thư mục không có tệp → không có gì để gửi; một thư mục con bị cấm liệt kê (ACL deny thật) → **cả lượt** `FileUnreadable` |
| `MiStateMachineTests` (+3) | `RunStarted` từ Nghỉ, từ khay đang nở, và khi pill đang chạy |
| `SendNoticesTests` (+1) | câu "dừng giữa chừng" không nói "hỏng" và chỉ việc làm tiếp |

**Đột biến đã ép đỏ:** cho `FilesUnder` trả danh sách rỗng thay vì `null` khi lỗi — test ACL đỏ.

## Nghiệm thu thật — `ci/check-mi-targets.ps1`

Kéo thật từ Explorer (`SendInput`, màn che + `Release-Over`), Snappy chạy ngoài gói MSIX, bản AOT.
Máy nhận là **iOS Simulator** chạy app iOS của repo, ghép đôi SAS thật với Snappy (hai đầu cùng
`146 784`). 📐 03/10/2026, năm lượt:

| Lượt | Kết quả |
|---|---|
| Thiết bị nhận được | khay nở với 3 avatar; nhắm/bỏ nhắm; ô Shelf; thả avatar → `Waiting -> Sending -> Done`, iPhone lưu tệp, SHA-256 trùng; thư mục → iPhone dựng lại `Thu muc C/con/hai.txt`, SHA-256 trùng; menu "Tất cả" chọn máy bằng bàn phím → pill; Esc và "Chỉ giữ ở Shelf" không gửi; bấm pill → ẩn, lượt gửi chạy tiếp |
| `-Offline` (app iPhone đóng) | `Pill Waiting -> Failed (NotSeen)` đúng 15,0 s; bong bóng "Đã mời … mở app trên … để nhận" |
| Thiết bị thật chưa có bảng năng lực | pill "Cần ghép đôi lại với iPhone của Tuấn" + hộp thoại hướng dẫn sau cú thả |
| `-StallMB 3000` + tắt Simulator giữa chừng | `interrupted after 67567616 of 3145728000 bytes`; `Failed (Stalled)` 20,0 s sau; bong bóng "dừng giữa chừng"; Mục ở lại Ngăn |
| CI Jenkins #155, #156 | đạt (vàng chỉ ở ký số); junit 706/0; bộ cài 11,01 MB |

Ảnh thật (vẽ trên màn hình): avatar đang nhắm sáng xanh, nhãn "iPhone của Tuấn" đủ chữ, pill
"Đang mời iPhone 17 Pro…" rồi "Đang gửi tới iPhone 17 Pro · 26%".

## Số đo RAM

| | private | tổng |
|---|---|---|
| sau kịch bản đầy đủ (ba lượt) | 7,52 · 8,09 · 9,67 MB ✅ KPI 25 | 28,91 · **31,32** · **32,78** MB ⚠️ red line 30 |
| A/B một lượt kéo, `main` | +0,84 · +0,83 | +3,04 · +3,00 |
| A/B một lượt kéo, nhánh | +0,99 · +1,04 | +3,92 · +3,89 (thêm `TextShaping.dll`) |

Cùng một exe `main` khởi động ở 23,3 rồi 25,9 MB trong mười phút — mốc nền trôi nhiều hơn phần của
cả ticket. Câu hỏi về red line ghi ở `peekvn/apps/windows/README.md` §"Red line RAM tổng".

## Treo

- **iPhone thật**: không cắm máy nào lúc làm ticket; cả hai thiết bị thật trong Trust Store có bản
  ghép đôi chưa có bảng năng lực (chỉ đo được đường "Cần ghép đôi lại").
- **Đa màn hình, DPI ≠ 100%**: máy dev một màn hình 100% (test thuần đã phủ DPI 96–288).
- **Capsule M3**: chưa tồn tại, hợp đồng `FootprintChanged` chưa có người nghe.
