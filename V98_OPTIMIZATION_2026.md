# v98_TylerFVG — Tối ưu SL / thoát lệnh / khung giờ (15/11/2026)

> Câu hỏi đặt ra: *"Giãn SL ×2 khá rủi ro. Giữ nguyên luật vào lệnh 4 bước như video, tìm cách đặt SL / thoát lệnh an toàn hơn; có thể mở 24/24 và thử các cặp phiên Âu/Mỹ; ưu tiên Vàng."*
>
> Công cụ: `backtest/sweep_v98.ps1` (chạy hàng loạt theo plan JSON) + `backtest/v98_sweep_summary.py` (tổng hợp theo **R**). Toàn bộ plan và kết quả từng lượt nằm trong `backtest/v98_out/plan_s*.json`, `s*_*.csv/.htm`, log `sweep_log.txt`. Tổng cộng **~195 lượt** Strategy Tester.

---

## 0. Kết luận nhanh

| | Cấu hình | Kết quả 2024.01 → 2026.11 (1% risk, $10k) | Từng năm (R) |
|---|---|---|---|
| ✅ **Đề xuất chính** | 24/24 · Buy+Sell · **SL = chân sóng 5 nến − 0.2 ATR** · thoát MA Stack · **BE tại +1.5R** | **+96.5R**, 371 lệnh, win 37.7%, PF 1.60, DD 14.7% | 2024 **+21.5** · 2025 **+43.8** · 2026 **+31.3** |
| ✅ Biến thể êm hơn | như trên nhưng **chỉ vào lệnh 10:00–00:00 Server** (FVG vẫn ghi nhận cả ngày) | +74.3R, 253 lệnh, win 39.5%, PF 1.69, **DD 10.1%** | +23.4 · +29.9 · +20.9 |
| ✅ Nếu chỉ muốn Buy (ưu tiên Vàng) | 24/24 · **chỉ Buy** · SL chân sóng 5 − 0.2 ATR · BE 1.5R | +78.1R, 238 lệnh, win 38.2%, **PF 1.76**, DD 11.8% | +33.1 · +39.2 · **+5.8** (2026 yếu) |
| ⚠️ Lãi cao nhất nhưng DD cao | 24/24 · Buy+Sell · SL chân sóng 5 − 0.2 ATR · MA Stack (không BE) | +107.7R, PF 1.59, DD 16.3% | +12.7 · +65.1 · +29.8 |
| ❌ Cách cũ của Gemini | SL đáy nến rút râu **×2.0** | +81.6R, PF 1.47, DD 15.8% | **−6.8** · +54.3 · +34.1 |
| ❌ SL sát đáy nến rút râu (gốc) | Rej − $0.30 | +78.5R, PF 1.30, **DD 32.2%**, 63% lệnh bị quét −1R | **−14.4** · +66.1 · +26.8 |
| ❌ London 10:00–13:30 (như video), chỉ Buy | mọi cách đặt SL | 12 tháng qua: 10 lệnh, **0 thắng**, −7.6R … −9R | — |
| ❌ 4 cặp tiền EURUSD / GBPUSD / USDJPY / USDCAD | 24/24, mọi cách SL | **đều âm** (PF 0.6–0.85) | — |

**Ba điều đáng tin nhất (bền qua cả 3 năm, cả 2 chiều, M1 và Real Ticks):**
1. **Neo SL vào chân sóng 5 nến (SL_SWING_LOW) tốt hơn mọi cách neo vào đáy nến rút râu** — kể cả ×2.0 hay cộng ATR. Đây là cách duy nhất dương cả 3 năm ở mọi biến thể.
2. **Phải bật cả Sell trên Vàng** nếu muốn sống qua năm 2026: Buy 2026 chỉ +5.8R, Sell +25.4R. Ngược lại 2024–2025 Buy gánh.
3. **Lợi nhuận đến từ phiên Mỹ (14:00–02:00 Server), phiên Á âm năm 2026, phiên London 10:00–13:30 không đủ lệnh.** Luận điểm "chỉ đánh London 3:00–6:30 NY" của video không đứng vững trên Vàng 2026.

