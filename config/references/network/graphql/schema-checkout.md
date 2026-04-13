# GraphQL — Schema (Guide): Checkout (queries + mutations)

Tóm tắt nhóm **Checkout** trên Adobe: **queries** dùng khi hoàn tất checkout (điều khoản, token thanh toán đã lưu, URL PayPal hosted / Payflow Link), còn **mutations** dùng để tạo token/payment session và xử lý phản hồi từ cổng thanh toán. File này tổng hợp **queries** §3–§6 và **mutations** §8–§13. Chi tiết field: từng link **Nguồn**.  
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

## 7. Checkout — danh sách mutations (mục lục)

Nguồn: [Checkout mutations](https://developer.adobe.com/commerce/webapi/graphql/schema/checkout/mutations/)

Tóm tắt theo Adobe: nhóm này chủ yếu phục vụ các luồng thanh toán tích hợp cổng ngoài (Braintree, Klarna, PayPal family) và thao tác token đã lưu.

| Mutation | Nguồn Adobe | Trong file này |
|----------|-------------|----------------|
| `createBraintreeClientToken` | [createBraintreeClientToken](https://developer.adobe.com/commerce/webapi/graphql/schema/checkout/mutations/create-braintree-client-token/) | §8 |
| `createKlarnaPaymentsSession` | [createKlarnaPaymentsSession](https://developer.adobe.com/commerce/webapi/graphql/schema/checkout/mutations/create-klarna-payments-session/) | §9 |
| `createPayflowProToken` | [createPayflowProToken](https://developer.adobe.com/commerce/webapi/graphql/schema/checkout/mutations/create-payflow-pro-token/) | §10 |
| `createPaypalExpressToken` | [createPaypalExpressToken](https://developer.adobe.com/commerce/webapi/graphql/schema/checkout/mutations/create-paypal-express-token/) | §11 |
| `deletePaymentToken` | [deletePaymentToken](https://developer.adobe.com/commerce/webapi/graphql/schema/checkout/mutations/delete-payment-token/) | §12 |
| `handlePayflowProResponse` | [handlePayflowProResponse](https://developer.adobe.com/commerce/webapi/graphql/schema/checkout/mutations/handle-payflow-pro-response/) | §13 |

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

## 8. `createBraintreeClientToken`

Nguồn: [createBraintreeClientToken](https://developer.adobe.com/commerce/webapi/graphql/schema/checkout/mutations/create-braintree-client-token/)

- Tạo **client token** để khởi tạo Braintree JavaScript SDK ở frontend.
- Cú pháp (theo doc): `createBraintreeClientToken: String`
- Lỗi thường gặp: **The Braintree payment method is not active.** (Braintree chưa bật trong Admin).

---

## 9. `createKlarnaPaymentsSession`

Nguồn: [createKlarnaPaymentsSession](https://developer.adobe.com/commerce/webapi/graphql/schema/checkout/mutations/create-klarna-payments-session/)

- **PaaS only** và Adobe ghi chú rõ: Klarna Vendor Bundled Extension đã bị gỡ từ **2.4.4**, vì vậy mutation này **không còn được hỗ trợ** trong codebase hiện tại.
- Khi còn dùng được ở hệ cũ, mutation trả `client_token` và danh sách `payment_method_categories`; khi set payment method cần dùng `payment_method.code = klarna_<identifier>`.
- Nên gọi lại mutation nếu cart thay đổi (sản phẩm/địa chỉ/ship/coupon/tổng tiền) để đồng bộ phương thức Klarna khả dụng.
- Cú pháp (theo doc): `createKlarnaPaymentsSession(input: createKlarnaPaymentsSessionInput!): createKlarnaPaymentsSessionOutput`

---

## 10. `createPayflowProToken`

Nguồn: [createPayflowProToken](https://developer.adobe.com/commerce/webapi/graphql/schema/checkout/mutations/create-payflow-pro-token/)

- **PaaS only**. Khởi tạo giao dịch Payflow Pro và nhận token (`secure_token`, `secure_token_id`, `result`, `response_message`).
- Input bắt buộc `cart_id` và bộ URL (`return_url`, `cancel_url`, `error_url`) trong `urls`.
- Adobe mô tả URL là **relative URL**; ứng dụng sẽ prepend base URL để tạo full callback URL.
- Cú pháp (theo doc): `createPayflowProToken(input: PayflowProTokenInput): CreatePayflowProTokenOutput`

---

## 11. `createPaypalExpressToken`

Nguồn: [createPaypalExpressToken](https://developer.adobe.com/commerce/webapi/graphql/schema/checkout/mutations/create-paypal-express-token/)

- **PaaS only**. Bắt đầu luồng authorize cho các phương thức Express Checkout (PayPal Express, Payflow Pro/Link with Express Checkout).
- Input gồm `cart_id`, `code`, cờ `express_button`, và `urls` (`return_url`, `cancel_url`) dạng relative URL.
- Response trả `token` và `paypal_urls` (`start`, `edit`); token này sẽ dùng ở bước `setPaymentMethodOnCart`.
- Cú pháp (theo doc): `createPaypalExpressToken(input: PaypalExpressTokenInput!): PaypalExpressTokenOutput`

---

## 12. `deletePaymentToken`

Nguồn: [deletePaymentToken](https://developer.adobe.com/commerce/webapi/graphql/schema/checkout/mutations/delete-payment-token/)

- Xoá payment token đã lưu của customer (vault) dựa trên `public_hash`.
- **Bắt buộc** customer authorization token ở header; thường lấy danh sách trước bằng query `customerPaymentTokens`.
- Cú pháp (theo doc): `deletePaymentToken(public_hash: String!): DeletePaymentTokenOutput`
- Lỗi thường gặp: không tìm thấy token theo `public_hash`, hoặc customer chưa được authorize.

---

## 13. `handlePayflowProResponse`

Nguồn: [handlePayflowProResponse](https://developer.adobe.com/commerce/webapi/graphql/schema/checkout/mutations/handle-payflow-pro-response/)

- **PaaS only**. Nhận payload silent post từ Payflow Pro gateway và gửi về application server để hoàn tất cập nhật trạng thái thanh toán lên cart.
- Input gồm `cart_id` và chuỗi `paypal_payload` (querystring payload từ gateway; nội dung thay đổi theo merchant/đơn hàng/địa chỉ).
- Cú pháp (theo doc): `handlePayflowProResponse(input: PayflowProResponseInput!): PayflowProResponseOutput`
- Kết quả thường trả cart đã chọn phương thức thanh toán `payflowpro`.

---

## Liên kết

- Mục lục folder: [`README.md`](./README.md)
- Giỏ & đặt hàng (core): [`schema-cart.md`](./schema-cart.md)
- Checkout mutations (Adobe): [Checkout mutations](https://developer.adobe.com/commerce/webapi/graphql/schema/checkout/mutations/)
- Tutorial: [GraphQL checkout tutorial](https://developer.adobe.com/commerce/webapi/graphql/tutorials/checkout/)
