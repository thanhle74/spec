# GraphQL — Web API (Adobe Commerce)

Tài liệu gốc nằm trên **Adobe Developer — Commerce Web APIs → GraphQL**. Trong spec này, toàn bộ mục **Usage** (endpoint, token, cache, filter, response, headers, introspection, protected mutations, security, staging) gom tại [`usage.md`](./usage.md); **Reference** schema theo phiên bản tại [`reference.md`](./reference.md).

Nhánh **Development** (định nghĩa `schema.graphqls`, resolver/batch resolver, mở rộng schema, Identity/cache tag, urlResolver tùy chỉnh, debug, exception, functional test): [`development.md`](./development.md).  
**App Server / resolver stateless** (ràng buộc khi chạy long-lived PHP): [`../graphql-app-server.md`](../graphql-app-server.md) — bổ sung cho doc Adobe, không thay thế.

---

## Phiên bản (align với project)

- Constitution: **Magento 2.4.8** → khi tra **GraphQL API reference**, chọn bản **2.4.8** (PaaS / Magento Open Source / Adobe Commerce on-prem), không nhầm với chỉ **SaaS** nếu bạn không dùng stack đó.

---

## Mục lục nhanh

| Chủ đề | Adobe (chính thức) | Trong `.spec` |
|--------|-------------------|----------------|
| Schema reference (queries/mutations theo bản) | [GraphQL API reference](https://developer.adobe.com/commerce/webapi/graphql/reference/) | [`reference.md`](./reference.md) |
| Chạy query/mutation, endpoint, usage tổng quan | [GraphQL — Usage](https://developer.adobe.com/commerce/webapi/graphql/usage/) | [`usage.md`](./usage.md) §1 |
| Authorization / token | [Authorization](https://developer.adobe.com/commerce/webapi/graphql/usage/authorization-tokens/) | [`usage.md`](./usage.md) §2 |
| Caching | [Caching](https://developer.adobe.com/commerce/webapi/graphql/usage/caching/) | [`usage.md`](./usage.md) §3 |
| Lọc theo custom attributes | [Custom filters](https://developer.adobe.com/commerce/webapi/graphql/usage/custom-filters/) | [`usage.md`](./usage.md) §4 |
| HTTP status & payload lỗi | [Status codes and responses](https://developer.adobe.com/commerce/webapi/graphql/usage/api-response/) | [`usage.md`](./usage.md) §5 |
| Request headers (Store, Currency, Preview, Captcha, …) | [Headers](https://developer.adobe.com/commerce/webapi/graphql/usage/headers/) | [`usage.md`](./usage.md) §6 |
| Introspection (`__schema`, tắt trên production) | [Introspection queries](https://developer.adobe.com/commerce/webapi/graphql/usage/introspection-queries/) | [`usage.md`](./usage.md) §7 |
| Mutation cần CAPTCHA / reCAPTCHA | [Protected mutations](https://developer.adobe.com/commerce/webapi/graphql/usage/protected-mutations/) | [`usage.md`](./usage.md) §8 |
| Giới hạn độ phức tạp/độ sâu query | [Security configuration](https://developer.adobe.com/commerce/webapi/graphql/usage/security-configuration/) | [`usage.md`](./usage.md) §9 |
| Preview nội dung staging (Adobe Commerce) | [Staging queries](https://developer.adobe.com/commerce/webapi/graphql/usage/staging-queries/) | [`usage.md`](./usage.md) §10 |

### Development (schema, resolver, mở rộng)

| Chủ đề | Adobe (chính thức) | Trong `.spec` |
|--------|-------------------|----------------|
| Định nghĩa schema module (`schema.graphqls`, query/mutation/enum, `@doc`, caching) | [Define schema](https://developer.adobe.com/commerce/webapi/graphql/develop/) | [`development.md`](./development.md) §1 |
| Resolvers (`ResolverInterface`, batch) | [Resolvers](https://developer.adobe.com/commerce/webapi/graphql/develop/resolvers/) | [`development.md`](./development.md) §2 |
| Mở rộng schema (stitching, plugin, …) | [Extend existing schema](https://developer.adobe.com/commerce/webapi/graphql/develop/extend-existing-schema/) | [`development.md`](./development.md) §3 |
| Identity / cache tags (`IdentityInterface`, `@cache`) | [Identity class](https://developer.adobe.com/commerce/webapi/graphql/develop/identity-class/) | [`development.md`](./development.md) §4 |
| `urlResolver` + URL rewrite | [Custom urlResolver](https://developer.adobe.com/commerce/webapi/graphql/develop/create-custom-url-resolver/) | [`development.md`](./development.md) §5 |
| Debug (Xdebug, PhpStorm) | [Debugging](https://developer.adobe.com/commerce/webapi/graphql/develop/debugging/) | [`development.md`](./development.md) §6 |
| Exception (`ClientAware`, `GraphQl*Exception`) | [Exceptions](https://developer.adobe.com/commerce/webapi/graphql/develop/exceptions/) | [`development.md`](./development.md) §7 |
| Functional testing | [Functional testing](https://developer.adobe.com/commerce/webapi/graphql/develop/functional-testing/) | [`development.md`](./development.md) §8 |

---

## Liên kết

- [GraphQL Guide (Adobe)](https://developer.adobe.com/commerce/webapi/graphql/) — cổng tổng
- Development (tóm tắt Adobe): [`development.md`](./development.md)
- App Server & resolver stateless: [`../graphql-app-server.md`](../graphql-app-server.md)
- REST/SOAP Web API: [`../web-api.md`](../web-api.md)
