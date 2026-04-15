# Feature Plan - product-shipping-method

## Mục tiêu kỹ thuật
- Tạo mới module `Secomm_ShippingPerProduct` và custom shipping carrier theo chuẩn Magento 2.
- Tính phí shipping theo rule đã chốt: `Qty x giá ship sản phẩm` cho các sản phẩm thuộc subset áp dụng.
- Đảm bảo phương thức shipping có thể cấu hình trong Admin và hoạt động ổn định tại checkout.

## Thiết kế giải pháp
1. Tạo skeleton module `app/code/Secomm/ShippingPerProduct` (registration, module.xml, composer nếu cần).
2. Tạo cấu hình carrier trong `config.xml` + `system.xml` với field tối thiểu: `active`, `title`, `name`; không dùng giá flat vì phí tính theo từng sản phẩm.
3. Implement carrier class trong `Model/Carrier` kế thừa `AbstractCarrier` và implement `CarrierInterface`.
4. Trong `collectRates`, duyệt quote items, lọc các sản phẩm thuộc subset áp dụng, đọc giá ship sản phẩm và tính tổng: `sum(qty * product_shipping_price)`.
5. Chỉ trả về method khi quote có ít nhất một sản phẩm thuộc subset và carrier đang active; nếu không thì không hiển thị method.
6. Verify theo testcase: hiển thị đúng ở checkout, ẩn đúng khi không thỏa điều kiện, place order lưu đúng method code/title.

## Phạm vi file/module dự kiến sửa
- `app/code/Secomm/ShippingPerProduct/registration.php`
- `app/code/Secomm/ShippingPerProduct/etc/module.xml`
- `app/code/Secomm/ShippingPerProduct/etc/config.xml`
- `app/code/Secomm/ShippingPerProduct/etc/adminhtml/system.xml`
- `app/code/Secomm/ShippingPerProduct/Model/Carrier/*.php`
- `app/code/Secomm/ShippingPerProduct/etc/di.xml` (nếu cần)
- `app/code/Secomm/ShippingPerProduct/composer.json` (khuyến nghị)

## Rủi ro + rollback
- Rủi ro: Chưa chốt rõ `attribute code` cho giá ship sản phẩm và tiêu chí subset, dễ lệch logic khi tính cước.
- Cách giảm rủi ro: Chốt 2 tham số nghiệp vụ trước khi code, thêm guard/fallback cho item không có giá ship hợp lệ.
- Rollback: Disable carrier qua Admin hoặc revert toàn bộ thay đổi trong `Secomm_ShippingPerProduct`, sau đó `cache:flush`.
