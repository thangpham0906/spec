# Spec Workflow (Quick)

## 30s Quick Start

Copy block này khi mở chat mới:

```text
Follow .spec workflow in this repo.
Feature: <tên ngắn>
Bối cảnh/vấn đề: <đau ở đâu>
Kết quả mong muốn: <muốn đạt gì>
Phạm vi biết chắc: <module/file nếu có, không có thì để trống>
```

## Default behavior

- Mặc định AI tự động áp dụng `.spec workflow` trong repo này, bạn không cần nhắc lại mỗi chat.
- AI luôn đọc `constitution` + `magento-patterns`, xác nhận scope trước khi code, và báo cáo theo output bắt buộc.
- Với mọi task `implement/review/debug`: bắt buộc chọn ít nhất 1 skill từ `.spec/skills/skills-list.md` trước khi thực hiện.
- Chỉ khi bạn override (ví dụ: "không dùng skills", "chỉ sửa module X"), AI mới đổi cách làm theo yêu cầu đó.
- Ưu tiên `template-first`: bám `.spec/features/_template/*` khi tạo tài liệu feature mới.
- Tránh lặp ý giữa các file: mỗi nội dung chỉ nên nằm ở đúng file theo vai trò.

## Vai trò từng file (single source of truth)

- `spec.md`: Business context + scope + acceptance criteria + testcase (trả lời câu hỏi **làm gì** và **vì sao**).
- `plan.md`: Technical design + module/file scope + risks/rollback (trả lời câu hỏi **làm như thế nào**).
- `tasks.md`: Checklist thực thi + verify steps cho từng task (trả lời câu hỏi **làm theo thứ tự nào**).
- `status.md`: Stage hiện tại + quyết định mới nhất + blockers + next best action (không lặp lại toàn bộ spec/plan/tasks).

## Nguyên tắc viết ngắn gọn (anti lan man)

- Không viết transcript hội thoại vào `spec/plan/tasks`; chỉ ghi kết luận đã chốt.
- Không lặp cùng một nội dung ở nhiều file; nếu đã có ở `spec` thì `plan/tasks` chỉ tham chiếu theo vai trò.
- Mỗi mục chỉ giữ thông tin đủ để implement/review được, tránh giải thích dài dòng.
- Ưu tiên bullet ngắn; mỗi bullet tối đa 1 ý.
- Khi có nhiều hướng, chỉ giữ tối đa 2 options + 1 recommendation + quyết định cuối.
- Nếu đã chốt quyết định, cập nhật `status.md` thay vì thêm lịch sử dài trong `spec.md`.

## Proactive behavior (bắt buộc)

- AI không chỉ thi hành; phải phân tích độc lập và đề xuất phương án tốt hơn khi phù hợp.
- Trước khi implement, AI phải nêu:
  1. Requirement understanding
  2. Risks/assumptions
  3. 1-2 options + recommendation
- Nếu phát hiện rủi ro business/technical (khả năng lỗi, khó scale, chi phí vận hành cao), AI phải cảnh báo và đề xuất điều chỉnh.
- Nếu thiếu thông tin quan trọng, AI phải hỏi lại; không tự suy đoán để code.
- Cuối mỗi bước, AI phải đề xuất `next best action`.

## AI sẽ làm theo thứ tự

1. Đọc `.spec/config/constitution.md` và `.spec/config/magento-patterns.md`.
2. Tạo/cập nhật spec tại `.spec/features/<tên-feature>/spec.md`.
3. **Dừng bắt buộc để xác nhận** requirement + scope file/module (theo Scope Governance) + gợi ý 1-2 hướng giải pháp.
4. Nếu thiếu thông tin quan trọng: hỏi lại trước khi làm bước tiếp theo.
5. Chỉ khi user trả lời `OK spec` mới tạo `.spec/features/<tên-feature>/plan.md` và `.spec/features/<tên-feature>/tasks.md`.
6. Cập nhật trạng thái tại `.spec/features/<tên-feature>/status.md` sau mỗi bước lớn.
7. Chỉ khi user trả lời `OK implement` mới implement + verify theo testcase, rồi báo cáo kết quả theo DoD.

## Output bắt buộc

1. `files changed`
2. `lý do thay đổi`
3. `verify steps đã chạy`
4. `kết quả testcase`
5. `xác nhận đạt Definition of Done (DoD)`

## Prompt nhanh theo ngữ cảnh

### Bắt đầu feature mới

```text
Đọc `.spec/config/constitution.md` và `.spec/config/magento-patterns.md`.
Tạo/cập nhật spec tại `.spec/features/<tên-feature>/spec.md`.
Nếu task cần review/implement/debug, chọn skill phù hợp theo `.spec/skills/skills-list.md`.
Viết tài liệu ngắn gọn theo `.spec/features/_template/*`, không lặp ý giữa spec/plan/tasks.
DỪNG và hỏi xác nhận bắt buộc: requirement understanding + scope sửa + điểm còn thiếu + gợi ý 1-2 hướng + khuyến nghị hướng nên chọn.
Chỉ khi tôi trả lời "OK spec" mới tạo plan/tasks.
Chỉ khi tôi trả lời "OK implement" mới được code.
```

### Làm task đã có sẵn

```text
Đọc `.spec/config/constitution.md`.
Đọc `.spec/features/<tên-feature>/tasks.md`.
Implement task số <N>.
```

Dùng block này khi:
- Feature đã có `tasks.md` rõ ràng.
- Bạn muốn implement 1 task cụ thể.
- Không dùng cho trường hợp chỉ review code.

### Review/check

```text
Đọc `.spec/config/checklist.md`.
Kiểm tra code của module <tên-module> theo checklist.
```

Dùng block này khi:
- Mục tiêu là review/audit code hiện có.
- Không dùng để yêu cầu implement feature mới.

### Làm lại từ đầu (code đã bị xoá)

```text
Follow .spec workflow in this repo.
Feature: <tên ngắn>
Bối cảnh/vấn đề: <đau ở đâu>
Kết quả mong muốn: <muốn đạt gì>
Phạm vi biết chắc: <module/file, business rules đã chốt>
```

Nếu muốn implement lại nhanh theo spec cũ, thêm:

```text
Dùng lại spec tại `.spec/features/<tên-feature>/spec.md` và implement lại từ đầu.
```

## File tham chiếu

- Rules: `.spec/config/constitution.md`
- Magento patterns: `.spec/config/magento-patterns.md`
- DoD + Scope Governance: xem trong `.spec/config/constitution.md`
- Checklist: `.spec/config/checklist.md`
- Skills map: `.spec/skills/skills-list.md`
- Spec template (feature workspace): `.spec/features/_template/spec.md`
- Plan template (feature workspace): `.spec/features/_template/plan.md`
- Tasks template (feature workspace): `.spec/features/_template/tasks.md`
- Status template (feature workspace): `.spec/features/_template/status.md`
- Task contracts: `.spec/templates/task-contract*.md`
- Feature workspace (mới): `.spec/features/<tên-feature>/`
