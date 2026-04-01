# Thư viện Linh kiện UI (Common UI Components)

Tài liệu này tổng hợp các linh kiện UI phổ biến được sử dụng trong Admin Panel và các form/grid phức tạp.

---

## 1. Button component

Linh kiện đại diện cho một nút bấm với các tính năng nâng cao như xác nhận (confirmation) và liên kết hành động.

### Cấu hình XML mẫu:
```xml
<button name="save" class="Magento\Catalog\Block\Adminhtml\Category\Edit\SaveButton">
    <settings>
        <label translate="true">Lưu sản phẩm</label>
        <class>primary</class> <!-- CSS class: primary, secondary... -->
        <dataScope>data</dataScope>
    </settings>
</button>
```

### Các tính năng quan trọng:
- **actions**: Định nghĩa các hành động JS sẽ chạy khi click (gọi hàm của component khác).
- **confirm**: Hiển thị popup xác nhận trước khi thực hiện.
```xml
<confirm>
    <message translate="true">Bạn có chắc chắn muốn xóa?</message>
    <title translate="true">Xác nhận xóa</title>
</confirm>
```

---

## 2. ActionsColumn & ActionDelete

Dùng trong Grid (Listing) để hiển thị các liên kết thao tác cho từng dòng dữ liệu.

### Cấu hình XML:
```xml
<actionsColumn name="actions" class="Vendor\Module\Ui\Component\Listing\Column\Actions">
    <settings>
        <indexField>entity_id</indexField>
        <label translate="true">Thao tác</label>
    </settings>
</actionsColumn>
```

### Xử lý PHP (DataProvider):
Class phải kế thừa `Magento\Ui\Component\Listing\Columns\Column` và ghi đè `prepareDataSource`.
```php
public function prepareDataSource(array $dataSource) {
    if (isset($dataSource['data']['items'])) {
        foreach ($dataSource['data']['items'] as & $item) {
            $item[$this->getData('name')] = [
                'edit' => [
                    'href' => $this->urlBuilder->getUrl('route/to/edit', ['id' => $item['entity_id']]),
                    'label' => __('Sửa')
                ],
                'delete' => [
                    'href' => '#',
                    'label' => __('Xóa'),
                    'isAjax' => true,
                    'confirm' => ['title' => __('Xác nhận'), 'message' => __('Xóa dòng này?')]
                ]
            ];
        }
    }
    return $dataSource;
}
```

---

## 3. Bookmarks (Views Management)

Linh kiện cho phép người dùng lưu lại trạng thái của Grid (cột đang ẩn/hiện, filter, thứ tự sắp xếp).

### Vị trí: 
Thường nằm trong `<listingToolbar>`.

### Cấu hình:
```xml
<listingToolbar name="listing_top">
    <bookmarks name="bookmarks"/>
    <!-- ... các linh kiện khác như filters, paging ... -->
</listingToolbar>
```

### Lưu ý:
- Bookmarks yêu cầu `namespace` của listing phải duy nhất để lưu vào database (table `ui_bookmark`).
- Đây là linh kiện "im lặng", nó tự động theo dõi thay đổi của các component khác trong cùng namespace và lưu trạng thái.

---

## 4. Checkbox & Selection Components

Dùng để nhận giá trị Boolean hoặc tập hợp các giá trị được chọn.

### A. Checkbox (Single)
Linh kiện checkbox cơ bản cho Form.
```xml
<field name="is_active" formElement="checkbox">
    <settings>
        <dataType>boolean</dataType>
        <label translate="true">Kích hoạt</label>
    </settings>
    <formElements>
        <checkbox>
            <settings>
                <valueMap>
                    <map name="false" xsi:type="number">0</map>
                    <map name="true" xsi:type="number">1</map>
                </valueMap>
                <prefer>toggle</prefer> <!-- Hiển thị dạng switch/toggle -->
            </settings>
        </checkbox>
    </formElements>
</field>
```

### B. CheckboxSet
Dùng để chọn nhiều giá trị trong một danh sách mảng.
```xml
<field name="category_ids" formElement="checkboxset">
    <settings>
        <label translate="true">Danh mục</label>
        <options class="Magento\Catalog\Model\Product\Option\Source\Category"/>
    </settings>
</field>
```

