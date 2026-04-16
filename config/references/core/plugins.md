# Tham khảo: Plugin (Interceptor)

Nguồn: https://developer.adobe.com/commerce/php/development/components/plugins

---

## Tổng quan

Plugin (interceptor) là class cho phép thay đổi hành vi của method public bằng cách chèn code **trước** (before), **sau** (after), hoặc **bao quanh** (around) method đó.

Ưu điểm: Giảm xung đột giữa các extension cùng thay đổi 1 class. Plugin chạy tuần tự theo `sortOrder`, không xung đột lẫn nhau.

---

## Giới hạn - KHÔNG dùng Plugin được cho:

- Method `final`
- Class `final`
- Method không public (private, protected)
- Method static
- `__construct` và `__destruct`
- Virtual types
- Object khởi tạo trước khi `Magento\Framework\Interception` bootstrap
- Object implement `Magento\Framework\ObjectManager\NoninterceptableInterface`

---

## Khai báo Plugin trong di.xml

```xml
<type name="{ClassHoặcInterfaceCầnPlugin}">
    <plugin name="{tênPlugin}"
            type="{ClassPlugin}"
            sortOrder="10"
            disabled="false" />
</type>
```

### Các thuộc tính:

| Thuộc tính    | Bắt buộc | Mô tả                                                                |
| ------------- | -------- | -------------------------------------------------------------------- |
| `type name`   | Có       | Class hoặc interface mà plugin theo dõi                              |
| `plugin name` | Có       | Tên plugin (dùng để merge config, phải duy nhất)                     |
| `plugin type` | Có       | Class plugin. Convention: `\Vendor\Module\Plugin\<ClassName>`        |
| `sortOrder`   | Không    | Thứ tự chạy (số nhỏ chạy trước). Mặc định tuỳ theo module load order |
| `disabled`    | Không    | `true` để tắt plugin. Mặc định `false`                               |

### Ví dụ theo convention <Vendor>:

```xml
<type name="Magento\Customer\Model\AccountManagement">
    <plugin name="<vendor>_customer_validate_phone"
            type="<Vendor>\Customer\Plugin\ValidatePhonePlugin"
            sortOrder="10" />
</type>
```

---

## Quy tắc đặt tên method

Viết hoa chữ cái đầu của method gốc, thêm tiền tố `before`, `after`, hoặc `around`.

| Method gốc   | Before             | After             | Around             |
| ------------ | ------------------ | ----------------- | ------------------ |
| `setName`    | `beforeSetName`    | `afterSetName`    | `aroundSetName`    |
| `save`       | `beforeSave`       | `afterSave`       | `aroundSave`       |
| `_construct` | `before_construct` | `after_construct` | `around_construct` |

Ngoại lệ: Nếu method gốc bắt đầu bằng `_` (underscore), không viết hoa.

---

## 1. Before Method

Chạy **trước** method gốc. Dùng để thay đổi tham số đầu vào.

### Quy tắc:

- Tham số đầu tiên: `$subject` (object của class gốc)
- Các tham số sau: giống method gốc
- **Trả về mảng** các tham số đã sửa, hoặc `null` nếu không sửa

### Cảnh báo — không `unset` tham số rồi `return`

- Trong `before*`, nếu bạn `return [$a, $b, ...]` thì **không** được `unset($a)`, `unset($b)`, v.v. Magento sẽ dùng mảng trả về làm tham số mới; biến đã `unset` gây `Undefined variable` (thường lộ qua GraphQL dưới dạng warning/error).
- Đừng dùng `unset($subject, ...)` cho mọi tham số chỉ để im lặng cảnh báo “unused”: bỏ `unset` hoặc dùng công cụ static analysis/annotation đúng cách; `$subject` không dùng vẫn **phải** nằm trong signature.

### Ví dụ - Sửa tham số trước khi gọi method gốc:

```php
<?php

declare(strict_types=1);

namespace <Vendor>\Catalog\Plugin;

use Magento\Catalog\Model\Product;

class ProductNameNormalizer
{
    public function beforeSetName(Product $subject, string $name): array
    {
        // Trim khoảng trắng trước khi lưu tên sản phẩm
        return [trim($name)];
    }
}
```

### Ví dụ - Không sửa tham số (chỉ log):

```php
public function beforeSave(Product $subject): void
{
    $this->logger->info('Sắp lưu sản phẩm: ' . $subject->getSku());
    // Không return = không thay đổi tham số
}
```

---

## 2. After Method

Chạy **sau** method gốc. Dùng để thay đổi kết quả trả về.

### Quy tắc:

- Tham số đầu tiên: `$subject`
- Tham số thứ hai: `$result` (kết quả của method gốc)
- Các tham số sau (tuỳ chọn): các tham số của method gốc
- **Bắt buộc trả về** `$result` (đã sửa hoặc giữ nguyên)
- Nếu method gốc return `void`, `$result` sẽ là `null`

### Ví dụ - Sửa kết quả trả về:

```php
<?php

declare(strict_types=1);

namespace <Vendor>\Catalog\Plugin;

use Magento\Catalog\Model\Product;

class ProductNameFormatter
{
    public function afterGetName(Product $subject, ?string $result): string
    {
        // Thêm prefix vào tên sản phẩm
        return '[NTX] ' . $result;
    }
}
```

### Ví dụ - Truy cập tham số method gốc:

```php
public function afterLogin(
    \Magento\Backend\Model\Auth $subject,
    $result,
    string $username
): void {
    // $username là tham số của method login() gốc
    $this->logger->info('User ' . $username . ' đã đăng nhập.');
}
```

