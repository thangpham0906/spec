# Spec - paysquad-group-payment

## Input
- Feature: `PaySquad Group Payment`
- Vấn đề: Khách hàng muốn thanh toán đơn hàng theo nhóm — nhiều người cùng góp tiền cho một đơn
- Mục tiêu: Tích hợp PaySquad vào Magento 2 như một payment method async, order fulfil sau khi nhận webhook `paysquad.succeeded`
- Module/scope biết chắc: `app/code/Laybyland/PaySquad` (module mới)

## Business + scope
- In-scope:
  - Hiển thị PaySquad tại checkout
  - Tạo PaySquad session qua API, redirect customer đến hosted flow
  - Nhận và xử lý webhook (`succeeded`, `failed`, `cancelled`)
  - Tạo Invoice duy nhất khi đủ 100% đóng góp
  - Refund toàn bộ qua Credit Memo
  - Giao diện My Account: progress bar, contributor list, share link
  - Giao diện Admin Order: danh sách contributions
  - Logging tập trung tại `var/log/paysquad_debug.log`
- Out-of-scope:
  - Partial refund (PaySquad chưa hỗ trợ)
  - Reconcile nightly job (có thể làm sau)
  - Webhook refund-completed (PaySquad chưa có)
  - Multi-currency conversion (dùng currency của order)

## Acceptance criteria

**AC-01 — Cấu hình Admin:**
- Admin config có: Merchant_ID, API_Secret_Key, Webhook_Secret, mode (Sandbox/Live)
- API_Secret_Key và Webhook_Secret được encrypt at rest
- Sandbox → base URL `https://api.sandbox.paysquad.co`; Live → `https://api.paysquad.co`
- Dùng sai environment key → 401, không crash silently

**AC-02 — Database:**
- Thêm `paysquad_id` (VARCHAR 255) và `contribution_link` (TEXT) vào `sales_order_payment`
- Tạo bảng `paysquad_transactions`: `transaction_id` PK, `order_id`, `contributor_name`, `amount` (INT minor units), `txn_id`, `status`, `created_at`

**AC-03 — Checkout:**
- PaySquad hiển thị trong danh sách payment methods khi module enabled
- Ẩn khi module disabled

**AC-04 — Place Order:**
- Tạo order với status `pending_payment`
- Gọi `POST /api/merchant/paysquad` với Basic Auth `base64(MerchantId:ApiSecretKey)`, payload gồm `items[]`, `currency`, `total` (minor units), `meta` (order increment ID), `successRedirectUrl`, `cancelRedirectUrl`
- `total` phải ≥ minimum contribution của currency trước khi gọi API
- Lưu `paySquadId` + `contributionLink` (từ response, không tự construct) vào `sales_order_payment`
- Redirect customer đến `contributionLink`
- Lỗi 4xx/5xx → cancel order, log, hiển thị lỗi thân thiện

**AC-05 — Webhook:**
- Endpoint `/paysquad/webhook/receive`
- Validate signature: `base64_decode(Webhook_Secret)` → HMAC key → `base64_encode(hash_hmac('sha256', rawBody, key, true))` so sánh với header `X-Paysquad-Signature` (constant-time)
- Sai signature → HTTP 401, log
- Trả HTTP 200 ngay, xử lý async
- Idempotency key: `paySquadId` + `eventName`
- `paysquad.succeeded` → GET details → lưu contributions → tạo 1 Invoice → order `processing`
- `paysquad.failed` / `paysquad.cancelled` → GET details → log `reason`/`subReason` → cancel order + restock
- Duplicate webhook cho order đã invoice → HTTP 200, không tạo invoice thêm

**AC-06 — Refund:**
- Admin tạo Credit Memo → gọi `POST /api/merchant/paysquad/refund` với `{ paySquadId, description, reference }`
- Chỉ refund được khi PaySquad status = `Complete`; chỉ full refund
- Response 202 → notify Admin refund đang queue
- Lỗi 400/404 → throw Magento payment exception, log

