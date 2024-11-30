


**Command Query Responsibility Segregation (CQRS)** — это архитектурный подход, при котором операции чтения и записи в приложении разделяются, создавая две отдельные модели данных и соответствующие интерфейсы. Это помогает повысить производительность, масштабируемость и гибкость системы, особенно в сложных проектах.

Идея CQRS (Command Query Responsibility Segregation) заключается именно в **разделении ответственности** между двумя типами операций: **изменение данных (команды)** и **чтение данных (запросы)**. Это разделение помогает сделать систему более понятной, эффективной и гибкой.


### Как это работает:

1. **Команды (Commands):**
    
    - Используются только для изменения данных.
    - Пример: добавление нового заказа, обновление профиля, удаление записи.
    - **Ключевой момент:** команды никогда не возвращают данные. Они просто выполняют действие.



**Запросы (Queries):**
   
   - Используются только для чтения данных.
   - Пример: получение списка заказов, деталей пользователя, статистики.
   - **Ключевой момент:** запросы никогда не изменяют данные. Они просто возвращают результат.



В подходе **CQRS** (Command Query Responsibility Segregation) разделение кода на операции чтения (Query) и записи (Command) может происходить на различных уровнях системы. Это разделение может быть как **логическим**, так и **физическим**, в зависимости от того, насколько глубоко и на каких уровнях архитектуры производится разделение.


### 1. **Логическое разделение** (Logical Segregation):

Это разделение на уровне **кода**, где команды и запросы обрабатываются разными компонентами внутри одного приложения.

- **Команды** (Commands): Это операции, которые изменяют состояние системы. Например, создание заказа или изменение данных о пользователе.
- **Запросы** (Queries): Это операции, которые только читают данные без их изменения. Например, извлечение списка заказов или статистики.

Логическое разделение имеет свои преимущества:

- Упрощает понимание структуры кода.
- Чёткое разделение ответственности.
- Облегчает тестирование и поддержку.

---
### Логическое разделение
### Работа с сущностями в Symfony при использовании CQRS или других паттернов

Если ты планируешь внедрить **CQRS** или похожий паттерн, организация кода в Symfony может потребовать дополнительной структуры.

1. **Создание отдельных сервисов**:
    
    - **Команды (Commands)**: Создаются сервисы, которые отвечают за выполнение операций записи.
        - Например: `OrderService::createOrder()`.
    - **Запросы (Queries)**: Выделяются сервисы или репозитории, которые занимаются чтением данных.
        - Например: `OrderQueryService::findOrdersByCustomerId()`.
    
    **Структура папок может выглядеть так:**
    
    ```sql
    src/
    ├── Command/
    │   ├── CreateOrderCommand.php
    │   ├── CreateOrderHandler.php
    └── Query/
        ├── GetOrdersQuery.php
        ├── GetOrdersHandler.php
    ```
    
2. **Разделение сущностей (опционально):**
    
    - Если чтение и запись данных требуют разных моделей, можешь разделить сущности:
        - **Write Entities**: Полные сущности с логикой и связями для записи.
        - **Read Models**: Упрощённые DTO или специальные классы для чтения.
    
    **Пример структуры:**
    
    ```sql
    src/
    ├── Entity/
    │   ├── Order.php (Write Model)
    │   └── OrderReadModel.php (Read Model)
    ```
    
3. **Репозитории и сервисы:**
    
    - Для работы с базой данных можно выделить отдельные репозитории.
        - Репозиторий для записи: `OrderRepository`.
        - Сервис для чтения: `OrderQueryService`.
    
    **Пример:**
    
    ```php
    // Репозиторий записи
    class OrderRepository {
        public function save(Order $order) {
            // Логика сохранения
        }
    }
    
    // Сервис чтения
    class OrderQueryService {
        public function findOrdersByCustomerId(int $customerId): array {
            // Чтение данных
            return $this->entityManager->createQuery(
                'SELECT o FROM App\Entity\OrderReadModel o WHERE o.customerId = :id'
            )->setParameter('id', $customerId)->getResult();
        }
    }
    ```
    
