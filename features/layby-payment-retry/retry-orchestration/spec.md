# Spec - retry-orchestration

## Input
- Feature: `layby-payment-retry / retry-orchestration`
- Vấn đề: `Retry cho recurring payment đang làm thủ công, dễ miss case và không nhất quán.`
- Mục tiêu: `Tự động lên lịch và thực thi retry charge cho payment dishonored.`
- Module/scope biết chắc: `Laybyland_LaybyStripe`, `Laybyland_LaybyLog`

## Business + scope
- In-scope: `Xác định trigger dishonored, tạo lịch retry, gọi charge retry, log kết quả`
- In-scope: `Đảm bảo idempotency để một kỳ không bị charge trùng`
- Out-of-scope: `Suspend order và gửi email chi tiết (xử lý ở sub-feature khác)`

## Acceptance criteria
- AC-01: Khi installment layby bị dishonored, hệ thống tự tạo retry job theo lịch đã cấu hình/chốt.
- AC-02: Retry job chỉ xử lý đúng installment đang fail và không tạo giao dịch trùng.
- AC-03: Nếu retry thành công, trạng thái installment được cập nhật và dừng escalation chain.
- AC-04: Mỗi lần retry tạo log chuẩn trong `Laybyland_LaybyLog` gồm: order, installment, attempt, kết quả, lý do lỗi.

## Decision
- Requirement understanding:
  - Cần một flow tự động, không phụ thuộc thao tác thủ công của team vận hành.
  - Retry là phần nền tảng cho thông báo và escalation nên phải hoàn thành trước.
  - Rule "thất bại liên tiếp" cần nguồn dữ liệu attempt chính xác và truy vết được.
- Risks/assumptions:
  - Chưa chốt khoảng cách retry (ví dụ 24h hay theo cấu hình admin).
  - Chưa chốt timeout/retry khi gateway phản hồi trạng thái không xác định.
- Approach đã chốt: `Dùng cron/job orchestration trong module layby hiện có + log tập trung trong Laybyland_LaybyLog`.
- Business rules chính:
  - Chỉ retry cho layby recurring payment bị dishonored.
  - Retry attempt phải tuần tự, không song song cho cùng installment.
- Scope được phép sửa: `app/code/Laybyland/LaybyStripe`, `app/code/Laybyland/LaybyLog`

## Testcase
- Happy: TC-01 `Dishonored -> tạo retry job -> retry thành công`, TC-02 `Retry fail lần 1 -> chờ lần 2`
- Edge: TC-03 `Cron chạy overlap nhưng không tạo charge trùng`
- Negative: TC-04 `Gateway timeout/exception -> log lỗi đầy đủ, không mark thành công`

## Câu hỏi cần xác nhận trước OK spec
- Retry schedule cụ thể là gì (sau bao lâu cho lần 1, lần 2)?
- Tổng số lần retry tối đa là 2 hay còn lần khác ngoài rule suspend sau fail thứ 2 liên tiếp?
- "Thất bại liên tiếp" được tính theo cùng installment hay toàn bộ order?