**AC-07 — My Account (Order Detail):**
- Progress bar: % tổng amount đã đóng góp
- Contributor list: name + amount (formatted từ minor units)
- "Copy Link" button: copy `contribution_link` vào clipboard
- Khi order `pending_payment`: progress bar + share link active
- Khi order `processing`: progress bar 100%, ẩn share link

**AC-08 — Admin Order Detail:**
- Section "PaySquad Contributions": Contributor Name, Amount, Transaction ID (stripePaymentId), Status, Created At
- Chưa có contribution → hiển thị "No contributions recorded yet."

**AC-09 — API Client:**
- Basic Auth mọi request
- 401 → log + throw config exception
- 400 → log full error body (field-level errors) + throw validation exception
- 429 → retry exponential backoff
- 5xx → retry exponential backoff → throw gateway exception
- Log mọi request/response vào `var/log/paysquad_debug.log`

**AC-10 — Logging:**
- Mọi log PaySquad → `var/log/paysquad_debug.log`
- Không ghi vào `system.log` hay `exception.log`

## Decision
- Requirement understanding:
  - PaySquad là async payment: order fulfil qua webhook, không phải tại checkout
  - Amount luôn dùng minor units (integer) — cần convert từ Magento decimal
  - Webhook Secret phải base64-decode trước khi dùng làm HMAC key — khác với cách thông thường
  - `paymentCompleteWebhook` phải truyền trong mỗi Create request — PaySquad không tự biết URL
- Risks/assumptions:
  - Webhook có thể đến trước order được lưu xong → cần idempotency + retry-safe handler
  - Refund async (202) → không có webhook confirm — Admin cần biết refund chưa hoàn tất ngay
  - `AsyncOrder` module wrap `PaymentInformationManagementInterface` → after-plugin không được gọi
- Approach đã chốt:
  - Module mới `Laybyland/PaySquad`, không sửa core Magento
  - **Method code: `paysquad`** (không phải `laybyland_paysquad`) để tránh conflict với layby prefix filter
  - Webhook xử lý synchronous (không cần MQ) — handler chạy < 10s
  - `CreateHandler` set session data trực tiếp thay vì dùng after-plugin (vì AsyncOrder)
  - `payment_action = authorize` — không tạo Invoice tại checkout, chờ webhook
- Business rules chính:
  - Chỉ tạo 1 Invoice duy nhất khi đủ 100%
  - Chỉ full refund, chỉ khi PaySquad status = `Complete`
  - Contributions luôn được lưu kể cả khi order đã có Invoice (idempotency tách riêng)
- Scope được phép sửa: `app/code/Laybyland/PaySquad/` (toàn bộ module mới)

## Testcase
- Happy:
  - TC-01: Customer place order → API trả 201 → redirect đến contributionLink
  - TC-02: Webhook `paysquad.succeeded` hợp lệ → GET details → lưu contributions → tạo Invoice → order `processing`
  - TC-03: Admin tạo Credit Memo → API trả 202 → notify Admin
  - TC-04: Customer xem My Account → thấy progress bar 75%, contributor list, copy link
- Edge:
  - TC-05: Webhook `paysquad.succeeded` gửi lại lần 2 → không tạo Invoice thứ 2
  - TC-06: `total` order nhỏ hơn minimum contribution → không gọi API, hiển thị lỗi
  - TC-07: Webhook `paysquad.failed` với reason `Expired` → cancel order + restock
  - TC-08: Webhook `paysquad.cancelled` với subReason `RequestedByCustomer` → cancel order + restock
- Negative:
  - TC-09: Webhook signature sai → HTTP 401, không xử lý
  - TC-10: API trả 401 khi place order → cancel order, log, hiển thị lỗi
  - TC-11: Refund khi PaySquad status ≠ `Complete` → API trả 400 → throw exception, log
  - TC-12: Dùng sandbox key với production URL → 401, log rõ nguyên nhân
