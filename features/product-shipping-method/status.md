# Feature Status - product-shipping-method

- Current stage: `done`
- Owner: `AI Agent`
- Last updated: `2026-04-15`

## Last decision
- User đã xác nhận `OK implement`; đã implement module `Secomm_ShippingPerProduct`, tạo attribute `shipping_per_product`, và shipping carrier áp dụng subset theo category với công thức `SUM(qty * shipping_per_product)`.

## Blockers
- None

## Next best action
- Chạy verify manual trên môi trường Magento (setup:upgrade, cache flush, checkout testcases TC-01..TC-04) để chốt DoD vận hành.
