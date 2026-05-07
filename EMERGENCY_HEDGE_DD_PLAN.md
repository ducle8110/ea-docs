# DD Hedge — Hedge khẩn cấp theo nhiều mức DD, hedge tham gia TP như lệnh thường

## 1. Bối cảnh & Mục tiêu

**Vấn đề Emergency Hedge hiện tại** (file `v91_PartialTP.mq5`, hàm `CheckEmergencySafety` dòng 1998–2023, `PlaceHedgeSingleNet` dòng 1840–1895):
- Chỉ có **1 ngưỡng duy nhất** (`EmergencyHedgePercent = 50%`).
- Khi trigger → đặt **1 lệnh hedge net** rồi **freeze EA vĩnh viễn** (`g_emergencyHedgeActive = true`).
- Hedge có comment `EMERGENCY_HEDGE` → `IsHedgePosition()` trả `true` → CloseFar/Combo TP **bỏ qua hedge** → không TP được cho đến khi user can thiệp tay.

**Mục tiêu user yêu cầu (DD Hedge mới)**:
1. Trigger theo **nhiều mức DD** (multi-tier), thay vì 1 ngưỡng duy nhất.
2. **Cho phép nhiều lệnh hedge cùng tồn tại** — KHÔNG freeze EA sau khi hedge.
3. **Lệnh hedge coi như lệnh cluster bình thường** → CloseFar / Combo / Cluster TP gom được như lệnh thường.
4. Trigger gate kép: `DD%` **VÀ** `số lệnh chênh lệch BUY/SELL` cả hai phải đạt ngưỡng.
5. Inputs cho từng "mức hedge".

## 1.5 Quyết định đã chốt với user

| # | Quyết định | Giá trị |
|---|------------|---------|
| 1 | Volume hedge | **N lệnh nhỏ × FixedLot** (KHÔNG phải 1 lệnh tổng) — đồng nhất với cluster |
| 2 | LotMultiplier mỗi tier | **1.0× đều mọi tier** (imbalance 51 → hedge 51 lệnh ở mọi tier kích hoạt) |
| 3 | Emergency Hedge cũ | **Bỏ hẳn** — set `EnableEmergencyHedge=false`, không có safety net freeze |
| 4 | Slot tier | Query broker comment (`HasDDHedgeAtTier`) — không dùng flag RAM, không hysteresis |
| 5 | Hedge tham gia TP | KHÔNG add `DD_HEDGE_T*` vào `IsHedgePosition()` → CloseFar/Combo gom bình thường |
| 6 | File triển khai | Sửa trực tiếp `tool/v91_PartialTP.mq5` |
| 7 | Số tier + ngưỡng DD% | **3 tier: 25% / 35% / 45%** |
| 8 | MinImbalanceCount default | **20** (hedge trễ — chỉ fire khi imbalance đáng kể) |
| 9 | Cooldown | 60s chung cho mọi tier (default) |
| 10 | Alert | SendLog hiện có (không thêm Push/Email/Sound ở giai đoạn này) |

## 2. Khác biệt so với Rescue Hedge plan

| Khía cạnh | Rescue Hedge (plan trước) | DD Hedge (plan này) |
|-----------|---------------------------|---------------------|
| Trigger | Pip distance + cluster size | DD% + imbalance count |
| Tách biệt với TP | Có (comment `RESCUE_HEDGE`, TP bỏ qua) | KHÔNG (hedge tham gia TP thường) |
| Số hedge concurrent | 1 | Nhiều (1 hedge/tier) |
| File triển khai | File mới `v92_RescueHedge.mq5` | **Sửa trực tiếp** `v91_PartialTP.mq5` |
| Mức quan trọng | Cứu cluster cụ thể bị bỏ rơi | Cân tài khoản theo cấp độ rủi ro |

Hai phương án **độc lập**, có thể chạy song song trong tương lai.

## 3. Kiến trúc DD Hedge

### 3.1 Multi-tier triggers

3 tier (đã chốt):

| Tier | DD% | Hành động |
|------|-----|-----------|
| 1 | 25% | Hedge `imbalanceCount × 1.0` lệnh, mỗi lệnh = `FixedLot` |
| 2 | 35% | Hedge `imbalanceCount × 1.0` lệnh, mỗi lệnh = `FixedLot` |
| 3 | 45% | Hedge `imbalanceCount × 1.0` lệnh, mỗi lệnh = `FixedLot` |

