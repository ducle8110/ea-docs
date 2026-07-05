# Múi giờ broker MT5 + Khung giờ vàng (XAUUSD) thực tế

> Knowledge document — **đọc trước** khi viết bất kỳ script nào phân tích timestamp từ MT5, hoặc khi setup time-of-day filter cho EA.
>
> Báo cáo phân tích chi tiết kèm chart: [VOLATILITY_REPORT_5Y.md](VOLATILITY_REPORT_5Y.md)

---

## 1. Múi giờ broker — quirk lớn nhất của MT5

### Vấn đề

`MetaTrader5` Python API trả về `time` là Unix timestamp (số giây), nhưng **giá trị này KHÔNG phải UTC**. Nó là **broker server clock đọc được at that bar/tick, lưu như Unix timestamp**.

Ví dụ với broker Vantage (UTC+3, DST đang active):
- Nến H1 đóng lúc real UTC 14:00 → broker clock đọc là 17:00 → MT5 trả Unix timestamp tương ứng với "17:00 UTC".
- Code naive `pd.to_datetime(time, unit="s", utc=True)` sẽ hiển thị nến là "17:00 UTC" → **sai +3 giờ**.
- Nếu sau đó convert sang VN bằng `+7`, sẽ ra "00:00 VN" — thực tế nến này thuộc về **21:00 VN**.

### Hệ quả nếu không sửa

- Tất cả phân tích "giờ nào volatile" lệch +offset giờ về tương lai.
- Time-of-day filter trong EA chạy sai khung giờ.
- Cross-reference với lịch tin (Forex Factory ghi giờ UTC chuẩn) sẽ lệch.

### Cách convert đúng

```python
import pandas as pd
from datetime import datetime, timezone

BROKER_OFFSET = 3  # Vantage trong DST. Ngoài DST = 2.

rates = mt5.copy_rates_range(symbol, mt5.TIMEFRAME_H1, start, end)
df = pd.DataFrame(rates)
df["broker_time"] = pd.to_datetime(df["time"], unit="s", utc=True)
df["utc_real"]    = df["broker_time"] - pd.Timedelta(hours=BROKER_OFFSET)
df["vn_time"]     = df["utc_real"].dt.tz_convert("Asia/Ho_Chi_Minh")
```

Quan trọng: **trừ offset trước khi convert sang VN**, không phải cộng vào.

### Cách dò offset chính xác

```python
import time
import MetaTrader5 as mt5

mt5.initialize()
tick = mt5.symbol_info_tick("XAUUSD.crp")
diff = tick.time - time.time()  # broker_clock_unix - real_utc_unix
offset_hours = round(diff / 3600)
# Lưu ý: chỉ chính xác khi market đang open (tick fresh < 5 phút).
```

Khi market đóng (weekend), tick.time là Friday close → diff lớn → không dùng được. Fallback bằng lịch DST (last Sun of March → last Sun of October là DST ở châu Âu/US).

Đã implement hàm `detect_broker_offset_hours()` trong `scripts/mt5_analysis/volatility_5y.py`.

---

## 2. Broker đang dùng

| Trường | Giá trị |
|--------|---------|
| Broker server | `VantageInternational-Demo` |
| Symbol XAUUSD | `XAUUSD.crp` |
| Múi giờ server | **UTC+3 trong DST** (cuối tháng 3 → cuối tháng 10) |
|                | **UTC+2 ngoài DST** (cuối tháng 10 → cuối tháng 3) |
| Daily candle rollover | Broker hour 00:00 (≈ real UTC 21:00 trong DST = **4h sáng VN**) |
| Weekly close | Friday broker hour 23:59 (≈ real UTC 20:59 trong DST = **3h59 sáng Sat VN**) |
| Weekly open | Sunday/Monday broker hour 01:00 (≈ real UTC 22:00 Sun = **5h sáng Monday VN**) |

DST switching theo lịch **châu Âu/Mỹ** (last Sunday of March / last Sunday of October), không theo VN.

---

## 3. Khung giờ vàng theo giờ Việt Nam (đã sửa timezone) — từ 5 năm dữ liệu

> Số liệu sau đây dựa trên **30.020 nến H1** của XAUUSD.crp từ 2021-04-25 đến 2026-05-22, sau khi đã convert đúng từ broker time sang VN time.

### Bảng giờ vàng

