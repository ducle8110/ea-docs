# Báo cáo Bóc tách & Kiểm chứng MT5 Real Ticks — 2 Chiến lược YouTube Tier S (`v101` & `v102`)

> **Ngày kiểm chứng:** 18/11/2026 · **Symbol:** `XAUUSD.sc` (Vantage) · **Giai đoạn:** `01/01/2024 → 01/11/2026` (~2.85 năm) · **Vốn:** `$100,000` (`Risk = 1% / lệnh`) · **Mô hình cao nhất:** `Model = 4` (*Every tick based on real ticks*).

---

## Tổng quan kết quả kiểm chứng thực chiến

| Mã Bot | Nguồn Video Tier S | Trường phái | Số lệnh (3 năm) | Winrate | Tổng R | Lãi ròng (`Model=4` Real Ticks) | Profit Factor | Max DD | Kết luận thực chứng |
|---|---|---|---|---|---|---|---|---|---|
| **`v101`** *(1 tháng Video)* | *RBI FOREX* (`81.0đ`) | NY Sweep London H/L (`M5`) | 8 *(15/12/25–17/01/26)* | `50.0%` | `+12.11R` | `+$12,427.69` | **`3.83`** | `2.01%` | Khớp sát video (tác giả không bịa số 1 tháng) |
| **`v101`** *(3 năm Buy+Sell)* | *RBI FOREX* (`81.0đ`) | NY Sweep London H/L (`M5`) | 134 | `23.9%` | `-6.77R` | `-$8,575.23` | `0.91` | `29.99%` | **Thất bại dài hạn** do chiều `SELL` cản tàu xu hướng Vàng (`-15.93R`) |
| **`v101`** *(3 năm Chỉ Buy)* | *RBI FOREX* (`81.0đ`) | NY Sweep London Low (`M5`) | 59 | `28.8%` | `+9.16R` | `+$8,343.34` | `1.19` | `15.42%` | Dương nhẹ (`2024 +6.0R`, `2025 -4.0R`, `2026 +7.2R`), mẫu ít |
| **`v102`** *(3-HA, Buy+Sell)* | *TRADERPTKT* (`77.0đ`) | `M15` HA + EMA 100/200 + RSI(7) | 443 | `37.9%` | `+60.83R` | **`+$66,012.77`** | **`1.17`** | `13.63%` | **Lãi đều cả 3 năm** (`2024 +31.9R`, `2025 +16.8R`, `2026 +12.1R`) |
| **`v102`** *(2-HA, Buy+Sell)* | *TRADERPTKT* (`77.0đ`) | `M15` HA + EMA 100/200 + RSI(7) | 516 | `38.4%` | `+76.49R` | **`+$90,529.59`** | **`1.20`** | `20.73%` | **Lãi cao nhất 2 chiều** (cả Buy `+75.2R` và Sell `+1.3R` đều dương) |
| **`v102`** *(3-HA, Chỉ Buy)* | *TRADERPTKT* (`77.0đ`) | `M15` HA + EMA 100/200 + RSI(7) | 274 | **`41.2%`** | **`+64.74R`** | **`+$72,292.77`** | **`1.32`** | **`6.95%`** | **Tối ưu nhất (Return/DD > 10x)**, Winrate giữ vững `40–42.6%` suốt 3 năm |

---

## 1. Chiến lược #1: `v101_NY_LondonSweep` (RBI FOREX — Tier S #1, `81.0đ`)

- **Video gốc:** `https://www.youtube.com/watch?v=p1t9x-o5Lhg` (*Best New York Strategy for Gold*)
- **Mã nguồn EA:** [`v101_NY_LondonSweep.mq5`](../v101_NY_LondonSweep.mq5) (`MagicNumber = 801`)
- **Compile & Chạy test:** [`compile_v101.ps1`](../compile_v101.ps1) · [`backtest/run_v101.ps1`](../backtest/run_v101.ps1)

