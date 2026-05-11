# Plan - paysquad-group-payment

> Technical design only. Business context → `spec.md`.

## Mục tiêu kỹ thuật
- Module `Secomm/PaySquad` (`Secomm_PaySquad`) tích hợp PaySquad như payment method (order fulfil qua webhook / cron sync)
- Implement Payment Gateway pattern (Facade + CommandPool) với authorize/capture/refund
- Webhook receiver: validate signature → xử lý **đồng bộ** trong request qua `Processor` (không Message Queue trong implementation hiện tại)
- Custom logger riêng tại `var/log/paysquad_debug.log`
- Frontend: progress bar + contributor list + share link tại My Account và Admin Order

## Blueprint tham khảo
- `examples/integration/custom-payment-offline-blueprint.md` — Payment Adapter pattern (Facade, ValueHandlerPool, di.xml)
- `examples/integration/magento-module-bulk-email-rabbitmq-blueprint.md` — Message Queue async consumer
- `examples/integration/magento-module-custom-logger-blueprint.md` — Custom Monolog logger

## References sử dụng
- `config/references/security/payment-gateway.md` — Facade, CommandPool, RequestBuilder, TransferFactory, ResponseHandler
- `config/references/business/laybyland-payment-integration.md` — **Laybyland-specific patterns**: whitelist observers, method code naming, JS renderer, async flow
- `config/references/infrastructure/logging.md` — VirtualType logger → `paysquad_debug.log`
- `config/references/core/declarative-schema.md` — `db_schema.xml` cho `paysquad_transactions` + alter `sales_order_payment`

## Thiết kế

### 1. Module Structure

```
app/code/Secomm/PaySquad/
├── etc/
│   ├── module.xml                  # sequence: Magento_Sales, Magento_Payment, Magento_Checkout
│   ├── config.xml                  # payment flags: can_authorize, can_capture, can_refund, order_status=pending_payment
│   ├── di.xml                      # Facade, CommandPool, Logger, plugins
│   ├── adminhtml/
│   │   └── system.xml              # Admin config: Merchant ID, API Secret Key, Webhook Secret, Mode
│   ├── frontend/
│   │   ├── di.xml
│   │   └── routes.xml
│   └── crontab.xml                 # sync fallback (giờ — theo implementation)
│   # Không dùng communication.xml / queue_* trong implementation hiện tại
├── Gateway/
│   ├── Config/
│   │   └── Config.php              # Đọc system config, build Basic Auth header, resolve base URL
│   ├── Http/
│   │   └── Client.php              # Gọi PaySquad API (Zend/Curl), retry 429/5xx, log request/response
│   ├── Request/
│   │   ├── CreatePaySquadBuilder.php   # Build payload Create: items, currency, total (minor units), meta, redirectUrls
│   │   └── RefundBuilder.php           # Build payload Refund: paySquadId, description, reference
│   ├── Response/
│   │   ├── CreateHandler.php       # Lưu paySquadId + contributionLink vào sales_order_payment
│   │   └── RefundHandler.php       # Xử lý 202 Accepted
│   └── Validator/
│       ├── CreateValidator.php     # Validate HTTP 201, check paySquadId present
│       └── RefundValidator.php     # Validate HTTP 202
├── Model/
│   ├── Ui/
│   │   └── ConfigProvider.php      # JS config cho checkout (code, title, description)
│   ├── Webhook/
│   │   ├── SignatureValidator.php  # base64_decode(secret) → hash_hmac → base64_encode → hash_equals
│   │   ├── Processor.php           # Dispatch event theo eventName, idempotency check
│   │   ├── Handler/
│   │   │   ├── SucceededHandler.php    # GET details → save contributions → create Invoice → order processing
│   │   │   ├── FailedHandler.php       # GET details → log reason/subReason → cancel + restock
│   │   │   └── CancelledHandler.php    # GET details → log subReason → cancel + restock
│   ├── Transaction/
│   │   ├── Repository.php          # CRUD paysquad_transactions
│   │   └── ResourceModel/
│   │       ├── Transaction.php
│   │       └── Collection.php
│   └── AmountConverter.php         # Magento decimal ↔ minor units (integer)
├── Controller/
│   ├── Webhook/
│   │   └── Receive.php             # POST /paysquad/webhook/receive → validate sig → Processor → HTTP 200
│   └── Payment/
│       └── Redirect.php            # Redirect customer đến contributionLink sau place order
├── Observer/
│   └── DataAssignObserver.php      # Lưu additional_information khi place order
├── Block/
│   ├── Order/
│   │   └── PaySquadInfo.php        # ViewModel cho My Account order detail
│   └── Adminhtml/
│       └── Order/
│           └── Contributions.php   # Block cho Admin order detail
├── ViewModel/
│   └── Order/
│       └── GroupPayment.php        # Progress %, contributor list, contribution_link
├── Setup/
│   └── db_schema.xml               # Alter sales_order_payment + create paysquad_transactions
├── view/
│   ├── frontend/
│   │   ├── layout/
│   │   │   ├── checkout_index_index.xml    # Thêm payment renderer
│   │   │   └── sales_order_view.xml        # Thêm PaySquad block vào My Account
│   │   ├── web/js/
│   │   │   └── view/payment/method-renderer/paysquad.js  # KO component
│   │   └── templates/
│   │       └── order/
│   │           └── group-payment.phtml     # Progress bar + contributor list + copy link
│   └── adminhtml/
│       ├── layout/
│       │   └── sales_order_view.xml        # Thêm Contributions block vào Admin order
│       └── templates/
│           └── order/
│               └── contributions.phtml
└── Test/Unit/
    ├── Gateway/
    │   ├── Request/CreatePaySquadBuilderTest.php
    │   └── Request/RefundBuilderTest.php
    ├── Model/
    │   ├── Webhook/SignatureValidatorTest.php
    │   ├── Webhook/ProcessorTest.php
    │   └── AmountConverterTest.php
    └── Gateway/
        └── Config/ConfigTest.php
```