**Trả lời thẳng câu "SL ×2 rủi ro":**
- Với `LotMode = LOT_RISK_PCT`, **tiền rủi ro mỗi lệnh KHÔNG đổi** khi giãn SL — EA tự giảm lot. Cái mất là R-multiple nhỏ hơn (lệnh thắng ăn ít R hơn) và lỗ mềm khi EMA cắt. Chỉ khi dùng `LOT_FIXED` thì giãn SL mới nhân đôi tiền lỗ.
- SL chân sóng 5 nến **không hẹp hơn** SL ×2 (trung vị $8.4 so với $9.0 trên 2024–2026) nhưng neo vào cấu trúc thật nên ít bị quét râu hơn và sống qua 2024. Cái làm DD giảm mạnh là **BE tại 1.5R**: DD 16.3% → 14.7% (24/24) và 10.1% (khung 10:00–00:00), đổi lại mất ~10% tổng R.

---

## 1. Cách làm

- **Dữ liệu:** `XAUUSD.sc` (Vantage Live, contract 1 oz/lot), M30. Vốn $10,000, **1% risk/lệnh** → so sánh theo **R** (độc lập cỡ lot). Mô hình tick: **M1 OHLC** cho sweep (13 giây/lượt), **Real Ticks** để xác nhận ứng viên cuối.
- **Cửa sổ:** sweep trên **12 tháng gần nhất (2025.11.01 → 2026.11.01)**; kiểm tra **ngoài mẫu** trên 12 tháng trước đó (2024.11 → 2025.11); ứng viên cuối chạy **toàn giai đoạn 2024.01 → 2026.11**.
- **Giữ nguyên:** toàn bộ luật vào lệnh (EMA stack, FVG ≥ $0.20, râu ≥ 40%, nến xác nhận không đóng vượt đỉnh nến rút râu, market khi đóng nến). Chỉ đổi: cách neo SL, cách thoát, khung giờ, chiều, số lệnh/ngày.
- **Số lượt:** S1 48 (SL × phiên × chiều) · S2 107 (thoát lệnh × SL, khung giờ) · S3 24 (4 cặp tiền) · S4 21 (ngoài mẫu, Real Ticks, toàn giai đoạn, phiên Á) · S5 8 (khung giờ Âu/Mỹ giữ FVG cả ngày) · S6 8 (toàn giai đoạn ứng viên cuối) · S7 3 (Real Ticks toàn giai đoạn).

---

## 2. Giai đoạn 1 — Cách neo SL (12 tháng gần nhất, thoát MA Stack)

12 cách SL × {London / 24/24} × {Buy / Buy+Sell}. Trích nhóm **24/24 Buy+Sell** (124 lệnh) và **London chỉ Buy** (10 lệnh):

| Cách neo SL | 24/24 B+S: sumR | expR | PF | ddR | % bị SL quét | London Buy: sumR |
|---|---|---|---|---|---|---|
| Rej + 0.5 ATR | **+36.9** | **0.298** | 1.63 | 11.1 | 35% | −8.8 |
| Rej − $0.30, sàn 1.0 ATR | +33.7 | 0.269 | 1.50 | 11.7 | 43% | −8.9 |
| Rej ×2.0 (Gemini) | +32.1 | 0.259 | 1.59 | 10.3 | 29% | −8.2 |
| **Chân sóng 5 nến − 0.2 ATR** | +31.1 | 0.251 | 1.52 | 10.7 | 35% | −7.8 |
| Rej − $0.30 (gốc) | +31.0 | 0.246 | 1.38 | 13.2 | **60%** | −9.0 |
| FVG đáy − 0.25 ATR | +25.7 | 0.207 | 1.41 | 11.8 | 41% | −9.0 |
| EMA21 − 0.25 ATR | +25.2 | 0.203 | 1.48 | 10.1 | 28% | −7.8 |
| Rej ×1.5 | +25.0 | 0.202 | 1.38 | 11.6 | 45% | −9.0 |
| Chân sóng 3 nến − 0.2 ATR | +24.3 | 0.194 | 1.33 | 12.7 | 51% | −9.0 |
| Rej − $0.30, sàn 1.5 ATR | +23.1 | 0.186 | 1.44 | 10.4 | 28% | −7.6 |
| Rej + 0.25 ATR | +22.1 | 0.177 | 1.31 | 12.9 | 50% | −9.0 |
| Rej + 1.0 ATR | +20.4 | 0.165 | 1.44 | **8.2** | 20% | −7.8 |

