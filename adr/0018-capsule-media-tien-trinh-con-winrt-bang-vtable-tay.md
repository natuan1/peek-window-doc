# ADR: Capsule media chạy trong một tiến trình con thường trực, gọi WinRT bằng vtable tay — không projection CsWinRT trong exe

Date: 2026-10-03
Status: Accepted

## Context

Ticket 15 ([natuan1/peekvn#146](https://github.com/natuan1/peekvn/issues/146)) đòi một capsule media trên Mí: bài đang phát, bìa album, play/pause/next qua GSMTC (`Windows.Media.Control`), và % pin tai nghe qua GATT Battery Service `0x180F`.

Ba ràng buộc đã có:

- **RAM.** Tiến trình nền đã sát red line 30 MB tổng sau Ticket 13 ([ADR-0016](0016-kpi-ram-nen-do-bang-private-working-set.md)). GSMTC nạp `Windows.Media` và tầng RPC tới dịch vụ media của shell. Bìa album phải đi qua WIC. Một DLL đã nạp thì không bao giờ nhả ([ADR-0017](0017-tinh-nang-nang-phan-shell-chay-trong-tien-trinh-con.md)).
- **Không biết lúc nào nhạc bắt đầu.** Khay thẻ Shelf sống theo lượt mở. Capsule thì phải nghe GSMTC suốt.
- **Projection có sẵn.** TFM `net10.0-windows10.0.22000.0` đã kèm projection CsWinRT, nên `using Windows.Media.Control` biên dịch ngay.

Bản đầu dùng thẳng projection ấy và đặt toàn bộ capsule trong tiến trình con. 📐 Đo ngày 03/10/2026 trên bản AOT, ngoài gói MSIX, xen kẽ với `main` publish trong worktree riêng:

| | `main` | Bản projection CsWinRT | Bản vtable tay |
|---|---|---|---|
| exe | 10 042 KB | 11 803 KB | **10 159 KB** |
| Tiến trình nền lúc nghỉ, private | 4,82 MB | 5,92 MB | **5,05 MB** |
| Tiến trình nền lúc nghỉ, tổng | 20,8 MB | 23,4 MB | **21,9 MB** |
| DLL mới ở tiến trình nền | — | `oleaut32` | **không có** |
| Tiến trình con khi đang phát, private / tổng | — | 8,1 / 34,9 MB | **5,2 / 27,8 MB** |

Tiến trình nền của bản projection **không bao giờ gọi WinRT**, vậy mà vẫn trả +1,1 MB private và +2,6 MB tổng. Bản thăm dò không khởi động con vẫn nạp `oleaut32`. Nguyên nhân: `WinRT.Runtime.dll` và `Microsoft.Windows.SDK.NET.dll` mang `[ModuleInitializer]` (`ProjectionInitializer::InitalizeProjection`). Dưới Native AOT, mọi module initializer của assembly có mặt trong ảnh đều chạy lúc khởi động. Tắt `CsWinRTAotExportsEnabled` không đổi gì.

Cùng bản ấy còn ném `InvalidCastException: Failed to create a CCW … IIterable<string>` khi truyền một `List<string>` (hay collection expression) **vào** WinRT từ thư viện `Snappy.Interop`. Bộ sinh mã AOT của CsWinRT không sinh vtable cho trường hợp đó.

## Decision

1. **Capsule sống trọn trong `Snappy.exe --media-capsule`**, một tiến trình con thường trực. Cha khởi động nó cùng app và giết nó lúc thoát. Con chết bất thường thì cha dựng lại sau 5 s, tối đa 3 lần mỗi lượt chạy. Cửa sổ, GSMTC, WIC và GATT đều thuộc về con.
2. **Cha gửi đúng một thứ: khung Mí đang chiếm.** Đó là `MiWindow.FootprintChanged`, gửi qua stdin dưới dạng dòng `footprint\tL\tT\tR\tB` hoặc `footprint\t-`. Con chỉ gửi về nhật ký, cùng dạng dòng `log` của khay thẻ. Mí không biết capsule tồn tại.
3. **Không projection CsWinRT nào trong exe.** WinRT được gọi bằng vtable tay (`Snappy.Interop/WinRt.cs`): `HSTRING`, `RoGetActivationFactory`, chờ `IAsyncOperation` bằng cách hỏi `IAsyncInfo.Status` trên một luồng MTA riêng, và delegate sự kiện là một đối tượng COM bốn slot dựng tay. IID tham số hoá và slot tra từ `winrt/*.h` của Windows SDK 10.0.26100. Bìa album đi `CreateStreamOverRandomAccessStream` → WIC, cũng bằng vtable.
4. **Pin tai nghe đi Win32, không WinRT.** `SetupDiGetClassDevs` liệt kê lớp giao diện `0000180F-…`, rồi `BluetoothGATTGetCharacteristics` / `GetCharacteristicValue` đọc `0x2A19`. "Tai nghe" là thiết bị mà device container có `System.Devices.CategoryIds` thuộc nhóm `Audio`, đọc bằng `DevGetObjectProperties`.

## Consequences

- Tiến trình nền gần như không đổi so với `main`: không DLL mới, +0,23 MB private. Phần +1,1 MB tổng là đường sinh tiến trình con (`Process`, hai luồng đọc ống).
- **Người dùng có thêm một tiến trình Snappy thường trực**, ~4,3 MB private lúc nghỉ và ~5,3 MB khi đang phát. Cột "Memory" của Task Manager gộp hai tiến trình dưới cùng một app, khoảng 10–11 MB private. KPI và red line của ADR-0016 vẫn chỉ đo tiến trình nền. Có giữ nguyên cách đo đó với một con thường trực hay không là câu hỏi cho chủ dự án.
- Bộ cài 11,01 → 11,06 MB.
- Mỗi API WinRT mới phải tra slot và IID từ header SDK, không ghi từ trí nhớ (ADR-0002). Đổi lại, bề mặt WinRT của app đếm được bằng một lần đọc `WinRt.cs` và `MediaSessions.cs`.
- **Đừng thêm `using Windows.*` vào bất kỳ project nào sinh ra `Snappy.exe`.** Riêng việc projection có mặt trong ảnh exe đã làm tiến trình nền tốn RAM, kể cả khi không dòng nào gọi tới nó.
- Chưa đo được với tai nghe BLE thật: máy dev không có Bluetooth adapter. Tai nghe Bluetooth Classic (A2DP/HFP) không có GATT, nên capsule không có ô pin cho chúng.

## Related

- [ADR-0017](0017-tinh-nang-nang-phan-shell-chay-trong-tien-trinh-con.md): tính năng nặng chạy trong tiến trình con, lớp `ChildProcess`.
- [ADR-0016](0016-kpi-ram-nen-do-bang-private-working-set.md): thước RAM.
- [ADR-0014](0014-mi-ve-bang-layered-window-khong-composition.md): layered window. Capsule vẽ cùng cách, bỏ Composition vì cùng loại lý do.
- [ADR-0002](0002-ui-stack-aot-spike-pass.md): IID phải tra header; gọi IUnknown qua vtable tay dưới AOT.
- [Capsule media trên Windows](../features/capsule-media-windows/overview.md).
- `peekvn/docs/bai-hoc.md` §209–§211.