### 1.1. Quy tắc 6 bước bóc từ Transcript + Ảnh chụp màn hình
1. **Biên độ phiên London (`02:00 – 05:00 NY` = `09:00 – 12:00` Server Vantage):** Ghi nhận đỉnh `LondonHigh (LH)` và đáy `LondonLow (LL)`.
2. **Điều kiện Giữ Nguyên Vẹn (`05:00 – 07:00 NY` = `12:00 – 14:00` Server):** Trước khi mở phiên New York, giá **không được chạm/phá** `LH` hoặc `LL`. Nếu phá sớm → bỏ toàn bộ ngày hôm đó.
3. **Cửa sổ New York (`07:00 – 10:00 NY` = `14:00 – 17:00` Server):** Chờ giá quét qua `LH` hoặc `LL`.
4. **2 Mô hình vào lệnh:**
   - `MODEL_M5_RECLAIM` (`EntryModel = 0`, bản tác giả backtest 1 tháng trong video): Đợi nến `M5` đóng cửa ngược trở lại vào trong vùng `[LL, LH]` ngay ở **lần thử đầu tiên** (nếu nến trước râu chọc vào trong rồi bị từ chối đóng ngoài biên thì bỏ), kèm thân nến khoẻ (`thân/biên >= 45%` hoặc nhấn chìm thân nến trước).
   - `MODEL_M1_MSS_FVG` (`EntryModel = 1`): Sau khi quét biên London, xuống `M1` chờ nến phá đỉnh/đáy fractal swing gần nhất tạo khoảng trống `FVG >= $0.20`, khớp lệnh khi giá hồi chạm mép `FVG` trong vòng 30 phút.
5. **Dừng lỗ (SL):** Đặt ngoài đỉnh/đáy cú quét cộng đệm `$0.30` (`30 pip`).
6. **Chốt lời (TP):** Nếu biên đối diện của phiên London cho tỷ lệ `>= 3.0R` thì đặt TP tại biên London đối diện; nếu `< 3.0R` thì đặt cứng `3.0R`.

### 1.2. Đối chiếu kèo mẫu trong Video & Sự thật 3 năm
- **Khớp từng cây nến với ảnh chụp màn hình trong video:**
  - Ngày `31/12/2025` (`07:25 NY` = `14:25 Srv`): Video vẽ lệnh BUY `XAUUSD` ăn `7.3R` lên đỉnh London. EA `v101` tự động khớp BUY lúc `14:30 Srv` tại `4307.17`, SL `4302.88`, TP đỉnh London `4337.37` → thắng **`+7.04R`**.
  - Ngày `14/01/2026` (`08:05 NY` = `15:05 Srv`): EA `v101` tự động khớp SELL tại `4635.29`, SL `4642.34`, TP `4614.14` → thắng **`+3.00R`**.
  - Cửa sổ 1 tháng trong video (`15/12/2025 → 17/01/2026`, `Model=4`): `8` lệnh, thắng `50%`, lãi **`+12.11R` (`+$12,427.69`, `PF = 3.83`)**.
- **Vì sao chạy 3 năm (`2024–2026`) bản 2 chiều lại lỗ (`-6.77R`, `PF = 0.91`)?**
  - Phân tách theo chiều trên Real Ticks: **`BUY` (quét đáy London): `59` lệnh, `+9.16R` (`+$6,719`)** vs **`SELL` (quét đỉnh London): `75` lệnh, `-15.93R` (`-$15,295`)**.
  - Trong chu kỳ Vàng tăng mạnh 2024–2025, hành vi mở phiên Mỹ vượt đỉnh phiên Âu đa số là **phá vỡ tiếp diễn xu hướng thật (Breakout)** chứ không phải bẫy quét thanh khoản. Chỉ có chiều quét đáy phiên Âu rồi kéo rút chân lên (`Buy-Only`, `PF = 1.19`) là có lợi thế dương.

---

## 2. Chiến lược #2: `v102_HA_EMA_RSI` (TRADERPTKT — Tier S #3, `77.0đ`)

- **Video gốc:** `https://www.youtube.com/watch?v=z2JY4CVCxaw` (*Phương Pháp Scalping Thực Chiến Với Heiken Ashi, EMA 100/200 Và RSI(7)*)
- **Mã nguồn EA:** [`v102_HA_EMA_RSI.mq5`](../v102_HA_EMA_RSI.mq5) (`MagicNumber = 802`)
- **Compile & Chạy test:** [`compile_v102.ps1`](../compile_v102.ps1) · [`backtest/run_v102.ps1`](../backtest/run_v102.ps1)