- **London 10:00–13:30, chỉ Buy: 10 lệnh / 0 thắng ở cả 12 cách SL.** London Buy+Sell: 13 lệnh, chỉ 2 cách dương (+4.2R, +2.0R). Phiên London trong 12 tháng qua không có đủ setup để đánh.
- **24/24 chỉ Buy:** 70 lệnh, +0.8R … +10.7R — gần hoà. **24/24 Buy+Sell:** +20R … +37R ở **mọi** cách SL → chiều Sell là nguồn lãi 2026.
- SL càng sát (gốc, ×1.5, +0.25 ATR, chân sóng 3) càng bị quét (45–60%); đệm ≥ 0.5 ATR hoặc chân sóng 5 nến hạ tỷ lệ bị quét xuống ~35%.

## 3. Giai đoạn 2 — Cách thoát lệnh & khung giờ (24/24, Buy+Sell)

5 cách SL tốt nhất × 11 cách thoát. Trích theo SL **chân sóng 5 − 0.2 ATR** và **Rej + 0.5 ATR**:

| Cách thoát | Chân sóng 5: sumR / win / ddR | Rej + 0.5 ATR: sumR / win / ddR |
|---|---|---|
| MA Stack 5-9-13-21 (chuẩn Tyler) | +31.1 / 30.6% / 10.7 | +36.9 / 31.5% / 11.1 |
| MA Stack, chỉ cho EMA đóng lời khi ≥ 1R (`MinProfitExit_RR=1`) | +39.4 / 24.4% / 12.5 | **+40.2** / 23.8% / 12.8 |
| **MA Stack + BE tại 1.5R** | +34.5 / **37.6%** / **9.7** | +27.7 / 38.9% / 10.7 |
| MA Stack + Trailing 3.0 ATR từ +1R | +27.8 / 31% / 10.0 | +31.8 / 32.1% / 10.1 |
| EMA5 × EMA13 | +22 / 30% / 12 | +30.5 / 30.1% / 11.7 |
| EMA9 × EMA21 | thấp hơn MA Stack ở mọi SL | |
| TP cứng 5R (không EMA) | **+45.0** / 23.4% / **16.0** (77% lệnh −1R) | +31 / 22% / 14 |
| TP cứng 2R–3R | +15 … +26, kém MA Stack | |
| Trailing bậc R (STEP_RR) | kém MA Stack | |

- Thoát MA Stack của Tyler đã tốt; **BE 1.5R** là cách duy nhất vừa tăng winrate (31% → 38%) vừa giảm DD, đổi lại mất vài R.
- TP 5R cho tổng R cao nhất nhưng 77% lệnh chạm SL, chuỗi thua dài, DD cao — khó cầm tâm lý.
- **Số lệnh/ngày:** 1 lệnh/ngày +29.8R, 3 lệnh/ngày +36.9R, 10 lệnh/ngày = 3 lệnh/ngày (không bao giờ chạm trần vì chỉ 1 lệnh mở cùng lúc).

**Khung giờ (SL Rej + 0.5 ATR, Buy+Sell, MA Stack):**

