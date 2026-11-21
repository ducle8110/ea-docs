# Tài liệu bàn giao — `v104_Portfolio` (v2.01, sáu bot)

> **Tên cũ:** `v104_Portfolio3` (tới v1.25, bốn bot). Từ v2.00 file đổi tên thành `v104_Portfolio` và có thêm hai bot v105, v106.
> **File nguồn:** [`v104_Portfolio.mq5`](../v104_Portfolio.mq5) + hai chỉ báo nhúng [`v104_Line.mq5`](../v104_Line.mq5), [`v104_MACD.mq5`](../v104_MACD.mq5) · Biên dịch: [`compile_v104.ps1`](../compile_v104.ps1) (tự biên dịch hai chỉ báo trước) · Backtest: [`backtest/run_v104.ps1`](../backtest/run_v104.ps1)
> **Preset chạy demo:** [`v104_demo.set`](../v104_demo.set) · **Báo cáo sau demo:** `python -X utf8 backtest/v104_demo_report.py`
> **Soi từng lệnh, Gốc và A + B vẽ chồng trên nến:** [`v104_AB_samples.html`](../v104_AB_samples.html) — dựng bằng `python -X utf8 backtest/v104_ab_viewer.py`
> **Kết quả thô:** `backtest/v104_out/v200_*` (sáu bot), `v121_*` và `v120_*` (bốn bot), `v111_*` (v1.11), `v110_*` (v1.10 của Gemini)
> **Bản sao lưu:** `backtest/v104_out/v104_Portfolio3_v125_4bot.mq5` (bốn bot, ngay trước khi ghép v105 + v106), `v104_Portfolio3_v110_goc.mq5` (gốc của Gemini), `v104_Portfolio3_v111.mq5` (trước khi gộp v98), `v104_Portfolio3_v120_68input.mq5` (trước khi rút gọn input)
> **Hai bot mới:** nghiên cứu và số kiểm chứng riêng của v105, v106 nằm ở [`V105_V106_REVIEW_V111.md`](V105_V106_REVIEW_V111.md). Bản v2.00 được dựng từ bản 4 bot bằng `backtest/v104_out/build_v200/build_portfolio.py` (kèm hai khối mã `blocks_trade.mq5`, `blocks_dash.mq5`) — giữ lại để xem chính xác đã thêm gì.
> **Ảnh dashboard:** `backtest/v104_out/dashboard_v200_gon.png` (bảng gọn), `dashboard_v200_du.png` (bảng đủ), `dashboard_v200_info_v105.png` (bảng giải thích).
> Cập nhật 21/11/2026. Mọi con số dưới đây lấy từ MT5 Strategy Tester, trừ chỗ ghi rõ là mô phỏng.

---

## 1. Kiến trúc: sáu chiến lược độc lập trong một EA

Treo trên **một** biểu đồ XAUUSD, khung nào cũng được (khuyên M15), tài khoản Hedging. Mỗi bot có `CTrade` và Magic riêng, tự đọc khung thời gian của mình và giữ tối đa một vị thế.

| Bot | Magic | Khung | Tín hiệu | Chiều | Thoát lệnh (mặc định) |
|---|---|---|---|---|---|
| `v99` | 799 | M30 | Phá hộp Donchian 20 nến + EMA 50/200 + 2 lọc bẫy | Buy + Sell | SL 1.5×ATR(14); Chandelier 3.0×ATR xét khi đóng nến; không TP, giữ qua cuối tuần |
| `v102` | 802 | M15 | Heikin Ashi + EMA 100/200 + RSI(7), phiên 08–20 server | Chỉ Buy | SL đáy 12 nến − 0.25×ATR (trần 0.80% giá); TP 2R |
| `v103` | 803 | H4 → H1 → M15 | MACD đa khung lật histogram, phiên 08–20 server | Buy + Sell | SL đỉnh/đáy 12 nến ± 0.25×ATR (trần 0.80% giá); TP 2R |
| `v98` | 798 | M30 | Tyler FVG + EMA xếp chồng 5/9/13/21/200, lọc thị trường lặng | Buy + Sell | SL chân sóng 5 nến + 0.20×ATR; thoát khi EMA mất xếp chồng; không TP |
| `v105` | 805 | M30 (lọc H4) | Bollinger(20, 2) nằm trong Keltner(20, 1.5 ATR) ở ≥ 4/6 nến rồi nến đóng ra ngoài dải; thuận EMA 50/200 M30 và H4; EMA50 dốc ≤ 0.3 ATR / 4 nến; phiên 08–20 server | Buy + Sell | SL 1.5×ATR(14); đóng nửa ở Chandelier 3.0, nửa ở 6.0; nửa đầu chốt lãi ≥ 1R thì dời SL về hoà vốn; không TP |
| `v106` | 806 | M30 | Nến đóng thoát khỏi vùng giá trị (70% khối lượng) của Volume Profile 3 ngày trước, sau khi ≥ 4/12 nến đóng trong vùng; thuận EMA 50/200; phiên 08–20 server | **Chỉ Buy** | SL đáy 8 nến − 0.20×ATR (tối thiểu 1.5×ATR); chốt nửa ở 2R + dời hoà vốn; nửa còn lại TP 5R, sau TP1 nến đóng lùi 2R từ đỉnh thì đóng hết |

- **v98 là lõi của `v98_TylerFVG.mq5` v2.06** chuyển sang, không kèm dashboard, hộp FVG và đường EMA của bản riêng. Đừng chạy bản v98 riêng song song trên cùng tài khoản: trùng Magic 798.
- **v105 và v106 là lõi của hai bản riêng v1.11** (`v105_BB_KeltnerSqueeze.mq5`, `v106_VP_ValueEscape.mq5`), tự tính chỉ báo y như bản riêng. Cũng đừng chạy hai bản riêng song song: trùng Magic 805 / 806.
- **v106 mặc định chỉ Buy và nên coi là bot thử nghiệm.** Bộ tham số của nó là điểm cao nhất trong vùng lân cận ở backtest (kỳ vọng thực tế khoảng một nửa), và chiều Sell 8,5 năm gần như không lãi. Chi tiết ở `V105_V106_REVIEW_V111.md` mục 3.
- **v105 cùng họ với v99** (phá vỡ M30 + EMA 50/200 + Chandelier 3/6): khoảng một nửa số lệnh v105 vào cách lệnh v99 cùng chiều không quá 2 giờ. Hai bot hay cùng thắng cùng thua.
- **Hiệu năng:** việc nặng chỉ chạy khi có nến M15/M30 mới. Tester xử lý 250 triệu tick (4,4 năm, bốn bot, có ghi log) trong 169 giây, tức khoảng 0,7 phần triệu giây mỗi tick.

---

## 2. Input (14 cái)

| Nhóm | Input | Mặc định | Ý nghĩa |
|---|---|---|---|
| CHUNG | `RiskPct` | 0.50 | % vốn rủi ro mỗi lệnh, tính riêng cho từng bot |
| | `FixedRiskCapital` | 0 | Vốn tính rủi ro cố định (USD); 0 = theo số dư, tức lãi kép |
| | `MaxSpreadPip` | 50 | Spread tối đa để vào lệnh (vàng 50 pip = 0,50 USD) |
| | `ShowDashboard` | true | Hiện bảng 6 bot trên chart |
| | `D1_DownGuard` | true | Chốt chặn ngày: không bot nào mở lệnh Buy khi xu hướng ngày đang giảm (mục 9) |
| v99 | `v99_Mode` | Buy + Sell | Tắt / Buy + Sell / chỉ Buy / chỉ Sell |
| | `v99_TwoTierExit` | true | **A**: thoát 2 tầng, nửa lệnh ở Chandelier 3.0, nửa ở 6.0 (false = đóng hết ở 3.0 như bản gốc) |
| v102 | `v102_Mode` | chỉ Buy | Tắt / Buy + Sell / chỉ Buy / chỉ Sell |
| | `v102_TrailExit` | true | **B**: TP 4R + trailing cách đỉnh 2R (false = TP 2R cố định như bản gốc) |
| v103 | `v103_Mode` | Buy + Sell | Tắt / Buy + Sell / chỉ Buy / chỉ Sell |
| v98 | `v98_Mode` | Buy + Sell | Tắt / Buy + Sell / chỉ Buy / chỉ Sell |
| | `v98_TimeStopBars` | 0 | Đóng lệnh nếu sau N nến M30 giá đóng vẫn bất lợi (0 = tắt; đề xuất 10) |
| v105 | `v105_Mode` | Buy + Sell | Tắt / Buy + Sell / chỉ Buy / chỉ Sell |
| v106 | `v106_Mode` | chỉ Buy | Tắt / Buy + Sell / chỉ Buy / chỉ Sell |

