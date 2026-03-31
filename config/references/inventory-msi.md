# Tham khảo: Inventory Management (MSI)

Nguồn: https://developer.adobe.com/commerce/php/development/components/web-api/inventory-management

---

## 1. Khái niệm cơ bản (MSI)

- **Source (Nguồn):** Đại diện cho một kho hàng vật lý (Kho Quận 1, Kho Thủ Đức...).
- **Stock (Kho bán hàng):** Một nhóm các Sources được gán cho một Sales Channel (Website).
- **Salable Quantity (Số lượng có thể bán):** Số lượng thực tế khách có thể mua. 
  `Salable Qty = Qty (trong Source) - Reservations`.

---

## 2. Các Service Interface chính

Luôn ưu tiên dùng các Interface mới trong namespace `Magento\Inventory*` thay vì `Magento\CatalogInventory`.

### Kiểm tra tồn kho (Sales)
- `Magento\InventorySalesApi\Api\IsProductSalableInterface`: Kiểm tra sản phẩm còn hàng để bán không.
- `Magento\InventorySalesApi\Api\GetProductSalableQtyInterface`: Lấy số lượng có thể bán (Salable Qty).

### Quản lý kho (Admin/Management)
- `Magento\InventoryApi\Api\SourceRepositoryInterface`: Quản lý danh sách các kho vật lý.
- `Magento\InventoryApi\Api\SourceItemRepositoryInterface`: Quản lý số lượng sản phẩm trong từng kho cụ thể.
- `Magento\InventoryApi\Api\SourceItemsSaveInterface`: Cập nhật số lượng (Qty) cho sản phẩm theo từng kho.

---

## 3. Ví dụ Code (PHP)

**Lấy số lượng có thể bán cho một SKU tại Website hiện tại:**

```php
use Magento\InventorySalesApi\Api\GetProductSalableQtyInterface;
use Magento\InventorySalesApi\Api\StockResolverInterface;
use Magento\Store\Model\StoreManagerInterface;
use Magento\InventorySalesApi\Api\Data\SalesChannelInterface;

// ... Inject vào constructor ...

$stockId = $this->stockResolver->execute(
    SalesChannelInterface::TYPE_WEBSITE,
    $this->storeManager->getWebsite()->getCode()
)->getStockId();

$salableQty = $this->getProductSalableQty->execute($sku, $stockId);
```

---

## 4. Lưu ý về Reservations

Khi một đơn hàng được đặt, Magento **không** trừ ngay vào cột `qty` trong bảng Source. Thay vào đó, nó tạo một bản ghi trong bảng `inventory_reservation`.
- Sau khi đơn hàng được Ship (Shipment created), bản ghi reservation sẽ được xóa và `qty` thực tế mới bị trừ.
- Điều này giúp tránh việc "Lock" bảng dữ liệu kho khi có quá nhiều người mua cùng lúc.

---

## Liên kết

- Service Contracts: xem [service-contracts.md](./service-contracts.md)
- Web API: xem [web-api.md](./web-api.md)
- Quy tắc chung: xem [../constitution.md](../constitution.md)
