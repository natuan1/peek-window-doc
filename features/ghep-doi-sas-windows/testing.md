# Ghép đôi SAS — đo bằng gì

Bốn lớp bằng chứng, mỗi lớp trả lời một câu khác nhau. Không lớp nào thay được
lớp khác.

## 1. Vector liên implementation — "có ra cùng con số với bản Rust không"

`CryptoVectorTests` (15 test) so với
`interoperability/fixtures/ultp-v1-vectors.json`, do bản Rust sinh ra.

Đây là loại bằng chứng khác hẳn phần còn lại của bộ test. Mọi test khác hỏi *"mã
này có nhất quán với chính nó không"*; chỗ này hỏi *"mã này có ra cùng con số với
một implementation độc lập không"* — và với SAS thì chỉ câu thứ hai có nghĩa. Một
transcript sai **nhất quán** vẫn sinh ra sáu chữ số trông hoàn hảo; nó chỉ hỏng ở
chỗ iPhone hiện một con số khác, và lúc ấy không ai biết bên nào sai.

Cặp `vectors` của `interoperability/run.sh` đứng canh chiều còn lại: vector ấy
vẫn là thứ Rust sinh ra **hôm nay**. Hai hàng rào khác nhau — cặp ấy giữ vector
không cũ đi, `CryptoVectorTests` giữ Snappy không lệch khỏi vector.

Bao gồm một vector riêng cho SAS có **số 0 đứng đầu**: xác suất 1/10, đủ thường
để dạy hư người dùng và không đủ thường để bị bắt trong test thủ công.

## 2. MITM thật — "ràng buộc kênh có hoạt động không"

`MitmTests` + `MitmRelay.cs`. Kẻ tấn công kết thúc TLS về phía nạn nhân bằng
certificate của chính nó, mở kênh TLS riêng tới responder, chuyển tiếp **nguyên
văn từng byte**. Không parse HTTP, không sửa gì. Mã tấn công nằm trong `tests/`
nên không link được vào sản phẩm.

> 📐 **Số đo của lỗ hổng.** Gỡ `responderTLSSPKI` khỏi transcript rồi chạy lại:
>
> ```
> Assert.NotEqual() Failure: Strings are equal
> Expected: Not "203235"
> Actual:       "203235"
> ```
>
> Dưới một MITM hoàn chỉnh, hai đầu tính ra **cùng một** SAS. Người dùng làm đúng
> mọi thứ — nhìn hai màn hình, so hai con số, thấy khớp — và cấp phép cho kẻ đứng
> giữa.

**Nhóm đối chứng âm là bắt buộc** (SECURITY.md §5.7): `TcpRelay` chuyển tiếp ở
tầng TCP, cùng số chặng, không ai đổi certificate → SAS phải **khớp**. Nó vẫn
xanh dưới đột biến trên, tức nó thật sự phân biệt "ràng buộc kênh chạy" với "một
bug làm SAS lệch mọi lúc" — và bug thứ hai không vô hại: một protocol báo động
giả liên tục dạy người dùng bỏ qua cảnh báo, lúc đó §7.3 sụp đổ.

Kèm một test ghi lại **giới hạn đã biết**:
`Nguoi_dung_bam_bua_thi_ke_tan_cong_o_lai_vinh_vien`. Nó không phải lỗ hổng — nó
là §7.3 ở dạng đo được. Nếu một ngày nó đỏ, nghĩa là đã có thêm một lớp phòng thủ
và §7.3 cần được viết lại.

## 3. Cặp `tls-handshake` — "TLS stack này có verify chữ ký handshake không"

```
./interoperability/run.sh tls-handshake
```

📐 19/09/2026, `SslStream`/SChannel, đối thủ là
`reference/rust-host/examples/stolen-cert.rs`:

| Nhánh | Certificate | Khoá ký | Kết quả |
|---|---|---|---|
| `--honest` | của nạn nhân | của nạn nhân | **hoàn tất** (đối chứng) |
| `--honest` + pin đúng | — | — | **hoàn tất** (đối chứng) |
| `--honest` + pin lệch 1 ký tự | — | — | `deviceIdentityChanged` (S14) |
| `--stolen` | của nạn nhân | của kẻ tấn công | **hỏng** |

Xem [ADR-0012](../../adr/0012-cap-harness-do-hanh-vi-nen-tang-phai-chay-cho-tung-tls-stack.md)
cho lý do cặp này phải chạy cho **mỗi** TLS stack, và cho lỗi tự bắt được ở chính
đầu dò.

⚠️ Nhánh Swift `BỎQUA` trên máy Windows và ngược lại. **Bỏ qua = chưa đo, không
phải đã đạt** — `run.sh` in đúng câu ấy ở cuối bảng.

## 4. Bản AOT thật — "thứ người dùng tải về có chạy không"

Bộ test chạy trên runtime thường, nơi `JsonSerializer` còn đường phản chiếu. Bản
publish AOT thì không, và nó **không đỏ lúc build** — nó ném
`NotSupportedException` lúc chạy, tức chỉ lộ ra trên máy người dùng.

```
snappy-interop pair --host 127.0.0.1:8443 --name "Dau do"
```

📐 19/09/2026, một tiến trình thật gõ cửa `Snappy.exe` đã publish:

```
PAIR sas=034212 peer=dev_hHDZJdu4lums5uk-SJt985_NQdvSM1ecLpR6lakBiJE name=NATUAN1
```

Bốn thứ được khẳng định cùng lúc:

* `D6` giữ **số 0 đứng đầu** qua ILCompiler;
* nhật ký server ghi *"đang chờ người dùng so mã"* và **không** ghi SAS;
* cửa sổ `Snappy — ghép đôi` có thật trên màn hình (đếm bằng `EnumWindows`);
* và **không** có `trusted-peers.json` nào được tạo — bất biến **S6**, không ai
  xác nhận thì không có Trusted Peer.

## Bảng tổng

| Lớp | Số test | Câu hỏi nó trả lời |
|---|---|---|
| `CryptoVectorTests` | 15 | khớp bản Rust từng byte không |
| `PairingFlowTests` | 10 | hai máy thật có ra cùng số không, và S5/S6 |
| `MitmTests` | 5 | ràng buộc kênh có chặn được kẻ đứng giữa không |
| `AuthFlowTests` | 16 | S7/S8/S9, uỷ quyền, hai dòng §4.3 |
| `TrustStoreTests` | 10 | bền qua restart, hình dạng schema, tệp hỏng |

Tổng bộ test Windows: **226**, xanh.

## Cái không đo được từ đây

**CryptoKit của Apple và `System.Security.Cryptography` của .NET có ra cùng một
`pairKey` không.** Vector làm phép so ấy cho *Rust ↔ Windows*; cạnh *iOS ↔
Windows* thì chưa ai đo. Và không phép đo nào chạm tới câu **hai con số có thật
sự đọc được bằng mắt không** — cùng cỡ, cùng chỗ, đủ lâu để so.

Bài đo là `interoperability/manual-ios.md`, mục *"iPhone ↔ Snappy (Windows) —
ghép đôi SAS, Ticket 06"*. Tám bước, và bước 7–8 (bền qua restart) là phần dễ bỏ
nhất — chỉ làm tới bước 6 thì "ghép đôi xong" và "ghép đôi xong rồi quên sau khi
restart" trông giống hệt nhau.
