# Lessons Learned — Live Trading v91_PartialTP

> Bài học tích lũy từ vận hành live EA `v91_PartialTP` trên tài khoản Vantage Cent, XAUUSD.sc, magic 777.

---

## Case Study: 30 Ngày (2026-04-20 → 2026-05-17)

### Số liệu thô (raw)

| Chỉ tiêu | Giá trị |
|----------|---------|
| Tổng deals | 11,677 |
| Win rate | 81.2% |
| Avg win | +40 USC |
| Avg loss | -212 USC |
| **Profit Factor** | **0.81** |
| Tổng P/L 30D | -45,208 USC (-$452) |
| Floating P/L | -32,927 USC (-$329) |
| **Net 30D** | **-78,135 USC (-$781)** |

### Phân tích outlier

**1 ngày duy nhất (2026-05-08) lỗ $713** = gần như toàn bộ thua lỗ 30 ngày.

Nếu loại bỏ ngày này: PF = ~1.05, tổng P/L = +$261 lãi.

→ **Bài học #1**: 1 ngày bất thường có thể xóa sạch lãi cả tháng.

---

## Bài Học #1 — Không Can Thiệp Thủ Công Vào Grid

### Tình huống 05-08
Trong giờ London Open (8h broker), volatility XAUUSD tăng đột biến. User can thiệp thủ công:
- Hedge tay với lot lớn (5 lot)
- Cắt lỗ tay với comment `[sl 4704.00]` → mất -6,845 USC trong 1 lệnh

**EA bị "loạn"** vì state không còn khớp với logic Partial TP — grid không thể "thoát" theo kế hoạch.

### Quy tắc
- **KHÔNG can thiệp tay khi EA đang chạy grid** — trừ khi có lý do extreme (broker issue, news risk lớn hơn risk DD)
- Nếu cần can thiệp: **TẮT HẲN EA** + đóng toàn bộ positions có kế hoạch, không nửa chừng
- Volatility cao (London Open, NFP, FOMC) là lúc tệ nhất để can thiệp

### Indicator phát hiện can thiệp tay trong log
- Comment có dạng `[sl XXXX]`, `[tp XXXX]`, `manual`
- Lot bất thường (> 0.5 trên tài khoản nhỏ với grid 0.04)
- Magic khác với magic EA
- Time-gap bất thường giữa các lệnh

---

## Bài Học #2 — Win Rate Cao ≠ Strategy Tốt

### Profit Factor mới là thật
- PF < 1.0 = strategy đang thua (kể cả win rate 95%)
- PF 1.0 - 1.3 = marginal, dễ bị slippage/commission ăn hết
- PF > 1.5 = strategy có lợi thế đáng kể

### Quy tắc đánh giá EA
1. Đếm **avg loss** vs **avg win**, không chỉ win rate
2. Lệnh thua trung bình lớn gấp 5x lệnh thắng → cần xem xét lại exit logic
3. Backtest dài (1-3 năm) + nhiều regime (trending, ranging, crisis) → PF ổn định mới là EA tốt

---

## Bài Học #3 — DD Hedge Là "Đóng Băng", Không Phải "Cứu"

### Cơ chế thực tế
- DD Hedge T1 mở khi DD = 20% → "đóng băng" mức lỗ tại thời điểm đó
- Nếu giá tiếp tục đi ngược → BUY lỗ thêm, SELL hedge có lời, nhưng **net thường vẫn lỗ thêm** (vì lot BUY > lot hedge)
- Nếu giá đảo chiều → hedge lỗ, BUY giảm lỗ

### Trap: T2/T3 cascade
Nếu enable T2 (40%), T3 (60%) → mỗi tier mở thêm 20 lệnh hedge → tổng lot hedge tăng nhanh → khi giá hồi cuối cùng, hedge tạo lỗ rất lớn.

**Đề xuất**: Disable T2/T3 (set 100%) cho đến khi backtest chứng minh có lợi. Đây là cấu hình hiện tại.

---

## Bài Học #4 — Margin Level An Toàn Không = Position An Toàn

### Trường hợp 16/05/2026
- Margin Level: 3,281% (an toàn cực cao)
- Floating P/L: -$329 (lỗ 26% balance)
- 104 positions, BE = 4668, giá hiện tại 4538 (cần +130 USD để hoà vốn)

→ **Vấn đề không phải margin call**, mà là **chôn vốn dài hạn**.

