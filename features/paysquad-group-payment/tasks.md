# Tasks - paysquad-group-payment

> Không lặp business context. Mỗi task: scope + AC + unit test + verify.
> TDD flow bắt buộc với task có business logic: **viết test trước → implement → test pass → code review**.
> Nếu test fail hoặc review còn Critical/High issue: **DỪNG, không được báo task hoàn thành**.
> - Test fail → phân tích root cause → sửa implementation (không sửa test để "cheat") → chạy lại. Tối đa **3 lần**; nếu vẫn fail sau 3 lần thì DỪNG, báo người dùng kèm phân tích nguyên nhân.
> - Review Critical/High → sửa implementation → chạy lại test → review lại. Tối đa **3 lần**; nếu vẫn còn issue thì DỪNG, báo người dùng kèm danh sách issue còn lại.
>
> **Task labels:**
> - `[enabler]` — scaffold, config, setup; không có business logic, không cần unit test
> - `[feature]` — có business logic, bắt buộc TDD

---

## - [ ] Task 1 - Module scaffold + Admin config [enabler]

- Depends-on: `—`
- Context: Tạo skeleton module `Laybyland_PaySquad` với đầy đủ cấu hình Admin (Merchant ID, API Secret Key, Webhook Secret, Mode Sandbox/Live). Credentials phải được encrypt at rest.
- Scope: `app/code/Laybyland/PaySquad/`
- Acceptance criteria:
  - `registration.php`, `etc/module.xml` (sequence: Magento_Sales, Magento_Payment, Magento_Checkout), `composer.json` tồn tại
  - `etc/adminhtml/system.xml`: section PaySquad dưới Sales → Payment Methods, có 5 fields: Enabled, Title, Merchant ID, API Secret Key (encrypted), Webhook Secret (encrypted), Mode (Sandbox/Live)
  - `etc/config.xml`: default values (enabled=0, mode=sandbox, title="PaySquad Group Payment")
  - `bin/magento module:enable Laybyland_PaySquad` không lỗi
- Verify:
  - `bin/magento module:enable Laybyland_PaySquad && bin/magento setup:upgrade` → no error
  - `bin/magento cache:clean config` → Admin → Stores → Config → Sales → Payment Methods → thấy section PaySquad
  - Lưu API Secret Key → kiểm tra DB `core_config_data` value bị encrypt (không phải plaintext)

---

## - [ ] Task 2 - Database schema [enabler]

