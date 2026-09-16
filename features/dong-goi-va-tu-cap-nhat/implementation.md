# Đóng gói & tự cập nhật — hiện thực

Mã ở `peekvn/apps/windows/`, nhánh `ticket/02-velopack-ky-so`.

## Seam: quyết định ở một bên, Velopack ở bên kia

```
AppHost                    ← Win32, thread thông điệp
   │  PostMessageW
   ▼
UpdateCoordinator          ← MỌI quyết định, thread pool, không chạm Win32
   │  IUpdateSource
   ▼
VelopackUpdateSource       ← mỏng tới mức không còn quyết định nào
   │
   ▼
Velopack.UpdateManager
```

**Vì sao cắt ở đây.** Nhịp 4 giờ là một tiêu chí nghiệm thu, và một tiêu chí không kiểm được là một tiêu chí không tồn tại. Gọi thẳng `UpdateManager` từ `AppHost` thì ít mã hơn thật, nhưng nhịp ấy cùng mọi nhánh hỏng sẽ không có cách nào kiểm ngoài việc chờ thật.

`UpdateCoordinator` giữ: bao lâu kiểm một lần, hỏng thì làm gì, khi nào dừng, **và mọi chữ người dùng đọc**. Chữ viết thẳng ở chỗ vẽ là chữ không test nào với tới — và AGENTS.md §7.2 cấm đúng loại chữ lọt ra màn hình.

## File chính

| File | Vai trò |
|---|---|
| `src/Snappy.Core/IUpdateSource.cs` | Kênh phát hành nhìn từ phía app: `IsInstalled`, `CheckAsync`, `DownloadAsync`, `ApplyOnExit`, `ApplyAndRestartNow` |
| `src/Snappy.Core/UpdateCoordinator.cs` | Vòng 4 giờ, máy trạng thái 7 trạng thái, và các hàm static sinh chữ cho người dùng |
| `src/Snappy.Core/VelopackUpdateSource.cs` | Adapter thật. Chọn kênh theo dạng của `SNAPPY_UPDATE_FEED`: URL GitHub → `GithubSource`, thư mục có thật → `SimpleFileSource`, còn lại → `SimpleWebSource` |
| `src/Snappy.Core/VelopackLogBridge.cs` | `IVelopackLogger` → `AppLog`. Không nối vào thì mọi lý do một lượt cập nhật hỏng đều biến mất (AGENTS.md §6) |
| `src/Snappy.Core/Program.cs` | `VelopackApp.Build().Run()` là dòng đầu tiên |
| `src/Snappy.Core/AppHost.cs` | Khởi động vòng kiểm, cầu `PostMessageW` từ thread pool về thread thông điệp, mục menu khay |
| `src/Snappy.Core/TrayMenu.cs` | `Build(updateReady)` — mục "Khởi động lại để cập nhật" chỉ có khi bấm vào nó thật sự xảy ra chuyện gì |
| `src/Snappy.Interop/TrayIcon.cs` | Thêm `UpdateTooltip` |
| `ci/pack.ps1` | Dựng bộ cài; đối chiếu phiên bản; dựng metadata Trusted Signing từ biến môi trường |
| `ci/check-installer.ps1` + `ci/PeManifest.cs` | KPI 15MB trên bộ cài; đọc manifest nhúng để chứng minh không đòi UAC |
| `ci/check-delta.ps1` | Tự dựng phần "đã đổi" rồi đo gói vá |
| `ci/verify-signature.ps1` | `signtool verify /pa /v` trên cả bộ cài lẫn app bên trong gói |
| `pack.cmd` | Một lệnh cho máy dev: publish → đóng gói → hàng rào bộ cài |

## Máy trạng thái cập nhật

```
NotChecked ──┬─► NotInstalled                      (chạy từ publish\, dừng hẳn)
             ├─► Checking ──┬─► UpToDate    ──┐
             │              ├─► CheckFailed ──┤ chờ 4 giờ, lặp
             │              └─► Downloading ──┴─► PendingRestart (dừng hẳn)
```

Trạng thái và số phiên bản nằm trong **một** ô nhớ (`record Snapshot`), đọc/ghi qua `Volatile`. Hai field rời thì có lúc giao diện đọc được `PendingRestart` trong khi phiên bản vẫn `null`, và người dùng nhận một câu "Đã tải xong bản" cụt đuôi. `Snapshot` là kiểu **tham chiếu** vì `Volatile.Read<T>` chỉ nhận kiểu tham chiếu.

## Đóng gói

```
apps\windows\pack.cmd
```