### 3.2 Trigger condition (gate kép)

```
trigger tier i khi ĐỒNG THỜI:
  A. EnableDDHedge = true
  B. ddPercent >= DDHedge_Tier[i].Threshold
  C. !HasDDHedgeAtTier(i)                          // chưa có lệnh hedge tier i đang mở trên broker
  D. (DDHedge_MinImbalanceCount == 0)              // = 0 → BỎ QUA check imbalance (chỉ dùng DD)
     HOẶC imbalanceCount >= DDHedge_MinImbalanceCount
  E. imbalanceLot >= MinLot của broker             // có lệnh để đối ứng
```

Trong đó:
- `ddPercent` = `MathAbs(totalProfitExcludeHedge) / balance * 100` (đã có sẵn dòng 2007)
- `imbalanceCount` = `MathAbs(CountByDir(BUY) - CountByDir(SELL))`
- `imbalanceLot` = `MathAbs(GetTotalLotByDir(BUY) - GetTotalLotByDir(SELL))`
- `HasDDHedgeAtTier(i)` = duyệt position trên broker, có lệnh nào với comment `DD_HEDGE_T<i+1>` đang mở không

> **Quy ước `MinImbalanceCount = 0`**: nếu user set giá trị này = 0 trong inputs, EA sẽ **bỏ qua** kiểm tra imbalance count → chỉ cần DD% đạt ngưỡng + imbalanceLot ≥ MinLot là fire. Hữu ích khi user muốn hedge bất kể số lệnh chênh lệch.

### 3.3 Slot tier qua comment broker — không cần hysteresis

**Cách chặn fire lặp**: mỗi tier dùng 1 comment riêng (`DD_HEDGE_T1`, `DD_HEDGE_T2`, `DD_HEDGE_T3`). Trước khi fire tier i, EA quét toàn bộ position broker xem có comment `DD_HEDGE_T<i>` đang mở không:
- **Có** → tier i đang "in use" → SKIP fire.
- **Không** → tier i "free" → fire.

Hedge tier i **chỉ được giải phóng tự nhiên** khi lệnh hedge đó bị TP đóng (qua CloseFar/Combo/Cluster vì hedge tham gia TP thường) hoặc đóng tay. Không cần flag RAM, không cần reset thủ công.

**Ví dụ timeline** (tier 1 = 30%):

| Thời điểm | DD% | Lệnh hedge tier 1 trên broker | Hành động |
|-----------|-----|-------------------------------|-----------|
| t=0 | 28% | (không có) | — |
| t=1 | 30% | (không có) | **FIRE** → 1 lệnh `DD_HEDGE_T1` mở |
| t=2 | 31% | 1 lệnh open | SKIP (tier 1 đang in use) |
| t=3 | 28% | 1 lệnh open | SKIP |
| t=4 | 32% | 1 lệnh open | SKIP — đúng, tránh dồn hedge |
| t=5 | 25% | TP gom đóng hedge | (không có) |
| t=6 | 31% | (không có) | **FIRE lần 2** → tier 1 lại có 1 lệnh `DD_HEDGE_T1` |

**Lợi ích so với hysteresis tính toán**:
- Broker là **single source of truth** — không cần state RAM phải sync.
- **Restart EA an toàn** — query broker là đủ, không cần `RebuildState`.
- **Logic tự nhiên khớp với "hedge tham gia TP"** — TP đóng hedge = tier tự free.
- **Không cần input `HysteresisPct`** — bỏ 1 tham số, ít rối hơn.

**Lưu ý**: cần giữ `DDHedge_CooldownSec` (vd 60s) để chặn case hedge tier vừa bị TP đóng tức thì → tick liền sau lại fire lại do DD vẫn cao. Cooldown chung cho mọi tier là đủ.

### 3.4 Volume hedge mỗi tier — đặt N lệnh nhỏ

