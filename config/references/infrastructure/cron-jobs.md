# Tham khảo: Cron Jobs (Tự động hóa tác vụ)

Nguồn: https://experienceleague.adobe.com/en/docs/commerce-operations/configuration-guide/crons/custom-cron

---

## 1. Cấu hình crontab (crontab.xml)

Định nghĩa các tác vụ chạy ngầm.

**`etc/crontab.xml`**:
```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Cron:etc/crontab.xsd">
    <group id="default">
        <job name="my_custom_cron_job" instance="Vendor\Module\Cron\MyTask" method="execute">
            <!-- Cách 1: Hardcode lịch chạy -->
            <schedule>0 * * * *</schedule>
            
            <!-- Cách 2: Lấy từ cấu hình Admin (Khuyên dùng) -->
            <!-- <config_path>my_module/cron/schedule</config_path> -->
        </job>
    </group>
</config>
```

---

## 2. Tạo Group riêng (cron_groups.xml)

Dùng khi tác vụ nặng, cần cấu hình thời gian chờ hoặc chạy tiến trình riêng.

**`etc/cron_groups.xml`**:
```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Cron:etc/cron_groups.xsd">
    <group id="custom_group">
        <schedule_generate_every>1</schedule_generate_every>
        <schedule_ahead_for>4</schedule_ahead_for>
        <schedule_lifetime>2</schedule_lifetime>
        <history_cleanup_every>10</history_cleanup_every>
        <history_success_lifetime>60</history_success_lifetime>
        <history_failure_lifetime>600</history_failure_lifetime>
        <use_separate_process>1</use_separate_process>
    </group>
</config>
```

---

## 3. Class thực thi (PHP)

```php
namespace Vendor\Module\Cron;

class MyTask {
    public function execute() {
        // Logic xử lý tự động ở đây
    }
}
```

---

## 4. Lệnh CLI hỗ trợ
- `bin/magento cron:run`: Chạy mọi tác vụ cron.
- `bin/magento cron:run --group="default"`: Chỉ chạy cho group cụ thể.

---

## Liên kết
- Maintenance CLI: xem [maintenance-cli.md](./maintenance-cli.md)
- Quy tắc chung: xem [../constitution.md](../constitution.md)
