# Spec-Driven Development cho Magento 2.4.8

## Cấu trúc thư mục

```
.spec/
├── README.md                    # File này - hướng dẫn sử dụng
├── config/
│   ├── constitution.md          # Quy tắc chung, coding standard, conventions
│   ├── magento-patterns.md      # Các pattern Magento bắt buộc tuân theo
│   └── checklist.md             # Checklist kiểm tra trước khi hoàn thành
│
├── specs/
│   └── <tên-feature>.md         # Spec của từng feature (bạn viết)
│
├── plans/
│   └── <tên-feature>.md         # Plan kỹ thuật (AI tạo)
│
└── tasks/
    └── <tên-feature>.md         # Danh sách task (AI tạo)
│
└── skills/
    ├── README.md                # Danh sách skills dùng cho project
    └── skills-list.md           # Skill registry + lệnh cài + cách dùng
│
└── templates/
    ├── task-contract.md                 # Template chung anti-vibe coding
    ├── task-contract-crud-module.md     # Mẫu task CRUD admin module
    ├── task-contract-graphql-field.md   # Mẫu task GraphQL field
    ├── task-contract-rest-endpoint.md   # Mẫu task REST endpoint
    └── spec-business-testcase-first.md  # Mẫu spec: business-first + testcase-first
```

## Quy trình làm việc

1. Bạn viết spec trong `specs/<tên-feature>.md`
2. Nói AI: "Đọc .spec/config/constitution.md rồi đọc .spec/specs/<tên-feature>.md"
3. AI tạo plan trong `plans/<tên-feature>.md`
4. AI chia task trong `tasks/<tên-feature>.md`
5. AI implement từng task

## Quy tắc đặt tên file

- Dùng kebab-case: `customer-phone.md`, `custom-shipping.md`
- Tên file trùng nhau giữa specs/, plans/, tasks/ để dễ theo dõi
- Ví dụ: specs/customer-phone.md -> plans/customer-phone.md -> tasks/customer-phone.md

## Cách nói chuyện với AI

## Task Contract (để AI làm đúng output)

- Khi task quan trọng, tạo 1 contract từ `.spec/templates/task-contract.md`.
- Nếu là task Magento cụ thể, copy từ 1 trong 3 mẫu:
  - `.spec/templates/task-contract-crud-module.md`
  - `.spec/templates/task-contract-graphql-field.md`
  - `.spec/templates/task-contract-rest-endpoint.md`
- Sau đó prompt AI đọc contract trước khi sửa code.

Prompt gợi ý:

```text
Đọc `.spec/config/constitution.md` và đọc task contract tại `<path-contract>`.
Chỉ sửa trong phạm vi được phép.
Implement theo Acceptance Criteria.
Báo cáo đúng Output contract.
Nếu thiếu thông tin quan trọng, hỏi lại trước khi sửa code.
```

### Lần đầu (hoặc khi bắt đầu feature mới):

```
Đọc file .spec/config/constitution.md và .spec/config/magento-patterns.md
Sau đó đọc spec tại .spec/specs/<tên-feature>.md
Nếu task cần review/implement/debug, chọn skill phù hợp theo hướng dẫn trong `.spec/skills/skills-list.md`.
Xác nhận bạn hiểu đúng requirements + review scope trước khi làm gì.
Nêu rõ phạm vi được phép sửa: <files/module>.
Thực hiện và báo cáo: (1) files changed, (2) lý do thay đổi, (3) verify steps đã chạy.
Nếu thiếu thông tin quan trọng, hỏi lại trước khi sửa code.
```

### Khi đã có plan và tasks:

```
Đọc .spec/config/constitution.md
Đọc .spec/tasks/<tên-feature>.md
Implement task số <N>.
```

### Khi cần kiểm tra:

```
Đọc .spec/config/checklist.md
Kiểm tra code của module <tên-module> theo checklist.
```

"Dựa theo quy chuẩn trong thư mục .spec/, hãy tạo data patch..."
https://experienceleague.adobe.com/en/docs/commerce

## Skills cho project

Quản lý skill theo project tại:

- `.spec/skills/README.md`
- `.spec/skills/skills-list.md`