### C. CheckboxToggleNotice
Checkbox đặc biệt có hiển thị cảnh báo (warning message) khi người dùng thay đổi trạng thái. Thường dùng trong cấu hình hệ thống quan trọng.

---

## 5. Specialized Inputs

### ColorPicker
Linh kiện chọn màu sắc chuyên dụng.
```xml
<field name="bg_color" formElement="colorPicker">
    <settings>
        <label translate="true">Màu nền</label>
        <colorPickerConfig>
            <item name="placeholder" xsi:type="string">#FFFFFF</item>
        </colorPickerConfig>
    </settings>
</field>
```

---

## 6. Grid Columns (Listing)

Hệ thống quản lý cột dữ liệu trong Grid.

### A. Column (Base)
Linh kiện hiển thị dữ liệu của một field. Có thể tùy biến render bằng `bodyTmpl`.

### B. Columns (Container)
Vùng chứa toàn bộ các cột. Đây là nơi cấu hình **Inline Edit** (Sửa trực tiếp trên Grid).

### C. ColumnsEditor (Inline Edit)
Cấu hình bên trong `<columns>` để bật tính năng sửa nhanh.
```xml
<columns name="listing_columns">
    <settings>
        <editorConfig>
            <param name="indexField" xsi:type="string">entity_id</param>
            <param name="enabled" xsi:type="boolean">true</param>
            <param name="selectProvider" xsi:type="string">listing_name.listing_name.listing_top.ids</param>
        </editorConfig>
    </settings>
    <!-- Cột cho phép sửa -->
    <column name="title">
        <settings>
            <editor>
                <editorType>text</editorType>
                <validation>
                    <rule name="required-entry" xsi:type="boolean">true</rule>
                </validation>
            </editor>
            <label translate="true">Tiêu đề</label>
        </settings>
    </column>
</columns>
```

---

## 7. Các tùy chỉnh Cột (Grid Customization)

Nâng cao trải nghiệm người dùng trên Grid.

### A. ColumnsControls (Show/Hide Columns)
Linh kiện quản lý menu thả xuống "Columns" để người dùng tự chọn cột muốn xem.
```xml
<listingToolbar name="listing_top">
    <columnsControls name="columns_controls"/>
</listingToolbar>
```
- **minVisible**: Số lượng cột tối thiểu không được ẩn (mặc định: 1).
- **maxVisible**: Số lượng cột tối đa hiển thị trong menu (mặc định: 30).

### B. ColumnsResize
Cho phép người dùng thay đổi độ rộng của cột bằng cách kéo thả cạnh tiêu đề.
```xml
<columns name="listing_columns">
    <settings>
        <resizeConfig>
            <enabled>true</enabled>
            <rootSelector>${ $.columnsProvider }:container</rootSelector>
        </resizeConfig>
    </settings>
</columns>
```

---

## 8. Inline Editor Sub-components

Hệ thống Inline Edit hoạt động dựa trên sự phối hợp của nhiều linh kiện nhỏ:

- **ColumnsEditingClient**: Quản lý trạng thái dữ liệu tạm thời trên trình duyệt (Client-side state).
- **ColumnsEditingBulk**: Xử lý việc lưu hàng loạt (Bulk save) khi người dùng sửa nhiều dòng cùng lúc.
- **ColumnsEditorView**: Hiển thị các nút "Save", "Cancel" ở phía trên Grid khi đang ở chế độ sửa.
- **ColumnsEditorRecord**: Đại diện cho logic của một dòng dữ liệu đang được sửa.

Thông thường, lập trình viên chỉ cần cấu hình `editorConfig` trong XML, Magento sẽ tự động khởi tạo các sub-components này.

---

## 9. Container Component

Linh kiện bọc (wrapper) đơn giản nhất, dùng để nhóm các linh kiện khác lại với nhau mà không cần thêm logic phức tạp. Kế thừa từ `uiCollection`.

```xml
<container name="my_group">
    <argument name="data" xsi:type="array">
        <item name="config" xsi:type="array">
            <item name="label" xsi:type="string" translate="true">Nhóm Linh Kiện</item>
        </item>
    </argument>
    <!-- Các component con ở đây -->
</container>
```

---

## 10. Linh kiện Ngày tháng & Email

