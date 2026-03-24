# Kiến Thức Trading Vàng (XAUUSD) trên MT5

> Tài liệu dành cho người mới bắt đầu

---

## 1. Khái Niệm Cơ Bản

### Vàng trên MT5 là gì?
- Symbol: **XAUUSD** (1 ounce vàng tính bằng USD)
- Ví dụ: XAUUSD = 3050 nghĩa là 1 ounce vàng = 3,050 USD

### Hai loại lệnh chính
| Lệnh | Ý nghĩa | Lãi khi | Lỗ khi |
|-------|----------|---------|--------|
| **BUY** (Long) | Mua vàng | Giá **tăng** | Giá **giảm** |
| **SELL** (Short) | Bán khống vàng | Giá **giảm** | Giá **tăng** |

### Pip / Point là gì?
- Với vàng trên hầu hết broker: **1 pip = 0.01 USD** (ví dụ: 3050.00 → 3050.01)
- Một số broker dùng **1 pip = 0.1 USD** — kiểm tra broker của bạn
- **Point** = đơn vị nhỏ nhất mà giá thay đổi

### Lot là gì?
| Loại | Kích thước | 1 pip ≈ bao nhiêu $ (vàng) |
|------|-----------|---------------------------|
| **1 lot** | 100 ounce | ~$1.00/pip (0.01) hoặc ~$10/pip (0.1) |
| **0.1 lot** (mini) | 10 ounce | ~$0.10/pip hoặc ~$1/pip |
| **0.01 lot** (micro) | 1 ounce | ~$0.01/pip hoặc ~$0.1/pip |

> ⚠️ Giá trị pip phụ thuộc vào broker. Hãy kiểm tra bằng cách mở 1 lệnh 0.01 lot và xem profit thay đổi bao nhiêu khi giá dịch 1 pip.

---

## 2. Các Loại Lệnh (Order Types)

### Lệnh thị trường (Market Order)
- **Buy** — Mua ngay tại giá hiện tại
- **Sell** — Bán ngay tại giá hiện tại

### Lệnh chờ (Pending Order)
| Loại | Ý nghĩa | Đặt ở đâu |
|------|----------|-----------|
| **Buy Limit** | Chờ giá **xuống** rồi mua | **Dưới** giá hiện tại |
| **Sell Limit** | Chờ giá **lên** rồi bán | **Trên** giá hiện tại |
| **Buy Stop** | Giá **vượt lên** thì mua | **Trên** giá hiện tại |
| **Sell Stop** | Giá **phá xuống** thì bán | **Dưới** giá hiện tại |

### Sơ đồ trực quan

Giả sử giá vàng hiện tại đang ở **3050**:

```
    GIÁ
     |
3070 |  ............  SELL LIMIT (chờ giá lên đây rồi BÁN ↓)
     |                  "Giá lên tới đây chắc sẽ quay đầu giảm"
     |
3060 |  ............  BUY STOP (giá vượt lên đây thì MUA ↑)
     |                  "Giá phá lên đây là sẽ bay tiếp"
     |
3050 |  ====<<<<====  GIÁ HIỆN TẠI
     |
3040 |  ............  SELL STOP (giá phá xuống đây thì BÁN ↓)
     |                  "Giá rớt tới đây là sẽ rớt tiếp"
     |
3030 |  ............  BUY LIMIT (chờ giá xuống đây rồi MUA ↑)
     |                  "Giá xuống tới đây chắc sẽ nảy lên"
     |
```

### Cách nhớ đơn giản

```
                    TRÊN giá hiện tại
              ┌─────────────────────────────┐
              │  SELL LIMIT    BUY STOP     │
              │  (bán đỉnh)   (mua breakout)│
              └─────────────────────────────┘
         ============ GIÁ HIỆN TẠI ============
              ┌─────────────────────────────┐
              │  BUY LIMIT     SELL STOP    │
              │  (mua đáy)    (bán breakdown)│
              └─────────────────────────────┘
                    DƯỚI giá hiện tại
```

### So sánh LIMIT vs STOP

```
  LIMIT = "Đợi giá tốt hơn"          STOP = "Đợi giá xác nhận"
  (giá quay đầu - đảo chiều)          (giá tiếp tục - phá vỡ)

  Giá đang 3050, muốn BUY:            Giá đang 3050, muốn BUY:

       3050 ← giá hiện tại                 3060 ← BUY STOP ở đây
         |    đợi giá xuống                   ↑   "giá phá lên = tăng tiếp"
         ↓    rồi mới mua                    |
       3030 ← BUY LIMIT ở đây            3050 ← giá hiện tại
              "giá rẻ hơn = mua"

  Giá đang 3050, muốn SELL:           Giá đang 3050, muốn SELL:

       3070 ← SELL LIMIT ở đây           3050 ← giá hiện tại
              "giá cao hơn = bán"           |
         ↑    đợi giá lên                   ↓   "giá phá xuống = giảm tiếp"
         |    rồi mới bán                 3040 ← SELL STOP ở đây
       3050 ← giá hiện tại
```

