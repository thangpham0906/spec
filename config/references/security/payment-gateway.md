# Tham khảo: Payment Gateway (Tích hợp thanh toán)

Nguồn: https://developer.adobe.com/commerce/php/development/payments-integrations/payment-gateway/

---

## 0. Khởi tạo Module (module.xml)

Module thanh toán bắt buộc phải khai báo phụ thuộc (`sequence`) để đảm bảo hệ thống load đúng thứ tự.

```xml
<module name="Vendor_MyPayment" setup_version="1.0.0">
    <sequence>
        <module name="Magento_Sales"/>
        <module name="Magento_Payment"/>
        <module name="Magento_Checkout"/>
    </sequence>
</module>
```

---

## 1. Cấu trúc Facade & Command Pool (di.xml)

Mọi phương thức thanh toán hiện đại đều sử dụng Class Facade (`Magento\Payment\Model\Method\Adapter`) làm giao diện chính.

### 1a. Khai báo Facade (Method Adapter)
```xml
<virtualType name="MyPaymentFacade" type="Magento\Payment\Model\Method\Adapter">
    <arguments>
        <argument name="code" xsi:type="string">my_payment_code</argument>
        <argument name="formBlockType" xsi:type="string">Magento\Payment\Block\Form</argument>
        <argument name="infoBlockType" xsi:type="string">Vendor\Module\Block\Info</argument>
        <argument name="valueHandlerPool" xsi:type="object">MyPaymentValueHandlerPool</argument>
        <argument name="commandPool" xsi:type="object">MyPaymentCommandPool</argument>
        <argument name="validatorPool" xsi:type="object">MyPaymentValidatorPool</argument>
    </arguments>
</virtualType>
```

### 1b. Value Handler Pool (Xử lý logic cấu hình)
Dùng để ghi đè các tham số trong `config.xml` bằng logic PHP (ví dụ: `can_void` tùy theo trạng thái giao dịch).
```xml
<virtualType name="MyPaymentValueHandlerPool" type="Magento\Payment\Gateway\Config\ValueHandlerPool">
    <arguments>
        <argument name="handlers" xsi:type="array">
            <item name="default" xsi:type="string">MyPaymentConfigValueHandler</item>
            <item name="can_void" xsi:type="string">Vendor\Module\Gateway\Config\CanVoidHandler</item>
        </argument>
    </arguments>
</virtualType>
```

### 1c. Validator Pool (Kiểm tra điều kiện)
Dùng để kiểm tra Country, Currency, hoặc rủi ro trước khi đặt hàng.
```xml
<virtualType name="MyPaymentValidatorPool" type="Magento\Payment\Gateway\Validator\ValidatorPool">
    <arguments>
        <argument name="validators" xsi:type="array">
            <item name="country" xsi:type="string">Vendor\Module\Gateway\Validator\CountryValidator</item>
        </argument>
    </arguments>
</virtualType>
```

### 1d. Khai báo Command Pool
Ánh xạ các hành động của Đơn hàng vào các class xử lý tương ứng.
```xml
<virtualType name="MyPaymentCommandPool" type="Magento\Payment\Gateway\Command\CommandPool">
    <arguments>
        <argument name="commands" xsi:type="array">
            <item name="authorize" xsi:type="string">MyPaymentAuthorizeCommand</item>
            <item name="capture" xsi:type="string">MyPaymentCaptureCommand</item>
            <item name="void" xsi:type="string">MyPaymentVoidCommand</item>
            <item name="refund" xsi:type="string">MyPaymentRefundCommand</item>
        </argument>
    </arguments>
</virtualType>
```

---

## 2. Chi tiết một Gateway Command

Mỗi lệnh thường là một `virtualType` của `Magento\Payment\Gateway\Command\GatewayCommand` với các tham số sau:

| Tham số | Vai trò |
|---------|---------|
| **RequestBuilder** | Chuyển dữ liệu Order/Payment thành mảng (Array) cho API. |
| **TransferFactory**| Tạo đối tượng request HTTP (URL, Method, Headers). |
| **Client** | Thực hiện gọi API (Sử dụng Curl hoặc SDK). |
| **Validator** | Kiểm tra mã lỗi trả về từ API để quyết định thành công/thất bại. |
| **Handler** | Lưu thông tin giao dịch (Transaction ID) vào Payment object. |

---

## 3. Request Builder (Dữ liệu gửi đi)

Build từ `PaymentDataObject`. Tránh sử dụng trực tiếp Model Order để đảm bảo tính Stateless.

### 3a. Builder đơn lẻ
Class thực thi `Magento\Payment\Gateway\Request\BuilderInterface`.

