# Đóng gói & tự cập nhật — quy trình

## Người dùng đi qua những gì

### Cài lần đầu

1. Tải `Snappy-win-Setup.exe` (10,02 MB).
2. Bấm đúp. **Không có hộp thoại UAC**, không có câu hỏi "cài ở đâu".
3. Vài giây sau: app nằm ở `%LocalAppData%\Snappy\`, có shortcut trong Start Menu.
4. Mở Snappy → icon ở khay hệ thống. (Windows 11 giấu icon mới vào phần tràn sau dấu `^` — xem [bài học 2026-09-15](../../lessons-learned.md); dạy người dùng ghim ra là việc của Ticket 17.)

**Không có Desktop shortcut**, có chủ ý: app chạy nền dưới khay, một icon trên màn hình chỉ là rác. Start Menu là đường vào đủ, và là chỗ gõ "Snappy" tìm ra được.

### Có bản mới

Người dùng **không làm gì cả**. Trình tự họ thấy:

1. 30 giây sau khi mở app, rồi mỗi 4 giờ: app hỏi kênh phát hành ở nền. Không có dấu hiệu nào ra ngoài.
2. Có bản mới → app tải gói vá xuống, vẫn ở nền.
3. Xong: **tooltip khay đổi** thành "Snappy — có bản 1.0.1, khởi động lại để dùng", bảng trạng thái hiện thêm một dòng sáng lên, và menu chuột phải **mọc thêm** mục "Khởi động lại để cập nhật".
4. Từ đây hai đường, cùng đích:
   - Bấm "Khởi động lại để cập nhật" → app đóng, vá, mở lại ngay.
   - Không bấm gì, cứ dùng tiếp rồi thoát như mọi khi → bản vá áp ngay sau lúc thoát, lần mở sau đã là bản mới.

Mục menu ấy **chỉ tồn tại khi có bản vá chờ sẵn**. Bày sẵn một mục mà bấm vào không xảy ra gì là cách nhanh nhất làm người dùng hết tin vào menu.

### Không có mạng / kênh phát hành im lặng

Bảng trạng thái nói: *"Chưa tìm được bản mới lúc này. Snappy sẽ tự thử lại sau."*

Câu này cố ý **không** đổ lỗi cho mạng. Kênh phát hành có thể im vì chục lý do khác — chưa có bản phát hành nào là một — và một câu đổ lỗi sai sẽ gửi người dùng đi kiểm tra Wi-Fi cho một việc không phải lỗi của Wi-Fi.

### Gỡ app

`Update.exe --uninstall` (hoặc qua Apps & features) xoá **toàn bộ** `%LocalAppData%\Snappy` — gồm cả nhật ký và TempDrops. Không sót file nào. Đây là hệ quả cố ý của việc gốc cài trùng gốc dữ liệu ([ADR-0006 §2](../../adr/0006-dong-goi-velopack-cai-peruser.md)).

## Hệ thống đi qua những gì

### Lúc khởi động

```
Main()
 └─ VelopackApp.Build().SetLogger(…).OnFirstRun(…).OnRestarted(…).Run()
      ├─ là lượt hook (--veloapp-install / --veloapp-updated / --veloapp-uninstall)?
      │    → chạy hook rồi thoát hẳn. Nhật ký đi qua OutputDebugString.
      └─ không → trả về, chạy tiếp
 └─ SnappyPaths.EnsureCreated() + AppLog.Initialize()
 └─ SingleInstance.TryAcquire()
 └─ AppHost.Run() → StartUpdateChecks()
```

**Thứ tự này là một ràng buộc, không phải sở thích.** Velopack phải đi trước khoá một-bản-đang-chạy (lúc bộ cài gọi lại exe, bản cũ có thể vẫn giữ khoá) *và* trước việc dựng thư mục dữ liệu (ở lượt gỡ, dựng `logs\` là tạo lại một thư mục ngay trong cây mà `Update.exe` đang xoá).

### Vòng kiểm cập nhật

```
RunAsync(ct)                               ← thread pool, không phải thread thông điệp
 └─ chờ 30 giây
 └─ lặp:
      CheckOnceAsync
        ├─ không phải bản cài  → NotInstalled, thoát vòng lặp
        ├─ hỏi hỏng            → CheckFailed, chờ 4 giờ rồi thử lại
        ├─ không có bản mới    → UpToDate,    chờ 4 giờ rồi thử lại
        └─ có bản mới          → Downloading → PendingRestart, **thoát vòng lặp**
      mỗi lần đổi trạng thái → StatusChanged → PostMessageW tới cửa sổ host
