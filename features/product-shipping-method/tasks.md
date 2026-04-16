# Tasks - product-shipping-method

## Task 1 - Scaffold module + carrier skeleton
- Scope: `Secomm/ShippingPerProduct/{registration.php, etc/module.xml, Model/Carrier/ShippingPerProduct.php}`
- AC: Module enable được; carrier class đúng namespace, constructor khớp `AbstractCarrier` 2.4.8.
- Verify: `bin/magento module:status | grep ShippingPerProduct` → enabled; `setup:di:compile` không lỗi.

## Task 2 - Admin config
- Scope: `etc/config.xml`, `etc/adminhtml/system.xml`
- AC: Fields `active`, `title`, `name`, `category_ids` lưu/đọc được; bật/tắt carrier từ Admin.
- Verify: Admin > Stores > Configuration > Sales > Delivery Methods → thấy section, lưu thành công; `cache:clean config`.

## Task 3 - Product attribute + collectRates
- Scope: `Setup/Patch/Data/AddShippingPerProductAttribute.php`, `Model/Carrier/ShippingPerProduct.php`
- AC: Attribute `shipping_per_product` tồn tại; `collectRates` tính đúng `SUM(qty × attribute)`; trả `false` khi không có item subset hoặc carrier inactive.
- Verify: Manual TC-01, TC-02, TC-03, TC-04; đối soát tổng phí thủ công với giỏ nhiều item.

## Task 4 - Regression
- Scope: Checkout shipping flow
- AC: TC-01..TC-04 Pass; các carrier hiện có không bị ảnh hưởng.
- Verify: Test song song method mới và method cũ; `bin/magento cache:flush`.

---

> Rules: Constructor carrier → khớp `AbstractCarrier` 2.4.8; Logger → `Psr\Log\LoggerInterface`; `system.xml` → `cache:clean config` + verify menu path.

## Completion report
1. Files changed
2. Lý do thay đổi
3. Verify steps + kết quả testcase
4. Xác nhận DoD
