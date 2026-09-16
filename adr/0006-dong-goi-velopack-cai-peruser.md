# ADR-0006: Đóng gói Velopack — cài `PerUser`, gốc cài trùng gốc dữ liệu, KPI 15MB chuyển sang bộ cài

Date: 2026-09-16
Status: Accepted

## Context

[Spec #1](https://github.com/natuan1/peek-window-doc/issues/1) đã chốt Velopack từ trước: *"cài user-space `%LocalAppData%\Snappy\`, không UAC; `VelopackApp.Build().Run()` ở entry; kiểm tra cập nhật mỗi 4 giờ, tải delta, áp khi khởi động lại"*. ADR này **không** ghi lại lựa chọn ấy — nó ghi những quyết định phải ra **trong lúc** hiện thực [Ticket 02 (#3)](https://github.com/natuan1/peek-window-doc/issues/3), mỗi cái đều có một đường mặc định dễ đi hơn và sai.

Bốn câu hỏi thật sự phải trả lời:

1. **"Không popup UAC" là một lời hứa hay một cấu hình?** Mặc định `--instLocation` của `vpk` là `Either` — bộ cài hỏi người dùng muốn cài ở đâu. Với một tiêu chí nghiệm thu ghi thẳng "không popup UAC", "để người dùng chọn" nghĩa là một phần số lần cài sẽ có UAC.
2. **App cài ở đâu so với dữ liệu app?** `--packId Snappy` kéo theo thư mục cài `%LocalAppData%\Snappy` — **đúng** thư mục mà `SnappyPaths.Root` đã dùng từ Ticket 01.
3. **KPI 15MB đo trên vật gì?** Ticket 01 đo exe và tự ghi chú rằng đó là số tạm, phải chuyển sang bộ cài khi bộ cài ra đời.
4. **Bao nhiêu phần của logic cập nhật được phép nằm trong Velopack?** Nhịp 4 giờ là một tiêu chí nghiệm thu; một tiêu chí không kiểm được là một tiêu chí không tồn tại.

## Decision

### 1. `--instLocation PerUser`, khai báo tường minh

Không dựa vào mặc định. `PerUser` giữ app ra khỏi Program Files, và chính vì ra khỏi Program Files nên bộ cài chạy được với `asInvoker` — tức Windows không có lý do gì để hỏi UAC.

**Hàng rào**: `ci/check-installer.ps1` đọc thẳng manifest nhúng trong `Snappy-win-Setup.exe` và đỏ nếu thấy `requireAdministrator` / `highestAvailable`, hoặc nếu không thấy `asInvoker`. UAC do đúng một dòng `requestedExecutionLevel` quyết định, nên đó là chỗ duy nhất tự động hoá được tiêu chí này.

Ngược lại, **"cài vào user-space" KHÔNG có hàng rào riêng** — và đó cũng là một quyết định. `--instLocation` không để lại dấu vết nào trong artifact để soi, nên mọi phép thử viết ra cho câu đó sẽ là một dòng luôn xanh. Phép thử manifest thay thế nó: một bộ cài `asInvoker` không ghi nổi vào Program Files.

### 2. Gốc cài **trùng** gốc dữ liệu, có chủ ý

`%LocalAppData%\Snappy\` vừa là thư mục Velopack cài app (`current\`, `packages\`, `Update.exe`) vừa là `SnappyPaths.Root` (`logs\`, `TempDrops\`). Hệ quả tốt: **gỡ app là xoá sạch dấu vết**, đúng lời hứa của `SnappyPaths`, không cần một bước dọn riêng mà người dùng phải tin là có chạy.

Hệ quả phải trả giá: **`VelopackApp.Build().Run()` bắt buộc là dòng đầu tiên của `Main`**. Dựng `logs\` trước nó nghĩa là ở lượt hook `--veloapp-uninstall`, app tự tạo lại một thư mục ngay trong cây mà `Update.exe` đang xoá. Đổi lại, trong các lượt chạy hook thì nhật ký chưa mở, nên `AppLog` rơi xuống `OutputDebugString` — không mất dấu vết, chỉ đổi đường, và đổi đúng chiều vì một lượt gỡ không nên ghi gì xuống đĩa.

### 3. KPI 15MB chuyển sang bộ cài

`ci/check-artifacts.ps1` **không còn** trần dung lượng. Nó giữ đúng việc của nó: thư mục publish chỉ được có `Snappy.exe` + `Snappy.pdb`. Trần 15MB sống ở `ci/check-installer.ps1`, đo `Snappy-win-Setup.exe`.

Giữ hai con số cùng mang tên "KPI" thì không con số nào là KPI. Exe không nén, bộ cài thì có, và chỉ một trong hai là thứ người dùng tải về.

### 4. Quyết định cập nhật nằm ngoài Velopack

`UpdateCoordinator` giữ **toàn bộ** phần có thể sai: bao lâu kiểm một lần, hỏng thì làm gì, khi nào dừng, và nói gì với người dùng. Velopack nằm sau `IUpdateSource`, mỏng tới mức không còn quyết định nào.

Nhịp 4 giờ đi qua `TimeProvider` chứ không `Task.Delay` trần, nên test tua đồng hồ giả. Đây là chỗ duy nhất chứng minh được con số 4 giờ mà không phải ngồi chờ bốn tiếng.

### 5. Ký số chạy trên Jenkins, không phải GitHub Actions

Spec #1 ghi *"ký số Azure Trusted Signing trên GitHub Actions (`windows-latest`)"*. Câu đó viết trước [ADR-0005](0005-ci-cd-desktop-qua-jenkins-noi-bo.md): Actions đã tắt, CI desktop đã chuyển sang Jenkins nội bộ. Đường ký số đi theo CI thật, không theo câu chữ cũ.

`vpk` mang sẵn cả dlib của Azure lẫn `signtool`, nên chỉ cần sáu secret. Ký **chỉ** chạy khi build có tham số `PHAT_HANH` — mỗi lần ký là một lời gọi tính tiền và ghi nhật ký kiểm toán, và một bí mật có mặt trong mọi lượt build là một bí mật sớm muộn cũng rơi vào log của một bước không liên quan.

Bước `Kiểm chữ ký` có **ba** màu: xanh khi `signtool verify /pa /v` thật sự chạy và pass, **vàng** khi chưa ký (signtool chưa hề được gọi), đỏ khi chữ ký hỏng. Gộp "chưa ký" vào màu xanh là nói dối đúng chỗ dễ tin nhất.

## Consequences

**Được:**

- "Không UAC" có hàng rào máy móc, không phải lời hứa. Đã ép đỏ bằng một exe `requireAdministrator` thật và bằng một exe không có manifest.
- KPI đo đúng vật: bộ cài **10,02 MB** / trần 15 MB.
- Gỡ app sạch tuyệt đối — đã kiểm: `Update.exe --uninstall` xoá hết `%LocalAppData%\Snappy`, không sót file nào.
- Nhịp 4 giờ và toàn bộ máy trạng thái cập nhật kiểm được in-process, không cần máy chủ phát hành.

**Trả giá:**

- **Velopack ăn ~6 MB exe và ~2,5 MB RAM nền**: exe 1,69 → 7,71 MB, working set 12,51 → 15,04 MB. Cả hai vẫn dưới KPI, nhưng biên đã hẹp đi thật, và Velopack là thư viện *đầu tiên* của cả bốn project. Mọi ticket sau nên đọc bảng số đo trong `apps/windows/README.md` trước khi thêm thư viện thứ hai.
- Gỡ app xoá luôn nhật ký và TempDrops. Đúng ý muốn, nhưng nghĩa là **không có cách nào gỡ lỗi một lần gỡ hỏng bằng nhật ký trên đĩa** — phải đọc `OutputDebugString`.
- Thứ tự trong `Main` trở thành một ràng buộc ngầm: ai chèn một dòng chạm đĩa lên trước `VelopackApp.Run()` sẽ làm hỏng lượt gỡ, và không có test nào bắt được.
- `--packId Snappy` giờ là bất biến: đổi nó là đổi chỗ ở của app, và bản cũ trên máy người dùng thành một bản mồ côi không ai gỡ.

**Chưa nghiệm thu được, để treo có chủ ý:**

- `signtool verify /pa /v` pass — chưa có tài khoản Azure Trusted Signing. Đường ống đã dựng xong và đã ép đỏ ở nhánh "chưa ký"; cái thiếu là credential, không phải mã.
- SmartScreen không cảnh báo trên máy sạch — phụ thuộc cả chữ ký lẫn danh tiếng tích luỹ của chứng chỉ, nên kể cả có credential hôm nay thì hôm nay cũng chưa kiểm được.
- **Hạ tầng phát hành thật chưa tồn tại.** Spec #1 đã đẩy hạ tầng web ra ngoài phạm vi. Tới khi có, lượt kiểm hỏng êm và bảng trạng thái nói "Chưa tìm được bản mới lúc này. Snappy sẽ tự thử lại sau." — không đổ lỗi cho mạng, vì kênh phát hành im lặng vì chục lý do khác.

## Alternatives Considered

- **`--instLocation Either` (mặc định)** — để người dùng chọn. Bỏ vì đó chính là cách sinh ra popup UAC mà tiêu chí nghiệm thu cấm, và vì một lựa chọn mà người dùng không có đủ thông tin để chọn là một câu hỏi không nên hỏi.
- **Tách gốc dữ liệu ra khỏi gốc cài** (ví dụ `%LocalAppData%\Snappy.Data\`). Tránh được ràng buộc thứ tự trong `Main`, nhưng đổi lấy việc gỡ app để lại rác mà không ai dọn — và `SnappyPaths` đã hứa ngược lại từ Ticket 01.
- **Giữ trần 15MB ở cả exe lẫn bộ cài.** Bỏ vì hai con số cùng tên "KPI" thì khi một cái đỏ sẽ không ai biết cái nào mới là lời hứa với người dùng.
- **Gọi thẳng `UpdateManager` từ `AppHost`**, không có `IUpdateSource`. Ít mã hơn thật, nhưng nhịp 4 giờ và mọi nhánh hỏng sẽ không có cách nào kiểm ngoài việc chờ thật.

## Related

- [ADR-0005](0005-ci-cd-desktop-qua-jenkins-noi-bo.md) — vì sao ký số chạy ở Jenkins chứ không GitHub Actions
- [ADR-0004](0004-cau-truc-app-windows-bon-project.md) — bốn project; `Snappy.Core` là nơi `UpdateCoordinator` sống
- [ADR-0002](0002-ui-stack-aot-spike-pass.md) — số đo AOT gốc mà ADR này vừa làm hẹp biên
- [Feature: Đóng gói & tự cập nhật](../features/dong-goi-va-tu-cap-nhat/overview.md)
- [Ticket 02 (#3)](https://github.com/natuan1/peek-window-doc/issues/3)

## Decision Log

- 2026-09-16: Accepted, cùng lượt implement Ticket 02
