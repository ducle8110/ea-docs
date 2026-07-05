# Báo cáo biến động XAUUSD 2026 (H1 & H4)

**Dữ liệu nguồn:** MT5 đang chạy (symbol `XAUUSD.crp`)
**Khoảng phân tích:** 2026-01-01 → 2026-05-24 (~5 tháng đầu năm)
**Số nến:** 2.302 H1 và 600 H4
**Script:** `scripts/mt5_analysis/volatility_2026.py`
**Output CSV:** `backtest/cache/volatility_2026_h1.csv`, `volatility_2026_h4.csv`

> **Lưu ý giờ:** Bảng dùng cả **UTC** (chuẩn) và **giờ Việt Nam (UTC+7)**. Giờ broker thường UTC+2/+3 sẽ lệch tương ứng.

---

## 1. Tổng quan biến động

| Khung | Avg range/nến | Median range | Max range | Avg |return| |
|-------|---------------|--------------|-----------|---------------|
| H1    | **27.0 USD**  | 20.3 USD     | 393.95 USD | 0.272% |
| H4    | **56.9 USD**  | 42.5 USD     | 447.16 USD | 0.560% |

Vàng năm 2026 cực kỳ biến động — biên độ trung bình H1 ~27 USD/giờ và có những phiên H4 chạy gần 450 USD trong 4 tiếng.

---

## 2. Biến động H1 theo giờ trong ngày (giờ Việt Nam)

Sắp xếp theo `avg_range_usd` giảm dần (số nến mẫu mỗi giờ ~100):

| Giờ VN | UTC | Avg range USD | Median | Max | Avg |return|% | Phiên thị trường |
|--------|-----|---------------|--------|------|-----------------|---|
| **00h** | 17 | **43.29** | 33.31 | 393.95 | 0.42% | NY chính (peak) |
| **23h** | 16 | **40.49** | 35.49 | 159.24 | 0.40% | London-NY overlap |
| **10h** | 03 | 36.52 | 25.52 | 193.00 | 0.39% | Tokyo chính |
| **08h** | 01 | 35.62 | 26.67 | 182.82 | 0.38% | Tokyo open |
| **01h** | 18 | 34.87 | 26.65 | 189.65 | 0.33% | NY trễ |
| **11h** | 04 | 34.64 | 24.58 | 200.58 | 0.36% | Tokyo → London pre |
| **22h** | 15 | 34.80 | 28.41 | 155.75 | 0.35% | NY open |
| 15h | 08 | 28.10 | 20.06 | 171.17 | 0.31% | London chính |
| 21h | 14 | 27.21 | 19.84 | 192.38 | 0.30% | US premarket / news |
| 09h | 02 | 26.60 | 20.26 | 143.91 | 0.27% | Tokyo |
| 03h | 20 | 26.10 | 20.17 | 202.60 | 0.23% | NY chiều |
| 17h | 10 | 25.46 | 19.13 | 133.72 | 0.21% | London |
| 16h | 09 | 25.28 | 19.41 | 147.08 | 0.27% | London early |
| 02h | 19 | 25.48 | 19.45 | 185.20 | 0.23% | NY chiều |
| 05h | 22 | 25.19 | 16.85 | 124.66 | 0.30% | NY close |
| 18h | 11 | 24.62 | 18.49 | 189.02 | 0.24% | London |
| 12h | 05 | 24.48 | 20.02 | 99.01 | 0.25% | giao phiên |
| 04h | 21 | 24.53 | 17.41 | 104.77 | 0.25% | NY trễ |
| 20h | 13 | 23.30 | 18.01 | 190.19 | 0.21% | US data window |
| 14h | 07 | 21.88 | 15.77 | 197.58 | 0.24% | London early |
| 19h | 12 | 21.46 | 17.26 | 121.41 | 0.18% | London → NY |
| 06h | 23 | 19.20 | 13.14 | 86.81 | 0.20% | đầu Á |
| **13h** | 06 | **17.21** | 14.54 | 110.04 | 0.15% | giờ lặng nhất |

### Kết luận H1

