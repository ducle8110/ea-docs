# Chiến lược Tyler Goedtel (TG Capital) — London Session FVG + EMA Stack

> Nguồn bóc tách: Video TikTok `@alvormarket` (`7707269530665422098`) kết hợp đối chiếu tài liệu gốc của **Tyler Goedtel (@TylerG_Capital / TG Capital)** và xác minh trực tiếp trên dữ liệu nến **M30 MetaTrader 5** (`EURUSD.sc` ngày `07/04/2026` & `XAUUSD.sc` ngày `28/08/2025`).

---

## 1. Giải mã các lỗi chính tả (ASR) từ bản thu âm

| Cụm bị sai trong tool cũ | Thuật ngữ chuẩn | Ý nghĩa trong chiến lược |
|---|---|---|
| `trê đình`, `Ovochay`, `SCAP`, `sư Quinh` | **Trading, Overtrade, Scalp, Swing** | Phong cách giao dịch ít lệnh, giữ sóng dài |
| `20h`, `2 giờ`, `8r`, `5h` | **20R, 2R+, 8R, 5R** | Tỷ lệ Lợi nhuận / Rủi ro (Risk:Reward multiple) |
| `xác áp`, `xét áp`, `xét đam` | **Setup** | Thiết lập giao dịch đủ điều kiện |
| `MacOS`, `MacDash`, `bách test` | **Backtest** | Kiểm chứng dữ liệu quá khứ |
| `MLA`, `mA`, `M ở án`, `em ở A` | **EMA (5, 9, 13, 21, 200)** | Bộ 5 đường trung bình động hàm mũ |
| `Epicure`, `FAG`, `Epic Games`, `Eprego`, `Effigur`, `FVGA`, `AppLog`, `FBG` | **FVG (Fair Value Gap)** | Khoảng trống giá 3 nến (mất cân bằng cung cầu) |
| `cây niến tử trôi giá` / `dâu dưới dài` | **Nến từ chối giá (Rejection Candle)** | Nến rút râu mạnh khi chạm vùng FVG |
| `Elro trên USD` / `khung ngửa gian 3 phút` | **EUR/USD** / **Khung 30 phút (M30)** | Khung thực thi chuẩn là **M30** cho cả EURUSD và XAUUSD |

---

## 2. Bản Transcript Tiếng Việt đã hiệu đính chuẩn

**Mở đầu:**
Tyler, hay còn được biết đến với cái tên **TG Capital** (*Tyler Goedtel*), là một hiện tượng đang nổi lên cực nhanh trong cộng đồng trading quốc tế. Chỉ trong thời gian ngắn, anh ta gây chú ý với những con số gần như khó tin: tỷ lệ lời/lỗ thường từ **20R** trở lên, tỷ lệ thắng được công bố khoảng **90%**. Chính vì những thành tích phi lý ở độ tuổi còn rất trẻ, Tyler cũng trở thành nhân vật gây nhiều tranh cãi. Vậy nên trong video hôm nay, chúng ta sẽ không chỉ bóc tách chi tiết toàn bộ phương pháp, mà còn kiểm chứng trên biểu đồ các thị trường khác nhau — mục tiêu không phải thần thánh hóa bất kỳ ai, mà đánh giá khách quan điểm mạnh, hạn chế và rủi ro tiềm ẩn.

**Tư duy cốt lõi & Chọn lọc thời gian:**
Trước khi đi vào kỹ thuật, cần nói thẳng: hệ thống này không dành cho tất cả mọi người. Nếu bạn thích giao dịch liên tục mỗi ngày, hệ thống này sẽ cực kỳ tra tấn tâm lý. Tyler nói thẳng: mỗi cặp tiền chỉ có khoảng **10 setup chất lượng trong cả năm**. Nhưng nếu mỗi lệnh có lợi nhuận lớn (nhiều R), 10 setup/năm vẫn tạo ra kết quả mà 200 lệnh scalp không làm được.

