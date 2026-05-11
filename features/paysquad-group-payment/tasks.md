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
- Context: Tạo skeleton module `Secomm_PaySquad` với đầy đủ cấu hình Admin (Merchant ID, API Secret Key, Webhook Secret, Mode Sandbox/Live). Credentials phải được encrypt at rest.
- Scope: `app/code/Secomm/PaySquad/`
- Acceptance criteria:
  - `registration.php`, `etc/module.xml` (sequence: Magento_Sales, Magento_Payment, Magento_Checkout), `composer.json` tồn tại
  - `etc/adminhtml/system.xml`: section PaySquad dưới Sales → Payment Methods, có 5 fields: Enabled, Title, Merchant ID, API Secret Key (encrypted), Webhook Secret (encrypted), Mode (Sandbox/Live)
  - `etc/config.xml`: default values (enabled=0, mode=sandbox, title="PaySquad Group Payment")
  - `bin/magento module:enable Secomm_PaySquad` không lỗi
- Verify:
  - `bin/magento module:enable Secomm_PaySquad && bin/magento setup:upgrade` → no error
  - `bin/magento cache:clean config` → Admin → Stores → Config → Sales → Payment Methods → thấy section PaySquad
  - Lưu API Secret Key → kiểm tra DB `core_config_data` value bị encrypt (không phải plaintext)

---

## - [ ] Task 2 - Database schema [enabler]

