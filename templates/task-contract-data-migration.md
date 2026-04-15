# Task Contract - Data Migration (Schema + Patch)

## 1) Metadata

- Task name:
- Migration type: `schema | data | both`
- Priority: `P0|P1|P2`
- Reversible: `yes | no`

## 2) Goal

- Thay đổi cần thực hiện trên DB/data:
- Lý do cần migration:

## 3) Scope được phép sửa

- `app/code/<Vendor>/<Module>/etc/db_schema.xml`
- `app/code/<Vendor>/<Module>/etc/db_schema_whitelist.json`
- `app/code/<Vendor>/<Module>/Setup/Patch/Data/`
- `app/code/<Vendor>/<Module>/Setup/Patch/Schema/` (nếu cần schema patch)

## 4) Out of scope

- Không dùng `InstallSchema` / `UpgradeSchema` / `InstallData` / `UpgradeData`.
- Không sửa bảng core Magento trực tiếp.

## 5) Inputs / References bắt buộc

- `.spec/config/constitution.md`
- `.spec/config/references/core/declarative-schema.md`
- `.spec/config/references/core/data-schema-patch.md`

## 6) Constraints kỹ thuật

**Schema (`db_schema.xml`):**
- Tên bảng: `<vendor>_<module>_<entity>`.
- Bắt buộc có `db_schema_whitelist.json` sau khi thêm/sửa schema.
- Thêm index cho column thường dùng trong WHERE/JOIN.
- Không xóa column đang dùng mà không có migration plan.

**Data Patch:**
- Implement `DataPatchInterface`, không implement `PatchRevertableInterface` trừ khi patch thực sự reversible.
- Khai báo `getDependencies()` nếu patch phụ thuộc patch khác.
- Patch chỉ chạy 1 lần — không viết logic có side effect khi chạy lại.
- Không hardcode store ID / website ID / customer group ID.

## 7) Acceptance Criteria

- [ ] Schema thay đổi đúng như thiết kế.
- [ ] `db_schema_whitelist.json` đã được generate lại.
- [ ] Data patch chạy thành công, không lỗi.
- [ ] Không mất dữ liệu hiện có (nếu alter column).
- [ ] Không lỗi compile.

## 8) Verify steps bắt buộc

- [ ] `bin/magento setup:db-declaration:generate-whitelist --module-name=<Vendor_Module>`
- [ ] `bin/magento setup:upgrade`
- [ ] `bin/magento setup:di:compile`
- [ ] `bin/magento cache:flush`
- [ ] Kiểm tra DB: bảng/column/index tồn tại đúng.
- [ ] Kiểm tra patch đã ghi vào `patch_list` table.

## 9) Rollback plan

- Schema rollback: `<mô tả cách revert nếu cần>`
- Data rollback: `<backup strategy hoặc ghi N/A nếu không reversible>`

## 10) Output contract

1. Files changed
2. Lý do thay đổi
3. Verify steps đã chạy + kết quả
4. Risk còn lại (nếu có)
