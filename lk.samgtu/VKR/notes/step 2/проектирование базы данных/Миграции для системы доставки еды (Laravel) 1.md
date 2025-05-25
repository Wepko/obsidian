# Миграции для системы доставки еды (Laravel)

## 1. Пользователи (users)

```php
Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('email')->unique();
    $table->timestamp('email_verified_at')->nullable();
    $table->string('password');
    $table->string('phone')->nullable();
    $table->string('avatar')->nullable();
    $table->enum('type', ['client', 'courier', 'shop_staff', 'admin'])->default('client');
    $table->enum('status', ['active', 'inactive', 'banned'])->default('active');
    $table->foreignId('current_zone_id')->nullable()->constrained('zones');
    $table->rememberToken();
    $table->timestamps();
    $table->softDeletes();
});
```

## 2. Роли (roles)

```php
Schema::create('roles', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('guard_name')->default('web');
    $table->timestamps();
});
```

## 3. Магазины (shops)

```php
Schema::create('shops', function (Blueprint $table) {
    $table->id();
    $table->foreignId('owner_id')->constrained('users');
    $table->string('name');
    $table->text('description')->nullable();
    $table->string('address');
    $table->decimal('latitude', 10, 8);
    $table->decimal('longitude', 11, 8);
    $table->string('phone');
    $table->string('email');
    $table->string('logo')->nullable();
    $table->string('cover_image')->nullable();
    $table->enum('status', ['active', 'inactive', 'closed'])->default('active');
    $table->boolean('is_featured')->default(false);
    $table->integer('delivery_time')->default(30);
    $table->decimal('delivery_fee', 8, 2)->default(0);
    $table->decimal('min_order', 8, 2)->default(0);
    $table->decimal('tax', 5, 2)->default(0);
    $table->foreignId('module_id')->constrained('modules');
    $table->foreignId('zone_id')->constrained('zones');
    $table->timestamps();
    $table->softDeletes();
});
```

## 4. Зоны покрытия (zones)

```php
Schema::create('zones', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->geometry('coordinates'); // Для хранения полигона GeoJSON
    $table->enum('status', ['active', 'inactive'])->default('active');
    $table->timestamps();
});
```

## 5. Курьеры (couriers)

```php
Schema::create('couriers', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained('users')->unique();
    $table->enum('vehicle_type', ['bike', 'car', 'walk']);
    $table->enum('courier_type', ['freelance', 'employee']);
    $table->integer('min_delivery_area')->default(1);
    $table->integer('max_delivery_area')->default(10);
    $table->decimal('delivery_fee', 8, 2)->default(0);
    $table->decimal('rating', 3, 2)->default(5);
    $table->enum('status', ['available', 'unavailable', 'delivering'])->default('available');
    $table->point('current_location')->nullable();
    $table->timestamps();
});
```

## 6. Категории товаров (categories)

```php
Schema::create('categories', function (Blueprint $table) {
    $table->id();
    $table->foreignId('parent_id')->nullable()->constrained('categories');
    $table->string('name');
    $table->string('image')->nullable();
    $table->integer('position')->default(0);
    $table->enum('status', ['active', 'inactive'])->default('active');
    $table->timestamps();
    $table->softDeletes();
});
```

## 7. Товары (products)

```php
Schema::create('products', function (Blueprint $table) {
    $table->id();
    $table->foreignId('shop_id')->constrained('shops');
    $table->foreignId('category_id')->constrained('categories');
    $table->string('name');
    $table->text('description')->nullable();
    $table->string('image')->nullable();
    $table->decimal('price', 8, 2);
    $table->decimal('discount_price', 8, 2)->nullable();
    $table->string('unit')->default('шт');
    $table->integer('stock')->default(0);
    $table->boolean('is_featured')->default(false);
    $table->boolean('is_active')->default(true);
    $table->timestamps();
    $table->softDeletes();
});
```

## 8. Атрибуты товаров (product_attributes)

```php
Schema::create('product_attributes', function (Blueprint $table) {
    $table->id();
    $table->foreignId('product_id')->constrained('products');
    $table->string('name');
    $table->decimal('price', 8, 2)->default(0);
    $table->timestamps();
});
```

## 9. Дополнения товаров (product_addons)

```php
Schema::create('product_addons', function (Blueprint $table) {
    $table->id();
    $table->foreignId('product_id')->constrained('products');
    $table->string('name');
    $table->decimal('price', 8, 2)->default(0);
    $table->timestamps();
});
```

## 10. Заказы (orders)

