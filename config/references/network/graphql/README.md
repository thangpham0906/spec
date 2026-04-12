# GraphQL — Web API (Adobe Commerce)

Tài liệu gốc nằm trên **Adobe Developer — Commerce Web APIs → GraphQL**. Trong spec này, toàn bộ mục **Usage** (endpoint, token, cache, filter, response, headers, introspection, protected mutations, security, staging) gom tại [`usage.md`](./usage.md); **Reference** schema theo phiên bản tại [`reference.md`](./reference.md). Nhánh **Schema (guide)** — mục lục query/mutation theo domain trên Adobe: [`schema-attributes.md`](./schema-attributes.md) (**Attributes**), [`schema-cart.md`](./schema-cart.md) (**Cart** — queries §2–§4; mutations mục lục §5 chi tiết §6–§39; **interfaces** `CartItemInterface` §40), [`schema-catalog-service.md`](./schema-catalog-service.md) (**Catalog Service** + **`productSearch`** Live Search — §1–§7).

Nhánh **Development** (định nghĩa `schema.graphqls`, resolver/batch resolver, mở rộng schema, Identity/cache tag, urlResolver tùy chỉnh, debug, exception, functional test): [`development.md`](./development.md).  
**App Server / resolver stateless** (ràng buộc khi chạy long-lived PHP): [`../graphql-app-server.md`](../graphql-app-server.md) — bổ sung cho doc Adobe, không thay thế.

---

## Phiên bản (align với project)

- Constitution: **Magento 2.4.8** → khi tra **GraphQL API reference**, chọn bản **2.4.8** (PaaS / Magento Open Source / Adobe Commerce on-prem), không nhầm với chỉ **SaaS** nếu bạn không dùng stack đó.

---

## Mục lục nhanh

