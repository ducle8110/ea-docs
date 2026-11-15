# v92 1.35 — phiên VN, sideway 30 giá, step cố định và khóa lỗ

## Cấu hình đã chốt

- XAUUSD, tài khoản USD **hedging**, vốn backtest 100.000 USD.
- FixedLot = 1; MaxBuy = MaxSell = 5. Lot không tăng theo vốn.
- Vào lệnh trong `[23:00, 17:00)` giờ Việt Nam; phiên qua đêm.
- AutoTP: box 24 nến M5 đã đóng, rộng tối đa 30 giá; vượt biên thêm 2 giá thì CloseFar.
- Chỉ nhận box mới khi giá mid hiện tại nằm trong box. Không dùng nến tương lai.
- TP tổng = 500 USD; TP từng lệnh trong sideway = 300 USD.
- CloseFar: tối đa 5 lệnh dương bù **trọn** 1 lệnh âm xa nhất, còn lời ít nhất 500 USD.
  `PartialTP_ClosePct=100` tắt việc gặm một phần lot. TP tổng vẫn có ưu tiên trước CloseFar.
- Chỉ dùng `StepPip=150` cố định ở cả hai chiều (1,5 giá trên symbol có Pip=0,01).
  Dynamic Step đã được xóa: không còn step co, input bật/tắt, cap/outward hay lệnh `_DYN` mới.
- Trend Filter giữ cấu hình hiện tại.

Các báo cáo `verified_feb26`, `verified_jun26`, `audit_jun26_fresh` và phân tích lỗ
đã tạo trước đây thuộc bản 1.33 có Dynamic Step; không dùng làm kết quả bản 1.34.
Mẫu `fixedstep_v134_feb26` là 5 ngày giao dịch với box 20 giá của bản 1.34.
Bản 1.35 dùng box 30 giá; các bài tháng đầy đủ dùng ngày đầu tháng tới ngày đầu tháng sau (exclusive),
vốn khởi đầu 100k riêng cho từng tháng.
Đợt `sw30_v135_202602` tới `sw30_v135_202606` chạy 5 bài riêng tháng 2–6/2026,
không nối vốn, vị thế hay trạng thái. Bảng tổng hợp: `backtest/results/v92_sw30_v135_monthly_202602_06/index.html`.

Nạp binary `v92_PropFirm.ex5` mới và Load preset `docs/v92_session_100k.set` trong Inputs.
Chart/tester cũ có thể giữ input đã lưu, không tự nhận defaults mới khi recompile.
Vốn 100k đặt trong Strategy Tester; EA không thay đổi số vốn tài khoản.

## Khóa lỗ

Kiểm tra **trước** pending ảo và tất cả TP:

- SELL: `Ask - giá mở SELL thấp nhất >= 50`.
- BUY: `giá mở BUY cao nhất - Bid >= 50`.

Đây là chênh lệch giá vàng (4000 → 4050), không phải pip. Giá thanh lý tính cả spread.
Khi chạm ngưỡng, latch trạng thái dừng ngay, xóa pending ảo, mở phía thiếu lot cho tới
khi tổng lot BUY = tổng lot SELL. Ví dụ 3 SELL + 1 BUY → thêm 2 BUY → 3–3.
Mỗi lệnh bổ sung tối đa FixedLot, vẫn giữ cap từng chiều; phần thiếu lẻ chỉ dùng khi
có khớp/cắt một phần. Hedge bỏ qua trend, spacing, toggle và giới hạn spread vì đây
là xử lý khóa, không phải vào grid mới. Chỉ xác nhận HEDGED sau kiểm tra lot thực tế.

`SessionStopAction`:

| Giá trị | Khi khóa | Lúc hết giờ | Chạy lại |
|---|---|---|---|
| `STOP_CLOSE_ALL` (mặc định) | Hedge rồi đóng sạch | Đóng sạch | Phiên 23:00 tiếp theo |
| `STOP_HOLD_MANUAL` (live) | Giữ hedge; ngừng mọi TP/entry | Giữ rổ, ngừng bot | Xử lý hết lệnh rồi bấm RESET |

Trong **Strategy Tester luôn dùng CLOSE_ALL**, kể cả input HOLD_MANUAL. Đây là quy ước
backtest thay cho việc người dùng quản lý tay. Nếu hedge bị broker từ chối/không còn
slot/không đủ min lot: CLOSE_ALL ghi bằng chứng và thanh lý, không ghi HEDGED giả;
HOLD_MANUAL giữ latch và thử bổ sung lại mỗi giây. Khi đóng chưa hết, retry mỗi giây,
không chạy TP/grid xen giữa. MT5 nghỉ/không có tick thì xử lý ở tick giao dịch kế tiếp.
Nếu mất tick qua giờ kết thúc, rổ phiên cũ phải được xử lý trước khi vào lệnh mới.

Trạng thái live lưu theo account + magic + symbol và sống qua restart/đổi chart.
HOLD_MANUAL không tự nhả khóa khi qua ngày. RESET bị chặn khi rổ dừng vẫn còn lệnh.
Tester không tải/lưu Global Variables của live hay bài test trước.

## Giờ broker