### 2. Data Flow

**Place Order:**
```
Customer → Place Order
  → Magento Order (pending_payment)
  → CreatePaySquadBuilder: convert total → minor units, build items[], meta={orderId}
  → Gateway\Http\Client: POST /api/merchant/paysquad (Basic Auth)
  → CreateHandler: save paySquadId + contributionLink → sales_order_payment
  → Redirect → contributionLink
```

**Webhook:**
```
PaySquad → POST /paysquad/webhook/receive
  → SignatureValidator: base64_decode(secret) → HMAC → compare X-Paysquad-Signature
  → HTTP 200 ngay
  → Publisher: publish('laybyland.paysquad.webhook', payload)
  → Consumer: Processor::process()
    → idempotency check (paySquadId + eventName)
    → GET /api/merchant/paysquad/{id} để lấy full state
    → dispatch Handler theo eventName
```

**Refund:**
```
Admin → Credit Memo
  → CommandPool::refund
  → RefundBuilder: { paySquadId, description, reference }
  → POST /api/merchant/paysquad/refund
  → 202 Accepted → notify Admin (async queue)
```

### 3. Authentication

Mọi API call dùng HTTP Basic Auth:
```
Authorization: Basic base64(MerchantId:ApiSecretKey)
```
`Config.php` build header này từ encrypted config values. Không bao giờ log raw credentials.

### 4. Amount Conversion

`AmountConverter.php`:
- `toMinorUnits(float $amount, string $currency): int` — nhân theo currency precision (hầu hết ×100, JPY ×1)
- `fromMinorUnits(int $amount, string $currency): float` — chia ngược lại để hiển thị

### 5. Message Queue

- Topic: `laybyland.paysquad.webhook`
- Connection: `db` (dev/staging), `amqp` (production)
- Consumer: `PaySquadWebhookConsumer` → `Processor::process()`
- Idempotency: check `paysquad_transactions` hoặc order status trước khi xử lý

### 6. Logger

VirtualType `PaySquadLogger` → handler ghi vào `/var/log/paysquad_debug.log`. Inject vào tất cả class trong module. Không dùng default Magento logger.

### 7. Admin Config (system.xml)

```
Stores → Configuration → Sales → Payment Methods → PaySquad Group Payment
  ├── Enabled (yes/no)
  ├── Title
  ├── Merchant ID (encrypted)
  ├── API Secret Key (encrypted, backend_model: Magento\Config\Model\Config\Backend\Encrypted)
  ├── Webhook Secret (encrypted)
  └── Mode (Sandbox / Live)
```

### 8. Frontend (My Account)

