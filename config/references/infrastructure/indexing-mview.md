# Tham khảo: Indexing & MView

Nguồn: https://developer.adobe.com/commerce/php/development/components/indexing/

---

## 1. Indexing là gì?

Indexing là quá trình biến đổi dữ liệu (ví dụ: giá sản phẩm, tồn kho) thành các bảng được tối ưu để truy vấn nhanh hơn. 

**Hai chế độ Index:**
- **Update on Save:** Cập nhật ngay khi dữ liệu gốc thay đổi. (Phù hợp với dữ liệu ít, cần ngay).
- **Update by Schedule:** Theo dõi thay đổi và cập nhật qua Cron job. (Khuyên dùng cho dữ liệu lớn để tránh làm chậm thao tác lưu).

---

## 2. Cách tạo Custom Indexer

### Bước 1: Khai báo trong `etc/indexer.xml`
```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Indexer/etc/indexer.xsd">
    <indexer id="nulltracex_custom_indexer" view_id="nulltracex_custom_view" class="<Vendor>\<Module>\Model\Indexer\CustomIndexer">
        <title translate="true">NullTraceX Custom Indexer</title>
        <description translate="true">Mô tả indexer của bạn</description>
    </indexer>
</config>
```

### Bước 2: Viết class Indexer
Implement `Magento\Framework\Indexer\ActionInterface` và `Magento\Framework\Mview\ActionInterface`.

```php
namespace <Vendor>\<Module>\Model\Indexer;

class CustomIndexer implements \Magento\Framework\Indexer\ActionInterface, \Magento\Framework\Mview\ActionInterface
{
    /**
     * Chạy khi reindex toàn bộ qua CLI: bin/magento indexer:reindex id
     */
    public function executeFull() { ... }

    /**
     * Chạy khi có sự thay đổi hàng loạt (Mass Action trong Admin)
     */
    public function executeList(array $ids) { ... }

    /**
     * Chạy khi một bản ghi duy nhất thay đổi (Plugin/Runtime)
     */
    public function executeRow($id) { ... }
    
    /**
     * QUAN TRỌNG: Method này dùng cho MView (Update by Schedule).
     * Được gọi bởi Cron job khi phát hiện thay đổi trong bảng subscriptions.
     */
    public function execute($ids) { ... }
}
```

---

## 3. MView (Materialized View) - Theo dõi thay đổi

Để dùng chế độ "Update by Schedule", bạn phải khai báo `etc/mview.xml`. Magento sẽ tự động tạo Trigger trong Database để theo dõi bảng gốc.

```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Mview/etc/mview.xsd">
    <view id="nulltracex_custom_view" class="<Vendor>\<Module>\Model\Indexer\CustomIndexer" group="indexer">
        <subscriptions>
            <table name="my_custom_table" entity_column="entity_id" />
        </subscriptions>
    </view>
</config>
```

---

## 4. Lưu ý cho bản 2.4.8

- Từ bản 2.4.8, `customer_grid` đã hỗ trợ cả 2 chế độ (trước đây chỉ hỗ trợ Update on Save).
- Luôn ưu tiên dùng **Update by Schedule** cho các tác vụ tốn tài nguyên để đảm bảo trải nghiệm người dùng không bị gián đoạn.

---

## 5. Tối ưu hóa Indexer (Optimization)

Khi xử lý dữ liệu lớn, cần cấu hình tối ưu để tránh treo server hoặc tràn bộ nhớ.

### A. Cấu hình Batch Size (Chia nhỏ dữ liệu)
Mặc định Magento chia dữ liệu theo lô (Batch). Bạn có thể điều chỉnh lô này trong `di.xml` để tối ưu tốc độ.
- **Price Indexer:** Mặc định 5000. Giảm xuống 1000 có thể giúp Reindex nhanh hơn nếu có nhiều Website/Tier Price.
- **EAV Indexer:** Mặc định 1000.

**Ví dụ cấu hình lại Batch Size cho Price Indexer:**
```xml
<type name="Magento\Catalog\Model\ResourceModel\Product\Indexer\Price\BatchSizeCalculator">
    <arguments>
        <argument name="batchRowsCount" xsi:type="array">
            <item name="configurable" xsi:type="number">1000</item>
        </argument>
    </arguments>
</type>
```

### B. Cơ chế Table Switching (Replica)
Magento dùng bảng `_replica` để thực hiện Reindex ngầm. Khi hoàn tất, nó mới tráo bảng này thành bảng chính.
- **Lợi ích:** Frontend không bị lock, khách vẫn xem web bình thường khi đang reindex.
- **Điều kiện:** Phải để chế độ **Update by Schedule**. Nếu để "Update on Save", cơ chế Replica sẽ không hoạt động tối ưu.

### C. Vấn đề bộ nhớ (Memory)
Nếu Reindex bị lỗi "Memory Limit", hãy kiểm tra Batch size. Chia nhỏ batch size thường giải quyết được vấn đề này mà không cần nâng RAM server quá cao.

---

## 6. Các lệnh CLI hỗ trợ

| Lệnh | Mô tả |
|------|-------|
| `bin/magento indexer:status` | Xem trạng thái các indexer. |
| `bin/magento indexer:reindex [id]` | Thực hiện reindex (Full). |
| `bin/magento indexer:reset [id]` | Reset indexer nếu bị treo (về trạng thái Invalid). |
| `bin/magento indexer:set-mode {realtime|schedule} [id]` | Đổi chế độ index. |

---

## Liên kết

- DI & Codegen: xem [di-codegen.md](./di-codegen.md)
- Quy tắc chung: xem [../constitution.md](../constitution.md)
- Checklist: xem [../checklist.md](../checklist.md)
