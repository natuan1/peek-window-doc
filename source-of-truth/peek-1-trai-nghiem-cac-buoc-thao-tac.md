# **peek 1.0: Spec trải nghiệm và các bước thao tác**

Nov 13, 2026 · @Hưng Khúc

## **Tổng quan**

Spec này mô tả thao tác của peek 1.0 trên Windows, macOS, Android và iPhone. Người dùng mở ra là biết phải làm gì, cái gì rườm rà thì bỏ.

**Phạm vi**

* **Windows 1.0:** gửi và nhận file, Mí, Khay. Lịch sử clipboard, nhạc và pin tai nghe có trong app nhưng nằm ngoài spec này.  
* **macOS 1.0:** gửi và nhận file, Mí quanh notch. Người dùng Mac hay gửi sang máy Windows nên phải có chiều gửi ngay từ đầu.  
* **Android và iPhone 1.0:** gửi và nhận file với máy tính.  
* **Chưa làm ở 1.0:** laptop tự phát Wi-Fi, Bluetooth "gõ cửa" iPhone, gửi qua internet.

**Năm nguyên tắc**

1. **Chỉ có hai nhóm máy:** Máy của tôi và Máy xung quanh.  
2. **Máy của tôi không bao giờ hỏi, máy xung quanh luôn hỏi.** Có hộp hỏi "Nhận" là biết chắc không phải máy của mình.  
3. **Gửi theo cách quen thuộc trước.** Người mới mở peek, chọn file, chọn máy. Lối tắt (Mí, chuột phải, menu Chia sẻ) luôn có sẵn và được gợi ý dần.  
4. **Nhận không cần mở app, trừ iPhone.** Trên máy tính, mọi thứ hiện trên Mí, không dùng thông báo của hệ điều hành.  
5. **Ít chữ nhất có thể, không từ kỹ thuật.** Lỗi nào cũng nói rõ bước tiếp theo.

**Thuật ngữ**

| Từ | Nghĩa |
| :---- | :---- |
| Mí | Thanh mảnh ở giữa cạnh trên màn hình (Windows) hoặc quanh notch (Mac). Ló ra khi cần |
| Khay | Nơi giữ file tạm trên Windows, kéo vào kéo ra được |
| Máy của tôi | Các máy của chính người dùng. Gửi qua lại tự nhận, không hỏi |
| Máy xung quanh | Máy có peek của người khác trong cùng mạng. Mỗi lượt gửi phải bấm Nhận |
| Hiện với máy xung quanh | Nút cho máy xung quanh thấy máy mình trong 10 phút |

## **Nhận diện peek**

Tên là **peek**, luôn viết thường, kể cả trong câu chữ trên giao diện ("Mở peek trên iPhone để nhận"). Tagline: **"Xong trong một cái liếc"**. Concept: mọi thao tác đơn giản, giúp người dùng xong việc nhanh.

**Biểu tượng**

* Hình bong bóng chat dạng giọt. Ba góc bo khoảng 29% cạnh, góc dưới phải bo lớn hơn một chút (khoảng 39%), góc dưới trái để sắc làm đuôi.  
* Cặp mắt hình vòm nửa tròn, đặt ở 44% chiều cao, lệch sang phải như đang liếc.  
* Nền tối: bong bóng vàng, mắt đen. Nền sáng hoặc nền vàng: bong bóng đen, mắt vàng hoặc kem.  
* Avatar mạng xã hội cắt theo hình tròn, mắt giữ đúng vị trí.

**Màu sắc**

| Màu | Mã | Dùng cho |
| :---- | :---- | :---- |
| Vàng chanh | \#E8FF3A | Màu chủ đạo: icon và điểm nhấn |
| Đen nền | \#0B0B0F / \#08080B | Nền chính |
| Kem | \#F5F3EC | Chữ trên nền tối |
| Mực | \#14141A | Chữ và icon trên nền sáng |
| Nền sáng | \#F3F3EE | Nền ở chế độ sáng |
| Vàng trầm | \#8A9A00 | Điểm nhấn trên nền sáng |

**Font chữ**

* Wordmark: Montserrat ExtraBold (800), khoảng cách chữ −0.07em, chữ "k" kéo sát thêm.  
* Tagline: Inter Medium (500), màu xám nhạt.  
* Chữ trong app: font hệ thống để giống native. Segoe UI Variable trên Windows, SF Pro trên macOS và iPhone, Roboto trên Android.  
* Tiêu đề lớn và website: Space Grotesk.

**Bố cục lockup**

* Thứ tự trên một hàng: icon, chữ peek, tagline. Hai khoảng cách bằng nhau (32px ở cỡ lớn).  
* Cặp mắt trong icon thẳng hàng với đường giữa thân chữ "peek".

## **Phong cách thiết kế**

