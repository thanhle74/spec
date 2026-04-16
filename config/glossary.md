# Glossary - Thuật ngữ domain Magento

> Mục đích: tránh AI suy đoán sai khi gặp các khái niệm dễ nhầm lẫn trong Magento.
> Khi gặp thuật ngữ dưới đây, AI phải dùng đúng định nghĩa này, không tự suy diễn.

---

## Scope / Store hierarchy

| Thuật ngữ | Định nghĩa |
|---|---|
| `global` | Phạm vi toàn hệ thống, không phân biệt website/store |
| `website` | Đơn vị cao nhất trong store hierarchy (`store_website` table) |
| `store` (store group) | Nhóm store thuộc 1 website, có 1 root category (`store_group` table) |
| `store_view` | Ngôn ngữ/giao diện cụ thể thuộc 1 store group (`store` table) |

> Lưu ý: trong code Magento, `\Magento\Store\Model\Store` đại diện cho `store_view`, không phải `store group`.

---

## Catalog

| Thuật ngữ | Định nghĩa |
|---|---|
| `product` | Entity sản phẩm (`catalog_product_entity`) |
| `category` | Entity danh mục (`catalog_category_entity`) |
| `attribute` | Thuộc tính EAV của product/category |
| `attribute_set` | Nhóm attributes gán cho product type |
| `simple product` | Sản phẩm đơn, không có variant |
| `configurable product` | Sản phẩm có variant (size, color...) — parent không có stock trực tiếp |
| `child product` | Simple product thuộc configurable — đây là entity có stock thực |
| `virtual product` | Sản phẩm không vận chuyển (dịch vụ, download) |
| `bundle product` | Sản phẩm gói nhiều item tùy chọn |

---

## Inventory (MSI)

| Thuật ngữ | Định nghĩa |
|---|---|
| `source` | Kho vật lý (`inventory_source`) |
| `stock` | Nhóm sources gán cho sales channel (`inventory_stock`) |
| `source_item` | Số lượng tồn kho tại 1 source cụ thể (`inventory_source_item`) |
| `stock_item` | Legacy table (`cataloginventory_stock_item`) — chỉ dùng cho single-source hoặc backward compat |
| `salable_quantity` | Số lượng có thể bán = tổng source_item trừ reservations |
| `reservation` | Lượng hàng tạm giữ khi có order chưa ship (`inventory_reservation`) |

> Không nhầm `source_item` với `stock_item`. MSI dùng `source_item`; `stock_item` là legacy.

---

## Cart / Checkout / Order

| Thuật ngữ | Định nghĩa |
|---|---|
| `quote` | Giỏ hàng chưa đặt (`quote` table) — có thể là guest hoặc customer |
| `quote_item` | Dòng sản phẩm trong giỏ hàng |
| `order` | Đơn hàng đã đặt (`sales_order` table) |
| `order_item` | Dòng sản phẩm trong đơn hàng |
| `invoice` | Chứng từ thanh toán của order |
| `shipment` | Chứng từ vận chuyển của order |
| `creditmemo` | Phiếu hoàn tiền |
| `address` | Địa chỉ giao hàng/thanh toán — tồn tại ở cả quote và order |

---

## Pricing

| Thuật ngữ | Định nghĩa |
|---|---|
| `regular_price` | Giá gốc của sản phẩm |
| `special_price` | Giá khuyến mãi trực tiếp trên product |
| `tier_price` | Giá theo số lượng mua |
| `catalog_price_rule` | Rule giảm giá áp dụng trên catalog (không cần coupon) |
| `cart_price_rule` | Rule giảm giá áp dụng trên giỏ hàng (có thể cần coupon) |
| `final_price` | Giá cuối sau khi áp dụng tất cả rule |

---

## DI / Architecture

| Thuật ngữ | Định nghĩa |
|---|---|
| `preference` | Override hoàn toàn 1 class bằng class khác trong `di.xml` |
| `plugin` | Interceptor — thêm logic before/after/around method mà không override class |
| `virtual type` | Instance ảo của 1 class với constructor args khác nhau, không tạo file PHP mới |
| `proxy` | Lazy-load dependency — dùng khi dependency nặng và không phải lúc nào cũng cần |
| `factory` | Class tạo instance của object (thường auto-generated: `<ClassName>Factory`) |
| `repository` | Service contract để CRUD entity, thay thế trực tiếp gọi Model/ResourceModel |
| `data object` | DTO truyền dữ liệu giữa các layer, implement `Api/Data/<Entity>Interface` |

---

## API

| Thuật ngữ | Định nghĩa |
|---|---|
| `REST API` | HTTP API theo `webapi.xml`, trả JSON/XML |
| `GraphQL` | Query language API, schema định nghĩa trong `schema.graphqls` |
| `service contract` | Interface trong `Api/` — đây là public API của module |
| `ACL` | Access Control List — phân quyền resource trong `acl.xml` |

---

## Liên kết

- Quy tắc chung: [constitution.md](./constitution.md)
- Patterns: [magento-patterns.md](./magento-patterns.md)
- Domain glossary dự án: [project-glossary.md](./project-glossary.md)
