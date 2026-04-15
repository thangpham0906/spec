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

## AI sẽ làm theo thứ tự

1. Đọc `.spec/config/constitution.md` và `.spec/config/magento-patterns.md`.
2. Tạo/cập nhật spec tại `.spec/specs/<ten-feature>.md`.
3. Xác nhận requirement + nêu rõ scope file/module trước khi code.
4. Nếu thiếu thông tin quan trọng: hỏi lại trước khi sửa code.
5. Implement + verify theo testcase, rồi báo cáo kết quả.

## Output bắt buộc

1. `files changed`
2. `lý do thay đổi`
3. `verify steps đã chạy`
4. `kết quả testcase`

## Prompt nhanh theo ngữ cảnh

### Bắt đầu feature mới

```text
Đọc `.spec/config/constitution.md` và `.spec/config/magento-patterns.md`.
Đọc spec tại `.spec/specs/<tên-feature>.md`.
Nếu task cần review/implement/debug, chọn skill phù hợp theo `.spec/skills/skills-list.md`.
Trước khi làm: xác nhận requirement + scope sửa.
```

### Làm task đã có sẵn

```text
Đọc `.spec/config/constitution.md`.
Đọc `.spec/tasks/<tên-feature>.md`.
Implement task số <N>.
```

### Review/check

```text
Đọc `.spec/config/checklist.md`.
Kiểm tra code của module <tên-module> theo checklist.
```

## File tham chiếu

- Rules: `.spec/config/constitution.md`
- Magento patterns: `.spec/config/magento-patterns.md`
- Checklist: `.spec/config/checklist.md`
- Skills map: `.spec/skills/skills-list.md`
- Spec template: `.spec/specs/_template.md`
- Task contracts: `.spec/templates/task-contract*.md`
