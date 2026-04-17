# Feature Spec - custom-order-status

## 1) Feature input
- Feature: `custom-order-status`
- Bối cảnh/vấn đề: `Cần thêm order status "Awaiting Supplier" để team warehouse biết đơn đang chờ nhà cung cấp xác nhận.`
- Kết quả mong muốn: `Admin có thể set status này thủ công, hiển thị đúng trên order grid và email.`
- Phạm vi biết chắc: `Triển khai theo Option B (module custom + data patch), không sửa core Magento`

## 2) Business + scope
- Business problem: `Thiếu trạng thái nghiệp vụ thể hiện đơn đang chờ nhà cung cấp, gây khó cho warehouse khi theo dõi xử lý đơn.`
- Business goal: `Bổ sung trạng thái "Awaiting Supplier" để admin set thủ công và đồng bộ hiển thị nhất quán trên các điểm chạm đã yêu cầu.`
- In-scope:
  - `Tạo custom order status "Awaiting Supplier" theo cơ chế chuẩn Magento (không sửa core).`
  - `Gán status vào state phù hợp để admin có thể chọn khi cập nhật order trong Admin.`
  - `Hiển thị status mới trên order grid Admin với label đúng.`
  - `Status mới xuất hiện đúng trên email có chứa order status label.`
- Out-of-scope:
  - `Tự động chuyển trạng thái theo rule nghiệp vụ nhà cung cấp.`
  - `Thiết kế workflow/phân quyền warehouse mới ngoài ACL hiện có.`
  - `Sửa đổi toàn bộ template email không liên quan tới order status.`

## 3) Acceptance criteria
1. `AC-01`: Hệ thống có status mới code = `awaiting_supplier`, label = `Awaiting Supplier`.
2. `AC-02`: Admin có thể set thủ công status này trên order hợp lệ trong Admin (order view/status history).
3. `AC-03`: Sau khi set, status hiển thị đúng trên Admin order grid.
4. `AC-04`: Trong scope E1 (`order update email` + `order comment notify`), email có render order status phải hiển thị label `Awaiting Supplier` khi order đang ở status này.
5. `AC-05`: Không chỉnh sửa Magento core; triển khai bằng module custom + data patch/config chuẩn.

## 4) Core-first analysis + options
- Kiểm tra core:
  - `Magento core đã hỗ trợ cơ chế thêm order status và map vào order state qua bảng sales_order_status/sales_order_status_state.`
  - `Core cũng hỗ trợ admin đổi status thủ công trong Order History nếu status được map đúng state của order.`

- Option A - `Dùng Admin UI (Stores > Order Status)`:
  - Ưu điểm: `Nhanh, không cần code, thao tác trực tiếp trên môi trường.`
  - Nhược điểm: `Dễ lệch cấu hình giữa các môi trường; khó version control; khó đảm bảo CI/CD repeatable.`

- Option B - `Module custom + Data Patch tạo status/state mapping` (**khuyến nghị**):
  - Ưu điểm: `Có thể version control, triển khai nhất quán giữa dev/staging/prod, rollback/audit tốt hơn.`
  - Nhược điểm: `Tốn effort ban đầu (module/patch/test/verify).`

- Recommendation: `Chọn Option B để đảm bảo tính ổn định vận hành và đồng bộ môi trường.`

## 5) Risks/assumptions cần chốt
- Quyết định đã chốt: `State mapping cho awaiting_supplier = pending.`
- Quyết định đã chốt: `Scope code = Option B (module custom + data patch).`
- Quyết định đã chốt: `Email scope = E1 (order update email + order comment notify).`
- Rủi ro còn mở: `Nếu chỉ tạo status nhưng template email hiện tại không render status, AC-04 sẽ fail dù data đúng.`

## 6) Câu hỏi cần xác nhận trước "OK spec"
1. `Label tiếng Anh đã chốt là "Awaiting Supplier"; chưa yêu cầu đa ngôn ngữ ở pha này.`

## 7) Testcase (draft)
- Happy path:
  - `TC-01`: Admin set status `Awaiting Supplier` thành công trên order đủ điều kiện.
  - `TC-02`: Order grid hiển thị đúng label sau khi update.
  - `TC-03`: Email thuộc scope yêu cầu hiển thị đúng label status.
- Edge:
  - `TC-04`: Order ở state không hỗ trợ status này -> không thể set hoặc bị từ chối đúng chuẩn Magento.
- Negative:
  - `TC-05`: Store chưa deploy patch -> status không tồn tại; sau setup:upgrade thì có và hoạt động đúng.
