# Spec - customer-notification

## Input
- Feature: `layby-payment-retry / customer-notification`
- Vấn đề: `Thông báo retry failure/success hiện làm thủ công, dễ gửi sai lần hoặc gửi thiếu.`
- Mục tiêu: `Gửi email đúng sự kiện, đúng lần retry, không gửi trùng.`
- Module/scope biết chắc: `Laybyland_Layby`, `Laybyland_LaybyLog`

## Business + scope
- In-scope: `Xác định event cần gửi email trong retry flow và chống gửi trùng`
- In-scope: `Log việc gửi email (sent/skipped/failed) để audit`
- Out-of-scope: `Thiết kế lại template email marketing và translation đa ngôn ngữ nâng cao`

## Acceptance criteria
- AC-01: Sau mỗi retry thất bại, hệ thống gửi đúng email tương ứng với số lần thất bại (attempt index).
- AC-02: Khi retry thành công sau thất bại trước đó, hệ thống gửi email success recovery (nếu business yêu cầu).
- AC-03: Một event chỉ gửi tối đa 1 email cho cùng order/installment/attempt.
- AC-04: Trạng thái gửi email được log đầy đủ trong `Laybyland_LaybyLog`.

## Decision
- Requirement understanding:
  - Mục tiêu chính là đúng thời điểm và đúng số lần, không phải redesign nội dung email.
  - Notification phải bám dữ liệu retry thực tế, không tự suy từ order status tổng quát.
  - Cần trace được lý do "không gửi" để team vận hành không xử lý mù.
- Risks/assumptions:
  - Chưa chốt danh sách template/subject cho từng attempt.
  - Chưa chốt có cần gửi success recovery email hay chỉ notify fail.
- Approach đã chốt: `Event-driven từ retry result + cơ chế idempotency key cho mỗi notification event`.
- Business rules chính:
  - Không gửi lại email nếu event cùng khóa idempotency đã được mark sent.
  - Thứ tự gửi email phải tương thích escalation (không gửi suspend email trước khi suspend thực sự).
- Scope được phép sửa: `app/code/Laybyland/Layby`, `app/code/Laybyland/LaybyLog`

## Testcase
- Happy: TC-01 `Retry fail lần 1 -> gửi email fail #1`, TC-02 `Retry fail lần 2 -> gửi email fail #2`
- Edge: TC-03 `Job retry rerun cùng event -> không gửi email trùng`
- Negative: TC-04 `Lỗi SMTP/provider -> log failed và không mark sent`

## Câu hỏi cần xác nhận trước OK spec
- Có gửi email khi retry thành công không? Nếu có thì gửi ở attempt nào?
- Mapping template/email type theo từng attempt fail là gì?
- Có yêu cầu throttle/cooldown để tránh spam khi nhiều order fail cùng lúc không?