```php
Schema::create('orders', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained('users');
    $table->foreignId('shop_id')->constrained('shops');
    $table->foreignId('courier_id')->nullable()->constrained('couriers');
    $table->string('order_number')->unique();
    $table->enum('order_type', ['delivery', 'pickup']);
    $table->string('delivery_address')->nullable();
    $table->point('delivery_location')->nullable();
    $table->timestamp('scheduled_at')->nullable();
    $table->integer('items_count')->default(0);
    $table->decimal('items_price', 10, 2)->default(0);
    $table->decimal('delivery_fee', 8, 2)->default(0);
    $table->decimal('tax', 8, 2)->default(0);
    $table->decimal('discount', 8, 2)->default(0);
    $table->decimal('total', 10, 2)->default(0);
    $table->enum('payment_method', ['cash', 'card', 'online']);
    $table->enum('payment_status', ['pending', 'paid', 'failed'])->default('pending');
    $table->enum('order_status', ['pending', 'processing', 'ready_for_delivery', 'on_delivery', 'delivered', 'cancelled'])->default('pending');
    $table->boolean('shop_confirmed')->default(false);
    $table->boolean('courier_confirmed')->default(false);
    $table->text('notes')->nullable();
    $table->timestamps();
    $table->softDeletes();
});
```

## 11. Позиции заказа (order_items)

```php
Schema::create('order_items', function (Blueprint $table) {
    $table->id();
    $table->foreignId('order_id')->constrained('orders');
    $table->foreignId('product_id')->constrained('products');
    $table->integer('quantity')->default(1);
    $table->decimal('price', 8, 2);
    $table->decimal('total', 8, 2);
    $table->timestamps();
});
```

## 12. Атрибуты позиций заказа (order_item_attributes)

```php
Schema::create('order_item_attributes', function (Blueprint $table) {
    $table->id();
    $table->foreignId('order_item_id')->constrained('order_items');
    $table->foreignId('attribute_id')->constrained('product_attributes');
    $table->timestamps();
});
```

## 13. Дополнения позиций заказа (order_item_addons)

```php
Schema::create('order_item_addons', function (Blueprint $table) {
    $table->id();
    $table->foreignId('order_item_id')->constrained('order_items');
    $table->foreignId('addon_id')->constrained('product_addons');
    $table->timestamps();
});
```

## 14. Транзакции (transactions)

```php
Schema::create('transactions', function (Blueprint $table) {
    $table->id();
    $table->foreignId('order_id')->nullable()->constrained('orders');
    $table->foreignId('user_id')->constrained('users');
    $table->decimal('amount', 10, 2);
    $table->enum('type', ['payment', 'withdrawal', 'refund']);
    $table->enum('status', ['pending', 'completed', 'failed'])->default('pending');
    $table->string('payment_method');
    $table->json('payment_details')->nullable();
    $table->text('notes')->nullable();
    $table->timestamps();
});
```

## 15. Модули (modules)

```php
Schema::create('modules', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('type');
    $table->string('theme');
    $table->string('icon');
    $table->string('thumbnail');
    $table->enum('status', ['active', 'inactive'])->default('active');
    $table->timestamps();
});
```

## 16. Настройки бизнеса (business_settings)

```php
Schema::create('business_settings', function (Blueprint $table) {
    $table->id();
    $table->string('key')->unique();
    $table->text('value')->nullable();
    $table->timestamps();
});
```

## 17. Переводы (translations)

```php
Schema::create('translations', function (Blueprint $table) {
    $table->id();
    $table->string('table_name');
    $table->string('column_name');
    $table->unsignedBigInteger('foreign_key');
    $table->string('locale');
    $table->text('value');
    $table->timestamps();

    $table->unique(['table_name', 'column_name', 'foreign_key', 'locale']);
});
```

## 18. Промежуточные таблицы для отношений многие-ко-многим

```php
// Связь пользователей с ролями
Schema::create('model_has_roles', function (Blueprint $table) {
    $table->unsignedBigInteger('role_id');
    $table->string('model_type');
    $table->unsignedBigInteger('model_id');
    
    $table->foreign('role_id')->references('id')->on('roles')->onDelete('cascade');
    $table->primary(['role_id', 'model_id', 'model_type']);
});
```

Эти миграции создают полную структуру базы данных для системы доставки еды с учетом всех требований. Для работы с геоданными (полигоны зон, точки местоположения) потребуется установка дополнительных пакетов для Laravel, таких как `geoip` или `laravel-geo`.