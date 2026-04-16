# Feature Plan - product-shipping-method

> Technical design only. Business context → `spec.md`.

## Mục tiêu kỹ thuật
- Custom carrier `Secomm_ShippingPerProduct` theo chuẩn Magento 2.
- Tính phí `SUM(qty × shipping_per_product)` cho subset sản phẩm theo category.
- Cấu hình được trong Admin, hoạt động ổn định tại checkout.

## Thiết kế

1. **Carrier class** — `Model/Carrier/ShippingPerProduct` kế thừa `AbstractCarrier`, implement `CarrierInterface`. Constructor khớp parent 2.4.8: `ScopeConfigInterface`, `ErrorFactory`, `LoggerInterface`.

2. **collectRates** — duyệt quote items, lọc item thuộc subset category (đọc category IDs từ config), đọc `shipping_per_product` attribute, tính `SUM(qty × price)`. Trả `false` nếu không có item hợp lệ hoặc carrier inactive.

3. **Config** — `carriers/secomm_shipping_per_product/*`: fields `active`, `title`, `name`, `category_ids` (multiselect).

4. **Product attribute** — `shipping_per_product` (decimal); tạo qua Data Patch.

## Files dự kiến
```
Secomm/ShippingPerProduct/
├── etc/{module.xml, config.xml, adminhtml/system.xml}
├── Model/Carrier/ShippingPerProduct.php
├── Setup/Patch/Data/AddShippingPerProductAttribute.php
└── registration.php
```

## Risks + rollback
- Risk: Attribute thiếu data → carrier ẩn hoặc phí = 0; thêm guard + log warning.
- Rollback: Tắt carrier trong Admin hoặc `module:disable` → `cache:flush`.
