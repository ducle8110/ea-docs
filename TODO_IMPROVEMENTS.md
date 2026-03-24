# TODO: Cải thiện GWTW EA

## 1. Partial TP ưu tiên lệnh âm nhiều nhất (PENDING)

### Vấn đề hiện tại
Logic Partial TP hiện tại chọn combo có **tổng profit cao nhất**. Điều này có thể không tối ưu khi có lệnh đang lỗ nặng cần được "cứu" sớm.

### Đề xuất
Thay đổi logic để ưu tiên combo chứa lệnh **âm sâu nhất** thay vì combo có tổng profit cao nhất.

### Ví dụ
```
Lệnh: BUY +$2, BUY +$0.3, SELL -$1.2, SELL -$0.5
Ngưỡng PartialTP = $0.5

Hiện tại: Chọn BUY($2) + BUY($0.3) + SELL(-$0.5) = $1.8 (tổng cao nhất)
Đề xuất:  Chọn BUY($2) + BUY($0.3) + SELL(-$1.2) = $1.1 (cứu lệnh -$1.2)
```

### Lợi ích
- Giải phóng lệnh đang lỗ nặng sớm hơn
- Giảm exposure/rủi ro
- Phù hợp khi thị trường trending mạnh

### Trạng thái
- [ ] Cần nghiên cứu thêm cách implement
- [ ] Test trên backtest để so sánh hiệu quả

---

## 2. Stop Loss per Position (PENDING)

### Vấn đề
Khi thị trường trending mạnh, các lệnh có thể lỗ rất nhiều mà không có giới hạn.

### Đề xuất
Thêm input `StopLossPip` để đặt SL cố định cho mỗi lệnh:
- BUY: SL = giá vào - StopLossPip
- SELL: SL = giá vào + StopLossPip

### Trạng thái
- [ ] Cần kiểm tra lại cách trade.Buy/Sell hoạt động với SL
- [ ] Test trên demo account

---

*Cập nhật lần cuối: 2026-01-25*
