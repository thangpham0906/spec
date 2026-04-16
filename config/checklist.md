# Checklist - Kiểm tra trước khi hoàn thành

> Tham khảo chi tiết: `constitution.md`, `magento-patterns.md`

---

## 1. Code Quality

- [ ] Mọi file PHP có `declare(strict_types=1)`
- [ ] Mọi method có return type
- [ ] Không có ObjectManager::getInstance()
- [ ] Không có code chết, code comment out
- [ ] Không có `exit`, `die`, `var_dump`, `print_r`
- [ ] Dependency inject qua constructor, không `new` trong business code
- [ ] Logger: inject `Psr\Log\LoggerInterface`
- [ ] Type-hint constructor DI khớp với parent class
- [ ] Exception cụ thể, không dùng \Exception chung
- [ ] Đã kiểm tra core capability trước khi tạo custom

## 2. Cấu trúc Module

- [ ] Có `registration.php`
- [ ] Có `etc/module.xml` với sequence đúng
- [ ] Không có folder rỗng
- [ ] Namespace đúng: `<Vendor>\<Module>`

## 3. Database

- [ ] Dùng `db_schema.xml`, không dùng InstallSchema
- [ ] Có `db_schema_whitelist.json`
- [ ] Tên bảng: `<vendor>_<module>_<entity>`
- [ ] Data Patch cho dữ liệu mặc định

## 4. API / Service Contract

- [ ] Có Repository Interface trong `Api/`
- [ ] Có Data Interface trong `Api/Data/`
- [ ] Khai báo preference trong `di.xml`
- [ ] Khai báo trong `webapi.xml` (nếu expose API)

## 5. Frontend

- [ ] Logic nằm trong ViewModel, không trong template
- [ ] Layout XML khai báo block đúng
- [ ] JS dùng RequireJS
- [ ] CSS dùng LESS

## 6. Plugin / Observer

- [ ] Plugin có sortOrder
- [ ] Plugin tên đúng convention
- [ ] Hạn chế dùng `around` plugin
- [ ] `before` plugin: không `unset()` tham số trong return array
- [ ] Observer chỉ làm 1 việc

## 7. Config

- [ ] Ưu tiên dùng config core trước khi tạo mới
- [ ] Section riêng: `<vendor>_<module>`, không nhét vào section core
- [ ] `system.xml` có `resource` hợp lệ
- [ ] Nếu đổi vị trí UI nhưng giữ key cũ: dùng `config_path`
- [ ] Input type phù hợp nghiệp vụ (không mặc định `text`)
- [ ] Ưu tiên `source_model` core
- [ ] Có giá trị mặc định trong `config.xml`
- [ ] Verify UI trong Admin sau khi sửa
- [ ] Chạy `cache:clean config` + `cache:flush`

## 8. Testing

- [ ] Chạy `setup:di:compile` sau khi sửa DI
- [ ] Custom carrier: verify checkout method hiển thị đúng
- [ ] Custom thay core: verify so sánh behavior

---

## Liên kết

- Quy tắc chi tiết: [constitution.md](./constitution.md)
- Patterns: [magento-patterns.md](./magento-patterns.md)
