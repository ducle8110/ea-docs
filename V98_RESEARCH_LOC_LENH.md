# v98 Tyler FVG — Nghiên cứu giảm lệnh lỗ / tăng lãi lệnh thắng (18/11/2026)

Tài liệu này ghi lại một đợt nghiên cứu lớn trên EA `v98_TylerFVG` v2.05: đã thử gì, cái gì hỏng, cái gì giữ được,
và cách tái lập. Đọc trước khi định thêm bộ lọc hay đổi luật thoát lệnh — phần lớn ý tưởng "hiển nhiên" đã được thử và thất bại.

> **Trạng thái (18/11/2026):** Q và T đã được đưa vào EA chính ở **v2.06** — `QuietATR_Ratio = 0.70` (mặc định bật) và
> `TimeStopBars = 0` (mặc định tắt, đề xuất 10) theo quyết định của người dùng. E không đưa vào. Hồi quy v2.06 trùng từng xu
> với bản nghiên cứu (`QuietATR_Ratio = 0` thì trùng v2.05).

## 1. Kết luận ngắn

1. EA gốc có lợi thế thật trên 9 năm (5/2018–10/2026: 1.030 lệnh, +216R, PF 1,50), không chỉ ở 2024–2026.
   Nhưng 4/9 năm quanh hoà vốn hoặc lỗ, và **10 lệnh lớn nhất = 203R trên tổng 216R**. Đây là hệ đuôi dài:
   ~27% lệnh thắng, toàn bộ lợi nhuận nằm ở vài % lệnh lớn nhất.
2. Vì vậy **mọi thứ đụng vào lệnh đang thắng đều làm kết quả tệ đi**: BE, chốt từng phần ở 1–3R, trailing sát,
   khoá % đỉnh lãi (−$10k đến −$80k trên 9 năm).
3. Hầu hết bộ lọc lúc vào lệnh chỉ đổi **tỉ lệ dính SL**, không đổi **kỳ vọng**: lệnh SL hẹp dễ bị quét nhưng khi thắng
   thì thắng nhiều R. Lệnh lỗ và lệnh thắng lớn trông giống nhau lúc vào.
4. Chỉ **hai cải tiến** qua được mọi vòng kiểm tra (thiết kế → ngoài mẫu → 9 năm → MT5 thật):
   - **Q — bỏ tín hiệu khi thị trường quá lặng**: ATR(14) của nến xác nhận < 0,7 × trung vị ATR của 480 nến gần nhất (10 ngày).
   - **T — time stop**: sau 10 nến (5 giờ) mà giá đóng vẫn bất lợi so với giá vào thì đóng lệnh.
5. **E — thoát hai pha** (8 nến đầu thoát theo luật gốc, sau đó chỉ thoát khi EMA 9/13/21 mất thứ tự) làm tổng lãi tăng nhiều nhất
   nhưng gần như toàn bộ đến từ 5 lệnh → coi là tuỳ chọn rủi ro, không phải cải tiến chắc chắn.

## 2. Dữ liệu và công cụ

| Thứ | Ở đâu | Ghi chú |
|---|---|---|
| Nến M30 `XAUUSD.sc` 16/5/2022 → 17/11/2026 | `backtest/v98_research/rates_XAUUSD.sc_M30.csv` | tài khoản thật Vantage |
| Nến M30 `XAUUSD` 24/5/2018 → 15/10/2026 | `backtest/v98_research/rates_XAUUSD_M30.csv` | lịch sử server demo, chạy tester ngoại tuyến (tài khoản demo đã hết hạn) |
| Xuất nến | `backtest/v98_research/dump_rates.ps1` + EA `C:\MT5_Test\MQL5\Experts\DumpRates.mq5` | |
| Engine mô phỏng | `backtest/v98_research/sim2.py` | tái hiện EA; mọi biến thể là tuỳ chọn trong `DEF` |
| Chấm điểm | `lab.py`, `long_batch.py`, `long2.py`, `sig_long.py` | |
| Danh sách giả thuyết | `fams.py`, `long2.py`, `long3.py` | khoảng 250 cấu hình khác nhau, hơn 500 lượt chạy; chạy lại được |
| Bản EA nghiên cứu (Q/T/E) | `make_research_ea.py` → `C:\MT5_Test\MQL5\Experts\v98_research.mq5` | KHÔNG sửa file EA chính |
| Backtest MT5 bản nghiên cứu | `run_research.ps1 -Feed live|demo` | kết quả ở `backtest/v98_research/mt5_out/` |

