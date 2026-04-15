# Task Contract - Observer / Plugin

## 1) Metadata

- Task name:
- Type: `observer | plugin | both`
- Priority: `P0|P1|P2`

## 2) Goal

- Mục tiêu thay đổi behavior:
- Event/method target:

## 3) Scope được phép sửa

- `app/code/<Vendor>/<Module>/Plugin/`
- `app/code/<Vendor>/<Module>/Observer/`
- `app/code/<Vendor>/<Module>/etc/di.xml`
- `app/code/<Vendor>/<Module>/etc/events.xml`

## 4) Out of scope

- Không sửa class target trực tiếp.
- Không dùng `<preference>` nếu plugin đủ dùng.

## 5) Inputs / References bắt buộc

- `.spec/config/constitution.md`
- `.spec/config/references/core/plugins.md`
- `.spec/config/references/core/events-observers.md`

## 6) Constraints kỹ thuật

**Plugin:**
- Loại plugin: `before | after | around` — ưu tiên `before`/`after`, tránh `around`.
- `sortOrder` bắt buộc khai báo trong `di.xml`.
- Tên plugin: `<vendor>_<module>_<mô_tả>`.

**Observer:**
- Mỗi observer chỉ làm 1 việc (Single Responsibility).
- Không dispatch event mới trong observer nếu không cần thiết (tránh chain event khó debug).
- Không throw exception trong observer trừ khi có lý do rõ ràng.

## 7) Acceptance Criteria

- [ ] Plugin/Observer đăng ký đúng trong `di.xml` / `events.xml`.
- [ ] Logic chỉ chạy đúng điều kiện (có guard nếu cần).
- [ ] Không ảnh hưởng behavior ngoài scope.
- [ ] Không lỗi compile (`bin/magento setup:di:compile`).

## 8) Verify steps bắt buộc

- [ ] `bin/magento setup:di:compile`
- [ ] `bin/magento cache:flush`
- [ ] Test tay trigger event/method và xác nhận behavior đúng.
- [ ] Xác nhận không regression ở flow liên quan.

## 9) Output contract

1. Files changed
2. Lý do thay đổi
3. Verify steps đã chạy + kết quả
4. Risk còn lại (nếu có)