- Depends-on: `Task 1`
- Context: Thêm `paysquad_id` + `contribution_link` vào `sales_order_payment`. Tạo bảng `secomm_paysquad_transaction` lưu từng contribution lẻ (và bảng log webhook `secomm_paysquad_webhook_log` nếu dùng schema hiện tại). Amount lưu dạng integer (minor units).
- Scope: `app/code/Secomm/PaySquad/etc/db_schema.xml`
- Acceptance criteria:
  - `sales_order_payment` có thêm: `paysquad_id` VARCHAR(255) NULL, `contribution_link` TEXT NULL
  - Bảng `secomm_paysquad_transaction` có: `transaction_id` INT PK auto-increment, `order_id` INT NOT NULL, `contributor_name` VARCHAR(255), `amount` INT NOT NULL (minor units), `txn_id` VARCHAR(255), `status` VARCHAR(50), `created_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP
  - Index trên `order_id` trong `secomm_paysquad_transaction`
- Verify:
  - `bin/magento setup:upgrade` → no error
  - `DESCRIBE sales_order_payment` → thấy 2 cột mới
  - `DESCRIBE secomm_paysquad_transaction` → đúng schema
  - `bin/magento setup:db-declaration:generate-whitelist --module-name=Secomm_PaySquad` → tạo `db_schema_whitelist.json`

---

## - [ ] Task 3 - Gateway Config + API Client [feature]

- Depends-on: `Task 1`
- Context: `Config.php` đọc system config, build Basic Auth header `base64(MerchantId:ApiSecretKey)`, resolve base URL theo mode. `Client.php` gọi PaySquad API với retry cho 429/5xx, log mọi request/response vào `paysquad_debug.log`. Không log raw credentials.
- Scope:
  - `app/code/Secomm/PaySquad/Gateway/Config/Config.php`
  - `app/code/Secomm/PaySquad/Gateway/Http/Client.php`
  - `app/code/Secomm/PaySquad/Model/AmountConverter.php`
  - `app/code/Secomm/PaySquad/Logger/Handler.php` (extend `Magento\Framework\Logger\Handler\Base`, fileName=`/var/log/paysquad_debug.log`)
  - `app/code/Secomm/PaySquad/Logger/Logger.php` (extend `Monolog\Logger` — marker class)
  - `app/code/Secomm/PaySquad/etc/di.xml` (wire Handler + Logger, inject Logger vào các class)
- Acceptance criteria:
  - `Config::getAuthHeader()` trả về `Authorization: Basic base64(merchantId:apiSecretKey)`
  - `Config::getBaseUrl()` trả về sandbox URL khi mode=sandbox, production URL khi mode=live
  - `Client::call()` retry tối đa 3 lần với exponential backoff khi nhận 429 hoặc 5xx
  - `Client::call()` throw `LocalizedException` với message phù hợp cho 401, 400, 404
  - `Secomm\PaySquad\Logger\Logger` (concrete class, extend `Monolog\Logger`) ghi vào `var/log/paysquad_debug.log`
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
  - `./vendor/bin/phpunit app/code/Secomm/PaySquad/Test/Unit/Gateway/Config/ConfigTest.php` → all pass
  - `./vendor/bin/phpunit app/code/Secomm/PaySquad/Test/Unit/Model/AmountConverterTest.php` → all pass
- Code review (sau test pass):
  - Dùng skill `magento-code-reviewer`
  - Đối chiếu `config/checklist.md`

---

## - [ ] Task 4 - Payment Gateway: Facade + CommandPool [feature]

- Depends-on: `Task 3`
- Context: Khai báo Payment Facade (`Secomm_PaySquad`) dùng `Magento\Payment\Model\Method\Adapter`. CommandPool gồm `order`, `authorize`, `capture`, `refund`. `config.xml` set `payment_action=order`, `can_order=1`, `order_status` theo patch/status hiện tại. `ConfigProvider.php` cung cấp JS config cho checkout.
- Scope:
  - `app/code/Secomm/PaySquad/etc/di.xml` (Facade, CommandPool, ValueHandlerPool)
  - `app/code/Secomm/PaySquad/etc/config.xml` (payment flags — bắt buộc có `<can_order>1</can_order>` và `<payment_action>order</payment_action>`)
  - `app/code/Secomm/PaySquad/Model/Ui/ConfigProvider.php`
  - `app/code/Secomm/PaySquad/Gateway/Request/CreatePaySquadBuilder.php`
  - `app/code/Secomm/PaySquad/Gateway/Request/RefundBuilder.php`
  - `app/code/Secomm/PaySquad/Gateway/Response/CreateHandler.php`
  - `app/code/Secomm/PaySquad/Gateway/Validator/CreateValidator.php`
  - `app/code/Secomm/PaySquad/Observer/SetOrderPendingPayment.php` (event `sales_order_place_after`)
  - `app/code/Secomm/PaySquad/Setup/Patch/Data/AddAwaitingGroupPaymentStatus.php`
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
  - `./vendor/bin/phpunit app/code/Secomm/PaySquad/Test/Unit/Gateway/` → all pass
  - Checkout → Payment Methods → thấy "PaySquad Group Payment"
  - Place order với PaySquad → order tạo với status `pending_payment`
- Code review: skill `magento-code-reviewer` + `config/checklist.md`

---

## - [ ] Task 5 - Place Order flow + Redirect [feature]

- Depends-on: `Task 4`
- Context: Sau khi order được tạo với status `pending_payment`, gọi PaySquad API Create, lưu `paySquadId` + `contributionLink`, redirect customer đến `contributionLink`. Lỗi API → cancel order + log + hiển thị lỗi thân thiện.
- Scope:
  - `app/code/Secomm/PaySquad/Controller/Payment/Redirect.php`
  - `app/code/Secomm/PaySquad/Plugin/` hoặc `Observer/` để hook vào place order flow
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

## - [ ] Task 6 - Webhook receiver (xử lý đồng bộ trong request) [feature]

- Depends-on: `Task 3`
- Context: Endpoint `POST /paysquad/webhook/receive` validate HMAC (base64-decode secret trước), gọi `Processor` xử lý ngay trong request (không Message Queue). Idempotency: bảng log webhook + logic Processor (không có `communication.xml` / `queue_*`).
- Scope:
  - `app/code/Secomm/PaySquad/Controller/Webhook/Receive.php`
  - `app/code/Secomm/PaySquad/Model/Webhook/SignatureValidator.php`
  - `app/code/Secomm/PaySquad/Model/Webhook/Processor.php`
- Acceptance criteria:
  - Signature: `base64_decode(webhookSecret)` → HMAC key → `base64_encode(hash_hmac('sha256', rawBody, key, true))` so sánh với `X-Paysquad-Signature` bằng `hash_equals()`
  - Sai signature → HTTP 401, log
  - Đúng signature → HTTP 200, xử lý qua Processor (handlers succeeded/failed/cancelled)
  - Log webhook nhận: eventName, paySquadId, timestamp, header environment (theo code hiện tại)
- Unit test (`Test/Unit/Model/Webhook/SignatureValidatorTest.php`, `ProcessorTest.php`): giữ như cũ (đúng với flow sync)
- Reference đọc trước khi implement:
  - `config/references/network/paysquad.md`
- Verify sau implement:
  - `./vendor/bin/phpunit app/code/Secomm/PaySquad/Test/Unit/Model/Webhook/` → all pass
  - POST `/paysquad/webhook/receive` signature sai → HTTP 401
  - POST signature đúng → HTTP 200, order/invoice cập nhật theo event (không kiểm tra MQ)
- Code review: skill `magento-code-reviewer` + `config/checklist.md`

---

## - [ ] Task 7 - Webhook handlers: succeeded / failed / cancelled [feature]

- Depends-on: `Task 6`, `Task 2`
- Context: `SucceededHandler` gọi GET details → lưu contributions vào `secomm_paysquad_transaction` → tạo 1 Invoice → order `processing`. `FailedHandler` và `CancelledHandler` gọi GET details → log reason/subReason → cancel order + restock. Tất cả phải idempotent.
- Scope:
  - `app/code/Secomm/PaySquad/Model/Webhook/Handler/SucceededHandler.php`
  - `app/code/Secomm/PaySquad/Model/Webhook/Handler/FailedHandler.php`
  - `app/code/Secomm/PaySquad/Model/Webhook/Handler/CancelledHandler.php`
  - `app/code/Secomm/PaySquad/Model/Transaction/Repository.php`
  - `app/code/Secomm/PaySquad/Model/Transaction/ResourceModel/Transaction.php`
  - `app/code/Secomm/PaySquad/Model/Transaction/ResourceModel/Collection.php`
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
  - `./vendor/bin/phpunit app/code/Secomm/PaySquad/Test/Unit/Model/Webhook/Handler/` → all pass
  - Trigger `paysquad.succeeded` (sandbox) → kiểm tra `secomm_paysquad_transaction` có rows, order status = processing, Invoice tồn tại
  - Trigger `paysquad.failed` (sandbox) → order bị cancel, inventory restocked
- Code review: skill `magento-code-reviewer` + `config/checklist.md`

---

## - [ ] Task 8 - Refund (Credit Memo) [feature]

- Depends-on: `Task 4`
- Context: Khi Admin tạo Credit Memo, CommandPool gọi refund command → `POST /api/merchant/paysquad/refund`. Chỉ full refund, chỉ khi PaySquad status = `Complete`. Response 202 = async queue, không phải completed ngay.
- Scope:
  - `app/code/Secomm/PaySquad/Gateway/Request/RefundBuilder.php` (đã tạo Task 4, bổ sung nếu cần)
  - `app/code/Secomm/PaySquad/Gateway/Validator/RefundValidator.php`
  - `app/code/Secomm/PaySquad/Gateway/Response/RefundHandler.php`
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
  - `./vendor/bin/phpunit app/code/Secomm/PaySquad/Test/Unit/Gateway/Request/RefundBuilderTest.php` → all pass
  - Admin tạo Credit Memo cho PaySquad order → thấy message async
  - Simulate API 400 → Admin thấy error, Credit Memo không được tạo
- Code review: skill `magento-code-reviewer` + `config/checklist.md`

---

## - [ ] Task 9 - My Account: progress bar + contributor list + share link [enabler]

- Depends-on: `Task 7`
- Context: Tại trang Order Detail trong My Account, hiển thị progress bar (% đóng góp), danh sách contributors (name + amount formatted), và nút "Copy Link". Dữ liệu theo DB tại thời điểm load trang (polling real-time nằm backlog — đã xóa khỏi danh sách task). Khi order `processing`: progress = 100%, ẩn share link. Block dùng `Magento\Sales\Block\Order\Info` để tự load order từ registry, inject vào `referenceContainer name="content"` sau `sales.order.view`.
- Scope:
  - `app/code/Secomm/PaySquad/ViewModel/Order/GroupPayment.php`
  - `app/code/Secomm/PaySquad/view/frontend/layout/sales_order_view.xml`
  - `app/code/Secomm/PaySquad/view/frontend/templates/order/group-payment.phtml`
  - `app/code/Secomm/PaySquad/view/frontend/web/js/group-payment.js` (Clipboard API)
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

## - [ ] Task 10 - Admin Order: tab contributions / transactions [enabler]

- Depends-on: `Task 7`
- Context: Trong Admin Order Detail, tab hiển thị danh sách contributions từ bảng `secomm_paysquad_transaction` (code: tab "PaySquad Transactions", không phải block Contributions riêng).
- Scope:
  - `app/code/Secomm/PaySquad/Block/Adminhtml/Order/View/Tab/Transactions.php`
  - `app/code/Secomm/PaySquad/view/adminhtml/layout/sales_order_view.xml`
  - `app/code/Secomm/PaySquad/view/adminhtml/templates/order/view/tab/transactions.phtml`
- Acceptance criteria:
  - Tab hiển thị khi order có `paysquad_id` (hoặc additional_information tương đương)
  - Bảng: Contributor Name, Amount (formatted), Transaction ID, Status, Created At
  - Không có contribution → copy kiểu "No contributions recorded yet." hoặc tương đương trong template
  - Order thường → tab không hiển thị
- Verify:
  - Admin → Orders → PaySquad order → mở tab PaySquad Transactions
  - Sau webhook succeeded → rows đúng
  - Chưa có contribution → empty state đúng

---

## - [ ] Task 11 - Checkout UI: payment renderer [enabler]

- Depends-on: `Task 4`
- Context: Thêm PaySquad vào danh sách payment methods tại checkout với KnockoutJS component. Hiển thị description giải thích group payment.
- Scope:
  - `app/code/Secomm/PaySquad/view/frontend/layout/checkout_index_index.xml`
  - `app/code/Secomm/PaySquad/view/frontend/web/js/view/payment/method-renderer/paysquad.js`
  - `app/code/Secomm/PaySquad/view/frontend/web/template/payment/paysquad.html`
  - `app/code/Secomm/PaySquad/view/frontend/requirejs-config.js`
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

> **Backlog (chưa có trong code):** polling `GET /paysquad/order/progress` mỗi 5s + cập nhật DOM real-time theo `spec.md` AC-07 — không tạo task riêng cho đến khi implement.

---

> Rules bổ sung:
> - Logger: inject `Secomm\PaySquad\Logger\Logger` (concrete class, không phải PSR interface) để ghi vào `var/log/paysquad_debug.log` — xem `examples/integration/magento-module-custom-logger-blueprint.md`
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
