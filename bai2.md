# Bài 2: Tối Ưu Prompt Sinh Docstring & Giải Thích Logic

## 1. Phân tích lỗ hổng của prompt thô sơ hiện tại

Prompt gốc của thực tập sinh: *"Giải thích hàm này làm gì và thêm comment vào code giúp tôi: [Đoạn code]"*

Prompt này thất bại vì thiếu hầu hết các thành phần quan trọng:

- **Thiếu Vai trò (Role):** Không gán vai trò chuyên gia cho AI → AI trả lời ở mức chung chung, không dùng thuật ngữ nghiệp vụ (business terminology) của dự án.
- **Thiếu Ngữ cảnh (Context):** AI không biết `a`, `b`, `c` đại diện cho dữ liệu gì → chỉ đoán mò, giải thích kiểu "biến a là tham số đầu tiên" thay vì "a là tổng tiền đơn hàng". Kết quả trả về không khớp với ngôn ngữ nghiệp vụ thực tế.
- **Thiếu Ràng buộc (Constraints):** Không có chỉ thị "KHÔNG được thay đổi logic hiện tại" → AI rất có thể tự ý refactor code, gây ra regression bug — đúng điều mà đội ngũ đang lo sợ.
- **Thiếu Định dạng (Format):** Không yêu cầu sinh Javadoc chuẩn → AI có thể chỉ thêm vài dòng comment thường (`//`) thay vì Javadoc đầy đủ (`/** */` với `@param`, `@return`).
- **Thiếu Mục tiêu (Task) rõ ràng:** "Giải thích hàm này làm gì" quá mơ hồ — không nói rõ cần giải thích ở mức nghiệp vụ hay mức kỹ thuật.

---

## 2. Prompt tối ưu mới

```
Đóng vai Senior Java Developer đang bảo trì hệ thống E-commerce TechShop.

Nhiệm vụ: Dịch ngược logic nghiệp vụ và bổ sung Javadoc, inline comment cho đoạn code tính toán chiết khấu bên dưới. Đặt lại tên biến cho có nghĩa.

Ngữ cảnh: Đây là hệ thống tính chiết khấu đơn hàng. Ý nghĩa các biến trong code gốc:
- `a` = tổng tiền đơn hàng (totalAmount)
- `b` = hạng thành viên: b=1 là Silver, b=2 là Gold
- `c` = trạng thái voucher freeship: true = có voucher, false = không có

Ràng buộc:
- TUYỆT ĐỐI KHÔNG thay đổi logic tính toán hiện tại (để tránh regression bug).
- Đổi tên biến `a` → `totalAmount`, `b` → `memberTier`, `c` → `hasFreeshipVoucher`.
- Viết Javadoc chuẩn cho method, bao gồm @param và @return.
- Thêm inline comment giải thích rõ logic nghiệp vụ tại mỗi khối điều kiện.

Định dạng: Chỉ trả về mã nguồn Java hoàn thiện trong một code block duy nhất. Không giải thích thêm.

Đoạn code gốc:
public double cD(double a, int b, boolean c) {
    double r = a;
    if (b == 1) {
        r = a - (a * 0.05);
    } else if (b == 2) {
        r = a - (a * 0.10);
    }
    if (!c) {
        r += 30000;
    }
    return r;
}
```

---

## 3. Kết quả mã nguồn do AI sinh ra

```java
/**
 * Tính toán tổng tiền cuối cùng của đơn hàng sau khi áp dụng chiết khấu thành viên
 * và phụ phí vận chuyển.
 *
 * <p>Quy tắc nghiệp vụ:</p>
 * <ul>
 *   <li>Hạng Silver (memberTier = 1): Giảm 5% tổng tiền đơn hàng.</li>
 *   <li>Hạng Gold (memberTier = 2): Giảm 10% tổng tiền đơn hàng.</li>
 *   <li>Nếu không có voucher freeship: Cộng thêm 30.000 VNĐ phí vận chuyển.</li>
 * </ul>
 *
 * @param totalAmount        Tổng số tiền của đơn hàng trước khi áp dụng chiết khấu (VNĐ).
 * @param memberTier         Hạng thành viên của khách hàng (1: Silver, 2: Gold, khác: không giảm).
 * @param hasFreeshipVoucher {@code true} nếu khách hàng có voucher miễn phí vận chuyển,
 *                           {@code false} nếu không có (sẽ bị tính phí ship 30.000 VNĐ).
 * @return Tổng số tiền cuối cùng khách hàng cần thanh toán sau khi trừ chiết khấu
 *         và cộng phí vận chuyển (nếu có).
 */
public double calculateDiscount(double totalAmount, int memberTier, boolean hasFreeshipVoucher) {
    // Khởi tạo số tiền cuối cùng bằng tổng tiền gốc
    double finalAmount = totalAmount;

    // Áp dụng mức chiết khấu theo hạng thành viên
    if (memberTier == 1) {
        // Hạng Silver: Giảm 5% trên tổng tiền đơn hàng
        finalAmount = totalAmount - (totalAmount * 0.05);
    } else if (memberTier == 2) {
        // Hạng Gold: Giảm 10% trên tổng tiền đơn hàng
        finalAmount = totalAmount - (totalAmount * 0.10);
    }
    // Nếu memberTier không phải 1 hoặc 2 → Không áp dụng chiết khấu nào

    // Kiểm tra voucher freeship
    if (!hasFreeshipVoucher) {
        // Không có voucher → Cộng thêm phụ phí vận chuyển 30.000 VNĐ
        finalAmount += 30000;
    }

    return finalAmount;
}
```
