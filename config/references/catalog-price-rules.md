# Tham khảo: Catalog Price Rules (Khuyến mãi sản phẩm)

Nguồn: https://developer.adobe.com/commerce/php/development/components/catalog-rules

---

## 1. Cơ chế hoạt động

Catalog Price Rule được tính toán **trước** khi sản phẩm được cho vào giỏ hàng. Nó ảnh hưởng trực tiếp đến giá hiển thị trên trang danh sách và chi tiết sản phẩm. 

- **Khác biệt:** Khác với Cart Price Rules (chỉ tính trong giỏ hàng), Catalog Rules cần được **Apply** để sinh dữ liệu vào bảng `catalogrule_product`.

---

## 2. Thêm điều kiện tùy chỉnh (Custom Conditions)

Để thêm một điều kiện mới (ví dụ: "Nếu sản phẩm là hàng dễ vỡ"), bạn cần thực hiện 3 bước:

### Bước 1: Tạo lớp Condition
Class phải kế thừa `Magento\Rule\Model\Condition\AbstractCondition`.

```php
namespace Vendor\Module\Model\Rule\Condition;

class IsFragile extends \Magento\Rule\Model\Condition\AbstractCondition
{
    /**
     * Logic kiểm tra điều kiện
     */
    public function validate(\Magento\Framework\Model\AbstractModel $model): bool
    {
        // $model ở đây là đối tượng Product
        $isFragile = (bool)$model->getData('is_fragile');
        return $this->validateAttribute($isFragile);
    }

    public function getValueElementChooserUrl() { return ''; }
}
```

### Bước 2: Đăng ký vào danh sách Dropdown (Plugin)
Plugin cho `Magento\CatalogRule\Model\Rule\Condition\Combine`.

```php
public function afterGetNewChildSelectOptions($subject, $result)
{
    $customCondition = [
        'value' => \Vendor\Module\Model\Rule\Condition\IsFragile::class,
        'label' => __('Sản phẩm là hàng dễ vỡ'),
    ];

    $result = array_merge_recursive($result, [
        [
            'label' => __('Điều kiện tùy chỉnh'),
            'value' => [$customCondition]
        ]
    ]);

    return $result;
}
```

### Bước 3: Khai báo di.xml
```xml
<type name="Magento\CatalogRule\Model\Rule\Condition\Combine">
    <plugin name="add_custom_condition" type="Vendor\Module\Plugin\CatalogRule\Condition\CombinePlugin" />
</type>
```

---

## 3. Lưu ý về Performance

- **Indexing:** Mỗi khi Catalog Rule được thay đổi hoặc thêm mới, bạn phải chạy `bin/magento indexer:reindex catalogrule_rule` để giá mới được hiển thị.
- **Attributes:** Nếu điều kiện của bạn dựa trên Product Attribute, hãy đảm bảo attribute đó được set `Used in Product Listing = Yes`.

---

## Liên kết

- Attributes (EAV): xem [attributes.md](./attributes.md)
- Plugin: xem [plugins.md](./plugins.md)
- Indexing: xem [indexing-mview.md](./indexing-mview.md)
