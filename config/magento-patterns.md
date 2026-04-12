# Magento Patterns - Các pattern bắt buộc tuân theo

Tham khảo từ: [constitution.md](../constitution.md)

---

## 1. Plugin (Interceptor)

Tham khảo chi tiết: xem [core/plugins.md](./references/core/plugins.md)

Ưu tiên dùng plugin thay vì preference.

### Khi nào dùng:
- Cần thay đổi input/output của 1 method có sẵn.
- Cần thêm logic trước/sau method gốc.

### Quy tắc:
- Đặt trong folder `Plugin/`.
- Tên class: `<TargetClass><MôTả>Plugin.php`.
- Khai báo trong `etc/di.xml`.

---

## 2. Observer (Event)

Tham khảo chi tiết: xem [core/events-observers.md](./references/core/events-observers.md)

- Khai báo trong `etc/events.xml`.
- Class đặt trong `Observer/`.
- Mỗi observer chỉ làm một việc (Single Responsibility).

---

## 3. Repository Pattern & Service Contracts

Tham khảo chi tiết: xem [core/service-contracts.md](./references/core/service-contracts.md)

### Cấu trúc bắt buộc:
- **Api/**: Interfaces của Repository và Search Results.
- **Model/**: Cài đặt (Implementation) của Repository và Resource Models.
- Luôn sử dụng `Data Object` để truyền dữ liệu giữa các lớp.

---

## 4. ViewModel (Modern Frontend)

Tham khảo chi tiết: xem [frontend/frontend-view-models.md](./references/frontend/frontend-view-models.md)

- Thay thế cho Block class cũ.
- Implement `ArgumentInterface`.
- Truyền vào `phtml` thông qua layout XML.

---

## 5. Cấu trúc Schema & Dữ liệu

### Declarative Schema
Tham khảo chi tiết: xem [core/declarative-schema.md](./references/core/declarative-schema.md)
- File: `etc/db_schema.xml`.
- Dùng Whitelist để đảm bảo tính an toàn cho DB.

### Data Patches
Tham khảo chi tiết: xem [core/data-schema-patch.md](./references/core/data-schema-patch.md)
- Dùng để thêm dữ liệu mặc định hoặc thay đổi cấu hình qua code.

---

## 6. Logic & Vận hành

### Cron Job
Tham khảo chi tiết: xem [infrastructure/cron-jobs.md](./references/infrastructure/cron-jobs.md)
- Xử lý định kỳ (Sync, Cleanup, Report).

### Console Command (CLI)
Tham khảo chi tiết: xem [ops/maintenance-cli.md](./references/ops/maintenance-cli.md)
- Tên command chuẩn: `<vendor>:<module>:<action>`.

### Message Queues
Tham khảo chi tiết: xem [network/message-queues.md](./references/network/message-queues.md)
- Xử lý bất đồng bộ (Asynchronous) các tác vụ nặng.

---

## 8. UI Components & Data Sources

Tham khảo chi tiết: xem [frontend/ui-components.md](./references/frontend/ui-components.md)

### Quy tắc:
- Sử dụng **Admin UI Components** (`listing`, `form`) cho toàn bộ giao diện quản trị mới.
- Tách biệt logic dữ liệu vào **DataProvider** class.
- Sử dụng **Settings** thay vì **Arguments** trong cấu hình XML nếu có thể.
- Tận dụng **Template Literals** (`${ }`) để bind dữ liệu động trong JavaScript thay vì hardcode giá trị.

---

## 7. Cấu trúc thư mục Module chuẩn

```text
Vendor/Module/
├── etc/                # Cấu hình XML (di.xml, module.xml, events.xml, routes.xml)
├── Api/                # Service Contracts (Interfaces)
├── Model/              # Business Logic & Data Models
├── Controller/         # Web Actions
├── Block/              # View Logic (Deprecated)
├── ViewModel/          # Modern Frontend Logic
├── Ui/                 # UI Data Providers & Component Logic
├── Plugin/             # Interceptors
├── Observer/           # Event Handlers
├── Setup/              # Patch dữ liệu
├── view/               # Frontend (layout, templates, web)
└── registration.php    # Đăng ký module
```

---

## Liên kết Tra cứu Nhanh (Enterprise Map)

### 🔷 Core & Logic (Nền tảng)
- [Hợp đồng dịch vụ (Service Contracts)](./references/core/service-contracts.md)
- [Plugin Interceptor](./references/core/plugins.md)
- [Thiết kế Mẫu (Patterns)](./references/core/architectural-patterns.md)
- [DI & Code Gen](./references/core/di-codegen.md)
- [Schema & Patches](./references/core/declarative-schema.md)
- [Events & Observers](./references/core/events-observers.md)
- [Routing & Controllers](./references/core/routing-controllers.md)

### 🌐 Network & AI (Kết nối)
- [Web API (REST/SOAP)](./references/network/web-api.md)
- [GraphQL — Web API (reference & usage)](./references/network/graphql/README.md)
- [GraphQL — Schema guide — Attributes](./references/network/graphql/schema-attributes.md)
- [GraphQL — Schema guide — Cart](./references/network/graphql/schema-cart.md)
- [GraphQL — Development (schema, resolver, test)](./references/network/graphql/development.md)
- [GraphQL & App Server (resolver, stateless)](./references/network/graphql-app-server.md)
- [Message Queues](./references/network/message-queues.md)
- [HTTP Client](./references/network/http-client.md)

### ⚙️ Infrastructure (Vận hành)
- [Quản lý Cache (Redis/Valkey)](./references/infrastructure/cache-management.md)
- [Lưu trữ (S3)](./references/infrastructure/storage-media.md)
- [Tìm kiếm (Search)](./references/infrastructure/search-navigation.md)
- [Logging](./references/infrastructure/logging.md)
- [Cron Jobs](./references/infrastructure/cron-jobs.md)

### 🛡️ Security & Payments (Bảo mật)
- [Bảo mật Best Practices](./references/security/security-best-practices.md)
- [Payment Gateways](./references/security/payment-gateway.md)
- [Vault & Token](./references/security/payment-vault.md)

### 📊 Business & Sales (Nghiệp vụ)
- [Quản lý Kho (MSI)](./references/inventory/inventory-msi.md)
- [Giỏ hàng & Tổng phí](./references/business/quote-totals.md)
- [Khuyến mãi (Price Rules)](./references/business/catalog-price-rules.md)
- [Báo cáo (Reporting)](./references/business/advanced-reporting.md)

### 🛠️ Ops & Tools (Quản trị)
- [Công cụ dòng lệnh (CLI)](./references/ops/maintenance-cli.md)
- [Đường dẫn cấu hình (Paths)](./references/ops/config-paths.md)
- [Quản lý Config](./references/ops/configuration-management.md)
- [Quy trình Deploy](./references/ops/deployment-pipeline.md)
- [Kiểm thử (Unit Testing)](./references/ops/unit-testing.md)

### 🎨 Frontend (Giao diện)
- [UI Components](./references/frontend/ui-components.md)
- [View Models](./references/frontend/frontend-view-models.md)
- [Admin UI Grid](./references/frontend/admin-ui-grid.md)

