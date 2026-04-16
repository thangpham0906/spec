# Plan - product-shipping-method

> Technical design only. Business context → `spec.md`.

## Thiết kế

1. **Carrier class** — `Model/Carrier/ShippingPerProduct` kế thừa `AbstractCarrier`, implement `CarrierInterface`. Constructor khớp parent 2.4.8: `ScopeConfigInterface`, `ErrorFactory`, `LoggerInterface`.

2. **collectRates logic** — duyệt quote items, lọc item thuộc subset category (đọc category IDs từ config), đọc `shipping_per_product` attribute, tính `SUM(qty × price)`. Trả `false` nếu không có item hợp lệ hoặc carrier inactive.

3. **Config** — `etc/config.xml` + `etc/adminhtml/system.xml`: fields `active`, `title`, `name`, `category_ids` (multiselect category). Path: `carriers/secomm_shipping_per_product/*`.

4. **Product attribute** — `shipping_per_product` (decimal); tạo qua Data Patch nếu chưa có.

## Files dự kiến
```
Secomm/ShippingPerProduct/
├── etc/{module.xml, config.xml, di.xml, adminhtml/system.xml}
├── Model/Carrier/ShippingPerProduct.php
├── Setup/Patch/Data/AddShippingPerProductAttribute.php
└── registration.php
```

## Risks + rollback
- Risk: Attribute thiếu data → carrier ẩn hoặc phí = 0; thêm guard + log warning.
- Rollback: Tắt carrier trong Admin hoặc `module:disable Secomm_ShippingPerProduct` → `cache:flush`.