Tinh tế, siêu đơn giản, ít chữ như AirDrop và Quick Share: nhìn là biết bấm vào đâu. Mỗi màn hình chỉ có một việc chính.

**Native từng nền tảng, nhưng nhìn như một gia đình**

* Windows theo Windows 11: bo góc, nền Mica, hiển thị đúng ở tỉ lệ 100% đến 200% và khi cắm nhiều màn hình.  
* macOS và iPhone theo chuẩn của Apple, Android theo Material.  
* Điểm chung trên mọi nền tảng: cùng icon, cùng màu nhấn vàng chanh, cùng cách xếp hai nhóm máy (Máy của tôi ở trên, Máy xung quanh ở dưới), cùng hình đại diện máy (biểu tượng loại máy kèm tên).

**Mí trên Windows: học Snap Layouts**

* Vị trí và hình dáng giống bảng Snap Layouts của Windows 11: trượt xuống từ giữa cạnh trên màn hình khi kéo file lên, bo góc, nền mờ.  
* Thay các ô bố cục cửa sổ bằng các máy. Rê file vào máy nào thì máy đó sáng viền vàng chanh.  
* Chuyển động mượt như Snap Layouts: trượt xuống và nở ra nhẹ, thu lại khi kéo file ra xa.  
* Màu theo peek thay cho màu mặc định của Windows.

**Mí trên macOS: nở ra từ notch**

* Máy có notch: Mí nở ra từ notch và hòa vào notch, chuyển động khớp với viền notch.  
* Máy không có notch hoặc màn hình rời: Mí là một viên nhỏ màu đen ở giữa cạnh trên, nở ra cùng một kiểu.

**Màu trong app**

* Theo chế độ sáng hoặc tối của hệ điều hành.  
* Tối: nền \#0B0B0F, chữ kem \#F5F3EC, nhấn vàng chanh \#E8FF3A.  
* Sáng: nền \#F3F3EE, chữ mực \#14141A, nhấn vàng trầm \#8A9A00. Vàng chanh gần như không đọc được trên nền sáng nên không dùng cho chữ.

**Ít chữ**

* Nút chỉ một hai từ: "Gửi", "Nhận", "Đồng ý".  
* Dùng biểu tượng loại máy và tên máy thay cho câu mô tả.  
* Số liệu viết gọn: "3 file · 45%".  
* Không quảng cáo, không thông báo rác. Hộp đánh giá có sẵn của hệ điều hành chỉ hiện một lần, sau lần gửi thành công thứ năm.

## **Cơ chế**

**Hai nhóm máy**

|  | Máy của tôi | Máy xung quanh |
| :---- | :---- | :---- |
| Là máy nào | Máy người dùng đã thêm vào | Máy có peek khác trong cùng mạng |
| Thấy nhau khi nào | Luôn thấy khi cùng mạng | Chỉ khi máy kia bấm "Hiện với máy xung quanh" |
| Khi nhận file | Tự nhận, không hỏi | Mỗi lượt đều hỏi Nhận hay Từ chối |
| Cần mở app để nhận | Không, trừ iPhone | Không, trừ iPhone |

**Ẩn mặc định**

* Máy chỉ hiện với Máy của tôi.  
* Muốn người khác gửi cho mình: bấm "Hiện với máy xung quanh". Máy hiện trong 10 phút rồi tự ẩn.  
* Muốn gửi cho người khác: máy họ bấm nút đó, máy họ sẽ hiện trong danh sách Máy xung quanh. Người gửi không cần bấm gì thêm.  
* Tên hiển thị là tên người dùng đặt lúc cài, đổi được trong Cài đặt.

**Thêm vào Máy của tôi**

* Trong danh sách Máy xung quanh, bấm "Thêm vào máy của tôi" ở máy muốn thêm.  
* Máy kia hiện lời hỏi: **\[Tên máy\] muốn thêm vào Máy của tôi**. Dòng phụ: "Hai máy sẽ gửi cho nhau không cần hỏi." Nút: Từ chối, Đồng ý.  
* Đồng ý xong, cả hai máy cùng chuyển lên nhóm Máy của tôi của nhau. Không QR, không so số.  
* Hai máy nhớ nhau bằng khóa riêng, không bằng tên. Đổi tên máy hay đổi mạng vẫn nhận ra nhau, máy lạ đặt trùng tên cũng không giả được.  
* Giống ghép tai nghe Bluetooth: ghép một lần, lần sau tự vào. Khác ở chỗ tai nghe không tự gửi file cho anh.

**Lời hỏi khi nhận từ máy xung quanh**

