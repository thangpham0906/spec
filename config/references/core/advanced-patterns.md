# Tham khảo: Advanced Patterns trong Magento 2

Nguồn:
- https://dev.to/grahamcrocker/implement-a-strategy-pattern-in-magento2-2mc5 — strategy pattern
- https://developer.adobe.com/commerce/php/development/payments-integrations/payment-gateway/command-pool/ — command pool
- https://developer.adobe.com/commerce/php/development/build/dependency-injection-file/ — DI patterns

---

## 1. Command pattern — CommandInterface, CommandPoolInterface

Dùng trong payment gateway, nhưng áp dụng được cho mọi use case cần pool of commands.

### Interface

```php
<?php
declare(strict_types=1);

namespace Vendor\Module\Api;

/**
 * @api
 * @since 1.0.0
 */
interface CommandInterface
{
    /**
     * @param array $commandSubject
     * @return \Magento\Payment\Gateway\Command\ResultInterface|null
     */
    public function execute(array $commandSubject): mixed;
}
```

### CommandPool

```php
<?php
declare(strict_types=1);

namespace Vendor\Module\Model;

use Vendor\Module\Api\CommandInterface;
use Magento\Framework\Exception\NotFoundException;

class CommandPool
{
    /**
     * @param CommandInterface[] $commands
     */
    public function __construct(
        private readonly array $commands = []
    ) {}

    public function get(string $commandCode): CommandInterface
    {
        if (!isset($this->commands[$commandCode])) {
            throw new NotFoundException(
                __('Command "%1" is not defined.', $commandCode)
            );
        }

        if (!$this->commands[$commandCode] instanceof CommandInterface) {
            throw new \LogicException(
                sprintf('Command "%s" must implement CommandInterface.', $commandCode)
            );
        }

        return $this->commands[$commandCode];
    }
}
```

### di.xml — cấu hình command pool

```xml
<!-- Tạo command pool với virtualType -->
<virtualType name="Vendor\Module\Model\OrderCommandPool"
             type="Vendor\Module\Model\CommandPool">
    <arguments>
        <argument name="commands" xsi:type="array">
            <item name="create" xsi:type="string">Vendor\Module\Model\Command\CreateOrderCommand</item>
            <item name="cancel" xsi:type="string">Vendor\Module\Model\Command\CancelOrderCommand</item>
            <item name="refund" xsi:type="string">Vendor\Module\Model\Command\RefundOrderCommand</item>
        </argument>
    </arguments>
</virtualType>

<!-- Inject pool vào service -->
<type name="Vendor\Module\Service\OrderService">
    <arguments>
        <argument name="commandPool" xsi:type="object">
            Vendor\Module\Model\OrderCommandPool
        </argument>
    </arguments>
</type>
```

### Sử dụng

```php
<?php
declare(strict_types=1);

namespace Vendor\Module\Service;

use Vendor\Module\Model\CommandPool;

class OrderService
{
    public function __construct(
        private readonly CommandPool $commandPool
    ) {}

    public function processOrder(string $action, array $data): mixed
    {
        return $this->commandPool->get($action)->execute($data);
    }
}
```

---

## 2. Strategy pattern — pool of strategies, dynamic selection via DI

Thay thế switch/if-else bằng pool of strategy classes.

### Interface

```php
<?php
declare(strict_types=1);

namespace Vendor\Module\Model\Processor;

/**
 * @api
 * @since 1.0.0
 */
interface ProcessorInterface
{
    /**
     * Kiểm tra strategy này có xử lý được type này không
     */
    public function isApplicable(string $type): bool;

    /**
     * Xử lý
     */
    public function process(array $data): array;
}
```

### Strategy pool

```php
<?php
declare(strict_types=1);

namespace Vendor\Module\Model\Processor;

use Magento\Framework\Exception\LocalizedException;

class ProcessorPool
{
    /**
     * @param ProcessorInterface[] $processors
     */
    public function __construct(
        private readonly array $processors = []
    ) {}

    public function getProcessor(string $type): ProcessorInterface
    {
        foreach ($this->processors as $processor) {
            if ($processor->isApplicable($type)) {
                return $processor;
            }
        }

        throw new LocalizedException(
            __('No processor found for type "%1".', $type)
        );
    }
}
```

### Concrete strategies

