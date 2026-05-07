# EA Knowledge - GoneWithTheWind v9.1

> File này dùng để Claude Code đọc trước khi sửa code, tiết kiệm token.
> Cập nhật mỗi khi code thay đổi.

## Version hiện tại
- **main** (`Virtualgwtw2Withcomment.mq5`): v9.00
- **v91-partial-tp** (`v91_PartialTP.mq5`): v9.15

## Danh sách hàm

| Hàm | Dòng (v91) | Mô tả |
|-----|------------|-------|
| Pip() | 130 | Tính giá trị 1 pip |
| SpreadPip() | 141 | Spread hiện tại (pip) |
| LogStopReason() | 150 | Log lý do dừng giao dịch |
| IsMonday() | 165 | Check ngày Thứ 2 |
| HasHedgePosition() | 192 | Có lệnh hedge? |
| GetHedgeProfit() | 206 | Tổng profit hedge |
| CloseHedgePosition() | 222 | Đóng hedge |
| CountPositions() | 243 | Đếm tổng lệnh (trừ hedge) |
| CountByDir() | 265 | Đếm lệnh theo hướng (1=BUY, -1=SELL) |
| GetTotalProfitExcludeHedge() | 313 | Tổng profit (trừ hedge) |
| DrawDashboard() | 360 | Vẽ dashboard trên chart |
| AllowStart() | 588 | Lọc EMA: \|EMA10-EMA50\| >= 1.5 pip |
| UpdateAutoEnforce() | 621 | Tự động bật/tắt EnforceStep theo EMA trend + order count |
| DrawVirtualPendingLines() | 613 | Vẽ đường pending ảo |
| DeleteVirtualPendingLines() | 671 | Xóa đường pending |
| HasAnyVirtualPending() | 680 | Có pending ảo nào? |
| HasVirtualPendingPair() | 689 | Có ĐỦ CẶP pending (BUY+SELL)? |
| ClearVirtualPending() | 698 | Xóa tất cả pending ảo |
| RebuildAfterTP() | 722 | Rebuild pending sau TP |
| FindNearestPositionPrice() | 734 | Tìm entry gần nhất |
| PlacePendingPair() | 769 | Đặt cặp BUY/SELL STOP ảo |
| IsTooCloseToSameDir() | 845 | Check khoảng cách tối thiểu cùng chiều |
| CheckAndTriggerVirtualPending() | 878 | Giá chạm pending → mở MARKET |
| CheckAndExecuteFullTP() | 932 | Full TP: đóng tất cả khi tổng lời >= threshold |
| CheckAndExecutePartialTP() | 984 | Dispatcher: gọi CloseFar hoặc Combo theo PartialTP_Mode |
| CheckPartialTP_CloseFar() | 992 | CloseFar: scan top 5 lệnh lỗ (xa→gần), gom lời bù (đóng trọn/gặm lot) |
| CheckPartialTP_Combo() | 1166 | Combo: tìm 3 lệnh (2+1) có tổng lời >= threshold |
| CheckAllTPTypes() | 1210 | Chạy Full TP → Partial TP theo ưu tiên |
| IsWeekendCloseTime() | 1205 | Check giờ gần đóng sàn |
| PlaceWeekendHedge() | 1308 | Đặt hedge cuối tuần |
| CloseWeekendHedgeMonday() | 1322 | Đóng hedge sáng Thứ 2 |
| CloseAllPositions() | 1413 | Đóng tất cả vị thế |
| CheckEmergencySafety() | 1438 | Check DD% max → block giao dịch |
| GetNearestVirtualPendingDistancePip() | 1472 | Khoảng cách đến pending gần nhất |
| NeedRebuildVirtualPending() | 1504 | Check cần rebuild pending? |
| OnInit() | 1572 | Khởi tạo EA |
| OnTick() | 1634 | Hàm chính, chạy mỗi tick |

## Flow OnTick()