Bước chọn lọc đầu tiên mà Tyler nhấn mạnh: **Thời gian quan trọng hơn hành động giá.**
- Tyler chỉ giao dịch trong **vùng thanh khoản London**: từ **03:00 sáng đến 06:30 sáng giờ New York** (tương đương **14:00–17:30 giờ VN mùa hè EDT** hoặc **15:00–18:30 giờ VN mùa đông EST**; tức **10:00–13:30 giờ Server MT5 Vantage** cố định quanh năm).
- Đặc biệt, các cây nến M30 mở lúc **02:30, 03:00 và 03:30 sáng New York** (`09:30, 10:00, 10:30` giờ Server) thường là các nến bùng nổ thanh khoản cực mạnh và cực sạch khi dòng tiền ngân hàng/tổ chức châu Âu bắt đầu tham gia. Các vùng **FVG** xuất hiện trong giai đoạn này có xác suất tiếp diễn xu hướng cao hơn hẳn các thời điểm khác.

**Quy trình 4 bước chi tiết (Khung M30):**

1. **Bước 1 — Xác nhận xu hướng bằng bộ EMA và tìm FVG:**
   - Khung thời gian: **M30 (30 phút)**.
   - Bộ chỉ báo: **EMA 5, EMA 9, EMA 13, EMA 21 và EMA 200**.
   - **EMA 200** làm trục xu hướng chính: Giá nằm trên EMA 200 chỉ tìm lệnh **Mua (BUY)**; nằm dưới EMA 200 chỉ tìm lệnh **Bán (SELL)**.
   - **Xếp chồng EMA (EMA Stack):**
     - Lệnh Mua: `EMA 5 > EMA 9 > EMA 13 > EMA 21 > EMA 200` — toàn bộ xếp chồng đúng thứ tự, cùng hướng, không đan chéo nhau.
     - Lệnh Bán: `EMA 5 < EMA 9 < EMA 13 < EMA 21 < EMA 200`.
     - Nếu các đường EMA rối loạn hoặc đan chéo nhau → **bỏ qua ngay** (thị trường cân bằng, dễ quét thanh khoản đảo chiều).
   - **Tìm FVG (Fair Value Gap):** Chờ xuất hiện khoảng trống giá 3 nến thuận xu hướng (với lệnh Mua: vẽ từ đỉnh cao nhất của nến thứ 1 `High[3]` đến đáy thấp nhất của nến thứ 3 `Low[1]`, khi `Low[1] > High[3]`).

2. **Bước 2 — Chờ nến từ chối giá (Rejection Candle) hồi về FVG:**
   - Không mua đuổi khi giá đang tăng mạnh, kiên nhẫn chờ giá quay lại kiểm định vùng FVG.
   - Chờ xuất hiện **1 cây nến từ chối giá rõ ràng** tại vùng FVG: với lệnh Mua là cây nến có **râu dưới dài** (phe bán cố đẩy giá xuống nhưng thất bại, phe mua kéo ngược lên).
   - *Lưu ý:* Râu nến có thể xuyên thủng qua cả vùng FVG, và thân nến có thể xanh hoặc đỏ miễn râu từ chối dài rõ rệt.

3. **Bước 3 — Nến xác nhận KHÔNG đóng cửa vượt đỉnh nến từ chối & Vào lệnh:**
   - Ngay cả khi có nến rút râu đẹp, chưa vào lệnh ngay mà chờ cây nến tiếp theo.
   - **Điều kiện bắt buộc:** Với lệnh Mua, **cây nến tiếp theo KHÔNG ĐƯỢC đóng cửa vượt lên trên đỉnh (`High`) của cây nến từ chối giá vừa xuất hiện** (và không phá thủng đáy nến từ chối). Theo thống kê backtest của Tyler, nếu nến tiếp theo đóng cửa phá vượt đỉnh ngay lập tức thì lực mua dễ cạn kiệt và sớm quay đầu.
   - Khi cây nến xác nhận này đóng cửa thỏa điều kiện (cấu trúc EMA vẫn xếp chồng) → **Vào lệnh ngay tại giá đóng cửa của cây nến xác nhận**.

4. **Bước 4 — Dừng lỗ (SL) & Nuôi lệnh theo cấu trúc EMA (Exit):**
   - **Stop Loss (SL):** Đặt ngay dưới đáy (`Low`) của cây nến từ chối giá (với lệnh Mua; đặt trên `High` của nến từ chối với lệnh Bán) cộng thêm khoảng đệm nhỏ.
   - **Take Profit / Thoát lệnh:** Không dùng TP cố định. Giữ lệnh càng lâu càng tốt miễn là các đường EMA nhanh (`5, 9, 13, 21`) còn giữ trạng thái xếp chồng đúng thứ tự. Khi các đường EMA bắt đầu thu hẹp và **đan xen / mất thứ tự xếp chồng** → báo hiệu động lượng suy yếu → **chủ động đóng lệnh**.