| Giờ VN | Avg range H1 (USD) | Avg range (%) | Phiên | Đánh giá |
|--------|-------------------|---------------|-------|----------|
| 19h | 12.7 | 0.49% | London-NY overlap | 🔥 **Vàng** |
| **20h** | **13.7** | **0.51%** | NY peak | 🔥 **Đỉnh ngày** |
| **21h** | **13.5** | **0.51%** | NY peak (US data) | 🔥 **Đỉnh ngày** |
| 22h | 10.1 | 0.36% | NY mid-session | 🔥 Cao |
| 23h | 7.7 | 0.28% | NY late | OK |
| 0h | 7.2 | 0.26% | NY tail | OK |
| 1h | 6.8 | 0.25% | NY đóng | Trung bình |
| 2h | 6.5 | 0.23% | Sau NY | Trung bình |
| 3h | 4.7 | 0.16% | Yên | Yên |
| **4h** | **1.2** | **0.02%** | **Broker rollover GAP** | ⚠️ Gap kỹ thuật, không phải giờ yên thật |
| 5h | 7.2 | 0.25% | Pre-Tokyo | Trung bình |
| 6h | 5.9 | 0.19% | Pre-Tokyo | Yên |
| 7h | 8.2 | 0.30% | Tokyo open | 🟡 Peak phụ |
| **8h** | **9.0** | **0.31%** | Tokyo chính | 🟡 **Peak phụ cao nhất Á** |
| 9h | 6.8 | 0.27% | Tokyo | Trung bình |
| 10h | 5.3 | 0.19% | Tokyo trưa | Yên |
| 11h | 5.6 | 0.19% | Tokyo trưa | Yên |
| 12h | 7.2 | 0.24% | Pre-London | Trung bình |
| 13h | 7.7 | 0.26% | Pre-London | Trung bình |
| 14h | 8.2 | 0.28% | London open | 🟡 Peak phụ |
| 15h | 7.6 | 0.25% | London | Trung bình |
| 16h | 6.7 | 0.23% | London | Trung bình |
| 17h | 6.9 | 0.27% | London | Trung bình |
| 18h | 7.5 | 0.28% | Pre-NY | Trung bình |

### 3 cụm giờ rõ rệt

```
PRIME (19h–22h VN)    ─── 10–14 USD/H1 ── London-NY overlap + NY peak
   ↑ Tin Mỹ (NFP, CPI, FOMC, GDP, Claims) thường ra 19h30–21h30 VN

SECONDARY (7h–9h VN)  ─── 8–9 USD/H1  ── Tokyo open
   ↑ Tin Á (PMI Trung Quốc, BoJ, Aussie)

SECONDARY (14h VN)    ─── 8.2 USD/H1   ── London open
   ↑ Tin Anh, EU
```

### Giờ AVOID

- **4h VN** — broker rollover, không có nến (≠ "thị trường yên").
- **10h–11h VN** — Tokyo trưa, biên độ thấp (~5 USD).
- **3h, 6h VN** — sau NY đóng, trước Tokyo (~5 USD).
- **Đầu tuần (Monday 5h–7h VN)** — gap weekend, spread rộng.
- **Friday 23h VN trở đi** — gần weekly close, thanh khoản giảm.

---

## 4. Lịch tin Mỹ → giờ VN (DST: tháng 3 → tháng 11)

| Loại tin | Giờ Mỹ ET | Giờ UTC | **Giờ VN (DST)** | **Giờ VN (winter)** |
|----------|-----------|---------|------------------|---------------------|
| NFP | 8:30 AM Fri 1st | 12:30 (DST) / 13:30 | **19:30 Fri** | **20:30 Fri** |
| CPI | 8:30 AM | 12:30 / 13:30 | **19:30** | **20:30** |
| Initial Jobless Claims | 8:30 AM Thu | 12:30 / 13:30 | **19:30 Thu** | **20:30 Thu** |
| Retail Sales | 8:30 AM | 12:30 / 13:30 | **19:30** | **20:30** |
| ISM Manufacturing | 10:00 AM Mon 1st | 14:00 / 15:00 | **21:00 Mon** | **22:00 Mon** |
| ISM Services | 10:00 AM | 14:00 / 15:00 | **21:00** | **22:00** |
| JOLTS | 10:00 AM | 14:00 / 15:00 | **21:00** | **22:00** |
| FOMC statement | 2:00 PM Wed | 18:00 / 19:00 | **01:00 Thu** | **02:00 Thu** |
| FOMC press conf (Powell) | 2:30 PM Wed | 18:30 / 19:30 | **01:30 Thu** | **02:30 Thu** |
| Powell speech (other) | thay đổi | thay đổi | thay đổi | thay đổi |

Đối chiếu lịch chính xác từng tuần: https://www.forexfactory.com/calendar (set timezone GMT+7).

---

## 5. Tổng quan biến động theo năm + xu hướng giá

Từ phân tích 5 năm:

