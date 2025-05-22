### **Анализ таблицы `addon_settings` (настройки платежных систем)**

Эта таблица хранит конфигурации различных платежных шлюзов (HyperPay, Flutterwave и др.). Рассмотрим ее структуру и назначение полей.

---

## **1. Структура таблицы и назначение полей**

| Поле              | Тип данных       | Описание                                                                 |
|--------------------|------------------|-------------------------------------------------------------------------|
| `id`              | UUID/VARCHAR     | Уникальный идентификатор записи                                         |
| `key_name`        | VARCHAR          | Название платежного шлюза (например, `hyper_pay`, `flutterwave`)       |
| `live_values`     | JSON/TEXT        | Данные для **боевого режима** (production)                             |
| `test_values`     | JSON/TEXT        | Данные для **тестового режима** (sandbox)                              |
| `settings_type`   | VARCHAR          | Тип настроек (здесь всегда `payment_config`)                           |
| `mode`            | VARCHAR          | Текущий режим работы (`live` или `test`)                               |
| `is_active`       | TINYINT/BOOLEAN  | Активен ли шлюз (`1` — да, `0` — нет)                                 |
| `created_at`      | TIMESTAMP        | Дата создания записи                                                   |
| `updated_at`      | TIMESTAMP        | Дата последнего обновления                                             |
| `additional_data` | JSON/TEXT        | Дополнительные данные (название, логотип, кастомизация)                |

---

## **2. Примеры записей и их расшифровка**

### **Запись #1: HyperPay (неактивен)**
```json
{
  "id": "133d9647-cabb-11ed-8fec-0c7a158e4469",
  "key_name": "hyper_pay",
  "live_values": {
    "gateway": "hyper_pay",
    "mode": "live",
    "status": "0",
    "entity_id": "",
    "access_code": ""
  },
  "test_values": {
    "gateway": "hyper_pay",
    "mode": "live",
    "status": "0",
    "entity_id": "",
    "access_code": ""
  },
  "settings_type": "payment_config",
  "mode": "test",
  "is_active": 0,
  "created_at": null,
  "updated_at": "2023-04-09 01:59:22",
  "additional_data": null
}
```
**Что это значит?**  
- Платежка **HyperPay** подключена, но **неактивна** (`is_active=0`).  
- В `live_values` и `test_values` нет ключей (`entity_id` и `access_code` пустые) — значит, шлюз не настроен.  
- Режим работы (`mode`) стоит `test`, но так как ключей нет, платежи не работают.  

---

### **Запись #2: Flutterwave (активен)**
```json
{
  "id": "d4f3f5f1-d6a0-11ed-962c-0c7a158e4469",
  "key_name": "flutterwave",
  "live_values": {
    "gateway": "flutterwave",
    "mode": "live",
    "status": "1",
    "secret_key": "FLWSECK_TEST-...",
    "public_key": "FLWPUBK_TEST-...",
    "hash": "FLWSECK_TEST..."
  },
  "test_values": {
    "gateway": "flutterwave",
    "mode": "live",
    "status": "1",
    "secret_key": "FLWSECK_TEST-...",
    "public_key": "FLWPUBK_TEST-...",
    "hash": "FLWSECK_TEST..."
  },
  "settings_type": "payment_config",
  "mode": "live",
  "is_active": 1,
  "created_at": null,
  "updated_at": "2024-09-27 23:27:25",
  "additional_data": {
    "gateway_title": "Flutterwave",
    "gateway_image": "2024-09-27-66f6f90dc95e9.png"
  }
}
```
**Что это значит?**  
- Платежка **Flutterwave** активна (`is_active=1`) и в **боевом режиме** (`mode=live`).  
- В `live_values` указаны тестовые ключи (`FLWSECK_TEST`), что странно — вероятно, ошибка конфигурации.  
- В `additional_data` есть название и логотип для отображения в интерфейсе.  

---

## **3. Проблемы и несоответствия**
1. **Flutterwave в `live_mode`, но с тестовыми ключами**  
   - В `live_values` должны быть **боевые ключи** (начинающиеся с `FLWSECK_LIVE-...`).  
   - Сейчас система может работать некорректно при реальных платежах.

2. **HyperPay не настроен**  
   - Поля `entity_id` и `access_code` пустые — шлюз неработоспособен.

3. **Дублирование данных**  
   - В `live_values` и `test_values` часто одинаковые данные (как у Flutterwave). Лучше разделять.

---

## **4. Как правильно использовать эту таблицу?**
### **Для добавления нового шлюза (например, Tinkoff)**
```sql
INSERT INTO addon_settings (
  id,
  key_name,
  live_values,
  test_values,
  settings_type,
  mode,
  is_active,
  additional_data
) VALUES (
  UUID(),
  'tinkoff',
  '{
    "gateway": "tinkoff",
    "mode": "live",
    "terminal_key": "TINKOFF_LIVE_TERMINAL_KEY",
    "secret_key": "TINKOFF_LIVE_SECRET_KEY"
  }',
  '{
    "gateway": "tinkoff",
    "mode": "test",
    "terminal_key": "TINKOFF_TEST_TERMINAL_KEY",
    "secret_key": "TINKOFF_TEST_SECRET_KEY"
  }',
  'payment_config',
  'test',
  1,
  '{
    "gateway_title": "Tinkoff",
    "gateway_image": "tinkoff-logo.png"
  }'
);
```

### **Для проверки активных платежек в коде**
```php
$activeGateways = DB::table('addon_settings')
    ->where('settings_type', 'payment_config')
    ->where('is_active', 1)
    ->get();

foreach ($activeGateways as $gateway) {
    $config = ($gateway->mode === 'live') 
        ? json_decode($gateway->live_values, true) 
        : json_decode($gateway->test_values, true);
    
    echo "Шлюз: {$gateway->key_name}, режим: {$gateway->mode}";
}
```

---

## **Вывод**
- Таблица **`addon_settings`** хранит гибкие настройки платежных систем.  
- **Проблемы**:  
  - У Flutterwave тестовые ключи в `live_mode`.  
  - HyperPay не настроен.  
- **Рекомендации**:  
  - Разделять `live` и `test`-ключи.  
  - Добавить валидацию при сохранении.  

Если нужно доработать логику платежей — пишите! 🚀