---

## 3. Đối chiếu 2 ví dụ thực tế trong Video trên dữ liệu MT5 của Broker

| Ví dụ trong Video | Dữ liệu nến M30 khớp trên MT5 (`Vantage`) | Kết quả thực tế |
|---|---|---|
| **Ví dụ 1: `EURUSD` ngày `07/04/2026`** | • `15:00 VN` (`11:00 Srv`): EMA `5>9>13>21>200` xếp chồng tăng.<br>• `15:30 VN` (`11:30 Srv`): Tạo Bullish FVG `1.15450 – 1.15590`.<br>• `17:00 VN` (`13:00 Srv`): Nến rút râu chạm đúng đáy FVG `L=1.15450`, `H=1.15630`, `C=1.15550` (`lw=9.2 pip > body=7.8 pip`).<br>• `17:30 VN` (`13:30 Srv`): Nến xác nhận `L=1.15460 > 1.15450`, `C=1.15540 <= 1.15630` → **BUY @ ~1.1554, SL dưới 1.1545**. | Thoát lúc EMA5 cắt xuống EMA9 ngày `08/04/2026 16:30 VN` tại `1.16789` → **Lãi ~8.3R** (khớp con số *"khoảng 8R"* trong video). |
| **Ví dụ 2: `XAUUSD` ngày `28/08/2025`** | • `16:00 VN` (`12:00 Srv`): EMA xếp chồng tăng, tạo Bullish FVG `3397.4 – 3399.3`.<br>• `17:00 VN` (`13:00 Srv`): Nến đỏ thân vừa (`$2.46`), râu dưới dài (`$2.89`) xuyên FVG về `L=3393.8`, `H=3399.2`, `C=3396.7`.<br>• `17:30 VN` (`13:30 Srv`): Nến xanh nhỏ `C=3397.0 <= 3399.2` → **BUY @ ~3397.0, SL ~3393.5**.<br>• `19:30–20:00 VN`: Giá sập mạnh về `3394.3` nhưng chưa chạm SL, EMA vẫn xếp chồng. | Thoát lúc EMA5 cắt xuống EMA9 sáng `29/08/2025 06:30 VN` tại `3414.8` → **Lãi ~5.1R–5.5R** (khớp con số *"khoảng 5R"* trong video). |

> **Hiệu đính 15/11/2026 — đọc kèm bảng trên:**
> - Bảng trên mô tả đúng **kèo trong video** trên dữ liệu MT5. Riêng **XAUUSD 28/08/2025**, EA `v98` trong backtest **không vào kèo này** mà vào một setup **sớm hơn cùng ngày**: FVG tạo bởi 3 nến `09:00 / 09:30 / 10:00` Srv (`3392.58–3393.81`), nến rút râu `10:30` (râu dưới 63%), nến xác nhận `11:00` (`C=3395.63 ≤ H(rej)=3396.10`), vào `3395.83` lúc `11:30` Srv, SL gốc `3393.41` → thoát cùng điểm `3414.78` → `+7.83R`. Kèo `13:30` của video sau đó bị bỏ qua vì `MaxTradesPerDay = 1` và đang có lệnh mở.
> - Vì vậy câu "khớp 100% cả 2 ví dụ" ở các tài liệu cũ chỉ đúng với **EURUSD 07/04/2026**. Với XAUUSD, cả hai setup đều hợp lệ theo luật 4 bước; EA chọn cái xuất hiện trước.
> - Quy ước đánh số nến FVG trong tài liệu này dùng chỉ số MQL (`High[3]` = nến cũ nhất, `Low[1]` = nến mới nhất). Tài liệu bàn giao dùng cách gọi **nến A (cũ) – B – C (mới)**: Bullish FVG khi `Low(C) − High(A) ≥ MinFVG_Pip`.
> - Mọi giờ trong bảng là **giờ MỞ nến M30** theo giờ Server; lệnh khớp lúc nến xác nhận **đóng** (ví dụ xác nhận `13:30` → khớp `14:00` Srv = `07:00` NY).
