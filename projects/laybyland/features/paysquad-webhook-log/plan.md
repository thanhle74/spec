# Plan - paysquad-webhook-log

> Technical design only. Business context → `spec.md`.

## Mục tiêu kỹ thuật
- Tạo DB layer (`WebhookLog` model + ResourceModel + Collection) theo pattern `Transaction` đã có
- Tạo `WebhookLogRepository` để ghi log từ `Processor`
- Sửa `Processor` để wrap handler call trong try/catch và ghi log mọi outcome
- Tạo admin grid read-only với UI component chuẩn Magento

## Blueprint tham khảo
- `examples/integration/magento-module-admin-grid-employee-blueprint.md` — Admin Grid + Repository pattern

## References sử dụng
- Pattern `Transaction` + `ResourceModel/Transaction` trong module hiện tại — bám theo cấu trúc này

## Thiết kế

### 1. DB layer
Bảng `laybyland_paysquad_webhook_log`:

| Column | Type | Note |
|---|---|---|
| `log_id` | int unsigned, PK, auto_increment | |
| `pay_squad_id` | varchar(255), NOT NULL | |
| `event_name` | varchar(100), NOT NULL | paysquad.succeeded / failed / cancelled |
| `order_id` | int unsigned, NULL | nullable — order not found case |
| `status` | varchar(20), NOT NULL | processed / skipped / error |
| `message` | text, NULL | mô tả kết quả hoặc exception message |
| `created_at` | timestamp, NOT NULL, DEFAULT CURRENT_TIMESTAMP | |

Files:
- `etc/db_schema.xml` — thêm table definition
- `etc/db_schema_whitelist.json` — cập nhật

### 2. Model layer
Theo pattern `Model/Transaction.php`:
- `Model/WebhookLog.php` — AbstractModel, `_init(WebhookLogResource::class)`
- `Model/ResourceModel/WebhookLog.php` — AbstractDb, `_init('laybyland_paysquad_webhook_log', 'log_id')`
- `Model/ResourceModel/WebhookLog/Collection.php` — AbstractCollection

### 3. Repository + Factory
- `Model/WebhookLog/Repository.php` — chỉ cần method `save(WebhookLog $log)`, không cần getList/delete vì read-only từ admin
- `Model/WebhookLogFactory.php` — auto-generated bởi DI, không cần tạo tay
- `etc/di.xml` — đăng ký factory nếu cần

### 4. Sửa Processor
`Model/Webhook/Processor.php` — wrap `$handler->handle($payload)` trong try/catch:
- Trước khi dispatch: chuẩn bị `$logStatus`, `$logMessage`, `$orderId = null`
- Idempotency skip → `$logStatus = 'skipped'`, `$logMessage = 'duplicate event'`
- No handler → `$logStatus = 'skipped'`, `$logMessage = 'no handler registered'`
- Handler success → `$logStatus = 'processed'`
- Handler exception → `$logStatus = 'error'`, `$logMessage = $e->getMessage()`, rethrow exception
- Sau mỗi outcome: gọi `$this->webhookLogRepository->save($log)`

### 5. Admin grid (read-only)
- `etc/adminhtml/routes.xml` — thêm route `paysquad_log` nếu chưa có (kiểm tra routes hiện tại)
- `etc/adminhtml/menu.xml` — thêm parent `PaySquad` + item `Webhook Logs` dưới `Laybyland_Base::menu`
- `etc/acl.xml` — thêm resource `Laybyland_PaySquad::webhook_log`
- `Controller/Adminhtml/WebhookLog/Index.php` — extends `\Magento\Backend\App\Action`, render layout
- `view/adminhtml/layout/paysquad_webhooklog_index.xml` — layout handle
- `view/adminhtml/ui_component/paysquad_webhook_log_listing.xml` — grid UI component
- `Ui/DataProvider/WebhookLog/ListingDataProvider.php` — data provider cho grid

## Files dự kiến
```
app/code/Laybyland/PaySquad/
├── etc/
│   ├── db_schema.xml                          (sửa — thêm table)
│   ├── db_schema_whitelist.json               (sửa)
│   ├── acl.xml                                (sửa — thêm resource)
│   └── adminhtml/
│       ├── menu.xml                           (sửa — thêm menu item)
│       └── routes.xml                         (sửa nếu cần)
├── Model/
│   ├── WebhookLog.php                         (mới)
│   ├── ResourceModel/
│   │   ├── WebhookLog.php                     (mới)
│   │   └── WebhookLog/
│   │       └── Collection.php                 (mới)
│   ├── WebhookLog/
│   │   └── Repository.php                     (mới)
│   └── Webhook/
│       └── Processor.php                      (sửa — thêm log + try/catch)
├── Controller/
│   └── Adminhtml/
│       └── WebhookLog/
│           └── Index.php                      (mới)
├── Ui/
│   └── DataProvider/
│       └── WebhookLog/
│           └── ListingDataProvider.php        (mới)
└── view/
    └── adminhtml/
        ├── layout/
        │   └── paysquad_webhooklog_index.xml  (mới)
        └── ui_component/
            └── paysquad_webhook_log_listing.xml (mới)
```

## Risks + rollback
- Risk: `Processor` hiện tại không có try/catch — sửa có thể thay đổi behavior (exception hiện tại propagate lên MQ consumer). Sau khi sửa, exception vẫn được rethrow để MQ consumer xử lý đúng.
- Rollback: revert `Processor.php` + drop table `laybyland_paysquad_webhook_log` + `bin/magento setup:upgrade`