- **Khung giờ vàng (volatile nhất):** **22h–02h VN** (15–19 UTC) — trùng phiên London-NY overlap và NY chính. Trung bình ~36 USD/giờ, cao nhất tới ~393 USD/H1 candle.
- **Khung phụ:** **08h–11h VN** (1–4 UTC) — phiên Tokyo open, trung bình ~34 USD/giờ.
- **Khung yên tĩnh:** **12h–14h và 19h VN** — biên độ chỉ ~17–22 USD. Đây là giờ chuyển giao phiên London chiều/NY morning, dùng để quan sát, không vào lệnh breakout.

---

## 3. Biến động H4 theo phiên

H4 có 6 nến/ngày, mỗi nến đại diện 1 phiên:

| Nến H4 (UTC) | Nến H4 (giờ VN) | Avg range USD | Max range | Avg |return|% | Phiên |
|--------------|-----------------|---------------|-----------|-----------------|---|
| **16:00 UTC** | **23h–03h VN** | **75.72** | **447.16** | 0.72% | **NY chính** |
| 00:00 UTC | 07h–11h VN | 60.25 | 292.79 | 0.62% | Tokyo |
| 08:00 UTC | 15h–19h VN | 53.23 | 328.37 | 0.54% | London chính |
| 12:00 UTC | 19h–23h VN | 52.65 | 268.80 | 0.45% | London → NY |
| 04:00 UTC | 11h–15h VN | 51.02 | 274.50 | 0.53% | Tokyo → London |
| 20:00 UTC | 03h–07h VN | 48.41 | 256.56 | 0.50% | NY close |

### Kết luận H4

- **Phiên NY (23h–03h VN)** chạy mạnh nhất — biên độ trung bình **75.7 USD/4h**, gấp **1.5× các phiên khác**. Đây là phiên có tin Mỹ, FOMC, NFP, CPI.
- Các phiên còn lại biên độ khá đồng đều (~50–60 USD/4h).
- Phiên NY close (03h–07h VN) và phiên đầu Á yếu nhất, phù hợp đóng vị thế swing hoặc nghỉ.

---

## 4. Top 10 nến H1 biến động mạnh nhất

| Time (UTC) | Time (VN) | Open | High | Low | Close | Range USD | % | Hướng |
|------------|-----------|------|------|-----|-------|-----------|---|-------|
| 2026-01-29 17:00 | 30/01 00:00 | 5496.49 | 5496.49 | 5102.54 | 5182.46 | **393.95** | -5.71% | ⬇ |
| 2026-01-30 20:00 | 31/01 03:00 | 4849.69 | 4893.87 | 4691.27 | 4879.93 | 202.60 | +0.62% | ⬆ |
| 2026-01-30 04:00 | 30/01 11:00 | 5300.89 | 5312.80 | 5112.22 | 5224.84 | 200.58 | -1.44% | ⬇ |
| 2026-02-02 07:00 | 02/02 14:00 | 4679.75 | 4696.07 | 4498.49 | 4549.90 | 197.58 | -2.78% | ⬇ |
| 2026-02-02 03:00 | 02/02 10:00 | 4766.74 | 4784.64 | 4591.64 | 4737.64 | 193.00 | -0.61% | ⬇ |
| 2026-03-23 14:00 | 23/03 21:00 | 4270.56 | 4462.44 | 4270.06 | 4377.89 | 192.38 | +2.51% | ⬆ |
| 2026-03-03 13:00 | 03/03 20:00 | 5247.33 | 5264.06 | 5073.87 | 5141.31 | 190.19 | -2.02% | ⬇ |
| 2026-02-12 18:00 | 13/02 01:00 | 5060.19 | 5068.22 | 4878.57 | 4953.58 | 189.65 | -2.11% | ⬇ |
| 2026-01-30 11:00 | 30/01 18:00 | 5117.17 | 5131.81 | 4942.79 | 5013.63 | 189.02 | -2.02% | ⬇ |
| 2026-01-30 19:00 | 31/01 02:00 | 4985.46 | 4986.12 | 4800.92 | 4849.58 | 185.20 | -2.73% | ⬇ |

## 5. Top 10 nến H4 biến động mạnh nhất

