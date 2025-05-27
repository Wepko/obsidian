# **Отчет по настройке и развертыванию программного комплекса**  
**Тема:** Реализация и тестирование программного комплекса  
**Раздел:** 3.1. Настройка и развертывание  

## **1. Развертывание Laravel (PHP)**  

### **1.1. Установка зависимостей**  
Перед развертыванием Laravel необходимо установить:  
- PHP (8.0+)  
- Composer  
- Веб-сервер (Apache/Nginx)  

**Шаги:**  
1. Установка PHP и зависимостей:  
   ```bash
   sudo apt update
   sudo apt install php php-cli php-mysql php-mbstring php-xml php-curl php-zip
   ```  
2. Установка Composer:  
   ```bash
   curl -sS https://getcomposer.org/installer | php
   sudo mv composer.phar /usr/local/bin/composer
   ```  

### **1.2. Установка Laravel**  
```bash
composer global require laravel/installer
laravel new clover
cd clover
```  

### **1.3. Настройка окружения**  
Скопировать `.env.example` в `.env` и настроить:  
```bash
cp .env.example .env
php artisan key:generate
```  

## **2. Развертывание MySQL и подключение к Laravel**  

### **2.1. Установка MySQL**  
```bash
sudo apt install mysql-server
sudo mysql_secure_installation
```  

### **2.2. Создание базы данных**  
```bash
sudo mysql -u root -p
CREATE DATABASE clover_db;
CREATE USER 'laravel_user'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON laravel_db.* TO 'laravel_user'@'localhost';
FLUSH PRIVILEGES;
```  

### **2.3. Подключение MySQL к Laravel**  
В файле `.env` указать:  
```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_db
DB_USERNAME=laravel_user
DB_PASSWORD=password
```  

Затем выполнить миграции:  
```bash
php artisan migrate
```  

## **3. Установка Redis и подключение к Laravel**  

### **3.1. Установка Redis**  
```bash
sudo apt install redis-server
sudo systemctl enable redis
sudo systemctl start redis
```  

### **3.2. Подключение Redis к Laravel**  
Установить пакет Predis:  
```bash
composer require predis/predis
```  

В `.env` указать:  
```env
REDIS_HOST=127.0.0.1
REDIS_PORT=6379
```  

## **4. Установка Elasticsearch и подключение к Laravel**  

### **4.1. Установка Elasticsearch**  
```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list
sudo apt update
sudo apt install elasticsearch
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch
```  

### **4.2. Подключение Elasticsearch к Laravel**  
Установить пакет `elasticsearch/elasticsearch`:  
```bash
composer require elasticsearch/elasticsearch
```  

В `.env` добавить:  
```env
ELASTICSEARCH_HOST=localhost:9200
```  

## **5. Настройка Flutter (Dart) и подключение Firebase**  

### **5.1. Установка Flutter**  
1. Скачать Flutter SDK:  
   ```bash
   wget https://storage.googleapis.com/flutter_infra_release/releases/stable/linux/flutter_linux_3.13.0-stable.tar.xz
   tar xf flutter_linux_3.13.0-stable.tar.xz
   export PATH="$PATH:`pwd`/flutter/bin"
   ```  
2. Проверить установку:  
   ```bash
   flutter doctor
   ```  

### **5.2. Подключение Firebase к Flutter**  
1. Создать проект в [Firebase Console](https://console.firebase.google.com).  
2. Добавить приложение (Android/iOS).  
3. Установить пакет `firebase_core`:  
   ```bash
   flutter pub add firebase_core
   ```  
4. Добавить конфигурацию (`google-services.json` для Android / `GoogleService-Info.plist` для iOS).  
5. Инициализировать Firebase в `main.dart`:  
   ```dart
   import 'package:firebase_core/firebase_core.dart';
   
   void main() async {
     WidgetsFlutterBinding.ensureInitialized();
     await Firebase.initializeApp();
     runApp(MyApp());
   }
   ```  

## **6. Тестирование развертывания**  
- **Laravel:** Запустить `php artisan serve` и проверить `http://localhost:8000`.  
- **MySQL:** Проверить подключение через `php artisan db:show`.  
- **Redis:** Проверить командой `redis-cli ping` (должен ответить `PONG`).  
- **Elasticsearch:** Проверить `curl http://localhost:9200`.  
- **Flutter:** Запустить `flutter run` и проверить работу Firebase.  

## **Вывод**  
Все компоненты успешно развернуты и интегрированы. Программный комплекс готов к дальнейшей разработке и тестированию.

### **Отчет по настройке и развертыванию программного комплекса**  

#### **Серверная часть (Backend)**  
Для работы серверной части на Laravel требуется установка PHP 8.0+, Composer и веб-сервера (Apache/Nginx). После настройки окружения создается новый проект, генерируется ключ приложения, и настраивается файл `.env`.  

База данных MySQL развертывается через установку пакета `mysql-server`, создание пользователя и БД. В Laravel подключение к MySQL настраивается в `.env`, после чего выполняются миграции для инициализации таблиц.  

Для кэширования и работы с очередями используется Redis, который устанавливается как системная служба. Laravel подключается к нему через пакет `predis/predis` с указанием хоста и порта в конфигурации.  

Elasticsearch добавляется для полнотекстового поиска. После установки и запуска службы в Laravel подключается клиентская библиотека `elasticsearch/elasticsearch`, а в `.env` прописывается хост.  

#### **Клиентская часть (Frontend/Mobile)**  
Flutter устанавливается через загрузку SDK и добавление его в PATH. После проверки работоспособности через `flutter doctor` проект подключается к Firebase.  

В Firebase Console создается проект, добавляется конфигурация для Android/iOS (`google-services.json` или `GoogleService-Info.plist`). В Dart-коде инициализируется Firebase Core, что позволяет использовать все сервисы Firebase (аутентификацию, базу данных, аналитику и т. д.).  

#### **Тестирование работоспособности**  
Серверная часть проверяется запуском Laravel (`php artisan serve`), тестированием подключения к MySQL, Redis (`redis-cli ping`) и Elasticsearch (`curl localhost:9200`). Клиентская часть запускается через `flutter run` с проверкой интеграции с Firebase.  

Все компоненты успешно развернуты и готовы к дальнейшей разработке.