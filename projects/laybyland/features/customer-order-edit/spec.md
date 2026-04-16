# Feature Spec - customer-order-edit

## 1) Feature input
- Feature: `Chỉnh sửa đơn hàng (customer) tại My Account`
- Bối cảnh/vấn đề: `Muốn chỉnh sửa order ngay tại trang đơn hàng trong My Account, không qua màn hình thao tác đơn trong Admin.`
- Kết quả mong muốn: `Khách đăng nhập có thể thêm/xóa dòng sản phẩm và đổi số lượng trên đơn (khi được phép), lưu lại và thấy order cập nhật trên storefront; mọi thay đổi có dấu vết cho CS.`
- Phạm vi biết chắc: `Chỉ phía customer (storefront); không làm UI chức năng "sửa đơn" trong Admin; module `Secomm_CustomerOrderEdit`.`

## 2) Business + scope
- Business problem: `Khách cần tự điều chỉnh nội dung đơn (item/qty) trước khi đơn được xử lý, thay vì nhờ CS thao tác trong Admin.`
- Business goal: `Cho phép chỉnh sửa line items trên My Account trong giới hạn trạng thái đơn an toàn, đồng bộ totals/tồn kho đúng chuẩn Magento và có lịch sử thay đổi.`
- In-scope:
  - `Trang chi tiết order (My Account): khi đơn đủ điều kiện, hiển thị luồng "Chỉnh sửa đơn" (UI do module triển khai).`
  - `Thao tác trên order items: đổi qty, thêm sản phẩm (dòng mới), xóa dòng sản phẩm; sau lưu, order trên storefront phản ánh đúng items và tổng tiền (subtotal/tax/shipping/total theo kết quả tính lại hợp lệ).`
  - `Ràng buộc trạng thái: mặc định chỉ cho phép khi đơn ở trạng thái `pending`; danh sách trạng thái cho phép có thể cấu hình qua system config của module (Stores > Configuration).`
  - `Ghi nhận lịch sử: mỗi lần lưu thay đổi phải ghi lại được cho CS (tối thiểu order comment theo chuẩn Magento, mô tả thay đổi có thể đọc được).`
  - `Kiểm tra quyền: chỉ customer sở hữu đơn (customer_id khớp) mới được thao tác.`
- Out-of-scope:
  - `Màn hình/flow "edit order" trong Admin.`
  - `Đổi phương thức thanh toán, vận chuyển, địa chỉ.`
  - `Re-assign order sang customer khác.`
  - `Chỉnh sửa đơn đã có invoice/shipment/credit memo.`

## 3) Acceptance criteria
1. `AC-01`: Khách chỉ chỉnh được đơn của chính mình; account khác bị từ chối.
2. `AC-02`: Chỉ đơn có status trong whitelist config (default: `pending`) mới cho sửa.
3. `AC-03`: Sau lưu, items + totals trên storefront nhất quán với payload.
4. `AC-04`: Validate salable qty (MSI); SKU không hợp lệ hoặc vượt stock bị từ chối với message rõ.
5. `AC-05`: Mỗi lần lưu thành công tạo order comment CS đọc được trong Admin.
6. `AC-06`: Form key bắt buộc; lỗi không lộ stack trace.

## 4) Xác nhận trước khi code
- Requirement understanding:
  - `Phạm vi "edit order" = chỉnh line items: thêm, xóa, đổi qty trên storefront.`
  - `Điều kiện mặc định: chỉ `pending`; merchant có thể cấu hình mở rộng tập trạng thái qua config module.`
  - `Bắt buộc có audit/history cho CS khi mỗi lần lưu thành công.`
- Risks/assumptions:
  - `Thay đổi items ảnh hưởng MSI reservation, totals, tax, shipping.`
  - `Thanh toán đã authorize + total thay đổi → cần rà payment method thực tế.`
  - `Phase 1: chỉ simple + virtual product.`
- Approach đã chốt: `Storefront MVC + service trong Secomm_CustomerOrderEdit; không dùng headless API.`
- Scope được phép sửa: `app/code/Secomm/CustomerOrderEdit/**`

## 5) Testcase
- Happy path: TC-01 đổi qty → totals đúng + comment; TC-02 thêm SKU mới → dòng mới; TC-03 xóa dòng → biến mất + totals đúng
- Edge: TC-04 đơn rời whitelist status → không cho sửa; TC-05 qty vượt salable → từ chối
- Negative: TC-06 account khác → từ chối; TC-07 form key sai → từ chối; TC-08 SKU không hợp lệ → từ chối
