# Research Log

> Mục đích: ghi lại topic nào đã research rồi để hook tránh chạy lại.
> Cập nhật tự động sau mỗi lần hook chạy xong.
> Để force re-research 1 topic: xóa dòng đó khỏi danh sách bên dưới.

---

## Đã research

| Topic | Ngày | Kết quả |
|---|---|---|
| Plugin (interceptor): before/after/around, sortOrder, anti-patterns | 2026-04-22 | Tạo mới `config/references/core/plugins.md` |
| Observer/Event: dispatch, area-specific events, performance impact | 2026-04-22 | Cập nhật `config/references/core/events-observers.md` (thêm §6-8: disable, shared, cyclical loop) |
| Repository pattern: getList, SearchCriteria, FilterGroup | 2026-04-22 | Đã có đầy đủ trong `config/references/core/service-contracts.md` — không cần update thêm |
| Service Contracts: @api, @since, FQCN trong docblock | 2026-04-22 | Đã có đầy đủ trong `config/references/core/service-contracts.md` — không cần update thêm |
| Declarative Schema: db_schema.xml, whitelist, column types | 2026-04-22 | Đã có đầy đủ trong `config/references/core/declarative-schema.md` — không cần update thêm |
| Data Patch: getDependencies, revertable, idempotent | 2026-04-22 | Đã có đầy đủ trong `config/references/core/data-schema-patch.md` — không cần update thêm |
| DI: virtual type, proxy, factory, preference vs plugin | 2026-04-22 | Đã có đầy đủ trong `config/references/core/di-codegen.md` — không cần update thêm |
| GraphQL: resolver, schema.graphqls, context, cache | 2026-04-22 | Cập nhật `config/references/network/graphql/development.md` (thêm resolver template, context API, exception types) |
| REST API: webapi.xml, ACL, input validation, error format | 2026-04-22 | Đã có đầy đủ trong `config/references/network/rest/overview.md` — không cần update thêm |
| Message Queue: amqp, mysql, consumer, publisher | 2026-04-22 | Cập nhật `config/references/network/message-queues.md` (thêm §10: MySQL vs AMQP comparison table) |
| Cache: cache types, invalidation, Varnish, FPC | 2026-04-22 | Đã có đầy đủ trong `config/references/infrastructure/cache-management.md` — không cần update thêm |
| Indexer/Mview: reindex, schedule mode, custom indexer | 2026-04-22 | Đã có đầy đủ trong `config/references/infrastructure/indexing-mview.md` — không cần update thêm |
| EAV: attribute, attribute_set, flat table | 2026-04-22 | Đã có trong `config/references/core/attributes.md` — không cần update thêm |
| MSI: source, stock, salable_qty, reservation | 2026-04-22 | Đã có trong `config/references/inventory/inventory-msi.md` — không cần update thêm |
| Order lifecycle: state vs status, invoice, shipment, creditmemo | 2026-04-22 | Đã có trong `config/references/business/order-lifecycle.md` — không cần update thêm |
| Checkout: totals, quote, address, payment method | 2026-04-22 | Đã có trong `config/references/business/quote-totals.md` — không cần update thêm |
| Customer: account, address, group, session | 2026-04-22 | Đã có trong `config/references/business/customer-management.md` — không cần update thêm |
| Catalog: product types, price, tier price, special price | 2026-04-22 | Đã có trong `config/references/business/catalog-product-types.md` — không cần update thêm |
| ACL: acl.xml, resource, role | 2026-04-22 | Tạo mới `config/references/security/acl.md`; cập nhật `config/magento-patterns.md` |
| Layout XML: handles, blocks, containers, arguments | 2026-04-22 | Đã có đầy đủ trong `config/references/frontend/layout-xml.md` — không cần update thêm |
| ViewModel: ArgumentInterface, phtml binding | 2026-04-22 | Đã có đầy đủ trong `config/references/frontend/frontend-view-models.md` — không cần update thêm |
| UI Component: listing, form, DataProvider, columns | 2026-04-22 | Đã có trong `config/references/frontend/ui-components.md` — không cần update thêm |
| Hyva: Alpine.js, Magewire, Tailwind, compatibility | 2026-04-22 | Đã có đầy đủ trong `config/references/frontend/hyva-theme.md` — không cần update thêm |
| Performance: N+1 query, eager loading, profiler | 2026-04-22 | Đã có trong `config/references/infrastructure/performance.md` — không cần update thêm |
| Security: CSRF, XSS, SQL injection, input validation | 2026-04-22 | Cập nhật `config/references/security/security-best-practices.md` (thêm §9-10: SQL injection, input validation) |
| Logging: Monolog, virtual type logger, log levels | 2026-04-22 | Đã có trong `config/references/infrastructure/logging.md` — không cần update thêm |
| Cron: cron_schedule, group, catchup | 2026-04-22 | Cập nhật `config/references/infrastructure/cron-jobs.md` (thêm §4-8: groups, CLI, cron_schedule table, logging, config_path) |
| CLI Command: InputInterface, OutputInterface, naming convention | 2026-04-22 | Đã có đầy đủ trong `config/references/ops/maintenance-cli.md` — không cần update thêm |
| Multi-store: scope, store_view, config inheritance | 2026-04-22 | Đã có trong `config/references/ops/multi-store.md` — không cần update thêm |
| Deployment: setup:upgrade, di:compile, static-content:deploy | 2026-04-22 | Đã có trong `config/references/ops/deployment-pipeline.md` — không cần update thêm |
| Payment method: integration, offline vs online, vault token, 3DS | 2026-04-22 | Cập nhật `config/references/security/payment-gateway.md` (thêm §14: offline Adapter pattern); tạo `examples/integration/custom-payment-offline-blueprint.md` |
| Shipping carrier: AbstractCarrier, collectRates, tracking, multi-warehouse | 2026-04-22 | Cập nhật `config/references/infrastructure/shipping-carrier.md` (thêm system.xml, tracking); tạo `examples/integration/custom-shipping-carrier-blueprint.md` |
| Email template: transactional email, variables, transport, inline CSS | 2026-04-22 | Tạo `examples/integration/transactional-email-blueprint.md` |
| Import/Export: ImportModel, CSV format, behavior (add/update/delete/replace) | 2026-04-22 | Đã có đầy đủ trong `config/references/infrastructure/import-export.md` — không cần update thêm |
| Widget: WidgetInterface, instance types, layout XML integration | 2026-04-22 | Tạo mới `config/references/frontend/widget.md`; cập nhật `config/magento-patterns.md` |
| Extension Attributes: ExtensionAttributesInterface, join directive, search | 2026-04-22 | Tạo mới `config/references/core/extension-attributes.md`; cập nhật `config/magento-patterns.md` |
| CSP: Content Security Policy, whitelist, nonce, report-only mode | 2026-04-22 | Tạo mới `config/references/security/csp.md`; cập nhật `config/magento-patterns.md` |
| Redis: session storage, cache backend, sentinel, cluster config | 2026-04-22 | Tạo mới `config/references/infrastructure/redis.md`; cập nhật `config/magento-patterns.md` |
| Elasticsearch/OpenSearch: index mapping, analyzer, relevance tuning | 2026-04-22 | Cập nhật `config/references/infrastructure/search-navigation.md` (note: OpenSearch là default từ 2.4.8, Elasticsearch bị loại bỏ) |
| Upgrade: upgrade_compatibility_tool, breaking changes, deprecation | 2026-04-22 | Tạo mới `config/references/ops/upgrade.md`; cập nhật `config/magento-patterns.md` |
| Composer: module versioning, metapackage, patches (cweagans) | 2026-04-22 | Tích hợp vào `config/references/ops/upgrade.md` (§3-4) |
| Testing: PHPUnit mock, integration test fixtures, MFTF basics | 2026-04-22 | Đã có đầy đủ trong `config/references/ops/testing-guide.md` + `unit-testing.md` — không cần update thêm |
| Two-Factor Authentication: provider, config, bypass for API | 2026-04-22 | Tạo mới `config/references/security/two-factor-auth.md`; cập nhật `config/magento-patterns.md` |
| Rate Limiting & Throttling: API rate limit, DDoS protection | 2026-04-22 | Tạo mới `config/references/security/rate-limiting.md`; cập nhật `config/magento-patterns.md` |
| Custom product type: TypeInterface, price model, stock handling | 2026-04-22 | Tạo mới `config/references/business/custom-product-type.md`; tạo `examples/integration/custom-product-type-blueprint.md`; cập nhật `config/magento-patterns.md` |
| Adobe I/O Events: webhook, event provider, Commerce eventing | 2026-04-22 | Tạo mới `config/references/network/adobe-io-events.md`; cập nhật `config/magento-patterns.md` |
| API Mesh: resolver composition, transform, rate limiting | 2026-04-22 | Tạo mới `config/references/network/api-mesh.md`; cập nhật `config/magento-patterns.md` |
| PWA/Headless: GraphQL coverage, storefront compatibility, CORS | 2026-04-22 | Tạo mới `config/references/network/pwa-headless.md`; cập nhật `config/magento-patterns.md` |
| B2B: company, shared catalog, negotiable quote, purchase order | 2026-04-22 | Tạo mới `config/references/ops/b2b-modules.md`; cập nhật `config/magento-patterns.md` |
| Blueprint: Custom product type | 2026-04-22 | Tạo `examples/integration/custom-product-type-blueprint.md`; cập nhật `examples/INDEX.md` |
| Blueprint: Extension attributes với join | 2026-04-22 | Tạo `examples/integration/extension-attributes-join-blueprint.md`; cập nhật `examples/INDEX.md` |
| Blueprint: Custom widget | 2026-04-22 | Tạo `examples/integration/custom-widget-blueprint.md`; cập nhật `examples/INDEX.md` |
| Blueprint: Integration test module | 2026-04-22 | Tạo `examples/integration/integration-test-module-blueprint.md`; cập nhật `examples/INDEX.md` |
| Blueprint: Console command với progress bar | 2026-04-22 | Tạo `examples/integration/console-command-progress-bar-blueprint.md`; cập nhật `examples/INDEX.md` |
| Blueprint: Custom price modifier (catalog rule) | 2026-04-22 | Tạo `examples/integration/custom-price-modifier-blueprint.md`; cập nhật `examples/INDEX.md` |