| Khung | Cách lọc | 12 tháng qua | 12 tháng trước |
|---|---|---|---|
| 24/24 | không lọc | +36.9R / 124 lệnh / ddR 11.1 | +55.5R / 137 / 7.3 |
| **Vào lệnh 10:00–00:00, FVG ghi nhận cả ngày** | `EnableTimeFilter=true`, FVG 00:00, Entry 10:00–23:30 | +28.5R / 82 / **7.3** (expR 0.347) | +40.6R / 96 / **7.1** (expR 0.423) |
| Chỉ phiên Mỹ 14:00–00:00 | FVG 00:00, Entry 14:00–23:30 | +21.1R / 68 | +7.1R / 69 |
| Âu+Mỹ 09:30–20:00, FVG reset mỗi ngày | filter cũ | +7.8R / 61 | +20.7R / 61 |
| Phiên Á 00:30–09:00 | | **−10.2R** / 38 | +15.2R / 46 |
| London 10:00–13:30 (video) | | +2.0R / 13 | — |

Lỗ phiên Á 2026 kéo DD; bỏ phiên Á (khung 10:00–00:00) làm kỳ vọng/lệnh tăng và DD giảm ở **cả hai năm**, đổi lại ít lệnh hơn ~35%. Lưu ý filter kiểu cũ (reset FVG theo ngày) làm mất các FVG hình thành đêm trước — vì vậy khung Âu+Mỹ ở S2 cho kết quả kém, không phải vì giờ đó xấu.

## 4. Giai đoạn 3 — 4 cặp tiền phiên Âu/Mỹ (12 tháng qua, Buy+Sell)

| Cặp | 24/24 (3 cách SL) | London 10:00–13:30 |
|---|---|---|
| EURUSD | −9.9 … −14.8R, PF 0.77–0.85 | +1.3 … +3.6R (10 lệnh) |
| GBPUSD | −16.4 … −23.2R, PF 0.71–0.79 | −1.8 … +4.9R (13–14 lệnh) |
| USDJPY | −14.9 … −20.8R, PF 0.74–0.82 | −3.9 … −5.3R |
| USDCAD | −22.6 … −31.5R, PF 0.61–0.66 | **−14 … −15.5R** (1 thắng / 20 lệnh) |

Luật 4 bước **không chuyển sang Forex** trong 12 tháng qua. London-only EURUSD/GBPUSD dương nhẹ trên 10–14 lệnh, không có ý nghĩa thống kê. Không khuyến nghị chạy cặp tiền.

## 5. Giai đoạn 4–6 — Ngoài mẫu, Real Ticks, toàn giai đoạn

**Real Ticks khớp M1** (12 tháng qua, 24/24 B+S): Rej+0.5ATR +36.2R (M1 +36.9) · Chân sóng 5 + BE +33.4R (M1 +34.5) · Rej ×2 +31.1R (M1 +32.1) · Gốc +29.4R (M1 +31.0). Sai lệch < 1.5R → dùng M1 cho sweep là chấp nhận được.

**Toàn giai đoạn 2024.01 → 2026.11 (M1), xếp theo tổng R:**

