# GraphQL API — Reference (schema theo phiên bản)

Nguồn: [GraphQL API reference](https://developer.adobe.com/commerce/webapi/graphql/reference/)

## Cách dùng trang reference

- Một endpoint GraphQL, toàn bộ **queries / mutations / types** được mô tả theo **phiên bản** (vd. **2.4.8** cho stack PaaS/on-prem trùng `constitution.md`).
- Có bản **SaaS only** — chỉ áp dụng nếu kiến trúc Commerce của bạn là SaaS; không trộn nhầm schema với on-prem.

## Gợi ý tra cứu

- **Cart / checkout / customer / products** — đúng nhóm trong sidebar Schema trên Adobe.
- **B2B** (Company, Negotiable quotes, Requisition lists, …) — chỉ khi bật module B2B.
- **Catalog Service / Live Search** — tích hợp dịch vụ Adobe; đọc điều kiện license & stack.

Chi tiết từng field và đối số: luôn đối chiếu đúng **bản reference** đã chọn.
