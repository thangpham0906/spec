# Checklist - Kiểm tra trước khi hoàn thành

> Chi tiết rule: `constitution.md` | Chi tiết pattern: đọc reference tương ứng trong `magento-patterns.md`

---

## 1. Code Quality

- [ ] `declare(strict_types=1)` mọi file PHP
- [ ] Return type mọi method
- [ ] Không có `ObjectManager::getInstance()`
- [ ] Không có code chết, comment out, `exit`, `die`, `var_dump`, `print_r`
- [ ] Không có `new` trong business code — dùng factory/DI
- [ ] Logger: `Psr\Log\LoggerInterface`
- [ ] Constructor DI type-hint khớp parent class
- [ ] Exception cụ thể, không dùng `\Exception` chung
- [ ] Core capability đã kiểm tra trước khi tạo custom

## 2. Module Structure

- [ ] `registration.php` + `etc/module.xml` với sequence đúng
- [ ] Không có folder rỗng
- [ ] Namespace: `<Vendor>\<Module>`

## 3. Database

- [ ] `etc/db_schema.xml` (không dùng InstallSchema; **không** đặt trong `Setup/`)
- [ ] `etc/db_schema_whitelist.json`
- [ ] Tên bảng: `<vendor>_<module>_<entity>`
- [ ] Data Patch cho seed data

## 4. API / Service Contract

- [ ] Repository Interface trong `Api/`
- [ ] Data Interface trong `Api/Data/`
- [ ] Preference trong `di.xml`
- [ ] `webapi.xml` nếu expose REST

## 5. Frontend

- [ ] Logic trong ViewModel, không trong template
- [ ] Layout XML khai báo block đúng
- [ ] JS dùng RequireJS, CSS dùng LESS
- [ ] Admin form Save and Continue: xem [references/frontend/admin-save-and-continue.md](./references/frontend/admin-save-and-continue.md)

## 6. Plugin / Observer

- [ ] Plugin có `sortOrder`, tên đúng convention
- [ ] Hạn chế `around` plugin
- [ ] `before` plugin: không `unset()` tham số trong return array
- [ ] Observer chỉ làm 1 việc
- [ ] Observer dùng `strpos()` để filter: kiểm tra method code có bị nhận nhầm không (ví dụ `laybyland_` bắt đầu bằng `layby`)

## 7. Config (`system.xml`)

- [ ] Section riêng `<vendor>_<module>`, không nhét vào section core
- [ ] `resource` ACL hợp lệ
- [ ] `config_path` nếu đổi vị trí UI nhưng giữ key cũ
- [ ] Input type phù hợp nghiệp vụ (không mặc định `text`)
- [ ] `source_model` core ưu tiên trước custom
- [ ] Giá trị mặc định trong `config.xml`
- [ ] `cache:clean config` + `cache:flush` + verify menu path Admin

## 8. Payment Gateway (bổ sung)

- [ ] `<is_gateway>1</is_gateway>` trong `config.xml` — bắt buộc cho `Magento\Payment\Model\Method\Adapter`
- [ ] `CompositeConfigProvider` đăng ký trong `etc/frontend/di.xml`, không phải `etc/di.xml`
- [ ] `Magento\Checkout\Block\Cart\Sidebar` plugin đăng ký trong `etc/frontend/di.xml`
- [ ] `checkout_index_index.xml`: node `billing-step` có `<item name="component" xsi:type="string">uiComponent</item>`
- [ ] Nếu dùng Mageplaza OSC: tạo thêm `onestepcheckout_index_index.xml`

## 9. Testing

- [ ] Task có business logic: unit test viết trước (TDD), đặt tại `Test/Unit/`
- [ ] Cover happy path + ít nhất 1 edge/negative case
- [ ] Mock qua `createMock()`, không dùng ObjectManager trong test
- [ ] `./vendor/bin/phpunit` → all pass
- [ ] `setup:di:compile` sau khi sửa DI
- [ ] Custom carrier: verify checkout method đúng điều kiện
- [ ] Custom thay core: verify so sánh behavior

---

> Khi có dấu hỏi về cách implement: tra `magento-patterns.md` → đọc reference tương ứng.
