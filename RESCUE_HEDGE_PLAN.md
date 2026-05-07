# Rescue Hedge — Cứu cluster lệnh xa khi giá chạy 1 chiều

## 1. Bối cảnh

Trader đang chạy `tool/v91_PartialTP.mq5` (grid hedging EA, MQL5) cho XAUUSD.

**Vấn đề thực tế từ ảnh chụp dashboard ngày 5–7/5/2026:**
- Giá XAU chạy mạnh 4500 → 4700 không có nhịp hồi.
- 81 lệnh đang mở: 15 BUY (vùng 4690–4720), 66 SELL (vùng 4500–4670).
- Profit floating: **−$44,378.88**, DD **57%** (đã chạm Emergency Hedge 50%).
- Cluster SELL ở 4500–4550 bị "bỏ rơi" — không có lệnh BUY đối ứng đủ để Combo/CloseFar TP gom.
- Mode hiện tại: TP Mode = Combo. Đã Partial 78 lần ($46,625), Loss cut 70 lần (−$24,327), nhưng cluster xa vẫn kẹt.

**Ý tưởng:** khi giá đã đi xa khỏi cluster xa nhất một khoảng đủ lớn → chủ động đặt 1 lệnh hedge đối ứng tại giá hiện tại với volume = tổng lot cluster xa. Hedge này hoạt động như "rescue order":
- Giá tiếp tục đi → hedge lời, đủ bù lỗ cluster → đóng cả hai (cắt lỗ có kiểm soát).
- Giá quay đầu → cắt 1 phần hedge để giảm thiệt hại; nếu hồi đủ thì đóng cả hedge + cluster (theo lựa chọn user).

## 2. Quyết định đã chốt với user (5/2026)

| # | Quyết định | Giá trị |
|---|------------|---------|
| 1 | Đơn vị | **pip** (đồng bộ với code hiện có; 150 pip = $1.5 trên XAU broker này → 1 pip = $0.01) |
| 2 | Trigger | **Pure pip distance + cluster size**, KHÔNG dùng DD% |
| 3 | Recovery exit | **Đóng cả hedge + cluster** (mục đích chính = cắt lỗ có kiểm soát) |
| 4 | MaxConcurrent | **1** rescue hedge tại 1 thời điểm |
| 5 | Cách triển khai | **Tạo file mới** `tool/v92_RescueHedge.mq5` (copy v91 + thêm module) — tránh conflict với v91 đang chạy live |
| 6 | Branch | Commit branch `dual-tp` hiện tại trước, sau đó tạo file mới |

## 3. Kiến trúc Rescue Hedge

### 3.1 Trigger — khi nào đặt rescue hedge

Trigger khi đồng thời:

```
A. EnableRescueHedge = true
B. KHÔNG có rescue hedge active (g_rescueSlotActive == false)
C. KHÔNG có Emergency hedge active
D. Đã qua cooldown (>= RescueHedge_CooldownSec) từ lần rescue trước
E. Có 1 cluster "xa kẹt":
   - majorityDir = bên có nhiều lot hơn (SELL nếu sumLotSell > sumLotBuy)
   - farthestEntry = entry xa nhất giá hiện tại theo majorityDir
   - distFromCurrent(farthestEntry) >= RescueHedge_TriggerPip
   - Đếm số lệnh majorityDir có entry trong [farthestEntry, farthestEntry ± ClusterRangePip]
   - Số lệnh >= RescueHedge_MinClusterCount
   - Tổng lot cluster >= RescueHedge_MinClusterLot
```

**Ví dụ kiểm chứng theo tình huống user mô tả:**
> "Giá sell thấp nhất 4000, có 20 lệnh sell từ 4000–4050, giá hiện tại lên 4100"
- farthestEntry = 4000, ClusterRangePip = 5000 (= $50) → cluster = 4000–4050 (20 lệnh) ✓
- distFromCurrent = 4100 − 4000 = $100 = 10000 pip → so với TriggerPip = 10000 → trigger ✓
- majorityDir = SELL → mở **BUY hedge** tại 4100, lot = tổng lot 20 SELL.

### 3.2 Volume hedge

`hedgeLot = cluster.totalLot` (đối ứng 1:1). Hướng = ngược majorityDir.