Mọi backtest chạy trong **bản MT5 riêng `C:\MT5_Test` (chế độ /portable)**, không đụng terminal người dùng đang mở.
Bản này đã gỡ chart có gắn EA và các file ini đều đặt `[Experts] Enabled=0` để không bao giờ giao dịch thật.

**Độ khớp mô phỏng ↔ MT5** (1-minute OHLC): 2024–2026 trùng ~98% số lệnh, lệch R trung vị 0,001.
Với ứng viên cuối trên `.sc` 6/2022–11/2026: mô phỏng Q+T = $166,5k / PF 1,84 / DD 15,1%; MT5 = $168,1k / PF 1,86 / DD 14,6%.

Chi tiết cần nhớ khi mô phỏng: EA **xoá hết FVG đang theo dõi mỗi khi vào lệnh** (`ResetDailyFVGs` trong `ExecuteTylerEntry`);
tester MT5 **nhớ giá trị input cũ** của EA nên phải truyền input qua `[TesterInputs]`.

### Thước đo

- `$` — lãi với lot cố định 1 lot chuẩn (bị năm 2026 chi phối vì ATR 2026 gấp 4 lần 2024).
- `R` — lãi / rủi ro ban đầu. `Rw` — R kẹp trần +10 (loại ảnh hưởng của vài siêu lệnh; góp ý của Gemini).
- `A` — lãi / ATR lúc vào (tương đương lot theo biến động).
- Luôn tách theo năm; một cải tiến phải tốt hơn gốc ở ≥ 7/9 năm, có vùng phẳng tham số, và có lý do thị trường.

### Quy trình

1. Tập thiết kế 2024 → 11/2026 (`.sc`): chẩn đoán, sàng lọc đặc trưng, ~150 cấu hình.
2. Chốt 6 ứng viên + tham số → mở tập độc lập 6/2022–12/2023 đúng một lần.
3. Lấy thêm được 2018–5/2022 (chưa từng xem) → chấm lại 6 ứng viên.
4. Chạy lại toàn bộ cấu hình trên 9 năm, kiểm tra vùng phẳng, bootstrap theo tháng (100 tháng).
5. Viết ứng viên cuối vào bản EA nghiên cứu, backtest trên MT5 thật.

## 3. Chẩn đoán EA gốc

### Theo năm (XAUUSD, lot cố định 1 lot chuẩn, mô phỏng)

| Năm | Lệnh | Lãi $k | R | PF | Vàng trong năm |
|---|---|---|---|---|---|
| 2018 (nửa sau) | 56 | −2,7 | −19 | 0,55 | −1% |
| 2019 | 127 | +6,7 | +24 | 1,38 | +18% |
| 2020 | 132 | −7,5 | +3 | 0,81 | +25% |
| 2021 | 112 | +22,5 | +96 | 2,07 | −4% |
| 2022 | 118 | +1,4 | +1 | 1,06 | 0% |
| 2023 | 120 | −0,8 | +9 | 0,97 | +13% |
| 2024 | 126 | +10,0 | +13 | 1,31 | +27% |
| 2025 | 140 | +29,3 | +67 | 1,45 | +65% |
| 2026 (tới 15/10) | 99 | +98,3 | +22 | 2,14 | −5% |

