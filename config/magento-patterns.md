# Magento Patterns - Các pattern bắt buộc tuân theo

> version: 1.1.3 | last_updated: 2026-04-16

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
- **`before` plugin:** nếu method trả về `array` để truyền lại tham số cho method gốc, **không được** `unset()` bất kỳ tham số nào trong mảng đó (kể cả để “tránh unused variable”). `unset` làm biến mất trước `return` → cảnh báo/lỗi runtime (vd: `Undefined variable` trong GraphQL). Chỉ `return [$arg1, $arg2, ...]` đúng thứ tự; tham số không dùng vẫn giữ nguyên tên trong signature.
- **`before` plugin:** muốn “chỉ chạy side effect” và không đổi tham số → khai báo return `void` hoặc `null` theo đúng contract interception (xem `references/core/plugins.md`); không được vừa `unset` tham số vừa `return` cùng các tham số đó.

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

### Constructor DI — Logger (PSR-3)

- Khi inject logger qua constructor, **luôn** type-hint `Psr\Log\LoggerInterface` (PSR-3).
- **Không** type-hint `Magento\Psr\Log\LoggerInterface` hoặc class/interface logger khác không được `di.xml` global preference map — dễ gây lỗi generate code: `Impossible to process constructor argument ... LoggerInterface`.
- Magento core và pattern carrier chuẩn cũng dùng `Psr\Log\LoggerInterface`; bám theo đó cho module tùy chỉnh.
- Sau khi thêm/sửa constructor: chạy `bin/magento setup:di:compile` để bắt sớm lỗi wiring.

### No `new` in business code

- Cấm dùng `new` trực tiếp trong class nghiệp vụ (`Model/Service/Plugin/Observer/Controller/Resolver/ViewModel`).
- Với thư viện bên thứ ba cần input runtime (vd reCAPTCHA secret key), tạo qua factory/wrapper inject bằng DI.
- Khi code review, phải rà direct instantiation và thay bằng factory/wrapper trước khi merge.

### Core capability first

- Trước khi đề xuất tạo module/feature mới, bắt buộc kiểm tra khả năng có sẵn của Magento core và các module đã enable trong project.
- Thứ tự ưu tiên kỹ thuật:
  1) Dùng chức năng core có sẵn
  2) Cấu hình hoặc wiring nhẹ trên nền core
  3) Chỉ build custom khi 1) và 2) không đáp ứng đủ
- Khi chọn phương án custom, tài liệu `spec.md` hoặc `plan.md` phải nêu rõ gap: `core thiếu gì`, `vì sao không thể dùng lại`.
- Với task API/headless: bắt buộc kiểm tra module core liên quan (Webapi/GraphQl/Customer/Security...) trước khi tạo guard/service riêng.

---

## 7. Admin System Config Pattern (`etc/adminhtml/system.xml`)

- Ưu tiên dùng **config core Magento đã có** cho cùng domain; chỉ tạo config mới khi core không đáp ứng đủ nghiệp vụ.
- Trước khi tạo field mới, phải rà soát config path core liên quan để tránh duplicate/overlap cấu hình.
- Ưu tiên tạo section riêng theo module (`<vendor>_<module>`) và gắn vào tab vendor (vd: `secomm`), thay vì nhét group mới vào section core như `sales`.
- Không mặc định dùng `type="text"` cho mọi field. Ưu tiên chọn type theo nghiệp vụ:
  - `select` / `multiselect` khi dữ liệu thuộc tập option hữu hạn.
  - `text` chỉ dùng khi thật sự là free-text.
- Ưu tiên tái sử dụng `source_model` có sẵn của Magento trước; chỉ tạo source model custom khi core không đáp ứng đúng domain.
- Với field dạng list (`multiselect`), ưu tiên backend model chuẩn để lưu mảng (ví dụ `Magento\Config\Model\Config\Backend\Serialized\ArraySerialized`) và cập nhật class đọc config để parse đúng format.
- Với field có thể đổi vị trí UI nhưng cần giữ key cũ, bắt buộc dùng `<config_path>` để giữ tương thích dữ liệu.
- Luôn khai báo `<resource>` đúng ACL resource của module; nếu thiếu quyền, config sẽ không hiện dù XML đúng.
- Sau khi đổi `system.xml`, verify theo thứ tự:
  1) `bin/magento cache:clean config`
  2) `bin/magento cache:flush`
  3) kiểm tra đúng menu path trong Admin bằng role có quyền.

---

## 7. Carrier (Shipping Method) Pattern

Áp dụng khi tạo custom shipping carrier kế thừa `Magento\Shipping\Model\Carrier\AbstractCarrier`.

### Quy tắc bắt buộc:
- Constructor của class carrier **phải khớp đúng type-hint** với constructor parent theo phiên bản Magento hiện tại.
- Không suy đoán namespace factory; luôn đối chiếu trực tiếp với file core trước khi code.
- Với Magento 2.4.8, `AbstractCarrier` dùng:
  - `\Magento\Framework\App\Config\ScopeConfigInterface`
  - `\Magento\Quote\Model\Quote\Address\RateResult\ErrorFactory`
  - `\Psr\Log\LoggerInterface`
- `collectRates` trả `false` khi không áp dụng method; chỉ append method khi đủ điều kiện nghiệp vụ.
- Nếu method chỉ áp dụng subset sản phẩm, phải có guard rõ ràng để không ảnh hưởng carrier khác.

### Verify tối thiểu:
- Chạy `bin/magento setup:di:compile` sau khi thêm/sửa constructor DI.
- Chạy flow checkout để xác nhận method hiển thị/ẩn đúng điều kiện.

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

### 📖 Reference
- [Glossary — Domain terms](./glossary.md)

