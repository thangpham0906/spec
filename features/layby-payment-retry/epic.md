# Epic - layby-payment-retry

> File này mô tả boundary tổng thể và danh sách sub-features.
> Không chứa task implement. Mỗi sub-feature có folder riêng với spec/plan/tasks/status của nó.

## Mục tiêu tổng thể
Tự động hoá xử lý thanh toán layby bị dishonored để giảm xử lý thủ công, đảm bảo khách hàng được thông báo đúng thời điểm, và áp dụng escalation nhất quán theo business rule.

## Boundary

### In-scope
- `Laybyland_Layby` - cập nhật trạng thái đơn layby và rule escalation.
- `Laybyland_LaybyStripe` - thực thi retry charge theo lịch.
- `Laybyland_LaybyLog` - ghi log retry/escalation phục vụ audit và vận hành.

### Out-of-scope
- Tích hợp thêm payment gateway mới ngoài Stripe.
- Thay đổi toàn bộ nội dung template email marketing không liên quan retry.
- Sửa các flow thanh toán one-time không thuộc layby recurring.

## Sub-features

| Sub-feature | Folder | Phụ thuộc vào | Stage |
|---|---|---|---|
| Retry orchestration | `layby-payment-retry/retry-orchestration/` | — | `draft` |
| Customer notification | `layby-payment-retry/customer-notification/` | `Retry orchestration` | `draft` |
| Order escalation | `layby-payment-retry/order-escalation/` | `Retry orchestration` | `draft` |

## Dependency order

```
retry-orchestration
    ├── customer-notification
    └── order-escalation
```

## Acceptance criteria tổng (Epic-level)

- AC-E01: Hệ thống tự động phát hiện installment layby bị dishonored và lên lịch retry theo policy đã chốt.
- AC-E02: Mỗi lần retry thất bại/thành công đều có log đủ dữ liệu truy vết trong `Laybyland_LaybyLog`.
- AC-E03: Email thông báo được gửi đúng sự kiện và không gửi trùng.
- AC-E04: Đơn layby bị suspend sau lần thất bại thứ 2 liên tiếp theo đúng business rule đã chốt.
- AC-E05: Flow tự động không làm thay đổi hành vi của các order không thuộc layby recurring.

## Risks tổng

- Rủi ro race condition giữa cron retry và hành động manual của admin (retry/suspend cùng lúc).
- Rủi ro gửi email trùng khi job retry bị chạy overlap hoặc retry lại sau timeout.
- Rủi ro hiểu sai "2 lần thất bại liên tiếp" (trên cùng installment hay trên toàn order) dẫn tới suspend sai.

## Progress

- [ ] `retry-orchestration`
- [ ] `customer-notification`
- [ ] `order-escalation`
