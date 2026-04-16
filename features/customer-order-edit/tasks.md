# Tasks - customer-order-edit

## Task 1 - Scaffold module
- Scope: `Secomm/CustomerOrderEdit/{registration.php, etc/module.xml, etc/di.xml}`
- AC: Module enable được; sequence đúng; `declare(strict_types=1)` mọi file PHP.
- Verify: `bin/magento module:status Secomm_CustomerOrderEdit` → enabled; `setup:di:compile` không lỗi.

## Task 2 - System config + Model/Config
- Scope: `etc/config.xml`, `etc/adminhtml/{system.xml,acl.xml}`, `Model/Config.php`
- AC: Default `pending`; multiselect store scope; `Model/Config` inject được và trả đúng giá trị.
- Verify: Admin > Stores > Configuration → lưu thành công; đổi giá trị → `Model/Config` trả đúng.

## Task 3 - Guard / Validator
- Scope: `Model/Validator.php`
- AC: Từ chối đúng 4 điều kiện (ownership, status whitelist, có chứng từ, 0 item còn lại).
- Verify: Manual TC-06 (account khác), TC-04 (status ngoài whitelist) → redirect/message đúng.

## Task 4 - OrderItemEditService + MSI reservation
- Scope: `Model/Service/OrderItemEditService.php`, `etc/di.xml`
- AC: Đổi qty/xóa/thêm SKU (simple+virtual); validate salable trước save; `collectTotals` + `OrderRepository::save`; reservation delta đúng sau save.
- Verify: Manual TC-01, TC-02, TC-03, TC-05; kiểm tra `inventory_reservation` table sau edit.

## Task 5 - Audit comment
- Scope: trong `OrderItemEditService` hoặc service riêng
- AC: Mỗi save thành công có order comment CS đọc được, mô tả thay đổi SKU/qty.
- Verify: Admin > Sales > Order > Comments → thấy dòng mới sau edit.

## Task 6 - Storefront UI + Controller
- Scope: `Controller/Order/`, `ViewModel/`, `view/frontend/`
- AC: Link "Chỉnh sửa" chỉ hiện khi `canEdit()`; form key bắt buộc; lỗi qua message manager, không leak exception; redirect đúng sau save.
- Verify: Manual TC-07 (form key sai), TC-08 (SKU không hợp lệ); `cache:flush` sau deploy layout.

## Task 7 - i18n + full regression
- Scope: `i18n/vi_VN.csv`, toàn luồng
- AC: Chuỗi user-facing qua `__()`; TC-01..TC-08 có kết quả Pass/Fail.
- Verify: Chạy lại toàn bộ testcase, ghi kết quả vào completion report.

---

> Rules: `before` plugin → không `unset()` tham số; Logger → `Psr\Log\LoggerInterface`; `system.xml` → `cache:clean config` + verify menu path.

## Completion report
1. Files changed
2. Lý do thay đổi
3. Verify steps + kết quả testcase
4. Xác nhận DoD
