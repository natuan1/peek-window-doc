# **MASTER PLAN: DỰ ÁN ỨNG DỤNG TIỆN ÍCH DESKTOP SNAPPY (WINDOWS v1.0)**

# **I. TỔNG QUAN DỰ ÁN & MỤC TIÊU CỐT LÕI**

* **Tên dự án:** Snappy (Desktop Utility & Local File Sharing).  
* **Mục tiêu định vị:** "Ứng dụng quốc dân" cho người dùng máy tính và văn phòng tại Việt Nam và quốc tế. Mô hình chuẩn UniKey: nhẹ dưới 20MB, xin ít quyền, không bắt tạo tài khoản, không quảng cáo, chạy nền bền bỉ nhiều năm.

## **Chỉ số đo lường thành công (KPIs)**

| Chỉ số | Mục tiêu |
| :---- | :---- |
| Tỷ lệ gỡ cài đặt sau 30 ngày | \< 25% (Chỉ số quan trọng nhất) |
| Tỷ lệ hoàn tất Onboarding | \> 80% |
| Tỷ lệ gửi thành công ít nhất 1 file (7 ngày đầu) | \> 40% |
| Dung lượng bộ cài | \< 15MB |
| Mức tiêu thụ RAM khi chạy nền | \< 25MB |

## **Mô hình thương mại**

* **Tính năng truyền file:** Miễn phí vĩnh viễn, không giới hạn dung lượng/số file, hỗ trợ gửi cả thư mục.  
* **Bản Pro:** Mua một lần trọn đời (249.000đ – 349.000đ). Bán tại các điểm ma sát (ví dụ: gợi ý nén file lớn trước khi gửi), kích hoạt không cần tài khoản.

# **II. KIẾN TRÚC KỸ THUẬT ĐÃ ĐƯỢC PHÊ DUYỆT**

## **1\. Core & UI Engine**

* **Ngôn ngữ & Runtime:** C\# .NET 9 biên dịch Native AOT kết hợp WPF / WinUI 3\.  
* **Server mạng nội bộ:** Kestrel Web Server nhúng trực tiếp trong tiến trình Core, tận dụng Windows Registered I/O (RIO) cho tốc độ streaming tối đa.

## **2\. Tương tác Mí & Shelf**