| Time (UTC) | Time (VN) | Open | High | Low | Close | Range USD | % | Hướng |
|------------|-----------|------|------|-----|-------|-----------|---|-------|
| 2026-01-29 16:00 | 29/01 23h–30/01 03h | 5510.43 | 5549.70 | 5102.54 | 5335.24 | **447.16** | -3.18% | ⬇ |
| 2026-02-02 08:00 | 02/02 15h–19h | 4549.88 | 4731.14 | 4402.77 | 4719.48 | 328.37 | +3.73% | ⬆ |
| 2026-01-30 16:00 | 30/01 23h–31/01 03h | 5019.49 | 5113.38 | 4800.92 | 4849.58 | 312.46 | -3.39% | ⬇ |
| 2026-02-02 00:00 | 02/02 07h–11h | 4809.61 | 4884.43 | 4591.64 | 4737.64 | 292.79 | -1.50% | ⬇ |
| 2026-03-23 08:00 | 23/03 15h–19h | 4367.49 | 4380.13 | 4098.82 | 4251.11 | 281.31 | -2.67% | ⬇ |
| 2026-02-02 04:00 | 02/02 11h–15h | 4737.64 | 4772.99 | 4498.49 | 4549.90 | 274.50 | -3.96% | ⬇ |
| 2026-03-23 12:00 | 23/03 19h–23h | 4251.15 | 4492.56 | 4223.76 | 4480.82 | 268.80 | +5.40% | ⬆ |
| 2026-01-30 08:00 | 30/01 15h–19h | 5184.82 | 5205.45 | 4942.79 | 5013.63 | 262.66 | -3.30% | ⬇ |
| 2026-01-30 20:00 | 31/01 03h–07h | 4849.69 | 4947.83 | 4691.27 | 4885.50 | 256.56 | +0.74% | ⬆ |
| 2026-03-23 00:00 | 23/03 07h–11h | 4473.01 | 4536.25 | 4319.91 | 4354.25 | 216.34 | -2.66% | ⬇ |

---

## 6. Top 15 ngày biến động lớn nhất

| Ngày | Thứ | Range D1 (USD) | % | Ghi chú khả năng tin tức |
|------|-----|----------------|---|--------------------------|
| **2026-01-30** | Friday | **759.63** | **-9.24%** | Tiếp diễn FOMC 28/01 + PCE (Fri cuối tháng) — crash flash đỉnh năm |
| 2026-01-29 | Thursday | 496.01 | -0.70% | Sau FOMC 28/01 + GDP Q4 (thường Thu 8:30am ET) |
| 2026-02-02 | Monday | 481.66 | -3.14% | ISM Manufacturing PMI (đầu tháng) + tiếp dư âm FOMC |
| 2026-03-23 | Monday | 437.43 | -1.48% | Sau FOMC 18/03 + PMI flash (S&P Global PMI thường ~21–23) |
| 2026-03-03 | Tuesday | 383.76 | -4.40% | ISM Services + JOLTS |
| 2026-03-19 | Thursday | 364.63 | -3.54% | Sau FOMC 18/03 + Initial Jobless Claims |
| 2026-02-03 | Tuesday | 327.92 | +5.88% | JOLTS + Factory Orders + hồi sau crash |
| 2026-02-06 | Friday | 316.31 | +3.61% | **NFP tháng 1** (thứ Sáu đầu tháng) |
| 2026-02-05 | Thursday | 264.01 | -3.81% | Initial Jobless Claims, Unit Labor Cost |
| 2026-01-28 | Wednesday | 260.16 | +4.59% | **FOMC announcement + Powell press conf** |
| 2026-03-20 | Friday | 258.41 | -3.41% | Quad witching (3rd Fri tháng 3) + dư âm FOMC |
| 2026-04-02 | Thursday | 246.26 | -1.72% | ISM Services + Initial Claims |
| 2026-02-04 | Wednesday | 238.77 | +0.38% | ADP Employment + ISM Services |
| 2026-02-12 | Thursday | 221.35 | -3.23% | **CPI Mỹ tháng 1** (giữa tháng) |
| 2026-03-18 | Wednesday | 209.37 | -3.74% | **FOMC announcement tháng 3** |

