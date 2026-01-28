Вы правы! Добавим недостающие элементы в **практичную Laravel-структуру**:

```
app/
├── Models/
│   └── User.php
├── Services/                     # Бизнес-логика
│   ├── UserService.php           # Основная логика пользователей
│   └── PaginationService.php     # Универсальная пагинация
├── Repositories/                 # Работа с данными
│   ├── UserRepository.php        # User-specific queries
│   └── BaseRepository.php        # Общая логика
├── Http/
│   ├── Controllers/
│   │   └── UserController.php
│   ├── Requests/
│   │   └── UserListRequest.php
│   └── Resources/
│       ├── UserResource.php
│       └── PaginationResource.php
├── Support/                      # Вспомогательные классы
│   ├── Pagination/
│   │   ├── Strategies/           # Стратегии пагинации
│   │   │   ├── LengthAwareStrategy.php
│   │   │   ├── SimpleStrategy.php
│   │   │   ├── CursorStrategy.php
│   │   │   └── JsonApiStrategy.php
│   │   └── PaginatorContext.php  # Контекст для стратегий
│   └── DTO/                      # Data Transfer Objects
│       ├── User/
│       │   └── UserListDTO.php
│       └── Pagination/
│           ├── PaginationRequestDTO.php
│           └── PaginationResultDTO.php
├── Traits/
│   └── Paginatable.php
└── Enums/
    └── PaginationType.php
```

## 1. Репозиторий с методом пагинации

### app/Repositories/UserRepository.php
```php
<?php

namespace App\Repositories;

use App\Models\User;
use App\Support\DTO\Pagination\PaginationRequestDTO;
use App\Support\DTO\Pagination\PaginationResultDTO;
use App\Services\PaginationService;

class UserRepository
{
    public function __construct(
        private User $model,
        private PaginationService $paginationService
    ) {}
    
    public function paginate(PaginationRequestDTO $dto): PaginationResultDTO
    {
        $query = $this->model->newQuery();
        
        // Применяем фильтры из DTO
        foreach ($dto->filters as $field => $value) {
            if ($value !== null) {
                $query->where($field, 'like', "%{$value}%");
            }
        }
        
        // Применяем сортировку
        if ($dto->sortBy) {
            $query->orderBy($dto->sortBy, $dto->sortDirection ?? 'asc');
        }
        
        // Используем сервис пагинации
        return $this->paginationService->paginate($query, $dto);
    }
    
    public function getForInfiniteScroll(int $perPage = 20, ?string $cursor = null)
    {
        $query = $this->model->newQuery()->orderBy('created_at', 'desc');
        
        return $query->cursorPaginate($perPage, ['*'], 'cursor', $cursor);
    }
}
```

## 2. Сервис пагинации со стратегиями

### app/Services/PaginationService.php
```php
<?php

namespace App\Services;

use Illuminate\Database\Eloquent\Builder;
use App\Support\DTO\Pagination\PaginationRequestDTO;
use App\Support\DTO\Pagination\PaginationResultDTO;
use App\Support\Pagination\PaginatorContext;
use App\Enums\PaginationType;

class PaginationService
{
    public function __construct(
        private PaginatorContext $paginatorContext
    ) {}
    
    public function paginate(Builder $query, PaginationRequestDTO $dto): PaginationResultDTO
    {
        // Устанавливаем стратегию
        $this->paginatorContext->setStrategy($dto->type);
        
        // Выполняем пагинацию
        return $this->paginatorContext->paginate($query, $dto);
    }
    
    public function paginateWithAutoStrategy(Builder $query, PaginationRequestDTO $dto): PaginationResultDTO
    {
        // Автоматически выбираем лучшую стратегию
        $optimalType = $this->detectOptimalStrategy($query, $dto);
        $dto->type = $optimalType;
        
        return $this->paginate($query, $dto);
    }
    
    private function detectOptimalStrategy(Builder $query, PaginationRequestDTO $dto): PaginationType
    {
        // Если клиент явно указал тип - используем его
        if ($dto->type !== PaginationType::AUTO) {
            return $dto->type;
        }
        
        // Проверяем, есть ли курсор (для бесконечного скролла)
        if (request()->has('cursor')) {
            return PaginationType::CURSOR;
        }
        
        // Оцениваем размер данных
        try {
            $count = $query->count();
            
            if ($count > 100000) {
                return PaginationType::CURSOR;
            }
            
            if ($count > 50000) {
                return PaginationType::SIMPLE;
            }
            
            return PaginationType::LENGTH_AWARE;
        } catch (\Exception $e) {
            // Если COUNT медленный, используем простую пагинацию
            return PaginationType::SIMPLE;
        }
    }
}
```