### A. Date / DateTime
Linh kiện chọn ngày tháng cho Form (sử dụng jQuery UI Datepicker).
```xml
<field name="special_from_date" formElement="date">
    <settings>
        <dataType>text</dataType>
        <label translate="true">Từ ngày</label>
        <validation>
            <rule name="validate-date" xsi:type="boolean">true</rule>
        </validation>
    </settings>
</field>
```

### B. Date Column (Grid)
Hiển thị ngày tháng trên Listing.
```xml
<column name="created_at" component="Magento_Ui/js/grid/columns/date">
    <settings>
        <filter>dateRange</filter>
        <dataType>date</dataType>
        <label translate="true">Ngày tạo</label>
        <dateFormat>MMM d, yyyy</dateFormat>
    </settings>
</column>
```

### C. Email
Linh kiện nhập văn bản kế thừa `input` nhưng được tích hợp sẵn validator email.

---

## 11. Danh sách động (Dynamic Rows)

Linh kiện cho phép quản lý một tập hợp dữ liệu lặp lại (ví dụ: Tier Price, Bundle Options). Đây là một trong những component phức tạp nhất.

### Cấu trúc cơ bản:
1. **DynamicRows**: Container chính.
2. **Container (Record)**: Lớp vỏ bọc cho một dòng dữ liệu (sử dụng `isTemplate = true`).
3. **Fields**: Các trường nhập liệu bên trong mỗi dòng.

### Cấu hình XML mẫu:
```xml
<dynamicRows name="my_dynamic_list">
    <settings>
        <addButtonLabel translate="true">Thêm dòng mới</addButtonLabel>
        <componentType>dynamicRows</componentType>
    </settings>
    <container name="record" component="Magento_Ui/js/dynamic-rows/record">
        <settings>
            <isTemplate>true</isTemplate>
            <is_collection>true</is_collection>
            <componentType>container</componentType>
        </settings>
        <!-- Các field trong một dòng -->
        <field name="title" formElement="input">
            <settings>
                <label translate="true">Tiêu đề</label>
            </settings>
        </field>
        <actionDelete name="action_delete"/> <!-- Nút xóa dòng -->
    </container>
</dynamicRows>
```

### Các tính năng nâng cao:
- **dndConfig (Drag & Drop)**: Hỗ trợ kéo thả để thay đổi thứ tự các dòng.
- **pageSize**: Hỗ trợ phân trang ngay bên trong danh sách động nếu số lượng dòng quá lớn.
- **identificationProperty**: Property dùng để định danh duy nhất mỗi dòng (mặc định là `record_id`).

---

## 12. Cấu trúc Form (Form Structure)

### Fieldset
Linh kiện dùng để nhóm các trường nhập liệu liên quan lại với nhau, giúp giao diện gọn gàng hơn.

```xml
<fieldset name="product_details">
    <settings>
        <label translate="true">Thông tin chi tiết</label>
        <collapsible>true</collapsible>
        <opened>true</opened> <!-- Trạng thái mặc định khi load trang -->
    </settings>
    <!-- Các field con ở đây -->
</fieldset>
```

---

## 13. Xử lý Tệp tin (File Management)

### File Uploader
Linh kiện tải tệp lên (Ảnh, Tài liệu) thông qua AJAX, hỗ trợ xem trước (preview).

```xml
<field name="image" formElement="fileUploader">
    <settings>
        <label translate="true">Ảnh đại diện</label>
        <componentType>fileUploader</componentType>
    </settings>
    <formElements>
        <fileUploader>
            <settings>
                <allowedExtensions>jpg jpeg gif png</allowedExtensions>
                <maxFileSize>2097152</maxFileSize> <!-- Bytes -->
                <uploaderConfig>
                    <param xsi:type="string" name="url">route/to/upload/controller</param>
                </uploaderConfig>
            </settings>
        </fileUploader>
    </formElements>
</field>
```

---

## 14. Công cụ Grid (Grid Tools)

Các linh kiện bổ trợ nằm trên thanh công cụ của Listing.

### A. Export Button
Cung cấp tính năng xuất dữ liệu Grid ra tệp tin. Thường hỗ trợ CSV và Excel XML.
```xml
<listingToolbar name="listing_top">
    <exportButton name="export_button"/>
</listingToolbar>
```

