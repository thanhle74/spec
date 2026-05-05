# Plan - paysquad-express-checkout

> Technical design only. Business context → `spec.md`.

## Mục tiêu kỹ thuật
- Thêm nút "Buy Now with PaySquad" vào minicart Mageplaza QuickCart (KO template)
- AJAX endpoint set session flag `paysquad_express` trong `Magento\Checkout\Model\Session`
- Plugin `afterGetActiveQuoteMethods` filter payment methods khi flag active
- Mở rộng `ConfigProvider` hiện tại để expose `isExpressCheckout` cho JS auto-select tại OSC
- Plugin `Block\Cart\Sidebar::afterGetConfig` inject `isPaySquadExpressEnabled` vào `window.checkout`
- Observer clear flag sau order success và navigate away

## Blueprint tham khảo
- `examples/integration/custom-payment-offline-blueprint.md` — tham khảo pattern plugin + observer

## References sử dụng
- `config/references/core/events-observers.md` — Observer pattern cho clear flag
- `config/references/core/plugins.md` — Plugin `afterGetActiveQuoteMethods` + `afterGetConfig`
- `config/references/frontend/requirejs-knockoutjs.md` — KO component cho minicart button
- `config/references/ops/config-paths.md` — Thêm config `express_checkout_enabled`

## Thiết kế

### 1. Admin Config

Thêm vào `etc/config.xml`:
```xml
<express_checkout_enabled>0</express_checkout_enabled>
<sort_order>0</sort_order>
<min_order_total/>
<max_order_total/>
<allowspecific>0</allowspecific>
```

Thêm vào `etc/adminhtml/system.xml` (trong group `laybyland_paysquad`):
- Express Checkout Button (yes/no), Sort Order, Min/Max Order Total
- New Order Status (source: `Laybyland\PaySquad\Model\Config\Source\Order\Status\PendingPayment`)
- Payment from Applicable Countries, Payment from Specific Countries

Thêm method vào `Gateway/Config/Config.php`:
```php
public function isExpressCheckoutEnabled(): bool
```

### 2. AJAX Endpoint — Set Flag

`Controller/Express/Set.php` — POST `/paysquad/express/set`:
- Validate form key (CSRF)
- Check cart không trống
- Set `$checkoutSession->setData('paysquad_express', 1)`
- Return JSON `{"success": true}` hoặc `{"success": false, "message": "..."}`

Route khai báo trong `etc/frontend/routes.xml` (đã có, dùng frontName `paysquad`).

### 3. Minicart Button — Mageplaza QuickCart

**Quan trọng:** Minicart dùng Mageplaza QuickCart với KO template riêng — không dùng PHP Block + layout XML.

**Approach thực tế:**
- Thêm button trực tiếp vào `app/design/frontend/Laybyland/layup/Mageplaza_QuickCart/web/template/minicart/content.html`
- Thêm `isPaySquadExpressEnabled` observable và `proceedWithPaySquad` method vào `app/design/frontend/Laybyland/layup/Mageplaza_QuickCart/web/js/view/minicart.js`
- `checkoutConfig` trong minicart.js = `window.checkout` (không phải `window.checkoutConfig`)

**Plugin inject config vào `window.checkout`:**
`Plugin/Block/Cart/Sidebar.php` — `afterGetConfig` trên `Magento\Checkout\Block\Cart\Sidebar`:
```php
$result['isPaySquadExpressEnabled'] = $this->config->isActive()
    && $this->config->isExpressCheckoutEnabled();
```

**KO observable trong minicart.js:**
```js
this.isPaySquadExpressEnabled = ko.observable(!!checkoutConfig?.isPaySquadExpressEnabled);
```

**Method `proceedWithPaySquad`:**
```js
proceedWithPaySquad: function () {
    $.ajax({ url: BASE_URL + 'paysquad/express/set', type: 'POST', ... })
        .then(() => location.href = BASE_URL + 'onestepcheckout');
}
```

> `Block/Minicart/Button.php` và `Observer/AddExpressShortcut.php` tồn tại nhưng không được dùng cho minicart QuickCart.

### 4. Plugin — Filter Payment Methods

