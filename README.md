# Spec Workflow

> Quy tắc đầy đủ cho AI: xem `AGENTS.md`

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
Feature: <tên ngắn>
Bối cảnh/vấn đề: <đau ở đâu>
Kết quả mong muốn: <muốn đạt gì>
Phạm vi biết chắc: <module/file nếu có>
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

### Làm lại từ đầu (code đã bị xoá)

```text
Follow .spec workflow in this repo.
Feature: <tên ngắn>
Bối cảnh/vấn đề: <đau ở đâu>
Kết quả mong muốn: <muốn đạt gì>
Phạm vi biết chắc: <module/file, business rules đã chốt>
Dùng lại spec tại features/<tên-feature>/spec.md và implement lại từ đầu.
```

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