**Quan trọng**: hedge KHÔNG phải 1 lệnh tổng lot, mà là **nhiều lệnh nhỏ** mỗi lệnh = `FixedLot` (giống các lệnh cluster thường). Lý do:
- Đồng nhất kích thước lot với các lệnh thường → CloseFar/Combo TP gom dễ (mọi lệnh đều cùng `FixedLot`).
- Tránh case 1 lệnh hedge to vượt `VOLUME_MAX` của broker.
- Khi TP đóng từng lệnh hedge một, không phải đóng cả khối lot lớn cùng lúc.

```
imbalanceCount = |CountByDir(BUY) - CountByDir(SELL)|     // số lệnh chênh lệch
hedgeOrderCount = MathRound(imbalanceCount * DDHedge_Tier[i].LotMultiplier)
                                                            // số lệnh hedge cần đặt
hedgeOrderCount = MathMax(1, hedgeOrderCount)               // đảm bảo ≥ 1
hedgeDir = ngược majorityDir                                // BUY nhiều hơn → SELL hedge

for j = 1 to hedgeOrderCount:
    trade.Buy/Sell(FixedLot, _Symbol, price, 0, 0, "DD_HEDGE_T<i+1>")
```

**Ví dụ**: imbalance count = 51 (15 BUY vs 66 SELL như tình huống ảnh), `LotMultiplier = 1.0`:
- hedgeOrderCount = 51 → đặt **51 lệnh BUY**, mỗi lệnh `FixedLot = 0.01` → tổng 0.51 lot.
- Tất cả 51 lệnh đều có comment `DD_HEDGE_T1`.
- `HasDDHedgeAtTier(0)` chỉ cần thấy 1 trong 51 lệnh là trả `true` → tier 1 "in use".
- Khi TP đóng dần các hedge này (cùng cluster), comment vẫn còn trên các lệnh chưa đóng → tier 1 vẫn in use cho đến lệnh cuối cùng đóng.

**Khi tất cả lệnh hedge tier 1 đã đóng** → `HasDDHedgeAtTier(0) = false` → tier 1 free → có thể fire lại sau cooldown.

**LotMultiplier ý nghĩa thực tế**:
- `1.0` (mặc định): hedge bằng đúng số lệnh chênh lệch (cân tuyệt đối).
- `0.5`: hedge nửa imbalance (vd 51 imbalance → 26 lệnh hedge) — giảm rủi ro nếu user lo hedge quá đà.
- `1.5`+: hedge dư (vd 51 imbalance → 77 lệnh hedge) — đảo bias sang chiều ngược.

> Nếu broker giới hạn số lệnh open tối đa, cần handle case `hedgeOrderCount` quá lớn — split qua nhiều tick hoặc clamp.

### 3.5 Hedge tham gia TP như lệnh thường — chi tiết kỹ thuật

**Comment**: `DD_HEDGE_T<i>` (vd `DD_HEDGE_T1`, `DD_HEDGE_T2`).

**Quan trọng**: `IsHedgePosition()` (dòng ~263) hiện tại check `WEEKEND_HEDGE` và `EMERGENCY_HEDGE`. **KHÔNG thêm** `DD_HEDGE_T*` vào danh sách check này → CloseFar/Combo/Cluster TP **vẫn gom hedge bình thường**.

Hệ quả tự nhiên:
- `CountPositions()`, `CountByDir()`, `GetTotalProfitExcludeHedge()` → đếm cả hedge mới như lệnh thường.
- `ClusterTP_USD` (Full TP) → tổng P/L bao gồm hedge.
- `CheckPartialTP_CloseFar` / `CheckPartialTP_Combo` → có thể chọn hedge để gom.

**Tác dụng phụ cần lưu ý**:
- Sau khi đặt hedge, `CountByDir` sẽ thay đổi → Anti-Pile (`ComputeDynamicStepPip`) sẽ tính lại step. Có thể là tốt (cân bằng tự nhiên) — không cần can thiệp thêm.
- `g_hedgeActive` cũ (dùng cho weekend/emergency) **KHÔNG được set** bởi DD Hedge. Tách flag riêng `g_ddHedgeFiredCount`.

### 3.6 Quan hệ với Emergency Hedge cũ

Đề xuất giữ Emergency Hedge cũ làm **safety net cuối cùng** ở mức cao hơn tất cả tier DD Hedge (vd Emergency = 60%, DD Hedge tier cao nhất = 50%). Lý do:
- DD Hedge có thể fail (không đủ imbalance) → vẫn cần freeze cuối cùng.
- Tránh phá vỡ logic cũ đã chạy ổn.

