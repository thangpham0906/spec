# Checklist - Kiểm tra trước khi hoàn thành

Tham khảo từ: constitution.md, magento-patterns.md

---

## 1. Code Quality

- [ ] Mọi file PHP có `declare(strict_types=1)`
- [ ] Mọi method có return type
- [ ] Không có ObjectManager::getInstance()
- [ ] Không có code chết, code comment out
- [ ] Không có `exit`, `die`, `var_dump`, `print_r`
- [ ] Dependency inject qua constructor, không `new` trực tiếp trong code nghiệp vụ
- [ ] Không có `new` trong `Model/Service/Plugin/Observer/Controller/Resolver/ViewModel`; nếu cần tạo object động thì qua DI factory/wrapper
- [ ] Nếu có third-party object (vd reCAPTCHA validator): đã tạo qua DI factory/wrapper, không `new` ngay trong service
- [ ] Type-hint constructor DI khớp chính xác với parent class/interface đang kế thừa (đúng namespace class)
- [ ] Logger: inject `Psr\Log\LoggerInterface`, **không** dùng `Magento\Psr\Log\LoggerInterface` (tránh lỗi compile DI)
- [ ] Exception cụ thể, không dùng \Exception chung
- [ ] Đã kiểm tra khả năng core/module có sẵn trước khi tạo class/module custom mới

## 2. Cấu trúc Module

- [ ] Có `registration.php`
- [ ] Có `etc/module.xml` với sequence đúng
- [ ] Không có folder rỗng
- [ ] Namespace đúng: `<Vendor>\<Module>`

## 3. Database

- [ ] Dùng `db_schema.xml`, không dùng InstallSchema
- [ ] Có `db_schema_whitelist.json`
- [ ] Tên bảng theo convention: `<vendor>_<module>_<entity>`
- [ ] Data Patch cho dữ liệu mặc định

## 4. API / Service Contract

- [ ] Có Repository Interface trong `Api/`
- [ ] Có Data Interface trong `Api/Data/`
- [ ] Khai báo preference trong `di.xml`
- [ ] Khai báo trong `webapi.xml` (nếu expose API)

## 5. Frontend

- [ ] Logic nằm trong ViewModel, không nằm trong template
- [ ] Layout XML khai báo block đúng
- [ ] JS dùng RequireJS
- [ ] CSS dùng LESS

## 6. Plugin / Observer

- [ ] Plugin có sortOrder
- [ ] Plugin tên đúng convention: `<vendor>_<module>_<mô_tả>`
- [ ] Hạn chế dùng `around` plugin
- [ ] `before` plugin: nếu `return` mảng tham số → **không** `unset()` các biến đó (tránh `Undefined variable` / lỗi GraphQL)
- [ ] Observer chỉ làm 1 việc

## 7. Config

- [ ] Đã ưu tiên dùng config core Magento cho domain tương ứng trước khi tạo config mới
- [ ] Nếu tạo config mới: có lý do rõ ràng vì sao config core không đáp ứng
- [ ] Section trong system.xml: `<vendor>_<module>`
- [ ] **Không gắn group mới trực tiếp vào section core** (`sales`, `catalog`, ...) nếu không thật sự cần; ưu tiên section riêng + tab riêng của vendor
- [ ] Nếu buộc phải đặt field ở section core, phải có verify hiển thị UI ngay sau khi deploy
- [ ] `system.xml` có `resource` hợp lệ và role admin test có quyền tương ứng
- [ ] Nếu đổi cấu trúc section/group nhưng giữ config key cũ, dùng `config_path` để đảm bảo backward compatibility
- [ ] Không lạm dụng `type="text"`: đã chọn input type phù hợp nghiệp vụ (`select`, `multiselect`, `obscure`, ...)
- [ ] Với field dạng option/list: đã ưu tiên `source_model` core trước khi tạo custom source model
- [ ] Với `multiselect`: backend model lưu mảng + code đọc config parse đúng format dữ liệu
- [ ] Có giá trị mặc định trong `config.xml`
- [ ] Đọc config qua Helper/Config model
- [ ] Verify thủ công đường dẫn config trong Admin (ghi rõ menu path)
- [ ] Chạy cache refresh cho config sau khi đổi `system.xml`:
  - `bin/magento cache:clean config`
  - `bin/magento cache:flush`

## 8. Testing (khi cần)

- [ ] Unit test cho logic phức tạp
- [ ] Integration test cho repository
- [ ] Test đặt trong `Test/Unit/` hoặc `Test/Integration/`
- [ ] Nếu có thay đổi constructor DI/module wiring: chạy `bin/magento setup:di:compile`
- [ ] Nếu có custom carrier: verify checkout method xuất hiện/biến mất đúng điều kiện
- [ ] Nếu có tạo custom thay cho core: có ghi chú verify so sánh behavior core vs custom (để chứng minh cần custom)

---

## Liên kết

- Quy tắc chung: xem [constitution.md](./constitution.md)
- Các pattern: xem [magento-patterns.md](./magento-patterns.md)