### Ví dụ thực tế với EA GoneWithTheWind

```
  EA dùng BUY STOP + SELL STOP (chiến lược breakout):

       3051.50 ← BUY STOP  (+150 pip = +$1.50)
                  "nếu giá phá lên → mua theo"

       3050.00 ← GIÁ HIỆN TẠI

       3048.50 ← SELL STOP (-150 pip = -$1.50)
                  "nếu giá phá xuống → bán theo"

  → Khi 1 lệnh khớp, lệnh kia bị hủy, rồi đặt cặp mới
```

### Mẹo nhớ
- **Limit** = bạn nghĩ giá sẽ **quay đầu** → đặt ngược hướng → "mua rẻ, bán đắt"
- **Stop** = bạn nghĩ giá sẽ **tiếp tục** → đặt cùng hướng breakout → "theo trend"

---

## 3. Spread, Swap, Commission

### Spread
- = **Giá Ask - Giá Bid** (chênh lệch mua/bán)
- Vàng thường có spread **15–50 point** tùy broker và thời điểm
- Spread **rộng** lúc: tin tức lớn, đầu tuần (mở cửa), cuối ngày
- Spread **hẹp** lúc: phiên London + New York chồng nhau (20:00–00:00 VN)

### Swap (phí qua đêm)
- Giữ lệnh qua đêm sẽ bị/được tính swap
- Swap vàng thường **âm** (bị trừ tiền) cho cả Buy và Sell
- **Thứ 4** swap x3 (tính cho thứ 7 + Chủ nhật)
- Xem swap: chuột phải symbol → Specification → Swap Long / Swap Short

### Commission
- Một số broker tính phí commission riêng (ví dụ: $7/lot/lượt)
- Một số broker không tính commission nhưng spread rộng hơn

---

## 4. Quản Lý Vốn (Money Management) ⭐

> **Đây là phần QUAN TRỌNG NHẤT** — hầu hết người mới thua lỗ vì quản lý vốn kém, không phải vì phân tích sai.

### Nguyên tắc cơ bản
1. **Không bao giờ rủi ro quá 1-2% tài khoản** cho 1 lệnh
2. **Luôn đặt Stop Loss** — KHÔNG BAO GIỜ trade mà không có SL
3. **Tính lot size trước** khi vào lệnh

### Công thức tính lot size
```
Lot = (Số tiền chấp nhận mất) / (Khoảng cách SL tính bằng pip × Giá trị 1 pip)
```

**Ví dụ:**
- Tài khoản: $1,000
- Rủi ro: 1% = $10
- Stop Loss: 50 pip (= $5.00 với vàng)
- Giá trị 1 pip cho 0.01 lot = $0.01 (kiểm tra broker)
- Lot = $10 / (50 × $0.10) = $10 / $5 = **0.02 lot**

### Bảng gợi ý lot size (rủi ro 1%, SL 50 pip)
| Tài khoản | Lot size gợi ý |
|-----------|---------------|
| $100 | 0.01 lot |
| $500 | 0.01 lot |
| $1,000 | 0.01–0.02 lot |
| $5,000 | 0.05–0.10 lot |
| $10,000 | 0.10–0.20 lot |

> ⚠️ Bảng trên chỉ mang tính tham khảo. **Luôn tự tính** dựa trên broker của bạn.

---

## 5. Stop Loss (SL) và Take Profit (TP)

### Stop Loss — Điểm cắt lỗ
- Giới hạn số tiền bạn có thể mất
- **BUY**: SL đặt **dưới** giá vào
- **SELL**: SL đặt **trên** giá vào
- Nên đặt SL dựa trên **cấu trúc giá** (đỉnh/đáy gần nhất), không phải con số tròn

### Take Profit — Điểm chốt lời
- **BUY**: TP đặt **trên** giá vào
- **SELL**: TP đặt **dưới** giá vào

### Tỷ lệ Risk:Reward (R:R)
- Tối thiểu nên **1:1.5** hoặc **1:2**
- Nghĩa là: nếu SL = 50 pip thì TP nên ≥ 75–100 pip
- Với R:R = 1:2, bạn chỉ cần thắng **34%** lệnh là đã hòa vốn

---

## 6. Phiên Giao Dịch Vàng

| Phiên | Giờ VN (UTC+7) | Đặc điểm |
|-------|----------------|----------|
| **Sydney** | 04:00 – 13:00 | Ít biến động, spread rộng |
| **Tokyo** | 06:00 – 15:00 | Biến động nhẹ |
| **London** | 14:00 – 23:00 | Biến động mạnh, volume lớn |
| **New York** | 19:00 – 04:00 | Biến động mạnh nhất |
| **London + NY chồng** | 19:00 – 23:00 | ⭐ Thời điểm tốt nhất |

