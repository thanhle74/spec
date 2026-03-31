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

## 5. Phân quyền (ACL Resource Accessibility)

Trong `webapi.xml`, mỗi route phải khai báo ít nhất một `<resource>`.

### Các giá trị đặc biệt:
- **`anonymous`**: Cho phép truy cập công khai (Guest). Dùng cho các tính năng như Đăng ký khách hàng, Xem sản phẩm.
- **`self`**: Cho phép khách hàng đã đăng nhập truy cập dữ liệu của chính mình.
- **`Magento_Module::resource`**: Chỉ cho phép Admin hoặc Integration có quyền tương ứng trong `acl.xml`.

### Ví dụ phân quyền nâng cao:
```xml
<!-- Cho khách hàng tự cập nhật profile của mình -->
<route url="/V1/customers/me" method="PUT">
    <service class="Magento\Customer\Api\CustomerRepositoryInterface" method="save"/>
    <resources>
        <resource ref="self"/>
    </resources>
    <data>
        <parameter name="customer.id" force="true">%customer_id%</parameter>
    </data>
</route>

<!-- Cho Admin có quyền manage mới được sửa thông tin khách hàng bất kỳ -->
<route url="/V1/customers/:customerId" method="PUT">
    <service class="Magento\Customer\Api\CustomerRepositoryInterface" method="save"/>
    <resources>
        <resource ref="Magento_Customer::manage"/>
    </resources>
</route>
```

---

## 6. Custom Routes & Aliases (Asynchronous API)

Bạn có thể đặt bí danh cho các route để URL ngắn gọn và dễ hiểu hơn thông qua file `etc/webapi_async.xml`.

### Quy tắc:
- Chỉ áp dụng cho các route **không có tham số biến** (ví dụ: không có `:id`).
- Giúp che giấu cấu trúc URL thực tế của hệ thống.

### Ví dụ:
```xml
<services xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_WebapiAsync:etc/webapi_async.xsd">
    <!-- Gọi POST /createWidget sẽ tương đương POST /V1/widgets -->
    <route url="V1/widgets" method="POST" alias="createWidget" />
</services>
```

---

## 7. Request Processors (Sync vs Async vs Bulk)

Magento phân loại luồng xử lý API dựa trên URL. Hiểu điều này giúp tối ưu hiệu suất tích hợp.

| Loại | URL Pattern | Cơ chế | Sử dụng khi |
|------|-------------|--------|-------------|
| **Sync** | `/V1/...` | Chạy trực tiếp, trả kết quả ngay. | Thao tác đơn lẻ, cần kết quả tức thì. |
| **Async** | `/async/V1/...` | Đưa vào Queue (RabbitMQ/DB), trả về `accepted`. | Các tác vụ nặng, không cần kết quả ngay. |
| **Async Bulk**| `/async/bulk/V1/...` | Đưa mảng dữ liệu lớn vào Queue. | Tích hợp dữ liệu lớn (Import hàng loạt). |

### Lưu ý:
Để sử dụng **Async/Bulk**, bạn phải bật và cấu hình **Message Queues** (bằng RabbitMQ hoặc Database) trong dự án.

---

## Liên kết

- Service Contracts: xem [service-contracts.md](./service-contracts.md)
- GraphQL: xem [graphql-app-server.md](./graphql-app-server.md)
- Quy tắc chung: xem [../constitution.md](../constitution.md)