## 3. Стратегии пагинации

### app/Support/Pagination/Strategies/LengthAwareStrategy.php
```php
<?php

namespace App\Support\Pagination\Strategies;

use Illuminate\Database\Eloquent\Builder;
use App\Support\DTO\Pagination\PaginationRequestDTO;
use App\Support\DTO\Pagination\PaginationResultDTO;
use App\Support\Pagination\Contracts\PaginatorStrategy;

class LengthAwareStrategy implements PaginatorStrategy
{
    public function paginate(Builder $query, PaginationRequestDTO $dto): PaginationResultDTO
    {
        $paginator = $query->paginate(
            $dto->perPage,
            ['*'],
            'page',
            $dto->page
        );
        
        return new PaginationResultDTO(
            items: $paginator->items(),
            meta: [
                'total' => $paginator->total(),
                'per_page' => $paginator->perPage(),
                'current_page' => $paginator->currentPage(),
                'last_page' => $paginator->lastPage(),
                'from' => $paginator->firstItem(),
                'to' => $paginator->lastItem(),
                'type' => 'length_aware'
            ],
            links: [
                'first' => $paginator->url(1),
                'last' => $paginator->url($paginator->lastPage()),
                'prev' => $paginator->previousPageUrl(),
                'next' => $paginator->nextPageUrl(),
            ]
        );
    }
}
```

### app/Support/Pagination/Strategies/CursorStrategy.php
```php
<?php

namespace App\Support\Pagination\Strategies;

use Illuminate\Database\Eloquent\Builder;
use App\Support\DTO\Pagination\PaginationRequestDTO;
use App\Support\DTO\Pagination\PaginationResultDTO;
use App\Support\Pagination\Contracts\PaginatorStrategy;

class CursorStrategy implements PaginatorStrategy
{
    public function paginate(Builder $query, PaginationRequestDTO $dto): PaginationResultDTO
    {
        // Для курсорной пагинации нужен порядок
        if (empty($dto->sortBy)) {
            $query->orderBy('id', 'desc');
        }
        
        $paginator = $query->cursorPaginate(
            $dto->perPage,
            ['*'],
            'cursor',
            $dto->cursor
        );
        
        return new PaginationResultDTO(
            items: $paginator->items(),
            meta: [
                'per_page' => $paginator->perPage(),
                'has_more' => $paginator->hasMorePages(),
                'type' => 'cursor'
            ],
            links: [
                'next_cursor' => $paginator->nextCursor()?->encode(),
                'prev_cursor' => $paginator->previousCursor()?->encode(),
            ]
        );
    }
}
```

## 4. DTO (Data Transfer Objects)

### app/Support/DTO/Pagination/PaginationRequestDTO.php
```php
<?php

namespace App\Support\DTO\Pagination;

use App\Enums\PaginationType;
use Spatie\DataTransferObject\DataTransferObject;

class PaginationRequestDTO extends DataTransferObject
{
    public PaginationType $type = PaginationType::AUTO;
    public int $perPage = 15;
    public ?int $page = null;
    public ?string $cursor = null;
    public ?string $sortBy = null;
    public ?string $sortDirection = 'asc';
    public array $filters = [];
    
    public static function fromRequest(array $data): self
    {
        return new self([
            'type' => PaginationType::tryFrom($data['pagination_type'] ?? 'auto'),
            'perPage' => $data['per_page'] ?? $data['perPage'] ?? 15,
            'page' => $data['page'] ?? null,
            'cursor' => $data['cursor'] ?? null,
            'sortBy' => $data['sort_by'] ?? $data['sortBy'] ?? null,
            'sortDirection' => $data['sort_direction'] ?? $data['sortDirection'] ?? 'asc',
            'filters' => $data['filters'] ?? [],
        ]);
    }
}
```

