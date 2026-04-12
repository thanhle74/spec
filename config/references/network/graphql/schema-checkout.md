# GraphQL — Schema (Guide): Checkout (queries)

Tóm tắt nhóm **Checkout** trên Adobe: **queries** dùng khi hoàn tất checkout (điều khoản, token thanh toán đã lưu, URL PayPal hosted / Payflow Link). **Mutations** checkout (Braintree, Klarna, PayPal, …) nằm cùng nhóm trên sidebar — trong file này chỉ **queries** §3–§6; mục lục mutations → [Checkout](https://developer.adobe.com/commerce/webapi/graphql/schema/checkout/). Chi tiết field: từng link **Nguồn**.  
**Chữ ký theo bản:** [GraphQL API reference](https://developer.adobe.com/commerce/webapi/graphql/reference/) — xem [`reference.md`](./reference.md).

---

## 1. Checkout (nhóm schema)

Nguồn: [Checkout](https://developer.adobe.com/commerce/webapi/graphql/schema/checkout/)

- Schema **Checkout** hỗ trợ hoàn tất quy trình thanh toán: lấy / quản lý **payment token**, **checkout agreements**, và luồng tích hợp **PayPal** (từng phương thức có workflow riêng — xem doc “Core payment methods” trên Adobe).
- Cài **Payment Services** (≥ 2.3.0) có thêm query checkout (vd. `getPaymentConfig`, `getPaymentOrder`, `getPaymentSDK`) — không nằm trong các mục §3–§6; xem [Checkout queries](https://developer.adobe.com/commerce/webapi/graphql/schema/checkout/queries/) và tài liệu Payment Services.

---

## 2. Checkout — danh sách queries (mục lục)

Nguồn: [Checkout queries](https://developer.adobe.com/commerce/webapi/graphql/schema/checkout/queries/)

Tóm tắt theo Adobe: mỗi query có **phạm vi hẹp** — chỉ dùng khi bật điều kiện tương ứng (Terms & Conditions, Vault, hoặc đúng cổng PayPal).

| Query | Nguồn Adobe | Trong file này |
|-------|-------------|----------------|
| `checkoutAgreements` | [checkoutAgreements](https://developer.adobe.com/commerce/webapi/graphql/schema/checkout/queries/agreements/) | §3 |
| `customerPaymentTokens` | [customerPaymentTokens](https://developer.adobe.com/commerce/webapi/graphql/schema/checkout/queries/customer-payment-tokens/) | §4 |
| `getHostedProUrl` | [getHostedProUrl](https://developer.adobe.com/commerce/webapi/graphql/schema/checkout/queries/get-hosted-pro-url/) | §5 |
| `getPayflowLinkToken` | [getPayflowLinkToken](https://developer.adobe.com/commerce/webapi/graphql/schema/checkout/queries/get-payflow-link-token/) | §6 |

---

## 3. `checkoutAgreements`

Nguồn: [checkoutAgreements](https://developer.adobe.com/commerce/webapi/graphql/schema/checkout/queries/agreements/)

- Lấy **checkout agreements** (điều khoản). Nếu **Enable Terms and Conditions** = **No** trong Admin, query **luôn trả mảng rỗng** — config `checkout/options/enable_agreements` (theo Adobe).
- Trường **`content`** có thể là HTML hoặc text thuần; dùng **`is_html`** để phân biệt.
- Cú pháp (theo doc): `checkoutAgreements: [CheckoutAgreement]`

---

## 4. `customerPaymentTokens`

Nguồn: [customerPaymentTokens](https://developer.adobe.com/commerce/webapi/graphql/schema/checkout/queries/customer-payment-tokens/)

- Khi tích hợp hỗ trợ **vault** và đã bật, khách có thể **lưu thẻ**; lần sau xem danh sách phương thức đã lưu (vd. **Braintree**; bên thứ ba có thể hỗ trợ tương tự).
- Trả về mảng token đã lưu; xoá token bằng mutation **`deletePaymentToken`** (theo Adobe).
- **Bắt buộc** header **customer authorization token** (đã đăng nhập).
- Cú pháp (theo doc): `customerPaymentTokens: CustomerPaymentTokens` (cấu trúc `items`: `details`, `public_hash`, `payment_method_code`, `type`, … — xem Reference).

---

## 5. `getHostedProUrl`

Nguồn: [getHostedProUrl](https://developer.adobe.com/commerce/webapi/graphql/schema/checkout/queries/get-hosted-pro-url/)

- **PaaS only** (badge trên doc). Cần khi chọn **PayPal Website Payments Pro Hosted Solution**: lấy URL do PayPal tạo để PWA/client mở form, khách nhập tài khoản PayPal và hoàn tất giao dịch.
- Gọi **sau** khi đã **set payment method** và **place order** (theo Adobe).
- Cú pháp (theo doc): `getHostedProUrl(input: HostedProUrlInput!): HostedProUrl` — output ví dụ: **`secure_form_url`**.
- Lỗi thường gặp (theo doc): không tìm thấy giỏ, giỏ không active, thiếu `cart_id`.

---

## 6. `getPayflowLinkToken`

Nguồn: [getPayflowLinkToken](https://developer.adobe.com/commerce/webapi/graphql/schema/checkout/queries/get-payflow-link-token/)

- **PaaS only** (badge trên doc). Lấy credential PayPal cho giao dịch **PayPal Payflow Link** (`secure_token`, `secure_token_id`, `mode`, `paypal_url`, …).
- Gọi **sau** khi đã **set payment method** và **place order**; workflow chi tiết: doc **PayPal Payflow Link** (theo Adobe).
- Cú pháp (theo doc): `getPayflowLinkToken(input: PayflowLinkTokenInput): PayflowLinkToken`
- Lỗi (theo doc): ví dụ **No such entity with cartId** khi `cart_id` không hợp lệ.

---

## Liên kết

- Mục lục folder: [`README.md`](./README.md)
- Giỏ & đặt hàng (core): [`schema-cart.md`](./schema-cart.md)
- Tutorial: [GraphQL checkout tutorial](https://developer.adobe.com/commerce/webapi/graphql/tutorials/checkout/)