Lãi không đi theo "năm vàng có xu hướng": 2020 (+25%) lỗ, 2021 (−4%) lãi lớn.

### Cấu trúc lãi lỗ (9 năm)

| Kiểu kết thúc | Lệnh | Tổng R | TB R | Số nến giữ (trung vị) |
|---|---|---|---|---|
| Dính SL | 386 (37%) | −386 | −1,00 | 3 |
| Thoát theo EMA, lỗ | 369 (36%) | −151 | −0,41 | 8 |
| Thoát theo EMA, lãi | 275 (27%) | +754 | +2,74 | 22 |

- Một nửa số lệnh dính SL là dính ngay trong 1–2 nến đầu.
- Lệnh từng lãi 2–4R trung bình chỉ giữ được 0,9R lúc thoát; nhưng mọi cách "khoá" phần lãi đó đều cắt mất lệnh lớn.
- Luật thoát "EMA 5/9/13/21 mất thứ tự" thực tế **luôn** là EMA5 cắt EMA9 (kiểm chứng: đổi sang chỉ xét EMA5×EMA9 cho kết quả y hệt).
- Khung H1 cùng luật: 559 lệnh, +$0,5k, PF 1,00 trên 9 năm → lợi thế chỉ có ở M30, không nên chạy thêm H1 để "đa dạng hoá".
- Nhạy với chi phí: thêm trượt giá $0,2/oz mỗi chiều thì tổng R giảm từ 216 xuống 102; $0,4 thì âm. Các năm ATR thấp (2018–2023)
  lợi thế rất mỏng so với chi phí.

## 4. Đã thử và loại (đừng thử lại y hệt)

Số liệu là chênh lệch so với gốc trên 9 năm trừ khi ghi khác.

| Nhóm | Đã thử | Kết quả |
|---|---|---|
| Độ dày / tuổi / số lần chạm FVG | mọi cách chuẩn hoá; FVG một lần chạm (Gemini 1B) | không quy luật; một lần chạm −$68k |
| Nến từ chối | râu ≥ 45–60%, râu ≥ 20–35%, không đóng vượt biên xa FVG (Gemini 1A), độ xuyên FVG, biên độ ≤ k ATR | râu khác 40% đều tệ hơn; lọc nến to tốt 7/9 năm nhưng xấu ở 2024 và 2026 |
| Nến xác nhận | đóng nửa thuận (Gemini 2A), cùng chiều lệnh, bỏ điều kiện đóng cửa, biên mềm 0,1–1,0 ATR (Gemini 2B) | 2A −$8k; biên mềm 0,25 đẹp ở 2024–26 (+$21k) nhưng 5/9 năm trên bộ dài, PF giảm → chỉ là "thêm lệnh cùng chất lượng" |
| Xu hướng | dốc EMA200, cách EMA200, độ mở/độ bung EMA5–21, hở EMA5–9, tuổi chuỗi xếp chồng, H4 EMA50 (Gemini 3), D1 EMA20, bỏ EMA200 | không cái nào nhất quán; H4 chỉ loại 5% lệnh |
| Kiểu vào lệnh | stop dưới nến xác nhận / nến từ chối, limit giữa nến từ chối / mép FVG / giữa nến xác nhận, vào ngay sau nến từ chối, vào lại sau SL | tất cả tệ hơn (−$10k đến −$78k) |
| SL | chân sóng 2–12 nến, đệm 0–1,0 ATR, theo nến từ chối, theo biên xa FVG, SL tối thiểu 0,5–1,5 ATR (MT5), bỏ lệnh SL quá xa | 5 nến + 0,2 ATR đã là tối ưu cục bộ |
| Thoát theo EMA | EMA5×13, 5×21, 9×21, 13×21, đóng qua EMA21/EMA50, gãy 2–3 nến liền, ân hạn 2 nến đầu (Gemini) | tệ hơn hoặc không đổi |
| Khoá lãi | BE ở 1–4R, khoá 30–50% đỉnh lãi, chandelier 3–8 ATR, trail theo chân 8–32 nến / EMA21 / EMA50, chốt 50% ở 1–8R | BE và chốt sớm −$10k…−$81k; chandelier 5 và chốt ở 8R gần như trung tính |
| Thoát sớm | đóng vượt nến từ chối / biên xa FVG, time stop 2–7 nến, lệnh chưa đạt 0,5–2R thì thoát thêm khi giá đóng cắt ngược EMA9/13/21 | tệ hơn (−$0…−$86k) |
| Kỷ luật | nghỉ sau chuỗi thua, "đánh giả" sau lệnh SL, nghỉ N nến sau SL, tối đa lệnh mỗi đợt xếp chồng | "đánh giả sau SL" đẹp ở 2024–26 (DD giảm nửa) nhưng −$15k ở 2018–22 |
| Thời gian | bỏ phiên Á, bỏ thứ Sáu, chỉ giờ 7–20 / 8–18 / 10–17 server | R tăng nhưng $ giảm, không nhất quán |
| Chiều | chỉ BUY / chỉ SELL | mỗi chiều thắng ở những năm khác nhau |
| Vào lại theo xu hướng | sau khi thoát theo EMA, trong 3–24 nến mà EMA xếp chồng lại thì vào ngay (không chờ FVG), có/không đòi lệnh trước lãi ≥ 0–4R | tất cả âm (−$10k…−$60k): bước FVG + nến từ chối + xác nhận thật sự có giá trị |
| Nhồi lệnh | tối đa 2/3/5 lệnh, nhồi khi lệnh đầu ≥ 0–3R, có/không dời SL chung | +$6k…+$16k nhưng DD 17% → 20–22%, năm 2026 âm; thực chất là tăng khối lượng |
| Khung | H1 song song | H1 không có lợi thế |
| Chi phí / biến động khác | spread ≤ 3–15% ATR, SL ≥ 5–24× spread, ATR đang tăng so với 8/24 nến trước | không cái nào hơn Q; Q không chỉ là đại diện cho chi phí |
| Lot | rủi ro % theo SL, theo ATR, lai | xem mục 6 |

