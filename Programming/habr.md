Вот **максимально полный отчет** по архитектуре сервисов в Laravel, основанный на всех наших обсуждениях, с добавлением важного раздела о статических методах.

---

# Полный отчет: Архитектура сервисов в Laravel

## Содержание

1. [Введение: зачем нужны сервисы](#введение-зачем-нужны-сервисы)
2. [Почему не статические методы?](#почему-не-статические-методы)
3. [Основная архитектура: фасад + специализированные сервисы](#основная-архитектура-фасад--специализированные-сервисы)
4. [Детальное описание каждого уровня](#детальное-описание-каждого-уровня)
5. [Полная схема обработки запроса (включая Exception Handler)](#полная-схема-обработки-запроса-включая-exception-handler)
6. [Два типа методов в фасаде: прокси и оркестраторы](#два-типа-методов-в-фасаде-прокси-и-оркестраторы)
7. [Пример полной реализации для Order](#пример-полной-реализации-для-order)
8. [Когда один запрос использует несколько сервисов](#когда-один-запрос-использует-несколько-сервисов)
9. [Итоговые принципы и чек-лист](#итоговые-принципы-и-чек-лист)

---

## Введение: зачем нужны сервисы

В Laravel контроллеры должны быть **тонкими**. Их задача — принять запрос, вызвать бизнес-логику и вернуть ответ. Сама бизнес-логика живёт в **сервисах**.

**Проблема, которую решает сервис:** контроллеры не должны знать, как работает создание заказа, расчёт скидки или проверка остатков. Это знание инкапсулируется в сервисы.

**Плохо (вся логика в контроллере):**
```php
class OrderController extends Controller
{
    public function store(Request $request)
    {
        // 50 строк бизнес-логики: проверки, расчёты, транзакции...
        // Невозможно переиспользовать
        // Невозможно протестировать отдельно
    }
}
```

**Хорошо (логика в сервисе):**
```php
class OrderController extends Controller
{
    public function store(Request $request, OrderService $orderService)
    {
        $order = $orderService->createFromData($request->toDto());
        return new OrderResource($order);
    }
}
```

---

## Почему не статические методы?

Статические методы — это антипаттерн для бизнес-логики в Laravel. Вот почему.

### Проблема 1. Жёсткая связанность

```php
// Статический подход
class OrderService
{
    public static function create(array $data): Order
    {
        // напрямую вызывает другой статический метод
        PriceCalculator::calculate($data['items']);
    }
}

// В контроллере
OrderService::create($data);
```

**Проблема:** вы не можете подменить `PriceCalculator` на другой (например, для теста). Классы жёстко связаны через имена.

**Решение через DI (не статический):**
```php
class OrderService
{
    public function __construct(
        private PriceCalculator $calculator  // можно заменить моком
    ) {}
    
    public function create(array $data): Order
    {
        $this->calculator->calculate($data['items']);
    }
}
```

### Проблема 2. Невозможно тестировать изолированно

```php
// Статический метод
OrderService::create($data);  // реально вызывает БД, API, всё по-настоящему

// С DI
$mockCalculator = Mockery::mock(PriceCalculator::class);
$service = new OrderService($mockCalculator);  // можно тестировать только логику OrderService
```

### Проблема 3. Состояние и глобальные зависимости

Статические методы не могут иметь своё состояние через конструктор. Всё приходится передавать параметрами или использовать глобальные фасады Laravel:

```php
// Статический подход (кошмар)
public static function create(array $data): Order
{
    // Куда делся пользователь? Достаём из глобального состояния
    $user = auth()->user();  // скрытая зависимость
    
    // Логирование? Тоже глобально
    Log::info('Creating order');
    
    // Кэш? Опять глобально
    Cache::forget('user_orders_' . $user->id);
}
```

**С DI всё явно:**
```php
public function __construct(
    private AuthManager $auth,
    private LoggerInterface $logger,
    private CacheRepository $cache,
) {}
```

### Проблема 4. Нарушение SOLID (принцип инверсии зависимостей)

Высокоуровневый модуль (`OrderController`) зависит от конкретной реализации (`OrderService::create`), а не от абстракции.

### Единственное место, где статические методы оправданы

**Хелперы для чистых функций** (без состояния, без внешних вызовов):

```php
class PriceFormatter
{
    public static function format(float $price): string
    {
        return number_format($price, 2, ',', ' ') . ' ₽';
    }
}

// Использование
$formatted = PriceFormatter::format($order->total);
```

Здесь нет зависимостей, нет БД, нет внешних API — статика допустима.

### Итог по статическим методам

| Где использовать | Где НЕ использовать |
|-----------------|---------------------|
| Хелперы (форматирование, простые расчёты) | Бизнес-логика, работа с БД, вызов API |
| Валидаторы (чистые функции) | Сервисы с зависимостями |
| Конвертеры, трансформеры | Любой код, который нужно тестировать с моками |

**Золотое правило:** если метод вызывает другой сервис, обращается к БД, использует кэш или лог — он **не должен быть статическим**.

---

## Основная архитектура: фасад + специализированные сервисы

После долгих обсуждений мы пришли к архитектуре, которая:

- Не заставляет прыгать по методам внутри одного файла.
- Не плодит сотни мелких классов (action-классы).
- Позволяет контроллеру работать с одним сервисом.
- Легко масштабируется.

### Структура

```
app/Services/
├── OrderService.php                      # ФАСАД (единая точка входа для контроллеров)
└── Order/                                # Специализированные сервисы
    ├── OrderCreationService.php          # Только создание
    ├── OrderCancellationService.php      # Только отмена
    └── OrderStatusService.php            # Только управление статусами
```

### Роли

| Класс | Роль | Содержит |
|-------|------|----------|
| **`OrderService`** | Фасад | Методы-прокси (1 строка) и методы-оркестраторы (координация нескольких сервисов) |
| **`OrderCreationService`** | Специализированный | Всю логику создания заказа (проверки, расчёты, транзакции) |
| **`OrderCancellationService`** | Специализированный | Всю логику отмены (возврат товаров, обновление статуса) |
| **`OrderStatusService`** | Специализированный | Всю логику изменения статусов (с проверкой допустимых переходов) |

---

## Детальное описание каждого уровня

### 1. Специализированный сервис (например, `OrderCreationService`)

Содержит **реальную бизнес-логику**. Приватные методы внутри него допустимы, так как они относятся только к этой операции.

```php
class OrderCreationService
{
    public function create(CreateOrderDto $dto): Order
    {
        $this->checkStock($dto->items);           // приватный метод
        $total = $this->calculateTotal($dto->items); // приватный метод
        
        return DB::transaction(function () use ($dto, $total) {
            $order = Order::create([...]);
            // создание позиций, списание остатков
            return $order;
        });
    }
    
    private function checkStock(array $items): void { ... }
    private function calculateTotal(array $items): float { ... }
}
```

### 2. Фасад (`OrderService`)

Содержит **публичные методы** для контроллеров. Бывает двух типов.

**Тип А: Методы-прокси (одна строка)**
```php
public function createFromData(CreateOrderDto $dto): Order
{
    return $this->orderCreator->create($dto);
}
```

**Тип Б: Методы-оркестраторы (координация нескольких сервисов)**
```php
public function createAndPay(CreateOrderDto $dto, array $paymentData): Order
{
    $order = $this->orderCreator->create($dto);
    $this->payment->charge($order, $paymentData);
    $this->orderStatus->markAsPaid($order);
    $this->notifier->send($order->user, new OrderCreatedNotification($order));
    return $order;
}
```

### 3. Контроллер

Всегда работает **только с фасадом** (`OrderService`), не знает о специализированных сервисах.

```php
class OrderController extends Controller
{
    public function store(StoreOrderRequest $request, OrderService $orderService)
    {
        $order = $orderService->createFromData($request->toDto());
        return new OrderResource($order);
    }
    
    public function quickOrder(QuickOrderRequest $request, OrderService $orderService)
    {
        $order = $orderService->createAndPay($request->toDto(), $request->paymentData);
        return new OrderResource($order);
    }
}
```

---

## Полная схема обработки запроса (включая Exception Handler)

```
HTTP запрос
    ↓
[1] Маршрут + Middleware (аутентификация, throttle, CORS)
    ↓
[2] FormRequest (валидация полей, авторизация действия)
    ↓
[3] Контроллер (получает проверенные данные, вызывает фасад)
    ↓
[4] OrderService (фасад) → делегирует специализированному сервису
    ↓
[5] Специализированный сервис (бизнес-логика, транзакции)
    ↓
    Если исключение → Exception Handler → JSON с ошибкой (422, 400, 401...)
    Если успех ↓
[6] Ресурс (форматирование ответа)
    ↓
HTTP ответ (200/201)
```

### Роль Exception Handler

Handler перехватывает исключения с **любого уровня** (от middleware до сервиса) и превращает их в HTTP-ответ:

```php
// app/Exceptions/Handler.php
public function render($request, Throwable $e)
{
    if ($request->expectsJson()) {
        if ($e instanceof ValidationException) {
            return response()->json(['errors' => $e->errors()], 422);
        }
        if ($e instanceof OutOfStockException) {
            return response()->json(['error' => $e->getMessage()], 422);
        }
        if ($e instanceof AuthenticationException) {
            return response()->json(['error' => 'Unauthenticated'], 401);
        }
    }
    return parent::render($request, $e);
}
```

**Преимущества:** контроллеры и сервисы не содержат try-catch, просто выбрасывают исключения.

---

## Два типа методов в фасаде: прокси и оркестраторы

### Прокси (одна строка)

Используется, когда операция полностью делегируется одному специализированному сервису.

```php
public function cancel(Order $order): void
{
    $this->orderCanceller->cancel($order);
}
```

**Когда использовать:** операция относится к одной узкой области, нет необходимости координировать другие сервисы.

### Оркестратор (несколько строк)

Используется, когда бизнес-сценарий требует координации **двух и более** специализированных сервисов.

```php
public function cancelWithRefundAndNotify(Order $order, string $reason): void
{
    $this->orderCanceller->cancelWithRefund($order, $reason);
    $this->notifier->send($order->user, new OrderCancelledNotification($order));
}
```

**Когда использовать:** контроллеру нужен один вызов, но внутри нужно выполнить несколько действий (например, отменить + вернуть деньги + уведомить).

### Почему не выносить оркестрацию в отдельный класс?

Можно, но для тестового проекта это избыточно. Оркестрация внутри `OrderService` — это нормально, потому что:

- Контроллер всё равно работает с `OrderService`.
- Дополнительный класс `OrderCancelOrchestrator` — ещё одна абстракция.
- В больших проектах вынесение оркестраторов оправдано, но не на старте.

---

## Пример полной реализации для Order

### Структура файлов

```
app/
├── DTO/
│   └── CreateOrderDto.php
├── Services/
│   ├── OrderService.php
│   └── Order/
│       ├── OrderCreationService.php
│       ├── OrderCancellationService.php
│       └── OrderStatusService.php
├── Http/
│   ├── Controllers/
│   │   └── OrderController.php
│   ├── Requests/
│   │   └── StoreOrderRequest.php
│   └── Resources/
│       └── OrderResource.php
└── Exceptions/
    ├── OutOfStockException.php
    ├── OrderCannotBeCancelledException.php
    └── InvalidStatusTransitionException.php
```

### Код специализированного сервиса (пример)

```php
// app/Services/Order/OrderCreationService.php
namespace App\Services\Order;

use App\DTO\CreateOrderDto;
use App\Models\Order;
use App\Models\Product;
use App\Exceptions\OutOfStockException;
use Illuminate\Support\Facades\DB;

class OrderCreationService
{
    public function create(CreateOrderDto $dto): Order
    {
        $this->checkStock($dto->items);
        $total = $this->calculateTotal($dto->items);
        
        $order = DB::transaction(function () use ($dto, $total) {
            $order = Order::create([
                'user_id' => $dto->userId,
                'delivery_address' => $dto->deliveryAddress,
                'total' => $total,
                'status' => 'pending',
            ]);
            
            foreach ($dto->items as $item) {
                $order->items()->create($item);
                Product::where('id', $item['product_id'])->decrement('stock', $item['quantity']);
            }
            
            return $order;
        });
        
        return $order->load(['items.product', 'user']);
    }
    
    private function checkStock(array $items): void { /* ... */ }
    private function calculateTotal(array $items): float { /* ... */ }
}
```

### Код фасада

```php
// app/Services/OrderService.php
namespace App\Services;

use App\DTO\CreateOrderDto;
use App\Models\Order;
use App\Services\Order\OrderCreationService;
use App\Services\Order\OrderCancellationService;
use App\Services\Order\OrderStatusService;

class OrderService
{
    public function __construct(
        private OrderCreationService $orderCreator,
        private OrderCancellationService $orderCanceller,
        private OrderStatusService $orderStatus,
    ) {}
    
    // Прокси
    public function createFromData(CreateOrderDto $dto): Order
    {
        return $this->orderCreator->create($dto);
    }
    
    // Прокси
    public function cancel(Order $order, string $reason = null): void
    {
        $this->orderCanceller->cancel($order, $reason);
    }
    
    // Оркестратор (несколько сервисов)
    public function cancelWithRefundAndNotify(Order $order, string $reason, NotificationService $notifier): void
    {
        $this->orderCanceller->cancelWithRefund($order, $reason);
        $notifier->send($order->user, new OrderCancelledNotification($order));
    }
    
    // Прокси
    public function markAsPaid(Order $order, string $paymentId = null): void
    {
        $this->orderStatus->markAsPaid($order, $paymentId);
    }
}
```

---

## Когда один запрос использует несколько сервисов

В реальных приложениях контроллер может вызывать **несколько фасадов** или **фасад + другие сервисы**.

### Пример: создание заказа + начисление бонусов + отправка уведомления

```php
class OrderController extends Controller
{
    public function store(
        StoreOrderRequest $request,
        OrderService $orderService,
        LoyaltyService $loyalty,
        NotificationService $notifier,
    ) {
        $order = $orderService->createFromData($request->toDto());
        
        $loyalty->addPoints($order->user, $order->total);
        $notifier->send($order->user, new OrderCreatedNotification($order));
        
        return new OrderResource($order);
    }
}
```

### Как упростить? Добавить метод-оркестратор в `OrderService`

```php
// В OrderService
public function createWithBonusAndNotification(CreateOrderDto $dto, LoyaltyService $loyalty, NotificationService $notifier): Order
{
    $order = $this->orderCreator->create($dto);
    $loyalty->addPoints($order->user, $order->total);
    $notifier->send($order->user, new OrderCreatedNotification($order));
    return $order;
}
```

**Контроллер становится однострочным:**
```php
$order = $orderService->createWithBonusAndNotification($request->toDto(), $loyalty, $notifier);
```

---

## Итоговые принципы и чек-лист

### Принципы, которые мы выработали

1. **Статические методы — только для чистых хелперов** (без зависимостей, БД, API).
2. **Один фасад на сущность** (`OrderService`, `ProductService` и т.д.).
3. **Фасад скрывает сложность** — контроллер работает только с ним.
4. **Специализированные сервисы** содержат реальную бизнес-логику.
5. **В фасаде допустимы два типа методов:** прокси (1 строка) и оркестраторы (координация нескольких сервисов).
6. **Исключения** выбрасываются из сервисов и обрабатываются глобальным Handler.
7. **DTO** используются для передачи данных между слоями.
8. **Ресурсы** форматируют только успешные ответы.

### Чек-лист для создания нового сервиса

- [ ] Создан фасад в `app/Services/EntityService.php`
- [ ] Создана папка `app/Services/Entity/`
- [ ] В папке созданы специализированные сервисы (по одному на операцию)
- [ ] Фасад принимает специализированные сервисы через конструктор (DI)
- [ ] Методы фасада — либо прокси, либо оркестраторы
- [ ] Контроллер инжектит только фасад
- [ ] Бизнес-логика не дублируется между специализированными сервисами
- [ ] Исключения имеют свои классы и обрабатываются в Handler

### Когда расширять архитектуру?

| Симптом | Решение |
|---------|---------|
| `OrderService` стал содержать 15+ методов | Сгруппировать методы по смыслу, создать новый специализированный сервис |
| Появились сложные сценарии с 5+ сервисами | Создать отдельный оркестратор (`app/Orchestrators/`) |
| Проект вырос до 50+ моделей | Перейти на модульную архитектуру (`app/Modules/Order/`) |
| Нужно кешировать результаты | Добавить слой `OrderCachingService` с декоратором |

---

## Заключение

Эта архитектура — результат долгих обсуждений и поиска баланса между:

- **Чистотой кода** (SOLID, тестируемость)
- **Простотой понимания** (не нужно прыгать по 10 файлам)
- **Реалиями тестового задания** (не нужно переусложнять)

Она не идеальна для гигантского проекта с 100 разработчиками, но **идеальна для старта, роста и тестового задания**.

Главное, что мы выяснили: **статические методы в бизнес-логике — зло**, а фасад с делегированием специализированным сервисам — золотая середина между монолитом и атомарными action-классами.