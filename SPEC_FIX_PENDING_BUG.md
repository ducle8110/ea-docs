# OPEN SPEC FIX: Bug EA "Đứng" - Không Đặt Lệnh Mới

**File:** `Gonewiththewind2.mq5`
**Version:** 7.40 → 7.41
**Ngày:** 2024-12-24

---

## 1. MÔ TẢ VẤN ĐỀ

### 1.1 Triệu chứng
EA ngừng vào lệnh mới sau khi một pending order được kích hoạt, mặc dù thị trường vẫn hoạt động bình thường.

### 1.2 Nguyên nhân chính (Bug Logic)

**Kịch bản lỗi:**
```
1. EA đặt BUY_STOP và SELL_STOP
2. Một pending được kích hoạt (VD: BUY_STOP → vị thế BUY)
3. OnTradeTransaction() KHÔNG được gọi đúng cách
   (do network lag, history chưa sync, MT5 bận...)
4. Kết quả: Có vị thế NHƯNG không có pending nào
5. EA bị "kẹt" vô thời hạn
```

**Phân tích logic trong OnTick():**

| Dòng | Hàm kiểm tra | Kết quả | Giải thích |
|------|--------------|---------|------------|
| 566 | `NeedRebuildPending()` | `false` | Vì `buy==0 && sell==0` → return false tại dòng 487-488 |
| 576 | `CountPositions()==0` | `false` | Vì có vị thế đang mở |
| - | **Không có nhánh xử lý** | - | EA không làm gì → "đứng" |

### 1.3 Nguyên nhân phụ

| Nguyên nhân | Dòng | Điều kiện trigger |
|-------------|------|-------------------|
| EMA quá gần nhau | 158 | `MathAbs(EMA10-EMA50) < 0.5 pip` → `AllowStart()=false` |
| Spread quá cao | 561 | `SpreadPip() > 50` → return sớm |
| Đạt MaxPerSide | 251, 255 | `CountByDir() >= 200` → không đặt thêm |
| History chưa sync | 519 | `HistoryDealSelect()` fail |

---

## 2. GIẢI PHÁP

### 2.1 Thêm logic fallback trong OnTick()

**Vị trí:** Sau block "FALLBACK: REBUILD PENDING ORDERS", trước block "ĐẶT LỆNH CHỜ BAN ĐẦU"

**Code thêm mới:**
```mql5
//--- 3.5. FIX BUG: Có vị thế nhưng không có pending ---
// Xử lý trường hợp OnTradeTransaction không được gọi
if(CountPositions() > 0 && !HasPending())
{
    PlacePendingPair();
    return;
}
```

### 2.2 Code OnTick() sau khi sửa

```mql5
void OnTick()
{
    //--- 1. KIỂM TRA TAKE PROFIT (ƯU TIÊN CAO NHẤT) ---
    if (CheckFullClusterTP())
        return;
    CheckPartialClusterTP();
    CheckSingleTP();

    //--- 2. KIỂM TRA SPREAD ---
    if (SpreadPip() > MaxSpreadPip)
        return;

    //--- 3. FALLBACK: REBUILD PENDING ORDERS ---
    if (NeedRebuildPending())
    {
        DeleteAllPending();
        PlacePendingPair();
        return;
    }

    //--- 3.5. FIX BUG: Có vị thế nhưng không có pending ---
    // Trường hợp OnTradeTransaction fail hoặc không được gọi
    // EA có vị thế nhưng không có pending → đặt lại cặp pending
    if (CountPositions() > 0 && !HasPending())
    {
        PlacePendingPair();
        return;
    }

    //--- 4. ĐẶT LỆNH CHỜ BAN ĐẦU ---
    if (CountPositions() == 0 && !HasPending() && AllowStart())
        PlacePendingPair();
}
```

---

## 3. CHI TIẾT THAY ĐỔI

### 3.1 File: Gonewiththewind2.mq5

| Hành động | Vị trí | Mô tả |
|-----------|--------|-------|
| EDIT | Dòng 15 | Đổi version `"7.40"` → `"7.41"` |
| INSERT | Sau dòng 571 | Thêm block fix bug (8 dòng) |

### 3.2 Diff Preview

```diff
 #property strict
-#property version "7.40"
+#property version "7.41"
```

```diff
     if (NeedRebuildPending())
     {
         DeleteAllPending();
         PlacePendingPair();
         return;
     }

+    //--- 3.5. FIX BUG: Có vị thế nhưng không có pending ---
+    // Trường hợp OnTradeTransaction fail hoặc không được gọi
+    // EA có vị thế nhưng không có pending → đặt lại cặp pending
+    if (CountPositions() > 0 && !HasPending())
+    {
+        PlacePendingPair();
+        return;
+    }
+
     //--- 4. ĐẶT LỆNH CHỜ BAN ĐẦU ---
```

---

## 4. KIỂM THỬ

### 4.1 Test Cases

| # | Kịch bản | Kỳ vọng | Trạng thái |
|---|----------|---------|------------|
| 1 | Có 1 vị thế BUY, không có pending | EA đặt cặp BUY_STOP + SELL_STOP mới | [ ] |
| 2 | Có 2 vị thế (BUY+SELL), không có pending | EA đặt cặp pending mới | [ ] |
| 3 | Không có vị thế, không có pending | EA đặt pending nếu AllowStart()=true | [ ] |
| 4 | Có pending + có vị thế | EA không đặt thêm (đúng) | [ ] |
| 5 | Spread > 50 pip | EA không đặt pending (đúng) | [ ] |

### 4.2 Cách test thủ công

1. Chạy EA trên demo account
2. Đợi pending được kích hoạt
3. Manually xóa tất cả pending orders còn lại
4. Quan sát: EA phải đặt cặp pending mới trong tick tiếp theo

---

## 5. RỦI RO VÀ LƯU Ý

### 5.1 Rủi ro
- **Thấp:** Fix này chỉ thêm logic bổ sung, không thay đổi logic hiện tại
- Logic `PlacePendingPair()` đã có kiểm tra `HasPending()` bên trong → không duplicate

### 5.2 Tương thích ngược
- **100% tương thích:** Không thay đổi input parameters
- Không ảnh hưởng đến các vị thế/pending hiện có

---

## 6. CHECKLIST TRIỂN KHAI

- [ ] Backup file `Gonewiththewind2.mq5` gốc
- [ ] Áp dụng thay đổi theo mục 3.1
- [ ] Compile thành công (0 errors, 0 warnings)
- [ ] Test trên demo account
- [ ] Hoàn thành tất cả test cases mục 4.1
- [ ] Deploy lên live account

---

## 7. NGƯỜI PHÊ DUYỆT

| Vai trò | Tên | Ngày | Chữ ký |
|---------|-----|------|--------|
| Developer | | | |
| Reviewer | | | |
| Tester | | | |
