# Đích thả — kiểm thử

## Test tự động — `dotnet test apps/windows/Snappy.slnx`

706 test xanh (654 trước ticket). Nhóm mới:

| Lớp test | Khẳng định |
|---|---|
| `MiTargetsTests` | ô Shelf đầu trái, avatar kế tiếp, "Tất cả" dính mép phải; cột cao trọn khay; thiết bị thừa đi qua "Tất cả"; DPI; hit-test; mép trái + 40 px là ô Shelf ở mọi DPI × mọi số thiết bị, và **đối chứng**: x = 20% là avatar khi có ba máy; tô không bao giờ hạ alpha của thân; đích đang nhắm đổi màu; thanh tiến trình đúng tỉ lệ; chưa có byte thì không vẽ thanh |
| `SendPillTests` | mọi mép giờ ở đúng mili-giây (45 s không bị hỏi, 20 s byte đứng, 120 s chờ người, giữ 2/5 s); Paused không phải đứt và đếm lại từ đầu khi tiếp tục; không "0%"; xong một phần không nói "đã gửi"; không mã kỹ thuật nào lên pill |
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
| `-Offline` (app iPhone đóng) | `Pill Waiting -> Failed (NotSeen)` đúng hạn (15 s lúc đo, nay 45 s); bong bóng "Đã mời … mở app trên … để nhận" |
| Thiết bị thật chưa có bảng năng lực | pill "Cần ghép đôi lại với iPhone của Tuấn" + hộp thoại hướng dẫn sau cú thả |
| `-StallMB 3000` + tắt Simulator giữa chừng | `interrupted after 67567616 of 3145728000 bytes`; `Failed (Stalled)` 20,0 s sau; bong bóng "dừng giữa chừng"; Mục ở lại Ngăn |
| CI Jenkins #155, #156 | đạt (vàng chỉ ở ký số); junit 706/0; bộ cài 11,01 MB |

Ảnh thật (vẽ trên màn hình): avatar đang nhắm sáng xanh, nhãn "iPhone của Tuấn" đủ chữ, pill
"Đang mời iPhone 17 Pro…" rồi "Đang gửi tới iPhone 17 Pro · 26%".

## iPhone thật — 03/10/2026, qua iPhone Mirroring

- Bản ghép đôi cũ (23/9, trước Ticket 09) → pill "Cần ghép đôi lại"; làm đúng như hộp thoại: quên NATUAN1 trên iPhone, ghép lại (SAS `948 604` hai đầu).
- Thả lên avatar "iPhone của Tuấn" → iPhone hiện "Có tệp đang chờ bạn" → Nhận → "Đã nhận gui di B.txt từ NATUAN1".
- **Lộ lỗi:** ngưỡng "Không thấy" 15 s báo nhầm offline 0,9 s trước lần hỏi đầu của iPhone. 76 lần hỏi đo được: 17% khe dài 15–21 s. Ngưỡng nay 45 s (bài học 208 của `peekvn`).
- Lượt sau iPhone được cầm lên, app xuống nền, **0** lần hỏi: `Failed (NotSeen)` ở giây 45 — báo đúng.
- ⚠️ Lời mời thư mục xếp hàng sau một hộp mời đang mở **không hiện** trên iPhone dù được kéo về hơn 20 lần — nghi ở hàng đợi lời mời phía iOS, chưa đo riêng, tách việc.
- **Lượt đủ 17:24, ngưỡng 45 s:** B chờ 24 s rồi `Waiting -> Sending -> Done`; thư mục → "Đã nhận 2 tệp" (`hai.txt`, `mot.txt`) và pill `Done`; bấm pill ẩn, lượt gửi chạy tiếp; menu "Tất cả" gửi tới iPhone, Esc / "Chỉ giữ ở Shelf" không gửi. Lời mời thư mục lần này hiện ngay — hàng đợi ở dòng trên vẫn chưa giải thích.

## Số đo RAM

| | private | tổng |
|---|---|---|
| sau kịch bản đầy đủ (ba lượt) | 7,52 · 8,09 · 9,67 MB ✅ KPI 25 | 28,91 · **31,32** · **32,78** MB ⚠️ red line 30 |
| A/B một lượt kéo, `main` | +0,84 · +0,83 | +3,04 · +3,00 |
| A/B một lượt kéo, nhánh | +0,99 · +1,04 | +3,92 · +3,89 (thêm `TextShaping.dll`) |

Cùng một exe `main` khởi động ở 23,3 rồi 25,9 MB trong mười phút — mốc nền trôi nhiều hơn phần của
cả ticket. Câu hỏi về red line ghi ở `peekvn/apps/windows/README.md` §"Red line RAM tổng".

## Treo

- **Đứt giữa chừng trên iPhone thật**: không đo được qua Mirroring (đã đo trên Simulator).
- **Đa màn hình, DPI ≠ 100%**: máy dev một màn hình 100% (test thuần đã phủ DPI 96–288).
- **Capsule M3**: chưa tồn tại, hợp đồng `FootprintChanged` chưa có người nghe.
