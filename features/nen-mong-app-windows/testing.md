# Nền móng app Windows — kiểm thử

## Seam chính không nằm trong app Windows

Seam kiểm thử chính của Windows là **interop suite** `peekvn/interoperability/run.sh`: hai *tiến trình* độc lập nói cùng protocol. Một test in-process chỉ chứng minh mã nhất quán với chính nó; cái cần chứng minh là Windows nhất quán với `protocol/SPEC.md`, đúng cách mà rust-host và PeekKit đã phải chứng minh. Windows gia nhập suite đó ở [Ticket 03](https://github.com/natuan1/peekvn/issues/134).

Ticket 01 chưa nói protocol nên chưa chạm được seam đó. Những gì có ở đây là **biên phụ**.

## Biên phụ: 37 test xUnit

`apps/windows/tests/Snappy.Tests` — chỉ phần logic chạy được mà không cần hệ điều hành:

| Nhóm | Kiểm cái gì |
|---|---|
| `AppLifecycleTests` | `BeginShutdown()` trả `true` đúng một lần, kể cả khi hỏng lúc khởi động; xin thoát hai lần không dọn hai lần |
| `DisplayScaleTests` | Quy đổi DIP → pixel ở 100/125/150/175/200/250%; làm tròn về gần nhất chứ không cắt cụt; giá trị dương không bao giờ ra 0; DPI = 0 rơi về 96 thay vì chia cho 0 |
| `TrayMenuTests` | Menu có đủ hai mục người dùng cần; nhãn tiếng Việt không lọt mã kỹ thuật; id bắt đầu từ 1 (vì `TPM_RETURNCMD` trả 0 khi người dùng bấm huỷ) |
| `SnappyPathsTests` | Gốc nằm dưới `%LocalAppData%\Snappy`; mọi thư mục con nằm trong gốc; **không** tạo sẵn TempDrops |
| `SingleInstanceTests` | Bản thứ hai không giành được khoá; nhả ra rồi thì bản sau giành được; khoá là `Local\` chứ không `Global\` |

```sh
dotnet test apps/windows/Snappy.slnx
```

## Hàng rào CI — Jenkins nội bộ

Job `snappy-windows` trên `http://192.168.1.235:9096`, agent là **máy dev Windows 11 thật** (`win11-snappy`). Pipeline khai báo trong `apps/windows/Jenkinsfile`. Vì sao không phải GitHub Actions: [ADR-0005](../../adr/0005-ci-cd-desktop-qua-jenkins-noi-bo.md).

Sáu bước, một lượt ~65 giây:

1. **Lấy mã** — deploy key SSH chỉ-đọc cho riêng `peekvn`.
2. **Biên dịch** — `dotnet build -c Release`.
3. **Test** — 37 test, xuất JUnit nên Jenkins báo cáo từng test chứ không chỉ "bước này đỏ".
4. **Publish Native AOT** — gọi đúng `publish.cmd` mà người thật gõ. Bước riêng vì `dotnet build` xanh mà publish AOT đỏ là chuyện xảy ra thật: ILCompiler mới là cái phát hiện reflection lọt vào mã.
5. **Hàng rào KPI** — `ci/check-artifacts.ps1`: thư mục publish chỉ được có `Snappy.exe` + `Snappy.pdb`, exe dưới trần dung lượng.
6. **Chạy thử app thật** — `ci/smoke-test.ps1`: khởi động Snappy, kiểm cửa sổ host tồn tại, đọc nhật ký của chính app để biết icon khay thêm được, đo RAM nền, thoát bằng `WM_CLOSE`, đếm tiến trình sót.

Bước 6 là bước đáng giá nhất và là bước runner đám mây không làm được. Bản đầu tiên của Ticket 01 publish sạch rồi chết ngay lúc khởi động (mã 1400) — không có bước này thì lỗi đó đi thẳng tới người dùng.

Hai script hàng rào nằm trong repo nên chạy được y hệt trên máy dev:

```
powershell -File apps\windows\ci\check-artifacts.ps1
powershell -File apps\windows\ci\smoke-test.ps1
```

**Cả bốn đường đỏ của chúng đã được ép cho đỏ một lần** — file lạ trong thư mục publish, vỡ trần dung lượng, thiếu exe, vỡ KPI RAM. Một hàng rào chưa bao giờ đỏ là một hàng rào chưa được chứng minh.

> ⚠️ **GitHub Actions vẫn tắt** (`workflow_dispatch`). Hệ quả: các job Rust/Swift/protocol **không có hàng rào tự động nào** — chỉ app Windows có. Bảng lệnh chạy tay ở `peekvn/AGENTS.md` §1 giữ chỗ đó.

## Còn phải kiểm bằng tay (trên Windows 11)

1. **Icon khay** — nhìn thấy và bấm được. Windows 11 mặc định giấu icon mới vào phần tràn sau dấu `^`; xem [bài học 2026-09-15](../../lessons-learned.md).
2. **DPI** — đưa con trỏ sang màn hình có mức scale khác rồi bấm icon khay; bảng phải hiện sắc nét ở đúng tỉ lệ màn hình đó. (Không phải "kéo cửa sổ giữa hai màn hình" như tiêu chí gốc viết — bảng không có thanh tiêu đề để kéo.)
3. **Explorer restart** — kết thúc `explorer.exe` từ Task Manager rồi chạy lại; icon Snappy phải tự quay lại khay.

Nhánh xử lý `TaskbarCreated` **đã kiểm** bằng cách gửi thẳng thông điệp tới cửa sổ app (không kill Explorer): app sống, ghi nhật ký "Explorer khởi động lại — thêm lại icon khay", bảng trạng thái vẫn mở được, thoát sạch. Phần chưa kiểm là Explorer thật có phát thông điệp đó đúng lúc không.

## Treo tới khi có máy

**Windows 10 1809 sạch** — chạy exe trên máy chưa cài .NET runtime. Quyết định 2026-09-15: ưu tiên Windows 11 trước; floor sản phẩm không đổi và `SupportedOSPlatformVersion` vẫn ghim 10.0.17763.0, nên khi có máy thì đây là *thêm một phép thử*, không phải thêm một đợt sửa mã.