### 3.3 Cut profit (giá tiếp tục đi xa cluster)

Đóng cả hedge + cluster khi:

```
hedgeProfit + clusterCurrentLoss >= RescueHedge_CutNetUSD
   HOẶC
hedgePip >= RescueHedge_CutProfitPip      (fallback theo pip)
```

### 3.4 Partial cut & Recovery exit (giá quay đầu)

```
Priority 1 — Recovery exit:
   IF accountNet (TotalProfitExcludeHedge + hedgeProfit) >= RescueHedge_RecoveryNetUSD:
       → đóng CẢ hedge + cluster (theo lựa chọn user)

Priority 2 — Damage control (partial):
   IF hedgeLossPip >= RescueHedge_PartialCutTriggerPip
      AND grace period đã qua (RescueHedge_GraceSec)
      AND partialCutDone < RescueHedge_MaxPartialCuts:
       → cắt RescueHedge_PartialCutPct% lot hedge
       → partialCutDone++
       IF partialCutDone >= MaxPartialCuts:
           → đóng nốt phần hedge còn lại + đóng cluster (force close)
```

### 3.5 Tách biệt với cơ chế hiện có

| Item | Cách xử lý |
|------|-----------|
| Comment | `RESCUE_HEDGE\|S0\|A<anchorEntry>` (slot ID + anchor để rebuild khi EA restart) |
| `IsHedgePosition()` | **Mở rộng** trả `true` cho RESCUE_HEDGE → CloseFar/Combo/Cluster TP đều bỏ qua |
| `IsRescueHedgePosition()` | **MỚI** — check riêng |
| `g_hedgeActive` cũ | KHÔNG dùng cho rescue. Tách flag riêng `g_rescueSlotActive` (rescue KHÔNG block grid mở lệnh mới như Emergency làm) |
| Vs Emergency | Emergency vẫn ưu tiên cao hơn. Nếu trong lúc rescue active mà DD chạm 50% → Emergency vẫn fire bình thường |
| Restart EA | Trong `OnInit`: scan position có comment chứa `RESCUE_HEDGE` → parse anchor → re-detect cluster theo entry range để rebuild slot |

## 4. Inputs mới (thêm sau group Anti-Pile)

```mql5
input string Setting_Rescue = "=====<Rescue Hedge>=====";
input bool   EnableRescueHedge          = false;     // OFF default — bật explicit
input double RescueHedge_TriggerPip     = 10000;     // Khoảng cách giá → cluster xa nhất ($100 trên XAU)
input double RescueHedge_ClusterRangePip = 5000;     // Range gom cluster ($50 — như user mô tả 4500-4550)
input int    RescueHedge_MinClusterCount = 5;        // Số lệnh tối thiểu trong cluster
input double RescueHedge_MinClusterLot  = 0.30;      // Tổng lot tối thiểu
input double RescueHedge_CutNetUSD      = 0.5;       // Cut khi (hedgeProfit + clusterLoss) >= USD
input double RescueHedge_CutProfitPip   = 15000;     // Cut fallback theo pip ($150)
input double RescueHedge_RecoveryNetUSD = 0.3;       // Recovery exit khi tổng tài khoản dương
input double RescueHedge_PartialCutTriggerPip = 2000; // Lỗ hedge ($20) → cắt phần
input int    RescueHedge_PartialCutPct  = 30;         // % lot hedge cắt mỗi lần
input int    RescueHedge_MaxPartialCuts = 2;          // Hết quota → force close
input int    RescueHedge_GraceSec       = 60;         // Không partial cut trong 60s đầu
input int    RescueHedge_CooldownSec    = 300;        // Cooldown giữa 2 lần rescue
```

> Số mặc định dựa trên ví dụ user mô tả (4000–4050 cluster, giá lên 4100 → trigger). Tinh chỉnh sau backtest.

## 5. State variables mới

