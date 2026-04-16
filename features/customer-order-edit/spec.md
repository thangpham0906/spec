# Spec - customer-order-edit

## Business
- Vấn đề: Khách cần tự điều chỉnh line items trên đơn trước khi xử lý, thay vì nhờ CS thao tác Admin.
- Mục tiêu: Cho phép chỉnh sửa qty/thêm/xóa item trên My Account, đồng bộ totals/MSI, có audit trail cho CS.
- In-scope:
  - Trang order view (My Account): thêm/xóa/đổi qty item khi đơn đủ điều kiện
  - Ràng buộc trạng thái: mặc định `pending`, cấu hình được qua system config
  - Ghi order comment sau mỗi lần lưu thành công
  - Kiểm tra quyền: chỉ customer sở hữu đơn
- Out-of-scope: Admin edit UI, đổi payment/shipping/address, re-assign order, đơn đã có invoice/shipment/creditmemo

## Acceptance criteria
- AC-01: Chỉ customer sở hữu đơn mới chỉnh được; account khác bị từ chối.
- AC-02: Chỉ đơn có status trong whitelist config (default: `pending`) mới cho sửa.
- AC-03: Sau lưu, items + totals trên storefront nhất quán với payload.
- AC-04: Validate salable qty (MSI); SKU không hợp lệ hoặc vượt stock bị từ chối với message rõ.
- AC-05: Mỗi lần lưu thành công tạo order comment CS đọc được trong Admin.
- AC-06: Form key bắt buộc; lỗi không lộ stack trace.

## Decision
- Approach: Storefront MVC + service trong `Secomm_CustomerOrderEdit`; không dùng headless API.
- Business rules:
  - Phase 1: chỉ simple + virtual product
  - Từ chối nếu đơn đã có invoice/shipment/creditmemo dù status vẫn trong whitelist
  - Tối thiểu 1 dòng sau save
- Scope được phép sửa: `app/code/Secomm/CustomerOrderEdit/**`
- Risks:
  - Sai lệch MSI reservation hoặc totals sau edit → cần transaction + validate trước save
  - Thanh toán đã authorize nhưng total thay đổi → QA checklist cần rà payment method thực tế

## Testcase
- Happy: TC-01 đổi qty dòng simple → totals đúng + comment; TC-02 thêm SKU mới → dòng mới xuất hiện; TC-03 xóa dòng → biến mất + totals đúng
- Edge: TC-04 đơn rời whitelist status → không cho sửa; TC-05 qty vượt salable → từ chối, order không đổi
- Negative: TC-06 account khác cố sửa → từ chối; TC-07 form key sai → từ chối; TC-08 SKU không hợp lệ → từ chối
