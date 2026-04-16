# Constitution - Quy tắc phát triển chung

> version: 1.1.4 | last_updated: 2026-04-16

## Thông tin project

- Magento: 2.4.8-p4
- PHP: 8.3
- Vendor: Theo module thực tế (ví dụ: `Secomm`, `NullTraceX`)
- Namespace gốc: `<Vendor>\<ModuleName>`

---

## 1. Coding Standard

### PHP

- PHP 8.3, sử dụng typed properties, union types, named arguments khi cần
- Tuân theo PSR-1, PSR-4, PSR-12
- Strict types bắt buộc: `declare(strict_types=1);` ở mọi file PHP
- Không dùng `@suppress`, không dùng `@codingStandardsIgnore`
- Return type bắt buộc cho mọi method (kể cả `: void`)
- Docblock chỉ khi cần giải thích logic phức tạp, không lặp lại type hint

### Nguyên tắc thiết kế (Technical Guidelines)

- **SOLID bắt buộc:**
  - S: Mỗi class chỉ làm 1 việc.
  - O: Mở rộng bằng plugin/event, không sửa core.
  - L: Subclass phải thay thế được parent.
  - I: Interface nhỏ, cụ thể, không ép implement method thừa.
  - D: Inject dependency qua constructor, không `new` trực tiếp trong code nghiệp vụ.
- **Object Readiness:** Object phải sẵn sàng sử dụng ngay sau khi `__construct`. Không dùng hàm `init()` hoặc `load()` công khai.
- **Constructor Logic:** Chỉ chứa phép gán (assignment) và kiểm soát tham số. Không thực hiện logic nghiệp vụ hay dispatch Event trong constructor.
- **Exceptions (Ngoại lệ):**
  - Không ném generic `\Exception`. Luôn dùng loại cụ thể (vd: `LocalizedException`).
  - Không được `catch` ngoại lệ ngay trong chính hàm vừa ném nó ra.
  - Không dùng Exception để điều khiển luồng logic nghiệp vụ.
  - Khi dùng tài nguyên (file, stream), bắt buộc có `finally` để giải phóng.
- **DI (Dependency Injection):** Qua `di.xml`, KHÔNG dùng ObjectManager trực tiếp.
- **No direct instantiation policy (bắt buộc):**
  - Cấm `new` trong `Model/`, `Service/`, `Plugin/`, `Observer/`, `Controller/`, `Resolver/`, `ViewModel/`.
  - Nếu cần tạo object theo runtime data (vd secret key/token/input động), phải tạo qua factory/wrapper được inject bằng DI.
  - Ngoại lệ duy nhất: class factory/wrapper chuyên dụng của chính module để tạo object bên thứ ba; không đặt logic nghiệp vụ trong factory đó.
- **Composition hơn inheritance.**
- **::class Keyword:** Luôn dùng `ClassName::class` thay vì dùng chuỗi string cho tên class.

---

## 2. Những điều KHÔNG được làm

- KHÔNG sửa file core Magento
- KHÔNG dùng ObjectManager::getInstance() trong code (chỉ cho trong factory tạo bởi Magento)
- KHÔNG hardcode store ID, website ID, customer group ID
- KHÔNG viết logic trong template (.phtml)
- KHÔNG dùng `<preference>` khi có thể dùng plugin
- KHÔNG để code chết, code comment out
- KHÔNG dùng `exit`, `die`, `var_dump`, `print_r` trong code chính thức
- KHÔNG tạo bảng DB mà không có `db_schema.xml` + `db_schema_whitelist.json`

---

## 3. Cấu trúc Module bắt buộc

Mỗi module  phải có tối thiểu:

```
app/code/<Vendor>/<TênModule>/
├── registration.php
├── etc/
│   └── module.xml
├── composer.json           (không bắt buộc nhưng nên có)
```

Chỉ tính năng nào cần, mới thêm folder tương ứng. Không tạo folder rỗng.

---

## 4. Ngôn ngữ và Format

