# Feature Plan - custom-order-status

> Technical design only. Business context -> `spec.md`.

## Mục tiêu kỹ thuật
- Tạo custom order status `awaiting_supplier` bằng module custom + data patch, map với state `pending`.
- Cho phép admin set thủ công status này trong order history theo cơ chế chuẩn Magento.
- Đảm bảo status hiển thị đúng trên Admin order grid và email scope E1 (`order update email` + `order comment notify`) mà không sửa Magento core.

## Blueprint tham khảo
- `Không có blueprint phù hợp trực tiếp trong examples/integration; áp dụng pattern core Data Patch + Sales status mapping.`

## References sử dụng
- `config/references/core/data-schema-patch.md` (Data Patch lifecycle, idempotent setup logic).
- `config/references/core/di-codegen.md` (DI/factory conventions cho patch dependencies).
- `config/references/ops/unit-testing.md` (test strategy cho class có business logic).
- `config/constitution.md` -> `Scope Governance`, `Testing Policy`, `Definition of Done`.

## Thiết kế
1. **Module scaffold**
   - Tạo module mới `Secomm_CustomOrderStatus` (hoặc module Secomm hiện hữu theo xác nhận scope implement), có `registration.php`, `etc/module.xml`, sequence với `Magento_Sales`.

2. **Data patch tạo status + mapping**
   - Dùng data patch insert/upsert status `awaiting_supplier` với label `Awaiting Supplier`.
   - Map status vào state `pending` trong `sales_order_status_state` với cấu hình dùng được ở admin.
   - Thiết kế patch idempotent để chạy lại không tạo duplicate.

3. **Khả năng set thủ công từ Admin**
   - Dựa trên cơ chế native của `Magento_Sales`: khi status map vào state hợp lệ, admin có thể chọn status trong order comment/history.
   - Không custom override UI nếu core đã đáp ứng.

4. **Hiển thị trên Admin order grid**
   - Grid đọc status label từ core sales status mapping; verify sau patch + cache clean.
   - Chỉ bổ sung code nếu phát hiện môi trường có custom grid override làm mất label.

5. **Hiển thị trong email E1**
   - Verify `order update email` và `order comment notify` có render status đúng từ order data.
   - Chỉ override template/transports nếu verify thực tế cho thấy không render đúng label.

## Files dự kiến
- `app/code/Secomm/CustomOrderStatus/registration.php`
- `app/code/Secomm/CustomOrderStatus/etc/module.xml`
- `app/code/Secomm/CustomOrderStatus/Setup/Patch/Data/AddAwaitingSupplierOrderStatus.php`
- `app/code/Secomm/CustomOrderStatus/composer.json` *(optional, nếu convention project yêu cầu)*
- `app/code/Secomm/CustomOrderStatus/Test/Unit/Setup/Patch/Data/AddAwaitingSupplierOrderStatusTest.php`
- `app/code/Secomm/CustomOrderStatus/i18n/vi_VN.csv` *(nếu cần localize label sau này)*

## Risks + rollback
- Risk: Patch ghi dữ liệu thiếu idempotent gây duplicate/status conflict.
- Risk: Một số order state flow thực tế không cho set status mapped `pending` ở mọi tình huống.
- Risk: Email template hiện tại không render status trong một trong hai luồng E1.
- Rollback:
  - Disable module: `bin/magento module:disable Secomm_CustomOrderStatus`
  - Re-run upgrade/cache: `bin/magento setup:upgrade && bin/magento cache:flush`
  - Data rollback (nếu cần) qua data patch revert thủ công/SQL migration có kiểm soát.
