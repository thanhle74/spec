# Constitution - Quy tắc phát triển chung

## Thông tin project

- Magento: 2.4.8
- PHP: 8.3
- Vendor: NullTraceX
- Namespace gốc: NullTraceX\<TênModule>

---

## 1. Coding Standard

### PHP
- PHP 8.3, sử dụng typed properties, union types, named arguments khi cần
- Tuân theo PSR-1, PSR-4, PSR-12
- Strict types bắt buộc: `declare(strict_types=1);` ở mọi file PHP
- Không dùng `@suppress`, không dùng `@codingStandardsIgnore`
- Return type bắt buộc cho mọi method (kể cả `: void`)
- Docblock chỉ khi cần giải thích logic phức tạp, không lặp lại type hint

### Nguyên tắc thiết kế
- SOLID bắt buộc:
  - S: Mỗi class chỉ làm 1 việc
  - O: Mở rộng bằng plugin/event, không sửa core
  - L: Subclass phải thay thế được parent
  - I: Interface nhỏ, cụ thể, không ép implement method thừa
  - D: Inject dependency qua constructor, không `new` trực tiếp
- DI (Dependency Injection) qua `di.xml`, KHÔNG dùng ObjectManager trực tiếp
- Composition hơn inheritance

---

## 2. Những điều KHÔNG được làm

- KHÔNG sửa file core Magento
- KHÔNG dùng ObjectManager::getInstance() trong code (chỉ cho trong factory tạo bởi Magento)
- KHÔNG hardcode store ID, website ID, customer group ID
- KHÔNG viết logic trong template (.phtml)
- KHÔNG dùng `<preference>` khi có thể dùng plugin
- KHÔNG để code chết, code comment out
- KHÔNG dùng `exit`, `die`, `var_dump`, `print_r` trong code chính thức
- KHÔNG tạo bảng DB mà không có `db_schema.xml` + `db_schema_whitelist.json`

---

## 3. Cấu trúc Module bắt buộc

Mỗi module <Vendor> phải có tối thiểu:
```
app/code/<Vendor>/<TênModule>/
├── registration.php
├── etc/
│   └── module.xml
├── composer.json           (không bắt buộc nhưng nên có)
```

Chỉ tính năng nào cần, mới thêm folder tương ứng. Không tạo folder rỗng.

---

## 4. Ngôn ngữ và Format

- Tên class, method, variable: tiếng Anh
- Comment giải thích logic: tiếng Việt hoặc tiếng Anh đều được
- Commit message: tiếng Anh
- Spec, plan, task: tiếng Việt
- **Document Tags (PHPDoc):**
    - `@api`: Bắt buộc cho mọi Public Interface (`Api/`).
    - `@since`: Bắt buộc cho Interface và Method trong Api folder (ví dụ: `@since 1.0.0`).
    - `@inheritdoc`: Dùng trong class thực thi (Implementation) để kế thừa docblock từ interface.
    - **Lưu ý đặc biệt cho Web API:** Mọi `@param` và `@return` trong Public Interface phải sử dụng **Fully Qualified Class Name** (ví dụ: `\Vendor\Module\Api\Data\MyInterface` thay vì `Data\MyInterface`) để hệ thống REST/SOAP tự động nhận diện đúng kiểu dữ liệu.

---

## 5. Xử lý lỗi

- Dùng exception cụ thể (LocalizedException, NoSuchEntityException...), không dùng \Exception chung
- Log lỗi bằng Psr\Log\LoggerInterface, không dùng echo hay file_put_contents
- Validate input đầu vào, không tin tưởng dữ liệu từ frontend

---

## 6. Database

- Dùng Declarative Schema (`db_schema.xml`), KHÔNG dùng InstallSchema/UpgradeSchema
- Đặt tên bảng: `<vendor>_<module>_<entity>` (ví dụ: `<vendor>_employee_record`)
- Tạo Data Patch cho dữ liệu mặc định, KHÔNG dùng InstallData/UpgradeData
- Index cho các column thường query

---

## 7. Frontend

- Dùng UI Component / KnockoutJS theo chuẩn Magento
- Layout XML để khai báo block, không hardcode trong template
- CSS dùng LESS, đặt trong `view/frontend/web/css/source/`
- JS dùng RequireJS, khai báo trong `requirejs-config.js`

---

## 8. API và Service Contract

- Cung cấp API interface trong `Api/` cho mọi entity chính
- Repository pattern: `Api/<Entity>RepositoryInterface.php`
- Data interface: `Api/Data/<Entity>Interface.php`
- Search Results: `Api/Data/<Entity>SearchResultsInterface.php`
- Khai báo trong `etc/di.xml` và `etc/webapi.xml` (nếu cần expose REST/GraphQL)

---

## Liên kết

- Các pattern chi tiết Magento: xem [magento-patterns.md](./magento-patterns.md)
- Checklist trước khi hoàn thành: xem [checklist.md](./checklist.md)
