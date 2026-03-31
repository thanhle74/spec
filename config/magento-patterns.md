# Magento Patterns - Các pattern bắt buộc tuân theo

Tham khảo từ: constitution.md

---

## 1. Plugin (Interceptor)

Ưu tiên dùng plugin thay vì preference.

### Khi nào dùng:
- Cần thay đổi input/output của 1 method có sẵn
- Cần thêm logic trước/sau method gốc

### Quy tắc:
- Đặt trong folder `Plugin/`
- Tên class: `<TargetClass><MôTả>Plugin.php`
  - Ví dụ: `Plugin/Customer/AccountSavePlugin.php`
- Khai báo trong `etc/di.xml`:
```xml
<type name="Magento\Customer\Model\AccountManagement">
    <plugin name="nulltracex_<module>_<mô_tả>"
            type="NullTraceX\<Module>\Plugin\Customer\AccountSavePlugin"
            sortOrder="10" />
</type>
```
- Method convention: `before<Method>`, `after<Method>`, `around<Method>`
- Hạn chế dùng `around` - chỉ khi thật sự cần thiết

---

## 2. Observer (Event)

### Khi nào dùng:
- Cần phản ứng với 1 sự kiện mà không cần thay đổi kết quả
- Logic tách rời, không phụ thuộc vào kết quả method gốc

### Quy tắc:
- Đặt trong folder `Observer/`
- Khai báo trong `etc/events.xml` (hoặc `etc/frontend/events.xml`, `etc/adminhtml/events.xml`)
- Mỗi observer chỉ làm 1 việc

---

## 3. Repository Pattern

### Cấu trúc bắt buộc cho mỗi entity chính:

```
Api/
├── <Entity>RepositoryInterface.php    # get, save, delete, getList
└── Data/
    ├── <Entity>Interface.php          # getter/setter cho entity
    └── <Entity>SearchResultsInterface.php

Model/
├── <Entity>.php                       # Model chính, implement Data Interface
├── <Entity>Repository.php             # Implement Repository Interface
├── ResourceModel/
│   ├── <Entity>.php                   # Resource model
│   └── <Entity>/
│       └── Collection.php             # Collection
```

### Khai báo trong di.xml:
```xml
<preference for="NullTraceX\<Module>\Api\<Entity>RepositoryInterface"
            type="NullTraceX\<Module>\Model\<Entity>Repository" />
<preference for="NullTraceX\<Module>\Api\Data\<Entity>Interface"
            type="NullTraceX\<Module>\Model\<Entity>" />
```

---

## 4. ViewModel

### Khi nào dùng:
- Cần truyền dữ liệu từ PHP sang template
- Thay thế việc viết logic trong Block class

### Quy tắc:
- Đặt trong folder `ViewModel/`
- Implement `Magento\Framework\View\Element\Block\ArgumentInterface`
- Khai báo trong layout XML:
```xml
<block name="my.block" template="NullTraceX_<Module>::my-template.phtml">
    <arguments>
        <argument name="viewModel" xsi:type="object">
            NullTraceX\<Module>\ViewModel\MyViewModel
        </argument>
    </arguments>
</block>
```

---

## 5. Data Patch

### Khi nào dùng:
- Thêm dữ liệu mặc định (attribute, config, CMS block...)
- Migrate dữ liệu khi thay đổi schema

### Quy tắc:
- Đặt trong `Setup/Patch/Data/`
- Implement `Magento\Framework\Setup\Patch\DataPatchInterface`
- Tên class mô tả rõ việc làm: `AddPhoneAttribute.php`, `MigrateOldConfig.php`
- KHÔNG dùng InstallData, UpgradeData

---

## 6. Declarative Schema

### Quy tắc:
- File: `etc/db_schema.xml`
- Chạy `bin/magento setup:db-declaration:generate-whitelist --module-name=NullTraceX_<Module>` sau khi thay đổi schema
- Commit cả `db_schema.xml` và `db_schema_whitelist.json`
- Đặt tên bảng: `nulltracex_<module>_<entity>`
- Primary key: `entity_id` (int, auto increment) hoặc UUID tùy trường hợp

---

## 7. Cron Job

### Khi nào dùng:
- Xử lý định kỳ (sync, cleanup, report...)

### Quy tắc:
- Khai báo trong `etc/crontab.xml`
- Class đặt trong `Cron/`
- Method chính: `execute(): void`
- Nhóm (group): tạo nhóm riêng trong `etc/cron_groups.xml` nếu cần

---

## 8. Console Command

### Quy tắc:
- Đặt trong `Console/Command/`
- Khai báo trong `etc/di.xml`:
```xml
<type name="Magento\Framework\Console\CommandListInterface">
    <arguments>
        <argument name="commands" xsi:type="array">
            <item name="nulltracex_<module>_<tên>" xsi:type="object">
                NullTraceX\<Module>\Console\Command\MyCommand
            </item>
        </argument>
    </arguments>
</type>
```
- Tên command: `nulltracex:<module>:<hành-động>`
  - Ví dụ: `nulltracex:employee:sync`

---

## 9. Config (System Configuration)

### Cấu trúc:
```
etc/
├── adminhtml/
│   └── system.xml          # Khai báo field trong admin
├── config.xml              # Giá trị mặc định
```

### Quy tắc:
- Section: `nulltracex_<module>`
- Path: `nulltracex_<module>/<group>/<field>`
- Dùng Helper hoặc Config model để đọc giá trị, không hardcode path

---

## Liên kết

- Quy tắc chung: xem [constitution.md](./constitution.md)
- Checklist: xem [checklist.md](./checklist.md)
