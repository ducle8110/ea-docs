# Structured Logger Design (v9.23)

> Design rationale, schema, và cách sử dụng hệ thống logging tích hợp vào `v91_PartialTP.mq5` từ phiên bản 9.22.

---

## 1. Vì Sao Cần Structured Logger

Trước v9.23, EA chỉ có `Print()` ghi vào Experts journal — gặp 3 vấn đề:

1. **Không cấu trúc**: text tự do, khó parse, khó query
2. **Bị MT5 rotate/xóa**: journal có giới hạn dung lượng, log cũ bị mất
3. **Không có context**: thiếu equity, margin level, DD% tại thời điểm event

**Hệ quả thực tế**: khi muốn điều tra "tại sao ngày 05-08 lỗ $713?", không có data để truy nguyên.

→ **Giải pháp**: ghi CSV structured vào `FILE_COMMON` để Python phân tích.

---

## 2. Vị Trí File

```
%APPDATA%\MetaQuotes\Terminal\Common\Files\v91_log_YYYYMMDD.csv
```

Mỗi ngày một file. EA tự rotate khi qua ngày mới (`Log_RotateIfNewDay()`).

**Lý do dùng `FILE_COMMON`** (thay vì `FILE_TXT` thường):
- Common folder dùng chung giữa các terminal MT5
- Python pull được mà không cần biết terminal ID
- Không phụ thuộc instance — vẫn truy cập được khi đổi terminal

---

## 3. CSV Schema (18 cột)

| Cột | Kiểu | Mô tả |
|-----|------|-------|
| `time_broker` | datetime | Giờ broker (MT5 server time) |
| `time_gmt` | datetime | Giờ GMT — dùng correlate với tin tức |
| `event` | string | Loại event (xem section 4) |
| `ticket` | uint64 | Ticket lệnh (0 nếu không liên quan lệnh) |
| `symbol` | string | Symbol (vd XAUUSD.sc) |
| `type` | string | BUY/SELL hoặc rỗng |
| `lot` | double | Khối lượng |
| `price_open` | double | Giá mở (cho OPEN) hoặc giá entry gốc (CLOSE) |
| `price_close` | double | Giá đóng (cho CLOSE) |
| `profit` | double | P/L của lệnh (cho CLOSE) |
| `equity` | double | Account equity tại thời điểm event |
| `balance` | double | Account balance |
| `margin_level` | double | Margin level % |
| `floating_pl` | double | Tổng floating P/L hiện tại |
| `dd_pct` | double | Drawdown % = (balance - equity) / balance × 100 |
| `reason` | string | Nguyên nhân (vd `partial_tp_close_far`) |
| `comment` | string | Comment lệnh (vd `VIRTUAL_BUY`, `DD_HEDGE_T1`) |
| `context` | string | Context bổ sung (vd `buys=42 sells=8`) |

---

## 4. Event Types

| Event | Khi nào | Use case phân tích |
|-------|---------|-------------------|
| `INIT` | EA attach lên chart | Verify settings tại từng phiên |
| `DEINIT` | EA bị gỡ | Tìm crash/restart |
| `OPEN` | Sau khi `trade.Buy/Sell` thành công | Phân tích nhịp grid |
| `CLOSE` | TRƯỚC khi `trade.PositionClose` | Phân tích kết quả từng lệnh |
| `OPEN_FAIL` | trade.Buy/Sell fail | Diagnose tại sao không mở được lệnh |
| `CLOSE_FAIL` | trade.PositionClose fail | Diagnose lỗi đóng lệnh |
| `HEDGE_FIRE` | Trước khi fire DD Hedge tier | Track timing hedge |
| `HEDGE_CLOSE_ALL` | Sau khi force close DD Hedge | Manual intervention tracking |
| `FULL_TP_START` | Trước khi đóng all theo ClusterTP | Track full TP events |
| `PARTIAL_TP_CLOSEFAR_START` | Trước Partial TP CloseFar | Track partial TP |
| `PARTIAL_TP_COMBO_START` | Trước Partial TP Combo 2+1 | Track combo TP |
| `SNAPSHOT` | Mỗi N giây (mặc định 900s) | Equity curve |

---

## 5. Reason Codes (cột `reason`)

### Cho OPEN events
- `virtual_buy_trigger` — Grid BUY trigger từ pending ảo
- `virtual_sell_trigger` — Grid SELL trigger từ pending ảo
- `ddhedge_t1`, `ddhedge_t2`, `ddhedge_t3` — DD Hedge tier
- `weekend_hedge` — Lệnh hedge cuối tuần