### B. Filters
Bảng điều khiển chứa các bộ lọc dữ liệu. Magento sẽ tự động thu thập các cột có khai báo `<filter>` và đưa vào đây.
```xml
<listingToolbar name="listing_top">
    <filters name="listing_filters"/>
</listingToolbar>
```

### C. Filters Chips (Active Filters)
Hiển thị các "nhãn" (tags) của những bộ lọc đang được người dùng áp dụng. Cho phép người dùng xóa nhanh từng bộ lọc. Thường nằm trong `filters` component.

### D. Expandable Column (Grid Detail)
Cột đặc biệt cho phép "bung" chi tiết của dòng dữ liệu ngay tại chỗ.
- Cần chỉ định `bodyTmpl` để hiển thị nội dung mở rộng.
- Thường dùng để hiển thị thông tin metadata phức tạp của bản ghi.

---

## 15. Kiến trúc Form & Dữ liệu (Form Architecture)

### A. Form Component
Linh kiện gốc cho các trang chỉnh sửa/tạo mới. Nó quản lý việc gửi dữ liệu (submit), xác thực (validation) và trạng thái của toàn bộ các field bên trong.

```xml
<form xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <settings>
        <buttons>
            <button name="save" class="Vendor\Module\Block\Adminhtml\Edit\SaveButton"/>
            <button name="back" class="Vendor\Module\Block\Adminhtml\Edit\BackButton"/>
        </buttons>
        <namespace>my_form_identifier</namespace>
        <dataScope>data</dataScope>
        <deps>
            <dep>my_form_identifier.my_form_data_source</dep>
        </deps>
    </settings>
    <dataSource name="my_form_data_source">
        <argument name="data" xsi:type="array">
            <item name="js_config" xsi:type="array">
                <item name="component" xsi:type="string">Magento_Ui/js/form/provider</item>
            </item>
        </argument>
        <settings>
            <submitUrl path="route/to/save"/>
        </settings>
        <dataProvider class="Vendor\Module\Ui\DataProvider\MyDataProvider" name="my_form_data_source">
            <settings>
                <requestFieldName>id</requestFieldName>
                <primaryFieldName>entity_id</primaryFieldName>
            </settings>
        </dataProvider>
    </dataSource>
</form>
```

### B. Form & Grid DataProvider Interface
Lớp PHP chịu trách nhiệm chuẩn bị mảng dữ liệu để JS UI Component có thể hiểu được.
- **Form DataProvider**: Trả về dữ liệu dạng `[entity_id => [data_fields]]`.
- **Grid DataProvider (SearchResult)**: Trả về danh sách `items` và `totalRecords`.

---

## 16. Nhập liệu Cơ bản & Nội dung

### A. Input (Text)
Trường nhập văn bản tiêu chuẩn.
```xml
<field name="title" formElement="input">
    <settings>
        <label translate="true">Tiêu đề</label>
        <dataType>text</dataType>
        <visible>true</visible>
    </settings>
</field>
```

### B. Hidden
Dùng để lưu trữ các giá trị cần thiết cho submit nhưng không hiển thị cho người dùng (ví dụ: `entity_id`).
```xml
<field name="entity_id" formElement="hidden">
    <settings>
        <dataType>text</dataType>
    </settings>
</field>
```

### C. HtmlContent
Cho phép nhúng mã HTML tĩnh hoặc từ một Block PHP vào UI Component.
```xml
<htmlContent name="html_content">
    <block name="my_block" class="Vendor\Module\Block\Adminhtml\CustomHtml"/>
</htmlContent>
```

---

## 17. Xử lý Hình ảnh (Image Handling)

### ImageUploader
Kế thừa từ FileUploader nhưng chuyên dụng cho hình ảnh (có thumbnail preview và xóa ảnh).

```xml
<field name="image" formElement="imageUploader">
    <settings>
        <label translate="true">Ảnh sản phẩm</label>
        <componentType>imageUploader</componentType>
    </settings>
    <formElements>
        <imageUploader>
            <settings>
                <allowedExtensions>jpg jpeg gif png</allowedExtensions>
                <maxFileSize>2097152</maxFileSize>
                <uploaderConfig>
                    <param xsi:type="string" name="url">route/to/upload</param>
                </uploaderConfig>
                <previewTmpl>Magento_Catalog/image-preview</previewTmpl>
            </settings>
        </imageUploader>
    </formElements>
</field>
```

