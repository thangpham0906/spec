# Magento Patterns - Các pattern bắt buộc tuân theo

Tham khảo từ: constitution.md

---

## 1. Plugin (Interceptor)

Tham khảo chi tiết: xem [plugins.md](./references/plugins.md)

Ưu tiên dùng plugin thay vì preference.

### Khi nào dùng:

- Cần thay đổi input/output của 1 method có sẵn
- Cần thêm logic trước/sau method gốc

### Quy tắc:

- Đặt trong folder `Plugin/`
- Tên class: `<TargetClass><MôTả>Plugin.php`
    - Ví dụ: `Plugin/Customer/AccountSavePlugin.php`
- Khai báo trong `etc/di.xml`:

```xml
<type name="Magento\Customer\Model\AccountManagement">
    <plugin name="<vendor>_<module>_<mô_tả>"
            type="<Vendor>\<Module>\Plugin\Customer\AccountSavePlugin"
            sortOrder="10" />
</type>
```

- Method convention: `before<Method>`, `after<Method>`, `around<Method>`
- Hạn chế dùng `around` - chỉ khi thật sự cần thiết

---

## 2. Observer (Event)

### Khi nào dùng:

- Cần phản ứng với 1 sự kiện mà không cần thay đổi kết quả
- Logic tách rời, không phụ thuộc vào kết quả method gốc

### Quy tắc:

- Đặt trong folder `Observer/`
- Khai báo trong `etc/events.xml` (hoặc `etc/frontend/events.xml`, `etc/adminhtml/events.xml`)
- Mỗi observer chỉ làm 1 việc

---

## 3. Repository Pattern

### Cấu trúc bắt buộc cho mỗi entity chính:

```
Api/
├── <Entity>RepositoryInterface.php    # get, save, delete, getList
└── Data/
    ├── <Entity>Interface.php          # getter/setter cho entity
    └── <Entity>SearchResultsInterface.php

Model/
├── <Entity>.php                       # Model chính, implement Data Interface
├── <Entity>Repository.php             # Implement Repository Interface
├── ResourceModel/
│   ├── <Entity>.php                   # Resource model
│   └── <Entity>/
│       └── Collection.php             # Collection
```

### Khai báo trong di.xml:

```xml
<preference for="<Vendor>\<Module>\Api\<Entity>RepositoryInterface"
            type="<Vendor>\<Module>\Model\<Entity>Repository" />
<preference for="<Vendor>\<Module>\Api\Data\<Entity>Interface"
            type="<Vendor>\<Module>\Model\<Entity>" />
```

---

## 4. ViewModel

### Khi nào dùng:

- Cần truyền dữ liệu từ PHP sang template
- Thay thế việc viết logic trong Block class

### Quy tắc:

- Đặt trong folder `ViewModel/`
- Implement `Magento\Framework\View\Element\Block\ArgumentInterface`
- Khai báo trong layout XML:

```xml
<block name="my.block" template="<Vendor>_<Module>::my-template.phtml">
    <arguments>
        <argument name="viewModel" xsi:type="object">
            <Vendor>\<Module>\ViewModel\MyViewModel
        </argument>
    </arguments>
</block>
```

---

## 5. Data Patch

Tham khảo chi tiết: xem [data-schema-patch.md](./references/data-schema-patch.md)

### Khi nào dùng:

- Thêm dữ liệu mặc định (attribute, config, CMS block...)
- Migrate dữ liệu khi thay đổi schema

### Quy tắc:

- Đặt trong `Setup/Patch/Data/`
- Implement `Magento\Framework\Setup\Patch\DataPatchInterface`
- Tên class mô tả rõ việc làm: `AddPhoneAttribute.php`, `MigrateOldConfig.php`
- KHÔNG dùng InstallData, UpgradeData

---

## 6. Declarative Schema

Tham khảo chi tiết: xem [declarative-schema.md](./references/declarative-schema.md)

### Quy tắc:

- File: `etc/db_schema.xml`
- Chạy `bin/magento setup:db-declaration:generate-whitelist --module-name=<Vendor>_<Module>` sau khi thay đổi schema
- Commit cả `db_schema.xml` và `db_schema_whitelist.json`
- Đặt tên bảng: `<vendor>_<module>_<entity>`
- Primary key: `entity_id` (int, auto increment) hoặc UUID tùy trường hợp

---

## 7. Cron Job

### Khi nào dùng:

- Xử lý định kỳ (sync, cleanup, report...)

### Quy tắc:

- Khai báo trong `etc/crontab.xml`
- Class đặt trong `Cron/`
- Method chính: `execute(): void`
- Nhóm (group): tạo nhóm riêng trong `etc/cron_groups.xml` nếu cần

---

## 8. Console Command

### Quy tắc:

- Đặt trong `Console/Command/`
- Khai báo trong `etc/di.xml`:

```xml
<type name="Magento\Framework\Console\CommandListInterface">
    <arguments>
        <argument name="commands" xsi:type="array">
            <item name="<vendor>_<module>_<tên>" xsi:type="object">
                <Vendor>\<Module>\Console\Command\MyCommand
            </item>
        </argument>
    </arguments>
</type>
```

- Tên command: `<vendor>:<module>:<hành-động>`
    - Ví dụ: `<vendor>:employee:sync`

---

## 9. Config (System Configuration)

### Cấu trúc:

```s
etc/
├── adminhtml/
│   └── system.xml          # Khai báo field trong admin
├── config.xml              # Giá trị mặc định
```

### Quy tắc:

- Section: `<vendor>_<module>`
- Path: `<vendor>_<module>/<group>/<field>`
- Dùng Helper hoặc Config model để đọc giá trị, không hardcode path

