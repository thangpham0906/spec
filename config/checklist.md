# Checklist - Kiểm tra trước khi hoàn thành

Tham khảo từ: constitution.md, magento-patterns.md

---

## 1. Code Quality

- [ ] Mọi file PHP có `declare(strict_types=1)`
- [ ] Mọi method có return type
- [ ] Không có ObjectManager::getInstance()
- [ ] Không có code chết, code comment out
- [ ] Không có `exit`, `die`, `var_dump`, `print_r`
- [ ] Dependency inject qua constructor, không `new` trực tiếp
- [ ] Type-hint constructor DI khớp chính xác với parent class/interface đang kế thừa (đúng namespace class)
- [ ] Exception cụ thể, không dùng \Exception chung

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
- [ ] Observer chỉ làm 1 việc

## 7. Config

- [ ] Section trong system.xml: `<vendor>_<module>`
- [ ] Có giá trị mặc định trong `config.xml`
- [ ] Đọc config qua Helper/Config model

## 8. Testing (khi cần)

- [ ] Unit test cho logic phức tạp
- [ ] Integration test cho repository
- [ ] Test đặt trong `Test/Unit/` hoặc `Test/Integration/`
- [ ] Nếu có thay đổi constructor DI/module wiring: chạy `bin/magento setup:di:compile`
- [ ] Nếu có custom carrier: verify checkout method xuất hiện/biến mất đúng điều kiện

---

## Liên kết

- Quy tắc chung: xem [constitution.md](./constitution.md)
- Các pattern: xem [magento-patterns.md](./magento-patterns.md)