| Năm | Avg range H1 (USD) | Avg range (%) | Giá vàng cuối năm |
|-----|--------------------|----------------|---------------------|
| 2021 | ~4 USD | ~0.21% | ~1830 USD |
| 2022 | ~4.5 USD | ~0.24% | ~1810 USD |
| 2023 | ~4 USD | ~0.21% | ~2070 USD |
| 2024 | ~7 USD | ~0.27% | ~2620 USD |
| 2025 | ~11 USD | ~0.36% | ~3700 USD |
| 2026 (5 tháng đầu) | ~27 USD | ~0.55% | ~4500 USD (giảm từ peak 5500) |

**Insight quan trọng:**
1. Biên độ USD **gấp ~5×** từ 2021 đến 2026 — phần lớn do giá vàng tăng.
2. Biên độ % cũng tăng (~0.21% → 0.55%) nhưng nhẹ hơn — tức là **vàng cũng đang volatile hơn về cấu trúc**, không chỉ về USD.
3. **Pattern peak theo giờ không đổi** qua 5 năm: vẫn là 19h–22h VN và 7h–9h VN.

---

## 6. Bài học cho EA / strategy

### Khi config EA

- **Step grid không cố định USD** — 30 USD năm 2021 = 1.5% giá, năm 2026 = 0.6% giá. Dùng **ATR(H1) × hệ số** hoặc **% giá**.
- **DD Hedge** phải chịu được ngày 700+ USD range (đã xảy ra 30/01/2026).
- **Time-of-day filter:**
  - Trade breakout: chỉ vào lệnh trong khung **19h–22h VN** hoặc **8h–9h VN**.
  - Trade range/grid: vẫn chạy nhưng giảm lot trong khung 10h–11h VN (Tokyo trưa) và 3h–6h VN.
  - Pause hoặc giảm lot: **30 phút trước/sau tin Mỹ high-impact** (NFP, CPI, FOMC).
- **Broker rollover 4h VN**: đừng tin tick/spread giờ này, có thể spread giãn 5–10× bình thường.

### Khi backtest

- **Backtest tối thiểu 2 năm** bao gồm cả giai đoạn giá thấp (2021–2023) lẫn cao (2024–2026).
- Backtest chỉ 2021–2023 → underestimate biên độ thật của 2026.
- Backtest chỉ 2026 → overestimate, EA có thể fail khi vàng quay lại regime ổn định.

### Khi đối chiếu log EA

- File log của EA (`v91_log_YYYYMMDD.csv`) đang dùng `TimeCurrent()` của broker → **giờ trong log là broker time, không phải VN time**.
- Để đối chiếu sự kiện log với lịch tin Forex Factory: trừ `BROKER_OFFSET` (3 trong DST, 2 ngoài) để ra UTC, hoặc trừ thêm để ra VN.

---

## 7. Reproduce phân tích

```bash
# Pull 5 năm H1 + sinh 9 charts (auto dò broker offset)
python scripts/mt5_analysis/volatility_5y.py

# Override offset (khi market đóng, auto-detect không chính xác)
python scripts/mt5_analysis/volatility_5y.py --broker-offset 3   # DST
python scripts/mt5_analysis/volatility_5y.py --broker-offset 2   # winter

# Custom: chỉ 3 năm, symbol khác
python scripts/mt5_analysis/volatility_5y.py --years 3 --symbol XAUUSD
```

**Outputs:**
- `backtest/cache/volatility_5y_h1.csv` — raw data 30k rows
- `backtest/cache/volatility_5y_summary.json` — số liệu tóm tắt
- `backtest/cache/charts/01–09_*.png` — 9 charts

**Report đã viết sẵn:** [VOLATILITY_REPORT_5Y.md](VOLATILITY_REPORT_5Y.md) (link tới các chart).

---

## 8. Tham khảo nhanh

- Báo cáo chi tiết kèm chart: [VOLATILITY_REPORT_5Y.md](VOLATILITY_REPORT_5Y.md)
- Báo cáo riêng 2026 (số liệu cũ — chỉ xem pattern): [VOLATILITY_REPORT_2026.md](VOLATILITY_REPORT_2026.md)
- Trading basic XAUUSD: [TRADING_KNOWLEDGE.md](TRADING_KNOWLEDGE.md) section 6
- Framework phân tích đa khung: [MARKET_ANALYSIS_FRAMEWORK.md](MARKET_ANALYSIS_FRAMEWORK.md)
- MT5 Python toolkit: [`scripts/mt5_analysis/README.md`](../scripts/mt5_analysis/README.md)

---

_Generated 2026-05-24. Cập nhật khi broker đổi server hoặc khi DST chuyển._