---

## Chưa research

### Nâng cao / chuyên sâu (Adobe Commerce only)
- Staging & Preview (Commerce): version, update, campaign, timeline

### Core / Architecture
- Around plugin: khi nào dùng, performance cost, callable pattern đúng
- Interceptor chain: thứ tự thực thi khi nhiều plugin cùng target 1 method
- Object Manager pool: shared vs non-shared instance, scope
- Generated code: Interceptor, Factory, Proxy — khi nào regenerate, troubleshoot
- Config XML merge: load order, area (global/frontend/adminhtml/webapi_rest/cron)
- Module sequence: `<sequence>` trong module.xml, circular dependency
- Area code: frontend/adminhtml/webapi_rest/webapi_soap/graphql/cron — ảnh hưởng DI
- AbstractModel vs DataObject: khi nào dùng cái nào, magic getter/setter
- Collection: addFieldToFilter, join, group, having — performance pitfalls
- ResourceModel: _beforeSave/_afterSave, connection, table prefix

### Frontend
- RequireJS: mixins, map, shim, bundles, async loading
- KnockoutJS: observable, computed, custom binding, component lifecycle
- Checkout steps: custom step, payment renderer, shipping method renderer
- Admin form: fieldset, field types, dependencies, dynamic rows
- Admin grid: filters, mass actions, inline edit, bookmarks
- LESS/CSS: extend, mixin, variables, critical CSS, theme inheritance
- Luma theme: parent theme override, _module.less, _extend.less
- Pager/Toolbar: product list toolbar, sort, limit, custom toolbar

