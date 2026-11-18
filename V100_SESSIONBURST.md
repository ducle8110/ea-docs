# v100_SessionBurst — Đón động lượng đầu phiên (8h & 15h VN)

> **Trạng thái:** v1.02 (18/11/2026). Bảy điểm mơ hồ của mô tả gốc đã được user chốt (6 qua comment trang giải thích 17/11, Q4 trả lời trong chat 18/11) và đã vào code. Còn 2 điểm chưa có câu trả lời (mục 1b).
> **File:** [`v100_SessionBurst.mq5`](../v100_SessionBurst.mq5) · Magic `800` · chạy trên chart bất kỳ (tín hiệu đọc M5/M1 qua CopyRates) · script backtest [`backtest/run_v100.ps1`](../backtest/run_v100.ps1).
> **Trang giải thích có hình vẽ (để trao đổi với người đề xuất):** https://claude.ai/artifact/VxZRwGvLEDiVmGnNZWj4dK
> **Trang soi 107 lệnh v1.02 (mục 4a):** https://claude.ai/artifact/Xti5gyPniUGYE1fnDHe4Hf

## 1. Quy tắc đã chốt (ví dụ chiều SELL, BUY đối xứng)

| Bước | Luật | Input liên quan |
|---|---|---|
| Cửa sổ | Mốc giờ **VN** (mặc định `08:00,15:00`). EA tự quy đổi sang giờ server. | `WindowsVN`, `AutoServerOffset`, `ServerUtcOffsetHours` |
| Nến M5 chuẩn | Nến M5 mở đúng giờ cửa sổ (08:00–08:05). Phải là **Marubozu**: thân ≥ 3.5 × tổng râu. Không so với nến trước (engulfing là viết nhầm, đã bỏ ở v1.01). Nến giảm → cả cửa sổ chỉ SELL. Không đạt → bỏ cửa sổ hôm đó. | `M5_MinBodyWickRatio`, `M5_MinBodyPip` (0 = tắt) |
| Cụm 3 nến M1 | 3 nến **liên tiếp**, mở từ 08:05 trở đi. M1.1 giảm · M1.2 tăng, **đỉnh thấp hơn trung điểm (H+L)/2 của nến M5** · M1.3 giảm, **đóng cửa dưới đáy M1.1** và **đóng cửa dưới đáy nến M5** (nằm trong biên M5 dễ bị hồi). | `M12_HighMode`, `M13_ContMode`, `M13_RequireBreakM5` |
| Vào lệnh | SELL market ngay khi M1.3 đóng. | `EntryMinutes` = 30 → M1.3 phải đóng trước 08:30 |
| SL | Đỉnh cao nhất của 3 nến + `SL_BufferPip` (150 pip = $1.5, user xác nhận). | `SL_BufferPip`, `MaxSL_Pip` (0 = tắt) |
| Lot | Risk% (mặc định 1% Balance) hoặc lot cố định. | `LotMode`, `RiskPercent`, `FixedLot` |
| Thoát | Không TP. Trailing theo **chân 5 nến M1 gần nhất** (cách A), **chỉ bắt đầu khi lệnh đã lãi ≥ 1R**: mỗi nến M1 đóng, SL = đỉnh cao nhất của 5 nến vừa đóng + `TrailBufferPip` (50 pip), chỉ siết. Giá đã vượt chân cụm trong nến đang chạy → đóng market (`STRUCT_BREAK`). | `TrailTF` = M1, `TrailBars` = 5, `TrailBufferPip`, `TrailStartR` = 1.0 |
| Re-entry | Lệnh đóng (SL hoặc lãi) → tìm cụm 3 nến **mới** (mở sau lúc thoát), vào trước phút thứ `ReentryMinutes`. **Dừng cửa sổ sau `MaxConsecSL` = 2 lệnh SL liên tiếp; lệnh thắng không tính và đưa chuỗi về 0.** | `MaxConsecSL`, `ReentryAfterTP`, `ReentryMinutes` |
| Giới hạn | 1 lệnh mở tại một thời điểm: lệnh 8h còn sống thì cửa sổ 15h không vào. | cố định (xem 1b) |

