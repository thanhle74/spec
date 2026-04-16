# Status - product-shipping-method

- Stage: `done`
- Last updated: `2026-04-15`

## Last decision
Đã implement module `Secomm_ShippingPerProduct`, tạo attribute `shipping_per_product`, carrier áp dụng subset theo category với công thức `SUM(qty × shipping_per_product)`.

## Blockers
None

## Next action
Chạy verify manual trên môi trường Magento (setup:upgrade, cache flush, checkout TC-01..TC-04) để chốt DoD vận hành.
