# Magento Module Blueprint: `NullTraceX_Employee` (Admin Grid + Form + Repository)

Mục tiêu: lưu một reference "golden path" từ module thật để AI bám đúng cấu trúc khi bạn yêu cầu build module mới.

Nguồn chuẩn: `app/code/NullTraceX/Employee`

---

## 1) Bộ file cốt lõi cần có

### 1.1 Module bootstrap

- `registration.php`
- `etc/module.xml`
- `composer.json` (optional nhưng nên có)

### 1.2 DB + Model layer

- `etc/db_schema.xml`
- `Model/Employee.php`
- `Model/ResourceModel/Employee.php`
- `Model/ResourceModel/Employee/Collection.php`

### 1.3 Service contract (API nội bộ + webapi)

- `Api/Data/EmployeeInterface.php`
- `Api/EmployeeRepositoryInterface.php`
- `Model/EmployeeRepository.php`
- `etc/di.xml`
- `etc/webapi.xml`

### 1.4 Admin routing + ACL + menu

- `etc/adminhtml/routes.xml`
- `etc/adminhtml/menu.xml`
- `etc/acl.xml`

### 1.5 Admin controllers (CRUD)

- `Controller/Adminhtml/Employee/Index.php`
- `Controller/Adminhtml/Employee/NewAction.php`
- `Controller/Adminhtml/Employee/Edit.php`
- `Controller/Adminhtml/Employee/Save.php`
- `Controller/Adminhtml/Employee/Delete.php`

### 1.6 Admin UI component (grid + form)

- `view/adminhtml/layout/employee_employee_index.xml`
- `view/adminhtml/layout/employee_employee_edit.xml`
- `view/adminhtml/layout/employee_employee_new.xml`
- `view/adminhtml/ui_component/employee_listing.xml`
- `view/adminhtml/ui_component/employee_form.xml`
- `Ui/DataProvider/EmployeeDataProvider.php`
- `Ui/Component/Listing/Column/EmployeeBlockActions.php`
- `Ui/Component/Listing/Column/Status.php`

### 1.7 Optional UI customizations

- `Block/Form/Employee/Back.php`
- `Block/Form/Employee/Save.php`
- `Block/Form/Employee/Delete.php`
- `Block/Form/Employee/SaveAndContinueButton.php`
- `view/adminhtml/web/js/format-price.js`
- `view/adminhtml/web/template/grid/cells/response-request.html`

---

## 2) Build order khuyến nghị (để ít lỗi nhất)

1. Tạo bootstrap module (`registration.php`, `module.xml`).
2. Tạo `db_schema.xml` và chạy upgrade.
3. Tạo Model + ResourceModel + Collection.
4. Tạo Service Contract (`Api/*`, repository, `di.xml`).
5. Tạo admin route/menu/acl.
6. Tạo controllers CRUD admin.
7. Tạo layout handles + UI components grid/form.
8. Tạo DataProvider + custom listing columns/actions.
9. Tạo block buttons/custom JS templates nếu cần.
10. Verify với cache clean, compile, và truy cập admin grid.

---

## 3) Prompt mẫu để tái sử dụng blueprint này

```text
Dùng module blueprint tại `.spec/examples/integration/magento-module-admin-grid-employee-blueprint.md`.
Tạo module mới `<Vendor_Module>` theo cùng kiến trúc như `NullTraceX_Employee`.
Chỉ sửa trong phạm vi: `app/code/<Vendor>/<Module>`.
Yêu cầu output:
1) Danh sách file sẽ tạo theo từng phase,
2) Code cho từng file,
3) Các lệnh verify sau khi tạo xong.
```

---

## 4) Verify checklist nhanh

- `bin/magento setup:upgrade`
- `bin/magento setup:di:compile`
- `bin/magento cache:flush`
- Grid load được ở admin menu.
- Create/Edit/Delete hoạt động.
- ACL chặn đúng quyền.
- Web API route chạy đúng với token phù hợp.

---

## 5) Notes khi clone sang module khác

- Đổi toàn bộ namespace/path/resource key theo module mới.
- Đồng bộ tên `frontName`, route controller, và layout handles.
- Đồng bộ `primary key` trong:
  - db schema
  - model/resource model
  - ui listing data source (`indexField`, `primaryFieldName`, `requestFieldName`)
- Giữ naming nhất quán để giảm lỗi mapping ở UI component.