```mql5
struct RescueHedgeRecord {
   ulong    hedgeTicket;
   int      majorityDir;        // SELL=−1 (cluster bị kẹt là SELL), BUY=1
   double   anchorEntry;        // farthest cluster entry
   double   hedgeEntry;
   double   hedgeLot;
   double   clusterTotalLossAtTrigger;
   ulong    clusterTickets[200];
   int      clusterTicketCount;
   int      partialCutDone;
   datetime triggerTime;
   datetime lastCutTime;
};
RescueHedgeRecord g_rescueSlot;
bool     g_rescueSlotActive    = false;
datetime g_rescueLastCloseTime = 0;     // cho cooldown
int      g_rescueHedgeCount    = 0;     // cumulative
double   g_rescueHedgeProfit   = 0;     // cumulative net từ rescue (đã đóng)
int      g_rescuePartialCount  = 0;
```

## 6. Function signatures mới

```mql5
// === Detection ===
bool   IsRescueHedgePosition(ulong ticket);
bool   HasRescueHedgePosition();
double GetRescueHedgeProfit();        // profit hedge của slot active

// === Cluster analysis ===
struct ClusterInfo {
   ulong   tickets[200];
   int     count;
   double  totalLot;
   double  totalLoss;       // currentLoss (sum negative profits)
   double  anchorEntry;
   int     dir;              // majority dir
};
bool   ComputeFarCluster(int majorityDir, ClusterInfo &out);

// === Core ===
bool   CheckRescueHedgeTrigger();              // gọi mỗi tick
bool   PlaceRescueHedge(const ClusterInfo &cluster);
void   ManageRescueHedge();                    // chỉ gọi khi g_rescueSlotActive
void   CloseRescueSlot(string reason);         // đóng cả hedge + cluster
void   PartialCloseRescueHedge();
void   SyncRescueSlotState();                  // đầu OnTick: kiểm tra hedge bị đóng manual
void   RebuildRescueSlotFromBroker();          // gọi trong OnInit
```

### `CheckRescueHedgeTrigger()` flow
```
if (!EnableRescueHedge) return false
if (g_rescueSlotActive) return false
if (HasEmergencyHedgePosition()) return false
if (TimeCurrent() - g_rescueLastCloseTime < RescueHedge_CooldownSec) return false

double lotBuy  = GetTotalLotByDir(POSITION_TYPE_BUY)
double lotSell = GetTotalLotByDir(POSITION_TYPE_SELL)
int majorityDir = (lotSell > lotBuy) ? POSITION_TYPE_SELL : POSITION_TYPE_BUY

ClusterInfo cluster
if (!ComputeFarCluster(majorityDir, cluster)) return false
if (cluster.count < RescueHedge_MinClusterCount) return false
if (cluster.totalLot < RescueHedge_MinClusterLot) return false

double currentMid = SymbolInfoDouble(Symbol(), SYMBOL_BID)  // hoặc ASK tuỳ dir
double distPip = MathAbs(currentMid - cluster.anchorEntry) / Pip()
if (distPip < RescueHedge_TriggerPip) return false

return PlaceRescueHedge(cluster)
```

### `ManageRescueHedge()` flow
```
double hedgeProfit = GetRescueHedgeProfit()
double clusterLoss = SumCurrentLossOfCluster(g_rescueSlot.clusterTickets)
double accountNet  = GetTotalProfitExcludeHedge() + hedgeProfit
double hedgePip    = ComputeHedgePipP&L()

// Priority 1: Cut profit (giá tiếp tục đi xa cluster)
if (hedgeProfit + clusterLoss >= RescueHedge_CutNetUSD ||
    hedgePip >= RescueHedge_CutProfitPip) {
    CloseRescueSlot("CUT_PROFIT")
    return
}

// Priority 2: Recovery exit (giá hồi đủ)
if (accountNet >= RescueHedge_RecoveryNetUSD) {
    CloseRescueSlot("RECOVERY")  // đóng cả hedge + cluster (theo user)
    return
}

// Priority 3: Partial cut on retracement
if (TimeCurrent() - g_rescueSlot.triggerTime < RescueHedge_GraceSec) return
double hedgeLossPip = (hedgePip < 0) ? -hedgePip : 0
if (hedgeLossPip >= RescueHedge_PartialCutTriggerPip
    && g_rescueSlot.partialCutDone < RescueHedge_MaxPartialCuts) {
    PartialCloseRescueHedge()
    g_rescueSlot.partialCutDone++
    if (g_rescueSlot.partialCutDone >= RescueHedge_MaxPartialCuts) {
        CloseRescueSlot("PARTIAL_EXHAUSTED")
    }
}
```

