# Spec Workflow

> Quy tắc đầy đủ cho AI: xem `AGENTS.md`

---

## Flow tổng thể

```
1. SPEC [vai BA]      Mô tả feature → AI phân tích business → spec.md → "OK spec"
2. PLAN [vai Dev]     AI thiết kế technical → plan.md + tasks.md → "OK implement"
3. IMPLEMENT [Dev+QA] Với mỗi task có business logic:
                        a. AI viết unit test trước (test sẽ fail)
                        b. AI implement code
                        c. Chạy test → phải pass (max 3 lần retry)
                        d. Code review bằng skill + checklist (max 3 lần retry)
                        e. Báo done + completion report
4. STATUS             AI cập nhật status.md sau mỗi bước lớn
```

> Task không có business logic (scaffold, layout, i18n): bỏ qua bước unit test, chỉ cần verify + review.

---

## Checklist trước khi gõ "OK spec"

Trước khi gõ `OK spec`, hãy tự hỏi:

- [ ] Acceptance criteria đúng và đủ chưa? Không bị thiếu case quan trọng?
- [ ] Scope in/out-of-scope rõ ràng chưa? Biết chắc cái gì làm, cái gì không làm?
- [ ] Decision/approach đã chốt chưa? Không còn câu hỏi mở nào?

Nếu chưa chắc → hỏi lại AI, đừng OK vội.

## Checklist trước khi gõ "OK implement"

Trước khi gõ `OK implement`, hãy tự hỏi:

- [ ] Technical approach trong `plan.md` hợp lý chưa? Risk nào chưa được xử lý?
- [ ] Danh sách task đủ chưa? Thứ tự có hợp lý không?
- [ ] Verify steps trong từng task đủ để kiểm tra kết quả không?
- [ ] Scope file/module được phép sửa đã chốt rõ chưa?

Nếu chưa chắc → hỏi lại AI, đừng OK vội.

---

## Quick Start (30 giây)

Copy block này khi mở chat mới:

```text
Follow .spec workflow in this repo.
Feature: <tên ngắn>
Bối cảnh/vấn đề: <đau ở đâu>
Kết quả mong muốn: <muốn đạt gì>
Phạm vi biết chắc: <module/file nếu có, không có thì để trống>
```

---

## Prompt nhanh theo ngữ cảnh

### Bắt đầu feature mới

```text
Follow .spec workflow in this repo.
Project: <tên-project>
Feature: <tên ngắn>
Bối cảnh/vấn đề: <đau ở đâu>
Kết quả mong muốn: <muốn đạt gì>
Phạm vi biết chắc: <module/file nếu có>
```

### Bắt đầu feature có sửa Admin system config

```text
Follow .spec workflow in this repo.
Feature: <tên ngắn>
Bối cảnh/vấn đề: <đau ở đâu>
Kết quả mong muốn: <muốn đạt gì>
Phạm vi biết chắc: <module/file nếu có>
Áp dụng Admin System Config Pattern trong config/magento-patterns.md:
- section riêng + tab vendor
- có resource ACL
- dùng config_path nếu cần giữ key cũ
- verify cache/config UI sau khi sửa
```

### Làm task đã có sẵn

```text
Đọc config/constitution.md.
Đọc features/<tên-feature>/tasks.md.
Implement task số <N>.
```

### Review / audit code

```text
Đọc config/checklist.md.
Kiểm tra code của module <tên-module> theo checklist.
```

### Hotfix (production urgent)

```text
HOTFIX: <mô tả lỗi ngắn gọn>
Scope: <file/module bị ảnh hưởng>
```

> AI sẽ bỏ qua "OK spec", fix ngay. Vẫn giữ unit test + code review.
> Bắt buộc cleanup spec/plan đầy đủ trong vòng 1-2 ngày sau.

### Hỏi có tham khảo spec/code (không tạo file)

```text
Q: <câu hỏi>
```

> AI sẽ đọc `.spec/config/references`, `.spec/examples`, code liên quan trước khi trả lời.
> Không tạo spec/plan/task, không trigger workflow.

---

### Xác nhận update spec trong lúc implement

```text
OK update spec
```

> AI phát hiện vấn đề trong lúc implement → ghi vào `status.md` section `⚠️ Pending Clarification` → DỪNG chờ xác nhận.
> Gõ `OK update spec` để cho phép AI cập nhật `spec.md`/`plan.md`.
> Gõ `Skip` để bỏ qua, `Defer` để ghi backlog xử lý sau.

---

### Làm lại từ đầu (code đã bị xoá)

```text
Follow .spec workflow in this repo.
Project: <tên-project>
Feature: <tên ngắn>
Bối cảnh/vấn đề: <đau ở đâu>
Kết quả mong muốn: <muốn đạt gì>
Phạm vi biết chắc: <module/file, business rules đã chốt>
Dùng lại spec tại projects/<tên-project>/features/<tên-feature>/spec.md và implement lại từ đầu.
```

---

## Cấu trúc thư mục

```
.spec/
├── config/          → core rules (bưng đi dự án nào cũng được)
├── skills/          → agent skills
├── examples/        → blueprints + module skeleton
├── templates/       → task contracts
├── features/        → core features (reusable)
└── projects/
    └── <tên-project>/   → project-specific
        ├── glossary.md  → thuật ngữ domain riêng
        └── features/    → features của project này
```

> Khi bưng `.spec/` sang dự án mới: giữ nguyên tất cả, chỉ tạo folder `projects/<tên-project-mới>/` mới.

---

## Vai trò từng file (single source of truth)

| File | Trả lời câu hỏi |
|---|---|
| `spec.md` | Làm gì và vì sao (business context, AC, testcase) |
| `plan.md` | Làm như thế nào (technical design, scope, risks) |
| `tasks.md` | Làm theo thứ tự nào (checklist + verify steps) |
| `status.md` | Đang ở đâu (stage hiện tại, blocker, next action) |

---

## File tham chiếu

| Mục đích | File |
|---|---|
| Quy tắc AI + workflow | `AGENTS.md` |
| Coding standards + DoD | `config/constitution.md` |
| Magento patterns | `config/magento-patterns.md` |
| Domain glossary | `config/glossary.md` |
| Checklist review | `config/checklist.md` |
| Skills map | `skills/skills-list.md` |
| Templates feature | `features/_template/` |
| Task contracts | `templates/task-contract*.md` |
| Blueprints index | `examples/INDEX.md` |
| Frontend theme references (Luma + Figma) | `config/references/frontend/luma-figma-theme-docs-index.md` |

---

## Frontend Luma + Figma (dùng cho nhiều page)

Khi làm frontend theme (homepage/PLP/PDP/cart/checkout), đọc trước:

- `config/references/frontend/luma-figma-theme-docs-index.md`

Bộ docs này chuẩn hóa:

- Kiến trúc theme theo XML + PHTML + LESS
- Token + breakpoint strategy (mobile-first)
- Component contract + QA gate
- Section-module pattern (Data Patch tạo CMS block + render trong PHTML)