- Tên class, method, variable: tiếng Anh
- Comment giải thích logic: tiếng Anh
- Commit message: tiếng Anh
- Spec, plan, task: tiếng Việt có dấu
- **Document Tags (PHPDoc):**
  - `@api`: Bắt buộc cho mọi Public Interface (`Api/`).
  - `@since`: Bắt buộc cho Interface và Method trong Api folder (ví dụ: `@since 1.0.0`).
  - `@inheritdoc`: Dùng trong class thực thi (Implementation) để kế thừa docblock từ interface.
  - **Lưu ý đặc biệt cho Web API:** Mọi `@param` và `@return` trong Public Interface phải sử dụng **Fully Qualified Class Name** (ví dụ: `\Vendor\Module\Api\Data\MyInterface` thay vì `Data\MyInterface`) để hệ thống REST/SOAP tự động nhận diện đúng kiểu dữ liệu.

---

## 5. Xử lý lỗi

- Dùng exception cụ thể (LocalizedException, NoSuchEntityException...), không dùng \Exception chung
- Log lỗi bằng Psr\Log\LoggerInterface, không dùng echo hay file_put_contents
- Validate input đầu vào, không tin tưởng dữ liệu từ frontend

---

## 6. Database

- Dùng Declarative Schema (`db_schema.xml`), KHÔNG dùng InstallSchema/UpgradeSchema
- Đặt tên bảng: `<vendor>_<module>_<entity>` (ví dụ: `<vendor>_employee_record`)
- Tạo Data Patch cho dữ liệu mặc định, KHÔNG dùng InstallData/UpgradeData
- Index cho các column thường query

---

## 7. Frontend

- Dùng UI Component / KnockoutJS theo chuẩn Magento
- Layout XML để khai báo block, không hardcode trong template
- CSS dùng LESS, đặt trong `view/frontend/web/css/source/`
- JS dùng RequireJS, khai báo trong `requirejs-config.js`

---

## 8. API và Service Contract

- Cung cấp API interface trong `Api/` cho mọi entity chính
- Repository pattern: `Api/<Entity>RepositoryInterface.php`
- Data interface: `Api/Data/<Entity>Interface.php`
- Search Results: `Api/Data/<Entity>SearchResultsInterface.php`
- Khai báo trong `etc/di.xml` và `etc/webapi.xml` (nếu cần expose REST/GraphQL)

---

## 9. Scope Governance (bắt buộc)

- Trước khi code, phải chốt rõ phạm vi file/module được phép sửa.
- KHÔNG sửa ngoài scope đã chốt nếu chưa được xác nhận lại.
- Nếu phát sinh sửa ngoài scope (do dependency/bug liên quan), phải báo và xin xác nhận trước khi sửa.
- Core-first rule: trước khi tạo module mới / plugin mới / API mới, bắt buộc kiểm tra Magento core hoặc module đã cài có hỗ trợ sẵn hay không.
- Nếu core đã hỗ trợ đủ yêu cầu, ưu tiên cấu hình/tích hợp lại thay vì build mới.
- Nếu vẫn cần custom mới, phải ghi rõ trong spec/plan: `vì sao core không đáp ứng` + phạm vi phần cần custom thêm.

---

## 10. Testing Policy

### Unit Test (TDD — bắt buộc cho task có business logic)

- Áp dụng TDD: **viết test trước (sẽ fail) → implement → chạy lại test (phải pass) → code review**.
- Sau khi test pass: bắt buộc review bằng skill `magento-code-reviewer` + đối chiếu `config/checklist.md` trước khi báo task hoàn thành.

### Blocking rules (không được bỏ qua)

