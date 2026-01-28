Отлично 🚀  
Docker-часть мы закрыли, теперь делаем **первый “прикладной” этап**, который сразу показывает зрелость проекта:

> **.env + подключение MySQL / Redis / Elasticsearch**

Я покажу:

- **что именно менять**
    
- **почему так**
    
- **как проверить, что всё реально работает**
    

Без лишней магии.

---

# 1️⃣ `.env` — базовая настройка окружения

Открой файл:

```bash
src/.env
```

### 1.1 APP

```env
APP_NAME=LaravelProductsApi
APP_ENV=local
APP_KEY=base64:...
APP_DEBUG=true
APP_URL=http://localhost:8080
```

`APP_URL` важно указать именно `8080`, т.к. nginx там.

---

# 2️⃣ Подключение MySQL (из Docker)

Смотри на `docker-compose.yml`:

```yaml
mysql:
  image: mysql:8.0
  container_name: laravel_mysql
  environment:
    MYSQL_DATABASE: laravel
    MYSQL_USER: laravel
    MYSQL_PASSWORD: secret
```

👉 **Имя сервиса = hostname в Docker-сети**

### 2.1 `.env` → database

```env
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=laravel
DB_PASSWORD=secret
```

⚠️ **НЕ localhost**  
В Docker:

- `localhost` = сам контейнер
    
- `mysql` = сервис MySQL
    

---

### 2.2 Проверка подключения MySQL

```bash
docker compose exec app php artisan migrate:status
```

Если видишь список миграций (пусть даже пустой) — ✔ подключение работает.
Если ошибка `SQLSTATE[HY000]` → сразу скажи, посмотрим.

---

# 3️⃣ Подключение Redis

Redis у нас:

```yaml
redis:
  image: redis:7-alpine
  container_name: laravel_redis
```

### 3.1 `.env`

```env
REDIS_CLIENT=phpredis

REDIS_HOST=redis
REDIS_PASSWORD=null
REDIS_PORT=6379
```

Laravel 11 по умолчанию **умеет Redis из коробки**, мы уже поставили расширение `redis`.

---

### 3.2 Проверка Redis

```bash
docker compose exec app php artisan tinker
```

В tinker:

```php
Cache::put('test_key', 'redis_ok', 60);
Cache::get('test_key');
```

Если вернуло:

```text
"redis_ok"
```

🔥 Redis подключён правильно.

---

# 4️⃣ Подключение Elasticsearch

Elasticsearch у нас:

```yaml
elasticsearch:
  image: docker.elastic.co/elasticsearch/elasticsearch:8.11.3
  container_name: laravel_elasticsearch
```

### 4.1 `.env`

Добавь в конец `.env`:

```env
ELASTICSEARCH_HOST=elasticsearch
ELASTICSEARCH_PORT=9200
ELASTICSEARCH_SCHEME=http
```

> Мы **НЕ используем localhost**, а имя сервиса.

---

### 4.2 Быстрая проверка ES (без Laravel)

```bash
docker compose exec app curl http://elasticsearch:9200
```

Ожидаемый ответ (пример):

```json
{
  "name": "laravel_elasticsearch",
  "cluster_name": "docker-cluster",
  "version": {
    "number": "8.11.3"
  }
}
```

Если ответ есть — ES жив и доступен.

---

# 5️⃣ Мини-рефактор config (best practice)

Чтобы не тянуть `.env` напрямую дальше, подготовим `config`.

### 5.1 `config/services.php`

Добавь:

```php
'elasticsearch' => [
    'host' => env('ELASTICSEARCH_HOST', 'localhost'),
    'port' => env('ELASTICSEARCH_PORT', 9200),
    'scheme' => env('ELASTICSEARCH_SCHEME', 'http'),
],
```

Это пригодится, когда будем делать клиент ES.

---

# 6️⃣ Контрольный чеклист (ВАЖНО)

Ты должен уметь ответить **ДА** на всё:

-  Laravel открывается в браузере
    
-  `php artisan migrate:status` работает
    
-  Redis работает через `Cache`
    
-  Elasticsearch отвечает по `curl`
    

Если где-то ❌ — мы **останавливаемся и чиним**, не идём дальше.

---

# 7️⃣ Почему это выглядит сильно для тестового

Ты уже можешь честно сказать:

> Проект полностью dockerized, использует MySQL, Redis и Elasticsearch как отдельные сервисы, конфигурация вынесена в `.env` и `config`, без зависимости от локального окружения.

Это **уровень middle+**.

---

# Следующий шаг (логичный)

Если всё выше ок — дальше **самое вкусное**:

👉 **Миграции `categories` и `products`**  
👉 связи  
👉 индексы под фильтры и поиск

Напиши:

1. прошёл ли `migrate:status`
2. прошёл ли Redis-тест
3. ответил ли Elasticsearch

И двигаемся дальше 💪