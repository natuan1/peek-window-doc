# **KẾ HOẠCH TRIỂN KHAI CHI TIẾT TỪNG GIAI ĐOẠN (DETAILED IMPLEMENTATION PLAN)**

# **DỰ ÁN TIỆN ÍCH DESKTOP SNAPPY (WINDOWS v1.0)**

## **GIAI ĐOẠN 0: NỀN MÓNG KỸ THUẬT, CI/CD, KÝ SỐ & BẢN QUYỀN (TUẦN 1 – 2\)**

### **0.1. Thiết lập Cấu trúc Solution C\# .NET 9 Native AOT**

* **Đầu ra kỹ thuật**: Solution `Snappy.sln` tối ưu hóa Native AOT.  
* **Chi tiết triển khai**:  
  * `Snappy.Core`: Khởi tạo ứng dụng WPF/WinUI 3 không phụ thuộc runtime bên ngoài, cấu hình Per-Monitor V2 DPI Awareness, quản lý vòng đời ứng dụng và biểu tượng khay hệ thống (System Tray).  
  * `Snappy.Protocol`: Module mạng độc lập, tích hợp Kestrel Web Server nhúng (HTTP/1.1, HTTP/2, TLS 1.3, WebSocket) và mDNS client/server.  
  * `Snappy.Interop`: Các interface P/Invoke và COM Win32 bậc thấp (`IDropTarget`, `IDataObject`, `IStream`, `SHAppBarMessage`, `GSMTC`, Windows Bluetooth APIs).  
  * `Snappy.Shared`: DTOs, Data Contracts, IPC Models, File utilities.  
  * Cấu hình file `.csproj`: Bật `<PublishAot>true</PublishAot>`, `<StripSymbols>true</StripSymbols>`, `<OptimizationPreference>Size</OptimizationPreference>`, cấu hình file `rd.xml` cho các thư viện COM interop.  
* **Tiêu chí nghiệm thu (DoD)**: Lệnh `dotnet publish` sinh ra 1 tệp `.exe` duy nhất dung lượng \< 18MB, chạy được trên máy ảo Windows 10/11 sạch không cài đặt sẵn .NET SDK.

### **0.2. Pipeline Tự động Build & Ký số Azure Trusted Signing (GitHub Actions)**

* **Đầu ra kỹ thuật**: Workflow `.github/workflows/build-and-sign.yml`.  
* **Chi tiết triển khai**:  
  * Khởi tạo workflow chạy trên runner `windows-latest`.  
  * Thiết lập bước biên dịch Native AOT.  
  * Tích hợp action `azure/trusted-signing-action` để ký số tệp `.exe` bằng chứng chỉ Azure Trusted Signing của dự án.  
  * Kiểm tra xác thực chữ ký số bằng công cụ `signtool.exe verify /pa /v`.  
* **Tiêu chí nghiệm thu (DoD)**: Tệp `.exe` sau khi build trên GitHub Actions có chữ ký số hợp lệ được bảo chứng bởi Microsoft.

### **0.3. Tích hợp Đóng gói & Tự cập nhật Vi sai (Velopack)**