### 1a. Lịch sử chốt (comment 17/11/2026)
- 17/11: Q1 Marubozu thay engulfing · Q2 M1.3 đóng dưới đáy M1.1 · Q3 M1.2 chỉ cần dưới trung điểm M5 · Q4 trailing cách A, 5 nến · Q5a giới hạn 2 SL liên tục, lệnh thắng không tính · Q6 cụm phải thủng đáy M5.
- 18/11 (chat): Q4 chi tiết — chân 5 nến lấy trên **M1**, bắt đầu trail **sau khi lãi đủ 1R** → v1.02 đổi mặc định `TrailTF` M5 → M1. Real ticks 3 năm: 107 lệnh, +6.8R, PF 1.10, DD 7.2% (khớp `rt_trailM1`).

### 1b. Chưa chốt (EA đang dùng giá trị tạm)
- **Q5b:** cửa sổ 15h có mở lệnh khi lệnh 8h còn chạy? EA: không.
- Lệnh thoát có lãi vẫn được vào lại (`ReentryAfterTP` = true) — user chốt có nhưng trong 15 hoặc 30 phút; để 30.

## 2. Múi giờ (điểm dễ sai nhất)
- Input nhập giờ **VN (UTC+7)**. Live: offset server = `TimeTradeServer() − TimeGMT()`. Tester: `TimeGMT()` = giờ server nên EA dùng **lịch DST châu Âu** (Chủ nhật cuối tháng 3 → Chủ nhật cuối tháng 10 = UTC+3, còn lại UTC+2) — đúng cho Vantage.
- Broker không theo DST EU (vd Exness UTC+0): tắt `AutoServerOffset`, nhập `ServerUtcOffsetHours`.
- Đã kiểm chứng: lệnh 8h VN vào lúc 03:xx server mùa đông, 04:xx mùa hè; 15h VN lúc 10:xx / 11:xx.

## 3. Backtest bộ luật đã chốt (2024.01 → 2026.11, XAUUSD.sc, $100k, Risk 1%)

### 3a. Model 1 (1-minute OHLC, nhanh)

```powershell
.\backtest\run_v100.ps1 -From 2024.01.01 -To 2026.11.01 -Model 1 -Tag f_base
.\backtest\run_v100.ps1 -From 2024.01.01 -To 2026.11.01 -Model 1 -TrailTF 1 -Tag f_trailM1
.\backtest\run_v100.ps1 -From 2024.01.01 -To 2026.11.01 -Model 1 -TrailStartR 0 -Tag f_trail0R
.\backtest\run_v100.ps1 -From 2024.01.01 -To 2026.11.01 -Model 1 -M5Ratio 2.5 -Tag f_ratio25
```

| Tag | Khác mặc định | Lệnh | Win | Tổng R | Net | PF | DD | 8h | 15h |
|---|---|---|---|---|---|---|---|---|---|
| `f_base` | mặc định v1.01 (M5, 5 nến, trail từ 1R) | 107 (~36/năm) | 35.5% | +3.35R | +$2,177 | 1.03 | 9.1% | 55 lệnh, −3.5R | 52 lệnh, +6.9R |
| `f_trailM1` | `TrailTF` M1 (= mặc định v1.02) | 107 | 43.9% | +8.01R | +$7,336 | 1.12 | 7.2% | −3.9R | +11.9R |
| `f_trail0R` | `TrailStartR` 0 (trail ngay) | 107 | 35.5% | +11.19R | +$10,597 | 1.18 | 6.7% | +1.2R | +10.0R |
| `f_ratio25` | `M5_MinBodyWickRatio` 2.5 | 141 | 31.9% | −5.51R | −$6,926 | 0.92 | 15.3% | +2.4R | −7.9R |