| Cấu hình (24/24 trừ khi ghi khác) | n | win | sumR | expR | PF | ddR | DD% | 2024 | 2025 | 2026 |
|---|---|---|---|---|---|---|---|---|---|---|
| B+S · Chân sóng 5 − 0.2 ATR · MA Stack | 366 | 29.0% | **+107.7** | 0.294 | 1.59 | 21.3 | 16.3% | +12.7 | +65.1 | +29.8 |
| **B+S · Chân sóng 5 − 0.2 ATR · MA Stack + BE 1.5R** | 371 | 37.7% | **+96.5** | 0.260 | 1.60 | 16.1 | 14.7% | **+21.5** | +43.8 | +31.3 |
| B+S · Rej ×2.0 · MA Stack (Gemini) | 369 | 29.0% | +81.6 | 0.221 | 1.47 | 22.4 | 15.8% | −6.8 | +54.3 | +34.1 |
| B+S · Rej + 0.5 ATR · MA Stack ≥ 1R | 367 | 23.7% | +79.9 | 0.218 | 1.40 | 24.8 | 18.5% | −13.2 | +50.9 | +42.2 |
| B+S · Rej − $0.30 · MA Stack (gốc) | 384 | 22.4% | +78.5 | 0.204 | 1.30 | **38.3** | **32.2%** | −14.4 | +66.1 | +26.8 |
| **Chỉ Buy · Chân sóng 5 − 0.2 ATR · BE 1.5R** | 238 | 38.2% | +78.1 | **0.328** | **1.76** | 12.5 | 11.8% | +33.1 | +39.2 | +5.8 |
| **B+S · Chân sóng 5 · BE 1.5R · vào lệnh 10:00–00:00** | 253 | 39.5% | +74.3 | 0.294 | 1.69 | **11.7** | **10.1%** | +23.4 | +29.9 | +20.9 |
| B+S · Rej + 0.5 ATR · MA Stack | 371 | 29.1% | +73.8 | 0.199 | 1.38 | 27.7 | 24.2% | −16.5 | +52.1 | +38.2 |
| B+S · Rej + 0.5 ATR · MA Stack · 10:00–00:00 | 255 | 31.4% | +63.6 | 0.249 | 1.48 | 18.1 | 16.5% | −3.8 | +40.2 | +27.2 |
| B+S · Rej + 0.5 ATR · BE 1.5R | 377 | 37.9% | +58.6 | 0.155 | 1.34 | 21.4 | 15.8% | −9.8 | +39.8 | +28.6 |
| Chỉ Buy · Rej + 0.5 ATR · MA Stack | 240 | 28.8% | +56.0 | 0.233 | 1.43 | 19.4 | 12.3% | +2.2 | +45.9 | +7.9 |

- **Mọi cách neo vào đáy nến rút râu đều lỗ năm 2024** (−4 … −16R, chiều Sell thua −19 … −27R). **Chân sóng 5 nến dương cả 3 năm** ở cả 4 biến thể.
- Rej + 0.5 ATR thắng sweep 12 tháng nhưng thua năm 2024 → đúng kiểu "tối ưu trong mẫu"; đây là lý do phải kiểm tra ngoài mẫu.
- Khoảng SL trung vị 2024–2026: gốc $4.5 · Rej+0.5ATR $7.6 · chân sóng 5 $8.4 · ×2.0 $9.0 (2026: $9 / $15 / $16.6 / $18.4).

**Real Ticks toàn giai đoạn cho 3 cấu hình đề xuất (S7):** xem mục 8 (được điền sau khi chạy xong).

---

## 6. Cấu hình đề xuất (mặc định v1.22; **v1.23** đổi `LotMode = LOT_FIXED` + trailing 2 ATR theo yêu cầu; **v1.24 tắt luôn BE và trailing** sau khi xem mục 9.4 — thoát MA Stack đơn thuần)

```
LotMode = LOT_RISK_PCT, RiskPercent = 0.5–1.0     (KHÔNG dùng LOT_FIXED với SL rộng)
EnableBuy = true, EnableSell = true, MaxTradesPerDay = 3
EnableTimeFilter = false                           (24/24)
   → biến thể êm hơn: EnableTimeFilter = true, FVG_Start 00:00, Entry 10:00 → 23:30
SignalTF = M30, MinFVG_Pip = 20, MinRejectWickRatio = 0.40
SL_Mode = SL_SWING_LOW, SL_SwingBars = 5, SL_BufferPip = 0, SL_BufferATR = 0.20,
SL_DistMult = 1.0, MinSL_FloorATR = 0, MaxSL_Pip = 0
ExitMode = EXIT_MA_STACK_BREAK, FixedTP_RR = 0, MinProfitExit_RR = 0
EnableBreakEven = true, BE_TriggerRR = 1.5, BE_OffsetPip = 1
TrailMode = TRAIL_NONE
```

Chỉ Buy: giữ y nguyên, đặt `EnableSell = false` (PF 1.76, nhưng 2026 chỉ +5.8R).

## 7. Những gì vẫn phải nhớ