### Business / Domain
- Catalog price: price waterfall, custom price modifier, group price
- Cart rules: condition combine, action types, coupon generation
- Tax: tax class, tax rule, FPT, display settings, store config
- Shipping: rate request, rate result, free shipping, table rates
- Order management: hold/unhold, cancel, reorder, partial invoice/ship
- Return/RMA: return request, item condition, resolution (Commerce)
- Gift card: account, balance, usage (Commerce)
- Reward points: earn/spend rules, balance, expiry (Commerce)
- Customer segment: condition, auto-assign, use in price rules (Commerce)
- Catalog permission: category/product access by customer group (Commerce)

### Infrastructure / DevOps
- Varnish: VCL config, ESI, cache purge, X-Magento-Tags
- RabbitMQ: exchange, queue, binding, dead letter, management UI
- MySQL: query optimization, EXPLAIN, slow query log, connection pool
- New Relic: APM integration, custom attributes, transaction naming
- Blackfire: profiling, assertions, CI integration
- Xdebug: step debug, profiling, remote debug Docker/DDEV
- Docker/DDEV: service config, custom commands, Xdebug toggle
- CI/CD: GitHub Actions / GitLab CI cho Magento, ECE-tools
- Cloud (Adobe Commerce Cloud): ece-tools, .magento.env.yaml, patches
- Monitoring: health check endpoint, cron health, queue health