4. **Разделение слоёв**:
    
    - Используй **сервисы Symfony** для бизнес-логики.
    - Компонент `Messenger` можно использовать для обработки команд и запросов.
    
    **Пример использования Symfony Messenger:**
    
    ```php
    // Команда
    class CreateOrderCommand {
        public function __construct(public int $customerId, public array $items) {}
    }
    
    // Обработчик
    class CreateOrderHandler {
        private OrderRepository $repository;
    
        public function __construct(OrderRepository $repository) {
            $this->repository = $repository;
        }
    
        public function __invoke(CreateOrderCommand $command): void {
            $order = new Order($command->customerId, $command->items);
            $this->repository->save($order);
        }
    }
    ```
    
5. **Инфраструктура и инструменты:**
    
    - Используй Doctrine для работы с сущностями, но можешь адаптировать запросы под свои нужды.
    - Например, для моделей чтения можно использовать SQL-запросы без сложной ORM-логики.

---


### 2. **Физическое разделение** (Physical Segregation):

В этом случае разделение происходит не только на уровне логики, но и на уровне инфраструктуры. Это означает, что команды и запросы могут использовать разные базы данных, сервисы или даже технологии для выполнения своих операций.

- **Модели записи** (Write Model): Используется для выполнения операций изменения состояния, может быть более сложной и нормализованной.
- **Модели чтения** (Read Model): Оптимизированы для быстрого извлечения данных, могут быть денормализованы, индексированы или кэшированы для ускорения работы.

Это физическое разделение подразумевает использование **разных баз данных или серверов** для чтения и записи:

- Операции чтения и записи могут использовать **разные источники данных**, что позволяет оптимизировать производительность.
- В некоторых случаях можно использовать **параллельные базы данных** (например, SQL для записи и NoSQL для чтения) или разные схемы данных для чтения и записи.


### Пример физического разделения в CQRS

Физическое разделение в CQRS предполагает, что операции **чтения** и **записи** используют разные базы данных, сервисы или даже технологии. Это позволяет оптимизировать каждую из этих операций по отдельности, улучшая производительность и масштабируемость.

В качестве примера давайте рассмотрим систему интернет-магазина, где для записи используется реляционная база данных (например, **PostgreSQL**), а для чтения — **NoSQL база данных** (например, **Elasticsearch**).

---

### 1. **Модели для записи и чтения**

- **Write Model** (модель для записи): Сложная, нормализованная модель данных, которая используется для изменения состояния системы (например, создание заказа).
- **Read Model** (модель для чтения): Оптимизированная модель, которая может быть денормализована или индексирована для быстрого извлечения данных (например, поисковый индекс для заказов).

### 2. **Репозитории и сервисы для записи и чтения**

- **OrderWriteRepository**: Репозиторий для работы с моделью записи.
- **OrderReadRepository**: Репозиторий для работы с моделью чтения.

---

### 3. **Пример кода**

#### **Модели**

- **OrderWriteModel** (для записи)

```php
namespace App\Entity;

class Order
{
    private int $id;
    private int $customerId;
    private array $items;
    private \DateTime $createdAt;

    public function __construct(int $customerId, array $items)
    {
        $this->customerId = $customerId;
        $this->items = $items;
        $this->createdAt = new \DateTime();
    }

    // Геттеры и сеттеры
}
```

- **OrderReadModel** (для чтения)

```php
namespace App\Entity;

class OrderReadModel
{
    private int $orderId;
    private int $customerId;
    private array $items;
    private string $status;

    public function __construct(int $orderId, int $customerId, array $items, string $status)
    {
        $this->orderId = $orderId;
        $this->customerId = $customerId;
        $this->items = $items;
        $this->status = $status;
    }

    // Геттеры и сеттеры
}
```