```php
public function build(array $buildSubject)
{
    $paymentDO = SubjectReader::readPayment($buildSubject);
    return ['AMOUNT' => $paymentDO->getOrder()->getGrandTotalAmount()];
}
```

### 3b. Builder Composite (Khuyên dùng)
Dùng `di.xml` để gom nhiều builder nhỏ lại.

```xml
<virtualType name="MyPaymentAuthorizeRequest" type="Magento\Payment\Gateway\Request\BuilderComposite">
    <arguments>
        <argument name="builders" xsi:type="array">
            <item name="customer" xsi:type="string">Vendor\Module\Gateway\Request\CustomerBuilder</item>
            <item name="address" xsi:type="string">Vendor\Module\Gateway\Request\AddressBuilder</item>
            <item name="amount" xsi:type="string">Vendor\Module\Gateway\Request\AmountBuilder</item>
        </argument>
    </arguments>
</virtualType>
```

---

## 4. Các file cấu hình & UI

- **`etc/payment.xml`**: Khai báo phương thức thanh toán.
- **`etc/config.xml`**: Cấu hình mặc định.
- **`Block/Info.php`**: Hiển thị thông tin thanh toán trong trang Order (Admin/Frontend).
- **`Block/Form.php`**: Template nhập liệu (thường dùng trong Admin khi tạo đơn hàng thủ công).

---

## 5. Lưu ý quan trọng

1. **Security:** Không bao giờ lưu số thẻ (PAN) hoặc CVV vào database của Magento.
2. **Stateless:** Các lớp trong Gateway nên luôn là Stateless để tương thích với App Server.
3. **Logging:** Luôn dùng `Magento\Payment\Model\Method\Logger` để ghi log debug các giao dịch (nhớ ẩn các thông tin nhạy cảm).

---

## 6. Gateway Client & Transfer Factory

Đây là lớp thực hiện kết nối HTTP thực tế.

- **Transfer Factory:** Chuẩn bị "gói hàng" (URL, Method, Headers, Body).
- **Client:** "Gửi hàng" đi (Sử dụng `Magento\Payment\Gateway\Http\Client\Zend` mặc định hoặc Custom Curl).

```php
// Ví dụ trong TransferFactory
public function create(array $request) {
    return $this->transferBuilder
        ->setMethod(Curl::POST)
        ->setHeaders(['Content-Type' => 'application/json'])
        ->setBody(json_encode($request))
        ->setUri($this->getApiUrl())
        ->build();
}
```

---

## 7. Response Validator (Kiểm tra phản hồi)

Quyết định xem phản hồi từ ngân hàng là Thành công hay Thất bại.

```php
public function validate(array $validationSubject) {
    $response = SubjectReader::readResponse($validationSubject);
    $isValid = isset($response['RESULT']) && $response['RESULT'] == 'SUCCESS';
    
    return $this->createResult($isValid, $isValid ? [] : [__('Giao dịch bị từ chối.')]);
}
```

---

## 8. Response Handler (Lưu dữ liệu)

Lưu các thông tin quan trọng (Transaction ID, Card Type...) vào đối tượng Payment của Magento.

```php
public function handle(array $handlingSubject, array $response) {
    $paymentDO = $this->subjectReader->readPayment($handlingSubject);
    $payment = $paymentDO->getPayment();
    
    // Đánh dấu giao dịch và lưu ID từ ngân hàng
    $payment->setTransactionId($response['TXN_ID']);
    $payment->setIsTransactionClosed(false); // Giữ transaction mở để có thể Void/Refund sau này
}
```

---

## 9. Error Code Mapper

Giúp ánh xạ mã lỗi kỹ thuật sang ngôn ngữ con người. Khai báo trong `di.xml` thông qua `errorMessageMapper`.

- **Mã kỹ thuật:** `1001` -> **Thông báo:** "Thẻ của quý khách đã hết hạn."
- **Mã kỹ thuật:** `2005` -> **Thông báo:** "Số dư không đủ để thực hiện giao dịch."

---

## 10. Cấu hình Flags (config.xml)

Nơi định nghĩa các tính năng mà cổng thanh toán của bạn hỗ trợ.

```xml
<default>
    <payment>
        <my_payment_code>
            <active>1</active>
            <model>MyPaymentFacade</model> <!-- Trỏ tới virtualType Facade trong di.xml -->
            <title>Thanh toán qua Ngân hàng</title>
            <can_authorize>1</can_authorize>
            <can_capture>1</can_capture>
            <can_void>1</can_void>
            <can_refund>1</can_refund>
            <can_use_checkout>1</can_use_checkout>
            <can_use_internal>1</can_use_internal> <!-- Cho phép dùng trong Admin -->
            <payment_action>authorize_capture</payment_action>
            <order_status>processing</order_status>
        </my_payment_code>
    </payment>
</default>
```

