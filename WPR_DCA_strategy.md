# Chiến lược WPR_DCA_Hedge — reverse-engineer từ "Bot U Viet 2"

> Nguồn: `ReportHistory-265896192.xlsx` (XAUUSD, Exness Real Hedge, 2026.05.25 → 2026.06.24, 1098 lệnh).
> Không có source bot gốc — toàn bộ logic suy ra từ **comment trong từng deal** của report.
> EA clone: `vWPR_DCA_Hedge.mq5`. Script tick: `scripts/ExportTicks.mq5`.

## 1. Bản chất chiến lược

Máy trạng thái **2 engine vào lệnh + DCA có kiểm soát + 2 lớp hedge + SL cứng**, chạy trên account **Hedge mode**
(BUY-basket và SELL-basket song song). Khung tín hiệu ~M5.

| Mô-đun | Comment gốc | Hành vi đo được |
|---|---|---|
| Engine 1 — WPR fade | `WPR BUY/SELL FIRST STATE` | Williams %R cực trị → vào ngược. Luôn lot 0.01 |
| Engine 2 — Trend | `Initial BUY/SELL clear trend` | Vào thuận khi trend rõ (ADX cao). Luôn lot 0.01 |
| DCA (WPR) | `WPR ... STATE DCA T1/T2` | Nhồi lot **hình học ~1.5×**: 0.02→0.03→0.05→0.08→0.12. Step giá ~5.5 USD |
| DCA (trend) | `DCA ... trend valid` | Nhồi **phẳng 0.01**, chỉ khi trend còn đúng. Step ~4.3 USD |
| Recovery | `RECOVERY ... clear trend only` | Gỡ lỗ tăng dần, chỉ khi trend rõ |
| Fast hedge | `FAST HEDGE bad ... entry` | Mở ngược khi FIRST entry sai ngay; lot khớp size hỏng |
| Exposure hedge | `HEDGE balance ... exposure` | Cân net exposure khi lệch |
| Exit | TP rổ + `[sl ...]` | TP rổ ~3.2 USD giá; SL cứng 78 lần; close-by offset hedge |

## 2. Hiệu suất tham chiếu (mục tiêu backtest tiệm cận)

| Chỉ số | Giá trị |
|---|---|
| Net Profit | **+$985** |
| Profit Factor | **1.33** |
| Win rate | **70%** (short 74% > long 65%) |
| Max Balance DD | **18.44% ($413.34)** |
| Avg win / Avg loss | $5.21 / −$9.19 |
| Rổ blow-up tệ nhất | −$209 và −$200 |
| Chuỗi thua dài nhất | 9 lệnh |

> ⚠️ Profile DCA mean-reversion: lời gặm nhỏ đều, lỗ thưa nhưng to. Sống tốt khi gold **range**, chết khi **trend 1 chiều
> không hồi**. `MaxBasketLossUsd` là van an toàn quan trọng nhất.

## 3. Map module → code (`vWPR_DCA_Hedge.mq5`)

- Snapshot trạng thái mỗi tick: `ScanDir()` (đọc lại từ position thực → bền vững qua restart).
- Entry (bar mới): `TryEntries()` — chọn engine theo `TrendDir()` (ADX) vs `GetWpr()`.
- DCA: `ManageDca()` — WPR nhồi `InpBaseLot*InpDcaMult^tier`; trend nhồi phẳng, gate `TrendDir()==dir`.
- Recovery: `ManageRecovery()` — gate floating ≤ −`RecoveryTriggerUsd` và trend rõ.
- Hedge: `ManageHedge()` — fast hedge (FIRST non + âm) + exposure balance.
- Exit: `ManageExits()` — TP rổ scale theo lot, hard SL `MaxBasketLossUsd`.

## 4. Bảng input mặc định (bám report)

| Input | Default | Ý nghĩa |
|---|---|---|
| `InpBaseLot` | 0.01 | Lot FIRST |
| `InpWprPeriod / Buy / Sell` | 14 / −80 / −20 | Williams %R fade |
| `InpAdxPeriod / Clear` | 14 / 25 | Ngưỡng "clear trend" |
| `DcaStepUsd` | 5.0 | Khoảng cách nhồi |
| `DcaMaxTier / InpDcaMult` | 3 / 1.5 | Số tier & hệ số lot WPR |
| `RecoveryTriggerUsd / MaxAdds` | 15 / 3 | Recovery |
| `HedgeWindowSec / TriggerUsd` | 300 / 4.0 | Fast hedge |
| `ExposureMaxLot` | 0.06 | Ngưỡng cân exposure |
| `BasketTpUsd` | 3.0 | TP rổ (scale theo lot) |
| `MaxBasketLossUsd` | 120 | SL rổ chặn blow-up |
| `MaxTotalLot / MaxAccountDDPercent` | 1.0 / 20 | Trần rủi ro |

> Các threshold (WPR period, ADX, step) là **ước lượng** — chưa có source gốc. Phải backtest/forward để hiệu chỉnh.

## 5. Quy trình backtest đối chiếu

1. **Compile:** MetaEditor → mở `vWPR_DCA_Hedge.mq5` → F7 (phải 0 error).
2. **Export tick:** kéo `scripts/ExportTicks.mq5` vào chart XAUUSD → `From=2026.05.25, To=2026.06.25` → OK.
   File ra `MQL5\Files\XAUUSD_ticks_*.csv` (dùng để hiệu chỉnh WPR/step).
3. **Strategy Tester:**
   - Symbol **XAUUSD**, TF **M5**, model **"Every tick based on real ticks"**.
   - Range **2026.05.25 – 2026.06.24**, deposit **~$1000**, account **Hedging**.
   - Tinh chỉnh input để tiệm cận: **Net ≈ +$985, PF ≈ 1.33, Win ≈ 70%, MaxDD ≤ ~18%**.
   - Không kỳ vọng trùng khít — mục tiêu là **cùng profile rủi ro/đường vốn**.
4. **Forward demo** ≥ 2–4 tuần trước khi nghĩ tới live (DCA+hedge dễ blow-up khi gold trend mạnh).

## 6. Cảnh báo

- **Bắt buộc Hedge account** (Exness OK). Netting account sẽ phá logic 2 rổ + fast-hedge.
- Đây là DCA/mean-reversion → **tail-risk lớn**. Đừng tăng `InpBaseLot`/`DcaMaxTier` trước khi backtest kỹ.
- Spread XAUUSD cao → giữ `MaxSpreadPip` hợp lý; rule kinh nghiệm Spread/Step ≤ 30%.