#### **Репозитории**

- **OrderWriteRepository**: Репозиторий для записи

```php
namespace App\Repository;

use App\Entity\Order;

class OrderWriteRepository
{
    private $entityManager;

    public function __construct($entityManager)
    {
        $this->entityManager = $entityManager;
    }

    public function save(Order $order)
    {
        $this->entityManager->persist($order);
        $this->entityManager->flush();
    }
}
```

- **OrderReadRepository**: Репозиторий для чтения

```php
namespace App\Repository;

use App\Entity\OrderReadModel;
use Doctrine\ORM\EntityManagerInterface;

class OrderReadRepository
{
    private $elasticsearchClient;

    public function __construct($elasticsearchClient)
    {
        $this->elasticsearchClient = $elasticsearchClient;
    }

    public function findByCustomerId(int $customerId)
    {
        // Пример запроса к Elasticsearch
        $query = [
            'index' => 'orders',
            'body' => [
                'query' => [
                    'match' => [
                        'customerId' => $customerId
                    ]
                ]
            ]
        ];

        $response = $this->elasticsearchClient->search($query);
        return $response['hits']['hits'];
    }
}
```

#### **Команды и обработчики**

- **CreateOrderCommand**: Команда для создания заказа

```php
namespace App\Command;

class CreateOrderCommand
{
    public int $customerId;
    public array $items;

    public function __construct(int $customerId, array $items)
    {
        $this->customerId = $customerId;
        $this->items = $items;
    }
}
```

- **CreateOrderHandler**: Обработчик команды

```php
namespace App\Handler;

use App\Repository\OrderWriteRepository;
use App\Command\CreateOrderCommand;
use App\Entity\Order;

class CreateOrderHandler
{
    private $orderWriteRepository;

    public function __construct(OrderWriteRepository $orderWriteRepository)
    {
        $this->orderWriteRepository = $orderWriteRepository;
    }

    public function handle(CreateOrderCommand $command)
    {
        $order = new Order($command->customerId, $command->items);
        $this->orderWriteRepository->save($order);
    }
}
```

#### **Сервисы для запросов**

- **OrderQueryService**: Сервис для чтения данных

```php
namespace App\Service;

use App\Repository\OrderReadRepository;

class OrderQueryService
{
    private $orderReadRepository;

    public function __construct(OrderReadRepository $orderReadRepository)
    {
        $this->orderReadRepository = $orderReadRepository;
    }

    public function getOrdersByCustomerId(int $customerId)
    {
        return $this->orderReadRepository->findByCustomerId($customerId);
    }
}
```

---

### 4. **Инфраструктура**

В этом примере используется **Symfony** с **Doctrine** для работы с базой данных для записи и **Elasticsearch** для чтения.

- **Запись**: Для операций записи используется реляционная база данных, например, PostgreSQL через Doctrine ORM.
- **Чтение**: Для операций чтения используется Elasticsearch, что позволяет выполнять быстрые поисковые запросы и извлекать данные с минимальной задержкой.

---

### 5. **Использование**

Когда пользователь добавляет заказ:

- **Создаётся команда `CreateOrderCommand`**, которая обрабатывается через **обработчик `CreateOrderHandler`**, сохраняющий данные в базу данных.

Когда пользователь хочет получить список своих заказов:

- **Запрос `getOrdersByCustomerId`** используется для извлечения данных через **OrderQueryService**, который делает запрос в **Elasticsearch**.

---

### Преимущества физического разделения:

1. **Оптимизация производительности**:
    
    - Для записи используется нормализованная база данных, а для чтения — быстрый поисковый движок.
2. **Масштабируемость**:
    
    - Чтение и запись могут масштабироваться независимо. Если количество чтений значительно превышает количество записей, можно масштабировать только репликацию **Elasticsearch**.
3. **Гибкость**:
    
    - Можно использовать разные хранилища для разных типов данных, что даёт большую гибкость в обслуживании запросов и команд.
