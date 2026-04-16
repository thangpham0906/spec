# Feature Plan - magento-recaptcha-v3-backend

> Tham chiếu nghiệp vụ/AC: `spec.md`. File này chỉ mô tả technical design.

## Mục tiêu kỹ thuật
- Tạo module reusable (đề xuất: `Secomm_RecaptchaGuard`) cung cấp service verify reCAPTCHA v3 cho bối cảnh headless API-first.
- Áp lớp guard cho 2 flow phase 1: customer login/register; enforce policy fail-closed và validate `score >= 0.5` + `action` đúng.
- Ưu tiên dùng config core Magento reCAPTCHA, không tạo system config mới trong phase 1.

## Thiết kế giải pháp
1. **Module reusable + service contract nội bộ**
   - Tạo module `app/code/Secomm/RecaptchaGuard` với các thành phần:
     - `Api/RecaptchaVerifierInterface`
     - `Model/RecaptchaVerifier` (gọi Google siteverify qua HTTP client)
     - `Model/Config/CoreRecaptchaConfig` (adapter đọc config core Magento reCAPTCHA)
     - `Model/Dto/RecaptchaVerificationResult` (hoặc struct nội bộ tương đương)
   - Service trả kết quả chuẩn hoá gồm: `isValid`, `score`, `action`, `errorCodes`, `rawReason` (không log secret/token đầy đủ).

2. **Đọc config core Magento reCAPTCHA (không tạo config mới)**
   - Mapping config path core sẽ chốt khi code sau khi đối chiếu module reCAPTCHA hiện có trong project.
   - Tối thiểu cần lấy:
     - secret key dùng cho backend verify
     - threshold (nếu core có; nếu không có thì hard default 0.5 ở module)
     - cờ enable theo form/flow (nếu core hỗ trợ mapping flow tương ứng)
   - Fallback behavior:
     - thiếu secret key hoặc config bắt buộc -> fail verification theo policy fail-closed.

3. **Verify token qua Google API**
   - Endpoint: `https://www.google.com/recaptcha/api/siteverify`.
   - Payload tối thiểu: `secret`, `response` (`recaptcha_token`), và `remoteip` nếu lấy được từ request context.
   - Kiểm tra response:
     - `success === true`
     - `score >= 0.5`
     - `action` khớp expected action của flow (`customer_login` / `customer_register`, tên action chốt ở task đầu)
   - Timeout/network lỗi: trả fail có reason rõ ràng (fail-closed).

4. **Điểm tích hợp cho flow headless login/register**
   - Ưu tiên tích hợp tại API layer đang xử lý login/register thực tế của hệ thống (GraphQL resolver/REST endpoint/custom service).
   - Cách áp dụng:
     - Plugin `before/around` lên service login/register hiện có, hoặc gọi verifier trực tiếp trong service handler của endpoint.
     - Lấy `recaptcha_token` từ request payload/header theo contract FE-BE đã chốt.
   - Trả lỗi thống nhất khi fail verify (HTTP/API error code rõ ràng, message an toàn).

5. **Observability + security**
   - Log warning/error với correlation id (nếu có), không log secret key và không log full token.
   - Thêm metric/log marker theo reason (`missing_token`, `google_timeout`, `score_too_low`, `action_mismatch`, ...).

## Phạm vi file/module dự kiến sửa
- `app/code/Secomm/RecaptchaGuard/registration.php`
- `app/code/Secomm/RecaptchaGuard/etc/module.xml`
- `app/code/Secomm/RecaptchaGuard/etc/di.xml`
- `app/code/Secomm/RecaptchaGuard/Api/*.php`
- `app/code/Secomm/RecaptchaGuard/Model/**/*.php`
- `app/code/Secomm/RecaptchaGuard/Plugin/**` hoặc adapter tại module endpoint login/register thực tế
- `app/code/Secomm/RecaptchaGuard/composer.json` (khuyến nghị)

## Rủi ro + rollback
- Rủi ro:
  - Mapping sai endpoint login/register headless thực tế -> guard không chạy đúng chỗ.
  - Mapping sai config path core -> verify fail toàn bộ.
  - Fail-closed có thể tăng tỷ lệ chặn nhầm khi Google outage.
- Cách giảm rủi ro:
  - Chốt endpoint map trước khi code Task 3.
  - Viết lớp config adapter có check rõ “missing config” + message nội bộ.
  - Timeout ngắn + message lỗi rõ cho FE retry hợp lý.
- Rollback:
  - Disable module `Secomm_RecaptchaGuard` hoặc tạm tháo plugin binding tại flow login/register.
  - `bin/magento cache:flush` sau rollback.
