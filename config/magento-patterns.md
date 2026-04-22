# Magento Patterns - Index

> version: 1.2.0 | last_updated: 2026-04-17
>
> File này là **index** — mỗi pattern chỉ ghi tên, khi nào dùng, và link reference chi tiết.
> Khi implement pattern nào: **bắt buộc đọc reference tương ứng trước khi code**.

---

## Quy tắc cứng (áp dụng mọi pattern)

- Không sửa file core Magento
- Không dùng `ObjectManager::getInstance()`
- Không hardcode store ID, website ID, customer group ID
- `declare(strict_types=1)` bắt buộc mọi file PHP
- Inject dependency qua constructor, cấm `new` trong `Model/Service/Plugin/Observer/Controller/Resolver/ViewModel`
- Logger: type-hint `Psr\Log\LoggerInterface` (không dùng `Magento\Psr\Log\LoggerInterface`)
- Ưu tiên plugin thay vì `<preference>`
- `before` plugin return array: không `unset()` tham số sẽ được return

---

## Pattern Index

### Core & Architecture

| Pattern | Dùng khi | Reference |
|---|---|---|
| Plugin (Interceptor) | Thêm logic before/after/around method có sẵn | [core/plugins.md](./references/core/plugins.md) |
| Observer (Event) | React với event Magento dispatch | [core/events-observers.md](./references/core/events-observers.md) |
| Repository + Service Contract | CRUD entity, public API của module | [core/service-contracts.md](./references/core/service-contracts.md) |
| Declarative Schema | Tạo/sửa bảng DB | [core/declarative-schema.md](./references/core/declarative-schema.md) |
| Data Patch | Seed dữ liệu mặc định, migration data | [core/data-schema-patch.md](./references/core/data-schema-patch.md) |
| DI & Code Generation | Virtual type, proxy, factory, preference | [core/di-codegen.md](./references/core/di-codegen.md) |
| Routing & Controllers | Tạo route frontend/adminhtml | [core/routing-controllers.md](./references/core/routing-controllers.md) |
| Architectural Patterns | SOLID, composition, design patterns | [core/architectural-patterns.md](./references/core/architectural-patterns.md) |
| Extension Attributes | Mở rộng API Data Interface, expose qua REST | [core/extension-attributes.md](./references/core/extension-attributes.md) |

### Network & API

| Pattern | Dùng khi | Reference |
|---|---|---|
| Web API (REST/SOAP) | Expose endpoint qua `webapi.xml` | [network/web-api.md](./references/network/web-api.md) |
| GraphQL | Query/mutation API headless | [network/graphql/README.md](./references/network/graphql/README.md) |
| GraphQL Schema — Attributes | Thêm field vào GraphQL schema | [network/graphql/schema-attributes.md](./references/network/graphql/schema-attributes.md) |
| GraphQL Schema — Cart | Mở rộng cart GraphQL | [network/graphql/schema-cart.md](./references/network/graphql/schema-cart.md) |
| GraphQL Development | Resolver, test, app server | [network/graphql/development.md](./references/network/graphql/development.md) |
| GraphQL App Server | Stateless resolver | [network/graphql-app-server.md](./references/network/graphql-app-server.md) |
| Message Queue (RabbitMQ) | Xử lý bất đồng bộ, tác vụ nặng | [network/message-queues.md](./references/network/message-queues.md) |
| HTTP Client | Gọi external API | [network/http-client.md](./references/network/http-client.md) |
| Adobe I/O Events | Emit events đến App Builder, event-driven integration | [network/adobe-io-events.md](./references/network/adobe-io-events.md) |
| API Mesh | Compose nhiều API sources thành 1 GraphQL endpoint | [network/api-mesh.md](./references/network/api-mesh.md) |
| PWA/Headless | GraphQL storefront, CORS, authentication | [network/pwa-headless.md](./references/network/pwa-headless.md) |

### Frontend

| Pattern | Dùng khi | Reference |
|---|---|---|
| ViewModel | Logic cho phtml, thay Block cũ | [frontend/frontend-view-models.md](./references/frontend/frontend-view-models.md) |
| Layout XML | Khai báo cấu trúc trang, blocks, containers | [frontend/layout-xml.md](./references/frontend/layout-xml.md) |
| UI Components (Admin Grid/Form) | Admin listing, form CRUD | [frontend/ui-components.md](./references/frontend/ui-components.md) |
| Admin UI Grid | Admin grid với filter/sort/action | [frontend/admin-ui-grid.md](./references/frontend/admin-ui-grid.md) |
| Admin Form Save and Continue | Admin form giữ lại trang edit sau save | [frontend/admin-save-and-continue.md](./references/frontend/admin-save-and-continue.md) |
| Hyvä Theme | Alpine.js, Magewire, Tailwind CSS frontend | [frontend/hyva-theme.md](./references/frontend/hyva-theme.md) |
| Widget | Reusable CMS component cấu hình từ Admin | [frontend/widget.md](./references/frontend/widget.md) |

### Infrastructure & Ops

