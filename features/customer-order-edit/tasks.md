# Feature Tasks - customer-order-edit

> Tham chiếu AC/testcase: `spec.md`. Tham chiếu thiết kế: `plan.md`.

## Task 1 - Scaffold module Secomm_CustomerOrderEdit
- Scope: `app/code/Secomm/CustomerOrderEdit/`
- Acceptance criteria:
  - Module enable được; `sequence` khai báo đúng dependency (`Magento_Sales`, `Magento_Customer`, `Magento_Catalog`, và MSI nếu dùng API reservation).
  - Không có ObjectManager trực tiếp; `declare(strict_types=1);` mọi file PHP mới.
- Verify:
  - `bin/magento module:status Secomm_CustomerOrderEdit` → `Module is enabled`
  - `bin/magento setup:di:compile` (hoặc `setup:upgrade` nếu có schema) không lỗi fatal liên quan module.

## Task 2 - System config: trạng thái cho phép sửa
- Scope: `etc/config.xml`, `etc/adminhtml/system.xml`, `etc/adminhtml/acl.xml`, `Model/Config.php`
- Acceptance criteria:
  - Default chỉ `pending`; merchant có thể mở rộng danh sách status (định dạng rõ ràng trong UI comment hoặc field multiselect/text).
  - Config scope store view hoạt động; đọc config qua class inject được.
- Verify:
  - Admin: Stores > Configuration → tìm section module → lưu thành công.
  - Đổi giá trị và xác nhận `Model/Config` trả đúng trên store đang test.

## Task 3 - Validator + quyền truy cập đơn
- Scope: `Model/*Validator*.php` hoặc service guard
- Acceptance criteria:
  - Từ chối khi customer không sở hữu order.
  - Từ chối khi status không thuộc whitelist config.
  - Từ chối khi có invoice/shipment/credit memo (theo `plan.md`).
- Verify:
  - Manual: đăng nhập user A, mở increment_id của user B → không được edit (404/redirect/message).
  - Manual: đơn `processing` (không trong whitelist mặc định) → không hiện / save từ chối.

## Task 4 - OrderItemEditService (nghiệp vụ cốt lõi)
- Scope: `Model/Service/*.php`, `etc/di.xml`
- Acceptance criteria:
  - Hỗ trợ đổi qty, xóa dòng, thêm SKU (Phase 1: simple + virtual); từ chối type khác với message rõ.
  - Validate salable/stock trước khi persist; không để order ở trạng thái không nhất quán khi lỗi.
  - Sau thay đổi: `collectTotals` + `OrderRepository::save` thành công; totals khớp hiển thị storefront.
  - Đồng bộ MSI reservation với delta thay đổi (không lệch so với qty mới sau khi save).
- Verify:
  - Manual TC-01..TC-03 (`spec.md`) trên đơn `pending`, chỉ simple products.
  - Manual TC-05: thêm qty vượt salable → từ chối, order không đổi.

## Task 5 - Audit: order status history / comment
- Scope: service hoặc method trong `OrderItemEditService`
- Acceptance criteria:
  - Mỗi lần save thành công có ít nhất một bản ghi history/comment CS đọc được trong Admin (order Comments).
  - Nội dung mô tả thay đổi (trước/sau hoặc tóm tắt SKU/qty).
- Verify:
  - Manual: sau edit, Admin > Sales > Order > Comments thấy dòng mới đúng nội dung.

## Task 6 - Storefront: route, ViewModel, layout, template, Save controller
- Scope: `Controller/Order/`, `ViewModel/`, `view/frontend/*`
- Acceptance criteria:
  - Trang order view hiển thị entry "Chỉnh sửa" chỉ khi `canEdit`.
  - Form có form key; POST validate; lỗi hiển thị qua message manager, không leak exception.
  - Sau save redirect về order view hoặc edit page và thấy item/total mới.
- Verify:
  - Manual TC-04, TC-06, TC-07, TC-08.
  - `bin/magento cache:flush` sau deploy layout.

## Task 7 - i18n + regression theo spec
- Scope: `i18n/*.csv`, rà soát toàn luồng
- Acceptance criteria:
  - Chuỗi user-facing qua `__()`; có bản `vi_VN` tối thiểu cho message chính (nếu project dùng locale này).
  - Bảng testcase `spec.md` có cột Pass/Fail + ghi chú môi trường.
- Verify:
  - Chạy lại toàn bộ TC-01..TC-08; ghi kết quả vào báo cáo hoàn thành.

## Completion report format
1. Files changed
2. Lý do thay đổi
3. Verify steps đã chạy
4. Kết quả testcase
5. Xác nhận đạt DoD