---

## 18. InsertForm (Sub-form)
Cho phép nhúng một Form UI Component hoàn chỉnh vào bên trong một Form khác. Rất hữu ích khi cần tái sử dụng logic form ở nhiều nơi (ví dụ: nhúng Form tạo Category vào Form tạo Product).

```xml
<insertForm name="my_sub_form">
    <settings>
        <formSubmitType>ajax</formSubmitType>
        <renderUrl path="mui/index/render_handle">
            <param name="handle">full_ui_component_name</param>
        </renderUrl>
        <loadingUrl path="mui/index/render"/>
        <toolbarContainer>${ $.parentName }</toolbarContainer>
        <ns>target_form_namespace</ns>
    </settings>
</insertForm>
```

---

## 19. Kiến trúc Grid & Listing (Listing Architecture)

### A. Listing (Grid) Component
Linh kiện cốt lõi để hiển thị danh sách dữ liệu trong Admin. Nó phối hợp giữa `dataSource` (PHP) và `columns` (JS) để tạo ra bảng dữ liệu có tính năng lọc, phân trang và sắp xếp.

```xml
<listing xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <settings>
        <buttons>
            <button name="add" class="Vendor\Module\Block\Adminhtml\Grid\AddButton"/>
        </buttons>
        <spinner>my_columns_name</spinner>
        <deps>
            <dep>my_listing_identifier.my_data_source</dep>
        </deps>
    </settings>
    <!-- Config DataSource tương tự Form nhưng component là grid/provider -->
</listing>
```

### B. InsertListing
Dùng để nhúng một trang Listing hoàn chỉnh vào một Component khác (thường là Form). Ví dụ: hiển thị danh sách đơn hàng của khách hàng ngay trong trang chi tiết khách hàng.

### C. MassActions
Cung cấp các hành động thực hiện trên nhiều bản ghi cùng lúc.
```xml
<listingToolbar name="listing_top">
    <massaction name="listing_massaction">
        <action name="delete">
            <settings>
                <confirm>
                    <message translate="true">Xóa các bản ghi đã chọn?</message>
                    <title translate="true">Xác nhận</title>
                </confirm>
                <url path="route/to/massDelete"/>
                <type>delete</type>
                <label translate="true">Xóa</label>
            </settings>
        </action>
    </massaction>
</listingToolbar>
```

---

## 20. Linh kiện Popup & Layout

### A. Modal (Popup)
Linh kiện tạo cửa sổ hội thoại (Dialog). Có thể chứa Form hoặc bất kỳ nội dung UI khác.
```xml
<modal name="my_modal">
    <settings>
        <options>
            <option name="title" xsi:type="string">Thông báo</option>
            <option name="type" xsi:type="string">popup</option>
        </options>
    </settings>
    <!-- Nội dung bên trong modal -->
</modal>
```

### B. Masonry Layout
Sắp xếp các khối linh kiện con theo dạng "gạch xếp" (giống Pinterest), tự động tối ưu không gian hiển thị.

---

## 21. Nhập liệu Phức hợp (Complex Inputs)

### A. Multiselect
Cho phép chọn nhiều giá trị từ một danh sách cố định.
```xml
<field name="store_ids" formElement="multiselect">
    <settings>
        <label translate="true">Cửa hàng hiển thị</label>
        <options class="Magento\Store\Model\ResourceModel\Store\Collection"/>
    </settings>
</field>
```

### B. Multiline
Dùng để nhóm nhiều ô nhập liệu thành một khối logic (ví dụ: Địa chỉ gồm dòng 1, dòng 2).
```xml
<field name="street" formElement="multiline">
    <settings>
        <label translate="true">Địa chỉ</label>
    </settings>
</field>
```

---

## 22. Linh kiện Cột Grid (Bổ sung)

- **LinkColumn**: Hiển thị text có gắn link tĩnh hoặc động.
- **MultiselectColumn**: Cột chứa checkbox ở đầu mỗi dòng Grid để phục vụ MassActions.

---

## 23. ListingToolbar & Công cụ Phân trang

