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

## Hành vi mặc định

- Tự động áp dụng spec workflow này, không cần nhắc lại mỗi chat.
- Luôn đọc `constitution` + `magento-patterns` + xác nhận scope trước khi code.
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
5. Chỉ tạo `plan.md` + `tasks.md` khi user trả lời `OK spec`
6. Cập nhật `features/<tên-feature>/status.md` sau mỗi bước lớn
7. Chỉ implement khi user trả lời `OK implement`

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
- Luôn inject dependency qua constructor, không `new ClassName()` (trừ DTO/Exceptions)
- `declare(strict_types=1)` bắt buộc ở mọi file PHP
- Ưu tiên plugin thay vì `<preference>`

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
3. Verify steps đã chạy + kết quả
4. Kết quả testcase (Pass/Fail)
5. Xác nhận đạt Definition of Done (DoD)
