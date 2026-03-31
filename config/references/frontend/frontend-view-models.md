# Tham khảo: View Models (Frontend Logic)

Nguồn: https://developer.adobe.com/commerce/php/development/components/view-models

---

## 1. Nguyên tắc sử dụng

**KHÔNG** kế thừa Block core (`Magento\Catalog\Block\Product\View`, v.v.) chỉ để thêm một hàm helper. Thay vào đó, hãy sử dụng **View Model**.

## 2. Cách triển khai

### Bước 1: Tạo class View Model
Class phải thực thi `Magento\Framework\View\Element\Block\ArgumentInterface`.

```php
namespace Vendor\Module\ViewModel;

use Magento\Framework\View\Element\Block\ArgumentInterface;

class CustomData implements ArgumentInterface
{
    public function getGreeting(): string
    {
        return "Chào mừng bạn đến với Magento 2.4.8!";
    }
}
```

### Bước 2: Nhúng vào Block qua Layout XML
Sử dụng thẻ `<argument>` với `xsi:type="object"`. Tên argument nên là `view_model`.

```xml
<referenceBlock name="product.info.main">
    <arguments>
        <argument name="view_model" xsi:type="object">Vendor\Module\ViewModel\CustomData</argument>
    </arguments>
</referenceBlock>
```

### Bước 3: Sử dụng trong Template (.phtml)

```php
<?php
/** @var \Vendor\Module\ViewModel\CustomData $viewModel */
$viewModel = $block->getViewModel();
?>

<div class="custom-greeting">
    <?= $escaper->escapeHtml($viewModel->getGreeting()) ?>
</div>
```

---

## 3. Ưu điểm vượt trội

- **Tách biệt logic:** Template chỉ lo hiển thị, View Model lo cung cấp dữ liệu, Block lo về cấu trúc layout.
- **Dependency Injection:** Bạn có thể inject bất kỳ Service/Repository nào vào View Model một cách dễ dàng.
- **Hiệu suất:** Nhẹ hơn rất nhiều so với việc khởi tạo một đối tượng Block đầy đủ.

---

## Liên kết

- Quy tắc chung: xem [../constitution.md](../constitution.md)
- Architectural Patterns: xem [architectural-patterns.md](./architectural-patterns.md)
- Maintenance CLI: xem [maintenance-cli.md](./maintenance-cli.md)
