# Feature Tasks - product-shipping-method

## Task 1 - Tạo module Secomm_ShippingPerProduct và scaffold carrier
- Scope: `app/code/Secomm/ShippingPerProduct/`
- Acceptance criteria:
  - Có module mới `Secomm_ShippingPerProduct` với cấu trúc tối thiểu hợp lệ (`registration.php`, `etc/module.xml`).
  - Có class carrier mới trong `Model/Carrier` với `carrier_code` và method code rõ ràng.
- Verify:
  - Kiểm tra module enable: `bin/magento module:status | rg Secomm`
  - Kiểm tra class/carrier code tồn tại đúng path và namespace.

## Task 2 - Cấu hình Admin cho shipping method
- Scope: `etc/config.xml`, `etc/adminhtml/system.xml`
- Acceptance criteria:
  - Có các config tối thiểu: `active`, `title`, `name`.
  - Có config phục vụ nhận diện subset sản phẩm (nếu chọn cấu hình qua admin).
  - Có thể bật/tắt shipping method trong Admin.
- Verify:
  - Manual: Vào Admin > Stores > Configuration > Sales > Delivery Methods kiểm tra field.
  - Manual: Lưu config thành công, không báo lỗi validate.

## Task 3 - Tính phí theo Qty x giá ship sản phẩm
- Scope: `Model/Carrier/*`, `etc/di.xml` (nếu cần)
- Acceptance criteria:
  - Carrier chỉ áp dụng cho subset sản phẩm theo tiêu chí đã chốt.
  - Tổng phí shipping tính đúng theo công thức: `SUM(qty * giá ship sản phẩm)`.
  - Khi bật method, checkout hiển thị phương thức shipping mới với title/name đúng.
  - Khi tắt method, checkout không hiển thị phương thức shipping mới.
  - Đặt đơn thành công và order lưu đúng shipping method code/title.
- Verify:
  - Manual happy path: tạo quote hợp lệ, vào checkout kiểm tra method và place order.
  - Manual edge: giỏ hàng có nhiều item với qty khác nhau, đối soát tổng phí theo công thức.
  - Manual negative: tắt config `active`, refresh checkout và xác nhận method biến mất.

## Task 4 - Regression check theo testcase trong spec
- Scope: Checkout shipping flow liên quan module `Secomm_ShippingPerProduct`
- Acceptance criteria:
  - TC-01, TC-02, TC-03, TC-04 được chạy và có kết quả Pass/Fail kèm ghi chú.
  - Không ảnh hưởng các shipping method hiện có.
- Verify:
  - Manual: test song song method mới và các method hiện hữu.
  - Command: `bin/magento cache:flush` sau khi thay đổi config/code.

## Completion report format
1. Files changed
2. Lý do thay đổi
3. Verify steps đã chạy
4. Kết quả testcase
5. Xác nhận đạt DoD