- Test fail → **DỪNG**. Phân tích nguyên nhân fail: nếu logic sai thì sửa implementation, nếu test sai thì sửa test (ghi rõ lý do). Chạy lại đến khi pass. Không được sửa test chỉ để cho pass mà không fix root cause.
- Review còn Critical/High issue → **DỪNG**. Fix implementation theo từng issue, chạy lại test sau khi sửa, review lại cho đến khi sạch Critical/High.
- Chỉ được báo task hoàn thành khi: test pass **VÀ** không còn Critical/High issue từ review.
- Framework: Magento's `\PHPUnit\Framework\TestCase` (extend từ `\Magento\TestFramework\TestCase\AbstractController` hoặc `\PHPUnit\Framework\TestCase` tùy loại).
- Đặt test tại: `Test/Unit/<mirror-path-của-class>.php`.
- **Bắt buộc unit test** cho các class có business logic:
  - `Service/`, `Model/` (logic), `Validator/`, `Plugin/`, `Observer/`, `Carrier/collectRates`
  - Config reader có transform/normalize logic
- **Không bắt buộc unit test** cho:
  - Scaffold (registration, module.xml), Controller routing, Layout/Template, i18n, Data Patch đơn giản
- Mỗi unit test phải cover tối thiểu: happy path + 1 edge/negative case.
- Mock dependency qua `createMock()` / `getMockBuilder()`; không dùng ObjectManager trong test.

### Verify steps (mọi task)

- Mỗi task phải có verify steps rõ ràng (command + manual steps nếu có).
- Với feature thay đổi behavior nghiệp vụ, bắt buộc có testcase tối thiểu theo nhóm:
  - Happy path
  - Edge case
  - Negative case
- Kết quả verify phải được báo theo testcase: Pass/Fail + ghi chú ngắn.

---

## 11. Definition of Done (DoD)

Task chỉ được xem là hoàn thành khi đạt đủ:

1. Đúng acceptance criteria trong spec/task contract.
2. Không vi phạm rule trong constitution + magento-patterns.
3. Unit test đã viết (nếu task thuộc nhóm bắt buộc) và pass.
4. Code review bằng skill `magento-code-reviewer` + checklist.md đã chạy — không còn issue Critical/High.
5. Đã chạy verify steps và báo kết quả rõ ràng.
6. Báo cáo cuối có đủ:
  - Files changed
  - Lý do thay đổi
  - Verify steps đã chạy
  - Kết quả testcase

---

## 12. Git/PR Convention

- Branch name (khuyến nghị): `<type>/<feature-kebab-case>` (ví dụ: `feat/product-shipping-method`).
- Commit message: tiếng Anh, ngắn gọn, nêu rõ ý nghĩa thay đổi.
- PR description tối thiểu:
  - Mục tiêu business
  - Scope thay đổi
  - Test/verify đã chạy
  - Rủi ro còn lại (nếu có)

---

## 13. Security & Performance Baseline

- Không log dữ liệu nhạy cảm (token, password, secret key, PII không cần thiết).
- Luôn validate input nhận từ frontend/API trước khi xử lý.
- Tránh N+1 query và query không có index ở các luồng đọc lớn.
- Chỉ thêm cache khi có lợi ích rõ ràng và có invalidation strategy phù hợp.
- Với external integration/API call: có timeout hợp lý và xử lý lỗi rõ ràng.

---

## 14. Versioning & Backward Compatibility

- Tránh breaking changes trên interface/public API nếu không có kế hoạch migration.
- Nếu buộc phải thay đổi behavior public:
  - Ghi rõ impact trong spec/task
  - Nêu phương án tương thích hoặc migration
  - Cập nhật tài liệu liên quan

---

## 15. Proactive Analysis & Advisory (bắt buộc)

> Quy tắc đầy đủ: xem `AGENTS.md` — mục "Hành vi chủ động".

Tóm tắt: phân tích độc lập, nêu rõ requirement understanding + risks + options trước khi code, cảnh báo rủi ro, hỏi lại khi thiếu thông tin, đề xuất next best action sau mỗi bước lớn.

## Liên kết

- Các pattern chi tiết Magento: xem [magento-patterns.md](./magento-patterns.md)
- Checklist trước khi hoàn thành: xem [checklist.md](./checklist.md)