Nếu user muốn **bỏ hẳn Emergency Hedge cũ** → set `EnableEmergencyHedge = false` (input đã có sẵn dòng ~61).

## 4. Inputs mới (thêm sau group `Hedging Settings` hiện có, trước `Anti-Pile`)

```mql5
input string Setting_DDHedge       = "=====<DD Hedge - Multi-Tier>=====";
input bool   EnableDDHedge         = false;     // OFF default — bật explicit

// Tier thresholds (theo thứ tự tăng dần) — đã chốt 25/35/45
input double DDHedge_Tier1_DDPct   = 25.0;      // Mức DD% kích hoạt tier 1
input double DDHedge_Tier1_LotMult = 1.0;       // Hệ số lot hedge tier 1 (1.0 = đối ứng đầy số lệnh imbalance)
input double DDHedge_Tier2_DDPct   = 35.0;
input double DDHedge_Tier2_LotMult = 1.0;
input double DDHedge_Tier3_DDPct   = 45.0;
input double DDHedge_Tier3_LotMult = 1.0;

// Gate chung
input int    DDHedge_MinImbalanceCount = 20;    // Số lệnh chênh lệch BUY/SELL tối thiểu mới fire hedge (default 20 — "hedge trễ").
                                                 //   = 0 → BỎ QUA check imbalance, chỉ dùng DD% (luôn fire khi đạt ngưỡng DD)
                                                 //   > 0 → cần |countBuy - countSell| >= giá trị này
input int    DDHedge_CooldownSec       = 60;    // Cooldown giây giữa 2 lần fire bất kỳ tier nào.
                                                 //   Chặn fire dồn dập khi DD nhảy gap qua nhiều tier cùng lúc,
                                                 //   hoặc khi tier vừa bị TP đóng và DD vẫn cao.
```

> Ghi chú: 3 tier là default. Nếu user muốn 4 tier, thêm `DDHedge_Tier4_*`. Số tier giới hạn 5 cho gọn.

## 5. State variables mới

```mql5
#define DD_HEDGE_TIER_COUNT  3
#define DD_HEDGE_COMMENT_PREFIX "DD_HEDGE_T"

// State (chỉ cần cooldown timer + cumulative counter — KHÔNG cần fired flag)
datetime g_ddHedgeLastFireTime = 0;                  // Cho cooldown
int      g_ddHedgeFiredCount   = 0;                  // Tổng số lần đã hedge (cumulative, hiển thị dashboard)

// Shadow vars cho input (giống pattern hiện có dòng 176)
double g_DDHedge_TierThreshold[DD_HEDGE_TIER_COUNT];
double g_DDHedge_TierLotMult[DD_HEDGE_TIER_COUNT];
int    g_DDHedge_MinImbalanceCount;
int    g_DDHedge_CooldownSec;
bool   g_EnableDDHedge;
```

## 6. Function signatures mới

```mql5
// === Detection ===
bool   IsDDHedgePosition(ulong ticket);              // check comment có prefix DD_HEDGE_T
bool   HasDDHedgeAtTier(int tierIndex);              // check broker có lệnh comment "DD_HEDGE_T<tier+1>" mở chưa
int    CountDDHedgePositions();                       // hiển thị dashboard
double GetDDHedgeProfit();                            // hiển thị dashboard

// === Core ===
void   InitDDHedgeShadow();                           // gọi trong InitShadowVariables (dòng 631)
void   CheckDDHedgeTriggers();                        // gọi mỗi tick trong CheckEmergencySafety
bool   FireDDHedgeTier(int tierIndex);                // đặt 1 lệnh hedge tier i
```

> Không cần `RebuildDDHedgeStateFromBroker` — broker đã giữ comment, query trực tiếp khi cần.

### `HasDDHedgeAtTier(int i)` flow

```
string targetComment = "DD_HEDGE_T" + IntegerToString(i + 1)   // "DD_HEDGE_T1"
for (int j = 0; j < PositionsTotal(); j++) {
    ulong tk = PositionGetTicket(j)
    if (!PositionSelectByTicket(tk)) continue
    if (PositionGetInteger(POSITION_MAGIC) != MagicNumber) continue
    if (PositionGetString(POSITION_COMMENT) == targetComment) return true
}
return false
```

