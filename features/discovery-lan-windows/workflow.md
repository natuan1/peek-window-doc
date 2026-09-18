# Discovery LAN trên Windows — quy trình

## Người dùng thấy gì

```
Mở Snappy (hoặc nó tự chạy nền)
        ↓
Bấm icon khay → bảng trạng thái
        ↓
Mục "Thiết bị lân cận"
   ├── chưa thấy ai → "Chưa thấy máy nào. Mở app trên điện thoại cùng Wi-Fi."
   ├── thấy         → "• iPhone của Tuấn · 192.168.1.50"
   └── vừa mất      → "• iPhone của Tuấn · không thấy nữa"
```

Danh sách tối đa bốn dòng, phần còn lại gộp thành *"và N thiết bị nữa"* — bảng
neo ở cạnh màn hình và không cuộn được, nên một danh sách dài sẽ mọc ra ngoài
màn hình.

## Hệ thống làm gì

```
Khởi động
   ├── mở/sinh Device Identity (P-256, kho khoá CNG)   → deviceId
   ├── ĐĂNG KÝ  `<tên máy> [dấu Device]._peek._tcp.local`
   │             → SRV: <tên máy>.local : 8443
   │             → TXT: pv=1 · id=<deviceId> · port=8443
   └── DUYỆT    `_peek._tcp.local`  (theo sự kiện, không hỏi vòng — SPEC §36)
                     ↓
        callback (vài lần mỗi giây, mỗi lần một mẻ bản ghi rời rạc)
                     ↓
        ghép PTR + SRV + TXT + A/AAAA thành endpoint
                     ↓
        bỏ qua nếu deviceId là CHÍNH MÌNH  (lọc theo deviceId, không theo IP)
                     ↓
        vào bảng: làm mới "lần cuối nghe thấy"
                     ↓
        chỉ khi có gì NHÌN THẤY ĐƯỢC đổi → vẽ lại bảng trạng thái + ghi nhật ký

Mỗi 10 giây: quét bảng → ai im quá 120 giây thì đánh dấu offline (không xoá)

Thoát: RÚT đăng ký trước khi tắt vòng lặp thông điệp
```

## Hai nhịp, hai lý do khác nhau

| Nhịp | Con số | Vì sao |
|---|---|---|
| Quét bảng | 10 giây | quyết định **độ trễ** của việc một máy đã tắt biến khỏi màn hình |
| Ngưỡng offline | 120 giây | TTL bản ghi DNS-SD (RFC 6762 §10), trùng con số Android dùng |

Trộn hai con số này là sai: quét dày không làm peer chết nhanh hơn, nó chỉ làm
màn hình nói thật sớm hơn.
