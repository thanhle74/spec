# AGENTS.md — AI Agent Instructions

> Format: [openai/agents.md](https://github.com/openai/agents.md)
> Compatible with: Claude Code, Kiro, Cursor, Codex, GitHub Copilot, OpenCode, Trae, Qoder

---

## Dự án

Kho spec phát triển Magento 2.4.8-p4 / PHP 8.3.

Repo này chứa quy tắc, patterns, skills và spec feature cho quy trình phát triển Magento có hỗ trợ AI.

---

## Đọc bắt buộc trước mọi task

1. `config/constitution.md` — chuẩn code, quy tắc SOLID, DoD, scope governance
2. `config/magento-patterns.md` — các pattern Magento bắt buộc (plugin, observer, repository, v.v.)
3. `config/glossary.md` — thuật ngữ domain, tránh suy đoán sai

---

## Quy trình làm việc

Tuân theo spec workflow định nghĩa trong `README.md`:

1. Đọc `config/constitution.md` + `config/magento-patterns.md`
2. Tạo/cập nhật spec tại `features/<tên-feature>/spec.md`
3. DỪNG và xác nhận: requirement understanding + scope + 1-2 options + khuyến nghị
4. Chỉ tạo `plan.md` + `tasks.md` khi user trả lời `OK spec`
5. Chỉ implement khi user trả lời `OK implement`
6. Cập nhật `features/<tên-feature>/status.md` sau mỗi bước lớn

---

## Quy tắc cứng (không được vi phạm)

- Không sửa file core Magento
- Không dùng `ObjectManager::getInstance()` trong code tùy chỉnh
- Không hardcode store ID, website ID, customer group ID
- Luôn dùng `db_schema.xml` (declarative schema), không dùng `InstallSchema`
- Luôn inject dependency qua constructor, không `new ClassName()` (trừ DTO/Exceptions)
- `declare(strict_types=1)` bắt buộc ở mọi file PHP
- Ưu tiên plugin thay vì `<preference>`

---

## Skills

37 agent skills có sẵn trong `skills/.agents/skills/`.
Luôn chọn skill phù hợp trước khi implement/review/debug.
Xem `skills/skills-list.md` để tra cứu task → skill.

---

## Templates & Contracts

- Feature workspace: `features/_template/`
- Task contracts: `templates/task-contract*.md`
- Blueprints: `examples/integration/`

---

## Output contract (bắt buộc cho mọi task)

1. Files changed (đường dẫn cụ thể)
2. Lý do thay đổi
3. Verify steps đã chạy + kết quả
4. Kết quả testcase (Pass/Fail)
5. Xác nhận đạt Definition of Done (DoD)
