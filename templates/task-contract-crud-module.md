# Task Contract Example - Admin CRUD Module

## 1) Metadata

- Task name: Add admin CRUD for `<Entity>`
- Priority: `P1`

## 2) Goal

- Tạo/chỉnh sửa module để quản lý `<Entity>` trên Admin Grid + Form.
- Không phá vỡ các route/ACL hiện có.

## 3) Scope được phép sửa

- `app/code/<Vendor>/<Module>/etc/*`
- `app/code/<Vendor>/<Module>/Controller/Adminhtml/*`
- `app/code/<Vendor>/<Module>/Model/*`
- `app/code/<Vendor>/<Module>/Model/ResourceModel/*`
- `app/code/<Vendor>/<Module>/view/adminhtml/*`

## 4) Out of scope

- Không sửa module khác ngoài `<Vendor>/<Module>`.
- Không đổi conventions trong `.spec/config/constitution.md`.

## 5) Inputs / References bắt buộc

- `.spec/config/constitution.md`
- `.spec/examples/integration/module-skeleton/README.md`
- `.spec/examples/integration/module-skeleton/file-map.md`
- `.spec/examples/integration/module-skeleton/templates.md`
- Blueprint gần nhất (nếu có):
  - `.spec/examples/integration/magento-module-admin-grid-employee-blueprint.md`

## 6) Constraints kỹ thuật

- Follow service contract/repository pattern.
- ACL/menu/route phải map đúng.
- UI component XML naming theo convention module.

## 7) Acceptance Criteria

- [ ] Có admin menu vào trang grid.
- [ ] Grid render dữ liệu đúng, có action edit.
- [ ] Form save thành công cho create/update.
- [ ] ACL đúng, user không quyền không truy cập được.
- [ ] Không lỗi compile.

## 8) Verify steps bắt buộc

- [ ] `bin/magento setup:upgrade`
- [ ] `bin/magento setup:di:compile`
- [ ] `bin/magento cache:flush`
- [ ] Vào Admin, mở URL grid và test create/update.

## 9) Output contract

1. Files changed
2. Lý do thay đổi
3. Verify steps + kết quả
4. Risk còn lại