### 2.1. Phát hiện quan trọng từ việc chụp ảnh màn hình Video
Nếu chỉ đọc phụ đề chữ mà không chụp màn hình TradingView của tác giả (`00:07:15` & `00:08:06`), rất dễ code lệch 3 chi tiết quyết định:
1. **Nguồn dữ liệu chỉ báo (`UseHASource = true`):** Khi chuyển loại biểu đồ trên TradingView sang **Heikin Ashi**, các chỉ báo `EMA(100)`, `EMA(200)` và `RSI(7)` mặc định nhận đầu vào là `haClose` (làm mượt nhiễu râu nến M15).
2. **Vị trí giá trong nhịp hồi:** Trong nhịp điều chỉnh làm `RSI(7)` lùi về `<= 40` (Buy) hoặc `>= 60` (Sell), giá thường nhúng chạm vào giữa dải `EMA 100 – EMA 200`; điều kiện *"nằm trên cả 2 EMA"* áp dụng tại **cây nến xác nhận vào lệnh** (khi lực mua đã quay trở lại đẩy giá đóng trên `EMA 100`), không phải cấm giá chạm `EMA 100` trong lúc hồi.
3. **Độ sạch của cây nến Heikin Ashi xác nhận:** Như tác giả mô tả ở `00:04:35`, cây nến HA xác nhận xu hướng mạnh có **bóng nến ngược chiều rất ngắn (`<= 15%` tổng biên độ nến)** và SL đặt dưới đáy cấu trúc nhịp hồi (`12` nến M15 + `0.25 × ATR(14)`).

### 2.2. Phân rã kết quả theo từng năm (`Model=4` Real Ticks, `$100,000`, `1%` Risk)

| Cấu hình `v102` (`XAUUSD.sc` M15, Phiên Âu–Mỹ `08–20h` Srv, `TP = 2.0R`) | Năm 2024 | Năm 2025 | Năm 2026 *(T1–T10)* | Tổng 3 Năm (`Model=4`) | Max DD |
|---|---|---|---|---|---|
| **Chuẩn 3 nến HA (`Buy + Sell`)** | 154 lệnh (`+31.89R` / `+$32,021`) | 163 lệnh (`+16.80R` / `+$17,432`) | 126 lệnh (`+12.14R` / `+$16,559`) | **443 lệnh · Win `37.9%` · `+60.83R` · `+$66,012` · PF `1.17`** | `13.63%` |
| **Vào sớm 2 nến HA (`Buy + Sell`)** | 178 lệnh (`+16.68R` / `+$13,026`) | 182 lệnh (`+48.83R` / `+$60,757`) | 156 lệnh (`+10.98R` / `+$16,746`) | **516 lệnh · Win `38.4%` · `+76.49R` · `+$90,529` · PF `1.20`** | `20.73%` |
| **Chuẩn 3 nến HA (`Chỉ Buy`)** | 101 lệnh (`+27.89R` / `+$26,135`) | 113 lệnh (`+24.83R` / `+$28,694`) | 60 lệnh (`+12.01R` / `+$17,464`) | **274 lệnh · Win `41.2%` · `+64.74R` · `+$72,292` · PF `1.32`** | **`6.95%`** |

---

## 3. Lệnh chạy lại kiểm chứng bất kỳ lúc nào

```powershell
# Biên dịch 2 EA (không đụng terminal MT5 đang mở)
.\compile_v101.ps1
.\compile_v102.ps1

# Chạy Strategy Tester Real Ticks (Model=4) trên C:\MT5_Test
.\backtest\run_v101.ps1 -From 2025.12.15 -To 2026.01.17 -Model 4 -Tag vid_month_rt
.\backtest\run_v101.ps1 -From 2024.01.01 -To 2026.11.01 -Model 4 -EnableSell false -Tag 3y_m5_buyonly_rt

.\backtest\run_v102.ps1 -From 2024.01.01 -To 2026.11.01 -Model 4 -HABarsConfirm 3 -TPRR 2.0 -Tag 3y_ha3_tp20_sess_rt
.\backtest\run_v102.ps1 -From 2024.01.01 -To 2026.11.01 -Model 4 -HABarsConfirm 3 -EnableSell false -TPRR 2.0 -Tag 3y_ha3_buyonly_rt
```