```php
<?php
declare(strict_types=1);

namespace Vendor\Module\Model\Processor;

class SimpleProductProcessor implements ProcessorInterface
{
    public function isApplicable(string $type): bool
    {
        return $type === 'simple';
    }

    public function process(array $data): array
    {
        // Logic cho simple product
        return array_merge($data, ['processed_by' => 'simple']);
    }
}

class ConfigurableProductProcessor implements ProcessorInterface
{
    public function isApplicable(string $type): bool
    {
        return $type === 'configurable';
    }

    public function process(array $data): array
    {
        // Logic cho configurable product
        return array_merge($data, ['processed_by' => 'configurable']);
    }
}
```

### di.xml

```xml
<type name="Vendor\Module\Model\Processor\ProcessorPool">
    <arguments>
        <argument name="processors" xsi:type="array">
            <item name="simple" xsi:type="object">
                Vendor\Module\Model\Processor\SimpleProductProcessor
            </item>
            <item name="configurable" xsi:type="object">
                Vendor\Module\Model\Processor\ConfigurableProductProcessor
            </item>
        </argument>
    </arguments>
</type>
```

---

## 3. Composite pattern — CompositeInterface, chaining processors

Dùng khi cần áp dụng nhiều processors theo thứ tự.

```php
<?php
declare(strict_types=1);

namespace Vendor\Module\Model\Processor;

class CompositeProcessor implements ProcessorInterface
{
    /**
     * @param ProcessorInterface[] $processors Sorted by sortOrder
     */
    public function __construct(
        private readonly array $processors = []
    ) {}

    public function isApplicable(string $type): bool
    {
        return true; // Composite áp dụng cho mọi type
    }

    public function process(array $data): array
    {
        foreach ($this->processors as $processor) {
            if ($processor->isApplicable($data['type'] ?? '')) {
                $data = $processor->process($data);
            }
        }
        return $data;
    }
}
```

---

## 4. Pipeline pattern — processor chain, SortedList

Magento dùng pattern này nhiều trong checkout totals, shipping rates...

```php
<?php
declare(strict_types=1);

namespace Vendor\Module\Model;

use Vendor\Module\Api\ProcessorInterface;

/**
 * Sorted processor chain — processors chạy theo sortOrder
 */
class ProcessorChain
{
    private array $sortedProcessors = [];

    /**
     * @param array $processors ['sortOrder' => ProcessorInterface]
     */
    public function __construct(array $processors = [])
    {
        // Sort theo key (sortOrder)
        ksort($processors);
        $this->sortedProcessors = $processors;
    }

    public function process(mixed $subject): mixed
    {
        foreach ($this->sortedProcessors as $processor) {
            $subject = $processor->process($subject);
        }
        return $subject;
    }
}
```

```xml
<!-- di.xml — processors với sortOrder -->
<type name="Vendor\Module\Model\ProcessorChain">
    <arguments>
        <argument name="processors" xsi:type="array">
            <item name="10" xsi:type="object">Vendor\Module\Model\Processor\ValidateProcessor</item>
            <item name="20" xsi:type="object">Vendor\Module\Model\Processor\TransformProcessor</item>
            <item name="30" xsi:type="object">Vendor\Module\Model\Processor\EnrichProcessor</item>
        </argument>
    </arguments>
</type>
```

---

## 5. Modifier pattern — UI Component modifier, pool modifier

Dùng trong UI Component để modify data/meta.

```php
<?php
declare(strict_types=1);

namespace Vendor\Module\Ui\DataProvider\Product\Form\Modifier;

use Magento\Catalog\Ui\DataProvider\Product\Form\Modifier\AbstractModifier;

class CustomModifier extends AbstractModifier
{
    public function modifyData(array $data): array
    {
        // Modify data trước khi render form
        foreach ($data as $productId => $productData) {
            $data[$productId]['product']['custom_field'] = 'custom_value';
        }
        return $data;
    }

    public function modifyMeta(array $meta): array
    {
        // Modify meta (field config, visibility...)
        $meta['product-details']['children']['custom_field'] = [
            'arguments' => [
                'data' => [
                    'config' => [
                        'label' => __('Custom Field'),
                        'componentType' => 'field',
                        'formElement' => 'input',
                        'dataType' => 'text',
                        'sortOrder' => 100,
                    ],
                ],
            ],
        ];
        return $meta;
    }
}
```

