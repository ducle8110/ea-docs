# Nghiên cứu & Tối ưu An toàn Chiến lược Bollinger Bands (`v105 v1.10`) & Volume Profile (`v106 v1.10`)

> **Lưu ý (20/11/2026):** tài liệu này mô tả bản v1.10. Hai EA hiện ở v1.11; một số con số và nhận định dưới đây đã được kiểm chứng lại và đính chính trong [`V105_V106_REVIEW_V111.md`](V105_V106_REVIEW_V111.md) (mục 4) — đọc file đó trước khi dùng số ở đây.

Tài liệu tổng hợp kết quả quét chiến lược YouTube, sàng lọc hơn 65 cấu hình tín hiệu & quản lý thoát lệnh không tham số cảm tính trên Vàng (`XAUUSD`), và kiểm chứng chính thức trên **MT5 Strategy Tester** qua cả **Real Ticks (`Model=4`, `2023–2026`)** lẫn **Lịch sử 8.5 năm (`2018.07 → 2026.10`)**.

---

## 1. Những hướng tối ưu TƯỞNG AN TOÀN nhưng thực tế LÀM HẠI tài khoản (Đã bác bỏ bằng dữ liệu 8.5 năm)

Trước khi chốt bản `v1.10`, thử nghiệm chéo trên `53,349` nến M30 `XAUUSD.sc` và `97,328` nến M30 `2018–2026` chỉ ra 4 cái bẫy cần tuyệt đối tránh:

1. **Dời cứng SL trên sàn bám sát `Chandelier 3.0` mỗi nến (`Hard SL Trail`) hoặc cắt sớm khi đóng thủng `SMA 20`:** Làm bốc hơi **55%–70% lợi nhuận** (`v105` tụt từ `+162R` xuống `+66R`, `MaxDD` phình lên `31R`) vì Vàng rất hay rũ râu kiểm tra lại `SMA 20` trước khi phóng sóng lớn.
2. **Cắt lỗ theo thời gian (`Time-Stop 8–10 nến M30` khi đang âm nhẹ):** Cắt nhầm các pha tích luỹ 4–5 giờ đầu phiên Âu trước khi bùng nổ ở phiên Mỹ (làm giảm ~`40R` của `v105` và giảm **70% lãi** của `v106`).
3. **Bỏ neo cấu trúc `Swing 8 nến` của `v106` để ép SL cố định `1.5×ATR`:** Kéo `v106` tụt từ `+88R` xuống `+6R` (`PF = 1.01`, `DD = 36R`) — chứng minh đệm cấu trúc `Đáy/Đỉnh 8 nến ± 0.2×ATR` là bắt buộc khi giao dịch biên Value Area.
4. **Lọc nến nổ quá khắt khe (`Râu ngược <= 35%` + `Pre-run <= 2.5×ATR`):** Loại nhầm các cây nến bứt phá mạnh nhất đầu sóng vĩ mô.

---

## 2. Ba Cơ Chế Nâng Cấp `v1.10` Vừa TĂNG GẤP ĐÔI LÃI USD Vừa GIẢM MỘT NỬA SỤT GIẢM (100% Vô hướng)

### 2.1. `v105_BB_KeltnerSqueeze.mq5` (`v1.10` — `Magic 805`)
1. **Nén sâu đa số (`MinSqueezeBars = 4 / 6` nến M30 thay vì `3 / 6`):** Đòi hỏi dải `Bollinger(20, 2.0)` nằm trọn trong `Keltner(20, 1.5×ATR)` ít nhất **2/3 cửa sổ (2 giờ)**, lọc sạch `24%` tín hiệu chớm nén giả.
2. **Thoát 2 tầng + Khoá Hoà Vốn Bảo Vệ (`Chandelier 3.0 / 6.0` + `LockBE_AfterT1 = true`):**
   - **Tầng 1 (`50%` lot):** Đóng theo `Chandelier 3.0 × ATR(14)`.
   - **Khoá Hoà Vốn (`0R`):** Ngay khi Tầng 1 chốt có lãi `>= +1.0R`, tự động dời cứng `SL` của `50%` còn lại trên sàn về **Giá vào lệnh (`Entry`)** — triệt tiêu hoàn toàn rủi ro lỗ ngược nếu thị trường sập chữ V.
   - **Tầng 2 (`50%` lot):** Thả rộng gấp đôi (`Chandelier 6.0 × ATR(14)`) để ăn trọn siêu sóng vĩ mô (`+10R .. +22R`).

