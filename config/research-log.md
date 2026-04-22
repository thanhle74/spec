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

---

## Chưa research

### Nâng cao / chuyên sâu
- Payment method: integration, offline vs online, vault token, 3DS
- Shipping carrier: AbstractCarrier, collectRates, tracking, multi-warehouse
- Email template: transactional email, variables, transport, inline CSS
- Import/Export: ImportModel, CSV format, behavior (add/update/delete/replace)
- Widget: WidgetInterface, instance types, layout XML integration
- Custom product type: TypeInterface, price model, stock handling
- Extension Attributes: ExtensionAttributesInterface, join directive, search
- Staging & Preview (Commerce): version, update, campaign (nếu dùng Adobe Commerce)
- B2B: company, shared catalog, negotiable quote, purchase order (nếu dùng B2B)
- PWA/Headless: GraphQL coverage, storefront compatibility, CORS
- Adobe I/O Events: webhook, event provider, Commerce eventing
- API Mesh: resolver composition, transform, rate limiting
- Composer: module versioning, metapackage, patches (cweagans)
- Testing: PHPUnit mock, integration test fixtures, MFTF basics
- Upgrade: upgrade_compatibility_tool, breaking changes, deprecation
- CSP: Content Security Policy, whitelist, nonce, report-only mode
- Two-Factor Authentication: provider, config, bypass for API
- Rate Limiting & Throttling: API rate limit, DDoS protection
- Redis: session storage, cache backend, sentinel, cluster config
- Elasticsearch/OpenSearch: index mapping, analyzer, relevance tuning

### Ví dụ blueprint còn thiếu
- Blueprint: Custom shipping carrier (AbstractCarrier đầy đủ)
- Blueprint: Custom payment method (offline)
- Blueprint: Import CSV custom entity
- Blueprint: Custom product type
- Blueprint: Extension attributes với join
- Blueprint: Transactional email template
- Blueprint: Custom widget
- Blueprint: Integration test module
- Blueprint: Console command với progress bar
- Blueprint: Custom price modifier (catalog rule)