### Security
- Admin security: brute force protection, session lifetime, IP whitelist
- API token: integration token, customer token, admin token lifecycle
- Encryption: key rotation, sensitive config, env.php
- File permissions: var/, pub/, generated/ — production vs dev
- Dependency confusion: composer package naming, private packagist

### Testing & Quality
- Unit test: mock ObjectManager, mock Repository, data provider
- Integration test: fixtures, rollback, database isolation
- API functional test: WebApiAbstract, REST/GraphQL test
- Static analysis: PHPStan, PHPMD, Magento coding standard (PHPCS)
- Mutation testing: infection/phpunit, score interpretation
- Load testing: k6, JMeter cho Magento checkout flow

### Blueprint còn thiếu
- Blueprint: Custom checkout step (frontend + backend)
- Blueprint: Custom admin form với dynamic rows
- Blueprint: Custom mass action trong admin grid
- Blueprint: Custom tax rule / FPT
- Blueprint: Custom cart price rule condition
- Blueprint: Varnish VCL cho Magento
- Blueprint: RabbitMQ consumer với retry/dead letter
- Blueprint: Custom REST API với pagination + filtering
- Blueprint: GraphQL mutation với input validation
- Blueprint: Custom report / grid với custom DataProvider
- Blueprint: Module với unit test + integration test đầy đủ
- Blueprint: Custom attribute (product/customer/category) với frontend renderer
- Blueprint: Custom checkout payment renderer (JS + PHP)
- Blueprint: Custom shipping rate với table rates override
- Blueprint: Multi-store config với store-specific override

### Advanced Patterns / Architecture
- Command pattern: Command bus, CommandInterface, CommandPoolInterface
- Strategy pattern: pool of strategies, dynamic selection via DI
- Composite pattern: CompositeInterface, chaining processors
- Pipeline pattern: processor chain, SortedList, PipelineInterface
- Specification pattern: business rule objects, isSatisfiedBy
- Event Sourcing: domain events, event store, replay
- CQRS: separate read/write models trong Magento context
- Saga pattern: distributed transaction, compensating actions
- Decorator pattern: wrapping service với extra behavior
- Null Object pattern: tránh null check, default implementation

### Data Layer
- Collection vs Repository: khi nào dùng cái nào, performance trade-off
- SearchCriteria deep dive: FilterGroup logic (AND/OR), SortOrder, PageSize
- Custom SearchResults: SearchResultsInterface, TotalCount
- Database transaction: beginTransaction, commit, rollback trong ResourceModel
- Optimistic locking: version field, conflict detection
- Soft delete: is_active flag, filter trong collection
- Bulk operations: insertMultiple, insertOnDuplicate, deleteByIds
- Database connection: read/write split, slave connection
- Custom DB function: MySQL stored procedure từ Magento
- Schema migration: alter column safely, zero-downtime migration

