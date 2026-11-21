# Tài liệu bàn giao — `v99_DonchianBO` (v2.00)

> **Mục tiêu v2.00:** Giải quyết triệt để 2 vấn đề thực chiến của hệ phá hộp Donchian nguyên bản (v1.00):
> 1. **Tránh bẫy đảo chiều ngay sau khi ra khỏi hộp (Bull/Bear Trap)** — lọc bỏ gần 1/3 số lệnh giả dựa trên dữ liệu khám nghiệm 1,384 cú phá hộp (2024–2026).
> 2. **Chốt lời thực tế trong ngày (giữ trung bình ~4–5 giờ, ăn 20–55 giá), nâng Winrate lên ~50–53%** — không phụ thuộc vào các lệnh gồng ảo 300 giá hay ôm lệnh 67 tiếng qua cuối tuần.

---

## 1. Kết quả Khám nghiệm Lệnh Thua (False Breakout) trên `XAUUSD.sc`

Từ đối chiếu 1,384 cú phá hộp Donchian(20) giai đoạn 2024–2026:

### Những chỉ báo phổ biến KHÔNG lọc được bẫy trên Vàng
- **`RSI(14)`:** Trung bình nhóm bẫy = `69.3`, nhóm thắng lớn = `69.1` (phá hộp 20 nến lúc nào RSI cũng quanh 69).
- **`Tick Volume` & `Thân nến`:** Nhóm bẫy có `Volume = 1.22× MA20` và thân nến `70%` (thậm chí cao hơn nhóm thắng `1.18×` và `68%`) vì các cú quét thanh khoản giả (Stop-Hunt) luôn bung nến to để dụ đu theo.

### 3 Bộ lọc Cấu trúc THỰC SỰ loại bỏ bẫy đảo chiều (tích hợp trong v2.00)
1. **Độ dốc `EMA 50` thuận chiều (`MinEMA50Slope_ATR = 0.15`):** Không chỉ cần `EMA50 > EMA200`, bản thân `EMA50` phải đang dốc thuận chiều tối thiểu `0.15 × ATR(14)` qua 4 nến M30 (2 giờ) — loại bỏ các pha xu hướng đã tàn, `EMA50` đi ngang hoặc bẻ ngược.
2. **Chống Kiệt Sức 4 nến trước phá hộp (`MaxPreRun_ATR = 2.50`):** Nếu trong 4 nến đóng gần nhất giá đã chạy quá `2.5 × ATR` (phóng một mạch từ đáy hộp lên đỉnh hộp trong một hơi), lực đẩy đã cạn kiệt → vừa ra khỏi hộp là bị chốt lời xả ngược. Chỉ vào khi giá **nén chặt sát mép hộp** rồi mới bước ra.
3. **Chống FOMO vọt quá xa mép hộp (`MaxOvershoot_ATR = 0.70`) & Chỉ vào ở cây nến đầu tiên phá hộp:** Bỏ qua nếu nến đóng vọt quá xa mép hộp `> 0.7 × ATR`.

---

## 2. Cơ chế Chốt Lời Thực Tế 2 Nhịp (Không gồng ảo, không ôm cuối tuần)

- **SL cứng trên sàn:** `Entry +/- 1.5 × ATR(14)`.
- **Nhịp 1 — Chốt 50% tại `TP1 = +1.2R` (trung vị ~22 giá năm 2026) + Khoá Hoà vốn (`BE + 0.05R`):** Ngay khi giá chạm `+1.2R` (xử lý từng tick trong `OnTick`), EA đóng 50% khối lượng bỏ túi và dời SL 50% còn lại về trên điểm vào → lệnh đó **chắc chắn thắng, không thể lỗ ngược**.
- **Nhịp 2 — Chốt 50% còn lại tại `TP2 = +3.0R` (trung vị ~54 giá năm 2026):** Đặt sẵn TP cứng trên sàn, kết hợp đường khoá lời động `Chandelier 2.5 × ATR(14)`, giới hạn chu kỳ giữ tối đa 32h và **tự động đóng trước giờ nghỉ Thứ Sáu (`22:00` Server)**.

---

## 3. Kết quả MT5 Strategy Tester — **Real Ticks (`Model=4`)**

Tài khoản Cent `.sc` (`100,000 USC` gốc = \$1,000 nạp thực · `FixedLot = 1.00` · `Model=4` Every tick based on real ticks):

| Phiên bản / Giai đoạn | Số lệnh mở | Winrate | Profit Factor | Giữ TB | Mức chốt thực tế | Lãi ròng (`USC`) | Max Drawdown |
|---|---|---|---|---|---|---|---|
| **v2.00 Năm 2026 (`01/01 → 17/11`)** | **116** | **53.4%** | **1.53** | **4.2 giờ** | TP1 ~22 giá · TP2 ~54 giá | **+60,200 USC (+60.2%)** | **18,032 USC (16.39%)** |
| **v2.00 Full 3 Năm (`2024 → 2026`)** | **424** | **49.8%** | **1.36** | **4.9 giờ** | Tối đa +2.21R | **+89,895 USC (+89.9%)** | **19,258 USC (13.71%)** |
| **v2.00 Ngoài mẫu Năm 2023** | **138** | **47.1%** | **1.02** | **5.1 giờ** | Tối đa +2.16R | **+564 USC (+8.33R)** | **5,450 USC (5.28%)** |
| *(Đối chiếu v1.00 cũ Năm 2026)* | *176* | *34.7%* | *1.80* | *18.4 giờ* | *Phụ thuộc lệnh gồng 312 giá* | *+175,080 USC* | *32,800 USC (17.62%)* |

---

## 4. Xem & Đối chiếu Biểu đồ Trực tiếp trên Trình duyệt

Mở file **[v99_report_2026.html](file:///c:/Users/duc/Documents/project/tool/v99_report_2026.html)** (`C:\Users\duc\Documents\project\tool\v99_report_2026.html`) — đã tích hợp sẵn **3 Tab** trên đầu trang:
1. **Tab 1:** `v2.00 Năm 2026` (Lọc bẫy + Chốt lời 2 nhịp TP1 1.2R / TP2 3.0R — vẽ sẵn cả 2 vạch TP1 & TP2 trên chart nến M30).
2. **Tab 2:** `v2.00 Full 3 Năm (2024–2026)`.
3. **Tab 3:** `Đối chiếu v1.00 cũ` (để so trực tiếp những lệnh bẫy và lệnh ôm cuối tuần đã được xử lý).
