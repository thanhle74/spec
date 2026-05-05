# Tasks - paysquad-webhook-log

> Không lặp business context. Mỗi task: scope + AC + unit test + verify.
> TDD flow bắt buộc với task có business logic: **viết test trước → implement → test pass → code review**.
> Nếu test fail hoặc review còn Critical/High issue: **DỪNG, không được báo task hoàn thành**.

## - [x] Task 1 - DB schema + Model layer [enabler]

- Depends-on: `—`
- Context: Tạo bảng `laybyland_paysquad_webhook_log` và Model/ResourceModel/Collection theo pattern `Transaction` đã có trong module.
- Scope:
  - `app/code/Laybyland/PaySquad/etc/db_schema.xml`
  - `app/code/Laybyland/PaySquad/etc/db_schema_whitelist.json`
  - `app/code/Laybyland/PaySquad/Model/WebhookLog.php`
  - `app/code/Laybyland/PaySquad/Model/ResourceModel/WebhookLog.php`
  - `app/code/Laybyland/PaySquad/Model/ResourceModel/WebhookLog/Collection.php`
- Acceptance criteria:
  - Bảng `laybyland_paysquad_webhook_log` tồn tại sau `setup:upgrade` với đúng columns theo plan
  - `WebhookLog` model khởi tạo được qua factory
- Verify:
  - `bin/magento setup:upgrade`
  - `bin/magento setup:db-status` → up to date
  - Kiểm tra bảng tồn tại trong DB với đúng columns

---

## - [x] Task 2 - WebhookLog Repository [feature]

- Depends-on: `Task 1`
- Context: Repository đơn giản chỉ cần method `save()` — dùng để ghi log từ `Processor`. Bám pattern `Model/Transaction/Repository.php` đã có.
- Scope:
  - `app/code/Laybyland/PaySquad/Model/WebhookLog/Repository.php`
  - `app/code/Laybyland/PaySquad/etc/di.xml` (nếu cần đăng ký)
- Acceptance criteria:
  - `Repository::save(WebhookLog $log)` lưu bản ghi vào DB thành công
  - Inject được qua constructor DI
- Unit test (`Test/Unit/Model/WebhookLog/RepositoryTest.php`) — viết trước khi implement:
  - `save()` gọi `ResourceModel::save()` với đúng object
  - `save()` propagate exception nếu ResourceModel throw
- Implement: `Model/WebhookLog/Repository.php`
- Reference đọc trước khi implement: bám `Model/Transaction/Repository.php` trong module
- Verify sau implement:
  - `./vendor/bin/phpunit app/code/Laybyland/PaySquad/Test/Unit/Model/WebhookLog/RepositoryTest.php` → all pass
  - `bin/magento setup:di:compile` → no errors
- Code review (sau test pass):
  - Dùng skill `magento-code-reviewer`
  - Đối chiếu `config/checklist.md`

---

## - [x] Task 3 - Sửa Processor để ghi webhook log [feature]

- Depends-on: `Task 2`
- Context: `Processor::process()` hiện tại không có try/catch bao handler call và không ghi DB log. Cần inject `WebhookLogRepository` + `WebhookLogFactory`, wrap handler call trong try/catch, ghi log cho mọi outcome: `processed`, `skipped`, `error`. Exception vẫn phải được rethrow sau khi ghi log.
- Scope:
  - `app/code/Laybyland/PaySquad/Model/Webhook/Processor.php`
- Acceptance criteria:
  - Mọi outcome (processed/skipped/error) đều tạo 1 bản ghi trong `laybyland_paysquad_webhook_log`
  - `order_id` được lấy từ payload nếu có, null nếu không
  - Exception từ handler vẫn được rethrow sau khi ghi log
  - Idempotency skip → status=`skipped`, message=`duplicate event skipped`
  - No handler → status=`skipped`, message=`no handler registered`
  - Handler success → status=`processed`
  - Handler exception → status=`error`, message=exception message
- Unit test (`Test/Unit/Model/Webhook/ProcessorTest.php`) — viết trước khi implement:
  - Handler success → `WebhookLogRepository::save()` được gọi với status=`processed`
  - Idempotency skip → `save()` được gọi với status=`skipped`
  - No handler → `save()` được gọi với status=`skipped`
  - Handler throw exception → `save()` được gọi với status=`error`, exception được rethrow
- Implement: `Model/Webhook/Processor.php`
- Reference đọc trước khi implement: `Model/Webhook/Processor.php` hiện tại
- Verify sau implement:
  - `./vendor/bin/phpunit app/code/Laybyland/PaySquad/Test/Unit/Model/Webhook/ProcessorTest.php` → all pass
- Code review (sau test pass):
  - Dùng skill `magento-code-reviewer`
  - Đối chiếu `config/checklist.md`

---

## - [x] Task 4 - ACL + Menu [enabler]

- Depends-on: `Task 1`
- Context: Thêm ACL resource `Laybyland_PaySquad::webhook_log` và menu item `Webhook Logs` dưới `Laybyland > PaySquad`.
- Scope:
  - `app/code/Laybyland/PaySquad/etc/acl.xml`
  - `app/code/Laybyland/PaySquad/etc/adminhtml/menu.xml`
- Acceptance criteria:
  - Menu group `PaySquad` hiển thị dưới `Laybyland` (parent: `Laybyland_Base::menu`)
  - Menu item `Webhook Logs` hiển thị dưới `Laybyland > PaySquad`
  - ACL resource `Laybyland_PaySquad::webhook_log` xuất hiện trong Roles > Resources
- Verify:
  - `bin/magento cache:clean config`
  - Mở admin → `Laybyland > PaySquad > Webhook Logs` → thấy menu item
  - Mở `System > Permissions > User Roles` → thấy resource trong tree

---

## - [x] Task 5 - Admin Grid UI [enabler]

- Depends-on: `Task 4`
- Context: Read-only admin grid hiển thị `laybyland_paysquad_webhook_log`. Không cần form edit, không cần mass action delete. Columns: ID, PaySquad ID, Event Name, Order ID, Status, Message, Created At. Filter + sort trên tất cả columns.
- Scope:
  - `app/code/Laybyland/PaySquad/Controller/Adminhtml/WebhookLog/Index.php`
  - `app/code/Laybyland/PaySquad/Ui/DataProvider/WebhookLog/ListingDataProvider.php`
  - `app/code/Laybyland/PaySquad/view/adminhtml/layout/paysquad_webhooklog_index.xml`
  - `app/code/Laybyland/PaySquad/view/adminhtml/ui_component/paysquad_webhook_log_listing.xml`
  - `app/code/Laybyland/PaySquad/etc/adminhtml/routes.xml` (sửa nếu cần thêm route)
- Acceptance criteria:
  - Grid load được tại URL `/admin/paysquad_log/webhooklog/index`
  - Hiển thị đủ 7 columns
  - Filter theo `event_name`, `status`, `pay_squad_id`, `order_id`, `created_at` hoạt động
  - Sort theo tất cả columns hoạt động
  - Truy cập không có quyền → redirect về 403/dashboard
- Verify:
  - `bin/magento setup:di:compile && bin/magento cache:flush`
  - Mở grid trong admin → load thành công
  - Thử filter từng column → kết quả đúng
  - Thử sort từng column → kết quả đúng
  - Dùng role không có quyền `webhook_log` → bị redirect

---

## Completion report
1. Files changed
2. Lý do thay đổi
3. Unit test results (pass/fail)
4. Code review results (Critical/High issues nếu có)
5. Verify steps + kết quả testcase
6. Xác nhận DoD
