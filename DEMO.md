# .spec — AI-Driven Development System
> Demo overview cho sếp | Mục tiêu: apply vào quy trình phát triển Magento của team

---

## Vấn đề đang giải quyết

Khi dùng AI để code mà không có hệ thống:
- AI làm đúng đầu, sai cuối — không ai kiểm soát được
- Mỗi dev prompt khác nhau → output không nhất quán
- Không có "nguồn sự thật" chung → AI tự suy đoán, tự mở rộng scope
- Feature lớn → AI mất context → code drift, bug khó trace

`.spec` giải quyết bằng cách biến AI thành một **team có quy trình**, không phải một cái máy gõ code.

---

## Cách hoạt động — 4 bước

```
1. SPEC      Dev mô tả feature (3 dòng) → AI đóng vai BA → ra spec.md
             Human review → gõ "OK spec"

2. PLAN      AI đóng vai Dev → thiết kế technical → plan.md + tasks.md
             Human review → gõ "OK implement"

3. IMPLEMENT Với mỗi task có business logic:
             AI viết test trước (TDD) → implement → test pass → code review
             Fail 3 lần → DỪNG, báo người

4. STATUS    AI cập nhật status.md sau mỗi bước lớn
```

**Human chỉ cần quyết định 2 lần:** "OK spec" và "OK implement".
Mọi thứ còn lại AI tự chạy theo quy trình.

---

## Cấu trúc hệ thống

```
.spec/
├── config/
│   ├── constitution.md        → Coding standards, SOLID, DoD, scope rules
│   ├── magento-patterns.md    → Index 30+ patterns với link reference chi tiết
│   ├── checklist.md           → Checklist review trước khi done
│   └── references/            → 30+ reference docs chi tiết theo domain
│       ├── core/              → Plugin, Observer, Repository, DI, Schema...
│       ├── network/           → REST API, GraphQL, RabbitMQ...
│       ├── frontend/          → ViewModel, UI Components, Admin Grid...
│       ├── infrastructure/    → Cache, S3, Search, Logging, Cron...
│       └── security/          → Payment, Vault, Security best practices
│
├── skills/                    → 39 Agent Skills (AI tools chuyên biệt)
│   └── .agents/skills/        → magento-code-reviewer, magento-graphql...
│
├── examples/                  → Blueprints thực tế (module skeleton, GraphQL, REST...)
├── templates/                 → Task contracts chuẩn hóa
│
├── features/                  → Core features (reusable across projects)
│   └── _template/             → Template: spec.md, plan.md, tasks.md, status.md
│
└── projects/
    └── laybyland/             → Project-specific features + domain glossary
```

---

## Điểm mạnh so với "vibe coding" thông thường

| | Vibe coding | .spec system |
|---|---|---|
| Scope control | AI tự mở rộng | Scope chốt trước, AI không được sửa ngoài |
| Consistency | Mỗi dev prompt khác nhau | Cùng 1 quy trình, cùng output format |
| Code quality | Không có gate | TDD bắt buộc + code review bằng skill |
| Feature lớn | Mất kiểm soát | Tách Epic → sub-features, mỗi cái chạy độc lập |
| Onboarding | Dev mới không biết context | Đọc spec là hiểu ngay business + technical |
| Audit trail | Không có | spec.md + plan.md + status.md = full history |

---

## 39 Agent Skills — AI chuyên biệt theo domain

Thay vì dùng AI generic, mỗi task gọi đúng skill:

**Backend / Architecture**
`magento-module-developer` `magento-feature-developer` `magento-model-developer`
`magento-api-developer` `magento-graphql` `magento-order-specialist` `magento-catalog-specialist`

**Frontend / Theme**
`magento-hyva-specialist` `magento-ui-component-developer` `magento-knockout-specialist`
`magento-alpine-specialist` `magento-magewire-specialist`

**Quality / Security**
`magento-code-reviewer` `magento-security-analyst` `magento-issue-debugger`
`magento-performance-analyst` `magento-cache-analyst`

**DevOps / Environment**
`magento-deployment-engineer` `ddev-magento` `magento-upgrade-specialist`

**Meta / Workflow**
`writing-clearly-and-concisely` `context-engineering-collection`

---

## Epic / Feature lớn — tách nhỏ có kiểm soát

Trước khi làm, AI phải phân loại:

```
Feature nhỏ (≤ 5 tasks, 1 module, 1 domain)
→ Tạo thẳng features/<tên>/

Feature lớn (nhiều module, nhiều domain, > 5 tasks)
→ Tạo epic.md trước → tách sub-features → mỗi sub chạy độc lập
```

Ví dụ thực tế — `layby-payment-retry`:
```
epic.md  (boundary + dependency map)
├── retry-orchestration/   (Stripe + LaybyLog)
├── customer-notification/ (depends on retry-orchestration)
└── order-escalation/      (depends on retry-orchestration)
```

---

## Quick Start — 30 giây để bắt đầu feature

```text
Follow .spec workflow in this repo.
Feature: custom-order-status
Bối cảnh/vấn đề: Thiếu trạng thái "Awaiting Supplier" cho warehouse theo dõi đơn hàng
Kết quả mong muốn: Admin set thủ công được, hiển thị đúng trên grid và email
Phạm vi biết chắc: Secomm_CustomOrderStatus module
```

AI sẽ tự:
1. Phân tích business, đề xuất options, hỏi những gì còn mơ hồ
2. Tạo `spec.md` với AC, testcase, decision
3. Chờ "OK spec" → tạo `plan.md` + `tasks.md`
4. Chờ "OK implement" → implement từng task theo TDD

---

## Definition of Done — không thể bỏ qua

Task chỉ được báo hoàn thành khi:

1. Đúng acceptance criteria trong spec
2. Không vi phạm constitution + magento-patterns
3. Unit test pass (nếu có business logic)
4. Code review bằng `magento-code-reviewer` — không còn Critical/High issue
5. Verify steps đã chạy, kết quả rõ ràng

---

## Portable — bưng sang dự án mới trong 1 phút

```
.spec/config/     → giữ nguyên (rules chung)
.spec/skills/     → giữ nguyên (39 skills)
.spec/examples/   → giữ nguyên (blueprints)
.spec/features/   → giữ nguyên (core features)

.spec/projects/<tên-project-mới>/   → tạo mới
    glossary.md   → thuật ngữ domain riêng
    features/     → features của project này
```

---

## Tóm lại

`.spec` không phải tool — nó là **quy trình làm việc với AI** được chuẩn hóa cho Magento.

- Dev junior có thể làm feature phức tạp vì AI hướng dẫn từng bước
- Dev senior không cần review từng dòng vì có gate tự động
- Team lead có full audit trail mà không cần họp
- Onboard người mới: đọc spec là hiểu ngay context