### Thời điểm nên tránh trade
- **Đầu tuần** (thứ 2 sáng sớm) — gap, spread rộng
- **Cuối tuần** (thứ 6 tối) — biến động bất thường
- **Trước/trong tin NFP, FOMC, CPI** — biến động cực mạnh, dễ bị quét SL
- **Ngày lễ lớn** (Giáng sinh, Năm mới) — thanh khoản thấp

---

## 7. Phân Tích Cơ Bản Cho Vàng

### Vàng tăng giá khi:
- **USD yếu** (DXY giảm)
- **Lạm phát cao** (vàng là nơi trú ẩn)
- **Bất ổn địa chính trị** (chiến tranh, khủng hoảng)
- **Lãi suất giảm** hoặc kỳ vọng FED hạ lãi suất
- **Nhu cầu mua vàng** từ ngân hàng trung ương tăng

### Vàng giảm giá khi:
- **USD mạnh** (DXY tăng)
- **Lãi suất tăng** (tiền gửi hấp dẫn hơn vàng)
- **Thị trường chứng khoán tốt** (tiền chảy vào cổ phiếu)
- **Lạm phát được kiểm soát**

### Tin tức quan trọng ảnh hưởng vàng
| Tin | Tần suất | Tác động |
|----|---------|---------|
| **NFP** (Non-Farm Payrolls) | Thứ 6 đầu tháng | ⭐⭐⭐ Rất mạnh |
| **FOMC / Quyết định lãi suất** | 8 lần/năm | ⭐⭐⭐ Rất mạnh |
| **CPI** (Lạm phát) | Hàng tháng | ⭐⭐⭐ Mạnh |
| **PPI** (Giá sản xuất) | Hàng tháng | ⭐⭐ Trung bình |
| **GDP** | Hàng quý | ⭐⭐ Trung bình |
| **Jobless Claims** | Hàng tuần | ⭐ Nhẹ |

> Xem lịch tin tức tại: **ForexFactory.com** (chọn filter → USD, High Impact)

---

## 8. Phân Tích Kỹ Thuật Cơ Bản

### Support & Resistance (Hỗ trợ & Kháng cự)
- **Support** (hỗ trợ): vùng giá mà giá **khó giảm** qua → cân nhắc BUY
- **Resistance** (kháng cự): vùng giá mà giá **khó tăng** qua → cân nhắc SELL
- Khi giá **phá vỡ** support → support cũ trở thành resistance mới (và ngược lại)

### Trendline (Đường xu hướng)
- **Uptrend**: nối các đáy cao dần → xu hướng tăng → ưu tiên BUY
- **Downtrend**: nối các đỉnh thấp dần → xu hướng giảm → ưu tiên SELL

### Moving Average (Đường trung bình)
- **EMA 20**: xu hướng ngắn hạn
- **EMA 50**: xu hướng trung hạn
- **EMA 200**: xu hướng dài hạn
- Giá trên EMA → xu hướng tăng; Giá dưới EMA → xu hướng giảm
- **Golden Cross**: EMA ngắn cắt lên EMA dài → tín hiệu tăng
- **Death Cross**: EMA ngắn cắt xuống EMA dài → tín hiệu giảm

### Candlestick cơ bản (Nến Nhật)
| Mẫu nến | Ý nghĩa |
|---------|---------|
| **Hammer** (Búa) | Đuôi dài phía dưới → có thể đảo chiều tăng |
| **Shooting Star** (Sao băng) | Đuôi dài phía trên → có thể đảo chiều giảm |
| **Engulfing tăng** | Nến xanh nuốt nến đỏ trước → tăng mạnh |
| **Engulfing giảm** | Nến đỏ nuốt nến xanh trước → giảm mạnh |
| **Doji** | Thân rất nhỏ → thị trường do dự, chờ xác nhận |

---

## 9. Các Sai Lầm Phổ Biến Của Người Mới ❌

1. **Không đặt Stop Loss** — Sai lầm chết người. 1 lệnh xóa sạch tài khoản.
2. **Vào lot quá lớn** — Tham lam → margin call → cháy tài khoản.
3. **Revenge trading** — Thua → tức giận → vào lệnh liều → thua thêm.
4. **Overtrading** — Vào quá nhiều lệnh 1 ngày. Chất lượng > Số lượng.
5. **Trade theo cảm xúc** — Sợ hãi, tham lam, hy vọng đều là kẻ thù.
6. **Không có kế hoạch** — Phải biết trước: vào ở đâu, SL ở đâu, TP ở đâu.
7. **Dời Stop Loss ra xa** — Khi lệnh đang lỗ mà dời SL = phá vỡ kỷ luật.
8. **Chốt lời quá sớm** — Thắng 5 pip đóng, lỗ 50 pip mới cắt → R:R ngược.
9. **Trade tin tức khi chưa có kinh nghiệm** — Biến động mạnh, spread rộng, dễ lỗ nặng.
10. **Không ghi nhật ký giao dịch** — Không học từ lỗi sai → lặp lại mãi.