### Cho CLOSE events
- `full_tp` — Đóng all theo ClusterTP_USD
- `partial_tp_close_far_profit` — Đóng lệnh lời trong combo CloseFar
- `partial_tp_close_far_worst_partial` — Đóng partial lot lệnh lỗ xa nhất
- `partial_tp_combo21` — Đóng 1 trong 3 lệnh của combo 2+1
- `ddhedge_manual_close` — Manual close qua button BTN_CloseHedge
- `weekend_hedge_close` — Đóng hedge sáng thứ 2

---

## 6. Nguyên Tắc Thiết Kế

### 6.1 Silent Failure
Logger **không bao giờ** throw exception hoặc block strategy logic. Nếu `Log_Init()` fail:
- `g_logInitialized = false`
- Mọi `Log_*()` call trở thành no-op
- Strategy vẫn chạy bình thường

### 6.2 FileFlush sau mỗi event
- Chi phí: ~1ms I/O
- Lợi ích: nếu MT5 crash, không mất event quan trọng
- Acceptable: tần suất event grid là 10-50/giờ → 50ms/giờ tổng

### 6.3 Capture Trước Khi Close
```mql5
Log_TradeClose(reason, ticket);  // Lấy info TRƯỚC
trade.PositionClose(ticket);     // Rồi mới đóng
```

Lý do: sau khi `PositionClose`, position info biến mất → không log được giá entry, lot, comment.

### 6.4 Daily Rotation
Mỗi ngày một file → dễ phân tích, dễ backup, không có file quá lớn (~5-50KB/ngày).

---

## 7. Cách Phân Tích Log

### Phân tích tự động (Python)
```bash
# Log hôm nay
python scripts/mt5_analysis/v91_log_analyzer.py

# Ngày cụ thể
python scripts/mt5_analysis/v91_log_analyzer.py 20260520

# Khoảng ngày
python scripts/mt5_analysis/v91_log_analyzer.py 20260501-20260517

# Chỉ HEDGE events
python scripts/mt5_analysis/v91_log_analyzer.py --hedge-only
```

### Phân tích thủ công
Mở file `.csv` bằng Excel hoặc pandas:
```python
import pandas as pd
df = pd.read_csv("v91_log_20260520.csv")

# Mọi event HEDGE
print(df[df["event"].str.contains("HEDGE")])

# Equity curve theo giờ
snaps = df[df["event"] == "SNAPSHOT"]
snaps[["time_broker", "equity", "dd_pct"]].plot()
```

---

## 8. Tham Số Input

```mq5
input string SettingLogger          = "=====<Structured Logger>=====";
input bool   EnableStructuredLog    = true;     // Default ON cho v9.23+
input int    LogSnapshotIntervalSec = 900;      // 15 phút
```

### Khuyến nghị
- **Live trading**: bật, snapshot 900s (15 phút)
- **Backtest**: tắt — chậm I/O không cần thiết
- **Demo/test**: bật, snapshot 60-300s để debug nhanh

---

## 9. Hooks Đã Thêm Vào EA (12 vị trí)

| Function | Event log |
|----------|-----------|
| `OnInit` | INIT |
| `OnDeinit` | DEINIT |
| `OnTick` | SNAPSHOT (theo timer) |
| `CheckAndTriggerVirtualPending` | OPEN ×2 (BUY+SELL) |
| `FireDDHedgeTier` | HEDGE_FIRE + OPEN ×N |
| `PlaceHedgeNOrders` | OPEN ×N (weekend) |
| `CheckAndExecuteFullTP` | FULL_TP_START + CLOSE ×N |
| `CheckPartialTP_CloseFar` | PARTIAL_TP_CLOSEFAR_START + CLOSE ×N |
| `CheckPartialTP_Combo` | PARTIAL_TP_COMBO_START + CLOSE ×3 |
| `CloseAllDDHedgePositions` | HEDGE_CLOSE_ALL + CLOSE ×N |
| `CloseHedgePosition` | CLOSE ×N (weekend close) |
| Mọi `_FAILED` path | OPEN_FAIL / CLOSE_FAIL |

---

## 10. Kế Hoạch Mở Rộng

### Nên thêm trong tương lai
- **Latency log**: thời gian từ tín hiệu → fill
- **Slippage log**: chênh lệch giá yêu cầu vs giá fill
- **Spread snapshot**: spread tại mỗi event quan trọng
- **Tick stats**: số tick/giây trong volatility cao

### Pattern cho v92, v93
Logger này nên **kế thừa** sang v92_FixedHedge và v93_GridFX với schema giống nhau → phân tích chung được. Đề xuất:
- Đổi `LOG_FILE_PREFIX` thành `v92_log_`, `v93_log_`
- Giữ nguyên schema 18 cột
- Adapter trong `v91_log_analyzer.py` để hỗ trợ nhiều prefix