---

## 19. Cấu trúc thư mục Module chuẩn

```text
Vendor/Module/
├── etc/                # Cấu hình XML (di.xml, module.xml, events.xml, routes.xml)
├── Api/                # Service Contracts (Interfaces)
├── Model/              # Business Logic & Data Models
├── Controller/         # Web Actions
├── Block/              # View Logic (Deprecated - dùng ViewModels thay thế)
├── ViewModel/          # Modern Frontend Logic
├── Plugin/             # Interceptors
├── Observer/           # Event Handlers
├── Setup/              # Patch dữ liệu & Recurring Scripts
├── view/               # Frontend (layout, templates, web)
└── registration.php    # Đăng ký module với hệ thống
```

---

## 20. Vòng đời Module (Recurring Scripts)

Dùng `Magento\Framework\Setup\InstallSchemaInterface` (nhưng đặt trong class `Recurring`) để thực thi code mỗi khi chạy lệnh `bin/magento setup:upgrade`.

- **Mục đích:** Cập nhật metadata, dọn dẹp cache đặc thù, hoặc đồng bộ dữ liệu động sau khi schema đã ổn định.

---

## Liên kết

- Quy tắc chung: xem [constitution.md](./constitution.md)
- Checklist: xem [checklist.md](./checklist.md)
- Tham khảo Declarative Schema: xem [declarative-schema.md](./references/declarative-schema.md)
- Tham khảo Plugin: xem [plugins.md](./references/plugins.md)
- Tham khảo Data/Schema Patch: xem [data-schema-patch.md](./references/data-schema-patch.md)
- Tham khảo Attribute (EAV/Extension): xem [attributes.md](./references/attributes.md)
- Tham khảo DI & Code Generation (Factories, Proxies): xem [di-codegen.md](./references/di-codegen.md)
- Tham khảo GraphQL & App Server (Stateless code): xem [graphql-app-server.md](./references/graphql-app-server.md)
- Tham khảo Service Contract (Api/Data & Repositories): xem [service-contracts.md](./references/service-contracts.md)
- Tham khảo Unit Testing (ObjectManager Helper): xem [unit-testing.md](./references/unit-testing.md)
- Tham khảo Events & Observers: xem [events-observers.md](./references/events-observers.md)
- Tham khảo Routing & Controllers: xem [routing-controllers.md](./references/routing-controllers.md)
- Tham khảo Indexing & MView: xem [indexing-mview.md](./references/indexing-mview.md)
- Tham khảo Web API (REST/SOAP): xem [web-api.md](./references/web-api.md)
- Tham khảo HTTP Client (gọi API ngoài): xem [http-client.md](./references/http-client.md)
- Tham khảo Kho hàng (MSI): xem [inventory-msi.md](./references/inventory-msi.md)
- Tham khảo Message Queues (Hàng đợi): xem [message-queues.md](./references/message-queues.md)
- Tham khảo Admin UI Grid: xem [admin-ui-grid.md](./references/admin-ui-grid.md)
- Tham khảo Quote Totals (Phí & Giảm giá): xem [quote-totals.md](./references/quote-totals.md)
- Tham khảo CLI & Bảo trì (Clear Cache): xem [maintenance-cli.md](./references/maintenance-cli.md)
- Tham khảo Các mẫu kiến trúc (Adapter/Proxy/Factory): xem [architectural-patterns.md](./references/architectural-patterns.md)
- Tham khảo View Models (Frontend Logic): xem [frontend-view-models.md](./references/frontend-view-models.md)
- Tham khảo Khuyến mãi sản phẩm (Price Rules): xem [catalog-price-rules.md](./references/catalog-price-rules.md)
- Tham khảo Tích hợp thanh toán (Payment Gateway): xem [payment-gateway.md](./references/payment-gateway.md)
- Tham khảo Lưu thẻ & Token (Vault): xem [payment-vault.md](./references/payment-vault.md)
- Tham khảo Quản lý cấu hình (Config & Security): xem [configuration-management.md](./references/configuration-management.md)
- Tham khảo Bảo mật hệ thống (Security): xem [security-best-practices.md](./references/security-best-practices.md)
- Tham khảo Tiện ích hệ thống (Utilities): xem [framework-utilities.md](./references/framework-utilities.md)
- Tham khảo Phiên bản & Tương thích (SemVer): xem [versioning-compatibility.md](./references/versioning-compatibility.md)
- Tham khảo Quản lý Cache: xem [cache-management.md](./references/cache-management.md)
- Tham khảo Báo cáo nâng cao: xem [advanced-reporting.md](./references/advanced-reporting.md)
- Tham khảo Lập lịch tác vụ (Cron Jobs): xem [cron-jobs.md](./references/cron-jobs.md)
- Tham khảo Hệ thống Ghi nhật ký (Logging): xem [logging.md](./references/logging.md)
- Tham khảo Quản lý Đa website (Multi-site): xem [multi-site-management.md](./references/multi-site-management.md)
- Tham khảo Tìm kiếm & Điều hướng (Search): xem [search-navigation.md](./references/search-navigation.md)
- Tham khảo Lưu trữ & Phương tiện (Storage): xem [storage-media.md](./references/storage-media.md)
- Tham khảo Quy trình Triển khai (Deployment): xem [deployment-pipeline.md](./references/deployment-pipeline.md)
- Tham khảo Danh mục Đường dẫn cấu hình: xem [config-paths.md](./references/config-paths.md)
- Tham khảo Quản lý Kho hàng (MSI): xem [inventory-msi.md](./references/inventory-msi.md)
