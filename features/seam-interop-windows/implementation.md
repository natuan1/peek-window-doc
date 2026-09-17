# Seam interop cho Windows — Implementation

## 🏗️ Kiến trúc

```
interoperability/run.sh          dò toolchain, dựng từng phía, đếm verdict
  ├─ lib.sh                      emit() + require_side() dùng chung
  └─ tests/fixture.sh            so Rust với từng phía có mặt

reference/rust-host              oracle — examples/interop
apps/ios/PeekKit                 peek-interop
apps/windows
  ├─ src/Snappy.Protocol/Fixture.cs     generator SPEC §43
  └─ tools/Snappy.Harness/              console exe `snappy-interop`
```

## 📁 Tệp chính

| Tệp | Vai trò |
|---|---|
| `apps/windows/src/Snappy.Protocol/Fixture.cs` | `FillAt` / `Sha256Hex` / `WriteTo` — nửa Windows của hợp đồng §43 |
| `apps/windows/tools/Snappy.Harness/Program.cs` | mặt cắt dòng lệnh; `AssemblyName` = `snappy-interop` |
| `apps/windows/tests/Snappy.Tests/FixtureTests.cs` | cùng khẳng định, ở dạng chạy trong mili giây |
| `interoperability/lib.sh` | `emit`, `require_side` |
| `interoperability/tests/fixture.sh` | `compare_with_rust <phía> <binary>` |
| `.gitattributes` (gốc `peekvn`) | ghim `eol=lf` cho `*.sh` và vector mật mã |

## 🔑 Ba quyết định nằm trong mã

**Generator ở `Snappy.Protocol`, không ở `Snappy.Harness`.** Hai phía kia cũng
đặt nó trong thư viện protocol (`ultp_host`, `ProtocolCore`): nó là một hằng số
của SPEC, không phải tiện ích của công cụ. Giá phải trả đáng lẽ là vài KB mã
chỉ-dùng-cho-test đi vào exe người dùng — **đo ra là 0**: publish Native AOT có
và không có tệp ấy đều cho `Snappy.exe` đúng 8.091.648 byte, vì `Snappy.Core`
không gọi tới nên ILCompiler cắt sạch.

**Exe riêng, không phải một cờ của `Snappy.exe`.** `Snappy.Core` là `WinExe` —
app chạy nền, không có console, nên không có chỗ nào in ra stdout cho harness
đọc. Nhét một nhánh CLI vào đó còn kéo theo Velopack và khoá
một-bản-đang-chạy vào một lệnh chỉ cần sinh byte.

**Byte thô ra stdout đi qua `Console.OpenStandardOutput()`, không qua
`Console.Out`.** `Console.Out` là `TextWriter` có encoder và trên Windows có thể
chèn BOM. Chẩn đoán ra stderr thì ngược lại: ghi UTF-8 thẳng, vì
`Console.Error` mã hoá theo code page console (437) và biến mọi dấu tiếng Việt
thành ký tự rác. Không đụng `Console.OutputEncoding` — setter đó áp cho cả
stdout, tức chạm đúng đường phải giữ sạch từng byte.

## 🔌 Hợp đồng dòng lệnh

Giống nhau ở cả ba phía:

```text
<interop> fixture-sha256 --seed <u64> --len <u64>    → hex thường, không xuống dòng
<interop> fixture-bytes  --seed <u64> --len <u64>    → byte thô ra stdout
```

Một khác biệt có chủ ý: phía Windows **thoát 2** khi một cờ có mặt mà thiếu giá
trị hoặc không đọc được thành số, trong khi Rust và Swift lặng lẽ rơi về mặc
định. `--len abc` hoá thành `--len 0` là một fixture rỗng đem so với một fixture
rỗng — hai hash khớp nhau trong khi chưa đo gì. Harness không bao giờ truyền
rác nên khác biệt này không đổi kết quả cặp nào.

## 🧮 Hợp đồng generator

```text
block(i) = SHA-256( u64_be(seed) ‖ u64_be(i) )      32 byte
byte(n)  = block(n / 32)[n % 32]
```

Byte tại offset `n` phụ thuộc **duy nhất** vào `(seed, n)` — không trạng thái
nối tiếp, không phụ thuộc byte order, không phụ thuộc cỡ buffer. Ba chỗ mà một
PRNG nhanh hơn sẽ làm ba ngôn ngữ lệch nhau, và lệch theo cách chỉ lộ ra ở tệp
lớn. Giá phải trả là tốc độ: SHA-256 chạy khoảng 1–2 GB/s.
