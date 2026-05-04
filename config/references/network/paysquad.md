# Paysquad — Group Payment Platform

> Source: https://docs.paysquad.co/reference/introduction  
> Last synced: May 2026

---

## Tổng quan

Paysquad là nền tảng **group-payment**: tại checkout, merchant tạo một Paysquad qua API, redirect Shopper vào hosted flow của Paysquad. Shopper trở thành Squad Leader, chia sẻ link cho bạn bè/gia đình, mọi người đóng góp, Paysquad capture và settle toàn bộ khi đủ tổng tiền.

**Đặc điểm kỹ thuật:**
- REST over HTTPS, JSON request/response
- Authentication: HTTP Basic Auth
- Amounts: luôn dùng **minimum fractional currency units** (ví dụ `22050` = $220.50 NZD)

---

## Environments

| Environment | API Base URL | Hosted Flow | Merchant Dashboard |
|---|---|---|---|
| Production | `https://api.paysquad.co` | `https://app.paysquad.co` | `https://merchant.paysquad.co` |
| Sandbox | `https://api.sandbox.paysquad.co` | `https://app.sandbox.paysquad.co` | `https://merchant.sandbox.paysquad.co` |

- Mỗi environment có API key và webhook signing key riêng biệt, **không dùng lẫn được**.
- Gọi production API với sandbox key → trả về `401`.

---

## Authentication

Mọi request phải có HTTP Basic Auth header:

```
Authorization: Basic {base64(MerchantId:ApiSecretKey)}
```

**Ví dụ:**
```
MerchantId  = 00000000-0000-0000-0000-000000000000
ApiSecretKey = SECRETKEY
→ Authorization: Basic MDAwMDAwMDAtMDAwMC0wMDAwLTAwMDAtMDAwMDAwMDAwMDAwOlNFQ1JFVEtFWQ==
```

**Lấy keys:** Merchant Portal → Developer → API Keys.  
**Rotation:** key cũ bị invalidate ngay lập tức khi rotate — schedule trong low-traffic window.

---

## Payment Process Flow

```
1. Shopper chọn Paysquad tại checkout
2. Merchant server → POST /api/merchant/paysquad → nhận paySquadId + contributionLink
3. Merchant redirect Shopper đến contributionLink
4. Paysquad hosted flow: Squad Leader sign in, đóng góp phần của mình, invite others
5. Contributors đóng góp qua cùng link (mỗi contribution được authorise, chưa charge)
6. Tổng tiền đủ → Paysquad capture tất cả → status = Complete
7. Webhook paysquad.succeeded → Merchant fulfil order
8. Paysquad settle cho Merchant (net of fees) theo payout schedule
```

**Failure paths:**

| Scenario | Status | Reason | Webhook |
|---|---|---|---|
| Không có contribution trước expiry | Failed | Abandoned | `paysquad.failed` |
| Có contribution nhưng chưa đủ tổng trước expiry | Failed | Expired | `paysquad.failed` |
| Squad Leader cancel trong hosted flow | Failed | Cancellation | `paysquad.cancelled` |
| Merchant cancel từ Merchant Portal | Failed | Cancellation | `paysquad.cancelled` |

---

## API Endpoints

### POST /api/merchant/paysquad — Tạo Paysquad mới

**Request body:**

| Field | Type | Required | Mô tả |
|---|---|---|---|
| `items` | array | ✅ | Danh sách items trong basket |
| `currency` | string | ✅ | ISO 4217 currency code (xem Supported Currencies) |
| `total` | integer | ✅ | Tổng tiền, đơn vị minor units. Phải ≥ minimum contribution của currency |
| `metadata` | string | ❌ | Free-form string (≤ 2KB). Dùng để attach order/cart ID của merchant |
| `successRedirectUrl` | string | ❌ | URL "back to shop" sau khi Squad Leader đóng góp lần đầu |
| `cancelRedirectUrl` | string | ❌ | URL redirect nếu Squad Leader cancel |
| `paymentCompleteWebhook` | string | ❌ | Per-Paysquad webhook URL. Nếu set, override Portal-level webhook |
| `enforcedExpiry` | string (UTC) | ❌ | Fixed expiry timestamp. Nếu merchant có max-checkout-duration, lấy giá trị nhỏ hơn |

**Response `201 Created`:**
```json
{
  "paySquadId": "...",
  "contributionLink": "https://app.paysquad.co/flow/{paySquadId}"
}
```

