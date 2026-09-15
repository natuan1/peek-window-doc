# Nền móng app Windows — kiểm thử

## Seam chính không nằm trong app Windows

Seam kiểm thử chính của Windows là **interop suite** `peekvn/interoperability/run.sh`: hai *tiến trình* độc lập nói cùng protocol. Một test in-process chỉ chứng minh mã nhất quán với chính nó; cái cần chứng minh là Windows nhất quán với `protocol/SPEC.md`, đúng cách mà rust-host và PeekKit đã phải chứng minh. Windows gia nhập suite đó ở [Ticket 03](https://github.com/natuan1/peek-window-doc/issues/4).

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

## Hàng rào CI

Job `windows` trong `.github/workflows/ci.yml` (`windows-latest`):

1. `dotnet build` — biên dịch.
2. `dotnet test` — 37 test trên.
3. `dotnet publish -c Release` — **AOT**. Bước riêng, vì `dotnet build` xanh mà publish AOT đỏ là chuyện xảy ra thật: ILCompiler mới là cái phát hiện reflection lọt vào mã.
4. Đếm file trong thư mục publish + đo kích thước exe.
5. **Chạy thử exe** rồi đo working set. Bước này có vì bản đầu tiên của ticket publish sạch rồi chết ngay lúc khởi động (mã 1400) — không có nó thì lỗi đó đi thẳng tới người dùng.

> ⛔ **CI đã tắt chạy tự động (2026-09-15).** Tài khoản GitHub Actions bị chặn vì lý do thanh toán — mọi job, kể cả Rust/Swift có sẵn, đều không khởi động được. `ci.yml` chuyển sang `workflow_dispatch`: để `on: push` thì mỗi commit đẻ ra một run đỏ không mang tin gì, và một hàng rào luôn đỏ là hàng rào không ai đọc nữa.
>
> **Hàng rào duy nhất lúc này là máy dev.** Trước khi đóng bất kỳ issue Windows nào, bắt buộc chạy tay `dotnet test apps/windows/Snappy.slnx` **và** `apps\windows\publish.cmd` — test xanh không chứng minh publish AOT xanh.
>
> Khôi phục: bỏ chú thích hai khối `push`/`pull_request` ở đầu `ci.yml`. Job còn nguyên, không phải dựng lại. Job `windows` vì vậy **chưa được chứng minh là chạy được trên runner**.

## Còn phải kiểm bằng tay (trên Windows 11)

1. **Icon khay** — nhìn thấy và bấm được. Windows 11 mặc định giấu icon mới vào phần tràn sau dấu `^`; xem [bài học 2026-09-15](../../lessons-learned.md).
2. **DPI** — đưa con trỏ sang màn hình có mức scale khác rồi bấm icon khay; bảng phải hiện sắc nét ở đúng tỉ lệ màn hình đó. (Không phải "kéo cửa sổ giữa hai màn hình" như tiêu chí gốc viết — bảng không có thanh tiêu đề để kéo.)
3. **Explorer restart** — kết thúc `explorer.exe` từ Task Manager rồi chạy lại; icon Snappy phải tự quay lại khay.

Nhánh xử lý `TaskbarCreated` **đã kiểm** bằng cách gửi thẳng thông điệp tới cửa sổ app (không kill Explorer): app sống, ghi nhật ký "Explorer khởi động lại — thêm lại icon khay", bảng trạng thái vẫn mở được, thoát sạch. Phần chưa kiểm là Explorer thật có phát thông điệp đó đúng lúc không.

## Treo tới khi có máy

**Windows 10 1809 sạch** — chạy exe trên máy chưa cài .NET runtime. Quyết định 2026-09-15: ưu tiên Windows 11 trước; floor sản phẩm không đổi và `SupportedOSPlatformVersion` vẫn ghim 10.0.17763.0, nên khi có máy thì đây là *thêm một phép thử*, không phải thêm một đợt sửa mã.
