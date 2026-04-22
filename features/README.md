# Feature Workspace

Mỗi feature dùng 1 thư mục riêng:

`.spec/features/<tên-feature>/`

Tối thiểu gồm:

- `spec.md`
- `plan.md`
- `tasks.md`
- `status.md`

Bạn có thể copy nhanh từ:

`.spec/features/_template/`

Legacy path vẫn dùng được nếu cần:

- `.spec/specs/`
- `.spec/plans/`
- `.spec/tasks/`

---

## Phân loại Feature trước khi tạo spec

Trước khi tạo bất kỳ file spec nào, AI **bắt buộc** xác định feature thuộc loại nào.

### Feature nhỏ — dùng template thông thường

Đáp ứng **TẤT CẢ** các điều kiện sau:

- Chỉ liên quan 1 module hoặc 1 bounded domain
- Có thể hoàn thành trong ≤ 5 tasks
- Không có phần async/queue tách biệt với phần sync
- Không có phần API + Admin UI hoàn toàn độc lập nhau

→ Tạo thẳng `features/<tên>/` dùng `_template/`

### Feature lớn / Epic — cần tách sub-features

Có **ÍT NHẤT 1** trong các dấu hiệu sau:

- Cần tạo hoặc sửa hơn 1 module
- Span qua nhiều bounded domain (ví dụ: catalog + order + payment)
- Có phần async/queue tách biệt với phần sync
- Có phần API + Admin UI độc lập nhau
- Ước tính hơn 5 tasks nếu làm 1 spec duy nhất

→ Tạo `features/<tên-epic>/epic.md` trước để định nghĩa boundary và sub-features, sau đó mỗi sub-feature dùng `_template/` như bình thường

### Câu hỏi AI phải hỏi user khi chưa rõ

> "Feature này có liên quan đến nhiều module hoặc nhiều domain không?
> Ước tính bao nhiêu tasks nếu làm 1 spec?"

Chỉ sau khi có câu trả lời mới được quyết định tạo Epic hay Feature thường.