## 5. Ứng viên giữ lại

### Q — bỏ tín hiệu khi thị trường quá lặng

Điều kiện: tại nến xác nhận, `ATR(14) < 0,7 × trung vị(ATR(14) của 480 nến M30 gần nhất)` thì không vào lệnh.

- Lý do: lúc thị trường lặng (cuối phiên, nghỉ lễ, trước tin) EMA vẫn xếp chồng được trên nhiễu, SL rất hẹp, chi phí spread
  chiếm tỉ trọng lớn, và ít khi có sóng theo sau.
- Vùng phẳng: ngưỡng 0,6–0,8 × cửa sổ 240–960 nến, dùng trung vị hay trung bình đều dương; có quan hệ liều lượng
  (ngưỡng 0,5 không tác dụng → 0,75 mạnh nhất → 0,9 bắt đầu cắt quá tay).
- 9 năm: loại 96 lệnh có kỳ vọng −0,28 Rw/lệnh; tốt hơn gốc 8/9 năm (theo A); bootstrap 100 tháng P(tốt hơn) = 0,93 ($) / 0,97 (Rw) / 0,99 (A).
- Giai đoạn 2018–2022 (chưa xem khi nghĩ ra ý này) dương cả 5 năm.

### T — time stop 10 nến

Điều kiện: ở mỗi nến đóng, nếu lệnh đã giữ ≥ 10 nến (tính cả nến vào) và giá đóng bất lợi so với giá vào thì đóng lệnh.

