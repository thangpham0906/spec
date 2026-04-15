# Feature Spec - product-shipping-method

## 1) Feature input
- Feature: `Tạo shipping method theo từng product`
- Bối cảnh/vấn đề: `Muốn có thêm 1 phương thức shipping`
- Kết quả mong muốn: `Có 1 phương thức shipping`
- Phạm vi biết chắc: `Module có vendor tên là Secomm`

## 2) Business + scope
- Business problem: `Hiện tại chưa có phương thức vận chuyển đáp ứng nghiệp vụ shipping theo từng sản phẩm, gây hạn chế khi tính và hiển thị lựa chọn vận chuyển cho khách hàng.`
- Business goal: `Bổ sung một shipping method mới để phục vụ nghiệp vụ shipping theo từng product trong module thuộc vendor Secomm.`
- In-scope:
  - `Tạo mới custom shipping carrier trong module Secomm`
  - `Cho phép enable/disable và cấu hình cơ bản của phương thức shipping trong Admin`
  - `Hiển thị được phương thức shipping mới tại checkout khi đủ điều kiện`
- Out-of-scope:
  - `Tối ưu thuật toán tính phí phức tạp theo nhiều rule business chưa được mô tả`
  - `Thay đổi core Magento shipping flow`
  - `Tích hợp external API vận chuyển bên thứ ba`

## 3) Acceptance criteria
1. `AC-01`: Hệ thống có thêm 1 shipping method mới thuộc module vendor Secomm và có thể bật/tắt trong Admin.
2. `AC-02`: Khi shipping method được bật và quote hợp lệ, phương thức mới hiển thị tại bước chọn vận chuyển ở checkout.
3. `AC-03`: Khi shipping method bị tắt hoặc không thỏa điều kiện áp dụng, phương thức không hiển thị ở checkout.

## 4) Xác nhận trước khi code
- Requirement understanding:
  - `Mục tiêu chính là tạo thêm 1 custom shipping method, không phải refactor toàn bộ luồng shipping hiện có.`
  - `Shipping method mới cần hoạt động đúng chuẩn carrier của Magento và cấu hình được trong Admin.`
  - `Ưu tiên triển khai trong module vendor Secomm, tuân thủ DI, không dùng ObjectManager trực tiếp.`
- Risks/assumptions:
  - `Hiện chưa thấy module Secomm nào tồn tại trong app/code; có khả năng cần tạo mới module Secomm để chứa carrier.`
  - `Chưa có rule tính phí chi tiết theo từng product, nên nếu không chốt sớm sẽ dễ implement sai nghiệp vụ.`
  - `Nếu scope "theo từng product" cần đọc product attribute tùy biến, phải xác nhận tên attribute và fallback behavior.`
- Phương án đề xuất:
  - `Option A (khuyến nghị): Tạo mới module Secomm_ShippingPerProduct, dùng custom carrier với mức phí khởi tạo dạng flat để đi vào vận hành nhanh, sau đó mở rộng rule theo product ở phase tiếp theo.`
  - `Option B: Tạo carrier mới nhưng tính phí ngay theo product attribute từ phase đầu (đầy đủ nghiệp vụ hơn nhưng phụ thuộc mạnh vào dữ liệu catalog và nhiều rủi ro edge case).`
- Quyết định đã chốt với user:
  - `Chọn Option B.`
  - `Module scope: Secomm_ShippingPerProduct.`
  - `Pricing rule: Tổng phí shipping = SUM(qty sản phẩm * giá ship của sản phẩm).`
  - `Apply condition: chỉ áp dụng cho một vài sản phẩm (subset).`
- Phạm vi được phép sửa:
  - `app/code/Secomm/ShippingPerProduct/*`
  - `etc/config.xml`, `etc/adminhtml/system.xml`, `etc/di.xml` (nếu cần trong module Secomm)
  - `Model/Carrier/*` và các class hỗ trợ liên quan trong module Secomm
- Điểm cần xác nhận thêm:
  - `Đã xác nhận: tạo mới product attribute code = shipping_per_product.`
  - `Đã xác nhận: subset sản phẩm xác định theo category.`

## 5) Testcase
- Happy path: `TC-01` Bật shipping method trong Admin, checkout thấy method mới và chọn được; `TC-02` Đặt đơn thành công với method mới, order lưu đúng shipping method code/title.
- Edge case: `TC-03` Giỏ hàng có nhiều sản phẩm với cấu hình shipping khác nhau, method vẫn tính và hiển thị theo rule đã định.
- Negative case: `TC-04` Tắt method trong Admin hoặc quote không thỏa điều kiện, checkout không hiển thị method mới.
