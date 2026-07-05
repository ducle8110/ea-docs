# Báo cáo biến động XAUUSD 5 năm (H1) — có chart

**Dữ liệu nguồn:** MT5 đang chạy (`XAUUSD.crp` — broker `VantageInternational-Demo`)
**Khoảng phân tích:** 2021-04-25 → 2026-05-22 (~5 năm, 30.020 nến H1)
**Script:** `scripts/mt5_analysis/volatility_5y.py`
**Data raw:** `backtest/cache/volatility_5y_h1.csv`
**Charts:** `backtest/cache/charts/*.png`

> **Múi giờ:** Broker Vantage chạy **UTC+3 (DST)**. Script đã trừ 3h để về UTC thật, rồi cộng 7h sang **giờ Việt Nam**. Tất cả số liệu dưới đây đều theo giờ VN.
>
> _Cảnh báo bản trước: phiên bản đầu tiên của report đã NHẦM khi xử lý timestamp broker như UTC — toàn bộ giờ bị lệch +3 giờ. Báo cáo hiện tại đã sửa._

---

## 1. Tổng quan 5 năm

| Chỉ số | Giá trị |
|--------|---------|
| Số nến H1 | 30.020 |
| Avg range/nến H1 | **7.4 USD** |
| Median range | 5.1 USD |
| Max range trong 1 H1 | 393.95 USD (29/01/2026 21h VN) |
| Avg \|return\| mỗi H1 | ~0.27% |

**Lưu ý scale:** Giá vàng tăng từ ~1800 (2021) lên ~5500 (đầu 2026) → range USD bị méo theo giá. So sánh năm với năm cần dùng **range %** (chart 08, 09).

---

## 2. Chart 1 — Biến động trung bình H1 theo giờ VN

![avg range by hour](../backtest/cache/charts/01_avg_range_by_hour_vn.png)

### Đọc nhanh — pattern thực tế khớp với quan sát của trader

| Khung giờ VN | Avg range (USD) | Phiên thị trường |
|--------------|-----------------|------------------|
| **19h–22h (PEAK)** | **10–14 USD** | **London-NY overlap + NY peak** |
| 7h–9h | 7–9 USD | Tokyo open (peak phụ) |
| 13h–18h | 7–8 USD | Pre-London + London chính |
| 23h | 7.7 USD | NY giảm dần |
| 0h–2h | 6.5–7.2 USD | NY tail/đóng |
| 3h | 4.7 USD | NY close, gần như tan |
| 4h | 1.2 USD | **GAP broker** (chỉ 6 candle trong 5 năm — đây là giờ broker rollover daily candle, không phải "thị trường yên") |
| 5h–6h | 5.9–7.2 USD | Pre-Tokyo |
| 10h–12h | 5.3–7.2 USD | Trưa Tokyo (lull) |

### Tóm cứng

- **Giờ vàng:** 19h–22h VN. Đỉnh ở **20h VN (13.7 USD)** và **21h VN (13.5 USD)**.
- **Peak phụ:** 7h–9h VN (Tokyo open).
- **Yên nhất (sau loại gap broker):** 3h VN và 10h VN.

→ Khớp **chính xác** với quan sát "biến động mạnh 19h~23h VN" của trader.

---

## 3. Chart 2 — Heatmap Giờ × Năm

![heatmap hour year](../backtest/cache/charts/02_heatmap_hour_year.png)

- Cột 2026 đậm khắp các giờ, đỉnh đậm nhất ở **20h–21h VN**.
- Pattern peak ở 19h–22h và 7h–9h **lặp lại mọi năm** → đây là cấu trúc thị trường, không phải hiện tượng nhất thời của riêng 2026.
- Cột 2021–2023 màu nhạt vì giá vàng còn vùng 1800–2000 USD.

---

## 4. Chart 3 — Heatmap Giờ × Thứ trong tuần

![heatmap hour dow](../backtest/cache/charts/03_heatmap_hour_dow.png)

### Pattern nổi bật

- **Thứ 5 và Thứ 6 lúc 19h–21h VN** đậm nhất → trùng giờ tin Mỹ:
  - 19h30–20h30 VN = NFP/CPI/Claims release (thường 8:30 ET = 12:30 UTC vào DST, 13:30 UTC vào winter).
  - 21h VN = 14:00 UTC = data thứ cấp (FOMC statement, retail sales…).
- **Thứ 2 lúc 5h VN (12:00 UTC weekend gap)** = mở cửa tuần mới.
- 4h VN luôn nhạt — broker gap.

---

## 5. Chart 4 — Box plot phân phối biên độ theo giờ

![boxplot hour](../backtest/cache/charts/04_boxplot_hour.png)

- 19h–22h VN: box cao + whisker dài nhất → biên độ trung bình cao, nhưng cũng có nhiều outlier khi có tin.
- 3h và 10–11h VN: box thấp & hẹp.

---

## 6. Chart 5 — Biến động theo tháng vs Giá vàng

![monthly volatility](../backtest/cache/charts/05_monthly_volatility.png)

- 2 đường (xanh = avg range, cam = giá đóng tháng) gần như **đi cùng nhau**.
- 2021-2023: giá ~1700–2000, range H1 ~4–5 USD.
- 2024–2025: giá ~2500–3500, range ~6–15 USD.
- 2026-01 đến 2026-03: giá đỉnh ~5400, range H1 ~35 USD — đây là giai đoạn volatile nhất 5 năm.
- 2026-04 đến 2026-05: giá hạ về ~4500, range giảm xuống ~20 USD.