Không dùng `TimeGMT()` để đo offset trong tester. Input mặc định:
`BrokerWinterUTC=2`, `BrokerSummerUTC=3`, `BrokerDST=BROKER_DST_US`.
US = Chủ nhật thứ hai tháng 3 tới Chủ nhật đầu tháng 11. Có lựa chọn EU = Chủ nhật
cuối tháng 3 tới Chủ nhật cuối tháng 10, hoặc NONE = offset cố định mùa đông.
**Chọn lịch đúng với server/dữ liệu đang test**; điều này đặc biệt ảnh hưởng các tuần
tháng 3 và tháng 10–11. Các bài mẫu tháng 2/tháng 6 không phân biệt US với EU.
Đã đối chiếu [hướng dẫn MT5 của Vantage](https://www.vantagemarketsea.com/en/academy/metatrader-5-tutorial/):
server GMT+2/GMT+3 theo DST Mỹ. Bài tháng 3 dùng US DST, đổi mùa từ 08/03/2026.

## Backtest và bằng chứng

```powershell
rtk proxy python backtest/run_v92_session.py --from 2026.02.02 --to 2026.02.07 --tag winter_example
rtk proxy python backtest/run_v92_session.py --from 2026.06.01 --to 2026.06.08 --tag summer_example
rtk proxy python backtest/run_v92_session.py --from 2026.02.02 --to 2026.02.03 --tag scenario_example --scenario

# Chạy riêng từng tháng 2–6/2026, mỗi lệnh tạo tester mới với vốn 100k:
foreach ($testMonth in 2..6) {
  $testFrom = "2026.{0:D2}.01" -f $testMonth
  $testTo = "2026.{0:D2}.01" -f ($testMonth + 1)
  $testTag = "sw30_v135_2026{0:D2}" -f $testMonth
  rtk proxy python backtest/run_v92_session.py --from $testFrom --to $testTo --tag $testTag
  if ($LASTEXITCODE -ne 0) { throw "Backtest failed: $testTag" }
}
rtk proxy python backtest/v92_monthly_report.py
```

ToDate của tester là exclusive. Dùng tag mới mỗi lần để giữ bằng chứng cũ. Runner:

1. Compile và yêu cầu `Result: 0 errors, 0 warnings`.
2. Tạo terminal **portable riêng** ở `backtest/results/v92_terminal`; copy config tài khoản
   đã lưu và cache XAUUSD cục bộ, không copy Profiles/EA live, không dừng terminal đang mở.
3. Dùng tài khoản VantageMarkets-Demo đã lưu (có thể truyền `--server` và `--login`),
   deposit 100k, real ticks Model=4, input lấy từ source hiện tại.
4. Lưu report MT5 gốc, input, tester banner, CSV lệnh/sự kiện/nến và chart vào
   `backtest/results/v92_<tag>/index.html`.

CSV trong `Common/Files/v92_<tag>_XAUUSD_777_*.csv`:

- `events`: BEFORE, HEDGED, AFTER, hoặc HEDGE_FAILED. Có bid/ask, balance/equity,
  số lệnh/lot, floating, box và mode. SESSION_END không cần HEDGED.
- `positions`: snapshot từng ticket/position ID, giá mở, lot, profit, swap, SL/TP.
- `bars`: M5 đã đóng, tiếp tục ghi sau stop và ngoài phiên.
- `deals`: history EA nhìn thấy; MT5 có thể không cho EA thấy thanh lý cuối test/stop-out.
- `deals_complete.csv` trong thư mục kết quả: bổ sung **chỉ** deal `end of test` hoặc `so ...`
  từ report MT5 và log agent để biết position ID, có cột `source` chỉ nguồn; giữ nguyên CSV gốc.
  Không tự tạo deal dựa trên P&L hay giá ước lượng.

Mỗi chart thể hiện 3 giờ trước / tối đa 6 giờ sau sự kiện, đường entry của rổ trước stop,
điểm hedge và giá stop thực. Bảng hiển thị từng lệnh ở các phase cùng giá khớp/phí thực.
Đường floating nếu tiếp tục giữ rổ chỉ là **ước tính**: giữ entry/lot/spread lúc stop,
chưa cộng swap/phí mới; không phải kết quả giao dịch sau stop. Cuối tuần hoặc cuối
bài test có thể không đủ nến tương lai.

Verifier replay từng deal để kiểm tra cap, lot cố định, giờ entry, thời gian nghỉ,
ngưỡng khóa, cân bằng hedge và rổ AFTER trống; tổng net phải khớp report MT5 tới cent,
kể cả thanh lý cuối test. Scenario EA chỉ chạy trong tester, kiểm tra biên giờ/DST,
3 SELL + 1 BUY → 3–3 và đối xứng 3 BUY + 1 SELL → 3–3.
Nếu broker stop-out khiến tester dừng sớm, bảng tháng ghi rõ thời điểm stop-out;
không diễn giải ngày cuối được yêu cầu là ngày bot thực sự đã chạy tới.

Không dùng các harness v92 cũ để xác nhận defaults mới: chúng có thể đè input cũ
(ví dụ FixedLot=0.2, MaxPerSide đã đổi tên, TP/cắt lot cũ).
