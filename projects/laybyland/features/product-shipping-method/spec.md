# Feature Spec - product-shipping-method

## 1) Feature input
- Feature: `Tạo shipping method theo từng product`
- Bối cảnh/vấn đề: `Chưa có phương thức vận chuyển tính phí theo từng sản phẩm.`
- Kết quả mong muốn: `Custom carrier tính phí SUM(qty × giá ship sản phẩm) cho subset sản phẩm theo category.`
- Phạm vi biết chắc: `Module Secomm_ShippingPerProduct`

## 2) Business + scope
- Business problem: `Chưa có shipping method đáp ứng nghiệp vụ tính phí theo từng sản phẩm.`
- Business goal: `Bổ sung custom carrier tính phí đúng rule, enable/disable được trong Admin.`
- In-scope: Custom carrier trong `Secomm_ShippingPerProduct`; config Admin; hiển thị đúng ở checkout.
- Out-of-scope: Rule tính phí phức tạp đa điều kiện, thay đổi core shipping flow, external API.

## 3) Acceptance criteria
1. `AC-01`: Carrier bật/tắt được trong Admin; khi bật + quote hợp lệ → method hiển thị ở checkout.
2. `AC-02`: Phí tính đúng: `SUM(qty × shipping_per_product)` cho item thuộc subset category.
3. `AC-03`: Khi tắt hoặc không có item thuộc subset → method không hiển thị.

## 4) Xác nhận trước khi code
- Requirement understanding:
  - `Tạo custom carrier, không refactor luồng shipping hiện có.`
  - `Subset xác định theo category; attribute: shipping_per_product (decimal).`
  - `Item không có attribute hợp lệ → bỏ qua, không block carrier.`
- Risks/assumptions:
  - `Attribute chưa có data → carrier ẩn hoặc phí = 0; cần fallback rõ ràng.`
  - `Constructor carrier phải khớp AbstractCarrier 2.4.8 → verify setup:di:compile.`
- Approach đã chốt: `Option B — tính phí theo product attribute từ phase đầu.`
- Scope được phép sửa: `app/code/Secomm/ShippingPerProduct/**`

## 5) Testcase
- Happy: TC-01 bật method + checkout → hiển thị đúng; TC-02 place order → lưu đúng method code/title
- Edge: TC-03 giỏ nhiều item qty khác nhau → tổng phí đúng công thức
- Negative: TC-04 tắt method hoặc không có item subset → method không hiển thị