- Lý do: lệnh thắng lớn thường có lãi sớm; lệnh sau 5 giờ vẫn âm phần lớn kết thúc bằng lỗ lớn hơn.
- Vùng phẳng: 8–12 nến, ngưỡng R < 0 đến R < 0,2 đều dương; ≤ 7 nến thì tệ.
- Được chốt từ tập thiết kế (bản 8 nến), giai đoạn 2018–2022 xác nhận (+$6k, +14 Rw). Bootstrap P = 0,999 ($).
- Tác dụng nhỏ: khoảng 60 lệnh trong 9 năm đổi kết quả, tổng +$8,5k.

### Kết quả Q + T

Mô phỏng, XAUUSD 6/2018–10/2026:

| | Lệnh | Lệnh lỗ | Tổng lỗ | Tổng lãi | Ròng | PF | DD | Tổng R |
|---|---|---|---|---|---|---|---|---|
| Gốc | 1.030 | 755 | −$317k | $474k | $157k | 1,50 | 16,7% | 216 |
| Q + T | 945 | 690 | −$285k | $462k | $177k | 1,62 | 14,9% | 249 |

MT5 thật (1-minute OHLC), `XAUUSD.sc` 15/6/2022 → 18/11/2026, 1 lot chuẩn:

| | Lệnh | Lệnh lỗ | Tổng lỗ | Ròng | PF | DD | 2022 | 2023 | 2024 | 2025 | 2026 |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Gốc | 550 | 404 | −$220k | $151,8k | 1,69 | 19,9% | −8,5 | −0,4 | +5,9 | +27,7 | +127,0 |
| Q | 499 | 359 | −$202k | $161,3k | 1,80 | 16,3% | −7,7 | +1,2 | +8,2 | +32,9 | +126,7 |
| T | 551 | 408 | −$213k | $157,9k | 1,74 | 18,2% | −8,3 | −1,4 | +7,9 | +30,2 | +129,5 |
| Q + T | 499 | 362 | −$195k | $168,1k | 1,86 | 14,6% | −7,5 | +0,1 | +10,5 | +35,7 | +129,2 |
| Q + T + E | 477 | 367 | −$194k | $188,7k | 1,97 | 16,9% | −7,3 | −1,1 | +11,3 | +37,3 | +148,5 |

MT5 thật, `XAUUSD` 1/7/2018 → 15/6/2022 (ngoại tuyến): xem mục 5.1.

**Độ bền khi luật gốc bị xê dịch** (`robust.py`, 9 năm): đổi râu 35/45%, chân sóng 4/6/8 nến, đệm SL 0,1/0,35 ATR, FVG sống 16/36 nến,
FVG tối thiểu 0,5$ — ở cả 11 biến thể, Q + T vẫn cộng thêm $16–20k, PF +0,10…+0,12, DD giảm 1,6–4,5 điểm, tốt hơn 6–9/9 năm.
Tức là hiệu ứng không phụ thuộc vào bộ tham số gốc cụ thể.

**Q loại những lệnh nào** (9 năm): 95/1.030 lệnh (9%), tỉ lệ thắng 16% (gốc 27%), dính SL 52% (gốc 37%), TB −0,24R (gốc +0,21R);
phần lớn rơi vào 7–13h giờ VN (cuối phiên Á, trước phiên Âu). Có mất một lệnh +15R.

**T đóng những lệnh nào**: 63 lệnh, đóng ở TB −0,15R; cùng các lệnh đó ở bản gốc kết thúc TB −0,24R (8 lệnh dính SL, 8 lệnh lãi, lớn nhất +3,4R).

**Khớp từng lệnh mô phỏng ↔ MT5** cho cả 5 biến thể (gốc, Q, T, Q+T, Q+T+E): trùng giờ vào ~99%, cùng lý do thoát 99–100%, lệch R trung vị 0,001.

### E — thoát hai pha (tuỳ chọn, không chắc chắn)

8 nến đầu dùng luật thoát gốc; từ nến thứ 9 chỉ thoát khi EMA 9/13/21 mất thứ tự.

