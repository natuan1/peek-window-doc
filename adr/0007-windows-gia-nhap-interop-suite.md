# ADR-0007: Windows gia nhập interop suite — Rust là oracle, phía vắng mặt phải nói ra

Date: 2026-09-17
Status: Accepted

> ⚠️ Kho `peekvn` có một dãy ADR **độc lập** với dãy này. `ADR-0007` ở đây không
> phải `ADR-0007` của `peekvn`.

## Context

Snappy là implementation ULTP thứ **ba**, sau `reference/rust-host` và
`apps/ios/PeekKit`. Hai bên kia đã có một chỗ chứng minh mình nói cùng giao
thức: `peekvn/interoperability/run.sh`, so hai **tiến trình** chứ không hai thư
viện. [Ticket 03 (#134)](https://github.com/natuan1/peekvn/issues/134) mở seam
đó cho Windows.

Việc thêm phía thứ ba làm lộ ra một giả định mà harness mang từ lúc chỉ có hai
phía: **mọi máy đều dựng được mọi phía**. Giả định đó sai ngay từ ngày đầu của
ba phía — máy dev macOS không chạy được exe Windows, máy dev Windows không có
`swift` — nhưng nó chưa bao giờ phải đúng, vì cả Rust lẫn Swift đều dựng được
trên macOS.

Câu hỏi phải trả lời: một cặp **không chạy được trên máy này** thì in ra cái gì?

## Decision

**1. Rust là oracle.** Swift và Windows đều so **với Rust**, không so với nhau.
Thiếu `cargo` thì `run.sh` dừng hẳn kèm câu lệnh cài, chứ không chạy tiếp với
một cặp khác.

**2. `BỎQUA` là một verdict thật, in ra kèm lý do.** Một phía không dựng được
thì cặp của nó **chưa đo**, và dòng tổng kết đếm riêng nó cùng câu
*"bỏ qua = CHƯA ĐO trên máy này, không phải đã đạt"*. Exit code vẫn chỉ phản
ánh `FAIL`.

**3. Không có phía thứ hai nào thì thoát `1`.** Một bảng toàn `BỎQUA` với exit
code 0 là đúng cái hàng rào xanh-mà-chưa-đo-gì mà thư mục này tồn tại để chặn.

**4. Generator fixture sống trong `Snappy.Protocol`, CLI sống trong một exe
riêng `tools/Snappy.Harness`.** Generator là hằng số của SPEC nên ở cùng chỗ
với hai bản kia (`ultp_host`, `ProtocolCore`); CLI phải là exe riêng vì
`Snappy.Core` là `WinExe`, không có console để in ra stdout.

## Consequences

### Tích cực

- Mọi ticket protocol sau của Windows có sẵn vòng kiểm chứng liên tiến trình —
  không phải dựng lại, chỉ thêm một cặp.
- Kết quả harness đọc được trên mọi máy: người đọc biết chính xác cái gì đã đo
  và cái gì chưa, thay vì thấy một bảng ngắn hơn bình thường mà không hiểu vì
  sao.
- Generator trong thư viện protocol **không** làm exe to thêm: đo ra
  `Snappy.exe` đúng 8.091.648 byte cả khi có lẫn khi không có tệp ấy — ILCompiler
  cắt sạch vì không dòng nào trong `Snappy.Core` gọi tới.

### Đánh đổi

- Không máy nào chạy được toàn bộ suite. Một lượt chạy xanh trên Windows **không
  nói gì** về phía Swift, và ngược lại. Đó là lý do `BỎQUA` phải to và ồn.
- Harness cần `cargo` trên máy Windows — một toolchain nữa phải cài
  (`winget install Rustlang.Rustup`).
- Hợp đồng dòng lệnh giờ có ba bản phải giữ đồng bộ bằng tay.

## Alternatives Considered

- **Bỏ qua trong im lặng khi thiếu toolchain.** Rẻ nhất, và sai nhất: một cặp
  chưa đo trông y hệt một cặp đã pass. Đúng họ lỗi số 1 của
  `peekvn/docs/bai-hoc.md`.
- **`FAIL` khi thiếu toolchain.** Đỏ mọi lượt chạy trên mọi máy dev vì một lý do
  không dính tới ticket nào. Một hàng rào đỏ vì lý do không liên quan sẽ được
  học cách bỏ qua, và lần nó đỏ thật thì không ai nhìn.
- **Thêm `fixture-sha256` vào chính `Snappy.exe`.** `WinExe` không có console;
  và nó kéo Velopack cùng khoá một-bản-đang-chạy vào một lệnh chỉ cần sinh byte.
- **Thêm cờ `--offset` cho phía Windows** để đo tính địa chỉ hoá liên tiến
  trình trực tiếp hơn. Bị bỏ: hợp đồng dòng lệnh phải giống nhau ở cả ba phía,
  và thêm một cờ chỉ một phía có là làm lệch chính cái mặt cắt đang được so.
  Tính chất ấy vẫn được đo, bằng trường hợp "512 byte đầu là tiền tố của 1 KB".

## Related

- [Seam interop cho Windows](../features/seam-interop-windows/overview.md)
- [ADR-0004 — Cấu trúc app Windows bốn project](0004-cau-truc-app-windows-bon-project.md)
- `peekvn/protocol/SPEC.md` §40, §43 — nguồn sự thật của hợp đồng generator
- `peekvn/docs/bai-hoc.md` mục 166, 167, 168

## Decision Log

- 2026-09-17: Accepted
