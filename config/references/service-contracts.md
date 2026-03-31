# Tham khảo: Service Contracts (API & Repositories)

Nguồn: https://developer.adobe.com/commerce/php/development/components/service-contracts/

---

## 1. Service Contract là gì?

Service Contract là một bộ các PHP Interface được định nghĩa cho một module. Nó đóng vai trò là "lớp bảo vệ" bên ngoài, che giấu logic phức tạp bên trong Model và Resource Model. Mọi Interface trong Service Contract được coi là **Public API** (bắt buộc có tag `@api`).

**Cấu trúc folder bắt buộc (trong `app/code/<Vendor>/<Module>/`):**
- **`Api/`**: Chứa các **Service Interfaces** (ví dụ: Repository, Management interfaces).
- **`Api/Data/`**: Chứa các **Data Interfaces** (định nghĩa các getter/setter cho entity).

---

## 2. Data Interfaces (Thực thể dữ liệu)

Nằm trong `Api/Data/`. Giúp định nghĩa cấu trúc dữ liệu mà không quan tâm nó lưu ở bảng nào (EAV hay Flat table).

### Ví dụ: `EmployeeInterface.php`

```php
namespace <Vendor>\<Module>\Api\Data;

interface EmployeeInterface
{
    const ENTITY_ID = 'entity_id';
    const NAME = 'name';

    /** @return int|null */
    public function getEntityId();

    /** @param int $entityId @return $this */
    public function setEntityId($entityId);

    /** @return string|null */
    public function getName();

    /** @param string $name @return $this */
    public function setName($name);
}
```

---

## 3. Service Interfaces (Repository)

Nằm trong `Api/`. Thường là các Repository dùng để Lưu, Xoá, Lấy dữ liệu.

### Ví dụ: `EmployeeRepositoryInterface.php`

```php
namespace <Vendor>\<Module>\Api;

use <Vendor>\<Module>\Api\Data\EmployeeInterface;

interface EmployeeRepositoryInterface
{
    /** @param int $id @return EmployeeInterface */
    public function getById($id);

    /** @param EmployeeInterface $employee @return EmployeeInterface */
    public function save(EmployeeInterface $employee);

    /** @param int $id @return bool */
    public function deleteById($id);
}
```

---

## 4. Tại sao AI phải tuân thủ?

1. **Auto-generated Web API:** Khi định nghĩa đúng Interface trong `Api/`, chúng ta chỉ cần khai báo trong `webapi.xml`, Magento sẽ tự tạo endpoint `/V1/employees/:id`.
2. **Decoupling:** Các module khác khi gọi đến module này sẽ gọi qua `Api/Interface`, không quan tâm logic bên trong `Model/` thay đổi thế nào.
3. **Sử dụng `@api` tag:** Luôn thêm `@api` trong docblock của Interface để đánh dấu đây là Public API ổn định.

---

## 5. Quy trình làm việc đề xuất:

Khi tạo một tính năng mới (Feature), chúng ta sẽ làm theo thứ tự:
1. Tạo **Data Interface** (`Api/Data/`).
2. Tạo **Service Interface** (`Api/`).
3. Tạo **Model** implement Data Interface.
4. Tạo **Repository** implement Service Interface.
5. Cấu hình **`di.xml`** để map Interface -> Class thực tế (Preference).

---

## 6. Repository Design Pattern (CRUD chuẩn)

Một Repository chuẩn cho một Entity (ví dụ: `Employee`) bắt buộc phải có các method sau:

- `save(EmployeeInterface $employee)`
- `getById($id)`
- `getList(SearchCriteriaInterface $searchCriteria)`: Trả về `EmployeeSearchResultsInterface`
- `delete(EmployeeInterface $employee)`
- `deleteById($id)`

---

## 7. Search Results Interface (Kết quả tìm kiếm)

Nằm trong `Api/Data/`. Giúp type-hinting chính xác danh sách các thực thể trả về từ `getList()`.

### Ví dụ: `EmployeeSearchResultsInterface.php`

```php
namespace <Vendor>\<Module>\Api\Data;

use Magento\Framework\Api\SearchResultsInterface;

interface EmployeeSearchResultsInterface extends SearchResultsInterface
{
    /** @return \ <Vendor>\<Module>\Api\Data\EmployeeInterface[] */
    public function getItems();

    /** @param \ <Vendor>\<Module>\Api\Data\EmployeeInterface[] $items @return $this */
    public function setItems(array $items);
}
```

---

## 8. Management & Metadata Interfaces

- **Management Interface:** Dùng cho logic nghiệp vụ KHÔNG liên quan đến CRUD entity.
    - Ví dụ: `AccountManagementInterface::changePassword()`, `ShippingManagementInterface::estimateRate()`.
- **Metadata Interface:** Dùng để lấy thông tin mô tả về entity (attributes, phiên bản...).
    - Ví dụ: `ProductMetadataInterface::getVersion()`.

---

## Liên kết

- DI & Code Generation: xem [di-codegen.md](./di-codegen.md)
- Quy tắc chung: xem [../constitution.md](../constitution.md)
- Các pattern: xem [../magento-patterns.md](../magento-patterns.md)