### `CheckDDHedgeTriggers()` flow

```
if (!g_EnableDDHedge) return
if (CountPositions() == 0) return

// Cooldown chung
if (TimeCurrent() - g_ddHedgeLastFireTime < g_DDHedge_CooldownSec) return

double bal = AccountInfoDouble(ACCOUNT_BALANCE)
double totalProfit = GetTotalProfitExcludeHedge()      // tự động cộng cả DD hedge vì IsHedgePosition không bao gồm DD_HEDGE_T
double ddPct = (totalProfit < 0 && bal > 0) ? MathAbs(totalProfit) / bal * 100 : 0

// Check imbalance (skip nếu MinImbalanceCount = 0)
if (g_DDHedge_MinImbalanceCount > 0) {
    int imbalanceCount = MathAbs(CountByDir(POSITION_TYPE_BUY) - CountByDir(POSITION_TYPE_SELL))
    if (imbalanceCount < g_DDHedge_MinImbalanceCount) return
}

// Quét tier từ cao xuống thấp — fire tier cao nhất chưa có lệnh (tránh duplicate khi DD nhảy mạnh qua nhiều tier)
for (int i = DD_HEDGE_TIER_COUNT - 1; i >= 0; i--) {
    if (ddPct < g_DDHedge_TierThreshold[i]) continue
    if (HasDDHedgeAtTier(i)) continue                // tier đang có lệnh → skip
    if (FireDDHedgeTier(i)) {
        g_ddHedgeLastFireTime = TimeCurrent()
        g_ddHedgeFiredCount++
        return  // chỉ fire 1 tier mỗi tick
    }
}
```

> **Lưu ý quan trọng**: Có 1 chi tiết phải kiểm chứng — `GetTotalProfitExcludeHedge()` (dòng 414–430) hiện check `IsHedgePosition(tk)` để **loại trừ hedge khỏi tổng P/L**. Vì `IsDDHedgePosition` KHÔNG nằm trong `IsHedgePosition`, nên DD hedge **SẼ được cộng vào** tổng P/L → ddPct tự động giảm khi hedge lời, tăng khi hedge lỗ. Đây đúng là behavior user yêu cầu ("coi như lệnh cluster bình thường"). ✓

### `FireDDHedgeTier(i)` flow

```
double lotBuy  = GetTotalLotByDir(POSITION_TYPE_BUY)
double lotSell = GetTotalLotByDir(POSITION_TYPE_SELL)
int hedgeDir
double imbalanceLot
if (lotBuy > lotSell) { imbalanceLot = lotBuy - lotSell;  hedgeDir = SELL }
else                  { imbalanceLot = lotSell - lotBuy;  hedgeDir = BUY  }

double hedgeLot = NormalizeDouble(imbalanceLot * g_DDHedge_TierLotMult[i], 2)
clamp [VOLUME_MIN, VOLUME_MAX]
if (hedgeLot < VOLUME_MIN) return false

string comment = DD_HEDGE_COMMENT_PREFIX + IntegerToString(i + 1)   // "DD_HEDGE_T1"
ok = trade.Buy/Sell(hedgeLot, _Symbol, price, 0, 0, comment)
log + SendLog("dd_hedge", ...)
return ok
```

## 7. Vị trí thay đổi code (theo dòng)