* Hiện ở đâu: máy tính hiện trên Mí, không dùng thông báo của Windows hay macOS. Android hiện thông báo. iPhone hiện trong app.  
* Nội dung: biểu tượng loại máy và tên máy gửi, tên file với đuôi đầy đủ, số file và dung lượng. Ví dụ: "Lan's iPhone · 3 file · 120 MB".  
* Không xem trước nội dung file trước khi bấm Nhận.  
* Nút: Từ chối, Nhận. Trong menu ba chấm: Chặn máy này.  
* Không bấm gì trong 1 phút thì tự từ chối.

**An toàn**

* Máy lạ không bao giờ tự đưa file vào máy mình được, luôn phải qua nút Nhận.  
* File nhận từ máy xung quanh được Windows và macOS coi như file tải từ mạng, nên hệ điều hành vẫn kiểm tra khi mở.  
* Mất máy: vào Máy của tôi trên máy còn lại, xóa máy đó. Máy bị xóa không gửi tự do được nữa.

## **Theo từng nền tảng**

|  | Windows | macOS | Android | iPhone |
| :---- | :---- | :---- | :---- | :---- |
| peek nằm ở đâu | Icon ở khay hệ thống, tự chạy khi bật máy | Icon trên thanh menu, tự chạy khi bật máy | App, chạy nền để nhận | App, phải mở mới nhận được |
| Gửi, cách chính | Bấm icon, mở cửa sổ peek, chọn file, chọn máy | Bấm icon, mở cửa sổ peek, chọn file, chọn máy | Mở app, bấm Gửi, chọn file, chọn máy | Mở app, bấm Gửi, chọn file, chọn máy |
| Lối tắt gửi | Kéo file lên Mí; chuột phải, Gửi bằng peek; phím tắt | Kéo file lên Mí ở notch; Finder, Chia sẻ, peek; phím tắt | Chia sẻ, peek | Chia sẻ, peek |
| Nhận từ Máy của tôi | Tự nhận, Mí hiện tiến độ và kết quả | Tự nhận, Mí hiện tiến độ và kết quả | Tự nhận, thông báo khi xong | Tự nhận khi app đang mở |
| Nhận từ máy xung quanh | Mí mở ra với lời hỏi | Mí mở ra với lời hỏi | Thông báo có nút Nhận, Từ chối | Lời hỏi trong app |
| File lưu ở đâu | Downloads\\peek | Downloads/peek | Download/peek | Ảnh và video vào Ảnh, file khác vào Tệp, thư mục peek |
| Khác | Có Khay giữ file tạm | Không có Khay ở bản 1.0 |  | Nhắc giữ app mở khi đang nhận |

Trên máy tính, mọi lời hỏi, tiến độ và kết quả đều nằm trên Mí. peek không dùng thông báo của Windows hay macOS.

## **Danh sách màn hình**

18 màn hình. Mỗi màn hình một việc chính. Màn hình đánh dấu (chính) là nơi người mới dùng để gửi.

**Windows**

| Mã | Màn hình | Việc chính | Thành phần |
| :---- | :---- | :---- | :---- |
| W1 | Icon ở khay hệ thống | Mở peek | Bấm: mở W2. Chuột phải: Hiện với máy xung quanh, Cài đặt, Thoát |
| W2 | Cửa sổ peek (chính) | Chọn file và máy nhận | Vùng thả file kèm nút Chọn file; Máy của tôi; Máy xung quanh; nút Hiện với máy xung quanh; icon Cài đặt |
| W3 | Mí | Gửi nhanh, hỏi, báo tiến độ | Xem mục Mí trên Windows và macOS |
| W4 | Khay | Giữ file tạm | Dải file nhỏ, kéo vào kéo ra |
| W5 | Cài lần đầu | Đặt tên, thêm điện thoại | 2 bước, xem Luồng 1 |
| W6 | Cài đặt | Chỉnh vài thứ | Xem mục Cài đặt |
| W7 | Chuột phải vào file | Gửi không cần mở peek | Dòng "Gửi bằng peek", mở danh sách máy |

**macOS**

| Mã | Màn hình | Việc chính | Thành phần |
| :---- | :---- | :---- | :---- |
| M1 | Icon trên thanh menu và cửa sổ peek (chính) | Chọn file và máy nhận | Giống W2, theo giao diện macOS, xổ xuống từ icon |
| M2 | Mí ở notch | Gửi nhanh, hỏi, báo tiến độ | Xem mục Mí trên Windows và macOS |
| M3 | Cài lần đầu | Đặt tên, cấp quyền, thêm điện thoại | Xem Luồng 1 |
| M4 | Cài đặt | Chỉnh vài thứ | Xem mục Cài đặt |
| M5 | Finder, Chia sẻ, peek | Gửi không cần mở peek | Danh sách máy trong khung Chia sẻ |

**Điện thoại (Android và iPhone, cùng bố cục)**

