# Tasks - customer-order-edit

> Không lặp business context. Mỗi task: scope + AC + unit test + verify.
> TDD flow bắt buộc với task có business logic: **viết test trước → implement → test pass → code review**.
> Nếu test fail hoặc review còn Critical/High: **DỪNG**.
> - Test fail → fix implementation (max 3 lần) → escalate nếu vẫn fail.
> - Review Critical/High → fix → retest → review lại (max 3 lần).

## Task 1 - Scaffold module *(không cần unit test)*
- Scope: `Secomm/CustomerOrderEdit/{registration.php, etc/module.xml, etc/di.xml}`
- AC: Module enable được; sequence đúng; `declare(strict_types=1)` mọi file PHP.
- Verify: `bin/magento module:status Secomm_CustomerOrderEdit` → enabled; `setup:di:compile` không lỗi.

## Task 2 - System config + Model/Config
- Scope: `etc/config.xml`, `etc/adminhtml/{system.xml,acl.xml}`, `Model/Config.php`
- AC: Default `pending`; multiselect store scope; `Model/Config` trả đúng giá trị.
- Unit test (`Test/Unit/Model/ConfigTest.php`) — viết trước:
  - `testGetAllowedStatusesReturnsDefault` — không có config → trả `['pending']`
  - `testGetAllowedStatusesReturnsConfigValue` — có config → trả đúng array
- Implement: `Model/Config.php`
- Verify: `./vendor/bin/phpunit Test/Unit/Model/ConfigTest.php` → pass; Admin lưu config thành công.
- Code review: skill `magento-code-reviewer` + `checklist.md`

## Task 3 - Validator
- Scope: `Model/Validator.php`
- AC: Từ chối đúng 4 điều kiện (ownership, status whitelist, có chứng từ, 0 item còn lại).
- Unit test (`Test/Unit/Model/ValidatorTest.php`) — viết trước:
  - `testRejectsWrongCustomer`
  - `testRejectsStatusNotInWhitelist`
  - `testRejectsOrderWithInvoice`
  - `testAllowsValidOrder`
- Implement: `Model/Validator.php`
- Verify: `./vendor/bin/phpunit Test/Unit/Model/ValidatorTest.php` → pass; manual TC-06, TC-04.
- Code review: skill `magento-code-reviewer` + `checklist.md`

## Task 4 - OrderItemEditService + MSI reservation
- Scope: `Model/Service/OrderItemEditService.php`, `etc/di.xml`
- AC: Đổi qty/xóa/thêm SKU (simple+virtual); validate salable trước save; `collectTotals` + `OrderRepository::save`; reservation delta đúng.
- Unit test (`Test/Unit/Model/Service/OrderItemEditServiceTest.php`) — viết trước:
  - `testChangeQtyUpdatesTotals`
  - `testAddSkuCreatesNewItem`
  - `testRemoveItemDeletesRow`
  - `testRejectsQtyExceedingSalable`
  - `testRejectsUnsupportedProductType`
- Implement: `Model/Service/OrderItemEditService.php`
- Verify: `./vendor/bin/phpunit Test/Unit/Model/Service/` → pass; manual TC-01, TC-02, TC-03, TC-05.
- Code review: skill `magento-code-reviewer` + `checklist.md`

## Task 5 - Audit comment
- Scope: trong `OrderItemEditService`
- AC: Mỗi save thành công có order comment CS đọc được, mô tả thay đổi SKU/qty.
- Unit test — viết trước:
  - `testSaveCreatesHistoryComment`
  - `testCommentContainsSkuAndQtyChanges`
- Implement: method trong service
- Verify: `./vendor/bin/phpunit` → pass; Admin > Sales > Order > Comments thấy dòng mới.
- Code review: skill `magento-code-reviewer` + `checklist.md`

## Task 6 - Storefront UI + Controller *(không cần unit test)*
- Scope: `Controller/Order/`, `ViewModel/`, `view/frontend/`
- AC: Link "Chỉnh sửa" chỉ hiện khi `canEdit()`; form key bắt buộc; lỗi qua message manager; redirect đúng sau save.
- Verify: manual TC-07, TC-08; `cache:flush` sau deploy layout.

## Task 7 - i18n + full regression *(không cần unit test)*
- Scope: `i18n/vi_VN.csv`, toàn luồng
- AC: Chuỗi user-facing qua `__()`; TC-01..TC-08 có kết quả Pass/Fail.
- Verify: chạy lại toàn bộ testcase, ghi kết quả vào completion report.

---

> Rules: Logger → `Psr\Log\LoggerInterface`; `system.xml` → `cache:clean config` + verify menu path Admin.

## Completion report
1. Files changed
2. Lý do thay đổi
3. Unit test results (pass/fail)
4. Code review results (Critical/High issues nếu có)
5. Verify steps + kết quả testcase
6. Xác nhận DoD
