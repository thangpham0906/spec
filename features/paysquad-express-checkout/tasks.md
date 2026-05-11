# Tasks - paysquad-express-checkout

> **Code thực tế:** `app/code/Secomm/PaySquad/` (payment code `paysquad`, config `payment/paysquad/*`). Nút minicart trên storefront: thêm theme `Laybyland/layup/Mageplaza_QuickCart` (`web/js/view/minicart.js`, `web/template/minicart/content.html`) + `Plugin/Block/Cart/Sidebar` inject `isPaySquadExpressEnabled`.

> Không lặp business context. Mỗi task: scope + AC + unit test + verify.
> TDD flow bắt buộc với task có business logic: **viết test trước → implement → test pass → code review**.
> Nếu test fail hoặc review còn Critical/High issue: **DỪNG, không được báo task hoàn thành**.

---

## Task 1 - Admin config + Gateway Config method [enabler]

- Depends-on: `—`
- Context: Thêm config `express_checkout_enabled` (default=0) vào `config.xml` và field UI vào `system.xml`. Thêm method `isExpressCheckoutEnabled()` vào `Gateway/Config/Config.php`.
- Scope:
  - `app/code/Secomm/PaySquad/etc/config.xml`
  - `app/code/Secomm/PaySquad/etc/adminhtml/system.xml`
  - `app/code/Secomm/PaySquad/Gateway/Config/Config.php`
- Acceptance criteria:
  - `config.xml` có `<express_checkout_enabled>0</express_checkout_enabled>` trong group `paysquad`
  - `system.xml` có field `express_checkout_enabled` type=select, source=Yesno, sortOrder=70
  - `Config::isExpressCheckoutEnabled()` đọc `payment/paysquad/express_checkout_enabled` trả về bool
- Verify:
  - `bin/magento cache:clean config`
  - Admin → Stores → Config → Sales → Payment Methods → PaySquad → thấy field "Express Checkout Button"

---

## Task 2 - AJAX endpoint set session flag [feature]

- Depends-on: `Task 1`
- Context: POST `/paysquad/express/set` validate form key + cart không trống, set `paysquad_express=1` trong `Magento\Checkout\Model\Session`, trả JSON. Route `paysquad` đã có trong `etc/frontend/routes.xml`.
- Scope:
  - `app/code/Secomm/PaySquad/Controller/Express/Set.php`
- Acceptance criteria:
  - Chỉ chấp nhận POST (implement `HttpPostActionInterface`)
  - Validate form key — Magento tự xử lý qua `CsrfAwareActionInterface` hoặc form key validator
  - Cart trống → return `{"success": false, "message": "Cart is empty"}`
  - Cart có item → set `checkoutSession->setData('paysquad_express', 1)` → return `{"success": true}`
  - Không set flag nếu `Config::isActive() === false`
- Unit test (`Test/Unit/Controller/Express/SetTest.php`):
  - Happy path: cart có item, PaySquad active → flag set, return success
  - Negative: cart trống → flag không set, return error
  - Negative: PaySquad disabled → flag không set, return error
- Reference: `config/references/core/routing-controllers.md`
- Verify sau implement:
  - `./vendor/bin/phpunit app/code/Secomm/PaySquad/Test/Unit/Controller/Express/SetTest.php` → all pass
  - POST `/paysquad/express/set` với cart có item + `form_key` → JSON success (controller hiện bypass CSRF strict; không kỳ vọng 403 mặc định)
- Code review: `config/checklist.md`

---

## Task 3 - Plugin filter payment methods [feature]

- Depends-on: `Task 2`
- Context: `afterGetActiveQuoteMethods` plugin trên `Magento\Payment\Model\MethodList`. Khi `paysquad_express=1` trong session, filter result chỉ giữ method có code `paysquad`. OSC gọi `PaymentMethodManagementInterface::getList()` → bên trong gọi `MethodList::getActiveQuoteMethods()` → plugin intercept được.
- Scope:
  - `app/code/Secomm/PaySquad/Plugin/ExpressPaymentFilter.php`
  - `app/code/Secomm/PaySquad/etc/di.xml` (thêm plugin)
- Acceptance criteria:
  - Flag active + PaySquad trong list → chỉ trả về `[paysquad]`
  - Flag active + PaySquad không trong list (disabled) → trả về `[]`
  - Flag không active → trả về `$result` nguyên vẹn
- Unit test (`Test/Unit/Plugin/ExpressPaymentFilterTest.php`):
  - Happy path: flag=1, list có 3 methods → chỉ còn paysquad
  - Edge: flag=1, PaySquad không trong list → empty array
  - Negative: flag=0 → list không bị filter
- Reference: `config/references/core/plugins.md`
- Verify sau implement:
  - `./vendor/bin/phpunit app/code/Secomm/PaySquad/Test/Unit/Plugin/ExpressPaymentFilterTest.php` → all pass
  - Set flag manually trong session → vào OSC → chỉ thấy PaySquad
- Code review: `config/checklist.md`

---

## Task 4 - ConfigProvider expose isExpressCheckout [feature]

- Depends-on: `Task 2`
- Context: Mở rộng `Model/Ui/ConfigProvider.php` inject thêm `Magento\Checkout\Model\Session` để expose `isExpressCheckout` flag cho JS. JS component đọc flag này để auto-select PaySquad.
- Scope:
  - `app/code/Secomm/PaySquad/Model/Ui/ConfigProvider.php`
