# GraphQL — Schema (Guide): Customer (queries)

Tóm tắt nhóm **Customer** trên Adobe: đây là nhóm query lấy dữ liệu khách hàng đăng nhập (profile, cart, đơn hàng, downloadable products, gift card, customer group/segments...). File này tổng hợp phần **queries** theo danh sách Adobe bạn yêu cầu. Chi tiết field: từng link **Nguồn**.  
**Chữ ký theo bản:** [GraphQL API reference](https://developer.adobe.com/commerce/webapi/graphql/reference/) — xem [`reference.md`](./reference.md).

---

## 1. Customer (nhóm schema)

Nguồn: [Customer](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/)

- Customer là shopper đã có account trên store.
- Adobe khuyến nghị dùng customer token ở header cho các GraphQL call liên quan customer (cũng có thể dùng session auth trong một số luồng).
- Với B2B Adobe Commerce, object `Customer` có thêm field top-level như `job_title`, `role`, `status`, `team`, `telephone`, `requisition_lists`.

---

## 2. Customer — danh sách queries (mục lục)

Nguồn: [Customer queries](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/)

| Query | Nguồn Adobe | Trong file này |
|-------|-------------|----------------|
| `customer` | [customer](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/customer/) | §3 |
| `customerCart` | [customerCart](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/cart/) | §4 |
| `customerDownloadableProducts` | [customerDownloadableProducts](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/downloadable-products/) | §5 |
| `customerGroup` | [customerGroup](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/customer-group/) | §6 |
| `customerOrders` | [customerOrders](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/orders/) | §7 |
| `customerSegments` | [customerSegments](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/customer-segments/) | §8 |
| `giftCardAccount` | [giftCardAccount](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/giftcard-account/) | §9 |
| `isEmailAvailable` | [isEmailAvailable](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/is-email-available/) | §10 |

---

## 3. `customer`

Nguồn: [customer](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/customer/)

- Trả thông tin customer đăng nhập: profile, address, custom attributes, orders summary (qua `customer.orders`), wishlist, store credit, và các field mở rộng khác theo module.
- Adobe khuyến nghị gửi customer token ở header.
- Cú pháp (theo doc): `customer: Customer`

---

## 4. `customerCart`

Nguồn: [customerCart](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/cart/)

- Trả active cart của customer đăng nhập; nếu chưa có cart thì query sẽ tạo mới.
- Không nhận input `cart_id` (khác với query `cart`).
- Có thể lấy `id` cart để dùng trong `mergeCarts` khi hợp nhất guest cart vào customer cart.
- Cú pháp (theo doc): `customerCart: Cart!`
- Yêu cầu customer authorization token.

---

## 5. `customerDownloadableProducts`

Nguồn: [customerDownloadableProducts](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/downloadable-products/)

- **PaaS only**.
- Trả danh sách downloadable products customer đã mua.
- Cú pháp (theo doc): `customerDownloadableProducts: CustomerDownloadableProducts`
- Lỗi thường gặp: `The current customer isn't authorized.`

---

## 6. `customerGroup`

Nguồn: [customerGroup](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/customer-group/)

- Query thuộc **Storefront Compatibility Package**, dự kiến thêm vào Adobe Commerce **2.4.9**.
- Trả encoded `uid` của customer group cho logged-in customer hoặc guest.
- Cú pháp (theo doc): `customerGroup: CustomerGroupStorefront!`
- Với logged-in customer nên gửi token; với guest thì không truyền token.

---

## 7. `customerOrders`

Nguồn: [customerOrders](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/orders/)

- **PaaS only**.
- Query này đã **deprecated**; Adobe khuyến nghị dùng `customer.orders` thay thế.
- Cú pháp (theo doc): `customerOrders: CustomerOrders`
- Lỗi thường gặp: `The current customer isn't authorized.`

---

## 8. `customerSegments`

Nguồn: [customerSegments](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/customer-segments/)

- Query thuộc **Storefront Compatibility Package**, sẽ có trong Adobe Commerce **2.4.9**.
- Trả encoded `uid` của customer segments gán cho logged-in customer hoặc guest.
- Cú pháp (theo doc): `customerSegments(cartId: String!): [CustomerSegmentStorefront!]`
- Với logged-in customer nên gửi token; guest thì không truyền token.

---

## 9. `giftCardAccount`

Nguồn: [giftCardAccount](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/giftcard-account/)

- Trả thông tin một gift card cụ thể (code, balance, expiration_date...).
- Cú pháp (theo doc): `giftCardAccount(code: String!): GiftCardAccount`  
  (Doc ví dụ dùng input object `gift_card_code`; khi triển khai thực tế nên kiểm tra reference theo version đang chạy).
- Lỗi thường gặp: gift card không tồn tại/đã redeem hết, hoặc thiếu `gift_card_code`.

---

## 10. `isEmailAvailable`

Nguồn: [isEmailAvailable](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/is-email-available/)

- Kiểm tra email đã được dùng để tạo account customer chưa.
- Adobe lưu ý thay đổi từ **2.4.7**: mặc định query luôn trả `true`; hành vi cũ có thể bật lại qua Admin (`Enable Guest Checkout Login`), nhưng có rủi ro lộ thông tin cho user chưa auth.
- Cú pháp (theo doc): `isEmailAvailable(email: String!): IsEmailAvailableOutput`
- Lỗi thường gặp: email format sai, hoặc thiếu argument `email`.

---

## Liên kết

- Mục lục folder: [`README.md`](./README.md)
- Cart: [`schema-cart.md`](./schema-cart.md)
- Checkout: [`schema-checkout.md`](./schema-checkout.md)
- Company (B2B): [`schema-company.md`](./schema-company.md)