### API / Integration
- Async REST API: bulk endpoint, /async/bulk, UUID tracking
- Webhook: outbound webhook, retry, signature verification
- OAuth 1.0a: integration token flow, consumer key/secret
- JWT: customer token structure, expiry, refresh
- SOAP API: WSDL generation, complex types, fault handling
- GraphQL subscription: real-time updates (nếu có)
- API versioning: V1/V2, backward compat strategy
- Hypermedia: HATEOAS trong Magento REST response
- API documentation: Swagger/OpenAPI generation từ webapi.xml
- Rate limit per customer/IP: custom implementation

### Frontend Advanced
- Custom form element: UI Component field type mới
- Dynamic rows: UI Component dynamic_rows, record template
- Modal component: modal, slide, popup — trigger từ JS
- Notification component: messages, global messages, inline
- Multiselect/Select2: custom options provider, AJAX load
- File upload: UI Component file uploader, validation
- Date/time picker: UI Component date, timezone handling
- Color picker: custom field type
- WYSIWYG: TinyMCE integration, custom plugin
- Map/Location: Google Maps integration trong admin form

### Checkout Advanced
- Custom checkout field: shipping/billing address custom field
- Custom checkout validation: JS mixin, custom validator
- One-step checkout: layout override, step removal
- Guest checkout: convert guest to customer post-order
- Persistent cart: remember cart across sessions
- Mini cart: custom item renderer, totals
- Order summary: custom totals renderer
- Checkout agreements: terms and conditions, checkbox
- Address suggestion: Google Places API integration
- Checkout with multiple addresses: multishipping flow

### Customer Advanced
- Customer attribute: custom EAV attribute, frontend input type
- Customer group pricing: tier price per group
- Customer segment: dynamic segment, condition types (Commerce)
- Wishlist: add/remove, share, move to cart
- Compare products: comparison list, attributes shown
- Recently viewed: session vs persistent storage
- Customer notification: subscription, unsubscribe
- Social login: OAuth provider integration
- GDPR: data export, anonymization, consent
- Account lockout: failed login attempts, unlock

### Catalog Advanced
- Layered navigation: custom filter, price filter, swatch filter
- Product relations: upsell, crosssell, related — custom rule
- Bundle product: dynamic price, fixed price, ship separately
- Downloadable product: link, sample, purchase limit
- Virtual product: no shipping, service-based
- Grouped product: associated products, qty handling
- Configurable product: swatch, super attribute, child visibility
- Product video: YouTube/Vimeo embed, gallery
- Product attachment: custom downloadable file per product
- Dynamic category: smart category rules, auto-assign products

### Sales / Order Advanced
- Custom order status: new status, state mapping, notification
- Order comment: visible to customer, email notification
- Partial refund: creditmemo with adjustment fee/refund
- Order archive: archive old orders, performance
- Custom invoice: PDF customization, logo, fields
- Custom packing slip: PDF template
- Order export: CSV/XML export, scheduled export
- Return merchandise: RMA flow, label generation
- Fraud detection: custom fraud check, order hold
- Split order: multiple shipments, partial fulfillment

### Performance Advanced
- Full Page Cache: hole punching, private content, ESI
- HTTP/2: server push, multiplexing, header compression
- Image optimization: WebP, lazy load, srcset, CDN
- JS bundling: RequireJS bundle, critical JS, defer
- CSS critical path: above-fold CSS, async load
- Database query cache: query cache vs Redis cache
- Flat catalog: flat product/category table, pros/cons
- Async email: queue email sending, batch processing
- Deferred stock update: async inventory deduction
- Profiling: Magento built-in profiler, Blackfire, Tideways

