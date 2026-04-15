# Task Contract Example - GraphQL Field Extension

## 1) Metadata

- Task name: Add GraphQL field `<field_name>` to `<type>`
- Priority: `P1`

## 2) Goal

- Mở rộng GraphQL schema để trả về `<field_name>` cho `<type>`.
- Đảm bảo resolver xử lý đúng store/context và không làm chậm query hiện có.

## 3) Scope được phép sửa

- `app/code/<Vendor>/<Module>/etc/schema.graphqls`
- `app/code/<Vendor>/<Module>/Model/Resolver/*`
- `app/code/<Vendor>/<Module>/etc/di.xml`
- `app/code/<Vendor>/<Module>/etc/module.xml` (neu can)

## 4) Out of scope

- Không sửa schema module khác nếu không cần thiết.
- Không đổi response field khác ngoài mục tiêu.

## 5) Inputs / References bắt buộc

- `.spec/config/constitution.md`
- Blueprint gần nhất:
  - `.spec/examples/integration/magento-module-address-extension-graphql-blueprint.md`
- Reference GraphQL:
  - `.spec/config/references/network/graphql/schema-products.md` (hoặc file schema liên quan)

## 6) Constraints kỹ thuật

- Resolver phải throw exception theo kiểu Magento GraphQL phù hợp.
- Không query DB thừa (ưu tiên service/repository đã có).
- Bảo toàn backward compatibility schema.

## 7) Acceptance Criteria

- [ ] Schema validate, field mới xuất hiện đúng type.
- [ ] Query mẫu trả về field mới đúng giá trị.
- [ ] Không ảnh hưởng query hiện có.
- [ ] Compile pass.

## 8) Verify steps bắt buộc

- [ ] `bin/magento setup:di:compile`
- [ ] `bin/magento cache:flush`
- [ ] Chạy query GraphQL mẫu (ghi rõ query + expected response).

## 9) Output contract

1. Files changed
2. Lý do thay đổi
3. Query verify + kết quả
4. Risk còn lại