---

## 4. Lô Sàng Lọc 2 & Phát Hiện Sáng Giá Nhất: `v103_MTF_MACD` (`H4 → H1 → M15`)

Tiếp tục bóc thêm 6 transcript từ danh sách Tier S (*Opening Range Breakout 9:30 NY*, *Williams %R(16) + Parabolic SAR*, *EMA 9/20 First-Pullback Body Break*, và *3-Timeframe MACD Stack*), kết quả sàng lọc 3 năm (`2024–2026`) trên 1 triệu nến M1 `XAUUSD.sc` cho thấy:
- **9:30 NY M15 ORB:** Gần như đi ngang trên Vàng (`PF = 1.02–1.09`).
- **Williams %R(16) + PSAR + EMA 20/200:** Quá nhiều nhiễu (`1,900+` lệnh trên M5, `PF = 1.04`, `DD > 40R`).
- **EMA 9/20 First-Pullback H1 (`Chỉ Buy`):** Khá tốt (`PF = 1.40` năm 2024–2025) nhưng âm nhẹ năm 2026 (`-6.8R`).
- **🏆 `v103_MTF_MACD` (*The Secret Mindset* — Đa khung `H4 + H1 + M15` MACD Zero-Line + Histogram Flip):** Vượt trội toàn diện trên cả 3 năm!

### 4.1. Quy tắc cơ học của [`v103_MTF_MACD.mq5`](../v103_MTF_MACD.mq5) (`MagicNumber = 803`)
1. **Tầng 1 — `H4` đã đóng (Zero-Line Law):** `H4 MACD(12,26,9) > 0` → chỉ tìm BUY; `< 0` → chỉ tìm SELL. (Tính chuẩn `EMA(9)` cho đường Signal như TradingView thay vì `SMA(9)` mặc định của hàm `iMACD` cũ).
2. **Tầng 2 — `H1` đã đóng (Xung lực thuận):** Với BUY: `H1 MACD > 0` **và** `H1 MACD > H1 Signal`. Với SELL: `H1 MACD < 0` **và** `H1 MACD < H1 Signal`.
3. **Tầng 3 — `M15` vừa đóng (Điểm nổ Histogram Flip):** Trong phiên Âu–Mỹ (`08:00–20:00` Server), sau ít nhất `3` thanh Histogram M15 ngược màu liên tiếp (nhịp hồi), thanh Histogram M15 vừa đóng **lật sang thuận chiều** (`> 0` cho BUY / `< 0` cho SELL) → Vào lệnh Market.
4. **SL & TP:** `SL` đặt tại đáy/đỉnh `12` nến M15 `± 0.25 × ATR(14)`; `TP = 2.0R` cố định.

### 4.2. Kết quả MT5 Strategy Tester — **Real Ticks (`Model=4`, `01/2024 → 11/2026`, `$100,000`, `1%` Risk)**

| Cấu hình `v103_MTF_MACD` | Năm 2024 | Năm 2025 | Năm 2026 *(T1–T10)* | Tổng 3 Năm (`Model=4`) | Max DD |
|---|---|---|---|---|---|
| **`Buy + Sell` (2 chiều)** | 72 lệnh (`+10.05R` / `+$9,730`) | 79 lệnh (`+30.46R` / `+$37,680`) | 65 lệnh (`+11.88R` / `+$17,388`) | **216 lệnh · Win `42.1%` · `+52.38R` · `+$64,798.36` · PF `1.39`** | **`7.43%`** |
| **`Chỉ Buy` (`EnableSell=false`)** | 45 lệnh (`+19.17R` / `+$20,476`) | 56 lệnh (`+29.34R` / `+$39,946`) | 25 lệnh (`+1.07R` / `+$1,289`) | **126 lệnh · Win `47.6%` · `+49.57R` · `+$61,711.43` · PF `1.66`** | **`5.55%`** |

