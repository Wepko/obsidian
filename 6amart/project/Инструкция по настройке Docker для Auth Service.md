Вот детальная инструкция по настройке Docker для Auth Service на Laravel в рамках микросервисной архитектуры:

### Docker-конфигурация для Laravel Auth Service

#### 1. Структура проекта
```
auth-service/
├── app/              # Laravel application code
├── bootstrap/
├── config/
├── database/
├── docker/           # Docker-specific files
│   ├── php/
│   │   └── Dockerfile
│   └── nginx/
│       └── default.conf
├── storage/
├── tests/
├── .env              # Environment variables
├── .env.docker       # Docker-specific env
├── docker-compose.yml
├── artisan
└── composer.json
```

#### 2. Dockerfile для PHP-FPM
`docker/php/Dockerfile`:
```dockerfile
FROM php:8.2-fpm

# Install system dependencies
RUN apt-get update && apt-get install -y \
    git \
    curl \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    zip \
    unzip \
    libpq-dev

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Install PHP extensions
RUN docker-php-ext-install pdo pdo_mysql mbstring exif pcntl bcmath gd

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Set working directory
WORKDIR /var/www

# Copy existing application directory contents
COPY . /var/www

# Install PHP dependencies
RUN composer install --no-interaction --optimize-autoloader --no-dev

# Permissions
RUN chown -R www-data:www-data /var/www/storage
RUN chmod -R 775 /var/www/storage

# Expose port 9000 for PHP-FPM
EXPOSE 9000

CMD ["php-fpm"]
```

#### 3. Nginx конфигурация
`docker/nginx/default.conf`:
```nginx
server {
    listen 80;
    index index.php index.html;
    server_name auth-service.local;
    root /var/www/public;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass auth-service:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }

    error_log /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
}
```

#### 4. docker-compose.yml
```yaml
version: '3.8'

services:
  auth-service:
    build:
      context: .
      dockerfile: docker/php/Dockerfile
    container_name: auth-service
    restart: unless-stopped
    volumes:
      - ./:/var/www
      - ./docker/php/php.ini:/usr/local/etc/php/conf.d/custom.ini
    networks:
      - app-network
    depends_on:
      - auth-db

  auth-nginx:
    image: nginx:alpine
    container_name: auth-nginx
    restart: unless-stopped
    ports:
      - "8001:80"
    volumes:
      - ./:/var/www
      - ./docker/nginx/default.conf:/etc/nginx/conf.d/default.conf
    networks:
      - app-network
    depends_on:
      - auth-service

  auth-db:
    image: mysql:8.0
    container_name: auth-db
    restart: unless-stopped
    environment:
      MYSQL_DATABASE: ${DB_DATABASE}
      MYSQL_ROOT_PASSWORD: ${DB_PASSWORD}
      MYSQL_PASSWORD: ${DB_PASSWORD}
      MYSQL_USER: ${DB_USERNAME}
    volumes:
      - auth-db-data:/var/lib/mysql
    ports:
      - "33061:3306"
    networks:
      - app-network

  auth-redis:
    image: redis:alpine
    container_name: auth-redis
    ports:
      - "63791:6379"
    volumes:
      - auth-redis-data:/data
    networks:
      - app-network

volumes:
  auth-db-data:
  auth-redis-data:

networks:
  app-network:
    driver: bridge
```

#### 5. .env.docker
```ini
APP_NAME=AuthService
APP_ENV=local
APP_KEY=
APP_DEBUG=true
APP_URL=http://auth-service.local

LOG_CHANNEL=stack

DB_CONNECTION=mysql
DB_HOST=auth-db
DB_PORT=3306
DB_DATABASE=auth_service
DB_USERNAME=root
DB_PASSWORD=secret

REDIS_HOST=auth-redis
REDIS_PASSWORD=null
REDIS_PORT=6379

JWT_SECRET=
```

#### 6. Настройка Laravel для микросервиса

1. Установите необходимые пакеты:
```bash
composer require laravel/passport tymon/jwt-auth
```

2. Настройте аутентизацию (config/auth.php):
```php
'guards' => [
    'api' => [
        'driver' => 'jwt',
        'provider' => 'users',
    ],
],
```

3. Создайте базовые маршруты (routes/api.php):
```php
Route::prefix('auth')->group(function () {
    Route::post('register', [AuthController::class, 'register']);
    Route::post('login', [AuthController::class, 'login']);
    Route::post('logout', [AuthController::class, 'logout'])->middleware('auth:api');
    Route::post('refresh', [AuthController::class, 'refresh'])->middleware('auth:api');
    Route::get('me', [AuthController::class, 'me'])->middleware('auth:api');
});
```

#### 7. Запуск сервиса

1. Соберите и запустите контейнеры:
```bash
cp .env.docker .env
docker-compose up -d --build
```

2. Установите ключи приложения:
```bash
docker exec -it auth-service php artisan key:generate
docker exec -it auth-service php artisan jwt:secret
```

3. Запустите миграции:
```bash
docker exec -it auth-service php artisan migrate
```

4. (Опционально) Запустите сидеры:
```bash
docker exec -it auth-service php artisan db:seed
```

#### 8. Проверка работы

1. Проверьте доступность сервиса:
```bash
curl http://localhost:8001/auth/health
```

2. Протестируйте регистрацию:
```bash
curl -X POST http://localhost:8001/auth/register \
  -H "Content-Type: application/json" \
  -d '{"name":"Test User","email":"test@example.com","password":"password"}'
```

### Интеграция с другими сервисами

Для взаимодействия с другими микросервисами добавьте в docker-compose.yml:
```yaml
services:
  # ... существующие сервисы
  
  api-gateway:
    image: nginx
    ports:
      - "8000:80"
    volumes:
      - ./gateway/nginx.conf:/etc/nginx/conf.d/default.conf
    networks:
      - app-network
    depends_on:
      - auth-service
      - store-service
      # ... другие сервисы
```

Пример nginx.conf для API Gateway:
```nginx
server {
    listen 80;

    location /auth/ {
        proxy_pass http://auth-service:80/;
    }

    location /stores/ {
        proxy_pass http://store-service:80/;
    }
    
    # ... другие сервисы
}
```

Эта конфигурация обеспечит:
- Изолированную работу Auth Service
- Возможность масштабирования
- Легкую интеграцию с другими сервисами
- Стандартизированное API для аутентификации