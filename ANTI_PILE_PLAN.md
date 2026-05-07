# Anti-Pile (Asymmetric Step) — v9.16

> **Trạng thái**: Implemented & tested ✓
> **File**: `tool/v91_PartialTP_AntiPile.mq5` (v9.16, MagicNumber 777)
> **Base**: `tool/v91_PartialTP.mq5` (v9.15, không đụng — giữ để rollback)

---

## 1. Bối cảnh & vấn đề

### 1.1. Cơ chế EA hiện tại

EA `v91_PartialTP.mq5` là **grid theo trend**:
- BUY STOP đặt **trên** giá → giá lên chạm → mở BUY market
- SELL STOP đặt **dưới** giá → giá xuống chạm → mở SELL market
- Khoảng cách: `StepPip` (mặc định 150 pip)
- Sau mỗi lần fill: cancel pending còn lại, đặt lại 1 cặp mới
- Cắt lỗ bằng `PartialTP_CloseFar`: gom **N** lệnh lời (cả BUY+SELL) để đóng kèm 1 lệnh lỗ xa giá nhất
  - **CASE 1**: nếu sumProfit ≥ |loss| + PartialTP_USD → đóng trọn lệnh lỗ
  - **CASE 2**: nếu sumProfit < |loss| nhưng sumProfit > PartialTP_USD → đóng phần lot lệnh lỗ ("gặm lot")

### 1.2. Tình huống nguy hiểm

```
Giai đoạn 1: Giá đi xuống mạnh dài
  → mỗi 150 pip mở thêm 1 SELL theo trend
  → sau 13500 pip xuống, tích lũy 90 SELL ở vùng đáy
  → tổng SELL đang LỜI lớn

Giai đoạn 2: Giá đảo chiều lên mạnh
  → 90 SELL từ lời thành lỗ
  → BUY mới mở mỗi 150 pip → BUY có lời ngay
  → CloseFar gom BUY lời + cắt 1 SELL lỗ xa
  → mỗi BUY = 1 SELL được TP/gặm
  → tốc độ tạo BUY (1 / 150 pip) << tốc độ DD tăng (90 lệnh × giá đi xa)
  → DD leo nhanh → có nguy cơ cháy nếu trend lên đủ dài
```

### 1.3. Nguyên tắc thiết kế

User chỉ ra:
- ❌ **Không hard cap số lệnh majority**: nếu giá lên cao và có nhịp giảm, vẫn cần SELL mới (entry tốt) để có lệnh lời cắt SELL âm
- ❌ **Không co StepPip của minority quá thấp**: tránh tạo cluster lệnh dồn ở 1 vùng giá
- ❌ **Không tăng step đều cả 2 chiều**: càng khó tạo BUY lời để cắt SELL âm

→ Giải pháp: chỉ giãn step **bên majority**, tinh chỉnh nhẹ minority, có cap để khống chế.

---

## 2. Cơ chế: Asymmetric Step (bậc thang ±50 pip)

### 2.1. Logic

Step điều chỉnh theo **bậc thang cộng/trừ cố định**:
- Mỗi bậc: majority **+ AsymStep_Increment** (mặc định +50 pip), minority **− AsymStep_Decrement** (mặc định −50 pip)
- Mỗi bậc cần thêm `AsymStep_BandWidth` (mặc định **20**) imbalance
- Cap: `AsymStep_MaxCapMaj` (mặc định 400 pip), `AsymStep_MinCapMin` (mặc định 100 pip — có thể set xuống 50 nếu cần aggressive hơn)