Mọi thông số tín hiệu của v105 và v106 (kể cả hai bộ lọc H4 và độ dốc của v105) là hằng số trong khối `HẰNG SỐ ĐÃ CHỐT QUA BACKTEST`.

Tới v1.20 EA có 68 input. Ở v1.21:

- **57 thông số thành hằng số**, nằm trong khối `HẰNG SỐ ĐÃ CHỐT QUA BACKTEST` đầu file, giữ nguyên tên cũ: Magic, chu kỳ Donchian, hệ số ATR, ngưỡng RSI, giờ phiên, đệm SL, lọc FVG… Muốn thử giá trị khác thì sửa hằng rồi biên dịch lại.
- **Gộp:** bật/tắt + cho phép Buy + cho phép Sell của mỗi bot thành một lựa chọn `…_Mode`; `v99_Chandelier2_ATR` thành công tắc `v99_TwoTierExit`; `v102_TP_RR` + `v102_Trail_R` thành công tắc `v102_TrailExit`; bốn `RiskPct_…` thành một `RiskPct` chung (chia lệch tỷ trọng giữa các bot không cải thiện lãi/DD).
- **Bỏ hẳn:** chế độ lot cố định (`LotMode`, `FixedLot_…`).
- **Đổi mặc định:** `RiskPct` 1.00 → 0.50.

---

## 3. Lịch sử thay đổi

### v2.01 (21/11/2026) — chốt chặn xu hướng ngày
- Input mới `D1_DownGuard` (mặc định bật): khi EMA50 của nến ngày nằm dưới EMA200 quá 0,5 ATR ngày thì không bot nào được **mở** lệnh Buy. Lệnh Sell và lệnh đang giữ không bị đụng tới. Lý do, số kiểm chứng và giới hạn ở mục 9.
- Dashboard: cạnh dòng trạng thái có "Ngày: TĂNG / chuyển tiếp / GIẢM · chặn BUY" kèm giá trị chỉ số (ảnh `backtest/v104_out/dashboard_v201_chan_buy.png`). Log sự kiện thêm `D1_REGIME` (đổi trạng thái) và `D1_BLOCK` (một tín hiệu Buy bị chặn).
- Hồi quy: tắt `D1_DownGuard` thì file lệnh sáu bot giống từng byte với v2.00 (1.923 lệnh, feed live).

### v2.00 (21/11/2026) — sáu bot, đổi tên file
- **Đổi tên** `v104_Portfolio3` → `v104_Portfolio`. File log thành `v104_portfolio_*.csv`; lần đầu chạy bản mới, log tên cũ `v104_portfolio3_*` được chép sang tên mới để ghi nối tiếp (file equity có thêm cột nên bản cũ được đổi tên `..._cu_<ngày>.csv`). Tên biến toàn cục (`V104_…`) không đổi nên lệnh đang mở của bốn bot cũ vẫn khôi phục được.
- **Ghép v105 và v106** (mục 1). Hồi quy trên feed live 2022.06 → 2026.11:
  - Tắt v105 + v106: file lệnh của bốn bot cũ **giống từng byte** với v1.23 (1.741 lệnh, lượt A + B).
  - Chỉ bật v105: **70/70 lệnh trùng** bản v105 riêng v1.11 (giờ vào, giờ ra, lot, giá, lãi USD).
  - Chỉ bật v106 (Buy + Sell): **189/189 lệnh trùng** bản v106 riêng v1.11.
  - Hai phép so trên dùng bản biên dịch tắt bước thử lại lệnh đóng (`TierExitRetry = false`), vì hai bản riêng không thử lại.
- **Lệnh đóng của v105 / v106 bị từ chối được thử lại mỗi 5 giây** (như v99, v98 từ trước). Hay gặp ở tick 01:00 khi phiên chưa mở. Ảnh hưởng tới kết quả: xem mục 4.0.
- **Dashboard** chuyển từ 4 cột sang lưới 3 cột × 2 hàng, mỗi bot một ô, có hai dạng: gọn (mặc định, 8 dòng mỗi bot, cao khoảng 440 px — thấp hơn bảng 4 cột cũ) và đủ (12 dòng mỗi bot, khoảng 650 px). Dòng cách thoát của ô đổi theo lệnh đang giữ. Nút `i` và nút `Chart` có cho cả sáu bot (mục 6). Bố cục đã chụp kiểm tra trong tester trực quan ở cả năm trạng thái (gọn, đủ, thu gọn, bảng giải thích v105 và v106); các nút chưa bấm thử trên chart thật vì tester không phát sự kiện click.
- 13 input (thêm `v105_Mode`, `v106_Mode`).

### v1.25 (20/11/2026)
- Nút `i` ở mỗi cột dashboard mở bảng giải thích cách bot vào lệnh, đặt SL và thoát lệnh, kèm số liệu lúc đó. Nút Chart bật thì vẽ lệnh đang mở (giá vào, SL, TP, mức thoát v99) có nhãn R. Màu đường tự đổi theo nền chart sáng / tối.

### v1.24 (19/11/2026)
- A và B **mặc định bật** (`v99_TwoTierExit = true`, `v102_TrailExit = true`). Code giao dịch không đổi so với v1.23. Trong các bảng ở mục 4, "mặc định" vẫn nghĩa là cấu hình gốc tắt A và B; `run_v104.ps1` luôn truyền rõ hai công tắc này (`-A`, `-B`).
- Lưu ý MT5: EA đã nằm sẵn trên chart sẽ giữ giá trị input cũ khi thay file `.ex5`. Muốn nhận mặc định mới phải gỡ EA ra rồi gắn lại, hoặc bấm Reset ở tab Inputs.

### v1.23 (19/11/2026) — sửa theo lần gắn lên chart thật đầu tiên
- **Hiển thị trên chart mặc định bật** cho các bot đang chạy (lần đầu gắn EA chart không còn trống); bấm nút `Chart` để tắt từng bot.
- Sửa nhãn rỗng hiện chữ `Label` dưới mỗi cột (chuỗi chưa gán trong MQL5 là NULL, `NULL == ""` cho kết quả sai) và khung bảng thiếu chiều cao.
- **Không gửi lại lệnh bị từ chối vì lý do không tạm thời.** Log chạy thật cho thấy lệnh v99 bị `not enough money` được gửi lại 24 lần, cách nhau 5 giây. Nay chỉ thử lại với requote, giá đổi, timeout, mất kết nối.
- **Không mở lệnh ở nến mà EA vừa khởi động lại.** Đổi khung chart, đổi input hay mở lại terminal làm EA nạp lại; trước đây tín hiệu của nến vừa đóng bị xét lại giữa nến và có thể vào lệnh lần hai.
- **Không vào lệnh trên tài khoản không phải Hedging** (EA cần mỗi bot một vị thế riêng trên cùng mã).
- Dashboard có dòng trạng thái giao dịch cạnh tiêu đề: sẵn sàng / Algo Trading đang tắt / số dư bằng 0 / tài khoản không phải Hedging.
- Hồi quy: file lệnh của lượt bốn bot mặc định và lượt A + B vẫn giống từng byte với v1.20.

### v1.22 (19/11/2026)
- Nút **Chart** ở mỗi cột dashboard: bật thì vẽ các đường và vùng của bot đó lên chart, tắt thì gỡ (mục 6). Chỉ để quan sát, không đụng logic vào/thoát lệnh; file lệnh của lượt bốn bot mặc định và lượt A + B vẫn giống từng byte với v1.20.
- Hai chỉ báo nhúng trong `.ex5`: `v104_Line` (EMA tính trên khung của bot, vẽ đúng ở mọi khung chart) và `v104_MACD` (MACD tính như bot v103, Signal là EMA 9). Chạy thật không cần cài thêm gì.

### v1.21 (19/11/2026)
- Rút 68 input còn 11 (mục 2). Hồi quy: file lệnh của lượt bốn bot mặc định và lượt A + B giống từng byte với v1.20.
- Dashboard 4 cột, mỗi bot một cột: trạng thái và cách thoát, lệnh đang giữ (giá vào, SL/TP, R hiện tại, đỉnh/đáy, thời gian giữ), tín hiệu ở nến vừa đóng, kết quả hôm nay / từ đầu / tỉ lệ thắng, sự kiện gần nhất.