Bộ lọc Marubozu 3.5: 1217 / 1464 cửa sổ-ngày bị loại (83%), 247 nến M5 hợp lệ (8h: 61 giảm + 69 tăng; 15h: 47 giảm + 70 tăng), 107 lần có cụm 3 nến (43%).

Nhận xét (mẫu ~107 lệnh, chưa đủ kết luận):
- Mặc định hoà vốn nhẹ. Hai chi tiết còn mở của trailing (Q4) quyết định phần lớn kết quả: trail ngay từ khi vào (+11R) hoặc trail M1 (+8R) đều hơn hẳn chờ 1R trên M5.
- Cửa sổ 15h ổn định hơn 8h trong mọi cấu hình. Chưa thử 14h (London mở mùa hè).
- Hạ Marubozu xuống 2.5 thêm 34 lệnh nhưng lỗ → giữ 3.5.
- Lệnh thắng lớn nhất 6–7R; hệ thống sống bằng winrate và trail, không bằng kèo lớn.

### 3b. Model 4 (real ticks, 18/11/2026) — kết quả tin được hơn

```powershell
.acktest
un_v100.ps1 -From 2024.01.01 -To 2026.11.01 -Model 4 -Tag rt_base -TimeoutSec 3000
.acktest
un_v100.ps1 -From 2024.01.01 -To 2026.11.01 -Model 4 -TrailStartR 0 -Tag rt_trail0R -TimeoutSec 3000
.acktest
un_v100.ps1 -From 2024.01.01 -To 2026.11.01 -Model 4 -TrailTF 1 -Tag rt_trailM1 -TimeoutSec 3000
.acktest
un_v100.ps1 -From 2024.01.01 -To 2026.11.01 -Model 4 -TrailTF 1 -TrailStartR 0 -Tag rt_trailM1_0R -TimeoutSec 3000
```

| Tag | Trail | Lệnh | Win | Tổng R | Net | PF | DD | 8h / 15h | 2024 / 2025 / 2026 | DD lớn nhất (R) |
|---|---|---|---|---|---|---|---|---|---|---|
| `rt_base` | M5, từ 1R (mặc định v1.01) | 107 | 35.5% | +2.30R | +$1,058 | 1.02 | 9.3% | −4.2R / +6.5R | +0.2 / +6.6 / −4.4 | 9.5R |
| `rt_trail0R` | M5, trail ngay | 107 | 34.6% | **+10.13R** | +$9,400 | 1.16 | 6.8% | +0.5R / +9.7R | +1.7 / +11.2 / −2.7 | 6.9R |
| `rt_trailM1` (= **mặc định v1.02**, tag `v102_rt` cho kết quả y hệt) | M1, từ 1R | 107 | 43.9% | +6.79R | +$6,032 | 1.10 | 7.2% | −4.6R / +11.4R | +0.9 / +7.4 / −1.4 | 7.4R |
| `rt_trailM1_0R` | M1, trail ngay | 115 | 29.6% | +5.50R | +$5,055 | 1.13 | 6.8% | −0.8R / +6.3R | −3.9 / +10.8 / −1.4 | 7.0R |

- 107 lệnh của `rt_base` **khớp 1-1** với `f_base` (cùng điểm vào, cùng lý do thoát); real ticks chỉ kém ~1R do trượt giá tại SL (4 lệnh −1.06R…−1.09R thay vì −1.00R). Logic vào lệnh không phụ thuộc mô hình tick.
- Thứ hạng giữa các biến thể giữ nguyên như Model 1: trail ngay trên M5 lãi tổng cao nhất, giảm DD từ 9.5R xuống 6.9R; trail M1 từ 1R tăng winrate (44%) nhưng lãi trung bình mỗi lệnh thắng nhỏ hơn (1.38R so với 1.80R). **User chốt M1 từ 1R** (đúng mô tả của người đề xuất); cấu hình trail ngay M5 giữ làm tham chiếu, bật bằng `TrailTF=5 TrailStartR=0`.
- Trail M1 ngay từ 0R quá sát: thoát toàn bộ bằng TRAIL_SL, thêm 8 lệnh re-entry, 2024 lỗ −3.9R → loại.
- Năm 2026 (10 tháng) âm ở mọi cấu hình, chủ yếu cửa sổ 8h. Mẫu nhỏ, chưa kết luận; theo dõi thêm khi forward test.