| Mã | Màn hình | Việc chính | Thành phần |
| :---- | :---- | :---- | :---- |
| P1 | Cài lần đầu | Đặt tên, cấp quyền | Xem Luồng 1 |
| P2 | Màn hình chính (chính) | Gửi | Nút Gửi lớn; Máy của tôi; Máy xung quanh; nút Hiện với máy xung quanh; icon Cài đặt |
| P3 | Lời hỏi | Nhận hay Từ chối, Đồng ý thêm máy | Tấm nổi từ dưới lên |
| P4 | Tiến độ | Xem đang gửi, nhận | Vòng tiến độ trên ô máy, nút Hủy |
| P5 | Chia sẻ, peek | Gửi từ app khác | Danh sách máy |
| P6 | Cài đặt | Chỉnh vài thứ | Xem mục Cài đặt |

## **Mí trên Windows và macOS**

Mí là nơi duy nhất peek nói chuyện với người dùng máy tính khi không mở cửa sổ: hỏi, báo tiến độ, báo kết quả, và là lối tắt để gửi bằng cách kéo thả.

**Các trạng thái**

| Trạng thái | Khi nào | Windows | macOS có notch |
| :---- | :---- | :---- | :---- |
| Nghỉ | Không có việc gì | Không thấy gì. Nếu bật "Luôn hiện vệt mảnh": một vệt ngắn ở giữa cạnh trên | Không thấy gì, notch vẫn là notch |
| Gợi ý | Bắt đầu kéo một file | Vệt nhỏ có icon peek ló xuống ở giữa cạnh trên, giống lúc kéo cửa sổ lên để gọi Snap Layouts | Notch phình nhẹ sang hai bên |
| Mở | Kéo file tới gần Mí | Bảng trượt xuống, mỗi ô là một máy. Hàng trên Máy của tôi, hàng dưới Máy xung quanh. Rê vào ô nào ô đó sáng viền vàng chanh. Thả là gửi | Notch giãn xuống thành khối đen liền mạch với notch, các ô máy giống Windows |
| Hỏi | Nhận từ máy xung quanh, hoặc có máy xin thêm vào Máy của tôi | Mí tự mở xuống với lời hỏi và hai nút | Notch tự giãn xuống với lời hỏi và hai nút |
| Đang chạy | Đang gửi hoặc nhận | Viên nhỏ: icon máy, vòng tiến độ, "3 file · 45%". Bấm vào để xem hoặc Hủy | Notch dài ra hai bên: trái icon máy, phải vòng tiến độ |
| Xong | Gửi hoặc nhận xong | "Đã nhận 3 file", nút Mở. Tự thu lại sau vài giây | Giống Windows, thu về notch |

Mac không có notch hoặc màn hình rời: Mí là viên đen nhỏ ở giữa cạnh trên, đi qua đủ các trạng thái như trên.

**Chuyển động**

* Windows: giống Snap Layouts, trượt xuống và hiện dần, chậm dần về cuối, khoảng 200 đến 250 ms. Thu lại nhanh hơn mở.  
* macOS: nở ra từ notch theo kiểu lò xo nhẹ như Dynamic Island, không nảy mạnh. Viền khối luôn dính liền notch, không tách rời.  
* Mượt ở 60 hình mỗi giây trở lên, kể cả máy cấu hình thấp và màn hình tần số quét cao.  
* Người dùng bật "Giảm chuyển động" trong hệ điều hành: chỉ hiện dần, không trượt.

**Quy tắc**

* Lời hỏi không cướp bàn phím. Đang gõ dở thì vẫn gõ tiếp được, không lỡ tay bấm Nhận.  
* Nhiều màn hình: Mí hiện ở màn hình đang có con trỏ chuột.  
* Đang xem video, chơi game, trình chiếu toàn màn hình: Mí không tự mở. Lời hỏi chờ đến khi thoát toàn màn hình, máy gửi thấy "Đang chờ". File từ Máy của tôi vẫn nhận ngầm, xong mới báo.  
* Nhiều lời hỏi cùng lúc: hiện từng cái, có số "1/2".

## **Luồng 1: cài lần đầu**

Mục tiêu: dưới một phút, máy tính và điện thoại đã là Máy của tôi của nhau. Không có màn hình chào, không có hướng dẫn nhiều trang.

**Máy tính (Windows và macOS)**

| Bước | Người dùng thấy | Người dùng làm |
| :---- | :---- | :---- |
| 1\. Cài | Tải từ peek.vn. Windows: file cài. Mac: kéo peek vào Applications | Mở peek |
| 2\. Đặt tên | Icon peek, ô tên điền sẵn tên máy. Dòng phụ: "Máy khác sẽ thấy tên này" | Sửa nếu muốn, bấm Tiếp |
| 3\. Quyền mạng | Hộp của hệ điều hành. Windows: tường lửa hỏi cho peek dùng mạng riêng. Mac: hỏi cho peek tìm thiết bị trong mạng | Bấm Cho phép |
| 4\. Thêm điện thoại | Mã QR tải app peek cho điện thoại, bên cạnh là danh sách máy xung quanh đang hiện, cập nhật liên tục | Khi điện thoại hiện trong danh sách, bấm Thêm vào máy của tôi. Hoặc bấm Để sau |
| 5\. Xong | Cửa sổ peek mở ra. Một mũi tên nhỏ chỉ vào icon: "peek ở đây" | Bắt đầu gửi |

