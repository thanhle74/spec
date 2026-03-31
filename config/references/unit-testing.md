# Tham khảo: Unit Testing & ObjectManager Helper

Nguồn: https://developer.adobe.com/commerce/php/development/components/object-manager/helper

---

## 1. Tại sao phải dùng Helper khi Unit Test?

Các class Magento thường có constructor rất dài (nhiều dependency). Việc tạo Mock thủ công cho từng tham số trong file Test là cực kỳ mất thời gian. 

Dùng `Magento\Framework\TestFramework\Unit\Helper\ObjectManager` để tự động hóa việc này.

---

## 2. Cách sử dụng cơ bản

### Khởi tạo Helper trong `setUp()`:

```php
use Magento\Framework\TestFramework\Unit\Helper\ObjectManager as ObjectManagerHelper;

class MyTest extends \PHPUnit\Framework\TestCase
{
    private $objectManagerHelper;
    private $model;

    protected function setUp(): void
    {
        $this->objectManagerHelper = new ObjectManagerHelper($this);
        
        // Tự động mock tất cả dependency và tạo instance của class cần test
        $this->model = $this->objectManagerHelper->getObject(
            \<Vendor>\<Module>\Model\MyModel::class
        );
    }
}
```

---

## 3. Các method quan trọng

### getObject($className, array $arguments = [])
- Tạo instance của `$className`.
- Nếu tham số trong `$arguments` không được truyền, Helper sẽ **tự động tạo Mock** cho tham số đó.
- Dùng khi bạn chỉ muốn Mock 1-2 class quan trọng, còn lại để Helper lo.

```php
$loggerMock = $this->getMockBuilder(\Psr\Log\LoggerInterface::class)->getMock();

$model = $this->objectManagerHelper->getObject(
    \<Vendor>\<Module>\Model\MyModel::class,
    ['logger' => $loggerMock] // Chỉ định Mock cụ thể cho logger, các class khác tự động mock
);
```

### getConstructArguments($className, array $arguments = [])
- Trả về mảng các Mock object của constructor mà không khởi tạo class.
- Hữu ích khi bạn cần test một **Abstract Class**.

---

## 4. Quy tắc viết Test chuẩn (theo convention <Vendor>)

1. **Vị trí file:** `app/code/<Vendor>/<Module>/Test/Unit/`.
2. **Namespace:** Phải bắt đầu bằng `<Vendor>\<Module>\Test\Unit`.
3. **Tên class:** Kết thúc bằng `Test` (ví dụ `MyModelTest.php`).
4. **Mục tiêu:** Unit Test chỉ test logic của class đó, KHÔNG kết nối Database thực tế (luôn Mock Resource Model/Repository).

---

## Liên kết

- DI & Codegen: xem [di-codegen.md](./di-codegen.md)
- Quy tắc chung: xem [../constitution.md](../constitution.md)
- Checklist: xem [../checklist.md](../checklist.md)
