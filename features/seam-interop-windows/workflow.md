# Seam interop cho Windows — Quy trình

## 👤 Người dùng (ở đây là người viết mã) làm gì

Từ gốc repo `peekvn`, trong Git Bash:

```bash
./interoperability/run.sh fixture     # chỉ cặp fixture, ~5 giây sau lần dựng đầu
./interoperability/run.sh             # mọi cặp
./interoperability/run.sh --full fixture   # thêm mốc 100 MB · 1 GB · 4 GB, vài phút
```

Đọc bảng kết quả. Ba màu, ba nghĩa:

| | |
|---|---|
| `PASS` | đã đo, khớp |
| `FAIL` | đã đo, lệch — exit code khác 0 |
| `BỎQUA` | **chưa đo trên máy này**, kèm lý do. Không phải đã đạt. |

## 🖥️ Hệ thống làm gì

1. `run.sh` **dò** từng toolchain thay vì giả định đủ cả ba:
   - `cargo` **bắt buộc** — `rust-host` là oracle mà mọi phía so với. Thiếu là
     dừng hẳn, kèm câu lệnh cài.
   - `swift` có thì dựng `peek-interop`, không thì phía Swift vắng mặt.
   - đang chạy **trên Windows** và có `dotnet` thì dựng `snappy-interop`.
2. Không dựng được phía thứ hai nào → thoát `1`. Một bảng toàn `BỎQUA` với exit
   code 0 chính là hàng rào xanh-mà-chưa-đo-gì.
3. Mỗi cặp là một script trong `tests/`, ghi kết quả ra `$RESULTS` dạng
   `verdict|trường hợp|ghi chú`. `run.sh` đếm và tô màu.
4. Cặp `fixture` so **Rust với từng phía có mặt**, không so Swift với Windows.

## 📊 Các trường hợp của cặp `fixture`

Cho mỗi phía (`swift`, `windows`):

| Trường hợp | Chứng minh |
|---|---|
| SHA-256 ở 1 byte · 1 KB · 1 MB (`--full`: thêm 100 MB · 1 GB · 4 GB) | cùng byte ở mọi cỡ, kể cả vượt biên 32-bit |
| `cmp` 1 KB byte thô | hai bên **thật sự sinh byte** — hash khớp vẫn có thể là "cả hai cùng hash chuỗi rỗng" |
| 512 byte đầu là tiền tố của 1 KB | generator không có trạng thái nối tiếp, tức địa chỉ hoá được |

## 🔄 Trường hợp biên

- **Máy dev macOS**: phía Windows `BỎQUA` (exe Windows không chạy được), cặp
  Swift chạy đủ.
- **Máy dev Windows**: phía Swift `BỎQUA`, ba cặp cần Swift (`transfer`,
  `tls-handshake`, nửa sau của `vectors`) cũng `BỎQUA`.
- **Không có `cargo`**: dừng ngay, không chạy cặp nào.
- **Chỉ gọi một cặp không tồn tại**: thoát `2` kèm tên cặp đã gõ.
