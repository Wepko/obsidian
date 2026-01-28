**Паттерн Specification** (или "Спецификация") — это поведенческий паттерн проектирования, который позволяет инкапсулировать бизнес-правила или условия в отдельные объекты-спецификации. Эти объекты можно комбинировать, чтобы строить сложные запросы или проверки.

## Основная идея
Вместо того чтобы размещать логику проверок прямо в доменных объектах или сервисах, мы создаем отдельные классы, каждый из которых отвечает за одно конкретное правило.

## Ключевые компоненты

1. **Интерфейс Specification** 
```java
public interface Specification<T> {
    boolean isSatisfiedBy(T candidate);
    Specification<T> and(Specification<T> other);
    Specification<T> or(Specification<T> other);
    Specification<T> not();
}
```

2. **Конкретные спецификации** — реализуют конкретные бизнес-правила

## Пример применения

### Без паттерна:
```java
public class OrderService {
    public List<Order> getActiveOrders(Customer customer) {
        return orders.stream()
            .filter(o -> o.getCustomer().equals(customer))
            .filter(o -> o.getStatus() == Status.ACTIVE)
            .filter(o -> o.getTotal() > 1000)
            .collect(Collectors.toList());
    }
}
```

### С паттерном Specification:

```java
// Создаем спецификации
public class CustomerSpecification implements Specification<Order> {
    private Customer customer;
    
    public boolean isSatisfiedBy(Order order) {
        return order.getCustomer().equals(customer);
    }
}

public class ActiveStatusSpecification implements Specification<Order> {
    public boolean isSatisfiedBy(Order order) {
        return order.getStatus() == Status.ACTIVE;
    }
}

public class MinimumAmountSpecification implements Specification<Order> {
    private double minAmount;
    
    public boolean isSatisfiedBy(Order order) {
        return order.getTotal() > minAmount;
    }
}

// Использование
Specification<Order> spec = new CustomerSpecification(customer)
    .and(new ActiveStatusSpecification())
    .and(new MinimumAmountSpecification(1000));

List<Order> activeOrders = orders.stream()
    .filter(spec::isSatisfiedBy)
    .collect(Collectors.toList());
```

## Преимущества

1. **Устранение дублирования** — бизнес-правила используются повторно
2. **Гибкость** — правила легко комбинировать
3. **Читаемость** — код становится более декларативным
4. **Тестируемость** — каждую спецификацию можно тестировать отдельно
5. **Отделение бизнес-логики** от кода доступа к данным

## Расширенные варианты

### 1. **Спецификация для репозиториев** (Query Specification)
```java
public interface Specification<T> {
    Predicate<T> toPredicate();
    // Для использования в JPA/Hibernate
}
```

### 2. **Спецификация для валидации**
```java
public class Validator {
    public ValidationResult validate(Specification spec, Object obj) {
        return spec.isSatisfiedBy(obj) 
            ? ValidationResult.valid()
            : ValidationResult.invalid(spec.getErrorMessage());
    }
}
```

## Практические сценарии использования

1. **Фильтрация данных** в UI или API
2. **Валидация сложных бизнес-правил**
3. **Построение динамических запросов** к БД
4. **Проверка прав доступа**
5. **Поиск объектов** по различным критериям

## Реализация в разных языках

- **C#** — LINQ Specification pattern
- **Java** — часто используется с JPA Criteria API
- **TypeScript** — может быть реализован с помощью функций или классов

Паттерн особенно полезен в Domain-Driven Design (DDD), где он помогает четко отделять бизнес-правила от инфраструктурного кода.