Mã QR chỉ là link tải app, không dùng để ghép máy.

**Điện thoại (Android và iPhone)**

| Bước | Người dùng thấy | Người dùng làm |
| :---- | :---- | :---- |
| 1\. Đặt tên | Ô tên. Android điền sẵn tên máy. iPhone để trống, gợi ý "iPhone của \[tên\]" | Nhập hoặc sửa, bấm Tiếp |
| 2\. Quyền | Mỗi quyền một dòng lý do rồi đến hộp của hệ điều hành. iPhone: Mạng cục bộ. Android: Thông báo | Bấm Cho phép |
| 3\. Nối với máy tính | Nút lớn "Hiện với máy xung quanh". Dòng phụ: "Mở peek trên máy tính để thêm máy này" | Bấm nút. Hoặc Để sau |
| 4\. Đồng ý | Lời hỏi "\[Tên máy tính\] muốn thêm vào Máy của tôi" | Bấm Đồng ý |
| 5\. Xong | Màn hình chính, máy tính đã nằm trong Máy của tôi. Nút "Gửi thử một ảnh" | Gửi thử hoặc bỏ qua |

Quyền truy cập Ảnh trên iPhone chỉ hỏi lần đầu nhận ảnh, không hỏi lúc cài.

## **Luồng 2: thêm máy**

Dùng khi có thêm máy mới: laptop công ty, máy bàn ở nhà, iPad. Gọi máy đang có là A, máy mới là B.

**Thêm**

| Bước | Máy | Làm gì |
| :---- | :---- | :---- |
| 1 | B | Bấm "Hiện với máy xung quanh" |
| 2 | A | Thấy B trong Máy xung quanh. Bấm nút \+ trên ô của B (Thêm vào máy của tôi) |
| 3 | B | Lời hỏi "\[A\] muốn thêm vào Máy của tôi". Bấm Đồng ý |
| 4 | Cả hai | Ô máy kia nhẹ nhàng chuyển lên Máy của tôi. B tự tắt chế độ hiện |

B bấm Từ chối: A thấy "Chưa được đồng ý", không có gì thay đổi.

**Xóa**

* Máy tính: chuột phải vào ô máy, "Xóa khỏi Máy của tôi". Điện thoại: nút ba chấm trên ô máy.  
* Không hỏi lại. Hiện "Đã xóa · Hoàn tác" trong vài giây.  
* Xóa ở một máy là đủ. Hai máy đều thôi tự nhận của nhau, lần sau gửi sẽ bị hỏi như máy xung quanh.

## **Luồng 3: gửi giữa các máy của tôi**

Lúc đầu người dùng mở peek để gửi, giống LocalSend. Lối tắt có sẵn từ đầu nhưng được gợi ý dần.

**Từ máy tính (Windows và macOS)**

| Bước | Làm gì | Thấy gì |
| :---- | :---- | :---- |
| 1 | Bấm icon peek ở khay hệ thống hoặc thanh menu | Cửa sổ peek |
| 2 | Kéo file vào cửa sổ, hoặc bấm Chọn file | "3 file · 120 MB" |
| 3 | Bấm vào máy nhận trong Máy của tôi | Gửi ngay, không có nút xác nhận. Vòng tiến độ trên ô máy |
| 4 | Đóng cửa sổ nếu muốn | Mí báo tiến độ, xong thì "Đã gửi" |

Bấm vào máy khi chưa chọn file: mở hộp chọn file, chọn xong là gửi.

Lối tắt:

* Kéo file lên Mí (giữa cạnh trên trên Windows, notch trên Mac), thả vào ô máy.  
* Windows: chuột phải vào file, Gửi bằng peek, chọn máy.  
* Mac: trong Finder bấm Chia sẻ, chọn peek, chọn máy.  
* Phím tắt mở cửa sổ peek.

**Từ điện thoại**

| Bước | Làm gì | Thấy gì |
| :---- | :---- | :---- |
| 1 | Mở app, bấm Gửi | Hộp chọn ảnh hoặc file của hệ điều hành |
| 2 | Chọn xong | Danh sách máy, Máy của tôi ở trên |
| 3 | Bấm vào máy nhận | Gửi ngay, vòng tiến độ trên ô máy, xong thì dấu tích |

Lối tắt: trong Ảnh hoặc app bất kỳ, bấm Chia sẻ, chọn peek, chọn máy.