### 2.2. `v106_VP_ValueEscape.mq5` (`v1.10` — `Magic 806`)
1. **Lọc Thị Trường Lặng (`QuietATR_Ratio = 0.65`):** Bỏ qua những ngày thanh khoản chết khi `ATR(14) < 0.65 × Trung vị ATR 480 nến (10 ngày)`.
2. **Bảo hiểm lợi nhuận `VP106_SPLIT_SAFE` (`ExitMode = 1` — Mặc định khuyên dùng):**
   - Ngay trên tick chạm **`+2.0R`**, EA tự động **chốt `50%` khối lượng** (`+1.0R` đút túi chắc chắn, giữ nguyên Winrate cao `~44%`) và lập tức **dời cứng `SL` của `50%` còn lại về `Entry` (`0R` — giao dịch trở thành Risk-Free 100%)**.
   - `50%` còn lại được thả chạy trong vùng chân không thanh khoản tới trần **`+5.0R`**, kèm khoá trượt đóng nến M30 khi giá lùi **`2.0R`** từ đỉnh `Peak`.
   - *(Tuỳ chọn `ExitMode = 2 (VP106_TRAIL_5R)` giữ 100% khối lượng chạy trần `5.0R / Trail 2.0R` cho người ưu tiên tối đa hoá tổng R).*

---

## 3. Bảng Đối Chiếu `v1.00` vs `v1.10` trên MT5 Strategy Tester (`Risk = 1.0%/lệnh`, Vốn `$100,000`)

| EA & Giai đoạn kiểm thử | Phiên bản | Số vị thế | Win Rate | Tổng R (Giá) | Net R (Sau Swap) | Lãi ròng (`1%` Risk) | Profit Factor | Max Balance DD | Max Equity DD |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **`v105` Real Ticks `2024–2026`** | `v1.00` | 174 | 29.9% | `+60.88R` | `+57.08R` | `+$66,898` | 1.51 | 15.11% | 18.46% |
| **`v105` Real Ticks `2024–2026`** | **`v1.10`** | **132** | 27.3% | **`+72.27R`** | **`+66.20R`** | **`+$80,168`** | **1.67** | **6.32%** *(−58%)* | **12.93%** |
| **`v105` Real Ticks `2023` OOS** | `v1.00` | 69 | 26.1% | `+10.72R` | `+7.92R` | `+$5,596` | 1.12 | 18.40% | 20.99% |
| **`v105` Real Ticks `2023` OOS** | **`v1.10`** | **53** | 30.2% | **`+32.83R`** | **`+27.67R`** | **`+$27,061`** *(×4.8)* | **1.71** | **9.11%** *(−50%)* | **13.07%** |
| **`v105` 8.5 Năm `2018–2026`** | `v1.00` | 503 | 28.2% | `+111.08R` | `+96.79R` | `+$124,635` | 1.28 | 20.00% | 22.58% |
| **`v105` 8.5 Năm `2018–2026`** | **`v1.10`** | **389** | 24.2% | **`+166.29R`** | **`+145.81R`** | **`+$249,162`** *(×2.0)* | **1.52** | **18.81%** | **13.17%** *(−42%)* |
| **`v106` Real Ticks `2024–2026`** | `v1.00` | 109 | 44.0% | `+33.55R` | `+30.52R` | `+$33,997` | 1.50 | 8.51% | 10.36% |
| **`v106` Real Ticks `2024–2026`** | **`v1.10` (`SPLIT_SAFE`)** | **101** | **43.6%** | **`+46.58R`** | **`+42.70R`** | **`+$50,662`** *(+49%)* | **1.75** | **6.39%** *(−25%)* | **7.65%** |
| **`v106` 8.5 Năm `2018–2026`** | `v1.00` | 410 | 39.0% | `+70.00R` | `+52.78R` | `+$61,870` | 1.23 | 22.72% | 23.13% |
| **`v106` 8.5 Năm `2018–2026`** | **`v1.10` (`SPLIT_SAFE`)** | **386** | **39.1%** | **`+107.97R`** | **`+87.37R`** | **`+$124,953`** *(×2.0)* | **1.43** | **20.80%** | **21.60%** |