### Cảnh báo level thực tế
- Margin Level < 1000% trên grid trading = đáng lo
- Floating DD > 30% balance = nghiêm trọng — cần xem xét rút ra
- Margin Level < 200% = chuẩn bị giảm risk, không mở thêm

---

## Bài Học #5 — Logging Là Hạ Tầng, Không Phải Tính Năng

### Trước v9.22
- Không log decision → không biết tại sao DD Hedge fire
- Không log price ở mỗi event → không reconstruct được timeline
- Không log equity snapshot → không có equity curve

### Sau v9.22
- CSV structured ghi mọi event quan trọng
- Daily rotation + FILE_COMMON
- Python analyzer có sẵn

### Quy tắc cho EA tương lai
1. Log từ ngày đầu — không "thêm sau"
2. Schema chuẩn áp dụng cho mọi version (v92, v93, v94...)
3. Backtest cũng phải log để so sánh với live

---

## Bài Học #6 — Pair & Volatility Profile Cần Match Strategy

### XAUUSD vs grid:
- Volatility cao (ATR D1 ~ 100 USD)
- Trend mạnh, dài → khó cho grid không hedge
- StepPip cần ≥ 100-150 để spread/step tỷ lệ ≤ 30%

### AUD/CAD vs grid (cho v93_GridFX):
- Volatility thấp hơn (ATR D1 ~ 50-100 pip)
- Ranging nhiều hơn → favorable cho grid
- StepPip 8-10 đủ

### Quy tắc
- Grid + DD Hedge → có thể chạy XAU, nhưng risk cao
- Pure grid (không hedge) → ưu tiên pair sideway, volatility thấp
- Tránh grid trên pair có gap nhiều (JPY crosses, exotic)

---

## Bài Học #7 — Multi-Instance Cùng Magic = Xung Đột

### Phát hiện từ log 17/05
4 chart instance của v91_PartialTP đang attach (H1, H4, M15, M30) cùng XAUUSD.sc cùng Magic 777. Nếu chạy đồng thời cùng magic → tranh lệnh lẫn nhau.

### Quy tắc
- **1 EA, 1 chart, 1 magic** cho mỗi symbol
- Nếu muốn test nhiều biến thể → đổi Magic Number (ví dụ 777, 778, 779)
- Verify trước khi attach EA mới: tắt EA cũ hoặc remove khỏi chart

---

## Bài Học #8 — Tài Khoản Cent Là Trường Học Đắt Vừa Phải

### Lợi ích
- Cent = 1/100 standard → mất $1 thật chỉ là 100 USC trên dashboard
- Có thể test EA gần như "tâm lý live" mà không lỗ nhiều
- 26% DD trên 1,247 USD = $324 mất, đắt nhưng còn rẻ hơn $32,400 trên Standard

### Hạn chế
- Spread cent thường cao hơn standard 1.5-2x
- Slippage nhiều hơn
- Một số broker hạn chế strategy/leverage

### Quy tắc
- Tài khoản cent là **demo "có cảm xúc"** — không phải bằng demo, cũng không phải bằng standard
- Profitable trên cent với spread cent cao = lạc quan có lý cho standard
- Lỗ trên cent = chắc chắn lỗ trên standard

---

## Tổng Kết — Quy Tắc Vận Hành

| # | Quy tắc | Áp dụng |
|---|---------|---------|
| 1 | Không can thiệp tay khi EA chạy grid | Mọi lúc |
| 2 | Đánh giá EA bằng PF, không phải win rate | Khi review |
| 3 | T2/T3 DD Hedge disable cho đến khi backtest OK | Setting |
| 4 | Cảnh báo khi DD% > 30%, không phải khi margin call | Monitoring |
| 5 | Mọi EA mới phải có logger từ v1 | Coding |
| 6 | Match pair với strategy profile | Setup |
| 7 | 1 EA 1 magic 1 symbol | Setup |
| 8 | Test trên cent trước, scale lên standard sau | Roadmap |

---

## Tham Khảo

- `LOGGING_DESIGN.md` — chi tiết hệ logging v9.22
- `RSI_GUIDE.md` — RSI cho timing
- `MARKET_ANALYSIS_FRAMEWORK.md` — đa khung analysis
- `EA_KNOWLEDGE.md` — kiến trúc EA (existing)
- `TRADING_KNOWLEDGE.md` — kiến thức XAUUSD cơ bản (existing)
- `scripts/mt5_analysis/` — toolkit Python để phân tích
