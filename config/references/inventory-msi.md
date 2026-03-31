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

## 4. Reservations (Giữ chỗ tồn kho)

Khi đơn hàng được đặt, Magento tạo bản ghi "Giữ chỗ" (Reservation) thay vì trừ thẳng vào kho vật lý.

- **Dữ liệu:** Lưu trong bảng `inventory_reservation`.
- **Cơ chế:** Append-only (Luôn thêm mới, không bao giờ UPDATE bản ghi cũ).
- **Phép tính:** `Salable Qty = Stock Qty + Sum(Reservations)`.
- **Hoàn tất:** Khi tạo Shipment, Reservation sẽ được xóa/bù trừ và kho vật lý mới bị trừ thực tế.

---

## 5. Source Selection Algorithms (SSA - Thuật toán chọn kho)

Dùng để quyết định lấy hàng từ kho (Source) nào cho Shipment.

- **Priority Algorithm:** Ưu tiên kho có độ ưu tiên cao nhất đã thiết lập.
- **Distance Priority:** Ưu tiên kho có khoảng cách địa lý gần khách hàng nhất (dựa trên Google Maps API hoặc dữ liệu Offline).
- **Custom SSA:** Implement `Magento\InventorySourceSelectionApi\Api\SourceSelectionInterface`.

```php
public function execute(InventoryRequestInterface $inventoryRequest): SourceSelectionResultInterface {
    // Thuật toán tùy biến để chọn Source tối ưu nhất
}
```

---

## Liên kết

- Service Contracts: xem [service-contracts.md](./service-contracts.md)
- Web API: xem [web-api.md](./web-api.md)
- Framework Utilities: xem [framework-utilities.md](./framework-utilities.md)
- Quy tắc chung: xem [../constitution.md](../constitution.md)
