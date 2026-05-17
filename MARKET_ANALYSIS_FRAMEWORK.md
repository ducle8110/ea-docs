# Market Analysis Framework — 3-Kịch-Bản từ Phân Tích Đa Khung

> Khuôn khổ ra quyết định trading khi phân tích XAUUSD đa khung thời gian (D1/H4/H1/M15) trên MT5.

---

## 1. Nguyên Tắc Cốt Lõi

Phân tích đa khung KHÔNG phải để tìm "câu trả lời đúng" — mà để **xác định mình đang ở vùng nào**, từ đó chọn 1 trong 3 kịch bản hành động:

| Kịch bản | Khi nào dùng | Đặc điểm |
|----------|-------------|----------|
| **1. BOUNCE** | Tại hỗ trợ + RSI oversold | Rủi ro nhỏ, R:R cao |
| **2. BREAKDOWN** | Phá hỗ trợ + volume tăng | Cần xác nhận, retest entry |
| **3. NO TRADE** | Vùng xám, không rõ hướng | An toàn nhất, chờ thêm tín hiệu |

---

## 2. Bộ Chỉ Báo Cần Thu Thập (Đa Khung)

Pull bằng `scripts/mt5_analysis/market_analyzer.py`:

| Indicator | D1 | H4 | H1 | M15 |
|-----------|----|----|----|----|
| Close | ✓ | ✓ | ✓ | ✓ |
| EMA20 | ✓ | ✓ | ✓ | ✓ |
| EMA50 | ✓ | ✓ | ✓ | ✓ |
| EMA200 | ✓ | ✓ | ✓ | ✓ |
| RSI14 | ✓ | ✓ | ✓ | ✓ |
| ATR14 | ✓ | ✓ | ✓ | ✓ |

**Bonus** từ D1 30 ngày gần nhất:
- 3 đỉnh cao nhất → vùng kháng cự
- 3 đáy thấp nhất → vùng hỗ trợ
- Range 30D, 90D

---

## 3. Phân Loại Trend Mỗi Khung

```
STRONG UP    : close > EMA20 > EMA50 > EMA200
UP           : close > EMA50 > EMA200
MIXED/SIDEWAY: cross-aligned
DOWN         : close < EMA50 < EMA200
STRONG DOWN  : close < EMA20 < EMA50 < EMA200
```

---

## 4. Đọc Tổ Hợp Đa Khung

### Pattern thường gặp

| D1 | H4 | H1 | M15 | Tình huống | Bias |
|----|----|----|----|-----------|------|
| UP | UP | UP | UP | Trend đồng thuận tăng | Strong BUY |
| UP | UP | DOWN | DOWN | Pullback trong uptrend | BUY khi giảm về hỗ trợ |
| UP | DOWN | DOWN | DOWN | Pullback sâu trong uptrend D1 | BUY tại vùng hỗ trợ lớn |
| MIXED | DOWN | STRONG DOWN | STRONG DOWN | Trend D1 yếu, có thể đảo | Cẩn thận, ưu tiên SELL ngắn |
| DOWN | DOWN | DOWN | DOWN | Trend đồng thuận giảm | Strong SELL |

### Quy luật chung
- **Khung lớn (D1) thắng khung nhỏ** khi mâu thuẫn — không trade ngược D1
- **Khung nhỏ xác định entry** — khi D1+H4 đồng ý, đợi H1+M15 cho timing chính xác
- **Đếm số khung "STRONG XYZ"** — càng nhiều khung cùng STRONG → trend càng mạnh

---

## 5. RSI Bias Theo Đa Khung

| Trạng thái | Khả năng diễn biến |
|------------|-------------------|
| RSI D1 < 40 + RSI H4 < 30 + RSI H1 < 30 | Đáy chính sắp tới — chuẩn bị BUY |
| RSI D1 > 60 + RSI H4 > 70 + RSI H1 > 70 | Đỉnh ngắn hạn — chuẩn bị SELL |
| RSI D1 ~50 + RSI H4 dao động | Sideway — grid hoạt động tốt |
| RSI H4 < 25 (cực hiếm) | Bounce kỹ thuật rất cao (>70% xác suất) |
| RSI H4 > 75 (cực hiếm) | Pullback kỹ thuật rất cao |

