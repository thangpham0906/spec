# Feature Spec - customer-order-edit

## 1) Feature input
- Feature: `Chỉnh sửa đơn hàng (customer) tại My Account`
- Bối cảnh/vấn đề: `Muốn chỉnh sửa order ngay tại trang đơn hàng trong My Account, không qua màn hình thao tác đơn trong Admin.`
- Kết quả mong muốn: `Khách đăng nhập có thể thêm/xóa dòng sản phẩm và đổi số lượng trên đơn (khi được phép), lưu lại và thấy order cập nhật trên storefront; mọi thay đổi có dấu vết cho CS.`
- Phạm vi biết chắc: `Chỉ phía customer (storefront); không làm UI chức năng "sửa đơn" trong Admin; module `Secomm_CustomerOrderEdit`.`

## 2) Business + scope
- Business problem: `Khách cần tự điều chỉnh nội dung đơn (item/qty) trước khi đơn được xử lý, thay vì nhờ CS thao tác trong Admin.`
- Business goal: `Cho phép chỉnh sửa line items trên My Account trong giới hạn trạng thái đơn an toàn, đồng bộ totals/tồn kho đúng chuẩn Magento và có lịch sử thay đổi.`
- In-scope:
  - `Trang chi tiết order (My Account): khi đơn đủ điều kiện, hiển thị luồng "Chỉnh sửa đơn" (UI do module triển khai).`
  - `Thao tác trên order items: đổi qty, thêm sản phẩm (dòng mới), xóa dòng sản phẩm; sau lưu, order trên storefront phản ánh đúng items và tổng tiền (subtotal/tax/shipping/total theo kết quả tính lại hợp lệ).`
  - `Ràng buộc trạng thái: mặc định chỉ cho phép khi đơn ở trạng thái `pending`; danh sách trạng thái cho phép có thể cấu hình qua system config của module (Stores > Configuration).`
  - `Ghi nhận lịch sử: mỗi lần lưu thay đổi phải ghi lại được cho CS (tối thiểu order comment theo chuẩn Magento, mô tả thay đổi có thể đọc được).`
  - `Kiểm tra quyền: chỉ customer sở hữu đơn (customer_id khớp) mới được thao tác.`
- Out-of-scope:
  - `Màn hình/flow "edit order" trong Admin (không bổ sung chức năng tương đương cho admin user trên đơn).`
  - `Đổi phương thức thanh toán, đổi phương thức vận chuyển, đổi địa chỉ giao/thanh toán (trừ khi sau này mở rộng spec riêng).`
  - `Re-assign order sang customer khác.`
  - `Chỉnh sửa đơn đã có invoice/shipment/credit memo (trừ khi sau này nới rule — hiện tại giả định chỉ giai đoạn chưa xử lý giao/thanh toán thực tế; rule chi tiết nằm ở plan).`

## 3) Acceptance criteria
1. `AC-01`: Khách chỉ chỉnh được đơn của chính mình; thao tác với increment_id/order thuộc account khác bị từ chối.
2. `AC-02`: Chỉ khi `state`/`status` của đơn thuộc tập trạng thái được cấu hình (mặc định chỉ `pending`) mới hiển thị/kích hoạt chỉnh sửa; đơn khác trạng thái không cho sửa hoặc báo lỗi rõ ràng.
3. `AC-03`: Sau khi lưu thành công, danh sách item và số lượng trên My Account khớp payload hợp lệ; grand total (và các total liên quan sau khi recollect) nhất quán với dữ liệu order.
4. `AC-04`: Thêm/xóa/sửa qty tuân thủ tồn kho/salable (MSI) và rule sản phẩm có thể mua; trường hợp không đủ hàng hoặc SKU không hợp lệ có thông báo rõ, không corrupt order.
5. `AC-05`: Mỗi lần lưu thành công tạo bản ghi lịch sử trên đơn mà CS có thể xem (order comment hoặc cơ chế tương đương đã chốt trong plan), mô tả thay đổi (trước/sau hoặc tóm tắt hợp lệ).
6. `AC-06`: Validate đầu vào và bảo vệ form key; lỗi không lộ stack trace.

## 4) Xác nhận trước khi code
- Requirement understanding:
  - `Phạm vi "edit order" = chỉnh line items: thêm, xóa, đổi qty trên storefront.`
  - `Điều kiện mặc định: chỉ `pending`; merchant có thể cấu hình mở rộng tập trạng thái qua config module.`
  - `Bắt buộc có audit/history cho CS khi mỗi lần lưu thành công.`
  - `Triển khai trong module `Secomm_CustomerOrderEdit` (`app/code/Secomm/CustomerOrderEdit`).`
- Risks/assumptions:
  - `Thay đổi items ảnh hưởng MSI reservation, totals, tax, shipping — cần thiết kế recollect/cập nhật reservation đúng thứ tự; giả định đơn ở `pending` chưa giao/chưa có chứng từ.`
  - `Nếu tích hợp thanh toán online đã authorize, đổi total có thể cần xử lý bổ sung (capture/void) — cần rà trong plan theo phương thức thanh toán thực tế của shop.`
  - `Sản phẩm configurable/bundle có rule add-to-cart phức tạp; plan cần chốt MVP (ví dụ chỉ simple có SKU cố định) hoặc đầy đủ option chọn variant.`
- Phương án kỹ thuật đã chốt:
  - `Option A — Storefront MVC + service nghiệp vụ trong module Secomm_CustomerOrderEdit: layout/view model/controller (POST có form key), logic qua class service; dùng API nội bộ Magento (order repository, các service hỗ trợ quote/order/recollect totals, reservation/MSI theo thiết kế trong plan); không sửa core.`
  - `Loại trừ: Option B (Web API/GraphQL tách riêng cho headless) — không dùng cho feature này.`
- Quyết định đã chốt với user:
  - `Chỉnh sửa: order items — qty, thêm, xóa sản phẩm.`
  - `Trạng thái: mặc định chỉ `pending`; có thể cấu hình (system config).`
  - `Lịch sử thay đổi: có (bắt buộc).`
  - `Module: `Secomm_CustomerOrderEdit`.`
  - `Kiến trúc triển khai: Option A (Storefront MVC + service trong module).`
- Phạm vi được phép sửa:
  - `app/code/Secomm/CustomerOrderEdit/**`
  - `Không sửa vendor/magento core.`
- Điểm còn lại cho `plan.md` (không chặn spec):
  - `Chi tiết thuật toán đồng bộ quote/order, reservation MSI, và product types hỗ trợ trong phase 1.`

## 5) Testcase
- Happy path: `TC-01` Đơn `pending`, khách đổi qty một dòng, lưu — totals và items đúng, có order comment; `TC-02` Khách thêm SKU mới hợp lệ, lưu — dòng mới xuất hiện; `TC-03` Khách xóa một dòng, lưu — dòng biến mất, totals đúng.
- Edge case: `TC-04` Đơn chuyển khỏi trạng thái được phép (hoặc merchant thu hẹp config), UI/API không cho sửa; `TC-05` Thêm qty vượt salable — từ chối với message rõ.
- Negative case: `TC-06` Khách khác cố chỉnh đơn — từ chối; `TC-07` Form key sai / POST không hợp lệ — từ chối; `TC-08` SKU không tồn tại hoặc không bán được — từ chối.
