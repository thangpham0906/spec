# Tasks - <tên-feature>

> Không lặp business context. Mỗi task: scope + AC + verify.

## Task 1 - <tên>
- Scope: `<path>`
- AC: `<...>`
- Verify: `<command hoặc manual step>`

## Task 2 - <tên>
- Scope: `<path>`
- AC: `<...>`
- Verify: `<command hoặc manual step>`

---

> Rules bổ sung áp dụng khi liên quan — xem `constitution.md` + `magento-patterns.md`:
> - `before` plugin return array → không `unset()` tham số
> - Logger → `Psr\Log\LoggerInterface`
> - `system.xml` → verify `cache:clean config` + menu path Admin

## Completion report
1. Files changed
2. Lý do thay đổi
3. Verify steps + kết quả testcase
4. Xác nhận DoD