`GroupPayment.php` ViewModel:
- `getProgressPercent()`: sum(amount từ paysquad_transactions) / order_grand_total × 100
- `getContributors()`: load từ `paysquad_transactions` by order_id
- `getContributionLink()`: từ `sales_order_payment.contribution_link`
- `isOrderPending()`: order status = `pending_payment`

Copy link dùng JS Clipboard API, fallback `document.execCommand('copy')`.

## Files dự kiến

| File | Mục đích |
|---|---|
| `etc/module.xml` | Module declaration + sequence |
| `etc/config.xml` | Payment flags defaults |
| `etc/di.xml` | Facade, CommandPool, Logger, MQ wiring |
| `etc/adminhtml/system.xml` | Admin config fields |
| `etc/communication.xml` + queue XMLs | Message Queue setup |
| `Setup/db_schema.xml` | DB schema |
| `Gateway/Config/Config.php` | API config + auth header builder |
| `Gateway/Http/Client.php` | HTTP client với retry |
| `Gateway/Request/CreatePaySquadBuilder.php` | Create payload builder |
| `Gateway/Request/RefundBuilder.php` | Refund payload builder |
| `Gateway/Response/CreateHandler.php` | Save paySquadId + contributionLink |
| `Model/Webhook/SignatureValidator.php` | HMAC validation |
| `Model/Webhook/Processor.php` | Webhook dispatcher + idempotency |
| `Model/Webhook/Handler/SucceededHandler.php` | Invoice creation flow |
| `Model/Webhook/Handler/FailedHandler.php` | Cancel + restock flow |
| `Model/Webhook/Handler/CancelledHandler.php` | Cancel + restock flow |
| `Model/AmountConverter.php` | Minor units conversion |
| `Model/Transaction/Repository.php` | paysquad_transactions CRUD |
| `Controller/Webhook/Receive.php` | Webhook endpoint |
| `Controller/Payment/Redirect.php` | Post-order redirect |
| `Observer/DataAssignObserver.php` | Payment data assign |
| `ViewModel/Order/GroupPayment.php` | My Account frontend logic |
| `Block/Adminhtml/Order/Contributions.php` | Admin contributions block |
| `view/frontend/web/js/view/payment/method-renderer/paysquad.js` | Checkout KO component |
| `view/frontend/templates/order/group-payment.phtml` | Progress bar + list + copy link |
| `view/adminhtml/templates/order/contributions.phtml` | Admin contributions table |

## Risks + rollback

- Risk: Webhook đến trước order được commit vào DB → idempotency + retry của MQ sẽ xử lý lại sau
  - Rollback: log + return HTTP 200 để tránh PaySquad retry storm; xử lý khi consumer chạy lại
- Risk: `db` MQ adapter mất message nếu server crash trước khi consumer xử lý
  - Rollback: implement reconcile job (nightly GET /api/merchant/paysquad/{id}) — backlog
- Risk: Refund async (202) không có webhook confirm → Admin không biết khi nào xong
  - Rollback: hiển thị rõ "Refund đang được xử lý" trong Credit Memo, không mark là completed ngay
- Risk: Minor units conversion sai với JPY (không có decimal) hoặc currency lạ
  - Rollback: validate currency trong `CreatePaySquadBuilder`, throw exception nếu currency không trong supported list
- Risk: `sales_order_payment` alter có thể conflict với module khác cũng alter bảng này
  - Rollback: dùng `db_schema.xml` declarative — Magento tự handle conflict; test trên staging trước

## Gotchas khi implement

### 1. `is_gateway` bắt buộc trong `config.xml`

`Magento\Payment\Model\Method\Adapter::isAvailable()` kiểm tra `isGateway()`. Nếu thiếu `<is_gateway>1</is_gateway>` trong `config.xml`, method sẽ không xuất hiện trong payment list dù `active=1`.

```xml
<!-- etc/config.xml -->
<is_gateway>1</is_gateway>
```

### 2. `CompositeConfigProvider` phải đăng ký trong `etc/frontend/di.xml`

Không đăng ký trong `etc/di.xml` (global scope). Magento chỉ load `CompositeConfigProvider` trong frontend context.

### 3. `checkout_index_index.xml` phải có `component` trên `billing-step`

