# Status - order-escalation

- Stage: `draft`
- Last updated: `2026-04-22`

## Last decision
Escalation tách riêng để kiểm soát business rule suspend độc lập, tránh coupling chặt với logic charge.

## Blockers
Chưa chốt mapping state/status khi suspend và hành vi với retry jobs đang pending.

## Next action
Chốt rule escalation còn mở để đạt `spec-approved`.