`Plugin/ExpressPaymentFilter.php` — `afterGetActiveQuoteMethods` trên `Magento\Payment\Model\MethodList`:
- Flag active → filter chỉ giữ `laybyland_paysquad`
- Flag không active → return nguyên vẹn

OSC gọi `PaymentMethodManagementInterface::getList()` → bên trong gọi `MethodList::getActiveQuoteMethods()` → plugin intercept được.

### 5. ConfigProvider — Expose isExpressCheckout

`Model/Ui/ConfigProvider.php` inject `Magento\Checkout\Model\Session`:
```php
'isExpressCheckoutEnabled' => $this->config->isExpressCheckoutEnabled(),
'isExpressCheckout'        => (bool) $this->checkoutSession->getData('paysquad_express'),
```

JS component `view/frontend/web/js/view/payment/method-renderer/paysquad.js` đọc `isExpressCheckout` và gọi `selectPaymentMethod()` trong `initialize()`.

### 6. Clear Flag

**Observer/ClearExpressFlagOnSuccess.php** — event `checkout_onepage_controller_success_action`

**Observer/ClearExpressFlagOnNavigate.php** — event `controller_action_predispatch`:
- Check `$request->getModuleName() !== 'onestepcheckout' && !$request->isAjax()`

### 7. Data Flow

```
Minicart (QuickCart KO template) → click "Buy Now with PaySquad"
  → proceedWithPaySquad() → AJAX POST /paysquad/express/set
    → validate cart not empty
    → checkoutSession->setData('paysquad_express', 1)
    → return {"success": true}
  → JS redirect → /onestepcheckout

OSC load:
  → ConfigProvider::getConfig() → isExpressCheckout: true
  → Plugin::afterGetActiveQuoteMethods() → filter → chỉ laybyland_paysquad
  → JS paysquad.js initialize() → selectPaymentMethod()
  → Customer thấy chỉ PaySquad, đã được select

Place Order:
  → order success → Observer clear flag
  → hoặc navigate away → Observer clear flag
```

## Files dự kiến

| File | Mục đích |
|---|---|
| `etc/config.xml` | Default values cho tất cả config fields |
| `etc/adminhtml/system.xml` | Admin config fields đầy đủ |
| `etc/di.xml` | Plugin ExpressPaymentFilter + Sidebar |
| `etc/events.xml` | 2 observers clear flag + AddExpressShortcut |
| `Gateway/Config/Config.php` | `isExpressCheckoutEnabled()` |
| `Model/Ui/ConfigProvider.php` | `isExpressCheckout` + `isExpressCheckoutEnabled` |
| `Model/Config/Source/Order/Status/PendingPayment.php` | Source model cho New Order Status |
| `Controller/Express/Set.php` | AJAX endpoint set flag |
| `Plugin/ExpressPaymentFilter.php` | Filter payment methods |
| `Plugin/Block/Cart/Sidebar.php` | Inject `isPaySquadExpressEnabled` vào `window.checkout` |
| `Observer/ClearExpressFlagOnSuccess.php` | Clear flag sau order success |
| `Observer/ClearExpressFlagOnNavigate.php` | Clear flag khi navigate away |
| `Observer/AddExpressShortcut.php` | (Không dùng cho QuickCart) |
| `Block/Minicart/Button.php` | (Không dùng cho QuickCart) |
| `view/frontend/templates/minicart/button.phtml` | (Không dùng cho QuickCart) |
| `view/frontend/web/js/view/payment/method-renderer/paysquad.js` | Auto-select logic |
| `app/design/.../Mageplaza_QuickCart/web/template/minicart/content.html` | Thêm button vào KO template |
| `app/design/.../Mageplaza_QuickCart/web/js/view/minicart.js` | KO observable + proceedWithPaySquad |

## Risks + rollback

- Risk: `controller_action_predispatch` fire cho cả AJAX calls của OSC → clear flag sớm
  - Mitigation: check `$request->isAjax()` trước khi clear

- Risk: Customer mở 2 tab — tab 1 express checkout, tab 2 checkout thường → flag ảnh hưởng cả 2
  - Accepted: behavior của session-based flag, giống PayPal

- Risk: `window.checkout` chưa load khi minicart.js khởi tạo
  - Mitigation: `window.checkout` được inject server-side vào tất cả trang qua `Sidebar::getConfig()`