### A. ListingToolbar
Container phía trên Grid, chứa tất cả các tiện ích điều khiển.
```xml
<listingToolbar name="listing_top">
    <bookmark name="bookmarks"/>
    <columnsControls name="columns_controls"/>
    <exportButton name="export_button"/>
    <filterSearch name="fulltext"/>
    <filters name="listing_filters"/>
    <massaction name="listing_massaction">
        <!-- Các action -->
    </massaction>
    <paging name="listing_paging"/>
</listingToolbar>
```

### B. Paging
Phân trang cho Grid. Tự động tạo giao diện "Trang 1 / N" với điều hướng.
```xml
<paging name="listing_paging"/>
```

### C. Sizes
Cho phép người dùng thay đổi số lượng bản ghi hiển thị trên mỗi trang (20, 30, 50, 100, 200).

### D. Search
Ô tìm kiếm nhanh (fulltext search) trên Grid.
```xml
<filterSearch name="fulltext">
    <argument name="data" xsi:type="array">
        <item name="config" xsi:type="array">
            <item name="provider" xsi:type="string">ns.ns.listing_top.my_data_source</item>
            <item name="chipsProvider" xsi:type="string">ns.ns.listing_top.listing_filters_chips</item>
            <item name="storageConfig" xsi:type="array">
                <item name="provider" xsi:type="string">ns.ns.listing_top.bookmarks</item>
                <item name="namespace" xsi:type="string">current.search</item>
            </item>
        </item>
    </argument>
</filterSearch>
```

### E. Range (Filter)
Bộ lọc theo khoảng giá trị (from/to), thường dùng cho số lượng và giá.

### F. TreeMassActions
Phiên bản nâng cao của MassActions, hỗ trợ trình đơn con dạng cây (ví dụ: "Thay đổi trạng thái > Bật / Tắt").

---

## 24. Nhập liệu Form (Bổ sung)

### A. Select (Dropdown)
Chọn một giá trị từ danh sách.
```xml
<field name="status" formElement="select">
    <settings>
        <label translate="true">Trạng thái</label>
        <options class="Magento\Config\Model\Config\Source\Yesno"/>
    </settings>
</field>
```

### B. RadioSet
Nhóm nút radio cho phép chọn một giá trị duy nhất.
```xml
<field name="type" formElement="radioset">
    <settings>
        <label translate="true">Loại</label>
        <options class="Vendor\Module\Model\Source\TypeOptions"/>
    </settings>
</field>
```

### C. Textarea
Ô nhập văn bản nhiều dòng.
```xml
<field name="description" formElement="textarea">
    <settings>
        <label translate="true">Mô tả</label>
        <rows>5</rows>
        <cols>15</cols>
    </settings>
</field>
```

### D. WYSIWYG
Trình soạn thảo văn bản trực quan (What You See Is What You Get) tích hợp TinyMCE.
```xml
<field name="content" formElement="wysiwyg">
    <settings>
        <label translate="true">Nội dung</label>
    </settings>
    <formElements>
        <wysiwyg>
            <settings>
                <wysiwyg>true</wysiwyg>
            </settings>
        </wysiwyg>
    </formElements>
</field>
```

### E. UrlInput
Linh kiện nhập URL chuyên dụng, hỗ trợ chọn liên kết đến Product, Category hoặc CMS Page qua giao diện chọn.

### F. UI-Select
Linh kiện dropdown nâng cao (typeahead/searchable), hỗ trợ tìm kiếm ajax, chọn nhiều và hiển thị dạng cây.

---

## 25. Cột Grid (Bổ sung cuối)

- **OnOffColumn**: Hiển thị switch bật/tắt cho giá trị boolean ngay trên Grid.
- **SelectColumn**: Hiển thị giá trị của một field Select dưới dạng nhãn (label) thay vì giá trị thô.
- **ThumbnailColumn**: Hiển thị hình thu nhỏ (thumbnail) cho cột hình ảnh trong Grid.

---

## 26. Tab (Navigation)

Linh kiện tạo giao diện Tab (thẻ chuyển đổi) trong Form. Mỗi Tab chứa một Fieldset.
```xml
<htmlContent name="my_tab_content">
    <block class="Vendor\Module\Block\Adminhtml\MyTabBlock"/>
</htmlContent>
```

---

## Liên kết
- [Kiến trúc UI Components](./ui-components.md)
- [Admin UI Grid](./admin-ui-grid.md)
- [Thư viện JavaScript](./ui-components-js-library.md)