---

## 6. 3 Kịch Bản Chi Tiết

### 6.1 Kịch Bản 1 — BOUNCE (Mua tại hỗ trợ)

**Điều kiện kích hoạt:**
- Giá test vùng hỗ trợ chính (đáy 30D, EMA200 D1)
- RSI H4 < 30 (lý tưởng < 25)
- Xuất hiện nến đảo chiều H1 (pin bar, engulfing) + RSI H4 thoát >30

**Đặt lệnh:**
- Entry: tại hoặc gần hỗ trợ
- SL: dưới hỗ trợ ~10-20% ATR D1
- TP1: EMA20 H1 (mục tiêu gần)
- TP2: EMA20 H4 (mean reversion)
- TP3: EMA50 H4 hoặc resistance gần nhất

**R:R target**: ≥ 1:2 cho TP1, ≥ 1:4 cho TP3.

### 6.2 Kịch Bản 2 — BREAKDOWN (Bán khi phá hỗ trợ)

**Điều kiện kích hoạt:**
- Đóng nến H4 dưới hỗ trợ chính
- Volume tăng (so với 20 nến gần nhất)
- KHÔNG entry ngay — chờ retest

**Đặt lệnh:**
- Entry SELL: khi giá bật lên test lại vùng vừa phá
- SL: trên vùng đã phá ~50% ATR H4
- TP1: support tiếp theo (EMA200 D1 thường là target chính)
- TP2: vùng đáy 90D nếu trend mạnh

### 6.3 Kịch Bản 3 — NO TRADE (Đứng ngoài)

**Khi nào chọn:**
- Vùng giữa range, không gần S/R chính
- Đa khung mâu thuẫn (D1 up, H4 down)
- Volatility cao + spread cao (>50 pip)
- Sắp có tin lớn (NFP, FOMC, CPI)

**Hành động:**
- Đặt alert tại các mốc quan trọng
- Chờ giá xác nhận hướng (đóng nến quan trọng)
- Tiếp tục theo dõi nhưng không vào lệnh

> **Quy tắc vàng**: "Không có setup tốt = không vào lệnh" thường mang lại lợi nhuận dài hạn cao hơn việc cố vào lệnh ép.

---

## 7. Kết Hợp Với EA Grid (v91_PartialTP)

### Khi EA đang chạy
- **Bias D1 UP + giá pullback sâu** → để EA tiếp tục grid + manual không can thiệp
- **Bias D1 DOWN mạnh + RSI cực bán** → cân nhắc tắt EA (không mở grid mới), giữ positions
- **DD Hedge T1 fire + RSI H4 < 25** → khả năng giá bật → hedge có thể lỗ → KHÔNG tay sell thêm

### Sau khi đợt position đóng
- Dùng `v91_log_analyzer.py` xem snapshot khi DD cao nhất — RSI H4 lúc đó là bao nhiêu?
- Nếu RSI thường < 25 khi T1 fire → tinh chỉnh `DDHedge_DD_Tier1` cao hơn (vd 25%)
- Nếu RSI dao động trung tính khi T1 fire → ngưỡng 20% là phù hợp

---

## 8. Anti-Pattern Cần Tránh

1. **Bán chỉ vì RSI = 70** trong strong uptrend D1
2. **Mua chỉ vì RSI = 30** trong strong downtrend D1
3. **Vào lệnh trong vùng giữa** chỉ vì "có vẻ rẻ"
4. **Đặt SL quá chặt** (< 0.5 × ATR khung trade) — dễ bị quét
5. **TP quá xa** mà không chia lệnh — bỏ lỡ cơ hội lock lời
6. **Phớt lờ khung D1** vì khung nhỏ thấy "đẹp"
7. **Tay can thiệp khi EA đang chạy** — phá vỡ logic Partial TP

---

## 9. Code Tham Khảo

- `scripts/mt5_analysis/market_analyzer.py` — pull đa khung + classify trend
- `scripts/mt5_analysis/positions_inspector.py` — stress test khi vào thêm lệnh
- `RSI_GUIDE.md` — chi tiết về RSI
- `EA_KNOWLEDGE.md` — kiến trúc EA grid (đã có sẵn)
