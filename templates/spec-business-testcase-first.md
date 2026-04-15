# Spec Template - Business First + Testcase First

> Mục tiêu: giảm "vibe coding", chốt đúng nghiệp vụ trước khi code, và verify bằng testcase đã định nghĩa.

## 1) Thông tin feature

- Feature name: `<kebab-case-feature-name>`
- Owner/requester: `<team/person>`
- Priority: `<P0/P1/P2>`
- Deadline (nếu có): `<date>`
- Related ticket/link: `<Jira/GitHub/...>`

---

## 2) Business analysis (viết trước)

### 2.1 Business problem

Mô tả vấn đề hiện tại theo ngôn ngữ nghiệp vụ:

- Hiện trạng: `<đang xảy ra gì>`
- Tác động business: `<mất doanh thu / tăng chi phí / UX xấu / rủi ro vận hành>`
- Đối tượng bị ảnh hưởng: `<admin/customer/ops/...>`

### 2.2 Business goal

- Mục tiêu cần đạt: `<kết quả business mong muốn>`
- KPI/Success metric (nếu đo được): `<metric + target>`

### 2.3 Scope nghiệp vụ

- In-scope:
  - `<flow/chức năng nằm trong phạm vi>`
- Out-of-scope:
  - `<flow/chức năng không làm ở task này>`

### 2.4 Acceptance criteria (business)

1. `<AC-01 theo format Given/When/Then hoặc điều kiện rõ ràng>`
2. `<AC-02 ...>`
3. `<AC-03 ...>`

---

## 3) Functional/technical requirements

- Functional requirements:
  - `<FR-01>`
  - `<FR-02>`
- Non-functional requirements:
  - `<performance/security/audit/logging/...>`

Ràng buộc Magento/project:

- Tuân theo `.spec/config/constitution.md`
- Tuân theo `.spec/config/magento-patterns.md`

---

## 4) Bắt buộc xác nhận trước khi implement

> AI/Dev cần trả lời rõ phần này trước khi sửa code.

### 4.1 Xác nhận hiểu đúng requirement

- Tóm tắt lại requirement bằng 3-6 bullet:
  - `<ý 1>`
  - `<ý 2>`
  - `<ý 3>`

### 4.2 Review scope trước khi làm

- Review/implement/debug type: `<review | implement | debug | mixed>`
- Phạm vi được phép sửa (bắt buộc ghi rõ file/module):
  - `<app/code/Vendor/ModuleA/...>`
  - `<app/code/Vendor/ModuleB/...>`
- Phạm vi không được sửa:
  - `<core Magento hoặc module khác>`

### 4.3 Các điểm cần xác nhận thêm (nếu thiếu thông tin)

- Câu hỏi 1: `<...>`
- Câu hỏi 2: `<...>`
- Câu hỏi 3: `<...>`

> Nếu còn thiếu thông tin quan trọng: dừng và hỏi lại trước khi code.

---

## 5) Gợi ý giải pháp (options + khuyến nghị)

### Option A - `<tên approach>`
- Mô tả ngắn: `<...>`
- Ưu điểm: `<...>`
- Nhược điểm/rủi ro: `<...>`

### Option B - `<tên approach>`
- Mô tả ngắn: `<...>`
- Ưu điểm: `<...>`
- Nhược điểm/rủi ro: `<...>`

### Khuyến nghị
- Chọn option: `<A/B/...>`
- Lý do chọn theo business impact + maintainability: `<...>`

---

## 6) Testcase design (viết trước hoặc song song code)

### 6.1 Mapping AC -> Testcases

| AC ID | Testcase IDs |
|---|---|
| AC-01 | TC-01, TC-02 |
| AC-02 | TC-03 |
| AC-03 | TC-04, TC-05 |

### 6.2 Danh sách testcase

#### Happy path
- TC-01: `<mô tả>`
- TC-02: `<mô tả>`

#### Edge cases
- TC-03: `<mô tả>`
- TC-04: `<mô tả>`

#### Negative cases
- TC-05: `<mô tả>`
- TC-06: `<mô tả>`

### 6.3 Dữ liệu test

- Store/website scope: `<...>`
- Product/customer/sample data: `<...>`
- Preconditions: `<...>`

---

## 7) Implementation plan

1. `<bước 1>`
2. `<bước 2>`
3. `<bước 3>`

Files/module dự kiến thay đổi:

- `<path-1>`
- `<path-2>`

---

## 8) Verify steps (chạy sau khi code)

### 8.1 Automated checks

- `<unit/integration/static test command>`
- `<phpcs/phpstan/...>`

### 8.2 Manual checks theo testcase

| Testcase ID | Kết quả | Ghi chú |
|---|---|---|
| TC-01 | Pass/Fail | `<...>` |
| TC-02 | Pass/Fail | `<...>` |
| TC-03 | Pass/Fail | `<...>` |

---

## 9) Output report (bắt buộc khi hoàn thành)

1. Files changed:
   - `<file-path>`
2. Lý do thay đổi:
   - `<why>`
3. Verify steps đã chạy:
   - `<command/step + result>`

---

## 10) Prompt mẫu để giao cho AI

```text
Đọc `.spec/config/constitution.md` và `.spec/config/magento-patterns.md`.
Đọc spec tại `.spec/specs/<ten-feature>.md`.
Nếu task cần review/implement/debug, chọn skill phù hợp theo `.spec/skills/skills-list.md`.

Trước khi làm:
1) Xác nhận lại bạn hiểu đúng requirements.
2) Nêu rõ review scope và phạm vi được phép sửa: <files/module>.
3) Nêu các điểm còn thiếu cần mình xác nhận.
4) Đề xuất 1-2 hướng giải pháp và khuyến nghị hướng tốt nhất.

Sau đó mới implement.
Báo cáo cuối gồm: (1) files changed, (2) lý do thay đổi, (3) verify steps đã chạy và kết quả testcase.
Nếu thiếu thông tin quan trọng, hỏi lại trước khi sửa code.
```