```xml
<!-- di.xml -->
<virtualType name="Magento\Catalog\Ui\DataProvider\Product\Form\Modifier\Pool">
    <arguments>
        <argument name="modifiers" xsi:type="array">
            <item name="vendor_custom" xsi:type="array">
                <item name="class" xsi:type="string">
                    Vendor\Module\Ui\DataProvider\Product\Form\Modifier\CustomModifier
                </item>
                <item name="sortOrder" xsi:type="number">100</item>
            </item>
        </argument>
    </arguments>
</virtualType>
```

---

## 6. Validator chain — ValidatorInterface, CompositeValidator

```php
<?php
declare(strict_types=1);

namespace Vendor\Module\Model\Validator;

use Magento\Framework\Validation\ValidationResult;
use Magento\Framework\Validation\ValidationResultFactory;

interface ValidatorInterface
{
    public function validate(array $data): ValidationResult;
}

class CompositeValidator implements ValidatorInterface
{
    /**
     * @param ValidatorInterface[] $validators
     */
    public function __construct(
        private readonly ValidationResultFactory $validationResultFactory,
        private readonly array $validators = []
    ) {}

    public function validate(array $data): ValidationResult
    {
        $errors = [];

        foreach ($this->validators as $validator) {
            $result = $validator->validate($data);
            if (!$result->isValid()) {
                $errors = array_merge($errors, $result->getErrors());
            }
        }

        return $this->validationResultFactory->create(['errors' => $errors]);
    }
}
```

```xml
<type name="Vendor\Module\Model\Validator\CompositeValidator">
    <arguments>
        <argument name="validators" xsi:type="array">
            <item name="required_fields" xsi:type="object">
                Vendor\Module\Model\Validator\RequiredFieldsValidator
            </item>
            <item name="email_format" xsi:type="object">
                Vendor\Module\Model\Validator\EmailFormatValidator
            </item>
        </argument>
    </arguments>
</type>
```

---

## 7. Builder pattern — SearchCriteriaBuilder, FilterBuilder deep dive

```php
<?php
declare(strict_types=1);

namespace Vendor\Module\Service;

use Magento\Framework\Api\SearchCriteriaBuilder;
use Magento\Framework\Api\FilterBuilder;
use Magento\Framework\Api\SortOrderBuilder;

class ProductSearchService
{
    public function __construct(
        private readonly SearchCriteriaBuilder $searchCriteriaBuilder,
        private readonly FilterBuilder $filterBuilder,
        private readonly SortOrderBuilder $sortOrderBuilder
    ) {}

    public function searchActiveProducts(
        string $nameQuery,
        float $minPrice,
        int $pageSize = 20,
        int $currentPage = 1
    ): \Magento\Framework\Api\SearchCriteriaInterface {
        // Reset builder state (quan trọng khi dùng lại builder)
        $this->searchCriteriaBuilder->create(); // reset

        // Filter: status = 1 (active)
        $statusFilter = $this->filterBuilder
            ->setField('status')
            ->setValue(1)
            ->setConditionType('eq')
            ->create();

        // Filter: name LIKE %query%
        $nameFilter = $this->filterBuilder
            ->setField('name')
            ->setValue('%' . $nameQuery . '%')
            ->setConditionType('like')
            ->create();

        // Filter: price >= minPrice
        $priceFilter = $this->filterBuilder
            ->setField('price')
            ->setValue($minPrice)
            ->setConditionType('gteq')
            ->create();

        // Sort: price ASC
        $sortOrder = $this->sortOrderBuilder
            ->setField('price')
            ->setDirection('ASC')
            ->create();

        return $this->searchCriteriaBuilder
            ->addFilters([$statusFilter])   // Group 1: status=1
            ->addFilters([$nameFilter])     // Group 2: AND name LIKE
            ->addFilters([$priceFilter])    // Group 3: AND price >=
            ->addSortOrder($sortOrder)
            ->setPageSize($pageSize)
            ->setCurrentPage($currentPage)
            ->create();
    }
}
```

---

## 8. Null Object pattern — tránh null check

```php
<?php
declare(strict_types=1);

namespace Vendor\Module\Model;

use Vendor\Module\Api\DiscountProviderInterface;

/**
 * Null Object — trả về khi không có discount provider nào phù hợp
 */
class NullDiscountProvider implements DiscountProviderInterface
{
    public function getDiscount(float $price): float
    {
        return 0.0; // Không có discount
    }

    public function isApplicable(string $customerGroup): bool
    {
        return false;
    }
}
```