**Máy nhận**

* Máy tính: tự nhận, Mí hiện tiến độ rồi "Đã nhận 3 file" với nút Mở.  
* Android: tự nhận, thông báo khi xong.  
* iPhone: tự nhận nếu app đang mở. App đang đóng thì máy gửi thấy "Mở peek trên \[tên iPhone\]".

**Gợi ý lối tắt**

* Sau lần gửi thứ ba qua cửa sổ, hiện một dòng kèm hình động nhỏ, một lần duy nhất. Windows: "Lần sau kéo file lên mép trên màn hình." Mac: "Lần sau kéo file lên notch." Điện thoại: "Gửi từ Ảnh: Chia sẻ, peek."  
* Nút Đã hiểu. Không nhắc lại.

**Nhánh phụ**

* Kéo thư mục: gửi cả thư mục, giữ nguyên cấu trúc.  
* Hủy: bấm vào ô máy đang chạy hoặc vào Mí, bấm Hủy. Bên nhận xóa phần đang dở.  
* File trùng tên bên nhận: tự thêm số, ví dụ "baocao (2).pdf". Không hỏi.

## **Luồng 4: gửi với máy xung quanh**

Ví dụ: Mac của Lan gửi file sang máy Windows của Hưng ở cùng văn phòng.

| Bước | Ai | Làm gì | Thấy gì |
| :---- | :---- | :---- | :---- |
| 1 | Hưng | Bấm "Hiện với máy xung quanh" | Nút đổi thành "Đang hiện · 9:58", đếm ngược. Bấm lại để tắt sớm |
| 2 | Lan | Mở peek, chọn file, bấm vào máy của Hưng trong Máy xung quanh | "Đang chờ Hưng nhận" |
| 3 | Hưng | Mí mở ra: "Lan's Mac · 3 file · 120 MB", tên từng file. Bấm Nhận | Tiến độ trên Mí, xong thì "Đã nhận" với nút Mở |
| 4 | Lan | Không cần làm gì | Tiến độ trên ô máy, xong thì dấu tích |

Người nhận dùng điện thoại: Android hiện thông báo có nút Nhận và Từ chối. iPhone hiện lời hỏi trong app, app phải đang mở.

**Nhánh phụ**

* Từ chối: người gửi thấy "Đã từ chối".  
* Không bấm gì trong 1 phút: người gửi thấy "Không phản hồi".  
* Chặn: trong lời hỏi, menu ba chấm, Chặn máy này. Máy bị chặn không thấy mình và không gửi được nữa. Bỏ chặn trong Cài đặt.  
* Hết 10 phút giữa lúc đang nhận: vẫn nhận nốt, chỉ thôi hiện với máy mới.  
* Hai người cần gửi qua lại thường xuyên vẫn dùng cách này. Máy của tôi chỉ dành cho máy của chính mình.

## **Cài đặt**

Một trang, chia nhóm như LocalSend. Mỗi dòng một việc, không có trang con trừ danh sách máy.

| Nhóm | Mục | Lựa chọn | Mặc định | Nền tảng |
| :---- | :---- | :---- | :---- | :---- |
| Máy này | Tên máy | Sửa được | Tên đặt lúc cài | Tất cả |
| Máy | Máy của tôi | Danh sách, xóa từng máy |  | Tất cả |
| Máy | Máy đã chặn | Danh sách, bỏ chặn |  | Tất cả |
| Nhận file | Lưu vào | Chọn thư mục | Downloads/peek | Windows, macOS, Android |
| Mí | Cách hiện Mí | Luôn hiện vệt mảnh, Chỉ hiện khi cần | Chỉ hiện khi cần | Windows, macOS |
| Mí | Phím tắt mở peek | Đổi được | Ctrl \+ Alt \+ P trên Windows, Control \+ Option \+ P trên Mac | Windows, macOS |
| Chung | Tự chạy khi bật máy | Bật, Tắt | Bật | Windows, macOS |
| Chung | Ngôn ngữ | Tiếng Việt, English | Theo hệ điều hành | Tất cả |
| Về peek | Phiên bản, Gửi góp ý, Quyền riêng tư |  |  | Tất cả |

Không có: cổng mạng, mã hóa, chọn giao diện sáng tối, tự nhận từ máy xung quanh, tắt Mí. Những thứ này peek tự lo hoặc cố ý không cho chỉnh.

Cập nhật trên máy tính tự chạy ngầm, xong thì Mí báo một lần "peek đã cập nhật". Điện thoại cập nhật qua App Store và Google Play.

## **Lỗi và trường hợp đặc biệt**

Trên máy tính, lỗi hiện trong cửa sổ peek nếu đang mở, nếu không thì trên Mí. Không dùng thông báo của hệ điều hành. Mỗi lỗi một câu ngắn và tối đa một nút.

