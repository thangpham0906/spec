# Feature Spec - magento-recaptcha-v3-backend

## 1) Feature input
- Feature: `Tích hợp reCAPTCHA v3 cho backend Magento`
- Bối cảnh/vấn đề: `Flow form/API hiện tại chưa có lớp chống bot/spam ở backend nên dễ bị abuse request.`
- Kết quả mong muốn: `Magento nhận recaptcha_token từ FE, verify với Google bằng secret key cấu hình, chỉ xử lý business logic khi token hợp lệ.`
- Phạm vi biết chắc: `Chỉ làm phía Magento 2 (BE) cho kiến trúc headless. FE Next.js chỉ là nguồn gửi token, không triển khai FE trong feature này.`

## 2) Business + scope
- Business problem: `Request spam/bot đi vào endpoint nghiệp vụ làm tăng rủi ro bảo mật, tốn tài nguyên và gây nhiễu dữ liệu.`
- Business goal: `Thêm lớp verify reCAPTCHA v3 ở Magento theo cách tách service, cấu hình linh hoạt và không phá flow hiện tại.`
- In-scope:
  - `Headless-first: verify ở layer API (GraphQL/REST/custom API) cho flow customer login/register, không phụ thuộc form controller truyền thống.`
  - `Nhận `recaptcha_token` từ request payload/header cho 2 flow phase 1: customer login và customer register (không áp dụng cho admin login).`
  - `Dùng config core Magento reCAPTCHA cho secret/threshold và các tham số verify liên quan; module chỉ đọc/adapter, không tạo system config mới nếu không cần.`
  - `Service riêng gọi Google siteverify API, parse response, trả kết quả chuẩn hoá cho layer nghiệp vụ.`
  - `Validation gate trước business logic: token invalid/fail -> từ chối request với message/error code rõ ràng.`
  - `Logging mức phù hợp cho case fail/network error (không log secret key/token raw đầy đủ).`
- Out-of-scope:
  - `Code tích hợp FE Next.js (load site key, lấy token ở client).`
  - `Thay đổi business logic lõi ngoài phần guard verify.`
  - `Chuyển đổi toàn bộ endpoint hiện có sang reCAPTCHA nếu chưa nằm trong scope endpoint đã chốt.`

## 3) Acceptance criteria
1. `AC-01`: Chỉ 2 flow trong scope phase 1 (`customer login`, `customer register`) xử lý business khi verify reCAPTCHA hợp lệ; token thiếu/sai/hết hạn bị chặn.
2. `AC-02`: Secret key/threshold/action policy được đọc từ config core Magento reCAPTCHA (env/config path), không hardcode trong code.
3. `AC-03`: Khi Google verify API lỗi (timeout/network/5xx), hệ thống xử lý theo policy fail-closed: từ chối request, trả lỗi rõ ràng và không làm crash flow.
4. `AC-04`: Thiết kế theo service tách biệt, có thể tái sử dụng cho nhiều endpoint/form khác.
5. `AC-05`: Không làm thay đổi hành vi endpoint hiện tại ngoài việc thêm lớp kiểm tra bảo mật trước xử lý nghiệp vụ cho login/register.

## 4) Xác nhận trước khi code
- Requirement understanding:
  - `Mục tiêu là thêm guard reCAPTCHA ở BE Magento, không implement FE.`
  - `Token từ FE phải được verify server-side với secret key cấu hình.`
  - `Giải pháp cần reusable, theo hướng headless API-first, không gắn cứng vào controller form truyền thống.`
- Risks/assumptions:
  - `Nếu chọn fail-open khi Google lỗi sẽ giảm bảo mật; fail-closed an toàn hơn nhưng có thể ảnh hưởng UX khi Google outage.`
  - `Cần kiểm soát timeout HTTP client để tránh treo request.`
  - `Score threshold quá cao có thể chặn nhầm user thật; quá thấp làm giảm hiệu quả chống bot.`
- Options + recommendation:
  - `Option A (khuyến nghị): Tạo module riêng reusable (ví dụ `Secomm_RecaptchaGuard`) chỉ chứa adapter/service verify; ưu tiên đọc config core Magento reCAPTCHA; các flow customer login/register gọi verifier qua DI; policy fail-closed, validate `score >= 0.5` và `action` đúng context.`
  - `Option B: Plugin/observer intercept nhiều endpoint để verify tập trung; mạnh nhưng khó kiểm soát side effect ở phase đầu.`
- Decision đã chốt:
  - `Scope phase 1: flow customer login và customer register (không gồm admin).`
  - `Bối cảnh hệ thống: headless, ưu tiên điểm tích hợp ở API layer (GraphQL/REST/custom endpoint).`
  - `Policy khi Google verify lỗi: fail-closed.`
  - `Validation phase 1: `score >= 0.5` + `action` đúng với flow.`
  - `Kỹ thuật: tách module riêng theo hướng reusable, nhưng dùng config core Magento reCAPTCHA (không tạo system config mới ở phase 1).`
  - `Secret key cấu hình qua system/env config, không hardcode.`
- Phạm vi được phép sửa:
  - `app/code/<Vendor>/<ModuleRecaptcha>/**` (module riêng cho reCAPTCHA guard/adapter)
  - `etc/di.xml`, service/model/plugin/controller/resolver liên quan trong scope endpoint`
  - `Không sửa vendor/magento core.`
- Điểm cần xác nhận thêm:
  1. `Endpoint/controller cụ thể map cho login/register trong hệ thống hiện tại (customer/account/loginPost, createPost hay custom API route)?`
  2. `Action string chuẩn sẽ dùng cho từng flow (ví dụ `customer_login`, `customer_register`) để FE/BE thống nhất.`
  3. `Map cụ thể các config path core Magento reCAPTCHA sẽ được dùng cho BE headless trong phase 1?`

## 5) Testcase
- Happy path: `TC-01` login với token hợp lệ (`action` đúng, `score >= 0.5`) -> pass; `TC-02` register với token hợp lệ (`action` đúng, `score >= 0.5`) -> pass.
- Edge case: `TC-03` Google verify timeout/5xx -> hệ thống trả lỗi theo policy, không crash.
- Negative case: `TC-04` thiếu token; `TC-05` token invalid/expired; `TC-06` action mismatch; `TC-07` score < 0.5.