* **Đầu ra kỹ thuật**: Bộ cài đặt tự động và cơ chế cập nhật ngầm.  
* **Chi tiết triển khai**:  
  * Cài đặt thư viện Velopack vào `Snappy.Core`.  
  * Khởi tạo `VelopackApp.Build().Run();` ngay tại entry point `Program.cs`.  
  * Cấu hình đóng gói installer vào User space (`%LocalAppData%\Snappy\`), tuyệt đối không đòi quyền Administrator (UAC).  
  * Viết Background Service kiểm tra cập nhật ngầm mỗi 4 giờ từ release server, tự động tải gói cập nhật vi sai (delta package) và áp dụng khi người dùng khởi động lại ứng dụng.  
* **Tiêu chí nghiệm thu (DoD)**: Cài đặt ứng dụng êm ái không hiện UAC popup; cập nhật từ bản 1.0.0 lên 1.0.1 chỉ tải gói vá vài MB và tự cập nhật ngầm thành công.

### **0.4. Máy chủ Cấp Bản quyền & Kích hoạt Pro qua Machine ID**

* **Đầu ra kỹ thuật**: Serverless API (Cloudflare Worker) và Client License Service.  
* **Chi tiết triển khai**:  
  * Phía Client: Thuật toán sinh Machine ID duy nhất từ thông tin phần cứng máy tính (Motherboard UUID \+ CPU hash), hoàn toàn ẩn danh, không thu thập PII.  
  * Phía Server: Webhook nhận thông báo thanh toán thành công (VietQR / Stripe), sinh mã License Key định dạng `SNPY-XXXX-XXXX-XXXX-XXXX` có chữ ký điện tử Ed25519.  
  * Endpoint `/v1/license/activate`: Kiểm tra key, lưu ánh xạ với Machine ID (cho phép kích hoạt tối đa 3 thiết bị/key), trả về Signed License Token.  
  * Phía Client: Lưu token bản quyền vào Windows Credential Manager (`CredWriteW`).  
* **Tiêu chí nghiệm thu (DoD)**: Kích hoạt Pro thành công không cần tài khoản, khởi động lại app vẫn giữ trạng thái Pro, hoạt động ngoại tuyến sau khi kích hoạt.

## **GIAI ĐOẠN 1: GIAO DIỆN MÍ, DROP ZONE & XƯƠNG SỐNG SHELF (TUẦN 3 – 5\)**

### **1.1. Cửa sổ Mí Cạnh Trên & Quản lý Đa Màn hình**

* **Đầu ra kỹ thuật**: Cửa sổ neo màn hình WPF/WinUI 3 tùy biến.  
* **Chi tiết triển khai**:  
  * Thiết lập thuộc tính Win32: `WS_POPUP`, `WS_EX_TOOLWINDOW` (không hiện trên Taskbar/Alt+Tab), `WS_EX_TOPMOST`.  
  * Tọa độ: Neo ở cạnh trên màn hình, chiều rộng chiếm 30% màn hình, mặc định lệch sang bên trái (bắt đầu từ 5% chiều rộng màn hình, chừa trống 40% khoảng giữa màn hình để tránh Snap Layouts).  
  * Lắng nghe `WM_DISPLAYCHANGE`: Tự động nhận diện khi người dùng cắm/rút màn hình phụ và dịch chuyển Mí sang màn hình có con trỏ chuột.  
  * Bộ phát hiện Fullscreen: Lắng nghe `SHAppBarMessage` và `GetForegroundWindow`; khi người dùng bật app toàn màn hình hoặc chơi game, Mí tự động ẩn 100%.  
* **Tiêu chí nghiệm thu (DoD)**: Mí hiển thị chuẩn xác trên các màn hình có tỉ lệ scale khác nhau (100%, 125%, 150%, 200%), không che khuất nút tắt cửa sổ, tự ẩn khi mở game full screen.

### **1.2. Vùng Tiếp nhận Drop Zone 60px OLE & Chuyển đổi Trạng thái**

* **Đầu ra kỹ thuật**: Lớp tiếp nhận kéo thả chuẩn OLE.  
* **Chi tiết triển khai**:  
  * Tạo cửa sổ trong suốt cao 60px neo cạnh trên, đăng ký `RegisterDragDrop` với `IDropTarget`.  
  * Quản lý máy trạng thái (State Machine):  
    * Trạng thái 0 (Nghỉ): Ẩn hoặc vệt 3px mờ.  
    * Trạng thái 2 (Sẵn sàng nhận): Khi chuột kéo dữ liệu vào vùng trúng và giữ \> 80ms, mở rộng Mí thành khay cao 90px (hiệu ứng Cubic ease-out 200ms).  
    * Trạng thái 3 (Nhắm đích): Khi hover trên một đích cụ thể (Shelf hoặc Avatar thiết bị).  
    * Trạng thái 4 (Đang chạy): Co lại thành viên nhỏ (Pill) hiển thị thanh tiến trình.  
  * Trễ rời vùng: Áp dụng trễ 250ms khi con trỏ rời khỏi vùng để chống rung tay.  
* **Tiêu chí nghiệm thu (DoD)**: Kéo file vào cạnh trên kích hoạt Mí nở xuống mượt mà, di chuột lướt nhanh qua không bị kích hoạt nhầm, animation dưới 200ms không giật lag.

### **1.3. Bộ Trích xuất File Thật & File Ảo từ Outlook / Trình Duyệt (Xương sống Shelf)**

* **Đầu ra kỹ thuật**: Lớp xử lý dữ liệu OLE đa định dạng `SnappyDataObjectReader`.  
* **Chi tiết triển khai**:  
  * Kiểm tra `CF_HDROP`: Đọc danh sách file vật lý trên đĩa thông qua `DragQueryFileW`.  
  * Kiểm tra `FileGroupDescriptorW` & `FileContents`: Khi kéo file đính kèm từ Outlook Desktop hoặc ảnh từ trình duyệt web, trích xuất cấu trúc `FILEDESCRIPTORW`, đọc dữ liệu qua COM `IStream`, ghi luồng đệm vào `%LocalAppData%\Snappy\TempDrops\{Guid}\`.  
  * Trích xuất icon và metadata (tên file, dung lượng, MIME type) bằng `SHGetFileInfoW`.  
  * Bộ dọn dẹp (Cleanup Manager): Tự động xóa các file tạm trong thư mục `TempDrops` khi đóng ứng dụng hoặc sau khi file đã được gửi thành công.  
* **Tiêu chí nghiệm thu (DoD)**: Kéo file từ File Explorer, file đính kèm từ Outlook, và ảnh từ Chrome/Edge vào Shelf đều được nhận diện 100% với tên và dung lượng chính xác.

### **1.4. Khay Shelf & Action Plugin Registry**

* **Đầu ra kỹ thuật**: Khay giữ file tạm và khung cắm plugin hành động.  
* **Chi tiết triển khai**:  
  * Giao diện Shelf: Hiển thị danh sách thẻ file ngang, nút xóa, nút mở thư mục chứa. Bản Free hỗ trợ 1 ngăn; cấu trúc code sẵn sàng mở rộng nhiều ngăn cho bản Pro.  
  * Interface `IShelfActionPlugin`: Định nghĩa các hàm `GetManifest()`, `CanHandle(files)`, `ExecuteActionAsync(files)`.  
  * Local Named Pipe Host: Lắng nghe kết nối từ các module bên ngoài để nhận diện hành động mới.  
* **Tiêu chí nghiệm thu (DoD)**: Người dùng có thể kéo nhiều file vào Shelf để gom lại, sau đó kéo cả nhóm ra ngoài Explorer hoặc chọn gửi sang thiết bị khác.

## **GIAI ĐOẠN 2: LÕI TRUYỀN FILE ULTP TRÊN WINDOWS (TUẦN 6 – 8\)**

### **2.1. Discovery Plane (mDNS / DNS-SD)**

* **Đầu ra kỹ thuật**: Bộ tự động tìm kiếm thiết bị nội bộ `SnappyDiscoveryService`.  
* **Chi tiết triển khai**:  
  * Đăng ký dịch vụ `_ultp._tcp.local` với TXT record nhỏ gọn: `pv=1`, `id=<deviceId>`, `port=<port>`.  
  * Lắng nghe broadcast trong mạng LAN, duy trì bảng `PeerTable` (DeviceId, IP, Port, Platform, LastSeen).  
  * Xử lý timeout: Tự động đánh dấu thiết bị offline nếu không nhận được tín hiệu sau 15 giây.  
* **Tiêu chí nghiệm thu (DoD)**: Máy tính Windows tự động nhìn thấy iPhone/Android ngay khi mở app trên điện thoại cùng mạng Wi-Fi, không cần cấu hình IP.

### **2.2. Control Plane & Quy trình Ghép đôi SAS Bảo mật**

* **Đầu ra kỹ thuật**: Kênh điều khiển HTTPS REST \+ WebSocket và xác thực SAS.  
* **Chi tiết triển khai**:  
  * Định danh thiết bị: Sinh cặp khóa P-256 ECDSA khi cài đặt, lưu khóa bí mật an toàn vào Windows CNG.  
  * Các API Kestrel Controller:  
    * `GET /v1/info`: Cung cấp thông tin phiên bản giao thức và capabilities.  
    * `POST /v1/pairings`: Trao đổi ephemeral ECDH key, tính toán shared secret và sinh mã bảo mật SAS 6 chữ số (`HMAC-SHA256`).  
    * `POST /v1/pairings/{id}/confirm`: Nhận xác nhận mã số từ thiết bị kia, lưu mã băm SPKI vào cơ sở dữ liệu `TrustedDevices`.  
    * `POST /v1/transfers`: Tạo phiên chuyển file (Transfer Offer) kèm danh sách manifest.  
  * WebSocket `/v1/events`: Duy trì kết nối hai chiều để nhận thông báo tức thì khi có thiết bị gửi file tới.  
* **Tiêu chí nghiệm thu (DoD)**: Ghép đôi lần đầu hiển thị mã 6 số trùng khớp trên cả 2 màn hình; xác nhận xong tự động nhớ thiết bị tin cậy cho các lần gửi sau.

### **2.3. Data Plane (Streaming Kestrel Server & Resumable Upload)**

* **Đầu ra kỹ thuật**: Bộ truyền nhận dữ liệu băng thông cao `SnappyDataPlane`.  
* **Chi tiết triển khai**:  
  * PULL Server (Windows phục vụ cho iPhone tải về):  
    * Endpoint `GET /v1/transfers/{id}/items/{itemId}` hỗ trợ header `Range: bytes=start-end`.  
    * Trả về mã HTTP `206 Partial Content`, stream trực tiếp từ đĩa với buffer 64KB, không nạp toàn bộ file vào RAM.  
  * PUSH Server (Windows nhận file từ iPhone/Android):  
    * Hỗ trợ giao thức `draft-ietf-httpbis-resumable-upload-12` (Creation, Offset, Incomplete).  
    * Hỗ trợ Single HTTP upload stream trực tiếp ra thư mục `%UserProfile%\Downloads\Snappy\`.  
  * Phòng chống tấn công Path Traversal: Chuẩn hóa đường dẫn UTF-8, chặn toàn bộ ký tự `../`, `..\` hoặc đường dẫn tuyệt đối; tự động đổi tên khi trùng file (`photo (1).jpg`).  
* **Tiêu chí nghiệm thu (DoD)**: Truyền thành công file video dung lượng 10GB giữa máy tính và điện thoại qua Wi-Fi với tốc độ tối đa của router; ngắt kết nối giữa chừng có thể tiếp tục truyền tải mà không phải gửi lại từ đầu.

## **GIAI ĐOẠN 3: TÍCH HỢP MODULE v1.0 & TRẢI NGHIỆM ONBOARDING (TUẦN 9 – 10\)**

### **3.1. Module M3: Media Controller & Pin Bluetooth (Dynamic Capsule)**

* **Đầu ra kỹ thuật**: Tiện ích phát nhạc và đọc pin trên Mí.  
* **Chi tiết triển khai**:  
  * Tích hợp `GlobalSystemMediaTransportControlsSessionManager` (GSMTC): Lắng nghe bài hát, nghệ sĩ, bìa album và điều khiển Play/Pause/Next của Spotify/Chrome/Windows Media.  
  * Tích hợp `Windows.Devices.Bluetooth`: Quét pin tai nghe Bluetooth qua GATT Battery Service (`0x180F`); nếu không đọc được, hiển thị dấu `-`, tuyệt đối không hiện `0%`.  
  * Hiển thị Dynamic Capsule trên Mí: Khi phát nhạc, Mí hiện viên capsule nhỏ hiển thị thông tin bài hát; khi có thao tác kéo file vào Mí, viên capsule tự động trượt sang một bên nhường chỗ cho vùng thả.  
* **Tiêu chí nghiệm thu (DoD)**: Điều khiển media mượt mà, đọc đúng % pin tai nghe Bluetooth, không bị xung đột giao diện khi kéo thả file.

### **3.2. Module M4: Clipboard History Bảo Mật**

* **Đầu ra kỹ thuật**: Quản lý lịch sử bảng tạm bảo vệ mật khẩu.  
* **Chi tiết triển khai**:  
  * Đăng ký `AddClipboardFormatListener` trong Background Worker của Core.  
  * Lưu trữ 5 mục gần nhất cho bản Free và không giới hạn cho bản Pro.  
  * Bộ lọc bảo mật nhạy cảm: Tự động kiểm tra và bỏ qua dữ liệu copy có gắn các format `Clipboard Viewer Ignore`, `ExcludeClipboardContentFromMonitor`, `CanIncludeInClipboardHistory` (từ 1Password, Bitwarden, KeePass).  
  * Phím tắt mở bảng dán nhanh (`Win + Shift + V` hoặc tùy biến), nút tạm dừng theo dõi và nút xóa sạch lịch sử.  
* **Tiêu chí nghiệm thu (DoD)**: Lịch sử lưu trữ tức thì các văn bản copy; copy mật khẩu từ 1Password/Bitwarden hoàn toàn không bị lưu lại vào lịch sử.

### **3.3. Quy trình Interactive Onboarding 3 Bước**

* **Đầu ra kỹ thuật**: Trải nghiệm hướng dẫn người dùng trong 60 giây đầu tiên.  
* **Chi tiết triển khai**:  
  * Bước 1: Màn hình hướng dẫn thực hành kéo thử một file mẫu có sẵn vào góc trên bên trái màn hình để xem Mí nở ra.  
  * Bước 2: Tùy chỉnh vị trí Mí (Trái/Phải) và thiết lập phím tắt mở panel.  
  * Bước 3: Cung cấp mã QR để tải ứng dụng di động (iOS/Android) từ website.  
* **Tiêu chí nghiệm thu (DoD)**: Đạt tỷ lệ \> 80% người dùng hoàn tất bài hướng dẫn kéo thả trong lần đầu cài đặt.

## **GIAI ĐOẠN 4: KIỂM THỬ ĐỘ BỀN, TỐI ƯU HÓA & PHÁT HÀNH (TUẦN 11 – 12\)**

### **4.1. Kiểm thử Độ Ổn Định & Chống Crash (Zero-Crash Testing)**

* **Đầu ra kỹ thuật**: Báo cáo kiểm thử chất lượng toàn diện.  
* **Chi tiết triển khai**:  
  * Multi-Monitor Testing: Kiểm tra cắm/rút màn hình 4K, xoay màn hình dọc, thay đổi DPI từ 100% đến 200%.  
  * Fullscreen Games: Kiểm tra hơn 10 tựa game phổ biến (fullscreen exclusive và borderless) để đảm bảo Mí không bao giờ hiện đè lên game.  
  * Stress Testing: Truyền thư mục chứa 5.000 file nhỏ liên tục trong 24 giờ.  
  * Memory Profiling: Đo đạc bằng JetBrains dotMemory, đảm bảo mức tiêu thụ RAM chạy nền duy trì ổn định dưới 25MB, không bị rò rỉ bộ nhớ (memory leaks).  
* **Tiêu chí nghiệm thu (DoD)**: Ứng dụng chạy liên tục 72 giờ không có crash, mức tiêu thụ CPU khi nghỉ \= 0%, RAM \< 25MB.

### **4.2. Xuất bản Bộ Cài & Ký số Release**

* **Đầu ra kỹ thuật**: Bộ cài đặt `.exe` chính thức hoàn chỉnh.  
* **Chi tiết triển khai**:  
  * Biên dịch bản Release Native AOT, đóng gói qua Velopack.  
  * Ký số Azure Trusted Signing trên toàn bộ gói nhị phân.  
  * Kiểm thử cài đặt trên các máy tính Windows 10 và Windows 11 mới cài lại win, kiểm tra SmartScreen không cảnh báo nguy hiểm.  
* **Tiêu chí nghiệm thu (DoD)**: Bộ cài dung lượng \< 15MB, cài đặt trong 3 giây, mở lên chạy ngay không lỗi.

### **4.3. Landing Page & Vận hành Cổng Thanh toán VietQR**

* **Đầu ra kỹ thuật**: Trang web phân phối và hệ thống bán hàng tự động.  
* **Chi tiết triển khai**:  
  * Landing page tối ưu SEO, tự động nhận diện hệ điều hành để hiện nút tải Windows.  
  * Đoạn video ngắn demo tính năng kéo file bay sang điện thoại trong 3 giây.  
  * Tích hợp cổng thanh toán tạo mã VietQR động: Khách quét mã chuyển khoản \-\> Webhook nhận tiền \-\> Cấp License Key qua Email sau 30 giây.  
* **Tiêu chí nghiệm thu (DoD)**: Quy trình mua và nhận key tự động 100%, tỷ lệ kích hoạt key thành công đạt 100%.

&nbsp;