→ **Khi giá tăng, biên độ tăng gần như tuyến tính.** Backtest/tune EA cần phân biệt regime giá thấp vs cao.

---

## 7. Chart 6 — Top 20 ngày biến động lớn nhất 5 năm

![top volatile days](../backtest/cache/charts/06_top_volatile_days.png)

- **17/20 ngày top thuộc về 2026** — chứng tỏ 2026 là giai đoạn cực đoan.
- **Top 1: 30/01/2026 (Fri) — range 760 USD, -9.24%** — crash sau tuần FOMC.
- Cụm tuần đầu tháng 2/2026 chiếm 5 ngày trong top 20.
- Cụm 18–23/03/2026 (FOMC tháng 3): 4 ngày trong top 20.
- 2 ngày 2025 lọt top: **21/10/2025 (-5.30%)** và **29/12/2025 (-4.53%)**.

---

## 8. Chart 7 & 8 — So sánh từng năm

### USD tuyệt đối (chart 07)
![yearly compare](../backtest/cache/charts/07_yearly_compare.png)

### Chuẩn hoá theo % (chart 08)
![yearly compare pct](../backtest/cache/charts/08_yearly_compare_pct.png)

- Chart 07 (USD): các năm 2021–2024 dồn ở dưới, 2026 vọt cao — bị méo bởi giá.
- Chart 08 (%): các đường gần nhau hơn. **Hình dạng pattern peak ở 19h–21h giống hệt qua mọi năm** → quy luật theo phiên thị trường ổn định.

---

## 9. Chart 9 — Biến động % theo giờ (chuẩn hoá)

![avg range pct by hour](../backtest/cache/charts/09_avg_range_pct_by_hour.png)

- Theo %: biên độ vàng dao động **0.16% (10h VN) → 0.51% (20h VN)** — peak gấp **~3× giờ yên nhất**.
- 20h và 21h VN ngang nhau ~0.5%.

---

## 10. Tổng hợp khung giờ ưu tiên (đã đúng giờ VN)

```
                                  Avg %     Avg USD (2026)    Avg USD (5Y)
  19h–22h VN  (NY peak)           ~0.50%    ~33 USD           ~12 USD   ← PRIME
  07h–09h VN  (Tokyo open)        ~0.30%    ~25 USD           ~8 USD    ← phụ
  14h–18h VN  (London)            ~0.28%    ~25 USD           ~7 USD    ← phụ
  23h, 0h–2h  (NY late/đóng)      ~0.27%    ~20 USD           ~7 USD    ← giảm dần
  10h–13h VN  (trưa lull)         ~0.20%    ~17 USD           ~6 USD    ← yếu
  03h, 05h–06h VN                 ~0.22%    ~17 USD           ~6 USD    ← yếu
  04h VN                          —         —                  —        ← GAP broker
```

### Giải thích ngắn

- **19h–22h VN** là **London-NY overlap + NY peak**. Tin Mỹ ra 19h30, 21h, 21h30 VN.
- **07h–09h VN** là Tokyo open + tin Á (PMI Trung Quốc, BoJ).
- **04h VN** là broker rollover (gap kỹ thuật, không phải thị trường yên thực sự).

---

## 11. Bài học cho EA v91_PartialTP

1. **Step grid không nên cố định theo USD** — 30 USD ở 2021 = 1.5% giá, 30 USD ở 2026 = 0.6% giá. Cân nhắc step theo **ATR(H1) × hệ số** hoặc theo **% giá**.

2. **DD Hedge phải tính tới ngày extreme**: 5 năm có ngày range 760 USD. Tier hedge thưa quá sẽ không cover, dày quá sẽ fire trong phiên NY bình thường (range 30–40 USD).

3. **Time-of-day filter (giờ VN):**
   - Vào lệnh breakout: **19h–22h VN**.
   - Đóng/giảm lot trước đêm: **23h trở đi**.
   - Tránh broker rollover: **4h VN**.
   - Phòng tin Mỹ: 19h30 Thứ 5 (Claims), Thứ 6 đầu tháng (NFP).

4. **Weekend hedge:** Thứ 2 sáng VN có gap → bảo hiểm trước Friday close.

5. **Tránh backtest 1 năm rồi deploy:** biên độ USD 2026 gấp 5× của 2021. Backtest 2021–2023 sẽ underestimate cho 2026.

---

## 12. Reproduce

```bash
# Pull data + sinh 9 charts (tự dò broker offset)
python scripts/mt5_analysis/volatility_5y.py

# Override offset nếu cần
python scripts/mt5_analysis/volatility_5y.py --broker-offset 3

# Chỉ 3 năm
python scripts/mt5_analysis/volatility_5y.py --years 3

# Symbol khác
python scripts/mt5_analysis/volatility_5y.py --symbol XAUUSD
```

**Files sinh ra:**
- `backtest/cache/volatility_5y_h1.csv` (30k rows)
- `backtest/cache/volatility_5y_summary.json`
- `backtest/cache/charts/01–09_*.png`

---

_Generated 2026-05-24 dựa trên dữ liệu MT5 broker Vantage (UTC+3 DST). Phiên bản ngắn của report 2026 detail xem `VOLATILITY_REPORT_2026.md` (lưu ý: file đó vẫn dùng số liệu broker time chưa fix — bỏ qua mục giờ cụ thể, chỉ xem pattern)._