### DevOps / Infrastructure Advanced
- Zero-downtime deployment: blue-green, rolling update
- Database backup: automated backup, point-in-time recovery
- Log aggregation: ELK stack, Graylog, Papertrail
- Error tracking: Sentry integration, error grouping
- Uptime monitoring: Pingdom, StatusCake, custom health check
- CDN: Fastly, Cloudflare, cache rules cho Magento
- Object storage: S3/GCS cho media, pub/media sync
- Kubernetes: Magento on K8s, horizontal scaling, session sharing
- Terraform: infrastructure as code cho Magento cloud
- Ansible: configuration management, deployment automation

### Blueprint còn thiếu (nâng cao)
- Blueprint: Custom checkout field (address + JS validation)
- Blueprint: Custom layered navigation filter
- Blueprint: Custom order PDF (invoice/packing slip)
- Blueprint: GDPR data export + anonymization
- Blueprint: Custom customer attribute với frontend renderer
- Blueprint: Async bulk REST API endpoint
- Blueprint: Custom GraphQL subscription
- Blueprint: Full Page Cache hole punching (private content)
- Blueprint: Custom configurable product swatch
- Blueprint: Zero-downtime deployment script
- Blueprint: Custom fraud detection plugin
- Blueprint: Multi-source inventory custom algorithm
- Blueprint: Custom report với chart (admin dashboard)
- Blueprint: Webhook outbound với retry queue
- Blueprint: Custom search adapter (OpenSearch)

### PHP / OOP Deep Dive (Magento context)
- Abstract class vs Interface: khi nào dùng cái nào trong Magento
- Trait: dùng trong Magento có hợp lệ không, use case thực tế
- PHP 8.x features: named arguments, match expression, nullsafe operator, fibers
- Readonly properties: PHP 8.1+, dùng trong DTO/Data Object
- Enum: PHP 8.1+, dùng thay constant trong Magento
- First-class callable: PHP 8.1+, array_map với method reference
- Intersection types: PHP 8.1+, type safety nâng cao
- Generator: yield, lazy collection, memory-efficient iteration
- Closure binding: Closure::bind, Closure::fromCallable
- Anonymous class: test double, inline implementation

### Magento Code Patterns (thực chiến)
- Fluent interface: method chaining trong Builder/Query
- Builder pattern: SearchCriteriaBuilder, FilterBuilder deep dive
- Registry pattern: Magento\Framework\Registry — legacy, cách thay thế
- Action pool: array of actions, dynamic dispatch
- Modifier pattern: UI Component modifier, pool modifier
- Converter pattern: toDataModel, toArray, hydrator
- Validator chain: ValidatorInterface, CompositeValidator
- Processor chain: ProcessorInterface, sorted processor pool
- Mapper pattern: DB row → Data Object mapping
- Hydrator pattern: populate object từ array data

### Debugging & Troubleshooting (thực chiến)
- Xdebug step debug: breakpoint, watch, call stack trong PhpStorm
- Magento profiler: enable/disable, HTML output, custom profiler
- Query log: enable query log, slow query, EXPLAIN trong Magento
- DI compile error: common errors, how to fix, regenerate
- Plugin conflict: debug interceptor chain, identify conflicting plugin
- Observer infinite loop: detect, prevent, area restriction
- Memory leak: detect với Blackfire, common causes trong Magento
- 500 error debug: exception.log, system.log, display_errors
- White screen of death: common causes, recovery steps
- Cache corruption: symptoms, flush strategy, cache backend check

### Code Generation & Tooling
- Pestle: module scaffold, di.xml generation, common commands
- Mage2Gen: online generator, module skeleton
- n98-magerun2: common commands, custom commands, scripting
- PHPStorm Magento plugin: DI navigation, plugin generation, inspections
- PHP CS Fixer: Magento ruleset, auto-fix, CI integration
- PHPStan: Magento extension, level config, baseline
- Rector: automated refactoring, Magento-specific rules
- GrumPHP: pre-commit hooks, quality gates
- Composer scripts: post-install, post-update automation
- Makefile: common Magento dev tasks automation

