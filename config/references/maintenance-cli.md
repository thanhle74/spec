# Tham khảo: CLI & Dọn dẹp (Maintenance)

Nguồn: https://developer.adobe.com/commerce/php/development/components/clear-directories

---

## 1. Khi nào cần dọn dẹp cái gì?

| Thay đổi | Thư mục cần xóa / Lệnh cần chạy |
|----------|--------------------------------|
| **Constructor, Plugin, Factory, Proxy** | `rm -rf generated/code/<Vendor>/<Module>` (Bắt buộc để sinh lại code Interceptor/Factory) |
| **`di.xml` hoặc XML cấu hình** | `bin/magento cache:clean config` |
| **Layout (.xml), Template (.phtml)** | `bin/magento cache:clean layout block_html full_page` |
| **JS, CSS, Less, UI HTML** | `rm -rf pub/static/_view_preprocessed/*` và `pub/static/frontend/*` |
| **DB Schema hoặc Data Patch** | `bin/magento setup:upgrade` |
| **Thêm/Đổi Module mới** | `bin/magento module:enable <Module_Name>` và `setup:upgrade` |

---

## 2. Các lệnh CLI quan trọng nhất

### Quản lý Cache
- `bin/magento cache:status`: Xem trạng thái các loại cache.
- `bin/magento cache:clean`: Chỉ xóa các file cache (Nhanh, khuyên dùng).
- `bin/magento cache:flush`: Xóa toàn bộ bộ nhớ đệm (Mạnh hơn, có thể ảnh hưởng hiệu suất tạm thời).

### Biên dịch & Nâng cấp
- `bin/magento setup:di:compile`: Sinh code và kiểm tra lỗi Dependency Injection (Dùng trước khi deploy).
- `bin/magento setup:upgrade`: Cập nhật schema database và xóa mã đã biên dịch cũ.
- `bin/magento setup:static-content:deploy -f`: Ép buộc sinh lại CSS/JS (Thường dùng ở mode Production).

---

## 3. Chế độ (Modes)

Bạn có thể kiểm tra mode hiện tại bằng lệnh: `bin/magento deploy:mode:show`.

- **Developer Mode:** Thích hợp để phát triển. Lỗi hiện trực tiếp trên trình duyệt, static content tự sinh.
- **Production Mode:** Dùng cho môi trường thực tế. Hiệu suất cao nhất, mọi thứ phải được compile sẵn.

---

## 4. Troubleshooting (Xử lý sự cố)

Nếu bạn sửa code mà không thấy thay đổi, hãy thử "Combo thần thánh":
```bash
rm -rf generated/code/* var/cache/* var/page_cache/*
bin/magento cache:flush
```

---

## 3. Tạo lệnh CLI tùy biến (Custom Command)

Dùng để tạo các script import, export hoặc xử lý dữ liệu hàng loạt.

### Bước 1: Tạo class Command
```php
namespace Vendor\Module\Console\Command;

use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;

class SyncData extends Command {
    protected function configure() {
        $this->setName('vendor:sync:data')
             ->setDescription('Đồng bộ dữ liệu từ API bên thứ ba');
        parent::configure();
    }

    protected function execute(InputInterface $input, OutputInterface $output): int {
        $output->writeln('<info>Bắt đầu đồng bộ...</info>');
        // Logic xử lý...
        $output->writeln('<info>Hoàn tất!</info>');
        return 0; // Success
    }
}
```

### Bước 2: Đăng ký trong di.xml
```xml
<type name="Magento\Framework\Console\CommandListInterface">
    <arguments>
        <argument name="commands" xsi:type="array">
            <item name="vendor_sync_data" xsi:type="object">Vendor\Module\Console\Command\SyncData</item>
        </argument>
    </arguments>
</type>
```

---

## 4. Chế độ Vận hành (Application Modes)

Magento có 3 chế độ vận hành chính, ảnh hưởng trực tiếp tới hiệu năng và bảo mật.

- **Developer:** Hiện lỗi trực tiếp, không cache file tĩnh. Dùng khi phát triển.
- **Production:** Tốc độ tối đa, ẩn lỗi, bắt buộc phải biên dịch trước (Compile/Deploy).
- **Default:** Chế độ mặc định của hệ thống (không tối ưu).

**Lệnh chuyển chế độ**:
- `bin/magento deploy:mode:show`: Xem chế độ hiện tại.
- `bin/magento deploy:mode:set developer`: Chuyển sang Developer.
- `bin/magento deploy:mode:set production`: Chuyển sang Production (mất vài phút).

---

## 5. Chế độ Bảo trì (Maintenance)

Dùng khi nâng cấp hệ thống để tránh khách hàng gặp lỗi.

- `bin/magento maintenance:enable`: Bật bảo trì (chặn tất cả).
- `bin/magento maintenance:enable --ip=123.123.123.123`: Chỉ cho phép IP này truy cập để sửa lỗi.
- `bin/magento maintenance:disable`: Tắt bảo trì.
- `bin/magento maintenance:status`: Kiểm tra trạng thái.

---

## Liên kết

- DI & Codegen: xem [di-codegen.md](./di-codegen.md)
- Declarative Schema: xem [declarative-schema.md](./declarative-schema.md)
- Quy tắc chung: xem [../constitution.md](../constitution.md)