| Tình huống | Người dùng thấy | Nút |
| :---- | :---- | :---- |
| Chưa thấy máy nào | "Chưa thấy máy nào. Hai máy cần cùng Wi-Fi và đều mở peek." | Thêm điện thoại |
| Một máy trong Máy của tôi đang tắt, khác mạng hoặc iPhone đóng app | Ô máy mờ, ghi "Không thấy". Bấm vào: "Mở peek trên \[tên\] và nối cùng Wi-Fi." |  |
| Wi-Fi không cho các máy thấy nhau (Wi-Fi khách, Wi-Fi công ty) | "Mạng này không cho các máy thấy nhau. Thử Wi-Fi khác hoặc phát Wi-Fi từ điện thoại." |  |
| Mất mạng giữa chừng | "Mất kết nối, sẽ gửi tiếp." Có mạng lại thì gửi tiếp từ chỗ dừng, không gửi lại từ đầu |  |
| 10 phút vẫn không nối lại được | "Chưa gửi được." | Thử lại |
| iPhone thoát app khi đang gửi hoặc nhận | Tạm dừng. Mở lại app thì chạy tiếp |  |
| Máy nhận không đủ chỗ | Báo trước khi gửi. Người gửi: "\[Tên\] không đủ chỗ trống." Người nhận: "Không đủ chỗ cho 3 file · 2 GB." |  |
| File lỗi khi truyền | peek tự kiểm tra và tự gửi lại phần lỗi, người dùng không thấy gì. Vẫn lỗi: "Chưa gửi được." | Thử lại |
| File chạy được (.exe, .bat, .msi, .app, .apk) từ máy xung quanh | Lời hỏi hiện đủ đuôi file và thêm nhãn nhỏ "File chạy được" |  |
| Windows chặn peek dùng mạng | "peek chưa được dùng mạng." | Mở cài đặt |
| macOS hoặc iPhone tắt quyền Mạng cục bộ | "Bật Mạng cục bộ cho peek để thấy các máy khác." | Mở cài đặt |
| Android tắt thông báo | Dải nhỏ trong app: "Bật thông báo để nhận file khi không mở app." | Bật |
| Máy kia dùng bản peek cũ không tương thích | "Cập nhật peek trên \[tên\] để gửi." |  |
| Hai máy xung quanh trùng tên | Hiện thêm loại máy để phân biệt. Máy của tôi không bị ảnh hưởng vì nhận nhau bằng khóa |  |

## **Câu hỏi còn mở**

Các câu anh quyết đã chốt hết. Còn lại là việc báo và hỏi dev.

* ☒ (Anh quyết) Đồng nghiệp gửi qua lại hằng ngày thì mỗi lần người nhận phải bấm "Hiện với máy xung quanh". Bản 1.0 chấp nhận điều này không?  
* ☒ (Báo dev) Bỏ quét mã QR và bước so dãy số khi thêm máy (ADR-0012 hiện đang yêu cầu so số), thay bằng nút "Thêm vào máy của tôi" kèm một lần "Đồng ý" ở máy kia.  
* ☒ (Đã chốt) Font: font hệ thống cho chữ trong app, Space Grotesk cho tiêu đề lớn và website.  
* ☒ (Đã chốt) Tagline: "Xong trong một cái liếc". Bỏ "Liếc phát biết".  
* ☒ (Đã chốt) Mọi tài liệu viết "peek", chữ thường hết.  
* ☐ (Báo dev) macOS 1.0 giờ có cả gửi: cửa sổ peek từ thanh menu, Mí ở notch, peek trong menu Chia sẻ của Finder. Kế hoạch cũ chỉ có nhận. Cần ước lượng công và ảnh hưởng tới ngày ra mắt.  
* ☐ (Báo dev) Bỏ thông báo của Windows và macOS. Mọi lời hỏi, tiến độ, kết quả và lỗi trên máy tính nằm trên Mí.  
* ☐ (Hỏi dev) Mí kiểu Snap Layouts trên Windows và Mí ở notch trên Mac có làm mượt được như Snap Layouts và Dynamic Island không, mà không cướp bàn phím khi người dùng đang gõ? Tốn bao nhiêu công?  
* ☐ (Hỏi dev) Sửa giao thức để máy chưa thêm vào Máy của tôi vẫn gửi được cho nhau thì tốn bao nhiêu công?  
* ☐ (Hỏi dev) Giả định trong spec: gửi cả thư mục, báo thiếu chỗ trước khi gửi, xóa phần dở khi hủy, xóa máy ở một bên là cả hai bên thôi tự nhận. Giao thức đã làm được chưa?  
* ☐ (Hỏi dev) Những hãng Android nào cần thêm bước cho phép chạy nền?  
* ☐ (Hỏi dev) Khi iPhone chưa mở peek, máy gửi chờ bao lâu thì báo "Mở peek trên \[tên iPhone\]"? Từ bao nhiêu MB thì tính là file lớn trên iPhone?  
* ☐ (Hỏi dev) Đánh dấu "đến từ mạng" cho file nhận trên Windows và macOS, và nhãn "File chạy được" đã có chưa?  
* ☐ (Hỏi dev) Mục "Gửi bằng peek" trong menu chuột phải Windows, peek trong menu Chia sẻ của Android, iPhone và Finder đã có chưa?
## **Bổ sung sau buổi chốt Windows (13/11/2026)**