```
BƯỚC 0:    Dashboard + Vẽ Pending Lines (1 lần/giây)
BƯỚC 0.1:  Check Reset Price → đóng hết nếu chạm
BƯỚC 0.15: UpdateAutoEnforce() → cập nhật g_enforceStepBuy/Sell
BƯỚC 0.2:  CheckAndTriggerVirtualPending() (nếu không hedge)
BƯỚC 1:   Xử lý Hedge (weekend hedge/close)
BƯỚC 2:   CheckAllTPTypes() → Full TP → Partial TP (CloseFar: đóng trọn/gặm | Combo: 2+1)
  ↳ EARLY EXIT nếu đang hedge
BƯỚC 2.5: CheckEmergencySafety() → block nếu DD% quá cao
BƯỚC 3:   Check Spread → skip nếu quá cao
BƯỚC 5:   NeedRebuildVirtualPending() → rebuild nếu cần
BƯỚC 6:   Đặt pending mới nếu chưa có + AllowStart()
```

## Input Parameters quan trọng

| Parameter | Default | Mô tả |
|-----------|---------|-------|
| MagicNumber | 777 | ID EA |
| FixedLot | 0.01 | Lot cố định |
| StepPip | 150 | Khoảng cách pip giữa lệnh |
| MaxPerSide | 200 | Max lệnh mỗi hướng (DD nặng) |
| MaxSpreadPip | 50 | Spread max cho phép |
| ClusterTP_USD | 1.5 | Full TP threshold ($) |
| PartialTP_USD | 0.3 | Partial TP threshold ($) |
| PartialTP_Mode | CLOSE_FAR | Chế độ Partial TP (CloseFar/Combo) |
| PartialTP_SameDir | 200 | Max lệnh lời gom (chỉ CloseFar) |
| MaxDrawdownPercent | 50 | DD% max block giao dịch |
| EnforceStepBuy | true | Manual: BUY cách >= StepPip (dùng khi Auto=false) |
| EnforceStepSell | true | Manual: SELL cách >= StepPip (dùng khi Auto=false) |
| AutoEnforceStep | true | Tự động bật/tắt EnforceStep theo order count |
| EnforceOnPct | 25 | % MaxPerSide để BẬT enforce |
| EnforceOffPct | 15 | % MaxPerSide để TẮT enforce (hysteresis) |
| RebuildBufferPercent | 30 | % buffer rebuild |

## Bugs đã fix

1. **Anchor fallback** (line 800): Dùng MID nếu nearest position xa > 2*StepPip
2. **Pending validation** (line 811): Đảm bảo BUY STOP >= ask, SELL STOP <= bid → tránh rebuild loop
3. **Wrong position safety** (line 1515): NeedRebuild detect pending sai phía
4. **Partial lot rounding** (line 1119): MathFloor trước NormalizeDouble
5. **EnforceStepBuy/Sell toggle** (v9.14): Tách enforce spacing riêng BUY/SELL, dashboard hiển thị x/max

## Changelog

- **2026-04-08**: CloseFar scan top 5 lệnh lỗ (xa→gần) thay vì chỉ 1 lệnh xa nhất. Tránh kẹt khi trend mạnh
- **2026-04-05**: Auto EnforceStep: bỏ trend-aware, chỉ dùng order count + hysteresis. Xóa EnforceTrendPip
- **2026-04-01**: Tách EnforceStepPip → EnforceStepBuy + EnforceStepSell. Dashboard hiển thị số lệnh dạng x/max
- **2026-04-01**: Bỏ DD-based spacing, thêm toggle EnforceStepPip. Xóa MinDistDD_Percent, MaxPerSide_Normal
- **2026-04-01**: Xóa Harvest TP, thêm enum PARTIAL_TP_MODE (CloseFar/Combo21). Tách CheckAndExecutePartialTP thành dispatcher + 2 hàm
- **2026-03-31**: Thêm Harvest TP (Case 3) - đóng lệnh lời khi lot quá nhỏ để gặm. Giảm default PartialTP_USD 0.5→0.3
- **2026-03-26**: Fix rebuild loop - SELL STOP không đặt được khi giá đi xuống (cả main và v91)