```
buyCount  = CountByDir(1)
sellCount = CountByDir(-1)
imbalance = |buyCount - sellCount|

if not EnableAsymmetricStep or imbalance <= AsymStep_Threshold:
    stepBuy  = g_StepPip
    stepSell = g_StepPip
else:
    excess = imbalance - AsymStep_Threshold
    band   = excess / AsymStep_BandWidth + 1     # band 1, 2, 3, ...
    stepMaj = g_StepPip + band * AsymStep_Increment
    stepMin = g_StepPip - band * AsymStep_Decrement

    # Cap
    if stepMaj > AsymStep_MaxCapMaj: stepMaj = AsymStep_MaxCapMaj
    if stepMin < AsymStep_MinCapMin: stepMin = AsymStep_MinCapMin

    if buyCount > sellCount:
        stepBuy  = stepMaj
        stepSell = stepMin
    else:
        stepSell = stepMaj
        stepBuy  = stepMin
```

### 2.2. Bảng tham chiếu

**Default**: `g_StepPip=150`, `Threshold=20`, `BandWidth=20`, `Increment=50`, `Decrement=50`, `MaxCapMaj=400`, `MinCapMin=100`

| Imbalance | Band | stepMaj (pip) | stepMin (MinCap=100) | stepMin (MinCap=50) |
|---|---:|---:|---:|---:|
| ≤ 20    | 0 | **150** | 150 | 150 |
| 21 – 40 | 1 | **200** | 100 | 100 |
| 41 – 60 | 2 | **250** | 100 (cap) | 50 (cap) |
| 61 – 80 | 3 | **300** | 100 (cap) | 50 (cap) |
| 81 –100 | 4 | **350** | 100 (cap) | 50 (cap) |
| 101 –120| 5 | **400** (cap) | 100 (cap) | 50 (cap) |
| ≥ 121   | 6+| 400 (cap) | 100 (cap) | 50 (cap) |

**Hiệu quả minh họa** (imbalance=70, có 70 SELL nhiều hơn BUY):
- stepSell = **300 pip** → giá phải đi xuống 300 pip mới mở thêm SELL (chậm tích thêm SELL lỗ)
- stepBuy = **100 pip** (default MinCap) → giá đi lên 100 pip đã mở thêm BUY → tạo BUY lời để cắt SELL âm

### 2.3. Hàm chính

```mql5
double ComputeDynamicStepPip(int dir, int buyCount, int sellCount)
{
   if (!g_EnableAsymmetricStep) return g_StepPip;

   int imbalance = MathAbs(buyCount - sellCount);
   if (imbalance <= g_AsymStep_Threshold) return g_StepPip;

   int excess = imbalance - g_AsymStep_Threshold;
   int band   = excess / g_AsymStep_BandWidth + 1;     // 1, 2, 3, ...

   double stepMaj = g_StepPip + band * g_AsymStep_Increment;
   double stepMin = g_StepPip - band * g_AsymStep_Decrement;

   if (stepMaj > g_AsymStep_MaxCapMaj) stepMaj = g_AsymStep_MaxCapMaj;
   if (stepMin < g_AsymStep_MinCapMin) stepMin = g_AsymStep_MinCapMin;

   int majoritySide = (buyCount > sellCount) ? 1 : -1;
   return (dir == majoritySide) ? stepMaj : stepMin;
}
```

### 2.4. Chỗ gọi

- `PlacePendingPair()` — tính `stepBuyPip`, `stepSellPip` riêng cho `buyStopPrice` và `sellStopPrice`.
- `IsTooCloseToSameDir(dir, fillPrice)` — dùng `ComputeDynamicStepPip(dir, buyCount, sellCount)` thay cho `g_StepPip`.
- `NeedRebuildVirtualPending()` — threshold rebuild dùng `MathMax(stepBuy, stepSell) * REBUILD_MULTIPLIER`.

---

## 3. Inputs & shadow vars