### app/Support/DTO/User/UserListDTO.php
```php
<?php

namespace App\Support\DTO\User;

use App\Enums\PaginationType;
use Spatie\DataTransferObject\DataTransferObject;

class UserListDTO extends DataTransferObject
{
    public ?string $search = null;
    public ?string $role = null;
    public bool $active = true;
    public int $perPage = 15;
    public PaginationType $paginationType = PaginationType::AUTO;
    public ?string $sortBy = 'created_at';
    public string $sortDirection = 'desc';
    
    public function toPaginationDTO(): PaginationRequestDTO
    {
        $filters = [];
        
        if ($this->search) {
            $filters['search'] = $this->search;
        }
        
        if ($this->role) {
            $filters['role'] = $this->role;
        }
        
        $filters['active'] = $this->active;
        
        return new PaginationRequestDTO([
            'type' => $this->paginationType,
            'perPage' => $this->perPage,
            'sortBy' => $this->sortBy,
            'sortDirection' => $this->sortDirection,
            'filters' => $filters,
        ]);
    }
}
```

## 5. UserService (бизнес-логика)

### app/Services/UserService.php
```php
<?php

namespace App\Services;

use App\Repositories\UserRepository;
use App\Support\DTO\User\UserListDTO;
use App\Support\DTO\Pagination\PaginationResultDTO;

class UserService
{
    public function __construct(
        private UserRepository $userRepository,
        private PaginationService $paginationService
    ) {}
    
    public function getPaginatedUsers(UserListDTO $dto): PaginationResultDTO
    {
        return $this->userRepository->paginate(
            $dto->toPaginationDTO()
        );
    }
    
    public function getUsersForAdmin(UserListDTO $dto): PaginationResultDTO
    {
        // Специфичная логика для админки
        $paginationDTO = $dto->toPaginationDTO();
        $paginationDTO->type = PaginationType::LENGTH_AWARE; // Админке всегда с total
        
        return $this->userRepository->paginate($paginationDTO);
    }
    
    public function getUsersForMobile(UserListDTO $dto): PaginationResultDTO
    {
        // Для мобильного приложения - курсорная пагинация
        $paginationDTO = $dto->toPaginationDTO();
        $paginationDTO->type = PaginationType::CURSOR;
        $paginationDTO->perPage = 20; // Больше записей для скролла
        
        return $this->userRepository->paginate($paginationDTO);
    }
}
```

## 6. Контроллер (финальная точка)

### app/Http/Controllers/UserController.php
```php
<?php

namespace App\Http\Controllers;

use App\Services\UserService;
use App\Http\Requests\UserListRequest;
use App\Http\Resources\UserResource;
use App\Http\Resources\PaginationResource;

class UserController extends Controller
{
    public function __construct(private UserService $userService)
    {}
    
    public function index(UserListRequest $request)
    {
        $dto = $request->toDTO();
        
        // Автоматический выбор стратегии
        $result = $this->userService->getPaginatedUsers($dto);
        
        return response()->json([
            'status' => 'success',
            'data' => UserResource::collection($result->items),
            'pagination' => PaginationResource::make($result)
        ]);
    }
    
    public function infiniteScroll(UserListRequest $request)
    {
        $dto = $request->toDTO();
        $dto->paginationType = PaginationType::CURSOR;
        
        $result = $this->userService->getPaginatedUsers($dto);
        
        return response()->json($result->toArray());
    }
}
```

## Преимущества этой структуры:

1. **`Repositories/`** - работа с данными
2. **`Services/`** - бизнес-логика
3. **`Support/Pagination/`** - стратегии пагинации
4. **`Support/DTO/`** - объекты данных
5. **Все на своих местах** без over-engineering
6. **Гибкость** - можно легко добавлять новые стратегии
7. **Laravel-way** - использует стандартные подходы

Это **практичная структура** для реальных проектов, которая масштабируется, но не переусложнена.