4. **Упрощение логики**:
    
    - Каждое хранилище данных оптимизировано под свой тип операций (чтение или запись), что снижает сложности и улучшает стабильность.

---

Это пример реализации **физического разделения** с использованием **CQRS**, где операции записи и чтения полностью разделены как на уровне логики, так и на уровне инфраструктуры.


### 3. **Разделение на уровне слоёв (Тiers)**:

В более сложных системах CQRS может включать разделение на **физические слои**, такие как слои сервисов или микросервисы. Это подход особенно актуален для крупных распределённых систем и микросервисной архитектуры.

- **Сервис для команд** (Command Service): Может быть отдельным микросервисом, обрабатывающим только команды и изменяющим состояние системы.
- **Сервис для запросов** (Query Service): Отдельный микросервис или сервис, отвечающий только за чтение данных и предоставляющий данные для пользователей.

Разделение на уровне слоёв может включать:

- **Команды**: Отправляются в сервисы, которые занимаются изменением данных, иногда через очередь сообщений или событий.
- **Запросы**: Обрабатываются другим сервисом, который может быть оптимизирован для быстрого чтения данных.

### Пример

Мы создадим два сервиса: один для обработки команд (изменения данных), а второй для обработки запросов (чтение данных).

Предположим, у нас есть приложение для работы с заказами. У нас будет два микросервиса:

1. **Order Command Service** — отвечает за создание и изменение заказов.
2. **Order Query Service** — отвечает за чтение данных о заказах.

---

### Архитектура папок

```
src/
├── Command/
│   ├── CreateOrder/
│   │   ├── CreateOrderCommand.php
│   │   ├── CreateOrderHandler.php
│   └── UpdateOrder/
│       ├── UpdateOrderCommand.php
│       ├── UpdateOrderHandler.php
├── Query/
│   ├── GetOrder/
│   │   ├── GetOrderQuery.php
│   │   ├── GetOrderHandler.php
│   └── GetOrdersByCustomer/
│       ├── GetOrdersByCustomerQuery.php
│       ├── GetOrdersByCustomerHandler.php
├── Entity/
│   ├── Order.php
│   └── OrderReadModel.php
├── Repository/
│   ├── OrderRepository.php
│   └── OrderReadModelRepository.php
└── Service/
    ├── OrderCommandService.php
    └── OrderQueryService.php
```

### 1. **Сервис для команд** (Order Command Service)

Сервис команд будет заниматься изменением состояния системы. Он будет работать с моделями записи и будет взаимодействовать с репозиториями для сохранения данных.

#### Пример кода для Command Service:

```php
// src/Command/CreateOrder/CreateOrderCommand.php
class CreateOrderCommand
{
    public function __construct(
        public int $customerId, 
        public array $items, 
        public string $status
    ) {}
}
```

```php
// src/Command/CreateOrder/CreateOrderHandler.php
class CreateOrderHandler
{
    private OrderRepository $orderRepository;

    public function __construct(OrderRepository $orderRepository)
    {
        $this->orderRepository = $orderRepository;
    }

    public function handle(CreateOrderCommand $command): void
    {
        $order = new Order($command->customerId, $command->items, $command->status);
        $this->orderRepository->save($order);
    }
}
```

```php
// src/Repository/OrderRepository.php
class OrderRepository
{
    private EntityManagerInterface $entityManager;

    public function __construct(EntityManagerInterface $entityManager)
    {
        $this->entityManager = $entityManager;
    }

    public function save(Order $order): void
    {
        $this->entityManager->persist($order);
        $this->entityManager->flush();
    }
}
```

```php
// src/Service/OrderCommandService.php
class OrderCommandService
{
    private CreateOrderHandler $createOrderHandler;

    public function __construct(CreateOrderHandler $createOrderHandler)
    {
        $this->createOrderHandler = $createOrderHandler;
    }

    public function createOrder(int $customerId, array $items, string $status): void
    {
        $command = new CreateOrderCommand($customerId, $items, $status);
        $this->createOrderHandler->handle($command);
    }
}
```