| Chủ đề | Adobe (chính thức) | Trong `.spec` |
|--------|-------------------|----------------|
| Schema reference (queries/mutations theo bản) | [GraphQL API reference](https://developer.adobe.com/commerce/webapi/graphql/reference/) | [`reference.md`](./reference.md) |
| Schema guide (mục lục theo domain: Attributes, Cart, Catalog Service, …) | [GraphQL schema](https://developer.adobe.com/commerce/webapi/graphql/schema/) | [`schema-attributes.md`](./schema-attributes.md) §1; [`schema-cart.md`](./schema-cart.md) §1; [`schema-catalog-service.md`](./schema-catalog-service.md) §1; nhóm khác → Adobe |
| Chạy query/mutation, endpoint, usage tổng quan | [GraphQL — Usage](https://developer.adobe.com/commerce/webapi/graphql/usage/) | [`usage.md`](./usage.md) §1 |
| Authorization / token | [Authorization](https://developer.adobe.com/commerce/webapi/graphql/usage/authorization-tokens/) | [`usage.md`](./usage.md) §2 |
| Caching | [Caching](https://developer.adobe.com/commerce/webapi/graphql/usage/caching/) | [`usage.md`](./usage.md) §3 |
| Lọc theo custom attributes | [Custom filters](https://developer.adobe.com/commerce/webapi/graphql/usage/custom-filters/) | [`usage.md`](./usage.md) §4 |
| HTTP status & payload lỗi | [Status codes and responses](https://developer.adobe.com/commerce/webapi/graphql/usage/api-response/) | [`usage.md`](./usage.md) §5 |
| Request headers (Store, Currency, Preview, Captcha, …) | [Headers](https://developer.adobe.com/commerce/webapi/graphql/usage/headers/) | [`usage.md`](./usage.md) §6 |
| Introspection (`__schema`, tắt trên production) | [Introspection queries](https://developer.adobe.com/commerce/webapi/graphql/usage/introspection-queries/) | [`usage.md`](./usage.md) §7 |
| Mutation cần CAPTCHA / reCAPTCHA | [Protected mutations](https://developer.adobe.com/commerce/webapi/graphql/usage/protected-mutations/) | [`usage.md`](./usage.md) §8 |
| Giới hạn độ phức tạp/độ sâu query | [Security configuration](https://developer.adobe.com/commerce/webapi/graphql/usage/security-configuration/) | [`usage.md`](./usage.md) §9 |
| Preview nội dung staging (Adobe Commerce) | [Staging queries](https://developer.adobe.com/commerce/webapi/graphql/usage/staging-queries/) | [`usage.md`](./usage.md) §10 |

### Development (schema, resolver, mở rộng)

| Chủ đề | Adobe (chính thức) | Trong `.spec` |
|--------|-------------------|----------------|
| Định nghĩa schema module (`schema.graphqls`, query/mutation/enum, `@doc`, caching) | [Define schema](https://developer.adobe.com/commerce/webapi/graphql/develop/) | [`development.md`](./development.md) §1 |
| Resolvers (`ResolverInterface`, batch) | [Resolvers](https://developer.adobe.com/commerce/webapi/graphql/develop/resolvers/) | [`development.md`](./development.md) §2 |
| Mở rộng schema (stitching, plugin, …) | [Extend existing schema](https://developer.adobe.com/commerce/webapi/graphql/develop/extend-existing-schema/) | [`development.md`](./development.md) §3 |
| Identity / cache tags (`IdentityInterface`, `@cache`) | [Identity class](https://developer.adobe.com/commerce/webapi/graphql/develop/identity-class/) | [`development.md`](./development.md) §4 |
| `urlResolver` + URL rewrite | [Custom urlResolver](https://developer.adobe.com/commerce/webapi/graphql/develop/create-custom-url-resolver/) | [`development.md`](./development.md) §5 |
| Debug (Xdebug, PhpStorm) | [Debugging](https://developer.adobe.com/commerce/webapi/graphql/develop/debugging/) | [`development.md`](./development.md) §6 |
| Exception (`ClientAware`, `GraphQl*Exception`) | [Exceptions](https://developer.adobe.com/commerce/webapi/graphql/develop/exceptions/) | [`development.md`](./development.md) §7 |
| Functional testing | [Functional testing](https://developer.adobe.com/commerce/webapi/graphql/develop/functional-testing/) | [`development.md`](./development.md) §8 |

### Schema — Attributes (queries EAV / custom metadata)

| Chủ đề | Adobe (chính thức) | Trong `.spec` |
|--------|-------------------|----------------|
| Attributes (tổng quan queries + mutations) | [Attributes](https://developer.adobe.com/commerce/webapi/graphql/schema/attributes/) | [`schema-attributes.md`](./schema-attributes.md) §2 |
| Danh sách queries | [Attributes queries](https://developer.adobe.com/commerce/webapi/graphql/schema/attributes/queries/) | [`schema-attributes.md`](./schema-attributes.md) §3 |
| `attributesForm` | [attributesForm](https://developer.adobe.com/commerce/webapi/graphql/schema/attributes/queries/attributes-form/) | [`schema-attributes.md`](./schema-attributes.md) §4 |
| `attributesList` | [attributesList](https://developer.adobe.com/commerce/webapi/graphql/schema/attributes/queries/attributes-list/) | [`schema-attributes.md`](./schema-attributes.md) §5 |
| `attributesMetadata` (deprecated) | [attributesMetadata](https://developer.adobe.com/commerce/webapi/graphql/schema/attributes/queries/attributes-metadata/) | [`schema-attributes.md`](./schema-attributes.md) §6 |
| `customAttributeMetadata` (deprecated) | [customAttributeMetadata](https://developer.adobe.com/commerce/webapi/graphql/schema/attributes/queries/custom-attribute-metadata/) | [`schema-attributes.md`](./schema-attributes.md) §7 |
| `customAttributeMetadataV2` | [customAttributeMetadataV2](https://developer.adobe.com/commerce/webapi/graphql/schema/attributes/queries/custom-attribute-metadata-v2/) | [`schema-attributes.md`](./schema-attributes.md) §8 |

### Schema — Attributes (mutations `setCustomAttributesOn*`)

| Chủ đề | Adobe (chính thức) | Trong `.spec` |
|--------|-------------------|----------------|
| Custom attribute mutations (tổng quan, cài PaaS, bảng mutation) | [Custom attribute mutations](https://developer.adobe.com/commerce/webapi/graphql/schema/attributes/mutations/) | [`schema-attributes.md`](./schema-attributes.md) §9 |
| `setCustomAttributesOnCart` | [setCustomAttributesOnCart](https://developer.adobe.com/commerce/webapi/graphql/schema/attributes/mutations/set-custom-cart/) | [`schema-attributes.md`](./schema-attributes.md) §10 |
| `setCustomAttributesOnCartItem` | [setCustomAttributesOnCartItem](https://developer.adobe.com/commerce/webapi/graphql/schema/attributes/mutations/set-custom-cart-item/) | [`schema-attributes.md`](./schema-attributes.md) §11 |
| `setCustomAttributesOnCompany` | [setCustomAttributesOnCompany](https://developer.adobe.com/commerce/webapi/graphql/schema/attributes/mutations/set-custom-company/) | [`schema-attributes.md`](./schema-attributes.md) §12 |
| `setCustomAttributesOnCreditMemo` | [setCustomAttributesOnCreditMemo](https://developer.adobe.com/commerce/webapi/graphql/schema/attributes/mutations/set-custom-credit-memo/) | [`schema-attributes.md`](./schema-attributes.md) §13 |
| `setCustomAttributesOnCreditMemoItem` | [setCustomAttributesOnCreditMemoItem](https://developer.adobe.com/commerce/webapi/graphql/schema/attributes/mutations/set-custom-credit-memo-item/) | [`schema-attributes.md`](./schema-attributes.md) §14 |
| `setCustomAttributesOnInvoice` | [setCustomAttributesOnInvoice](https://developer.adobe.com/commerce/webapi/graphql/schema/attributes/mutations/set-custom-invoice/) | [`schema-attributes.md`](./schema-attributes.md) §15 |
| `setCustomAttributesOnInvoiceItem` | [setCustomAttributesOnInvoiceItem](https://developer.adobe.com/commerce/webapi/graphql/schema/attributes/mutations/set-custom-invoice-item/) | [`schema-attributes.md`](./schema-attributes.md) §16 |
| `setCustomAttributesOnNegotiableQuote` | [setCustomAttributesOnNegotiableQuote](https://developer.adobe.com/commerce/webapi/graphql/schema/attributes/mutations/set-custom-negotiable-quote/) | [`schema-attributes.md`](./schema-attributes.md) §17 |
| Attribute interfaces (PaaS / SaaS) | [Interfaces](https://developer.adobe.com/commerce/webapi/graphql/schema/attributes/interfaces/) | [`schema-attributes.md`](./schema-attributes.md) §18 |

### Schema — Cart (queries)

| Chủ đề | Adobe (chính thức) | Trong `.spec` |
|--------|-------------------|----------------|
| Cart (tổng quan shopper / sidebar) | [Cart](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/) | [`schema-cart.md`](./schema-cart.md) §1 |
| Danh sách queries | [Cart queries](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/queries/) | [`schema-cart.md`](./schema-cart.md) §2 |
| `cart` | [cart](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/queries/cart/) | [`schema-cart.md`](./schema-cart.md) §3 |
| `pickupLocations` | [pickupLocations](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/queries/pickup-locations/) | [`schema-cart.md`](./schema-cart.md) §4 |

### Schema — Cart (mutations — thêm SP & khuyến mãi)

| Chủ đề | Adobe (chính thức) | Trong `.spec` |
|--------|-------------------|----------------|
| Mục lục mutations | [Cart mutations](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/) | [`schema-cart.md`](./schema-cart.md) §5 |
| `addProductsToCart` | [addProductsToCart](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/add-products/) | [`schema-cart.md`](./schema-cart.md) §6 |
| `addSimpleProductsToCart` | [addSimpleProductsToCart](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/add-simple-products/) | [`schema-cart.md`](./schema-cart.md) §7 |
| `addConfigurableProductsToCart` | [addConfigurableProductsToCart](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/add-configurable-products/) | [`schema-cart.md`](./schema-cart.md) §8 |
| `addBundleProductsToCart` | [addBundleProductsToCart](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/add-bundle-products/) | [`schema-cart.md`](./schema-cart.md) §9 |
| `addDownloadableProductsToCart` | [addDownloadableProductsToCart](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/add-downloadable-products/) | [`schema-cart.md`](./schema-cart.md) §10 |
| `addVirtualProductsToCart` | [addVirtualProductsToCart](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/add-virtual-products/) | [`schema-cart.md`](./schema-cart.md) §11 |
| `applyCouponToCart` | [applyCouponToCart](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/apply-coupon/) | [`schema-cart.md`](./schema-cart.md) §12 |
| `applyCouponsToCart` | [applyCouponsToCart](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/apply-coupons/) | [`schema-cart.md`](./schema-cart.md) §13 |
| `applyGiftCardToCart` | [applyGiftCardToCart](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/apply-giftcard/) | [`schema-cart.md`](./schema-cart.md) §14 |
| `applyRewardPointsToCart` | [applyRewardPointsToCart](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/apply-reward-points/) | [`schema-cart.md`](./schema-cart.md) §15 |

### Schema — Cart (mutations — gỡ dòng/mã, redeem, đặt hàng, giỏ, estimate)

| Chủ đề | Adobe (chính thức) | Trong `.spec` |
|--------|-------------------|----------------|
| `removeItemFromCart` | [removeItemFromCart](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/remove-item/) | [`schema-cart.md`](./schema-cart.md) §16 |
| `removeGiftCardFromCart` | [removeGiftCardFromCart](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/remove-giftcard/) | [`schema-cart.md`](./schema-cart.md) §17 |
| `removeCouponsFromCart` | [removeCouponsFromCart](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/remove-coupons/) | [`schema-cart.md`](./schema-cart.md) §18 |
| `removeCouponFromCart` | [removeCouponFromCart](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/remove-coupon/) | [`schema-cart.md`](./schema-cart.md) §19 |
| `redeemGiftCardBalanceAsStoreCredit` | [redeemGiftCardBalanceAsStoreCredit](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/redeem-giftcard-balance/) | [`schema-cart.md`](./schema-cart.md) §20 |
| `placeOrder` | [placeOrder](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/place-order/) | [`schema-cart.md`](./schema-cart.md) §21 |
| `mergeCarts` | [mergeCarts](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/merge/) | [`schema-cart.md`](./schema-cart.md) §22 |
| `estimateTotals` | [estimateTotals](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/estimate-totals/) | [`schema-cart.md`](./schema-cart.md) §23 |
| `estimateShippingMethods` | [estimateShippingMethods](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/estimate-shipping-methods/) | [`schema-cart.md`](./schema-cart.md) §24 |
| `createGuestCart` | [createGuestCart](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/create-guest-cart/) | [`schema-cart.md`](./schema-cart.md) §25 |
| `createEmptyCart` | [createEmptyCart](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/create-empty-cart/) | [`schema-cart.md`](./schema-cart.md) §26 |
| `clearCart` | [clearCart](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/clear-cart/) | [`schema-cart.md`](./schema-cart.md) §27 |
| `assignCustomerToGuestCart` | [assignCustomerToGuestCart](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/assign-customer-to-guest-cart/) | [`schema-cart.md`](./schema-cart.md) §28 |
| `applyStoreCreditToCart` | [applyStoreCreditToCart](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/apply-store-credit/) | [`schema-cart.md`](./schema-cart.md) §29 |

### Schema — Cart (mutations — chỉnh dòng, địa chỉ, ship, thanh toán, guest email, quà, gỡ reward/credit)

| Chủ đề | Adobe (chính thức) | Trong `.spec` |
|--------|-------------------|----------------|
| `updateCartItems` | [updateCartItems](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/update-items/) | [`schema-cart.md`](./schema-cart.md) §30 |
| `setShippingMethodsOnCart` | [setShippingMethodsOnCart](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/set-shipping-method/) | [`schema-cart.md`](./schema-cart.md) §31 |
| `setShippingAddressesOnCart` | [setShippingAddressesOnCart](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/set-shipping-address/) | [`schema-cart.md`](./schema-cart.md) §32 |
| `setPaymentMethodOnCart` | [setPaymentMethodOnCart](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/set-payment-method/) | [`schema-cart.md`](./schema-cart.md) §33 |
| `setPaymentMethodAndPlaceOrder` | [setPaymentMethodAndPlaceOrder](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/set-payment-place-order/) | [`schema-cart.md`](./schema-cart.md) §34 |
| `setGuestEmailOnCart` | [setGuestEmailOnCart](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/set-guest-email/) | [`schema-cart.md`](./schema-cart.md) §35 |
| `setGiftOptionsOnCart` | [setGiftOptionsOnCart](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/set-gift-options/) | [`schema-cart.md`](./schema-cart.md) §36 |
| `setBillingAddressOnCart` | [setBillingAddressOnCart](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/set-billing-address/) | [`schema-cart.md`](./schema-cart.md) §37 |
| `removeStoreCreditFromCart` | [removeStoreCreditFromCart](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/remove-store-credit/) | [`schema-cart.md`](./schema-cart.md) §38 |
| `removeRewardPointsFromCart` | [removeRewardPointsFromCart](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/mutations/remove-reward-points/) | [`schema-cart.md`](./schema-cart.md) §39 |

### Schema — Cart (interfaces)

| Chủ đề | Adobe (chính thức) | Trong `.spec` |
|--------|-------------------|----------------|
| `CartItemInterface` (implementations) | [Cart — Interfaces](https://developer.adobe.com/commerce/webapi/graphql/schema/cart/interfaces/) | [`schema-cart.md`](./schema-cart.md) §40 |

### Schema — Catalog Service & Live Search (`productSearch`)

| Chủ đề | Adobe (chính thức) | Trong `.spec` |
|--------|-------------------|----------------|
| Catalog Service (tổng quan) | [Catalog Service](https://developer.adobe.com/commerce/webapi/graphql/schema/catalog-service/) | [`schema-catalog-service.md`](./schema-catalog-service.md) §1 |
| `categories` | [categories](https://developer.adobe.com/commerce/webapi/graphql/schema/catalog-service/queries/categories/) | [`schema-catalog-service.md`](./schema-catalog-service.md) §3 |
| `products` | [products](https://developer.adobe.com/commerce/webapi/graphql/schema/catalog-service/queries/products/) | [`schema-catalog-service.md`](./schema-catalog-service.md) §4 |
| `productSearch` | [productSearch](https://developer.adobe.com/commerce/webapi/graphql/schema/live-search/queries/product-search/) | [`schema-catalog-service.md`](./schema-catalog-service.md) §5 |
| `refineProduct` | [refineProduct](https://developer.adobe.com/commerce/webapi/graphql/schema/catalog-service/queries/refine-product/) | [`schema-catalog-service.md`](./schema-catalog-service.md) §6 |
| `variants` | [variants](https://developer.adobe.com/commerce/webapi/graphql/schema/catalog-service/queries/product-variants/) | [`schema-catalog-service.md`](./schema-catalog-service.md) §7 |

---

## Liên kết

- [GraphQL Guide (Adobe)](https://developer.adobe.com/commerce/webapi/graphql/) — cổng tổng
- Schema guide — Cart: [`schema-cart.md`](./schema-cart.md)
- Schema guide — Catalog Service: [`schema-catalog-service.md`](./schema-catalog-service.md)
- Schema guide — Attributes: [`schema-attributes.md`](./schema-attributes.md)
- Development (tóm tắt Adobe): [`development.md`](./development.md)
- App Server & resolver stateless: [`../graphql-app-server.md`](../graphql-app-server.md)
- REST/SOAP Web API: [`../web-api.md`](../web-api.md)