| # | Vị trí | Thay đổi |
|---|--------|----------|
| 1 | sau dòng 19 (constants) | `#define DD_HEDGE_COMMENT_PREFIX "DD_HEDGE_T"` + `#define DD_HEDGE_TIER_COUNT 3` |
| 2 | sau group hedging inputs hiện có (~dòng 65) | Group `<DD Hedge - Multi-Tier>` inputs |
| 3 | sau state hedge vars (~dòng 91) | `g_ddHedgeLastFireTime`, `g_ddHedgeFiredCount` |
| 4 | sau dòng 176 (shadow vars) | Shadow vars cho DD Hedge (KHÔNG có hysteresis nữa) |
| 5 | dòng 263 (`IsHedgePosition`) | **KHÔNG sửa** — DD Hedge KHÔNG vào hàm này (ý đồ thiết kế) |
| 6 | sau dòng 290 | Thêm `IsDDHedgePosition`, `HasDDHedgeAtTier`, `CountDDHedgePositions`, `GetDDHedgeProfit` |
| 7 | sau dòng 596 (Dashboard TP block) | Section "===== DD HEDGE =====" hiển thị tier nào đang có lệnh, cumulative count, profit |
| 8 | sau dòng 631 (`InitShadowVariables`) | Gọi `InitDDHedgeShadow()` |
| 9 | sau dòng 1990 (`CloseWeekendHedgeMonday`) | Module mới: `CheckDDHedgeTriggers`, `FireDDHedgeTier`, `HasDDHedgeAtTier` |
| 10 | dòng 2010 (đầu `CheckEmergencySafety`, sau khối check `g_emergencyHedgeActive`) | Gọi `CheckDDHedgeTriggers()` **TRƯỚC** kiểm tra Emergency 50% — DD Hedge fire trước, Emergency là last resort |
| 11 | dashboard status logic (dòng 486–505) | Branch hiển thị "DD_HEDGE x N" trong status (dưới EMERGENCY priority) |

## 8. Critical files

| File | Vai trò |
|------|---------|
| `tool/v91_PartialTP.mq5` | **Sửa trực tiếp** — thêm module DD Hedge |
| `tool/CLAUDE.md` | Cập nhật module list (thêm DD Hedge) — file CLAUDE.md hiện đang refer đến file `Virtualgwtw2Withcomment.mq5` cũ, có thể cần refresh |

## 9. Functions sẵn có sẽ tái sử dụng

| Function | File:line | Mục đích |
|----------|-----------|----------|
| `GetTotalLotByDir(dir)` | v91:~345 | Tính imbalanceLot |
| `CountByDir(dir)` | v91:~345 | Tính imbalanceCount |
| `GetTotalProfitExcludeHedge()` | v91:~414 | Tính ddPct (auto-include DD hedge) |
| `AccountInfoDouble(ACCOUNT_BALANCE)` | MQL5 std | Tính ddPct |
| `trade.Buy() / trade.Sell()` | MQL5 std | Đặt hedge |
| `SendLog("dd_hedge", ...)` | v91 | Sync remote dashboard |
| `LogStopReason()` | v91 | Log lý do nếu tier cao nhất fire |

## 10. Risk / Edge cases

1. **Hedge tier vừa bị TP đóng, DD vẫn cao → fire lại ngay tick sau?** Có thể, nhưng `DDHedge_CooldownSec = 60s` chặn fire trong 60s tiếp theo → tránh dồn dập. Nếu sau cooldown DD vẫn cao và tier free → fire lần 2 là **đúng behavior** (vì DD chưa hồi).

2. **DD nhảy gap qua nhiều tier cùng lúc** (vd 25% → 55% trong 1 tick) → vòng lặp duyệt từ tier cao xuống thấp đảm bảo fire **tier cao nhất trước**, các tier thấp hơn fire ở tick sau (chờ cooldown). Tránh hedge dồn dập 3 lần trong 1 tick.

3. **EA restart khi đã có DD hedge active** → comment `DD_HEDGE_T<i>` còn trên broker → tick đầu tiên sau restart, `HasDDHedgeAtTier(i)` query thấy → tier vẫn được nhận diện đang in use. Không cần `RebuildState`. Cumulative count `g_ddHedgeFiredCount` reset = 0 sau restart (chỉ ảnh hưởng dashboard, không ảnh hưởng logic).

4. **User đóng tay 1 lệnh DD hedge** → comment biến mất khỏi broker → `HasDDHedgeAtTier(i)` trả false → tier có thể fire lại nếu DD vẫn ≥ ngưỡng. Đây là behavior **chấp nhận được**: user đóng có thể vì muốn EA tự xử lại. Nếu user muốn dừng hẳn DD Hedge thì set `EnableDDHedge=false`.

5. **Tier không thoả `MinImbalanceCount`** (vd account đã rất cân: 30 BUY vs 35 SELL = imbalance 5 < default 10) → không fire, dù DD cao. Là behavior mong muốn (không có lệnh để cân thì hedge cũng vô nghĩa). Set `MinImbalanceCount = 0` để bỏ qua check này.

