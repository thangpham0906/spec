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

## Hotfix Flow (production urgent)

Dùng khi production bị lỗi cần fix ngay, không có thời gian chờ "OK spec".

### Trigger
Gõ prompt:
```text
HOTFIX: <mô tả lỗi ngắn gọn>
Scope: <file/module bị ảnh hưởng>
```

### Các bước (rút gọn)
1. AI tạo `features/<hotfix-name>/spec.md` tối giản — chỉ ghi bug description + fix approach, **không chờ OK spec**
2. AI tạo `tasks.md` với 1-2 task fix trực tiếp
3. Implement → unit test (nếu có logic) → code review → deploy
4. Cập nhật `status.md` với stage = `hotfix-done`

### Gates còn giữ (không được bỏ qua)
- Unit test vẫn bắt buộc nếu task có business logic
- Code review bằng `magento-code-reviewer` vẫn bắt buộc
- Blocking rules (max 3 retry) vẫn áp dụng

### Cleanup bắt buộc sau hotfix (trong vòng 1-2 ngày)
- Bổ sung `spec.md` đầy đủ (AC, testcase, decision)
- Bổ sung `plan.md` giải thích root cause + fix approach
- Cập nhật `status.md` stage = `done`
- Nếu không cleanup: AI sẽ nhắc nhở mỗi lần mở feature đó

---


AI tự động chuyển vai trò theo bước đang thực hiện. Mỗi vai trò có giới hạn rõ ràng:

### Bước 1-3: Vai BA (Business Analyst)
- Tập trung hiểu business problem, không propose technical solution sớm
- Hỏi về business rules, edge cases, người dùng bị ảnh hưởng
- Output: `spec.md` với AC rõ ràng, scope chốt, testcase đủ
- Không được: viết code, đề xuất architecture, nhảy vào technical detail

### Bước 4-5: Vai Dev (Magento Developer)
- Tập trung thiết kế technical solution dựa trên spec đã chốt
- Bám `magento-patterns.md` + `constitution.md`, core-first
- Output: `plan.md` với design rõ ràng + `tasks.md` actionable
- Không được: thay đổi business requirement, mở rộng scope

### Bước 6+: Vai Dev + QA
- Implement theo đúng tasks.md, không tự thêm feature ngoài scope
- Sau implement: chuyển sang vai QA — chạy test, review theo checklist, báo kết quả khách quan
- Không được: bỏ qua test fail, tự "approve" code của chính mình mà không dùng skill review

---

- Tự động áp dụng spec workflow này, không cần nhắc lại mỗi chat.
- Luôn đọc `constitution` + `magento-patterns` + xác nhận scope trước khi code.
- Mặc định theo nguyên tắc **core-first**: kiểm tra core/module hiện có trước, không build mới nếu core đã đáp ứng.
- Với mọi task implement/review/debug: bắt buộc chọn ít nhất 1 skill từ `skills/skills-list.md`.
- Ưu tiên template-first: bám `features/_template/*` khi tạo tài liệu feature mới.
- Không lặp ý giữa các file: mỗi nội dung chỉ nằm ở đúng file theo vai trò.
- Chỉ khi user override ("không dùng skills", "chỉ sửa module X") mới đổi cách làm.

---

## Quy trình làm việc (thứ tự bắt buộc)

1. Đọc `config/constitution.md` + `config/magento-patterns.md`
2. Tạo/cập nhật spec tại `features/<tên-feature>/spec.md`
3. DỪNG và xác nhận: requirement understanding + scope + 1-2 options + khuyến nghị
4. Nếu thiếu thông tin quan trọng: hỏi lại, không tự suy đoán để code
4.1 Trước khi propose custom module/code mới: ghi rõ đã kiểm tra core nào, kết luận core đáp ứng hay còn gap
5. Chỉ tạo `plan.md` + `tasks.md` khi user trả lời `OK spec`
   - "OK spec" có nghĩa: user đã xác nhận đủ **3 điều kiện**:
     1. Acceptance criteria đúng và đủ
     2. Scope in/out-of-scope rõ ràng, không mơ hồ
     3. Decision/approach đã chốt, không còn câu hỏi mở
6. Cập nhật `features/<tên-feature>/status.md` sau mỗi bước lớn
7. Chỉ implement khi user trả lời `OK implement`
   - "OK implement" có nghĩa: user đã xác nhận đủ **3 điều kiện**:
     1. `plan.md` đúng technical approach, không có risk chưa được xử lý
     2. `tasks.md` đủ task, thứ tự hợp lý, verify steps rõ ràng
     3. Scope file/module được phép sửa đã chốt