Các quyết định dưới đây đè lên phần trên khi lệch nhau. Quy tắc chung: mockup quyết hình ảnh (bố cục, kích thước, màu, chuyển động, chữ trên nút), spec quyết hành vi.

**Màu bổ sung**

| Màu | Mã | Dùng cho |
| :---- | :---- | :---- |
| Nền cửa sổ tối | \#17171C | Nền dự phòng khi không có Mica |
| Nền ô máy tối | \#1C1C23 | Ô máy trong Máy của tôi |
| Chữ nhấn sáng | \#5F6B00 | Chữ nhấn trên nền sáng (viền chọn vẫn \#8A9A00) |
| Cam cảnh báo | \#FFC46B trên rgba(255,184,77,.16) | Nhãn "File chạy được" |

* Cửa sổ peek và Cài đặt: Mica khi Windows 11 hỗ trợ và người dùng không tắt hiệu ứng trong suốt, nếu không thì nền đặc \#17171C (tối) / \#F3F3EE (sáng). Theo sáng tối của hệ điều hành.
* Mí và Khay: luôn acrylic tối như mockup, không đổi theo sáng tối.

**Chi tiết mockup được nhận vào spec**

* W2: đã chọn file thì vùng thả thay bằng thẻ file ("3 file · 120 MB", tên file, nút X Bỏ chọn file). Nút Hủy hiện rõ dưới ô máy đang gửi. Cửa sổ không có nút Phóng to. Chưa có máy xung quanh: "Chưa có máy nào đang hiện. Máy kia bấm Hiện với máy xung quanh để xuất hiện ở đây."
* Ô máy hình tròn trong cửa sổ peek, hình chữ nhật 120×92 trên Mí. Cùng icon loại máy và tên máy.
* Lời hỏi trên Mí: dung lượng từng file, vạch đếm ngược 1 phút ở đáy.
* Khay: thả vào ô Khay ở cuối bảng Mí để giữ tạm; bảng "Khay · N file" có nút Xóa hết. Chỉ vào Khay qua Mí, không có phím tắt riêng.
* Cài đặt là một trang trong cửa sổ peek, có nút Quay lại; trang con Máy của tôi và Máy đã chặn cũng vậy.
* Cài lần đầu: chỉ báo bước 2 chấm, "Đang tìm thêm", "Chưa có app? Quét để tải". QR trỏ https://peek.vn/tai.
* Menu icon ở khay hệ thống: đúng ba dòng. Đang hiện với máy xung quanh thì dòng đầu đổi thành "Đang hiện · 9:58", bấm để tắt sớm.

**Hành vi chốt thêm**

* Ghép máy không có bước so số. Rủi ro đã chấp nhận: kẻ đứng giữa trong mạng ngay lúc ghép có thể thành Máy của tôi. Giảm bằng cách chỉ nhận lời xin ghép khi máy kia đang Hiện với máy xung quanh và lời xin xuất phát từ một cú bấm thật. Ghi trong ADR của repo mã nguồn.
* "Ẩn" là ẩn ở mức hiển thị: máy vẫn quảng bá trong mạng, kèm cờ ẩn; máy xung quanh không hiện nó và nó từ chối lượt gửi từ máy lạ khi đang ẩn.
* Máy của tôi luôn tự nhận, không có công tắc Tự nhận theo từng máy.
* Phím tắt Ctrl + Alt + P bị app khác dùng: dòng phím tắt trong Cài đặt hiện "Đang bị app khác dùng · Đổi".
* Mí không còn tuỳ chọn trái/phải, luôn ở giữa. Capsule nhạc (ngoài spec) chuyển sang góc trên phải.
* Lịch sử clipboard và capsule nhạc (ngoài spec) giữ tính năng, không có trong menu khay; bật tắt ở nhóm "Thêm" cuối trang Cài đặt, ngay trên Về peek.
* Hộp đánh giá sau lần gửi thứ năm chỉ áp dụng cho bản phát hành qua store. Windows (cài từ peek.vn) không có.
* Windows: "Gửi bằng peek" lên menu chuột phải đầu tiên của Windows 11 cần chứng chỉ ký số. Trước khi có, mục này nằm ở menu "Hiện thêm tùy chọn".