### Lưu ý quan trọng:

- After method không cần khai báo hết tham số của method gốc
- Chỉ khai báo tham số nào cần dùng
- Nhưng phải khai báo **tất cả tham số đứng trước** tham số cần dùng

---

## 3. Around Method

Chạy **bao quanh** method gốc. Có thể thay thế hoàn toàn method gốc.

### Quy tắc:

- Tham số đầu tiên: `$subject`
- Tham số thứ hai: `$proceed` (callable, gọi method gốc hoặc plugin tiếp theo)
- Các tham số sau: giống method gốc
- **Phải gọi `$proceed()`** để method gốc chạy (nếu muốn)
- Phải trả về kết quả

### Ví dụ - Thêm logic trước/sau:

```php
<?php

declare(strict_types=1);

namespace <Vendor>\Catalog\Plugin;

use Magento\Catalog\Model\Product;

class ProductSaveLogger
{
    public function aroundSave(Product $subject, callable $proceed): Product
    {
        // Code TRƯỚC method gốc
        $this->logger->info('Bắt đầu lưu sản phẩm: ' . $subject->getSku());

        // Gọi method gốc (hoặc plugin tiếp theo)
        $result = $proceed();

        // Code SAU method gốc
        $this->logger->info('Đã lưu sản phẩm: ' . $subject->getSku());

        return $result;
    }
}
```

### Ví dụ - Forward tham số bằng variadics:

```php
public function aroundSave(Product $subject, callable $proceed, ...$args): Product
{
    // Không cần biết tham số cụ thể
    // Forward toàn bộ tham số cho method gốc
    return $proceed(...$args);
}
```

### Ví dụ - Method gốc có tham số nullable:

```php
// Method gốc
public function save(SomeType $obj = null) { ... }

// Plugin PHẢI giữ nguyên default value
public function aroundSave(
    MyUtility $subject,
    callable $proceed,
    SomeType $obj = null  // PHẢI có = null, nếu thiếu sẽ fatal error
): mixed {
    return $proceed($obj);
}
```

### CẢNH BÁO với Around Method:

- **Hạn chế dùng** - chỉ khi before/after không đủ
- Nếu **không gọi `$proceed()`**, method gốc và các plugin sau sẽ **KHÔNG chạy**
- Phải match đúng type hint và default value của method gốc
- Nếu không cần sửa tham số, dùng variadics `...$args` để forward

---

## Thứ tự thực thi Plugin (sortOrder)

### Quy tắc chung:

1. Before: chạy từ sortOrder **thấp đến cao**
2. Around: bọc bên ngoài, sortOrder thấp bọc ngoài cùng
3. After: chạy từ sortOrder **thấp đến cao**

### Ví dụ 3 plugin cùng sortOrder 10, 20, 30:

**Khi chỉ có before/after (không có around):**

```
PluginA::before  (sortOrder 10)
PluginB::before  (sortOrder 20)
PluginC::before  (sortOrder 30)
  → Method gốc chạy
PluginA::after   (sortOrder 10)
PluginB::after   (sortOrder 20)
PluginC::after   (sortOrder 30)
```

**Khi PluginB có around (với callable):**

```
PluginA::before  (sortOrder 10)
PluginB::before  (sortOrder 20)
PluginB::around - phần 1
    PluginC::before  (sortOrder 30)
      → Method gốc chạy
    PluginC::after   (sortOrder 30)
PluginB::around - phần 2
PluginA::after   (sortOrder 10)
PluginB::after   (sortOrder 20)
```

**Khi PluginB có around KHÔNG gọi callable:**

```
PluginA::before  (sortOrder 10)
PluginB::before  (sortOrder 20)
PluginB::around  → DỪNG, method gốc và PluginC KHÔNG chạy
PluginA::after   (sortOrder 10)
PluginB::after   (sortOrder 20)
```

---

## Kế thừa cấu hình (Configuration Inheritance)

- Class con kế thừa plugin từ class cha
- Plugin khai báo ở scope `global` (`etc/di.xml`) áp dụng cho mọi area
- Có thể override/disable plugin theo area: `etc/frontend/di.xml`, `etc/adminhtml/di.xml`

---

## Tắt (Disable) Plugin

Tắt plugin của module khác bằng cách khai báo lại với `disabled="true"`:

```xml
<type name="Magento\Checkout\Block\Checkout\LayoutProcessor">
    <plugin name="ProcessPaymentConfiguration" disabled="true"/>
</type>
```

Lưu ý: Dùng đúng format namespace (có hoặc không có `\` đầu, nhưng phải khớp giữa khai báo và disable).

---

## Cấu trúc file Plugin theo convention <Vendor>

```
app/code/<Vendor>/<Module>/
├── Plugin/
│   ├── <TargetModule>/
│   │   └── <TargetClass><MôTả>Plugin.php
│   └── ...
├── etc/
│   ├── di.xml                    # Plugin scope global
│   ├── frontend/
│   │   └── di.xml                # Plugin chỉ cho frontend
│   └── adminhtml/
│       └── di.xml                # Plugin chỉ cho admin
```

Ví dụ:

```
Plugin/
├── Customer/
│   └── ValidatePhonePlugin.php
├── Catalog/
│   └── ProductNameNormalizerPlugin.php
```

---

## Liên kết

- Data/Schema Patch: xem [data-schema-patch.md](./data-schema-patch.md)
- Declarative Schema: xem [declarative-schema.md](./declarative-schema.md)
- Quy tắc chung: xem [../constitution.md](../constitution.md)
- Các pattern: xem [../magento-patterns.md](../magento-patterns.md)