---

## 10. Checklist Trước Khi Vào Lệnh ✅

```
□ Xu hướng chính là gì? (tăng/giảm/sideway)
□ Có vùng support/resistance rõ ràng không?
□ Lý do vào lệnh là gì? (không phải "cảm giác")
□ Stop Loss đặt ở đâu? Bao nhiêu pip?
□ Take Profit đặt ở đâu? R:R có ≥ 1:1.5 không?
□ Lot size đã tính đúng chưa? (≤ 1-2% rủi ro)
□ Có tin tức quan trọng sắp ra không?
□ Đang ở phiên giao dịch tốt không?
□ Tâm lý có ổn không? (không tức giận, không mệt mỏi)
```

---

## 11. Thuật Ngữ Thường Gặp

| Thuật ngữ | Giải thích |
|-----------|-----------|
| **Bid** | Giá bạn bán được (giá broker mua) |
| **Ask** | Giá bạn mua được (giá broker bán) |
| **Spread** | Ask - Bid |
| **Leverage** | Đòn bẩy (ví dụ 1:100 = $1 điều khiển $100) |
| **Margin** | Tiền ký quỹ cần để mở lệnh |
| **Free Margin** | Tiền còn lại có thể dùng mở lệnh mới |
| **Margin Level** | (Equity / Margin) × 100% |
| **Margin Call** | Cảnh báo khi Margin Level quá thấp (~100%) |
| **Stop Out** | Broker tự đóng lệnh khi Margin Level quá thấp (~20-50%) |
| **Equity** | Balance + Lãi/Lỗ đang mở |
| **Balance** | Số dư tài khoản (chưa tính lệnh đang mở) |
| **Drawdown** | Mức sụt giảm từ đỉnh equity |
| **Slippage** | Chênh lệch giá đặt lệnh vs giá khớp thực tế |
| **Gap** | Khoảng trống giá (thường xảy ra đầu tuần) |
| **Hedging** | Mở cả Buy + Sell cùng lúc để khóa lỗ/lãi |
| **Scalping** | Giao dịch siêu ngắn (vài phút) |
| **Day trading** | Giao dịch trong ngày, đóng hết trước khi ngủ |
| **Swing trading** | Giữ lệnh vài ngày đến vài tuần |

---

## 12. Lộ Trình Học Trading Cho Người Mới

### Tháng 1–2: Nền tảng
- [ ] Hiểu các khái niệm cơ bản (file này)
- [ ] Mở tài khoản **Demo** trên MT5
- [ ] Tập vào/ra lệnh, đặt SL/TP trên demo
- [ ] Học đọc nến Nhật cơ bản
- [ ] Học vẽ Support/Resistance

### Tháng 3–4: Xây dựng chiến lược
- [ ] Chọn 1 chiến lược đơn giản và tập trung vào nó
- [ ] Backtest chiến lược trên dữ liệu lịch sử
- [ ] Trade demo ít nhất 100 lệnh
- [ ] Ghi nhật ký giao dịch mỗi lệnh

### Tháng 5–6: Demo nâng cao
- [ ] Đánh giá kết quả: win rate, R:R, drawdown
- [ ] Điều chỉnh chiến lược
- [ ] Luyện quản lý tâm lý
- [ ] Đạt được lợi nhuận **ổn định** trên demo (≥ 2 tháng liên tiếp)

### Sau 6 tháng: Tài khoản thật
- [ ] Bắt đầu với số tiền **nhỏ** (có thể chấp nhận mất)
- [ ] Lot size **nhỏ nhất** (0.01)
- [ ] Giữ nguyên kỷ luật như trên demo
- [ ] Tăng dần khi có kết quả ổn định

---

## 13. Tài Nguyên Hữu Ích

- **ForexFactory.com** — Lịch tin tức kinh tế
- **TradingView.com** — Biểu đồ và phân tích
- **Investing.com** — Tin tức, dữ liệu kinh tế
- **MQL5.com** — Cộng đồng MT5, tài liệu MQL5
- **BabyPips.com** — Khóa học miễn phí (tiếng Anh)

---

> 📌 **Ghi nhớ**: Trading là marathon, không phải sprint. Bảo toàn vốn là ưu tiên số 1.
> Kiếm tiền từ trading là có thể, nhưng cần thời gian, kỷ luật, và kiên nhẫn.