Cờ `vpk` quan trọng, cả hai đều **ràng buộc bởi spec #1**:

- `--instLocation PerUser` — cài vào `%LocalAppData%\Snappy\`, nên bộ cài chạy `asInvoker`, nên Windows không hỏi UAC. Mặc định `Either` để người dùng chọn, và "để họ chọn" ở đây nghĩa là một phần số lần cài sẽ có UAC.
- `--packId Snappy` — quyết định luôn thư mục cài. **Bất biến từ đây**: đổi id là đổi chỗ ở của app, và bản cũ trên máy người dùng thành một bản mồ côi không ai gỡ.

Thêm `--shortcuts StartMenuRoot` (bỏ Desktop khỏi mặc định của vpk).

**`pack.ps1` dọn `releases\` trước mỗi lượt** (trừ `-KeepExisting`). Không phải sự tiện tay: `releases\` bị gitignore nên nó sống sót qua `checkout scm`, và Jenkins dùng lại workspace — giữ lại thì build thứ hai gặp đúng phiên bản mình sắp dựng đã nằm sẵn ở đó và đỏ vì lý do của build trước.

## Ký số

`vpk` mang sẵn **cả** dlib của Azure Trusted Signing lẫn `signtool` trong `vendor/signing/`, nên không phải cài thêm gì. CI chỉ cần cắm sáu secret:

| Biến môi trường | Nội dung |
|---|---|
| `AZURE_TENANT_ID`, `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET` | service principal, dlib tự đọc |
| `AZURE_CODESIGNING_ENDPOINT` | ví dụ `https://eus.codesigning.azure.net/` |
| `AZURE_CODESIGNING_ACCOUNT` | tên tài khoản Trusted Signing |
| `AZURE_CODESIGNING_PROFILE` | tên certificate profile |

`pack.ps1` tự dựng file `metadata.json` từ ba biến sau rồi xoá nó ở `finally` — CI không phải chép một file JSON vào workspace, mà một file JSON chép tay thì sớm muộn cũng có người commit nhầm nó.

Credential **chỉ** được nạp khi build chạy với tham số `PHAT_HANH` (xem [Testing](testing.md)).

## Cái bẫy đã gặp

**1. `vpk pack -v 1.0.1` trên một bản build 1.0.0 chạy êm.** Lúc nghiệm thu, app sau khi vá lên 1.0.1 ghi nhật ký `Snappy 1.0.0 đã chạy nền` trong khi Velopack ngay trên nói `v1.0.1`. `Directory.Build.props` đã có sẵn một comment cảnh báo đúng nguy cơ này — nhưng một comment cảnh báo **không phải** một hàng rào. `vpk` là một nguồn thứ hai của số phiên bản, nằm ngoài trình biên dịch. `pack.ps1` giờ đọc `ProductVersion` từ exe đã dựng và đỏ khi lệch. Ghi ở `peekvn/docs/bai-hoc.md` §162.

**2. `AppHost` là `unsafe class`, không `await` được trong đó.** Con trỏ hàm của `WndProc` bắt cả lớp phải `unsafe`, mà ngữ cảnh `unsafe` cấm `await`. Vòng kiểm khởi động bằng `ContinueWith(…, OnlyOnFaulted)` thay cho `async` lambda — fire-and-forget nhưng **không** im lặng.

**3. Hai API .NET không tồn tại dưới Windows PowerShell 5.1.** `ci/*.ps1` chạy trên .NET Framework 4.x: `Marshal.PtrToStringUTF8` và `RandomNumberGenerator.Fill` đều không có. Triệu chứng là lỗi biên dịch giữa một bước CI không liên quan gì tới chuỗi ký tự.

**4. `Expand-Archive` từ chối đuôi `.nupkg`** dù bên trong đúng là zip. Dùng `[IO.Compression.ZipFile]::ExtractToDirectory`.

## Kênh phát hành

`VelopackUpdateSource.DefaultFeedUrl` = `https://github.com/natuan1/peekvn`, đè được bằng `SNAPPY_UPDATE_FEED` — một **thư mục** cũng là kênh hợp lệ, và đó là đường nghiệm thu cập nhật mà không cần máy chủ nào.

**Hạ tầng phát hành thật chưa tồn tại** (ngoài phạm vi spec #1). Tới khi có, lượt kiểm hỏng êm và người dùng đọc "Chưa tìm được bản mới lúc này. Snappy sẽ tự thử lại sau." — không hứa suông, không bày mã lỗi, và không đổ lỗi cho mạng.