### v1.20 (19/11/2026)
- Gộp v98 Tyler FVG thành bot thứ tư. Hồi quy: **501/501 lệnh trùng** bản v98 riêng ở mọi trường (cùng lot cố định, 2022.06 → 2026.11). Ba bot cũ không đổi: file lệnh giống từng byte với v1.11.
- Lệnh đóng của v98 bị từ chối ở tick 01:00 nay được thử lại (5 lệnh đổi giờ đóng từ 01:30 về 01:01, tổng +0,3R).
- Bảng dashboard bốn bot, có nút thu/phóng; trạng thái thu/phóng được nhớ qua lần khởi động lại.
- Log cho demo: thêm 7 cột vào file lệnh, thêm file sự kiện và file equity (mục 6).

### v1.11 (19/11/2026)
Với input mặc định, v1.11 vào đúng 1.436 lệnh của v1.10.

| Lỗi ở v1.10 | Sửa |
|---|---|
| `ProcessBar102/103` nạp 550/220 nến ở **mọi tick** | Hỏi `iTime` trước, chỉ nạp khi có nến M15 mới. Backtest 4,4 năm: ~15 phút → ~2 phút |
| Lý do đóng lệnh v99 bị ghi trống (`ZeroMemory` biến string thành NULL, mà `NULL == ""` là sai) | `ResetState99()` gán lại chuỗi rỗng, so bằng `StringLen` |
| Lệnh **đóng** v99 ở tick 01:00:xx bị từ chối `[Market closed]` (phiên chỉ khớp từ 01:01) rồi bỏ luôn | Ghi nhớ và thử lại mỗi 5 giây |
| Tick mở nến spread > `MaxSpreadPip` hoặc lệnh lỗi thì mất tín hiệu | `EntryRetrySec` (mặc định 120): thử lại trong 120 giây đầu nến |
| Cột `r_multiple` không trừ swap | Thêm cột `net_r` |

Lệnh **vào** bị từ chối vì `Market closed` (nến 01:00, ngay sau giờ nghỉ) thì cố ý không thử lại: 11 lệnh kiểu này trong backtest lỗ gần hết (−9,6R).

Thêm ở v1.11, đều mặc định tắt: `FixedRiskCapital`, thoát 2 tầng cho v99 (A), TP 4R + trailing cho v102 (B), sàn SL cho v102 (C).

---

## 4. Kết quả kiểm chứng

Ký hiệu: **A** = `v99_TwoTierExit` (Chandelier 3.0 + 6.0) · **B** = `v102_TrailExit` (TP 4R + trailing 2R) · **C** = bỏ lệnh v102 có SL dưới 0,25% giá (đã loại, nay là hằng `v102_MinSL_PctPrice = 0`).
**R ròng** = đã trừ swap. Mọi lượt đều 1% rủi ro mỗi lệnh mỗi bot; vốn cố định $100.000 nên 1R = $1.000.

### 4.0. Sáu bot (v2.00) — feed live `XAUUSD.sc`, 2022.06.01 → 2026.11.17

Cấu hình mặc định của EA: A + B bật, v105 Buy + Sell, v106 chỉ Buy.

| Cấu hình | Lệnh | Lãi ròng | PF | Balance DD lớn nhất | Equity DD lớn nhất | R ròng | DD theo R (lệnh đóng) |
|---|---|---|---|---|---|---|---|
| Bốn bot cũ (A + B) | 1.741 | +$454.783 | 1,43 | $65.084 | $68.397 | +469,4 | 63,3 |
| **Sáu bot** | 1.923 | **+$606.286** | **1,52** | **$60.183** | **$62.439** | **+620,9** | **58,4** |

R ròng theo năm:

| | 2022 (từ T6) | 2023 | 2024 | 2025 | 2026 |
|---|---|---|---|---|---|
| Bốn bot cũ (A + B) | −49,2 | +96,4 | +86,6 | +217,6 | +118,0 |
| Sáu bot | −40,4 | +135,3 | +107,3 | +271,5 | +147,2 |

Theo bot (sáu bot): v98 +119,1R / 501 lệnh · v99 +190,1R / 582 · v102 +100,3R / 357 · v103 +59,9R / 301 · **v105 +113,2R / 70** · **v106 +38,3R / 112**.

- **Hai bot mới thêm khoảng 151R (+32%) mà sụt giảm không tăng**, vì đường vốn của v105 ít sụt (lệnh đóng: 4,3R) và nó lãi cả trong 2022 là năm bốn bot cũ lỗ.
- **Phần đóng góp gần như toàn của v105.** v106 chỉ Buy thêm +38R trong 112 lệnh (PF 1,57); chạy cả hai chiều thì +51R / 189 lệnh (PF 1,44) nhưng sụt giảm của riêng nó tăng từ 8,4R lên 12,3R.
- **Số của v105 và v106 là số trong mẫu.** Hai bộ lọc của v105 và bộ tham số của v106 đều được chọn trên dữ liệu gồm cả giai đoạn này; kỳ vọng thực tế thấp hơn, nhất là với v106 (`V105_V106_REVIEW_V111.md` mục 1.3 và 3.1).
- **Số tháng âm không đổi:** 19/54 tháng ở cả hai cấu hình. Sáu bot vẫn cùng lỗ khi vàng đi ngang.
- **Bật chốt chặn ngày (v2.01, mặc định):** cùng giai đoạn này sáu bot còn 1.770 lệnh, lãi ròng +$623.278, PF 1,59, Equity DD $53.579, R ròng +638,5, DD theo R 46,4; năm 2022 từ −40R về 0, năm 2026 từ +147R xuống +125R (mục 9).
- **Thử lại lệnh đóng của v105 / v106:** trong giai đoạn này không lệnh đóng nào của hai bot bị từ chối, nên bật hay tắt bước thử lại cho kết quả y hệt.

### 4.1. Bốn bot (v1.20) — feed live `XAUUSD.sc`, 2022.06.01 → 2026.11.17

| Cấu hình | Lệnh | Lãi ròng | PF | Equity DD lớn nhất | R ròng | DD theo R (lệnh đóng) |
|---|---|---|---|---|---|---|
| Mặc định | 1.937 | +$398.577 | 1,35 | $65.926 | +408,0 | 61,2 |
| **A + B** | 1.741 | **+$454.783** | **1,43** | $68.397 | **+469,4** | 63,3 |

R ròng theo năm:

| | 2022 (từ T6) | 2023 | 2024 | 2025 | 2026 |
|---|---|---|---|---|---|
| Mặc định | −44,4 | +63,9 | +61,4 | +177,9 | +148,9 |
| A + B | −49,2 | +96,3 | +86,6 | +217,5 | +118,0 |

Theo bot (mặc định): v98 +119,1R / 501 lệnh · v99 +168,9R / 739 · v102 +60,1R / 396 · v103 +59,9R / 301.

- **v98 thêm lãi nhưng không giảm sụt giảm.** Nó cũng lỗ trong 2022 (−16,5R), nên DD danh mục tăng từ 47,9R (ba bot) lên 61,2R (bốn bot).
- **Với bốn bot, A + B tăng lãi 15% nhưng DD gần như không đổi** (61 → 63R).
- **Một quãng ngắn không nói lên gì:** trong backtest, cứ ba quãng 63 ngày thì có khoảng một quãng danh mục âm; từng bot riêng âm 33–38% số quãng.
- **Lot trần 500 trên `.sc`:** SL của v98 có khi rất nhỏ (nhỏ nhất $0,85), nên 41/501 lệnh v98 chạm trần ở vốn $100.000 và thực tế rủi ro dưới 1%.

### 4.2. Ba bot (v1.11) — feed live, cùng giai đoạn

| Cấu hình | Lãi ròng | PF | Equity DD lớn nhất | R ròng | DD theo R | 12 tháng tệ nhất |
|---|---|---|---|---|---|---|
| Gốc (= v1.10) | +$286.723 | 1,32 | $51.251 | +288,9 | 47,9 | −27,7R |
| **A + B** | **+$342.929** | **1,43** | **$47.318** | **+350,2** | **44,1** | −21,2R |
| A + B + C | +$336.148 | 1,45 | $45.219 | +343,4 | 39,9 | −13,0R |

Theo bot (R ròng · PF · DD theo R · số lệnh): v99 +168,9 · 1,36 · 30,8 · 739 → **A** +190,1 · 1,50 · 24,1 · 582; v102 +60,1 · 1,24 · 15,5 · 396 → **B** +100,3 · 1,45 · 17,5 · 357; v103 +59,9 · 1,33 · 9,1 · 301 (không đổi).