## 7. File và thứ tự sửa đổi

### Bước 0 — Commit branch hiện tại
Branch `dual-tp` đang có thay đổi pending (`tool` modified, `ea-remote/` untracked, một số file deleted). Commit/stash trước khi tạo file mới.

### Bước 1 — Tạo file mới
- Source: `tool/v91_PartialTP.mq5` (2285 dòng)
- Destination: `tool/v92_RescueHedge.mq5` (copy nguyên rồi thêm module)
- Cập nhật version string trong file.

### Bước 2 — Vị trí thêm code

| # | Vị trí (dòng) | Thay đổi |
|---|--------|----------|
| 1 | sau 19 | `#define RESCUE_HEDGE_COMMENT "RESCUE_HEDGE"` |
| 2 | sau 76 (cuối Anti-Pile inputs) | Group `<Rescue Hedge>` inputs (13 dòng) |
| 3 | sau ~91 (state vars hedge cũ) | `g_rescueSlot`, `g_rescueSlotActive`, counters |
| 4 | sau ~131 (struct PartialCombo) | Struct `RescueHedgeRecord`, `ClusterInfo` |
| 5 | sau 176 (shadow vars) | Shadow cho rescue inputs |
| 6 | dòng 263 (`IsHedgePosition`) | Thêm `\|\| StringFind(comment, RESCUE_HEDGE_COMMENT) >= 0` |
| 7 | sau 290 | `IsRescueHedgePosition`, `HasRescueHedgePosition`, `GetRescueHedgeProfit` |
| 8 | sau 410 (`GetTotalLotByDir`) | `ComputeFarCluster`, helper `SumCurrentLossOfCluster` |
| 9 | sau 596 (Dashboard TP block) | Section "===== RESCUE HEDGE =====" trên dashboard |
| 10 | sau 631 (`InitShadowVariables`) | Init rescue shadow |
| 11 | sau 1990 (`CloseWeekendHedgeMonday`) | Module mới: `CheckRescueHedgeTrigger`, `PlaceRescueHedge`, `ManageRescueHedge`, `CloseRescueSlot`, `PartialCloseRescueHedge`, `SyncRescueSlotState`, `RebuildRescueSlotFromBroker` |
| 12 | trong `OnInit` | Gọi `RebuildRescueSlotFromBroker()` |
| 13 | đầu OnTick (sau ~2206) | `SyncRescueSlotState()` |
| 14 | trước `CheckAllTPTypes()` (~2236) | `if (g_rescueSlotActive) ManageRescueHedge();` |
| 15 | sau emergency check (~2240) | `CheckRescueHedgeTrigger()` (KHÔNG return — vẫn chạy step 3-6 grid) |
| 16 | dashboard status logic (486–505) | Branch RESCUE_ACTIVE (priority sau EMERGENCY) |

### File khác cần update
- `tool/CLAUDE.md` (nếu tồn tại): bump version, thêm section module Rescue Hedge.

## 8. Critical files

| File | Vai trò |
|------|---------|
| `C:\Users\duc\Downloads\Tool trade\tool\v91_PartialTP.mq5` | **Source** để copy (READ-ONLY trong v92) |
| `C:\Users\duc\Downloads\Tool trade\tool\v92_RescueHedge.mq5` | **NEW** — file chính chứa toàn bộ module mới |
| `C:\Users\duc\Downloads\Tool trade\tool\CLAUDE.md` | Cập nhật doc version + module list |

## 9. Functions sẵn có sẽ tái sử dụng

| Function | File:line | Mục đích |
|----------|-----------|----------|
| `Pip()`, `SpreadPip()` | v91:~93 | Convert pip ↔ price |
| `GetTotalLotByDir(dir)` | v91:~345 | Tổng lot 1 hướng |
| `CountByDir(dir)` | v91:~345 | Đếm lệnh |
| `GetTotalProfitExcludeHedge()` | v91:~414 | P/L không tính hedge — DÙNG CHO `accountNet` |
| `GetPositionTrueProfit(tk)` | v91 | Profit trừ swap+commission |
| `IsHedgePosition()` | v91:~263 | **CẦN MỞ RỘNG** để bao phủ rescue |
| `DrawDashboard()` | v91:~459 | **CẦN BỔ SUNG** section rescue |
| `OnTradeTransaction` | v91 | **CẦN BỔ SUNG** detect ticket bị đóng manual |

