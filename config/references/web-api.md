# Tham khảo: Web APIs (REST & SOAP)

Nguồn: https://developer.adobe.com/commerce/php/development/components/web-api/services

---

## 1. Cơ chế tự động hóa

Magento tự động tạo API dựa trên **Service Contract** (Interface). Một phương thức trong Interface có tên `save(\Vendor\Module\Api\Data\MyDataInterface $data)` sẽ được biến thành một POST request với body là JSON tương ứng với `MyDataInterface`.

---

## 2. Quy tắc DocBlock (Bắt buộc)

Do Magento dùng Reflection để parse kiểu dữ liệu cho API, docblock của Interface phải cực kỳ chính xác:

- **BẮT BUỘC** dùng Full Namespace cho các class/interface.
- **BẮT BUỘC** định nghĩa rõ kiểu dữ liệu trả về.

```php
/**
 * @param int $id
 * @param \Vendor\Module\Api\Data\MyInterface $data  <-- Phải có đầy đủ namespace
 * @return \Vendor\Module\Api\Data\ResultInterface[] <-- Mảng các object
 */
public function process(int $id, \Vendor\Module\Api\Data\MyInterface $data);
```

---

## 3. Cấu hình `webapi.xml`

Nằm trong thư mục `etc/webapi.xml`.

```xml
<routes xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Webapi:etc/webapi.xsd">
    <route url="/V1/my-feature/:id" method="GET">
        <service class="<Vendor>\<Module>\Api\MyRepositoryInterface" method="getById"/>
        <resources>
            <resource ref="<Vendor>_<Module>::my_resource"/> <!-- ACL Resource -->
        </resources>
    </route>
    
    <!-- Route cho khách hàng tự truy cập thông tin của mình -->
    <route url="/V1/my-feature/me" method="GET">
        <service class="<Vendor>\<Module>\Api\MyManagementInterface" method="getForCustomer"/>
        <resources>
            <resource ref="self"/> <!-- Cho phép user dùng Token cá nhân -->
        </resources>
        <data>
            <!-- Ép tham số customerId lấy từ Token đăng nhập, tránh giả mạo ID -->
            <parameter name="customerId" force="true">%customer_id%</parameter>
        </data>
    </route>
</routes>
```

---

## 4. Bảo mật & Xác thực (Authentication)

1. **Token-based:** Thường dùng cho Mobile App hoặc Integrations.
2. **OAuth:** Dùng cho các ứng dụng bên thứ 3.
3. **Session-based:** Dùng cho AJAX request từ chính Frontend của Magento.

---

## Liên kết

- Service Contracts: xem [service-contracts.md](./service-contracts.md)
- GraphQL: xem [graphql-app-server.md](./graphql-app-server.md)
- Quy tắc chung: xem [../constitution.md](../constitution.md)