6. **Emergency Hedge cũ vẫn fire ở 50%** → nếu user để cả 2 cùng bật + Emergency = 50% + DD Hedge tier 3 = 50% → tier 3 fire trước (cùng tick). Tick sau Emergency check vẫn trigger nếu DD vẫn ≥ 50% → freeze. Để tránh, user nên set Emergency = 60% nếu dùng DD Hedge.

7. **Hedge volume vượt VOLUME_MAX** → clamp về VOLUME_MAX (giống code cũ dòng 1865). Phần lot dôi không hedge — DD vẫn cao, nhưng comment `DD_HEDGE_T<i>` đã có trên broker → tier coi như "in use" → không fire bù lại lần nữa. Có thể cải tiến: split thành N lệnh `DD_HEDGE_T<i>` cùng comment. **Ghi nhận** — gặp thì xử lý.

8. **TP đóng đa lệnh nhanh trong cùng 1 cluster, gồm cả hedge tier 1**: comment biến mất → tier 1 free. Nếu DD vẫn ≥ 30% và cooldown đã qua → fire lại. Volume mới = imbalanceLot **hiện tại** (đã giảm sau TP) → hedge mới nhỏ hơn lần đầu. Hợp lý.

## 11. Verification Plan

### Tier 1 — Strategy Tester backtest A/B
- Period: chọn 2 giai đoạn — XAU one-way (4500→4700 hiện tại) và range giằng co.
- Cùng settings 2 run: A = `EnableDDHedge=false` (chỉ Emergency cũ), B = `EnableDDHedge=true` (3 tier 30/40/50).
- Metrics: Max DD%, số lần freeze (Emergency fire), Final P/L, số trade TP, Max concurrent positions.
- Pass: B có Max DD% ≤ A, Emergency fire ít hơn (lý tưởng = 0), Final P/L không tệ hơn quá 10%.

### Tier 2 — Sanity log
- Bật `Print` chi tiết: từng tier fire (DD%, imbalanceCount, lot, dir), từng lần `HasDDHedgeAtTier` skip.
- Verify: tier cao fire trước khi DD nhảy gap.
- Verify: hedge KHÔNG bị bỏ qua khỏi `CheckPartialTP_CloseFar` / `_Combo` (đặt breakpoint hoặc print khi hedge bị chọn để đóng).
- Verify: sau khi TP đóng hedge tier 1 → tick tiếp theo `HasDDHedgeAtTier(0)` trả false → có thể fire lại sau cooldown.

### Tier 3 — Demo 1 tuần
- Mở demo với `EnableDDHedge=true`, threshold conservative: tier 1 = 35%, tier 2 = 45%, tier 3 = 55%.
- Theo dõi dashboard section DD HEDGE — đếm số lần fire mỗi tier.
- Kill MT5 khi tier 1 đã fire → mở lại → verify `HasDDHedgeAtTier(0)` query thấy lệnh broker → không fire trùng.
- Đóng tay 1 hedge → verify tier có thể fire lại nếu DD vẫn cao + qua cooldown.

### Tier 4 — Production guardrails
- Default `EnableDDHedge = false`.
- Khuyến nghị test trên cent account / micro lot trước.
- Monitor dashboard — nếu fire 3 tier liên tiếp trong < 1 phút → có thể có vấn đề threshold quá gần nhau.

---

## 12. Quyết định cuối — đã đủ context để implement

Tất cả câu hỏi mở đã được chốt — xem bảng **1.5 Quyết định đã chốt** ở đầu file. Tóm gọn:

- **3 tier**: 25% / 35% / 45%
- **LotMultiplier**: 1.0× đều mọi tier
- **Volume mỗi lần fire**: `imbalanceCount` lệnh × `FixedLot` (N lệnh nhỏ, không phải 1 lệnh tổng)
- **MinImbalanceCount**: 20 (hedge trễ)
- **Emergency Hedge cũ**: bỏ hẳn (set `EnableEmergencyHedge=false`)
- **Cooldown**: 60s chung
- **Slot tier**: query broker comment, không hysteresis
- **Hedge tham gia TP**: KHÔNG add vào `IsHedgePosition()`