Kết quả bản v1.00 (engulfing + râu xuyên + 2 lệnh/cửa sổ, đã lỗi thời): 50 lệnh/3 năm, hoà vốn; xem git history nếu cần.

## 4. Chạy backtest / đọc kết quả
- Compile: copy file sang `MQL5\Experts` của terminal rồi `MetaEditor64 /compile:"...\v100_SessionBurst.mq5" /log:"..."`.
- `run_v100.ps1` ghi `.ini` + `.htm` + `.csv` vào `backtest/v100_out/`, in Net/PF/DD, số lệnh theo cửa sổ và theo lý do thoát.
- CSV lệnh (`Common\Files\v100_burst_deals_<symbol>_800[_test].csv`): 21 cột gồm cửa sổ VN, lệnh thứ mấy, OHLC nến M5 chuẩn, giờ M1.1, đỉnh/đáy cụm.
- Journal tester: `[v100] ... M5 loại: <lý do>` cho mọi cửa sổ bị loại, `nến M5 ... hợp lệ` cho cửa sổ qua lọc.

### 4a. Trang soi từng lệnh (kiểm tra EA có vào đúng luật không)
1. Dump nến M1 từ tester (terminal không giữ đủ M1 nhiều năm): compile `backtest/v100_BarDump.mq5` vào `MQL5\Experts`, chạy tester M1, Model 2 (open prices), 2023.12.28 → 2026.11.01 → `Common\Files100_bars_XAUUSD.sc.csv` (~1 triệu nến, 50 MB; ini mẫu `backtest/v100_out/bardump.ini`).
2. `python backtest/v100_trade_viewer.py backtest/v100_out/v102_rt.csv "%APPDATA%/MetaQuotes/Terminal/Common/Files/v100_bars_XAUUSD.sc.csv" backtest/v100_out/v102_rt_review.html`
3. Trang có bộ lọc (cửa sổ, chiều, lý do thoát, năm, lệnh đầu/vào lại, có lệch luật) và mỗi lệnh: hình "vào lệnh" (vùng M5 chuẩn, trung điểm, đáy/đỉnh M5, cụm M1.1–M1.3, SL) + hình "đến khi thoát" (SL ban đầu, các lần dời SL mô phỏng lại, điểm thoát) + checklist 10 luật tính lại độc lập từ nến (Marubozu, chiều, M1.1/M1.2/M1.3, thủng biên M5, SL = cụm ± 1.5, hạn 30 phút, vào lại sau khi thoát, lot 1%, thoát đúng SL/chân 5 nến).
4. Kết quả v1.02 (18/11/2026): **0/107 lệnh lệch luật**. Lưu ý khi đọc: nến là giá bid, lệnh SELL tính lãi/SL theo ask nên script ước lượng spread từ lệnh BUY (0.16); vài lệnh mô phỏng trailing lệch EA ở nến lãi sát 1R, đánh dấu "!" (thông tin), không tính lệch. Lot 500 là max của sàn (lệnh 13/06/2024 rủi ro 0.84%). Trang public: https://claude.ai/artifact/Xti5gyPniUGYE1fnDHe4Hf

## 5. Việc tiếp theo
1. ~~Chốt Q4~~ (xong 18/11). Chốt Q5b (cửa sổ 15h khi lệnh 8h còn sống).
2. ~~Chạy Model 4 (real ticks)~~ — đã chạy 18/11 (mục 3b), kết quả nhất quán với Model 1; mặc định v1.02 đã xác nhận lại (`v102_rt`).
3. Cân nhắc cửa sổ 14:00 và `MaxSL_Pip` (có lệnh SL $37 do nến tin).
4. Rút input (26 → ~12) sau khi ổn định.
