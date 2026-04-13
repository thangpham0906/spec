# GraphQL — Schema (Guide): Gift registry (queries + mutations)

Tóm tắt nhóm **Gift registry** trên Adobe: gồm query phục vụ tìm/đọc registry và mutation phục vụ tạo/cập nhật/chia sẻ/xóa registry. File này tổng hợp phần **queries + mutations** theo các link Adobe bạn gửi. Chi tiết field: từng link **Nguồn**.  
**Chữ ký theo bản:** [GraphQL API reference](https://developer.adobe.com/commerce/webapi/graphql/reference/) — xem [`reference.md`](./reference.md).

---

## 1. Gift registry (nhóm schema)

Nguồn: [Gift registry](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/)

- Đây là tính năng **Adobe Commerce exclusive** (B2C/B2B storefront có thể dùng làm wishlist nâng cao theo sự kiện).
- Luồng chính: customer tạo registry -> chia sẻ cho người nhận -> người nhận mua theo registry -> hệ thống cập nhật số lượng fulfillment.
- Cần bật và cấu hình Gift Registry trong Admin trước khi dùng đầy đủ workflow.

---

## 2. Gift registry — danh sách queries (mục lục)

Nguồn: [Gift registry queries](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/queries/)

| Query | Nguồn Adobe | Trong file này |
|-------|-------------|----------------|
| `giftRegistry` | [giftRegistry](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/queries/gift-registry/) | §3 |
| `giftRegistryEmailSearch` | [giftRegistryEmailSearch](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/queries/email-search/) | §4 |
| `giftRegistryIdSearch` | [giftRegistryIdSearch](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/queries/id-search/) | §5 |
| `giftRegistryTypes` | [giftRegistryTypes](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/queries/types/) | §6 |
| `giftRegistryTypeSearch` | [giftRegistryTypeSearch](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/queries/type-search/) | §7 |

---

## 3. `giftRegistry`

Nguồn: [giftRegistry](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/queries/gift-registry/)

- Trả chi tiết một gift registry theo `uid`.
- Query này yêu cầu **customer authentication token** hợp lệ.
- Adobe gợi ý lấy danh sách `uid` hợp lệ từ `customer` query (dữ liệu account owner).
- Cú pháp (theo doc): `giftRegistry(uid: ID!): GiftRegistry`

---

## 4. `giftRegistryEmailSearch`

Nguồn: [giftRegistryEmailSearch](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/queries/email-search/)

- Tìm registry theo email của **registrant** (không phải owner email).
- Có thể dùng cho guest hoặc customer để tra registry public/searchable.
- Cú pháp (theo doc): `giftRegistryEmailSearch(email: String!): [GiftRegistrySearchResult]`

---

## 5. `giftRegistryIdSearch`

Nguồn: [giftRegistryIdSearch](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/queries/id-search/)

- Tìm registry theo `giftRegistryUid` (ID thường nằm trong email mời chia sẻ registry).
- Trả danh sách kết quả dạng `GiftRegistrySearchResult`.
- Cú pháp (theo doc): `giftRegistryIdSearch(giftRegistryUid: String!): [GiftRegistrySearchResult]`

---

## 6. `giftRegistryTypes`

Nguồn: [giftRegistryTypes](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/queries/types/)

- Trả danh sách loại registry khả dụng (ví dụ birthday, wedding...), kèm metadata thuộc tính động cho từng loại.
- Hữu ích để build form tạo registry động ở frontend.
- Cú pháp (theo doc): `giftRegistryTypes: [GiftRegistryType]`

---

## 7. `giftRegistryTypeSearch`

Nguồn: [giftRegistryTypeSearch](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/queries/type-search/)

- Tìm registry theo tên registrant (`firstName`, `lastName`) và optional `giftRegistryTypeUid`.
- Có thể kết hợp với `giftRegistryTypes` để lấy type uid trước khi search.
- Cú pháp (theo doc): `giftRegistryTypeSearch(firstName: String!, lastName: String!, giftRegistryTypeUid: String): [GiftRegistrySearchResult]`

---

## 8. Gift registry — danh sách mutations (mục lục)

Nguồn: [Gift registry mutations](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/mutations/)

| Mutation | Nguồn Adobe | Trong file này |
|----------|-------------|----------------|
| `addGiftRegistryRegistrants` | [addGiftRegistryRegistrants](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/mutations/add-registrants/) | §9 |
| `createGiftRegistry` | [createGiftRegistry](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/mutations/create/) | §10 |
| `moveCartItemsToGiftRegistry` | [moveCartItemsToGiftRegistry](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/mutations/move-cart-items/) | §11 |
| `removeGiftRegistry` | [removeGiftRegistry](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/mutations/remove/) | §12 |
| `removeGiftRegistryItems` | [removeGiftRegistryItems](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/mutations/remove-items/) | §13 |
| `removeGiftRegistryRegistrants` | [removeGiftRegistryRegistrants](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/mutations/remove-registrants/) | §14 |
| `shareGiftRegistry` | [shareGiftRegistry](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/mutations/share/) | §15 |
| `updateGiftRegistry` | [updateGiftRegistry](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/mutations/update/) | §16 |
| `updateGiftRegistryItems` | [updateGiftRegistryItems](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/mutations/update-items/) | §17 |
| `updateGiftRegistryRegistrants` | [updateGiftRegistryRegistrants](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/mutations/update-registrants/) | §18 |

---

## 9. `addGiftRegistryRegistrants`

Nguồn: [addGiftRegistryRegistrants](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/mutations/add-registrants/)

- Thêm một hoặc nhiều `registrants` vào gift registry đã có.
- Yêu cầu **customer authentication token** hợp lệ.
- Cú pháp (theo doc): `addGiftRegistryRegistrants(giftRegistryUid: ID!, registrants: [AddGiftRegistryRegistrantInput!]!): AddGiftRegistryRegistrantsOutput`

---

## 10. `createGiftRegistry`

Nguồn: [createGiftRegistry](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/mutations/create/)

- Tạo mới gift registry cho customer đang đăng nhập.
- Yêu cầu **customer authentication token** hợp lệ.
- `id` trong input là optional; nếu không truyền thì hệ thống tự sinh.
- `dynamic_attributes` là mảng code-value để lưu thuộc tính động cho registry/registrant.
- Với `shipping_address`, chỉ truyền **một trong hai**: `address_data` hoặc `address_id`.
- Cú pháp (theo doc): `createGiftRegistry(giftRegistry: CreateGiftRegistryInput!): CreateGiftRegistryOutput`

---

## 11. `moveCartItemsToGiftRegistry`

Nguồn: [moveCartItemsToGiftRegistry](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/mutations/move-cart-items/)

- Di chuyển toàn bộ item từ cart sang gift registry chỉ định.
- Yêu cầu **customer authentication token** hợp lệ.
- Response thường gồm `gift_registry`, `status`, và `user_errors` để xử lý item không move được.
- Cú pháp (theo doc): `moveCartItemsToGiftRegistry(cartUid: ID!, giftRegistryUid: ID!): MoveCartItemsToGiftRegistryOutput`

---

## 12. `removeGiftRegistry`

Nguồn: [removeGiftRegistry](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/mutations/remove/)

- Xóa gift registry khỏi danh sách registry của customer.
- Yêu cầu **customer authentication token** hợp lệ.
- Output thường trả cờ `success`.
- Cú pháp (theo doc): `removeGiftRegistry(giftRegistryUid: ID!): RemoveGiftRegistryOutput`

---

## 13. `removeGiftRegistryItems`

Nguồn: [removeGiftRegistryItems](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/mutations/remove-items/)

- Xóa một hoặc nhiều item khỏi gift registry.
- Yêu cầu **customer authentication token** hợp lệ.
- Input dùng danh sách `itemsUid`.
- Cú pháp (theo doc): `removeGiftRegistryItems(giftRegistryUid: ID!, itemsUid: [ID!]!): RemoveGiftRegistryItemsOutput`

---

## 14. `removeGiftRegistryRegistrants`

Nguồn: [removeGiftRegistryRegistrants](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/mutations/remove-registrants/)

- Xóa một hoặc nhiều registrant khỏi gift registry.
- Yêu cầu **customer authentication token** hợp lệ.
- Input dùng danh sách `registrantsUid`.
- Cú pháp (theo doc): `removeGiftRegistryRegistrants(giftRegistryUid: ID!, registrantsUid: [ID!]!): RemoveGiftRegistryRegistrantsOutput`

---

## 15. `shareGiftRegistry`

Nguồn: [shareGiftRegistry](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/mutations/share/)

- Gửi lời mời qua email để người nhận có thể mua theo gift registry.
- Yêu cầu **customer authentication token** hợp lệ.
- Input cần `sender` và danh sách `invitees`.
- Cú pháp (theo doc): `shareGiftRegistry(giftRegistryUid: ID!, sender: ShareGiftRegistrySenderInput!, invitees: [ShareGiftRegistryInviteeInput!]!): ShareGiftRegistryOutput`

---

## 16. `updateGiftRegistry`

Nguồn: [updateGiftRegistry](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/mutations/update/)

- Cập nhật thông tin metadata của registry (ví dụ privacy, message, dynamic attributes).
- Không dùng mutation này để sửa item/registrant; dùng `updateGiftRegistryItems` hoặc `updateGiftRegistryRegistrants`.
- Yêu cầu **customer authentication token** hợp lệ.
- Cú pháp (theo doc): `updateGiftRegistry(giftRegistryUid: ID!, giftRegistry: UpdateGiftRegistryInput!): UpdateGiftRegistryOutput`

---

## 17. `updateGiftRegistryItems`

Nguồn: [updateGiftRegistryItems](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/mutations/update-items/)

- Cập nhật item trong registry: số lượng yêu cầu và ghi chú (`note`).
- Yêu cầu **customer authentication token** hợp lệ.
- Input dùng `items: [UpdateGiftRegistryItemInput!]!`.
- Cú pháp (theo doc): `updateGiftRegistryItems(giftRegistryUid: ID!, items: [UpdateGiftRegistryItemInput!]!): UpdateGiftRegistryItemsOutput`

---

## 18. `updateGiftRegistryRegistrants`

Nguồn: [updateGiftRegistryRegistrants](https://developer.adobe.com/commerce/webapi/graphql/schema/gift-registry/mutations/update-registrants/)

- Cập nhật thông tin của một hoặc nhiều registrant trong registry.
- Yêu cầu **customer authentication token** hợp lệ.
- Input dùng `registrants: [UpdateGiftRegistryRegistrantInput!]!`.
- Cú pháp (theo doc): `updateGiftRegistryRegistrants(giftRegistryUid: ID!, registrants: [UpdateGiftRegistryRegistrantInput!]!): UpdateGiftRegistryRegistrantsOutput`

---

## Liên kết

- Mục lục folder: [`README.md`](./README.md)
- Customer: [`schema-customer.md`](./schema-customer.md)
- Cart: [`schema-cart.md`](./schema-cart.md)
- Checkout: [`schema-checkout.md`](./schema-checkout.md)
