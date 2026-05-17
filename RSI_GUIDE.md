# RSI (Relative Strength Index) — Hướng Dẫn Sử Dụng

> Áp dụng cho XAUUSD trên MT5, kết hợp với grid trading (v91_PartialTP) hoặc manual.

---

## 1. RSI là gì?

RSI là **chỉ báo dao động động lượng** (momentum oscillator) do J. Welles Wilder phát triển năm 1978. Đo lường **tốc độ và mức độ thay đổi giá** trong một khoảng thời gian (mặc định 14 chu kỳ), kết quả ở thang **0 - 100**.

### Công thức
```
RSI = 100 - (100 / (1 + RS))
RS  = Average Gain / Average Loss (N kỳ)
```

Wilder dùng EMA smoothing với `alpha = 1/period`. Code Python tham khảo: `scripts/mt5_analysis/market_analyzer.py::rsi()`.

---

## 2. Các Ngưỡng Quan Trọng

| Vùng | RSI | Ý nghĩa |
|------|-----|---------|
| Quá mua | > 70 | Có thể đảo chiều giảm |
| Trung tính | 30 - 70 | Xu hướng đang vận động |
| Quá bán | < 30 | Có thể đảo chiều tăng |
| Cực đoan | > 80 hoặc < 20 | Tín hiệu mạnh hơn — rất hiếm |
| Đường trung tâm | 50 | Ranh giới uptrend/downtrend |

---

## 3. Các Tín Hiệu Chính

### 3.1 Quá mua / Quá bán (cơ bản)
- RSI > 70 + giá tại kháng cự → cân nhắc SELL
- RSI < 30 + giá tại hỗ trợ → cân nhắc BUY

⚠️ **Trap phổ biến**: trong trend mạnh, RSI có thể nằm dưới 30 (hoặc trên 70) suốt nhiều ngày. Không bán chỉ vì RSI = 70 trong uptrend mạnh.

### 3.2 Phân kỳ (Divergence) — tín hiệu mạnh nhất
- **Bullish divergence**: giá tạo đáy thấp hơn nhưng RSI tạo đáy cao hơn → tín hiệu tăng sắp tới
- **Bearish divergence**: giá tạo đỉnh cao hơn nhưng RSI tạo đỉnh thấp hơn → tín hiệu giảm sắp tới

### 3.3 Đường 50
- RSI cắt lên 50 từ dưới → xu hướng tăng
- RSI cắt xuống 50 từ trên → xu hướng giảm

### 3.4 Failure Swing (Wilder gốc)
- RSI vượt 70 → giảm → tăng lại nhưng không qua 70 → giảm phá đáy gần → tín hiệu SELL
- Ngược lại cho BUY

---

## 4. Cài Đặt Theo Trading Style

| Style | Period | Lý do |
|-------|--------|-------|
| Scalping | RSI 7 | Phản ứng nhanh, nhiều tín hiệu |
| Day trading | RSI 14 | Mặc định Wilder, cân bằng |
| Swing trading | RSI 21-25 | Mượt hơn, ít nhiễu |

---

## 5. RSI Đa Khung (Multi-Timeframe RSI)

Pattern hiệu quả: **RSI khung lớn xác định bias, RSI khung nhỏ xác định entry**.

| Khung | Vai trò |
|-------|---------|
| D1 RSI | Xu hướng dài hạn — không trade ngược |
| H4 RSI | Bias chính — chờ extremes (>70 hoặc <30) |
| H1 RSI | Confirm timing |
| M15 RSI | Entry trigger |

**Ví dụ setup BUY:**
- D1 RSI > 50 (uptrend)
- H4 RSI < 35 (pullback đủ sâu)
- H1 RSI cắt lên 30 (đảo chiều ngắn hạn)
- M15 RSI > 50 (entry trigger)

---

## 6. RSI cho Grid Trading (v91_PartialTP)

Grid là chiến lược **không định hướng** (non-directional), thường KHÔNG dùng RSI làm entry chính. Nhưng RSI vẫn hữu ích như:

### 6.1 Filter — tạm dừng grid khi RSI cực đoan
- RSI H4 > 80 hoặc < 20 → giá có thể tiếp tục mạnh → grid bên ngược sẽ chịu nhiều DD
- Cân nhắc tắt EA hoặc đặt MaxPerSide thấp hơn trong giai đoạn này

### 6.2 Timing DD Hedge
- Khi DD Hedge T1 fire, nếu RSI H4 < 25 + giá tại hỗ trợ → khả năng giá bật cao → hedge có thể bị lỗ khi giá hồi
- Ngược lại: DD T1 fire khi RSI H4 trung tính → hedge có khả năng bảo vệ tốt hơn

### 6.3 Confirm "thoát kẹt"
- Portfolio LONG bị DD lớn → RSI D1 < 30 + bullish divergence H4 → setup tốt để chờ Partial TP hoạt động khi giá hồi

---

## 7. Hạn Chế Của RSI

1. **Trending market**: cho nhiều tín hiệu giả khi xu hướng mạnh
2. **Sideway market**: hoạt động tốt nhất — nhưng cũng có nhiều "fake breakout"
3. **Lagging indicator**: RSI dựa trên giá quá khứ → không dự báo chính xác đỉnh/đáy
4. **Cần kết hợp**: tự RSI không đủ — phải combo với hỗ trợ/kháng cự, EMA, hoặc price action

---

## 8. Combo Phổ Biến Hiệu Quả

| Combo | Use case |
|-------|----------|
| RSI + Hỗ trợ/Kháng cự | Xác nhận điểm đảo chiều |
| RSI + EMA | Filter trend trước khi nghe RSI |
| RSI + Bollinger Bands | Mean reversion + extreme |
| RSI + Volume | Confirm breakout |
| RSI Multi-TF | Đa lớp confirm |

---

## 9. Tham Khảo Code

- Python: `scripts/mt5_analysis/market_analyzer.py` — tính RSI14 đa khung từ MT5 live
- MQL5: chưa tích hợp vào EA (grid không cần RSI làm entry)
- Backtest: dùng D1/H4 RSI làm filter khi backtest EA — cân nhắc input bool `EnableRSIFilter`
