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

### Lần đầu (hoặc khi bắt đầu feature mới):

```
Đọc file .spec/config/constitution.md và .spec/config/magento-patterns.md
Sau đó đọc spec tại .spec/specs/<tên-feature>.md
Xác nhận bạn hiểu đúng requirements trước khi làm gì.
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