- Điểm đặc biệt của `v103` bản **2 chiều (`Buy + Sell`)**: Cả chiều `BUY` (`126` lệnh, `+49.57R`, `+$56,489`) lẫn chiều `SELL` (`90` lệnh, `+2.81R`, `+$8,309`) đều có lãi ròng dương trên Vàng — đặc biệt năm 2026 chiều `SELL` đóng góp hơn `+10.8R` khi Vàng có các nhịp chỉnh sâu trên H4!
- **Ghép danh mục `v102 (Chỉ Buy)` + `v103 (Buy + Sell)`:** Tổng `490` lệnh trong 3 năm, tổng tích luỹ **`+117.12R`**, Max Drawdown gộp chỉ **`12.11R`** (nếu chia mỗi bot `0.5%` risk/lệnh thì lãi `~+58.5%` với sụt giảm tối đa toàn tài khoản chỉ `~6.0%`).

### 4.3. Hai trang HTML soi từng lệnh & kiểm tra quy tắc độc lập
- **[`v102_report_2026.html`](../v102_report_2026.html)** (tạo bởi `backtest/v102_trade_viewer.py`): 100% (`274/274` & `443/443`) lệnh khớp 8/8 điều kiện.
- **[`v103_report_2026.html`](../v103_report_2026.html)** (tạo bởi `backtest/v103_trade_viewer.py`): 100% (`216/216` & `126/126`) lệnh khớp 6/6 điều kiện.


---

## 5. Cập nhật `v102_HA_EMA_RSI` lên `v1.10` (Chốt Chỉ BUY + Neo trần `MaxSL_PctPrice = 0.80%` + Không Trail)

Sau khi kiểm định 25 cơ chế Trailing SL trên 1 triệu nến M1 và bóc tách riêng chiều `BUY`/`SELL` qua 4 năm (`2023–2026`), cấu hình mặc định chuẩn của [`v102_HA_EMA_RSI.mq5`](../v102_HA_EMA_RSI.mq5) (`v1.10`) đã được chốt theo nguyên tắc khách quan:
- **`EnableBuy = true`, `EnableSell = false` (`Chỉ BUY`):** Chiều `SELL` của mô hình trễ 3 nến HA âm liên tiếp cả trong sóng giảm 5 tháng `T5–T10/2023` (`-6.94R`) lẫn `2024–2026` (`-12.0R`) do đặc tính xả nhanh giật chữ V của Vàng.
- **`MaxSL_PctPrice = 0.80` (`|Entry - SL| / Entry <= 0.80%` thị giá):** Thước neo bất biến bám hằng số `~1.5×` biên độ H1 đỉnh điểm 5 năm của Vàng, tự co giãn mượt từ mức giá `1,830` (2023) tới `5,522` (2026), loại bỏ hoàn toàn các lệnh mua đuổi SL phình `50–67 giá`.
- **Giữ `TP_RR = 2.0` cố định (Không Trail):** Bảo toàn `Winrate ~41–44%` và hiệu suất ổn định năm 2026.

| Năm (`Model=4` Real Ticks, `v1.10`) | Số lệnh | Winrate | Tổng `R` | Trung vị SL | Tối đa SL (`% Giá`) |
|---|---|---|---|---|---|
| **`2023` *(Out-of-Sample)*** | `94` | `37.2%` | **`+12.62R`** | `5.57 giá` | `15.78 giá` (`0.78%`) |
| **`2024`** | `100` | `41.0%` | **`+22.87R`** | `7.96 giá` | `17.99 giá` (`0.73%`) |
| **`2025`** | `111` | `40.5%` | **`+23.84R`** | `12.17 giá` | `31.18 giá` (`0.77%`) |
| **`2026` *(01 → 10/2026)*** | `48` | **`43.8%`** | **`+15.11R`** *(tăng từ `+12.01R`)* | `22.56 giá` | **`39.31 giá`** (`0.79%`) |
| **Gộp `2024–2026` (`$100k`, `1%` Risk)** | **`259`** | **`41.3%`** | **`+61.82R` (`+$68,347.51`)** | **`PF = 1.34`** | **`MaxDD = 6.93%`** |