| Pattern | Dùng khi | Reference |
|---|---|---|
| Cache Management | Redis/Valkey, cache type custom | [infrastructure/cache-management.md](./references/infrastructure/cache-management.md) |
| Indexer & Mview | Custom indexer, reindex strategy, schedule mode | [infrastructure/indexing-mview.md](./references/infrastructure/indexing-mview.md) |
| Performance | N+1 query, eager loading, profiler, Varnish | [infrastructure/performance.md](./references/infrastructure/performance.md) |
| Storage (S3) | Upload/lưu file trên S3 | [infrastructure/storage-media.md](./references/infrastructure/storage-media.md) |
| Search & Navigation | OpenSearch, layered nav | [infrastructure/search-navigation.md](./references/infrastructure/search-navigation.md) |
| Logging | Custom logger, Monolog channel | [infrastructure/logging.md](./references/infrastructure/logging.md) |
| Cron Job | Tác vụ định kỳ | [infrastructure/cron-jobs.md](./references/infrastructure/cron-jobs.md) |
| Shipping Carrier | Custom shipping method | [infrastructure/shipping-carrier.md](./references/infrastructure/shipping-carrier.md) |
| Import/Export | Custom import entity CSV | [infrastructure/import-export.md](./references/infrastructure/import-export.md) |
| Redis | Session storage, cache backend, Sentinel | [infrastructure/redis.md](./references/infrastructure/redis.md) |
| CLI Command | Console command `bin/magento` | [ops/maintenance-cli.md](./references/ops/maintenance-cli.md) |
| Multi-Store | Scope, store_view, config inheritance | [ops/multi-store.md](./references/ops/multi-store.md) |
| Config Paths | Đọc/ghi system config | [ops/config-paths.md](./references/ops/config-paths.md) |
| Configuration Management | Scope, deploy config | [ops/configuration-management.md](./references/ops/configuration-management.md) |
| Deployment Pipeline | Deploy flow, setup commands | [ops/deployment-pipeline.md](./references/ops/deployment-pipeline.md) |
| Unit Testing | PHPUnit, mock, test structure | [ops/unit-testing.md](./references/ops/unit-testing.md) |
| Upgrade & Compatibility | UCT, cweagans patches, Composer versioning | [ops/upgrade.md](./references/ops/upgrade.md) |

### Security & Payments

| Pattern | Dùng khi | Reference |
|---|---|---|
| Security Best Practices | Input validation, XSS, CSRF, SQL injection | [security/security-best-practices.md](./references/security/security-best-practices.md) |
| ACL | Phân quyền Admin, API, block | [security/acl.md](./references/security/acl.md) |
| Payment Gateway | Tích hợp payment method (online) | [security/payment-gateway.md](./references/security/payment-gateway.md) |
| Vault & Token | Lưu token thanh toán | [security/payment-vault.md](./references/security/payment-vault.md) |
| CSP | Content Security Policy, whitelist, nonce | [security/csp.md](./references/security/csp.md) |
| Two-Factor Authentication | 2FA provider, bypass dev, REST API token | [security/two-factor-auth.md](./references/security/two-factor-auth.md) |
| Rate Limiting | Chống carding attack, API throttling, DDoS | [security/rate-limiting.md](./references/security/rate-limiting.md) |

### Business & Sales

| Pattern | Dùng khi | Reference |
|---|---|---|
| Inventory MSI | Multi-source inventory, reservation | [inventory/inventory-msi.md](./references/inventory/inventory-msi.md) |
| Order Lifecycle | State/status flow, invoice, shipment, creditmemo | [business/order-lifecycle.md](./references/business/order-lifecycle.md) |
| Customer Management | CustomerRepository, address, group, session | [business/customer-management.md](./references/business/customer-management.md) |
| Catalog Product Types | Product types, price hierarchy, EAV attributes | [business/catalog-product-types.md](./references/business/catalog-product-types.md) |
| Custom Product Type | Tạo product type mới với price model riêng | [business/custom-product-type.md](./references/business/custom-product-type.md) |
| Quote & Totals | Tính giá, totals collector | [business/quote-totals.md](./references/business/quote-totals.md) |
| Catalog Price Rules | Rule giảm giá catalog | [business/catalog-price-rules.md](./references/business/catalog-price-rules.md) |
| Reporting | Advanced reporting, grid report | [business/advanced-reporting.md](./references/business/advanced-reporting.md) |
| B2B Modules | Company, shared catalog, negotiable quote, purchase order | [ops/b2b-modules.md](./references/ops/b2b-modules.md) |

---

## Cấu trúc thư mục Module chuẩn

```text
Vendor/Module/
├── etc/                # di.xml, module.xml, events.xml, routes.xml
├── Api/                # Service Contracts (Interfaces)
├── Model/              # Business Logic & Data Models
├── Controller/         # Web Actions
├── ViewModel/          # Modern Frontend Logic (thay Block)
├── Ui/                 # UI Data Providers
├── Plugin/             # Interceptors
├── Observer/           # Event Handlers
├── Setup/              # Data/Schema Patches
├── Test/Unit/          # Unit tests (mirror src structure)
├── view/               # layout, templates, web
└── registration.php
```