## 10. Risk / Edge cases

1. **Cluster shrink trong khi hedge active**: Combo/CloseFar có thể đóng vài ticket trong cluster sau khi rescue mở. Slot lưu danh sách ticket cố định → cần verify ticket còn sống mỗi tick trước khi cộng loss.
2. **Hedge volume vượt SYMBOL_VOLUME_MAX**: Nếu cluster lot lớn (>6 lot), broker từ chối lệnh single. Cần split thành N lệnh `RESCUE_HEDGE` cùng slot. Nếu lệnh đầu fill mà lệnh sau fail → log + giữ partial hedge với hedgeLot thực fill.
3. **Giá nhảy gap ngay sau khi mở hedge**: hedge vào lỗ ngay → `RescueHedge_GraceSec=60s` chặn partial cut quá sớm.
4. **EA restart khi hedge đang active**: Comment vẫn còn trên broker, state RAM mất → `RebuildRescueSlotFromBroker()` parse comment `RESCUE_HEDGE|S0|A4525` để khôi phục `anchorEntry`. Cluster ticket được re-detect bằng entry range.
5. **Trigger lặp ngay sau cut**: Sau khi cut cluster, nếu vẫn còn cluster con khác đủ điều kiện → cooldown 300s chặn rescue back-to-back.
6. **User đóng hedge tay**: `SyncRescueSlotState()` đầu tick check `PositionSelectByTicket`; nếu false → reset slot, set `g_rescueLastCloseTime`, vẫn giữ cluster.

## 11. Verification Plan

### Tier 1 — Strategy Tester backtest A/B
- Period: chọn giai đoạn XAU one-way move (5/2026 4500→4700 hoặc Mar 2024 spike).
- Cùng settings 2 run: A = rescue OFF, B = rescue ON.
- Metrics: Max DD%, số lần Emergency trigger, Final P/L, max hedge concurrent.
- Pass: B có Max DD% thấp hơn A ≥ 5%, không có hedge stuck > 24h.

### Tier 2 — Sanity log trên Tester
- Bật `Print` chi tiết: trigger params (currentMid, anchor, distPip, clusterCount, totalLot), từng partial cut, từng recovery/cut event.
- Verify thứ tự sự kiện đúng.
- Verify rescue hedge KHÔNG bị Combo/CloseFar gom (test bằng force-state).

### Tier 3 — Demo 1 tuần
- `RescueHedge_MaxConcurrent=1`, threshold conservative (TriggerPip=15000 = $150).
- Theo dõi dashboard section RESCUE.
- Test EA restart: kill MT5, mở lại — slot rebuild từ broker.
- Test manual close hedge: state reset đúng, cluster vẫn quản lý bình thường.
- Verify Emergency vẫn fire nếu rescue thất bại.

### Tier 4 — Production guardrails
- Default `EnableRescueHedge = false`.
- `SendLog("rescue", action, params)` để remote dashboard track.
- Test trên cent account / micro lot trước live.

---

## Câu hỏi mở dành cho user review

1. **TriggerPip = 10000 (= $100)** có quá xa không? Trong tình huống ảnh, lệnh SELL 4500 vs giá 4700 = $200 = 20000 pip — quá ngưỡng từ lâu. Có thể giảm 5000 ($50) để fire sớm hơn.
2. **MinClusterLot = 0.30** với FixedLot=0.01 → cần ≥ 30 lệnh trong cluster. Có cao quá không cho cluster nhỏ ($50 range)?
3. **CutNetUSD = $0.5** có đúng tinh thần "cắt lỗ" không? Cluster đang lỗ vài chục $, hedge phải gồng đến khi lời gần bằng → có thể chỉnh lên $5–$10 để có buffer an toàn.
4. **PartialCutPct = 30%** mỗi lần, MaxPartialCuts = 2 → tổng cắt được 30% + 30%×70% = 51% lot hedge trước khi force close. Có muốn cắt mạnh hơn (50% × 2)?
5. Có muốn thêm **alert** (Notification/Email/Push) khi trigger / cut / partial không? EA hiện đã có `SendLog` cho remote sync.