### Event / Observer Patterns (thực chiến)
- sales_order_place_after: common use cases, data available
- catalog_product_save_after: product save hook, reindex trigger
- customer_login: session data, redirect logic
- checkout_cart_add_product_complete: cart modification
- controller_action_predispatch: request intercept, redirect
- layout_generate_blocks_after: dynamic block injection
- adminhtml_block_html_before: admin block modification
- clean_cache_by_tags: custom cache invalidation
- magento_customer_authenticated: post-auth hook
- Custom event dispatch: best practices, naming convention, area

### Plugin Patterns (thực chiến)
- Before plugin: modify arguments, add validation
- After plugin: modify return value, add data
- Around plugin: conditional execution, skip original
- Plugin on interface vs class: best practice
- Plugin disabled: di.xml disabled="true", area-specific disable
- Plugin sortOrder conflict: debug, resolve
- Plugin on Repository: common patterns (add filter, transform result)
- Plugin on Controller: redirect, modify response
- Plugin on Block: modify template, add data
- Plugin on Model: intercept save/load/delete

### Repository & Collection Patterns (thực chiến)
- Custom filter: addFilter với custom condition
- Join extension attribute: joinField, joinTable
- Custom sort: addOrder, custom sort direction
- Aggregate query: group by, count, sum trong collection
- Subquery: correlated subquery trong Magento collection
- Raw query: getConnection()->query() khi nào dùng
- Batch processing: load collection in chunks, memory management
- Collection cache: setPageSize, setCurPage, pagination
- Custom resource model: _getLoadSelect override
- Multi-table join: joinLeft, joinInner, alias

### GraphQL Deep Dive (thực chiến)
- Custom query: schema.graphqls, resolver, di.xml
- Custom mutation: input type, output type, validator
- Custom type: interface type, union type
- Resolver chain: ResolverInterface, composite resolver
- Context: UserContext, StoreContext, custom context
- Cache: @cache directive, cache identity, invalidation
- Error handling: GraphQlInputException, GraphQlNoSuchEntityException
- Authorization: isAllowed, customer context check
- Pagination: PageInfo, currentPage, pageSize
- Custom scalar type: custom scalar resolver

### REST API Deep Dive (thực chiến)
- Custom endpoint: webapi.xml, interface, implementation
- Request validation: custom validator, input filter
- Response transformation: custom response builder
- Bulk endpoint: /async/bulk, operation status
- Custom search criteria: custom filter, custom sort
- File upload via REST: multipart/form-data handling
- Streaming response: large data export
- Custom error response: WebapiException, HTTP status codes
- API versioning: V1/V2 coexistence
- Custom authentication: custom token provider

### Blueprint (dev-focused, code đầy đủ)
- Blueprint: Plugin trên OrderRepository (add custom filter)
- Blueprint: Before plugin validate input + throw exception
- Blueprint: After plugin transform response data
- Blueprint: Observer gửi email sau order place
- Blueprint: Observer sync data sang external API
- Blueprint: Custom collection với join + filter
- Blueprint: Repository với custom SearchCriteria filter
- Blueprint: Custom GraphQL query với auth check
- Blueprint: Custom GraphQL mutation với input validation
- Blueprint: Custom REST endpoint với file upload
- Blueprint: Custom REST bulk endpoint
- Blueprint: Custom CLI command với progress bar + batch
- Blueprint: Custom cron job với lock mechanism
- Blueprint: Custom message queue consumer với retry
- Blueprint: Custom validator chain (CompositeValidator)
- Blueprint: Custom price modifier plugin
- Blueprint: Custom checkout total collector
- Blueprint: Custom shipping rate provider
- Blueprint: Custom payment method (offline, simple)
- Blueprint: Custom product attribute với source model
- Blueprint: Custom customer attribute với frontend input
- Blueprint: Custom admin notification
- Blueprint: Custom system config field với custom renderer
- Blueprint: Custom ACL resource + check trong controller
- Blueprint: Custom layout handle + block injection
- Blueprint: Custom UI Component field type
- Blueprint: Custom DataProvider cho admin grid
- Blueprint: Custom mass action với confirmation
- Blueprint: Custom inline edit trong admin grid
- Blueprint: Debug plugin: log all method calls