* **Vị trí:** Neo cạnh trên màn hình, mặc định lệch sang Bên Trái (chừa trống 40% ở giữa để tránh Snap Layouts của Windows 11 và cụm nút điều khiển ở góc phải).  
* **Vùng tiếp nhận:** Dải cửa sổ trong suốt 60px đăng ký IDropTarget OLE chuẩn (không dùng Global Mouse Hook để tránh rủi ro Antivirus).  
* **Shelf (Khay giữ file tạm):** Bắt cả CF\_HDROP (file thật) và trích xuất luồng FileGroupDescriptorW \+ FileContents (IStream) lưu đệm vào `%LocalAppData%\Snappy\TempDrops\` cho các file ảo từ Outlook và Web.

## **3\. Giao thức Truyền File Đa Nền Tảng ULTP (Universal Local Transfer Protocol)**

* **Discovery:** mDNS/Bonjour (\_ultp.\_tcp.local).  
* **Control:** HTTPS REST \+ WebSocket, xác thực ghép đôi SAS 6 chữ số \+ SPKI Certificate Pinning.  
* **Data:** HTTPS Range (download) và Resumable HTTP Upload draft-12 (upload).  
* **Vai trò Windows:** Đóng vai trò song song cả HTTP Client lẫn HTTP Server.

## **4\. Kiến trúc Module On-Demand**

* **Tầng 1 (Built-in Toggle):** Module nhẹ (Media GSMTC, Clipboard History) chạy tiến trình riêng hoặc worker trong Core, chỉ kích hoạt khi bật trong Settings.  
* **Tầng 2 (Pro On-Demand):** Module nặng (Nén media FFmpeg, Đổi đuôi) chỉ tải về dạng Feature Pack khi kích hoạt Pro. Giao tiếp qua Local Named Pipes IPC.

## **5\. Phân phối Web & Tự cập nhật**

* **Kênh chính:** Tải trực tiếp file .exe từ Landing Page.  
* **Ký số:** Dịch vụ Microsoft Azure Trusted Signing trong CI/CD pipeline để vượt qua Windows Defender SmartScreen.  
* **Tự cập nhật:** Velopack cài đặt trong User space (`%LocalAppData%`), hỗ trợ cập nhật vi sai (delta update), không đòi quyền Administrator.

# **III. LỘ TRÌNH TRIỂN KHAI CHI TIẾT**

## **GIAI ĐOẠN 0: NỀN MÓNG DỰ ÁN, CI/CD & HẠ TẦNG (Tuần 1 – 2\)**

* Khởi tạo Solution với các thành phần Core, Protocol.ULTP và Common.  
* Thiết lập pipeline GitHub Actions tự động biên dịch Native AOT và ký số qua Azure Trusted Signing.  
* Cấu hình đóng gói installer qua Velopack.  
* Xây dựng hạ tầng License Server xử lý thanh toán VietQR/Stripe và sinh mã dựa trên Machine ID.

## **GIAI ĐOẠN 1: GIAO DIỆN MÍ, DROP ZONE & XƯƠNG SỐNG SHELF (Tuần 3 – 5\)**

* **Cửa sổ Mí:** Tạo cửa sổ WPF/WinUI không viền, hỗ trợ Per-Monitor V2 DPI Awareness. Lập trình 4 trạng thái chuyển động (Nghỉ, Sẵn sàng, Nhắm đích, Đang chạy) với các thông số trễ và hiệu ứng tối ưu.  
* **Bộ xử lý OLE:** Triển khai IDropTarget trên dải 60px cạnh trên, nhận diện link/text và file.  
* **Shelf:** Xây dựng khay giữ file tạm, xử lý trích xuất file ảo từ Outlook/Trình duyệt và tự động dọn dẹp bộ đệm.  
* **Action Registry:** Xây dựng plugin hành động (gửi sang điện thoại, ghim, xóa, copy).

## **GIAI ĐOẠN 2: LÕI TRUYỀN FILE ULTP TRÊN WINDOWS (Tuần 6 – 8\)**

* **Discovery Plane:** Lắng nghe và đăng ký dịch vụ mDNS LAN.  
* **Control Plane:** Triển khai xác thực SAS (ECDH \+ SPKI pinning) và quản lý thiết bị tin cậy.  
* **Data Plane:** Xử lý các luồng PUSH Transfer (Resumable Upload) và PULL Transfer (Accept-Ranges bytes) qua Kestrel nhúng.  
* **Kiểm thử:** Stress-test truyền file dung lượng lớn (1GB – 20GB) đảm bảo ổn định tài nguyên hệ thống.

## **GIAI ĐOẠN 3: TÍCH HỢP MODULE v1.0 & TRẢI NGHIỆM ONBOARDING (Tuần 9 – 10\)**

* **Module M3:** Tích hợp GSMTC API (Media) và Bluetooth API (Pin tai nghe), thiết kế Dynamic Capsule.  
* **Module M4:** Xây dựng Clipboard History với bộ lọc bảo mật cho trình quản lý mật khẩu (phát hiện cờ Ignore/Exclude).  
* **Onboarding:** Thiết kế quy trình hướng dẫn tương tác 3 bước: kéo thử file, tùy chỉnh vị trí Mí và quét mã QR kết nối di động.

## **GIAI ĐOẠN 4: QA, TỐI ƯU HÓA & PHÁT HÀNH (Tuần 11 – 12\)**

* **Kiểm thử ổn định:** Test tương thích đa màn hình, DPI scale, hành vi khi chơi game/phim và truyền số lượng lớn file nhỏ.  
* **Phát hành:** Đóng gói bản chính thức, kiểm tra trạng thái SmartScreen trên môi trường sạch.  
* **Vận hành:** Hoàn thiện Landing Page và cổng cấp key tự động sau thanh toán.

# **IV. LỘ TRÌNH CÁC PHIÊN BẢN TIẾP THEO**

| Phiên bản | Tính năng trọng tâm |
| :---- | :---- |
| **Windows 1.1** | Module M5: Chỉnh độ sáng màn hình qua DDC/CI (Pro hỗ trợ đa màn hình). |
| **Windows 1.2** | Cầu nối chuyển văn bản và liên kết nhanh Desktop-Mobile qua ULTP. |
| **Windows 1.3** | Module M6: Nén ảnh và video (Gói Pro On-Demand, tích hợp FFmpeg, libwebp). |
| **Windows 1.4** | Module M7: Chuyển đổi định dạng tệp (HEIC sang JPG, PNG sang WebP). |
| **macOS 1.0** | Bản client nhận file mỏng, đồng bộ hệ sinh thái với bản Windows. |

&nbsp;