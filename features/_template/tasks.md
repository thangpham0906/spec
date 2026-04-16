# Tasks - <tên-feature>

> Không lặp business context. Mỗi task: scope + AC + unit test + verify.
> TDD flow bắt buộc với task có business logic: **viết test trước → implement → test pass → code review**.
> Nếu test fail hoặc review còn Critical/High issue: **DỪNG, không được báo task hoàn thành**.
> - Test fail → phân tích root cause → sửa implementation (không sửa test để "cheat") → chạy lại. Tối đa **3 lần**; nếu vẫn fail sau 3 lần thì DỪNG, báo người dùng kèm phân tích nguyên nhân.
> - Review Critical/High → sửa implementation → chạy lại test → review lại. Tối đa **3 lần**; nếu vẫn còn issue thì DỪNG, báo người dùng kèm danh sách issue còn lại.

## Task 1 - <tên>
- Scope: `<path>`
- Acceptance criteria:
  - `<AC 1>`
  - `<AC 2>`
- Unit test (`Test/Unit/<mirror-path>.php`) — viết trước khi implement:
  - `<test case 1 — happy path>`
  - `<test case 2 — edge/negative>`
- Implement: `<path>`
- Verify sau implement:
  - `./vendor/bin/phpunit <test-path>` → all pass
  - `<manual step nếu cần>`
- Code review (sau test pass):
  - Dùng skill `magento-code-reviewer`
  - Đối chiếu `config/checklist.md`

## Task 2 - <tên> *(không có business logic — không cần unit test)*
- Scope: `<path>`
- Acceptance criteria:
  - `<AC 1>`
- Verify:
  - `<command hoặc manual step>`

---

> Rules bổ sung — áp dụng khi liên quan, xem chi tiết trong `constitution.md` + `magento-patterns.md`:
> - `before` plugin return array → không `unset()` tham số
> - Logger → `Psr\Log\LoggerInterface`
> - `system.xml` → `cache:clean config` + verify menu path Admin
> - Unit test bắt buộc: `Service/`, `Model/` logic, `Validator/`, `Plugin/`, `Observer/`, `Carrier/collectRates`
> - Unit test không bắt buộc: scaffold, layout/template, i18n, routing

## Completion report
1. Files changed
2. Lý do thay đổi
3. Unit test results (pass/fail)
4. Code review results (Critical/High issues nếu có)
5. Verify steps + kết quả testcase
6. Xác nhận DoD