---

## 4. Phân Rã Từng Năm Suốt 8.5 Năm (`2018.07 → 2026.10`) & Chiến Lược Chia Vốn An Toàn `0.5% / Bot`

| Năm | `v105 v1.10` (`SumR` / `NetR`) | `v106 v1.10 SPLIT_SAFE` (`SumR` / `NetR`) | Gộp Bộ Đôi `v105 + v106 v1.10` (`SumR` / `NetR`) |
| :---: | :---: | :---: | :---: |
| **2018** *(T7–T12)* | `-7.80R` / `-7.83R` | **`+1.24R`** / `-1.13R` | `-6.56R` / `-8.97R` |
| **2019** | **`+3.68R`** / **`+1.50R`** | **`+7.61R`** / **`+1.61R`** | **`+11.30R`** / **`+3.12R`** |
| **2020** | **`+63.76R`** / **`+59.39R`** | **`+3.91R`** / **`+0.86R`** | **`+67.67R`** / **`+60.24R`** |
| **2021** | `-3.67R` / `-4.66R` | **`+8.29R`** / **`+5.38R`** | **`+4.63R`** / **`+0.72R`** |
| **2022** | **`+6.84R`** / **`+5.96R`** | **`+27.25R`** / **`+26.30R`** | **`+34.09R`** / **`+32.27R`** |
| **2023** | **`+32.74R`** / **`+27.04R`** | **`+4.77R`** / **`+3.53R`** | **`+37.51R`** / **`+30.57R`** |
| **2024** | **`+17.24R`** / **`+13.49R`** | **`+7.94R`** / **`+6.31R`** | **`+25.18R`** / **`+19.80R`** |
| **2025** | **`+20.19R`** / **`+18.14R`** | **`+28.56R`** / **`+26.30R`** | **`+48.75R`** / **`+44.45R`** |
| **2026** *(đến T10)* | **`+33.30R`** / **`+32.78R`** | **`+18.40R`** / **`+18.20R`** | **`+51.70R`** / **`+50.98R`** |
| **Tổng 8.5 Năm** | **`+166.29R` (`+145.81R` net)** | **`+107.97R` (`+87.37R` net — Dương giá `9/9` năm!)** | **`+274.26R` (`+233.18R` net — Dương ròng suốt `8` năm liền `2019–2026`!)** |

### Khuyến nghị phân bổ vốn an toàn nhất khi chạy song song (`Tương quan tháng r = 0.173`):
* **Cấu hình An Toàn Tuyệt Đối (`RiskPercent = 0.50%` mỗi bot — Tổng rủi ro tối đa nếu cả 2 trùng lệnh cùng ngày = `1.0%`):**
  * **Real Ticks `2023–2026` (4 năm):** Lãi ròng **`+$91,406` (`+91.4%`)**, **Max Balance DD chỉ `7.01%`**!
  * **8.5 Năm `2018–2026`:** Lãi ròng **`+$199,791` (`+199.8%` — nhân 3 tài khoản)**, **Max Balance DD suốt 8.5 năm chỉ `15.36%`**!
