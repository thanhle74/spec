# Tham khảo: Quản lý Cache (Cache Management)

Nguồn: https://developer.adobe.com/commerce/php/development/cache/partial/

---

## 1. Khai báo Cache Type mới

Nếu module của bạn có dữ liệu tính toán phức tạp (vd: bảng giá riêng), hãy tạo Cache Type riêng.

**`etc/cache.xml`**:
```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Cache/etc/cache.xsd">
    <type name="my_custom_cache" translate="label,description">
        <label>My Custom Cache</label>
        <description>Dùng để lưu dữ liệu tính toán riêng của MyModule.</description>
    </type>
</config>
```

---

## 2. Thao tác với Cache (PHP)

Luôn dùng `Magento\Framework\App\CacheInterface` để thực hiện.

```php
// Inject CacheInterface và SerializerInterface
public function __construct(CacheInterface $cache, SerializerInterface $serializer) {
    $this->cache = $cache;
    $this->serializer = $serializer;
}

// LƯU DỮ LIỆU
$this->cache->save(
    $this->serializer->serialize($data),
    'unique_cache_id',
    ['my_custom_cache_tag'], // Tag để xóa hàng loạt
    3600 // Thời gian sống (giây)
);

// LẤY DỮ LIỆU
$data = $this->serializer->unserialize($this->cache->load('unique_cache_id'));
```

---

## 3. Tự động xóa Cache (IdentityInterface)

Dùng cho Models hoặc Blocks để tự động xóa cache khi thực thể thay đổi.

```php
use Magento\Framework\DataObject\IdentityInterface;

class MyBlock extends Template implements IdentityInterface {
    public function getIdentities() {
        return [\Vendor\Module\Model\MyModel::CACHE_TAG . '_' . $this->getId()];
    }
}
```

---

## 4. Vô hiệu hóa Cache (Invalidation)

Khi dữ liệu gốc thay đổi, hãy báo cho hệ thống cần làm mới cache.

```php
// Inject TypeListInterface
$this->cacheTypeList->invalidate('my_custom_cache');
```

---

## 5. Phân tách Nội dung Public & Private

Để tối ưu Full Page Cache (FPC/Varnish), phải phân tách dữ liệu:

- **Public Content (Server-side):** Thông tin chung (Mô tả SP, Layout). Lưu trong FPC. KHÔNG dùng Session PHP ở đây.
- **Private Content (Client-side):** Thông tin cá nhân (Giỏ hàng, Wishlist). KHÔNG lưu trong FPC. Load qua AJAX từ trình duyệt.

---

## 6. Xử lý dữ liệu Private (CustomerData)

### Bước 1: Tạo Section Source (PHP)
Class trả về mảng dữ liệu riêng tư.
```php
class MySection implements \Magento\Customer\CustomerData\SectionSourceInterface {
    public function getSectionData() {
        return ['custom_message' => 'Chào mừng bạn quay lại!'];
    }
}
```

### Bước 2: Đăng ký Section (di.xml)
```xml
<type name="Magento\Customer\CustomerData\SectionPoolInterface">
    <arguments>
        <argument name="sectionSourceMap" xsi:type="array">
            <item name="my-custom-data" xsi:type="string">Vendor\Module\CustomerData\MySection</item>
        </argument>
    </arguments>
</type>
```

### Bước 3: Cấu hình Invalidation (sections.xml)
Tự động làm mới dữ liệu khi có hành động POST.

**`etc/frontend/sections.xml`**:
```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Customer:etc/sections.xsd">
    <action name="mymodule/action/post">
        <section name="my-custom-data"/>
    </action>
</config>
```

### Bước 4: Hiển thị ở Frontend (JS/HTML)
Dùng Knockout.js để lấy dữ liệu từ `customerData`.
```javascript
this.myData = customerData.get('my-custom-data');
```

---

## Liên kết

- Maintenance CLI: xem [maintenance-cli.md](./maintenance-cli.md)
- Architectural Patterns: xem [architectural-patterns.md](./architectural-patterns.md)
- Quy tắc chung: xem [../constitution.md](../constitution.md)