```xml
<item name="billing-step" xsi:type="array">
    <item name="component" xsi:type="string">uiComponent</item>
    <item name="children" xsi:type="array">
        ...
    </item>
</item>
```
Thiếu khai báo này khiến jsLayout không merge đúng và renderer không được đăng ký vào `rendererList`.

### 4. `strpos` với prefix `layby` — cẩn thận với `laybyland_`

`strpos('laybyland_paysquad', 'layby')` trả về `0` (truthy). Nếu có observer nào filter payment methods dựa trên `strpos($code, 'layby')`, PaySquad sẽ bị nhận nhầm là layby method. Cần whitelist trong các observer đó.

**Các observer/class cần whitelist trong codebase Laybyland:**
- `Laybyland\Layby\Observer\PaymentCombine\PaymentMethodAvailable` — filter payment methods theo layby/buynow mode
- `Laybyland\Layby\Observer\QuoteSubmitSuccessObserver` — cancel order nếu không có layby_order trong registry
- `Laybyland\BnplExclusion\Observer\PlaceOrder` — block non-layby methods khi cart có BNPL-excluded items

Pattern fix chuẩn:
```php
private const ALWAYS_AVAILABLE_METHODS = ['paysquad'];

public function execute(Observer $observer)
{
    $code = $observer->getEvent()->getMethodInstance()->getCode();
    if (in_array($code, self::ALWAYS_AVAILABLE_METHODS, true)) {
        return; // bypass toàn bộ layby filter logic
    }
    // ... filter logic bình thường
}
```

### 5. Payment method code không được chứa prefix của module khác

**Đặt tên method code là `paysquad`, không phải `laybyland_paysquad`.**

`laybyland_paysquad` chứa `layby` → bị toàn bộ hệ thống Layby nhận nhầm là layby method. Dùng `paysquad` (không prefix) để tránh conflict.

Nếu đã deploy với tên cũ, cần:
1. `sed -i 's/laybyland_paysquad/paysquad/g'` trên tất cả PHP/XML/JS (trừ `db_schema.xml` và ResourceModel `_init()`)
2. `UPDATE core_config_data SET path = REPLACE(path, 'payment/laybyland_paysquad/', 'payment/paysquad/') WHERE path LIKE 'payment/laybyland_paysquad/%'`
3. ResourceModel `_init()` trỏ tới bảng thực tế trong DB (implementation: `secomm_paysquad_transaction`, `secomm_paysquad_webhook_log`)

### 6. JS renderer list — cần file `method-renderer.js` riêng

Layout XML khai báo `component: Vendor_Module/js/view/payment/method-renderer`. File này phải là **renderer list component** (gọi `rendererList.push()`), không phải actual renderer.

```
view/frontend/web/js/view/payment/
├── method-renderer.js          ← renderer list (gọi rendererList.push)
└── method-renderer/
    └── paysquad.js             ← actual renderer (extend default)
```

`method-renderer.js`:
```js
define(['uiComponent', 'Magento_Checkout/js/model/payment/renderer-list'],
function (Component, rendererList) {
    rendererList.push({
        type: 'paysquad',
        component: 'Secomm_PaySquad/js/view/payment/method-renderer/paysquad'
    });
    return Component.extend({});
});
```

**Không dùng `requirejs-config.js` alias** để map `method-renderer` → actual renderer. Alias bypass renderer list → method không hiện trong checkout.

### 7. `Laybyland_Checkout` override `Magento_Checkout/js/view/payment/list`

`app/code/Laybyland/Checkout/view/frontend/requirejs-config.js` map:
```js
"Magento_Checkout/js/view/payment/list": "Laybyland_Checkout/js/view/payment/list"
```

Custom list này chia payment methods thành 2 groups: `layby` và `buyNow` dựa trên `window.checkoutConfig.paymentGroups`. `paysquad` sẽ tự động vào `buyNow` group vì không có prefix `layby`.

`LaybyCheckoutConfigProvider` build `paymentGroups` dựa trên `strpos($code, 'layby')` — `paysquad` không bị ảnh hưởng.

### 8. `payment_action` phải là `authorize`, không phải `authorize_capture`

`authorize_capture` → Magento tự tạo Invoice ngay khi place order → webhook `paysquad.succeeded` đến sau thấy order đã có Invoice → skip tạo Invoice → **contributions không được lưu**.

```xml
<!-- etc/config.xml -->
<payment_action>authorize</payment_action>
```