> Dùng `contributionLink` từ response để redirect, không tự construct URL.

---

### GET /api/merchant/paysquad/{paySquadId} — Lấy thông tin Paysquad

Trả về thông tin đầy đủ: status, basket details, anchor info, contributor info.  
Dùng để reconciliation hoặc khi cần xác nhận state sau webhook.

---

### POST /api/merchant/paysquad/refund — Yêu cầu hoàn tiền

**Request body:**

| Field | Type | Required | Mô tả |
|---|---|---|---|
| `paySquadId` | string | ✅ | ID của Paysquad cần refund |
| `reason` | string | ✅ | Mô tả lý do refund |
| `merchantReference` | string | ❌ | Reference number của merchant |

**Constraints:**
- Chỉ refund được Paysquad ở status `Complete`
- Chỉ refund **một lần**, **toàn bộ** (partial refund chưa hỗ trợ)
- Refund được queue, không xử lý ngay — merchant nhận email khi hoàn tất
- Response: `202 Accepted`

---

## Statuses

| Status | Ý nghĩa |
|---|---|
| `Pending` | Đã tạo, chưa có contribution nào |
| `InProgress` | Có ít nhất 1 contribution, chưa đủ tổng |
| `Complete` | Đủ tổng, funds đã captured |
| `Refunded` | Đã hoàn tiền toàn bộ |
| `Failed` | Kết thúc mà không complete — xem `failureReason` |

**failureReason** (khi status = `Failed`):

| Reason | Ý nghĩa |
|---|---|
| `Abandoned` | Không có contribution nào trước expiry |
| `Expired` | Có contribution nhưng chưa đủ tổng trước expiry. Authorisations được release |
| `Cancellation` | Bị cancel bởi Squad Leader, Merchant, hoặc Paysquad support |

**failureSubReason** (optional, set trong cancellation/refund flows):

| Value | Ý nghĩa |
|---|---|
| `Duplicate` | Order bị duplicate |
| `Fraudulent` | Transaction bị flag là gian lận |
| `RequestedByCustomer` | Squad Leader hoặc contributor yêu cầu cancel |
| `Abandoned` | Bị abandon trước khi có contribution |

---

## Webhooks

### Events

| Event | Khi nào fire |
|---|---|
| `paysquad.succeeded` | Paysquad đạt tổng tiền và đã captured |
| `paysquad.cancelled` | Bị cancel bởi Squad Leader, Merchant, hoặc support |
| `paysquad.failed` | Kết thúc ở status Failed (abandoned hoặc expired) |

### Cấu hình endpoint

- **Merchant Portal** (Developer → Webhooks): nhiều endpoint, mỗi endpoint subscribe event riêng, cấu hình riêng cho Sandbox/Production.
- **Per-Paysquad** (`paymentCompleteWebhook` trong Create request): override Portal endpoint cho Paysquad đó. Events không duplicate sang cả hai.

### Request shape

```
POST {webhook_url}
Content-Type: application/json
X-Paysquad-Signature: {HMAC-SHA256 signature, base64-encoded}
X-Paysquad-Environment: Production | Sandbox | Beta
```

Body chứa `paySquadId` và `eventName`.

### Xác thực webhook signature

```php
function validateWebhookAndGetPayload(string $webhookSigningKey): array
{
    $signature = $_SERVER['HTTP_X_PAYSQUAD_SIGNATURE']
        ?? throw new Exception("Missing signature.");

    $rawBody = file_get_contents('php://input');

    // Signing key được base64-decode trước khi dùng
    $decodedKey = base64_decode($webhookSigningKey);

    $expected = base64_encode(hash_hmac('sha256', $rawBody, $decodedKey, true));

    if (!hash_equals($expected, $signature)) {
        throw new Exception("Invalid signature.");
    }

    $payload = json_decode($rawBody, true);

    if (!isset($payload['paySquadId'])) {
        throw new Exception("Invalid payload: missing paySquadId.");
    }

    return $payload;
}
```

### Retry behaviour

- Timeout: 10 giây để trả về HTTP 200
- Retry: tối đa 2 lần nữa (3 attempts tổng), gap tối thiểu 15 phút
- Sau 3 lần thất bại: event bị mark failed, không tự retry — có thể trigger manual retry từ Merchant Portal

### Best practices

