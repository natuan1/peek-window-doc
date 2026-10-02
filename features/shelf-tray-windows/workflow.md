# Khay thẻ Shelf — luồng

Hai tiến trình, một chủ dữ liệu. Tiến trình nền giữ Shelf; tiến trình khay chỉ vẽ và
bắt cử chỉ ([ADR-0017](../../adr/0017-tinh-nang-nang-phan-shell-chay-trong-tien-trinh-con.md)).

## Mở khay

```
người dùng: menu khay → "Mở Shelf (n mục)"
nền:   khay đang mở?  → có: gửi "show"  (ống gãy vì khay đang đóng dở → mở khay mới)
                      → không: Snappy.exe --shelf-tray, gửi ảnh chụp Ngăn đang mở
con:   đọc ảnh chụp tới dòng "end", kiểm tệp trên đĩa NGAY Ở LUỒNG ĐỌC ỐNG
       (ổ mạng vừa rút làm File.Exists treo vài giây — không được treo luồng UI)
       → hiện khay trên màn hình có con trỏ, lên foreground
```

## Shelf đổi trong lúc khay mở

Một cú thả mới lên Mí, một Mục bị trần LRU đẩy ra, "Xoá sạch Shelf" — `Shelf.Changed`
bắn, nền gửi ảnh chụp mới, khay vẽ lại. Khay không bao giờ tự sửa danh sách của nó.

## Kéo ra

```
con:  nhấn chuột trên thẻ → DragDetect → gửi "dragging 1"
nền:  Mí.Suppressed = true   (Mí không nở ra vì lượt kéo của chính Snappy)
con:  DoDragDrop(CF_HDROP của các tệp còn trên đĩa, chỉ Copy)  — chặn tới khi nhả
      → gửi "dragging 0", rồi "dragged <hr> <effect> <id đã đi> <id mất tệp>"
nền:  Dropped → Touch từng Mục, nhật ký "Dragged N items out"
      Refused (⊘) / Cancelled / lỗi → mỗi nhánh một dòng nhật ký
      có Mục mất tệp → bong bóng "Chỉ kéo được X trong Y mục"
```

OLE chỉ bắt chuột cho cú nhấn rơi vào cửa sổ của chính luồng gọi `DoDragDrop`. Vì vậy cú
nhấn, cú kéo và mọi DLL kéo-thả đều thuộc tiến trình khay.

## Bỏ Mục, mở thư mục

- ✕ → "remove <id>" → nền `Shelf.Remove`. Bản trích xuất đang mở ở app khác thì Mục ở
  lại, kèm bong bóng. Mục đã rời Shelf trước cú bấm (khay vẽ theo ảnh chụp cũ) thì chỉ
  ghi một dòng Info.
- Nút thư mục → con chạy `explorer.exe /select,"<đường dẫn>"`. Tệp đã mất thì con gửi
  "folder-missing <id>" và nền báo bằng bong bóng.

## Đóng khay

- Esc, bấm ra ngoài (`WM_ACTIVATE` inactive, trừ lúc đang kéo) → con ẩn cửa sổ, thoát mã 0.
- Snappy thoát → nền giết con (`ChildProcess.Close`, không ném), ghi Info.
- Nền chết đột ngột → stdin của con đóng → con tự đóng khay.
- Con chết giữa chừng → nền ghi **Error** kèm stderr của con, và trả Mí về bình thường
  (`Suppressed = false`) để Mí không điếc mãi.
