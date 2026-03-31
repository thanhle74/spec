# Tham khảo: Linh kiện giao diện (UI Components)

Nguồn: https://developer.adobe.com/commerce/frontend-core/ui-components/

---

## 1. Kiến trúc UI Components

UI Components là một tập hợp các thành phần giao diện được xây dựng trên nền tảng **KnockoutJS** và **RequireJS**, kết nối với Backend thông qua file cấu hình XML.

### Luồng xử lý (Lifecycle)
1. **Server-side:** 
   - PHP gộp các file XML (`view/[area]/ui_component/*.xml`).
   - Chuyển cấu hình XML thành định dạng JSON.
   - Chèn JSON vào trang web qua thẻ `<script type="text/x-magento-init">` gọi đến `Magento_Ui/js/core/app`.
2. **Client-side:**
   - `layout.js` đọc JSON và khởi tạo (instantiate) các component JS tương ứng.
   - KnockoutJS bind dữ liệu từ Component JS vào Template HTML.

---

## 2. Các thuộc tính cơ bản (Basic Attributes)

- `name`: Tên định danh duy nhất của component trong registry.
- `component`: Đường dẫn tới file JavaScript xử lý logic (ví dụ: `Magento_Ui/js/form/form`).
- `template`: Đường dẫn tới file HTML hiển thị (ví dụ: `ui/form/field`).
- `provider`: Tên của `dataSource` component cung cấp dữ liệu.
- `sortOrder`: Thứ tự hiển thị của component.
- `displayArea`: Vùng hiển thị (giống như "hole" trong layout) để các component con nhảy vào.

---

## 3. Các Component chính

- **Listing:** Dùng để hiển thị bảng dữ liệu (Grid) trong Admin.
- **Form:** Dùng để tạo các biểu mẫu nhập liệu phức tạp.
- **DataSource:** Thành phần trung gian kết nối với URL Backend để lấy/lưu dữ liệu.

---

## 4. Quy tắc lập trình UI Components

### JavaScript (KnockoutJS)
- Dùng `uiElement` làm class cơ sở.
- Sử dụng `observables` để dữ liệu tự động cập nhật lên giao diện.
- Luôn sử dụng `this._super()` khi ghi đè (override) các hàm mặc định.

### XML (Semantic UI)
- Tránh viết logic trong XML, chỉ dùng để cấu hình các tham số hiển thị.
- Sử dụng `etc/definition.xml` của module Magento_Ui để tham khảo các giá trị mặc định.

---

## Liên kết
- Quy tắc Magento Patterns: xem [../../magento-patterns.md](../../magento-patterns.md)
- Web API & GraphQL: xem [../web-api.md](../web-api.md)
- Quy tắc chung: xem [../../constitution.md](../../constitution.md)