- 9 năm: +$50k, PF 1,63, nhưng chỉ tốt hơn 5/9 năm; 5 lệnh lớn nhất đóng góp $73k trên $77k lãi thêm; số tháng xấu (56) nhiều hơn tháng tốt (43).
- Các luật chậm lân cận (EMA5×13, 5×21, 9×21) đều âm → không có vùng phẳng theo "độ chậm".
- Nếu dùng: chấp nhận đây là đặt cược vào vài lệnh rất lớn mỗi năm.

### 5.1 Kiểm chứng MT5 trên 2018–2022

MT5 thật (1-minute OHLC, ngoại tuyến), `XAUUSD` 1/7/2018 → 15/6/2022, 1 lot chuẩn — giai đoạn chưa xem khi nghĩ ra Q và T:

| | Lệnh | Lệnh lỗ | Tổng lỗ | Ròng | PF | DD | 2018 | 2019 | 2020 | 2021 | 2022 |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Gốc | 466 | 348 | −$97,8k | $21,0k | 1,22 | 17,8% | −2,8 | +4,4 | −8,0 | +20,3 | +7,2 |
| Q + T | 430 | 317 | −$88,3k | $28,7k | 1,33 | 16,4% | −2,4 | +4,6 | −5,9 | +22,6 | +10,0 |
| Q + T + E | 421 | 329 | −$88,3k | $28,8k | 1,33 | 13,9% | −3,1 | +5,7 | −3,9 | +20,9 | +9,2 |

Q + T tốt hơn gốc ở cả 5 năm (+$7,7k, +37%). E không thêm được gì ở giai đoạn này (+$0,1k) → xác nhận E chỉ ăn ở vài lệnh lớn của 2025–2026.
Tick thật (Model 4) `XAUUSD.sc` 8/1 → 18/11/2026 trên v2.06: gốc $127,4k PF 2,53 | lọc lặng $127,3k PF 2,61 | lọc lặng + time stop 10 $129,8k PF 2,70, DD 6,8% (gốc 8,0%).

## 6. Cách tính lot (nhìn trên 9 năm)

Cùng chuỗi lệnh gốc, chuẩn hoá về cùng tổng lãi:

| Cách tính | Sụt giảm lớn nhất | Lãi / sụt giảm | Số năm lỗ | Tỉ trọng lãi năm 2026 |
|---|---|---|---|---|
| Lot cố định | 19,1 | 5,2 | 3 | 63% |
| Rủi ro cố định theo SL | 12,3 | 8,1 | 1 | 10% |
| Cố định theo ATR | 30,9 | 3,2 | 3 | 20% |

Lot cố định thực chất là **rủi ro tăng dần theo giá vàng**: ATR 2026 gấp 4 lần 2024 nên mỗi lệnh 2026 "nặng" gấp 4.
Nó thắng ở 2023–2026 vì năm tốt nhất trùng với năm ATR cao nhất. Trên 9 năm, rủi ro cố định theo SL cho đường vốn đều hơn.
Đây là quyết định của người dùng (hiện chọn lot cố định); mô phỏng chưa tính trượt giá cho lệnh SL rất hẹp.

## 7. Tái lập

```powershell
cd backtest\v98_research
.\dump_rates.ps1 -Symbol XAUUSD.sc -From 2022.06.01                                   # nến tài khoản thật
.\dump_rates.ps1 -Symbol XAUUSD -From 2019.01.01 -Login 26205289 -Server VantageMarkets-Demo   # nến 2018+ (ngoại tuyến)
python validate2.py            # đối chiếu engine với CSV lệnh MT5
python long_batch.py setup lọc # ~190 cấu hình trên 9 năm
python long2.py atr ts combo   # vùng phẳng Q, T và tổ hợp
python sig_long.py             # bootstrap theo tháng + đối chiếu trên .sc
python make_research_ea.py     # tạo bản EA nghiên cứu (Q/T/E)
.\run_research.ps1 -Feed live -Quiet 0.7 -TimeStop 10 -Tag live_QT
```
