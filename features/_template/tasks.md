# Tasks - <tên-feature>

> Không lặp business context. Mỗi task: scope + AC + verify.

## Task 1 - <tên>
- Scope: `<path>`
- Acceptance criteria:
  - `<AC 1>`
  - `<AC 2>`
- Verify:
  - `<command hoặc manual step>`
  - `<expected result>`

## Task 2 - <tên>
- Scope: `<path>`
- Acceptance criteria:
  - `<AC 1>`
  - `<AC 2>`
- Verify:
  - `<command hoặc manual step>`
  - `<expected result>`

---

> Rules bổ sung — áp dụng khi liên quan, xem chi tiết trong `constitution.md` + `magento-patterns.md`:
> - `before` plugin return array → không `unset()` tham số
> - Logger → `Psr\Log\LoggerInterface`
> - `system.xml` → `cache:clean config` + verify menu path Admin

## Completion report
1. Files changed
2. Lý do thay đổi
3. Verify steps + kết quả testcase
4. Xác nhận DoD