```php
// Thay vì:
$provider = $this->getProvider($type);
if ($provider !== null) {
    $discount = $provider->getDiscount($price);
} else {
    $discount = 0.0;
}

// Dùng Null Object:
$provider = $this->getProvider($type) ?? $this->nullDiscountProvider;
$discount = $provider->getDiscount($price);
```

---

## 9. Registry pattern — legacy và cách thay thế

`Magento\Framework\Registry` là pattern cũ (global state), đang bị deprecated.

```php
// ❌ Legacy: dùng Registry (deprecated)
$this->registry->register('current_product', $product);
$product = $this->registry->registry('current_product');

// ✅ Thay thế: inject trực tiếp hoặc dùng context
// Trong Controller:
$this->_coreRegistry->register('current_product', $product);

// Trong Block: inject ProductRepositoryInterface và load theo ID từ request
public function getProduct(): ProductInterface
{
    $productId = (int)$this->getRequest()->getParam('id');
    return $this->productRepository->getById($productId);
}
```

---

## 10. PHP 8.x features trong Magento context

### Named arguments (PHP 8.0+)

```php
// Hữu ích khi gọi function với nhiều optional params
$result = array_slice(
    array: $items,
    offset: 0,
    length: 10,
    preserve_keys: true
);

// Trong Magento: dùng khi gọi factory với nhiều params
$model = $this->modelFactory->create(data: ['name' => 'Test']);
```

### Match expression (PHP 8.0+)

```php
// Thay thế switch statement
$label = match($status) {
    'pending' => __('Pending'),
    'processing' => __('Processing'),
    'complete' => __('Complete'),
    'canceled' => __('Canceled'),
    default => __('Unknown'),
};
```

### Nullsafe operator (PHP 8.0+)

```php
// Thay vì:
$city = $customer->getDefaultShippingAddress() !== null
    ? $customer->getDefaultShippingAddress()->getCity()
    : null;

// Dùng:
$city = $customer->getDefaultShippingAddress()?->getCity();
```

### Readonly properties (PHP 8.1+)

```php
// Dùng trong DTO/Value Object
class ProductData
{
    public function __construct(
        public readonly int $id,
        public readonly string $sku,
        public readonly float $price,
    ) {}
}

// Không thể thay đổi sau khi khởi tạo
$data = new ProductData(id: 1, sku: 'TEST-001', price: 99.99);
// $data->price = 50.0; // Error: Cannot modify readonly property
```

### Enum (PHP 8.1+)

```php
// Thay thế class constants
enum OrderStatus: string
{
    case Pending = 'pending';
    case Processing = 'processing';
    case Complete = 'complete';
    case Canceled = 'canceled';

    public function getLabel(): string
    {
        return match($this) {
            self::Pending => 'Pending',
            self::Processing => 'Processing',
            self::Complete => 'Complete',
            self::Canceled => 'Canceled',
        };
    }
}

// Sử dụng
$status = OrderStatus::Pending;
$label = $status->getLabel();
$value = $status->value; // 'pending'
$status = OrderStatus::from('processing'); // OrderStatus::Processing
```

> **Lưu ý Magento:** Enum chưa được dùng rộng rãi trong Magento core (vẫn dùng class constants). Có thể dùng trong custom module nhưng cần đảm bảo PHP 8.1+ requirement.

### First-class callable (PHP 8.1+)

```php
// Thay vì:
$names = array_map(function ($item) { return $item->getName(); }, $items);

// Dùng:
$names = array_map($item->getName(...), $items);

// Hoặc với static method:
$names = array_map(Formatter::format(...), $items);
```

### Generator — lazy collection, memory-efficient

```php
// Xử lý large dataset mà không load hết vào memory
function getProductsInBatches(int $batchSize = 1000): \Generator
{
    $currentPage = 1;
    do {
        $collection = $this->collectionFactory->create();
        $collection->setPageSize($batchSize)->setCurPage($currentPage);
        $collection->load();

        foreach ($collection as $product) {
            yield $product;
        }

        $lastPage = $collection->getLastPageNumber();
        $currentPage++;
        $collection->clear();
    } while ($currentPage <= $lastPage);
}

// Sử dụng
foreach ($this->getProductsInBatches() as $product) {
    $this->processProduct($product);
    // Chỉ 1 batch trong memory tại một thời điểm
}
```

---

## Liên kết

- Plugin patterns: xem [plugin-patterns.md](./plugin-patterns.md)
- DI & Generated code: xem [di-codegen.md](./di-codegen.md)
- Service Contracts: xem [service-contracts.md](./service-contracts.md)
