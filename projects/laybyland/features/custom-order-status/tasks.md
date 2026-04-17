# Tasks - custom-order-status

> Không lặp business context. Mỗi task: scope + AC + unit test + verify.
> TDD flow bắt buộc với task có business logic: **viết test trước -> implement -> test pass -> code review**.
> Nếu test fail hoặc review còn Critical/High issue: **DỪNG, không được báo task hoàn thành**.

## Task 1 - Scaffold module *(không có business logic - không cần unit test)*
- Scope: `app/code/Secomm/CustomOrderStatus/{registration.php,etc/module.xml,composer.json}`
- Acceptance criteria:
  - Module `Secomm_CustomOrderStatus` enable được.
  - `module.xml` có sequence tối thiểu với `Magento_Sales`.
  - Không vi phạm rule constitution (`strict_types`, không sửa core).
- Reference đọc trước khi implement: `config/references/core/architectural-patterns.md`.
- Verify:
  - `bin/magento module:enable Secomm_CustomOrderStatus`
  - `bin/magento setup:upgrade`
  - `bin/magento setup:di:compile`

## Task 2 - Data patch tạo status awaiting_supplier + mapping pending
- Scope: `app/code/Secomm/CustomOrderStatus/Setup/Patch/Data/AddAwaitingSupplierOrderStatus.php`
- Acceptance criteria:
  - Tạo status `awaiting_supplier` với label `Awaiting Supplier`.
  - Map status vào state `pending` để admin set thủ công được theo chuẩn core.
  - Patch idempotent, chạy lại không duplicate hoặc lỗi unique key.
- Unit test (`Test/Unit/Setup/Patch/Data/AddAwaitingSupplierOrderStatusTest.php`) - viết trước khi implement:
  - `testApplyCreatesStatusAndPendingMapping`
  - `testApplyIsIdempotentWhenStatusAlreadyExists`
- Implement: data patch + dependency cần thiết (resource/connection/repositories theo thiết kế).
- Reference đọc trước khi implement: `config/references/core/data-schema-patch.md` + `config/references/ops/unit-testing.md`.
- Verify sau implement:
  - `./vendor/bin/phpunit app/code/Secomm/CustomOrderStatus/Test/Unit/Setup/Patch/Data/AddAwaitingSupplierOrderStatusTest.php`
  - `bin/magento setup:upgrade`
  - SQL/manual check bảng `sales_order_status` và `sales_order_status_state` có dữ liệu đúng.
- Code review (sau test pass):
  - Dùng skill `magento-code-reviewer`
  - Đối chiếu `config/checklist.md`

## Task 3 - Verify admin manual set + order grid *(không có business logic - không cần unit test)*
- Scope: Không thêm code nếu core đáp ứng; chỉ bổ sung code khi verify chỉ ra gap cụ thể.
- Acceptance criteria:
  - Admin set thủ công status `Awaiting Supplier` được trên order hợp lệ.
  - Admin order grid hiển thị đúng label sau khi đổi status.
- Reference đọc trước khi implement: `config/references/frontend/ui-components.md` *(chỉ khi phát sinh custom grid code)*.
- Verify:
  - Manual: Sales > Orders > mở order > Order Comments > đổi status `Awaiting Supplier` > Submit
  - Manual: quay lại Sales > Orders grid, xác nhận cột Status hiển thị `Awaiting Supplier`
  - `bin/magento cache:flush`

## Task 4 - Verify email scope E1 *(không có business logic - không cần unit test)*
- Scope: Ưu tiên không code; chỉ thêm override email template nếu verify cho thấy chưa hiển thị status.
- Acceptance criteria:
  - `order update email` hiển thị đúng status label `Awaiting Supplier`.
  - `order comment notify email` hiển thị đúng status label `Awaiting Supplier`.
- Reference đọc trước khi implement: `config/references/ops/configuration-management.md` + `config/checklist.md`.
- Verify:
  - Trigger gửi `order update email` sau khi set status, kiểm tra nội dung email.
  - Trigger gửi `order comment notify email`, kiểm tra nội dung email.
  - Nếu thiếu status trong email: ghi rõ gap và phạm vi override cần thêm trước khi code.

---

## Completion report
1. Files changed
2. Lý do thay đổi
3. Unit test results (pass/fail)
4. Code review results (Critical/High issues nếu có)
5. Verify steps + kết quả testcase
6. Xác nhận DoD
