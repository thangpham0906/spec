# Feature Plan - customer-order-edit

> Tham chiếu nghiệp vụ/AC: `spec.md`. File này chỉ mô tả cách làm kỹ thuật.

## Mục tiêu kỹ thuật
- Triển khai module `Secomm_CustomerOrderEdit` (`app/code/Secomm/CustomerOrderEdit`) theo storefront MVC: route frontend, controller, ViewModel, template, service xử lý nghiệp vụ.
- Cho phép khách (đã đăng nhập) chỉnh `sales_order_item` trên đơn **chỉ khi** đơn thỏa rule trạng thái + an toàn vận hành; sau lưu order totals/items nhất quán, MSI reservation đúng với thay đổi, và có **order status history comment** cho CS.
- Không sửa core; không thêm màn "sửa đơn" trong Admin (chỉ có **system config** nếu cần — không thuộc "flow edit order" trong spec).

## Thiết kế giải pháp

### 1) Cấu hình
- `etc/adminhtml/system.xml` + `etc/config.xml`: path ví dụ `sales/secomm_customer_order_edit/allowed_statuses` — danh sách `status` (hoặc `state` nếu chuẩn hóa một kiểu) được phép sửa; **default** = `pending`.
- `etc/acl.xml`: quyền đọc section config (tối thiểu cho role có quyền cấu hình store).
- `Model/Config` (hoặc tương đương): đọc config theo store scope, normalize (trim, lowercase nếu cần).

### 2) Điều kiện "được phép sửa" (server-side, bắt buộc lặp lại ở mọi entry)
- Customer đã login; `order.getCustomerId()` khớp session customer.
- `order.getStatus()` (hoặc state — **chốt một chuẩn trong code**) nằm trong tập từ config.
- **Chốt an toàn bổ sung (MVP):** từ chối nếu đơn đã có **invoice** hoặc **shipment** hoặc **credit memo** (`hasInvoices()`, `hasShipments()`, số credit memo > 0) để tránh lệch chứng từ dù status vẫn nằm trong whitelist.
- Từ chối nếu không còn item nào sau thao tác (hoặc validate tối thiểu 1 dòng — tùy chính sách; mặc định **bắt buộc còn ít nhất 1 dòng** sau save).

### 3) Luồng chỉnh sửa line items (service tách biệt Controller)
- Class service ví dụ `Model/OrderItemEditService` (tên cuối tùy code): nhận DTO/struct "danh sách thao tác" (giữ item_id + qty mới; thêm: sku + qty; xóa: item_id) — **validate toàn bộ trước khi ghi DB**.
- **Phase 1 product types:** chỉ **simple** và **virtual** (đủ cho SKU cố định); **configurable/bundle/grouped** — từ chối rõ ràng hoặc ẩn UI cho đến phase sau (ghi trong release note/tasks).
- Thêm dòng: `ProductRepositoryInterface::get` theo SKU (store scope), kiểm tra `status`, `visibility`, `isSalable`/salable qty (MSI), không cho vượt số lượng có thể bán; tạo `OrderItem` mới gắn vào order theo pattern Magento (product options, price từ price info hiện tại — **ghi rõ trong code** là dùng giá catalog tại thời điểm sửa hay giữ logic đồng nhất với reorder; MVP khuyến nghị: **tính lại row total theo giá hiện tại + rule tax** và ghi vào comment để CS biết).
- Đổi qty / xóa dòng: kiểm tra item thuộc order; với configurable parent trong tương lai phải xử lý child — Phase 1 bỏ qua.
- Sau khi cấu trúc item ổn định: gọi `$order->collectTotals()` (hoặc pipeline tương đương được khuyến nghị trong phiên bản project), rồi `OrderRepositoryInterface::save`.
- **MSI / reservation:** sau khi đổi nội dung đơn, chênh lệch reservation phải được xử lý đúng (giải phóng phần giảm, đặt thêm phần tăng). Thiết kế: tách `Model/Reservation/*` hoặc dùng API/module `InventorySales` phù hợp 2.4.8 (compensation / place reservations theo sales event) — **không** để order lưu mà reservation lệch. Chi tiết class gọi cụ thể do implement bám vendor Magento trong môi trường build.

### 4) Audit / lịch sử
- Sau save thành công: thêm **status history** trên order với comment **customer-visible hoặc chỉ admin** tùy chính sách — MVP: `is_customer_notified = false`, `is_visible_on_front = false` để CS đọc trong Admin order view; nội dung: tóm tắt trước/sau (sku, qty) hoặc JSON ngắn gọn có thể đọc.
- Dùng API/history repository chuẩn (`OrderStatusHistoryRepositoryInterface` / factory từ order) thay vì raw SQL.

### 5) Storefront UI
- Layout XML: thêm link/block trên `sales_order_view` (hoặc route con) khi ViewModel `canEdit()` true.
- Trang edit: layout + template + ViewModel cung cấp items hiện tại, form key, action URL.
- Controller Save: `implements HttpPostActionInterface`, validate form key, delegate service, redirect với message manager; bắt `LocalizedException` hiển thị message an toàn.

### 6) Dependency & module sequence
- `etc/module.xml`: `sequence` tối thiểu `Magento_Sales`, `Magento_Customer`, `Magento_Catalog`; nếu dùng MSI API thì thêm `Magento_InventorySales` (hoặc module inventory tương ứng môi trường).

## Phạm vi file/module dự kiến
- `app/code/Secomm/CustomerOrderEdit/registration.php`
- `app/code/Secomm/CustomerOrderEdit/etc/module.xml`
- `app/code/Secomm/CustomerOrderEdit/etc/config.xml`
- `app/code/Secomm/CustomerOrderEdit/etc/adminhtml/system.xml`
- `app/code/Secomm/CustomerOrderEdit/etc/adminhtml/acl.xml`
- `app/code/Secomm/CustomerOrderEdit/etc/frontend/routes.xml`
- `app/code/Secomm/CustomerOrderEdit/etc/di.xml`
- `app/code/Secomm/CustomerOrderEdit/Controller/Order/*.php`
- `app/code/Secomm/CustomerOrderEdit/ViewModel/**/*.php`
- `app/code/Secomm/CustomerOrderEdit/Model/**/*.php` (Config, Service, Validator, Reservation helper)
- `app/code/Secomm/CustomerOrderEdit/view/frontend/layout/*.xml`, `templates/**/*.phtml`, `web/js` nếu cần
- `app/code/Secomm/CustomerOrderEdit/i18n/*.csv` (chuỗi UI)
- `app/code/Secomm/CustomerOrderEdit/composer.json` (khuyến nghị)

## Rủi ro + rollback
- **Rủi ro:** Sai lệch reservation/totals sau edit; thanh toán đã authorize nhưng total đổi — cần rà phương thức thanh toán thực tế (MVP: cảnh báo trong QA checklist; có thể giới hạn thêm "chỉ COD/bank transfer" bằng config sau nếu cần).
- **Cách giảm rủi ro:** Transaction DB khi phù hợp; validate toàn bộ trước save; kiểm tra MSI bằng integration test thủ công trên env có MSI; chặn invoice/shipment.
- **Rollback:** Disable module `Secomm_CustomerOrderEdit`, `setup:upgrade` nếu có schema, `cache:flush`; revert code theo VCS.
