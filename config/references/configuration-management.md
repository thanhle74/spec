# Tham khảo: Quản lý cấu hình (Configuration Management)

Nguồn: https://developer.adobe.com/commerce/php/development/configuration/

---

## 1. Cấu trúc lưu trữ cấu hình

Magento sử dụng 2 file chính để quản lý cấu hình dưới dạng mã (Config as Code):

- **`app/etc/config.php`**: Chứa danh sách module và các cấu hình hệ thống **không** nhạy cảm. (Nên commit vào Git).
- **`app/etc/env.php`**: Chứa cấu hình nhạy cảm (DB, Redis, API Keys) và cấu hình đặc thù môi trường. (KHÔNG commit vào Git).

---

## 2. Bảo mật dữ liệu (Sensitive Data)

Nếu một path cấu hình chứa mật khẩu hoặc API Key, bạn **phải** đăng ký nó là "nhạy cảm" thông qua `di.xml` của module.

```xml
<type name="Magento\Config\Model\Config\TypePool">
    <arguments>
        <argument name="sensitive" xsi:type="array">
            <!-- Đánh dấu path này là nhạy cảm -->
            <item name="payment/my_gateway/api_key" xsi:type="string">1</item>
        </argument>
        <argument name="environment" xsi:type="array">
            <!-- Đánh dấu path này là đặc thù môi trường (ví dụ URL/Port) -->
            <item name="payment/my_gateway/sandbox_url" xsi:type="string">1</item>
        </argument>
    </arguments>
</type>
```

---

## 3. Lệnh CLI quan trọng

- `bin/magento app:config:dump`: Xuất toàn bộ cấu hình từ Database ra file `config.php` và `env.php`. (Dùng để đồng bộ giữa các môi trường).
- `bin/magento app:config:import`: Nhập cấu hình từ file vào hệ thống (thường chạy tự động trong `setup:upgrade`).
- `bin/magento config:set <path> <value>`: Thiết lập cấu hình trực tiếp từ CLI.

---

## 4. Biến môi trường (Environment Variables)

Bạn có thể ghi đè bất kỳ giá trị nào bằng biến môi trường theo quy tắc:
`CONFIG__<SCOPE>__<SECTION>__<GROUP>__<FIELD>`

- **Ví dụ:** `export CONFIG__DEFAULT__WEB__SECURE__BASE_URL=https://example.com/`

---

## 5. Lưu ý cho AI

1. **Bảo mật:** Khi AI viết script cài đặt module, không bao giờ hardcode API Key. Hãy hướng dẫn người dùng dùng biến môi trường hoặc `env.php`.
2. **Deployment:** Luôn nhắc người dùng chạy `app:config:dump` sau khi thay đổi cấu hình quan trọng trong Admin để đồng bộ cho các máy chuyên gia (Dev) khác.

---

## Liên kết

- Maintenance CLI: xem [maintenance-cli.md](./maintenance-cli.md)
- Architectural Patterns: xem [architectural-patterns.md](./architectural-patterns.md)
- Quy tắc chung: xem [../constitution.md](../constitution.md)
