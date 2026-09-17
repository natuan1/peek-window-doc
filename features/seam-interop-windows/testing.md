# Seam interop cho Windows — Kiểm thử

## 📊 Chiến lược

Hai tầng, và tầng dưới **không** thay được tầng trên:

| Tầng | Chứng minh | Giá |
|---|---|---|
| `interoperability/run.sh` | hai **tiến trình** độc lập sinh cùng byte | cần cả hai toolchain, vài giây |
| `FixtureTests` (xUnit) | cùng khẳng định, đỏ ngay tại dòng gõ sai một hằng số | mili giây, chạy trong `dotnet test` |

Hằng số kỳ vọng trong `FixtureTests` **không** chép từ bản Rust hay bản Swift —
chúng tính độc lập bằng `SHA256` của PowerShell từ chính công thức §43. Chép từ
một implementation khác thì cả hai cùng sai vẫn xanh.

## 🧪 Unit test

`apps/windows/tests/Snappy.Tests/FixtureTests.cs` (10 test):

- block đầu và block thứ hai khớp giá trị tính độc lập;
- cỡ buffer (1, 7, 31, 32, 33, 512) không đổi một byte nào;
- đọc từ giữa bằng đọc từ đầu (địa chỉ hoá được);
- seed khác cho byte khác;
- `Sha256Hex` khớp hai giá trị tính độc lập;
- hash của fixture **không bao giờ** là hash của chuỗi rỗng — đây là chỗ bắt
  được bug "cả hai phía cùng hash chuỗi rỗng", thứ mà phép so hash liên tiến
  trình không phân biệt nổi;
- `WriteTo` đúng số byte và đúng nội dung ở 1000 byte và ở hơn một chunk.

## 🔗 Kiểm liên tiến trình

`./interoperability/run.sh --full fixture` — đã chạy trọn cả sáu mốc của §43
ngày 2026-09-17: 1 byte · 1 KB · 1 MB · 100 MB · 1 GB · **4 GB**, `pass 8 ·
fail 0`. Mốc 4 GB là chỗ duy nhất ép được phép cộng offset vượt biên 32-bit.

## ✅ Hàng rào này có đo gì không — đã ép đỏ

Một hàng rào chưa bao giờ đỏ là một hàng rào chưa ai biết nó đo cái gì. Phép
đột biến đã chạy: đổi `WriteUInt64BigEndian` thành `WriteUInt64LittleEndian`
cho `seed`.

- `run.sh fixture` → cả **5** trường hợp `FAIL`, exit code `1`, và ghi chú in
  ra hai hash lệch nhau;
- `dotnet test` → **4/81** đỏ.

Hoàn nguyên thì cả hai xanh lại. Cửa `BỎQUA` cũng đã kiểm riêng:
`run.sh transfer` trên Windows in đúng một dòng `BỎQUA` kèm lý do và thoát `0`,
còn `run.sh khongcothat` thoát `2`.

## 🏗️ CI

Cặp này **chưa** nằm trong Jenkins, có chủ ý: agent CI không có `cargo`, và một
nửa suite chạy trong CI sẽ tạo cảm giác "interoperability xanh" trong khi nửa
quan trọng hơn chưa từng chạy. Cái Jenkins giữ là `dotnet build Snappy.slnx` —
`tools/Snappy.Harness` nằm trong solution nên CI là thứ phát hiện nó mục ra khi
`Snappy.Protocol` đổi.
