# Task Contract Example - REST Endpoint

## 1) Metadata

- Task name: Add REST endpoint `/<route>`
- Priority: `P1`

## 2) Goal

- Tạo/mở rộng REST endpoint để xử lý use case `<use_case>`.
- Đảm bảo auth/ACL, input validation, response format rõ ràng.

## 3) Scope được phép sửa

- `app/code/<Vendor>/<Module>/etc/webapi.xml`
- `app/code/<Vendor>/<Module>/Api/*`
- `app/code/<Vendor>/<Module>/Api/Data/*` (neu can)
- `app/code/<Vendor>/<Module>/Model/*`
- `app/code/<Vendor>/<Module>/etc/di.xml`

## 4) Out of scope

- Không đổi hành vi endpoint hiện có không liên quan.
- Không sửa auth strategy toàn hệ thống.

## 5) Inputs / References bắt buộc

- `.spec/config/constitution.md`
- REST references:
  - `.spec/config/references/network/rest/overview.md`
  - `.spec/config/references/network/rest/tutorials.md`
- Blueprint gần nhất:
  - `.spec/examples/integration/magento-module-notes-webapi-blueprint.md`

## 6) Constraints kỹ thuật

- Service contract rõ input/output.
- Validation đầy đủ cho body/query params.
- Error response nhất quán (4xx/5xx) theo convention.

## 7) Acceptance Criteria

- [ ] Endpoint khai báo đúng trong `webapi.xml`.
- [ ] ACL/auth đúng với yêu cầu (`anonymous|customer|admin|integration`).
- [ ] Input invalid trả về lỗi đúng định dạng.
- [ ] Input valid trả về response đúng schema mong đợi.
- [ ] Compile pass.

## 8) Verify steps bắt buộc

- [ ] `bin/magento setup:di:compile`
- [ ] `bin/magento cache:flush`
- [ ] Gọi API bằng `curl`/Postman:
  - [ ] happy path
  - [ ] invalid input path
  - [ ] permission denied path

## 9) Output contract

1. Files changed
2. Lý do thay đổi
3. API verify commands + kết quả
4. Risk còn lại
