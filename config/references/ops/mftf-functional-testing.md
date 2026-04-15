# MFTF Functional Testing (Adobe Commerce)

Tài liệu này gom phần Functional Testing Framework (MFTF) theo hướng triển khai thực tế cho team.

Nguồn:

- [Functional Testing Framework - Introduction](https://developer.adobe.com/commerce/testing/functional-testing-framework/)
- [Getting Started](https://developer.adobe.com/commerce/testing/functional-testing-framework/getting-started)
- [Configuration](https://developer.adobe.com/commerce/testing/functional-testing-framework/configuration)
- [Git vs Composer Installation](https://developer.adobe.com/commerce/testing/functional-testing-framework/git-vs-composer-install)
- [How to use in CI/CD](https://developer.adobe.com/commerce/testing/functional-testing-framework/cicd)
- [Debugging Functional Tests](https://developer.adobe.com/commerce/testing/functional-testing-framework/debugging)
- [Interactive Pause](https://developer.adobe.com/commerce/testing/functional-testing-framework/interactive-pause)
- [Extending Functional Tests](https://developer.adobe.com/commerce/testing/functional-testing-framework/extending)
- [Input Testing Data](https://developer.adobe.com/commerce/testing/functional-testing-framework/data)
- [Custom Helpers](https://developer.adobe.com/commerce/testing/functional-testing-framework/custom-helpers)
- [Credentials](https://developer.adobe.com/commerce/testing/functional-testing-framework/credentials)
- [Two-factor authentication](https://developer.adobe.com/commerce/testing/functional-testing-framework/two-factor-authentication)
- [Merge points for testing extensions](https://developer.adobe.com/commerce/testing/functional-testing-framework/merge-points/)
- [Merge Tests](https://developer.adobe.com/commerce/testing/functional-testing-framework/merge-points/merge-tests)
- [Merge Sections](https://developer.adobe.com/commerce/testing/functional-testing-framework/merge-points/merge-sections)
- [Merge Pages](https://developer.adobe.com/commerce/testing/functional-testing-framework/merge-points/merge-pages)
- [Merge Data](https://developer.adobe.com/commerce/testing/functional-testing-framework/merge-points/merge-data)
- [Merge Action Groups](https://developer.adobe.com/commerce/testing/functional-testing-framework/merge-points/merge-action-groups)
- [Extend Tests](https://developer.adobe.com/commerce/testing/functional-testing-framework/merge-points/extend-tests)
- [Extend Data](https://developer.adobe.com/commerce/testing/functional-testing-framework/merge-points/extend-data)
- [Extend Action Groups](https://developer.adobe.com/commerce/testing/functional-testing-framework/merge-points/extend-action-groups)
- [Page structure](https://developer.adobe.com/commerce/testing/functional-testing-framework/page)
- [Section structure](https://developer.adobe.com/commerce/testing/functional-testing-framework/section/)
- [Parameterized selectors](https://developer.adobe.com/commerce/testing/functional-testing-framework/section/parameterized-selectors)
- [Locator functions](https://developer.adobe.com/commerce/testing/functional-testing-framework/section/locator-functions)
- [Metadata](https://developer.adobe.com/commerce/testing/functional-testing-framework/metadata)
- [Merging](https://developer.adobe.com/commerce/testing/functional-testing-framework/merging)
- [Suites](https://developer.adobe.com/commerce/testing/functional-testing-framework/suite)
- [CLI reference](https://developer.adobe.com/commerce/testing/functional-testing-framework/commands/)
- [MFTF CLI commands](https://developer.adobe.com/commerce/testing/functional-testing-framework/commands/mftf)
- [Codecept CLI commands](https://developer.adobe.com/commerce/testing/functional-testing-framework/commands/codeception)
- [Reporting](https://developer.adobe.com/commerce/testing/functional-testing-framework/reporting)

---

## 1) MFTF dùng khi nào?

- Cần kiểm thử end-to-end flow admin/storefront ở góc nhìn người dùng.
- Cần regression suite sau upgrade Magento hoặc sau refactor lớn.
- Cần xác nhận custom module không phá flow chuẩn.

Không thay thế cho unit/integration:

- Unit/Integration bắt bug kỹ thuật sâu hơn, chạy nhanh hơn.
- MFTF bắt lỗi integration theo hành vi thực tế.

---

## 2) Vị trí test materials (rất quan trọng)

MFTF merge dữ liệu theo thứ tự:

1. `app/code/<Vendor>/<Module>/Test/Mftf/`
2. `vendor/<Vendor>/<Module>/Test/Mftf/`
3. `dev/tests/acceptance/tests/functional/<Vendor>/<Module>/`

Kết luận thực dụng:

- Module custom của bạn: ưu tiên đặt test ở `app/code/.../Test/Mftf/`.
- Tránh sửa trực tiếp trong `vendor/`.

---

## 3) Quick start commands

Tại root Magento:

```bash
vendor/bin/mftf --version
vendor/bin/mftf build:project
vendor/bin/mftf generate:tests
vendor/bin/mftf run:test AdminLoginSuccessfulTest --remove
```

Chạy theo scope:

- `vendor/bin/mftf run:test <TestName>`
- `vendor/bin/mftf run:group <GroupName>`
- `vendor/bin/mftf run:manifest <manifest-file>`
- `vendor/bin/mftf run:failed`

---

## 4) CLI strategy: dùng `mftf` hay `codecept`?

Nguyên tắc:

- Ưu tiên `vendor/bin/mftf ...` cho workflow chính (generate/run/failed/group/manifest).
- Chỉ dùng `codecept` trực tiếp khi có nhu cầu debug nâng cao và hiểu rõ rủi ro workflow.

Lệnh `mftf` quan trọng nên nhớ:

- `build:project`
- `doctor`
- `generate:tests` / `generate:suite`
- `run:test` / `run:group` / `run:manifest` / `run:failed`
- `static-checks`
- `codecept:run` (wrapper an toàn hơn gọi codecept trực tiếp)

---

## 5) Cấu hình môi trường tối thiểu (`dev/tests/acceptance/.env`)

Biến bắt buộc:

- `MAGENTO_BASE_URL`
- `MAGENTO_BACKEND_NAME`
- `MAGENTO_ADMIN_USERNAME`
- `MAGENTO_ADMIN_PASSWORD` (qua credentials storage)

Thường dùng thêm:

- `BROWSER`
- `WAIT_TIMEOUT`
- `ENABLE_PAUSE`
- `SELENIUM_*` khi selenium ở host khác

Lưu ý:

- Nếu base URL có subdirectory, cấu hình `MAGENTO_CLI_COMMAND_PATH`.
- Đảm bảo web server cho phép MFTF gọi command endpoint theo hướng dẫn Adobe.

---

## 6) Page + Section structure (xương sống MFTF)

## A. Page

- `Page` định nghĩa URL và danh sách `Section` của trang.
- Tránh hardcode URL/selector trực tiếp trong test.
- Hỗ trợ page explicit và parameterized URL.

## B. Section

- `Section` là nơi định nghĩa element selector tái sử dụng.
- Có thể khai báo `parameterized="true"` để truyền biến vào selector.
- Có thể dùng `locatorFunction` thay selector thường cho case đặc biệt.

Nguyên tắc maintainable:

- Selector tập trung ở `Section`, test chỉ gọi tham chiếu `{{Section.element}}`.
- Mỗi lần UI đổi, sửa một nơi thay vì sửa hàng loạt test.

---

## 7) Metadata (createData/deleteData/updateData/getData)

Metadata mô tả cách MFTF map `Data` entity -> HTTP request (REST hoặc form) cho các action dữ liệu:

- `<createData .../>`
- `<deleteData .../>`
- `<updateData .../>`
- `<getData .../>`

Điểm then chốt:

- `entity type` trong Data phải khớp `operation dataType` trong Metadata.
- Metadata cho phép định nghĩa `auth`, `url`, `method`, `contentType`, request body schema.
- Có thể lấy dữ liệu response qua biến theo `stepKey` (ví dụ `$createStep.id$`).

Dùng khi:

- Cần seed/cleanup data chuẩn thay vì thao tác UI cho mọi precondition.

---

## 8) Credentials và secrets

MFTF hỗ trợ:

- file `.credentials`
- HashiCorp Vault
- AWS Secrets Manager

Precedence:

`.credentials` > Vault > AWS Secrets Manager

Trong test XML:

```xml
<fillField selector=".api-token" userInput="{{_CREDS.vendor/my_data_key}}" />
```

Quy tắc:

- Không commit secrets.
- Tách dữ liệu nhạy cảm khỏi XML test.
- Dùng `_CREDS` thay vì hardcode token/password.

---

## 9) 2FA trong MFTF

Khi bật 2FA admin:

- Cấu hình provider + leeway theo hướng dẫn Adobe.
- Set shared secret cho admin user.
- Lưu secret trong credentials storage tại key:
  - `magento/tfa/OTP_SHARED_SECRET`
- Dùng action `getOTP` trong test để nhập mã OTP runtime.

---

## 10) Reporting + Allure

Artifacts mặc định ở:

- `dev/tests/acceptance/tests/_output/`
- `allure-results/`
- file `failed` (danh sách test fail)

Lệnh thường dùng:

```bash
allure serve dev/tests/acceptance/tests/_output/allure-results/
allure generate dev/tests/acceptance/tests/_output/allure-results --clean -o dev/tests/acceptance/tests/_output/allure-report
allure open dev/tests/acceptance/tests/_output/allure-report
```

Khuyến nghị:

- Luôn đính kèm artifact path khi báo fail.
- Dùng `run:failed` để loop fix nhanh.

---

## 11) Debug test nhanh

## A. Interactive pause

- Bật `ENABLE_PAUSE=true`.
- Thêm `<pause />` tại step cần debug.
- Có thể pause tự động khi test fail.

## B. PhpStorm debug

- Cấu hình Codeception + working dir `dev/tests/acceptance/`.
- Dùng group riêng (ví dụ `testDebug`) để chạy đúng test đang sửa.

---

## 12) Suites (gom test theo điều kiện chạy)

Suites dùng để:

- gom test theo precondition/postcondition dùng chung,
- include/exclude theo test/group/module,
- tối ưu regression run theo chủ đề.

Lưu ý:

- Suite có `<before>` thì phải có `<after>` để hoàn nguyên state.
- Test đã nằm suite custom sẽ không xuất hiện trong default suite.

---

## 13) Tái sử dụng test bằng `extends`

MFTF hỗ trợ extend:

- `<test extends="...">`
- `<actionGroup extends="...">`
- `<entity extends="...">`

Dùng cho:

- thay 1-2 step/action/data thay vì copy nguyên XML.
- giảm duplication trong regression suite.

---

## 14) Merge vs Extend (cực quan trọng cho extension)

MFTF hỗ trợ 2 cách tùy biến:

- **Merge**: chèn/sửa trực tiếp vào object hiện có (test/action group/data/page/section).
- **Extend**: tạo object mới dựa trên object gốc, không làm thay đổi object gốc.

Chọn nhanh:

- Dùng **merge** khi muốn mọi test dùng object gốc đều nhận thay đổi mới.
- Dùng **extend** khi muốn tạo biến thể riêng cho extension, tránh ảnh hưởng test hiện có.

## Các merge points chính

- Merge Action Groups
- Merge Data
- Merge Pages
- Merge Sections
- Merge Tests

## Khi nào ưu tiên extends

- Bạn cần test extension-specific flow nhưng vẫn giữ nguyên behavior core test.
- Bạn muốn giảm rủi ro regression do thay đổi object dùng chung.
- Bạn cần tạo “phiên bản mở rộng” có annotation/title/assert riêng.

Quy tắc an toàn:

- Nếu chưa chắc phạm vi ảnh hưởng, **ưu tiên extends trước merge**.
- Chỉ merge khi đã xác nhận mục tiêu là thay đổi behavior dùng chung.

---

## 15) Data trong MFTF

Nguồn dữ liệu thường dùng:

- Entity data (`{{Entity.key}}`)
- Env (`{{_ENV.MAGENTO_ADMIN_USERNAME}}`)
- Credentials (`{{_CREDS.vendor/key}}`)
- Persisted step data (`$stepKey.field$`)
- Grabbed runtime value (`$grabStepKey`)

Khuyến nghị:

- `stepKey` nên mô tả rõ và unique để tránh đụng tham chiếu.
- Tách Data entity ra file riêng để tái sử dụng.

---

## 16) Parameterized selector + locatorFunction (mẹo tránh selector cứng)

- Dùng `parameterized="true"` cho selector cần input động.
- Tham chiếu theo dạng:
  - `{{Section.element(param1, param2)}}`
- Dùng `locatorFunction` cho các case selector khó diễn tả bằng CSS/XPath tĩnh.

Quy tắc:

- Không hardcode selector phức tạp trực tiếp trong test.
- Ưu tiên đưa logic locate vào section element để test readable hơn.

---

## 17) Custom Helper (chỉ khi bất khả kháng)

Adobe khuyến nghị:

- Chỉ viết custom helper khi built-in actions không đáp ứng được.
- Đặt tại `<Module>/Test/Mftf/Helper`.
- Giữ helper nhỏ, rõ input/output, tránh nhét logic nghiệp vụ dài.

---

## 18) CI/CD flow gợi ý

Luồng chuẩn:

1. Chuẩn bị Magento instance + config baseline.
2. `mftf generate:tests` (single hoặc `--config parallel`).
3. Chạy `run:manifest` theo node/group.
4. Thu thập artifacts `_output`.
5. Chạy `run:failed` (nếu cần).
6. Generate Allure report.

---

## 19) Checklist “Done” cho task có MFTF

- [ ] Có test cho happy path chính.
- [ ] Có ít nhất 1 assertion cho kết quả business quan trọng.
- [ ] Không hardcode credentials nhạy cảm trong XML.
- [ ] Nếu test trùng nhiều phần, đã cân nhắc `extends` trước khi merge.
- [ ] Đã xác nhận rõ thay đổi này nên là merge (ảnh hưởng global) hay extend (biến thể riêng).
- [ ] Có command chạy lại nhanh (`run:test` hoặc group/manifest).
- [ ] Báo cáo kèm artifact path khi fail.

---

## 20) Prompt mẫu giao AI viết/chỉnh MFTF

```text
Đọc `.spec/config/constitution.md`, `.spec/config/references/ops/testing-guide.md`
và `.spec/config/references/ops/mftf-functional-testing.md`.
Tạo/chỉnh MFTF test cho flow <flow_name> trong module <Vendor_Module>.
Ưu tiên tái sử dụng action group/data entity, tránh copy XML dư thừa.
Quyết định rõ dùng merge hay extends và nêu lý do (phạm vi ảnh hưởng).
Không hardcode secrets; dùng `_CREDS` khi cần.
Báo cáo:
1) files changed
2) command đã chạy
3) kết quả
4) artifact/debug note nếu fail
```

---

## Liên kết nội bộ

- Testing tổng quan: [testing-guide.md](./testing-guide.md)
- Unit + Static: [unit-testing.md](./unit-testing.md)