Flow đúng:
- Place order → `authorize` command → tạo PaySquad session → order `pending_payment`, **không có Invoice**
- Webhook `paysquad.succeeded` → lưu contributions → tạo Invoice → order `processing`

### 9. `SucceededHandler` — idempotency check phải sau khi lưu contributions

Nếu check `$order->hasInvoices()` và return sớm, contributions sẽ không được lưu khi webhook retry. Tách riêng:

```php
$alreadyInvoiced = $order->hasInvoices();

// Luôn lưu contributions
foreach ($contributions as $contribution) {
    $this->saveContribution($order, $contribution);
}

// Chỉ skip tạo Invoice nếu đã có
if (!$alreadyInvoiced) {
    $this->createInvoice($order);
}
```

### 10. `CreateValidator` không nên check `http_status` từ response array

`GatewayCommand` chạy Validator trước Handler. Response array chỉ chứa decoded JSON body từ API — không có `http_status`. Check `http_status` sẽ luôn là `0` → validation fail dù API trả 201.

```php
// ✅ ĐÚNG: chỉ check paySquadId có trong response không
$isValid = ($response['paySquadId'] ?? '') !== '';

// ❌ SAI: http_status không có trong response array
$isValid = ($response['http_status'] ?? 0) === 201;
```

### 11. `CreateHandler` phải set session data trực tiếp

`RedirectAfterPlaceOrder` plugin trên `PaymentInformationManagementInterface` không được gọi khi `AsyncOrder` module active (nó wrap inner call). Set `paysquad_contribution_link` vào session ngay trong `CreateHandler`:

```php
// Gateway/Response/CreateHandler.php
$this->checkoutSession->setData('paysquad_contribution_link', $contributionLink);
```

### 12. `config:set` CLI không encrypt encrypted fields

`php bin/magento config:set payment/paysquad/api_secret_key "value"` lưu plain text dù field có `backend_model="Magento\Config\Model\Config\Backend\Encrypted"`. `encryptor->decrypt(plain_text)` → binary garbage → 401.

**Cách set encrypted config đúng:**
```php
$enc = $om->get(EncryptorInterface::class);
$conn->insert('core_config_data', [
    'scope' => 'default', 'scope_id' => 0,
    'path' => 'payment/paysquad/api_secret_key',
    'value' => $enc->encrypt('actual_value')
]);
```

Hoặc set qua Admin UI (recommended).

**Detect và handle plain text trong `Config.php`:**
```php
public function getApiSecretKey(): string
{
    $value = $this->scopeConfig->getValue(self::XML_PATH_API_SECRET_KEY, ...);
    if ($value === '') return '';
    // Magento encrypted values có prefix "0:N:"
    if (!preg_match('/^\d+:\d+:/', $value)) {
        return $value; // plain text từ CLI
    }
    return $this->encryptor->decrypt($value);
}
```

### 13. `product->getImage()` trả về relative path

`$product->getImage()` trả về `/w/k/filename.jpg`, không phải absolute URL. PaySquad API validate `imageUrl` phải là URL hợp lệ → 400.

```php
// ✅ ĐÚNG
$baseUrl = $this->urlBuilder->getBaseUrl(['_type' => UrlInterface::URL_TYPE_MEDIA]);
$imageUrl = rtrim($baseUrl, '/') . '/catalog/product/' . ltrim($image, '/');

// ❌ SAI
$imageUrl = $product->getImage(); // "/w/k/filename.jpg"
```

### 14. `paymentCompleteWebhook` phải truyền trong Create payload

PaySquad chỉ gửi webhook về URL được chỉ định trong từng request, không phải URL cấu hình global trong dashboard (hoặc cả hai). Phải include trong payload:

```php
$payload = [
    // ...
    'paymentCompleteWebhook' => $this->urlBuilder->getUrl('paysquad/webhook/receive'),
];
```

### 15. SRI hash mismatch sau khi sửa JS

Khi sửa nội dung JS file, phải xóa `pub/static/frontend` và redeploy để Magento tính lại SHA-256 hash. Nếu chỉ flush cache mà không xóa static files, browser sẽ block file vì hash cũ không khớp nội dung mới.

```bash
rm -rf pub/static/frontend pub/static/_cache
php bin/magento setup:static-content:deploy -f
```