- **Luôn verify signature** trước khi xử lý
- **Respond nhanh** (HTTP 200 trước), xử lý async sau
- **Idempotent**: key off `paySquadId` + `eventName` để tránh xử lý duplicate khi retry
- **Fetch để confirm**: webhook chỉ mang `paySquadId` + event type — gọi `GET /api/merchant/paysquad/{id}` để lấy state thực tế trước khi thực hiện business logic
- **Reconcile daily**: nightly job gọi GET để đóng gap nếu webhook bị miss

---

## Error Handling

| Code | Ý nghĩa |
|---|---|
| `200` | OK |
| `201` | Created |
| `202` | Accepted (refund queued) |
| `400` | Bad Request — validation failed, response body có field-level detail |
| `401` | Unauthorized — API key thiếu, sai format, hoặc sai environment |
| `403` | Forbidden — key hợp lệ nhưng không có quyền truy cập resource |
| `404` | Not Found — resource không tồn tại hoặc không thuộc Merchant account |
| `429` | Too Many Requests — rate limit, back off và retry |
| `5xx` | Server error — transient, retry với exponential backoff |

**Error response shape (400):**
```json
{
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "Currency": ["Currency not supported."],
    "Total": ["Total is not greater then a minimum contribution for NZD"]
  }
}
```

- `4xx`: non-retryable, fix request
- `5xx`: retryable với exponential backoff

---

## Supported Currencies

| Code | Minimum contribution (minor units) | Ví dụ |
|---|---|---|
| NZD | 50 | $0.50 |
| AUD | 50 | A$0.50 |
| USD | 50 | $0.50 |
| CAD | 50 | C$0.50 |
| EUR | 50 | €0.50 |
| GBP | 30 | £0.30 |
| SGD | 50 | S$0.50 |
| CHF | 50 | CHF 0.50 |
| JPY | 50 | ¥50 |
| DKK | 250 | kr 2.50 |
| NOK | 300 | kr 3.00 |
| SEK | 300 | kr 3.00 |
| MXN | 1000 | MX$10.00 |
| HKD | 400 | HK$4.00 |

---

## Testing in Sandbox

**Test cards (Stripe test mode):**

| Card number | Behaviour |
|---|---|
| `4242 4242 4242 4242` | Succeeds — happy path |
| `4000 0000 0000 0002` | Card declined |
| `4000 0025 0000 3155` | Requires 3D Secure |
| `4000 0000 0000 9995` | Insufficient funds |

Dùng bất kỳ expiry date tương lai, CVC 3 chữ số, billing postcode bất kỳ.

**Trigger từng webhook event:**

| Event | Cách trigger |
|---|---|
| `paysquad.succeeded` | Complete hosted flow với đủ contributions |
| `paysquad.cancelled` | Vào hosted flow với tư cách Squad Leader, cancel từ trong flow |
| `paysquad.failed (Expired)` | Tạo Paysquad với `enforcedExpiry` ngắn, không đủ tổng trước khi hết hạn |
| `paysquad.failed (Abandoned)` | Tạo Paysquad với `enforcedExpiry` ngắn, không có contribution nào |

**Lưu ý sandbox:**
- Webhook signing key sandbox khác production — dùng đúng key khi validate
- Email notifications vẫn gửi đến địa chỉ thật — dùng test inbox
- Settlement không xảy ra trong sandbox

---

## Handoff / Redirect

Sau khi tạo Paysquad, redirect Shopper đến `contributionLink` từ response.

- `successRedirectUrl`: được offer cho Squad Leader sau khi họ đóng góp lần đầu ("back to shop")
- `cancelRedirectUrl`: redirect nếu Squad Leader cancel

> Cả hai redirect **không mang payment-status information** — chỉ là navigation convenience.  
> **Source of truth**: luôn dùng webhook hoặc `GET /api/merchant/paysquad/{id}`.

---

## Lưu ý tích hợp Magento

- Paysquad là **async payment method**: order được fulfil sau khi nhận `paysquad.succeeded` webhook, không phải ngay tại checkout.
- Cần implement webhook receiver endpoint (controller) để nhận và xử lý events.
- Nên lưu `paySquadId` vào order/quote để mapping khi nhận webhook.
- Dùng `metadata` field để attach Magento order increment ID hoặc quote ID.
- Xử lý webhook phải idempotent — Magento order có thể đã ở trạng thái khác khi webhook retry.