- Depends-on: `Task 1`
- Context: Thêm `paysquad_id` + `contribution_link` vào `sales_order_payment`. Tạo bảng `laybyland_paysquad_transaction` lưu từng contribution lẻ. Amount lưu dạng integer (minor units).
- Scope: `app/code/Laybyland/PaySquad/Setup/db_schema.xml`
- Acceptance criteria:
  - `sales_order_payment` có thêm: `paysquad_id` VARCHAR(255) NULL, `contribution_link` TEXT NULL
  - Bảng `laybyland_paysquad_transaction` có: `transaction_id` INT PK auto-increment, `order_id` INT NOT NULL, `contributor_name` VARCHAR(255), `amount` INT NOT NULL (minor units), `txn_id` VARCHAR(255), `status` VARCHAR(50), `created_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP
  - Index trên `order_id` trong `laybyland_paysquad_transaction`
- Verify:
  - `bin/magento setup:upgrade` → no error
  - `DESCRIBE sales_order_payment` → thấy 2 cột mới
  - `DESCRIBE laybyland_paysquad_transaction` → đúng schema
  - `bin/magento setup:db-declaration:generate-whitelist --module-name=Laybyland_PaySquad` → tạo `db_schema_whitelist.json`

---

## - [ ] Task 3 - Gateway Config + API Client [feature]

- Depends-on: `Task 1`
- Context: `Config.php` đọc system config, build Basic Auth header `base64(MerchantId:ApiSecretKey)`, resolve base URL theo mode. `Client.php` gọi PaySquad API với retry cho 429/5xx, log mọi request/response vào `paysquad_debug.log`. Không log raw credentials.
- Scope:
  - `app/code/Laybyland/PaySquad/Gateway/Config/Config.php`
  - `app/code/Laybyland/PaySquad/Gateway/Http/Client.php`
  - `app/code/Laybyland/PaySquad/Model/AmountConverter.php`
  - `app/code/Laybyland/PaySquad/Logger/Handler.php` (extend `Magento\Framework\Logger\Handler\Base`, fileName=`/var/log/paysquad_debug.log`)
  - `app/code/Laybyland/PaySquad/Logger/Logger.php` (extend `Monolog\Logger` — marker class)
  - `app/code/Laybyland/PaySquad/etc/di.xml` (wire Handler + Logger, inject Logger vào các class)
- Acceptance criteria:
  - `Config::getAuthHeader()` trả về `Authorization: Basic base64(merchantId:apiSecretKey)`
  - `Config::getBaseUrl()` trả về sandbox URL khi mode=sandbox, production URL khi mode=live
  - `Client::call()` retry tối đa 3 lần với exponential backoff khi nhận 429 hoặc 5xx
  - `Client::call()` throw `LocalizedException` với message phù hợp cho 401, 400, 404
  - `Laybyland\PaySquad\Logger\Logger` (concrete class, extend `Monolog\Logger`) ghi vào `var/log/paysquad_debug.log`
  - `AmountConverter::toMinorUnits()` và `fromMinorUnits()` đúng cho các currency (NZD ×100, JPY ×1)
- Unit test (`Test/Unit/Gateway/Config/ConfigTest.php`, `Test/Unit/Model/AmountConverterTest.php`):
  - `ConfigTest`: sandbox mode → đúng URL; live mode → đúng URL; auth header đúng format base64
  - `AmountConverterTest`: NZD 220.50 → 22050; JPY 500 → 500; fromMinorUnits ngược lại; currency không hỗ trợ → exception
- Reference đọc trước khi implement:
  - `config/references/security/payment-gateway.md`
  - `config/references/infrastructure/logging.md`
  - `config/references/network/paysquad.md`
  - `examples/integration/magento-module-custom-logger-blueprint.md`
- Verify sau implement:
  - `./vendor/bin/phpunit app/code/Laybyland/PaySquad/Test/Unit/Gateway/Config/ConfigTest.php` → all pass
  - `./vendor/bin/phpunit app/code/Laybyland/PaySquad/Test/Unit/Model/AmountConverterTest.php` → all pass
- Code review (sau test pass):
  - Dùng skill `magento-code-reviewer`
  - Đối chiếu `config/checklist.md`

---

## - [ ] Task 4 - Payment Gateway: Facade + CommandPool [feature]

- Depends-on: `Task 3`
- Context: Khai báo Payment Facade (`Laybyland_PaySquad`) dùng `Magento\Payment\Model\Method\Adapter`. CommandPool gồm `order`, `authorize`, `capture`, `refund`. `config.xml` set `payment_action=order`, `can_order=1`, `order_status=awaiting_group_payment`. `ConfigProvider.php` cung cấp JS config cho checkout.
- Scope:
  - `app/code/Laybyland/PaySquad/etc/di.xml` (Facade, CommandPool, ValueHandlerPool)
  - `app/code/Laybyland/PaySquad/etc/config.xml` (payment flags — bắt buộc có `<can_order>1</can_order>` và `<payment_action>order</payment_action>`)
  - `app/code/Laybyland/PaySquad/Model/Ui/ConfigProvider.php`
  - `app/code/Laybyland/PaySquad/Gateway/Request/CreatePaySquadBuilder.php`
  - `app/code/Laybyland/PaySquad/Gateway/Request/RefundBuilder.php`
  - `app/code/Laybyland/PaySquad/Gateway/Response/CreateHandler.php`
  - `app/code/Laybyland/PaySquad/Gateway/Validator/CreateValidator.php`
  - `app/code/Laybyland/PaySquad/Observer/SetOrderPendingPayment.php` (event `sales_order_place_after`)
  - `app/code/Laybyland/PaySquad/Setup/Patch/Data/AddAwaitingGroupPaymentStatus.php`
- Acceptance criteria:
  - `CreatePaySquadBuilder::build()` tạo payload đúng: `items[]` (id=SKU, description, price minor units), `currency`, `total` (minor units), `meta` chứa order increment ID, `successRedirectUrl`, `cancelRedirectUrl`
  - `total` phải ≥ minimum contribution của currency — throw exception nếu không đủ
  - `CreateHandler::handle()` lưu `paySquadId` + `contributionLink` vào `sales_order_payment`
  - `RefundBuilder::build()` tạo payload `{ paySquadId, description, reference }`
  - `CreateValidator` pass khi HTTP 201 + `paySquadId` present; fail otherwise
  - Payment method hiển thị trong checkout khi enabled
- Unit test:
  - `Test/Unit/Gateway/Request/CreatePaySquadBuilderTest.php`:
    - Happy path: order với 2 items → payload đúng format, total đúng minor units
    - Edge: total < minimum → exception
    - Edge: item không có imageUrl → field bị omit (không gửi null)
  - `Test/Unit/Gateway/Request/RefundBuilderTest.php`:
    - Happy path: paySquadId + description → payload đúng
- Reference đọc trước khi implement: `config/references/security/payment-gateway.md`
- Verify sau implement:
  - `./vendor/bin/phpunit app/code/Laybyland/PaySquad/Test/Unit/Gateway/` → all pass
  - Checkout → Payment Methods → thấy "PaySquad Group Payment"
  - Place order với PaySquad → order tạo với status `pending_payment`
- Code review: skill `magento-code-reviewer` + `config/checklist.md`

---

## - [ ] Task 5 - Place Order flow + Redirect [feature]

- Depends-on: `Task 4`
- Context: Sau khi order được tạo với status `pending_payment`, gọi PaySquad API Create, lưu `paySquadId` + `contributionLink`, redirect customer đến `contributionLink`. Lỗi API → cancel order + log + hiển thị lỗi thân thiện.
- Scope:
  - `app/code/Laybyland/PaySquad/Controller/Payment/Redirect.php`
  - `app/code/Laybyland/PaySquad/Plugin/` hoặc `Observer/` để hook vào place order flow
- Acceptance criteria:
  - Sau place order thành công: customer bị redirect đến `contributionLink` từ API response (không tự construct URL)
  - API 400 → cancel order, log full error body, hiển thị lỗi thân thiện
  - API 401 → cancel order, log, hiển thị "Please contact support"
  - API 5xx → cancel order, log, hiển thị "Please try again"
  - `successRedirectUrl` trỏ về trang order success của Magento
  - `cancelRedirectUrl` trỏ về trang cart
- Verify:
  - Happy path: place order → redirect đến PaySquad hosted flow URL
  - Simulate API 400 (mock): order bị cancel, customer thấy error message
  - Kiểm tra `var/log/paysquad_debug.log` có log request + response

---

## - [ ] Task 6 - Webhook receiver + Message Queue [feature]

- Depends-on: `Task 3`
- Context: Endpoint `/paysquad/webhook/receive` validate HMAC signature (base64-decode secret trước), trả HTTP 200 ngay, publish payload vào MQ topic `laybyland_paysquad.webhook`. Consumer xử lý async. Idempotency key: `paySquadId` + `eventName`.
- Scope:
  - `app/code/Laybyland/PaySquad/Controller/Webhook/Receive.php`
  - `app/code/Laybyland/PaySquad/Model/Webhook/SignatureValidator.php`
  - `app/code/Laybyland/PaySquad/Model/Webhook/Processor.php`
  - `app/code/Laybyland/PaySquad/etc/communication.xml`
  - `app/code/Laybyland/PaySquad/etc/queue_publisher.xml`
  - `app/code/Laybyland/PaySquad/etc/queue_topology.xml`
  - `app/code/Laybyland/PaySquad/etc/queue_consumer.xml`
- Acceptance criteria:
  - Signature validation: `base64_decode(webhookSecret)` → HMAC key → `base64_encode(hash_hmac('sha256', rawBody, key, true))` so sánh với `X-Paysquad-Signature` bằng `hash_equals()`
  - Sai signature → HTTP 401, log
  - Đúng signature → HTTP 200 ngay (< 10s), publish MQ
  - Idempotency: nếu `paySquadId` + `eventName` đã xử lý → skip, log, return
  - Log mọi webhook nhận được: eventName, paySquadId, timestamp, X-Paysquad-Environment
- Unit test (`Test/Unit/Model/Webhook/SignatureValidatorTest.php`):
  - Happy path: đúng secret + đúng body → valid
  - Negative: sai secret → invalid
  - Negative: đúng secret nhưng body bị tamper → invalid
  - Edge: secret chưa base64-decode → invalid (đảm bảo decode đúng)
- Unit test (`Test/Unit/Model/Webhook/ProcessorTest.php`):
  - Happy path: `paysquad.succeeded` → dispatch SucceededHandler
  - Happy path: `paysquad.failed` → dispatch FailedHandler
  - Idempotency: event đã xử lý → skip
- Reference đọc trước khi implement:
  - `config/references/network/message-queues.md`
  - `config/references/network/paysquad.md`
- Verify sau implement:
  - `./vendor/bin/phpunit app/code/Laybyland/PaySquad/Test/Unit/Model/Webhook/` → all pass
  - POST đến `/paysquad/webhook/receive` với signature sai → HTTP 401
  - POST với signature đúng → HTTP 200, message xuất hiện trong MQ
- Code review: skill `magento-code-reviewer` + `config/checklist.md`

---

## - [ ] Task 7 - Webhook handlers: succeeded / failed / cancelled [feature]

- Depends-on: `Task 6`, `Task 2`
- Context: `SucceededHandler` gọi GET details → lưu contributions vào `laybyland_paysquad_transaction` → tạo 1 Invoice → order `processing`. `FailedHandler` và `CancelledHandler` gọi GET details → log reason/subReason → cancel order + restock. Tất cả phải idempotent.
- Scope:
  - `app/code/Laybyland/PaySquad/Model/Webhook/Handler/SucceededHandler.php`
  - `app/code/Laybyland/PaySquad/Model/Webhook/Handler/FailedHandler.php`
  - `app/code/Laybyland/PaySquad/Model/Webhook/Handler/CancelledHandler.php`
  - `app/code/Laybyland/PaySquad/Model/Transaction/Repository.php`
  - `app/code/Laybyland/PaySquad/Model/Transaction/ResourceModel/Transaction.php`
  - `app/code/Laybyland/PaySquad/Model/Transaction/ResourceModel/Collection.php`
- Acceptance criteria:
  - `SucceededHandler`: GET `/api/merchant/paysquad/{id}` → lưu mỗi contribution (contributor_name=name, amount=minor units, txn_id=stripePaymentId, status) → tạo đúng 1 Invoice → order status `processing`
  - Duplicate `paysquad.succeeded` cho order đã invoice → skip, không tạo Invoice thứ 2
  - `FailedHandler`: log `reason` + `subReason` từ webhook payload → cancel order → restock inventory
  - `CancelledHandler`: log `subReason` → cancel order → restock inventory
  - `Transaction\Repository`: save, getByOrderId, getCollection
- Unit test (`Test/Unit/Model/Webhook/Handler/SucceededHandlerTest.php`):
  - Happy path: GET details trả 3 contributions → 3 rows lưu vào DB → Invoice tạo → order processing
  - Idempotency: order đã có Invoice → không tạo thêm
  - Edge: GET details trả contributions rỗng → không tạo Invoice, log warning
- Unit test (`Test/Unit/Model/Webhook/Handler/FailedHandlerTest.php`):
  - Happy path: reason=Expired → cancel order + restock
  - Happy path: reason=Abandoned → cancel order + restock
- Reference đọc trước khi implement:
  - `config/references/business/order-lifecycle.md`
  - `config/references/core/declarative-schema.md`
  - `config/references/network/paysquad.md`
- Verify sau implement:
  - `./vendor/bin/phpunit app/code/Laybyland/PaySquad/Test/Unit/Model/Webhook/Handler/` → all pass
  - Trigger `paysquad.succeeded` (sandbox) → kiểm tra `laybyland_paysquad_transaction` có rows, order status = processing, Invoice tồn tại
  - Trigger `paysquad.failed` (sandbox) → order bị cancel, inventory restocked
- Code review: skill `magento-code-reviewer` + `config/checklist.md`

---

## - [ ] Task 8 - Refund (Credit Memo) [feature]

- Depends-on: `Task 4`
- Context: Khi Admin tạo Credit Memo, CommandPool gọi refund command → `POST /api/merchant/paysquad/refund`. Chỉ full refund, chỉ khi PaySquad status = `Complete`. Response 202 = async queue, không phải completed ngay.
- Scope:
  - `app/code/Laybyland/PaySquad/Gateway/Request/RefundBuilder.php` (đã tạo Task 4, bổ sung nếu cần)
  - `app/code/Laybyland/PaySquad/Gateway/Validator/RefundValidator.php`
  - `app/code/Laybyland/PaySquad/Gateway/Response/RefundHandler.php`
- Acceptance criteria:
  - Refund command gọi `POST /api/merchant/paysquad/refund` với `{ paySquadId, description, reference }`
  - API 202 → Credit Memo được tạo, Admin thấy message "Refund is being processed asynchronously"
  - API 400 (đã refund hoặc không phải Complete) → throw `LocalizedException`, log
  - API 404 → throw `LocalizedException`, log
  - Không cho phép partial refund (Credit Memo amount < order total → throw exception với message rõ ràng)
- Unit test (`Test/Unit/Gateway/Request/RefundBuilderTest.php` — bổ sung từ Task 4):
  - Partial refund attempt → exception
  - Full refund → payload đúng
- Verify sau implement:
  - `./vendor/bin/phpunit app/code/Laybyland/PaySquad/Test/Unit/Gateway/Request/RefundBuilderTest.php` → all pass
  - Admin tạo Credit Memo cho PaySquad order → thấy message async
  - Simulate API 400 → Admin thấy error, Credit Memo không được tạo
- Code review: skill `magento-code-reviewer` + `config/checklist.md`

---

## - [ ] Task 9 - My Account: progress bar + contributor list + share link [enabler]

- Depends-on: `Task 7`
- Context: Tại trang Order Detail trong My Account, hiển thị progress bar (% đóng góp), danh sách contributors (name + amount formatted), và nút "Copy Link". Khi order `processing`: progress = 100%, ẩn share link. Block dùng `Magento\Sales\Block\Order\Info` để tự load order từ registry, inject vào `referenceContainer name="content"` sau `sales.order.view`.
- Scope:
  - `app/code/Laybyland/PaySquad/ViewModel/Order/GroupPayment.php`
  - `app/code/Laybyland/PaySquad/view/frontend/layout/sales_order_view.xml`
  - `app/code/Laybyland/PaySquad/view/frontend/templates/order/group-payment.phtml`
  - `app/code/Laybyland/PaySquad/view/frontend/web/js/group-payment.js` (Clipboard API)
- Acceptance criteria:
  - Block class = `Magento\Sales\Block\Order\Info`; inject vào `referenceContainer name="content"` với `after="sales.order.view"`
  - Template dùng `$block->getOrder()` (không phải `$block->getData('order')`)
  - `isPaysquadOrder()` check `paysquad_id` column trước, fallback sang `additional_information['paySquadId']`
  - `getContributionLink()` check `contribution_link` column trước, fallback sang `additional_information['contributionLink']`
  - Progress bar hiển thị `SUM(amount) / order_grand_total × 100` %
  - Contributor list: name + amount formatted từ minor units (ví dụ: $220.50)
  - "Copy Link" button copy `contribution_link` vào clipboard (Clipboard API + fallback execCommand)
  - Khi order `pending_payment`: progress bar + share link active
  - Khi order `processing`: progress = 100%, share link ẩn
  - Block chỉ render khi order có `paysquad_id` (không hiển thị với order thường)
- Verify:
  - My Account → Order Detail của PaySquad order → thấy progress bar, contributor list, copy link button
  - Click "Copy Link" → clipboard có đúng URL
  - Order ở `processing` → share link không hiển thị, progress = 100%
  - Order thường (không phải PaySquad) → block không hiển thị

---

## - [ ] Task 10 - Admin Order: contributions section [enabler]

- Depends-on: `Task 7`
- Context: Trong Admin Order Detail, thêm section "PaySquad Contributions" hiển thị danh sách contributions từ `laybyland_paysquad_transaction` để Admin reconcile.
- Scope:
  - `app/code/Laybyland/PaySquad/Block/Adminhtml/Order/Contributions.php`
  - `app/code/Laybyland/PaySquad/view/adminhtml/layout/sales_order_view.xml`
  - `app/code/Laybyland/PaySquad/view/adminhtml/templates/order/contributions.phtml`
- Acceptance criteria:
  - Section "PaySquad Contributions" hiển thị trong Admin Order Detail khi order có `paysquad_id`
  - Bảng có columns: Contributor Name, Amount (formatted), Transaction ID (stripePaymentId), Status, Created At
  - Không có contribution → hiển thị "No contributions recorded yet."
  - Order thường → section không hiển thị
- Verify:
  - Admin → Orders → chọn PaySquad order → thấy section "PaySquad Contributions"
  - Sau khi webhook succeeded xử lý → contributions hiển thị đúng
  - Order chưa có contribution → thấy "No contributions recorded yet."

---

## - [ ] Task 11 - Checkout UI: payment renderer [enabler]

- Depends-on: `Task 4`
- Context: Thêm PaySquad vào danh sách payment methods tại checkout với KnockoutJS component. Hiển thị description giải thích group payment.
- Scope:
  - `app/code/Laybyland/PaySquad/view/frontend/layout/checkout_index_index.xml`
  - `app/code/Laybyland/PaySquad/view/frontend/web/js/view/payment/method-renderer/paysquad.js`
  - `app/code/Laybyland/PaySquad/view/frontend/web/template/payment/paysquad.html`
  - `app/code/Laybyland/PaySquad/view/frontend/requirejs-config.js`
- Acceptance criteria:
  - PaySquad hiển thị trong checkout payment step khi module enabled
  - Có description text giải thích group payment concept
  - Khi module disabled → không hiển thị
  - Chọn PaySquad → click Place Order → trigger đúng flow (Task 5)
- Verify:
  - Checkout → Payment step → thấy "PaySquad Group Payment" với description
  - Disable module trong Admin → PaySquad không còn trong checkout
  - `bin/magento setup:static-content:deploy` → không có JS error

---

## - [ ] Task 12 - My Account: real-time progress polling [feature]

- Depends-on: `Task 9`
- Context: PaySquad không có webhook intermediate cho từng contribution lẻ. Để My Account hiển thị progress real-time khi order còn `pending_payment`, JS polling `GET /paysquad/order/progress` mỗi 5s, lấy data mới nhất từ PaySquad API, cập nhật DOM không reload trang.
- Scope:
  - `app/code/Laybyland/PaySquad/Controller/Order/Progress.php`
  - `app/code/Laybyland/PaySquad/etc/frontend/routes.xml` (route `paysquad` đã có)
  - `app/code/Laybyland/PaySquad/view/frontend/web/js/group-payment.js` (thêm polling logic)
- Acceptance criteria:
  - `GET /paysquad/order/progress?order_id=X` — chỉ chấp nhận GET, customer phải login và là owner của order
  - Không phải owner → HTTP 403 JSON `{"error": "Forbidden"}`
  - Order không có `paysquad_id` → HTTP 400 JSON `{"error": "Not a PaySquad order"}`
  - Controller gọi `GET /api/merchant/paysquad/{paySquadId}` → trả JSON:
    ```json
    {
      "status": "Pending|InProgress|Complete",
      "progressPercent": 45.5,
      "contributions": [
        {"name": "Alice", "amount": "100.00", "status": "Complete"}
      ],
      "isPending": true
    }
    ```
  - JS polling: chỉ chạy khi `isPending = true` trong initial page render
  - Interval: 5 giây; dừng khi `isPending = false` trong response
  - Tab visibility: dừng polling khi tab ẩn (`document.hidden`), tiếp tục khi tab active lại
  - Khi `isPending = false`: cập nhật progress = 100%, ẩn share link, dừng polling
  - Không tạo thêm request nếu request trước chưa complete (tránh queue up)
- Unit test (`Test/Unit/Controller/Order/ProgressTest.php`):
  - Happy path: customer là owner, order có paysquad_id → trả JSON đúng format
  - Negative: customer không phải owner → 403
  - Negative: order không có paysquad_id → 400
- Verify sau implement:
  - My Account → Order Detail (pending_payment) → mở DevTools Network → thấy request đến `/paysquad/order/progress` mỗi 5s
  - Contributor mới đóng góp trên PaySquad → progress bar + list cập nhật trong vòng 5s không reload
  - Ẩn tab → requests dừng; mở lại → tiếp tục
  - Order chuyển `processing` → polling dừng, progress = 100%
- Code review: `config/checklist.md`

---

> Rules bổ sung:
> - Logger: inject `Laybyland\PaySquad\Logger\Logger` (concrete class, không phải PSR interface) để ghi vào `var/log/paysquad_debug.log` — xem `examples/integration/magento-module-custom-logger-blueprint.md`
> - Amount luôn dùng minor units khi giao tiếp với PaySquad API
> - `system.xml` → `cache:clean config` sau khi sửa
> - Không log raw credentials (Merchant ID, API Secret Key, Webhook Secret)
> - Webhook handler phải idempotent — check order/invoice status trước khi thực hiện
> - `payment_action = order` bắt buộc phải có `<can_order>1</can_order>` trong `config.xml` — thiếu flag này API không được gọi
> - My Account block phải dùng `Magento\Sales\Block\Order\Info`, inject vào `referenceContainer name="content"` sau `sales.order.view`; template dùng `$block->getOrder()` không phải `$block->getData('order')`
> - Order status `awaiting_group_payment` tạo qua Data Patch (`Setup/Patch/Data/`), không tạo thủ công
> - Observer `SetOrderPendingPayment` set `status = AddAwaitingGroupPaymentStatus::STATUS_CODE`, không hardcode string
> - Static content deploy phải chỉ định đúng theme: `bin/magento setup:static-content:deploy -f en_AU en_US -t Laybyland/layup`; xóa `pub/static/frontend` trước khi deploy lại nếu CSS không apply
> - Shared logic giữa handlers → extract vào `Model/Service/`; `FailedHandler` + `CancelledHandler` extend `AbstractCancellationHandler`
> - Checkout payment template cần logo trong `<label>` của `payment-method-title` (xem pattern Afterpay)

## Completion report
1. Files changed
2. Lý do thay đổi
3. Unit test results (pass/fail)
4. Code review results (Critical/High issues nếu có)
5. Verify steps + kết quả testcase
6. Xác nhận DoD