1. **Vẫn là tối ưu trên dữ liệu đã nhìn.** 195 lượt chạy trên 2024–2026; tiêu chí chọn là "dương cả 3 năm, cả 2 chiều, ít tham số" thay vì "PF cao nhất", nhưng kỳ vọng thực tế vẫn nên hạ 30–50% so với bảng.
2. **Chuỗi lệnh không lãi dài nhất: 23 lệnh** (cấu hình đề xuất). Với 1% risk đó là −15…−20% tài khoản trước khi phục hồi. Dùng 0.5% nếu không chịu được.
3. **Lãi lệch về vài lệnh lớn:** lệnh lớn nhất +33R; 5 lệnh lớn nhất chiếm phần lớn R mỗi năm. Bỏ lỡ 1 lệnh (mất mạng, restart) có thể đổi kết quả cả quý → v1.21 đã thêm khôi phục trạng thái, nhưng VPS ổn định là bắt buộc.
4. **2026 phụ thuộc chiều Sell**; nếu Vàng quay lại xu hướng tăng mạnh, Sell sẽ lỗ như 2024 (−11.6R) và Buy gánh. Cấu hình chân sóng 5 chịu được cả hai pha, cách khác thì không.
5. **Chưa test:** trần SL theo ATR (thay MaxSL_Pip cố định), yêu cầu nến rút râu đóng trên đáy FVG, lọc tin. Mỗi thứ cần chạy lại sweep từ đầu.
6. Tốc độ 13 giây/lượt (M1, 1 năm) — sweep lại khi có dữ liệu mới là rẻ; nên lập lịch chạy lại mỗi quý để phát hiện suy giảm.

## 8. Xác nhận Real Ticks toàn giai đoạn (S7)

Model = Every tick based on real ticks, 2024.01.01 → 2026.11.01, $10k, 1% risk. So với M1 cùng cấu hình:

| Cấu hình | Real Ticks: n / sumR / PF / DD% | M1: sumR / DD% | Từng năm RT (R) |
|---|---|---|---|
| 24/24 · B+S · Chân sóng 5 · BE 1.5R | 371 / **+94.5R** / 1.58 / 14.7% | +96.5 / 14.7% | +20.7 · +43.6 · +30.2 |
| Vào lệnh 10:00–00:00 · B+S · Chân sóng 5 · BE 1.5R | 253 / **+72.6R** / 1.67 / 10.1% | +74.3 / 10.1% | +22.8 · +29.8 · +19.9 |
| 24/24 · chỉ Buy · Chân sóng 5 · BE 1.5R | 238 / **+76.7R** / 1.73 / 12.1% | +78.1 / 11.8% | +32.6 · +39.1 · +5.0 |

Sai lệch Real Ticks so với M1 chỉ −1.5 … −2R trên gần 3 năm; số lệnh trùng khớp. Winrate hiển thị thấp hơn (29–32% so với 38–40%) vì trên Real Ticks các lệnh thoát ở mức hoà vốn bị trừ spread/phí thành âm vài đô — không ảnh hưởng R.

File: `backtest/v98_out/s7_rtfull_*.csv`.

---

## 9. S8 — Vốn $100k, 1 lot chuẩn cố định, Trailing SL thay BE (từng năm 2024 / 2025 / 2026)

Yêu cầu: "chạy như vốn 100k, 1 lot, tính riêng từng năm" và "trailing SL chạy theo giá thay vì dời BE một lần".
Thiết lập: XAUUSD.sc, `LotMode = LOT_FIXED`, `FixedLot = 100` (mã .sc 1 oz/lot → 100 lot .sc = **1 lot chuẩn**, $1 giá = $100),
Deposit $100,000, luật vào lệnh / SL / thoát giữ như mục 6 (24/24, Buy+Sell, chân sóng 5 − 0.2 ATR, MA Stack).
Mỗi năm chạy riêng, vốn reset về $100k đầu năm; 2026 chỉ tới 01/11. 24 lượt M1 OHLC + 9 lượt Real Ticks.

### 9.1 Kết quả Real Ticks (Model 4) — số tiền sau 1 năm với 1 lot chuẩn

