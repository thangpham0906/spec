# Tham khảo: Công cụ dòng lệnh (Maintenance CLI)

Nguồn: https://experienceleague.adobe.com/en/docs/commerce-operations/configuration-guide/cli/common-cli-commands

---

## 1. Tổng quan lệnh bin/magento
Mọi thao tác quản trị Magento đều thực hiện qua lệnh:
`php bin/magento <lệnh> [tùy chọn]`

---

## 2. Nhóm lệnh Hệ thống & Mode

### 2.1 Quản lý Mode
- `bin/magento deploy:mode:show`: Kiểm tra mode hiện tại.
- `bin/magento deploy:mode:set {developer|production}`: Chuyển đổi mode.
  - Lưu ý: Production mode sẽ yêu cầu biên dịch (compile) và tạo static content.

### 2.2 Bảo trì (Maintenance)
- `bin/magento maintenance:enable`: Bật chế độ bảo trì.
- `bin/magento maintenance:disable`: Tắt chế độ bảo trì.
- `bin/magento maintenance:allow-ips <ip1> <ip2>`: Cho phép IP cụ thể truy cập trong khi bảo trì.

---

## 3. Nhóm lệnh Cache & Indexer

### 3.1 Cache
- `bin/magento cache:clean`: Xóa các mục cache hết hạn nhưng giữ lại cấu trúc (Thường dùng nhất).
- `bin/magento cache:flush`: Xóa sạch toàn bộ cache trong vùng lưu trữ (Redis/File).
- `bin/magento cache:enable / cache:disable`: Bật/tắt các loại cache cụ thể.

### 3.2 Indexer
- `bin/magento indexer:reindex`: Chạy lại toàn bộ indexer.
- `bin/magento indexer:status`: Kiểm tra trạng thái index.
- `bin/magento indexer:set-mode {realtime|schedule}`: Đặt chế độ cập nhật index.

---

## 4. Nhóm lệnh Deploy & Biên dịch (Biên dịch Static, DI)

### 4.1 Biên dịch DI
- `bin/magento setup:di:compile`: Tạo các file interceptor và proxies (Bắt buộc trong Production).

### 4.2 Static Content
- `bin/magento setup:static-content:deploy [-f]`: Tạo file tĩnh (CSS/JS/HTML) cho các area.

---

## 5. Nhóm lệnh Kho & Dữ liệu (Inventory MSI)

- `bin/magento inventory:aggregate:all`: Tính lại số lượng Salable Qty.
- `bin/magento inventory:reservation:list-inconsistencies`: Phát hiện sai lệch kho/đơn.
- `bin/magento inventory:reservation:cleanup`: Dọn dẹp bản ghi reservation cũ.
- `bin/magento inventory-geocoordinates:import [country]`: Nạp tọa độ GPS cho thuật toán giao hàng.

---

## 6. Lệnh cho Nhà phát triển (Developer Tools)

- `bin/magento dev:source-theme:deploy`: Copy file source theme vào `pub/static`.
- `bin/magento dev:tests:run`: Chạy bộ Unit Tests.
- `bin/magento dev:template-hints:enable / disable`: Bật gợi ý template trên giao diện.

---

## Liên kết
- Configuration Management: xem [configuration-management.md](./configuration-management.md)
- Quản lý Kho (MSI): xem [inventory-msi.md](./inventory-msi.md)
- Quy tắc chung: xem [../constitution.md](../constitution.md)
