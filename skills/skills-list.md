# Skills List

## Chọn skill theo task

| Task type | Skill gợi ý |
|---|---|
| Code review trước commit/PR | `magento-code-reviewer` |
| Security review / hardening | `magento-security-analyst` |
| Tạo/sửa module Magento 2 | `magento-module-developer` |
| Build feature end-to-end theo spec | `magento-feature-developer` |
| Thiết kế model/resource/collection/repository | `magento-model-developer` |
| Debug bug khó tìm (runtime/integration) | `magento-issue-debugger` |
| Thiết kế/duy trì REST + GraphQL APIs | `magento-api-developer` |
| Tối ưu performance (query, cache, bottleneck) | `magento-performance-analyst` |
| Phân tích indexer/reindex/search issues | `magento-index-analyst` |
| Frontend theme/PHTML/JS/CSS Magento | `magento-frontend-developer` |
| KnockoutJS components/bindings checkout | `magento-knockout-specialist` |
| UI Components (admin grid/form/listing) | `magento-ui-component-developer` |
| XML config (di/layout/routes/system/acl) | `magento-xml-specialist` |
| PHP refactor/chuẩn hóa coding standards | `magento-php-specialist` |
| Cấu hình hệ thống/Admin config scope | `magento-configuration-specialist` |
| Catalog domain (product/category/attributes/pricing flows) | `magento-catalog-specialist` |
| Hyva storefront customization | `magento-hyva-specialist` |
| Nâng cấp Magento / backward compatibility | `magento-upgrade-specialist` |
| Setup môi trường dev/staging/prod | `magento-environment-engineer` |
| Setup local stack (Docker/DDEV/WSL) | `magento-local-environment-specialist` |
| Setup local với DDEV cho Magento | `ddev-magento` |
| Build backend tổng hợp (module + patterns) | `magento2-backend-toolkit` |
| Module scaffolding / module workflow | `magento-module-development` |
| Doc/spec update (không code module) | Không bắt buộc skill, follow `.spec/config/*` |
| Viết spec/doc ngắn gọn, bỏ verbose | `writing-clearly-and-concisely` |
| Quản lý context khi làm Epic/feature lớn nhiều sub-features | `context-engineering-collection` |

---

## 1) Magento Code Reviewer

- Skill: `magento-code-reviewer`
- Nguồn: [maxnorm/magento2-agent-skills](https://skills.sh/maxnorm/magento2-agent-skills/magento-code-reviewer)
- Local path (nếu đã tải về project): `.spec/skills/.agents/skills/magento-code-reviewer/SKILL.md`
- Mục đích:
  - Code review trước commit/PR
  - Security/performance/architecture checks
  - Magento coding standard compliance

### Cài đặt

```bash
npx skills add https://github.com/maxnorm/magento2-agent-skills --skill magento-code-reviewer
```

### Cách gọi trong prompt

```text
Dùng skill magento-code-reviewer.
Đọc .spec/config/constitution.md và .spec/config/magento-patterns.md.
Review code scope: <files/thư mục>.
Trả kết quả theo mức độ: Critical -> High -> Medium -> Low.
```

---

## 2) Magento Module Developer

- Skill: `magento-module-developer`
- Nguồn: [maxnorm/magento2-agent-skills](https://skills.sh/maxnorm/magento2-agent-skills/magento-module-developer)
- Local path (nếu đã tải về project): `.spec/skills/.agents/skills/magento-module-developer/SKILL.md`
- Mục đích:
  - Tạo module Magento 2 theo best practices
  - Service contracts, repository, db schema, di.xml

### Cài đặt

```bash
npx skills add https://github.com/maxnorm/magento2-agent-skills --skill magento-module-developer
```

### Cách gọi trong prompt

```text
Dùng skill magento-module-developer.
Đọc .spec/config/constitution.md và .spec/config/magento-patterns.md.
Implement feature theo spec: .spec/specs/<ten-feature>.md
Chỉ sửa trong phạm vi: <module path>.
```

---

## 3) Skills đã cài trong project (local)

Danh sách phát hiện trong `.spec/skills/.agents/skills/*`: **37 skills**

### 3.1 Backend / Architecture / Core

- `magento2-backend-toolkit`
- `magento-module-development`
- `magento-module-developer`
- `magento-feature-developer`
- `magento-model-developer`
- `magento-php-specialist`
- `magento-xml-specialist`
- `magento-api-developer`
- `magento-graphql`
- `magento-order-specialist`
- `magento-catalog-specialist`
- `magento-multi-store`
- `magento-configuration-specialist`

### 3.2 Frontend / Theme / Storefront

- `magento-frontend-developer`
- `magento-theme-developer`
- `magento-hyva-specialist`
- `magento-luma-specialist`
- `magento-ui-component-developer`
- `magento-knockout-specialist`
- `magento-requirejs-specialist`
- `magento-alpine-specialist`
- `magento-magewire-specialist`
- `magento-css-specialist`
- `magento2-widget-creation`

### 3.3 Performance / Cache / Index / Quality

- `magento-performance-analyst`
- `magento-cache-analyst`
- `magento-index-analyst`
- `magento-indexing-caching`
- `magento-security-analyst`
- `magento-code-reviewer`
- `magento-issue-debugger`

### 3.5 Workflow / Context / Writing Quality

- `context-engineering-collection`
- `writing-clearly-and-concisely`

### 3.4 DevOps / Environment / Release

- `magento-environment-engineer`
- `magento-local-environment-specialist`
- `ddev-magento`
- `magento-deployment-engineer`
- `magento-upgrade-specialist`
- `magento-cronjob-developer`

Pattern gọi nhanh trong prompt:

```text
Dùng skill <tên-skill>.
Đọc .spec/config/constitution.md và .spec/config/magento-patterns.md.
Scope: <files/thư mục>.
Output: <yêu cầu mong muốn + verify>.
```

---

## Ghi chú vận hành

- Nếu chưa cài skill, vẫn có thể follow quy trình local trong `.spec/config/*`.
- Ưu tiên sử dụng skill khi làm task lặp lại (review, module setup, pre-release checks).
- Khi prompt trong task quan trọng, nên ghi rõ scope + expected output + verify steps.
