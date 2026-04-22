# Status - customer-notification

- Stage: `draft`
- Last updated: `2026-04-22`

## Last decision
Thông báo khách hàng được tách riêng để đảm bảo rule "đúng lần, không trùng" được test độc lập với charge retry.

## Blockers
Chưa chốt matrix email theo attempt và rule gửi email success recovery.

## Next action
Chốt business matrix email, rồi chuyển `spec-approved`.