### 4.3. Ba bot trên feed demo `XAUUSD`, 2018.07 → 2026.10 (M1 OHLC, lãi kép 1%/bot)

| Cấu hình | Lãi ròng | PF | Equity DD tương đối lớn nhất | Số dư thấp nhất | R ròng | R theo giá |
|---|---|---|---|---|---|---|
| Gốc v1.10 | +$194.795 | 1,17 | **77,6%** | $24.634 (08/2023) | +177,8 | +281,7 |
| A + B + C | +$659.465 | 1,31 | **63,1%** | $39.078 (12/2019) | +286,6 | +413,0 |
| A + B (ghép từ hai lượt) | — | — | — | — | +295,1 | +424,8 |

| R ròng theo năm | 2018 (từ T7) | 2019 | 2020 | 2021 | 2022 | 2023 | 2024 | 2025 | 2026 |
|---|---|---|---|---|---|---|---|---|---|
| Gốc | −51,8 | −22,4 | +34,3 | −39,2 | +3,1 | +37,1 | +26,7 | +104,3 | +85,6 |
| A + B | −49,6 | +18,8 | +57,1 | −48,9 | +7,3 | +64,9 | +51,9 | +136,2 | +57,4 |

Chưa có lượt bốn bot trên feed demo dài.

### 4.4. Đọc kết quả

- **A và B được xác nhận trên cả hai feed.** A tăng lãi v99 và giảm DD của bot. B tăng lãi v102 rõ nhưng DD của bot nhích lên.
- **Cái giá:** năm 2026 kém hơn ở cả hai feed (v99 −28R trên live), 2021 kém hơn trên demo.
- **C không đáng bật.** Chồng lên B thì mất 7–8R trên cả hai feed, DD của v102 gần như không đổi.
- **Thay đổi luật thoát làm tăng lãi nhiều hơn là giảm lỗ.** Các bot vẫn cùng lỗ khi vàng đi ngang: 2018, 2021, nửa cuối 2022.
- **Trước 2023 hệ thống gần như hoà vốn.** Demo 2019.01 → 2022.05, ba bot: gốc +56R theo giá trên 1.077 lệnh (PF 1,08); A + B +128R (PF 1,19).
- **v99 sống nhờ vài lệnh lớn:** 20 lệnh lớn nhất = 202R trên tổng 190R (cấu hình A).

### 4.5. Nhìn theo từng lệnh (ba bot, live)

Hai bản vào lệnh theo cùng tín hiệu, nên mỗi lệnh của bản mới ghép được với đúng một lệnh của bản gốc. Bản mới giữ lệnh lâu hơn nên bỏ qua một số lệnh mà bản gốc vào lại.

| Bot | Kết cục | Số lệnh | Gốc | Mới | Chênh |
|---|---|---|---|---|---|
| v99 | Nửa sau gồng thêm được | 61 | +112,9R | +256,9R | +144,1R |
| v99 | Nửa sau trả bớt lãi (chưa về SL) | 71 | +302,4R | +232,1R | −70,3R |
| v99 | Gần như nhau | 39 | +67,3R | +65,7R | −1,6R |
| v99 | Nửa sau quay về SL ban đầu | 88 | +12,8R | −38,4R | −51,2R |
| v99 | Cùng dính SL | 323 | −326,2R | −326,2R | 0 |
| v99 | Lệnh bản gốc vào lại, bản mới bỏ qua | 157 | −0,3R | — | +0,3R |
| v102 | Chạy tới TP 4R | 68 | +134,0R | +265,9R | +131,9R |
| v102 | Trailing đóng trước 4R | 76 | +150,6R | +54,7R | −95,8R |
| v102 | Cùng dính SL | 213 | −220,3R | −220,3R | 0 |
| v102 | Lệnh bản gốc vào lại, bản mới bỏ qua | 39 | −4,1R | — | +4,1R |

- **A kém nhiều lần hơn là hơn:** kém bản gốc từ 0,3R trở lên ở 126 lệnh (−116R), hơn từ 0,3R trở lên ở 61 lệnh (+144R).
- **B là đánh đổi gần 50/50:** trong 144 lệnh bản gốc chốt 2R, 68 lệnh chạy tiếp tới 4R, 76 lệnh bị trailing đóng ở trung bình +0,7R.
- **Dời SL của nửa sau** (v99, sau khi nửa đầu thoát) về hoà vốn làm kết quả kém đi trên cả hai feed; khoá +0,5R hay +1R chỉ hơn 1–3% (mô phỏng M30).
- Đường Chandelier dựng lại từ nến trùng nến thoát của MT5 ở 488/488 lệnh.

### 4.6. Giới hạn của dữ liệu

- "Real ticks" chỉ có từ 07/01/2026. Lượt live 2022–2026 có History Quality **19%**; phần còn lại là tick sinh từ nến M1.
- Tester áp swap hiện tại (BUY −$0,69/oz/đêm) cho cả quá khứ. Với giá vàng 2018–2021 mức này quá nặng, nên R ròng các năm đó bị bi quan; R theo giá thì lại bỏ qua swap.
- Feed demo năm 2018 thiếu khoảng 25% số nến M30.
- Sàn chỉ khớp lệnh XAUUSD từ 01:01 giờ server; tick 01:00:xx bị `Market closed`.

---

## 5. Cấu hình khuyến nghị

```
v99_TwoTierExit = true        (A)
v102_TrailExit  = true        (B)
RiskPct         = 0.25 – 0.50
```

Sụt giảm lịch sử trên live là **58–63R** (sáu bot 58R, bốn bot 61–63R). Ở 1%/bot đó là khoảng 60% vốn; ở 0,5% khoảng 30%; ở 0,25% khoảng 15%. Trên lịch sử dài (demo, ba bot) sụt giảm còn sâu hơn: 64–82R. Chưa có lượt sáu bot trên feed demo dài.

Sáu bot có thể cùng giữ sáu lệnh một lúc, phần lớn cùng chiều: `RiskPct` là rủi ro **mỗi bot**, tổng rủi ro mở cùng lúc có thể tới 6 × `RiskPct`.

File `v104_demo.set` đặt sẵn A + B, v106 chỉ Buy và 0,5% mỗi bot.

---

## 6. Chạy demo và lấy dữ liệu

