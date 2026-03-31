# Tham khảo: Nguyên tắc bảo mật (Security Best Practices)

Nguồn: https://developer.adobe.com/commerce/php/development/security/

---

## 1. XSS Prevention (Chống chèn mã độc)

**QUY TẮC:** Mọi dữ liệu hiển thị ra Template (.phtml) ĐỀU PHẢI được xử lý qua Escaper.

```php
// Đúng:
<div class="name"><?= $escaper->escapeHtml($user->getName()) ?></div>
<a href="<?= $escaper->escapeUrl($link) ?>">Link</a>
<input value="<?= $escaper->escapeHtmlAttr($val) ?>" />

// Sai:
<div><?= $user->getName() ?></div>
```

---

## 2. Content Security Policy (CSP)

Dùng để cho phép các script/style từ bên thứ ba (CDN, Google, Facebook).

**`etc/csp_whitelist.xml`:**
```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Csp:etc/csp_whitelist.xsd">
    <policy id="script-src">
        <values>
            <value id="google-analytics" type="host">*.google-analytics.com</value>
        </values>
    </policy>
</config>
```

---

## 3. Form Keys (Chống CSRF)

Mọi form thực hiện hành động (POST) trong Magento đều phải có Form Key.

```html
<form action="..." method="post">
    <?= $block->getBlockHtml('formkey') ?>
    <input type="text" name="name" />
    <button type="submit">Gửi</button>
</form>
```

---

## 4. Bảo mật Admin (ACL)

Mọi menu và controller trong Admin đều phải được gán tài nguyên và kiểm tra quyền.

**`etc/acl.xml`:**
```xml
<resource id="Vendor_Module::manage" title="Quản lý Module MyModule" sortOrder="10" />
```

**Controller:**
```php
protected function _isAllowed() {
    return $this->_authorization->isAllowed('Vendor_Module::manage');
}
```

---

## 5. Mã hóa dữ liệu (Encryption)

Dùng `EncryptorInterface` để mã hóa dữ liệu nhạy cảm trước khi lưu vào Database.

```php
// Mã hóa:
$encrypted = $this->encryptor->encrypt($password);

// Giải mã:
$decrypted = $this->encryptor->decrypt($encrypted);
```

---

## 6. Mass Assignment (Gán dữ liệu hàng loạt)

**CẢNH BÁO:** Tuyệt đối không dùng `$model->setData($request->getParams())`.
Hãy chỉ gán những trường được phép (Whitelist) hoặc dùng Service Contracts (Data Interface).

---

## Liên kết

- Configuration Management: xem [configuration-management.md](./configuration-management.md)
- Web API Auth: xem [web-api.md](./web-api.md)
- Quy tắc chung: xem [../constitution.md](../constitution.md)