| Cấu hình | 2024 | 2025 | 2026 (10 tháng) | Tổng 3 năm |
|---|---|---|---|---|
| **BE 1.5R, không trailing** (mặc định v1.22) | **+$7,651** (128 lệnh, +20.7R, DD 7.2%) | **+$20,259** (140, +43.7R, DD 19.2%) | **+$125,871** (103, +30.2R, DD 8.1%) | **+$153,781** |
| Không BE, không trailing | +$5,667 (128, +12.0R, DD 8.2%) | +$27,619 (136, +65.0R, DD 19.5%) | +$127,198 (102, +28.9R, DD 8.1%) | +$160,484 |
| **BE 1.5R + Trailing 2.0 ATR từ +1.5R** (mặc định v1.23) | +$11,032 (141, +10.9R, DD 6.4%) | +$6,823 (158, +27.4R, DD 20.2%) | +$76,190 (115, +22.3R, DD 10.6%) | +$94,045 |

Lưu ý khi đọc $: với lot cố định, **1R bằng bao nhiêu tiền phụ thuộc giá Vàng từng năm**
(SL trung vị 2024 $5.0 → $504/lệnh; 2025 $7.4 → $742; 2026 $16.6 → **$1,657**, SL lớn nhất 2026 $87 → −$8,737).
Vì vậy 2026 lãi $ gấp 6 lần 2025 dù R chỉ bằng 70%: không phải EA "hay hơn" mà vì mỗi lệnh to gấp đôi.
Lệnh lỗ đơn lớn nhất với 1 lot chuẩn: 2024 −$1,483 · 2025 −$3,397 · 2026 −$4,969. Chuỗi lỗ liên tiếp dài nhất: 7 / 11 / 8 lệnh.
Dùng 1 lot chuẩn trên $100k nghĩa là rủi ro mỗi lệnh ~0.5% (2024) đến ~1.7% (2026), và năm 2025 DD gần 20% ($20k).

### 9.2 So sánh 8 cách bảo vệ lệnh (M1 OHLC, $ sau mỗi năm, 1 lot chuẩn)

| Biến thể | 2024 | 2025 | 2026 | Tổng $ | Tổng R | Lệnh thắng lớn nhất (R) |
|---|---|---|---|---|---|---|
| BE 1.5R (không trail) | 7,947 | 20,377 | 127,415 | **155,739** | 96.5 | 33.3 / 11.7 / 8.8 |
| Không BE, không trail | 5,919 | 27,732 | 128,584 | 162,234 | 107.7 | 33.3 / 16.8 / 8.8 |
| Trail bám đỉnh $5 ngay từ đầu (cách Gemini đã thử) | 416 | 2,045 | 8,163 | 10,624 | 27.5 | 10.9 / 11.2 / 1.9 |
| Trail bám đỉnh cách 1R, bắt đầu +1R | 6,002 | 11,491 | 72,818 | 90,310 | 43.4 | 4.2 / 5.2 / 4.1 |
| Trail 1.5 ATR, bắt đầu +1R | 9,954 | 13,566 | 67,895 | 91,414 | 47.5 | 10.7 / 7.8 / 6.5 |
| Trail 2.0 ATR, bắt đầu +1.5R | 10,624 | 8,878 | 79,288 | 98,790 | 61.4 | 9.5 / 16.1 / 7.7 |
| Trail bậc thang STEP_RR (+1R→BE, +2R→+1R…) | 6,113 | 9,012 | 88,803 | 103,928 | 43.9 | 5.0 / 8.0 / 6.0 |
| BE 1.5R + Trail 2.0 ATR từ +1.5R | 11,550 | 7,435 | 78,296 | 97,281 | 64.2 | 9.5 / 16.1 / 7.7 |

### 9.3 Kết luận về trailing