### 2. **Сервис для запросов** (Order Query Service)

Сервис запросов будет отвечать за извлечение данных, например, получение списка заказов по идентификатору пользователя.

#### Пример кода для Query Service:

```php
// src/Query/GetOrder/GetOrderQuery.php
class GetOrderQuery
{
    public function __construct(public int $orderId) {}
}
```

```php
// src/Query/GetOrder/GetOrderHandler.php
class GetOrderHandler
{
    private OrderReadModelRepository $orderReadModelRepository;

    public function __construct(OrderReadModelRepository $orderReadModelRepository)
    {
        $this->orderReadModelRepository = $orderReadModelRepository;
    }

    public function handle(GetOrderQuery $query): ?OrderReadModel
    {
        return $this->orderReadModelRepository->findByOrderId($query->orderId);
    }
}
```

```php
// src/Repository/OrderReadModelRepository.php
class OrderReadModelRepository
{
    private EntityManagerInterface $entityManager;

    public function __construct(EntityManagerInterface $entityManager)
    {
        $this->entityManager = $entityManager;
    }

    public function findByOrderId(int $orderId): ?OrderReadModel
    {
        return $this->entityManager->getRepository(OrderReadModel::class)->find($orderId);
    }

    public function findAll(): array
    {
        return $this->entityManager->getRepository(OrderReadModel::class)->findAll();
    }
}
```

```php
// src/Service/OrderQueryService.php
class OrderQueryService
{
    private GetOrderHandler $getOrderHandler;

    public function __construct(GetOrderHandler $getOrderHandler)
    {
        $this->getOrderHandler = $getOrderHandler;
    }

    public function getOrder(int $orderId): ?OrderReadModel
    {
        $query = new GetOrderQuery($orderId);
        return $this->getOrderHandler->handle($query);
    }

    public function getAllOrders(): array
    {
        // Предполагается, что этот запрос может быть оптимизирован
        return $this->getOrderHandler->handle(new GetOrdersQuery());
    }
}
```

### 3. **Модели данных** (Write & Read Models)

- **Write Model** — класс, который будет содержать полные данные для записи.
- **Read Model** — оптимизированная модель для чтения данных, которая может быть денормализованной или использовать другие структуры данных для ускорения запросов.

```php
// src/Entity/Order.php
class Order
{
    private int $orderId;
    private int $customerId;
    private array $items;
    private string $status;

    public function __construct(int $customerId, array $items, string $status)
    {
        $this->customerId = $customerId;
        $this->items = $items;
        $this->status = $status;
    }

    // Геттеры и сеттеры
}
```

```php
// src/Entity/OrderReadModel.php
class OrderReadModel
{
    private int $orderId;
    private int $customerId;
    private array $items;
    private string $status;

    public function __construct(int $orderId, int $customerId, array $items, string $status)
    {
        $this->orderId = $orderId;
        $this->customerId = $customerId;
        $this->items = $items;
        $this->status = $status;
    }

    // Геттеры и сеттеры
}
```

### Архитектура и принцип работы

1. **Command Service**: обрабатывает команды (например, создание заказов) и изменяет данные в базе данных.
2. **Query Service**: обрабатывает запросы (например, получение заказов) и предоставляет данные для пользователя.
3. **Модели данных**: разделение на **Write Model** и **Read Model** позволяет оптимизировать хранение данных для разных операций.
4. **Репозитории**: разные репозитории для работы с записью и чтением данных.

В дальнейшем, если система будет масштабироваться, можно будет разделить сервисы на микросервисы, каждый из которых будет отвечать за свою область — командный микросервис и запросный микросервис, возможно, с использованием разных баз данных (например, реляционной базы для записи и NoSQL базы для чтения).

---
