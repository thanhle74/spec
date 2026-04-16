# Tasks - product-shipping-method

> TDD flow bắt buộc với task có business logic: **viết test trước → implement → test pass → code review**.
> Test fail hoặc review Critical/High → DỪNG → fix (max 3 lần) → escalate.

## Task 1 - Scaffold module + carrier skeleton *(không cần unit test)*
- Scope: `Secomm/ShippingPerProduct/{registration.php, etc/module.xml, Model/Carrier/ShippingPerProduct.php}`
- AC: Module enable được; constructor carrier khớp `AbstractCarrier` 2.4.8.
- Verify: `bin/magento module:status | grep ShippingPerProduct` → enabled; `setup:di:compile` không lỗi.

## Task 2 - Admin config *(không cần unit test)*
- Scope: `etc/config.xml`, `etc/adminhtml/system.xml`
- AC: Fields `active`, `title`, `name`, `category_ids` lưu/đọc được; bật/tắt carrier từ Admin.
- Verify: Admin > Stores > Configuration > Sales > Delivery Methods → thấy section, lưu thành công; `cache:clean config`.

## Task 3 - Product attribute + collectRates
- Scope: `Setup/Patch/Data/AddShippingPerProductAttribute.php`, `Model/Carrier/ShippingPerProduct.php`
- AC: Attribute `shipping_per_product` tồn tại; `collectRates` tính đúng `SUM(qty × attribute)`; trả `false` khi không có item subset hoặc inactive.
- Unit test (`Test/Unit/Model/Carrier/ShippingPerProductTest.php`) — viết trước:
  - `testCollectRatesReturnsMethodWhenSubsetItemExists`
  - `testCollectRatesReturnsFalseWhenNoSubsetItem`
  - `testCollectRatesReturnsFalseWhenInactive`
  - `testFeeCalculationIsCorrect` — `SUM(qty × price)` với nhiều item
- Implement: `Model/Carrier/ShippingPerProduct.php`
- Verify: `./vendor/bin/phpunit Test/Unit/Model/Carrier/` → pass; manual TC-01, TC-02, TC-03, TC-04.
- Code review: skill `magento-code-reviewer` + `checklist.md`

## Task 4 - Regression *(không cần unit test)*
- Scope: Checkout shipping flow
- AC: TC-01..TC-04 Pass; các carrier hiện có không bị ảnh hưởng.
- Verify: Test song song method mới và method cũ; `bin/magento cache:flush`.

---

> Rules: Constructor carrier → khớp `AbstractCarrier` 2.4.8; Logger → `Psr\Log\LoggerInterface`.

## Completion report
1. Files changed
2. Lý do thay đổi
3. Unit test results (pass/fail)
4. Code review results (Critical/High issues nếu có)
5. Verify steps + kết quả testcase
6. Xác nhận DoD