- Acceptance criteria:
  - `getConfig()` trả thêm `'isExpressCheckout' => (bool) $checkoutSession->getData('paysquad_express')`
  - Khi flag=0: `isExpressCheckout = false`
  - Khi flag=1: `isExpressCheckout = true`
- Unit test (`Test/Unit/Model/Ui/ConfigProviderTest.php`):
  - Flag=1 → `isExpressCheckout: true` trong config
  - Flag=0 → `isExpressCheckout: false` trong config
- Verify sau implement:
  - `./vendor/bin/phpunit app/code/Secomm/PaySquad/Test/Unit/Model/Ui/ConfigProviderTest.php` → all pass
  - Set flag → vào OSC → `window.checkoutConfig.payment.paysquad.isExpressCheckout === true`
- Code review: `config/checklist.md`

---

## Task 5 - Clear flag observers [feature]

- Depends-on: `Task 2`
- Context: 2 observers clear `paysquad_express` flag: (1) sau order success, (2) khi navigate ra khỏi OSC. Observer navigate away check `$request->getModuleName() !== 'onestepcheckout'` và `!$request->isAjax()`.
- Scope:
  - `app/code/Secomm/PaySquad/Observer/ClearExpressFlagOnSuccess.php`
  - `app/code/Secomm/PaySquad/Observer/ClearExpressFlagOnNavigate.php`
  - `app/code/Secomm/PaySquad/etc/events.xml`
- Acceptance criteria:
  - `ClearExpressFlagOnSuccess`: event `checkout_onepage_controller_success_action` → `checkoutSession->unsetData('paysquad_express')`
  - `ClearExpressFlagOnNavigate`: event `controller_action_predispatch` → check module != `onestepcheckout` AND !isAjax → unset flag
  - AJAX requests từ OSC không trigger clear
  - Requests đến `onestepcheckout/*` không trigger clear
- Unit test (`Test/Unit/Observer/ClearExpressFlagOnNavigateTest.php`):
  - Module=homepage, không ajax → flag bị clear
  - Module=onestepcheckout → flag không bị clear
  - Module=homepage, isAjax=true → flag không bị clear
- Reference: `config/references/core/events-observers.md`
- Verify sau implement:
  - `./vendor/bin/phpunit app/code/Secomm/PaySquad/Test/Unit/Observer/` → all pass
  - Set flag → navigate về homepage → flag bị clear (check session)
  - Set flag → place order → flag bị clear
- Code review: `config/checklist.md`

---

## Task 6 - Minicart button block + observer + template [enabler]

- Depends-on: `Task 1`, `Task 4`
- Context: Block `Block/Minicart/Button.php` implement `ShortcutInterface`. Observer `AddExpressShortcut` inject block vào minicart qua event `shortcut_buttons_container`. Template có nút với AJAX call + redirect. JS dùng RequireJS.
- Scope:
  - `app/code/Secomm/PaySquad/Block/Minicart/Button.php`
  - `app/code/Secomm/PaySquad/Observer/AddExpressShortcut.php`
  - `app/code/Secomm/PaySquad/view/frontend/templates/minicart/button.phtml`
  - `app/code/Secomm/PaySquad/etc/events.xml` (thêm observer)
- Acceptance criteria:
  - `Button::isActive()` = `Config::isActive() && Config::isExpressCheckoutEnabled()`
  - `Button::_toHtml()` trả `''` khi `!isActive()`
  - `Button::getAlias()` trả alias string
  - Template: nút "Buy Now with PaySquad", `data-express-set-url`, `data-checkout-url`
  - JS: AJAX POST đến set URL → nếu success redirect đến checkout URL → nếu fail hiển thị lỗi
  - Nút ẩn khi cart trống (KO binding `visible: itemsCount() > 0`)
- Verify:
  - `bin/magento setup:di:compile && bin/magento setup:static-content:deploy -f`
  - Minicart có item + PaySquad+Express enabled → thấy nút "Buy Now with PaySquad"
  - Minicart trống → nút ẩn
  - PaySquad disabled → nút ẩn

---

## Task 7 - JS auto-select PaySquad khi isExpressCheckout=true [enabler]

- Depends-on: `Task 4`, `Task 6`
- Context: Cập nhật `view/frontend/web/js/view/payment/method-renderer/paysquad.js` để đọc `isExpressCheckout` từ `window.checkoutConfig` và gọi `selectPaymentMethod()` trong `initialize()`.
- Scope:
  - `app/code/Secomm/PaySquad/view/frontend/web/js/view/payment/method-renderer/paysquad.js`
- Acceptance criteria:
  - Khi `isExpressCheckout = true`: gọi `this.selectPaymentMethod()` trong `initialize()`
  - Khi `isExpressCheckout = false`: không auto-select (behavior bình thường)
- Verify:
  - `bin/magento setup:static-content:deploy -f`
  - Set flag → vào OSC → PaySquad tự động được select, không cần click
  - Không set flag → vào OSC → PaySquad không tự select

---

> Rules bổ sung:
> - Inject `Secomm\PaySquad\Logger\Logger` (không phải PSR interface) cho các class cần log
> - Không tạo Factory thủ công — Magento auto-generate
> - `system.xml` → `cache:clean config` sau khi sửa
> - `controller_action_predispatch` observer: check `isAjax()` trước khi clear flag

## Completion report
1. Files changed
2. Lý do thay đổi
3. Unit test results (pass/fail)
4. Code review results (Critical/High issues nếu có)
5. Verify steps + kết quả testcase
6. Xác nhận DoD