---

## 11. Nhận dữ liệu từ Frontend (Data Assign Observer)

Khi khách hàng nhấn "Place Order", dữ liệu từ JS Checkout được gửi về. Bạn cần một Observer để lưu dữ liệu này vào `additional_information`.

**Events.xml:**
```xml
<event name="payment_method_assign_data_my_payment_code">
    <observer name="my_payment_data_assign" instance="Vendor\Module\Observer\DataAssignObserver" />
</event>
```

**Observer Class:**
Kế thừa `Magento\Payment\Observer\AbstractDataAssignObserver`.

```php
public function execute(Observer $observer) {
    $data = $this->readDataArgument($observer);
    $additionalData = $data->getData(PaymentInterface::KEY_ADDITIONAL_DATA);
    $paymentInfo = $this->readPaymentModelArgument($observer);

    if (isset($additionalData['payment_token'])) {
        $paymentInfo->setAdditionalInformation('payment_token', $additionalData['payment_token']);
    }
}
```

---

## 12. Phân vùng cấu hình (Areas)

- **`etc/frontend/di.xml`**: Chứa cấu hình riêng cho Checkout (ví dụ: cần 3D Secure).
- **`etc/adminhtml/di.xml`**: Chứa cấu hình cho Admin (ví dụ: bỏ qua 3D Secure khi nhân viên tạo đơn hàng).

---

## 13. Xác thực 3D Secure (CardinalCommerce)

3D Secure là lớp bảo mật bắt buộc để xác thực chủ thẻ qua ngân hàng phát hành.

### Luồng xử lý:
1. **JS Checkout:** Nhận mã `cardinalJWT` sau khi khách xác thực OTP.
2. **Observer:** Lưu `cardinalJWT` vào `additional_information`.
3. **Request Builder:** Giải mã JWT để lấy dữ liệu xác thực gửi cho Gateway API.

```php
/**
 * Trích xuất thông tin 3DS từ JWT để gửi đi
 */
public function build(array $buildSubject): array {
    $paymentDO = $this->subjectReader->readPayment($buildSubject);
    $payment = $paymentDO->getPayment();
    
    $cardinalJwt = (string)$payment->getAdditionalInformation('cardinalJWT');
    $jwtPayload = $this->jwtParser->execute($cardinalJwt); // Trả về array Payload
    
    $eciFlag = $jwtPayload['Payload']['Payment']['ExtendedData']['ECIFlag'] ?? '';
    $cavv = $jwtPayload['Payload']['Payment']['ExtendedData']['CAVV'] ?? '';

    return [
        'transactionRequest' => [
            'cardholderAuthentication' => [
                'authenticationIndicator' => $eciFlag,
                'cardholderAuthenticationValue' => $cavv
            ],
        ]
    ];
}
```

---

## 14. Offline Payment Method (Adapter Pattern)

Dùng `Magento\Payment\Model\Method\Adapter` qua `virtualType` — không extend `AbstractMethod` (deprecated).

**`etc/config.xml`** — các flag quan trọng cho offline:
```xml
<vendor_offline>
    <model>VendorOfflinePaymentFacade</model><!-- virtualType name trong di.xml -->
    <group>offline</group>
    <is_offline>1</is_offline>
    <can_use_checkout>1</can_use_checkout>
    <can_use_internal>1</can_use_internal>
    <order_status>pending</order_status>
</vendor_offline>
```

**`etc/di.xml`** — Facade tối giản cho offline (không cần commandPool):
```xml
<virtualType name="VendorOfflinePaymentFacade" type="Magento\Payment\Model\Method\Adapter">
    <arguments>
        <argument name="code" xsi:type="const">Vendor\Module\Model\Ui\ConfigProvider::CODE</argument>
        <argument name="formBlockType" xsi:type="string">Magento\Payment\Block\Form</argument>
        <argument name="infoBlockType" xsi:type="string">Magento\Payment\Block\Info</argument>
        <argument name="valueHandlerPool" xsi:type="object">VendorOfflinePaymentValueHandlerPool</argument>
        <!-- commandPool KHÔNG cần cho offline — không có authorize/capture/void -->
    </arguments>
</virtualType>
```

> Blueprint đầy đủ: xem [examples/integration/custom-payment-offline-blueprint.md](../../examples/integration/custom-payment-offline-blueprint.md)

---

## Liên kết

- Architectural Patterns: xem [architectural-patterns.md](./architectural-patterns.md)
- Service Contracts: xem [service-contracts.md](./service-contracts.md)
- CLI & Maintenance: xem [maintenance-cli.md](./maintenance-cli.md)
