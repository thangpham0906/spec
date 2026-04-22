# Status - retry-orchestration

- Stage: `draft`
- Last updated: `2026-04-22`

## Last decision
Tách retry orchestration thành sub-feature nền tảng để các flow notification/escalation dùng chung dữ liệu attempt.

## Blockers
Chưa chốt retry schedule và định nghĩa chính xác của "2 lần thất bại liên tiếp".

## Next action
Chốt business rules còn mở trong `spec.md`, sau đó mới chuyển stage `spec-approved`.
