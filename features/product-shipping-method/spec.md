# Spec - product-shipping-method

## Business
- Vấn đề: Chưa có shipping method tính phí theo từng sản phẩm.
- Mục tiêu: Bổ sung custom carrier tính phí `SUM(qty × giá ship sản phẩm)` cho subset sản phẩm theo category.
- In-scope: Custom carrier trong `Secomm_ShippingPerProduct`; enable/disable + config cơ bản trong Admin; hiển thị đúng ở checkout.
- Out-of-scope: Rule tính phí phức tạp đa điều kiện, thay đổi core shipping flow, tích hợp external API.

## Acceptance criteria
- AC-01: Carrier bật/tắt được trong Admin; khi bật + quote hợp lệ → method hiển thị ở checkout.
- AC-02: Phí tính đúng: `SUM(qty × shipping_per_product attribute)` cho item thuộc subset category.
- AC-03: Khi tắt hoặc không có item thuộc subset → method không hiển thị.

## Decision
- Approach: Custom carrier kế thừa `AbstractCarrier`, module `Secomm_ShippingPerProduct`.
- Business rules:
  - Subset xác định theo category
  - Attribute: `shipping_per_product` (decimal, giá ship per unit)
  - Item không có attribute hợp lệ → bỏ qua (không tính vào phí, không block carrier)
- Scope được phép sửa: `app/code/Secomm/ShippingPerProduct/**`
- Risks:
  - Attribute chưa có data → carrier trả 0 hoặc ẩn; cần fallback rõ ràng trong code
  - Constructor carrier phải khớp `AbstractCarrier` 2.4.8 — verify `setup:di:compile`

## Testcase
- Happy: TC-01 bật method + checkout → hiển thị đúng title/phí; TC-02 place order → lưu đúng method code/title
- Edge: TC-03 giỏ nhiều item qty khác nhau → tổng phí đúng công thức
- Negative: TC-04 tắt method hoặc không có item subset → method không hiển thị
