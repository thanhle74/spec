# Feature Tasks - <tên-feature>

> Quy tắc: task phải actionable; không mô tả lại business context đã có ở `spec.md`.

## Task 1 - <tên task>
- Scope: `<files/module>`
- Acceptance criteria:
  - `<AC 1>`
  - `<AC 2>`
- Verify:
  - `<command/manual step>`
  - `<expected result>`

## Task 2 - <tên task>
- Scope: `<files/module>`
- Acceptance criteria:
  - `<AC 1>`
  - `<AC 2>`
- Verify:
  - `<command/manual step>`
  - `<expected result>`

## Rule bổ sung khi task có `before*` plugin trả về tham số
- Acceptance criteria bắt buộc:
  - `Không unset() bất kỳ tham số nào sẽ xuất hiện trong return array cho method gốc`
- Verify bắt buộc:
  - `Smoke/API: gọi endpoint liên quan (vd GraphQL) không còn warning Undefined variable`

## Rule bổ sung khi task có class inject logger (PSR-3)
- Acceptance criteria bắt buộc:
  - `Constructor type-hint Psr\Log\LoggerInterface, không Magento\Psr\Log\LoggerInterface`
- Verify bắt buộc:
  - `bin/magento setup:di:compile` (bắt lỗi wiring sớm)

## Rule bổ sung khi task có sửa `etc/adminhtml/system.xml`
- Acceptance criteria bắt buộc:
  - `Section/group/field hiển thị đúng trong Admin theo menu path đã chỉ rõ`
  - `Có resource ACL hợp lệ; role test truy cập được`
  - `Nếu giữ key config cũ khi đổi vị trí UI: đã dùng config_path`
- Verify bắt buộc:
  - `bin/magento cache:clean config`
  - `bin/magento cache:flush`
  - `Manual: vào đúng menu path Admin, xác nhận thấy field và lưu config thành công`

## Completion report format
1. Files changed
2. Lý do thay đổi
3. Verify steps đã chạy
4. Kết quả testcase
5. Xác nhận đạt DoD