- **Mọi kiểu trailing đều làm tổng lãi giảm 35–95% so với chỉ BE**, ở cả 3 năm trừ 2024 (trailing nhỉnh hơn vài nghìn $ vì năm đó nhiều lệnh đảo chiều sau khi lãi 1–2R).
- Lý do: thoát MA Stack **đã là một trailing theo xu hướng** (EMA 5/9/13/21 chạy theo giá). Thêm trailing SL sát hơn chỉ cắt sớm các lệnh lớn: lệnh thắng lớn nhất từ +33R (2024) rơi xuống +5…+10R. Chiến lược này sống nhờ vài lệnh rất lớn, cắt chúng là mất lợi thế.
- Trailing $5 cố định (cách Gemini từng test) tệ nhất vì $5 trên Vàng 2025–2026 nhỏ hơn 1 ATR M30, lệnh bị quét ngay khi vừa lãi.
- Nếu vẫn muốn trailing, phương án ít hại nhất là **2.0 ATR, chỉ bắt đầu khi đã lãi 1.5R** (giữ được 60–65% tổng R, DD 2024 thấp nhất 6.4%). Đây là mặc định v1.23 theo yêu cầu; đặt `TrailMode = TRAIL_NONE` để quay về BE-only (+$153,781 / 3 năm thay vì +$94,045).

File: `backtest/v98_out/s8_*.csv` (M1), `s8rt_*.csv` (Real Ticks), plan `plan_s8.json`, `plan_s8rt.json`.

### 9.4 Có nên bỏ luôn BE? (Real Ticks, $100k, 1 lot chuẩn) → **v1.24 tắt BE và trailing**

| | 2024 | 2025 | 2026 (10 th.) |
|---|---|---|---|
| **Không BE, không trailing** — thắng / tổng | 27/128 = **21.1%** | 47/136 = **34.6%** | 32/102 = **31.4%** |
| … ăn trọn SL (≤ −0.9R) / gần hoà (|R|<0.15) | 49 / 11 | 51 / 10 | 36 / 11 |
| … TB lệnh thắng / TB lệnh lỗ | +$1,499 / −$345 | +$1,959 / −$724 | +$6,069 / −$1,147 |
| … PF ($) / net / max DD ($) / chuỗi lỗ | 1.16 / +$5,667 / $8,251 / 14 | 1.43 / +$27,619 / $28,787 / 12 | 2.42 / +$127,198* / $14,677 / 8 |
| **BE 1.5R** — thắng / tổng | 28/128 = 21.9% | 48/140 = 34.3% | 31/103 = 30.1% |
| … ăn trọn SL / gần hoà | 36 / 30 | 46 / 26 | 34 / 17 |
| … PF ($) / net / max DD ($) / chuỗi lỗ | 1.25 / +$7,651 / $7,668 / 14 | 1.33 / +$20,259 / $26,304 / 23 | 2.40 / +$125,871* / $14,677 / 8 |

\* 2026 gồm một lệnh Sell mở 30/10 còn treo tới hết kỳ test (+$13,283 khi tester đóng), CSV chỉ ghi lệnh đã đóng (+$113,915).

Đọc bảng:
- **Tỉ lệ thắng gần như không đổi** (21% / 35% / 31%) dù có BE hay không. BE chỉ chuyển ~13 lệnh/năm từ "ăn trọn SL" sang "gần hoà"; những lệnh đó nếu không BE thì một phần quay lại thành thắng nên tổng $ 2025–2026 lại cao hơn khi KHÔNG BE.
- BE giảm DD 2024 được $600 và PF 2024 từ 1.16 lên 1.25, nhưng làm 2025 mất $7.4k và chuỗi lỗ dài hơn (23 so với 12). Tổng 3 năm: không BE +$160.5k, BE +$153.8k.
- Chiến lược này thắng nhờ **TB lệnh thắng gấp 4–5 lần TB lệnh lỗ** (không nhờ winrate), nên mọi cơ chế đụng vào lệnh đang chạy (BE, trailing) đều chỉ có thể làm giảm vế này. **Kết luận: bỏ BE và trailing, giữ nguyên SL gốc + thoát MA Stack.** v1.24 đặt `EnableBreakEven = false`, `TrailMode = TRAIL_NONE`.
- Winrate 21% năm 2024 là thật: năm đó ~38% lệnh ăn trọn SL. Ai cần winrate cao hơn để chịu được về tâm lý thì bật lại BE (tỉ lệ hoà tăng, winrate không tăng).