> **Lưu ý quan trọng về phần "tin tức":** Cột "Ghi chú" là **suy đoán dựa trên pattern lịch tin định kỳ** (FOMC luôn Wed, NFP luôn Fri đầu tháng, CPI giữa tháng, PCE cuối tháng…). Để biết **chính xác tin gì high-impact giờ nào**, đối chiếu với:
> - Forex Factory: https://www.forexfactory.com/calendar
> - Investing.com economic calendar
> - Trading Economics calendar

---

## 7. Pattern key — tóm gọn cho trade

### Khung giờ ưu tiên (theo giờ Việt Nam UTC+7)

```
22h ─┬─ NY OPEN — biến động bật mạnh nhất
23h │
00h ┼─ NY peak (avg 43 USD/H1) — TOP volatile hour
01h │
02h ─┴─ NY chiều — vẫn cao
                              ← GIỜ ƯU TIÊN VÀO LỆNH
08h ─┬─ Tokyo open — biến động phụ
09h │
10h ┼─ Tokyo chính (avg 36 USD/H1)
11h ─┴─ Tokyo → pre-London
                              ← KHUNG PHỤ, tốt cho swing/scalp Á

12h-14h và 19h — giờ chuyển phiên, biên độ thấp → AVOID
```

### Phiên H4 ưu tiên

- **16:00 UTC (23h–03h VN)** — phiên NY: range trung bình **75 USD/4h**, gấp 1.5× phiên khác. Bắt buộc bám phiên này nếu cần entry breakout/news.
- Các phiên còn lại: ~50–55 USD/4h, ngang nhau.

### Thứ trong tuần theo data

- **Thứ 5 và thứ 6** chiếm 6/15 top biggest range days → tin Mỹ tập trung cuối tuần.
- **Thứ 2** xuất hiện 2 lần với biên độ rất lớn (>430 USD) — gap weekend + ISM/PMI đầu tháng.
- **Thứ 4** xuất hiện 3 lần — FOMC + ADP + ISM Services.

### Biến động bất thường để watchlist

- **Cuối tháng 1 đến đầu tháng 2 2026:** crash flash từ ~5500 xuống ~4400 (range cả tháng 1100 USD = ~20%). Đây là trigger quan trọng để xem xét quản lý rủi ro cho EA grid.
- **Giữa tháng 3 (18–23/03):** chuỗi 4 ngày range >200 USD liên tiếp xung quanh FOMC.

---

## 8. Hệ quả cho EA v91_PartialTP

Dựa trên pattern này, một số gợi ý:

1. **DD Hedge tier**: với H4 max range 447 USD, các tier hedge cần đủ rộng để không fire nhầm vào nhiễu của một phiên NY bình thường (~75 USD).
2. **Virtual pending step**: vào giờ 22h–02h VN, step nên rộng hơn (≥ ATR H1) để tránh fire liên tục. Vào giờ 12h–14h VN có thể siết chặt.
3. **Weekend hedge**: Friday VN (rạng sáng Sat UTC) thường biên độ lớn — đóng sớm hoặc bảo hiểm trước thứ 6 NY.
4. **Avoid news**: 3 ngày sau FOMC, đầu tháng Mon-Fri (ISM/NFP), giữa tháng (CPI) — flag để tạm dừng hoặc hạ lot.

---

## 9. Phụ lục — Cách reproduce

```bash
# 1. Pull data + tạo CSV/JSON
python scripts/mt5_analysis/volatility_2026.py

# 2. Tự phân tích bằng pandas
python -c "
import pandas as pd
df = pd.read_csv('backtest/cache/volatility_2026_h1.csv', parse_dates=['time'])
df['time_vn'] = df['time'].dt.tz_convert('Asia/Ho_Chi_Minh')
# ... query tuỳ ý
"
```

**File data lưu:**
- `backtest/cache/volatility_2026_h1.csv` (2.302 nến)
- `backtest/cache/volatility_2026_h4.csv` (600 nến)
- `backtest/cache/volatility_2026_summary.json`

---

_Generated 2026-05-24 — dựa trên dữ liệu MT5 broker (`XAUUSD.crp`)._