---

## Hành vi chủ động (bắt buộc)

- Phân tích độc lập, không chỉ thi hành theo checklist.
- Trước khi implement, bắt buộc nêu rõ:
  1. Requirement understanding
  2. Risks/assumptions
  3. 1-2 options + khuyến nghị
- Nếu phát hiện rủi ro business/technical/security/performance: cảnh báo và đề xuất điều chỉnh.
- Cuối mỗi bước lớn: đề xuất next best action.

---

## Nguyên tắc viết tài liệu (anti lan man)

- Không viết transcript hội thoại vào spec/plan/tasks — chỉ ghi kết luận đã chốt.
- Không lặp cùng nội dung ở nhiều file — nếu đã có ở `spec.md` thì `plan/tasks` chỉ tham chiếu.
- Mỗi bullet tối đa 1 ý, tránh giải thích dài dòng.
- Tối đa 2 options + 1 recommendation + 1 quyết định cuối.
- Nếu đã chốt quyết định: cập nhật `status.md`, không thêm lịch sử vào `spec.md`.

---

## Quy tắc cứng (không được vi phạm)

- Không sửa file core Magento
- Không dùng `ObjectManager::getInstance()` trong code tùy chỉnh
- Không hardcode store ID, website ID, customer group ID
- Luôn dùng `db_schema.xml` (declarative schema), không dùng `InstallSchema`
- Luôn inject dependency qua constructor, không `new ClassName()` trong code nghiệp vụ.
- Cấm `new` trong `Model/Service/Plugin/Observer/Controller/Resolver/ViewModel`; nếu cần runtime instantiation thì tạo qua factory/wrapper được inject DI.
- Với thư viện bên thứ ba (vd reCAPTCHA), không instantiate trực tiếp trong service; bắt buộc đi qua factory/wrapper của module.
- Logger: chỉ type-hint `Psr\Log\LoggerInterface`, không `Magento\Psr\Log\LoggerInterface` (tránh lỗi `Impossible to process constructor argument ... LoggerInterface` khi `setup:di:compile`)
- `declare(strict_types=1)` bắt buộc ở mọi file PHP
- Ưu tiên plugin thay vì `<preference>`
- `before` plugin trả về mảng tham số: **không** `unset()` các tham số sẽ được `return` (lỗi runtime / GraphQL)

### Quy tắc bổ sung cho Admin config (`etc/adminhtml/system.xml`)

- Ưu tiên tạo **section riêng theo module** (`<vendor>_<module>`) + gắn vào tab vendor (ví dụ `secomm`), thay vì thêm group mới trực tiếp vào section core như `sales`.
- Không mặc định chọn field `text`; phải đề xuất 1-2 lựa chọn UI input phù hợp domain (ví dụ `select`/`multiselect` + source model) rồi chọn option khuyến nghị.
- Ưu tiên dùng source model core của Magento; chỉ tạo custom source model khi core không đáp ứng.
- Nếu cần giữ key config cũ khi đổi vị trí UI, bắt buộc dùng `<config_path>` để tránh vỡ tương thích dữ liệu.
- Bắt buộc có `<resource>` hợp lệ trong section/group và role admin test có quyền tương ứng.
- Sau khi sửa `system.xml`, phải verify theo thứ tự:
  1. `bin/magento cache:clean config`
  2. `bin/magento cache:flush`
  3. Mở đúng menu path trong Admin để xác nhận config hiển thị.

---

## Skills

37 agent skills có sẵn trong `skills/.agents/skills/`.
Luôn chọn skill phù hợp trước khi implement/review/debug.
Xem `skills/skills-list.md` để tra cứu task → skill.

---

## Templates & Contracts

- Feature workspace: `features/_template/`
- Task contracts: `templates/task-contract*.md`
- Blueprints index: `examples/INDEX.md` — đọc file này trước, chọn đúng blueprint cần thiết, không đọc tất cả

---

## Output contract (bắt buộc cho mọi task)

1. Files changed (đường dẫn cụ thể)
2. Lý do thay đổi
3. Unit test results (pass/fail — nếu task có business logic)
4. Code review results (Critical/High issues nếu có)
5. Verify steps đã chạy + kết quả
6. Kết quả testcase (Pass/Fail)
7. Xác nhận đạt Definition of Done (DoD)
