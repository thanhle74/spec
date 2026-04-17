# Shipping Carrier Pattern

> Áp dụng khi tạo custom shipping carrier kế thừa `Magento\Shipping\Model\Carrier\AbstractCarrier`.

---

## Constructor (Magento 2.4.8)

`AbstractCarrier` yêu cầu đúng 3 dependency sau — không suy đoán, đối chiếu trực tiếp với core:

```php
public function __construct(
    \Magento\Framework\App\Config\ScopeConfigInterface $scopeConfig,
    \Magento\Quote\Model\Quote\Address\RateResult\ErrorFactory $rateErrorFactory,
    \Psr\Log\LoggerInterface $logger,
    // ... dependency riêng của module
) {
    parent::__construct($scopeConfig, $rateErrorFactory, $logger);
}
```

---

## collectRates

- Trả `false` khi carrier không áp dụng (inactive, không có item hợp lệ, v.v.)
- Chỉ append method khi đủ điều kiện nghiệp vụ
- Nếu method chỉ áp dụng subset sản phẩm: phải có guard rõ ràng, không ảnh hưởng carrier khác

```php
public function collectRates(RateRequest $request): bool|\Magento\Shipping\Model\Rate\Result
{
    if (!$this->getConfigFlag('active')) {
        return false;
    }

    // Guard: kiểm tra điều kiện nghiệp vụ
    $applicableItems = $this->getApplicableItems($request->getAllItems());
    if (empty($applicableItems)) {
        return false;
    }

    $result = $this->rateResultFactory->create();
    $method = $this->rateMethodFactory->create();

    $method->setCarrier($this->_code);
    $method->setCarrierTitle($this->getConfigData('title'));
    $method->setMethod($this->_code);
    $method->setMethodTitle($this->getConfigData('name'));
    $method->setPrice($this->calculatePrice($applicableItems));
    $method->setCost($method->getPrice());

    $result->append($method);
    return $result;
}
```

---

## Config chuẩn (`etc/config.xml`)

```xml
<carriers>
    <{carrier_code}>
        <active>0</active>
        <title>My Carrier</title>
        <name>My Shipping Method</name>
        <sort_order>10</sort_order>
        <sallowspecific>0</sallowspecific>
        <model>{Vendor}\{Module}\Model\Carrier\{ClassName}</model>
    </{carrier_code}>
</carriers>
```

---

## Verify bắt buộc

1. `bin/magento setup:di:compile` — bắt lỗi constructor DI sớm
2. Checkout flow: method hiển thị đúng khi đủ điều kiện
3. Checkout flow: method ẩn khi tắt config hoặc không có item hợp lệ
4. Place order: `sales_order.shipping_method` lưu đúng `{carrier_code}_{method_code}`

---

## Liên kết

- Patterns index: [../../magento-patterns.md](../../magento-patterns.md)
- Config paths: [../ops/config-paths.md](../ops/config-paths.md)
