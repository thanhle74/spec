# Tham khảo: Admin UI Grid

Nguồn: https://developer.adobe.com/commerce/php/development/components/add-admin-grid

---

## 1. Cấu trúc các thành phần

| Thành phần | Đường dẫn mẫu | Vai trò |
|------------|---------------|---------|
| **Layout** | `view/adminhtml/layout/route_index_index.xml` | Nhúng UI Component vào nội dung trang. |
| **UI XML** | `view/adminhtml/ui_component/custom_grid_listing.xml` | Khai báo cột, filter, paging, mass actions. |
| **Provider** | `Ui/DataProvider/ListingDataProvider.php` | Cầu nối cấp dữ liệu cho XML. |
| **Collection**| `Ui/DataProvider/Listing/Collection.php` | Chứa logic truy vấn DB (SQL). |
| **di.xml** | `etc/di.xml` | Khai báo `virtualType` để nối các thành phần. |

---

## 2. Khai báo UI Component trong Layout

```xml
<referenceContainer name="content">
    <uiComponent name="custom_grid_listing" />
</referenceContainer>
```

---

## 3. Cấu hình di.xml (Quan trọng nhất)

Sử dụng `virtualType` để tiết kiệm thời gian viết code.

```xml
<!-- Khai báo Collection riêng cho Grid -->
<virtualType name="MyCustomCollection" type="Magento\Framework\View\Element\UiComponent\DataProvider\SearchResult">
    <arguments>
        <argument name="mainTable" xsi:type="string">my_custom_table</argument>
        <argument name="resourceModel" xsi:type="string">Vendor\Module\Model\ResourceModel\CustomEntity</argument>
    </arguments>
</virtualType>

<!-- Đăng ký Collection vào Provider chung của Magento -->
<type name="Magento\Framework\View\Element\UiComponent\DataProvider\CollectionFactory">
    <arguments>
        <argument name="collections" xsi:type="array">
            <item name="custom_grid_listing_data_source" xsi:type="string">MyCustomCollection</item>
        </argument>
    </arguments>
</type>
```

---

## 4. UI Component XML (Basic)

File: `view/adminhtml/ui_component/custom_grid_listing.xml`

```xml
<listing xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <dataSource name="custom_grid_listing_data_source">
        <argument name="dataProvider" xsi:type="configurableObject">
            <argument name="class" xsi:type="string">Magento\Framework\View\Element\UiComponent\DataProvider\DataProvider</argument>
            <argument name="name" xsi:type="string">custom_grid_listing_data_source</argument>
            <argument name="primaryFieldName" xsi:type="string">entity_id</argument>
            <argument name="requestFieldName" xsi:type="string">id</argument>
        </argument>
    </dataSource>
    <columns name="custom_columns">
        <column name="entity_id">
            <settings>
                <filter>textRange</filter>
                <label translate="true">ID</label>
            </settings>
        </column>
        <column name="title">
            <settings>
                <filter>text</filter>
                <label translate="true">Tiêu đề</label>
            </settings>
        </column>
    </columns>
</listing>
```

---

## 5. Lưu ý khi Debug

- **Cache:** Luôn xóa cache `config` và `page_cache` sau khi sửa UI XML.
- **Data Source Name:** Tên `dataSource` trong XML phải khớp 100% với tên `item` trong `di.xml`.
- **Duplicate ID:** Đảm bảo `entity_id` là duy nhất, nếu không các cột checkbox sẽ hoạt động sai.

---

## Liên kết

- Routing & Controllers: xem [routing-controllers.md](./routing-controllers.md)
- Declarative Schema: xem [declarative-schema.md](./declarative-schema.md)
- Quy tắc chung: xem [../constitution.md](../constitution.md)