```mql5
input string  SettingAntiPile      = "=====<Anti-Pile>=====";
input bool    EnableAsymmetricStep = true;   // Bật giãn step bên majority + co minority
input int     AsymStep_Threshold   = 20;     // Imbalance bắt đầu kích hoạt
input int     AsymStep_BandWidth   = 20;     // Mỗi bậc cần thêm bao nhiêu imbalance
input double  AsymStep_Increment   = 50.0;   // Mỗi bậc majority TĂNG bao nhiêu pip
input double  AsymStep_Decrement   = 50.0;   // Mỗi bậc minority GIẢM bao nhiêu pip
input double  AsymStep_MaxCapMaj   = 400.0;  // Cap tối đa cho step majority (pip)
input double  AsymStep_MinCapMin   = 100.0;  // Cap tối thiểu cho step minority (pip) — có thể set 50
```

Tất cả 7 input có shadow var `g_*` tương ứng + được sync qua Remote (heartbeat JSON + apply_config).

---

## 4. Dashboard

Thêm 1 dòng mới sau "TP Mode:":

```
===== ANTI-PILE =====
StepB: 150 pip   StepS: 442 pip   ← step động hiện tại
```

Khi step động ≠ `g_StepPip` (đã kích hoạt) → label đổi màu vàng để dễ thấy.

---

## 5. Verification

### 5.1. Compile

- Mở MetaEditor → mở `tool/v91_PartialTP_AntiPile.mq5` → F7
- Yêu cầu: 0 error, 0 warning

### 5.2. Strategy Tester (XAUUSD M1, hedging)

Chọn data range có biến động lớn (vd: tháng XAUUSD biến động > 500 pip + đảo chiều mạnh). Run 2 lần:

1. **Baseline**: `EnableAsymmetricStep = false` → kết quả tương đương EA gốc (về mặt step logic)
2. **Improved**: `EnableAsymmetricStep = true` với default config

So sánh metrics:
| Metric | Kỳ vọng |
|---|---|
| Max DD% | giảm rõ rệt |
| Max imbalance | có thể vượt 20 nhưng tốc độ chậm hơn rõ rệt |
| Profit Factor | có thể giảm nhẹ (đánh đổi lấy an toàn) |
| Recovery Factor | tăng |
| Số lệnh tổng | giảm (do step majority giãn) |

### 5.3. Edge cases

- [x] `imbalance = 0` → bypass, không lỗi chia 0
- [x] `EnableAsymmetricStep = false` → `ComputeDynamicStepPip` luôn trả `g_StepPip`
- [x] `imbalance` cực lớn (vd 200) → stepMaj cap tại MaxCapMaj
- [x] `IsTooCloseToSameDir` với step động lớn → không reject nhầm lệnh hợp lệ
- [x] `NeedRebuildVirtualPending` không loop vô hạn khi 1 phía giãn rất xa

---

## 6. Lịch sử cơ chế đã thử & loại bỏ

| Cơ chế | Lý do bỏ |
|---|---|
| **Smart Filter** (skip lệnh nếu entry mới làm tăng DD theo avg) | Test thực tế không hiệu quả |
| **Dynamic N** (giảm N của PartialTP_CloseFar khi imbalance cao) | Test thực tế không hiệu quả |

Cả 2 đã được implement và test, sau đó remove. Chỉ giữ lại Asymmetric Step.

---

## 7. Tóm tắt thay đổi vs file gốc `v91_PartialTP.mq5`

| Loại | Chi tiết |
|---|---|
| Thêm function | `ComputeDynamicStepPip(dir, buyCount, sellCount)` |
| Sửa function | `PlacePendingPair()`, `IsTooCloseToSameDir()`, `NeedRebuildVirtualPending()` |
| Thêm inputs | 7 input Anti-Pile + 7 shadow vars + sync vào Remote |
| Dashboard | 1 dòng StepB/StepS động |
| Version | 9.15 → 9.16 |
| MagicNumber | 777 (giữ nguyên — sẽ replace EA gốc trên MT5) |

> ⚠️ **Lưu ý**: EA mới dùng cùng MagicNumber 777 với EA gốc. Trên MT5 chỉ chạy 1 trong 2, không chạy song song.
