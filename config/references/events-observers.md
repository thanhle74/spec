# Tham khảo: Events & Observers

Nguồn: https://developer.adobe.com/commerce/php/development/components/events-and-observers/

---

## 1. Cơ chế hoạt động

- **Dispatch (Phát):** Một class dùng `EventManager` để phát đi một sự kiện kèm dữ liệu.
- **Subscribe (Đăng ký):** File `etc/events.xml` khai báo các Observer sẽ "lắng nghe" sự kiện đó.
- **Observe (Xử lý):** Class Observer thực hiện logic khi nhận được sự kiện.

---

## 2. Cách tạo Custom Event (Dispatch)

Inject `Magento\Framework\Event\ManagerInterface` và gọi hàm `dispatch`.

```php
public function execute()
{
    // ... logic ...
    $this->eventManager->dispatch('nulltracex_feature_action_after', [
        'data_object' => $object,
        'custom_var' => $value
    ]);
}
```

---

## 3. Cách tạo Observer

### Bước 1: Khai báo trong `etc/events.xml`
(Có thể đặt trong `etc/frontend/` hoặc `etc/adminhtml/` để giới hạn phạm vi).

```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Event/etc/events.xsd">
    <event name="nulltracex_feature_action_after">
        <observer name="nulltracex_observer_unique_name" 
                  instance="<Vendor>\<Module>\Observer\MyObserver" />
    </event>
</config>
```

### Bước 2: Viết class Observer
Nằm trong `Observer/`, implement `ObserverInterface`.

```php
namespace <Vendor>\<Module>\Observer;

use Magento\Framework\Event\Observer;
use Magento\Framework\Event\ObserverInterface;

class MyObserver implements ObserverInterface
{
    public function execute(Observer $observer)
    {
        // Lấy dữ liệu từ event
        $object = $observer->getData('data_object');
        $customVar = $observer->getData('custom_var');

        // Thực hiện logic side-effect (e.g. log, sync api...)
    }
}
```

---

## 4. Best Practices (Quy tắc của <Vendor>)

1. **Ưu tiên Plugin:** Nếu có thể dùng Plugin (Before/After/Around), hãy dùng Plugin thay vì Observer. Plugin có hiệu suất tốt hơn và kiểm soát luồng dữ liệu tốt hơn.
2. **Stateless:** Giống như GraphQL, Observer phải stateless để tương thích với App Server.
3. **Tránh logic nặng:** Tránh viết logic DB nặng hoặc gọi API đồng bộ trong Observer vì nó sẽ làm chậm request chính của người dùng.
4. **Tên Observer:** Phải duy nhất trong toàn hệ thống để tránh bị merge node XML không mong muốn.

---

## 5. Danh sách các Sự kiện phổ biến (Tra cứu nhanh)

Dưới đây là một số sự kiện thường dùng nhất. Xem danh sách đầy đủ tại: [Official Event List](https://developer.adobe.com/commerce/php/development/components/events-and-observers/event-list)

### Giỏ hàng & Thanh toán:
- `checkout_cart_product_add_after`: Sau khi thêm sản phẩm vào giỏ.
- `checkout_cart_save_after`: Sau khi giỏ hàng được lưu.
- `sales_order_place_after`: Sau khi đơn hàng được đặt thành công (Dùng để gửi mail/sync data).
- `checkout_onepage_controller_success_action`: Khi người dùng tới trang success.

### Khách hàng:
- `customer_login`: Khi khách đăng nhập.
- `customer_logout`: Khi khách đăng xuất.
- `customer_save_after`: Khi thông tin khách hàng thay đổi.
- `customer_register_success`: Khi khách đăng ký tài khoản mới.

### Sản phẩm & Danh mục:
- `catalog_product_save_after`: Khi lưu sản phẩm (Dùng để clear cache ngoài...).
- `catalog_category_save_after`: Khi lưu danh mục.
- `catalog_product_import_finish_before`: Trước khi kết thúc quá trình import sản phẩm.

### Hệ thống:
- `controller_action_predispatch`: Trước khi một Controller chạy.
- `controller_action_postdispatch`: Sau khi một Controller chạy xong.
- `layout_generate_blocks_after`: Sau khi các block layout được tạo.

---

## Liên kết

- Plugin: xem [plugins.md](./plugins.md)
- DI & Codegen: xem [di-codegen.md](./di-codegen.md)
- Quy tắc chung: xem [../constitution.md](../constitution.md)
