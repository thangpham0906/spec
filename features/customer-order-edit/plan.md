# Plan - customer-order-edit

> Technical design only. Business context → `spec.md`.

## Thiết kế

1. **Config** — `etc/adminhtml/system.xml` + `etc/config.xml`: path `sales/secomm_customer_order_edit/allowed_statuses`, default `pending`, multiselect, store scope. `Model/Config` đọc + normalize.

2. **Guard** — kiểm tra tại mọi entry point: customer sở hữu đơn, status trong whitelist, không có invoice/shipment/creditmemo, còn ít nhất 1 item sau thao tác.

3. **OrderItemEditService** — nhận struct (item_id+qty / sku+qty / item_id xóa), validate toàn bộ trước khi ghi. Phase 1: simple + virtual only; configurable/bundle từ chối rõ. Sau cấu trúc item ổn: `collectTotals()` → `OrderRepository::save`. MSI: bù reservation delta qua `InventorySales` API (không để lệch sau save).

4. **Audit** — sau save: `OrderStatusHistoryFactory` tạo comment `is_customer_notified=false`, `is_visible_on_front=false`, nội dung tóm tắt SKU/qty trước-sau.

5. **Storefront UI** — layout inject block vào `sales_order_view` khi `ViewModel::canEdit()` true. Controller Save: `HttpPostActionInterface`, validate form key, delegate service, redirect + message manager, catch `LocalizedException`.

6. **Module sequence** — `Magento_Sales`, `Magento_Customer`, `Magento_Catalog`, `Magento_InventorySales`.

## Files dự kiến
```
Secomm/CustomerOrderEdit/
├── etc/{module.xml, config.xml, di.xml, frontend/routes.xml, adminhtml/{system.xml,acl.xml}}
├── Model/{Config.php, Service/OrderItemEditService.php, Validator.php}
├── Controller/Order/{Edit.php, Save.php}
├── ViewModel/OrderEdit.php
├── view/frontend/{layout/*.xml, templates/*.phtml}
├── i18n/vi_VN.csv
└── registration.php
```

## Risks + rollback
- Risk: Reservation lệch sau edit hoặc total sai nếu `collectTotals` bị skip → dùng transaction, validate trước save, integration test MSI thủ công.
- Risk: Payment đã authorize + total thay đổi → ghi vào QA checklist, có thể giới hạn thêm bằng config payment method whitelist ở phase sau.
- Rollback: `module:disable Secomm_CustomerOrderEdit` → `setup:upgrade` → `cache:flush`.