```

Ba chi tiết có lý do:

- **Lượt đầu chờ 30 giây**, không chạy ngay: mấy giây đầu là lúc người dùng vừa đăng nhập, đĩa và mạng đang bị hàng chục thứ khác giành.
- **Có bản vá là dừng hẳn**: bản vá đã nằm trên đĩa, không còn gì thay đổi được cho tới khi người dùng mở lại app. Kiểm tiếp chỉ tốn pin và băng thông.
- **`PostMessageW` chứ không gọi thẳng**: vòng kiểm chạy trên thread pool, mọi thứ Win32 chỉ đụng được từ thread thông điệp.

### Đường thoát

```
RequestExit()
 └─ _lifecycle.BeginShutdown()      ← trả true đúng một lần trong đời tiến trình
 └─ _stopUpdates.Cancel()
 └─ _updates.ApplyOnExitIfPending() ← hẹn Velopack vá ngay sau lúc thoát
 └─ gỡ icon khay → dọn cửa sổ → PostQuitMessage
```

Hẹn vá **trước** khi dọn. Đây là nửa còn lại của lời hứa "áp dụng khi khởi động lại app".

Nhánh này hỏng thì lời hứa **không** vỡ: gói vá vẫn nằm trong `packages\` và Velopack tự áp ở lần khởi động sau. Hẹn ở đây chỉ là đường nhanh hơn — nên một ngoại lệ ở đó tuyệt đối không được chặn đường thoát của app.

## Edge case

| Tình huống | Hành vi |
|---|---|
| Chạy từ thư mục `publish\`, chưa qua bộ cài | `IsInstalled = false` → trạng thái `NotInstalled`, **không hỏi mạng lần nào**. Mỗi lần dev chạy thử không còn ăn một cục lỗi không liên quan. |
| Người dùng bấm Thoát giữa hai lượt kiểm | `OperationCanceledException` bị nuốt **trong** `RunAsync` và ghi nhật ký. Để nó lọt ra là để tiến trình chết ngay lúc đang thoát — người dùng thấy "Snappy đã ngừng hoạt động" cho việc họ vừa yêu cầu. |
| Tải gói vá hỏng giữa chừng | `CheckFailed`, thử lại ở lượt sau. Không có bản vá nửa vời nào được hẹn áp. |
| Bấm "Khởi động lại để cập nhật" mà áp không được | Icon khay được thêm lại (và ghi nhật ký nếu thêm lại cũng hỏng), bảng trạng thái mở ra nói "Chưa cập nhật được lúc này. Snappy sẽ tự thử lại sau." |
| Bản Snappy thứ hai chạy giữa lúc đang vá | Thông điệp đánh thức đăng ký **theo tên**, nên hai tiến trình khác phiên bản vẫn gọi được nhau. |

## Lượt nghiệm thu thật (2026-09-16, máy dev Windows 11)

Đi bằng đúng cử chỉ người dùng đi, không đường tắt:

| Bước | Kết quả |
|---|---|
| `Snappy-win-Setup.exe --silent` | Mã thoát 0, **không popup UAC**. `%LocalAppData%\Snappy\` có `current\`, `packages\`, `Snappy.exe`, `Update.exe` |
| Mở app, đợi | 22:19:38 khởi động → **22:20:08** (đúng 30 giây) lượt kiểm đầu chạy |
| Thấy 1.0.1 trên kênh phát hành cục bộ | `Found newer remote release available (1.0.0 -> 1.0.1)` |
| Tải | `Downloading delta 1.0.1` → `Applying 1 patches` → `Delta update download complete` — **đi đúng đường gói vá**, không tải lại gói đầy đủ |
| Thoát bằng `WM_CLOSE` | Hẹn áp, thoát sạch |
| Mở lại | `sq.version` ghi **1.0.1**, hook `--veloapp-updated` chạy qua, lượt kiểm sau nói "Đang là bản mới nhất" |
| `Update.exe --uninstall --silent` | `%LocalAppData%\Snappy` biến mất hoàn toàn, 0 tiến trình sót |

Lượt này chứng minh **đường** delta được chọn và áp được. Nó **không** chứng minh cỡ gói vá — hai bản dùng chung một bộ nhị phân nên chẳng có gì để vá. Cỡ gói vá là việc của `ci/check-delta.ps1` (xem [Testing](testing.md)).
