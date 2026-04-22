# Spec - order-escalation

## Input
- Feature: `layby-payment-retry / order-escalation`
- Vấn đề: `Chưa có cơ chế tự động suspend order khi retry thất bại liên tiếp theo rule nghiệp vụ.`
- Mục tiêu: `Suspend order đúng điều kiện sau lần thất bại thứ 2 liên tiếp và đảm bảo trạng thái nhất quán.`
- Module/scope biết chắc: `Laybyland_Layby`, `Laybyland_LaybyLog`

## Business + scope
- In-scope: `Tính điều kiện escalation từ lịch sử retry và thực thi suspend order`
- In-scope: `Đồng bộ trạng thái order/layby schedule liên quan khi suspend`
- Out-of-scope: `Quy trình re-activate order thủ công hoặc tự động sau khi suspend`

## Acceptance criteria
- AC-01: Sau 2 lần thất bại liên tiếp đúng rule đã chốt, order layby được chuyển trạng thái suspend tự động.
- AC-02: Escalation chỉ chạy 1 lần cho cùng ngữ cảnh, không suspend lặp.
- AC-03: Khi escalation xảy ra, hệ thống lưu log đầy đủ trigger condition và thời điểm suspend.
- AC-04: Không suspend nhầm các order không thuộc layby recurring retry flow.

## Decision
- Requirement understanding:
  - Escalation là business safety rule, cần deterministic và audit được.
  - Điều kiện "2 fail liên tiếp" phải định nghĩa rõ phạm vi đếm để tránh suspend sai.
  - Escalation phải phụ thuộc kết quả retry thực tế, không dựa vào suy luận trạng thái thiếu dữ liệu.
- Risks/assumptions:
  - Chưa chốt mapping trạng thái cụ thể khi suspend (order state/status, layby status).
  - Chưa chốt việc hủy các retry job pending sau khi order bị suspend.
- Approach đã chốt: `Escalation checker chạy sau mỗi retry result, dựa trên attempt history đáng tin cậy và idempotent lock`.
- Business rules chính:
  - Suspend khi đạt ngưỡng thất bại liên tiếp theo rule đã chốt.
  - Sau khi suspend, không tiếp tục charge retry cho order đó.
- Scope được phép sửa: `app/code/Laybyland/Layby`, `app/code/Laybyland/LaybyLog`

## Testcase
- Happy: TC-01 `Fail #1 -> chưa suspend`, TC-02 `Fail #2 liên tiếp -> suspend thành công`
- Edge: TC-03 `Fail rồi success -> counter reset, không suspend`
- Negative: TC-04 `Order không phải layby recurring -> escalation bị bỏ qua`

## Câu hỏi cần xác nhận trước OK spec
- Suspend dùng state/status nào trong hệ thống hiện tại?
- Khi suspend, có hủy các job retry đang pending không?
- Có gửi thêm thông báo nội bộ (admin/ops) tại thời điểm suspend không?