1. Tài khoản demo **Hedging còn hoạt động và có số dư**, đòn bẩy ≥ 1:200. Copy `v104_Portfolio.ex5` vào `MQL5\Experts\`, mở một chart XAUUSD M15, kéo EA vào, bật Algo Trading. Ở tab Inputs bấm **Load** và chọn `v104_demo.set`. Dòng trạng thái cạnh tiêu đề dashboard phải ghi "Sẵn sàng giao dịch".
   - **Đang chạy bản cũ `v104_Portfolio3`:** gỡ nó khỏi chart rồi mới gắn `v104_Portfolio` (hai bản cùng Magic, không chạy song song). Lệnh đang mở của bốn bot cũ được bản mới nhận lại và quản lý tiếp. Nên xoá `v104_Portfolio3.ex5` khỏi `MQL5\Experts\` để khỏi gắn nhầm.
2. **Tắt các bản v98, v105, v106 riêng** trên tài khoản đó.
3. **Dashboard** ở góc trái trên, rộng khoảng 1.070 px: dòng tiêu đề (trạng thái giao dịch, sự kiện gần nhất), một dòng tài khoản, bên dưới là lưới 3 cột × 2 hàng, mỗi bot một ô.
   - **Bảng gọn** (mặc định, cao khoảng 440 px): mỗi ô 8 dòng — tên bot, chiều lệnh + cách thoát, lệnh đang giữ (giá vào, R và USD hiện tại, SL/TP, thời gian giữ), tín hiệu ở nến vừa đóng, kết quả hôm nay và từ đầu.
   - **Bảng đủ** (bấm nút `Bảng: gọn` để đổi, cao khoảng 650 px): thêm đỉnh/đáy theo R của lệnh đang giữ và sự kiện gần nhất của từng bot. Cửa sổ chart chính thấp hơn 680 px thì hàng dưới bị cắt (hay gặp khi đang bật cửa sổ MACD của v103).
   - Nút `[-]` thu gọn còn một dòng (số lệnh mở, lãi treo, kết quả hôm nay), `[+]` mở lại. Nút `i` ở mỗi ô mở bảng giải thích của bot đó; trong bảng giải thích có hàng nút để chuyển sang bot khác.
   - Dòng "cách thoát" đổi theo lệnh đang giữ: mức tầng 1, "đã đóng nửa, chờ tầng 2", mốc chốt nửa của v106…
4. **Xem từng bot trên chart:** mặc định bật cho mọi bot đang chạy. Bấm nút `Chart` ở ô của bot để tắt/bật (khi bảng thu gọn thì là sáu nút `v98` `v99` `v102` `v103` `v105` `v106`); lựa chọn được nhớ qua lần khởi động lại.

   | Bot | Vẽ gì |
   |---|---|
   | v98 | EMA 5/9/13/21/200 (M30) · các vùng FVG đang theo dõi (xanh = tăng, đỏ = giảm) · mũi tên ở nến từ chối đang chờ xác nhận |
   | v99 | EMA 50/200 (M30) · hộp Donchian 20 nến mà bot đang so · mức Chandelier 3.0 (và 6.0 nếu bật thoát 2 tầng) khi đang có lệnh |
   | v102 | EMA 100/200 của giá đóng Heikin Ashi (M15) |
   | v103 | MACD M15 ở cửa sổ phụ |
   | v105 | Khung dải Bollinger của nến M30 vừa đóng (vàng khi đã đủ 4/6 nến nén, xám khi chưa) · tầng 1 và tầng 2 khi đang có lệnh |
   | v106 | Khung vùng giá trị VAL – VAH và đường POC, kẻ suốt ngày hiện tại · mốc TP1 khi đang có lệnh, sau TP1 là mức trailing |

   Các đường tính trên khung của bot nên chart để khung nào cũng đúng (trên chart M15, EMA M30 có dạng bậc thang). Bảng mở rộng che gần hết chart, nên thường thu gọn bảng rồi mới bật xem.
5. EA ghi ba file vào `Common\Files` của MT5:

| File | Nội dung |
|---|---|
| `v104_portfolio_deals_<mã>.csv` | Mỗi lệnh đã đóng một dòng: bot, giờ, giá, SL, TP, lãi USD, `r_multiple`, lý do, `net_r`, `spread_pip`, `slip_pip`, `mfe_r`, `mae_r`, `risk_usd`, `swap_usd`, `balance` |
| `v104_portfolio_events_<mã>.csv` | `INIT` (kèm cấu hình), `OPEN`, `SPREAD_BLOCK`, `ORDER_FAIL`, `EXIT_FAIL`, `T1_CLOSE`, `T2_LOCK`, `TP1_CLOSE`, `TRAIL`, `RESTORE`, `DEINIT` |
| `v104_portfolio_equity_<mã>.csv` | Mỗi 15 phút: số dư, equity, lãi treo từng bot, số lệnh mở |

   `mfe_r` / `mae_r` là lãi / lỗ lớn nhất từng chạm trong lúc giữ lệnh. File cũ có cột khác bản EA đang chạy sẽ tự được đổi tên `..._cu_<ngày>.csv`.

6. **Báo cáo sau demo:**

```powershell
python -X utf8 backtest/v104_demo_report.py --symbol XAUUSD
# So từng lệnh với backtest cùng giai đoạn, cùng input (ngày kết thúc phải trước hôm nay):
backtest\run_v104.ps1 -Feed demo -From 2026.11.20 -To 2026.12.20 -Tag demo_period -A -B -Risk 0.5
python -X utf8 backtest/v104_demo_report.py --symbol XAUUSD --tester backtest/v104_out/demo_period.csv
```

   Báo cáo gồm: kết quả từng bot, chất lượng khớp lệnh (spread, trượt giá, MFE/MAE, sự kiện), equity DD, so từng lệnh demo với backtest, và kết quả demo đứng ở đâu so với mọi quãng dài bằng nó trong backtest.

Vài tuần demo đủ để trả lời "chạy thật có khớp backtest không". Chưa đủ để xếp hạng bot: sáu bot cộng lại khoảng 36 lệnh mỗi tháng, riêng v105 chỉ khoảng 1–2 lệnh.

---

## 7. Đã thử và không dùng

Mô phỏng nến M30 trên cả hai feed (v99) và dựng lại từ log lệnh (v102/v103):

- **v99:** dời hoà vốn, TP cứng 3R/5R/8R, siết Chandelier sau khi lãi, SL khác 1.5 ATR, nghỉ sau SL, chỉ BUY, lọc SELL/BUY theo EMA khung lớn, lọc giờ, time-stop, dời SL nửa sau về hoà vốn.
- **v102/v103:** đóng cuối ngày, giới hạn thời gian giữ, BE; trailing cho v103; sàn SL 0,25% cho v102 (C).
- **Danh mục:** cầu dao lỗ ngày, giảm risk khi các bot trùng chiều, chia lệch tỷ trọng giữa các bot, thử lại lệnh vào bị `Market closed` ở nến 01:00.
- **v98:** xem `docs/V98_RESEARCH_LOC_LENH.md`.

---

## 8. Chạy lại backtest

```powershell
# Sáu bot, mặc định của EA (A + B bật, v106 chỉ Buy), vốn cố định 100k (script mặc định -Risk 1.0, 2022.06.01 → 2026.11.17, Model 4)
backtest\run_v104.ps1 -Tag v200_all6 -FixedCap 100000 -A -B
# Bốn bot cũ như v1.2x: tắt hai bot mới
backtest\run_v104.ps1 -Tag v200_reg4 -FixedCap 100000 -A -B -Enable105 false -Enable106 false
# Một bot riêng: tắt các bot kia. v106 chạy cả hai chiều: thêm -Sell106
backtest\run_v104.ps1 -Tag only105 -FixedCap 100000 -Enable98 false -Enable99 false -Enable102 false -Enable103 false -Enable106 false
# Chạy một bản .ex5 khác (bản thử nghiệm): thêm -Ex5 <đường dẫn>
# Chụp thử dashboard trong tester trực quan (chạy bằng Windows PowerShell 5.1)
powershell -File backtest\visual_v104.ps1 -WaitSec 28 -Out dash
```

`run_v104.ps1` luôn truyền rõ `-A`, `-B`: không ghi thì hai công tắc đó tắt, khác mặc định của EA.

Các script dùng bản MT5 riêng `C:\MT5_Test` và **tắt terminal đang chạy trong thư mục đó** khi bắt đầu — kiểm tra không ai đang test trước khi chạy.

---

## 9. Chốt chặn xu hướng ngày (`D1_DownGuard`, v2.01)

**Câu hỏi:** có chỉ số nào báo trước "giai đoạn xấu" để dừng bot không, thay vì chỉ giảm cỡ lệnh?

**Cách tìm.** Chạy cả sáu bot trên feed demo 2018.07 → 2026.10 (3.609 lệnh, v106 chỉ Buy). Với mỗi lệnh, tính khoảng 60 chỉ số chỉ từ dữ liệu có trước lúc vào lệnh, rồi xem chỉ số nào tách được lệnh tốt khỏi lệnh xấu ở **cả hai nửa** lịch sử. Thước đo là R theo giá, vì tester áp swap hôm nay cho cả năm 2018 làm R ròng các năm đầu lệch nặng. Script: `backtest/v105_v106_research/regime.py`, `regime_sim.py`, `regime_d1.py`, `regime_alt.py`, `regime_bot.py`.

Không chốt chặn, sáu bot trên lịch sử dài: +928R theo giá nhưng **sụt 100R trong sáu tháng** (17/08/2018 → 15/02/2019); nửa cuối 2018 lỗ 56R. Chạy với vốn 100k và rủi ro cố định 1%/bot thì tài khoản cháy ngay trong đợt đó.

### 9.1. Chỉ số tìm được

**Chỉ số = (EMA50 − EMA200 của giá đóng ngày) / ATR(14) ngày**, tính trên các nến ngày đã đóng.

| Chỉ số lúc vào lệnh | Tỷ lệ lệnh | Lệnh Buy | Lệnh Sell |
|---|---:|---:|---:|
| dưới −2 (ngày giảm rõ) | 7% | 152 lệnh, **−54,0R** | 102 lệnh, +8,9R |
| −2 đến −0,5 | 9% | 233 lệnh, −2,7R | 87 lệnh, −13,2R |
| −0,5 đến +0,5 (chuyển tiếp) | 13% | 287 lệnh, +5,3R | 168 lệnh, +56,1R |
| +0,5 đến +2 | 15% | 335 lệnh, +188,3R | 191 lệnh, +62,3R |
| trên +2 (ngày tăng rõ) | 57% | 1.425 lệnh, +644,2R | 629 lệnh, +33,2R |

Lệnh Buy mở lúc chỉ số dưới −0,5: 385 lệnh, **−57R**. Toàn bộ lãi của chiều Buy đến từ lúc chỉ số trên +0,5.

Từng đợt chỉ số dưới −0,5:

| Đợt | Dài | Lệnh Buy mở trong đợt | Lệnh Sell mở trong đợt |
|---|---:|---:|---:|
| 2018.07 → 2019.01 | 183 ngày | 135 lệnh, **−55,2R** | 63 lệnh, −3,8R |
| 2021.03 → 2021.05 | 60 ngày | 67 lệnh, +28,5R | 19 lệnh, +10,8R |
| 2021.09 → 2021.11 | 38 ngày | 38 lệnh, −14,8R | 14 lệnh, −10,7R |
| 2022.07 → 2022.12 | 163 ngày | 116 lệnh, **−32,5R** | 84 lệnh, +7,8R |
| 2026.07 → 2026.08 | 29 ngày | 29 lệnh, +17,4R | 9 lệnh, −8,5R |

### 9.2. Luật và kết quả MT5

**Luật:** chỉ số dưới −0,5 thì không bot nào mở lệnh Buy. Hằng `D1G_DOWN_INDEX = -0.50`.

| | Tắt chốt chặn | Bật chốt chặn |
|---|---:|---:|
| **Feed demo 2018.07 → 2026.10** (R theo giá) | | |
| Số lệnh | 3.610 | 3.234 |
| Tổng R | +927,5 | **+975,6** |
| Sụt giảm lớn nhất | 100,4R (08/2018 → 02/2019) | **51,4R** (04/2023 → 07/2023) |
| Nửa cuối 2018 | −56R | −4R |
| Ba tháng liền tệ nhất | −56,0R | −31,2R |
| Số tháng âm | 40/100 | 38/100 |
| Lãi ròng · PF (tester áp swap hôm nay cho cả kỳ) | $731.770 · 1,32 | $820.951 · 1,40 |
| Equity DD lớn nhất (cùng lưu ý về swap) | $146.022 | $74.170 |
| **Feed live 2022.06 → 2026.11** (R ròng) | | |
| Số lệnh | 1.923 | 1.770 |
| Tổng R | +620,9 | **+638,5** |
| Sụt giảm lớn nhất | 58,4R | **46,4R** |
| Lãi ròng · PF | $606.286 · 1,52 | $623.278 · 1,59 |
| Equity DD lớn nhất | $62.439 | $53.579 |

Mô phỏng trên danh sách lệnh dự báo +985R và DD 51,4R cho feed demo; MT5 ra +975,6R và 51,4R (khác nhau vì bot rảnh tay vào thêm 9 lệnh). Hai lượt demo trong bảng chạy đúng cấu hình mặc định, v106 chỉ Buy (`backtest/v104_out/v201_demo_buy106_guard_off.csv`, `v201_demo_buy106_guard_on.csv`), rủi ro cố định $1.000 mỗi lệnh nên $1.000 = 1R.

Theo bot (feed demo, R theo giá, tắt → bật): v98 +225 → +254 · v99 +240 → +257 · v102 +133 → +142 · v103 +52 → +60 · v105 +188 → +194 · v106 +89 → +69. Chỉ v106 kém đi.

### 9.3. Giới hạn — đọc trước khi tin

- **Bằng chứng chỉ là 5 đợt.** Hai đợt giảm dài (2018, 2022) lỗ nặng và quyết định toàn bộ kết quả; ba đợt ngắn 1–2 tháng thì lẫn lộn (+28R, −15R, +17R). Đợt giảm ngắn rồi bật lại hình chữ V là lúc chốt chặn làm mất lãi: năm 2026 mất khoảng 24R trên feed live.
- **Nó không giảm số tháng thua** (40 → 38 trên 100). Nó cắt cái đuôi xấu nhất: mua liên tục trong một xu hướng ngày giảm kéo dài.
- **Kiểu giai đoạn xấu thứ hai thì không bắt được:** vàng giằng co trong lúc xu hướng ngày vẫn tăng (04 → 07/2023: −46R đến −51R; 08/2020 → 08/2021). Sụt giảm còn lại sau khi bật chốt chặn nằm ở đó.
- **Chỉ đúng với cấu trúc ngày chậm.** Đổi sang "giá đóng dưới EMA200 ngày" hoặc "EMA200 ngày dốc xuống" vẫn có lợi (DD 60–63R). Đổi sang thước nhanh hơn thì hết tác dụng: EMA50/EMA200 trên H4, động lượng 40 ngày, "giá dưới EMA50 và EMA50 dưới EMA200" đều cắt nhầm lệnh lãi.
- **Ngưỡng không nhạy:** chặn Buy ở −2 / −1 / −0,5 / 0 / +0,5 đều cho R cao hơn gốc 2–6% và DD thấp hơn 36–49%.
- **Không chặn Sell đối xứng.** Lệnh Sell lúc ngày tăng vẫn dương (+96R / 820 lệnh); chặn chúng mất lãi mà sụt giảm lịch sử dài không giảm.
- **Đầu mỗi feed EMA200 ngày chưa hội tụ** (feed demo bắt đầu 05/2018, feed live 05/2022). Đã kiểm lại đợt 2018 bằng lịch sử ngày thật từ 2015: tập lệnh bị chặn không đổi (mục 9.5). Chạy thật thì terminal có đủ lịch sử ngày.
- **Ngoài năm 2018 chốt chặn không tăng lãi**, chỉ hạ sụt giảm lớn nhất (mục 9.5). Coi nó là bảo hiểm cho thị trường gấu kéo dài.

### 9.4. Đã thử và không dùng

| Hướng | Kết quả trên 8,3 năm |
|---|---|
| Nhận biết giằng co: Efficiency Ratio 10/20/40 ngày, ADX ngày và H4, Choppiness Index, số lần giá cắt EMA20, số lần đảo chiều, tỷ số ATR | Không chỉ số nào tách lệnh nhất quán ở hai nửa. Lọc theo chúng làm mất 20–60% lãi, R/DD không hơn chốt chặn ngày. |
| Luật giằng co riêng cho v99 (bot lỗ nhiều nhất lúc giằng co) | Ghép thêm vào chốt chặn ngày: R giảm, DD gần như không đổi (45–60R). |
| Dừng khi danh mục đang thua: tổng 20/40/80 lệnh gần nhất, 10/20/40/60 ngày gần nhất, đang sụt quá 10/20/30R | Tốt nhất là "60 ngày gần nhất lỗ quá 10R thì dừng": DD 100 → 58R, R −8%. Kém chốt chặn ngày, ghép cả hai thì tệ hơn (R +768, DD 55R), và phải giao dịch ảo trong lúc dừng mới biết khi nào chạy lại. |
| Dừng bot sau 3–5 lệnh thua liền, hoặc khi 5 lệnh gần nhất của bot âm | Làm kết quả kém đi. Lệnh ngay sau chuỗi 3 lệnh thua của một bot vẫn có kỳ vọng dương bình thường. |

### 9.5. Phản biện của Codex và Antigravity (21/11/2026)

Chủ dự án yêu cầu hỏi thêm hai agent khác xem có hướng nào khác. Trao đổi hai vòng qua CLI; bản gốc nằm ở `backtest/v105_v106_research/peer/` (`brief.md`, `codex_reply_1.md`, `antigravity_reply_1.md`, `round2.md`, `codex_reply_2.md`, `antigravity_reply_2.md`). Mọi con số hai bên đưa ra đều được chạy lại bằng `peer_check.py`, `peer_check_c.py`, `peer_check_d.py`, `peer_check_e.py`, `peer_check_x.py` trong `backtest/v105_v106_research/`.

**Điều phải sửa trong cách nói về chốt chặn ngày**

- **Lợi ích dồn về năm đầu.** Tính từ 07/2019: tắt +920R / DD 66,5R; bật +915R / DD 51,4R. Ngoài năm 2018, chốt chặn không tăng lãi; nó hạ sụt giảm lớn nhất 23%, và phần hạ đó đến từ đúng một đợt (2022). Sau 2018 nó có ích một lần (2022: tránh −32,5R) và làm mất lãi hai lần (2021: +13,7R, 2026: +17,4R).
- **Nửa sau lịch sử gần như không có bằng chứng.** Lệnh Buy bị chặn: nửa đầu −51,9R / 274 lệnh, nửa sau −4,8R / 111 lệnh. Điểm chia đôi (23/08/2022) cắt ngang đợt 2022 nên câu "đúng ở cả hai nửa" ở mục 9 yếu hơn nó nghe.
- **Tên gọi đúng là bảo hiểm cho một thị trường gấu kéo dài**, không phải bộ lọc tăng lãi. Nó chưa được kiểm trên thị trường gấu nào ngoài 2018 và 2022, và bất đối xứng "chỉ chặn Buy" có thể đổi nếu vàng vào một thị trường gấu nhiều năm kiểu 2012–2015.

**Điều đã kiểm và giữ nguyên**

- *EMA200 ngày chưa hội tụ ở đầu feed* (cả hai bên nêu): nối 851 nến ngày thật 2015 → 05/2018 (vàng kỳ hạn, quy đổi 1,0003) vào trước feed rồi tính lại. Ngày vào trạng thái giảm lệch 7 ngày (21/06 so với 28/06/2018, đều trước ngày bắt đầu test), ngày ra trùng nhau (02/01/2019), tập lệnh bị chặn giống hệt (135 lệnh, −55,2R).
- *Bỏ lệnh Sell của v106 sau khi chạy không tương đương cấu hình chỉ Buy* (Codex): chạy lại MT5 đúng cấu hình mặc định, lệch 1 lệnh và 1R. Bảng ở mục 9.2 đã dùng hai lượt này.
- *Phép thử giả dược:* dịch vòng chuỗi "ngày giảm" 643 lần rồi áp cho lệnh Buy. Phần bị chặn thật −56,7R; giả dược trung vị +158R (5%..95%: +22..+294R); không lần nào tệ bằng. Riêng từ 07/2019: thật −1,5R, giả dược trung vị +147R, 3,4% số lần tệ bằng. Lệnh Buy lúc ngày giảm kém hẳn lệnh Buy thường, nhưng sau 2018 chúng hoà vốn chứ không lỗ.
- *Thử chéo* (lệnh v98 trên BTC / EURUSD / USDJPY): Buy ngược xu hướng ngày kém hơn ở 2/3 tài sản, Sell ngược xu hướng ngày không kém ở 2/3. Phép thử yếu vì v98 lỗ nền trên EURUSD và BTC.

**Sáu hướng mới hai bên đề xuất — không cái nào dùng được**

Nền so sánh: 3.233 lệnh đã bật chốt chặn, +976,6R, DD 51,4R, R/DD 19,0.

| Hướng | Ai đề xuất | Kết quả |
|---|---|---|
| Tỷ lệ phá vỡ thất bại trên H4 (≥ 10 sự kiện trong 30 ngày, ≥ 2/3 hỏng thì chặn chiều đó) | Codex | Chặn 137 lệnh −34,6R, DD không đổi. 123/137 lệnh nằm ở nửa đầu; không lệnh nào trong đợt 04–07/2023. Đổi ngưỡng sang 0,60 thì mất 63R, cửa sổ 60 ngày mất 76R. Codex rút lại. |
| Khoảng trống từ điểm vào tới biên hộp 60 ngày dưới 2R thì bỏ lệnh | Codex | Ngược hẳn: đó là nhóm lệnh tốt nhất (393 lệnh, +180R, dương ở cả hai nửa). |
| USD mạnh đồng loạt (≥ 5/7 cặp) thì chặn Buy | Codex | Lệnh Buy lúc đó vẫn +0,40 / +0,23 R mỗi lệnh; chặn mất 123R. |
| Tự tương quan bậc 1 của lợi suất ngày, 40 ngày (AC1_40 < −0,10 thì giảm nửa cỡ) | Antigravity | R 896 (−8%), DD 39,4R, R/DD 22,8; 7/10 đợt sụt nông đi; giả dược 0/643. Nhưng: chỉ số là nhiễu thuần (độ lệch chuẩn 0,154 so với 0,158 của nhiễu trắng, không tự lặp lại); feed live gần như trung tính (R −13%, R/DD 15,2 → 15,7); hiệu ứng rã hết khi dịch trễ tín hiệu 20 ngày; bản dừng hẳn kém nền; không lặp lại trên tài sản khác. Antigravity rút lại, Codex chỉ giữ làm ứng viên theo dõi. |
| Khoảng lùi từ đỉnh 60 ngày trên 4,5 ATR thì giảm nửa cỡ | Antigravity | R 942, DD 47,5R. Giả dược: 9% số lần dịch vòng đạt R/DD bằng, 24% đạt DD bằng. Trong 07/2020 → 08/2021 các lệnh bị giảm cỡ lại lãi +35R. |
| Vàng tính theo rổ ngoại tệ, chặn Buy khi chỉ số rổ < +0,5 | Antigravity | 177 lệnh −11,9R (nửa đầu −18,9R, nửa sau +7,1R), DD không đổi. |

Hai gợi ý về cấu trúc danh mục của Antigravity cũng đã kiểm:
- *Chia lại rủi ro theo 1/DD của từng bot:* trọng số tính từ nửa đầu áp cho nửa sau cho R/DD 12,4 → 17,0, nhưng chiều ngược lại cho 7,0 → 6,8. Trọng số dồn vào v105 (×2,2) và v106 (×1,4), hai bot ít lệnh nhất. Bỏ v103: R/DD 19,0 → 19,0. Bỏ v98: 19,0 → 9,4. Giữ rủi ro đều nhau.
- *Miễn chốt chặn cho v106:* mô phỏng +20,3R, DD không đổi, nhưng 2/3 phần lãi đến từ riêng năm 2021 (42 lệnh trong 4 đợt). Antigravity rút lại.

**Sụt 30–50R là mức bình thường của danh mục này.** Sau khi bật chốt chặn vẫn có mười đợt sụt trên 30R trong 8,3 năm (theo lệnh đóng, R theo giá):

| Đỉnh → đáy | Sâu | Lập đỉnh mới |
|---|---:|---|
| 17/08/2018 → 15/02/2019 | 48R | 06/2019 |
| 13/08/2019 → 23/12/2019 | 48R | 01/2020 |
| 16/04/2020 → 17/06/2020 | 35R | 07/2020 |
| 12/08/2020 → 28/10/2020 | 40R | 12/2020 |
| 11/01/2021 → 25/02/2021 | 34R | 03/2021 |
| 17/09/2021 → 22/11/2021 | 32R | 11/2021 |
| 24/11/2021 → 02/02/2022 | 43R | 02/2022 |
| 16/05/2022 → 30/06/2022 | 43R | 03/2023 |
| 18/04/2023 → 13/07/2023 | 51R | 10/2023 |
| 28/12/2023 → 01/03/2024 | 37R | 05/2024 |

Equity DD của tester (tính cả lệnh đang treo và swap) lớn hơn số theo lệnh đóng: feed live 53,6R, feed demo 74R (số demo bị đội lên vì tester áp swap hôm nay cho lot của năm 2018).

**Ba bên cùng kết luận**

1. Giữ `D1_DownGuard` bật, áp đều cho cả sáu bot, coi là bảo hiểm cho thị trường gấu kéo dài.
2. Chưa có chỉ số nào báo trước được kiểu giằng co khi xu hướng ngày còn nguyên. Đã thử khoảng 60 chỉ số ở mục 9.4 cộng sáu hướng ở trên; tiếp tục dò bộ lọc trên cùng 8 năm này chỉ làm tăng khả năng khớp quá khứ.
3. Cỡ lệnh phải chọn sao cho tài khoản chịu được một đợt sụt hơn 50R.
4. Muốn sụt giảm thấp hơn thì hướng đáng nghiên cứu là thêm một bot khác loại (hồi quy về trung bình), so ở cùng tổng rủi ro. Đưa v105 sang tài sản khác: mô phỏng nến cho thấy 7 cặp FX đều âm hoặc hoà (PF 0,71–1,09), riêng BTC dương (454 lệnh, +241R, PF 1,77) nhưng chưa kiểm bằng MT5 với spread và swap thật.

---

## 10. Tin tức và cuối tuần (21/11/2026)

**Câu hỏi của chủ dự án:** vàng chịu ảnh hưởng mạnh của tin, vậy có nên dừng bot quanh giờ ra tin không; và có nên đóng lệnh trước cuối tuần không.

**Trả lời ngắn:** không, cho cả hai. Tin và cuối tuần là lúc danh mục kiếm tiền; đóng lệnh trước chúng làm mất 12–34% lãi và không giảm sụt giảm.

**Dữ liệu.** Lượt MT5 8,3 năm đúng cấu hình mặc định (`backtest/v104_out/v201_demo_buy106_guard_on.csv`, 3.234 lệnh, +975,6R, DD 51,4R) kèm log sự kiện của cùng lượt để biết lúc nào lệnh đã chốt một nửa. Lịch kinh tế USD của MT5 từ 2018 đến 11/2026, xuất bằng `ExportCalendar.mq5`. Script: `backtest/v105_v106_research/news_weekend.py` (chính), `news_weekend2.py`, `news_weekend3.py`, `news_weekend4.py`.

- *Tin lớn:* NFP, CPI, quyết định lãi suất và họp báo Fed — 357 mốc.
- *Tin đỏ:* mọi tin USD mức cao của MT5, bỏ nhóm dầu / đấu giá trái phiếu / PMI của S&P / nhà ở — 1.749 mốc.
- Lịch MT5 đổi mọi mốc lịch sử sang lệch giờ của server **lúc xuất file**, nên tin mùa đông lệch 1 giờ nếu xuất vào mùa hè và ngược lại. Script đã sửa theo ngày xuất của từng file. Kiểm: nến M30 đúng giờ ra tin lớn có biên độ gấp 3,55 lần trung vị, nến ngay trước đó 1,35 lần.
- "Đóng trước tin / trước cuối tuần" được tính bằng cách định giá lại lệnh theo giá M30 tại thời điểm đó, có tính phần đã chốt một nửa. Phép dựng lại này lệch R thật của tester 1,9R trên cả kỳ. Nó không tính các lệnh mới mà bot có thể vào thêm khi rảnh tay.

### 10.1. Tin tức

**Lệnh mở gần giờ ra tin không kém hơn lệnh thường.**

| Thử | Số lệnh bị bỏ | Tổng R | DD |
|---|---:|---:|---:|
| Nền | — | +975,6 | 51,4 |
| Không mở lệnh 30 phút trước → 30 phút sau tin lớn | 27 | +956,4 | 51,4 |
| Không mở lệnh 60 phút trước → 60 phút sau tin lớn | 57 | +961,3 | 51,4 |
| Không mở lệnh 30 phút trước → 30 phút sau tin đỏ | 206 | +932,3 | 56,7 |
| Không mở lệnh 60 phút trước → 60 phút sau tin đỏ | 336 | +924,8 | 52,2 |
| Không mở lệnh 4 giờ trước → 2 giờ sau tin đỏ | 821 | +867,9 | 43,3 |

Dòng cuối hạ DD nhưng bằng cách bỏ một phần tư số lệnh và 11% lãi; R/DD gần như không đổi (19,0 → 20,1). Riêng v99 có 58 lệnh mở trong 30 phút quanh tin đỏ với trung bình −0,25R (lúc khác +0,28R); mẫu quá nhỏ để thành luật.

**Giữ lệnh qua tin là nguồn lãi.**

| Thử | Lệnh đang mở lúc ra tin | Giữ qua tin mang lại | Nếu đóng trước tin: tổng R / DD |
|---|---:|---:|---:|
| Đóng hết trước tin lớn đầu tiên gặp phải | 209 (6%) | +119,1R (nửa đầu +26,6, nửa sau +92,5) | +856,6 / 58,2 |
| Đóng hết trước tin đỏ đầu tiên gặp phải | 910 (28%) | +335,8R (nửa đầu +89,9, nửa sau +245,9) | +639,9 / 56,4 |

Năm trong sáu bot được lợi khi giữ qua tin; v103 hoà (−3,4R / −1,9R). Đóng tay trước một tin lớn bất kỳ tốn trung bình 0,33R.

### 10.2. Cuối tuần

- **Toàn bộ lãi của danh mục nằm ở các lệnh sống qua ít nhất một cuối tuần:** 385 lệnh (12%) lãi +1.089R, trung bình +2,83R mỗi lệnh. 88% lệnh còn lại cộng lại là −114R.
- **Khoảng trống giá sáng thứ Hai thường nhỏ và nghiêng về phía có lợi.** 443 lần thị trường đóng trên 40 giờ: khoảng trống trung vị 0,05 ATR ngày, 90% dưới 0,24 ATR, lớn nhất 1,34 ATR; trên 1 ATR chỉ 2 lần. Cộng mọi lần lệnh đi qua cuối tuần: +56,3R (nửa đầu +43,8, nửa sau +12,5).
- **Tính trên cả danh mục mỗi cuối tuần** (214 cuối tuần có lệnh, nhiều nhất 5 lệnh cùng lúc): trung vị +0,02R, 5%..95% từ −1,3R đến +2,6R. Tệ nhất −3,3R (11/04/2026), tốt nhất +10,3R (22/02/2020). Mất quá 2R: 6 lần; được quá 2R: 13 lần.

| Thử | Tổng R | DD | R/DD |
|---|---:|---:|---:|
| Nền | +975,6 | 51,4 | 19,0 |
| Đóng hết lúc đóng cửa thứ Sáu | +739,1 | 50,3 | 14,7 |
| Đóng hết 1 giờ trước giờ đóng cửa thứ Sáu | +734,3 | 50,6 | 14,5 |
| Không mở lệnh thứ Sáu từ 16h server | +869,8 | 47,4 | 18,4 |
| Không mở lệnh thứ Sáu từ 20h server | +968,4 | 50,4 | 19,2 |

Đóng hết chiều thứ Sáu mất 237R (24% lãi), đều ở hai nửa (+117 / +119R), và DD không giảm. Sau khi trừ phần lỗ nhảy giá mà tester bỏ sót (mục 10.3) con số là 231R. Đóng tay một cuối tuần bất kỳ đang có lệnh tốn trung bình khoảng 1,1R.

Lệnh mở vào thứ Sáu là nhóm tốt nhất tuần (+0,52R mỗi lệnh, +333R). Thứ Tư yếu ở nửa đầu (−61R) nhưng bình thường ở nửa sau (+76R), nên không phải quy luật.

### 10.3. Trượt giá qua SL — điều lượt 8 năm không thấy

Tester ở mô hình "1 minute OHLC" (feed demo) khớp SL **đúng giá đặt**, kể cả khi thị trường mở cửa nhảy qua SL: trong 3.234 lệnh không có lệnh nào lỗ quá 1,000R. Lượt feed live chạy "real ticks" thì có, và tick thật chỉ có từ 07/01/2026:

| Giai đoạn tick thật 07/01 → 16/11/2026 (298 lệnh, 190 lệnh thua) | Số lệnh lỗ quá 1R | Phần lỗ vượt 1R |
|---|---:|---:|
| Tổng | 71 | −5,0R (khoảng −6R/năm) |
| Mở cửa đầu tuần | 3 | −1,7R, nặng nhất −1,38R |
| Mở cửa lại sau giờ nghỉ hằng ngày | 2 | −0,4R |
| Trong 30 phút sau tin đỏ | 10 | −0,4R |
| Lúc khác (trượt giá thường) | 56 | −2,6R |

Lệnh nặng nhất: v106 Buy giữ qua cuối tuần 11/04/2026, mất 2,38R thay vì 1R. Tính lại trên feed demo bằng giá mở cửa thật: 8 lệnh trong 8,3 năm bị tester bỏ sót tổng −5,3R (−0,6R/năm).

Hệ quả: mọi số R của lượt 8 năm đẹp hơn thực tế khoảng 6R/năm vì trượt giá; phần do tin và cuối tuần trong đó chỉ khoảng 2–3R/năm.

### 10.4. Kết luận

1. Không thêm bộ lọc tin, không đóng lệnh trước tin, không đóng lệnh trước cuối tuần.
2. Rủi ro thật của cuối tuần là cái đuôi chưa có trong mẫu: một khoảng trống lớn gấp vài lần mức 1,34 ATR đã thấy, lúc 5 lệnh cùng chiều. Phí bảo hiểm chống lại nó (đóng mọi thứ Sáu) là khoảng 28R/năm, đắt hơn nhiều so với thiệt hại đã quan sát (tệ nhất −3,3R một cuối tuần).
3. Nếu có một cuối tuần hoặc một tin cụ thể đáng lo (bầu cử, chiến sự leo thang), đóng tay riêng lần đó là hợp lý: tốn trung bình khoảng 1,1R cho một cuối tuần, 0,33R cho một tin lớn.
