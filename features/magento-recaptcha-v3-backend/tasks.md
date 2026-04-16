# Feature Tasks - magento-recaptcha-v3-backend

> Tham chiếu AC/testcase: `spec.md`. Tham chiếu thiết kế: `plan.md`.

## Task 1 - Scaffold module Secomm_RecaptchaGuard + verifier contract
- Scope: `app/code/Secomm/RecaptchaGuard/`
- Acceptance criteria:
  - Có module mới `Secomm_RecaptchaGuard` với cấu trúc tối thiểu hợp lệ (`registration.php`, `etc/module.xml`, `etc/di.xml`).
  - Có interface/service verifier tách biệt (`RecaptchaVerifierInterface`) để module khác gọi reusable.
  - Không dùng ObjectManager trực tiếp.
- Verify:
  - `bin/magento module:status Secomm_RecaptchaGuard`
  - `bin/magento setup:di:compile` không lỗi liên quan module.

## Task 2 - Core config adapter + Google verify client
- Scope: `Model/Config/*`, `Model/RecaptchaVerifier.php`, HTTP client wrapper
- Acceptance criteria:
  - Đọc config từ core Magento reCAPTCHA (secret + tham số verify cần thiết) qua adapter riêng.
  - Gọi `siteverify` và parse response chuẩn hoá.
  - Enforce policy: fail-closed; validate `score >= 0.5` + `action` đúng.
  - Không log secret key/token raw đầy đủ.
- Verify:
  - Unit/manual mock: token hợp lệ -> pass; token invalid -> fail.
  - Mock timeout/network -> fail-closed với reason rõ.

## Task 3 - Tích hợp flow customer login/register (headless API-first)
- Scope: plugin/service binding tại endpoint login/register thực tế
- Acceptance criteria:
  - Guard chạy trước business logic ở cả 2 flow customer login + register.
  - Nhận `recaptcha_token` theo contract request đã chốt.
  - Fail verify trả lỗi API rõ ràng, không crash.
- Verify:
  - Manual/API test login với token hợp lệ -> pass.
  - Manual/API test register với token hợp lệ -> pass.
  - Token thiếu/invalid -> bị chặn.

## Task 4 - Action mapping + lỗi chuẩn hóa cho FE
- Scope: mapping rule, exception/error response formatter
- Acceptance criteria:
  - Mỗi flow có expected action rõ (`customer_login`, `customer_register` hoặc action name đã chốt).
  - Action mismatch bị chặn đúng reason.
  - Error response nhất quán để FE xử lý UX (retry/show message) dễ dàng.
- Verify:
  - Test action mismatch -> fail.
  - Kiểm tra format response lỗi giống nhau ở login/register.

## Task 5 - Regression + security checks theo spec
- Scope: flow login/register và logging
- Acceptance criteria:
  - TC-01..TC-07 trong `spec.md` có kết quả Pass/Fail rõ ràng.
  - Không ảnh hưởng flow admin login và endpoint ngoài scope.
  - Không lộ secret/token trong log.
- Verify:
  - Chạy test thủ công/API cho login/register + kiểm tra log.
  - Smoke test admin login không bị guard ảnh hưởng.

## Completion report format
1. Files changed
2. Lý do thay đổi
3. Verify steps đã chạy
4. Kết quả testcase
5. Xác nhận đạt DoD
