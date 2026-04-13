# GraphQL — Schema (Guide): Customer (queries + mutations)

Tóm tắt nhóm **Customer** trên Adobe: đây là nhóm schema cho dữ liệu và thao tác customer account. File này tổng hợp **queries** (§2–§10) và **mutations** (§11–§34) theo danh sách Adobe bạn yêu cầu. Chi tiết field: từng link **Nguồn**.  
**Chữ ký theo bản:** [GraphQL API reference](https://developer.adobe.com/commerce/webapi/graphql/reference/) — xem [`reference.md`](./reference.md).

---

## 1. Customer (nhóm schema)

Nguồn: [Customer](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/)

- Customer là shopper đã có account trên store.
- Adobe khuyến nghị dùng customer token ở header cho các GraphQL call liên quan customer (cũng có thể dùng session auth trong một số luồng).
- Với B2B Adobe Commerce, object `Customer` có thêm field top-level như `job_title`, `role`, `status`, `team`, `telephone`, `requisition_lists`.

---

## 2. Customer — danh sách queries (mục lục)

Nguồn: [Customer queries](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/)

| Query | Nguồn Adobe | Trong file này |
|-------|-------------|----------------|
| `customer` | [customer](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/customer/) | §3 |
| `customerCart` | [customerCart](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/cart/) | §4 |
| `customerDownloadableProducts` | [customerDownloadableProducts](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/downloadable-products/) | §5 |
| `customerGroup` | [customerGroup](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/customer-group/) | §6 |
| `customerOrders` | [customerOrders](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/orders/) | §7 |
| `customerSegments` | [customerSegments](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/customer-segments/) | §8 |
| `giftCardAccount` | [giftCardAccount](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/giftcard-account/) | §9 |
| `isEmailAvailable` | [isEmailAvailable](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/is-email-available/) | §10 |

---

## 3. `customer`

Nguồn: [customer](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/customer/)

- Trả thông tin customer đăng nhập: profile, address, custom attributes, orders summary (qua `customer.orders`), wishlist, store credit, và các field mở rộng khác theo module.
- Adobe khuyến nghị gửi customer token ở header.
- Cú pháp (theo doc): `customer: Customer`

---

## 4. `customerCart`

Nguồn: [customerCart](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/cart/)

- Trả active cart của customer đăng nhập; nếu chưa có cart thì query sẽ tạo mới.
- Không nhận input `cart_id` (khác với query `cart`).
- Có thể lấy `id` cart để dùng trong `mergeCarts` khi hợp nhất guest cart vào customer cart.
- Cú pháp (theo doc): `customerCart: Cart!`
- Yêu cầu customer authorization token.

---

## 5. `customerDownloadableProducts`

Nguồn: [customerDownloadableProducts](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/downloadable-products/)

- **PaaS only**.
- Trả danh sách downloadable products customer đã mua.
- Cú pháp (theo doc): `customerDownloadableProducts: CustomerDownloadableProducts`
- Lỗi thường gặp: `The current customer isn't authorized.`

---

## 6. `customerGroup`

Nguồn: [customerGroup](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/customer-group/)

- Query thuộc **Storefront Compatibility Package**, dự kiến thêm vào Adobe Commerce **2.4.9**.
- Trả encoded `uid` của customer group cho logged-in customer hoặc guest.
- Cú pháp (theo doc): `customerGroup: CustomerGroupStorefront!`
- Với logged-in customer nên gửi token; với guest thì không truyền token.

---

## 7. `customerOrders`

Nguồn: [customerOrders](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/orders/)

- **PaaS only**.
- Query này đã **deprecated**; Adobe khuyến nghị dùng `customer.orders` thay thế.
- Cú pháp (theo doc): `customerOrders: CustomerOrders`
- Lỗi thường gặp: `The current customer isn't authorized.`

---

## 8. `customerSegments`

Nguồn: [customerSegments](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/customer-segments/)

- Query thuộc **Storefront Compatibility Package**, sẽ có trong Adobe Commerce **2.4.9**.
- Trả encoded `uid` của customer segments gán cho logged-in customer hoặc guest.
- Cú pháp (theo doc): `customerSegments(cartId: String!): [CustomerSegmentStorefront!]`
- Với logged-in customer nên gửi token; guest thì không truyền token.

---

## 9. `giftCardAccount`

Nguồn: [giftCardAccount](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/giftcard-account/)

- Trả thông tin một gift card cụ thể (code, balance, expiration_date...).
- Cú pháp (theo doc): `giftCardAccount(code: String!): GiftCardAccount`  
  (Doc ví dụ dùng input object `gift_card_code`; khi triển khai thực tế nên kiểm tra reference theo version đang chạy).
- Lỗi thường gặp: gift card không tồn tại/đã redeem hết, hoặc thiếu `gift_card_code`.

---

## 10. `isEmailAvailable`

Nguồn: [isEmailAvailable](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/queries/is-email-available/)

- Kiểm tra email đã được dùng để tạo account customer chưa.
- Adobe lưu ý thay đổi từ **2.4.7**: mặc định query luôn trả `true`; hành vi cũ có thể bật lại qua Admin (`Enable Guest Checkout Login`), nhưng có rủi ro lộ thông tin cho user chưa auth.
- Cú pháp (theo doc): `isEmailAvailable(email: String!): IsEmailAvailableOutput`
- Lỗi thường gặp: email format sai, hoặc thiếu argument `email`.

---

## 11. Customer — danh sách mutations (mục lục)

Nguồn: [Customer mutations](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/)

| Mutation | Nguồn Adobe | Trong file này |
|----------|-------------|----------------|
| `updateCustomerV2` | [updateCustomerV2](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/update-v2/) | §12 |
| `updateCustomerEmail` | [updateCustomerEmail](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/update-email/) | §13 |
| `updateCustomerAddressV2` | [updateCustomerAddressV2](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/update-address-v2/) | §14 |
| `updateCustomerAddress` | [updateCustomerAddress](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/update-address/) | §15 |
| `updateCustomer` | [updateCustomer](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/update/) | §16 |
| `subscribeEmailToNewsletter` | [subscribeEmailToNewsletter](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/subscribe-email-to-newsletter/) | §17 |
| `sendEmailToFriend` | [sendEmailToFriend](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/send-email-to-friend/) | §18 |
| `revokeCustomerToken` | [revokeCustomerToken](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/revoke-token/) | §19 |
| `resetPassword` | [resetPassword](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/reset-password/) | §20 |
| `resendConfirmationEmail` | [resendConfirmationEmail](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/resend-confirmation-email/) | §21 |
| `requestPasswordResetEmail` | [requestPasswordResetEmail](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/request-password-reset-email/) | §22 |
| `generateCustomerTokenAsAdmin` | [generateCustomerTokenAsAdmin](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/generate-token-as-admin/) | §23 |
| `generateCustomerToken` | [generateCustomerToken](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/generate-token/) | §24 |
| `exchangeOtpForCustomerToken` | [exchangeOtpForCustomerToken](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/exchange-otp-customer-token/) | §25 |
| `exchangeExternalCustomerToken` | [exchangeExternalCustomerToken](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/exchange-external-customer-token/) | §26 |
| `deleteCustomerAddressV2` | [deleteCustomerAddressV2](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/delete-address-v2/) | §27 |
| `deleteCustomerAddress` | [deleteCustomerAddress](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/delete-address/) | §28 |
| `createCustomerV2` | [createCustomerV2](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/create-v2/) | §29 |
| `createCustomerAddress` | [createCustomerAddress](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/create-address/) | §30 |
| `createCustomer` | [createCustomer](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/create/) | §31 |
| `confirmEmail` | [confirmEmail](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/confirm-email/) | §32 |
| `changeCustomerPassword` | [changeCustomerPassword](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/change-password/) | §33 |
| `assignCompareListToCustomer` | [assignCompareListToCustomer](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/assign-compare-list/) | §34 |

---

## 12. `updateCustomerV2`

Nguồn: [updateCustomerV2](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/update-v2/)

- Mutation cập nhật thông tin customer hiện có; Adobe khuyến nghị dùng mutation này thay cho `updateCustomer`.
- Dùng input `CustomerUpdateInput` (khác với `CustomerInput` ở luồng create/update cũ).
- Từ 2.4.7 có thể cập nhật thêm `custom_attributes`.
- Cú pháp (theo doc): `updateCustomerV2(input: CustomerUpdateInput!): CustomerOutput`
- Cần customer token/session auth.

---

## 13. `updateCustomerEmail`

Nguồn: [updateCustomerEmail](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/update-email/)

- Dùng để đổi email của customer đang đăng nhập.
- Yêu cầu truyền email mới + password hiện tại của customer.
- Cú pháp (theo doc): `updateCustomerEmail(email: String!, password: String!): CustomerOutput`

---

## 14. `updateCustomerAddressV2`

Nguồn: [updateCustomerAddressV2](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/update-address-v2/)

- **SaaS only** và thuộc **Storefront Compatibility Package** (sẽ có trong 2.4.9).
- Cập nhật địa chỉ customer theo `uid` (ID dạng encoded).
- Hỗ trợ cập nhật `custom_attributesV2`.
- Cú pháp (theo doc): `updateCustomerAddressV2(uid: ID!, input: CustomerAddressInput): CustomerAddress`
- Lỗi thường gặp: thiếu/sai `uid`, không có quyền trên address, thiếu `input`, chưa auth.

---

## 15. `updateCustomerAddress`

Nguồn: [updateCustomerAddress](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/update-address/)

- Đã deprecated trên Adobe Commerce as a Cloud Service và sẽ deprecated trên Adobe Commerce 2.4.9; Adobe khuyến nghị chuyển sang `updateCustomerAddressV2`.
- Cập nhật địa chỉ customer theo `id` kiểu số nguyên.
- Cú pháp (theo doc): `updateCustomerAddress(id: Int!, input: CustomerAddressInput): CustomerAddress`
- Lỗi thường gặp: thiếu/sai `id`, không có quyền, thiếu `input`, chưa auth.

---

## 16. `updateCustomer`

Nguồn: [updateCustomer](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/update/)

- **PaaS only**.
- Mutation cập nhật thông tin cá nhân customer; Adobe khuyến nghị ưu tiên `updateCustomerV2`.
- Cú pháp (theo doc): `updateCustomer(input: CustomerInput!): CustomerOutput`
- Nếu đổi email trong input thì cần password hiện tại.

---

## 17. `subscribeEmailToNewsletter`

Nguồn: [subscribeEmailToNewsletter](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/subscribe-email-to-newsletter/)

- Cho phép guest hoặc customer đăng ký newsletter.
- Kết quả thường là `SUBSCRIBED` hoặc `NOT_ACTIVE` theo cấu hình xác nhận.
- Cú pháp (theo doc): `subscribeEmailToNewsletter(email: String!): SubscribeEmailToNewsletterOutput`
- Lỗi thường gặp: email sai format, email đã subscribe, guest subscription bị tắt.

---

## 18. `sendEmailToFriend`

Nguồn: [sendEmailToFriend](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/send-email-to-friend/)

- **PaaS only**.
- Cho phép gửi email giới thiệu sản phẩm cho bạn bè.
- Cần bật cấu hình Admin: **Catalog > Email to a friend > Enabled = Yes**.
- Cú pháp (theo doc): `sendEmailToFriend(input: SendEmailToFriendInput): SendEmailToFriendOutput`
- Có giới hạn theo giờ và có thể yêu cầu auth tùy cấu hình cho guest.

---

## 19. `revokeCustomerToken`

Nguồn: [revokeCustomerToken](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/revoke-token/)

- Thu hồi customer token hiện tại; trả `result: true` nếu thành công.
- Cú pháp (theo doc): `revokeCustomerToken: RevokeCustomerTokenOutput`
- Lỗi thường gặp: customer chưa đăng nhập/token không hợp lệ.

---

## 20. `resetPassword`

Nguồn: [resetPassword](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/reset-password/)

- Đặt mật khẩu mới bằng `resetPasswordToken` + email, sau bước `requestPasswordResetEmail`.
- Trả boolean (`true`/`false`).
- Cú pháp (theo doc): `resetPassword(email: String!, resetPasswordToken: String!, newPassword: String!): Boolean`
- Token reset có trong email reset (và có thể tra từ DB theo doc Adobe).

---

## 21. `resendConfirmationEmail`

Nguồn: [resendConfirmationEmail](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/resend-confirmation-email/)

- Gửi lại email confirm cho account chưa kích hoạt.
- Trả boolean (`true`/`false`).
- Cú pháp (theo doc): `resendConfirmationEmail(email: String!): Boolean`
- Lỗi thường gặp: email không tồn tại, customer đã confirm, lỗi gửi mail hệ thống.

---

## 22. `requestPasswordResetEmail`

Nguồn: [requestPasswordResetEmail](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/request-password-reset-email/)

- Khởi tạo luồng reset password bằng cách gửi email chứa reset link/token.
- Trả boolean (`true`/`false`).
- Cú pháp (theo doc): `requestPasswordResetEmail(email: String!): Boolean`
- Dùng token từ link để gọi `resetPassword`.

---

## 23. `generateCustomerTokenAsAdmin`

Nguồn: [generateCustomerTokenAsAdmin](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/generate-token-as-admin/)

- Admin tạo customer token để hỗ trợ mua hàng từ xa thay cho customer.
- Chỉ hoạt động khi customer bật `allow_remote_shopping_assistance`.
- Cú pháp (theo doc): `generateCustomerTokenAsAdmin(input: GenerateCustomerTokenAsAdminInput!): GenerateCustomerTokenAsAdminOutput`

---

## 24. `generateCustomerToken`

Nguồn: [generateCustomerToken](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/generate-token/)

- **SaaS only** theo trang doc hiện tại.
- Tạo customer access token cho luồng đăng nhập.
- Adobe mô tả có thể dùng `password` theo các kiểu credential hợp lệ (customer password / reset token / OTC trong luồng login-as-customer, tùy cấu hình).
- Cú pháp (theo doc): `generateCustomerToken(email: String!, password: String!): CustomerToken`

---

## 25. `exchangeOtpForCustomerToken`

Nguồn: [exchangeOtpForCustomerToken](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/exchange-otp-customer-token/)

- **SaaS only**.
- Đổi email + OTP thành customer token; OTP bị invalidate sau khi dùng.
- Có tích hợp reCAPTCHA để giảm abuse tự động.
- Cú pháp (theo doc): `exchangeOtpForCustomerToken(email: String!, otp: String!): CustomerToken`

---

## 26. `exchangeExternalCustomerToken`

Nguồn: [exchangeExternalCustomerToken](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/exchange-external-customer-token/)

- Thuộc **Storefront Compatibility Package**, sẽ có trong Adobe Commerce **2.4.9**.
- Hỗ trợ social/external login qua App Builder; có thể tạo mới customer nếu chưa tồn tại và trả token.
- Yêu cầu integration với external auth system (OAuth 1.0) và credentials integration tương ứng.
- Cú pháp (theo doc): `exchangeExternalCustomerToken(input: ExchangeExternalCustomerTokenInput!): ExchangeExternalCustomerTokenOutput`

---

## 27. `deleteCustomerAddressV2`

Nguồn: [deleteCustomerAddressV2](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/delete-address-v2/)

- **SaaS only** và thuộc **Storefront Compatibility Package** (2.4.9).
- Xóa địa chỉ customer theo `uid`; trả boolean thành công/thất bại.
- Không thể xóa địa chỉ đang là default billing/shipping.
- Cú pháp (theo doc): `deleteCustomerAddressV2(uid: ID!): Boolean`

---

## 28. `deleteCustomerAddress`

Nguồn: [deleteCustomerAddress](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/delete-address/)

- Đã deprecated trên Adobe Commerce as a Cloud Service và sẽ deprecated trên Adobe Commerce 2.4.9; Adobe khuyến nghị dùng `deleteCustomerAddressV2`.
- Xóa địa chỉ theo `id: Int!`; trả boolean.
- Không thể xóa địa chỉ default billing/shipping.
- Cú pháp (theo doc): `deleteCustomerAddress(id: Int!): Boolean`

---

## 29. `createCustomerV2`

Nguồn: [createCustomerV2](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/create-v2/)

- Mutation chuẩn để tạo customer account mới; thay thế `createCustomer`.
- Dùng input `CustomerCreateInput` (không dùng chung object với update v2).
- Từ 2.4.7 có thể truyền `custom_attributes`.
- Cú pháp (theo doc): `createCustomerV2(input: CustomerCreateInput!): CustomerOutput`

---

## 30. `createCustomerAddress`

Nguồn: [createCustomerAddress](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/create-address/)

- Tạo địa chỉ customer mới từ `CustomerAddressInput`.
- Hỗ trợ custom attributes cho address; trong SaaS có thêm case custom file upload.
- Cú pháp (theo doc): `createCustomerAddress(input: CustomerAddressInput!): CustomerAddress`
- Lỗi thường gặp: thiếu field bắt buộc, `street` vượt số dòng cho phép, chưa auth.

---

## 31. `createCustomer`

Nguồn: [createCustomer](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/create/)

- **PaaS only**.
- Mutation tạo customer kiểu cũ; Adobe khuyến nghị dùng `createCustomerV2`.
- Cú pháp (theo doc): `createCustomer(input: CustomerInput!): CustomerOutput`
- Lỗi thường gặp: email trùng, email sai format, thiếu firstname/email.

---

## 32. `confirmEmail`

Nguồn: [confirmEmail](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/confirm-email/)

- Hoàn tất kích hoạt account bằng `email` + `confirmation_key`.
- Cú pháp (theo doc): `confirmEmail(input: ConfirmEmailInput!): CustomerOutput`
- Lỗi thường gặp: token xác nhận không hợp lệ, account đã active, email không hợp lệ.

---

## 33. `changeCustomerPassword`

Nguồn: [changeCustomerPassword](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/change-password/)

- Đổi password cho customer đang đăng nhập.
- Cần truyền `currentPassword` và `newPassword`.
- Cú pháp (theo doc): `changeCustomerPassword(currentPassword: String!, newPassword: String!): Customer`
- Lỗi thường gặp: chưa auth, current password sai, account bị khóa.

---

## 34. `assignCompareListToCustomer`

Nguồn: [assignCompareListToCustomer](https://developer.adobe.com/commerce/webapi/graphql/schema/customer/mutations/assign-compare-list/)

- Gán compare list (thường tạo khi guest) cho customer đã đăng nhập.
- Yêu cầu customer authentication token hợp lệ.
- Cú pháp (theo doc): `assignCompareListToCustomer(uid: ID!): AssignCompareListToCustomerOutput`

---

## Liên kết

- Mục lục folder: [`README.md`](./README.md)
- Cart: [`schema-cart.md`](./schema-cart.md)
- Checkout: [`schema-checkout.md`](./schema-checkout.md)
- Company (B2B): [`schema-company.md`](./schema-company.md)
